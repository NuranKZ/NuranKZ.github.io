---
layout: post
title:      "Benefits of PCA"
date:       2019-12-12 15:22:55 +0000
permalink:  benefits_of_pca
---


Hi!

In this post, I will consider how useful it can be to reduce the dimension of the data within the task of building an effective classifier on real data.

This post is written in the course of the project of module 5. The initial task set for the project is to select and agree with the instructor the real data and build a classification model on that. 

I chose the dataset "TripAdvisor Restaurants Info for 31 Euro-Cities", available on Kaggle, in which:

* Available in about 125k samples;
* 11 columns.

After the data cleaning process and conversion of categorical data, the dataset changed its size to 78K samples and became rather sparse. 156 out of 159 predictors were sparse dummy-variables consisting of two groups: cities and cuisine types. The target variable "Price Range" consisting of three classes was chosen.

Of course, within the framework of the model learning process, such a dataset for several algorithms in the conditions of their launch on ordinary PCs seems to be a computationally laborious task and the reduction of dimensionality in this situation looks quite reasonable.

In general, the project itself consisted of the following stages:
* Data cleaning;
* EDA;
* PCA;
* Clustering;
* Grid Search;
* Optimal models fit and comparison;
* Monte-Carlo simulation on the final model.

The following classification algorithms were used within the framework of the project: Logistic Regression, Decision Tree, Naïve Bayes Classifier, KNN, SVC, XG-Boost, ADA-Boost, Random Forest. The best results were shown by the XG-boost and Linear Regression algorithms, but in general, the difference in accuracy score between the models was insignificant and the maximum difference is 5%. 

I suggest you take a look at the comparative analysis of the two models. By the way, I checked the correlation matrix of features before launching the PCA, for which there was no high correlation between the variables in general. This may indicate that there may be limitations to a significant reduction in dimensionality, provided that the information is preserved.

Fig. 1 shows the graph of total variance's dependence on a number of principal components.

Fig. 1
![Imgur](https://i.imgur.com/WZDFPK9.png)

As can be seen, having reduced the dimension by 57 variables out of 159 (36%), the feature matrix loses only 1% of the information.

Fig.2 shows graphs of the dependence of accuracy score on the test set of a number of PCA, built for Logistic Regression and XG-Boosting models. As a range of the PCAs under consideration, I take the range from 2 to 159.

Fig. 2
![Imgur](https://i.imgur.com/dl3Eaft.png)

As a whole, despite the fact that the accuracy value almost does not fall below 70%, I would like to note that this level in the project under consideration is actually critical and is marked by a red dotted line on the chart. This is due to the fact that the initial classes are not balanced and the major class has a weight of about 70%. In other words, by assigning a major class value to any observation, you can get an average of 70% accuracy. The optimal value for both algorithms is 87 principal components, which ensures that 97.5% of Total Variance and get 0.74-0.75 accuracy for the models is maintained.

Fig.3 shows the graphs of the dependence of accuracy score on the total variance portion, derived from the two previous graphs (Fig.1 and Fig.2). 

Fig. 3
![Imgur](https://i.imgur.com/wQyo2lS.png)

One of the reasons for the decrease of the dimensionality is the optimization of the code execution time which is important when using large data and complex algorithms. In our example, the relatively complex algorithm is XG-boost compared to Logistic Regression.

Fig.4 shows a comparison of the speed of both algorithms with the different number of variables in the predictors' matrix. 

Fig.4
![Imgur](https://i.imgur.com/uW8cq49.png)

As you can see from the chart, the running time of both algorithms increases linearly as the dimensionality increases, but the slope angle of the Linear Regression is lower, which is an advantage with a large amount of data. Based on this graph, we can draw another conclusion: the algorithm of decreasing the dimensionality has a higher value for XG-boost because of the rate of decrease in the speed of the algorithm - higher. 

Of course, it is impossible to generalize the above conclusions on any data, but when working with high-dimensional data with a relatively big number of samples such analysis could be useful. And as a conclusion, I can say, that applying PCA to datasets seems to be highly beneficial.








