---
layout: page
title: Library
permalink: /0/

title_i18n:
  en: Library
  ja: ライブラリ
---

{%- comment -%}
========================================
Explore links (Search / Tags / Archive)
- Put navigation first
- Keep bilingual labels via site.lang
========================================
{%- endcomment -%}

{% if site.lang == "ja" %}
## 探す / 辿る
{% else %}
## Explore
{% endif %}

<div class="explore-grid">

  <a class="explore-card" href="{{ '/0/search/' | relative_url }}">
    <div class="explore-title">
      {% if site.lang == "ja" %}検索{% else %}Search{% endif %}
    </div>
    <div class="explore-desc">
      {% if site.lang == "ja" %}
      語句・タグ・タイトル・content_idから探す
      {% else %}
      Find by phrases, tags, title, or content_id
      {% endif %}
    </div>
  </a>

  <a class="explore-card" href="{{ '/0/tags/' | relative_url }}">
    <div class="explore-title">
      {% if site.lang == "ja" %}タグ{% else %}Tags{% endif %}
    </div>
    <div class="explore-desc">
      {% if site.lang == "ja" %}
      タグごとに最新版を一覧表示
      {% else %}
      Browse latest versions grouped by tag
      {% endif %}
    </div>
  </a>

  <a class="explore-card" href="{{ '/0/archive/' | relative_url }}">
    <div class="explore-title">
      {% if site.lang == "ja" %}アーカイブ{% else %}Archive{% endif %}
    </div>
    <div class="explore-desc">
      {% if site.lang == "ja" %}
      すべてのバージョン履歴を見る
      {% else %}
      View full version history of all content
      {% endif %}
    </div>
  </a>

</div>

---

{%- assign featured_items = site.pages
  | where: "layout", "content"
  | where_exp: "p", "p.is_archive != true"
  | where: "featured", true
-%}

{%- if featured_items and featured_items.size > 0 -%}
{% if site.lang == "ja" %}
## 固定
{% else %}
## Pinned
{% comment %}## Featured (Pinned){% endcomment %}
{% endif %}

{% comment %}
include は トリム無し
Liquid の “空白トリムが効きすぎて、見出し ## ... と <ul> が同じ行に連結されてしまう
{% endcomment %}
{% comment %}{% include content-list-featured.html show_badge=true featured_style=true show_date=false %}{% endcomment %}
{% include content_section.html mode="featured" show_badge=true featured_style=true show_date=false %}

{% comment %}<hr class="section-divider" />{% endcomment %}
{%- endif -%}

{% if site.lang == "ja" %}
## すべて
{% else %}
## All
{% comment %}## Latest (All contents){% endcomment %}
{% endif %}

{% comment %}
include は トリム無し
Liquid の “空白トリムが効きすぎて、見出し ## ... と <ul> が同じ行に連結されてしまう
{% endcomment %}
{% comment %}{% include content-list-latest.html show_date=true %}{% endcomment %}
{% include content_section.html mode="latest" show_date=true %}

{% comment %}

{% if site.lang == "ja" %}
[- アーカイブ -]({{ "/0/archive/" | relative_url }})
{% else %}
[- Archived -]({{ "/0/archive/" | relative_url }})
{% comment %}See also: [Archive]({{ "/0/archive/" | relative_url }}){% endcomment %}
{% endif %}

{% endcomment %}
