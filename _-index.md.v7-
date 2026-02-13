---
layout: page
title: ""
---

# Clover Essays
{% comment %}### - Clover Essays -{% endcomment %}

Welcome.

{% comment %}A quiet canonical archive for long-term preservation.{% endcomment %}

<div class="home-links">
  <a class="btn" href="{{ site.baseurl }}/essays/">Essays</a> {% comment %}Browse essays{% endcomment %}
  <a class="btn ghost" href="{{ site.baseurl }}/about/">About</a>
</div>

{% comment %}
- Read: **[Essays]({{ site.baseurl }}/essays/)**
- About: **[About]({{ site.baseurl }}/about/)**
{% endcomment %}

{% comment %}---{% endcomment %}

## Featured {% comment %}（代表作）{% endcomment %}
{% comment %}#### Featured {% comment %}（代表作）{% endcomment %}{% endcomment %}

{% comment %}- (Add links to representative essays here){% endcomment %}
{% comment %}- [AI Isn’t Dangerous. Putting AI Inside an “Evaluation Structure” Is.]({{ site.baseurl }}/essays/ai-evaluation-structure/){% endcomment %}

{% comment %}
featured_rank を付けたものは、その順位どおり固定
それ以外の featured は「新しい順」
Featured が0件でも崩れない（リストが空になるだけ）
{% endcomment %}
<ul class="essay-list">
{%- assign items = site.pages
  | where: "layout", "essay"
  | where_exp: "p", "p.is_archive != true"
  | where: "featured", true
-%}

{%- comment -%}
並び順：
- featured_rank があればそれで昇順
- 無い場合は sort_date（last_updated→first_published→0000-00-00）で新しい順
{%- endcomment -%}

{%- assign ranked = items | where_exp: "p", "p.featured_rank != nil" | sort: "featured_rank" -%}
{%- assign unranked = items | where_exp: "p", "p.featured_rank == nil" -%}

{%- for p in unranked -%}
  {%- if p.last_updated and p.last_updated != "" -%}
    {%- assign p.sort_date = p.last_updated -%}
  {%- elsif p.first_published and p.first_published != "" -%}
    {%- assign p.sort_date = p.first_published -%}
  {%- else -%}
    {%- assign p.sort_date = "0000-00-00" -%}
  {%- endif -%}
{%- endfor -%}

{%- assign unranked_sorted = unranked | sort: "sort_date" | reverse -%}

{%- for p in ranked -%}
  <li>
    <a href="{{ site.baseurl }}{{ p.url }}">{{ p.title }}</a>
    {%- if p.summary and p.summary != "" -%}
      <div class="muted">{{ p.summary }}</div>
    {%- endif -%}
  </li>
{%- endfor -%}

{%- for p in unranked_sorted -%}
  <li>
    <a href="{{ site.baseurl }}{{ p.url }}">{{ p.title }}</a>
    {%- if p.summary and p.summary != "" -%}
      <div class="muted">{{ p.summary }}</div>
    {%- endif -%}
  </li>
{%- endfor -%}
</ul>

## Latest {% comment %}（最新）{% endcomment %}
{% comment %}#### Latest {% comment %}（最新）{% endcomment %}{% endcomment %}

{% comment %}- (Add links to latest essays here){% endcomment %}
{% comment %}- [AI Isn’t Dangerous. Putting AI Inside an “Evaluation Structure” Is.]({{ site.baseurl }}/essays/ai-evaluation-structure/){% endcomment %}

{% comment %}
注意点（1つだけ）
この sort_date 方式は 記事数が増えても普通に動くけど、Liquidの制約で「配列内の各要素にsort_dateを付ける」作業をしてる。GitHub Pagesでも大丈夫な書き方だけど、もし将来おかしな挙動が出たら、次善策として「日付を文字列に統一してソート」でもOK（YYYY-MM-DD は文字列でもソートが崩れない）。
{% endcomment %}
<ul class="essay-list">
{%- assign items = site.pages
  | where: "layout", "essay"
  | where_exp: "p", "p.is_archive != true"
-%}

{%- comment -%}
sortキーの安定化：last_updated → first_published → 0000-00-00
{%- endcomment -%}
{%- for p in items -%}
  {%- if p.last_updated and p.last_updated != "" -%}
    {%- assign p.sort_date = p.last_updated -%}
  {%- elsif p.first_published and p.first_published != "" -%}
    {%- assign p.sort_date = p.first_published -%}
  {%- else -%}
    {%- assign p.sort_date = "0000-00-00" -%}
  {%- endif -%}
{%- endfor -%}

{%- assign latest = items | sort: "sort_date" | reverse -%}

{%- for p in latest limit:3 -%}
  <li>
    <a href="{{ site.baseurl }}{{ p.url }}">{{ p.title }}</a>
    {%- if p.summary and p.summary != "" -%}
      <div class="muted">{{ p.summary }}</div>
    {%- endif -%}
    {%- if p.last_updated and p.last_updated != "" -%}
      <div class="muted"><small>Updated: {{ p.last_updated }}</small></div>
    {%- elsif p.first_published and p.first_published != "" -%}
      <div class="muted"><small>Published: {{ p.first_published }}</small></div>
    {%- endif -%}
  </li>
{%- endfor -%}
</ul>

