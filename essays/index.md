---
layout: page
title: Essays
permalink: /essays/
---

{%- assign items = site.pages
  | where: "layout", "essay"
  | where_exp: "p", "p.is_archive != true"
-%}

{%- assign featured_items = items | where: "featured", true -%}
{%- assign normal_items = items | where_exp: "p", "p.featured != true" -%}

{%- comment -%} Featured: ranked then unranked-by-date {%- endcomment -%}
{%- assign featured_ranked = featured_items | where_exp: "p", "p.featured_rank != nil" | sort: "featured_rank" -%}
{%- assign featured_unranked = featured_items | where_exp: "p", "p.featured_rank == nil" -%}

{%- for p in featured_unranked -%}
  {%- if p.last_updated and p.last_updated != "" -%}
    {%- assign p.sort_date = p.last_updated -%}
  {%- elsif p.first_published and p.first_published != "" -%}
    {%- assign p.sort_date = p.first_published -%}
  {%- else -%}
    {%- assign p.sort_date = "0000-00-00" -%}
  {%- endif -%}
{%- endfor -%}
{%- assign featured_unranked_sorted = featured_unranked | sort: "sort_date" | reverse -%}

{%- comment -%} Normal: date sort {%- endcomment -%}
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

{%- comment -%} ===== Render ===== {%- endcomment -%}

{% if featured_items and featured_items.size > 0 %}
## Featured (Pinned)

<ul class="essay-list">
  {%- for p in featured_ranked -%}
    <li class="essay-card is-featured">
      <div class="card-top">
        <a href="{{ site.baseurl }}{{ p.url }}">{{ p.title }}</a>
        <span class="badge">Featured</span>
      </div>
      {%- if p.summary and p.summary != "" -%}
        <div class="muted">{{ p.summary }}</div>
      {%- endif -%}
    </li>
  {%- endfor -%}

  {%- for p in featured_unranked_sorted -%}
    <li class="essay-card is-featured">
      <div class="card-top">
        <a href="{{ site.baseurl }}{{ p.url }}">{{ p.title }}</a>
        <span class="badge">Featured</span>
      </div>
      {%- if p.summary and p.summary != "" -%}
        <div class="muted">{{ p.summary }}</div>
      {%- endif -%}
    </li>
  {%- endfor -%}
</ul>

<hr class="section-divider" />
{% endif %}

## Latest (All essays)

<ul class="essay-list">
  {%- for p in normal_sorted -%}
    <li class="essay-card">
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
