+++
draft = false
image = "img/posts/ATUS-similar-days/ATUS_similar_days_thumbnail.png"
date = 2020-12-31T10:42:50-05:00
showonlyimage = false
title = "Who spends their day just like you?"
weight = 1
+++

*Finding similar sequences of categorical data*
<!--more-->
***

How people their time is fascinating. It's the most precious resource and consequently our time-use reflect our priorities. Finding people who spend their days similar to our own can be enlightening — both because it's intrinsically interesting and to understand ourselves through the lens of others.

A single day's time-use can be reduced to a string of activities. Here, every hour is represented by a rectangle and the activity by a color.

<div id="container_example"></div><br><br>
<div id='legend_example'><br></div>

This prototypical day tells us the person works a roughly 9-5 day and spends their free time relaxing or socializing. They most likely are not in school and do not have children requiring care. As a result it can be approximately inferred they have a steady full time job, are college educated, and make above median income.

Answer the questions below to see who spends their day similar to you. The question-series yields common weekday sequences that may match your own; however the sequences are entirely editable to fit your exact day.


<br>

<div id='visualization'>
  <div id="container_twenty_questions">
    <h3>How do you usually spend your weekdays?</h3>
    <h4>Do you work?</h4>
    <select id='Q1' class='form-control form-control-sm container_twenty_questions_dropdown'>
      <option value="Yes">Yes</option>
      <option value="No">No</option>
    </select>
    <div id='container_Q2'>
    <h4>When do you start work?</h4>
      <select id='Q2' class='form-control form-control-sm container_twenty_questions_dropdown'>
        <option value="Early morning">Early morning (before 8am)</option>
        <option value="Morning">Morning (after 8am)</option>
        <option value="Afternoon">Afternoon</option>
        <option value="Evening">Evening</option>
      </select>
    </div>
    <div id='container_Q3'>
    <h4>Are you a student?</h4>
      <select id='Q3' class='form-control form-control-sm container_twenty_questions_dropdown'>
        <option value="Yes">Yes</option>
        <option value="No">No</option>
      </select>
    </div>
    <div id='container_Q4'>
      <h4>Hmmm, how do you usually spend your day?</h4>
      <select id='Q4' class='form-control form-control-sm container_twenty_questions_dropdown' >
        <option value="Socializing, relaxation, leisure">Socializing, relaxation, or leisure</option>
        <option value="Household activities">Household activities</option>
        <option value="Care for others">Caring for others</option>
        <option value="Volunteering">Volunteering</option>
        <!--<option value="Religious">Religious or spiritual activities</option>-->
      </select>
    </div>
    <br><br>
  </div>
  <div>
    <h4>Is this how your day roughly looks? Customize by clicking each square</h4>
    <div id="container_user_input"></div>
    <br><br>
    <div id='legend'><br></div>
  </div>
  <h4>Now let's find people similar to you</h4>
  <div id='container_string_input' class='container'>
    <input type="button" class="btn" value="Find similar individuals" id="button_update">
  </div>
  <div id="plot_sequence"></div>
  <br>
  <h4>Who are these people?</h4>
  <p>The below plots reflect the above 25 individuals measured on six attributes. The heights of the bars represent the frequency for that category.</p>
  <div id="plot_histograms"></div>
</div>

<hr><br>


### How does this work?
In short, a day's time use activities are reduced to a single string of characters and the [edit distance](https://en.wikipedia.org/wiki/Edit_distance) between the strings is computed. More similar days have smaller distances.

#### Edit distance methods
There are many choices of edit distance method. Most rely on counting the number of insertions, deletions, and substitutions to transform one string to the other — with each method stipulating different costs for each operation. The below four are some of the most common:
- Hamming: only allows substitution  
- Longest common sub-sequence (LCS): only allows insertions and deletions  
- Dynamic Hamming: only allows substitutions but costs depend on position within the sequence  
- TRATE: costs derived from transition rates   

Each of these have advantages and disadvantages. For this project, dynamic hamming produced the most interesting clusters. It captured both the unique sequencing of the activities and their timing within the day.


#### Clustering
The data comes from the [American Time Use Survey](https://www.bls.gov/tus/) covering over 200,000 interviews between 2003 to 2018. The original 100,000 weekdays of data needs to be condensed down into a dataset small enough to be quickly loaded by the browser.

Hierarchical clustering based on the edit distance is used to discover the seven most interesting clusters. While a smaller number of clusters results in the best cohesion-to-separation (measured via [silhouette width](https://en.wikipedia.org/wiki/Silhouette_(clustering))), seven clusters provides the best balance between the numerical measure and subjective interpretations of cluster distinction.

The below dendrogram shows distances between the observations and how they coalesce at varying cluster k values. The colors represent the seven clusters.

<p align="center">
<img src="/img/posts/ATUS-similar-days/dendrogram.png" width=80%>
</p>

Each cluster's individual sequences reveal what makes the clusters differentiable. Cluster 1 consists of mostly socialites/relaxers; cluster 2 night workers; cluster 3 household activities; cluster 4 students; cluster 5 morning workers; cluster 6 early-morning workers; and cluster 7 evening workers.

<p align="center">
<img src="/img/posts/ATUS-similar-days/state_distributions_by_cluster.png" width=95%>
</p>

The clusters are reduced down to their modal sequence. These modes are used as the first step to classify the user's input. The user input is first matched to the best cluster (based on edit distance to the modal sequence) and then matched to the top 25 sequences of 200 potential sequences that are a priori sampled from the clusters.

<p align="center">
<img src="/img/posts/ATUS-similar-days/modal_sequences.svg" width=80%>
</p>

<br>

#### Survey information
- Contains nationally representative estimates of how, where, and with whom Americans spend their time
- Covers 200,000 interviews conducted from 2003 to 2018
- Only non-holiday weekday data is included here resulting in ~100,000 interviews
- Can be linked to [Current Population Survey](https://www.census.gov/programs-surveys/cps.html) for detailed household demographic information
- Sponsored by the Bureau of Labor Statistics and conducted by the Census Bureau

<br>

#### Limitations
- The American Time Use Survey measures primary activity so some activities are not accurately captured. For example, listening to music is rarely recorded because typically it is a secondary task. If I'm listening to music, I'm probably also doing housework or cooking.
- The data is self reported so [standard biases apply](https://en.wikipedia.org/wiki/Response_bias)
- The clustering process aims to provide distinct clusters but they may not be representative of all day types. Combined with the two-step algorithm specified above, some day types may not be matched well. Notably, the use of the [Levenshtein distance](https://en.wikipedia.org/wiki/Levenshtein_distance) to calculate distances between the user sequence and the modal sequences result in cluster 7 (evening workers) often being skipped over. Once time allows, I may write my own dynamic Hamming distance function in Javascript; one is not currently available.
- The above distributions are not representative of *all* people that do those activities, just the top 25 people most similar to you
- The raw time-use data details each minute of the respondent’s day by mapping it to a list of 465 activities. I aggregated these 465 activities into 15 activities based on the Bureau of Labor Statistics’ (BLS) hierarchical definitions and my judgment. Each 30 minutes of the day is categorized as the modal activity within each respective time frame. This results in some loss of information but severely reduces the distance matrix size.

<br>

#### Software thanks
- [d3.js](https://d3js.org/) and JavaScript for interactive visualizations
- [R](https://cran.r-project.org/) for analysis
  - [TraMineR](http://traminer.unige.ch/) for edit distance calculations
  - [fastcluster](https://cran.r-project.org/web/packages/fastcluster/index.html) and [NbClust](https://cran.r-project.org/web/packages/NbClust/index.html) for clustering
  - [tidyverse](https://www.tidyverse.org/) for data munging and [ggplot2](https://ggplot2.tidyverse.org/) for static visualizations

<br>

---
*2021 January*  
*Find the code here: [github.com/joemarlo/ATUS-similarity](https://github.com/joemarlo/ATUS-similarity)*

<link rel="stylesheet" type="text/css" href="/posts-css/ATUS-similarity.css">
<script src="https://d3js.org/d3.v5.js"></script>
<script src="https://d3js.org/d3-scale-chromatic.v1.min.js"></script>
<script src="https://cdn.jsdelivr.net/jstat/latest/jstat.min.js"></script>
<script src="//unpkg.com/string-similarity/umd/string-similarity.min.js"></script>
<script src="https://code.jquery.com/jquery-3.3.1.slim.min.js" integrity="sha384-q8i/X+965DzO0rT7abK41JStQIAqVgRVzpbzo5smXKp4YfRvH+8abtTE1Pi6jizo" crossorigin="anonymous"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.14.7/umd/popper.min.js" integrity="sha384-UO2eT0CpHqdSJQ6hJty5KVphtPhzWj9WO1clHTMGa3JDZwrnQq4sF86dIHNDz0W1" crossorigin="anonymous"></script>
<script src="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/js/bootstrap.min.js" integrity="sha384-JjSmVgyd0p3pXB1rRibZUAYoIIy6OrQ6VrjIEaFf/nJGzIxFDsf4x0xIM+B07jRM" crossorigin="anonymous"></script>
<script src="/d3/ATUS_similarity/UI.js"></script>
<script src="/d3/ATUS_similarity/levenshtein.js"></script>
<script src="/d3/ATUS_similarity/functions.js"></script>
<script src="/d3/ATUS_similarity/plotHistograms.js"></script>
<script src="/d3/ATUS_similarity/plotBars.js"></script>
<script src="/d3/ATUS_similarity/plotSequence.js"></script>
<script src="/d3/ATUS_similarity/legend.js"></script>
<script src="/d3/ATUS_similarity/d3Functions.js"></script>
<script src="/d3/ATUS_similarity/runD3.js"></script>
