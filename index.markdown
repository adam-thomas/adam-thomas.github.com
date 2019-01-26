---
layout: main
bodyid: index
title: Adam Thomas
---

{% for post in site.posts %}
  <h2><a href='{{ post.url }}'>{{ post.title }}</a></h2>
  {% if forloop.last == false %}<hr>{% endif %}
{% endfor %}
