---
layout: page
---

  <h1 class="content-listing-header sans">Data Science Projects</h1>
  {% newthought 'Pages in this section' %} describe projects I worked on while enrolled in the spring 2015 data science bootcamp at [Metis](http://www.thisismetis.com). The project names are taken from famous detectives on TV shows: Theo Kojak, Jessica Fletcher, Jimmy McNulty, John Luther, and Olivia Benson, but the projects themselves are not related to the shows.  Why, the idea is to suggest commonalities between good data scientists and good detectives, and I certainly prefer this to being compared to a unicorn!
  <ul class="content">
    {% for project in site.portfolio reversed %}
      <li class="listing">
        <a href="{{ project.url }}"><h4 class="contrast">{{ project.title }}</h4></a>
        {% if project.avatar %}
          <span class="marginnote"><img class="fullwidth" src="/assets/img/avatars/{{project.avatar}}" /></span>
        {% endif %}
        <span class="smaller">{{project.date | date: "%B %-d, %Y"}}</span>  <br/>
        {{ project.excerpt }}
        {% if project.excerpt_urls %}
          {% for excerpt_url in project.excerpt_urls %}
            <a href="{{site.url}}/assets/img/{{excerpt_url[1]}}">{{excerpt_url[0]}}</a>{% if forloop.last %}.{% else %}, {% endif %}
          {% endfor %}
        {% endif %}
        <hr class="slender">
      </li>
    {% endfor %}
  </ul>