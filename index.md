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

{%- assign all_items = site.pages
  | where: "layout", "essay"
  | where_exp: "p", "p.is_archive != true"
-%}

{%- assign featured_items = all_items | where: "featured", true -%}
{%- assign ranked = featured_items
  | where_exp: "p", "p.featured_rank != nil"
  | sort: "featured_rank"
-%}
{%- assign unranked = featured_items
  | where_exp: "p", "p.featured_rank == nil"
  | sort: "last_updated"
  | reverse
-%}

## Featured {% comment %}（代表作）{% endcomment %}
{% comment %}#### Featured {% comment %}（代表作）{% endcomment %}{% endcomment %}

{% comment %}- (Add links to representative essays here){% endcomment %}
{% comment %}- [AI Isn’t Dangerous. Putting AI Inside an “Evaluation Structure” Is.]({{ site.baseurl }}/essays/ai-evaluation-structure/){% endcomment %}

<ul class="essay-list">
  {%- for p in ranked -%}
    <li>
      <a href="{{ site.baseurl }}{{ p.url }}">{{ p.title }}</a>
      {%- if p.summary %}
        <div class="muted">{{ p.summary }}</div>
      {%- endif %}
    </li>
  {%- endfor -%}

  {%- for p in unranked -%}
    <li>
      <a href="{{ site.baseurl }}{{ p.url }}">{{ p.title }}</a>
      {%- if p.summary %}
        <div class="muted">{{ p.summary }}</div>
      {%- endif %}
    </li>
  {%- endfor -%}
</ul>

## Latest {% comment %}（最新）{% endcomment %}
{% comment %}#### Latest {% comment %}（最新）{% endcomment %}{% endcomment %}

{% comment %}- (Add links to latest essays here){% endcomment %}
{% comment %}- [AI Isn’t Dangerous. Putting AI Inside an “Evaluation Structure” Is.]({{ site.baseurl }}/essays/ai-evaluation-structure/){% endcomment %}

{%- assign latest = all_items
  | sort: "last_updated"
  | reverse
-%}

<ul class="essay-list">
  {%- for p in latest limit:3 -%}
    <li>
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
