{% assign name_parts = page.slug | split: "-"  %}
{% assign url_parts = page.url | split: "/" %}
{% assign base_url = url_parts | pop | join: "/" %}
{% assign next_val = page.order | plus: 1 %}
{% if page.order != 1 %}
  {% assign next_name = name_parts | pop | push: next_val | join: "-" %}
{% else %}
  {% assign next_name = name_parts | push: next_val | join: "-" %}
{% endif %}
{% assign next_post_name = include.link_title %}
{% assign next_post_href = base_url | append: "/" | append: next_name %}
