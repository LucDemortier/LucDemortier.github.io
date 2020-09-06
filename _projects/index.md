---
layout: full-width
title: projects
---

<h1 class="content-listing-header sans">Data Science Projects</h1>
{% newthought 'Pages in this section' %} describe projects I worked on while enrolled in the spring 2015 data science bootcamp at [Metis](https://www.thisismetis.com). The project names were taken from famous detectives on TV shows: Theo Kojak, Jessica Fletcher, Jimmy McNulty, John Luther, and Olivia Benson, but the projects themselves were not related to the shows.  I reckon the idea was to suggest commonalities between good data scientists and good detectives...

<ul class="content-listing">
  {% for project in site.projects reversed %}
      {% unless project.title == 'projects' %}
          <li class="listing">
          <hr class="slender">
          <a href="{{ project.url }}"><h3 class="contrast">{{ project.title }}</h3></a>
          <br><span class="smaller">{{project.date | date: "%B %-d, %Y"}}</span>  <br/>
          <div>
          {{ project.excerpt }}
          {% if project.excerpt_urls %}
              {% for excerpt_url in project.excerpt_urls %}
                  <a href="{{site.url}}/assets/img/{{excerpt_url[1]}}">{{excerpt_url[0]}}</a>{% if forloop.last %}.{% else %}, {% endif %}
              {% endfor %}
          {% endif %}
          </div>
          </li>
      {% endunless %}
  {% endfor %}
</ul>
