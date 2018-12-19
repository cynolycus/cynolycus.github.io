{% capture max_mod_date %}{{0 | date: %s}}{% endcapture %}
{% for post in site.posts %}
  {% capture post_date %}{{post.date | date: '%s'}}{% endcapture %}
  {% capture post_updated %}{{post.last_modified_at | date: '%s'}}{% endcapture %}
    {% if post_updated > max_mod_date %}
      {% assign max_mod_date = post_updated %}
    {% endif %}
    {% if post_date > max_mod_date %}
      {% assign max_mod_date = post_date %}
    {% endif %}
{% endfor %}
"@context": "http://schema.org",
"@type": "WebPage",
"name": "{{ site.title }}",
"url": "{{ site.url }}",
"description": "{{site.description }}",
"datePublished": "2018-12-01",
"dateModified":"{{ max_mod_date | date: '%Y-%m-%d' }}"
