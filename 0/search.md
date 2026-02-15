---
layout: page
title: Search
permalink: /0/search/
---

<div class="search-box">
  <input id="q" class="search-input" type="search" placeholder="Search title / summary / tags…" autocomplete="off" />
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

  let data = null;      // nested
  let flat = [];        // flattened list for searching

  function normalize(s) {
    return (s ?? "")
      .toString()
      .toLowerCase()
      .normalize("NFKC");
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

  function scoreItem(item, qTokens) {
    const title = normalize(item.title);
    const summary = normalize(item.summary);
    const tags = normalize((item.tags || []).join(" "));

    // Simple weighted scoring:
    // title match > tags > summary
    let score = 0;
    for (const t of qTokens) {
      if (!t) continue;
      if (title.includes(t)) score += 5;
      if (tags.includes(t)) score += 3;
      if (summary.includes(t)) score += 1;
    }

    // Prefer current over archive when scores tie
    if (item.kind === "current") score += 0.2;

    // Prefer higher versions slightly (useful when archives included)
    score += Math.min(0.19, (item.version || 0) * 0.01);

    return score;
  }

  function escapeHtml(s) {
    return (s ?? "").toString()
      .replaceAll("&", "&amp;")
      .replaceAll("<", "&lt;")
      .replaceAll(">", "&gt;")
      .replaceAll('"', "&quot;")
      .replaceAll("'", "&#39;");
  }

  function render(items, q) {
    $results.innerHTML = "";
    if (!q) {
      $status.textContent = "Type to search.";
      return;
    }
    if (items.length === 0) {
      $status.textContent = "No results.";
      return;
    }
    $status.textContent = `${items.length} result(s).`;

    for (const it of items) {
      const badge = it.kind === "archive"
        ? `<span class="badge">v${escapeHtml(it.version)}</span>`
        : "";

      const summary = it.summary ? `<div class="muted">${escapeHtml(it.summary)}</div>` : "";
      const tags = (it.tags && it.tags.length)
        ? `<div class="muted"><small>${escapeHtml(it.tags.join(" · "))}</small></div>`
        : "";

      const li = document.createElement("li");
      li.className = "content-card";
      li.innerHTML = `
        <div class="card-top">
          <a href="${escapeHtml(it.url)}">${escapeHtml(it.title || "(untitled)")}</a>
          ${badge}
        </div>
        ${summary}
        ${tags}
      `;
      $results.appendChild(li);
    }
  }

  function getQueryFromURL() {
    const u = new URL(window.location.href);
    const q = u.searchParams.get("q") || "";
    const a = u.searchParams.get("a") || ""; // "1" => include archives
    return { q, includeArchives: a === "1" };
  }

  function setQueryToURL(q, includeArchives) {
    const u = new URL(window.location.href);
    if (q) u.searchParams.set("q", q);
    else u.searchParams.delete("q");
    if (includeArchives) u.searchParams.set("a", "1");
    else u.searchParams.delete("a");
    history.replaceState(null, "", u.toString());
  }

  function run() {
    const q = $q.value.trim();
    const includeArchives = $inc.checked;

    setQueryToURL(q, includeArchives);
    flat = buildFlat(includeArchives);

    if (!q) {
      render([], "");
      return;
    }

    const qTokens = normalize(q).split(/\s+/).filter(Boolean);
    const scored = [];
    for (const it of flat) {
      const s = scoreItem(it, qTokens);
      if (s > 0) scored.push({ it, s });
    }

    scored.sort((a, b) => b.s - a.s);
    const top = scored.slice(0, 50).map(x => x.it);
    render(top, q);
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
