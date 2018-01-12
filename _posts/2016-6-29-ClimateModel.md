---
layout: post    
title: Basic Climate Modeling with Time Series (Part 1)
---

*This study will look at co2 measurements from ice core data over the long timescale of 120 - 800,000 years ago.  This co2 dataset perhaps extends back further than any other...*

**Data Source**  
[noaa ice core data](https://www.ncdc.noaa.gov/cdo/f?p=519:1:::::P1_STUDY_ID:15076)

**Why CO2?**  
It has long been established that carbon dioxide and temperature are strongly correlated.  Although the intent of this blog series is to introduce a number of time series techniques, the nature of the this data will inevitably open up some climate-related questions.
Perhaps the first question to emerge is in reference to the controversy surrounding the significant lag of co2 values behind temperature values:  "How could CO2 levels affect global temperature when temperature values changed first?".   
Researchers recently discovered the answer to this question.  Whether in your glass of champagne or in a ice core: *co2 bubbles will rise*.
[http://www.scientificamerican.com/article/ice-core-data-help-solve/](http://www.scientificamerican.com/article/ice-core-data-help-solve/)

Although this presents an additional challenge for climatologists, the lag issue is a fun exercise for modelers like us.
We will study the relationship between temp & co2 further in this series, but for now we will focus the relationship between co2 and time.

**Time Plot**  
Our first step will be to a generate a time plot of our co2 data vs time.
This visualization will indicate the presence of important features such as trend, seasonality and outliers.

![test](https://github.com/julialintern/julialintern.github.io/raw/master/images/Plot_1.png)

**The Quaternary Ice Age**  
The periodic structure of our dataset is due to glacial cycles.  As the plot illustrates, over the last 740,000 years there have
been eight glacial cycles with each cycle lasting roughly 100,000 years.  

However, this data is just a piece of the entire Quaternary Period picture.  The Quaternary ice age started approximately 2.5 million years ago.  The QP will maintain its status as the current ice age as long as one permanent 'large' ice sheet exists.   It's perhaps surprising that we are somehow still living within an ice age (and will continue to) until the last cube of Antartica has melted away.

**Back to our data..**    

*Decomposition:*   Time series analysis can be concerned with decomposing the series into trend, seasonal variation and cyclical components.  One method that deals with trend is the linear filter, which is used to smooth out local fluctuations and estimate local mean (aka: moving average).   This can be useful when a time series is strongly influenced by trend and seasonality.   At first glance of our time plot, one could assume that trend wouldn't play a part in this analysis.  But what we know about the waning current ice age is confirmed by the following moving average plot: CO2 is trending upwards.   
Agreed. This finding may seem obvious, but bear in mind that 99.9% of our data predates the Industrial Revolution with our most recent data point dating 1879!

![test](https://github.com/julialintern/julialintern.github.io/raw/master/images/Plott_3.png)

Alas, trend *could* play a role in this time series. What about seasonality?  
Seasonality is generally assumed to be annual in nature.  However, seasonality is really just periodic data with a known or fixed period. In this case our fixed period happens to be 100,000 years.   We can see from our 1st plot that there is a decent sinusoidal component.
Can we 'idealize' the seasonal component of our model and assume that it can be represented from a simple sinusoidal model ?   Wave forms with a more complicated structure (sinusoidal waves within sinusoidal waves) can be synthesized by using a series of sine and cosine functions whose frequencies are multiples of the fundamental seasonal frequency (in this case: multiples of $$\pi/100,000$$).To do a proper analysis, we want to know what other internal forces may be influencing our data.   Just as with fiscal data, we know that quarterly, monthly and even daily fluctuations are informing the annual time series.  However, understanding the internal forces at play within the QP ice age is a much harder problem.  So hard, that there is a wikipedia page attesting its difficulty.  
[The One Hundred Thousand year Problem:](https://en.wikipedia.org/wiki/100,000-year_problem)   
Essentially, scientists are not even sure of where the 100,000 periodic interval comes from.

**A Simple Sinusoidal**  
So we see that the model can become as complex as we would like to make it.

Perhaps a sound approach would be to start with the most basic model, and add complexity iteratively.
If we sense that a given time series contains a periodic structure ( a sinusoidal component) with a known frequency $$(\omega)$$ then we can apply the following model:

(Eqn 1)  
$$X_t=\mu +\alpha cos(\omega t)+\beta sin(\omega t) + Z_t$$

Expected values for this model can be represented by:  
$$ E(X)= A\theta$$

Where A is:

$$A=\begin{bmatrix}1,\    cos\omega,\, sin\omega \\..\ ..\ \\1,\ cosN\omega,\ sinN\omega \end{bmatrix}$$

And $$\theta$$ is: (Eqn 2)    
$$\hat{\theta} = (A^TA)^{-1}A^Tx$$

Because our Eqn 1 model is linear in parameters (alpha, beta and mu), it is a general linear model.
Eqn 2 allows us to perform a least squares estimate of theta which minimizes:

$$\sum^N_{t=1}(x_t-\mu - \alpha cos \omega t-\beta sin \omega t)^2$$

With a little matrix manipulation we can solve for theta:

$$ \theta = [ 230.94802743,    7.45602447,   -3.18249489]$$  
where  
$$ \mu= 230.94802743, \alpha=7.45602447, \beta = -3.18249489 $$

And now we have the simple sinusoidal plot of E(x):
![plot2](https://github.com/julialintern/julialintern.github.io/raw/master/images/Plot_2.png)

What do you think?  First of all, our most recent period seems to behave differently than the other periods.
The approximated values for the most recent period look quite compressed.
This is most likely due to the nature of the ice sheets from which the ice cores are drilled.
Ice cores can be hundreds of meters long.  As one can imagine, the deeper layers of the ice sheets are under tremendous pressure.  
Upper layers are under much less pressure; the ice is more porous and the air bubbles
(where are precious co2 data is contained) are much more likely to move around.  As a result, we can expect more recent ice core data to be more noisy or at least behave different than the older, more compressed data.

Because the first period is behaving like a completely different model, we should treat it as such.
We will pull the first period out and study it separately.    The result is the following:

![plot5](https://github.com/julialintern/julialintern.github.io/raw/master/images/Plot_5.png)

![plot6](https://github.com/julialintern/julialintern.github.io/raw/master/images/Plot_6.png)

Now that we have overlaid the approximated values with the actual values, we see that we still have a lot of work to do.  
Please refer to part 2 for our the next step in our model refinement process.



