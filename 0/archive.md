---
layout: page
title: Archive
permalink: /0/archive/

title_i18n:
  en: Archive
  ja: アーカイブ
---

{% comment %}

{% if site.lang == "ja" %}
このページでは、各コンテンツのすべてのアーカイブ版を、コンテンツごとに一覧表示しています。
{% comment %}このページでは、すべてのコンテンツのアーカイブ版を、エントリごとに一覧表示しています。{% endcomment %}
{% else %}
This page lists all archived versions of each entry, grouped by entry.
{% comment %}This page lists archived versions of all content, grouped by entry.{% endcomment %}
{% comment %}This page lists archived versions of all contents, grouped by content.{% endcomment %}
{% endif %}

{% endcomment %}

{% if site.lang == "ja" %}
アーカイブ版は、過去の状態を保存したものです。
{% comment %}アーカイブ版は、過去の状態を固定されたスナップショットとして保存したものです。{% endcomment %}
{% else %}
Archived versions preserve earlier states.
{% comment %}Archived versions preserve fixed snapshots of earlier states.{% endcomment %}
{% comment %}Archived versions are fixed snapshots of earlier versions.{% endcomment %}
{% comment %}Archived versions represent fixed snapshots of earlier intellectual stages.{% endcomment %}
{% endif %}

---

{% assign archive_pages = site.pages
  | where: "layout", "content"
  | where: "is_archive", true
%}

{% assign current_pages = site.pages
  | where: "layout", "content"
  | where_exp: "p", "p.is_archive != true"
%}

{% comment %}
Build sortable rows: "content_id::invVersionPad|||url"
- content_id fallback: page.url (if missing)
- invVersionPad: (9999 - version) padded => sorts newest version first within same content_id
{% endcomment %}

{% assign enriched = "" | split: "" %}

{% for p in archive_pages %}
  {% assign cid = p.content_id %}
  {% if cid == nil or cid == "" %}
    {% assign cid = p.url %}
  {% endif %}

  {% assign v = p.version | default: 0 | plus: 0 %}
  {% assign inv = 9999 | minus: v %}
  {% assign invpad = inv | prepend: "0000" | slice: -4, 4 %}

  {% capture key %}{{ cid }}::{{ invpad }}{% endcapture %}
  {% capture row %}{{ key }}|||{{ p.url }}{% endcapture %}
  {% assign enriched = enriched | push: row %}
{% endfor %}

{% assign enriched = enriched | sort %}

{% if enriched and enriched.size > 0 %}

{% assign current_cid = "" %}
{% assign open_ul = false %}
{% assign group_current_url = "" %}
{% assign group_current_title = "" %}

{% for row in enriched %}
  {% assign parts = row | split: "|||" %}
  {% assign key = parts[0] %}
  {% assign url = parts[1] %}
  {% assign key_parts = key | split: "::" %}
  {% assign cid = key_parts[0] %}

  {% comment %}
  Find the archive page object by url
  {% endcomment %}
  {% assign p = nil %}
  {% for q in archive_pages %}
    {% if q.url == url %}
      {% assign p = q %}
      {% break %}
    {% endif %}
  {% endfor %}
  {% if p == nil %}
    {% continue %}
  {% endif %}

  {% if cid != current_cid %}

    {% comment %}
    Close previous group + add link to current (if we had one)
    {% endcomment %}
    {% if open_ul %}
</ul>

{% if group_current_url != "" %}
<p class="muted">
  {% if site.lang == "ja" %}
  <a href="{{ group_current_url | relative_url }}">最新版を見る</a>
  {% comment %}最新版: <a href="{{ group_current_url | relative_url }}">{{ group_current_title }}</a>{% endcomment %}
  {% else %}
  <a href="{{ group_current_url | relative_url }}">View latest version</a>
  {% comment %}Latest version: <a href="{{ group_current_url | relative_url }}">{{ group_current_title }}</a>{% endcomment %}
  {% comment %}Current: <a href="{{ group_current_url | relative_url }}">{{ group_current_title }}</a>{% endcomment %}
  {% endif %}
</p>
{% endif %}

---
    {% endif %}

    {% assign current_cid = cid %}
    {% assign open_ul = true %}

    {% comment %}
    Find current (non-archive) page for this content_id
    {% endcomment %}
    {% assign cp = nil %}
    {% for x in current_pages %}
      {% if x.content_id == cid %}
        {% assign cp = x %}
        {% break %}
      {% endif %}
    {% endfor %}

    {% if cp %}
      {% assign group_current_url = cp.url %}
      {% assign group_current_title = cp.title %}
    {% else %}
      {% assign group_current_url = "" %}
      {% assign group_current_title = "" %}
    {% endif %}

### {% if cp %}{{ cp.title }}{% else %}{{ p.title }}{% endif %}

<ul class="content-list archive-list">
  {% endif %}

  <li class="archive-item">
    <div class="archive-title">
      <a href="{{ p.url | relative_url }}">Archived version {{ p.version }}</a>{% comment %}Version{% endcomment %}
      {% if p.last_updated and p.last_updated != "" %}
        <span class="muted"> · {{ p.last_updated }}</span>
      {% elsif p.first_published and p.first_published != "" %}
        <span class="muted"> · {{ p.first_published }}</span>
      {% endif %}
    </div>

    {% if p.summary and p.summary != "" %}
      <div class="archive-summary muted">{{ p.summary }}</div>
    {% endif %}
  </li>

{% endfor %}

{% if open_ul %}
</ul>

{% if group_current_url != "" %}
<p class="muted">
  {% if site.lang == "ja" %}
  最新版: <a href="{{ group_current_url | relative_url }}">{{ group_current_title }}</a>
  {% else %}
  Latest version: <a href="{{ group_current_url | relative_url }}">{{ group_current_title }}</a>
  {% comment %}Current: <a href="{{ group_current_url | relative_url }}">{{ group_current_title }}</a>{% endcomment %}
  {% endif %}
</p>
{% endif %}
{% endif %}

{% else %}

No archived versions yet.

{% endif %}
