---
layout: page
title: Archive
subtitle: Previous posts
bigimg: /img/smyrna.jpg
---

## Blog Posts

{% for post in site.posts %}
  * {{ post.date | date_to_string }} &raquo; [ {{ post.title }} ]({{
    post.url }})
{% endfor %}
