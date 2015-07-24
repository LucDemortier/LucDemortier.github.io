---
layout: post
title:  "Project Kojak: The Practice of Yoga in the City"
date: 25 June 2015
excerpt:
---
Yoga is a centuries-old practice {% marginfigure /assets/img/aum.jpg 'Sanskrit character for the "Aum" sound. In his "Autobiography of a Yogi", Paramahansa Yogananda writes "The potencies of sound and of the human voice have nowhere else been so profoundly investigated as in India.  The Aum vibration that reverberates throughout the universe has three manifestations, those of creation, preservation, and destruction.  Each time a man utters a word he puts into operation one of the three qualities of Aum."' %} that was started in India by ascetics who needed to maintain good physical health during their long meditation sessions.  While it declined in its country of origin, over the past fifty years yoga took off in the West.  In fact, one of the founders of the modern yoga movement, B.K.S. Iyengar, is reported to have said that yoga was saved by the West.  How did Westerners do this?  Well, they turned this revered practice into a business!  And now that it is a business, we can ask some pointed questions about it, and as data scientists maybe we can come up with some quantitative answers.

What motivates people to practice yoga?  What do they value in their practice?  Which types of yoga are most popular?  How does practice depend on where it is done?  How can we help people find their way in this world of yoga?  To answer these questions I looked at Yelp reviews of yoga businesses in the New York City and Los Angeles areas.  For the purpose of this project, a yoga business is any business with a connection to yoga, so this includes studios exclusively devoted to yoga teaching and practice, yoga apparel stores, gyms that offer yoga classes, etc.  The table below shows how many businesses and reviews I found for the New York City and Los Angeles metropolitan areas:

|  | New York City | Los Angeles |
|:--|--:|--:|
| Number of yoga businesses: | 796 | 741 |
| Number of reviews: | 9,299 | 27,953 |

The number of businesses is about the same in both areas, but the number of reviews is three times larger in Los Angeles, implying a difference in the number of reviews per business.  This is shown in the following superimposed histograms (each normalized to an area of 1 to facilitate comparison; also note the log scale along the y axis): {% maincolumn /assets/img/n_reviews_nyc_la.jpg 'Figure 1: Distributions of the number of reviews per yoga business in the LA (red) and NYC (green) regions.' %}
The distribution of the number of reviews per business has a longer tail for LA than for NYC.  A couple of outliers are worth noting: "Runyon Canyon Park" is a 160-acre park in LA, with lots of hiking trails and offering free yoga classes several times a week.  Not all the reviews about the park are about the yoga.  In NYC, "Yoga to the People" offers yoga classes (some of which are donation based) at several studios in Brooklyn and Manhattan.  They have a lot of customers.

To gain a better idea of the types of businesses that are addressed in the reviews, I ran Latent Semantic Indexing (LSI) on them.  As its name indicates, LSI helps to index a database of documents by identifying relevant keywords.  However it does this intelligently, associating keywords that are semantically close.  Here is what I obtained for the NYC corpus of reviews:

		*   Gym, Crunch, Machines, Equipment
		*   Bikram, Hot
		*   Massage, Facial, Exhale, Spa
		*   Pilates, Zumba
		*   Lululemon, Store, Pants
		*   Pregnancy, Prenatal
		*   Aerial, Antigravity

The list is pretty self-explanatory, although not without surprise (I didn't know there is such a thing as aerial or antigravity yoga, and that it is popular enough to deserve its own topic).  Aside from Bikram, prenatal, and aerial yoga, the list does not include other types of yoga practice (with typically more exotic names such as Vinyasa, Ashtanga, Anusara,...)  To find these, I did an explicit boolean keyword query, using the yoga types described in William Broad's book on the science of yoga {% sidenote 1 'William J. Broad, "The Science of Yoga: the Risks and the Rewards", Simon & Schuster, 298pp (2012).' %}.  For each yoga style keyword, I estimated the probability for it to appear in at least one review of a yoga business in NYC or LA.  The result is shown in the figure below: {% maincolumn /assets/img/yoga_styles_cloud.jpg 'Figure 2: Yoga styles represented as topic probabilities in the NYC and LA corpora.  Styles above the dashed diagonal line have a higher probability of appearing in an LA review than in a NYC one.  Error bars represent approximate 95% confidence intervals on the <i>difference</i> between the LA and NYC probabilities.  Whenever an error bar does not cross the diagonal line, it is drawn in red to indicate a significant difference (at the 5%, or 2-sigma level) between NYC and LA probabilities.  Non-significant differences are drawn in green.' %}
It is clear that prenatal, Bikram, and Vinyasa yoga are the most popular styles in both NYC and LA, with Vinyasa being significantly more popular in NYC than in LA.  Other significant findings are the popularity of Jivamukti in NYC and Kundalini in LA.  Jivamukti was founded in NYC in 1984, whereas Kundalini has New Age connotations that may explain its popularity on the West Coast.

Finally, to help interested readers find yoga businesses that cater to their needs, I produced an [interactive map for the NYC area](/assets/img/NYC_yoga_studios.html).  Please check it out!