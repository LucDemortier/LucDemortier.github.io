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
> <br> &ensp;&ensp;&ensp;[2.6 Summary](#Summary)
> <br>[3. Joint probabilities](#JointProbabilities)
> <br>[4. Classifier scores](#ScoreBasedMetrics)
> <br>[5. Likelihood ratios](#LikelihoodRatios)
> <br>[6. When is a classifier useful?](#ClassifierUsefulness)
> <br>[7. Performance metric estimation](#MetricsEstimation)
> <br>[8. Effect of prevalence on classifier training and testing](#EffectOfPrevalence)

---

<a name="Introduction"></a>
## 1. Introduction
I always thought that the *confusion* matrix was rather aptly named, a reference not so much to the mixed performance of a classifier as to my own bewilderment at the number of measures of that performance. Recently however, I encountered [a brief mention](https://www.dataschool.io/simple-guide-to-confusion-matrix-terminology/) of the possibility of a Bayesian interpretation of performance measures, and this inspired me to explore the idea a little further. It's not that Bayes' theorem is needed to understand or apply the performance measures, but it acts as an organizing and connecting principle, which I find helpful. New concepts are easier to remember if they fit inside a good story.

Another advantage of taking the Bayesian route is that this forces us to view performance measures as probabilities, which are *estimated* from the confusion matrix. Elementary presentations tend to *define* performance metrics in terms of ratios of confusion matrix elements, thereby ignoring the effect of statistical fluctuations.

Bayes' theorem is not the only way to generate performance metrics. One can also start from joint probabilities, classifier scores, or likelihood ratios. The next four subsections describe these approaches one by one. This is followed by a subsection discussing a minimal condition for a classifier to be useful, a subsection on estimating performance measures, and one on the effect of changing the prevalence in the training and testing data sets of a classifier.

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
If our sole purpose is to describe the performance of a classifier in general terms, the list of predictors {% m %}X{% em %} can be summarized by the *class label* {% m %}\ell{% em %}, or by the *score* {% m %}q{% em %} (a number between 0 and 1), assigned by the classifier. Thus for example, {% m %} p(\lambda\!=\!1 \mid \ell\!=\!0) {% em %} represents the posterior probability that the true class is 1 given a class label of 0, and {% m %}p(\lambda\!=\!1\mid q\!\ge\! q_{T}){% em %} is the posterior probability that the true class is 1 given that the score {% m %}q{% em %} is above a pre-specified threshold {% m %}q_{T}{% em %}.

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
The next ingredient is the conditional probability {% m %}p(X\mid\lambda){% em %}. For now let's work with classifier labels and replace {% m %}X{% em %} by {% m %}\ell{% em %}. This leads to the following four definitions:

- The **true positive rate**, also known as **sensitivity** or **recall**: {% m %}S_{e} \equiv p(\ell_{1}\mid \lambda_{1}){% em %},

- The **true negative rate**, also known as **specificity**: {% m %}S_{p}\equiv p(\ell_{0}\mid \lambda_{0}){% em %},

- The **false-positive or Type-I-error rate**: {% m %}\alpha\equiv p(\ell_{1}\mid\lambda_{0}) = 1 - p(\ell_{0}\mid\lambda_{0}){% em %}, and

- The **false-negative or Type-II-error rate**: {% m %}\beta\equiv p(\ell_{0}\mid\lambda_{1}) = 1 - p(\ell_{1}\mid\lambda_{1}){% em %}.

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

<a name="Summary"></a>
### 2.6 Summary
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
The most useful measures of classifier performance are *conditional* probabilities. By conditioning on something, we do not need to know whether or how that something is realized in order to make a valid statement. For example, by conditioning on true class, we ignore the prevalence when estimating sensitivity and specificity. Similarly, conditioning on class label allows us to derive predictive values without knowing the actual queue rate.

In contrast, *joint* probabilities typically yield statements about the behavior of a classifier in a *specific* sample and are therefore less general than conditional probabilities. Nevertheless, by combining different joint probabilities one can still obtain useful metrics. Start for example with the joint probability for class label and true class to be positive:
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
         \;=\; \mathit{ppv}\,p_{0} \;+\; \mathit{npv}\,p_{1}.
\end{align}
{% endmath %}
Note the dependence on prevalence or queue rate. An equivalent measure is the **misclassification rate**, defined as one minus the accuracy. A benchmark that is sometimes used is the **null error rate**, defined as the misclassification rate of a classifier that always predicts the majority class. It is equal to {% m %}\min\{\pi_{0}, \pi_{1}\}{% em %}. The complement of the null error rate is the **null accuracy**, which is equal to {% m %}\max\{\pi_{0}, \pi_{1}\}{% em %}.

<div style="text-align: right"><a href="#TopOfPage">Back to Top</a></div>
<a name="ScoreBasedMetrics"></a>
## 4. Classifier scores
As mentioned earlier, classifiers usually produce a score {% m %}q{% em %} to quantify the likelihood that an instance belongs to the positive class. The performance metrics introduced so far can be defined in terms of this score, but they only need it in a binary way: {% m %}\ell=1{% em %} if {% m %}q\ge q_{T}{% em %}, otherwise {% m %}\ell=0{% em %}, for some predefined threshold {% m %}q_{T}{% em %}. The next performance metric we discuss actually uses score values without reference to {% m %}q_{T}{% em %}. This is the so-called **Area Under the Receiver Operating Characteristic**, or **AUROC**, defined as the probability for a random positive instance to be scored higher than a random negative instance. Its name comes from an alternative definition, as the area under the curve of true positive rate versus false positive rate (or sensitivity versus one-minus-specificity; this curve is known as the Receiver Operating Characteristic, or ROC). By construction, the AUROC is independent of prevalence, and does not require a choice of threshold {% m %}q_{T}{% em %} on the classifier score {% m %}q{% em %}. In other words, it does not depend on the chosen operating point of the classifier.

An immediate interpretation of the AUROC is that it quantifies how well the score distributions for positive and negative instances are separated. If the score distribution for positives is to the right of that for negatives, and there is no overlap, the AUROC equals 1. If the positives are all to the left of the negatives, the AUROC equals 0, and if the two distributions overlap each other exactly, the AUROC equals 0.5.

The AUROC is also related to expectation values of sensitivity and specificity. To see this, note that we can write the sensitivity as a conditional expectation value:
{% math %}
S_{e} \;=\; \mathbb{P}\left[Q_{1}\ge q_{T}\right]
      \;=\; \mathbb{E}\left[\mathbb{1}(Q_{1} \ge q_{T})\right]
      \;=\; \mathbb{E}\left[\mathbb{1}(Q_{1} \ge Q^{\prime}) \mid Q^{\prime}=q_{T}\right],
{% endmath %}
where {% m %}Q_{1}{% em %} is a random variable distributed as the classifier scores of positive instances in the population of interest, and {% m %}\mathbb{1}(A){% em %} is the indicator function of set {% m %}A{% em %}. The conditioning trick in the rightmost expression allows us to extend the calculation: instead of fixing {% m %}Q^{\prime}{% em %} at the threshold {% m %}q_{T}{% em %}, let us draw it randomly from the *entire* population of instance scores, and calculate the expectation value of {% m %}S_{e}{% em %} with respect to {% m %}Q^{\prime}{% em %}. Thus, with probability {% m %}\pi_{0}{% em %} the variable {% m %}Q^{\prime}{% em %} equals a random draw {% m %}Q^{\prime}_{0}{% em %} from the population of negative instance scores, and with probability {% m %}\pi_{1}{% em %} it equals a random draw {% m %}Q^{\prime}_{1}{% em %} from the population of positive instance scores. Using the law of total expectation this yields:
{% math %}
\begin{align}
\mathbb{E}\left[S_{e}\right]
\;&=\; \mathbb{E}\left[ \;\mathbb{E}\left[ \mathbb{1}(Q_{1} \ge Q^{\prime})\mid Q^{\prime} \right]\; \right]\\[1mm]
\;&=\; \pi_{0}\,\mathbb{E}\left[ \mathbb{1}(Q_{1} \ge Q^{\prime}_{0}) \right] \;+\; \pi_{1}\,\mathbb{E}\left[ \mathbb{1}(Q_{1} \ge Q^{\prime}_{1}) \right]\\[1mm]
\;&=\; \pi_{0}\,\mathbb{P}\left(Q_{1} \ge Q^{\prime}_{0}\right) \;+\; \pi_{1}\,\mathbb{P}\left(Q_{1} \ge Q^{\prime}_{1}\right)\\[1mm]
\;&=\; \pi_{0}\, \textrm{AUROC} \;+\; \frac{\pi_{1}}{2}.
\end{align}
{% endmath %}
One can similarly show that:
{% math %}
\mathbb{E}\left[S_{p}\right] \;=\; \pi_{1}\, \textrm{AUROC} \;+\; \frac{\pi_{0}}{2}.
{% endmath %}
Using the expression for the accuracy A in terms of {% m %}S_{e}{% em %} and {% m %}S_{p}{% em %},
and the linearity of the expectation operator, we find:
{% math %}
\mathbb{E}\left[A\right] \;=\; 2\,\pi_{0}\,\pi_{1}\,\textrm{AUROC} \,+\, \frac{\pi_{0}^{2} + \pi_{1}^{2}}{2}.
{% endmath %}
The expectation values in the last three equations are over an ensemble of classifiers with operating points that are randomly drawn from the distribution of scores of the population of interest. Hence, these equations show how the AUROC aggregates classifier performance information in a way that's independent of prevalence and operating point (see also [this answer](https://www.quora.com/Machine-Learning-What-is-an-intuitive-explanation-of-AUC/answer/Peter-Flach) by Peter Flach on Quora).

<div style="text-align: right"><a href="#TopOfPage">Back to Top</a></div>
<a name="LikelihoodRatios"></a>
## 5. Likelihood ratios
In some fields it is customary to work with ratios of probabilities rather than with the probabilities themselves.  Thus one defines:

- The **positive likelihood ratio**: {% m %}{\rm lr+} \equiv \frac{p(\ell_{1}\mid\lambda_{1})}{p(\ell_{0}\mid\lambda_{1})} = \frac{S_{e}}{1-S_{e}}{% em %}, which represents the odds of a positive label if the true class is positive (in medical statistics for example, this would be the odds of a positive test if patient has the disease).

- The **negative likelihood ratio**: {% m %}{\rm lr-} \equiv \frac{p(\ell_{1}\mid\lambda_{0})}{p(\ell_{0}\mid\lambda_{0})} = \frac{1-S_{p}}{S_{p}}{% em %}, which represents the odds of a positive label if the true class is negative (in medical statistics, the odds of a positive test if patient does not have the disease).

Finally, it is instructive to take the ratio of these two likelihood ratios. This yields

- The **diagnostic odds ratio**: {% m %}\mathit{dor} \equiv \frac{\rm lr+}{\rm lr-} = \frac{S_{e}}{1-S_{e}}\frac{S_{p}}{1-S_{p}}{% em %}.

If {% m %}\mathit{dor} \gt 1{% em %}, the classifier is deemed useful, since this means that, in our medical example, the odds of a positive test are higher for a patient with the disease than for a healthy patient. If {% m %}\mathit{dor}=1{% em %} the classifier is useless, but if {% m %}\mathit{dor}\lt 1{% em %} the classifier is worse than useless, it is misleading. In this case one would be better off swapping the labels on the classifier output.

Note that the ratios {% m %}{\rm lr+}{% em %}, {% m %}{\rm lr-}{% em %}, and {% m %}\mathit{dor}{% em %} are all independent of prevalence.

<div style="text-align: right"><a href="#TopOfPage">Back to Top</a></div>
<a name="ClassifierUsefulness"></a>
## 6. When is a classifier useful?
The usefulness condition {% m %}\fbox{$\mathit{dor} \gt 1$}{% em %} is mathematically equivalent to any of the following conditions:

1. {% m %}\fbox{$S_{e}\gt 1 - S_{p}$}{% em %} The sensitivity must be larger than one minus the specificity. Equivalently, the probability of detecting a positive-class instance must be larger than the probability of mislabeling a negative-class instance. This condition explains why, for a useful classifier, the ROC always lies *above* the main diagonal in a plot of {% m %}S_{e}{% em %} versus {% m %}1 - S_{p}{% em %}. Reformulating in terms of Type-I and II error rates this condition becomes {% m %}\fbox{$\beta \lt 1 - \alpha$}{% em %}.

1. {% m %}\fbox{$S_{e} \gt p_{1}$}{% em %} The probability of encountering a positively labeled instance must be larger within the subset of positive-class instances than within the entire population. Similarly: {% m %}\fbox{$S_{p} \gt p_{0}$}{% em %}, {% m %}\fbox{$\alpha \lt p_{1}$}{% em %}, and {% m %}\fbox{$\beta \lt p_{0}$}{% em %}.

1. {% m %}\fbox{$\mathit{ppv} \gt \pi_{1}$}{% em %} The precision must be larger than the prevalence. In other words, the probability for an instance to belong to the positive class must be larger within the subset of instances with a positive label than within the entire population. If this is not the case, the classifier adds no useful information. Similarly: {% m %}\fbox{$\mathit{npv} \gt \pi_{0}$}{% em %}, {% m %}\fbox{$\mathit{fdr} \lt \pi_{0}$}{% em %}, and {% m %}\fbox{$\mathit{for} \lt \pi_{1}$}{% em %}.

The classifier usefulness condition also puts a bound on the accuracy:
{% math %}
{\rm A} \;=\; 1 - \alpha \;+\; (\alpha - \beta) \,\pi_{1}
        \;>\; 1 - \alpha \;+\; (2\alpha - 1) \,\pi_{1}.
{% endmath %}

It is straightforward to verify that the majority and minority classifiers defined earlier both fail any of the above usefulness conditions. As a counter-example, imagine starting with the majority classifier and flipping the label on a single positive instance. Thus, this classifier sets {% m %}\ell=1{% em %} on one {% m %}\lambda=1{% em %} instance and {% m %}\ell=0{% em %} on all other instances. It has 100% precision on positive labels ({% m %}\mathit{ppv}=1{% em %}). For negative labels the precision is {% m %}\mathit{npv} = \pi_{0}/(1-1/N){% em %}, with {% m %}N{% em %} the total number of instances. Hence this classifier passes the usefulness condition.

<div style="text-align: right"><a href="#TopOfPage">Back to Top</a></div>
<a name="MetricsEstimation"></a>
## 7. Performance metric estimation
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

\mbox{Positive likelihood ratio :} & \quad {\rm lr+} &&\equiv \frac{p(\ell_{1}\mid\lambda_{1})}{p(\ell_{0}\mid\lambda_{1})}&&\approx \frac{\rm tp}{\rm fn}\\[1mm]

\mbox{Negative likelihood ratio :} & \quad {\rm lr-} &&\equiv \frac{p(\ell_{1}\mid\lambda_{0})}{p(\ell_{0}\mid\lambda_{0})}&&\approx \frac{\rm fp}{\rm tn}\\[1mm]

\mbox{Diagnostic odds ratio :} & \quad \mathit{dor} &&\equiv \frac{\rm lr+}{\rm lr-} &&\approx \frac{\rm tp}{\rm fn} \frac{\rm tn}{\rm fp}\\[4mm]

\mbox{Accuracy :} & \quad A &&\equiv p(\ell=\lambda) &&\approx \dfrac{\rm tp + tn}{\rm tp + tn + fp + fn}
\end{aligned}
{% endmath %}
There is no simple expression for the AUROC, since the latter requires integration under the curve of true versus false positive rates.

Note that the relations between probabilities derived in the previous sections also hold between their estimates.

<div style="text-align: right"><a href="#TopOfPage">Back to Top</a></div>
<a name="EffectOfPrevalence"></a>
## 8. Effect of prevalence on classifier training and testing
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
