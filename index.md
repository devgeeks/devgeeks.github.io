---
layout: page
title: devgeeks.org
tagline: All over the place
---
{% include JB/setup %}

{% for post in site.posts %}
  <h2><a href="{{ post.url }}">{{ post.title }}</a></h2>
  <div style="text-align: right; margin-bottom: 1em;"><small><em>{{ post.date | date_to_string }}</em></small></div>
  <div>
    {{ post.content | split:'<!--break-->' | first }}
    {% if post.content contains '<!--break-->' %}
      <a href="{{ post.url }}">read more &raquo;</a>
    {% endif %}
  </div>
  <div style="margin-bottom: 3em;"></div>
{% endfor %}
