---
layout: post    
title: Basic Climate Modeling with ARIMA
---

*Wihin this post, we will continue our study of CO2 with ice core data with a range of 800,000 years*

**Where we left off**  
In the previous post, we developed a simple sinusoidal regression model to approximate the data. 
Simple indeed!  As we can see, the sinusoidal is unable to capture the patterns of the data and is quite underfit.
![plot6](https://github.com/julialintern/julialintern.github.io/raw/master/images/Plot_6.png)

**Why ARIMA?** 

It may seem that there are so many different flavors and approaches of time series models including.
There are models that use lagged features, differenced features, random walks, etc.  However, there is one type of time series
model that can combine all of these options: the ARIMA model.
ARIMA, Autoregressive Integrated Moving Average, is a time series model that incorporates 
both autoregressive and moving average features, while detrending the data.  
Autoregressive features are simply lagged features, where moving average features are generated from past error terms.
The 'integrated' component of ARIMA is the required detrending step.

