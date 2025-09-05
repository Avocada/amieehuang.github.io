---
layout: page
title: blog
permalink: /blog/
nav: true
nav_order: 3
---

{% comment %}
Option A: simple built-in list of latest posts (uses \_includes/latest_posts.liquid)
{% endcomment %}
{% include latest_posts.liquid show_excerpts=true limit=10 %}

{% comment %}
Option B: if you want a fully custom list, uncomment this block and delete the include above.

<ul class="post-list">
  {% for post in site.posts %}
    <li class="mb-4">
      <h3 class="mb-1"><a href="{{ post.url | relative_url }}">{{ post.title }}</a></h3>
      <small class="text-muted">{{ post.date | date: "%b %-d, %Y" }}</small>
      <p class="mt-1">{{ post.excerpt | strip_html | truncate: 180 }}</p>
    </li>
  {% endfor %}
</ul>
{% endcomment %}
