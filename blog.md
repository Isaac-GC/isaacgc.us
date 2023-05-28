---
layout: default
title: Blog
---

# <center>Blog Posts</center>

<dl>
{% assign posts = site.posts | sort: 'date' | reverse %}
{% for post in posts limit: 5 %}
  <!-- <center>  -->
    <dt>
      <a href="{{ post.url }}">{{ post.title }}</a> -- {{ post.date | date: "%d %b %Y" }}
    </dt>
    <dd>
      {{ post.excerpt }}
    </dd>
  <!-- </center> -->
  
{% endfor %}
</dl>