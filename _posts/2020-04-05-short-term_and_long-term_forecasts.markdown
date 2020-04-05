---
layout: post
title:      "Short-term and Long-term forecasts"
date:       2020-04-05 13:15:00 +0000
permalink:  short-term_and_long-term_forecasts
---


The topic of this short post is devoted to the use of recursive neural networks for forecasting of share quotations.
As part of my capstone project I was engaged in building ARIMA and LSTM models to create predictive models. In this post, I will build short and long term forecasts charts based on LSTM models.

#### Evaluation parameters
Before we proceed to the results, I would like to say a few words about how this curve will be built.

**Data to be used**
I will use only 1 real company which, among others, was used in the initial project and showed good R2 metrics. Quotes data are time series with daily close prices and cover about 1000 days. Also special data will be added to the space: day of month and month

**Splitting the sample**
Models will be trained on the first 80% of the sample, the remaining 20% represent the test part (216 days). Taking into account that LSTM data architecture assumes some window, actual learning period will be reduced by window size.

**Architecture selection**
Many individual models will be customized for each company.  In this case the set of models is based on window size variations: 10 days, 30 days, 90 days.  
In this case, the selection of optimal parameters will be made with the following limitations:

- Main architecture: 1 LSTM – 1 DropOut – FC(1)
-	Validation subset portion: 0.15
- Batch size: 0.16
-	Range for LSTM nodes: {32, 96, 128}
-	Range for DropOut: {0.1, 0.15, 0.25}
-	Activation function: {ReLU, Tanh}

**Generation of long-term forecast**
To generate a long-term forecast, we used our own function, which iteratively launches a forecast for 1 day followed by an update of the input tensor, where we use previous forecasts as real data.

The following optimal architectures were found for model training

- Window size = 10: LSTM(32, ReLU) - DropOut(0.10) - FC(1)
- Window size = 30: LSTM(128, ReLU) - DropOut(0.10) - FC(1)
- Window size = 90: LSTM(128, ReLU) - DropOut(0.10) - FC(1)


#### Analysis results
Let's have a look at the forecast charts for each variant of the window.

![Imgur](https://i.imgur.com/9SR6DiT.png?2)

As you can see, the short-term forecasts charts show high accuracy, while the long-term forecasts charts are unsatisfactory. It should be noted here that the forecast duration for each variant of the window is different and increases as the window decreases. The 30-day window variant has the best results.   

Thus, the use of even such complex models as RNN does not guarantee the quality of forecasts in the long term, where the degree of uncertainty increases rapidly. In addition, the models presented are very simple in terms of input data, where only historical data and dates are taken into account. In reality, the most difficult task is to identify the right influencing factors, whose inclusion in the model can improve the quality of the forecast.


