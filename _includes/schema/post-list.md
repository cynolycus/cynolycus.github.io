"@type": "ItemList",
    "itemListElement": [
      {% for post in include.posts %}
      {
        {% assign keywords = "" | split: "" %}
        {% for tag in post.tags %}
          {% assign tag_name = site.data.tags[tag] | default: tag %}
          {% assign keywords = keywords | push: tag_name %}
        {% endfor %}
        "@type":"BlogPosting",
        "@id": "https://cynolyc.us{{ post.url }}",
        "name":{{ post.title | escape | jsonify }},
        "headline": {{ post.title | escape | jsonify }},
        "description": {{ post.description | strip_html | jsonify }},
        "keywords":"{{ keywords | join: ", " | escape }}",
        "wordCount":"{{ wordcount }}",
        "url":"https://cynolyc.us{{ post.url }}",
        "datePublished":{{ post.date | date: '%Y-%m-%d'  | jsonify }},
        "dateModified":{{ post.last_modified_at | date: '%Y-%m-%d'  | jsonify}},
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
          }
        }
      {%- if forloop.last == false  -%},{% endif %}
      {% endfor %}
    ]
