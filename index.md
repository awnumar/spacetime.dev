---
layout: default
title: home
---

# awn umar

[email](mailto:awn@spacetime.dev) :: [pgp](x/key.txt) :: [github](https://github.com/awnumar) :: [twitter](https://twitter.com/awnumar)

Programmer mostly [working on](https://github.com/awnumar) security stuff. Mathematics and computer science at the [University of Bristol](https://en.wikipedia.org/wiki/University_of_Bristol).

Here I be plugging my projects and occasionally posting whatever I’m thinking about. I don’t receive a lot of emails (yet), so I tend to respond to all of them. If you have any questions or criticisms, feel free to contact me.

## posts

[all posts](posts) :: [subscribe](feed.xml)

{% for post in site.posts limit: 3 %}
- `{{ post.date | date: "%Y-%m-%d" }}` :: [{{ post.title }}]({{ post.url }}) {% endfor %}

## projects

- [memguard](https://github.com/awnumar/memguard) :: secure software enclave for storage of highly-sensitive values in memory