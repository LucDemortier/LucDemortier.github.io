---
layout: page
title:  "Project McNulty: Estimating the Risk of Heart Disease"
date: 15-May-2015
excerpt: What a single data set can teach us about indicators of heart disease...
---

My third project at the spring 2015 Metis data science bootcamp involved estimating the probability of heart disease for patients admitted to a hospital emergency room with symptoms of chest pain.  I used the so-called Cleveland heart disease data set available from the [UC Irvine Machine Learning Repository](https://archive.ics.uci.edu/ml/datasets/Heart+Disease).

The original Cleveland data set was donated in 1988 and contained 303 instances with 75 attributes each.  Unfortunately these data became [corrupted](https://archive.ics.uci.edu/ml/machine-learning-databases/heart-disease/WARNING) after the loss of a computing node, and the surviving data set contains only 14 attributes per instance.  These attributes are described in the table below.
{% marginnote 'Table 1: Attributes of the heart disease patients in the Cleveland data set; the last row (heart disease status) is the response variable; the type column indicates whether an attribute is binary (bin), integer (int), categorical (cat), or continuous (con).' %}

|     || attribute||description                                       | type| 
|----:|:-:|:---------|:-:|:-------------------------------------------|:-----|
|  1. ||age      ||age in years                                       | int  |
|  2. ||sex      ||0 = female, 1 = male                               | bin  |
|  3. ||cp       ||chest pain type (1,2,3, or 4)                      | cat  |
|  4. ||trestbps ||resting blood pressure (mm Hg)                     | con  |
|  5. ||chol     ||serum cholesterol (mg/dl)                          | con  |
|  6. ||fbs      ||fasting blood sugar (1 if > 120 mg/dl, 0 otherwise)| bin  |
|  7. ||restecg  ||resting electrocardiography results (0,1, or 2)    | cat  |
|  8. ||thalach  ||maximum heart rate achieved during thalium stress test| con  |
|  9. ||exang    ||exercise induced angina (1 = yes, 0 = no)          | bin  |
| 10. ||oldpeak  ||ST depression induced by exercise relative to rest | con  |
| 11. ||slope    ||slope of peak exercise ST segment (1,2, or 3)      | cat  |
| 12. ||ca       ||number of major vessels colored by fluoroscopy     | int  |
| 13. ||thal     ||thalium stress test result (3,6, or 7)             | cat  |
| 14. ||num      ||heart disease status: number of major vessels with >50% narrowing (0,1,2,3, or 4)| int |
 
Since I am interested in estimating a probability, it seems appropriate to use logistic regression.  The question is: What is the optimal set of attributes that should be used?  And how should we define optimality? Before answering these questions, it may be useful to look at how some of the attributes are distributed over the data set. 
{% maincolumn /assets/img/HeartDisease/mcnulty_fig1.jpg 'Figure 1: Histograms of age, resting blood pressure, serum cholesterol, and thalium test maximum heart rate for patients in the Cleveland data set, with and without heart disease.  All histograms are normalized to unit area.' %}
The plots show with various degrees of "obviousness" that people with heart disease tend to be older, have higher blood pressure, higher cholesterol levels, and lower maximum heart rate under stress than people without the disease.  Therefore, if we fit the simplest logistic model (log-odds linear in both the parameters and the features) to the Cleveland data, we would expect the coefficients of the age, blood pressure, and serum cholesterol features to be positive, and the coefficient of the maximum heart rate feature to be negative.  Here is what we actually find:
{% marginnote 'Table 2: Partial results of a fit to the Cleveland data of a logistic model with the thirteen features and one response variable of Table 1.  For each of the four features listed in the first column, the table shows the values of the fitted coefficient and standard error (columns 2 and 3), as well as their ratio (the z-value in column 4).' %}

| Feature | Coefficient | Standard Error | Z-Value |
|:--|--:|--:|--:|
| age       | -0.12 |  0.22 | -0.55 |
| trestbps  |  0.43 |  0.20 |  2.17 |
| chol      |  0.22 |  0.21 |  1.06 |
| thalach   | -0.42 |  0.26 | -1.63 |

The great surprise is that the coefficient of the age feature is negative, as if older people had a lower risk of heart disease{% sidenote 1 'Note that in describing Figure 1 I wrote that "people with heart disease tend to be older", whereas here I write that "older people [have] a lower risk of heart disease".  The first statement is mapped into the second via Bayes&#39; theorem, and this is taken care of by the logistic classifier.' %}!  This is not the whole story however.  The standard error on the age coefficient is quite large; it certainly allows for the true value of that coefficient to be zero or even positive: {% m%} -0.12 \pm 0.22 {% em %} means that, with 68% confidence, the true age coefficient is between -0.34 and 0.10.  This is also indicated by the z-value of -0.55, which is too small to provide convincing evidence against the null hypothesis that we don't need the age feature to account for the heart disease status of patients in the Cleveland data set.

The technical reason for this glitch is two-fold: the features in the Cleveland data set are correlated, and the sample size is too small to determine all the feature coefficients with sufficient precision.  This means that we need to be extra careful in selecting features for the final model.  One approach is to loop over all combinations{% sidenote 2 'The Cleveland data set provides thirteen features per patient, four of which are categorical (not including binary ones). When the categorical features are converted into binary "dummies" for fitting purposes, the total list of features expands to eighteen.  This yields 2<sup>18</sup>=262,144 possible combinations to check!' %} of features and select the one that yields the highest accuracy (defined as the fraction of times the logistic regression classifier correctly predicts the heart disease status of a patient).  Unfortunately this approach does not take precision into account: It simply assumes that we have a large enough sample to determine precisely the coefficient of any selected feature.  Sure enough, when I tried this approach, the age feature was included with a negative coefficient!

A better approach for this problem{% sidenote 3 'See section 4.4.2, pg. 124, of Trevor Hastie, Robert Tibshirani, and Jerome Friedman, ["The Elements of Statistical Learning: Data Mining, Inference, and Prediction"](http://statweb.stanford.edu/~tibs/ElemStatLearn/), Second Edition, Springer Series in Statistics (2009).' %} is to start with all features included, and then eliminate one by one features whose fitted coefficient is not significantly different from zero.  The order of elimination should be such that the model deviance (defined as minus twice the log-likelihood of the fitted model) remains the lowest possible.  Here is what the pseudo-code for this approach looks like:

```
    1.  Start by including all the features in the model.
    2.  Fit the model and extract the deviance.
    3.  Does any feature have a coefficient with z-value 
        less than 2?
    4.  If yes: 
    5.      For each feature still in the model:
    6.          remove the feature, 
    7.          refit, 
    8.          recompute the deviance, and
    9.          replace the feature.
    10.     Eliminate the feature whose removal caused 
            the smallest change in deviance.
    11.     Go back to step 2.
    12. If no:
    13.     Exit.
```

The output of this algorithm is the largest subset of features whose fit coefficients are at least two standard deviations away from zero, and such that the total change in fit deviance is the smallest possible.  When applied to the logistic regression model for the Cleveland data, seven features are selected, three of which are categorical.  This is the fit result:
{% marginnote 'Table 3: Final results of a fit to the Cleveland data of a logistic regression model with the optimal subset of features.  The cp_1, cp_2, and cp_3 features refer to the categorical variable cp (chest pain type) of Table 1.  For example, cp_1 is a binary feature indicating presence or absence of chest pain type 1.  Similar comments apply to slope_1 and thal_7.' %}

| Feature | Coefficient | Standard Error | Z-Value |
|:---|---:|---:|---:|
| sex      |      1.44 |     0.39 |     3.69  |
| trestbps |      0.44 |     0.18 |     2.48  |
| thalach  |     -0.44 |     0.22 |    -2.04  |
| ca       |      3.80 |     0.74 |     5.17  |
| cp_1     |     -2.23 |     0.62 |    -3.58  |
| cp_2     |     -1.27 |     0.51 |    -2.49  |
| cp_3     |     -2.11 |     0.44 |    -4.86  |
| slope_1  |     -1.47 |     0.38 |    -3.84  |
| thal_7   |      1.50 |     0.38 |     3.93  |

Age is not part of the optimal set of features, and the coefficients of all selected features have a z-value of at least 2 (in absolute value).

The coefficients listed in Table 3 allow us to construct a simple heart disease predictor.  For a graphical illustration based on D3.js, see [here](/assets/img/HeartDisease/heart_disease_predictor.html).