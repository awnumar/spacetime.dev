---
layout: default
title: home
---

# awn umar

[email](mailto:awn@spacetime.dev) :: [pgp](/my/key.txt) :: [github](https://github.com/awnumar) :: [keybase](https://keybase.io/awn)

Programmer mostly [working on](https://github.com/awnumar) security stuff. Mathematics and computer science at the [University of Bristol](https://en.wikipedia.org/wiki/University_of_Bristol).

Here I [post](/posts) about things I'm either working on or thinking about. I don't receive a lot of emails (yet) so I tend to respond to all of them. Feel free to use my inbox to introduce yourself or to fire any questions or concerns at.

## recent posts

[all posts](/posts) :: [subscribe](/feed.xml)

{% for post in site.posts limit: 4 %}
- `{{ post.date | date: "%Y-%m-%d" }}` :: [{{ post.title }}]({{ post.url }}) {% endfor %}

## projects

- [memguard](https://github.com/awnumar/memguard) :: secure software enclave for storage of highly-sensitive values in memory
