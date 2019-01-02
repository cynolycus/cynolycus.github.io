---
layout: page
title: Series
permalink: /series
description: Sometimes one post isn't enough. Here's each series covered so far.
---
{% if site.series.size > 0 %}
  <ul class="post-list">
    {% for post in site.series %}
    <li>
      {% assign date_format = site.minima.date_format | default: "%b %-d, %Y" %}
      <span class="post-meta">{{ post.date | date: date_format }}</span>
      <h3>
        <a class="post-link" href="{{ post.url | relative_url }}">
          {{ post.title | escape }}
        </a>
      </h3>
      {% if site.show_excerpts %}
        {{ post.excerpt }}
      {% endif %}
    </li>
    {%- endfor -%}
  </ul>
{% endif %}
