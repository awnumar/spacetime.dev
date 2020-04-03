---
layout: default
title: posts
---

# posts

{% for post in site.posts %}
- `{{ post.date | date: "%Y-%m-%d" }}` :: [{{ post.title }}]({{ site.url }}/{{ post.url }}.html) {% endfor %}

[subscribe with rss]({{ site.url }}/feed.xml)
