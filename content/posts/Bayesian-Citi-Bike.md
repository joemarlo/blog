+++
draft = false
image = "img/posts/Bayesian-Citi-Bike/Bayesian-CB-thumbnail.png"
date = 2020-05-30T13:49:44-04:00
showonlyimage = false
title = "A Bayesian approach to predicting Citi Bike rides"
weight = 4
+++

*Understanding Citi Bike through weather*
<!--more-->
***


Citi Bike is the [number three](https://www1.nyc.gov/html/dot/downloads/pdf/mobility-report-singlepage-2019.pdf) mode of public transportation in New York City behind the subway and buses. There were 1.2 million Citi Bike trips in February 2020 equaling to [40,000 daily trips](https://d21xlh2maitm24.cloudfront.net/nyc/February-2020-Citi-Bike-Monthly-Report.pdf?mtime=20200313132246). Unlike it's larger transit counterparts, Citi Bike daily ridership is difficult to predict as it is a mixture of commuter and leisure riders — the former is tied to weekdays and the latter is tied to the weather. 

Given this limited information of just weather and weekday, can Citi Bike ridership be accurately predicted?  

## Key findings

A Bayesian negative binomial model can predict the number of daily Citi Bike trips using just the weather and day of the week. It clearly captures the seasonality of bike rides, and it provides a credible interval that covers almost every actual observation. The model suggests there are more trips on weekdays, less during rainy days, more as the temperature increases, and less as the wind increases. These are all intuitive drivers of Citi Bike trips. However, using point estimates alone, the RMSE of 14,001 is too large for many practical uses.

<p align="center">
<img src="/img/posts/Bayesian-Citi-Bike/NG_out_of_sample_preds.svg" width=90%>
</p>

<br>

## Analysis process

* <a href="#citi-bike-data">**The data**</a>
* <a href="#model-selection">**Model selection**</a>
* <a href="#drawing-from-the-prior-predictive-distribution">**Drawing from the prior predictive distribution**</a>
* <a href="#conditioning-on-the-observed-data">**Conditioning on the observed data**</a>
* <a href="#data-characteristics">**Data characteristics**</a>
* <a href="#evaluating-the-negative-binomial-model">**Evaluating the negative binomial model**</a>
* <a href="#predicting-new-data">**Predicting new data**</a>
* <a href="#an-alternative-model-poisson">**An alternative model: Poisson**</a>
* <a href="#final-thoughts">**Final thoughts**</a>
* <a href="#addendum-a-frequentist-comparison">**Addendum: a frequentist comparison**</a>

<br>

## Citi Bike data

Citi Bike publishes real-time and [monthly datasets](https://www.citibikenyc.com/system-data) detailing each bike trip taken since 2013. Information includes the date, start- and end-times, departure and arrival stations, subscriber status, rider sex and rider birth year. Ridership has been steadily growing with an average daily ridership of 26,238 in 2013 compared to 56,306 in 2019. To minimize the impact of this omitted growth variable while maximizing the size of the training set, only data from 2017, 2018, and 2019 will be included. The data is then aggregated and [bike trip counts are calculated for each day](https://github.com/joemarlo/NYC-data/blob/master/Analyses/Bayesian-Citibike/Dataset_for_bayesian_project.R). A random sample of 80% is used to train the model and the remaining 20% is used for model prediction evaluation.

### Weather data
The Citi Bike dataset does not contain weather data. Weather information is obtained from the National Oceanic and Atmospheric Administration (NOAA) for the Central Park weather station. Information includes the daily precipitation, average temperature, and maximum gust speed. The data is collected during the day (i.e. ex post). The final model will be used for prediction so a practical application would require a weather forecast (i.e. ex ante). This discontinuity is acceptable as one-day weather forecasts are quite accurate.

### Final dataset

The [final dataset](https://github.com/joemarlo/NYC-data/blob/master/Analyses/Bayesian-Citibike/Daily_Citibike_counts.csv) consists of 938 observations and five variables.

<!-- table of variables-->
<div class="table-responsive">
<table class="table table-hover" style="margin-left: auto; margin-right: auto;">
 <thead>
  <tr>
   <th style="text-align:left;"> Variable </th>
   <th style="text-align:left;"> Type </th>
   <th style="text-align:left;"> Description </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> Trip_count </td>
   <td style="text-align:left;"> continuous </td>
   <td style="text-align:left;"> Count of daily Citi Bike trips </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Weekday </td>
   <td style="text-align:left;"> boolean </td>
   <td style="text-align:left;"> 1 = weekday, 0 = weekend </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Precipitation </td>
   <td style="text-align:left;"> boolean </td>
   <td style="text-align:left;"> 1 = rain, 0 = no rain </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Temp </td>
   <td style="text-align:left;"> integer </td>
   <td style="text-align:left;"> Average daily temperature in Fahrenheit </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Gust_speed </td>
   <td style="text-align:left;"> continuous </td>
   <td style="text-align:left;"> Maximum gust speed in miles per hour </td>
  </tr>
</tbody>
</table>
</div>

<br>

## Model selection

The outcome variable is a simple count of how many bike trips occurred in a single day. Count data is frequently modeled using a Poisson model or the more flexible negative binomial model. The data is assumed to not be zero-inflated as it seems unlikely there would be many days, if any, where there were no Citi Bike trips. 

First, we will fit a negative binomial then evaluate it against a Poisson using expected log predictive density, effective number of model parameters, and Pareto k values via leave-one-out cross-validation.

The model form in R syntax:  

<link rel="stylesheet" href="https://fonts.googleapis.com/css?family=Inconsolata">

><span style="font-family:'Inconsolata', 'Roboto', 'Georgia', 'Times New Roman';">Trip count ~ Weekday + Precipitation + Temperature + Gust speed</span>

The primarily tool for this analysis is the excellent `R` package for Bayesian generalized models [`brms`](https://mc-stan.org/users/interfaces/brms) — which interfaces with [Stan](https://mc-stan.org/).

<br>

## Drawing from the prior predictive distribution

Specifying priors is the first step of a Bayesian analysis. Each coefficient plus the intercept and shape parameter need a respective prior. Passing the model specification to `brms::get_prior()` returns the priors that need to be set along with their default values. These should be replaced with our own priors.

> **Interpreting coefficients (and priors)**  
> Negative binomial models' outcomes are specified in log counts so the coefficients need to be interpreted accordingly. For example, if the coefficient estimate is 0.50 then for a one-unit increase in this predictor variable, the difference in the expected log count of the number of bike trips is expected to increase by 0.50 — given everything else is held constant. When setting priors it's more helpful to think about the coefficient like this:
>- If our outcome variable is 50,000 bike trips and we believe changing only this one predictor will have a positive effect of 33,000 bike trips then the coefficient estimate for this predictor would be: **log(83,000) - log(50,000) = 0.50**
<!-- https://stats.idre.ucla.edu/r/dae/negative-binomial-regression/ -->

<br>

### Specifying priors

Temperature and gust speed are not likely to be associated with large effects on the outcome for each one unit increase. Temperature is in Fahrenheit and gust speed is in miles per hour. A one unit increase in either of these is unlikely to be associated with a measurable effect on the trips. Both are set to normal(0, 0.01).

Weekday and precipitation, I believe, are more likely to be associated with a larger effect since these are binary variables. However, their associated effects will be inverses of each other: weekdays have more commuter riders and rainy days have less overall riders. The priors are set to normal(0.5, 0.1) and normal(-0.5, 0.1) respectively.

<!-- table of variables-->
<div class="table-responsive">
<table class="table table-hover table-responsive" style="margin-left: auto; margin-right: auto;">
 <thead>
  <tr>
   <th style="text-align:left;"> Prior </th>
   <th style="text-align:left;"> Class </th>
   <th style="text-align:left;"> Coef </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> normal(0, 0.01) </td>
   <td style="text-align:left;"> b </td>
   <td style="text-align:left;"> Gust_speed </td>
  </tr>
  <tr>
   <td style="text-align:left;"> normal(-0.5, 0.1) </td>
   <td style="text-align:left;"> b </td>
   <td style="text-align:left;"> Precipitation </td>
  </tr>
  <tr>
   <td style="text-align:left;"> normal(0, 0.01) </td>
   <td style="text-align:left;"> b </td>
   <td style="text-align:left;"> Temp </td>
  </tr>
  <tr>
   <td style="text-align:left;"> normal(0.5, 0.1) </td>
   <td style="text-align:left;"> b </td>
   <td style="text-align:left;"> Weekday </td>
  </tr>
  <tr>
   <td style="text-align:left;"> normal(10, 1) </td>
   <td style="text-align:left;"> Intercept </td>
   <td style="text-align:left;">  </td>
  </tr>
  <tr>
   <td style="text-align:left;"> exponential(10) </td>
   <td style="text-align:left;"> shape </td>
   <td style="text-align:left;">  </td>
  </tr>
</tbody>
</table>
</div>

We can draw from this model using `brms::brm()` with the optional arguments `family = 'negbinomial'` and `sample_prior = "only"`. The resulting estimates are inline with our priors except for maybe that shape parameter seems a little off. More on that later.

```
prior_estimate <- brm(Trip_count ~ Weekday + Precipitation + Temp + Gust_speed,
                      data = Citibike_train, family = 'negbinomial',
                      prior = priors, sample_prior = "only")
```

<p align="center">
<img src="/img/posts/Bayesian-Citi-Bike/prior_estimates.svg" width=90%>
</p>

And finally, we compute the expected value using `brms::pp_expect()`. The below plot shows these draws from the prior distribution of the conditional expectation are reasonable. Examining the deciles shows that the middle 90% are covering a reasonable range of data. The values are little low but close to the actuals: the mean draw is 38,778 whereas the mean daily ridership for 2017-2019 is closer to 50,000.

```
prior_draws <- pp_expect(draws, nsamples = 4000)
```

<p align="center">
<img src="/img/posts/Bayesian-Citi-Bike/prior_draws.svg" width=90%>
</p>

<br>

## Conditioning on the observed data

Since the priors are reasonable we can now condition on the data. This is simple using the `brms` package; simply call `brms::update()` on the model with the argument `sample_prior = "no"`.

```
post_nb <- update(prior_estimate, sample_prior = "no")
```

The model converges and Rhat values are each 1.00. The effective-sample-sizes are all large as well, ranging from 2,800 to 5,500. Importantly, we see the estimates are close to the priors. 

The largest surprise is that precipitation is associated with a larger effect on the outcome than weekday which may indicate that the hypothesized weekday commuters do not have as strong a positive effect as rain has a negative effect.

<!-- table of model parameters-->
<div class="table-responsive">
<table class="table table-hover table-responsive" style="margin-left: auto; margin-right: auto;">
 <thead>
  <tr>
   <th style="text-align:left;">   </th>
   <th style="text-align:right;"> Estimate </th>
   <th style="text-align:right;"> Est.Error </th>
   <th style="text-align:right;"> l-95% CI </th>
   <th style="text-align:right;"> u-95% CI </th>
   <th style="text-align:right;"> Rhat </th>
   <th style="text-align:right;"> Bulk_ESS </th>
   <th style="text-align:right;"> Tail_ESS </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> Intercept </td>
   <td style="text-align:right;"> 9.41 </td>
   <td style="text-align:right;"> 0.08 </td>
   <td style="text-align:right;"> 9.25 </td>
   <td style="text-align:right;"> 9.58 </td>
   <td style="text-align:right;"> 1.000 </td>
   <td style="text-align:right;"> 4,933 </td>
   <td style="text-align:right;"> 3,338 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Weekday </td>
   <td style="text-align:right;"> 0.24 </td>
   <td style="text-align:right;"> 0.03 </td>
   <td style="text-align:right;"> 0.19 </td>
   <td style="text-align:right;"> 0.30 </td>
   <td style="text-align:right;"> 1.000 </td>
   <td style="text-align:right;"> 4,006 </td>
   <td style="text-align:right;"> 2,828 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Precipitation </td>
   <td style="text-align:right;"> -0.36 </td>
   <td style="text-align:right;"> 0.03 </td>
   <td style="text-align:right;"> -0.42 </td>
   <td style="text-align:right;"> -0.30 </td>
   <td style="text-align:right;"> 1.003 </td>
   <td style="text-align:right;"> 4,007 </td>
   <td style="text-align:right;"> 2,830 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Temp </td>
   <td style="text-align:right;"> 0.02 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 0.02 </td>
   <td style="text-align:right;"> 0.02 </td>
   <td style="text-align:right;"> 1.000 </td>
   <td style="text-align:right;"> 4,433 </td>
   <td style="text-align:right;"> 3,095 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Gust_speed </td>
   <td style="text-align:right;"> -0.01 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> -0.02 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 1.000 </td>
   <td style="text-align:right;"> 5,583 </td>
   <td style="text-align:right;"> 2,635 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> shape </td>
   <td style="text-align:right;"> 8.39 </td>
   <td style="text-align:right;"> 0.43 </td>
   <td style="text-align:right;"> 7.59 </td>
   <td style="text-align:right;"> 9.24 </td>
   <td style="text-align:right;"> 1.003 </td>
   <td style="text-align:right;"> 4,003 </td>
   <td style="text-align:right;"> 2,773 </td>
  </tr>
</tbody>
</table>
</div>

There's a few drawbacks to these priors. Notably, the shape parameter was misspecified, and the temperature and weekday beliefs may have been too strong (i.e. the resulting distributions of prior estimates are too narrow). It would have been more appropriate to have slightly wider priors, and the shape parameter should have been set with rate = 0.1 instead of rate = 10. Rate = 0.1 corresponds to a mean of approximately 10 which was the goal. Below, the darker grey fill are the posterior estimates and the black lines are the prior estimates.

<p align="center">
<img src="/img/posts/Bayesian-Citi-Bike/posterior_estimates.svg" width=90%>
</p>

The posterior draws are well within range — these values are, effectively, 4,000 estimates of the model's prediction for each observation so they should follow the data closely. The middle 90% [26,141, 82,748] fits the data well; the middle 90% of the actual data is [25,881, 75,358]. Additionally, the mean (52,636) and median (49,902) are close to the actual data (51,528 and 53,809 respectively).

<p align="center">
<img src="/img/posts/Bayesian-Citi-Bike/posterior_draws.svg" width=90%>
</p>

<br>

## Data characteristics

Since we've fit the model we can now examine the data closely. The trip count is roughly normally distributed but slightly left-skewed with a mean of 51,895. There are more trips on weekdays and less for rainy days. Trips are positively correlated (0.77) with temperature and negatively correlated (-0.44) with gust speed. Temperature is left skewed and gust speed is right skewed.

<p align="center">
<img src="/img/posts/Bayesian-Citi-Bike/pairs.svg" width=90%>
</p>

<br>

## Evaluating the negative binomial model

The model meets all the standard criteria. Executing leave-one-out cross-validation via `brms::loo()` we see the Pareto k estimates are fine with values less than 0.5 indicating no high leverage data points. The expected log predictive density (`elpd_loo`) is approximately -8,000 — this will be important when comparing against the Poisson model. And the expected number of parameters in the model, `p_loo`, is approximately six. Note that `looic` is just `elpd_loo` multiplied by -2 and is used similarly.

<!-- table of neg binom diagnostics-->
<div class="table-responsive">
<table class="table table-hover table-responsive" style="margin-left: auto; margin-right: auto;">
 <thead>
  <tr>
   <th style="text-align:left;">   </th>
   <th style="text-align:right;"> Estimate </th>
   <th style="text-align:right;"> SE </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> elpd_loo </td>
   <td style="text-align:right;"> -8,330.8 </td>
   <td style="text-align:right;"> 21.51 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> p_loo </td>
   <td style="text-align:right;"> 5.7 </td>
   <td style="text-align:right;"> 0.76 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> looic </td>
   <td style="text-align:right;"> 16,661.6 </td>
   <td style="text-align:right;"> 43.03 </td>
  </tr>
</tbody>
</table>
</div>

<br>

<p align="center">
<img src="/img/posts/Bayesian-Citi-Bike/PSIS_ng.svg" width=90%>
</p>

The model estimates are in-line with the actual observations, and it clearly captures the seasonality of trips. The credible interval may be wide but the middle of the estimates are close to the actuals.

<p align="center">
<img src="/img/posts/Bayesian-Citi-Bike/NG_preds.svg" width=90%>
</p>


<br>

## Predicting new data

The goal of this exercise is accurate prediction. The data was originally split 80% (757 observations) for training and 20% (181 observations) for testing.

Posterior predictions are made using `brms::posterior_predict()` with argument `newdata = Citibike_test` data. Similar to the in-sample estimates, the out-of-sample estimates fit the data well but with a wide credible interval.

<p align="center">
<img src="/img/posts/Bayesian-Citi-Bike/NG_out_of_sample_preds.svg" width=90%>
</p>

<br>

## An alternative model: Poisson

Count data is most frequently associated with Poisson models, which are a special case of negative binomial. Below, the negative binomial model is refitted as a Poisson using the same model form.

<!-- table of poisson parametrs -->
<div class="table-responsive">
<table class="table table-hover table-responsive" style="margin-left: auto; margin-right: auto;">
 <thead>
  <tr>
   <th style="text-align:left;">   </th>
   <th style="text-align:right;"> Estimate </th>
   <th style="text-align:right;"> Est.Error </th>
   <th style="text-align:right;"> l-95% CI </th>
   <th style="text-align:right;"> u-95% CI </th>
   <th style="text-align:right;"> Rhat </th>
   <th style="text-align:right;"> Bulk_ESS </th>
   <th style="text-align:right;"> Tail_ESS </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> Intercept </td>
   <td style="text-align:right;"> 9.65 </td>
   <td style="text-align:right;"> 0.001 </td>
   <td style="text-align:right;"> 9.65 </td>
   <td style="text-align:right;"> 9.66 </td>
   <td style="text-align:right;"> 1.001 </td>
   <td style="text-align:right;"> 4,018 </td>
   <td style="text-align:right;"> 3,070 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Weekday </td>
   <td style="text-align:right;"> 0.19 </td>
   <td style="text-align:right;"> 0.000 </td>
   <td style="text-align:right;"> 0.19 </td>
   <td style="text-align:right;"> 0.19 </td>
   <td style="text-align:right;"> 1.001 </td>
   <td style="text-align:right;"> 1,388 </td>
   <td style="text-align:right;"> 1,593 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Precipitation </td>
   <td style="text-align:right;"> -0.29 </td>
   <td style="text-align:right;"> 0.000 </td>
   <td style="text-align:right;"> -0.29 </td>
   <td style="text-align:right;"> -0.29 </td>
   <td style="text-align:right;"> 1.003 </td>
   <td style="text-align:right;"> 1,010 </td>
   <td style="text-align:right;"> 1,422 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Temp </td>
   <td style="text-align:right;"> 0.02 </td>
   <td style="text-align:right;"> 0.000 </td>
   <td style="text-align:right;"> 0.02 </td>
   <td style="text-align:right;"> 0.02 </td>
   <td style="text-align:right;"> 1.000 </td>
   <td style="text-align:right;"> 4,124 </td>
   <td style="text-align:right;"> 3,082 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Gust_speed </td>
   <td style="text-align:right;"> -0.01 </td>
   <td style="text-align:right;"> 0.000 </td>
   <td style="text-align:right;"> -0.01 </td>
   <td style="text-align:right;"> -0.01 </td>
   <td style="text-align:right;"> 1.002 </td>
   <td style="text-align:right;"> 3,676 </td>
   <td style="text-align:right;"> 2,567 </td>
  </tr>
</tbody>
</table>
</div>

Compared to the negative binomial model, it has a significantly worse ELPD, is more complicated (considerably larger `p_loo` value), and has a substantial number of large Pareto k.  532 or 70% of the observations have Pareto k values larger than 0.5 indicating the posterior distribution is sensitive. 373 or 49% of the observations have values over 1.

<!-- table of poisson diagnositics -->
<div class="table-responsive">
<table class="table table-hover table-responsive" style="margin-left: auto; margin-right: auto;">
 <thead>
  <tr>
   <th style="text-align:left;">   </th>
   <th style="text-align:right;"> Poisson
Estimate </th>
   <th style="text-align:right;"> Poisson
SE </th>
   <th style="text-align:right;"> Negative binomial
Estimate </th>
   <th style="text-align:right;"> Negative binomial
SE </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> elpd_loo </td>
   <td style="text-align:right;"> -1,236,590 </td>
   <td style="text-align:right;"> 64,663 </td>
   <td style="text-align:right;"> -8,330.8 </td>
   <td style="text-align:right;"> 21.513 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> p_loo </td>
   <td style="text-align:right;"> 11,250 </td>
   <td style="text-align:right;"> 483 </td>
   <td style="text-align:right;"> 5.7 </td>
   <td style="text-align:right;"> 0.764 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> looic </td>
   <td style="text-align:right;"> 2,473,179 </td>
   <td style="text-align:right;"> 129,326 </td>
   <td style="text-align:right;"> 16,661.6 </td>
   <td style="text-align:right;"> 43.025 </td>
  </tr>
</tbody>
</table>
</div>

<br>

<p align="center">
<img src="/img/posts/Bayesian-Citi-Bike/PSIS_pois.svg" width=90%>
</p>

The Poisson model is estimating the data well but is severely overfitting.

<p align="center">
<img src="/img/posts/Bayesian-Citi-Bike/Pois_preds.svg" width=90%>
</p>



<br>

### Prediction comparison

Simplifying the predictions down to just the point estimates, we see the predictions are similar. Using the mean estimates for each out-of-sample observation, the root-mean-squared-error (RMSE) of the negative binomial and the Poisson are close. However, the severe overfitting of the poisson relative to the negative binomial is evident in the below plot.

<!-- table of RMSEs -->
<div class="table-responsive">
<table class="table table-hover table-responsive" style="margin-left: auto; margin-right: auto;">
 <thead>
  <tr>
   <th style="text-align:left;">   </th>
   <th style="text-align:left;"> RMSE </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> Negative binomial </td>
   <td style="text-align:left;"> 14,001 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Poisson </td>
   <td style="text-align:left;"> 12,605 </td>
  </tr>
</tbody>
</table>
</div>

<br>

<div class="frame">
   <img src="/img/posts/Bayesian-Citi-Bike/Pred_comparison.svg">
</div>

<br>
<br>

## Final thoughts

Between these two models the negative binomial is superior. Negative-binomial models allow the variance of the distribution to be larger than the mean. This is important in the Citi Bike data as it is over-dispersed: the mean is 51,824 and the variance 418,294,720. 

Both models suggest that there are more riders on weekdays, less riders during rain, more riders as temperature increases, and slightly less riders as the wind increases in speed.  Overall model performance is mediocre, though. The negative binomial model captures much of the variation due to weather and day of the week. However, the RMSE of 14,001 is rather large so it may not be an effective model in practice.

<br>
<br>
<br>

### Addendum: a frequentist comparison

The frequentist approach to the negative binomial is simple. In practice, just fit the negative binomial model via `MASS::glm.nb()` with the same model form specified above. The coefficient estimates are similar to the Bayesian approach. The standard errors are notably tight and the p-values are effectively zero. These are usually interpreted as all good signs.

<!-- table of frequentist model -->
<div class="table-responsive">
<table class="table table-hover table-responsive" style="margin-left: auto; margin-right: auto;">
 <thead>
  <tr>
   <th style="text-align:left;">   </th>
   <th style="text-align:right;"> Estimate </th>
   <th style="text-align:right;"> Std. Error </th>
   <th style="text-align:right;"> z value </th>
   <th style="text-align:right;"> Pr(&gt;|z|) </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> (Intercept) </td>
   <td style="text-align:right;"> 9.45 </td>
   <td style="text-align:right;"> 0.08 </td>
   <td style="text-align:right;"> 122.47 </td>
   <td style="text-align:right;"> 0.0000 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Weekday </td>
   <td style="text-align:right;"> 0.22 </td>
   <td style="text-align:right;"> 0.02 </td>
   <td style="text-align:right;"> 9.09 </td>
   <td style="text-align:right;"> 0.0000 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Precipitation </td>
   <td style="text-align:right;"> -0.34 </td>
   <td style="text-align:right;"> 0.03 </td>
   <td style="text-align:right;"> -11.62 </td>
   <td style="text-align:right;"> 0.0000 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Temp </td>
   <td style="text-align:right;"> 0.02 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 30.20 </td>
   <td style="text-align:right;"> 0.0000 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Gust_speed </td>
   <td style="text-align:right;"> -0.01 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> -3.55 </td>
   <td style="text-align:right;"> 0.0004 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> shape </td>
   <td style="text-align:right;"> 10.75 </td>
   <td style="text-align:right;">  </td>
   <td style="text-align:right;">  </td>
   <td style="text-align:right;">  </td>
  </tr>
</tbody>
</table>
</div>

And the RMSE is effectively the same as the Bayesian negative binomial model.

<!-- table of RMSEs -->
<div class="table-responsive">
<table class="table table-hover table-responsive" style="margin-left: auto; margin-right: auto;">
 <thead>
  <tr>
   <th style="text-align:left;">   </th>
   <th style="text-align:left;"> RMSE </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> Negative binomial - Frequentist </td>
   <td style="text-align:left;"> 13,923 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Negative binomial - Bayesian </td>
   <td style="text-align:left;"> 14,001 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Poisson - Bayesian </td>
   <td style="text-align:left;"> 12,605 </td>
  </tr>
</tbody>
</table>
</div>

However, after plotting the data it is obvious the model is tremendously overfitting similar to the Bayesian Poisson model.

<p align="center">
<img src="/img/posts/Bayesian-Citi-Bike/Freq_preds.svg" width=90%>
</p>

<br>

#### A perk of Bayesian

Bayesians assume model parameters are random and data is fixed whereas Frequentists assume parameters are fixed and data random. The benefit of the Bayesian approach allows us to make probabilistic statements about these parameters. For example, what is the probability that `Weekday` has an associated positive effect greater than 0.25? **0.41**.

<p align="center">
<img src="/img/posts/Bayesian-Citi-Bike/weekday_probability.svg" width=90%>
</p>

<!--
A similar approach can be applied to the predictions. We can gather a better understanding of the models' predictions by examining the breadth. For example, what do we think is the probability that on a **rainy weekend** day with a temperature of **60F** and a gust speed of **10mph** that more than 40,000 bike trips will be taken? The Bayesian approach can answer this question by sampling from the posterior distribution and then calculating the proportion of samples greater than 40,000: **0.11**.

<p align="center">
<img src="/img/posts/Bayesian-Citi-Bike/probability.svg" width=90%>
</p>
-->

To wrap it all up, the plot below shows the spread of each models' predictions. The predictions have been standardized so 0 represents a perfect prediction, and the ranges — represented by dark to light green — cover roughly 69%, 96%, and 99% of the predictions for that observation. The methods are not directly comparable between Frequentist and Bayesian but this still gives an idea of the spread of each model's predictions. The Bayesian negative binomial model always covers the actual value — represented by the dark vertical line at x = 0.

<p align="center">
<img src="/img/posts/Bayesian-Citi-Bike/relative_error.svg" width=90%>
</p>

<br>

---
*2020 May*  
*Find the code here: [github.com/joemarlo/NYC-data/Analyses/Bayesian-Citibike](https://github.com/joemarlo/NYC-data/tree/master/Analyses/Bayesian-Citibike)*

<!-- Load custom CSS -->
<link rel="stylesheet" type="text/css" href="/posts-css/Bayesian-Citi-Bike.css">