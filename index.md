---
layout: default
title: 笔记索引
---

# 笔记索引

{% assign notes = site.pages | where_exp: "p", "p.path contains '/'" | sort: 'path' %}
{% assign groups = notes | group_by_exp: "p", "p.path | split: '/' | first" %}
{% for group in groups -%}
{%- unless group.name contains '.' %}
## {{ group.name }}

{% for page in group.items -%}
- [{{ page.path | split: '/' | last | remove: '.md' }}]({{ page.url | relative_url }})
{% endfor %}
{% endunless -%}
{% endfor %}
