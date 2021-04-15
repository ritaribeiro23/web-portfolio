---
date: "2020-10-19T20:00:00+01:00"
description: Exploratory Data Analysis for a e-Commerce Time-Series 
publishDate: "2020-10-19T20:00:00+01:00"
title: 'e-Commerce Time-Series Analysis Part 1: Intro and Exploratory Data Analysis'
---

Sometime ago someone asked me if I could pin-point the moment where my passion for Time Series was born, and although I could not express the exact moment, I could locate in my mind the assignment that had done it for me.

Back in 2018 I toke a course about Prediction Models and Time Series during my Master's. During that course I had to turn in a paper that was worth half the grade of the course, and was the during the development of said paper that I trully fell in love with the wonderous world of Time Series. Given that, I decided to publish my work so I may never forget it.

Since it is a very extensive paper, I will divide it in 3 parts so they are more easy to read.

* **Part 1:** Intro and Exploratory Data Analysis
* **Part 2:** Classical Time Series Prediction Models
* **Part 3:** ARIMA and SARIMA Models

# Part 1: Intro and Exploratory Data Analysis

For this Project, the Time Series chosen reflects the values of sales in e-commerce and mail house orders from January 2000 to December 2017. 

This data was obtained at the website of the [United States Census Bureau](https://www.census.gov/retail/index.html#mrts), and reflect the monthly revenue generated in commerce made via electronic means, like Amazon and Ebay, and sales made by catalog (the process analog to La Redoute in Portugal). Some "creative" data consolidation was needed given that the data is provided in an Excel file with one sheet per year (and by creative, I mean manual copy-paste of data. Not the most glamorous way of dealing with data, but I had to make due.)

For the sake of simplicity, I will be only referring as this time series as “e-commerce” or “ecm” (short for e-commerce). This time series (and subsequent analysis) stands very relevant in 2020 given the acceptance of these means of shopping from the general public. With the rise of Internet popularity and availability, this segment of business has the potential of generating big amounts of revenue for companies.

## Exploratory Data Analysis

To begin the exploration of my dataset, I began by using the appropriate object in R and plotting the data to check if some obvious characteristics would stand out, such as trends and seasonality, which would make this analysis viable. 

![img1](/blog/images/mpst/image51.png)
**Plot 1.1:** E-commerce revenue in the United States of America, from Jan 2000 to April 2018 

```R
plot(ecm, 
     main = "E-commerce revenue in the USA", 
     ylab="Values in millions of dollars",
     xlab='Time (in Months)')
```
**Code Snippet B1.1:** Plotting the time series

Observing Plot 1.1, one can easily observe a positive slope, which implies a growth (positive) trend. It is also clear the existence of a yearly pattern: January and February have lower values of revenue, throughout the year the revenue increases slightly and remains fairly stable up to October, where a clear peak of sales comprising the months of October, November and December is noticeable. This facts allow me to hypothesize that this data has seasonality (with a cycle of 12 months). A further look at the numbers is displayed on Appendix A, Table A1.1, where the full version of this data is shown.

### 1.1 Statistical Measures

Below on table 1, the statistical measures of this time series are displayed. I decided to also include the Coefficient of Variation (that correlates the Standard Deviation with the Sample Mean) to have a broader comprehension on the volatility of this Time Series. Although I don’t expect this Time Series to have a big volatility, given its context I find it relevant to further investigate and (try to) corroborate my intuitions. 

![img2](/blog/images/mpst/image68.png)
**Table 1.1:** Statistical Measures for the Time Series.

As mentioned before, the values for the Coefficient of Variation have indeed shown some interesting values; Although there is a change in dispersion within the data, the values are all in the same order of magnitude (between 12% and 19%) therefore this dataset is still relevant.

### 1.2 Seasonality

To assess Seasonality, I started by drawing the monthly plot for the Time Series:

![img3](/blog/images/mpst/image39.png)
**Plot 1.2:** Month plot

```R
monthplot(ecm, 
          main = "Month plot for E-commerce revenue", 
          ylab='Values in million of dollars',
          xlab = 'Month')
```

Observing Plot 1.2 I’m not able to guarantee the existence of seasonality since most months have peaks and vales around similar values. Nonetheless, and given that November and December show a clear difference from the previous months, I decided to conduct further testing to prove the existence of Seasonality.

I started by decomposing the series using both additive and multiplicative models and plotting its seasonal component. This step was also done for the logarithm transformed curve of the time series, for the sake of coherency.


| ![img4](/blog/images/mpst/image25.png)       | ![img5](/blog/images/mpst/image34.png)     |
| :------------- | :----------: |
|  ![img6](/blog/images/mpst/image3.png) | ![img7](/blog/images/mpst/image64.png)   |

**Plot 1.3:** Plots of the Seasonal Component of: a) ECM as Additive Model; b) Logarithmic Curve of ECM as Additive Model; c) ECM as Multiplicative Model; d) Logarithmic Curve of ECM as Multiplicative Model.

```R
plot(decompose(ecm, type="additive")$seasonal, 
     main="Seasonal component of ECM as an Additive Model",
     ylab='')

plot(decompose(log(ecm), type="additive")$seasonal, 
     main="Seasonal component of log ECM as an Additive Model",
     ylab='')

plot(decompose(ecm, type="multiplicative")$seasonal,
     main="Seasonal component of ECM as a Multiplicative Model",
     ylab='')

plot(decompose(log(ecm), type="multiplicative")$seasonal,
     main="Seasonal component of log ECM as a Multiplicative Model",
     ylab='')
```


As it is quite obvious by the observation of Plot 3, this series has, indeed, a Seasonal component. By the results of Plot 1.3, I can also conclude that it won’t be necessary to transform this series to its logarithmic curve, and that the Additive Model is a great candidate for a descriptor for this Series.


### 1.3. Trend
Since I could prove the existence of Seasonality, I decided to also display the seasonplot of this Series to try to confirm the existence of a growth trend.

![img8](/blog/images/mpst/image56.png)
**Plot 1.4:** SeasonPlot for ECM

```R
seasonplot(ecm,
           main = "Seasonplot for E-commerce revenue in the USA", 
           ylab="Values in millions of dollars",
           xlab='Time (in Months)',
           year.labels = TRUE,
           col = 1:20,
           pch = 10)
```

![img9](/blog/images/mpst/image57.png)
**Plot 1.5:** Trend of ECM as an Additive Model

Plots 1.4 and 1.5 prove that ECM does indeed have a growth trend, easily verifiable by: 
1. observing the volume of revenue grow by year in Plot 1.4; 
2. observing the crescent curve (displaying signs of a possible exponential curve) in Plot 1.5.

### 1.4. Stationarity

It is also relevant for this data exploration to assess the stationarity of the time series. Although the trend seems to be of growth, there is no way to legitimately assume a deterministic trend (note that Plot 1.5 exhibits one clear plateau circa 2007 to 2009). Given this, it is important to confirm non-stationarity to gain some insights on if a model from the ARIMA family might be able to reasonably model this data or if any transformation or differencing might be necessary.

Assessing stationarity can be done by studying the plots for ACF and PACF. Since this is a series that has Seasonality, the number of lags shown is a multiple of 12 (in this instance, 36).

| ![img10](/blog/images/mpst/image43.png)       | ![img11](/blog/images/mpst/image62.png)     |
| :------------- | :----------: |

**Plot 1.6:** ACF and PACF plots for the E-commerce revenue data

```R
acf(ts(ecm, freq=1), 
    main="a) Sample ACF for the First Differentiation of the ecm time series",
    lag.max = 36)

pacf(ts(ecm, freq=1), 
    main="b) Sample PACF for the First Differentiation of the ecm time series",
    lag.max = 36)
```

As it easily observed by Plot 1.6, this time series is non-stationary: ACF plot slowly decays to zero, and PACF plot shows clear peaks and overall instability. It is also possible to observe the effects of Seasonality in both Plots at lags 12, 24 and 36 for ACF and lags 12, 13 and 25 for PACF (note that lag 13 and 25 both represent the month of January, were usually is noticeable a decrease in sales).
Given this, and in order to obtain a stationary series, it will be necessary to do some differentiation in the time series. The number of differentiation steps needed to obtain a stationary series will be the order of I in the ARIMA model explored in future sections. Attending to the fact that this time series presents Seasonality, the first differentiation will take place at lag 12. The resulting ACF and PACF plots are displayed below.

| ![img12](/blog/images/mpst/image41.png)       | ![img13](/blog/images/mpst/image6.png)     |
| :------------- | :----------: |

**Plot 1.7:** ACF and PACF plots for the first differentiation of the E-commerce revenue data

```R
firstDiff <- diff(ecm2)
plot(firstDiff, main = "First Differentiation of the ecm time series")

acf(ts(firstDiff, freq=1), 
    main="a) Sample ACF for the First Differentiation of the ecm time series",
    lag.max = 36)

pacf(ts(firstDiff, freq=1), 
    main="b) Sample PACF for the First Differentiation of the ecm time series",
    lag.max = 36)

adf.test(firstDiff, alternative = 's')
```
  
Looking at Plot 1.7a, it is possible to see that the ACF approaches zero quickly after lag 3, with the exception of lags 12, 24 and 36. This is due to the seasonal component of the series and will be studied in detail in Section 4. To further corroborate the stationarity of the first differentiation, below are the results for the Dicker-Fuller test for unit root.
  
![img14](/blog/images/mpst/image61.png)

**Image 1.1:** Results of Dicker-Fuller test for stationarity.

The p-value displayed on image 1.1 allows for the acceptance of the null hypothesis of the Dicker-Fuller test, which implies that there are no unit roots for this differentiation and stationarity is archived.

## Conclusion

After this Data Analysis I can conclude that this Time Series presents a growth Trend; it also presents Seasonality with peaks in November and December and vales in January and February; the model that better fits this data is the Additive Model, without the need for any transformation. I was also able to verify that stationarity is archived on the first differentiation. These conclusions will be carried out to the further analysis present in this Project.

On Part 2 we will be exploring Classic Models for Time Series Forecasting. We will be doing Classical Decomposition and Seasonal Decomposition using Loess. After that, we will be forecasting some values using three different approaches: Classical Decomposition Forecast, Forecast using STL Method and Exponential Smoothing Modeling (also known as Triple Exponential Smoothing or Holt-Winters Method).

Stay tuned, I promise it wont be long!

## Bibliography

If you liked this project so far and want to further your understanding of Time Series, I wholeheartedly recommend you get acquainted with [Prof. Rob Hyndman](https://robjhyndman.com/)'s work. (He's one of my idols and he's on my bucketlist of people I really would like to meet!)

If you like reading, here are some references for you (can you spot Prof. Hyndman's inspiration?)

1. Cryer, J. and Chan, K; Time Series Analysis with Applications in R; Springer, Second Edition

2. Shumway, R. and Stoffer, D; Time Series Analysis and Its Applications With R Examples; Free texts in Statistics, Third Edition

3. [Hyndman, R; Forecasting: Principles and Practices; University of Western Australia, 2014](https://otexts.com/fpp2/)

4. [Using R for Time Series Analysis - Time Series 0.2 Documentation](http://a-little-book-of-r-for-time-series.readthedocs.io/en/latest/src/timeseries.html)

5. [Is my time series additive or multiplicative | R Bloggers](https://www.r-bloggers.com/is-my-time-series-additive-or-multiplicative/)

6. [Extracting Seasonality and Trend from Data: Decomposition Using R - Anomaly](https://anomaly.io/seasonal-trend-decomposition-in-r/)