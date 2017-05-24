---
layout: post
title: "Forecasting Wind Power"
date: 30 April 2017
excerpt: "Data collected on the power output of an array of wind turbines provide an interesting case study for machine learning algorithms. I probe it with a random forest from scikit-learn, the gradient boosting algorithm XGBoost, and a generalized linear model from the R package GAMLSS. Choosing the best model can be surprisingly tricky..."
comments: true
---

{% marginnote 'WTMN-1' '**Contents:** <br>[1. The Hackathon&#39;s Challenge](#TheChallenge) <br>[2. The Wind Turbine Data Set](#WindTurbineDataSet) <br>[3. Random Forest Analysis](#RandomForestAnalysis) <br>[4. Gradient Boosting with XGBoost](#XGBoost) <br>[5. A Generalized Linear Model](#GeneralizedAdditiveModel) <br>[6. Summary and Private Leaderboard](#Summary) <br>[7. Technical Note](#TechnicalNote)' %}
On July 19 and 20, 2016, the [H2O Open Tour](http://open.h2o.ai/nyc.html) came to the [IAC building](https://en.wikipedia.org/wiki/IAC_Building) in New York City to present their product and foster community with the help of tutorials, talks, social events, and a [hackathon](https://datahack.analyticsvidhya.com/contest/h2o-open-tour-nyc-hackathon/). The hackathon was much fun, so that I (sporadically) continued working on it after the conference. This post summarizes my findings.

<a name="TheChallenge"></a>

## The Hackathon's Challenge

The challenge was to predict the power output of an array of ten wind turbines using nearby wind velocity measurements. Each turbine was associated with a wind station that delivered four measurements: [zonal and meridional wind velocities](https://en.wikipedia.org/wiki/Zonal_and_meridional) at heights of 10 and 100 meters above ground (the heights of the turbines themselves were not provided).

The entire hackathon data set contained 168,000 *hourly* measurements made between January 1, 2012 and December 1, 2013 (100 weeks). Each measurement consisted of one turbine power output, plus the four associated wind measurements, plus a time stamp. Here is how the data set was split into training and testing parts:

{% marginnote "WTMN-2" "Table 1: Data sets used for the H2O hackathon in July 2016. The two leaderboard data sets were used for testing and did not reveal turbine power outputs." %}

| Data Set            | Data-Taking Period      | Number of observations |
|:--------------------|:-----------------------:|:----------------------:|
| Training            | 01/01/2012 - 07/31/2013 | 138,710                |
| Public Leaderboard  | 08/01/2013 - 09/30/2013 | 14,640                 |
| Private Leaderboard | 10/01/2013 - 12/01/2013 | 14,650                 |

The performance measure used to evaluate contributions was the [root-mean-square error (RMSE)](https://www.analyticsvidhya.com/blog/2016/02/7-important-model-evaluation-error-metrics/), i.e. the square root of the average squared difference between predicted and observed turbine outputs.

At the hackathon{% sidenote 'WTSN-0' 'A note about terminology: to train models I split the hackathon training data set into a subset for training (typically via cross-validation) and a subset for testing. For brevity I will refer to "the training (or testing) subset of the hackathon training data set" as "the training (or testing) subset". I will also abbreviate mention of "the entire hackathon training data set" into "the training data set".' %} I tried a random forest, which gave decent results. Later I improved on these by doing some feature engineering and using a more powerful learning algorithm. In the remainder of this post I describe the data set, feature engineering, and results obtained with a random forest and the gradient boosting algorithm known as XGBoost. I also try a generalized linear model based on a simplified information set; this gives surprisingly good results. I finish with a summary and technical note.


{% marginnote "btt-1" "[Back to Top](#TopOfPage)" %}

<a name="WindTurbineDataSet"></a>

## The Wind Turbine Data Set

The data set{% sidenote 'WTSN-1' 'Tao Hong, Pierre Pinson, Shu Fan, Hamidreza Zareipour, Alberto Troccoli and Rob J. Hyndman, "Probabilistic energy forecasting: Global Energy Forecasting Competition 2014 and beyond", International Journal of Forecasting, vol.32, no.3, pp 896-913, July-September, 2016.' %} was originally provided on the [Energy Forecasting blog of Dr. Tao Hong](http://blog.drhongtao.com/2017/03/gefcom2014-load-forecasting-data.html).

Each data record contains the following fields:

{% marginnote "WTMN-3" "Table 2: Features of the wind turbine data set. The zonal and meridional wind velocities are orthogonal vector components, so that their sum in quadrature equals the magnitude of the horizontal wind velocity. The response variable of interest is TARGETVAR." %}

|     | Feature   | Description                                      |
|----:|:----------|:-------------------------------------------------|
|  1. | ID        | Measurement ID                                   |
|  2. | ZONEID    | Zone id: an integer between 1 and 10             |
|     |           | (A zone comprises one wind turbine and           |
|     |           | one wind velocity measurement station.)          |
|  3. | TIMESTAMP | Date and time of measurement                     |
|  4. | TARGETVAR | Turbine power output, a number between 0 and 1   |
|  5. | U10       | Zonal wind velocity 10 meters above ground       |
|  6. | V10       | Meridional wind velocity 10 meters above ground  |
|  7. | U100      | Zonal wind velocity 100 meters above ground      |
|  8. | V100      | Meridional wind velocity 100 meters above ground |

As mentioned earlier, there are ten turbines. Turbine power outputs are normalized to their nominal capacities, making TARGETVAR a number between 0 and 1. A histogram of TARGETVAR in the training data set is shown in Figure 1.

{% maincolumn "assets/img/blog/WindTurbines/wind_turbine_outputs.png" "Figure 1: Histogram of turbine power output in the H2O training data set. All ten turbines are combined in this plot. For each turbine, power output is expressed as a fraction of total capacity." %}

The spectrum displays two noteworthy features: a large spike near 0, and a bump near 1. The bump corresponds to turbines working near full capacity. The large spike correlates with low wind velocities. Turbines won't produce any power if the wind speed falls below a threshold known as the *cut-in speed*{% sidenote 'WTSN-2' 'For some basic explanations of the workings of wind turbines, see the [Windpower Program](http://www.wind-power-program.com/turbine_characteristics.htm) website.' %}. The fraction of turbine outputs that are exactly zero is 8.5% for all turbines combined, but varies from 2.4% for turbine 2 to 17.0% for turbine 9.

Figure 2 illustrates the effect of the cut-in speed for turbine 9. Each dot in this figure is the tip of a horizontal wind velocity vector with tail at (0,0). Orange dots correspond to zero turbine output and congregate at low wind velocities. Blue dots correspond to non-zero turbine output and form a donut shape around the orange ones.

{% maincolumn "assets/img/blog/WindTurbines/donut_plots.png" "Figure 2: Scatter plots of zonal versus meridional wind velocity components at heights of 10m (left) and 100m (right) above ground, for turbine 9. Turbine output is zero for orange dots and greater than zero for blue dots. Note that the center point of both plots has coordinates (0,0)." %}

Since the measurements were taken at one-hour intervals, they are serially correlated. Figure 3 illustrates this correlation for the turbine-1 power output.

{% maincolumn "assets/img/blog/WindTurbines/autocorrelations.png" "Figure 3 Left: Scatter plot of turbine 1 output at time t versus turbine 1 output at time t minus one hour. Right: Autocorrelation of turbine 1 outputs for the first 200 lags. The dashed lines indicate the 99% confidence interval on autocorrelation if the latter is purely random." %}

Although the data set is structured in such a way that each record associates one of ten turbines with only one set of wind velocity measurements (U10, U100, V10, and V100), it is instructive to see how the power output of a given turbine correlates with wind measurements near other turbines. This is shown in Figure 4 for turbine 1. Note that for this turbine the strongest correlation appears to be with wind speeds measured near turbine 7.

{% fullwidth "assets/img/blog/WindTurbines/power_vs_windspeed.png" "Figure 4: Scatter plots of turbine 1 power output versus horizontal wind speed (the sum in quadrature of U10 and V10). Wind velocity is measured at eight stations located near the ten turbines. Turbines 4 and 5 share a wind measurement station, as do turbines 7 and 8. To facilitate comparison, the x- and y-scales are the same in all subplots." %}

Let's look at this more closely. How do turbine power and horizontal wind speed vary as a function of time? This is shown in Figure 5. Peaks in wind speed correspond to peaks in turbine power, and the effect of the cut-in speed is also visible (periods of zero turbine output associated with low, but non-zero wind speed).

There is also evidence of small random fluctuations on top of broad patterns. Presumably only the broad patterns are significant, being induced by actual changes in wind speed, and the small fluctuations are  measurement errors. For improving turbine output predictions it may help to average out the small fluctuations in measured wind speed, but the fluctuations in measured turbine output represent an irreducible limit on predictability.

{% fullwidth "assets/img/blog/WindTurbines/power_wind_vs_time.png" "Figure 5: Turbine 1 power and zone 7 wind speed, as a function of time in hours, for the first 400 hours in the training data set. The turbine power is expressed as a fraction of total capacity, and the wind speed is rescaled so as to fit in the same plot." %}

This brief exploration of the data set suggests some strategies for analysis:

1. Use *all* wind measurements for predicting each turbine output. Since there are eight wind velocity measurement stations and four measurements per station, this yields 32 features for predicting turbine output.

2. From the time stamp it may be useful to extract separately year, day of the year, and hour. This could help tease out effects due to daily and seasonal wind patterns.

3. Since the observed wind speeds exhibit fluctuations due to measurement error (Figure 5), we may be able to improve precision by computing a rolling window average. In other words, to predict turbine output at time t, use an average of wind speed measurements at times t, t-1, t-2, etc. Exactly how many measurements to include should be determined by looking at the desired performance measure, i.e. RMSE.

4. How should we handle the peak at zero turbine output? Would it help to split the machine learning problem into a classification first (zero turbine output versus nonzero turbine output), followed by a regression on the nonzero outputs?

{% marginnote "btt-2" "[Back to Top](#TopOfPage)" %}

<a name="RandomForestAnalysis"></a>

## Random Forest Analysis

In random forest regression one is averaging a large number of regression trees. Each tree is trained on a resampled version of the training data set, and each tree node is split on a random subset of the features. I tried two methods for making sure that *all* wind measurements can contribute to the forecasting of each turbine's output:

- **Method 1** is to transform the data so that each record contains all 32 wind speed measurements made at a given time{% sidenote 'WTSN-3' 'The original data set contains all 32 wind velocity measurements made *every hour* during the data-taking period.  These measurements are spread out over different records but there are no missing data.' %}. One can also put the 10 measured turbine outputs in each record, making the prediction target a 10-dimensional vector. The ```scikit-learn``` package can handle multi-dimensional targets, but others (e.g. XGBoost) can't. In that case I used ZONEID to split the data set into ten subsets, one per turbine, which I modeled independently.

- **Method 2** is to leave the given structure of the data in place (four wind measurements per record), but to convert ZONEID into a categorical variable and apply one-hot encoding to it. This yields nine binary variables, each one indicating whether or not TARGETVAR represents the power output of the turbine in the corresponding zone.{% sidenote 'WTSN-4' 'Technically there are ten binary variables, one per turbine, but the first one is deleted to avoid redundancy (since it corresponds to the nine remaining variables being zero).' %} At most one binary variable can be "on" in any given record.

It should be obvious that method 1 allows the modeling of correlations between a given turbine output and wind speed measurements in other zones, since each record in the transformed data contains *all* the wind measurements. Such correlations can also be modeled with method 2, due to the way decision trees work. One could for example have a tree node that is only reachable by zone-1 and zone-7 instances, and which splits based on one of the wind velocity components. This way, a prediction of the zone-1 turbine output could be affected by zone-7 wind measurements in the training sample, even though measurements in the two zones are always contained in separate data records.

Note how method 1 produces a dataset of 13,871 records, each of which contains 35 predictors ({% m %} 8*4 {% em %} wind measurements plus 3 time features: year, day of year, and hour). By contrast, method 2 produces a data set of 138,710 records, each with only 16 predictors (4 wind measurements, 9 zone id flags, and 3 time features). From the point of view of training a machine learning algorithm, method 2 may have an advantage --- more data records, fewer predictors per record --- that makes it easier to optimize. As a matter of fact, method 2 gave somewhat better results (see below).

Another optimization involved implementing a rolling average of wind measurements to predict turbine output. To select the window size of this rolling average, I tried including 1, 2, 3, and 4 measurements in the average. The best RMSE on the test subset was obtained with 3 measurements.

It is instructive to compare predicted and observed turbine output distributions for Method 2, separately in the training and testing subsets:

{% maincolumn "assets/img/blog/WindTurbines/wind_turbine_output_train_test_rfr16.png" "Figure 6 top: Fractional turbine power output in the training subset, as observed (green histogram) and modeled in Method 2 (blue histogram). The overlap between the two distributions appears in a blue-green shade. Bottom: same for the testing subset." %}

The random forest models don't do a very good job of modeling the spike at zero output, or the bump near one. This is shown above for Method 2, but is also true for Method 1.

For method 1, the public leaderboard of the hackathon{% sidenote 'WTSN-5' 'After the hackathon I gained access to the measured turbine powers (as fractions of total capacity) for the public and private testing data sets, see side-note 2.' %} yields:
{% math %}
{\rm RMSE} \;=\; 0.180311,
{% endmath %}
and for method 2:
{% math %}
{\rm RMSE} \;=\; 0.167223.
{% endmath %}
In an attempt to improve on these numbers I'll use a more powerful machine learning algorithm in the next section. Also, I'll be using method 1 from now on since that's the only one that works with a generalized linear model (see below).

The following couple of plots show the feature importances in Methods 1 and 2:

{% maincolumn "assets/img/blog/WindTurbines/rfr35_feature_importances.png" "Figure 7: Feature importances for the random forest model in Method 1. Wind speed labels indicate the zone they belong to. For example, the label of the top feature, U100_rm3_6, refers to the zonal wind speed 100 meters above ground near turbine 6, as a rolling mean over 3 measurements. Some hyper-parameter values used for this model are noted on the plot." %}

{% maincolumn "assets/img/blog/WindTurbines/rfr16_feature_importances.png" "Figure 8: Feature importances for the random forest model in Method 2. Some hyper-parameter values used for this model are noted on the plot. The ZoneId_n features are binary variables, which equal 1 in records where the wind and turbine measurements were made in zone n." %}

Wind speeds are more important than measurement dates or times, but no single feature stands out by itself.

{% marginnote "btt-3" "[Back to Top](#TopOfPage)" %}

<a name="XGBoost"></a>

## Gradient Boosting with XGBoost

In gradient boosting models, a weak learner is improved by the sequential addition of more weak learners.  Each learner in the sequence focuses on instances that are not well modeled by the previous learner. To understand how this might work{% sidenote 'WTSN-6' 'See this [blog post](https://gormanalysis.com/gradient-boosting-explained/) by Ben Gorman for an insightful introduction to gradient boosting.' %}, consider a regression model for example. The first weak learner attempts to fit the *response variable* directly, whereas the second weak learner tries to model the *residuals* of the fit by the first one. Thus the sum of these two learners should be better than the first one alone. The idea of boosting is to keep adding weak learners in this fashion until no further improvement is obtained. In practice, learners in gradient boosting models attempt to model the *gradient of a loss function*. This is more general than residuals and therefore more useful. Typically, shallow trees are used as weak learners, but the algorithm works with any kind of weak learner.

One of the most popular and powerful implementations of gradient boosting is XGBoost{% sidenote 'WTSN-7' 'Tianqi Chen and Carlos Guestrin, ["XGBoost: A scalable tree boosting system"](http://www.kdd.org/kdd2016/papers/files/rfp0697-chenAemb.pdf), KDD 2016 Proceedings of the 22nd ACM SIGKDD International Conference on Knowledge Discovery and Data Mining, pages 785-794
(San Francisco, California, USA, August 13-17, 2016).' %}. This package has already been used with much success in several Kaggle competitions. It supplements the basic gradient boosting algorithm with regularization, shrinkage (via a learning rate), data and feature subsampling, and a number of computational optimizations.

Proper tuning of the hyperparameters of XGBoost requires some exploration, but is fairly straightforward. I optimized each turbine separately on a hyperparameter grid, using five-fold cross-validation. For each turbine the training features consisted of all 32 wind speed measurements as well as the year, day of the year, and hour of the observations.

Figure 9 compares observed and predicted turbine output distributions in the training and testing subsets.

{% maincolumn "assets/img/blog/WindTurbines/wind_turbine_output_train_test_xgb.png" "Figure 9 top: Fractional turbine power output in the training subset, as observed (green histogram) and modeled by XGBoost (blue histogram). The overlap between the two distributions appears in a blue-green shade. Bottom: same for the testing subset." %}

{% marginfigure "Fig10" "assets/img/blog/WindTurbines/H100_vs_H10.png" "Figure 10: Scatter plot of horizontal wind speed at 100m above ground versus that at 10m above ground, in zone 9. (The wind speed units are unknown.)" %}Note how, in the training subset, modeling of the spike at 0 and the bump near 1 is significantly better than with the random forest models. Unfortunately this does not generalize quite as nicely to the testing subset, which hints at overfitting. For further insight, Figure 10 plots horizontal wind speeds measured at 100m above ground, versus 10m, in zone 9. The simplest model for vertical wind profiles is a [power law](https://en.wikipedia.org/wiki/Wind_profile_power_law), according to which the ratio of wind speeds at two different heights is a constant. Here we see that this model is not very good for the data at hand. The ratio between the two wind speeds in Figure 10 varies between about 1.3 and 3.0, suggesting that knowledge of the wind speed at one height provides only limited information about wind speed at another height. Since turbine heights are not known, and presumably different from the heights where wind speeds are measured, this will make it difficult for a regressor algorithm to correctly identify the cut-in speed of a turbine in the training subset, and hence to predict when the turbine power output will be zero in the testing subset. A similar argument explains the defective modeling of the bump near 1 in Figure 9, bottom.

In spite of these problems I computed the performance measure for XGBoost on the test set of the public leaderboard of the hackathon. This yielded
{% math %}
{\rm RMSE} \;=\; 0.167607.
{% endmath %}

For the record, Figure 11 shows a plot of feature importances, averaged over all turbines.

{% maincolumn 'assets/img/blog/WindTurbines/xgb35_feature_importances.png' 'Figure 11: Feature importances for the XGBoost model, averaged over all turbines. Feature labels are the same as those in Figure 7.' %}

XGBoost makes better use of the "DAYOFYEAR" and "HOUR" features than the random forest models. Here for example, "DAYOFYEAR" is the third most important feature, whereas in random forest models it scores below all the wind velocity measurements.

As suggested earlier, I tried to improve on this result by first classifying turbine outputs as either "equal to zero" or "greater than zero", and then performing a regression on the latter outputs. Both steps were performed with XGBoost, but no improvement was obtained. Perhaps this is not so surprising in light of the above discussion of Figure 10.

{% marginnote "btt-4" "[Back to Top](#TopOfPage)" %}

<a name="GeneralizedAdditiveModel"></a>

## A Generalized Linear Model

For predicting the power output of a wind turbine, one could argue that the only features that matter are the *magnitudes* of the horizontal wind velocities near the turbines. The directions of the wind velocity vectors shouldn't matter since a turbine always orients itself to maximize its capture of wind power. The timing of wind measurements could add some useful information due to the cyclic nature of daily and seasonal wind patterns. However this is not expected to be a major effect. By taking the wind velocity magnitudes as the only predictors, it should be possible to simplify the prediction model considerably and perhaps gain some interpretability.

Ideally one would like to use a linear regression model, but this won't work here since the response variable is bounded between 0 and 1. A generalized linear model with a two-parameter [beta distribution](https://en.wikipedia.org/wiki/Beta_distribution) for the response variable is a better option. Unfortunately the beta distribution does not assign finite probabilities to the boundaries 0 and 1, and in the problem at hand we have a large peak at 0 as well as a few instances at 1 (see Figure 1). A possible solution is to use the so-called "beta inflated distribution": a mixture of a beta distribution, a peak at 0, and a peak at 1. As far as I know, only R provides a package that can handle this problem, namely [GAMLSS](http://www.gamlss.org/), which stands for "Generalized Additive Models for Location, Scale, and Shape".

First a word about GAMLSS. This is a very general framework for regression models. In ordinary linear regression the response variable has a normal distribution; its mean is a linear function of the predictor variables and its width is constant (homoscedasticity). In GAMLSS the response variable can have almost any distribution, discrete or continuous, and *all* the parameters of the response variable distribution (not just the mean) can be modeled via sums of functions (not necessarily linear) of the predictors.

Generalized additive models have the advantages over tree models that they are not random, and that they are faster to train (fit).

To keep things simple, I only used GAMLSS to obtain a generalized *linear* model. In this type of model, any parameter of the distribution of the response variable {% m %}Y{% em %} can be related via a one-to-one link function to a linear combination of the predictors {% m %}X_{i}{% em %}. Let's start with the distribution of {% m %}Y{% em %}. As mentioned earlier, I used a beta inflated distribution:
{% math %}
f(y\mid p_{0}, p_{1}, \alpha, \beta) =
\begin{cases}
p_{0} & \textrm{if}\; y=0,\\[1mm]
(1-p_{0}-p_{1})\; \dfrac{y^{\alpha-1}\, (1-y)^{\beta-1}}{B(\alpha,\beta)} & \textrm{if}\; 0 < y < 1,\\[1mm]
p_{1} & \textrm{if}\; y=1,
\end{cases}
{% endmath %}
where {% m %}p_{0}, p_{1}{% em %} are probabilities and {% m %}\alpha, \beta{% em %} are the beta distribution parameters. In what follows it will be convenient to use an alternative parametrization of this distribution:
{% math %}
\begin{align}
\mu    & = \frac{\alpha}{\alpha+\beta}, \\[1mm]
\sigma & = \frac{1}{\sqrt{1+\alpha+\beta}}, \\[1mm]
\nu    & = \frac{p_{0}}{1-p_{0}-p_{1}}, \\[1mm]
\tau   & = \frac{p_{1}}{1-p_{0}-p_{1}}.
\end{align}
{% endmath %}

Next, I linked two parameters of this distribution to linear combinations of the predictors. The first one is the mean {% m %}\mu{% em %} of the beta component of the distribution:
{% math %}
g(\mu) = c_{0} + c_{1} X_{1} + \cdots + c_{p} X_{p},
{% endmath %}
where the {% m %}c_{i}{% em %} are fit coefficients and the {% m %}X_{i}{% em %} are the predictors (the 16 horizontal wind velocity magnitudes in our problem). The link function {% m %}g{% em %} must map a number between 0 and 1 to the entire real line. A standard choice for this is the logit function:
{% math %}
g(\mu) = \log\left(\frac{\mu}{1-\mu}\right).
{% endmath %}

The second parameter I linked is the combination {% m %}\nu{% em %} of peak probabilities:
{% math %}
h(\nu) = d_{0} + d_{1} X_{1} + \cdots + d_{p} X_{p}.
{% endmath %}
The link function {% m %}h{% em %} maps a positive number to the real line. A standard choice for this is the natural logarithm:
{% math %}
h(\nu) = \log(\nu).
{% endmath %}

In this way I linked two out of the four parameters of the beta inflated distribution. The remaining two parameters, {% m %}\sigma{% em %} and {% m %}\tau{% em %}, are assumed independent of the predictors.

The GAMLSS fitter then uses the entire training subset to produce fitted values for the coefficients {% m %}c_{0}, c_{1},\ldots,c_{p}{% em %} and {% m %}d_{0}, d_{1},\ldots,d_{p}{% em %}, as well as for {% m %}\sigma{% em %} and {% m %}\tau{% em %}.

Finally, given the fitted coefficients, the link functions {% m %}g{% em %} and {% m %}h{% em %}, and an instance from the training or testing subset, one can predict values for all four parameters of the distribution of {% m %}Y{% em %}. The predicted value of {% m %}Y{% em %} is then given by:
<a name="YpredEquation"></a>
{% math %}
y_{\rm pred} = p_{0}\times 0 + p_{1}\times 1 + (1-p_{0}-p_{1})\times \mu.
{% endmath %}

Figure 12 illustrates the performance of this model on the training and testing subsets. In both cases two plots are shown: the predicted turbine output distribution superimposed on the observed one, and a QQ plot. The QQ plot is a convenient way to check residuals in the case of a non-Gaussian response variable distribution. For a good fit the points on the QQ plot line up along the diagonal.

{% fullwidth "assets/img/blog/WindTurbines/wind_turbine_output_train_test_gamlss.png" "Figure 12 left: Fractional turbine power output in the training and testing subsets, as observed (green histogram) and as predicted by the GAMLSS model (blue histogram). The overlap between the two distributions appears in a blue-green shade. Right: QQ plots for the same training and testing subsets." %}

The fits are not very good; they fail to capture important features at both ends of the turbine power spectrum. However the fit to the testing subset does not appear to be worse than that to the training subset, suggesting that whatever features do get captured will generalize.

Here is the root-mean-square error for the public leaderboard of the hackathon:
{% math %}
{\rm RMSE} \;=\; 0.167303.
{% endmath %}
This is comparable to the results obtained with random forest models.

{% marginnote "btt-5" "[Back to Top](#TopOfPage)" %}

<a name="Summary"></a>

## Summary and Private Leaderboard Results

Table 3 summarizes the root-mean-square errors obtained for the four models on the public leaderboard of the hackathon.

{% marginnote "WTMN-4" "Table 3: Root-mean-square errors on the public leaderboard for the models described in this post." %}

| Model           | Public Leaderboard      |
|:----------------|:-----------------------:|
| Random Forest 1 | 0.180311                |
| Random Forest 2 | 0.167223                |
| XGBoost         | 0.167607                |
| GAMLSS          | 0.167303                |

Suppose that the hackathon was still going on, and that you had to choose a model for the (hidden) private leaderboard. Which one would you choose? The winner gets an iPad and a job offer at H2O...

Based on the public leaderboard one could go with either Random Forest 2 or GAMLSS, or even XGBoost. Tough choice? Unfortunately hindsight only adds to the difficulty. Take a look at the private leaderboard:

{% marginnote "WTMN-5" "Table 4: Root-mean-square errors on the public and private leaderboards for the models described in this post. These numbers can be compared with the top private leaderboard score obtained by the winner of the hackathon: RMSE=0.169463." %}

| Model           | Public Leaderboard      | Private Leaderboard      |
|:----------------|:-----------------------:|:------------------------:|
| Random Forest 1 | 0.180311                | 0.178977                 |
| Random Forest 2 | 0.167223                | 0.174105                 |
| XGBoost         | 0.167607                | 0.168778                 |
| GAMLSS          | 0.167303                | 0.176077                 |

Random Forest 2 and GAMLSS, the top two winners on the public leaderboard, don't do so well on the private leaderboard, where XGBoost is the clear winner.

Figure 13 provides another way to look at this case. It shows weekly root-mean-square errors over the entire hackathon data set. Again it is remarkable how well XGBoost fits the training data set. But when we look at the testing portion of the data set (the last 18 weeks), week-to-week fluctuations are so large that it is hard to pick out a winner.

{% maincolumn "assets/img/blog/WindTurbines/weekly_rmse_all.png" "Figure 13: Weekly root-mean-square errors over the entire hackathon data set, for the four models studied in this post. The first 82 weeks correspond to the training set and the remainder to the testing set (nine weeks for the public leaderboard followed by nine weeks for the private leaderboard)." %}

Note that the worse a model's fit is to a data set, the larger its week-to-week fluctuations are in the same data set. This is just a statistical effect. The RMSE is the square root of a [(non-central) chi-squared variate](https://en.wikipedia.org/wiki/Noncentral_chi-squared_distribution), for which variance and mean increase and decrease together.

The jump in performance between the training and testing data sets is quite striking for the tree-based models. A large jump could be an indication of overfitting. As argued above, XGBoost is probably overfitting the spike at 0 and the bump near 1 in the turbine output spectra. In contrast, there is no perceptible jump for the generalized linear model, and this could be an indication of underfitting.

I had hoped to use the above plot to draw conclusions about model trends beyond the training period (for example, does the RMSE get worse as time elapses after training?) Unfortunately the fluctuations are too large to permit a meaningful statement.

Here are some lessons that can be drawn from the  whole exercise:

1. A machine learning algorithm may perform differently depending on how the data are presented to it. This is the only difference between random forest models 1 and 2, and the latter is clearly better than the former on the testing set.

2. XGBoost is remarkable in its ability to model "singular" features such as the peak near zero turbine output. This is probably because of the way gradient boosting works, each tree acting mostly on instances mismodeled by the previous trees. In contrast, a random forest averages the results of independent trees, and this must have a smoothing effect on singularities. A similar effect is at work with generalized linear regression models (see the [equation for {% m %}y_{\rm pred}{% em %}](#YpredEquation) in the section on GAMLSS: {% m %}y_{\rm pred}{% em %} can never be exactly zero since it is a weighted average over the three components of a mixture.) Unfortunately, XGBoost's ability to model singularities in the training data set was partially the result of overfitting, the available wind speed measurements being not sufficiently informative.

3. Generalized linear models are very fast to train, but there may be a tradeoff to consider between training speed and modeling performance.

4. One way to take advantage of time series such as the hourly wind speed data set is to compute a rolling window average to smooth out random fluctuations. Here I used a simple arithmetic average over three consecutive measurements. However, a weighted average might make more sense, where older measurements are weighed down compared to recent ones. One option is the so-called Exponentially Weighted Moving Average, implemented by [the function `ewm()` in pandas](http://pandas.pydata.org/pandas-docs/stable/computation.html#exponentially-weighted-windows).

{% marginnote "btt-6" "[Back to Top](#TopOfPage)" %}

<a name="TechnicalNote"></a>

## Technical Note

Python and R Jupyter notebooks for this analysis can be found in [my GitHub repository WindTurbineOutputPrediction](https://github.com/LucDemortier/WindTurbineOutputPrediction).
