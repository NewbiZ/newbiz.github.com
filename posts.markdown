---
layout: page_posts
title: My coder basement
permalink:  /posts/
---
Recents posts
=============
{% for post in site.posts %}
  [{{ post.title }}]({{ post.url }}) [ {{ post.date | date_to_string }} ]
{% endfor %}