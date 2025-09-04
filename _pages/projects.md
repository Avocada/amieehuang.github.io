---
layout: page
title: projects
permalink: /projects/
description: A growing collection of your cool projects.
nav: true
nav_order: 3
display_categories: [work, fun]
horizontal: false
---

{%- assign EMPTY = "" | split: "" -%}
{%- assign ALL_PROJECTS = site.projects
      | default: site.collections.projects.docs
      | default: EMPTY -%}

<div class="projects">
  {% if site.enable_project_categories and page.display_categories %}
    {% for category in page.display_categories %}
      <a id="{{ category }}" href=".#{{ category }}">
        <h2 class="category">{{ category }}</h2>
      </a>

      {%- assign categorized_projects = ALL_PROJECTS | where: 'category', category | default: EMPTY -%}
      {%- assign sorted_projects = categorized_projects | sort: 'importance' -%}

      {% if page.horizontal %}
        <div class="container">
          <div class="row row-cols-1 row-cols-md-2">
            {% for project in sorted_projects %}
              {% include projects_horizontal.liquid %}
            {% endfor %}
          </div>
        </div>
      {% else %}
        <div class="row row-cols-1 row-cols-md-3">
          {% for project in sorted_projects %}
            {% include projects.liquid %}
          {% endfor %}
        </div>
      {% endif %}
    {% endfor %}

{% else %}
{%- assign sorted_projects = ALL_PROJECTS | sort: 'importance' -%}
{% if page.horizontal %}
<div class="container">
<div class="row row-cols-1 row-cols-md-2">
{% for project in sorted_projects %}
{% include projects_horizontal.liquid %}
{% endfor %}
</div>
</div>
{% else %}
<div class="row row-cols-1 row-cols-md-3">
{% for project in sorted_projects %}
{% include projects.liquid %}
{% endfor %}
</div>
{% endif %}
{% endif %}

</div>
