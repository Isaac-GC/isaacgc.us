---
layout: default
title: Blog
---

{% assign posts = site.posts | sort: 'date' | reverse %}
{% for post in posts %}
  <p><a href="{{ post.url }}">{{ post.title }}</a> -- (Posted {{ post.date | date: "%d %b %Y" }}
{% endfor %}