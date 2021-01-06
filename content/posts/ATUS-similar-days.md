+++
draft = false
image = "img/posts/ATUS-similar-days/ATUS_similar_days_thumbnail.png"
date = 2020-12-31T10:42:50-05:00
showonlyimage = false
title = "Who spends their day just like you?"
weight = 100
+++

*DRAFT*
<!--more-->
***

Sed ut perspiciatis unde omnis iste natus error sit voluptatem accusantium doloremque laudantium, totam rem aperiam, eaque ipsa quae ab illo inventore veritatis et quasi architecto beatae vitae dicta sunt explicabo. Nemo enim ipsam voluptatem quia voluptas sit aspernatur aut odit aut fugit, sed quia consequuntur magni dolores eos qui ratione voluptatem sequi nesciunt.

Neque porro quisquam est, qui dolorem ipsum quia dolor sit amet, consectetur, adipisci velit, sed quia non numquam eius modi tempora incidunt ut labore et dolore magnam aliquam quaerat voluptatem. Ut enim ad minima veniam, quis nostrum exercitationem ullam corporis suscipit laboriosam, nisi ut aliquid ex ea commodi consequatur? Quis autem vel eum iure reprehenderit qui in ea voluptate velit esse quam nihil molestiae consequatur, vel illum qui dolorem eum fugiat quo voluptas nulla pariatur?


<br>

<div class="corner_ribbon">DRAFT</div>
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
        <option value="Early morning">Early morning (6am)</option>
        <option value="Morning">Morning (8-9am)</option>
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
    <h4>Is this how your day roughly looks? Customize by clicking on each square</h4>
    <div id="container_user_input"></div>
    <br><br>
    <div id='legend'><br></div>
  </div>
  <h4>Now lets find people similar to you</h4>
  <div id='container_string_input' class='container'>
    <input type="button" class="btn" value="Find similar individuals" id="button_update">
  </div>
  <div id="plot_sequence" style="position: static"></div>
  <br>
  <h4>Who are these people?</h4>
  <p>Ut enim ad minima veniam, quis nostrum exercitationem ullam corporis suscipit laboriosam, nisi ut aliquid ex ea commodi consequatur? Quis autem vel eum iure reprehenderit qui in ea voluptate velit esse quam nihil molestiae consequatur, vel illum qui dolorem eum fugiat quo voluptas nulla pariatur? </p>
  <div id="plot_histograms"></div>
</div>

<br>

### How does this work?
In short, a day's time use activities are reduced down to a single string of characters and the [edit distance](https://en.wikipedia.org/wiki/Edit_distance) between the strings is computed. A smaller distance means two given days are more similar.

#### Edit distance methods
These many choices of edit distance metrics. Most are variations of different costs for insertions, deletions, and substitutions. The below four are some of the most common:
- Hamming: only allows substitution  
- Longest common subsequence (LCS): only allows insertions and deletions  
- Dynamic Hamming: only allows substitutions but costs depend on position within the sequence  
- TRATE: costs derived from transition rates   

Each of these have advantages and disadvantages. For this project, dynamic hamming produced the most interesting clusters. It captured both the unique sequencing of the activities and their timing within the day.


#### Clustering
The data needs to condensed down from the original 100,000 days into small dataset that can be quickly loaded with the webpage.

<p align="center">
<img src="/img/posts/ATUS-similar-days/dendrogram.png" width=70%>
</p>

<p align="center">
<img src="/img/posts/ATUS-similar-days/silhouette_width.svg" width=70%>
</p>

<p align="center">
<img src="/img/posts/ATUS-similar-days/state_distributions_by_cluster.png" width=90%>
</p>

<p align="center">
<img src="/img/posts/ATUS-similar-days/modal_sequences.svg" width=70%>
</p>

#### Limitations
- only weekdays  
- not comprehensive of all day types  
- list definitions of activities  
Sources: American Time Use Survey


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
