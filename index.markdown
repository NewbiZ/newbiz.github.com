---
layout: page_index
title: My coder basement
---
{% for post in paginator.posts %}
	{% include page_post.html %}
{% endfor %}