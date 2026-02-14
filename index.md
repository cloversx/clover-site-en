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

{%- assign featured_items = site.pages
  | where: "layout", "essay"
  | where_exp: "p", "p.is_archive != true"
  | where: "featured", true
-%}

{%- if featured_items and featured_items.size > 0 -%}
## Featured {% comment %}（代表作）{% endcomment %}
{% comment %}#### Featured {% comment %}（代表作）{% endcomment %}{% endcomment %}

{% comment %}- (Add links to representative essays here){% endcomment %}
{% comment %}- [AI Isn’t Dangerous. Putting AI Inside an “Evaluation Structure” Is.]({{ site.baseurl }}/essays/ai-evaluation-structure/){% endcomment %}

{%- include essay-list-featured.html show_badge=false featured_style=false show_date=false -%}

{% comment %}<hr class="section-divider" />{% endcomment %}
{%- endif -%}

## Latest {% comment %}（最新）{% endcomment %}
{% comment %}#### Latest {% comment %}（最新）{% endcomment %}{% endcomment %}

{% comment %}- (Add links to latest essays here){% endcomment %}
{% comment %}- [AI Isn’t Dangerous. Putting AI Inside an “Evaluation Structure” Is.]({{ site.baseurl }}/essays/ai-evaluation-structure/){% endcomment %}

{%- include essay-list-latest.html limit=3 show_date=true -%}
