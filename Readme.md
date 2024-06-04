# Proposal for Semester Project

```{=html}
<!-- 
Please render a pdf version of this Markdown document with the command below (in your bash terminal) and push this file to Github

quarto render Readme.md --to pdf
-->
```
**Patterns & Trends in Environmental Data / Computational Movement Analysis Geo 880**

| Semester:      | FS24                                                                             |
|:---------------|:---------------------------------------------------------------------------------|
| **Data:**      | GPX data related to running workouts measured with a Garmin Forerunner 245 watch |
| **Title:**     | Identification of patterns and trends in running training performance            |
| **Student 1:** | Daniel Cellerino                                                                 |
| **Student 2:** | Sophie Blatter                                                                   |

## Abstract

<!-- (50-60 words) -->

## Research Questions

<!-- (50-60 words) -->

Research Question 1: On which terrain levels (flat, moderate, steep) does Daniel gain or lose time compared to the average speed for these sections across different runs on the same route?

Research Question 2: Considering the slope of the terrain, does the frequency of curves on the track affect the speed?

(löschen) (ich gluabe ist schwierig zum machen und auch nicht so wichtig für diese Modul): Research Question 2: How does the temperature, precipitation and wind on the day of the run affect Daniel's overall speed on the same route over different years?

## Results / products

<!-- What do you expect, anticipate? -->

Research Question 1:

For the first research question we anticipate that Daniel's performance will vary significantly across different terrain levels. Specifically, we expect:

-   Daniel will likely maintain a more consistent and higher average speed on flat sections, with minimal time deviation from the average across different runs. We expect that Daniel will get faster over time, since he gets more experienced in running.

-   On moderate terrain, there may be a slight reduction in speed compared to flat sections, with some variability depending on specific conditions during each run.

-   The steep sections are expected to show the greatest variability in speed and the most significant time deviations. In these sections, physical conditions could be of significant importance and have a greater impact on the variability of the three analyzed runs.

We anticipate to identify specific sections where Daniel can focus his training to improve performance.

Research question 2:

-   For the second research question, we expect to find that in sections with steep inclines, the tortuosity of the path does not influence the running speed. On terrain with moderate or flat slopes, where speeds will be higher, we expect that numerous changes in direction, at the same incline, may affect the running speed.

(Löschen)

-   For the second research question we expect to find that higher temperatures may correlate with slower overall speeds due to increased physical stress. Conversely, moderate, cooler temperatures might be associated with faster speeds.

-   Runs on days with precipitation (rain, snow) are expected to show a decrease in overall speed due to slippery surfaces and bad vision.

-   Strong headwinds could slow down Daniel's runs, while tailwinds might help increase his speed.

We anticipate that adverse weather conditions will generally correlate with slower run times, while more favorable weather conditions will support faster overall speeds.

## Data

<!-- What data will you use? Will you require additional context data? Where do you get this data from? Do you already have all the data? -->

We will use movement data recorded by Daniel on Strava over several years. From this data, we will focus on runs that Daniel has done on the same route in different years. We will divide the route into sections based on three terrain levels (flat, moderate, steep) and compare the average speed in these sections across different runs. The terrain levels we will conclude by mapping the route on arcGIS and adding a layer with terrain information from Swisstopo.

(löschen) The weather data with information on temperature, wind and precipitation we will get from the archive of Swiss Meteo.

## Analytical concepts

In the semester project, we will use movements in space with discrete properties and the conceptual model of Network Space in three dimensions. The movement we will analyze was actively performed by Daniel in a confined area that uniformly includes trails and roads. The observation perspective will be active tracking performed using a Garmin Forerunner 245 watch with an integrated GPS sensor, with measurements taken at irregular intervals. <!-- Which analytical concepts will you use? What conceptual movement spaces and respective modelling approaches of trajectories will you be using? What additional spatial analysis methods will you be using? -->

## R concepts

First, we will import the GPX files into RStudio. We will change the coordinate system and measure the average speed between the measured points. Next, we want to measure the sinuosity using a time window. The problem is that our points are measured at irregular intervals. Therefore, we are considering inserting points at regular intervals artificially along the running trajectory. After that, we need to assign an elevation above sea level to each point. Then, we can categorize the slope and sinuosity of the route into three levels. ...

How do we then identify the patterns? Do we want to see if the speed decreases with the percentage increase in slope? Or do we only want to see a difference between the three categories? The same applies to sinuosity. Furthermore, how do we want to correct the running data? The points will not be identical due to GPS accuracy and precision errors. Which method do we want to use to correct them to make them homogeneous?

<!-- Which R concepts, functions, packages will you mainly use. What additional spatial analysis methods will you be using? -->

## Risk analysis

We need to decide whether to also consider the downhill sections of the run. In that case, the speed decreases compared to running on flat terrain. In this scenario, we could possibly create another category for the downhill sections.

It is likely that due to the GPS accuracy, it will not be possible to measure the sinuosity of the path since the measurement intervals are irregular and the watch's accuracy in optimal conditions is 3-5 meters. <!-- What could be the biggest challenges/problems you might face? What is your plan B? -->

## Questions?

How can we work with data that has been measured at irregular intervals? How can we add altitude above sea level to our points? How can we calculate sinuosity?

<!-- Which questions would you like to discuss at the coaching session? -->
