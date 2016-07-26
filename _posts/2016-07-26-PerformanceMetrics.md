---
layout: full-width-post
title: "Taking the Confusion out of the Confusion Matrix"
date: 26 July 2016
excerpt: "How Bayes' theorem helps organize the bewildering array of performance metrics that can be estimated from a classifier's confusion matrix."
comments: true
---

I always thought that the *confusion* matrix was rather aptly named, a reference not so much to the mixed performance of a classifier as to my own bewilderment at the number of measures of that performance. Recently however, I encountered a brief mention of the possibility of a Bayesian interpretation of performance measures, and this helped clarify things in my mind. I'd like to develop the idea a little further in this post. It's not that Bayes' theorem is needed to understand or apply the performance measures, but it acts as an organizing principle, which I find helpful. New concepts are easier to remember if they fit inside a good story.

Another advantage of taking the Bayesian route is that this forces us to view performance measures as probabilities, which are *estimated* from the confusion matrix. Elementary presentations tend to *define* performance metrics in terms of ratios of confusion matrix elements, which then begs the question of why a classifier doesn't perform as well on a test set as on its training set.

## Bayes' theorem
Let's start from Bayes' theorem in its general form:
{% math %}
p(\lambda\mid X) \;=\; \frac{p(X\mid\lambda)\;\pi(\lambda)}{\int p(X\mid\lambda^{\prime})\;\pi(\lambda^{\prime})\;d\lambda^{\prime}},
{% endmath %}
where {% m %}X{% em %} represents the observed data and {% m %}\lambda{% em %} a parameter of interest. On the left-hand side is the posterior probability density of {% m %}\lambda{% em %} given {% m %}X{% em %}. On the right-hand side, {% m %}p(X\mid\lambda){% em %} is the likelihood, or conditional probability density of {% m %}X{% em %} given {% m %}\lambda{% em %}, and {% m %}\pi(\lambda){% em %} is the prior probability density of {% m %}\lambda{% em %}. The denominator is the marginal likelihood, sometimes called evidence. One way to think about Bayes' theorem is that it uses the data {% m %}X{% em %} to update the prior information {% m %}\pi(\lambda){% em %} about {% m %}\lambda{% em %}, and returns the posterior {% m %}p(\lambda\mid X){% em %}.

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
To describe the performance of a classifier, {% m %}X{% em %} could be summarized either by the *class label* {% m %}\ell{% em %} assigned to it by the classifier, or by the *score* {% m %}q{% em %} (a number between 0 and 1) assigned by the classifier to the hypothesis that {% m %}X{% em %} belongs to, say, class 1. Thus for example, {% m %} p(\lambda\!=\!1 \mid \ell\!=\!0) {% em %} represents the posterior probability that the true class is 1 given a class label of 0, and {% m %}p(\lambda\!=\!1\mid q\!\lt\! q_{T}){% em %} is the posterior probability that the true class is 1 given that the score {% m %}q{% em %} is below a pre-specified threshold {% m %}q_{T}{% em %}.

## Three tracks to performance metrics
Each of the next three subsections describes a different track for deriving performance measures. The first one maps each "building block" of Bayes' theorem to a performance metric. The second subsection works with joint probabilities, and the third one with ratios of probabilities (likelihood ratios).

## Track 1: From prevalence to precision via Bayes' theorem
To simplify the notation I'll write {% m %}\lambda_{0}{% em %} to signify {% m %}\lambda=0{% em %}, {% m %}\lambda_{1}{% em %} for {% m %}\lambda=1{% em %}, and similarly with other variables. The first ingredient in the computation of the posterior is the prior {% m %}\pi(\lambda){% em %}. To fix ideas, let's assume that {% m %}\lambda_{1}{% em %} is the class of interest, generically labeled "positive"; it indicates "effect", "signal", "disease", "fraud", or "spam", depending on the context. Then {% m %}\lambda_{0}{% em %} indicates the lack of these things and is generically labeled "negative". To fully specify the prior we just need {% m %}\pi(\lambda_{1}){% em %}, since {% m %}\pi(\lambda_{0}) = 1 - \pi(\lambda_{1}){% em %}. The prior probability {% m %}\pi(\lambda_{1}){% em %} of drawing a positive instance from the population is called the **prevalence**.

The next ingredient is the conditional probability {% m %}p(X\mid\lambda){% em %}. For now let's work with classifier labels and replace {% m %}X{% em %} by {% m %}\ell{% em %}. This leads to the following four definitions:

The **true positive rate**, also known as **sensitivity** or **recall**: {% m %}S_{e} \equiv p(\ell_{1}\mid \lambda_{1}){% em %},

The **true negative rate**, also known as **specificity**: {% m %}S_{p}\equiv p(\ell_{0}\mid \lambda_{0}){% em %},

The **false-positive or Type-I-error rate**: {% m %}\alpha\equiv p(\ell_{1}\mid\lambda_{0}) = 1 - p(\ell_{0}\mid\lambda_{0}){% em %}, and

The **false-negative or Type-II-error rate**: {% m %}\beta\equiv p(\ell_{0}\mid\lambda_{1}) = 1 - p(\ell_{1}\mid\lambda_{1}){% em %}.

For example, {% m %}p(\ell_{1}\mid \lambda_{0}){% em %} is the conditional probability that the classifier assigned a label of 1 to an instance actually belonging to class 0. When viewed as a function of true class {% m %}\lambda{% em %}, for fixed label {% m %}\ell{% em %}, the conditional probability is called **likelihood**. Note that the above four rates are independent of the prevalence and are therefore intrinsic properties of the classifier.  In contrast, the prevalence is a property of the sample on which the classifier is applied.

Finally, we need the normalization factor in Bayes' theorem, the evidence. For a binary classifier the evidence has two components, corresponding to the two possible labels:

The **positive labeling rate**: {% m %}p(\ell_{1}) = p(\ell_{1}\mid\lambda_{0})\,\pi(\lambda_{0}) + p(\ell_{1}\mid\lambda_{1})\,\pi(\lambda_{1}){% em %}, and

The **negative labeling rate**: {% m %}p(\ell_{0}) = p(\ell_{0}\mid\lambda_{0})\,\pi(\lambda_{0}) + p(\ell_{0}\mid\lambda_{1})\,\pi(\lambda_{1}){% em %}.

Armed with the prior probability, the conditional probability, and the evidence, we can compute the posterior probability from Bayes' theorem.  There are four combinations of truth and label:

The **positive predictive value**, also known as **precision**: {% m %}{\rm ppv}\equiv p(\lambda_{1}\mid\ell_{1}) = \frac{p(\ell_{1}\mid\lambda_{1})\, \pi(\lambda_{1})}{p(\ell_{1})} {% em %},

The **negative predictive value**: {% m %} {\rm npv}\equiv p(\lambda_{0}\mid\ell_{0}) = \frac{p(\ell_{0}\mid\lambda_{0})\, \pi(\lambda_{0})}{p(\ell_{0})} {% em %},

The **false discovery rate**: {% m %} {\rm fdr}\equiv p(\lambda_{0}\mid\ell_{1}) = 1 - {\rm ppv}{% em %}, and

The **false omission rate**: {% m %} {\rm for}\equiv p(\lambda_{1}\mid\ell_{0}) = 1 - {\rm npv} {% em %}.

The precision, for example, quantifies the predictive value of a positive label by answering the question: If we see a positive label, what is the (posterior) probability that the true class is positive?

Note that the positive and negative predictive values are not intrinsic properties of the classifier since they depend on the prevalence. Sometimes the predictive values are "standardized" by evaluating them at {% m %} \pi(\lambda_{1}) = 1/2 {% em %}.

## Track 2: Joint probabilities
A couple of joint probabilities lead to performance metrics with very different properties.

Consider first the joint probability of class label and true class, which can be decomposed in terms of likelihood and prior:
{% math %}
p(\ell,\lambda) \;=\; p(\ell\mid\lambda)\,\pi(\lambda).
{% endmath %}
For a given classifier this is the joint distribution of (class label, true class) of instances randomly drawn from a population with the same prevalence as the training data set. Projecting this joint probability on the axis "{% m %} \ell=\lambda {% em %}" yields the **accuracy**, or the probability to correctly identify any instance, negative or positive:
{% math %}
{\rm A} \;\equiv\; p(\ell=\lambda) \;=\; p(\ell_{0}\mid\lambda_{0})\,\pi(\lambda_{0}) \;+\; p(\ell_{1}\mid\lambda_{1})\,\pi(\lambda_{1}) \;=\; S_{p}\,(1-\pi(\lambda_{1})) \;+\; S_{e}\,\pi(\lambda_{1}).
{% endmath %}
The expression in terms of sensitivity and specificity shows that the accuracy depends on the prevalence and is therefore not an intrinsic property of the classifier. An equivalent measure is the **misclassification rate**, defined as one minus the accuracy. A benchmark that is sometimes used is the **null error rate**, defined as the misclassification rate of a classifier that always predicts the majority class. It is equal to {% m %}\min\{\pi(\lambda_{1}), 1-\pi(\lambda_{1})\}{% em %}. The complement of the null error rate is the **null accuracy**, which is equal to {% m %}\max\{\pi(\lambda_{1}), 1-\pi(\lambda_{1})\}{% em %}.

One way to avoid dependence on the prevalence is to independently draw an instance from a population with positive true class and an instance from a population with negative true class. Consider the joint probability of the scores assigned by the classifier to such a pair of instances. A good measure of performance is the probability that the score assigned to the positive instance is larger than the score assigned to the negative instance. It can be shown that this is actually equal to the **Area Under the Receiver Operating Characteristic (AUROC)**. The Receiver Operating Characteristic is a curve of the true positive rate as a function of the false positive rate (or sensitivity versus one-minus-specificity). In addition to being independent of prevalence, the AUROC does not require a choice of threshold on the classifier score {% m %}q{% em %}.

## Track 3: Likelihood ratios and classifier usefulness
In some fields it is customary to work with ratios of probabilities rather than with the probabilities themselves.  Thus one defines:

- The **positive likelihood ratio**: {% m %}{\rm lr+} \equiv \frac{p(\ell_{1}\mid\lambda_{1})}{p(\ell_{0}\mid\lambda_{1})} = \frac{S_{e}}{1-S_{e}}{% em %}, which represents the odds of a positive label if the true class is positive (in medical statistics for example, this would be the odds of a positive test if patient has the disease).

- The **negative likelihood ratio**: {% m %}{\rm lr-} \equiv \frac{p(\ell_{1}\mid\lambda_{0})}{p(\ell_{0}\mid\lambda_{0})} = \frac{1-S_{p}}{S_{p}}{% em %}, which represents the odds of a positive label if the true class is negative (in medical statistics, the odds of a positive test if patient does not have the disease).

Finally, it is instructive to take the ratio of these two likelihood ratios. This yields

- The **diagnostic odds ratio**: {% m %}{\rm dor} \equiv \frac{\rm lr+}{\rm lr-}{% em %}.

If {% m %}{\rm dor} \gt 1{% em %}, the classifier is deemed useful, since this means that, in our medical example, the odds of a positive test are higher for a patient with the disease than for a healthy patient. If {% m %}{\rm dor}=1{% em %} the classifier is useless, but if {% m %}{\rm dor}\lt 1{% em %} the classifier is worse than useless, it is misleading. In this case one would be better off swapping the labels on the classifier output.

The usefulness condition {% m %}\fbox{dor $\gt 1$}{% em %} can be shown to be mathematically equivalent to any of the following conditions:

1. {% m %}\fbox{ppv $\gt \pi(\lambda_{1})$}{% em %} For a classifier to be useful, the precision must be larger than the prevalence. In other words, the probability for an instance to belong to the positive class must be larger within the subset of instances with a positive label than within the entire population. If this is not the case, the classifier adds no useful information.

1. {% m %}\fbox{npv $\gt 1 - \pi(\lambda_{1})$}{% em %} This results from applying condition 1 to negative instances.

1. {% m %}\fbox{$S_{e}\gt 1 - S_{p}$}{% em %} For a classifier to be useful, the sensitivity must be larger than one minus the specificity. Equivalently, the probability of detecting a positive-class instance must be larger than the probability of mislabeling a negative-class instance. This condition explains why, for a useful classifier, the ROC always lies *above* the main diagonal in a plot of {% m %}S_{e}{% em %} versus {% m %}1 - S_{p}{% em %}.

1. {% m %}\fbox{$\beta \lt 1 - \alpha$}{% em %} This is a reformulation of condition 3 in terms of Type-I and II error rates.

Finally, note that the ratios {% m %}{\rm lr+}{% em %}, {% m %}{\rm lr-}{% em %}, and {% m %}{\rm dor}{% em %} discussed here are all independent of prevalence.

## Estimating performance metrics from the confusion matrix
The performance metrics can be estimated from the confusion matrix that is obtained by applying the classifier to a test data set different from the training data set. Here is my notation for this matrix:
<br/><br/>

|                        | {% m %}\ell_{1}{% em %} | {% m %}\ell_{0}{% em %} |
|:-----------------------|:-----------------------:|:-----------------------:|
| {%m%}\lambda_{1}{%em%} | tp                      | fn                      |
| {%m%}\lambda_{0}{%em%} | fp                      | tn                      |

where tp stands for true positives, fn for false negatives, and so on. The columns correspond to class labels, the rows to true classes. We have then the following approximations:
{% math %}
\begin{aligned}
\mbox{Prevalence :} & \quad &&\equiv\pi(\lambda_{1}) &&\approx \dfrac{\rm fn+tp}{\rm fn+tp+fp+tn}\\[4mm]

\mbox{True positive rate, sensitivity, recall :} & \quad S_{e} &&\equiv p(\ell_{1}\mid\lambda_{1}) &&\approx \dfrac{\rm tp}{\rm tp+fn}\\[1mm]

\mbox{True negative rate, specificity :} & \quad S_{p} &&\equiv p(\ell_{0}\mid\lambda_{0}) &&\approx \dfrac{\rm tn}{\rm tn + fp}\\[1mm]

\mbox{False positive rate, Type-I error rate :} & \quad \alpha &&\equiv p(\ell_{1}\mid\lambda_{0})&&\approx \dfrac{\rm fp}{\rm tn + fp}\\[1mm]

\mbox{False negative rate, Type-II error rate :} & \quad \beta &&\equiv p(\ell_{0}\mid\lambda_{1})&&\approx \dfrac{\rm fn}{\rm tp + fn}\\[4mm]

\mbox{Positive predictive value, precision :} & \quad {\rm ppv} &&\equiv p(\lambda_{1}\mid\ell_{1}) &&\approx \dfrac{\rm tp}{\rm tp + fp}\\[1mm]

\mbox{Negative predictive value :} & \quad {\rm npv} &&\equiv p(\lambda_{0}\mid\ell_{0}) &&\approx \dfrac{\rm tn}{\rm tn + fn}\\[1mm]

\mbox{False discovery rate :} & \quad {\rm fdr} &&\equiv p(\lambda_{0}\mid\ell_{1}) &&\approx \dfrac{\rm fp}{\rm tp + fp}\\[1mm]

\mbox{False omission rate :} & \quad {\rm for} &&\equiv p(\lambda_{1}\mid\ell_{0}) &&\approx \dfrac{\rm fn}{\rm tn + fn}\\[4mm]

\mbox{Positive likelihood ratio :} & \quad {\rm lr+} &&\equiv \frac{p(\ell_{1}\mid\lambda_{1})}{p(\ell_{0}\mid\lambda_{1})}&&\approx \frac{\rm tp}{\rm fn}\\[1mm]

\mbox{Negative likelihood ratio :} & \quad {\rm lr-} &&\equiv \frac{p(\ell_{1}\mid\lambda_{0})}{p(\ell_{0}\mid\lambda_{0})}&&\approx \frac{\rm fp}{\rm tn}\\[1mm]

\mbox{Diagnostic odds ratio :} & \quad {\rm dor} &&\equiv \frac{\rm lr+}{\rm lr-} &&\approx \frac{\rm tp}{\rm fn} \frac{\rm tn}{\rm fp}\\[4mm]

\mbox{Accuracy :} & \quad A &&\equiv p(\ell=\lambda) &&\approx \dfrac{\rm tp + tn}{\rm tp + tn + fp + fn}
\end{aligned}
{% endmath %}
There is no simple expression for the AUROC, since the latter requires integration under the curve of true versus false positive rates.

Note that the relations between probabilities derived in the previous sections also hold between their estimates.

## Unbalanced samples
In this post I have tried to be careful in pointing out which performance metrics depend on the prevalence, and which don't. The reason is that prevalence is a sample property, whereas here I am interested in classifier properties. More precisely, I am interested in *trained-classifier* properties. The difference is worth emphasizing:

- When training a classifier, changing the prevalence in the training data *will affect* properties such as the sensitivity and specificity.

- When applying a trained classifier to test data, changing the prevalence in those data *will not affect* properties such as the sensitivity and specificity.

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

To be clear, for each row it is the same classifier (same number of trees, same maximum tree depth, etc.) that is being fit to a training data set with varying prevalence. The sensitivity, specificity, accuracy, and AUROC are then measured on a testing data set with a constant prevalence of 30%. Note how the sensitivity and specificity vary with training set prevalence. The AUROC, on the other hand, remains remarkably stable.

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

Training data sets are often unbalanced, with prevalences even lower than in the example discussed here. Modules such as scikit-learn's [`RandomForestClassifier`](http://scikit-learn.org/stable/modules/generated/sklearn.ensemble.RandomForestClassifier.html) provide a "class_weight" option, which can be set to "balanced" or "balanced_subsample" in order to assign a larger weight to the minority class. This re-weighting is applied both to the GINI criterion used for finding splits at decision tree nodes and to class votes in the terminal nodes. The sensitivity and specificity of a classifier, as well as its accuracy, will depend on the setting of the class_weight option. For scoring purposes when optimizing a classifier, AUROC tends to be a much more robust property.
