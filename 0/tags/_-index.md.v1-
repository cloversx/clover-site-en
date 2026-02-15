---
layout: page
title: Tags
permalink: /0/tags/
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

{% if uniq_tags and uniq_tags.size > 0 %}

{% for tag in uniq_tags %}

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
