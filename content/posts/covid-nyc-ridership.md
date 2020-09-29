+++
draft = false
image = "img/posts/covid-nyc-ridership/covid_ridership_thumbnail.png"
date = 2020-09-26T17:40:21-04:00
showonlyimage = false
title = "Disparities in the fall of NYC subway ridership"
weight = 1
+++

*Mapping the decline of subway ridership in the Covid era*
<!--more-->
***

<!-- script and CSS for map -->
<script src='https://api.mapbox.com/mapbox-gl-js/v1.12.0/mapbox-gl.js'></script>
<link href='https://api.mapbox.com/mapbox-gl-js/v1.12.0/mapbox-gl.css' rel='stylesheet' />

Ridership on New York City's subway system has declined significantly amid the Coronavirus pandemic. The Metropolitan Transportation Authority (MTA), which runs the subway, is in the [midst of a fiscal crisis](https://www.nytimes.com/2020/07/21/nyregion/mta-subway-financial-cuts.html) — reportedly facing a $16 billion deficit due to the decline.

It's easy to see the severity of the situation after visualizing the daily ridership over 2020. Within the course of one month, which included a New York statewide stay-at-home order, the MTA witnessed an unprecedented fall in straphangers. But is the decline in ridership spread equally  across NYC's diverse residents and neighborhoods?

<p align="center">
<img src="/img/posts/covid-nyc-ridership/ridership_timeseries.svg" width=90%>
</p>

<br>

The fall can be easily quantified by taking the difference between mean ridership before the precipice and mean ridership once it flattens out. This results in an 88% decline in system-wide ridership.

<p align="center">
<img src="/img/posts/covid-nyc-ridership/ridership_timeseries_bars.svg" width=90%>
</p>

<br>

### Neighborhood differences

Extending this basic methodology to each subway station reveals a distinct pattern playing out across the boroughs. Manhattan — home to the wealthy and the transient — saw the largest declines in ridership while the outer-boroughs — especially the Bronx — still saw large but relatively the smallest declines. A [Voronoi diagram](https://en.wikipedia.org/wiki/Voronoi_diagram) (below left) is overlaid on the map to estimate the encompassing neighborhoods around a given subway station. This provides a reasonable approximation of the persons who are likely trafficking the station.

To dig deeper into understanding who these persons are, projecting the points on to the [Census Public Use Microdata Areas](https://www.census.gov/programs-surveys/geography/guidance/geo-areas/pumas.html) (PUMA, below right) allows us to tie the ridership to demographic variables tracked by the Census via the [American Community Survey](https://www.census.gov/programs-surveys/acs) (ACS) such as income, job industry, etc.

<p align="center">
<div class="row">
 <div class="column">
  <img src="/img/posts/covid-nyc-ridership/change_in_ridership.svg"  style="width:100%">
  <p align="right" style="font-size:70%;">
  <i>
  <br>
  Regions mapped by proximity to subway station
  </i>
  </p>
 </div>
 <div class="column">
    <img src="/img/posts/covid-nyc-ridership/change_in_ridership_by_PUMA.svg" style="width:100%">
    <p align="right" style="font-size:70%;">
    <i>
    <br>
    Regions mapped by Census PUMA code
    </i>
    </p>
 </div>
</div>
</p>

The first question to be addressed: are smaller decreases in ridership explained by essential worker status? Essential worker status can be approximated based on the industry residents labor in. Aligning these industries to the [Delaware essential industry list](https://coronavirus.delaware.gov/wp-content/uploads/sites/177/2020/04/DE-Industry-List-4.21.pdf) (one of the few discrete lists available), the proportion of essential workers can be estimated (below left). It's difficult to verify the accuracy but it is in line with the US-wide estimate from the Economic Policy Institute which claims [55 million American workers are essential](https://www.epi.org/blog/who-are-essential-workers-a-comprehensive-look-at-their-wages-demographics-and-unionization-rates/).

Second, how does ridership correlate with income? Below right. The [New York Times found](https://www.nytimes.com/interactive/2020/05/15/upshot/who-left-new-york-coronavirus.html) that income was a strong predictor of the emptying of a neighborhood during Covid. Their analysis was based on smartphone location data of residents and excluded commuters.

The income story is right in line with expectations with Manhattan and the northern parts of Brooklyn clearly representing the wealthiest neighborhoods. The essential worker map is a mixed bag; parts of Brooklyn and Manhattan contain more essential workers than expected. And some contain less than expected such as East Williamsburg and Bushwick in Brooklyn and Manhattan near Columbia University. However, the Bronx and south Brooklyn contain the most which aligns well with the ridership map.

<p align="center">
<div class="row">
 <div class="column">
  <img src="/img/posts/covid-nyc-ridership/essential_worker.svg"  style="width:100%">
 </div>
 <div class="column">
    <img src="/img/posts/covid-nyc-ridership/income.svg" style="width:100%">
 </div>
</div>
</p>

<br>

### Trends

Just as the New York Times found on neighborhood activity, there is a clear trend between a neighborhood's household income and the decline in subway ridership at the surrounding stations using a simple regression with log transformed income (below left). However, this relationship becomes more nuanced once you control for the borough (below right) indicating that income alone is not enough to accurately predict ridership. Notably, the Bronx and Queens have different levels of ridership change at the similar income levels.

<p align="center">
<div class="row">
 <div class="column">
  <img src="/img/posts/covid-nyc-ridership/change_in_ridership_vs_income.svg"  style="width:100%">
 </div>
 <div class="column">
    <img src="/img/posts/covid-nyc-ridership/change_in_ridership_vs_income_borough.svg" style="width:100%">
 </div>
</div>
</p>

<br>

Unlike income, essential worker status does not appear to have a strong correlation with ridership change across the city (below left). After controlling for borough, the Bronx and Brooklyn have clear positive relationships and Manhattan clear negative (below right). Manhattan is a curious case, and I suspect it has something to do with non-resident ridership.

<p align="center">
<div class="row">
 <div class="column">
  <img src="/img/posts/covid-nyc-ridership/essential_vs_ridership.svg"  style="width:100%">
 </div>
 <div class="column">
    <img src="/img/posts/covid-nyc-ridership/essential_vs_ridership_borough.svg" style="width:100%">
 </div>
</div>
</p>

<br>

A linear model fitted with all three variables — essential worker status, household income, and borough — tells a similar story as the plots above. Controlling for household income and borough, a 1 percentage point increase in essential worker status is associated with a 0.4 percentage point increase in subway ridership. Whereas controlling for essential worker status and borough, every 10% increase in household income is associated with a 0.8 percentage point decline in ridership.

<table class="table table-hover table-responsive" style="margin-left: auto; margin-right: auto;">
 <thead>
  <tr>
   <th style="text-align:left;"> Term </th>
   <th style="text-align:right;"> Estimate </th>
   <th style="text-align:right;"> Standard error </th>
   <th style="text-align:right;"> p value </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> Intercept </td>
   <td style="text-align:right;"> -0.02 </td>
   <td style="text-align:right;"> 0.16 </td>
   <td style="text-align:right;"> 0.90 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Essential worker status </td>
   <td style="text-align:right;"> 0.43 </td>
   <td style="text-align:right;"> 0.20 </td>
   <td style="text-align:right;"> 0.04 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Household income (log) </td>
   <td style="text-align:right;"> -0.08 </td>
   <td style="text-align:right;"> 0.01 </td>
   <td style="text-align:right;"> 0.00 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Brooklyn </td>
   <td style="text-align:right;"> -0.03 </td>
   <td style="text-align:right;"> 0.01 </td>
   <td style="text-align:right;"> 0.05 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Manhattan </td>
   <td style="text-align:right;"> -0.04 </td>
   <td style="text-align:right;"> 0.02 </td>
   <td style="text-align:right;"> 0.03 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Queens </td>
   <td style="text-align:right;"> -0.02 </td>
   <td style="text-align:right;"> 0.02 </td>
   <td style="text-align:right;"> 0.15 </td>
  </tr>
</tbody>
</table>

> Formally, the coefficients can be interpreted as:
> - Controlling for household income and borough, a group of neighborhoods that had a 100 percentage point increase in essential worker status than another group would be expected have to a change in ridership that is 43 percentage points higher.
> - Controlling for essential worker status and borough, for a 10% increase in household income, the difference in expected mean ridership change between two groups of neighborhoods will be -0.08 * log(1.10) = -0.77 percentage points.

<br>

### Pulling it all together

There are disparities in the decline in subway ridership across NYC’s diverse neighborhoods. Some of which can be statistically explained by essential worker status and household income, however it is important not to interpret these causally. The above model does not make a causal claim and may not even capture all the relevant variables. For example, if someone's goal is to conflate ridership with residents' adherence to social distancing, the model does not account for the desertion of Manhattan by the wealthy or the impact of New Jersey and Connecticut commuters.

Explore the data yourself using the interactive map below. Hover over your neighborhood to see how you compare.

<div id='map' style='width: 100%; min-height: 600px'>
<div class='map-overlay' id='features'><h4>Neighborhood attributes</h4><div id='pd'><p>Hover over an area to get attributes!</p></div></div>
<div class='map-overlay' id='legend'>
  <strong>Change in ridership</strong>
  <nav class='legend clearfix'>
    <span style='background:#760cac;'></span>
    <span style='background:#09a59d;'></span>
    <span style='background:#f5d905;'></span>
    <label>-95%</label>
    <label>-85%</label>
    <label>-75%</label>
</div>
</div>


<br>
<br>
<br>

### Limitations

There are a handful of limitations to this data and analysis to be aware of:
- The ridership data only captures riders and does not distinguish between residents and non-residents. Neighborhoods such as lower Manhattan and Midtown that see a substantial amount of non-resident commuters enter the system would be expected to see large declines given that these non-residents are no longer trafficking the stations. This could potentially be remedied by only examining station entries in the morning, however straphangers transferring from rail (NJT, LIRR, Metro-North) would still be captured and nighttime essential workers would be excluded.
- These data are not evaluated on causality merits. While there may be strong correlations between ridership and income, it does not suggest that one directly leads to the other. Similarly, using ridership as a proxy for "social distancing", or some similar societal conformity measure, is fraught mostly due to the non-resident point above.
- The subway ridership data is messy. Calculating ridership and matching to station location has room for error. Some stations may not be perfectly mapped.
- Essential worker status is estimated from the ACS respondent's [NAICS](https://www.census.gov/eos/www/naics/) industry and the published Delaware list of essential industries. These may not be in full agreement with New York State's guidelines.  


<br>

---
*2020 September*  
*Find the code here: [github.com/joemarlo/NYC-data/](https://github.com/joemarlo/NYC-data/tree/master/Analyses/COVID-neighborhoods)*

<!-- Load Mapbox map -->
<script src="/d3/covid-nyc-ridership/covid_ridership.js"></script>

<!-- Load custom CSS -->
<link rel="stylesheet" type="text/css" href="/posts-css/covid-nyc-ridership.css">
