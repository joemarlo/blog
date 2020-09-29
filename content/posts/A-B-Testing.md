+++
draft = false
image = "img/posts/A-B-Testing/AB_thumbnail.png"
date = 2020-04-27T17:22:08-05:00
showonlyimage = false
title = "Common pitfalls of A/B testing"
weight = 4
+++

*Why you could be doing it wrong*
<!--more-->
***



A/B testing has become a buzz-word across the tech landscape. The method allows decision makers to see which version of a product performs better and enables leaders to make evidence-informed decisions. Companies, political campaigns, and market research firms often utilize these tests to improve website design, products, and marketing strategies. 

You’re probably already doing A/B testing, but you could be doing it wrong. There are four common errors that undermine the results of your test and lead to poor decisions:

* <a href="#optional-stopping">**Optional stopping**</a>: running the same test until you see a positive result
* <a href="#the-multiple-comparison-problem">**Multiple comparisons**</a>: if there is a negative result, trying a different outcome
* <a href="#insignificant-significant-results">**Insignificant “significant” results**</a>: disregarding effect size
* <a href="#randomization-is-not-always-enough">**Randomization is not always enough**</a>: ignoring interactions in your data

<br>

## Optional stopping

One choice during the A/B process is deciding when data collection should stop and testing should begin. Making quick decisions is valuable, and it is tempting to frequently test the results as data comes in. For instance, you could stop data collection after every 100 users and conduct your analysis; if there is no significant finding, you resume data collection and repeat the process.

This is a problem. Following A/B test theory, there is a 5% chance that a positive result is a false positive. Increasing the number of stops increases this chance of a false positive.

To illustrate this problem, let’s conduct a series of simulations with a varying number of stops. The data were simulated such that there is no true difference between version A and version B, so any positive result is a false positive:

* Results show that stopping five times increases the false positive rate to about 15%, whereas stopping as many as 20 times increases the false positive rate to about 25%. 
* Frequent stopping can seem like a way to deliver quick results, but it drastically increases the risk of reaching a false conclusion. 


<br>

<p align="center">
<img src="/img/posts/A-B-Testing/optional_stops.svg" width=90%>
</p>

<br>

## The multiple comparison problem

A single A/B test cycle can be used to answer many questions. For example:

* Was there a difference in time spent between version A and version B?
* How about a difference in the number of clicks or likes? 

<br>

Testing multiple outcomes allows you to get the most out of a single cycle, but careful considerations need to be taken to avoid drastically increasing the false positive rate. This risk increases with each additional outcome and without implementing the correct adjustment, the probability of coming to a false conclusion substantially rises. 


> To calculate the increase in risk of a false positive result, there is a simple formula based on the number of hypotheses you are testing *h* at a significance level *alpha* (usually 0.05):
> * Probability of at least one false positive = 1 - (1 - *alpha*)<sup>*h*</sup>
> * Example: 5 tests = 1 - (1 - 0.05)<sup>5</sup> = 23% chance of a false positive

The risk of multiple comparisons can be controlled by introducing the [Bonferroni correction](https://www.stat.berkeley.edu/~mgoldman/Section0402.pdf), which accounts for the additional false positive risk introduced with each additional outcome. 

To illustrate the implications of uncontrolled multiple comparison, let's conduct another series of simulations, but this time replace varying stops with varying numbers of comparisons. Like before, there is no difference between version A and version B, and any positive result is a false positive. 


The plot below shows the increase in false positives as the number of comparisons increases:

* With as few as five comparisons, the risk of observing at least one false positive has increased from 5% to 20%
* However, when the appropriate correction is implemented, the false positive rate is held constant at 5%

<br>

<p align="center">
<img src="/img/posts/A-B-Testing/multiple_comparisons.svg" width=90%>
</p>

<br>

## Insignificant “significant” results

Let’s say you ran your A/B test and achieved statistically significant results. You may think your work is finished, but it’s important to consider the meaning of the effect size that you’re measuring, and its implications.

The below examples visualize different effect sizes that all have statistically significant results (p-values less than 0.05). You can clearly see that although all effect sizes are statistically significant, they each can have different meanings (a 10% change could have a much larger impact on your company than 1%).

<p align="center">
<img src="/img/posts/A-B-Testing/effect_comp.svg" width=90%>
</p>

<br>

Now, if the true difference between both versions was one second spent on the website, is this meaningful enough to your team? Will it translate into higher revenue that can justify rolling out this new version? Sometimes implementing a new version of a product can be costly, so it’s imperative to review whether or not it’s worth it. You should think not only of statistical significance, but the effect size.  

<br>

#### Large sample, small effect

Incredibly small effects reach statistical significance as sample size grows. If you usually work with large sample sizes, a statistically significant result many not necessarily signal a meaningful change.

<p align="center">
<img src="/img/posts/A-B-Testing/effect_size.svg" width=90%>
</p>



<br>

## Pulling it together

Juggling all these conditions can become overwhelming. As you might expect they compound on each other and quickly turn into a combinatorics nightmare. If you are performing multiple stops while also comparing more than one distribution it is likely that you may find a false positive. Additionally, the probability of finding a true effect doesn't scale linearly and behaves differently at different sample sizes and variances.

You can estimate the effects of all these conditions using <a style="font-size:1.1em;" href="https://jmarlo.shinyapps.io/A-B-tool/" target="_blank">**this tool**</a>. The tool shows the *probability of finding at least one effect* given the conditions. Set the effect size to zero and find the false positive rate. Increase the effect size and variance but reduce the sample size and see how infrequently you may be capturing a real effect.

<!-- this is the Shiny app
<div class="iframe-container"> 
  <iframe class="resp-iframe" src="https://jmarlo.shinyapps.io/A-B-tool/" allowfullscreen></iframe>
</div>
-->

<!-- this is the start of the d3 -->
<!-- Sliders in a grid -->
<!--
<div class="grid-container">
  
  <div class="slidecontainer">
    <input type="range" list="tickmarks_stops" min="1" max="20" value="1" step="1" class="slider" id="slider_stops">
    <div class="slidetext" <p>Number of stops: <span id="value_stops"></span></p></div>
  </div>
  
  <div class="slidecontainer">
    <input type="range" min="1" max="20" value="1" step="1" class="slider" id="slider_comparisons">
    <div class="slidetext" <p>Number of comparisons: <span id="value_comparisons"></span></p></div>
  </div>
  
  <div class="slidecontainer">
    <input type="range" min="0" max="0.10" value="0.02" step="0.01" class="slider" id="slider_effectsize">
    <div class="slidetext" <p>Effect size: <span id="value_effectsize"></span></p></div>
  </div>
  
  <div class="slidecontainer">
    <input type="range" min="1000" max="10000" value="1000" step="1000" class="slider" id="slider_samplesize">
    <div class="slidetext"<p>Sample size: <span id="value_samplesize"></span></p></div>
  </div>
  
</div>

<!-- Results -->
<!--
<p><h4>Probability of finding at least one effect: <span id="expected_probability"></span></h4></p>

<br>

<!-- d3 density plots -->
<!--
<div <div id="main_plot" style="position: static" align="center"></div>
<p align="right" style="font-size:60%;">
<i>
Assumptions: 
<br>
- Standard deviation of distributions is X
<br>
- Results estimated by X simulations
</i>
</p>
<br>

-->


<br>

## Randomization is not always enough

The power of randomization is that it allows you to make causal conclusions. However, randomization is not enough to uncover hidden patterns within your test. 

For example, imagine you were a digital news organization interested in comparing the effects of how changing your website impacts how many minutes users spend reading site content. You know that some readers use their computers while others primarily use their phones to visit the site. 

You conduct your A/B test and use randomization to ensure an equal number of computer and mobile users received version A and version B. If you stopped here and conducted your analysis, you would conclude that there is no difference between version A and version B of the site. 

This simple A/B test does not account for the possibility that the effect of version A and version B may differ between computer and mobile users

<p align="center">
<img src="/img/posts/A-B-Testing/no_interaction.svg" width=90%>
</p>

When you examine the potential interaction between mediums (either computer or mobile), important differences emerge and a real difference becomes apparent. Among computer users, version B decreases time spent on the site, however, the same version increases time spent for users primarily interacting with the site by their phones. Without carefully exploring your data, underlying patterns can be lost.

<p align="center">
<img src="/img/posts/A-B-Testing/interaction.svg" width=90%>
</p>

<br>

## Final thoughts

A/B testing is a powerful tool that is frequently praised for its simplicity and ease of analysis. However, misguided conclusions can be made when optional stopping, multiple comparisons, effect size, and possible interactions are ignored. 

When unaccounted for: 

* Frequently checking your data mid experiment (optional stoppage) and conducting multiple comparisons without correction drastically increases the rate of false positives
* Statistical significance and meaningful difference are not the same
* Randomization alone is not enough to uncover hidden interactions within A/B tests

Executing your experiment properly requires being mindful of these four common errors. If you are not, the results of your A/B test may be invalid and lead to poor decision-making.


<br>

---
*2020 May*  
*Find the code here: [github.com/joemarlo/A-B-Testing-with-d3](https://github.com/joemarlo/A-B-Testing-with-d3)*


<!-- script for slider text --> 
<script src="/d3/A-B-Testing/sliders.js"></script>

<!-- Load d3.js -->
<script src="https://d3js.org/d3.v4.min.js"></script>
<script src="https://unpkg.com/d3-simple-slider"></script>

<!-- Load d3 plots -->
<script src="/d3/A-B-Testing/AB_testing.js"></script>

<!-- Load custom CSS -->
<link rel="stylesheet" type="text/css" href="/posts-css/A-B-Testing.css">
