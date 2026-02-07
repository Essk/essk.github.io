---
layout: default
title: Home
---

# {{ site.title }}

{{ site.description }}

## Posts

{% for post in site.posts %}
- [{{ post.title }}]({{ post.url }}) — {{ post.date | date: "%Y-%m-%d" }}
{% endfor %}
