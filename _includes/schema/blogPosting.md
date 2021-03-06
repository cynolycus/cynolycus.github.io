 "@context":"http://schema.org",
 "@type":"BlogPosting",
 "@id": "https://cynolyc.us{{ page.url }}",
 {% assign article_body = page.content | split: site.content_seperator %}
 "articleBody": {{ article_body[1] | default: page.content | strip_html | strip | jsonify }},
 "name":{{ page.title | jsonify }},
 "headline":{{ page.title | jsonify }},
  {% assign keywords = "" | split: "" %}
  {% for tag in page.tags %}
    {% assign tag_name = site.data.tags[tag] | default: tag %}
    {% assign keywords = keywords | push: tag_name %}
  {% endfor %}
 "keywords":{{ keywords | join: ', ' | jsonify }},
 "wordCount":{{ page.content | number_of_words | jsonify }},
 "url":"https://cynolyc.us{{ page.url }}",
 "datePublished":"{{ page.date | date: '%Y-%m-%d'  }}",
 "dateModified":"{{ page.last_modified_at | date: '%Y-%m-%d' }}",
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
 "mainEntityOfPage":{
    "@type":"BlogPosting",
    "@id":"https://cynolyc.us{{ page.url }}"
 }
