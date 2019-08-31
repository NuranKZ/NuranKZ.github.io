---
layout: post
title:      "AB-testing on Northwind database"
date:       2019-08-31 17:25:20 +0000
permalink:  ab-testing_on_northwind_database
---


This post was written as part of a project to statistical study and hypotheses testing on Northwind dataset.

In this post, I would like to show examples of my implementation of the questions raised to the statistical study. Therefore, some technical aspects of the project, such as loading and importing data from SQL, will not be covered.

The post presents 2 examples of hypothesis testing using AB-testing (the first hypothesis), as well as multiple AB-testing and ANOVA model (the second hypothesis). The choice of this or that method is determined by the structure of the initial data.

**1.	First hypothesis. AB-testing**

The basic question posed in the project is how much the discount amount affects the volume of product ordered. From a mathematical point of view, this means comparing two or more density distributions of commodity volume samples, with one sample constructed for goods without a discount, the other or others for goods with discounted prices. If we look at the data on discounts, we can see that the number of possible discount rates is relatively small (10 rates) and varies from 1% to 25%, the total portion of discounted goods is about 38%. The important question is how much the discount size influences the volume of goods in the basket of goods in the presence of statistical dependence.

Before starting to formulate and test statistical hypotheses, I decided to perform a small visual analysis.

*fig 1.*  
![figure 1](https://i.imgur.com/7YTJjSs.jpg)

Fig. 1 shows the density of volume distribution depending on the level of discount. At the same time, I would like to note that sub-samples themselves are formed according to the accumulation principle. For example, the discount >=0.01 sample contains all goods for which the discount rate is available in principle, while the discount >= 0.02 sample already has a smaller number of observations. This principle of sub-sampling allows us to study the threshold (critical) values of the discount rate, in which the statistical significance may appear or disappear. The graph shows that the basic density (without discount) is located to the left of the other rates, which indicates the presence of a certain positive effect of the discount on the volume. However, it is more difficult to make a clear distinction between KDEs that include a discount. For example, it seems that their peak values are almost at the same level.

*fig 2.*   
![figure 2](https://i.imgur.com/PVdln7T.jpg)

Fig. 2 shows the average values of production volumes depending on the discount rate. Some important explanations should be given here. The upper gross volumes curve was plotted on the samples according to the accumulation principle, as in the previous plot. That is, the plot shows an average increase in the volume of ordered products compared to the base volume (when there is no discount) for each level of the discount rate. The second curve already compares the delta with the increase. For example, the rate of 10% corresponds to the value of product volumes equal to the increase in volumes compared to the 6% rate (the nearest previous rate). Mathematically speaking, the lower curve is a derivative of the upper curve. This graph illustrates that there is a level of rates, after which the incremental effect of the growth of the discount becomes insignificant for the volume of goods.

The statistical test consisted in comparing the average values of the samples being tested with the following parameters and assumptions:  
* Assumption: all tested samples are **independent** from each other;    
* Null hypothesis: **E{Quantity | no discount} = E{Quantity | discount }**;  
* Testing type: **2-tailed** with alpha = 0.05 (a stricter variant than the one-way test);  
* Statistical criteria: **Welch’s t-test** – due to different sample sizes;  

Besides Welch's p-value statistics calculations, the effect size (Cohen's D) and test power were also analyzed because, as it can be seen from the previous charts, there may be situations when the effect size value is small, which may lead to a drop in the test power and, as a consequence, to the impossibility of applying the test.

The result of statistical testing is shown in Fig. 3 and Fig. 4. The main difference between them is based on the following: 
* Version 1 compares the difference between the base sample (without discount) and the samples (with discount). 
* Version 2 takes as the first sample a set of values of discounted volumes of goods below the given discount level (including goods without discount), and as the second sample - goods with a discount equal or higher than the level. Thus, the second version studies the marginal effect of the impact of rates.

*fig 3*   
![figure 3](https://i.imgur.com/nNYkvz8.jpg)

*fig 4*  
![figure 4](https://i.imgur.com/i0MLWOW.jpg)

The results of the study showed that:
* In general, the presence of a discount to the price increases the volume of goods ordered (Fig. 3).
* At the same time, the marginal effect is significant only in the range of rates of 1%-10%, for higher rates the increase in the rate does not affect the growth of goods, and as a result, falls effect size and test power.

Thus, we can conclude that there is a statistically significant positive effect of the discount rate on the volume of goods within the range from 1% to 10%.


**2.	Second hypothesis. Multiple AB-testing and ANOVA model**

As part of the additional hypothesis, I asked myself what could affect the delay in delivery of goods and tested the impact of Shipper's choice on ShippedDate – RequiredDate. This example illustrates the possibility of using multiple AB testing.

There are only three suppliers in the database: Federal Shipping, Speedy Express and United Packages, with portions of 31%, 30% and 39% of the total order volume, respectively, i.e. the proportions between them are roughly evenly distributed. 

The general zero hypothesis for testing the difference is as follows:  
* **Ho: E{Delay | Federal Shipping} = E{Delay | Speedy Express} = E{Delay | United Packages}**

Taking into account that the standard AB-test works with only two samples, such a hypothesis splits into 3 separate hypotheses comparing the corresponding sub-samples in pairs. The total number of such hypotheses is equal to the number of combinations of 3 by 2, equal to 3.

The test results were as follows:
* Federal Shipping vs Speedy Express: p-value = 0.55, Cohen’s D = 0.05, test power = 0.09;
* Federal Shipping vs United Package: p-value = 0.10, Cohen’s D = 0.14, test power = 0.34;
* Speedy Express vs United Package: p-value = 0.36, Cohen’s D = 0.08, test power = 0.14.

In general, we can say that there are no significant differences, but the low value of the test power does not allow us to draw an unambiguous conclusion of the study. Therefore, in such cases, a convenient alternative is to perform the test using the ANOVA model.

The ANOVA model is also useful in cases where the number of discrete predictor values (in this case Shipper) increases, which leads to a rapid increase in the number of combinations and complicates the execution of the code in standard AB-testing.

In this case, I tested a simple variation of the model with the formula "Delay ~ C(Shipper)", p-value F-stat was equal to - 0.28, which confirmed the null hypothesis of no influence Shipper on Order Delays.


