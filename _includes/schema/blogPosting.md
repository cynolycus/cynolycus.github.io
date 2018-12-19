 "@context":"http://schema.org",
 "@type":"BlogPosting",
 "@id": "https://cynolyc.us{{ page.url }}",
 "articleBody": {{ page.content | strip_html | jsonify }}
 "name":"{{ page.title | escape }}",
 "headline":"{{ page.title | escape }}",
 "keywords":"{{ page.tags | join: ',' | escape }}",
 "wordCount":"{{ page.content | number_of_words }}",
 "url":"https://cynolyc.us{{ page.url }}",
 "datePublished":"{{ page.date | date: '%Y-%m-%d'  }}",
 "dateModified":"{{ page.last_modified_at | date: '%Y-%m-%d' }}",
 "author":{
    "@type":"Person",
    "name":"Benjamin Sochor",
    "sameAs":[
      "https://twitter.com/cynolycus",
      "https://github.com/cynolycus"]
 },
 "publisher":{
   "@type":"Organization",
   "name":"cynolycus",
   "sameAs":[
     {%- if site.linkedin_username -%}"https://www.linkedin.com/in/{{ site.linkedin_username | cgi_escape | escape }}", {%- endif -%}
     {%- if site.twitter_username -%}"https://twitter.com/{{ site.twitter_username | cgi_escape | escape }}", {%- endif -%}
     "https://github.com/cynolycus"]
  },
 "mainEntityOfPage":{
    "@type":"BlogPosting",
    "@id":"https://cynolyc.us{{ page.url }}"
 }