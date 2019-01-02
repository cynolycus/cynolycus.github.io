{% assign name_parts = page.slug | split: "-"  %}
{% assign url_parts = page.url | split: "/" %}
{% assign base_url = url_parts | pop | join: "/" %}
{% assign prev_val = page.order | minus: 1 %}
{% if prev_val != 1 %}
  {% assign prev_name = name_parts | pop | push: prev_val | join: "-" %}
{% else %}
  {% assign prev_name = name_parts | pop | join: "-" %}
{% endif %}
{% assign prev_post_name = include.link_title %}
{% assign prev_post_href = base_url | append: "/" | append: prev_name %}
