---
date: "2020-10-28T20:00:00+01:00"
description: Classical Models for e-Commerce Sales Prediction 
publishDate: "2020-10-28T20:00:00+01:00"
title: 'e-Commerce Time-Series Analysis Part 2: Forecast Sales using Classical Models'
---

Welcome back!

Last post I discussed how this Series of blog posts compiles the assignment that got me started with Time Series and introduced the Exploratory Data Analysis for the Time Series I'll be studying. Remember Plot 1.1?

![img1](/blog/images/mpst/image51.png)
**Plot 1.1:** E-commerce revenue in the United States of America, from Jan 2000 to April 2018 

If you don't, please check Part 1 in the link below so you can get up to speed:

* **Part 1:** [Intro and Exploratory Data Analysis](https://www.reribeiro.pt/blog/ecm-timeseries-pt1/)

Ready? Let's go!


# 2. Time Series Decomposition

During this post, I will explore classical decomposition methods as well as the Seasonal Decomposition of Time Series using Loess (STL) to estimate the parameters of the ecm time series. With this information, I will then proceed to forecasting and later execute some model diagnostics.

## 2.1 Classical Decomposition
For this section, I’ll be carrying the assumptions of an increasing trend and existence of seasonality. I will also assume that this time series can be described as an additive model (this hypothesis will later be tested in Section 3).
Plot 2.1 displays the result of decomposing the ecm time series using R.


![img2](/blog/images/mpst/image13.png)
**Plot 2.1:** Components of the classical decomposition of the ecm time series.

```R
decECM <- decompose(ecm2, type = "additive")
plot(decECM)
```

![img3](/blog/images/mpst/image15.png)
**Table 2.1:** Trend Component Values

![img4](/blog/images/mpst/image63.png)
**Table 2.2:** Random Component Values

Since this decomposition uses a moving average of 12 months, the first and the last 6 values are missing from tables 2.1 and 2.2. This is to be expected and will be addressed before forecasting.

![img5](/blog/images/mpst/image70.png)
**Table 2.3:** Seasonal Component Values

As predicted in Section 2, table 2.1 allows for the corroboration of the hypothesis of an increasing trend; table 2.3 clearly show the existence of seasonality, with its highest values being 2063.38 and 8423.6, regarding November and December. The lowest point of revenue occurs in February.

## 2.2 Seasonal Decomposition of Time Series using Loess

For this section, I will maintain the hypothesis of an additive model and proceed with the analysis using this assumption. Below are the resulting plots of a decomposition using STL method.

![img6](/blog/images/mpst/image31.png)
**Plot 2.2:** Components of the STL decomposition of the ecm time series.

```R
stlECM <- stl(ecm2,
              s.window = "periodic",
              robust = TRUE)
plot(stlECM)
```

![img7](/blog/images/mpst/image15.png)
**Table 2.4:** STL Trend Component Values

![img8](/blog/images/mpst/image63.png)
**Table 2.5:** STL Remainder Component Values

![img9](/blog/images/mpst/image49.png)
**Table 2.6:** STL Seasonal Component Values

Looking closer at tables 2.4 through 2.6, I can convey that these values are in the same order of magnitude as the values obtained in the classical decomposition (although the STL values are usually higher), therefore, the sames conclusions for trend and seasonality still apply.

## 2.3 Forecasting with Decomposition Models

### 2.3.1 Classical Decomposition Forecasting

The first step when forecasting with a classical model is to deal with the missing values that resulted from the moving average. These values comprise the months of Jan of 2000 to June of 2000 and July 2016 to Dec 2016. To deal with the first set of missing months, I will just consider the values for the trend beginning in July of 2000. To deal with the second set of missing months, I will treat them as data to be predicted and execute a forecast for 18 months (instead of 12). For this forecast, I will use the Holt-winters method with gamma set to false, which is equivalent to a double exponential smoothing method. The results (and R code) for the trend forecast are shown below.

![img10](/blog/images/mpst/image46.png)
**Table 2.7:** Trend Forecast using Classical decomposition Method.

To obtain the prediction for 2017, one just needs to add the seasonal component to the 2017 values:

![img11](/blog/images/mpst/image66.png)
**Table 2.8:** predictions for year 2017 using classical decomposition.

The measures for accuracy of this forecast are:

![img12](/blog/images/mpst/image12.png)
**Table 2.9:** Accuracy measures for forecasting with classical decomposition model.

```R
classicTrend <- c(decECM$trend)
classicTrend <- classicTrend[-c(1:6)]
classicTrend <- classicTrend[-c(193:198)]
classicTrendTS <- ts(classicTrend,
                     start = c(2000, 7),
                     deltat = 1/12)
classicTrendPred <- HoltWinters(classicTrendTS,
                           gamma = FALSE)
classicTrendFcst <- predict(classicTrendPred,
                       n.ahead = 18)
classicPred <- ts(classicTrendFcst[7:18]+decECM$figure,
                  start = c(2017, 1),
                  deltat = 1/12)

accuracy(classicPred, k)
```

### 2.3.2 Forecasting using STL Method

For this section, I’ll be forecasting values of revenue for the ecm series for the year 2017 using the STL method. Table 2.11 holds the values for the point forecast, as well as the intervals of 80% and 95%.

![img13](/blog/images/mpst/image7.png)
**Table 2.10:** Forecast values using STL method.
 
The values for accuracy are displayed in table 2.11.


![img14](/blog/images/mpst/image60.png)
**Table 2.11:** Accuracy measures for the STL method.

```R
stlFcst <- forecast(stlECM, h=12)

accuracy(stlFcst, k)
```

### 2.3.3 Comparison between methods

For the ecm time series, both the Classic Decomposition Method and the STL Method perform quite poorly. To understand why, I created a table with the real values for 2017, the values predicted by the classic method and the values of the STL forecast.

![img15](/blog/images/mpst/image42.png)
**Table 2.12:** Real VS Classic VS STL

Looking at table 2.12, I can extract some conclusions: both models make optimistic predictions in months with a low volume of revenue. If we look back at tables 2.2 and 2.5, we can see that usually the values for both random and remainder components are negative for the months between January and November. Since these models don’t possess a mechanism to deal with these overestimations, the predicted values are quite higher than reality. It is also worth mentioning that, for December, the opposite occurs: both models offer conservative predictions. Once again, looking at tables 2.2 and 2.5, the values for December are usually positive. The conclusion for December is therefore analogous to the one of previous months. 


# 3. Exponential Smoothing Modeling
The Holt-Winter Method (also known as the Triple Exponential Smoothing) is used to forecast data points in series that exhibit strong Seasonality. Since this criterion is applicable to the ECM Time Series, the Holt-Winter Method will be carried out in the following sections.

For this analysis, two new time series are used: ecm2 contains data from Jan 2000 to Dec 2016 and k contains data from Jan 2017 to Dec 2017. This separation is done in order to have a training set and a testing set to obtain the results displayed in the following sections. 

## 3.1. Initial Holt-Winters Exploration

Before proceeding to the forecast using this method, it is important to explore and confirm the hypothesis of the section 2 of this report. To do so, I ran R code (displayed) for options “ZZZ” (model unknown), “AAA” (additive error, trend and seasonality) and “MMM” (multiplicative error, trend and seasonality), and the results are displayed below. The option “ZZZ” tries to find the model with optimal values for RMSE and MAPE in the training set; one should be aware that, even if a model is optimal in the training set, this result may not be true in the testing set.

| ![img16](/blog/images/mpst/image38.png)       | ![img17](/blog/images/mpst/image69.png)     |
| :------------- | :----------: |
|  ![img18](/blog/images/mpst/image21.png) |    |
**Table 3.1:** Values for the Holt-Winters method using different models for the ECM data.

```R
HW_ECM <- ets(ecm2, model="XXX")

HWfcst <- forecast(HW_ECM, h = 12)
```

When using the Holt-Winters Method, one should consider its parameter alpha, beta and gamma. Usually, lower values of these parameters are desirable because they reflect more stable and robust models; one the other hand, higher values for gamma reflect a bigger impact of newer observations on the model, which is also desirable.
Hence, a few conclusions can be drawn from Table 14: all three models present values for beta near zero, which indicates that this time series is almost smooth; the best value for beta is observed in the model “MMM”. Both models “MAM” and “MMM” show relatively low values for gamma, whereas the model “AAA” shows a high value for this parameter,which means that in this model, newer observations have more weight (which is desirable).
Taking into account these facts (relatively low beta and high gamma) and the previously known characteristics from Section 2, I believe that the Additive Model will produce better forecasting results; in any case, I will also produce the forecasts for the model “MAM” since it was indicated by the algorithm as having the optimal results for the training data.

## 3.2. Holt-Winters Forecasting
As discussed previously, I will start by comparing the accuracies for the forecasts obtained using the models “AAA” and “MAM’. Bellow there is the results.

![img19](/blog/images/mpst/image24.png)
**Table 3.2:** Several measures of accuracy for two different models.

As predicted previously, the Additive Model does produce the best results in terms of accuracy for the Holt-Winters Method, therefore the remainder analysis in this Section will be done using this Model.

Below is displayed the plot that compares the real data and the forecast for 95% and 80% accuracy. Table A3.1 in Appendix A showcases the same data in a table format.

![img20](/blog/images/mpst/image50.png)
**Plot 3.1:** Forecasted values using Holt-Winters Method

```R
autoplot(HWfcst, ylab = 'Values in millions of dollars')

HW_ACC <- accuracy(HWfcst, k)
HW_ACC
```

## 3.3. Residual Analysis
To assess if the model is a good fit for the ecm time series, it is relevant to conduct a residual analysis by studying both the ACF and PACF plots of the residuals, as well as executing a Ljung-Box test for non-correlation.

| ![img21](/blog/images/mpst/image45.png)       | ![img22](/blog/images/mpst/image19.png)     |
| :------------- | :----------: |
**Plot 3.2:** Sample ACF and PACF for the residuals of the HW forecast.


Plot 3.2.a shows that the sample ACF quickly decays to zero, where its values remain close. Plot 3.2.b shows a couple of values outside the significance lines, but overall this two plots suggest that the residuals are uncorrelated. To confirm, image 3.1 shows the results of the conducted Ljung-Box test.


![img23](/blog/images/mpst/image23.png)
**Image 3.1:** Results of the Ljung-Box test for the residuals

![img24](/blog/images/mpst/image28.png)
**Plot 3.3:** QQPlot for the residuals.

The Ljung-Box test confirms that there is no significant correlation between residuals; the QQPlot also suggests that the residuals follow a Normal Distribution. Therefore I feel confident to believe that the Holt-Winters model explored in this section models correctly the ecm time series without any need for further improvements.

```R
HW_residuals <- residuals(HWfcst)
acf(HW_residuals, 
    lag.max = 36,
    main="a) Sample ACF for the residuals of the HW forecast")
pacf(HW_residuals, 
     lag.max = 36,
     main="b) Sample PACF for the residuals of the HW forecast")
Box.test(HW_residuals, type = "Ljung-Box")
qqnorm(HW_residuals,
       main="QQPlot for residuals")
qqline(HW_residuals)
```

# Conclusion

In this blog post we saw the application of Classical Methods for Time Series Forecasting. We checked Classical decomposition, STL Method and the Holt-Winters

In the next part (which -heads up- will be *way* more intensive), we will execute forecast using ARIMA and SARIMA models.

Stay tunned!

## Bibliography

If you made it this far, you deserve a Bibliography!


Not much has changed since last post (still a huge fan of [Prof. Rob Hyndman](https://robjhyndman.com/)!), but I'll leave you the references:

1. Cryer, J. and Chan, K; Time Series Analysis with Applications in R; Springer, Second Edition

2. Shumway, R. and Stoffer, D; Time Series Analysis and Its Applications With R Examples; Free texts in Statistics, Third Edition

3. [Hyndman, R; Forecasting: Principles and Practices; University of Western Australia, 2014](https://otexts.com/fpp2/)

4. [Using R for Time Series Analysis - Time Series 0.2 Documentation](http://a-little-book-of-r-for-time-series.readthedocs.io/en/latest/src/timeseries.html)

5. [Is my time series additive or multiplicative | R Bloggers](https://www.r-bloggers.com/is-my-time-series-additive-or-multiplicative/)

6. [Extracting Seasonality and Trend from Data: Decomposition Using R - Anomaly](https://anomaly.io/seasonal-trend-decomposition-in-r/)

7. Holt-Winters Forecasting for Dummies (or Developers), Gregory Trubestskoy
[Part I](https://grisha.org/blog/2016/01/29/triple-exponential-smoothing-forecasting/)
[Part II](https://grisha.org/blog/2016/02/16/triple-exponential-smoothing-forecasting-part-ii/)
[Part III](https://grisha.org/blog/2016/02/17/triple-exponential-smoothing-forecasting-part-iii/)

8. [Exponential Smoothing - UC Business Analytics R Programming Guide](http://uc-r.github.io/ts_exp_smoothing#hw)