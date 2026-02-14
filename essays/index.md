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
include は トリム無し（{% include ... %}）
include 呼び出しは {%- をやめて {% にする
{% endcomment %}
{% include essay-list-featured.html show_badge=true featured_style=true show_date=false %}

<hr class="section-divider" />
{%- endif -%}

## Latest (All essays)

{% comment %}
include は トリム無し（{% include ... %}）
include 呼び出しは {%- をやめて {% にする
{% endcomment %}
{% include essay-list-latest.html show_date=true %}
