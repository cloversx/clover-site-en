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

{%- assign featured_ranked = featured_items
  | where_exp: "p", "p.featured_rank != nil"
  | sort: "featured_rank"
-%}

{%- assign featured_unranked = featured_items
  | where_exp: "p", "p.featured_rank == nil"
  | sort: "last_updated"
  | reverse
-%}

{%- assign all_sorted = items
  | sort: "last_updated"
  | reverse
-%}

{% if featured_items.size > 0 %}
## Featured (Pinned)

<ul class="essay-list">
  {%- for p in featured_ranked -%}
    <li class="essay-card is-featured">
      <div class="card-top">
        <a href="{{ site.baseurl }}{{ p.url }}">{{ p.title }}</a>
        <span class="badge">Featured</span>
      </div>
      {%- if p.summary %}
        <div class="muted">{{ p.summary }}</div>
      {%- endif %}
    </li>
  {%- endfor -%}

  {%- for p in featured_unranked -%}
    <li class="essay-card is-featured">
      <div class="card-top">
        <a href="{{ site.baseurl }}{{ p.url }}">{{ p.title }}</a>
        <span class="badge">Featured</span>
      </div>
      {%- if p.summary %}
        <div class="muted">{{ p.summary }}</div>
      {%- endif %}
    </li>
  {%- endfor -%}
</ul>

<hr class="section-divider" />
{% endif %}

## Latest (All essays)

<ul class="essay-list">
  {%- for p in all_sorted -%}
    <li class="essay-card">
      <a href="{{ site.baseurl }}{{ p.url }}">{{ p.title }}</a>
      {%- if p.summary %}
        <div class="muted">{{ p.summary }}</div>
      {%- endif %}
      {%- if p.last_updated %}
        <div class="muted"><small>Updated: {{ p.last_updated }}</small></div>
      {%- endif %}
    </li>
  {%- endfor -%}
</ul>
