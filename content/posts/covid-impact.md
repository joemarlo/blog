+++
draft = false
image = "img/posts/covid-impact/COVID_thumbnail.png"
date = 2020-03-31T10:04:35-05:00
showonlyimage = false
title = "Enumerating the economic and social shifts of March 2020"
weight = 6
+++

*The ex-health impact of Covid-19*
<!--more-->
***

I'm not an epidemiologist so I'll save you a poorly built model attempting to predict the future of the Covid-19. In lieu, I'm spending my isolation time tracking down various economic and social data illustrating the vast shifts and adjustments in our lives recently. Below is a sequence of time-series plots highlighting the non-health-related impact of Covid-19. 

<br>


<!-- Plot of subway ridership and citibke -->
#### Daily NYC subway ridership drops significantly in March and Citibike initially ticked up then dropped
<br>
<div <div id="subway_citibike" style="position: static"></div>
<p align="right" style="font-size:60%;">
<i>
Data from MTA turnstiles and Citibike
<br>
2019 data has been shifted to match 2020 weekdays
</i>
</p>
 
 
#### Daily NYC 311 calls reflect the city's changing atmosphere<br><i id='threeOneOneTitle' style='font-size:80%'>Non-emergency police matter</i>
<!-- arrows 
<div class="arrowed">
  <div class="arrow-6"></div>
</div>
-->

<!-- Dropbown menu -->
<div class="dropdown">
  <button class="dropbtn">Categories</button>
  <div class="dropdown-content">
    <a onclick="update311('Blocked_driveway'); changeText311('Blocked driveway')">Blocked driveway</a>
    <a onclick="update311('Consumer_complaint'); changeText311('Consumer complaint')">Consumer complaint</a>
    <a onclick="update311('Derelict_vehicles'); changeText311('Derelict vehicles')">Derelict vehicles</a>
    <a onclick="update311('For_hire_vehicle_complaint'); changeText311('For-hire vehicle complaint')">For-hire vehicle complaint</a>
    <a onclick="update311('General'); changeText311('General')">General</a>
    <a onclick="update311('Illegal_parking'); changeText311('Illegal parking')">Illegal parking</a>
    <a onclick="update311('Lost_property'); changeText311('Lost property')">Lost property</a>
    <a onclick="update311('Noise_commercial'); changeText311('Noise: commercial')">Noise: commercial</a>
    <a onclick="update311('Noise_street_sidewalk'); changeText311('Noise: street sidewalk')">Noise: street sidewalk</a>
    <a onclick="update311('Non_emergency_police_matter'); changeText311('Non-emergency police matter')">Non-emergency police matter</a>
    <a onclick="update311('Rodent'); changeText311('Rodent')">Rodent</a>
    <a onclick="update311('Taxi_complaint'); changeText311('Taxi complaint')">Taxi complaint</a>
  </div>
  
</div>
<br>
<br>
<div <div id="threeOneOne" style="position: static"></div>
<p align="right" style="font-size:60%;">
<i>
Data from NYC OpenData
<br>
2019 data has been shifted to match 2020 weekdays
<br>
</i></p>


#### Daily change in American mobile phone location reveal our new normal<br><i id='trendsTitle' style='font-size:80%'>Retail & recreation</i>
<button class="button" onclick="update('retail_rec'); changeText('Retail & recreation')">Retail & recreation</button>
<button class="button" onclick="update('groc_pharm'); changeText('Grocery & pharmacy')">Grocery & pharmacy</button>
<button class="button" onclick="update('parks'); changeText('Parks')">Parks</button>
<button class="button" onclick="update('transit'); changeText('Transit stations')">Transit stations</button>
<button class="button" onclick="update('workplaces'); changeText('Workplaces')">Workplaces</button>
<button class="button" onclick="update('Residential'); changeText('Residential')">Residential</button>
<div <div id="trends" style="position: static"></div>
<p align="right" style="font-size:60%;"><i>Data from Google Mobility Reports. United States.<br></i></p>


#### Daily worldwide flights have plummeted
<br>
<div <div id="flights" style="position: static"></div>
<p align="right" style="font-size:60%;">
<i>
Data from flightradar24.com
<br>
Includes passenger, cargo, charter, and some business jet flights
</i>
</p>


#### Weekly American unemployment claims have skyrocketed
<br>
<div <div id="unemp" style="position: static"></div>
<p align="right" style="font-size:60%;"><i>Data from St. Louis Federal Reserve</i></p>




<br>
<br>
<br>

### Data sources
- [NYC subway](http://web.mta.info/developers/turnstile.html). The data is extremely messy. Forking [this database](https://github.com/joemarlo/NYC-data) may be helpful if you would like to automatically clean it.
- [NYC Citibike](https://s3.amazonaws.com/tripdata/index.html)
- [NYC 311 calls](https://data.cityofnewyork.us/Social-Services/311-Service-Requests-from-2010-to-Present/erm2-nwe9)
- [Google trends](https://www.google.com/covid19/mobility/) and scraped by [Duncan Garmonsway](https://github.com/nacnudus/google-location-coronavirus)
- [Flights](https://www.flightradar24.com/data/statistics)
- [Unemployment](https://fred.stlouisfed.org/series/ICSA)












<br>

---
*2020 April*  
*Find the code here: [github.com/joemarlo/NYC-data](https://github.com/joemarlo/NYC-data/tree/master/Analyses/COVID-impact)*


<!-- Load d3.js -->
<script src="https://d3js.org/d3.v4.min.js"></script>
<script src="https://d3js.org/d3-selection-multi.v0.4.min.js"></script>

<!-- Load d3 plots -->
<script src="/d3/covid-impact/covid_plots.js"></script>

<!-- Load custom CSS -->
<link rel="stylesheet" type="text/css" href="/posts-css/covid-impact.css">

<!-- For changing text in google trends plot title -->
<script>
function changeText(newText)
{
 document.getElementById('trendsTitle').innerHTML = newText;
}

function changeText311(newText)
{
 document.getElementById('threeOneOneTitle').innerHTML = newText;
}

</script>
