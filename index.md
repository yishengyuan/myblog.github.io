---
layout: default
title: 笔记索引
---

# 笔记索引

{% assign sorted_pages = site.pages | sort: 'path' %}
{% for page in sorted_pages %}
{%- assign path = page.path -%}
{%- unless path == 'index.md' or path == 'README.md' or path contains '.github/' or path contains '.obsidian/' or path contains '.claude/' -%}
- [{{ path | remove: '.md' }}]({{ page.url | relative_url }})
{% endunless -%}
{% endfor %}
