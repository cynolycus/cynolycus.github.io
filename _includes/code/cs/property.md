{%- assign accessModifier = include.accessModifier | default: 'public' -%}
{%- assign accessModifier = include.accessModifier | default: 'public' -%}
{%- assign propType = include.propType | default: 'object' -%}
{%- assign propTypeAttr = include.propTypeAttr | default: "k" -%}
{%- assign propName = include.propName | default: 'Object' -%}
{% assign autoGet = include.autoGet | default: true %}
{% assign autoSet = include.autoSet | default: true %}
`{{ accessModifier }}`{:k} `{{ propType }}`{:{{propTypeAttr}}} {{ propName }} { {% if autoGet %}`get`{:k}; {% endif %}{% if autoSet %}`set`{:k}; {% endif %} }