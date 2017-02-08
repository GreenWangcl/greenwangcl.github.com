---
layout: page
title: Blogs
header: Post blogs
group: navigation
---
{% include JB/setup %}

<ul class="posts">
{% assign posts_collate = site.posts %}
{% include JB/posts_collate %}
