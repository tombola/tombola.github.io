---
layout: category-index
title: "Educational Technology"
date: 
modified:
excerpt:
tags: []
category: edtech
image:
  feature:
---

<!--<h1>{{ page.category }}</h1>-->

These are the posts so far that relate to educational technologies.

This blog is aimed at a more technical readership, so the #edtech posts listed below are solely the ones that cross that geeky line. For more general or conceptual output take a look on <a href="http://twitter.com/{{ author.twitter }}">Twitter</a> or <a href="https://medium.com/@{{ author.medium }}" target="_blank">Medium</a>.



{% for post in site.categories.['edtech'] %}  
  {% include _teaser.html %}
{% endfor %}