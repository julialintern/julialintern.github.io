---
layout: post    
title: Basic Climate Modeling with ARIMA & python
---

*Wihin this post, we will continue our study of CO2 with ice core data with a range of 800,000 years*

**Where we left off**  
In the previous post, we developed a simple sinusoidal regression model to approximate the ice core data.    
Simple indeed!  As we can see, the sinusoidal is unable to capture the patterns of the data.
![plot6](https://github.com/julialintern/julialintern.github.io/raw/master/images/Plot_6.png)

Let's see if we can do better.


**Why ARIMA?**   
It may seem that there are so many different flavors and approaches of time series models including: models that have lagged features, differenced features, random walks, etc.  However, there is one type of time series
model that can combine all of these options: the ARIMA model.
ARIMA, Autoregressive Integrated Moving Average, is a time series model that incorporates
both autoregressive and moving average features.  
Autoregressive features are lagged features, where moving average features are generated from past error terms.
The 'integrated' component of ARIMA is a detrending step.  

 Most importantly, we can leverage our ARIMA model to forecast future time steps.

**ARIMA modeling steps**   
Here is a quick look at the required steps to develop an ARIMA model.
We will break down each of the steps in detail below.

1) Decide if the original time-series requires a nonlinear transformation (logging, exponentiating, box-cox, etc.).  We
want the time series (Y) to be additive as opposed to multiplicative.   

2) Determine if Y is stationary.  If non-stationary, then apply first-differencing.
If still non-stationary, apply 2nd differencing.

3)Forecast for Y at time t.

4) Iterate.



**1) Additive vs Multiplicative**  

Remember, that there are three main components with any time series:
trend, seasonality and the random component (error).
In a multiplicative time series, the components multiply together to create
the time series.

Data = Trend x Seasonal x Random

In an additive time series, the components are added together.

Data = Trend + Seasonal + Random

Luckily, a multiplicative series is as easy to fit as an additive series -
if we simply take the log !

*But why do we strive for an additive model ?*

For the same reason that we take the log when dealing with skewed response variables
when dealing with any regression: our goals is to obtain residuals that have a normal distribution
and constant variance.  Additive models are able to achieve this requirement. [why?](https://en.wikipedia.org/wiki/Central_limit_theorem)

*How to determine if we require a transformation?*

We can simply look at the time series.
An additive series, even with a trend, will have roughly the same size
peaks and troughs.   

With a multiplicative series, the size of the seasonal effect is proportional to the mean.


Based on the relatively consistent peaks & troughs of our ice core dataset, we will forego any transformations at this
stage and move on to step #2.

![plot4](https://github.com/julialintern/julialintern.github.io/raw/master/images/Plot_1.png)

**#2 Stationarity Check**

In order to develop ARIMA, our input must be stationary.
A stationary time series has no trend, constant variance over time, and 'consistent' wiggliness.
In the real world, most time series are not stationary.   

*Humph! Why are we limited to working without stationary time series?*    
The observations of a time series are not iid, and any observation can be
dependent on other observations in different ways. However,
it just so happens that a lot of nice properties that hold for iid observations
also hold for stationary random variables, including the law of large numbers
and the central limit theorem!  Alas, without stationarity we would not be able to develop
time series forecasting models.

*How to assess for Stationarity:*   
A moving average plot can give us a sense of whether our data is stationary or not.
We can generate that easily with pandas.

First: a quick look at our data:

```
d.head()
```

![plot5](https://github.com/julialintern/julialintern.github.io/raw/master/images/data_head.png)

```
rm=pd.rolling_mean(data,window=100)
rm.plot(figsize=(10,8));
```

![plot6](https://github.com/julialintern/julialintern.github.io/raw/master/images/Plot_3.png)

*How to address non-stationarity*    
We can de-trend our time series with a first-difference transformation.
First-difference is the delta between two adjacent observations.
We can also calculate this readily with pandas:


```
data['first_dif']=data.co2.diff()
data.first_dif.plot()
```

If Y is replaced by the first difference of Y (delta Y), then we will have an 'integrated' model.
Leveraging the differencing transformation is the difference between developing an ARMA model.
vs ARIMA model.

*Why is it called 'integrated' when we are differencing?*   
Because the stationary model that is fitted to the differenced data has to be summed
("integrated") to provide a model for the original (non-differenced) data.

*How do we know when the data is stationary enough?*    
One way to assess if our original data is truly stationary is with the
Dickey Fuller Hypothesis test.  As per Wikipedia, the augmented Dickey-Fuller test (ADF)
tests the null hypothesis that a [unit root](https://en.wikipedia.org/wiki/Unit_root) is present and thus, our time series is not stationary.
The alternative hypothesis indicates that the series is stationary.

```
import statsmodels.tsa.stattools as ts

df_test=ts.adfuller(data.co2,autolag='AIC')
df_results=pd.Series(dftest[0:4],index=['Test Statistic','p-value','Lags Used','Observations Used'])
for key,value in df_test[4].items():
    df_results['Critical Value (%s)'%key] = value
print(df_results)

```

![plot6](https://github.com/julialintern/julialintern.github.io/raw/master/images/dickey.png)


What do you think?  Assuming a critical value of 0.05, it looks like we can reject the null hypothesis.
According to Dickey & Fuller our non-transformed data just might be stationary enough.

**#3) Developing the ARIMA**

Once we are confident that time series data is stationary, we can develop our ARIMA model.

The ARIMA equation for predicting Y is as follows:


$$ Y^{hat}$$ = constant + weighted sum of the last p values of y + weighted sum of the last q forecast errors

Here p and q denotes the number of lags on Y and the number of lagged errors respectively.

Formally, we have:  
$$ Y^{hat} = \mu + \phi_1y_{t-1} + \phi_2y_{t-2} +... - \Theta_1e_{t-1} - \Theta_2e_{t-2}...  $$

Our ARIMA model is completely specified by p,d,& q.
But determining how many lags to use for p & q can be tricky.
Luckily, there are some rules of thumb we can use to determine the best values of
p & q together with autocorrelation & partial autocorrelation plots.

Lets generate the plots:

```
### autocorrelation
from statsmodels.graphics.tsaplots import plot_acf

plot_acf(data.co2,lags=100)
```
![plot7](https://github.com/julialintern/julialintern.github.io/raw/master/images/auto_corr.png)

```
### partial autocorrelation     

from statsmodels.graphics.tsaplots import plot_pacf

plot_pacf(data.co2,lags=100)
```
![plot8](https://github.com/julialintern/julialintern.github.io/raw/master/images/partial_corr.png)

*Rules of Thumb    

i. If the ACF plot “cuts off sharply” at lag k (i.e., if the autocorrelation is significantly
different from zero at lag k and extremely low in significance at the next higher lag and
the ones that follow), while there is a more gradual “decay” in the PACF plot (i.e. if
the dropoff in significance beyond lag k is more gradual), then set q=k and p=0. This
is a so-called “MA(q) signature.”  

ii. On the other hand, if the PACF plot cuts off sharply at lag k while there is a more
gradual decay in the ACF plot, then set p=k and q=0. This is a so-called “AR(p)
signature.”  

iii. If there is a single spike at lag 1 in both the ACF and PACF plots, then set p=1 and q=0
if it is positive (this is an AR(1) signature), and set p=0 and q=1 if it is negative (this is
an MA(1) signature).*

[resrouce: duke's arima notes, page 4 ](http://people.duke.edu/~rnau/Notes_on_nonseasonal_ARIMA_models--Robert_Nau.pdf)


It looks like we are seeing strong signs of what is described in note (ii): an AR signature.  In this case, one with two lags ~ AR(2)

Lets use statsmodels to develop the model.
We'll first do a test train split.


```

import statsmodels.api as sm
ts=data.co2.as_matrix()
train=ts[:996]  # honor order when splitting time series data!  we'll just
test=ts[996:]   # just retaining last 100 observations for test data for now

# Develop Training model
sar = sm.tsa.statespace.sarimax.SARIMAX(train, order=(2,0,0), trend='c').fit()
sar.summary()
```

![plot9](https://github.com/julialintern/julialintern.github.io/raw/master/images/train.png)

We opted for a SARIMAX model here.
SARIMAX is similiar to ARIMA models, but it contains a bit of flexibility in that it allows
for additional features :    
('S' in SARIMAX is for seasonal):  Seasonal features are a good option if you have seasons within cyclical data
  or seasons within seasons.
('X' in SARIMAX is for Exogenous): Allows for additional explanatory variables.

We can a good bit about our model by examining the summary table, most of which aligns with a
typical OLS summary output.   We can see that the AR(1) lagged feauture is considerably stronger
than AR(2).  The sigma2 output in the coefficients table is the estimate of the variance of the error term.


Lets use SARIMAX to generate some forecasts.

```
# create model & predict one obs at a time

preds=[]
history=train

for t in range(len(test)):
    sar = sm.tsa.statespace.sarimax.SARIMAX(hist, order=(2,0,0), trend='c').fit()
    pred=sar.predict(start=(997+t),end=997+t)
    preds.append(pred[0])
    history=np.append(history,test[t])

```

![plot10](https://github.com/julialintern/julialintern.github.io/raw/master/images/test_plot.png)


Looks pretty good.  However, we haven't completed step #4.
Our decision to leverage an AR(2) model was based on theory.  However, it would be
wise to perform an iterative/grid search approach to confirm which hyper-parameters are truly optimal
via train/test/val splits & minimization of the loss metric.  This will be the focus of the next post.
Stay tuned !
