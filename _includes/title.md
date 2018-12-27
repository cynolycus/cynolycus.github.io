{% if page.title == "none" OR page.title.size == 0 %}
  {{ site.title | append: site.title_sep | append: site.tagline }}
{% else %}
  {{ page.title | append: site.title_sep | append: site.title }}
{% endif %}
