{% unless post %}
  {% assign post = page %}
{% endunless %}
{% if post.order and post.order != 1 %}
  {% assign page_name =  post.slug | split: "-" | pop | join: "-" | downcase %}
{% else %}
  {% assign page_name = post.slug  | downcase %}
{% endif %}
{% assign image_fallback =  site.url | append: site.default_image %}
{% assign images = site.static_files | where: "image", true %}
{% for image in images %}
  {% assign image_name = image.basename | slugify | downcase %}
  {% if image_name == page_name %}
    {% assign page_image = site.url | append: image.path %}
  {% endif %}
{% endfor %}
