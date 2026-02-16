---
layout: page
title: ""
---

{%- assign lang = page.lang | default: site.lang | default: "en" -%}

# Clover Library
{% comment %}### - Clover Library -{% endcomment %}
<br>

{% comment %}Welcome.{% endcomment %}

{% comment %}A quiet canonical archive for long-term preservation.{% endcomment %}

<div class="home-links">
  {% if lang == "ja" %}
    <a class="btn" href="{{ site.baseurl }}/0/">ライブラリ</a>{% comment %}図書{% endcomment %}
    {% comment %}<a class="btn ghost" href="{{ site.baseurl }}/about/">このサイトについて</a>{% endcomment %}{% comment %}図書{% endcomment %}
  {% else %}
    <a class="btn" href="{{ site.baseurl }}/0/">Library</a>{% comment %}Browse Library{% endcomment %}
    {% comment %}<a class="btn ghost" href="{{ site.baseurl }}/about/">About</a>{% endcomment %}
  {% endif %}
</div>


{% comment %}
- Read: **[Library]({{ site.baseurl }}/0/)**
- About: **[About]({{ site.baseurl }}/about/)**
{% endcomment %}

{% comment %}---{% endcomment %}

{%- assign featured_items = site.pages
  | where: "layout", "content"
  | where_exp: "p", "p.is_archive != true"
  | where: "featured", true
-%}

{%- if featured_items and featured_items.size > 0 -%}
{% if lang == "ja" %}
## 固定
{% else %}
## Pinned
{% comment %}## Featured {% endcomment %}{% comment %}（代表作）{% endcomment %}
{% endif %}

{% comment %}#### Featured {% comment %}（代表作）{% endcomment %}{% endcomment %}

{% comment %}- (Add links to representative contents here){% endcomment %}
{% comment %}- [AI Isn’t Dangerous. Putting AI Inside an “Evaluation Structure” Is.]({{ site.baseurl }}/0/ai-evaluation-structure/){% endcomment %}

{% comment %}
include は トリム無し
Liquid の “空白トリムが効きすぎて、見出し ## ... と <ul> が同じ行に連結されてしまう
{% endcomment %}
{% comment %}{% include content-list-featured.html show_badge=false featured_style=false show_date=false %}{% endcomment %}
{% include content_section.html mode="featured" show_badge=false featured_style=false show_date=false %}


{% comment %}<hr class="section-divider" />{% endcomment %}
{%- endif -%}

{% comment %}（
{% if lang == "ja" %}
## 最新 {% comment %}<span style="color:red">３件</span>{% endcomment %}
{% else %}
## Latest {% comment %}（最新）{% endcomment %}
{% endif %}
{% comment %}#### Latest {% comment %}（最新）{% endcomment %}{% endcomment %}

{% comment %}- (Add links to latest contents here){% endcomment %}
{% comment %}- [AI Isn’t Dangerous. Putting AI Inside an “Evaluation Structure” Is.]({{ site.baseurl }}/0/ai-evaluation-structure/){% endcomment %}

{% comment %}
include は トリム無し
Liquid の “空白トリムが効きすぎて、見出し ## ... と <ul> が同じ行に連結されてしまう
{% endcomment %}
{% comment %}{% include content-list-latest.html limit=3 show_date=true %}{% endcomment %}
{% include content_section.html mode="latest" limit=3 show_date=true %}
{% endcomment %}

