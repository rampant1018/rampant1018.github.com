---
layout: page
title: Welcome to Rampant's Blog!
tagline: Supporting tagline
---
{% include JB/setup %}

## Posts List

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>

