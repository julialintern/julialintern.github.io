*Wihin this post, we will continue our study of CO2 with ice core data with a range of 800,000 years*

**Where we left off**  
In the previous post, we developed a simple sinusoidal regression model to approximate the data.
Simple indeed!  As we can see, the sinusoidal is unable to capture the patterns of the data.
![plot6](https://github.com/julialintern/julialintern.github.io/raw/master/images/Plot_6.png)

Let's see if we can do better.


**Why ARIMA?**
It may seem that there are so many different flavors and approaches of time series models including: models that have lagged features, differenced features, random walks, etc.  However, there is one type of time series
model that can combine all of these options: the ARIMA model.
ARIMA, Autoregressive Integrated Moving Average, is a time series model that incorporates
both autoregressive and moving average features.  
Autoregressive features are lagged features, where moving average features are generated from past error terms.
The 'integrated' component of ARIMA is the required detrending step.  
(Don't worry, we will dive deeper into all of this!)

 Most importantly, we can leverage our ARIMA model to forecast future time steps.

**ARIMA modeling steps**
Here is a quick look at the outline of all required steps.
We will break down each of the steps in detail below.

1) Decide if the original time-series requires a nonlinear transformation (logging, exponentiating, box-cox,,).  We
want the time series (Y) to be additive as opposed to multiplicative.   

2) Determine if the time series is stationary.  If non-stationary, then apply first-differencing.
If still non-stationary, apply 2nd differencing.

3) Once we have our Y: (forecast for y at time t) y= constant + weighted sum of the last p values of y + weighted sum of the last q forecast errors

*4) Perform a train/test/validation split in order to assess ARIMA on unseen data.

*5) Iterate ? *
*? Examine the residuals from the fitted model to confirm if adequate.


**1) Additive vs Multiplicative**

Remember, that we have three main components with any time series:
trend, seasonality and the random component (error).
In a multiplicative time series, the components multiply together to create
the time series.

Data = Trend x Seasonal x Random

In an additive time series, the components are added together.

Data = Trend + Seasonal + Random

Luckily, a multiplicative series is as easy to fit as an additive series -
if we simply take the log !

But why do we strive for an additive model ?

For the same reason that we take the log when dealing with skewed response variables
when dealing with any regression: with the goal being residuals that have a normal distribution
and constant variance.  Additive models are able to achieve this requirement [why? ]; central limit

*How to assess?*

We can simply look at the time series.
An additive series, even with a trend, will have roughly the same size
peaks and troughs.

![plot2](https://github.com/julialintern/julialintern.github.io/raw/master/images/additive.png)

With a multiplicative series, the size of the seasonal effect is proportional to the mean (as shown.. ?),

![plot3](https://github.com/julialintern/julialintern.github.io/raw/master/images/multiplicative.png)

Based on the consistent peaks & troughs of this dataset, I will forego any transformations at this
stage and move on to step #2.

**#2 Stationarity Check**

A stationary time series has no trend, constant variance over time, and 'consistent' wiggliness.
In the real world, most time series are not stationary.   

*Humph! Stationarity ? why do we care?*
The observations of a time series are not iid, and any observation can be
dependent on other variations in different ways. However,
it just so happens that a lot of nice properties that hold for iid observations
also holds for stationary random variables, including the law of large numbers
and central limit theorem!  Alas, without stationarity we would not be able to develop
forcasting models.

*How to assess for Stationarity*
A quick look at a moving average plot to get a sense if stationarity is a concern.
We can generate that easily with pandas.

```
rm=pd.rolling_mean(data,window=100)
rm.plot(figsize=(10,8));
```

#OPEN : add plot here  & viz of data.head()

*How to address non-stationarity*
We can de-trend our time series with a first-difference transformation.
First-difference is the delta between two adjacent observations.
We can also calculate this readily with pandas:


```
data['first_dif']=data.co2.diff()
data.first_dif.plot()
```

If Y is replaced by the first difference of Y (delta Y), then we have an Integrated model.
Leveraging the differencing transformation is the difference between developing an ARMA
vs ARIMA model.

*why is it called 'integrated' when we are differencing?*
Because the stationary model that is fitted to the differenced data has to be summed
("integrated") to provide a model for the original (non-differenced) data.

* How to determine when our model is stationary?*

One way to assess if our original data is truly not stationary is with the
Dickey Fuller Hypothesis test.  As per Wiki, the augmented Dickey-Fuller test (ADF)
tests the null hypothesis that a unit root is present in a time series.
The alternative hypothesis indicates that the series is stationary.

```
import statsmodels.tsa.stattools as ts

df_test=ts.adfuller(data.co2,autolag='AIC')
df_results=pd.Series(dftest[0:4],index=['Test Statistic','p-value','Lags Used','Observations Used'])
for key,value in df_test[4].items():
    df_results['Critical Value (%s)'%key] = value
print(df_results)

```

Test Statistic            -4.141938
p-value                    0.000825
Lags Used                  4.000000
Observations Used       1091.000000
Critical Value (1%)       -3.436358
Critical Value (5%)       -2.864193
Critical Value (10%)      -2.568182
dtype: float64
