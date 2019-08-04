{% if page.title == "none" OR page.title.size == 0 %}
{% assign page_title = site.title | append: site.title_sep | append: site.tagline %}
{% else %}
{% assign page_title = page.title | append: site.title_sep | append: site.title %}
{% endif %}
