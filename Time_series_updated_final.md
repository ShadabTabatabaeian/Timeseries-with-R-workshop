Time\_series\_updated\_version
================
Shadi
4/10/2021

# Time Series Tutorial

## 

## What is Time series data?

-   A set of observations on the values that a variable takes at regular
    time intervals.

-   A univariate time series (a single variable over time):
    **X<sub>1</sub>, X<sub>2</sub>, … X<sub>t</sub>**  
    where ***t*** is the time and ***X<sub>t</sub>*** is the value of
    the time series at a particular point.

### Examples:

-   Weather forecast (daily temperature in Merced)

-   Business (sales and prices)

-   Economics (minute changes in stock market)

-   Neuroscience (EEG)

-   Ups and downs of relationship over time

-   Changes in the ecosystem of a lake over time

-   And more …

## What is time series analyses?

-   Understanding how a metric changes over time.

-   Forecasting future changes.

### Key concepts:

### **1- Autocorrelation**

Autocorrelation refers to the degree of similarity between a given time
series and a lagged version of itself over successive time intervals.

**X<sub>t = Merced temperature today</sub>**

**X<sub>t-1 = Merced temperature a day ago</sub>**

**X<sub>t-2 = Merced temperature two days ago</sub>**

(The number represents the lag or the period apart between pairs of
observations)

**Why should we care?** Autocorrelation helps us study how each time
series observation is related to its recent past. (Hint: early warning
signals!)

### 2- Stationarity

For some time series analyses, we assume that the statistical properties
(mean, variance, autocorrelation) of a time series do not change over
time.

<u>It does not mean that the series does not change over time, just that
the *way* it changes does not itself change over time.</u>

![Example of stationary
data](images/stationary.png "Example of stationary data")

A stationary series should show random oscillation around the mean.

However, the same type of random behavior often holds from one time
period to the next. For example, Merced temperature may have different
behavior from the previous year. But the mean, standard deviation, and
other statistical properties are often similar from one year to the
next.

![Example of non-stationary
data](images/Non_stationary.png "Example of non-stationary data")

**Why should we care?**

A stationary process can be modeled with fewer parameters and makes
things easier!

## Getting started with R…

### Data: Mathematical Insight

![](images/screen-shot-2021-04-13-4-52-26-pm.png)

![](images/screen-shot-2021-04-12-1-12-19-pm.png)

``` r
##What does the data look like?
library(tidyverse)
```

    ## ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
    ## ✔ dplyr     1.1.4     ✔ readr     2.1.5
    ## ✔ forcats   1.0.0     ✔ stringr   1.5.2
    ## ✔ ggplot2   3.5.2     ✔ tibble    3.3.0
    ## ✔ lubridate 1.9.4     ✔ tidyr     1.3.1
    ## ✔ purrr     1.2.0     
    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter() masks stats::filter()
    ## ✖ dplyr::lag()    masks stats::lag()
    ## ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors

``` r
experts_and_insights <- read_csv("experts_insights_combined.csv")
```

    ## Rows: 4750 Columns: 14
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr  (4): subName, problemName, subProb, Event
    ## dbl (10): start, end, duration, ob, problemNum, obNew, totalObjects, startn,...
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
Tutorial_data<- experts_and_insights %>%
select(subName, problemName, start, obNew, Event ) %>%
arrange(subName, problemName,start)

head(Tutorial_data, 5)
```

    ## # A tibble: 5 × 5
    ##   subName problemName start obNew Event    
    ##   <chr>   <chr>       <dbl> <dbl> <chr>    
    ## 1 Dami    Function     2300     1 attention
    ## 2 Dami    Function     6700     6 attention
    ## 3 Dami    Function     9550     6 attention
    ## 4 Dami    Function    12250     6 attention
    ## 5 Dami    Function    14650     6 attention

``` r
tail(Tutorial_data, 5)
```

    ## # A tibble: 5 × 5
    ##   subName problemName   start obNew Event    
    ##   <chr>   <chr>         <dbl> <dbl> <chr>    
    ## 1 Steven  Triangle    1129610    10 attention
    ## 2 Steven  Triangle    1129920    11 attention
    ## 3 Steven  Triangle    1137215    17 attention
    ## 4 Steven  Triangle    1141650    16 attention
    ## 5 Steven  Triangle    1145470    NA give_up

``` r
Tutorial_data %>% 
filter(subName == "Dami", Event=="insight")%>%
print()
```

    ## # A tibble: 2 × 5
    ##   subName problemName  start obNew Event  
    ##   <chr>   <chr>        <dbl> <dbl> <chr>  
    ## 1 Dami    Function    203000    NA insight
    ## 2 Dami    Triangle     91000    NA insight

### What are we investigating?

Is there a change in the way one interacts with notations right before
the insight occurs? is there a change in the attentional shifts before
insight?

Would we see more variability 30s prior to insight compared to random
30-second slices?

``` r
## Loading cool packages:
library(magrittr)
```

    ## 
    ## Attaching package: 'magrittr'

    ## The following object is masked from 'package:purrr':
    ## 
    ##     set_names

    ## The following object is masked from 'package:tidyr':
    ## 
    ##     extract

``` r
library(tidyverse)
library(scales)
```

    ## 
    ## Attaching package: 'scales'

    ## The following object is masked from 'package:purrr':
    ## 
    ##     discard

    ## The following object is masked from 'package:readr':
    ## 
    ##     col_factor

``` r
library(gridExtra)
```

    ## 
    ## Attaching package: 'gridExtra'

    ## The following object is masked from 'package:dplyr':
    ## 
    ##     combine

``` r
theme_set(theme_classic())
```

## Data Examination:

The first question to ask: Is my data stationary?

Three ways to find out:

1- Visualize the time series **plot** and look for constant mean and
variance:

``` r
Dami<- Tutorial_data%>%
  filter(subName== "Dami", problemName == "Function")
```

``` r
plot(Dami$start/1000, type= "l", xlab = "Time", ylab="Attention shifts/ms")
```

![](Time_series_updated_final_files/figure-gfm/unnamed-chunk-5-1.png)<!-- -->

2- Look at the **autocorrelation plot**. For stationary data, the
correlation between the time series across successive lags should
approach 0 very quickly.

``` r
acf(Dami$start/1000, main = "Autocorrelation Plot")
```

![](Time_series_updated_final_files/figure-gfm/unnamed-chunk-6-1.png)<!-- -->

3- Perform **unit root hypothesis test**,such as the augmented
Dickey-Fuller test (ADF). The null hypothesis is non-stationarity but
the alternative hypothesis is stationarity.

``` r
cat("ADF test not run in this GitHub version because the tseries package is not installed in this environment.\n")
```

    ## ADF test not run in this GitHub version because the tseries package is not installed in this environment.

The null cannot be rejected. Obviously, this data is not stationary.
What can we do?

## Data Preparation

### Making data stationary: Differencing

It can be used to remove the series dependence on time (the linear trend
of time).

Differencing can help stabilize the mean of the time series.

Differencing is performed by subtracting the previous observation from
the current observation:

***X<sub>t</sub>*−*X<sub>t−1</sub>***.

``` r
#Differencing in R
Dami_diff <-Dami %>% 
mutate(first_diff=c(NA, diff(start, 1, 1))) %>%
print()
```

    ## # A tibble: 121 × 6
    ##    subName problemName start obNew Event     first_diff
    ##    <chr>   <chr>       <dbl> <dbl> <chr>          <dbl>
    ##  1 Dami    Function     2300     1 attention         NA
    ##  2 Dami    Function     6700     6 attention       4400
    ##  3 Dami    Function     9550     6 attention       2850
    ##  4 Dami    Function    12250     6 attention       2700
    ##  5 Dami    Function    14650     6 attention       2400
    ##  6 Dami    Function    19250     6 attention       4600
    ##  7 Dami    Function    21350     6 attention       2100
    ##  8 Dami    Function    23300     6 attention       1950
    ##  9 Dami    Function    29800     6 attention       6500
    ## 10 Dami    Function    30550     2 attention        750
    ## # ℹ 111 more rows

``` r
Dami_diff %>% ggplot( aes(x=start/1000, y=first_diff)) + 
  geom_line() + ylab("Shifts in attention") + xlab("Time")
```

    ## Warning: Removed 1 row containing missing values or values outside the scale range
    ## (`geom_line()`).

![](Time_series_updated_final_files/figure-gfm/unnamed-chunk-9-1.png)<!-- -->

``` r
acf(na.omit(Dami_diff$first_diff), main = "Autocorrelation Plot")
```

![](Time_series_updated_final_files/figure-gfm/unnamed-chunk-10-1.png)<!-- -->

``` r
Dami_diff_na<- na.omit(Dami_diff)
cat("ADF test not run in this GitHub version because the tseries package is not installed in this environment.\n")
```

    ## ADF test not run in this GitHub version because the tseries package is not installed in this environment.

## Data Analyses

``` r
Dami_diff%>%
  ggplot(aes(x=start/1000, y=first_diff)) + 
  geom_line() + 
  geom_vline( data=Dami_diff %>% 
  filter(Event== "insight"),aes(xintercept=start/1000, linetype= 'dashed', color='red')) + 
  ylab("Shifts in Attention") +
  xlab("Time/s")+
  ggtitle("The insight moment in the time series")+
  theme_classic()
```

    ## Warning: Removed 1 row containing missing values or values outside the scale range
    ## (`geom_line()`).

![](Time_series_updated_final_files/figure-gfm/unnamed-chunk-12-1.png)<!-- -->

1- We need to calculate a variability measure, like SD, for an insight
moment and 30 seconds prior.

2- We need a frame of reference for the insight slice. So, we have to
make a distribution of, let’s say 1000, SDs for random 30 second slices
in our data.

3- We then compare the insight SD to the distribution to see if it is
any different.

``` r
#Calculating the sd for Dami's insight slice:
sd_insights_demo<-tibble()
    for (i in 1:length(Dami_diff$first_diff)){
      if (Dami_diff$Event[i] == "insight"){ 
        insight_moment <- Dami_diff[i,]
        prior_insight <- insight_moment$start - 30000 
        slice <- Dami_diff %>% filter(start <= insight_moment$start &  start  >= prior_insight)
        SD_Prior_ins<- sd(slice$first_diff,na.rm = TRUE) 
        sd_insights_demo <- bind_rows(sd_insights_demo, tibble(sd=SD_Prior_ins)) 
      }
    }  
```

``` r
sd_insights_demo
```

``` r
#Simulating a distribution of SDs based on Dami's data:

sd_dist_demo<-tibble()
    for (i in 1:1000){
      random<-sample(Dami_diff$start, 1)
      prior<- random - 30000
      slice <- Dami_diff %>% filter(start <= random & start >= prior)
      SD_Prior<- sd(slice$first_diff,na.rm = TRUE)
      print(SD_Prior)
      sd_dist_demo <- bind_rows(sd_dist_demo, tibble(iteration=i,sd=SD_Prior))
    }
```

``` r
#Make an empirical cumulative distribution based on the observed data:

ecdf_Dami<- ecdf(sd_dist_demo$sd)

plot(ecdf_Dami)
abline(v=1869.728, col="red", lty="dashed") #percentile for the insight sd
```

![](Time_series_updated_final_files/figure-gfm/unnamed-chunk-16-1.png)<!-- -->

``` r
#the estimated probability that the area of a sample is less than or equal to 1869 is about 0.52.

ecdf_Dami(1869.728) 
```

    ## [1] 0.5320448

Maybe 30 second slides are very large, and we should check shorter
window sizes.

The possibilities are infinite! You could do entropy….

Remembering some methodologies used in articles we have been reading.

## Modeling and Forecast

-   **Autoregressive models (AR):**

    -   *Predictor: past values (lags or p) of the time series variable*
        .

    -   Today’s observation is regressed on yesterday’s observation.

    -   The main parameter is the number of lags or p.

-   **Moving average models (MA):**

    -   Predictor: the past noise!

    -   Today’s observation is regressed on yesterday’s noise.

    -   The main parameter to include is q or the number of past random
        noises.

-   **Autoregressive Moving Average Models (ARMA)**

    -   Main parameters: q and p

-   **Autoregressive integrated Average Models (ARIMA)**

    -   parameters: q,d,p

### Example:

``` r
##Cookie sale data
Cookie<-BJsales
```

``` r
#The number of cookies that were sold in the last 150 days
head(Cookie,5)
```

    ## Time Series:
    ## Start = 1 
    ## End = 5 
    ## Frequency = 1 
    ## [1] 200.1 199.5 199.4 198.9 199.0

``` r
plot(Cookie, ylab="Number of sold cookies", xlab="days")
```

![](Time_series_updated_final_files/figure-gfm/unnamed-chunk-20-1.png)<!-- -->

### Check for parameters (p,d,q):

``` r
#Check for AR terms (p): the slope suggests that we should include at least one lag.
acf(Cookie, main = "Autocorrelation Plot")
```

![](Time_series_updated_final_files/figure-gfm/unnamed-chunk-21-1.png)<!-- -->

``` r
#Check for MA terms (q): the one significant line indicates one MA term (lagged noise or q).
#computes the correlation adjusting for previous lags/periods 
pacf(Cookie, main = "Partial Autocorrelation Plot")
```

![](Time_series_updated_final_files/figure-gfm/unnamed-chunk-22-1.png)<!-- -->

### Fit the AR model:

``` r
cookie_model<- arima(Cookie, order = c(1, 1, 1))
print(cookie_model)
```

    ## 
    ## Call:
    ## arima(x = Cookie, order = c(1, 1, 1))
    ## 
    ## Coefficients:
    ##          ar1      ma1
    ##       0.8800  -0.6415
    ## s.e.  0.0644   0.1035
    ## 
    ## sigma^2 estimated as 1.775:  log likelihood = -254.37,  aic = 514.74

``` r
#ar1: coefficient for a lag of one. It's 0.88: You can predict today's value from yesterday's value.
#The closer to one, the more predictable.
#ma1: lags of the forecast errors.
#There are measures such as log likelihood and aic for fit comparison.
```

### Check the model fit:

``` r
#the model does a good job predicting currrent values from the previous one
ts.plot(Cookie)
AR_fitted_2 <- Cookie - residuals(cookie_model)
points(AR_fitted_2, type = "l", col = 2, lty = 2)
```

![](Time_series_updated_final_files/figure-gfm/unnamed-chunk-24-1.png)<!-- -->

``` r
#The residuals are behaving well.
Box.test(residuals(cookie_model), type ="Ljung-Box", lag=1)
```

    ## 
    ##  Box-Ljung test
    ## 
    ## data:  residuals(cookie_model)
    ## X-squared = 0.10523, df = 1, p-value = 0.7456

``` r
#residuals are behaving well
acf(residuals(cookie_model), main = "Residual Autocorrelation Plot")
```

![](Time_series_updated_final_files/figure-gfm/unnamed-chunk-26-1.png)<!-- -->

### Forecasting:

``` r
cookie_forecast <- predict(cookie_model, n.ahead = 5)
plot(cookie_forecast$pred, type = "o", pch = 16,
     xlab = "Forecast Horizon", ylab = "Predicted Sales",
     main = "Five-Step Forecast")
```

![](Time_series_updated_final_files/figure-gfm/unnamed-chunk-27-1.png)<!-- -->

![](images/8a52958922229b2b83026f8d6baba00bd362c708320b80bb2e701e30e28c8f33_1-01.jpg)

## Useful sources:

Data Camp course: [Time series analyses in
R](https://learn.datacamp.com/courses/time-series-analysis-in-r)

Coursera course: [Intro to time series analyses in
R](https://www.coursera.org/projects/intro-time-series-analysis-in-r)

Intro to time series:
<https://www.aptech.com/blog/introduction-to-the-fundamentals-of-time-series-data-and-analysis/>

Autocorrelation:
<https://dganais.medium.com/autocorrelation-in-time-series-c870e87e8a65>

Stationarity: <https://otexts.com/fpp2/stationarity.html>

Modeling:
<https://towardsdatascience.com/the-complete-guide-to-time-series-analysis-and-forecasting-70d476bfe775>
