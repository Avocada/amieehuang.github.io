---
layout: page
title: publications
permalink: /publications/
nav: true
nav_order: 2
---

{% include bib_search.liquid %}

### selected

{% bibliography -f papers -q @*[selected=true] %}

### all

{% bibliography -f papers %}
