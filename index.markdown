---
layout: main
bodyid: index
title: Adam Thomas
---

{% for post in site.posts %}
  <div><a href='{{ post.url }}'>{{ post.title }}</a></div>
  {% if forloop.last == false %}<hr>{% endif %}
{% endfor %}
