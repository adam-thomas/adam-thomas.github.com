---
layout: main
bodyid: me
title: Adam Thomas
---

{% for post in site.posts %}
  <div><a href='{{ post.url }}'>{{ post.excerpt }}</a></div>
  {% if forloop.last == false %}<hr>{% endif %}
{% endfor %}
