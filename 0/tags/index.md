---
layout: page
title: Tags
permalink: /0/tags/
pinned_tags:
  - environment-alignment
  - ai-safety
  - AI安全性
  - 評価構造
  - GitHub Pages
---

This page groups current contents by tags.

---

{% assign items = site.pages
  | where: "layout", "content"
  | where_exp: "p", "p.is_archive != true"
  | sort: "last_updated"
  | reverse
%}

{% assign all_tags = "" | split: "" %}
{% for p in items %}
  {% if p.tags %}
    {% for t in p.tags %}
      {% assign all_tags = all_tags | push: t %}
    {% endfor %}
  {% endif %}
{% endfor %}

{% assign uniq_tags = all_tags | uniq | sort %}

{% assign pinned = page.pinned_tags | default: "" | split: "" %}

{%- comment -%}
Build ordered tag list:
1) pinned in given order (only if actually used)
2) remaining tags (sorted)
{%- endcomment -%}

{% assign ordered = "" | split: "" %}

{% if pinned and pinned.size > 0 %}
  {% for t in pinned %}
    {% if uniq_tags contains t %}
      {% assign ordered = ordered | push: t %}
    {% endif %}
  {% endfor %}
{% endif %}

{% for t in uniq_tags %}
  {% unless ordered contains t %}
    {% assign ordered = ordered | push: t %}
  {% endunless %}
{% endfor %}

{% if ordered and ordered.size > 0 %}

{% for tag in ordered %}

## {{ tag }}

<ul class="content-list">
{% for p in items %}
  {% if p.tags and p.tags contains tag %}
    {% include content-card.html page=p show_date=true %}
  {% endif %}
{% endfor %}
</ul>

---

{% endfor %}

{% else %}

No tags yet.

{% endif %}
