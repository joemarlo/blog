+++
draft = false
image = "img/posts/Citi-Bike-availability/citi_bike_availability_thumbnail.png"
date = 2020-12-10T18:00:13-04:00
showonlyimage = false
title = "Predicting Citi Bike stock in real time — on the cheap"
weight = 1
+++

*Extracting the best from Python and Shiny*
<!--more-->
***

The [Citi Bike](https://www.citibikenyc.com/) service is a ubiquitous feature of NYC, and the pandemic has perhaps elevated it to one of the primary modes of city transportation for many residents. Finding available docks to park your bike can be difficult though. I have created a continuously updating map of the Citi Bike stations that displays their current stock and their predicted stock in the next hour — perfect for figuring out where you can find an available bike for your next ride or an open dock at the end of your ride. The map is [live here](https://jmarlo.shinyapps.io/Citi-Bike-availability/).

The map is built using the best of Python and R Shiny. The former provides the modeling pipeline and the latter a simple frontend. It is all glued together using a [Raspberry Pi](https://en.wikipedia.org/wiki/Raspberry_Pi), a [cron](https://en.wikipedia.org/wiki/Cron) job, and an Azure server. The free [Shinyapps.io](https://www.shinyapps.io/) plan and $100 in Azure student credits makes this a $0 project.


<p align="center">
<a href="https://jmarlo.shinyapps.io/Citi-Bike-availability/">
<img src="/img/posts/Citi-Bike-availability/screenshot.png" width=90%>
</a>
</p>


### Creating it

The biggest hurdle was setting up a framework to digest the live data and somehow get it to the Shiny app. The modeling and Shiny app design are straightforward and aim to be as light as possible.

Citi Bike provides a [real time JSON feed](http://gbfs.citibikenyc.com/gbfs/gbfs.json) of the current data using the [General Bikeshare Feed Specification](https://github.com/NABSA/gbfs/blob/master/gbfs.md) format. This is easily parsed by Python and Pandas. Predictions can be made using your model of choice. I have used a XGBoost model; admittedly the modeling process could benefit from a little extra effort, but the focus of this project was on building a working pipeline, not a perfect model. The model still produces an RMSE of approximately two bikes, and, importantly, places little strain on the Raspberry Pi 1.4GHz ARM with 1GB of memory. An [ARIMA model](https://people.duke.edu/~rnau/411arim.htm) or [RNN](https://stanford.edu/~shervine/teaching/cs-230/cheatsheet-recurrent-neural-networks) could be a good 2.0 option for these time-series data.

The Python prediction script is automated to run every 15 minutes using cron. I recommend using [crontab.guru](https://crontab.guru/) for help parsing the crontab syntax. And finally the data is uploaded to an Azure database running MySQL. R Shiny connects to the database in the typical R way. Data is visualized using [leaflet](https://rstudio.github.io/leaflet/) and [ggplotly](https://plotly.com/ggplot2/getting-started/).

The full process is summarized into a simple diagram:

<p align="center">
<img src="/img/posts/Citi-Bike-availability/diagram.png" width=80%>
</p>

It's a bit cheating to say that Azure costs $0 due to the student credit. If you are running on a tight budget and do not have a large dataset, Dropbox and [rDrop2](https://github.com/karthik/rdrop2) package can be swapped for the Azure database. The downside is all the data will be pulled into memory by the Shiny app. Google Drive and [googledrive](https://googledrive.tidyverse.org/) package could also provide a similar solution.

The end product is satisfying: an automated process and a simple web interface to access helpful information for real daily decision making. Separating out the Python and R Shiny parts is a nice and tidy solution. The only interaction between the two is indirectly through the MySQL database. One could go a step further and replace Shiny with [Dash](https://plotly.com/dash/) or [Bokeh](https://bokeh.org/). Either way, Python and Shiny's respective strengths are showcased well here. And it's a great side project for that Raspberry Pi many people have laying around.


<br>

---
*2020 December*  
*Find the code here: [github.com/joemarlo/citi-bike-availability](https://github.com/joemarlo/citi-bike-availability)*
