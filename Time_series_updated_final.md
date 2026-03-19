# Time Series Tutorial

## What is Time Series Data?

- A set of observations on the values that a variable takes at regular time intervals.
- A univariate time series tracks a single variable over time: **X1, X2, ... Xt**.
- Here, **t** is time and **Xt** is the observed value at a given moment.

### Examples

- Weather forecasts
- Sales and prices
- Stock market changes
- EEG signals
- Ecosystem change
- Behavioral change over time

## What Is Time Series Analysis?

Time series analysis helps us understand how a metric changes over time and how to forecast future changes.

### Key Concept 1: Autocorrelation

Autocorrelation describes how similar a time series is to lagged versions of itself over successive time intervals.

- **Xt** = today's value
- **Xt-1** = yesterday's value
- **Xt-2** = value from two days ago

This matters because it tells us how strongly current observations depend on the recent past.

### Key Concept 2: Stationarity

Many time series methods assume that the statistical properties of the series, such as the mean, variance, and autocorrelation structure, do not change over time.

![Example of stationary data](images/stationary.png)

A stationary series tends to fluctuate around a stable mean.

![Example of non-stationary data](images/Non_stationary.png)

Non-stationary series often show trends, changing variance, or shifting correlation structure over time.

## Data: Mathematical Insight

![](images/screen-shot-2021-04-13-4-52-26-pm.png)

![](images/screen-shot-2021-04-12-1-12-19-pm.png)

## What Are We Investigating?

The motivating question is whether interaction with notation changes right before an insight occurs.

More specifically:

- Is there a change in attentional shifts before insight?
- Do we see more variability in the 30 seconds before insight than in random 30-second slices?

## Data Examination

The first question is whether the data are stationary.

Three common checks are:

1. Visualize the series and inspect whether the mean and variance look roughly constant.
2. Inspect the autocorrelation plot. For stationary data, autocorrelation should decline toward zero relatively quickly.
3. Run a unit root test such as the augmented Dickey-Fuller test.

In this example, the null of non-stationarity is not rejected, so the series is treated as non-stationary.

## Data Preparation

### Making Data Stationary: Differencing

Differencing can reduce trend-like dependence on time and stabilize the mean of a time series.

The first difference is:

**Xt - Xt-1**

After differencing, the series can be checked again with visualization, autocorrelation, and the ADF test.

## Data Analysis

The workflow used here is:

1. Calculate a variability measure such as standard deviation for the 30-second window before each insight.
2. Build a reference distribution from many random 30-second windows in the same data.
3. Compare the observed insight-window variability to that reference distribution.

This makes it possible to ask whether the pre-insight slice looks unusual relative to random slices.

## Modeling and Forecasting

Common model families include:

- **AR**: autoregressive models based on past values
- **MA**: moving average models based on past noise
- **ARMA**: combines autoregressive and moving average terms
- **ARIMA**: extends ARMA with differencing

### Example Forecast Output

![](images/8a52958922229b2b83026f8d6baba00bd362c708320b80bb2e701e30e28c8f33_1-01.jpg)

## Useful Sources

- [DataCamp: Time Series Analysis in R](https://learn.datacamp.com/courses/time-series-analysis-in-r)
- [Coursera: Intro to Time Series Analysis in R](https://www.coursera.org/projects/intro-time-series-analysis-in-r)
- [Introduction to the Fundamentals of Time Series Data and Analysis](https://www.aptech.com/blog/introduction-to-the-fundamentals-of-time-series-data-and-analysis/)
- [Autocorrelation in Time Series](https://dganais.medium.com/autocorrelation-in-time-series-c870e87e8a65)
- [Stationarity](https://otexts.com/fpp2/stationarity.html)
- [The Complete Guide to Time Series Analysis and Forecasting](https://towardsdatascience.com/the-complete-guide-to-time-series-analysis-and-forecasting-70d476bfe775)
