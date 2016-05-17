---
layout: full-width
title: papers
---

<h1 class="content-listing-header sans">Papers on Statistical Issues</h1>

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
