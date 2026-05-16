---
title: "How to Estimate Leaf Wetness Duration (LWD) Using the CART Method in R"
author: "Bhim Chaulagain"
date: 2020-06-08T21:13:14-05:00
categories: ["Agronomic Intelligence Lab"]
tags: ["leaf wetness duration", "CART method", "plant disease forecasting", "R", "weather data"]
summary: "A practical R walkthrough for estimating leaf wetness duration (LWD) with the CART method for plant disease forecasting and risk modeling."
description: "A practical R walkthrough for estimating leaf wetness duration (LWD) with the CART method for plant disease forecasting and risk modeling."
image:
  caption: ""
  focal_point: "Center"
  preview_only: true
---

Leaf wetness duration (LWD) is a critical environmental variable in plant disease epidemiology and is widely used in disease forecasting and risk assessment models. Among the available empirical approaches, the Classification and Regression Tree (CART) method provides a practical framework for estimating LWD from standard weather observations.

In this post, I demonstrate how to estimate leaf wetness duration in R using the CART approach, including data preparation, implementation steps, and interpretation of outputs for plant disease applications.

If you want to connect leaf wetness duration to a full infection-risk workflow, see [How to Code the Magarey Generic Infection Model for Foliar Fungal Plant Pathogens in Python]({{< relref "/post/2023-02-01-generic_infection_risks/index.md" >}}).

Let's generate the dummy weather data and work through it. You can find the equation and procedure of CART method of leaf wetness estimation in this link: https://doi.org/10.1094/PDIS.2002.86.2.179

```r
Average_temperature <- sample(2:25, 20, replace=TRUE)
Dew_Point_Depression <- sample(0.5:5, 20, replace=TRUE)
Relative_Humidity <- sample(55:99, 20, replace=TRUE)
Windspeed <- sample(1.5:6, 20, replace=TRUE)

data <- data.frame(Average_temperature, Dew_Point_Depression, Relative_Humidity, Windspeed)

names(data)[1:4] <- c('tair', 'dpd', 'rh', 'wspeed') 

head(data, 10)

#Use equations for CART method of leaf wetness estimation
wetdry <- numeric(dim(data)[1])
for (i in 1:length(wetdry)) {
  c1 <- 1.6064 * sqrt(data$tair[i]) + 0.0036 * data$tair[i]^2 + 
    0.1531 * data$rh[i] - 0.4599 * data$wspeed[i] * data$dpd[i] - 0.0035 * data$tair[i] * data$rh[i]
  c2 <- 0.7921 * sqrt(data$tair[i]) + 0.0046 * data$rh[i] - 2.3889 * data$wspeed[i] - 
    0.0390 * data$tair[i] * data$wspeed[i] + 1.0613 * data$wspeed[i] * data$dpd[i]
  
  if (data$dpd[i] >= 3.7) {
    wetdry[i] <- 'D'
  } else if (data$wspeed[i] < 2.5 & c1 > 14.4674) {
    wetdry[i] <- 'W'
  } else if (data$wspeed[i] < 2.5 & c1 <= 14.4674) {
    wetdry[i] <- 'D'
  } else if (data$rh[i] >= 87.8 & c2 > 37.0) {
    wetdry[i] <- 'W'
  } else {
    wetdry[i] <- 'D'
  }
}

wetdry
```


