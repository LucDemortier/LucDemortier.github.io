---
layout: full-width-post
title: "Taking the Confusion out of the Confusion Matrix"
date: 26 July 2016
excerpt: "How Bayes' theorem helps organize the bewildering array of performance metrics that can be estimated from a classifier's confusion matrix."
comments: true
---
[Revised 21 September 2017, 24 December 2019, 18 January 2021]

> **Contents**
> <br>[1. Introduction](#Introduction)
> <br>[2. Bayes' theorem](#FourApplications)
> <br> &ensp;&ensp;&ensp;[2.1 Notation](#Notation)
> <br> &ensp;&ensp;&ensp;[2.2 The prior](#Prior)
> <br> &ensp;&ensp;&ensp;[2.3 The likelihood](#Likelihood)
> <br> &ensp;&ensp;&ensp;[2.4 The model evidence](#ModelEvidence)
> <br> &ensp;&ensp;&ensp;[2.5 The posterior](#Posterior)
> <br> &ensp;&ensp;&ensp;[2.6 Classifier precision versus measurement precision](#MeasurementPrecision)
> <br> &ensp;&ensp;&ensp;[2.7 Summary](#Summary)
> <br>[3. Joint probabilities](#JointProbabilities)
> <br>[4. Likelihood ratios](#LikelihoodRatios)
> <br>[5. When is a classifier useful?](#ClassifierUsefulness)
> <br>[6. Classifier scores](#ScoreBasedMetrics)
> <br> &ensp;&ensp;&ensp;[6.1 The receiver operating characteristic](#ROC)
> <br> &ensp;&ensp;&ensp;[6.2 The precision-recall curve](#PRC)
> <br>[7. Classifier operating points](#OperatingPoints)
> <br>[8. Performance metric estimation](#MetricsEstimation)
> <br>[9. Trends and baselines](#TrendsAndBaselines)
> <br>[10. Effect of prevalence on classifier training and testing](#EffectOfPrevalence)
> <br>[11. Mathematical Appendix](#Appendix)
> <br> &ensp;&ensp;&ensp;[11.1 Expectation values of sensitivity, specificity, and accuracy](#ExpectationValues)
> <br> &ensp;&ensp;&ensp;[11.2 Performance metrics versus threshold](#MetricsVsThreshold)

---

<a name="Introduction"></a>
## 1. Introduction
I always thought that the *confusion* matrix was rather aptly named, a reference not so much to the mixed performance of a classifier as to my own bewilderment at the number of measures of that performance. Recently however, I encountered [a brief mention](https://www.dataschool.io/simple-guide-to-confusion-matrix-terminology/) of the possibility of a Bayesian interpretation of performance measures, and this inspired me to explore the idea a little further. It's not that Bayes' theorem is needed to understand or apply the performance measures, but it acts as an organizing and connecting principle, which I find helpful. New concepts are easier to remember if they fit inside a good story.

Another advantage of taking the Bayesian route is that this forces us to view performance measures as probabilities, which are *estimated* from the confusion matrix. Elementary presentations tend to *define* performance metrics in terms of ratios of confusion matrix elements, thereby ignoring the effect of statistical fluctuations.

Bayes' theorem is not the only way to generate performance metrics. One can also start from joint probabilities, likelihood ratios, or classifier scores. The next five sections describe these approaches one by one, and include a discussion of a minimal condition for a classifier to be useful. This is then followed by a section on estimating performance measures, one on the effect of changing the prevalence in the training and testing data sets of a classifier, and a technical appendix.

<div style="text-align: right"><a href="#TopOfPage">Back to Top</a></div>
<a name="FourApplications"></a>
## 2. Bayes' theorem
Let's start from Bayes' theorem in its general form:
{% math %}
p(\lambda\mid X) \;=\; \frac{p(X\mid\lambda)\;\pi(\lambda)}{\int p(X\mid\lambda^{\prime})\;\pi(\lambda^{\prime})\;d\lambda^{\prime}}.
{% endmath %}
Here {% m %}X{% em %} represents the observed data and {% m %}\lambda{% em %} a parameter of interest. On the left-hand side is the posterior probability density of {% m %}\lambda{% em %} given {% m %}X{% em %}. On the right-hand side, {% m %}p(X\mid\lambda){% em %} is the likelihood, or conditional probability density of {% m %}X{% em %} given {% m %}\lambda{% em %}, and {% m %}\pi(\lambda){% em %} is the prior probability density of {% m %}\lambda{% em %}. The denominator is called marginal likelihood or model evidence. One way to think about Bayes' theorem is that it uses the data {% m %}X{% em %} to update the prior information {% m %}\pi(\lambda){% em %} about {% m %}\lambda{% em %}, and returns the posterior {% m %}p(\lambda\mid X){% em %}.

For a binary classifier {% m %}\lambda{% em %} is the true class to which instance {% m %}X{% em %} belongs and can take only two values, say 0 and 1. The denominator in Bayes' theorem then simplifies to:
{% math %}
p(X) \;\equiv\; \int p(X\mid\lambda^{\prime})\;\pi(\lambda^{\prime})\;d\lambda^{\prime}
     \;=\; p(X\mid\lambda\!=\!0)\,\pi(\lambda\!=\!0)\;+\;
           p(X\mid\lambda\!=\!1)\,\pi(\lambda\!=\!1).
{% endmath %}

We'll take a look at each component of Bayes' theorem in turn: the prior, the likelihood, the model evidence, and finally the posterior. Each of these maps to a classifier performance measure or a population characteristic. But first, a word about notation.

<div style="text-align: right"><a href="#TopOfPage">Back to Top</a></div>
<a name="Notation"></a>
### 2.1 Notation
If our sole purpose is to describe the performance of a classifier in general terms, the data {% m %}X{% em %} can be replaced by the *class label* {% m %}\ell{% em %} or by the *score* {% m %}q{% em %} assigned by the classifier (this score is not necessarily a probability and not necessarily between 0 and 1). Thus for example, {% m %} p(\lambda\!=\!1 \mid \ell\!=\!0) {% em %} represents the posterior probability that the true class is 1 given a class label of 0, and {% m %}p(\lambda\!=\!1\mid q\!\ge\! q_{T}){% em %} is the posterior probability that the true class is 1 given that the score {% m %}q{% em %} is above a pre-specified threshold {% m %}q_{T}{% em %}.

To simplify the notation we'll write {% m %}\lambda_{0}{% em %} to mean {% m %}\lambda=0{% em %}, {% m %}\lambda_{1}{% em %} to mean {% m %}\lambda=1{% em %}, and similarly with {% m %}\ell_{0}{% em %} and {% m %}\ell_{1}{% em %}. Going one step further, where convenient we'll write the prior as {% m %}\pi_{i}{% em %} instead of {% m %}\pi(\lambda_{i}){% em %} and the queue rate as {% m %}p_{i}{% em %} instead of {% m %}p(\ell_{i}){% em %}, where {% m %}i=0,1{% em %}. With this notation the equation for the denominator of Bayes' theorem becomes:
{% math %}
p(X) \;=\; p(X\mid\lambda_{0})\,\pi_{0} \;+\; p(X\mid\lambda_{1})\,\pi_{1}.
{% endmath %}
Finally, we'll use the notation {% m %}A\;{% em %}&{% m %}\;B{% em %} to indicate the logical *and* of {% m %}A{% em %} and {% m %}B{% em %}, the notation {% m %}A \lor B{% em %} for its logical *or*, and {% m %}A \mid B{% em %} for the conditional {% m %}A{% em %} *given* {% m %}B{% em %}.

<div style="text-align: right"><a href="#TopOfPage">Back to Top</a></div>
<a name="Prior"></a>
### 2.2 The prior
The first ingredient in the computation of the posterior is the prior {% m %}\pi(\lambda){% em %}. To fix ideas, let's assume that {% m %}\lambda_{1}{% em %} is the class of interest, generically labeled "positive"; it indicates "effect", "signal", "disease", "fraud", or "spam", depending on the context. Then {% m %}\lambda_{0}{% em %} indicates the lack of these things and is generically labeled "negative". To fully specify the prior we just need {% m %}\pi_{1}{% em %}, since {% m %}\pi_{0} = 1 - \pi_{1}{% em %}. The prior probability {% m %}\pi_{1}{% em %} of drawing a positive instance from the population is called the **prevalence**. It is a property of the population, not the classifier.

<div style="text-align: right"><a href="#TopOfPage">Back to Top</a></div>
<a name="Likelihood"></a>
### 2.3 The likelihood
The next ingredient is the conditional probability {% m %}p(X\mid\lambda){% em %}. Working with the classifier label {% m %}\ell{% em %} instead of {% m %}X{% em %} leads to the following four combinations:

- The **true positive rate**, also known as **sensitivity** or **recall**: {% m %}S_{e} \equiv p(\ell_{1}\mid \lambda_{1}){% em %},

- The **true negative rate**, also known as **specificity**: {% m %}S_{p}\equiv p(\ell_{0}\mid \lambda_{0}){% em %},

- The **false-positive or Type-I-error rate**: {% m %}\alpha\equiv p(\ell_{1}\mid\lambda_{0}) = 1 - S_{p}{% em %}, and

- The **false-negative or Type-II-error rate**: {% m %}\beta\equiv p(\ell_{0}\mid\lambda_{1}) = 1 - S_{e}{% em %}.

For example, {% m %}p(\ell_{1}\mid \lambda_{0}){% em %} is the conditional probability that the classifier assigns a label of 1 to an instance actually belonging to class 0. Note that the above four rates are independent of the prevalence and are therefore intrinsic properties of the classifier.

When viewed as a function of true class {% m %}\lambda{% em %}, for fixed label {% m %}\ell{% em %}, the conditional probability {% m %}p(\ell\mid\lambda){% em %} is known as the **likelihood function**. This functional aspect of {% m %}p(\ell\mid\lambda){% em %} is not really used here, but it is good to remember the terminology.

<div style="text-align: right"><a href="#TopOfPage">Back to Top</a></div>
<a name="ModelEvidence"></a>
### 2.4 The model evidence
Finally, we need the normalization factor in Bayes' theorem, the model evidence. For a binary classifier this has two components, corresponding to the two possible labels:

- The **positive labeling rate**: {% m %}p_{1} = p(\ell_{1}\mid\lambda_{0})\,\pi_{0} + p(\ell_{1}\mid\lambda_{1})\,\pi_{1}{% em %}, and

- The **negative labeling rate**: {% m %}p_{0} = p(\ell_{0}\mid\lambda_{0})\,\pi_{0} + p(\ell_{0}\mid\lambda_{1})\,\pi_{1}{% em %}.

The labeling rates are more commonly known as **queue rates**.

<div style="text-align: right"><a href="#TopOfPage">Back to Top</a></div>
<a name="Posterior"></a>
### 2.5 The posterior
Armed with the prior probability, the likelihood, and the model evidence, we can compute the posterior probability from Bayes' theorem.  There are four combinations of truth and label:

- The **positive predictive value**, also known as **precision**: {% m %}\mathit{ppv}\equiv p(\lambda_{1}\mid\ell_{1}) = \frac{p(\ell_{1}\mid\lambda_{1})\, \pi_{1}}{p_{1}} {% em %},

- The **negative predictive value**: {% m %}\mathit{npv}\equiv p(\lambda_{0}\mid\ell_{0}) = \frac{p(\ell_{0}\mid\lambda_{0})\, \pi_{0}}{p_{0}} {% em %},

- The **false discovery rate**: {% m %}\mathit{fdr}\equiv p(\lambda_{0}\mid\ell_{1}) = 1 - \mathit{ppv}{% em %}, and

- The **false omission rate**: {% m %}\mathit{for}\equiv p(\lambda_{1}\mid\ell_{0}) = 1 - \mathit{npv} {% em %}.

The precision, for example, quantifies the predictive value of a positive label by answering the question: If we see a positive label, what is the (posterior) probability that the true class is positive? (Note that the predictive values are not intrinsic properties of the classifier since they depend on the prevalence. Sometimes they are "standardized" by evaluating them at {% m %} \pi_{0} = \pi_{1} = 1/2 {% em %}.)

Figure 1 shows how the posterior probabilities {% m %}\mathit{ppv}{% em %}, {% m %}\mathit{npv}{% em %}, {% m %}\mathit{fdr}{% em %} and {% m %}\mathit{for}{% em %} are related to the likelihoods {% m %}S_{e}{% em %}, {% m %}S_{p}{% em %}, {% m %}\alpha{% em %} and {% m %}\beta{% em %} via Bayes' theorem:

{% fullwidth "assets/img/blog/ConfusionMatrix/FourBayesTheorems.png" "Figure 1: For each combination of class and label, Bayes' theorem connects the corresponding performance metrics of a classifier." %}

Note that if the operating point of the classifier is chosen in such a way that {% m %}p_{0} = \pi_{0}{% em %}, we'll have {% m %}\mathit{ppv} = S_{e}{% em %}, {% m %}\mathit{npv} = S_{p}{% em %}, and {% m %}\mathit{fdr}\times \mathit{for} = \alpha\times \beta{% em %}.
<br>
<div style="text-align: right"><a href="#TopOfPage">Back to Top</a></div>
<a name="MeasurementPrecision"></a>
### 2.6 Classifier precision versus measurement precision
Although we defined precision as a property of a *classifier* in a given population, there are reasons and ways to extend that definition to precision as a property of a *measurement*. Consider for example the Covid-19 pandemic. Tests for this disease are effectively classifiers, assigning negative and positive labels to members of the population. The positive predictive value of these tests depends on the prevalence {% m %}\pi_{1}{% em %}, both directly and indirectly through the queue rate {% m %}p_{1}{% em %}. We can make this dependence explicit:
{% math %}
\mathit{ppv} \;=\; \frac{S_{e}}{S_{e} - \alpha + \alpha\left/\pi_{1}\right.},
{% endmath %}
showing that {% m %}\mathit{ppv}{% em %} will decrease if {% m %}\pi_{1}{% em %} does. Suppose now that I take the test, and the result is positive. What are the chances that I actually have the disease? One answer would be the positive predictive value. However that number describes the test performance in the population and does not reflect the particular circumstances of my life. If I have no symptoms, and remained at home in isolation for the past two weeks, it seems unlikely that I would actually have the disease. This is where it is useful to remember the Bayesian construction of {% m %}\mathit{ppv}{% em %} as the posterior probability obtained by updating the prior {% m %}\pi_{1}{% em %} with the data from a test result. If we are talking about my particular test result, this prior should fold in more information than just the population prevalence. It should take into account the precautions I took as an individual, which could reduce {% m %}\pi_{1}{% em %} significantly, and therefore also {% m %}\mathit{ppv}{% em %}. The question of how exactly one should estimate prior probabilities is beyond the scope of this blog post, but the distinction between classifier precision and measurement precision is an important one to keep in mind.

<div style="text-align: right"><a href="#TopOfPage">Back to Top</a></div>
<a name="Summary"></a>
### 2.7 Summary
This section introduced twelve quantities:
{% math %}
\pi_{0},\; \pi_{1},\; S_{e},\; S_{p},\; \alpha,\; \beta,\; p_{0},\; p_{1},\; \mathit{ppv},\; \mathit{npv},\; \mathit{fdr},\; \mathit{for}.
{% endmath %}
Only three of these are independent, say {% m %}\pi_{1}{% em %}, {% m %}\alpha{% em %} and {% m %}\beta{% em %}. To see this, note that by virtue of their definition the first six quantities satisfy three conditions:
{% math %}
\pi_{0} + \pi_{1} = 1,\quad S_{e} + \beta = 1, \quad S_{p} + \alpha = 1,
{% endmath %}
and the last six can be expressed in terms of the first six: for {% m %}p_{0}{% em %} and {% m %}p_{1}{% em %} we have:
{% math %}
p_{0} = S_{p} \pi_{0} +  \beta \pi_{1}, \quad p_{1} = \alpha \pi_{0} + S_{e} \pi_{1},
{% endmath %}
and Bayes' theorem takes care of the remaining four (Figure 1).

The final number of independent quantities matches the number of degrees of freedom of a two-by-two contingency table such as the confusion matrix, which is used to estimate all twelve quantities (see [below](#MetricsEstimation)). It also tells us that we only need two numbers to fully characterize a binary classifier (since {% m %}\pi_{1}{% em %} is not a classifier property). As trivial examples, consider the majority and minority classifiers. Assume that class 0 is more prevalent than class 1. Then the majority classifier assigns the label 0 to every instance and has {% m %}\alpha=0{% em %} and {% m %}\beta=1{% em %}. The minority classifier assigns the label 1 to every instance and has {% m %}\alpha=1{% em %} and {% m %}\beta=0{% em %}. These classifiers are of course not very useful. As we will show below, for a classifier to be useful, it must satisfy {% m %} \alpha + \beta \lt 1 {% em %}.

<div style="text-align: right"><a href="#TopOfPage">Back to Top</a></div>
<a name="JointProbabilities"></a>
## 3. Joint probabilities
In some sense the most useful measures of classifier performance are *conditional* probabilities. By conditioning on something, we do not need to know whether or how that something is realized in order to make a valid statement. For example, by conditioning on true class, we ignore the prevalence when evaluating sensitivity and specificity. Similarly, conditioning on class label allows us to evaluate the predictive values while ignoring the actual queue rate.

In contrast, *joint* probabilities typically require more specification about the classifier and/or the population of interest, and are therefore less general than conditional probabilities. Nevertheless, by combining joint probabilities one can still obtain useful metrics. Start for example with the joint probability for class label and true class to be positive:
{% math %}
p(\ell_{1} \;\&\; \lambda_{1}).
{% endmath %}
Adding the joint probability for class label to be positive and true class to be negative yields the positive queue rate introduced earlier:
{% math %}
p(\ell_{1} \;\&\; \lambda_{1}) \;+\; p(\ell_{1} \;\&\; \lambda_{0})
  \;=\; p(\ell_{1}\mid\lambda_{1})\,\pi_{1} \;+\; p(\ell_{1}\mid\lambda_{0})\,\pi_{0}
  \;=\; p_{1}.
{% endmath %}
If, instead, we add the joint probability for class label and true class to be negative, we obtain the so-called accuracy:
{% math %}
p(\ell_{1} \;\&\; \lambda_{1}) \;+\; p(\ell_{0} \;\&\; \lambda_{0})
  \;=\; p[(\ell_{1} \;\&\; \lambda_{1}) \;\lor\; (\ell_{0} \;\&\; \lambda_{0})]
  \;=\; p(\ell \!=\! \lambda)
  \;=\; {\rm A}.
{% endmath %}
In terms of quantities introduced previously this is:
{% math %}
\begin{align}
{\rm A}  \;=\; p(\ell_{1} \;\&\; \lambda_{1}) \;+\; p(\ell_{0} \;\&\; \lambda_{0})
        &\;=\; p(\ell_{1}\mid\lambda_{1})\,\pi_{1} \;+\; p(\ell_{0}\mid\lambda_{0})\,\pi_{0}
         \;=\; S_{e}\,\pi_{1} \;+\; S_{p}\,\pi_{0}\\[1mm]
        &\;=\; p(\lambda_{1}\mid\ell_{1})\,p_{1} \;+\; p(\lambda_{0}\mid\ell_{0})\,p_{0}
         \;=\; \mathit{ppv}\,p_{1} \;+\; \mathit{npv}\,p_{0}.
\end{align}
{% endmath %}
Note the dependence on prevalence or queue rate. An equivalent measure is the **misclassification rate**, defined as one minus the accuracy. A benchmark that is sometimes used is the **null error rate**, defined as the misclassification rate of a classifier that always predicts the majority class. It is equal to {% m %}\min\{\pi_{0}, \pi_{1}\}{% em %}. The complement of the null error rate is the **null accuracy**, which is equal to {% m %}\max\{\pi_{0}, \pi_{1}\}{% em %}.

Note that null accuracy and null error rate are not necessarily good, or even reasonable benchmarks. In a highly imbalanced dataset for example, the classifier that always predicts the majority class will appear to have excellent performance even though it does not use any *relevant* information in the data.


<div style="text-align: right"><a href="#TopOfPage">Back to Top</a></div>
<a name="LikelihoodRatios"></a>
## 4. Likelihood ratios
In some fields it is customary to work with ratios of probabilities rather than with the probabilities themselves.  Thus one defines:

- The **positive likelihood ratio**: {% m %}{\rm lr+} \equiv \frac{p(\ell_{1}\mid\lambda_{1})}{p(\ell_{1}\mid\lambda_{0})} = \frac{S_{e}}{1-S_{p}}{% em %}, which represents the odds of the true class being positive if the label is positive (in medical statistics for example, this would be the odds of disease if the test result is positive).

- The **negative likelihood ratio**: {% m %}{\rm lr-} \equiv \frac{p(\ell_{0}\mid\lambda_{1})}{p(\ell_{0}\mid\lambda_{0})} = \frac{1-S_{e}}{S_{p}}{% em %}, which represents the odds of the true class being positive if the label is negative (in medical statistics, the odds of disease if the test result is negative).

Finally, it is instructive to take the ratio of these two likelihood ratios. This yields

- The **diagnostic odds ratio**: {% m %}\mathit{dor} \equiv \frac{\rm lr+}{\rm lr-} = \frac{S_{e}}{1-S_{p}}\frac{S_{p}}{1-S_{e}}{% em %}.

If {% m %}\mathit{dor} \gt 1{% em %}, the classifier is deemed useful, since this means that, in our medical example, the odds of disease are higher when the test result is positive than when it is negative. If {% m %}\mathit{dor}=1{% em %} the classifier is useless, but if {% m %}\mathit{dor}\lt 1{% em %} the classifier is worse than useless, it is misleading. In this case one would be better off swapping the labels on the classifier output.

Note that the ratios {% m %}{\rm lr+}{% em %}, {% m %}{\rm lr-}{% em %}, and {% m %}\mathit{dor}{% em %} are all independent of prevalence.

<div style="text-align: right"><a href="#TopOfPage">Back to Top</a></div>
<a name="ClassifierUsefulness"></a>
## 5. When is a classifier useful?
The usefulness condition {% m %}\fbox{$\mathit{dor} \gt 1$}{% em %} is mathematically equivalent to any of the following conditions:

1. {% m %}\fbox{$S_{e}\gt 1 - S_{p}$}{% em %} The sensitivity must be larger than one minus the specificity. Equivalently, the probability of detecting a positive-class instance must be larger than the probability of mislabeling a negative-class instance.  Reformulating in terms of Type-I and II error rates this condition becomes {% m %}\fbox{$\beta \lt 1 - \alpha$}{% em %}.

1. {% m %}\fbox{$S_{e} \gt p_{1}$}{% em %} The probability of encountering a positively labeled instance must be larger within the subset of positive-class instances than within the entire population. Similarly: {% m %}\fbox{$S_{p} \gt p_{0}$}{% em %}, {% m %}\fbox{$\alpha \lt p_{1}$}{% em %}, and {% m %}\fbox{$\beta \lt p_{0}$}{% em %}.

1. {% m %}\fbox{$\mathit{ppv} \gt \pi_{1}$}{% em %} The precision must be larger than the prevalence. In other words, the probability for an instance to belong to the positive class must be larger within the subset of instances with a positive label than within the entire population. If this is not the case, the classifier adds no useful information. Similarly: {% m %}\fbox{$\mathit{npv} \gt \pi_{0}$}{% em %}, {% m %}\fbox{$\mathit{fdr} \lt \pi_{0}$}{% em %}, and {% m %}\fbox{$\mathit{for} \lt \pi_{1}$}{% em %}.

The classifier usefulness condition also puts a bound on the accuracy (but is *not* equivalent to it):
{% math %}
{\rm A} \;=\; S_{e}\,\pi_{1} \;+\; S_{p}\,\pi_{0}
        \;>\; p_{1}\,\pi_{1} \;+\; p_{0}\,\pi_{0}.
{% endmath %}

Since {% m %}p_{0} + p_{1} = 1{% em %}, the right-hand side of the above inequality is bounded between {% m %}\min(\pi_{0}, \pi_{1}){% em %} and {% m %}\max(\pi_{0}, \pi_{1}){% em %}. The upper bound is actually the accuracy of the majority classifier, which assigns the majority label to every instance. Note that the majority classifier itself is not a useful classifier, since its accuracy is *equal* to its corresponding bound, not *strictly higher* than it. In addition, it is entirely possible for a useful classifier to have an accuracy below that of the useless majority classifier. To illustrate this last point, consider the minority classifier, which assigns the minority label to every instance. This is a useless classifier, with accuracy {% m %}\pi_{1}{% em %} if we assume that {% m %}\pi_{1} \lt \pi_{0}{% em %}. To improve it, let's flip the label of one arbitrarily chosen majority instance. The accuracy of this new classifier is {% m %}\pi_{1} + 1/N{% em %}, with {% m %}N{% em %} the size of the total population (assumed finite in this example). This is larger than the accuracy bound, {% m %}p_{1}\pi_{1} + p_{0}\pi_{0} = \pi_{1} + 1/N - 2\pi_{1}/N{% em %}. Hence the new classifier is useful, even though for {% m %}N{% em %} large enough its accuracy will be lower than that of the majority classifier. Of course for other metrics the new classifier performs better than the majority one, for example sensitivity and the predictive values.

<div style="text-align: right"><a href="#TopOfPage">Back to Top</a></div>
<a name="ScoreBasedMetrics"></a>
## 6. Classifier scores
As mentioned earlier, classifiers usually produce a score {% m %}q{% em %} to quantify the likelihood that an instance belongs to the positive class. The performance metrics introduced so far can be defined in terms of this score, but they only need it in a binary way, through the class label {% m %}\ell{% em %}: {% m %}\ell=1{% em %} if {% m %}q\ge q_{T}{% em %}, otherwise {% m %}\ell=0{% em %}, for some predefined threshold {% m %}q_{T}{% em %}. The next two performance metrics we discuss use score values without reference to {% m %}q_{T}{% em %}. 

<a name="ROC"></a>
### 6.1 The Receiver Operating Characteristic (ROC)
The **Receiver Operating Characteristic**, or **ROC**, is defined as the curve of true positive rate versus false positive rate (or sensitivity versus one-minus-specificity). An example is shown in the figure below:
{% maincolumn "assets/img/blog/ConfusionMatrix/scores_distribution_and_roc.png" "Left: arbitrary classifier score distributions used for illustration. Right: corresponding ROC curve, obtained by varying the classifier threshold over the range of scores. In this example the area under the ROC is about 0.815." %}
Note how the main diagonal line {% m %}S_{e} = 1 - S_{p}{% em %} serves as baseline in the ROC plot, in accordance with the classifier usefulness condition {% m %}S_{e} \gt 1 - S_{p}{% em %} discussed in [section 5](#ClassifierUsefulness).

The area under the ROC curve, or **AUROC**, is a performance metric that's independent of prevalence and does not require a choice of threshold {% m %}q_{T}{% em %} on the classifier score {% m %}q{% em %}. In other words, it does not depend on the chosen operating point of the classifier. It can be shown that the AUROC is equal to the probability for a random positive instance to be scored higher than a random negative instance.

An immediate interpretation of the AUROC is that it quantifies how well the score distributions for positive and negative instances are separated. If the score distribution for positives is to the right of that for negatives, and there is no overlap, the AUROC equals 1. If the positives are all to the left of the negatives, the AUROC equals 0, and if the two distributions overlap each other exactly, the AUROC equals 0.5.

The AUROC is related to expectation values of sensitivity, specificity, and accuracy (see [appendix 11.1](#ExpectationValues) for a proof):
{% math %}
\begin{align}
\mathbb{E}\left[S_{e}\right] \;&=\; \pi_{0}\, \textrm{AUROC} \;+\; \frac{\pi_{1}}{2},\\
\mathbb{E}\left[S_{p}\right] \;&=\; \pi_{1}\, \textrm{AUROC} \;+\; \frac{\pi_{0}}{2},\\
\mathbb{E}\left[A\right]     \;&=\; 2\,\pi_{0}\,\pi_{1}\,\textrm{AUROC} \,+\, \frac{\pi_{0}^{2} + \pi_{1}^{2}}{2},
\end{align}
{% endmath %}
where the expectation values are over an ensemble of identical classifiers with operating points that are randomly drawn from the distribution of scores of the population of interest. These equations demonstrate how the AUROC aggregates classifier performance information in a way that's independent of prevalence and operating point (see also [this answer](https://www.quora.com/Machine-Learning-What-is-an-intuitive-explanation-of-AUC/answer/Peter-Flach) by Peter Flach on Quora).

<a name="PRC"></a>
### 6.2 The Precision-Recall Curve (PRC)
In cases where there is severe class imbalance in the data, the ROC may give too optimistic a view of classifier performance, because the true and false positive rates are both independent of prevalence, whereas precision deteriorates at low prevalence. It may therefore be preferable to plot a precision versus recall curve (or PRC). In the two illustrations below, the class distributions are the same, and only the class-1 prevalence is different. It can be seen that the ROC is independent of prevalence and looks quite good. On the other hand, the precision versus recall curve shows that the classifier performance leaves quite a bit to be desired in the case of imbalanced data. One can also calculate the area under the precision versus recall curve (or AUPRC) to summarize this effect.
{% maincolumn "assets/img/blog/ConfusionMatrix/scores_distribution_roc_prc_0.png" "Left: classifier score distributions in a balanced dataset. Center: ROC. Right: PRC. The areas under the ROC and the PRC are both about 0.92." %}
{% maincolumn "assets/img/blog/ConfusionMatrix/scores_distribution_roc_prc_1.png" "Left: classifier score distributions in a strongly imbalanced dataset. Center: ROC. Right: PRC. In this case the area under the ROC is still about 0.92 whereas that under the PRC is now 0.66." %}

In accordance with the classifier usefulness conditions discussed in [section 5](#ClassifierUsefulness), there are two baselines for the precision-recall curve. One is the horizontal line {% m %}\mathit{ppv} = \pi_{1}{% em %}, and the other is the curve of precision versus queue rate, since the latter is a baseline for the recall (just imagine plotting recall versus precision, where queue rate would be the natural baseline, and then flip the plot back to precision versus recall). This second baseline requires some care, however, as illustrated in the following example:
{% maincolumn "assets/img/blog/ConfusionMatrix/scores_distribution_roc_prc_2.png" "Left: classifier score distributions in an imbalanced dataset where class-0 instances extend beyond the class-1 distribution. Center: ROC. Right: PRC. The area under the ROC is ~0.94, and the area under the PRC is ~0.58." %}

It is important to understand that the curves discussed in this section don't show functional relationships, but rather operating characteristics. Consider the plot of precision versus recall for example. The definition of precision implies that, functionally, it is an increasing function of recall. This is easy to see:
{% math %}
\mathit{ppv} \;=\; \frac{S_{e}\, \pi_{1}}{p_{1}}
             \;=\; \frac{S_{e}\, \pi_{1}}{S_{e}\,\pi_{1} \,+\, \alpha\,\pi_{0}}
             \;=\; \frac{\pi_{1}}{\pi_{1} \,+\, \alpha\,\pi_{0}\left/S_{e}\right.}
{% endmath %}
And yet, the plot shows a decreasing function! This is because each point on the plotted curves represents an operating point, i.e., a choice of classifier threshold, and this choice affects not only {% m %}S_{e}{% em %}, but also {% m %}\alpha{% em %}. If we just wanted to show how {% m %}\mathit{ppv}{% em %} depends on {% m %}S_{e}{% em %}, we would keep {% m %}\alpha{% em %} constant. In fact, what the plot shows is the joint variation of precision and recall as the threshold is changed.


<div style="text-align: right"><a href="#TopOfPage">Back to Top</a></div>
<a name="OperatingPoints"></a>
## 7. Classifier operating points
The area under the ROC or under the PRC is a good performance measure when one wishes to decide between different types of classification algorithms (e.g., random forest versus na&iuml;ve Bayes versus support vector machine), because this area is independent of the actual classifier threshold. Once an algorithm has been selected however, one needs to choose an operating point, that is, a threshold at which to operate the classifier. This is not trivial. If, for example, one is mostly interested in precision, choosing the threshold that maximizes precision will typically result in zero queue rate. Similarly, maximizing recall typically leads to 100% queue rate (see the examples in the section on [trends](#TrendsAndBaselines) below). The best approach is to find the threshold that maximizes a balanced combination of metrics, such as precision and recall, or true and false positive rates. The following two options are common:

1. The {% m %}F_{\beta}{% em %} score, which is a weighted harmonic mean of precision and recall: {% m %}F_{\beta} \equiv \frac{(1+\beta^{2})\,\times\,\mathit{ppv}\,\times\, S_{e}}{\beta^{2}\,\times\,\mathit{ppv}\, +\, S_{e}} {% em %}. Usually {% m %}\beta{% em %} is set to 1, which yields the regular harmonic mean of precision and recall.

1. Youden's {% m %}J{%em %} statistic, which equals the true positive rate minus the false positive rate: {% m %}J = S_{e} - \alpha{% em %}.

In order to avoid bias, the maximization of {% m %}F_{\beta}{% em %} or {% m %}J{% em %} as a function of threshold should be done on a test dataset that's different from the one used to evaluate the resulting classifier's performance.

<div style="text-align: right"><a href="#TopOfPage">Back to Top</a></div>
<a name="MetricsEstimation"></a>
## 8. Performance metric estimation
The performance metrics can be estimated from a two-by-two contingency table known as the confusion matrix. It is obtained by applying the classifier to a test data set different from the training data set. Here is standard notation for this matrix:

{% fullwidth "assets/img/blog/ConfusionMatrix/ConfusionMatrix.png" 'Figure 2: Estimated confusion matrix. The notation tp stands for "number of true positives", fn for "number of false negatives", and so on.' %}

The rows correspond to class labels, the columns to true classes. We then have the following approximations:
{% math %}
\begin{aligned}
\mbox{Prevalence :} & \quad \pi_{1} &&\equiv\pi(\lambda_{1}) &&\approx \dfrac{\rm fn+tp}{\rm fn+tp+fp+tn}\\[1mm]

\mbox{Queue rate :} & \quad p_{1} &&\equiv p(\ell_{1}) &&\approx \dfrac{\rm fp+tp}{\rm fn+tp+fp+tn}\\[4mm]

\mbox{True-positive rate, sensitivity, recall :} & \quad S_{e} &&\equiv p(\ell_{1}\mid\lambda_{1}) &&\approx \dfrac{\rm tp}{\rm tp+fn}\\[1mm]

\mbox{True-negative rate, specificity :} & \quad S_{p} &&\equiv p(\ell_{0}\mid\lambda_{0}) &&\approx \dfrac{\rm tn}{\rm tn + fp}\\[1mm]

\mbox{False-positive rate, Type-I error rate :} & \quad \alpha &&\equiv p(\ell_{1}\mid\lambda_{0})&&\approx \dfrac{\rm fp}{\rm tn + fp}\\[1mm]

\mbox{False-negative rate, Type-II error rate :} & \quad \beta &&\equiv p(\ell_{0}\mid\lambda_{1})&&\approx \dfrac{\rm fn}{\rm tp + fn}\\[4mm]

\mbox{Positive predictive value, precision :} & \quad \mathit{ppv} &&\equiv p(\lambda_{1}\mid\ell_{1}) &&\approx \dfrac{\rm tp}{\rm tp + fp}\\[1mm]

\mbox{Negative predictive value :} & \quad \mathit{npv} &&\equiv p(\lambda_{0}\mid\ell_{0}) &&\approx \dfrac{\rm tn}{\rm tn + fn}\\[1mm]

\mbox{False-discovery rate :} & \quad \mathit{fdr} &&\equiv p(\lambda_{0}\mid\ell_{1}) &&\approx \dfrac{\rm fp}{\rm tp + fp}\\[1mm]

\mbox{False-omission rate :} & \quad \mathit{for} &&\equiv p(\lambda_{1}\mid\ell_{0}) &&\approx \dfrac{\rm fn}{\rm tn + fn}\\[4mm]

\mbox{Positive likelihood ratio :} & \quad {\rm lr+} &&\equiv \frac{p(\ell_{1}\mid\lambda_{1})}{p(\ell_{1}\mid\lambda_{0})}&&\approx \frac{\rm tp}{\rm tp+fn}\; \frac{\rm tn+fp}{\rm fp}\\[1mm]

\mbox{Negative likelihood ratio :} & \quad {\rm lr-} &&\equiv \frac{p(\ell_{0}\mid\lambda_{1})}{p(\ell_{0}\mid\lambda_{0})}&&\approx \frac{\rm fn}{\rm tp+fn}\; \frac{\rm tn+fp}{\rm tn}\\[1mm]

\mbox{Diagnostic odds ratio :} & \quad \mathit{dor} &&\equiv \frac{\rm lr+}{\rm lr-} &&\approx \frac{\rm tp}{\rm fp} \frac{\rm tn}{\rm fn}\\[4mm]

\mbox{Accuracy :} & \quad A &&\equiv p(\ell=\lambda) &&\approx \dfrac{\rm tp + tn}{\rm tp + tn + fp + fn}
\end{aligned}
{% endmath %}
There are no simple expressions for the AUROC and AUPRC, since these require integration under the ROC and PRC, respectively.

Note that the relations between probabilities derived in the previous sections also hold between their estimates.

<div style="text-align: right"><a href="#TopOfPage">Back to Top</a></div>
<a name="TrendsAndBaselines"></a>
## 9. Trends and baselines
For classifiers that generate scores it is useful to understand how various metrics change as the classifier threshold on the score is increased. This will of course depend on the actual score distributions of true class-0 and true class-1 instances. In the following four illustrations we keep these two distributions identical: one a Gaussian with mean 0.25 and width 0.25, and the other a Gaussian with mean 0.75 and width 0.25. What changes is the sample composition: balanced or imbalanced, high- or low-statistics; and which class is represented by which Gaussian.

Case 1: Balanced, high-statistics dataset, with class-1 scores mostly above class-0 scores

{% maincolumn "assets/img/blog/ConfusionMatrix/trends_5000_5000_01.png" "The recall and queue rate both decrease, whereas the precision increases. As noted previously, if the classifier threshold is chosen such that queue rate equals prevalence, precision will equal recall." %}

Case 2: Balanced, high-statistics dataset, with class-1 scores mostly *below* class-0 scores

{% maincolumn "assets/img/blog/ConfusionMatrix/trends_5000_5000_10.png" "Compared to Case 1 the recall is still decreasing with threshold, but the precision is now decreasing instead of increasing. The baselines are now <em>above</em> their corresponding metrics, indicating that the classifier is misleading." %}

Case 3: Balanced, low-statistics dataset, with class-1 scores mostly above class-0 scores

{% maincolumn "assets/img/blog/ConfusionMatrix/trends_0020_0020_01.png" "The curves are essentially the same as for Case 1, but with pronounced statistical fluctuations. Note that the recall never fluctuates up: it stays even or decreases. In contrast, the precision fluctuates both up and down." %}

Case 4: Imbalanced, high-statistics dataset, with class-1 scores mostly above class-0 scores

{% maincolumn "assets/img/blog/ConfusionMatrix/trends_9500_0500_01.png" "Compared to Case 1, the recall hasn't changed, but its baseline, the queue rate, has shifted down as a whole. The precision has also shifted down." %}

See appendix [11.2](#MetricsVsThreshold) for a more detailed explanation of the above trends.

<div style="text-align: right"><a href="#TopOfPage">Back to Top</a></div>
<a name="EffectOfPrevalence"></a>
## 10. Effect of prevalence on classifier training and testing
In this post I have tried to be careful in pointing out which performance metrics depend on the prevalence, and which don't. The reason is that prevalence is a population or sample property, whereas here we are interested in classifier properties. Of course, real-world classifiers must be trained and tested on finite samples, and this has implications for the actual effect of prevalence on classifier performance:

- When **training** a classifier, changing the prevalence in the training data *may affect* properties such as the sensitivity and specificity, depending on the characteristics of the training algorithm.

- When **testing** a trained classifier, changing the prevalence in the test data *will not affect* properties such as the sensitivity and specificity (within statistical fluctuations).

To illustrate this point I went back to a [previous post]({% post_url 2016-07-06-RandomForestsWiMLDS %}) on predicting flight delays with a random forest. The prevalence of the given flight delay data set is about 20%, but can be increased by randomly dropping class 0 instances (non-delayed flights). The table below shows the effect of changing the prevalence in this way in the *training* data:
<br/><br/>

| Training set prevalence | Sensitivity | Specificity | Accuracy | AUROC |
|:-----------------------:|:-----------:|:-----------:|:--------:|:-----:|
| 0.40                    | 0.16        | 0.96        | 0.72     | 0.72  |
| 0.42                    | 0.29        | 0.91        | 0.72     | 0.72  |
| 0.44                    | 0.46        | 0.82        | 0.71     | 0.72  |
| 0.46                    | 0.58        | 0.72        | 0.68     | 0.71  |
| 0.48                    | 0.63        | 0.67        | 0.66     | 0.72  |
| 0.50                    | 0.73        | 0.57        | 0.62     | 0.71  |

To be clear, for each row it is the same classifier (same hyper-parameters: number of trees, maximum tree depth, etc.) that is being fit to a training data set with varying prevalence. The sensitivity, specificity, accuracy, and AUROC are then measured on a testing data set with a constant prevalence of 30%. Note how the sensitivity and specificity, *as measured on an independent test data set*, vary with training set prevalence. The AUROC, on the other hand, remains stable.

Next, let's look at the effect of changing the prevalence in the *testing* data:
<br/><br/>

| Testing set prevalence | Sensitivity | Specificity | Accuracy | AUROC |
|:----------------------:|:-----------:|:-----------:|:--------:|:-----:|
| 0.20                   | 0.50        | 0.78        | 0.73     | 0.72  |
| 0.30                   | 0.48        | 0.80        | 0.70     | 0.72  |
| 0.40                   | 0.45        | 0.82        | 0.67     | 0.72  |
| 0.50                   | 0.46        | 0.80        | 0.63     | 0.71  |
| 0.60                   | 0.50        | 0.79        | 0.61     | 0.71  |

Here the prevalence in the training data was set at 0.45. The sensitivity and specificity appear much more stable; residual variations may be due to the effect on the training data set of the procedure used to vary the testing set prevalence.

Training data sets are often unbalanced, with prevalences even lower than in the example discussed here. Modules such as scikit-learn's [`RandomForestClassifier`](https://scikit-learn.org/stable/modules/generated/sklearn.ensemble.RandomForestClassifier.html) provide a `class_weight` option, which can be set to `balanced` or `balanced_subsample` in order to assign a larger weight to the minority class. This re-weighting is applied both to the GINI criterion used for finding splits at decision tree nodes and to class votes in the terminal nodes. The sensitivity and specificity of a classifier, as well as its accuracy, will depend on the setting of the `class_weight` option. For scoring purposes when optimizing a classifier, AUROC tends to be a much more robust property.
<div style="text-align: right"><a href="#TopOfPage">Back to Top</a></div>

<a name="Appendix"></a>
## 11. Mathematical Appendix
This appendix contains mathematical proofs of some of the statements made in the body of this post.

<div style="text-align: right"><a href="#TopOfPage">Back to Top</a></div>
<a name="ExpectationValues"></a>
### 11.1 Expectation values of sensitivity, specificity, and accuracy
To see how the AUROC is related to an expectation value of the sensitivity, note that for a given threshold {% m %} q_{T} {% em %} on the classifier output score, we can write the sensitivity as:
{% math %}
S_{e}(q_{T}) \;=\; \mathbb{P}\left[Q_{1}\ge q_{T}\right],
{% endmath %}
where {% m %}Q_{1}{% em %} is a *random variable distributed as the classifier scores of positive instances*. Let us now compute the expectation value of {% m %} S_{e}(q_{T}) {% em %} where {% m %} q_{T} {% em %} is varied over the distribution of classifier scores of all instances in the population of interest, class 0 and class 1. This score distribution can be modeled by a random variable {% m %} Q^{\prime} {% em %} defined as follows:
{% math %}
Q^{\prime} \;\equiv\; \mathbb{1}(U \le \pi_{0})\, Q^{\prime}_{0} \;+\;
                      \mathbb{1}(U \gt \pi_{0})\, Q^{\prime}_{1},
{% endmath %}
where {% m %}\mathbb{1}(A){% em %} is the indicator function of set {% m %}A{% em %}, and {% m %} U {% em %}, {% m %} Q^{\prime}_{0} {% em %} and {% m %} Q^{\prime}_{1} {% em %} are independent random variables, with {% m %} U {% em %} being uniform between 0 and 1, and {% m %} Q^{\prime}_{0} {% em %}, {% m %} Q^{\prime}_{1} {% em %} following the score distributions of class-0, resp. class-1 instances. Extending the definition of {% m %}S_{e}{% em %} to a function of a random threshold:
{% math %}
S_{e}(Q^{\prime}) \;\equiv\; \mathbb{P}\left[Q_{1} \ge Q^{\prime}\mid Q^{\prime}\right]
                  \; = \; \mathbb{E}\left[\mathbb{1}(Q_{1} \ge Q^{\prime}) \mid Q^{\prime}\right],
{% endmath %}
we then have:
{% math %}
\begin{align}
\mathbb{E}\left[S_{e}\right]
\;&\equiv\; \mathbb{E}\left[ \;\mathbb{E}\left[ \mathbb{1}(Q_{1} \ge Q^{\prime})\mid Q^{\prime} \right]\; \right]\\[1mm]
\;&=\; \mathbb{E}\left[ \mathbb{1}(Q_{1} \ge Q^{\prime})\right]\quad\textrm{by the law of total expectation,}\\[1mm]
\;&=\; \mathbb{E}\left[ \mathbb{1}(U \le \pi_{0})\;\mathbb{1}(Q_{1} \ge Q^{\prime}_{0}) \;+\; \mathbb{1}(U > \pi_{0})\;\mathbb{1}(Q_{1} \ge Q^{\prime}_{1}) \right]\quad\textrm{by definition of $Q^{\prime}$,}\\[1mm]
\;&=\; \mathbb{E}\left[ \mathbb{1}(U \le \pi_{0})\;\mathbb{1}(Q_{1} \ge Q^{\prime}_{0})\right] \;+\; \mathbb{E}\left[ \mathbb{1}(U > \pi_{0})\;\mathbb{1}(Q_{1} \ge Q^{\prime}_{1}) \right]\quad\textrm{by linearity of the expectation operator,}\\[1mm]
\;&=\; \mathbb{E}\left[ \mathbb{1}(U \le \pi_{0})\right] \; \mathbb{E}\left[ \mathbb{1}(Q_{1} \ge Q^{\prime}_{0})\right] \;+\; \mathbb{E}\left[ \mathbb{1}(U > \pi_{0})\right] \; \mathbb{E}\left[ \mathbb{1}(Q_{1} \ge Q^{\prime}_{1}) \right]\quad\textrm{by independence of $U$, $Q_{1}$, $Q_{0}^{\prime}$, $Q_{1}^{\prime}$,}\\[1mm]
\;&=\; \pi_{0}\,\mathbb{P}\left(Q_{1} \ge Q^{\prime}_{0}\right) \;+\; \pi_{1}\,\mathbb{P}\left(Q_{1} \ge Q^{\prime}_{1}\right)\quad\textrm{by definition of $\mathbb{P}$ in terms of $\mathbb{E}$,}\\[1mm]
\;&=\; \pi_{0}\, \textrm{AUROC} \;+\; \frac{\pi_{1}}{2}\quad\textrm{by definition of AUROC.}
\end{align}
{% endmath %}
One can similarly show that:
{% math %}
\mathbb{E}\left[S_{p}\right] \;=\; \pi_{1}\, \textrm{AUROC} \;+\; \frac{\pi_{0}}{2}.
{% endmath %}
Using the expression for the accuracy A in terms of {% m %}S_{e}{% em %} and {% m %}S_{p}{% em %},
and the linearity of the expectation operator, we then find:
{% math %}
\mathbb{E}\left[A\right] \;=\; 2\,\pi_{0}\,\pi_{1}\,\textrm{AUROC} \,+\, \frac{\pi_{0}^{2} + \pi_{1}^{2}}{2}.
{% endmath %}

<div style="text-align: right"><a href="#TopOfPage">Back to Top</a></div>
<a name="MetricsVsThreshold"></a>
### 11.2 Performance metrics versus threshold
To understand the behavior of the performance metrics in section [9](#TrendsAndBaselines), consider what happens when we increase the classifier threshold {% m %}t{% em %} by an amount {% m %}\Delta t \gt 0{% em %}. A number {% m %}\Delta p + \Delta n{% em %} of instances will see their labels flip from 1 to 0, where {% m %}\Delta p{% em %} are class-1 and {% m %}\Delta n{% em %} are class-0. The effect on the confusion matrix elements is:
{% math %}
\begin{align*}
{\rm tp} &\quad\longrightarrow\quad {\rm tp} - \Delta p\\
{\rm fn} &\quad\longrightarrow\quad {\rm fn} + \Delta p\\
{\rm tn} &\quad\longrightarrow\quad {\rm tn} + \Delta n\\
{\rm fp} &\quad\longrightarrow\quad {\rm fp} - \Delta n
\end{align*}
{% endmath %}
Applying these transformations to the metrics in section [8](#MetricsEstimation) yields the following results.

For the queue rates:
{% math %}
\begin{align*}
p_{1} &= \frac{\rm tp+fp}{\rm tn+fn+tp+fp} \quad\longrightarrow\quad p_{1} \;-\; \frac{\Delta p + \Delta n}{\rm tn+fn+tp+fp}\\
p_{0} &= \frac{\rm tn+fn}{\rm tn+fn+tp+fp} \quad\longrightarrow\quad p_{0} \;+\; \frac{\Delta p + \Delta n}{\rm tn+fn+tp+fp}
\end{align*}
{% endmath %}

For the likelihoods:
{% math %}
\begin{align*}
S_{e} &= \frac{\rm tp}{\rm tp+fn} \quad\longrightarrow\quad S_{e}\;-\; \frac{\Delta p}{\rm tp + fn}\\
\beta &= \frac{\rm fn}{\rm tp+fn} \quad\longrightarrow\quad \beta\;+\; \frac{\Delta p}{\rm tp + fn}\\
S_{p} &= \frac{\rm tn}{\rm tn+fp} \quad\longrightarrow\quad S_{p}\;+\; \frac{\Delta n}{\rm tn + fp}\\
\alpha &= \frac{\rm fp}{\rm tn+fp} \quad\longrightarrow\quad \alpha\;-\; \frac{\Delta n}{\rm tn + fp}
\end{align*}
{% endmath %}

Note in particular how the recall {% m %}S_{e}{% em %} and class-1 queue rate {% m %}p_{1}{% em %} *always* decrease when the threshold is increased, explaining the behavior in the graphs. However the change in predictive values is not linear. For the class-1 precision for example, we have:
{% math %}
\mathit{ppv} = \frac{\rm tp}{\rm tp+fp} \quad\longrightarrow\quad \frac{ {\rm tp} - \Delta p}{ {\rm tp} - \Delta p + {\rm fp} - \Delta n},
{% endmath %}
so that {% m %}\mathit{ppv}{% em %} will increase if and only if:
{% math %}
\frac{\Delta p}{\rm tp} \; <\; \frac{\Delta n}{\rm fp},
{% endmath %}
i.e., if the fractional loss of true positives is smaller than the fractional loss of false positives. This inequality can be violated "locally" by statistical fluctuations (Case 3 in section [9](#TrendsAndBaselines)), or "globally", when the class-0 score distribution is on the wrong side of the class-1 score distribution (Case 2 in section [9](#TrendsAndBaselines)).