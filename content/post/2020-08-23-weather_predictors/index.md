---
title: "Weather based predictors for predictive modeling"
author: "Bhim Chaulagain"
date: 2020-03-10T21:13:14-05:00
categories: ["Agronomic Intelligence Lab"]
tags: ["weather variables", "lubridate", "dplyr"]
summary: "Create weather predictors for Agronomic Intelligence using R"
image:
  caption: ""
  focal_point: "Center"
  preview_only: true
---

Weather-derived predictors play a fundamental role in plant disease predictive modeling and epidemiological analyses. During my research on sugarcane rust diseases, I developed multiple weather-based variables from high-frequency meteorological data to support disease forecasting and model development.

In this post, I share several R scripts for generating commonly used weather predictors, including hourly, daily, weekly, and monthly summaries, as well as variables derived from specific periods within the day. Future posts will cover more advanced and biologically relevant weather variables used in disease epidemiology and predictive modeling workflows.


```r
library(readxl)
library(lubridate)
library(plyr)

Wdata <- read_excel("weatherdata.xlsx")
head (Wdata, 10)
str(Wdata)
Wdata$Period <- as.POSIXct(Wdata$Period, format = "%m/%d/%Y %H: %M: %S")
str(Wdata)

#Create Multiple groups to split the data into Days, Weeks and Month

Wdata$days <- floor_date(Wdata$Period, "day")
Wdata$weeks <- floor_date(Wdata$Period, 'week')
Wdata$month <- floor_date(Wdata$Period, 'month')
Wdata$hours <- floor_date(Wdata$Period, 'hours')
```

Lets calculate the weekly average of the temperature. You can use same process for other weather variables just by changing the column name and sums by changing the function name.

Weekly average of temperature!

```r
WeeklyAvg_temp <- aggregate(Wdata$`2m T avg (F)`, by = list(Wdata$weeks), FUN = mean)

WeeklyAvg_temp
```
Lets calculate the weekly average of the temperature from(6 AM-9 PM) of each day.

Use lubridate package for subsetting the data which contains the values from 6 AM to 9 PM. 
```r
T6AMTo9PM <- subset(Wdata, hour(hours) >= 6 & hour(hours) <= 21 & days >= '2014-01-01' & days <= '2014-12-31')

```
After subsetting, lets calculate for temperature variable!

```r
T6AMTo9PM_WeeklyAvg_temp <- aggregate(T6AMTo9PM$`2m T avg (F)`, by = list(T6AMTo9PM$weeks), FUN = mean)
T6AMTo9PM_WeeklyAvg_temp
```

