---
layout: default
title: home
---

# awn umar

[email](mailto:awn@spacetime.dev){:rel="me"} :: [pgp](/my/key.txt){:rel="pgpkey"} :: [github](https://github.com/awnumar){:rel="me"} :: [keybase](https://keybase.io/awn){:rel="me"}

Programmer mostly [working on](https://github.com/awnumar) security stuff. Mathematics and computer science at the [University of Bristol](https://en.wikipedia.org/wiki/University_of_Bristol).

Here I [post](/posts) about things I'm working on or thinking about. I don't receive too many emails so I tend to respond to all of them. For sensitive information, my pgp key is `0xD4C316B83295F6FF`, available [here](/my/key.txt).

## recent posts

[all posts](/posts) :: [subscribe](/feed.xml)

{% for post in site.posts limit: 4 %}
- `{{ post.date | date: "%Y-%m-%d" }}` :: [{{ post.title }}]({{ post.url }}){% endfor %}

## projects

{% for project in site.data.projects %}
- [{{ project.name }}]({{ project.url }}) :: {{ project.description }}{% endfor %}
