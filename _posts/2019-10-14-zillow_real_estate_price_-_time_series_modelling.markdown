---
layout: post
title:      "Zillow Real Estate Price - time series modelling"
date:       2019-10-14 11:40:28 -0400
permalink:  zillow_real_estate_price_-_time_series_modelling
---


Hi!  
This post was written as part of a project to time series modelling on Zillow real estate prices dataset.  

The initial task set for us was to get an answer to the question, which 5 regions among those represented may be the best for investment. And before we move on to the main part of the blog, let me briefly describe the structure of the data. The data for work was a collection of more than 14,700 time-series corresponding to the regions and built on a single time scale. The time scale covered 265 months, or 22 years and 1 month. It is easy to calculate that the total amount of numerical data is about 3.9 million digits.  

To assess the investment efficiency, I chose the IRR parameter, the variation of which also calculated the risk measure. By the way, the IRR estimation is available as a built-in function in Numpy : `np.irr(array)`. All IRR estimates were based on forecast data, i.e., after the selected and adjusted predictive models, and I have already ranked the regions to answer the question about the top 5 regions.

In this post, I would like to show you the part of my work in which I had to find ways to optimize the volume of calculations to achieve a balance of calculation time and relative accuracy and quality of forecasts.

The easiest but inefficient way to do it in terms of calculations is to build 14,700 completely autonomous and unrelated models. Each model has its SARIMA parameters, which, depending on the range of parameters to be searched, can also be a capacious task. By the way, running one such model with the range `p = d = q = range(0.6)`, `s=12` could take several hours. Thus, I have been asked 2 conceptual questions:
*   Timeline: per month (as in original dataset) or per annum (resized);
*   Unified SARIMA parameters or individual for each region.  

Next, I will show you the results of the analysis on the first question: Month vs Year.   

Figure 1 below shows the regionally averaged price based on monthly data. The chart looks quite smooth without visible seasonality and fluctuations inside the year. However, this may be a consequence of the average price by region, so such conclusions are conditional. On the other hand, it is unproductive to study and draw separate conclusions on each of the 14,700 regions.  

fig.1

![Imgur](https://i.imgur.com/mQOUkk8.png?3)

Next, I created rolling-prices with 12-month window and compared it with original monthly prices (in Figure 2).

fig.2

![Imgur](https://i.imgur.com/0xm4do5.png?1)

As you can see, based on the visual representation, the graphs are similar to each other in form, and the standard deviation is close to 0 and has no serious fluctuations. This allowed me to choose an annual basis for further modelling, which reduced the amount of data by a factor of 12.  

I conducted the simulation itself according to the following scheme:
1. In the first step, I selected the optimal SARIMA parameters based on the regionally averaged time series (on an annual basis). In particular, I had parameters (0, 1, 1, 1) (0, 1, 1, 1)
2. In the second step of the loop by region, I am taking into account the parameters set to create and fit a separate model for each region.
3. In the third step, I left in the sample only the models of the regions with the AIC estimation lower than AIC according to the averaged price model and calculated IRR indicators for them.
4. The final step was the ranking procedure.  

After receiving the results of modelling and ranking, I decided to ask myself the question that maybe annual resizing was quite an extreme step and decided to test it additionally. In Fig.3 and Fig.4 you can see the histograms of residuals for annual and monthly models respectively. As you can see, the distribution of residuals of the annual model is closer to N(0.1) than that of the monthly model. Perhaps the reason for this is not the best selection of parameters for the monthly model (I used the found optimums (1,2,2,2)(2,2,2,12)).

fig.3

![Imgur](https://i.imgur.com/3rVZEDg.png?1)

fig.4

![Imgur](https://i.imgur.com/nCTOa0z.png?1)

However, let's take a look at and compare the 3-year forecast charts for the first model (Fig. 5) and the 36-month forecast for the second model (Fig. 6)  

fig.5

![Imgur](https://i.imgur.com/wrnXBGL.png?1)

fig.6

![Imgur](https://i.imgur.com/F8EGTOP.png?1)

As a result, the use of monthly forecasts seems to be more accurate, which is noticeable in the dynamics of the confidence intervals change. 
Thus, the use of resizing makes sense only when the calculation task (in case of a project - ranking) is not so fundamental to the forecast period. In other words, if the primary task was not so much to rank, as to obtain a forecast model directly for its further use in industrial purposes, I would have left the monthly calculation despite the increase in the volume of calculations.


