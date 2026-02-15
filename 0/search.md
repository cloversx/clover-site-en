---
layout: page
title: Search
permalink: /0/search/
---

<div class="search-box">
  <input id="q" class="search-input" type="search" placeholder="Search title / summary / tags… (tip: tag:ai-safety or tag:&quot;AI安全性&quot;)" autocomplete="off" />
  <label class="search-toggle">
    <input id="includeArchives" type="checkbox" />
    Include archives
  </label>
</div>

<div id="searchStatus" class="muted"></div>
<ul id="results" class="content-list"></ul>

<script>
(() => {
  const INDEX_URL = "{{ '/0/contents-nested.json' | relative_url }}";

  const $q = document.getElementById('q');
  const $inc = document.getElementById('includeArchives');
  const $status = document.getElementById('searchStatus');
  const $results = document.getElementById('results');

  let data = null;

  function normalize(s) {
    return (s ?? "")
      .toString()
      .toLowerCase()
      .normalize("NFKC");
  }

  function escapeHtml(s) {
    return (s ?? "").toString()
      .replaceAll("&", "&amp;")
      .replaceAll("<", "&lt;")
      .replaceAll(">", "&gt;")
      .replaceAll('"', "&quot;")
      .replaceAll("'", "&#39;");
  }

  function parseQuery(raw) {
    // Supports:
    // - normal tokens: foo bar
    // - tag filters: tag:ai-safety, tag:"AI安全性"
    const q = raw.trim();
    if (!q) return { textTokens: [], tagFilters: [], tagFiltersNorm: [] };

    const tagFilters = [];
    const textParts = [];

    const re = /(?:^|\s)tag:(?:"([^"]+)"|'([^']+)'|([^\s]+))/gi;
    let lastIndex = 0;
    let m;

    while ((m = re.exec(q)) !== null) {
      const start = m.index;
      const end = re.lastIndex;

      const between = q.slice(lastIndex, start);
      if (between.trim()) textParts.push(between.trim());

      const tag = (m[1] || m[2] || m[3] || "").trim();
      if (tag) tagFilters.push(tag);

      lastIndex = end;
    }

    const tail = q.slice(lastIndex);
    if (tail.trim()) textParts.push(tail.trim());

    const textTokens = normalize(textParts.join(" ")).split(/\s+/).filter(Boolean);
    const tagFiltersNorm = tagFilters.map(t => normalize(t));

    return { textTokens, tagFilters, tagFiltersNorm };
  }

  function buildFlat(includeArchives) {
    const out = [];
    for (const entry of (data || [])) {
      if (entry.current) {
        out.push({
          kind: "current",
          content_id: entry.content_id,
          version: entry.current.version ?? 0,
          title: entry.current.title ?? "",
          summary: entry.current.summary ?? "",
          tags: entry.current.tags ?? [],
          url: entry.current.url ?? ""
        });
      }
      if (includeArchives && Array.isArray(entry.archives)) {
        for (const a of entry.archives) {
          out.push({
            kind: "archive",
            content_id: entry.content_id,
            version: a.version ?? 0,
            title: a.title ?? "",
            summary: a.summary ?? "",
            tags: a.tags ?? [],
            url: a.url ?? ""
          });
        }
      }
    }
    return out;
  }

  function matchesTagFilters(item, tagFiltersNorm) {
    if (!tagFiltersNorm || tagFiltersNorm.length === 0) return true;

    const itemTags = (item.tags || []).map(t => normalize(t));

    // AND semantics: all requested tags must be present (exact match)
    for (const tf of tagFiltersNorm) {
      if (!itemTags.some(t => t === tf)) return false;
    }
    return true;
  }

  function scoreItem_AND(item, textTokens) {
    // AND semantics: every token must match at least one field (title/tags/summary).
    // If no text tokens, return 0 (tag-only handled elsewhere).
    if (!textTokens || textTokens.length === 0) return 0;

    const title = normalize(item.title);
    const summary = normalize(item.summary);
    const tags = normalize((item.tags || []).join(" "));

    let score = 0;

    for (const t of textTokens) {
      let matched = false;

      if (title.includes(t)) { score += 5; matched = true; }
      if (tags.includes(t))  { score += 3; matched = true; }
      if (summary.includes(t)){ score += 1; matched = true; }

      // If any token doesn't match anywhere, exclude.
      if (!matched) return 0;
    }

    // Tie-breakers
    if (item.kind === "current") score += 0.2;
    score += Math.min(0.19, (item.version || 0) * 0.01);

    return score;
  }

  function setQueryToURL(q, includeArchives) {
    const u = new URL(window.location.href);
    if (q) u.searchParams.set("q", q);
    else u.searchParams.delete("q");
    if (includeArchives) u.searchParams.set("a", "1");
    else u.searchParams.delete("a");
    history.replaceState(null, "", u.toString());
  }

  function getQueryFromURL() {
    const u = new URL(window.location.href);
    const q = u.searchParams.get("q") || "";
    const a = u.searchParams.get("a") || "";
    return { q, includeArchives: a === "1" };
  }

  function render(items, rawQuery, parsed) {
    $results.innerHTML = "";

    if (!rawQuery.trim()) {
      $status.textContent = "Type to search. (You can filter with tag:...)";
      return;
    }

    const tagNote = (parsed.tagFilters && parsed.tagFilters.length)
      ? `  Tags: ${parsed.tagFilters.join(", ")}`
      : "";

    if (items.length === 0) {
      $status.textContent = "No results." + tagNote;
      return;
    }

    $status.textContent = `${items.length} result(s).` + tagNote;

    for (const it of items) {
      const kindBadge = it.kind === "archive"
        ? `<span class="badge is-archive">Archive v${escapeHtml(it.version)}</span>`
        : `<span class="badge is-current">Current</span>`;

      const summary = it.summary ? `<div class="muted">${escapeHtml(it.summary)}</div>` : "";

      const tags = (it.tags && it.tags.length)
        ? `<div class="tag-row">${it.tags.map(t => `<button type="button" class="tag-pill" data-tag="${escapeHtml(t)}">${escapeHtml(t)}</button>`).join("")}</div>`
        : "";

      const li = document.createElement("li");
      li.className = "content-card";
      li.innerHTML = `
        <div class="card-top">
          <a href="${escapeHtml(it.url)}">${escapeHtml(it.title || "(untitled)")}</a>
          ${kindBadge}
        </div>
        ${summary}
        ${tags}
      `;
      $results.appendChild(li);
    }

    // Tag click -> add tag filter
    $results.querySelectorAll(".tag-pill").forEach(btn => {
      btn.addEventListener("click", () => {
        const tag = btn.getAttribute("data-tag") || "";
        if (!tag) return;

        const raw = $q.value.trim();
        const tagToken = tag.includes(" ") ? `tag:"${tag}"` : `tag:${tag}`;
        if (raw.includes(tagToken)) return;

        const next = raw ? `${raw} ${tagToken}` : tagToken;
        $q.value = next;
        run();
        $q.focus();
      });
    });
  }

  function run() {
    const raw = $q.value;
    const includeArchives = $inc.checked;

    setQueryToURL(raw.trim(), includeArchives);

    const parsed = parseQuery(raw);
    const flat = buildFlat(includeArchives);

    // 1) Tag filter first
    const tagFiltered = flat.filter(it => matchesTagFilters(it, parsed.tagFiltersNorm));

    // 2) If there are text tokens, apply AND matching and scoring.
    //    If there are NO text tokens (tag-only), keep all tag-matched items and rank gently.
    let results = [];

    if (parsed.textTokens.length > 0) {
      const scored = [];
      for (const it of tagFiltered) {
        const s = scoreItem_AND(it, parsed.textTokens);
        if (s > 0) scored.push({ it, s }); // STRICT: only matched items
      }
      scored.sort((a, b) => b.s - a.s);
      results = scored.slice(0, 50).map(x => x.it);
    } else {
      // tag-only (or empty after removing tag tokens)
      // Prefer current first, then version desc for archives
      const current = tagFiltered.filter(x => x.kind === "current");
      const archives = tagFiltered.filter(x => x.kind === "archive")
        .sort((a, b) => (b.version || 0) - (a.version || 0));
      results = current.concat(archives).slice(0, 50);
    }

    render(results, raw, parsed);
  }

  async function init() {
    $status.textContent = "Loading index…";
    try {
      const res = await fetch(INDEX_URL, { cache: "no-cache" });
      if (!res.ok) throw new Error(`HTTP ${res.status}`);
      data = await res.json();

      const initState = getQueryFromURL();
      $q.value = initState.q;
      $inc.checked = initState.includeArchives;

      $status.textContent = "Ready.";
      run();
    } catch (e) {
      console.error(e);
      $status.textContent = "Failed to load search index.";
    }
  }

  $q.addEventListener("input", () => run());
  $inc.addEventListener("change", () => run());

  init();
})();
</script>
