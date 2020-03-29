---
layout: post
title:      "Using correlation for splitting public companies into clusters"
date:       2020-03-29 11:57:04 +0000
permalink:  using_correlation_for_splitting_public_companies_into_clusters
---


**preconditions**


Hi!

The topic of this post is related to one of the stages of the Capstone Project, which is being developed as part of my training at Flatiron School. The original theme of this project is using ML algorithms to predict stock prices. As part of my data source search, I found API services that allow me to download open data on public company quotes. As a result, I chose the option to use close daily prices and downloaded data on 6 900 thousand issuers. 

One of the key questions of the project is comparing the predictive abilities of two types of models: ARIMA and Recurrent Neural Networks. It is hypothetically possible that one time series can learn better from the first model and the other from the second. Therefore, I realized that the choice of one specific time series is not the best idea in the context of such a generalized comparison. On the other hand, it is an unrealistic task to individually select for each company and to train a separate model, taking into account the fact that there are 6,900 companies under consideration. 

Thus, I came to the conclusion that it is necessary to somehow divide the entire space of the companies into some groups or clusters, to define for each cluster a certain centre, and to train already at these centres for further comparison of the two algorithms.

On this blog, I wanted to share one of the ways I called this separation based on correlation. First of all, I want to make a caveat that this idea is not some standard ML method and can be challenged or improved by ML experts. It is possible that in various more complex modifications such methods are already used in practice, which I may not know about.  Also, code that provides functions for their implementation may not be the most efficient and may well be optimized in terms of speed or syntactic style. So, I will be glad to receive any constructive criticism or suggestions. The source code of the project can be found at this [link](https://github.com/NuranKZ/CapstoneProject)

**algorithm description**

Below I will try to describe the main principles and results without going into details of the code.
The main idea is to form such groups or clusters where companies with positive and good cross-correlation are collected. 

Algorithm (simplified description):
1.    A correlation matrix (CM) is generated for the time series under consideration.
2.    For each company in CM a list of companies whose correlation value is higher than the given threshold is generated.
3.    A special dictionary is formed, where the keys are the companies for which the corresponding non-empty lists of correlated companies are found, and the dictionary values are these lists. This dictionary is sorted in descending order, the sorting parameter is the number of correlated companies.
4.    The formed clusters are formed by combining the keys and corresponding lists.

When implementing this option, there may be many clusters with a very wide range of companies in the cluster. The main problem is a very high degree of intersection of clusters. So, for example, in my project, such a launch on the correlation matrix of dimensionality 6890x6890 and threshold = 0.65 gave the following results:
- Total number of clusters: 2,293
- Biggest cluster size: 594
- Smallest cluster size: 2
- Obtained clusters cover 33% of all companies.

As you can see all the companies positively related to each other have concentrated within 33% of the companies, and many small clusters have generated. I should also note that almost all clusters are strongly crossed. And the number of clusters actually corresponds to the number of key companies. That is, this mechanism needs a number of adjustments. Therefore, the algorithm was extended by the mechanism of deleting duplicates, where priority was given to clusters with the higher power. In other words, all companies that were present in the list of correlated companies in the previous cluster were removed from the current cluster. I will immediately note that here I do not take into account specific values of correlation and the final distribution of power clusters - not even. After such a cleanup, the number of clusters was reduced to 634. But at the same time, the share of ultra-small clusters still accounts for a large number.

The next 2 steps of the algorithm are optional and are designed to cut off these mini-clusters:
- Limiting the maximum number of clusters (leaves the largest clusters in the sorted dictionary)
- Limiting the minimum number of companies in the cluster
After all these purges, I received 14 clusters with reduced number of companies.

The final step of the algorithm is to define the centre of the cluster.
In a simplified version, I could, of course, take some average or median, which is essentially a synthetic time series. However, here I used the graphical representation of a cluster, where the nodes are the cluster companies. Each cluster contains at least one pair that has a correlation above the threshold value. Between such pairs, a graph edge is formed. For this implementation, each cluster was converted into a dictionary with the keys corresponding to the companies of the cluster and the values presented by lists of related companies.  This step is similar to the first steps of the algorithm but already implemented within a single cluster. It should be noted that after such a step, there were some crossovers between clusters, but I did not clean them up. The total coverage of the companies is 1513.

Unlike the initial steps, in this case for any company in the cluster is guaranteed to have at least one related company, which means that the column will not have isolated nodes. The centre defined in this column has the largest number of ribs, in other words, it is the company that has the highest number of connections with other companies and becomes the most representative of the cluster as a whole.

Below is an example of a small cluster from the Project as a graph. The centre is highlighted in red.

![Imgur](https://i.imgur.com/V2pJgUN.png?1)

In general, finding a centre using the construction of the graph is not necessary, as such a centre can be found using more simple functions. However, the graph gives a good visual representation of the cluster with a relatively small number of nodes. Also, if this approach is extended and improved, you can set the edges of some weights that reflect already specific values of correlation between nodes and make the assessment of the centre more robust. 


**testing**

To verify the effectiveness of the grouping of companies, the methods under consideration require some alternative grouping, as well as some metrics by which the two approaches will be compared.  
As an alternative approach to clustering, I used a grouping of companies on a static basis - Sector, as a result, a comparable number of sectors was formed - 12, and as the center of the sector I used the median value of quotations at each point of time.  

Let's have a look at what time series of some centers look like for both approaches. Here time series are presented on the basis of Price Return instead of Absolute Price in order to compare different companies.  

![Imgur](https://i.imgur.com/BN8roab.png?1)

![Imgur](https://i.imgur.com/L9Pwryz.png?1)

The green zone around the time series forms an inter-apartment dimension (q25-q75), the more homogeneous is the cluster representatives. As can be seen, in general, the green zone is already in the corr-cluster. Visualization options for all clusters can be viewed on the repositories in the file `S4_EDA_Clustering.ipynb`.

Next, I have trained models in all centres using the LSTM algorithm and performed validation on the test sample. R2 calculation was performed for a 1-step-ahead forecast.  
For testing, I chose 3 centres from sector groups and 3 centres from correlation clusters:
-	Sector composites:
   - Basic Materials, R2 = 0.74
   - Utilities, R2 = 0.81
   - Financial, R2 = 0.76
- Correlated cluster centers:
   - AOA, R2 = 0.85
   - FXO, R2 = 0.87
   - FXL, R2 = 0.90

For each centre, I randomly selected 25 companies from the corresponding cluster and generated a forecast for the model trained in the centre of the cluster.
The result is shown in the chart below.

![Imgur](https://i.imgur.com/K9z03sd.png?1)

As can be seen above, in some cases, the models trained on centres can provide acceptable quality of metrics even for companies in the cluster of this composite. Of course, the results above are only a small simulation with random selection of companies, so this conclusion is evidence of the quality of the model itself, but only reflects some coherence of companies in the 1 cluster.

We can notice that the average value of R2 for companies in correlation clusters is closer to the R2 centers of corr-clusters than for clusters formed by sectors. This conclusion is not unambiguous, because I chose 6 centers relatively randomly, but in general, we can say that the idea of merging companies into clusters based on correlation dependence can be used and bring some benefit.







