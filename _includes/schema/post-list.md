"@type": "ItemList",
    "itemListElement": [
      {% for post in include.posts %}
      {
        "@type":"BlogPosting",
        "@id": "https://cynolyc.us{{ post.url }}",
        "name":{{ post.title | escape | jsonify }},
        "headline": {{ post.title | escape | jsonify }},
        "description": {{ post.description | strip_html | jsonify }},
        "keywords":{{ post.tags  | join: ',' | escape | jsonify }},
        "wordCount":"{{ wordcount }}",
        "url":"https://cynolyc.us{{ post.url }}",
        "datePublished":{{ post.date | date: '%Y-%m-%d'  | jsonify }},
        "dateModified":{{ post.last_modified_at | date: '%Y-%m-%d'  | jsonify}},
        "author":{
           "@type":"Person",
           "name":"{{site.author}}",
           "sameAs":[
             "https://twitter.com/cynolycus",
             "https://github.com/cynolycus"]
        },
        "publisher":{
          "@type":"Organization",
          "name":"cynolycus",
          "sameAs":[
            {%- if site.linkedin_username -%}"https://www.linkedin.com/in/{{ site.linkedin_username| cgi_escape | escape }}", {%- endif -%}
            {%- if site.twitter_username -%}"https://twitter.com/{{ site.twitter_username | cgi_escape | escape }}", {%- endif -%}
            "https://github.com/cynolycus"]
         },
          "articleBody": {{ post.content | strip_html | jsonify }}
        }
      {%- if forloop.last == false  -%},{% endif %}
      {% endfor %}
    ]