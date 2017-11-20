---
layout: page
title: AAPCS
permalink: /aapcs/
---
{% for item in site.aapcs %}
  <h2>{{ item.title }}</h2>
  <p>{{ item.description }}</p>
  <p><a href="{{ item.url }}">{{ item.title }}</a></p>
{% endfor %}
