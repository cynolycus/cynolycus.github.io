{% assign page_title = site.data.tags[page.slug] | default: page.slug | append: site.title_sep | append: site.title | prepend: "Tag: " %}
