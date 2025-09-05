---
layout: page
title: publications
permalink: /publications/
nav: true
nav_order: 2
---

{% include bib_search.liquid %}

{% bibliography -f papers --query @* %}