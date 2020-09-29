+++
draft = false
title = "Clustering NBA players"
weight = 7
image = "img/posts/NBA/Kmeans4 _thumbnail.png"
date = 2020-01-27T13:09:49-05:00
showonlyimage = false
+++

*Finding patterns between NBA compensation and skill-sets*
<!--more-->
***

NBA teams routinely over- or under-pay players for their current performance. Unsupervised machine learning can be used to find patterns or clusters of players to understand which teams are getting the most for individual player performance. We use K-means, hierarchical, and model-based clustering along with other techniques and tools such as principal component analysis, standardizing, scaling, and web-scraping to find these player clusters. 

This post is a brief summary of a more extensive academic project performed with fellow NYU graduate students Andrew Pagtakhan, George Perrett, and Bilal Waheed. The below language and plots are either directly or indirectly pulled from the project. Find the full analysis, code, and data here: [github.com/joemarlo/ML-NBA](https://github.com/joemarlo/ML-NBA)

<br>

### Methods

The analysis explored the 2016-2017 NBA season. We decided to use the 14 features that provided a good balance of offensive (e.g. points, assists) and defensive stats (e.g. rebounds, blocks). This is likely to provide balanced groupings between those who are more offensive and those who are better at defense. A full list of the features can be found in this [document](https://github.com/joemarlo/ML-NBA/blob/master/Analyses/Master_analysis.pdf).

<br>

#### Analysis process

The full clustering analysis followed the below steps. A selection (steps 4-6) are summarized in detail below.

1. Exploratory data analysis
2. Scaling and transformation
    - Features were standardized to a per-game rate
    - Data was scaled for all clustering methods
    - Data was transformed via cube-root for the model-based clustering to meet assumptions of normality
3. Principal component analysis
4. Clustering   
    - Hierarchical  
    - K-Means  
    - Model-based clustering
5. Cluster selection
6. Post-hoc analysis
    - Compensation and performance vs. clusters

<br>

#### Clustering and cluster selection

The results from the principal component analysis showed strong groupings that could be explained with domain knowledge. For example, LeBron James and James Harden are near each other, indicating star players may be grouped together. There are also similarities based on player position. For example, centers/power-forwards such as Anthony Davis and Karl-Anthony Towns are near each other.

Three clustering methods were then tested and assessed on separation and homogeneity: hierarchical (single, centroid, and Ward), K-Means, and model-based clustering. Most methods optimized for four or five clusters as measured by Calinski-Harabasz index and Silhouette width. K-Means provided the greatest balance of separation and homogeneity.

Visualizing the clusters in principal component space, we see clear separation based on apparent skillsets. For example, LeBron James and James Harden (star players) are in one cluster, whereas John Lucas and Danuel House (lower performers) are in the left-most cluster, opposite to the 'star player' cluster on the right.

<p align="center">
<img src="/img/posts/NBA/HCL4.svg" width=100%>
</p>

<p align="center">
<img src="/img/posts/NBA/Kmeans4.svg" width=100%>
</p>

<p align="center">
<img src="/img/posts/NBA/MC5.svg"" width=100%>
</p>

Cluster sizes tends to be imbalanced across methods which may be explained by the underlying inherently imbalanced player skillsets. This imbalance is reflected in univariate density plots where most densities were positively skewed suggesting that there a few players whose statistics significantly exceed those of the average player. It is also reflected in the average statistics by cluster shown below.

<br>

#### K-Means 

K-Means was the optimal solution. Hierarchical and K-Means cross-tab show 87% agreement in cluster membership but separation measures and the above plot reveals K-Means produces greater separation of players based on overall skillsets. The model-based clustering produced a five-cluster solution based on the BIC. Compared to the previous methods, model-based clustering tended to group players by style of play. On the other hand, K-Means and hierarchical appear to cluster on overall skillsets across statistics. This type of clustering suggests emphasis on style of play, e.g. better rebounders/blockers, better passers, scorers, etc.

We validated K-Means clusters by comparing the clusters against advanced statistics. [PER](https://en.wikipedia.org/wiki/Player_efficiency_rating), [VORP](https://en.wikipedia.org/wiki/Value_over_replacement_player), and [RPM](https://www.espn.com/nba/story/_/id/10740818/introducing-real-plus-minus) are advanced statistics commonly used to assess general player performance. None of these statistics were used in the cluster modeling. 

<p align="center">
<img src="/img/posts/NBA/ClusterPer.svg" width=100%>
</p>
  
<br>

### Post-hoc analysis

As shown in the above plot's labels, the four clusters can be relabeled to match their relative skillsets.
  
+ **Subpar Players:** players that average low statistics across the board, leading us to believe that these may be rookies, newly drafted players, or general low output players.
+ **Bench Players:** players that average approximately 25 minutes per game, consistent with non-starters, and have decent output in points and rebounds, indicating that they play a supporting role on their teams.
+ **Role Players:** similar to Bench players, these players average around 25 minutes per game, and double-digit points. However, they particularly have higher average rebounds and PER, which could indicate that these players are 'big-men' or rebounders/key roleplayers on the offensive and defensive ends. For example, players like Andre Drummond, and Greg Monroe aren't high scorers, but can rebound and impact the game in a significant way.
+ **Star Players:** franchise players, the MVPs, All-Stars, and the players that consistently perform exceptionally well (usually on winning teams). They play the most minutes and have the highest output. Examples are LeBron James, Anthony Davis, Steph Curry, etc.

We would expect player salary should be roughly commensurate to their cluster (not withstanding injuries), and we found this is generally true. However, there are a few cases that are not proportional. For example, Chandler Parsons was paid $22M but is considered a low performer and is paid more than star players such as Steph Curry ($12M) and Kawhi Leonard ($18M). In a perfectly just world, the below plot would be a positively sloped line indicating positive correlation between player performance measured by [PER](https://en.wikipedia.org/wiki/Player_efficiency_rating) and salary. Accordingly, cluster membership would increase monotonically left-to-right. This pattern is evident but not perfect.

<p align="center">
<img src="/img/posts/NBA/PERsalary.svg" width=100%>
</p>

The plot below indicates how much a player is over- or under-paid with respect to their cluster average salary. For example, Chandler Parsons was paid $22M vs. the 'Subpar' cluster salary average ($2M), indicating he was overpaid by $20M. It is important to note that there are players who may have been injured so their statistics may not be commensurate to their salaries.

<p align="center">
<img src="/img/posts/NBA/DIFFsalary.svg" width=100%>
</p>

<br>

#### Team compensation and performance vs. clusters

Although a team can have better players on average, there are many variables at play. A player can perform well on average but poor management or coaching can affect a team’s overall performance, e.g. New York Knicks. Interestingly, Golden State Warriors did not have the highest average cluster rating, because their bench is not as strong. This speaks to the strong influence that starter players can have on team performance. 

Teams can play well even if they do not have many all-stars or a strong overall team, e.g. Boston Celtics. Paying more does not necessarily translate into wining more games. For example, the Utah Jazz have the same winning percentage as the Los Angeles Clippers despite a substantial difference in player payroll spending. This could be driven by factors outside of the dataset such as good coaching and team chemistry. It is important to note that items such as injuries could greatly influence win %, even if players have high ratings. Although there is a correlation between overall team salary and win %, it is interesting that average player rating does not necessarily align with overall win %.

<p align="center">
<img src="/img/posts/NBA/WinVSalary.svg" width=100%>
</p>

<br>

#### Final thoughts

The K-means four cluster solution was able to accurately group players by general skill set and productivity which is reflected empirically when statistics such as VORP, RPM, and PER are explored at the cluster level. Our analysis provides interesting insight into the similarities and differences between NBA players in a given season. In general, teams look to have a diverse roster of players with varying skillsets and styles of play. As shown in the PER vs. Salary plot, some players are high producers and effective scorers, but are not highly compensated. This posits that some team managers are prudent at selecting good talent early, when they are under low salary contracts, and developing their players into future stars. 

On the other hand, in the Win % vs. Salary plot, it is clear that high team spending on higher-caliber players does not necessarily translate into higher winning percentages. For example, a team like Boston consisted of mostly average players and spent a relatively lower amount of money than most teams. However, their winning percentage was approximately 65%. 

Our unsupervised clustering methodology was effective in grouping and assessing NBA players based on similar characteristics, skill sets, and play styles. This information may be useful for team managers and coaches to build cost-effective, winning franchises.

<br>

---
*2020 January*  
*Find the code here: [github.com/joemarlo/ML-NBA](https://github.com/joemarlo/ML-NBA)*