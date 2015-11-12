---
layout: page
---

  <h1 class="content-listing-header sans">Data Science Projects</h1>
  {% newthought 'Pages in this section' %} describe projects I worked on while enrolled in the spring 2015 data science bootcamp at [Metis](http://www.thisismetis.com). Currently projects [Kojak]({{site.url}}/portfolio/5_Kojak.html) and [McNulty]({{site.url}}/portfolio/3_mcnulty.html) are up to date.

  <ul class="content">
    {% for project in site.portfolio reversed %}
      <li class="listing">
        <a href="{{ project.url }}"><h4 class="contrast">{{ project.title }}</h4></a>
        <span class="smaller">{{project.date | date: "%B %-d, %Y"}}</span>  <br/>
        {{ project.excerpt }}
        <hr class="slender">
      </li>
    {% endfor %}
  </ul>