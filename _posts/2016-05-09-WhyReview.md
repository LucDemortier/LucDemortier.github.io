---
layout: full-width-post
title: "The Eternal Sunshine of Causal Thinking"
date: 9 May 2016
excerpt: "A review of Samantha Kleinberg's latest book, \"WHY: A Guide to Finding and Using Causes\"."
---
<a name="top"></a>

On February 11 of this year, a collaboration of physicists known by the acronym [LIGO](http://www.ligo.org/) announced a remarkable discovery, that of gravitational waves.  Physicists had been searching for such waves since the 1960s, so it was a great relief to see confirmed one more prediction of Einstein's theory of general relativity.  However the real significance of the discovery was much more than historical, it was a breakthrough on three levels: the observation of a gravitational wave interacting with an apparatus on earth, the surmised origin of the wave as a merger of two black holes, and the advent of a new subfield,  gravitational-wave astronomy.

When physicists claim discoveries, they talk about evidence.  Evidence requires statistics, but what is statistical reasoning based on?  The underlying beliefs are that observations have causes, that we have methods to identify those causes, that we are not easily fooled by misleading evidence or cognitive biases, etc. These are difficult issues about which few scientists are truly knowledgeable, but they now have two new books to educate themselves.  Both were written by Samantha Kleinberg: ["Causality, Probability, and Time," Cambridge University Press (2013)](http://www.amazon.com/dp/1107026482), and ["Why: A Guide to Finding and Using Causes," O'Reilly Media (2016)](http://www.amazon.com/dp/1491949643).  The first book was written for an audience with knowledge of statistics and probability theory, whereas the second has no mathematics in it, only text and graphs.  This is the one I'm reviewing here.

{% fullwidth "assets/img/blog/SK_Books.jpg" "" %}

Causality is our home in this world. It is so basic to our way of thinking and acting that its conundrums show up everywhere, and this ubiquity makes it also hard to define.  Already in the first chapter the author warns us that no definition of causality covers all cases, and each definition has counter-examples that another does not. We don’t even know if causality is a fundamental building block of the world we live in, or a structure we impose on it. There is no unified theory of causes, nor a foolproof method for finding them. The last chapter reiterates this notion that "causality is still an unsolved problem".  In between these warnings the reader will find extensive discussions of how we learn about causes; how correlation relates to causation; the importance of timing; observation versus experimentation; computation; explanation; and the transition from causal understanding to decisions, policies, and interventions.

## A multitude of examples
One of the great strengths of the book is the multitude of concrete examples it gives to illustrate various points and stimulate the reader's thinking.  These examples range from the almost trivial (e.g. ice cream stands correlate with warm weather but do not cause it) to famous court cases (for example the 1999 Sally Clark case in the UK, involving the death of two children in the space of one year: was it a double case of sudden infant death syndrome, or murder?), from psychological experiments to understand the cognitive biases affecting the search for causes, to famous paradoxes (Simpson's paradox from probability theory and the Einstein-Podolsky-Rosen paradox in physics).  

## Languages for thinking about causality
The second strength of the book is that it helps the reader develop a language for thinking about causality. This language has a philosophical tradition: Hume talked about regular occurrence, contiguousness in time and space, time ordering of cause and effect, Kant about a priori knowledge, and more recently Mackie discussed causes as insufficient but necessary parts of unnecessary but sufficient conditions, whereas Suppes and others introduced a probabilistic approach to causality. There are many interesting concepts throughout the book to help describe the great variety of causal situations; for the interested reader I collected some examples in the [appendix](#appendix) to this review.

Then there is the psychological tradition, which has uncovered a number of cognitive obstacles to learning causes: superstition, confirmation bias, stereotype threat, placebo effect, side-effect effect, etc., as well as causal learning paths: the associative model, backward blocking, mechanisms,…  

Finally there is the mathematical/computational tradition, which to some extent is based on statistics and probability but needs something more to account for causality, as demonstrated by Simpson's paradox and Judea Pearl's insightful work.  As with all mathematical approaches however, there are assumptions: Have we identified all hidden common causes (confounders)?  Are our observations representative of the true system we are trying to describe?  And are we measuring the right things?  This is where methods and results from the philosophical and psychological traditions can be of great help.

## An example from hard science
Let's see how these concepts work in the case of the gravitational wave discovery mentioned at the beginning. This discovery was a one-time event, so Hume's criterion of regular occurrence doesn't apply.  The initial cause, a black hole merger, was not actually observed, it was merely inferred from the shape of the gravitational wave signal itself. So what is the causal evidence in this case? It consists of two pieces. First, the probability that noise alone caused the signal observed in the apparatus is extremely low (the signal is actually a time-coincidence of identically shaped signals in two independent detectors, one located in Livingston, LA, and the other in Hanford, WA) . Second, we actually have a detailed mechanism to explain the observation, and this mechanism is based on a powerful theory (general relativity) that has already explained plenty of other types of observations.  So the new observation fits nicely with our current model of how the universe works. Nevertheless the causal evidence is rather unusual by the standards described in the book and falls into the category of "finding causes and effects of infrequent events", described in chapter 1 as one of the open problems of causality research. This is clearly an area where domain knowledge plays a crucial role.

## Recipes and procedures
A third aspect of the book that I found useful is the possibility to extract procedures and recipes from it to approach certain problems.  For example, chapter 5 lists Mill's methods for learning about causes: the method of agreement, the method of difference, the joint method of agreement and difference, the method of residues, and the method of concomitant variation.  And in chapter 9 we encounter Bradford Hill's considerations for evaluating causal claims: strength, consistency, specificity, temporality, biological gradient, plausibility and coherence, experiment, and analogy.  In each case these methods and considerations are clearly described in practical terms. I have added some annotated examples of such techniques in the [appendix](#appendix) to this review.

## Causality and big data
In addition to reiterating some key principles of causal thinking, the last chapter of the book argues for the importance of causality research in the context of big data.  Contrary to what is sometimes claimed, big data often requires more rigorous causal thinking than "small data", and this for three reasons: data quality, which is often worse in large data sets; spurious signals, which are more likely when performing large numbers of tests; and interpretation, which is typically harder with big data.

## The book's structure
A possible criticism of the book is its weak support structure. By this I mean the lack of typographical design and proper sectioning to facilitate learning. The book has ten numbered chapters, but chapter sections and subsections are unnumbered (only the title capitalization differentiates the latter two). There is no subdivision below the subsection level, and I would argue that this is regrettable for a philosophical book of this nature. This lack of structural visibility obscures the flow of the book's arguments. It also prevents proper cross-referencing.  For example, the second paragraph on page 122 asks the reader to "recall the dead salmon study", without further location information.  Luckily in this case the reader can go look up "dead salmon fMRI study" in the index to find that it is discussed on pages 52 and 55.  But the reader shouldn't have to do this, there should be a direct reference to a subsection or paragraph number.  The notes system is similarly awkward.  Notes are at the end of the book, between pages 209 and 231, and are numbered separately for each chapter (an annoying burden on the reader).  The bibliography is between pages 233 and 266.  Almost 20% of the notes are just references to the bibliography, which means that the reader often has to make two jumps to get to the desired information. (I should add however, that the bibliography is extensive and useful.)

## The book's scope
As mentioned earlier, the author deliberately chose to make the book accessible to a general audience and therefore avoided mathematical formalities.  Most of the time she succeeds, but less so when it comes down to computational techniques, the subject of chapter 6. This is also the longest chapter (29 pages; the average is 20), as it introduces a number of difficult concepts: graphical models, Bayesian networks, causal significance, Granger causality, etc.  There are plenty of examples, but few rigorous definitions, and this can be a frustrating impediment to understanding. I would have been hard pressed to give a simple definition of Granger causality after reading this chapter, so I turned to Wikipedia for help (see the definition in the [appendix](#appendix) below).  Although the chapter is titled computation, the reader should not expect to learn to actually compute things.  This is rather a conceptual introduction to computation.

## Recommendation
Overall this is a fascinating book and I highly recommend it.  Be prepared for hard work though.  It will sometimes be necessary to consult works in the bibliography to gain a better understanding of a topic.  For example, the discussion of Simpson's paradox on pages 94-96 is quite illuminating, but ends with the rather vague recommendation that level of granularity matters when looking at data, as does background knowledge about a problem.  The reader will be well served to study Judea Pearl's paper on Simpson's paradox for a more quantitative, graph-based treatment. It may also be necessary to read the book twice, as the author occasionally indulges in forward references…

<a name="appendix"></a>

## Appendix
This appendix gathers some material I put together after reading the book, thinking it could be useful for further study or in practical applications.  It begins with a short [vocabulary](#vocab) of terms used in the causality literature (most definitions are adapted from Kleinberg's book, but I have indicated when this is not the case).  This is followed by four "techniques": [Mill's methods](#tech1) for finding causes from observations, [Bradford Hill's considerations](#tech2) for evaluating causal claims, considerations for [how to intervene](#tech3) to make a desired effect happen, and [key principles](#tech4) of causal analysis.

[Back to Top](#top).

<hr class="slender">
<a name="vocab"></a>

### **Partial index of interesting concepts to describe causal situations:**

1. **Backup cause:** when the absence of a cause triggers another cause of the same effect.
1. **Backward blocking:** if you see an effect when two factors are present, and the effect stays when one factor is removed, then that factor is probably not a cause.  We can infer this without seeing the effect of removing the other factor.
1. **Case-control study:** Type of study where one takes two groups that differ in some feature (e.g. hair color), and one goes back to see what's different about them (e.g. genetic variation).  Contrast with cohort study and randomized controlled trial.
1. **Cohort study:** Type of study where one group is prospectively followed over a period of time. Contrast with case-control study and randomized controlled trial.
1. **Confirmation bias:** to seek and remember evidence that supports one's existing beliefs.
1. **Confounding variable:** A variable that influences both the presumed cause and the observed effect, thereby explaining their connection.
1. **Counterfactual reasoning:** If C had not occurred, E would not have occurred.
1. **Crossover Study:** A study whereby two procedures A and B are tested sequentially on the same individual.
1. **Cum hoc ergo propter hoc:** Logical fallacy, meaning "with, therefore because".
1. **Double-blinded trial:** Clinical trial where neither patients nor those assessing the patients  know which patients are receiving the treatment and which are not. Contrast with single- and triple-blinded trials.
1. **Effectiveness:** How effective an intervention is in the real world. Contrast with efficacy.
1. **Efficacy:** How effective an intervention is in an idealized setting. Contrast with effectiveness.
1. **Eikosogram:** Diagram to represent conditional probabilities. Also called **mosaic**, or **marimekko diagram**.
1. **External validity:** Whether a finding can be extrapolated outside of a study population, or across time.
1. **Granger causality:** A variable X that evolves over time Granger-causes another evolving variable Y if predictions of the value of Y based on its own past values and on the past values of X are better than predictions of Y based only on its own past values (from [Wikipedia](https://en.wikipedia.org/wiki/Granger_causality)).
1. **Intermediate variable:** A variable that contributes to the appearance of the true cause, but isn't itself the true cause (e.g., living in a city is correlated with low body-mass index, but the real reason for low BMI is that one tends to walk more in a city.)
1. **J-shaped curve:** A type of nonlinear relationship between cause and effect.  For example, in the relationship between alcohol consumption and coronary heart disease, illness decreases as consumption goes from zero to two drinks a day, but increases after that.
1. **Omitted variable bias:** When the cause(s) of two or more variables are unmeasured.
1. **Overdetermined event:** When two or more factors could claim responsibility for the event, and one cannot be definitively ruled a cause. Contrast with preemption.
1. **Placebo effect:** Effect whereby a treatment with no known active ingredient still improves outcomes.
1. **Post hoc ergo propter hoc:** Logical fallacy, meaning "after, therefore because".
1. **Preemption:** When two or more factors could be responsible for an effect, but only one is in fact responsible. Contrast with overdetermined event.
1. **Prosecutor's fallacy:** When the probability of an event is argued to be the probability of guilt or innocence.
1. **Randomized controlled trial:** Type of experiment where there are two or more groups and participants are randomly assigned to each, so that the difference in treatment is supposed to be the only difference between the groups.  Contrast with cohort study and case-control study.
1. **Redundant causation:** Where several causes occur and any of them could have caused the effect.
1. **Replication:** Repeating an experiment with the exact same methodology under the exact same conditions to make sure the method is well documented and the findings are stable. Contrast with reproduction.
1. **Reproduction:** Repeating an experiment while introducing some variation to test generalizability. Contrast with replication.
1. **Sampling bias:** to focus on only one portion of the data.
1. **Screening off:** When conditioning on a common cause results in two effects becoming uninformative about each other. Example: a disease (D) causes both fatigue (F) and the taking of medication (M), but M by itself does not cause F. The probability of fatigue may nevertheless seem to depend on whether or not medication is taken, since patients who take medication are more likely to have the disease. However that dependence disappears if we study it separately on disease and no-disease cases.  
1. **Side-effect effect**, or **Knobe effect:** When no credit is given for positive effects that weren't intentional, but blame is given for negative effects that weren't the goal either.
1. **Simpson's paradox:** A relationship within subgroups might disappear or even  be reversed when the subgroups are combined.
1. **Single-blinded trial:** Clinical trial where patients do not know which group they have been assigned to (treatment or no-treatment). Contrast with double- and triple-blinded trials.
1. **Stereotype threat:** When knowing that one is part of a group with negative characteristics leads to a fear of confirming those stereotypes.
1. **Survival bias:** Stems from analyzing outcomes solely from the set of individuals who made it to some endpoint.
1. **Token-level cause:** That which caused a specific effect in a particular instance (used to determine legal liability, or to assign credit when awarding prizes). Contrast with type-level cause.
1. **Triple-blinded trial:** Clinical trial where patients, those assessing the patients, and those analyzing the data do not know who is receiving the treatment and who is not. Contrast with single- and double-blinded trials.
1. **Type-level cause:** That which causes a specific effect in general (provides knowledge that can be used for prediction). Contrast with token-level cause.
1. **Wallpaper effect:** When a study fails to replicate because, unbeknownst to the investigators, there was a real difference in experimental setup between the original investigation and the failed replication (the joke being that the
experiment was affected by the color of the wallpaper in the room). From [William A. Wilson, "Scientific Regress", First Things, May 2016](http://www.firstthings.com/article/2016/05/scientific-regress).

[Back to Appendix](#appendix); [Back to Top](#top).

<hr class="slender">
<a name="tech1"></a>

### **Technique 1: Mill's methods for finding causes from observations**

1. **Method of agreement:** what's the same in all cases where the effect happens? (necessity but not sufficiency)
1. **Method of difference:** what differs between when the effect occurs and doesn't? (sufficiency but not necessity)
1. **Joint method of agreement and difference:** What factors are common to all cases where the effect happens, and only to those cases?  Very strict condition, as real-world instances of causality sometimes fail to hold in every case.
1. **Method of residues:** Subtract known causes and effects.  If there is one cause left, it is the cause of the remaining effect.
1. **Method of concomitant variation:** As the amount of the cause increases, the amount of the effect increases. (But not always, cfr. J-shaped curve.)

[Back to Appendix](#appendix); [Back to Top](#top).

<hr class="slender">
<a name="tech2"></a>

### **Technique 2: Bradford Hill's considerations for evaluating causal claims**

1. **Strength:**
This can mean making an event more likely, or having a larger effect size.
When we see a strong correlation, questions to ask include: Is the relationship asymmetrical? Could the correlation be explained by a shared cause? By a methodological problem? Are we ignoring other factors strongly correlated with the effect?  Are we dealing with nonstationary time series data?
1. **Consistency (Repeatability):**
Variations introduced in repeating the experiment can lead to stronger conclusions about the robustness of the relationship.
Inconsistent results can be used to dispel seemingly strong causal findings.
Consistent findings may still be due to a common flaw or oversight in all the studies.
1. **Specificity:**
A cause can have many effects, but with different strengths of influence on each.
1. **Temporality:**
Order of events and time delay between cause and effect.
1. **Biological gradient:**
Does more of the cause lead to more of the effect (Mill's method of concomitant variation)? Caveat: some relationships have a J-shaped curve (e.g. risk of heart disease is higher with both low and high consumption of alcohol, but reduced somewhere in between.)
1. **Plausibility and coherence:**
Is there a mechanism that could connect cause and effect?
Plausibility: given what we know, we can conceive of some way the relationship could work.
Coherence: the connection between cause and effect does not contradict what we know.
1. **Experiment:**
If we intervene to introduce the cause or increase its presence, does the effect come about?
1. **Analogy:**
If we know of a similar relationship, the standards of evidence may be lowered.

[Back to Appendix](#appendix); [Back to Top](#top).

<hr class="slender">
<a name="tech3"></a>

### **Technique 3: Considerations for how to intervene to make a desired effect happen**

1. **Context:**
What factors make a cause effective, and are they present?
1. **Effectiveness and efficacy:**
What happens in the real world versus what happens in an idealized setting.
1. **Distribution of effect size:**
Is the average effect obscuring extremely high and/or extremely low values?
1. **Unintended consequences:**
Develop a detailed model that includes not only the cause but also the method of implementation.
Do a cost-benefit analysis.
Note that not all unintended consequences are negative.

[Back to Appendix](#appendix); [Back to Top](#top).

<hr class="slender">
<a name="tech4"></a>

### **Technique 4:  Key principles of causal analysis**

1. **Causation and correlation are not synonymous.**
Knowing all the ways there can be correlation without causation is important to evaluate findings and prevent ineffective interventions.
Without understanding why something is predictive, we cannot avoid unexpected failures.
1. **Think critically about bias.**
People may disagree about what caused an event and about the relative salience of different causes for a single event.
In many cases we need to collect more data, and from different sources, than originally planned.
Validation schemes are difficult to design.
1. **Time matters.**
When using historical data to estimate, say, a disease risk, ask whether data and causal relationships could have changed over time, and if they are still applicable at the time of interest.
What is the delay between cause and effect? Is it a plausible delay?
1. **"All experimentation" is not better than "all observation".**
In many cases, a thoughtful combination of experimental and observational data can address the limits of each.

[Back to Appendix](#appendix); [Back to Top](#top).