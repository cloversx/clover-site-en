---
layout: page
title: Library
permalink: /0/
---

{%- assign featured_items = site.pages
  | where: "layout", "content"
  | where_exp: "p", "p.is_archive != true"
  | where: "featured", true
-%}

{%- if featured_items and featured_items.size > 0 -%}
## Featured (Pinned)

{% comment %}
include は トリム無し
Liquid の “空白トリムが効きすぎて、見出し ## ... と <ul> が同じ行に連結されてしまう
{% endcomment %}
{% comment %}{% include content-list-featured.html show_badge=true featured_style=true show_date=false %}{% endcomment %}
{% include content_section.html mode="featured" show_badge=true featured_style=true show_date=false %}

{% comment %}<hr class="section-divider" />{% endcomment %}
{%- endif -%}

## Latest (All contents)

{% comment %}
include は トリム無し
Liquid の “空白トリムが効きすぎて、見出し ## ... と <ul> が同じ行に連結されてしまう
{% endcomment %}
{% comment %}{% include content-list-latest.html show_date=true %}{% endcomment %}
{% include content_section.html mode="latest" show_date=true %}
