---
layout: page
title: Tags
---
<!-- Page code borrowed from dbtek and psteadman -->
{% capture site_tags %}{% for tag in site.tags %}{{ tag | first }}{% unless forloop.last %},{% endunless %}{% endfor %}{% endcapture %}
{% assign tag_words = site_tags | split:',' | sort %}

<div class="tags">
    {% for item in (0..site.tags.size) %}{% unless forloop.last %}
      {% capture this_word %}{{ tag_words[item] | strip_newlines }}{% endcapture %}
      <a class="tag" href="/tag/{{ tag_name }}"><code class="highligher-rouge"><nobr>{{ tag_name }}</nobr></code>&nbsp;</a>
   {% endunless %}{% endfor %}
</div>

<div >
  {% for item in (0..site.tags.size) %}{% unless forloop.last %}
    {% capture this_word %}{{ tag_words[item] | strip_newlines }}{% endcapture %}
    <div id="{{ this_word | replace:' ','-' }}-ref">
      <h2 >Posts tagged  with {{ this_word }}</h2>
      <ul >
        {% for post in site.tags[this_word] %}{% if post.title != null %}
          <li ><a href="{{ site.BASE_PATH }}{{post.url}}">{{post.title}}</a> <span >- {{ post.date | date: "%B %d, %Y" }}</span></li>
        {% endif %}{% endfor %}
      </ul>
    </div>
  {% endunless %}{% endfor %}
</div>
