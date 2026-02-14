---
layout: page
title: Essays
permalink: /essays/
---

{%- assign featured_items = site.pages
  | where: "layout", "essay"
  | where_exp: "p", "p.is_archive != true"
  | where: "featured", true
-%}

{%- if featured_items and featured_items.size > 0 -%}
## Featured (Pinned)

{% comment %}
include は トリム無し
Liquid の “空白トリムが効きすぎて、見出し ## ... と <ul> が同じ行に連結されてしまう
{% endcomment %}
{% include essay-list-featured.html show_badge=true featured_style=true show_date=false %}

{% comment %}<hr class="section-divider" />{% endcomment %}
{%- endif -%}

## Latest (All essays)

{% comment %}
include は トリム無し
Liquid の “空白トリムが効きすぎて、見出し ## ... と <ul> が同じ行に連結されてしまう
{% endcomment %}
{% include essay-list-latest.html show_date=true %}
