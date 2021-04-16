---
date: "2019-11-01T16:18:12+01:00"
description: A small blog post about Multivariate Time Series Analysis
publishDate: "2019-11-01T16:18:12+01:00"
title: Multivariate Time Series Analysis
tags: ["Analysis", "Data", "TimeSeries", "Analytics"]
---
Last year I was challenged to solve a small multivarite time-series problem as a recruitment exercise.
Even though I wasn't picked for the position, I had so much fun preparing this document that I decided to share it on my blog. 

Keep in mind that this is only a theoretical problem, and that in a real-life scenario this dataset would be to small to render substantial conclusions.

### **The Problem**
The proposed problem is to predict if the sales of a given item will increase during a promotional action. 

### **0) Data Cleaning**

Before starting this study, I worked the dataset. I organized the data
into a .CSV file so the R software could read it easily. I also added a
new variable in substitution of the original *Promotion* variable,
called *Promo\_B*, that transformed the factors "Sim/Não" into a binary
1/0 variable.

### **1) Exploratory Data Analysis**

The first step in any Data Science project is to explore the data using
Statistical Measurements to gain basic data intuition.

After importing the data to my project, I executed some summaries on the
data:
``` R
summary(df_Sales[, 3:5])

Sales           Price       Promo   
Min.   : 3186   Min.   :1.000   No:40  
1st Qu.: 6968   1st Qu.:1.400   Yes: 8  
Median : 8670   Median :1.500           
Mean   : 9166   Mean   :1.433           
3rd Qu.:11870   3rd Qu.:1.500           
Max.   :17198   Max.   :1.800
```

The trivial conclusions one can draw at this point are:

- The mean sales value is 8670€;
- The most usual price of sale is 1.50;
- Usually, the product is not on sale.

To understand the correlation between the variables, I studied them
graphically:

#### **Sales VS Price**

![img_1](/blog/images/figure-markdown_strict/salesVSprices-1.png)

``` R
    cor(df_Sales[, 3], df_Sales[, 4])
```
    :: -0.3256287

The variables Sales and Prices exhibit a small negative correlation,
which means that there is a slight clue that the sales increase when the
prices are lower. Since the correlation is larger than -0.5, I can't
extrapolate correlation between these two variables.

#### **Price VS Promotion**

![](/blog/images/figure-markdown_strict/pricesVSpromotions-1.png)

```R
cor(df_Sales[, 4], df_Sales[, 6])
```
:: -0.4703023

Prices and promotions exhibit a lower negative correlation, which is to
be expected since is trivial to think that lower prices occur during
promotions. Although this conclusion might seem trivial at first, for
this example it is not since its value is higher than -0.5, therefore
the correlation is not significance.

#### **Sales VS Promotion**

```R
cor(df_Sales[, 3], df_Sales[, 6])
```
:: 0.3597347

For the sake of simplicity, I decided not to display the plot for this
correlation. By observing its value, I can conclude that Sales and
Promotions exhibit a small positive correlation, although not big enough
to establish correlation (and potentially eliminate one of these
variables).

After the calculation of these correlations, I conclude that I cannot
eliminate neither of the variables since the correlation factors are not
strong enough. Given this, I will proceed this study assuming the
variables Price and Promotion as exogenous (i.e., not correlated with
the time series Sales).

### **2) Seasonality, Trend and Stationarity**

#### **Seasonality**

To access if the Sales time series contains Seasonality, I studied is
monthplot:

```R
par(mfrow=c(1,1))
monthplot(ts_sales, 
        main = "Monthly Sales Values", 
        ylab="Values in euros",
        xlab = 'Month')
```

![](/blog/images/figure-markdown_strict/s_plot-1.png)

It is very obvious to observe peaks of sales usually occur during April
and November, and the lowest sales occur during February and June.

The next step is to access which transform best models the seasonality
of this time series:

**Candidate models for seasonality:**

```R
par(mfrow=c(2,2))
plot(decompose(ts_sales, type="additive")$seasonal, 
     main="Sales as Additive Model",
     ylab='')

plot(decompose(log(ts_sales), type="additive")$seasonal, 
     main="LOG(Sales) as Additive Model",
     ylab='')

plot(decompose(ts_sales, type="multiplicative")$seasonal,
     main="Sales as Multiplicative Model",
     ylab='')

plot(decompose(log(ts_sales), type="multiplicative")$seasonal,
     main="LOG(Sales) as Multiplicative Model",
     ylab='')
```

![](/blog/images/figure-markdown_strict/Season_plots-1.png)

Given that the Additive Model is the one which explains the models with
the least transforms, I will safely proceed my study with this
assumption.

#### **Trend**

To access the Trend of this series, I displayed the Trend of the Sales
series, assuming a additive model:

```R
plot(decompose(ts_sales, type="additive")$trend, 
     main="Trend of Sales Values as an Additive Model",
     ylab='')
```

![](/blog/images/figure-markdown_strict/trend-1.png)

It is not clear to assume an increasing Trend (even if the most recent
values show so). Further exploration on stationarity will be necessary,
but for now one can exclude classical models for forecasting (such as
classical decomposition, seasonal decomposition using Loess), since
these models depend on a well behaved trend.

#### **Stationarity**

One of the conditions for the ARIMA models to be applicable to a time
series is that said time series is stationary. One can verify the
stationarity of a time series by studying its Auto-Correlation Function
(ACF) and Partial ACF. These functions allow us to check for dependence
between one point and past values. A time series is stationary when both
ACF and PACF tend to zero after a short lag. One can also corroborate
this results with the Dicker-Fuller test.

If a time series is not stationary in its normal form, it might be
necessary to perform differentiates on the time series until one
archives stationarity. The number of differentiation steps needed to
obtain a stationary series will be the order of I in the ARIMA model.
```R
par(mfrow=c(2,1))

acf(ts_sales, 
    main="a) Sample ACF for the Sales time series",
    lag.max = 48)

pacf(ts_sales, 
     main="b) Sample PACF for the Sales time series",
     lag.max = 48)
```
![](/blog/images/figure-markdown_strict/stationarity-1.png)

```R
adf.test(ts_sales, alternative='s')

Augmented Dickey-Fuller Test
 
data:  ts_sales
Dickey-Fuller = -4.1286, Lag order = 3, p-value = 0.01204
alternative hypothesis: stationary
```

The p-value displayed allows for the acceptance of the null hypothesis
of the Dicker-Fuller test, which implies that there are no unit roots
for this time series in its normal form and stationarity is guaranteed.

The order of I in the ARIMA model used will be zero, since there was no
need for differentiation in this time series.

### **3) ARIMA Models and Forecasting**

Due to the Seasonal component of the Sales time series, can safely
assume that this modeling will use a Seasonal ARIMA model.

#### **Methodology**

Given that I'm trying to find the optimal model to forecast this time
series, I decided to use the Hold-Out method in the following fashion:

- From Jan 2015 to Sep 2018: training set;
- From Oct 2018 to Dec 2018: testing set.

After I find the optimal model, I will then use the full dataset for
training before predicting Jan 2019 to Mar 2019.
```R
part_point <- 45

sales_train <- ts(df_Sales[1:part_point, 3], 
                    start = c(2015, 1), 
                    deltat = 1/12)
sales_test <- ts(df_Sales[(part_point+1):48, 3], 
                    start = c(2018, 9), 
                    deltat = 1/12)

exo_vars <- as.matrix(df_Sales[1:part_point, 
                        c(4, 6)])
exo_vars_test <- as.matrix(df_Sales[(part_point+1):48, 
                            c(4, 6)])
```
#### **Finding the Model**

To confirm the findings of the previous section. I ran the following
code to obtain the optimal number of differentiations (series and
seasonal) suggested by R:

```R
ndiffs(ts_sales)
```
:: 0

```R
nsdiffs(ts_sales)
```
::0

As expected, no differentiations are needed.

**R model suggestion**

```R
auto.arima(sales_train, xreg = exo_vars)

Series: sales_train 
Regression with ARIMA(0,0,0)(1,0,0)[12] errors 
     
Coefficients:
         sar1  intercept      Price   Promo_B
       0.3519  13941.204  -3678.188  2177.182
 s.e.  0.1772   5087.843   3447.355  1636.796
     
sigma^2 estimated as 8123772:  log likelihood=-420.53
AIC=851.07   AICc=852.6   BIC=860.1
```

As seen above, the model suggested by R is a SARIMA (0,0,0)x(1,0,0).

I then proceeded to train the model (given the exoneous variables):

```R
fit1 <- Arima(sales_train, 
              order=c(0,0,0), 
              seasonal=list(order=c(1,0,0), period=12),
              xreg = exo_vars)
fit1

Series: sales_train 
Regression with ARIMA(0,0,0)(1,0,0)[12] errors 
 
Coefficients:
sar1  intercept      Price   Promo_B
0.3519  13941.204  -3678.188  2177.182
s.e.  0.1772   5087.843   3447.355  1636.796
 
sigma^2 estimated as 8123772:  log likelihood=-420.53
AIC=851.07   AICc=852.6   BIC=860.1
```
For the sake of coherency, I added 3 different models to the test:
```R
# (0,0,0)x(2,0,0)
fit2 <- Arima(sales_train, 
              order=c(0,0,0), 
              seasonal=list(order=c(2,0,0), period=12),
              xreg = exo_vars)

# (0,0,0)x(1,0,1)
fit3 <- Arima(sales_train, 
              order=c(0,0,0), 
              seasonal=list(order=c(1,0,1), period=12),
              xreg = exo_vars)

# (0,0,0)x(0,0,1)
fit4 <- Arima(sales_train, 
              order=c(0,0,0), 
              seasonal=list(order=c(0,0, 1), period=12),
              xreg = exo_vars)
```
To access the validity of the model, I need to check the values for the
Akaike Information Criteria (AIC) and the Bayesian Information Criteria
(BIC), the goal being to choose the candidate model that minimizes these
values:
```R
AIC(fit1)
```
:: 851.0651
```R
BIC(fit1)
```
:: 860.0984
```R
AIC(fit2)
```
:: 851.5745
```R
BIC(fit2)
```
:: 862.4145
```R
AIC(fit3)
```
:: 851.709
```R
BIC(fit3)
```
:: 862.549
```R
AIC(fit4)
```
:: 852.1649
```R
BIC(fit4)
```
:: 861.1982

The model that minimizes the values for AIC and BIC is indeed the model
initially suggested by the function auto.arima(), therefore I will
continue my study using this model.

Next, it is important to check the Residuals of the model, using
Ljung-Box test and Normal Distribution:
```R
par(mfrow=c(2,2))
plot(fit1$residuals, 
     main="a) Plot for residuals for the predicted model")
acf(fit1$residuals, 
    lag.max = 36,
    main="b) Sample ACF for residuals for the predicted model")
qqnorm(fit1$residuals,
    main="c) QQPlot for residuals for the predicted model")
qqline(fit1$residuals)
```
![](/blog/images/figure-markdown_strict/residuals-1.png)

The model also passes this test, with a stable ACF (near zero for higher
lags) and no noticeable tails in the QQPlot.

#### **Testing the Model**

After deciding the model that I will be using, I predicted known values,
in order to check the model's validity:
```R
ARfcst1 <- forecast(fit1, xreg = exo_vars_test, h= 3)
ARfcst1 
```
    ##          Point Forecast    Lo 80    Hi 80    Lo 95    Hi 95
    ## Oct 2018      10018.557 6365.849 13671.26 4432.222 15604.89
    ## Nov 2018      13524.000 9871.292 17176.71 7937.665 19110.33
    ## Dec 2018       8437.126 4784.418 12089.83 2850.791 14023.46

One can easily check that, for Oct 2018 and Nov 2018, the real values
are in the Hi 80 category. For Dec 2018, the real value is comprised
between the point forecast and the Hi 80 value.
```R
accuracy(ARfcst1, df_Sales[(part_point+1):48, 3] )
```
    ##                      ME     RMSE      MAE       MPE     MAPE      MASE
    ## Training set  -24.73526 2720.599 2245.077 -11.76117 29.72003 0.7460306
    ## Test set     3104.43916 3260.783 3104.439  22.12055 22.12055 1.0315932
    ##                   ACF1
    ## Training set 0.1300997
    ## Test set            NA

Using MAPE as a global marker, I can assume that this forecast is a
reasonably good - slightly above 20 and clearly below 50 - which, in
turn, validates the chosen model.

![](/blog/images/figure-markdown_strict/arima_model_plot-1.png)

#### **Forecasting**

After the previous model validation steps, I proceeded to forecast the
months of Jan to Mar 2019:
```R
part_point <- 48
exo_vars <- as.matrix(df_Sales[1:part_point, c(4, 6)])

exo_vars_final_forecast <- cbind(c(1.4, 1.5, 1.6), c(1, 0, 1))
colnames(exo_vars_final_forecast) <- c('Price', 'Promo_B')

final_model <-Arima(ts_sales, 
                    order=c(0,0,0), 
                    seasonal=list(order=c(1,0,0), period=12),
                    xreg = exo_vars)

final_forecast <- forecast(final_model , 
                            xreg = exo_vars_final_forecast)
final_forecast
```
    ## Point          Forecast    Lo 80    Hi 80    Lo 95    Hi 95
    ## Jan 2019      13199.079 9591.359 16806.80 7681.547 18716.61
    ## Feb 2019       7006.897 3399.177 10614.62 1489.365 12524.43
    ## Mar 2019      10375.525 6767.805 13983.25 4857.993 15893.06

After studying this model, I believe the Sales will have the following
values:

- Jan 2019: between 13199 and 16806
- Feb 2019: between 7006 and 10614
- Mar 2019: between 10375 and 13983

### **4) Neural Networks**

Given the limited size of the dataset, I decided to continue using the
forecast library and the *nnetar* function available for making
predictions using Neural Networks.

#### **Model**

The format of both the model and the forecasting is very similar to the
one used for the ARIMA model discussed previously.

Using the same methodology as before for the training and testing
dataset, I also proceeded to iterate over the parameters *repeats* and
*size* of the model until I found a optimal configuration.

According to the *nnetar* function documentation:

-   *size is the number of nodes in the hidden layer. Default is half of
    the number of input nodes (including external regressors, if given)
    plus 1.* Tunning this parameter to higher values showed a clear
    degradation of the MAPE measure. I believe this is due to
    over-fitting, since the higher the number of this parameter, the
    higher the number of nodes in the hidden layer, therefore, the
    neural network is too fitted to the input layer and not able to
    adjust to never seen before points.

-   *repeats is the number of networks to fit with different random
    starting weights. These are then averaged when producing forecasts.*
    This parameter allows creating ensembles of similar Neural Networks
    within the model.

**Note:** These two parameters work very well together in avoiding the
*bias-variance* trade-off. Given the limited size of the dataset, a high
bias is expected. Tunning the repeats parameter, I introduce variability
in the model. In bigger datasets this is very important so one can find
the optimal point for the trade-off (via iterative testing and
measuring). It is also important to note that the same configuration can
lead to different results in different iterations, therefore
experimentation is crucial.
```R
part_point <- 45
sales_train <- ts(df_Sales[1:part_point, 3], start = c(2015, 1), deltat = 1/12)
sales_test <- ts(df_Sales[(part_point+1):48, 3], start = c(2018, 9), deltat = 1/12)

exo_vars <- as.matrix(df_Sales[1:part_point, c(4, 6)])
exo_vars_test <- as.matrix(df_Sales[(part_point+1):48, c(4, 6)])

rep <- 4
siz <- 2

nn_model <- nnetar( sales_train,
                    xreg = exo_vars,
                    repeats = rep,
                    size = siz
                  )

nn_fcst <- forecast(nn_model, xreg = exo_vars_test)
nn_fcst
```
    ##            Oct       Nov       Dec
    ## 2018  8748.257 34002.090  6966.681

I then proceeded to iterate over the parameters *repeats* and *size* of
the model until I found what seemed to be the optimal configuration
(shown above).

#### **Testing**
```R
    accuracy(nn_fcst, df_Sales[(part_point+1):48, 3] )
```
    ##                         ME      RMSE      MAE       MPE     MAPE      MASE
    ## Training set     0.0651187  1506.795 1128.520 -4.292616 14.05083 0.3750029
    ## Test set     -2808.0089096 10320.879 8394.717 -9.701942 55.43776 2.7895324
    ##                   ACF1
    ## Training set 0.1385054
    ## Test set            NA

Keeping MAPE as the global marker, this model shows reasonably good
forecasting (around 20 in the test set), which doesn't fall too behind
the ARIMA model constructed before. Since this is a good model, I will
be using it for forecasting.

![](/blog/images/figure-markdown_strict/nn_plot-1.png)

#### **Forecasting**
```R
part_point <- 48
exo_vars <- as.matrix(df_Sales[1:part_point, c(4, 6)])

exo_vars_final_forecast <- cbind(c(1.4, 1.5, 1.6), c(1, 0, 1))
colnames(exo_vars_final_forecast) <- c('Price', 'Promo_B')

final_model <-nnetar( ts_sales,
                      xreg = exo_vars,
                      repeats = 10,
                      size = 10)

final_forecast <- forecast(final_model , xreg = exo_vars_final_forecast)
final_forecast
```
    ##            Jan       Feb       Mar
    ## 2019 29570.403  7783.019 19186.913

Given that, at each iteration, different values are calculated, I
decided to only forecast the magnitude of values: March will be the
month with the higher Sales value, followed by January and then
February.

### **5) Conclusion**

During this study, I was able to perform Multivariate Time Series
Analysis and Forecasting, using both ARIMA and Artificial Neural
Networks.

The results were satisfactory although slightly biased due to the
limited size of the data sample.

The ARIMA model produced the best results when compared to the results
obtained by the Neural Networks. I believe that, in a real life
application, with more data and using the scientific method, the Neural
Network model would be able to outperform the ARIMA model.

Other non-trivial models could be considered for this study that have
been applied to Time Series with great results, such as Random Forests.
Nonetheless, the data size should be increased for such a study to be
relevant and less biased.

Traditional methods, as discussed previously, are not good candidates
for this study since the Trend is a non-conforming one. Holt-Winters is
not an option given that it is exclusively applied to univariate time
series, which is not the case for this time series.