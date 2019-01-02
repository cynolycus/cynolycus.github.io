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
"@type": "Blog",
"name": "{{ site.title }}",
"url": "{{ site.url }}{{ page.url }}",
"description": "{{ site.description }}",
"datePublished": "2018-12-01",
"dateModified":"{{ max_mod_date | date: '%Y-%m-%d' }}",
"author":{
   "@type":"Person",
   "name":"{{ site.author }}",
   "sameAs":[
   {%- if site.linkedin_username -%}"https://www.linkedin.com/in/{{ site.linkedin_username | cgi_escape | escape }}", {%- endif -%}
   {%- if site.twitter_username -%}"https://twitter.com/{{ site.twitter_username | cgi_escape | escape }}" {%- endif -%}  ]
},
"publisher":{
  "@type":"Organization",
  "name":"cynolycus",
  "sameAs":[
    {%- if site.linkedin_username -%}"https://www.linkedin.com/in/{{ site.linkedin_username | cgi_escape | escape }}", {%- endif -%}
    {%- if site.twitter_username -%}"https://twitter.com/{{ site.twitter_username | cgi_escape | escape }}", {%- endif -%}
    "https://github.com/cynolycus"],
  "logo": {
         "@type": "ImageObject",
         "name": "cynolycusAvatar",
         "width": "128 px",
         "height": "128 px",
         "url": "{{site.url}}{{site.author_logo}}"
     }
 },
"mainEntity":{ {% include schema/post-list.md posts=site.posts %} }
