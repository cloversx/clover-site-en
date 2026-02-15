---
layout: page
title: Archive
permalink: /0/archive/
---

This page lists archived versions of all contents.

Archived versions represent fixed snapshots of earlier intellectual stages.

---

{%- assign archive_pages = site.pages
  | where: "layout", "content"
  | where: "is_archive", true
  | sort: "last_updated"
  | reverse
-%}

{%- if archive_pages and archive_pages.size > 0 -%}

<ul class="content-list archive-list">
{%- for p in archive_pages -%}

  <li class="archive-item">

    <div class="archive-title">
      <a href="{{ p.url | relative_url }}">
        {{ p.title }}
      </a>
    </div>

    <div class="archive-meta muted">
      Version {{ p.version }}
      {%- if p.last_updated and p.last_updated != "" -%}
        · {{ p.last_updated }}
      {%- elsif p.first_published and p.first_published != "" -%}
        · {{ p.first_published }}
      {%- endif -%}
    </div>

    {%- if p.summary and p.summary != "" -%}
      <div class="archive-summary muted">
        {{ p.summary }}
      </div>
    {%- endif -%}

  </li>

{%- endfor -%}
</ul>

{%- else -%}

No archived versions yet.

{%- endif -%}
