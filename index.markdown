---
layout: page_index
title: My coder basement
---
{% for post in site.posts limit:3 %}
{% include partial_post.html %}
{% endfor %}