---
layout: full-width
title: papers
---

<h1 class="content-listing-header sans">Papers</h1>
This page contains two sections, the first one listing papers on [statistical issues](#StatisticalIssues) and the second one papers on [experimental particle physics](#ExperimentalParticlePhysics).
<hr class="slender">
<p>&nbsp;</p>

<a name="StatisticalIssues"></a>
<h2 class="content-listing-header sans">Papers on Statistical Issues<a href= "#TopOfPage" style="float: right; font-style: normal; font-weight: normal; font-size: 50%; font-family: Palatino; line-height: normal;">Back to Top</a></h2>
{% newthought 'Pages in this section' %} summarize papers I wrote on various statistical problems. While this work was mainly intended for an audience of experimental particle physicists, its scope is more general.  The topics investigated cover hypothesis testing, confidence interval construction, fitting techniques, nuisance parameters, frequentist versus Bayesian statistics, and more.

<ul class="content-listing">
  {% for paper in site.papers reversed %}
      {% unless paper.title == 'papers' %}
          <li class="listing">
          <hr class="slender">
          <a href="{{ paper.url }}"><h3 class="contrast">{{ paper.title }}</h3></a>
          <br><span class="smaller">{{paper.date | date: "%B %-d, %Y"}}</span>  <br/>
          <div>
          {{ paper.excerpt }}
          {% if paper.excerpt_urls %}
              {% for excerpt_url in paper.excerpt_urls %}
                  <a href="{{site.url}}/assets/img/{{excerpt_url[1]}}">{{excerpt_url[0]}}</a>{% if forloop.last %}.{% else %}, {% endif %}
              {% endfor %}
          {% endif %}
          </div>
          </li>
      {% endunless %}
  {% endfor %}
</ul>

<a name="ExperimentalParticlePhysics"></a>
<h2 class="content-listing-header sans">Papers on Experimental Particle Physics
<a href= "#TopOfPage" style="float: right; font-style: normal; font-weight: normal; font-size: 50%; font-family: Palatino; line-height: normal;">Back to Top</a></h2>
{% newthought 'A list of all refereed publications' %} can be obtained via INSPIRE, please click <a href="https://inspirehep.net/search?p=find a demortier and tc p">here</a> (a total of 1191 as of 16 October 2017).

You can also list by **journal**:
- <a href="https://inspirehep.net/search?p=find j Phys.Rev.Lett. and a demortier">Physical Review Letters,</a>
- <a href="https://inspirehep.net/search?p=find j PHRVA and a demortier">Physical Review C or D,</a>
- <a href="https://inspirehep.net/search?p=find j Phys.Lett.B and a demortier">Physics Letters B,</a>
- <a href="https://inspirehep.net/search?p=find j JHEP and a demortier">Journal of High Energy Physics,</a>
- <a href="https://inspirehep.net/search?p=find j Eur.Phys.J.C and a demortier">European Physics Journal C,</a>
- <a href="https://inspirehep.net/search?p=find j J.Phys. and a demortier">Journal of Physics G,</a>
- <a href="https://inspirehep.net/search?p=find j Nucl.Instrum.Meth. and a demortier">Nuclear Instruments and Methods in Physics Research,</a>
- <a href="https://inspirehep.net/search?p=find j JINST and a demortier">Journal of Instrumentation,</a>

or by **experiment**:
- <a href="https://inspirehep.net/search?p=find a demortier and cn cdf and tc p">CDF,</a>
- <a href="https://inspirehep.net/search?p=find a demortier and cn cms and tc p">CMS.</a>
