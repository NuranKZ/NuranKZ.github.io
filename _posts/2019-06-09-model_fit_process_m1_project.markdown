---
layout: post
title:      "Model fit process (M1 project)"
date:       2019-06-09 14:28:34 -0400
permalink:  model_fit_process_m1_project
---


Hi!

   This post was written as part of a project to study pricing factors dataset King County Houses Sales.
A few words about this project.  The main goal of the project is to build an effective regression model (s), and the linear model calculated using the OLS method was used as the model type.  The raw data included 20 variables that could theoretically be used as predictors, the total sample size contained 21,597 samples, which was reduced to 16,762 observations during the detailed study due to missing data, incorrect data and outliers.  Also undergone changes and features, some of which were partially converted to dummy variables due to the discrete nature.  Some variables were excluded from the sample due to multicollinearity or the absence of any influence on the target.

   The purpose of this post is to describe the following process in the  OSEMN framework - the process of modelling the prepared data.  Immediately here I’d like to make a disclaimer that the stated points and conclusions do not claim to be absolute truth, but only set forth my opinion in the process of learning process of such an interesting discipline as Data Science.  I also want to note that I will not show the code here since it is quite voluminous and in some places contains references to variables, objects, functions and classes from other sections of the project, so it may be incomprehensible for those who do not see the whole program.  The main idea of the post is to share a general idea of how to implement the modelling phase.  Therefore, some results of code execution, including visualization will be included in this post.
 
In my work, the modelling process was conditionally divided into 4 successive stages:
 *1) Feature selection;  
 2) Testing the significance of features;  
 3) Determine the optimal level of subsets for training and testing;  
 4) Testing regression residuals on the selected model.*  

 I offer a brief overview of the applied algorithms and their results for each of the above stages.
 
 
**1)	Feature selection**  

   The dataframe containing features data consisted of 66 variables, most of which were dummy variables created from categorical-type variables.  Despite the fact that many of these variables could be identified as ordinal, they were nevertheless translated into dummy-types, since the nature of the price dependence on them was not unambiguous. In other words, it was impossible to say for sure that an increase in the factor led to an increase or decrease  prices for the entire range of factors.  This effect can be seen in Fig.1.
 

![](https://imgur.com/RJsw7fR.png)

Figure 1. KDE for "grade" feature  


   And in such cases, the consideration of each discrete value as a separate feature is, of course, more convenient.  At the same time, such replication led to a significant increase in the number of variables, and, as a result, to design complexity.  As follows from many theoretical materials, an excessive number of variables in the regression can lead to model overfitting, which makes it useless in real projects.  
   
To determine the effective number of variables, the RFE method was chosen, which is implemented in the SKLEARN library and imported from the team of `sklearn.feature_selection import RFE`.  The essence of the method is the ranking of variables in order of importance and selection of the best for target prediction variables for a given number.  However, in this one-time execution of the method, there is one drawback - we do not know how many variables are effective.  Therefore, this method is convenient for its implementation in the loops, in which such a number is iterated.  I in my project asked a rather laborious variant of enumeration: from 5 to 60 with a step of 1, i.e.  55 cycles.  Given that the sample for the project is relatively small, such detailing seemed reasonable to me.
   
 The choice of the optimal variant or variants was based on the implementation of the k-fold cross-validation algorithm in this cycle.  This check was implemented as follows: at each iteration, when the optimal set of features was recorded, the target and predictors samples were cut into 80/20 train and test subsets, after which the MEAN SQUARED ERRORS (MSE) values were formed, separately for test and train subsamples.  Given the separation of 80/20, such a step within the above iteration was repeated 5 times.  In other words, an internal loop was run to collect data on MSE.  The resulting MSE set was averaged and recorded as the result of the effectiveness of a particular combination of features.
   
 The importance of cross-validation was that the principle of dividing the sample into training and testing parts was preserved for assessing the quality of the model.  Of course, here we could strengthen the general scheme in order to run a cross-check as an external loop and recursively select parameters inside.  But, it seemed to me, and in my version, the results gave some stability of conclusions based on the visual analysis of the learning curve (Figure 2, left side).  Let's explore this chart.
 
 ![](https://imgur.com/P1jJBKB.png)
 
Figure 2. Learning Curve  

The x-axis represents the number of features, according to Oy, the averaged based on k-fold cross-validation value of MSE.  This value is presented in the form of negative numbers, which is associated with the program features of the cross_val_score function of the sklearn.model_selection module, but this does not change the essence of the conclusions; the smaller the absolute value of MSE, the better the forecast shows the regression.  It is important to note that this plot shows the values for the test sample values.
 As you can see, the shape of this curve is of a logarithmic type, with a decrease in growth rates as the independent variable increases.  The graph Y maximum is reached at maximum X, but visually you can see that the nature of the curve changes dramatically at point 23, after which the curve becomes fairly flat.  For some verification and confirmation of the findings on MSE, I also calculated the R-squared adjusted value in each iteration of the external (RFE) loop.  The summary of the results is shown in Figure 2 (right side) and is generally very similar in behavior to MSE, with the exception of the small volatility of the parameter Rsq in the X range from 50 to 60.
 For numerical values, the optimal options (min for |MSE| and maximum for R-sq) were 54 and 55 variables, respectively.  Based on the above conclusions, I chose two options for the subsequent stages: 55 variables - as the most optimal mathematically, and 23 variables - as an amount that seems to be the most balanced.
 
 
**2)	Testing the significance of features**

   Both of the selected options (23 and 55) actually contain a fairly large number of parameters, so it seemed to me right to test them for the significance of the coefficients (t-statistics) before proceeding to the final step of the simulation.  The significance of the coefficient lies in testing the null hypothesis that the coefficient value is 0. If the corresponding P-value exceeds the specified significance level (taken at 95%), this means that the probability of an alternative hypothesis (inequality to 0) is below 95% and  the feature is considered insignificant for regression purposes.
 Here I used the ran OLS-regression based on the whole datasets.  It is convenient to do this task using the statsmodels library.  The results can be seen in Table 1.



Table 1. RFE results

![](https://i.imgur.com/swJaMj6.png)

Thus, the final scenarios for the next step contained 50 and 21 variables for scenario 1 and 2, respectively.


**3)   Determine the optimal level of subsets for training and testing**

At this stage, the linear regression is calculated using the train_test_split function contained in the `sklearn.model_selection` module.  The purpose of this stage is to determine the train/test proportion level at which, on the one hand, regression quality indicators are optimal, on the other hand, the difference between these indicators for a train set and test set is minimal.  The implementation of the algorithm, as before, was carried out on the basis of loops with an enumeration of the train portion from 10% to 30% with step of 5% (6 iterations in total).  Figure 3 shows the results of calculations: on the left graph - the R-squared adjusted indicator, on the right - MSE (already in the form of positive values).  Both graphs show calculations on a test set.
 
![](https://imgur.com/P1jJBKB.png)

Figure 3. TTS results  

 
   The R-sq graph demonstrates enough U-shape for 21-feat-case and ^ - shape for 50-feat-case, and the only conclusion that can be made here is the intersection of the curves at 25%.  The MSE chart is more uniform for both scenarios (with U-shape) and shows a local minimum at 25%.  However, as mentioned above, it is also necessary to look at the convergence of the results for both samples.  Table 2 below shows the optimal values of the test fraction by Rsq and MSE values and calculated for each sample separately. 


Table 2. Optimal K in TTS	

![](https://i.imgur.com/GqT5taq.png)  


   As can be seen from the table, the variability of the final results is present, however, the variations of the indicators themselves (Rsq and MSE) are insignificant, and the most common value is 25%.  Here you can make a reservation that there is some subjectivity of conclusions, including the fact that the train-test-split algorithm cuts the sample randomly, i.e.  other results are possible with repeated iterations.  Of course, for the purity of the experiment, the enumeration algorithm could be expanded with an internal loop, with train-test-split generation generated N-times and with a random engine initialized in each inner iteration.  However, taking into account the best practice (20% - 25%) and a small difference of the Rsq and MSE values for different configurations, I considered these findings to be sufficient for choosing the level of 25% for the test sample and moving to the next stage.
	 
	 
**4)	Residuals testing**

   The essence of this stage is to generate and test residuals of the selected regression model on the properties of the normal distribution, the zero value of the expectation, and the absence of heteroscedasticity of the variance.
 
 ![](https://imgur.com/g41BBpC.png)

Figure 4. Residuals. Case 2 (21 features)  


   Figure 4 shows the plots of the residuals of both scenarios in the form of a KDE-plot and QQ-plot.  Visually, the distribution of residuals has a symmetric bell-shape, as evidenced by low skew values in the range of 0.07 to 0.1.  However, as can be seen from the QQ-plot graph, there is some problem with tails, namely, the distribution is platykurtic and the tails are “heavy”.  This means that at edge values close to q10 and q90, the difference between real and predicted values can grow.  Let's take a look at the plot in Figure 5, where the real prices are located along the X-axis, and the predicted prices are on the Y-axis.  Here, as a model, I took case 2 with 21 independent variables.
 
 ![](https://imgur.com/xaEuKpe.png)

Figure 5. Price vs Predicted Price  

   The problem found on QQ-plot is also confirmed here: starting from the level of 120,000 - 130,000, the size of the regression error increases.  This is especially noticeable in the interval starting from the level of 180,000, which mainly contains outliers.


**Conclusions**   

   In general, it cannot be said that the regression model obtained is the most optimal, the resulting R-sq (70% -75%) variant does not seem high.  As factors for improvement, I see, above all, work with emissions, which is carried out at the EDA stage, as well as more diverse options for working with categorical data.  Thus, my main conclusion is that balancing a regression model is an iterative process that covers almost all stages of the process within OSEMN, starting with Data Scrubbing to the Model fit stage.

