---
layout: default
title: posts
---

# posts

{% for post in site.posts %}
- {{ post.date | date: "%Y-%m-%d" }} : : [{{ post.title }}]({{ post.url }}) {% endfor %}

[subscribe with rss/atom](/feed.xml)
