

``` r
# Als erstes: Die Dateien sind im FIT Format: wir brauchen das Package FITfileR
# Install and load necessary libraries
if (!requireNamespace("rnaturalearth", quietly = TRUE)) {
  install.packages("rnaturalearth")
}

if (!requireNamespace("tmap", quietly = TRUE)) {
  install.packages("tmap")
}

if (!requireNamespace("sf", quietly = TRUE)) {
  install.packages("sf")
}

if (!requireNamespace("ggplot2", quietly = TRUE)) {
  install.packages("ggplot2")
}

if (!requireNamespace("dplyr", quietly = TRUE)) {
  install.packages("dplyr")
}

if (!requireNamespace("readr", quietly = TRUE)) {
  install.packages("readr")
}

if (!requireNamespace("tidyverse", quietly = TRUE)) {
  install.packages("tidyverse")
}

if (!requireNamespace("raster", quietly = TRUE)) {
  install.packages("raster")
}

if (!requireNamespace("leaflet", quietly = TRUE)) {
  install.packages("leaflet")
}

if (!requireNamespace("FITfileR", quietly = TRUE)) {
  if (!requireNamespace("remotes", quietly = TRUE)) {
    install.packages("remotes")
  }
  remotes::install_github("grimbough/FITfileR")
}

library(rnaturalearth)
library(tmap)
library(FITfileR)
library(sf)
library(ggplot2)
library(raster)
library(dplyr)
library(readr)
library(tidyverse)
library(leaflet)



#########################################################################################
# Definire eine Funktion zum konvertieren FIT-Datensätze mit geografischen Koordinaten in ein `sf`-Format.
fit2sf <- function(fit_path){
  
  fit_data <- readFitFile(fit_path)
  records <- records(fit_data)
  
  
  imap(records, function(x,y){
    if ("position_lat" %in% names(x) && "position_long" %in% names(x)) {
      
      st_as_sf(x, coords = c("position_long", "position_lat"), crs = 4326) |> 
        mutate(record = y)
    } else {
      NULL  # Gibt NULL zurück, wenn keine Koordinaten vorhanden sind
    }
  }) -> all_records
  
  bind_rows(all_records)
}

# Funktion zum Extrahieren von .fit-Dateien aus .gz-Dateien
extract_fit_from_gz <- function(gz_file, output_dir = "extracted_fit_files") {
  if (!dir.exists(output_dir)) {
    dir.create(output_dir)
  }
  
  # Voller Pfad für die Extraktion erstellen
  fit_file_path <- file.path(output_dir, gsub(".gz$", ".fit", basename(gz_file)))
  
  # Öffne die .gz-Datei und schreibe den Inhalt in eine .fit-Datei
  con <- gzfile(gz_file, "rb")
  fit_data <- readBin(con, what = raw(), n = 1e6)
  close(con)
  
  writeBin(fit_data, fit_file_path)
  
  return(fit_file_path)
}

# CSV-Datei mit Informationen zu Aktivitäten lesen
activity_info <- read_csv("export/activities.csv")
```

```
## Error: 'export/activities.csv' does not exist in current working directory ('C:/Users/sophi/OneDrive/Dokumente/ZHAW/Pattern and Trends/Semesterproject/Project_Daniel_Sophie').
```

``` r
# Laufaktivitäten finden und ihre Dateinamen extrahieren
running_activities <- activity_info %>%
  filter(`Tipo attività` == "Corsa") %>%
  pull(`Nome del file`)
```

```
## Error in eval(expr, envir, enclos): Objekt 'activity_info' nicht gefunden
```

``` r
# Unterordner im Verzeichnis "export/activities" finden
sub_dirs <- list.dirs(path = "export/activities", full.names = TRUE, recursive = TRUE)

# Alle .gz-Dateien in diesen Unterordnern finden
gz_files <- unlist(lapply(sub_dirs, function(dir) {
  list.files(path = dir, pattern = "\\.gz$", full.names = TRUE)
}))

# .fit-Dateien aus jeder .gz-Datei extrahieren
fit_files <- lapply(gz_files, extract_fit_from_gz)

# Filtern der NULL-Dateien und sicherstellen, dass die Pfade Zeichenvektoren sind
fit_files <- unlist(fit_files[!sapply(fit_files, is.null)])

# Dateinamen der vorhandenen .fit-Dateien im Verzeichnis "extracted_fit_files" finden
existing_fit_names <- list.files(path = "extracted_fit_files", pattern = "\\.fit$", full.names = TRUE)

# Vollständige Pfade der Dateien relativ zum Verzeichnis "extracted_fit_files" erstellen
existing_fit_paths <- file.path("extracted_fit_files", basename(existing_fit_names))

# Nur die Dateinamen der .fit-Dateien aus der Liste der vollständigen Pfade extrahieren und Pfad und Erweiterung entfernen
existing_fit_names <- tools::file_path_sans_ext(basename(existing_fit_paths))

# Nur die Dateinamen der Laufaktivitäten extrahieren und Pfad und Erweiterung entfernen
running_activity_names <- tools::file_path_sans_ext(basename(running_activities))
```

```
## Error in eval(expr, envir, enclos): Objekt 'running_activities' nicht gefunden
```

``` r
# Nur die .fit-Dateien behalten, die Laufaktivitäten entsprechen
fit_files_to_keep <- existing_fit_paths[existing_fit_names %in% running_activity_names]
```

```
## Error in eval(expr, envir, enclos): Objekt 'running_activity_names' nicht gefunden
```

``` r
# Nicht benötigte .fit-Dateien im Verzeichnis "extracted_fit_files" entfernen
files_to_remove <- setdiff(existing_fit_paths, fit_files_to_keep)
```

```
## Error in eval(expr, envir, enclos): Objekt 'fit_files_to_keep' nicht gefunden
```

``` r
for (fit_file in files_to_remove) {
  print(paste("Löschen der Datei:", fit_file))
  file.remove(fit_file)
}
```

```
## Error in eval(expr, envir, enclos): Objekt 'files_to_remove' nicht gefunden
```

``` r
#################################################################################################
#Importiere alle FIT-Dateien in RStudio und erstelle eine Liste
fit_files <- list.files(path = "extracted_fit_files/", pattern = "\\.fit$", full.names = TRUE)

############# Es braucht 10-15 min ###############################################################
combined_data <- map(fit_files[], function(file) {
  # Konvertiere FIT-Datei in ein Datenframe
  sf_data <- fit2sf(file)
  
  # Benenne Spalten um, wenn erforderlich
  colnames(sf_data) <- gsub("enhanced_speed", "speed", colnames(sf_data))
  colnames(sf_data) <- gsub("enhanced_altitude", "altitude", colnames(sf_data))
  
  # Überprüfe und entferne 'temperature' und 'heart_rate' Spalten, falls vorhanden
  if (any(c("temperature", "heart_rate") %in% colnames(sf_data))) {
    sf_data <- sf_data %>% dplyr::select(-any_of(c("temperature", "heart_rate")))
  }
  
  # Gib den verarbeiteten Datenframe zurück
  return(sf_data)
}, .progress = TRUE)

# Leeren Dataframes entfernen
combined_data_clean <- Filter(function(df) ncol(df) > 0, combined_data)
combined_data_sf <- do.call(rbind, combined_data_clean)

###############################################################################################
# Laden das DataFrame mit den LV95-Koordinaten
# Funzione per trasformare le coordinate in LV95
transform_to_lv95 <- function(df) {
  st_transform(df, crs = 2056)
}

# Trasforma tutti i data frame sf con geometrie di tipo POINT in coordinate LV95
sf_data_points_lv95 <- lapply(combined_data_clean, transform_to_lv95)

#eliminare # Unisci tutti i data frame trasformati in uno solo
combined_data_sf_lv95 <- bind_rows(sf_data_points_lv95)

# 1. Lade die Grenzen der Schweiz herunter und lade sie.
swiss_boundary <- ne_countries(scale = "large", country = "Switzerland", returnclass = "sf")

# 2. Transformiere die Grenzen der Schweiz gegebenenfalls in LV95
swiss_boundary <- st_transform(swiss_boundary, crs = 2056)

# 3. Filtrieren Geometrien ausserhalb der Grenzen der Schweiz
filtered_data <- st_intersection(combined_data_sf_lv95, swiss_boundary)
```

```
## Error in UseMethod("st_intersection"): nicht anwendbare Methode für 'st_intersection' auf Objekt der Klasse "c('tbl_df', 'tbl', 'data.frame')" angewendet
```

``` r
# 4. Wähle nur die gewünschten Spalten aus
filtered_data <- filtered_data[, c("timestamp", "distance", "speed", "altitude", "geometry")]
```

```
## Error in eval(expr, envir, enclos): Objekt 'filtered_data' nicht gefunden
```

``` r
# Zeige die gefilterten Daten auf einer Karte an
plot(filtered_data)
```

```
## Error in h(simpleError(msg, call)): Fehler bei der Auswertung des Argumentes 'x' bei der Methodenauswahl für Funktion 'plot': Objekt 'filtered_data' nicht gefunden
```

``` r
tm_shape(filtered_data)  +tm_dots()+ tmap_mode("view")
```

```
## Error in eval(expr, envir, enclos): Objekt 'filtered_data' nicht gefunden
```

``` r
# Zähle die Anzahl der Trainings-Tage
filtered_data$timestamp <- as.POSIXct(filtered_data$timestamp)
```

```
## Error in eval(expr, envir, enclos): Objekt 'filtered_data' nicht gefunden
```

``` r
date_uniq <- unique(as.Date(filtered_data$timestamp))
```

```
## Error in h(simpleError(msg, call)): Fehler bei der Auswertung des Argumentes 'x' bei der Methodenauswahl für Funktion 'unique': Objekt 'filtered_data' nicht gefunden
```

``` r
num_days <- length(date_uniq)
```

```
## Error in eval(expr, envir, enclos): Objekt 'date_uniq' nicht gefunden
```

``` r
print(num_days)
```

```
## Error in h(simpleError(msg, call)): Fehler bei der Auswertung des Argumentes 'x' bei der Methodenauswahl für Funktion 'print': Objekt 'num_days' nicht gefunden
```

``` r
#[1] 165

########################################################################################
#################################################
# Fragenstellung 1
# Importieren Sie die erforderlichen Bibliotheken
library(dplyr)
library(lubridate)
library(ggplot2)


# Funktion zur Berechnung der Zeitdifferenz in Sekunden
difftime_secs <- function(x, y) {
  as.numeric(difftime(x, y, units = "secs"))
}

# Funktion zur Berechnung der Entfernung zwischen aufeinanderfolgenden Elementen
distance_by_element <- function(later, now) {
  as.numeric(st_distance(later, now, by_element = TRUE))
}

# Funktion zur Berechnung der Steigung
calculate_slope <- function(alt1, alt2, dist) {
  (alt2 - alt1) / dist
}

# Daten nach aufsteigendem Zeitstempel sortieren 
filtered_data <- filtered_data %>%
  arrange(timestamp)
```

```
## Error in eval(expr, envir, enclos): Objekt 'filtered_data' nicht gefunden
```

``` r
# Initialisieren von cumulative_distance und run
filtered_data$cumulative_distance <- 0
```

```
## Error: Objekt 'filtered_data' nicht gefunden
```

``` r
filtered_data$run <- 1
```

```
## Error: Objekt 'filtered_data' nicht gefunden
```

``` r
# Iterieren durch die Zeilen und Aktualisieren von cumulative_distance
### Es braucht 1h oder 1.5 h ######################################
# Der Abstand zwischen den Punkten weicht leicht von den von STRAVA importierten Abstandswerten ab. Der Grund dafür liegt wahrscheinlich in den unterschiedlichen Koordinatensystemen. Jetzt wird der Abstand mit dem LV95-System neu berechnet.
for (i in 2:nrow(filtered_data)) {
  if (difftime_secs(filtered_data$timestamp[i], filtered_data$timestamp[i-1]) > 1800) {
    filtered_data$cumulative_distance[i] <- 0
    filtered_data$run[i] <- filtered_data$run[i-1] + 1
  } else {
    filtered_data$cumulative_distance[i] <- filtered_data$cumulative_distance[i-1] + distance_by_element(filtered_data$geometry[i], filtered_data$geometry[i-1])
    filtered_data$run[i] <- filtered_data$run[i-1]
  }
}
```

```
## Error in h(simpleError(msg, call)): Fehler bei der Auswertung des Argumentes 'x' bei der Methodenauswahl für Funktion 'nrow': Objekt 'filtered_data' nicht gefunden
```

``` r
# Spalten für Steplength, Höhenänderung und Steigungsprozent hinzufügen
filtered_data <- filtered_data %>%
  mutate(
    next_geometry = lead(geometry),
    next_altitude = lead(altitude),
    steplength = distance_by_element(next_geometry, geometry),
    altitude_change = next_altitude - altitude,
    slope_percent = (altitude_change / steplength) * 100
  ) %>%
  filter(steplength != 0) # Doppelte Punkte bei Bedarf entfernen
```

```
## Error in eval(expr, envir, enclos): Objekt 'filtered_data' nicht gefunden
```

``` r
# Segmente von 300 Metern innerhalb jedes Laufs nummerieren
filtered_data <- filtered_data %>%
  group_by(run) %>%
  mutate(
    segment = floor(cumulative_distance / 300) + 1
  ) %>%
  ungroup()
```

```
## Error in eval(expr, envir, enclos): Objekt 'filtered_data' nicht gefunden
```

``` r
# Zeilen zusammenführen, die zum gleichen Lauf und Segment gehören, und die erforderlichen Metriken berechnen
segment_summary <- filtered_data %>%
  group_by(run, segment) %>%
  summarize(
    start_time = min(timestamp),
    end_time = max(timestamp),
    elapsed_time = difftime(max(timestamp), min(timestamp), units = "secs"),
    start_distance = min(cumulative_distance),
    end_distance = max(cumulative_distance),
    segment_distance = max(cumulative_distance) - min(cumulative_distance),
    mean_slope_percent = mean(slope_percent, na.rm = TRUE),
    mean_speed = (as.numeric(elapsed_time) / 60) / (segment_distance / 1000) # Durchschnittsgeschwindigkeit in min/km
  ) %>%
  ungroup() %>%
  filter(segment_distance >= 250) # Zeilen mit segment_distance >= 250 filtern
```

```
## Error in eval(expr, envir, enclos): Objekt 'filtered_data' nicht gefunden
```

``` r
# Spalte für Steigungskategorie hinzufügen
segment_summary <- segment_summary %>%
  mutate(
    slope_category = case_when(
      mean_slope_percent > 8 ~ "Steiler Anstieg",         # Größer als 8%
      mean_slope_percent > 2 & mean_slope_percent <= 8 ~ "Sanfter Anstieg",   # Zwischen 2% und 8%
      mean_slope_percent > -2 & mean_slope_percent <= 2 ~ "Flach",       # Zwischen -2% und +2%
      mean_slope_percent < -8 ~ "Steiler Abstieg",       # Kleiner als -8%
      mean_slope_percent < -2 & mean_slope_percent >= -8 ~ "Sanfter Abstieg"  # Zwischen -2% und -8%
    )
  )
```

```
## Error in eval(expr, envir, enclos): Objekt 'segment_summary' nicht gefunden
```

``` r
# Ergebnis anzeigen
print(segment_summary)
```

```
## Error in h(simpleError(msg, call)): Fehler bei der Auswertung des Argumentes 'x' bei der Methodenauswahl für Funktion 'print': Objekt 'segment_summary' nicht gefunden
```

``` r
# Segmente entfernen, die nicht auf eine Laufaktivität zurückzuführen sind, Geschwindigkeit grösser als 15 min/km (sehr langsam, möglicherweise gab es Pausen durch Drücken der Stopp-Taste auf der Uhr).
segment_summary_run <- segment_summary %>%
  filter(mean_speed <= 15)
```

```
## Error in eval(expr, envir, enclos): Objekt 'segment_summary' nicht gefunden
```

``` r
# Erstellen des Boxplots
library(ggplot2)

plot2 <- ggplot(segment_summary_run, aes(x = slope_category, y = mean_speed)) +
  geom_boxplot(fill = "skyblue", color = "steelblue") +
  labs(
    title = "Geschwindigkeit über Steigungskategorien",
    x = "Steigungskategorie",
    y = "Durchschnittsgeschwindigkeit (min/km)"
  ) +
  theme_minimal()
```

```
## Error in eval(expr, envir, enclos): Objekt 'segment_summary_run' nicht gefunden
```

``` r
# Mediane, Standardabweichung, Gesamtanzahl der Segmente und durchschnittliche Steigungsprozente für jede Steigungskategorie berechnen
# Mediane, Standardabweichung, Gesamtanzahl der Segmente und durchschnittliche Steigungsprozente für jede Steigungskategorie berechnen
summary_stats1 <- segment_summary_run %>%
  select(-geometry) %>%  # Spalte 'geometry' ausschließen
  group_by(slope_category) %>%
  summarize(
    median = median(mean_speed),
    standardabweichung = sd(mean_speed),
    segmentanzahl = n(),  # Gesamtzahl der Segmente für diese Kategorie zählen
    durchschnittliche_steigungsprozente = mean(mean_slope_percent, na.rm = TRUE)  # Durchschnittliche Steigungsprozente berechnen
  ) %>%
  ungroup()
```

```
## Error in eval(expr, envir, enclos): Objekt 'segment_summary_run' nicht gefunden
```

``` r
# Zusammenfassungstabelle anzeigen
library(knitr)
library(kableExtra)
```

```
## Error in library(kableExtra): es gibt kein Paket namens 'kableExtra'
```

``` r
# Entferne die Spalte geometry und konvertiere sie in ein Data Frame
table1 <- st_drop_geometry(summary_stats1)
```

```
## Error in eval(expr, envir, enclos): Objekt 'summary_stats1' nicht gefunden
```

``` r
# Erstelle eine formatierte Tabelle
table1 <- kable(table1, caption = "Zusammenfassung der Geschwindigkeit über Steigungskategorien") %>%
  kable_styling(bootstrap_options = "striped", full_width = F)
```

```
## Error in kable_styling(., bootstrap_options = "striped", full_width = F): konnte Funktion "kable_styling" nicht finden
```

``` r
##################################################################################################
# Fragenstellung 2
# Berechnen der Gesamtanzahl der Segmente für jeden Lauf
segment_summary_run <- segment_summary_run %>%
  group_by(run) %>%
  mutate(total_segments = max(segment)) %>%
  ungroup()
```

```
## Error in eval(expr, envir, enclos): Objekt 'segment_summary_run' nicht gefunden
```

``` r
# Hinzufügen einer Spalte zur Identifizierung der Segmentphase
segment_summary_run <- segment_summary_run %>%
  mutate(
    phase = case_when(
      segment <= floor(total_segments / 3) ~ "Anfang",      # Erstes Drittel der Segmente
      segment > floor(total_segments / 3) & 
        segment <= 2 * floor(total_segments / 3) ~ "Mitte", # Zweites Drittel der Segmente
      segment > 2 * floor(total_segments / 3) ~ "Ende"    # Letztes Drittel der Segmente
    )
  ) %>%
  select(-total_segments)  # Entfernen der temporären Spalte total_segments
```

```
## Error in eval(expr, envir, enclos): Objekt 'segment_summary_run' nicht gefunden
```

``` r
# Nun enthält segment_summary_run eine neue Spalte 'phase', die anzeigt, ob sich jedes Segment am Anfang, in der Mitte oder am Ende des Trainings für jede Steigungskategorie (slope_category) befindet.

library(ggplot2)


# Berechnung der Durchschnittsgeschwindigkeit für jede Kombination aus slope_category und phase
segment_summary_run2 <- segment_summary_run %>%
  group_by(slope_category, phase) %>%
  summarize(
    mean_speed = mean(mean_speed, na.rm = TRUE),
    .groups = 'drop'  # Vermeiden der Warnung "summarize() ungrouping output"
  )
```

```
## Error in eval(expr, envir, enclos): Objekt 'segment_summary_run' nicht gefunden
```

``` r
# Erstellen des Diagramms mit den Durchschnittsgeschwindigkeiten
plot3 <- ggplot(segment_summary_run2, aes(x = slope_category, y = mean_speed, fill = phase)) +
  geom_bar(stat = "identity", position = "dodge") +
  labs(
    title = "Geschwindigkeit nach Steigungskategorie und Segmentphase",
    x = "Steigungskategorie",
    y = "Durchschnittsgeschwindigkeit (min/km)",
    fill = "Segmentphase"
  ) +
  theme_minimal() +
  theme(axis.text.x = element_text(angle = 45, hjust = 1))  # Drehen der Beschriftungen auf der x-Achse für eine bessere Lesbarkeit
```

```
## Error in eval(expr, envir, enclos): Objekt 'segment_summary_run2' nicht gefunden
```

``` r
plot3
```

```
## Error in eval(expr, envir, enclos): Objekt 'plot3' nicht gefunden
```

``` r
# Berechnung der Zusammenfassungsstatistiken
summary_stats2 <- segment_summary_run %>%
  group_by(slope_category, phase) %>%
  summarize(
    median = median(mean_speed, na.rm = TRUE),
    standardabweichung = sd(mean_speed, na.rm = TRUE),
    segmentanzahl = n(),  # Gesamtzahl der Segmente für diese Kombination aus Kategorie und Phase zählen
    durchschnittliche_steigungsprozente = mean(mean_slope_percent, na.rm = TRUE),  # Durchschnittliche Steigungsprozente berechnen
    .groups = 'drop'  # Vermeiden der Warnung "summarize() ungrouping output"
  )
```

```
## Error in eval(expr, envir, enclos): Objekt 'segment_summary_run' nicht gefunden
```

``` r
# Entfernen der Spalte geometry (falls vorhanden) und Umwandlung in ein Data Frame
table2 <- st_drop_geometry(summary_stats2)
```

```
## Error in eval(expr, envir, enclos): Objekt 'summary_stats2' nicht gefunden
```

``` r
# Erstellen einer formatierten Tabelle
table2 <- kable(table2, caption = "Zusammenfassung der Geschwindigkeit nach Steigungskategorie und Segmentphase") %>%
  kable_styling(bootstrap_options = "striped", full_width = F)
```

```
## Error in kable_styling(., bootstrap_options = "striped", full_width = F): konnte Funktion "kable_styling" nicht finden
```

``` r
save.image(file = "my_workspace.RData")

source("preprocessing.R ")
```

```
## Error: 'export/activities.csv' does not exist in current working directory ('C:/Users/sophi/OneDrive/Dokumente/ZHAW/Pattern and Trends/Semesterproject/Project_Daniel_Sophie').
```

