---
layout: page
title: Essays
permalink: /essays/
---

<ul class="essay-list">

{%- comment -%}
1) 現行版のみ抽出
{%- endcomment -%}
{%- assign items = site.pages
  | where: "layout", "essay"
  | where_exp: "p", "p.is_archive != true"
-%}

{%- comment -%}
2) featured と non-featured に分離
{%- endcomment -%}
{%- assign featured_items = items | where: "featured", true -%}
{%- assign normal_items = items | where_exp: "p", "p.featured != true" -%}

{%- comment -%}
3) featured の並び順
   - featured_rank があれば昇順
   - 無ければ日付で新しい順
{%- endcomment -%}
{%- assign ranked = featured_items | where_exp: "p", "p.featured_rank != nil" | sort: "featured_rank" -%}
{%- assign unranked_featured = featured_items | where_exp: "p", "p.featured_rank == nil" -%}

{%- for p in unranked_featured -%}
  {%- if p.last_updated and p.last_updated != "" -%}
    {%- assign p.sort_date = p.last_updated -%}
  {%- elsif p.first_published and p.first_published != "" -%}
    {%- assign p.sort_date = p.first_published -%}
  {%- else -%}
    {%- assign p.sort_date = "0000-00-00" -%}
  {%- endif -%}
{%- endfor -%}

{%- assign unranked_featured_sorted = unranked_featured | sort: "sort_date" | reverse -%}

{%- comment -%}
4) 通常記事も日付でソート
{%- endcomment -%}
{%- for p in normal_items -%}
  {%- if p.last_updated and p.last_updated != "" -%}
    {%- assign p.sort_date = p.last_updated -%}
  {%- elsif p.first_published and p.first_published != "" -%}
    {%- assign p.sort_date = p.first_published -%}
  {%- else -%}
    {%- assign p.sort_date = "0000-00-00" -%}
  {%- endif -%}
{%- endfor -%}

{%- assign normal_sorted = normal_items | sort: "sort_date" | reverse -%}

{%- comment -%}
5) 表示
{%- endcomment -%}

{%- for p in ranked -%}
  <li>
    <a href="{{ site.baseurl }}{{ p.url }}">{{ p.title }}</a>
    {%- if p.summary and p.summary != "" -%}
      <div class="muted">{{ p.summary }}</div>
    {%- endif -%}
  </li>
{%- endfor -%}

{%- for p in unranked_featured_sorted -%}
  <li>
    <a href="{{ site.baseurl }}{{ p.url }}">{{ p.title }}</a>
    {%- if p.summary and p.summary != "" -%}
      <div class="muted">{{ p.summary }}</div>
    {%- endif -%}
  </li>
{%- endfor -%}

{%- for p in normal_sorted -%}
  <li>
    <a href="{{ site.baseurl }}{{ p.url }}">{{ p.title }}</a>
    {%- if p.summary and p.summary != "" -%}
      <div class="muted">{{ p.summary }}</div>
    {%- endif -%}
  </li>
{%- endfor -%}

</ul>
