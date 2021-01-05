---
layout: default
title: home
---

# awn umar

<div class="nav">
<a href="mailto:awn@spacetime.dev" rel="me">email</a> : : <a href="{{ site.url }}/public-key.pgp" rel="me">pgp</a> : : <a href="https://github.com/awnumar" rel="me">github</a> : : <a href="https://read.cv/awn" rel="me">cv</a>
</div>

Programmer mostly [working on](https://github.com/awnumar) security stuff. Mathematics and computer science at the [University of Bristol](https://en.wikipedia.org/wiki/University_of_Bristol). Here I [post](/posts) about things I'm working on or thinking about.

## recent posts

<div class="nav">
<a href="/posts">all posts</a> : : <a href="/feed.xml">subscribe</a>
</div>

{% for post in site.posts limit: 4 %}
- {{ post.date | date: "%Y-%m-%d" }} : : [{{ post.title }}]({{ post.url }}){% endfor %}

## projects

{% for project in site.data.projects %}
- [{{ project.name }}]({{ project.url }}) : : {{ project.description }}{% endfor %}

## papers

{% for paper in site.data.papers %}
> {{ paper.title }} [[pdf]({{ paper.url }})]
> 
> {{ paper.abstract }}

{% endfor %}
