---
layout: page
title: Search
permalink: /0/search/
---

<div class="search-box">
  <input id="q" class="search-input" type="search" placeholder="Search title / summary / tags… (tip: tag:ai-safety, content_id:ai-evaluation-structure, or &quot;exact phrase&quot;)" autocomplete="off" />
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

  // Turn text into tokens:
  // - phrase tokens: "evaluation structure"
  // - word tokens: evaluation, structure
  function tokenizeText(text) {
    const raw = (text ?? "").toString();
    const tokens = [];

    const phraseRe = /"([^"]+)"/g;
    let remaining = raw;
    let m;

    while ((m = phraseRe.exec(raw)) !== null) {
      const phrase = normalize(m[1]).trim();
      if (phrase) tokens.push({ type: "phrase", value: phrase });
      remaining = remaining.replace(m[0], " ");
    }

    normalize(remaining)
      .split(/\s+/)
      .filter(Boolean)
      .forEach(w => tokens.push({ type: "word", value: w }));

    return tokens;
  }

  // Extract filters like:
  // - tag:foo / tag:"日本語"
  // - content_id:ai-evaluation-structure / content_id:"..."
  function extractFilters(raw, key) {
    // Matches: (start or whitespace) key:( "..." | '...' | token )
    // Returns { values: [original strings], remainingText: string }
    const values = [];
    const re = new RegExp(String.raw`(?:^|\s)${key}:(?:"([^"]+)"|'([^']+)'|([^\s]+))`, "gi");

    let remaining = raw;
    let m;
    while ((m = re.exec(raw)) !== null) {
      const v = (m[1] || m[2] || m[3] || "").trim();
      if (v) values.push(v);
      // remove the matched chunk from remaining
      remaining = remaining.replace(m[0], " ");
    }

    return { values, remainingText: remaining };
  }

  function parseQuery(raw) {
    const q = raw.trim();
    if (!q) {
      return {
        textTokens: [],
        tagFilters: [],
        tagFiltersNorm: [],
        contentIdFilters: [],
        contentIdFiltersNorm: []
      };
    }

    // 1) extract tag:
    const t1 = extractFilters(q, "tag");
    const tagFilters = t1.values;
    const tagFiltersNorm = tagFilters.map(t => normalize(t));

    // 2) extract content_id: from the remaining
    const t2 = extractFilters(t1.remainingText, "content_id");
    const contentIdFilters = t2.values;
    const contentIdFiltersNorm = contentIdFilters.map(v => normalize(v));

    // 3) tokenize remaining as text tokens (words + phrases)
    const textTokens = tokenizeText(t2.remainingText);

    return {
      textTokens,
      tagFilters,
      tagFiltersNorm,
      contentIdFilters,
      contentIdFiltersNorm
    };
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

  function matchesContentIdFilters(item, contentIdFiltersNorm) {
    if (!contentIdFiltersNorm || contentIdFiltersNorm.length === 0) return true;

    const cid = normalize(item.content_id);

    // AND semantics (usually 1 id)
    for (const f of contentIdFiltersNorm) {
      if (cid !== f) return false;
    }
    return true;
  }

  function scoreItem_AND(item, textTokens) {
    // AND semantics: every token (word/phrase) must match at least one field (title/tags/summary).
    // If no text tokens, return 0 (filter-only handled elsewhere).
    if (!textTokens || textTokens.length === 0) return 0;

    const title = normalize(item.title);
    const summary = normalize(item.summary);
    const tags = normalize((item.tags || []).join(" "));

    let score = 0;

    for (const tok of textTokens) {
      const t = tok.value;
      if (!t) continue;

      let matched = false;

      if (title.includes(t)) { score += 5; matched = true; }
      if (tags.includes(t))  { score += 3; matched = true; }
      if (summary.includes(t)) { score += 1; matched = true; }

      if (!matched) return 0;
    }

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
      $status.textContent = 'Type to search. (Filters: tag:..., content_id:..., and phrases "...")';
      return;
    }

    const notes = [];
    if (parsed.tagFilters && parsed.tagFilters.length) notes.push(`Tags: ${parsed.tagFilters.join(", ")}`);
    if (parsed.contentIdFilters && parsed.contentIdFilters.length) notes.push(`content_id: ${parsed.contentIdFilters.join(", ")}`);

    const noteText = notes.length ? `  ${notes.join(" | ")}` : "";

    if (items.length === 0) {
      $status.textContent = "No results." + noteText;
      return;
    }

    $status.textContent = `${items.length} result(s).` + noteText;

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

    // 1) content_id filter (most selective)
    const cidFiltered = flat.filter(it => matchesContentIdFilters(it, parsed.contentIdFiltersNorm));

    // 2) tag filter
    const tagFiltered = cidFiltered.filter(it => matchesTagFilters(it, parsed.tagFiltersNorm));

    // 3) text search (AND) or filter-only listing
    let results = [];

    if (parsed.textTokens.length > 0) {
      const scored = [];
      for (const it of tagFiltered) {
        const s = scoreItem_AND(it, parsed.textTokens);
        if (s > 0) scored.push({ it, s });
      }
      scored.sort((a, b) => b.s - a.s);
      results = scored.slice(0, 50).map(x => x.it);
    } else {
      // filter-only
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
