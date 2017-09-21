---
layout: full-width-post
title: "Taking the Confusion out of the Confusion Matrix"
date: 26 July 2016
excerpt: "How Bayes' theorem helps organize the bewildering array of performance metrics that can be estimated from a classifier's confusion matrix."
comments: true
---
[Revised 21 September 2017]

> **Contents**
> <br>[Introduction](#Introduction)
> <br>[Bayes' theorem](#FourApplications)
> <br> &ensp;&ensp;&ensp;[The prior](#Prior)
> <br> &ensp;&ensp;&ensp;[The likelihood](#Likelihood)
> <br> &ensp;&ensp;&ensp;[The model evidence](#ModelEvidence)
> <br> &ensp;&ensp;&ensp;[The posterior](#Posterior)
> <br> &ensp;&ensp;&ensp;[Summary](#Summary)
> <br>[Joint probabilities](#JointProbabilities)
> <br>[Likelihood ratios](#LikelihoodRatios)
> <br>[When is a classifier useful?](#ClassifierUsefulness)
> <br>[Performance metric estimation](#MetricsEstimation)
> <br>[Effect of prevalence on classifier training and testing](#EffectOfPrevalence)

---

<a name="Introduction"></a>
## Introduction
I always thought that the *confusion* matrix was rather aptly named, a reference not so much to the mixed performance of a classifier as to my own bewilderment at the number of measures of that performance. Recently however, I encountered a brief mention of the possibility of a Bayesian interpretation of performance measures, and this helped clarify things in my mind. I'd like to develop the idea a little further in this post. It's not that Bayes' theorem is needed to understand or apply the performance measures, but it acts as an organizing and connecting principle, which I find helpful. New concepts are easier to remember if they fit inside a good story.

Another advantage of taking the Bayesian route is that this forces us to view performance measures as probabilities, which are *estimated* from the confusion matrix. Elementary presentations tend to *define* performance metrics in terms of ratios of confusion matrix elements, thereby ignoring the effect of statistical fluctuations.

Bayes' theorem is not the only way to generate performance metrics. One can also start from joint probabilities or likelihood ratios. The next three subsections describe these approaches one by one. This is followed by a subsection discussing a general condition for a classifier to be useful, a subsection on estimating performance measures, and one on the effect of changing the prevalence in the training and testing data sets of a classifier.

<div style="text-align: right"><a href="#TopOfPage">Back to Top</a></div>
<a name="FourApplications"></a>
## Bayes' theorem
Let's start from Bayes' theorem in its general form:
{% math %}
p(\lambda\mid X) \;=\; \frac{p(X\mid\lambda)\;\pi(\lambda)}{\int p(X\mid\lambda^{\prime})\;\pi(\lambda^{\prime})\;d\lambda^{\prime}},
{% endmath %}
where {% m %}X{% em %} represents the observed data and {% m %}\lambda{% em %} a parameter of interest. On the left-hand side is the posterior probability density of {% m %}\lambda{% em %} given {% m %}X{% em %}. On the right-hand side, {% m %}p(X\mid\lambda){% em %} is the likelihood, or conditional probability density of {% m %}X{% em %} given {% m %}\lambda{% em %}, and {% m %}\pi(\lambda){% em %} is the prior probability density of {% m %}\lambda{% em %}. The denominator is called model evidence, or marginal likelihood. One way to think about Bayes' theorem is that it uses the data {% m %}X{% em %} to update the prior information {% m %}\pi(\lambda){% em %} about {% m %}\lambda{% em %}, and returns the posterior {% m %}p(\lambda\mid X){% em %}.

For a binary classifier {% m %}\lambda{% em %} is the true class to which instance {% m %}X{% em %} belongs and can take only two values, say 0 and 1. Bayes' theorem then simplifies to:
{% math %}
\begin{aligned}
p(\lambda \!=\!1\mid X) & \; =\; \frac{p(X\mid\lambda\!=\!1)\;\pi(\lambda\!=\!1)}
     {p(X\mid\lambda\!=\!0)\,\pi(\lambda\!=\!0) \;+\;
      p(X\mid\lambda\!=\!1)\,\pi(\lambda\!=\!1)},
\quad\textrm{and} \\[3mm]
p(\lambda \!=\!0\mid X) & \; =\; 1 \;-\; p(\lambda\!=\!1\mid X).
\end{aligned}
{% endmath %}
To describe the performance of a classifier, {% m %}X{% em %} could be summarized either by the *class label* {% m %}\ell{% em %} assigned to it by the classifier, or by the *score* {% m %}q{% em %} (a number between 0 and 1) assigned by the classifier to the hypothesis that {% m %}X{% em %} belongs to class 1. Thus for example, {% m %} p(\lambda\!=\!1 \mid \ell\!=\!0) {% em %} represents the posterior probability that the true class is 1 given a class label of 0, and {% m %}p(\lambda\!=\!1\mid q\!\ge\! q_{T}){% em %} is the posterior probability that the true class is 1 given that the score {% m %}q{% em %} is above a pre-specified threshold {% m %}q_{T}{% em %}.

<div style="text-align: right"><a href="#TopOfPage">Back to Top</a></div>
<a name="Prior"></a>
### The prior
To simplify the notation I'll write {% m %}\lambda_{0}{% em %} to signify {% m %}\lambda=0{% em %}, {% m %}\lambda_{1}{% em %} for {% m %}\lambda=1{% em %}, and similarly with other variables. The first ingredient in the computation of the posterior is the prior {% m %}\pi(\lambda){% em %}. To fix ideas, let's assume that {% m %}\lambda_{1}{% em %} is the class of interest, generically labeled "positive"; it indicates "effect", "signal", "disease", "fraud", or "spam", depending on the context. Then {% m %}\lambda_{0}{% em %} indicates the lack of these things and is generically labeled "negative". To fully specify the prior we just need {% m %}\pi(\lambda_{1}){% em %}, since {% m %}\pi(\lambda_{0}) = 1 - \pi(\lambda_{1}){% em %}. The prior probability {% m %}\pi(\lambda_{1}){% em %} of drawing a positive instance from the population is called the **prevalence**. It is a property of the population, not the classifier.

To simplify the notation even further I'll write {% m %}\pi_{1}{% em %} for {% m %}\pi(\lambda=1){% em %} and {% m %}\pi_{0}{% em %} for {% m %}\pi(\lambda=0){% em %}.

<div style="text-align: right"><a href="#TopOfPage">Back to Top</a></div>
<a name="Likelihood"></a>
### The likelihood
The next ingredient is the conditional probability {% m %}p(X\mid\lambda){% em %}. For now let's work with classifier labels and replace {% m %}X{% em %} by {% m %}\ell{% em %}. This leads to the following four definitions:

- The **true positive rate**, also known as **sensitivity** or **recall**: {% m %}S_{e} \equiv p(\ell_{1}\mid \lambda_{1}){% em %},

- The **true negative rate**, also known as **specificity**: {% m %}S_{p}\equiv p(\ell_{0}\mid \lambda_{0}){% em %},

- The **false-positive or Type-I-error rate**: {% m %}\alpha\equiv p(\ell_{1}\mid\lambda_{0}) = 1 - p(\ell_{0}\mid\lambda_{0}){% em %}, and

- The **false-negative or Type-II-error rate**: {% m %}\beta\equiv p(\ell_{0}\mid\lambda_{1}) = 1 - p(\ell_{1}\mid\lambda_{1}){% em %}.

For example, {% m %}p(\ell_{1}\mid \lambda_{0}){% em %} is the conditional probability that the classifier assigns a label of 1 to an instance actually belonging to class 0. When viewed as a function of true class {% m %}\lambda{% em %}, for fixed label {% m %}\ell{% em %}, the conditional probability is called **likelihood**. Note that the above four rates are independent of the prevalence and are therefore intrinsic properties of the classifier.  

<div style="text-align: right"><a href="#TopOfPage">Back to Top</a></div>
<a name="ModelEvidence"></a>
### The model evidence
Finally, we need the normalization factor in Bayes' theorem, the model evidence. For a binary classifier this has two components, corresponding to the two possible labels:

- The **positive labeling rate**: {% m %}p(\ell_{1}) = p(\ell_{1}\mid\lambda_{0})\,\pi_{0} + p(\ell_{1}\mid\lambda_{1})\,\pi_{1}{% em %}, and

- The **negative labeling rate**: {% m %}p(\ell_{0}) = p(\ell_{0}\mid\lambda_{0})\,\pi_{0} + p(\ell_{0}\mid\lambda_{1})\,\pi_{1}{% em %}.

The positive labeling rate is more commonly known as the **queue rate**, especially when it is calculated as the fraction of instances for which the classifier score {% m %}q{% em %} is above a pre-specified threshold {% m %}q_{T}{% em %}.

Again to simplify the notation I'll write {% m %}p_{0}{% em %} for {% m %}p(\ell_{0}){% em %} and {% m %}p_{1}{% em %} for {% m %}p(\ell_{1}){% em %} in the following.

<div style="text-align: right"><a href="#TopOfPage">Back to Top</a></div>
<a name="Posterior"></a>
### The posterior
Armed with the prior probability, the conditional probability, and the model evidence, we can compute the posterior probability from Bayes' theorem.  There are four combinations of truth and label:

- The **positive predictive value**, also known as **precision**: {% m %}{\rm ppv}\equiv p(\lambda_{1}\mid\ell_{1}) = \frac{p(\ell_{1}\mid\lambda_{1})\, \pi_{1}}{p_{1}} {% em %},

- The **negative predictive value**: {% m %} {\rm npv}\equiv p(\lambda_{0}\mid\ell_{0}) = \frac{p(\ell_{0}\mid\lambda_{0})\, \pi_{0}}{p_{0}} {% em %},

- The **false discovery rate**: {% m %} {\rm fdr}\equiv p(\lambda_{0}\mid\ell_{1}) = 1 - {\rm ppv}{% em %}, and

- The **false omission rate**: {% m %} {\rm for}\equiv p(\lambda_{1}\mid\ell_{0}) = 1 - {\rm npv} {% em %}.

The precision, for example, quantifies the predictive value of a positive label by answering the question: If we see a positive label, what is the (posterior) probability that the true class is positive? (Note that the predictive values are not intrinsic properties of the classifier since they depend on the prevalence. Sometimes they are "standardized" by evaluating them at {% m %} \pi_{0} = \pi_{1} = 1/2 {% em %}.)

<div style="text-align: right"><a href="#TopOfPage">Back to Top</a></div>
<a name="Summary"></a>
### Summary
Table 1 shows how the four quantities {% m %}{\rm ppv}{% em %}, {% m %}{\rm npv}{% em %}, {% m %}{\rm fdr}{% em %} and {% m %}{\rm for}{% em %} are related to the quantities {% m %}S_{e}{% em %}, {% m %}S_{p}{% em %}, {% m %}\alpha{% em %} and {% m %}\beta{% em %} via Bayes' theorem.

{% fullwidth "assets/img/blog/ConfusionMatrix/FourBayesTheorems.png" "Figure 1: Four applications of Bayes' theorem to the performance metrics of a classifier. Each application connects an observed value (the label, indicated by the corresponding row header) to a true value (the class, indicated by the corresponding column header)." %}

Note that only two of these four theorems are independent, since {% m %}{\rm npv} + {\rm for} = {\rm fdr} + {\rm ppv} = 1{% em %}. Five more identities connect the twelve quantities
{% math %}
\pi_{0},\; \pi_{1},\; p_{0},\; p_{1},\; \alpha,\; \beta,\; S_{e},\; S_{p},\; {\rm fdr},\; {\rm for},\; {\rm ppv},\; {\rm npv},
{% endmath %}
namely:
{% math %}
\alpha + S_{p} = 1,\quad \beta + S_{e} = 1,\quad \pi_{0} + \pi_{1} = 1,\quad p_{0} + p_{1} = 1,\quad p_{1} = (1-S_{p}) (1-\pi_{1}) + S_{e} \pi_{1}.
{% endmath %}
Altogether this yields nine equations among twelve variables, leaving only three independent ones, for example {% m %}S_{e}{% em %}, {% m %}S_{p}{% em %}, and {% m %} \pi_{1}{% em %}. The importance of this will become clear [further down](#MetricsEstimation), when we'll see how to *estimate* all the quantities discussed in this section from a two-by-two contingency table, which also has three degrees of freedom.

<div style="text-align: right"><a href="#TopOfPage">Back to Top</a></div>
<a name="JointProbabilities"></a>
## Joint probabilities
A couple of joint probabilities lead to performance metrics with very different properties.

Consider first the joint probability of class label and true class, which can be decomposed in terms of likelihood and prior:
{% math %}
p(\ell,\lambda) \;=\; p(\ell\mid\lambda)\,\pi(\lambda).
{% endmath %}
Projecting this joint probability on the axis "{% m %} \ell=\lambda {% em %}" yields the **accuracy**, or the probability to correctly identify any instance, negative or positive:
{% math %}
{\rm A} \;\equiv\; p(\ell=\lambda) \;=\; p(\ell_{0}\mid\lambda_{0})\,\pi_{0} \;+\; p(\ell_{1}\mid\lambda_{1})\,\pi_{1} \;=\; S_{p}\,(1-\pi_{1}) \;+\; S_{e}\,\pi_{1}.
{% endmath %}
The expression in terms of sensitivity and specificity shows that the accuracy depends on the prevalence and is therefore not an intrinsic property of the classifier. An equivalent measure is the **misclassification rate**, defined as one minus the accuracy. A benchmark that is sometimes used is the **null error rate**, defined as the misclassification rate of a classifier that always predicts the majority class. It is equal to {% m %}\min\{\pi_{1}, 1-\pi_{1}\}{% em %}. The complement of the null error rate is the **null accuracy**, which is equal to {% m %}\max\{\pi_{1}, 1-\pi_{1}\}{% em %}.

One way to avoid dependence on the prevalence is to independently draw an instance from a population with positive true class and an instance from a population with negative true class. Consider the joint probability of the scores assigned by the classifier to such a pair of instances. A good measure of performance is the probability that the score assigned to the positive instance is larger than the score assigned to the negative instance. It can be shown that this is actually equal to the **Area Under the Receiver Operating Characteristic (AUROC)**. The Receiver Operating Characteristic (**ROC**) is a curve of the true positive rate as a function of the false positive rate (or sensitivity versus one-minus-specificity). In addition to being independent of prevalence, the AUROC does not require a choice of threshold on the classifier score {% m %}q{% em %}.

<div style="text-align: right"><a href="#TopOfPage">Back to Top</a></div>
<a name="LikelihoodRatios"></a>
## Likelihood ratios
In some fields it is customary to work with ratios of probabilities rather than with the probabilities themselves.  Thus one defines:

- The **positive likelihood ratio**: {% m %}{\rm lr+} \equiv \frac{p(\ell_{1}\mid\lambda_{1})}{p(\ell_{0}\mid\lambda_{1})} = \frac{S_{e}}{1-S_{e}}{% em %}, which represents the odds of a positive label if the true class is positive (in medical statistics for example, this would be the odds of a positive test if patient has the disease).

- The **negative likelihood ratio**: {% m %}{\rm lr-} \equiv \frac{p(\ell_{1}\mid\lambda_{0})}{p(\ell_{0}\mid\lambda_{0})} = \frac{1-S_{p}}{S_{p}}{% em %}, which represents the odds of a positive label if the true class is negative (in medical statistics, the odds of a positive test if patient does not have the disease).

Finally, it is instructive to take the ratio of these two likelihood ratios. This yields

- The **diagnostic odds ratio**: {% m %}{\rm dor} \equiv \frac{\rm lr+}{\rm lr-} = \frac{S_{e}}{1-S_{e}}\frac{S_{p}}{1-S_{p}}{% em %}.

If {% m %}{\rm dor} \gt 1{% em %}, the classifier is deemed useful, since this means that, in our medical example, the odds of a positive test are higher for a patient with the disease than for a healthy patient. If {% m %}{\rm dor}=1{% em %} the classifier is useless, but if {% m %}{\rm dor}\lt 1{% em %} the classifier is worse than useless, it is misleading. In this case one would be better off swapping the labels on the classifier output.

Note that the ratios {% m %}{\rm lr+}{% em %}, {% m %}{\rm lr-}{% em %}, and {% m %}{\rm dor}{% em %} are all independent of prevalence.

<div style="text-align: right"><a href="#TopOfPage">Back to Top</a></div>
<a name="ClassifierUsefulness"></a>
## When is a classifier useful?
The usefulness condition {% m %}\fbox{dor $\gt 1$}{% em %} is mathematically equivalent to any of the following conditions:

1. {% m %}\fbox{$S_{e}\gt 1 - S_{p}$}{% em %} The sensitivity must be larger than one minus the specificity. Equivalently, the probability of detecting a positive-class instance must be larger than the probability of mislabeling a negative-class instance. This condition explains why, for a useful classifier, the ROC always lies *above* the main diagonal in a plot of {% m %}S_{e}{% em %} versus {% m %}1 - S_{p}{% em %}.

1. {% m %}\fbox{$\beta \lt 1 - \alpha$}{% em %} A reformulation of condition 1 in terms of Type-I and II error rates.

1. {% m %}\fbox{ppv $\gt \pi_{1}$}{% em %} The precision must be larger than the prevalence. In other words, the probability for an instance to belong to the positive class must be larger within the subset of instances with a positive label than within the entire population. If this is not the case, the classifier adds no useful information.

1. {% m %}\fbox{npv $\gt \pi_{0}$}{% em %} The probability for an instance to belong to the negative class must be larger within the subset of instances with a negative label than within the entire population.

1. {% m %}\fbox{$S_{e} \gt p_{1}$}{% em %} This follows from condition 3 and Bayes' theorem for {% m %}{\rm ppv}{% em %}. The probability of encountering a positively labeled instance must be larger within the subset of positive-class instances than within the entire population.

1. {% m %}\fbox{$S_{p} \gt p_{0}$}{% em %} This is similar to condition 5.

The classifier usefulness condition also puts a bound on the accuracy:
{% math %}
{\rm A} \;=\; S_{p}\,(1-\pi_{1}) \;+\; S_{e}\,\pi_{1} \;>\; p_{0}\,\pi_{0} \;+\; p_{1}\,\pi_{1}.
{% endmath %}
The bound is the accuracy of a *random* classifier with the same labeling rates {% m %}p_{0}{% em %} and {% m %}p_{1}{% em %} as the classifier of interest. By "random classifier" I mean one that assigns labels independently of true class. Now, note that the usefulness condition *implies* the above accuracy bound, but a stronger bound is needed for the converse to hold: the accuracy must be larger than the null accuracy {% m %}\max\{\pi_{0}, \pi_{1}\}{% em %}. The corresponding classifier, the majority classifier, is of course also random in the sense just defined.

<div style="text-align: right"><a href="#TopOfPage">Back to Top</a></div>
<a name="MetricsEstimation"></a>
## Performance metric estimation
The performance metrics can be estimated from a two-by-two contingency table known as the confusion matrix. It is obtained by applying the classifier to a test data set different from the training data set. Here is standard notation for this matrix:
{% fullwidth "assets/img/blog/ConfusionMatrix/ConfusionMatrix.png" 'Figure 2: Estimated confusion matrix. The notation tp stands for "number of true positives", fn for "number of false negatives", and so on.' %}
The rows correspond to class labels, the columns to true classes. We then have the following approximations:
{% math %}
\begin{aligned}
\mbox{Prevalence :} & \quad &&\equiv\pi(\lambda_{1}) &&\approx \dfrac{\rm fn+tp}{\rm fn+tp+fp+tn}\\[1mm]

\mbox{Queue rate :} & \quad &&\equiv p(\ell_{1}) &&\approx \dfrac{\rm fp+tp}{\rm fn+tp+fp+tn}\\[4mm]

\mbox{True-positive rate, sensitivity, recall :} & \quad S_{e} &&\equiv p(\ell_{1}\mid\lambda_{1}) &&\approx \dfrac{\rm tp}{\rm tp+fn}\\[1mm]

\mbox{True-negative rate, specificity :} & \quad S_{p} &&\equiv p(\ell_{0}\mid\lambda_{0}) &&\approx \dfrac{\rm tn}{\rm tn + fp}\\[1mm]

\mbox{False-positive rate, Type-I error rate :} & \quad \alpha &&\equiv p(\ell_{1}\mid\lambda_{0})&&\approx \dfrac{\rm fp}{\rm tn + fp}\\[1mm]

\mbox{False-negative rate, Type-II error rate :} & \quad \beta &&\equiv p(\ell_{0}\mid\lambda_{1})&&\approx \dfrac{\rm fn}{\rm tp + fn}\\[4mm]

\mbox{Positive predictive value, precision :} & \quad {\rm ppv} &&\equiv p(\lambda_{1}\mid\ell_{1}) &&\approx \dfrac{\rm tp}{\rm tp + fp}\\[1mm]

\mbox{Negative predictive value :} & \quad {\rm npv} &&\equiv p(\lambda_{0}\mid\ell_{0}) &&\approx \dfrac{\rm tn}{\rm tn + fn}\\[1mm]

\mbox{False-discovery rate :} & \quad {\rm fdr} &&\equiv p(\lambda_{0}\mid\ell_{1}) &&\approx \dfrac{\rm fp}{\rm tp + fp}\\[1mm]

\mbox{False-omission rate :} & \quad {\rm for} &&\equiv p(\lambda_{1}\mid\ell_{0}) &&\approx \dfrac{\rm fn}{\rm tn + fn}\\[4mm]

\mbox{Positive likelihood ratio :} & \quad {\rm lr+} &&\equiv \frac{p(\ell_{1}\mid\lambda_{1})}{p(\ell_{0}\mid\lambda_{1})}&&\approx \frac{\rm tp}{\rm fn}\\[1mm]

\mbox{Negative likelihood ratio :} & \quad {\rm lr-} &&\equiv \frac{p(\ell_{1}\mid\lambda_{0})}{p(\ell_{0}\mid\lambda_{0})}&&\approx \frac{\rm fp}{\rm tn}\\[1mm]

\mbox{Diagnostic odds ratio :} & \quad {\rm dor} &&\equiv \frac{\rm lr+}{\rm lr-} &&\approx \frac{\rm tp}{\rm fn} \frac{\rm tn}{\rm fp}\\[4mm]

\mbox{Accuracy :} & \quad A &&\equiv p(\ell=\lambda) &&\approx \dfrac{\rm tp + tn}{\rm tp + tn + fp + fn}
\end{aligned}
{% endmath %}
There is no simple expression for the AUROC, since the latter requires integration under the curve of true versus false positive rates.

Note that the relations between probabilities derived in the previous sections also hold between their estimates.

<div style="text-align: right"><a href="#TopOfPage">Back to Top</a></div>
<a name="EffectOfPrevalence"></a>
## Effect of prevalence on classifier training and testing
In this post I have tried to be careful in pointing out which performance metrics depend on the prevalence, and which don't. The reason is that prevalence is a population or sample property, whereas here I am interested in classifier properties. Of course, real-world classifiers must be trained and tested on finite samples, and this has implications for the actual effect of prevalence on classifier performance:

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

Training data sets are often unbalanced, with prevalences even lower than in the example discussed here. Modules such as scikit-learn's [`RandomForestClassifier`](http://scikit-learn.org/stable/modules/generated/sklearn.ensemble.RandomForestClassifier.html) provide a `class_weight` option, which can be set to `balanced` or `balanced_subsample` in order to assign a larger weight to the minority class. This re-weighting is applied both to the GINI criterion used for finding splits at decision tree nodes and to class votes in the terminal nodes. The sensitivity and specificity of a classifier, as well as its accuracy, will depend on the setting of the `class_weight` option. For scoring purposes when optimizing a classifier, AUROC tends to be a much more robust property.
<div style="text-align: right"><a href="#TopOfPage">Back to Top</a></div>
