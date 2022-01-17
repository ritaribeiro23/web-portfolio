---
date: "2020-11-05T20:00:00+01:00"
description: Seasonal ARIMA models for e-Commerce Sales Prediction 
publishDate: "2020-11-05T20:00:00+01:00"
title: 'e-Commerce Time-Series Analysis Part 3: Forecast Sales using Seasonal ARIMA Models'
socialImage: /blog/images/mpst/image51.png
---

Hi there, thanks for being back again!

In the past 2 blog posts in this project, I have established the necessary ground work by presenting the Exploratory Data Analysis and doing initial forecasts using classical models. In this post, we will be exploring Seasonal ARIMA models and, by the end, I will present a brief discussing and comparison between all the models used.

Let's remeber what our series looks like:

![img1](/blog/images/mpst/image51.png)
**Plot 1.1:** E-commerce revenue in the United States of America, from Jan 2000 to April 2018 

If you haven't read the previous parts I really advise you do so before exploring this current blog post. This post is a bit more complex and intensive, and you will need the foundations laid previously. Also, we will be comparing all obtained results and you will be very confused if this is your starting point!

You can find the previous blog posts in the following links:
* **Part 1:** [Intro and Exploratory Data Analysis](https://www.reribeiro.pt/blog/ecm-timeseries-pt1/)
* **Part 1:** [Forecast Sales using Classical Models](https://www.reribeiro.pt/blog/ecm-timeseries-pt2/)

Now that you're all up to speed, let's explore some new models!

# 4. Seasonal ARIMA modeling

In this blog post, I will explore the steps needed to model the ecm time series using a model from the ARIMA family. From the knowledge obtained so far, I can safely assume that this time series is better modelled by a Seasonal ARIMA model, due to the Seasonal component of the ECM time series found in Part 1.


## 4.1. Data Preparation and Candidate Models

As explored previously in section 2.4, this series becomes stationary after the first differentiation (d = 1. This fact, although corroborated by the Dicker-Fuller test conducted in Section 2.4, doesn’t deal with the Seasonal component of the series, which explain the observable peaks in both ACF and PACF plots. Given this, it is then necessary to do seasonal differentiation and assess the removal of known peaks of seasonality. Below are the plots for ACF and PACF obtained after the first seasonal differentiation (D=1).

| ![img25](/blog/images/mpst/image14.png)       | ![img26](/blog/images/mpst/image9.png)     |
| :------------- | :----------: |
**Plot 4.1:** Sample ACF and PACF for the first seasonal differentiation.

```R
secondDiff <- diff(firstDiff, 12)

acf(ts(secondDiff, freq=1), 
    main="a) Sample ACF for the First Seasonal Differentiation of the ecm time series",
    lag.max = 36)

pacf(ts(secondDiff, freq=1), 
     main="b) Sample PACF for the First Seasonal Differentiation of the ecm time series",
     lag.max = 36)

```

As expected, the first seasonal differentiation eliminates the peaks of seasonality. To further confirm my findings, I ran the functions ndiffs and nsdiffs from the R software to find the optimal values for d and D. 

![img27](/blog/images/mpst/image44.png)
**Table 4.1:** Number of series and seasonal differentiation suggested by R.

Since the values suggested by R confirm my findings, moving forward I will consider the values d=1 and D=1 for my candidate models.

Plot 4.1.b shows two clear peaks at lags 1 and 2. Plot 10a shows a clear peak at lag 1, which might be an indicator of seasonality. Taken this into account, Table 4.2 displays the models that I think are relevant to explore given the information I have so far in this project.

![img28](/blog/images/mpst/image48.png)
**Table 4.2:** Candidate SARIMA Models for ecm time series. 

## 4.2 Model Selection and Diagnostics

To evaluate if a model is representative of the ecm time series, I need to assess the statistical significance of its coefficients, according to equations (1) and (2) for both seasonal and non-seasonal components.

| ![img29](/blog/images/mpst/image1.png) | (1) |
|:----------------------------------------:|:-----:|
| ![img30](/blog/images/mpst/image2.png) | (2) |

Table 4.3 displays the results for every candidate model, with the indication of which models satisfy the criteria.

![img31](/blog/images/mpst/image37.png)
**Table 4.3:** Results and analysis of the candidate ARIMA Models for ecm time series. 

Observing Table 4.3, I’m led to believe that models with the combinations p = 0 or P = 0 produce better models. Given this, a short list of new candidate models was explored, and its results are displayed below.

![img32](/blog/images/mpst/image26.png)
**Table 4.4:** Results and analysis of new candidate ARIMA Models for ecm time series.

My intuition was indeed correct and one new candidate model (2.5), will be added to the list of candidate models that will be analysed in the following sections.

To obtain these results, I used the following R code, substituting the variables p, d, q, P, D, Q accordingly, and setting m to 12.

```R
# (p,d,q)x(P,D,Q)m
arima(ecm2, 
      order=c(p,d,q), 
      seasonal=list(order=c(P,D,Q), period=m))
```




To end this subsection, I tested for overfitting. Since the candidates accepted so far performed well with AR(2) or MA(2), I decided to test similar models with AR(3) and MA(3). If the initial models are indeed good candidates, I expect to see either low values for ar3 and ma3 or results that negate equations (1) or (2). The results (displayed bellow) corroborated my intuition and none of these models passed the test. 

![img33](/blog/images/mpst/image33.png)
**Table 4.5:** Models tested for overfitting.

### 4.2.1 Residual Analysis

For this analysis, I will use the Ljung-Box test to assess if the resulting residuals from the candidate models are uncorrelated (null hypothesis). Table 4.6 showcases the results for the Ljung-Box test.

![img34](/blog/images/mpst/image17.png)
**Table 4.6:** Results for the Ljung-Box test.


```R
Box.test(fit1$residuals, type = 'Ljung')
```

According to Table 4.6, all models pass the Ljung-Box test, with high values for p-value. It’s worth to note that the higher value for p-value belongs to model 2.3.


The next step in this analysis is to assess if the residuals have Normal Distribution (or a close distribution).

| ![img35](/blog/images/mpst/image22.png) 	| ![img36](/blog/images/mpst/image10.png) 	| ![img37](/blog/images/mpst/image53.png) 	|
|-----------------------------------------	|-----------------------------------------	|-----------------------------------------	|
| ![img38](/blog/images/mpst/image65.png) 	| ![img39](/blog/images/mpst/image59.png) 	| ![img40](/blog/images/mpst/image30.png) 	|
| ![img41](/blog/images/mpst/image32.png) 	| ![img35](/blog/images/mpst/image52.png) 	| ![img43](/blog/images/mpst/image11.png) 	|
| ![img44](/blog/images/mpst/image40.png) 	| ![img45](/blog/images/mpst/image29.png) 	| ![img46](/blog/images/mpst/image36.png) 	|

**Table 4.7:** Plots for Residual Normality Test.

```R
# Normality tests for model X
plot(fit1$residuals, 
     main="a) Plot for residuals of model X")
acf(fit1$residuals, 
    lag.max = 36,
    main="b) Sample ACF for residuals of model X")
qqnorm(fit1$residuals,
       main="c) QQPlot for residuals of model X")
qqline(fit1$residuals)
```


According to the results presented in Table 4.7, I can assume that the residuals of all models do not present Normal Distribution. All candidate models display tails on their QQPlots; regarding the ACF plot, models 0.1 and 0.2 present some values outside the desirable threshold. Moving forward, I’m not comfortable yet to discard any of these candidate models, so further analysis is needed.


### 4.2.2 AIC, AICC and BIC criteria

For this section, I will assess the values of three distinct values for every model: Akaike Information Criteria (AIC), the Corrected AIC (AICC) and the Bayesian Information Criteria (BIC). The goal is to choose the candidate model that minimizes these values, with particular attention to the AICC value.

![img50](/blog/images/mpst/image47.png)
**Table 4.8:** Values for AIC, AICC and BIC criteria

```R
# Model X
AIC(fitX)
BIC(fitX)

p <- length(fitX$coef) + 1
s <- length(fitX$res) - fitX$arma[6] - fitX$arma[7]*fitX$arma[5]
AIC(fitX) + 2*p*(s/(s-p-1)-1)
```


Attending to the values presented in Table 4.8, the models with better indicators are models 2.3 and 2.5.

## 4.3 Forecasting

In this section, I forecasted values for the year 2017 using the models above mentioned, and registered the values of several error indicators (for the testing set) in table 23. I will use MAPE as a global marker to choose the two best models and display their plots.

![img51](/blog/images/mpst/image27.png)
**Table 4.9:** Accuracy values for the forecasts using each of the candidate models.

The lowest MAPE and RMSE belong to models 0.2 and 2.3. This is a surprise because the function *auto.arima()* pointed to model 2.3 as the optimal model for the ecm time series (please see Image 4.2). Below are the plots with the respective forecasts. Tables 4.10 and 4.11 show the predicted values, as well as the points for 95% and 80% accuracy.

![img52](/blog/images/mpst/image18.png)
**Image 4.2:**  *auto.arima()* results for the ECM time series.

| ![img53](/blog/images/mpst/image35.png) 	| ![img54](/blog/images/mpst/image54.png) 	| 
|-----------------------------------------	|-----------------------------------------	|
**Plot 4.2:** Forecasting plots for models 0.2 and 2.3.

| ![img55](/blog/images/mpst/image20.png) 	| ![img56](/blog/images/mpst/image5.png) 	| 
|-----------------------------------------	|-----------------------------------------	|
|  **Table 4.10:** Predictions for Model 0.2 |  **Table 4.11:** Predictions for Model 2.3 |

# 5. Forecast Comparison

To decide which model best describes the ecm time series, I need to compare their respective measures of accuracy for the testing set. I will give particular relevance to MAPE and RMSE measures, in this order of importance.

![img57](/blog/images/mpst/image8.png)
**Table 5.1:** Accuracy measures for the studied models.

Studying Table 5.1, I can conclude that the model that best describes this data is the model SARIMA(0,1,2)x(1,1,1)12, followed very closely by the model obtained by using Exponential Smoothing (Holt-Winters). Both present satisfactory values for the accuracy measures used; this, in conjunction with the residual analysis performed in both models let me conclude that both performed well in modelling the original data.


# Conclusion
	
It was a long ride, but we are finally at the end! Before you go, I want to offer you a few conclusions that I draw from this project, and I hope that they might be of use to you!

When working in a process to model a time series, it is of vital importance to explore the statistical features of the time series, as well as assess the existence of trend and seasonality. These initial steps can provide helpful information when starting the modelling process. It is also relevant to assess stationarity when dealing with series with a seasonal component, since this information can help the bootstrapping the process of building a fitted Arima model.

The methods for series decomposition, although fell short in this data, are always relevant to study because a first instinct when looking at a series plot can be misleading, and these processes might be helpful to understand the components individually.

Exponential Smoothing is a very powerful tool to try when modelling time series. Using this method helps to better understand the form of a series (i.e. additive, multiplicative or mixed) which is of vital importance when modelling a process. This understanding can show the need for further data transformations, which usually results in better models.

SARIMA Models produce generally very good representations (when applicable). Understanding the parameters of a time series is necessary to correctly estimate the parameters of the SARIMA models; it is also necessary to keep in mind to run model diagnostics, such as statistical significance, information criteria and residual analysis, amongst others.

To conclude, there are two main lessons I learned while elaborating this project and that I feel are important to state: not always a diagnostic run automatically from a software is indeed correct. Usually the optimal parameters are estimated within the training dataset, and these optimal results might not translate into the testing set. Following this, when it is possible, is always better to split the data and corroborate results before making assumptions that might not be the best to make.



## Bibliography

Like I did in the previous posts, here's a bibliography for further research!

I still can't stress enough how much of a fan of [Prof. Rob Hyndman](https://robjhyndman.com/)!) I am, and I advise you to start with is materials.

You should also explore the following meterials that helped me imenselly during this project:

1. Cryer, J. and Chan, K; Time Series Analysis with Applications in R; Springer, Second Edition

2. Shumway, R. and Stoffer, D; Time Series Analysis and Its Applications With R Examples; Free texts in Statistics, Third Edition

3. Hyndman, R; Forecasting: Principles and Practices; University of Western Australia, 2014

4.  Silva, ME and Teles, P; Handouts for the course in Time Series and Prediction Models; School of Economics and Management - University of Porto, 2018

5. Website: Monthly Retail Trade - Monthly Retail Trade Survey Historical Data
Available at: https://www.census.gov/retail/mrts/historic_releases.html

6. Website: Using R for Time Series Analysis - Time Series 0.2 Documentation
Available at: http://a-little-book-of-r-for-time-series.readthedocs.io/en/latest/src/timeseries.html

7. Website: Is my time series additive or multiplicative | R Bloggers
Available at: https://www.r-bloggers.com/is-my-time-series-additive-or-multiplicative/

8. Website: Extracting Seasonality and Trend from Data: Decomposition Using R - Anomaly
Available at: https://anomaly.io/seasonal-trend-decomposition-in-r/

9. Website: Holt-Winters Forecasting for Dummies (or Developers) - Part I, II and III - Gregory Trubestskoy
Available at: https://grisha.org/blog/2016/01/29/triple-exponential-smoothing-forecasting/
https://grisha.org/blog/2016/02/16/triple-exponential-smoothing-forecasting-part-ii/
https://grisha.org/blog/2016/02/17/triple-exponential-smoothing-forecasting-part-iii/

10. Website: Exponential Smoothing - UC Business Analytics R Programming Guide
Available at: http://uc-r.github.io/ts_exp_smoothing#hw
