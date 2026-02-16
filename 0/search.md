---
layout: page
title: Search
permalink: /0/search/

title_i18n:
  en: Search
  ja: 検索

pinned_tags:
  - environment-alignment
  - ai-safety
  - AI安全性
  - 評価構造
  - GitHub Pages
---

<div class="search-box">
  <input id="q" class="search-input" type="search"
    placeholder="Search title / summary / tags… (tip: tag:ai-safety, content_id:ai-evaluation-structure, or &quot;exact phrase&quot;)"
    autocomplete="off" />

  <label class="search-toggle">
    <input id="onlyCurrent" type="checkbox" />
    <span data-i18n="onlyCurrent">Only current</span>
  </label>

  <label class="search-toggle">
    <input id="includeArchives" type="checkbox" />
    <span data-i18n="includeArchives">Include archives</span>
  </label>

  <label class="search-toggle">
    <span data-i18n="sortLabel">Sort:</span>
    <select id="sortBy" class="search-select">
      <option value="relevance" data-i18n-opt="sort_relevance">Relevance</option>
      <option value="updated_desc" data-i18n-opt="sort_updated_desc">Updated (newest)</option>
      <option value="updated_asc" data-i18n-opt="sort_updated_asc">Updated (oldest)</option>
      <option value="published_desc" data-i18n-opt="sort_published_desc">Published (newest)</option>
      <option value="published_asc" data-i18n-opt="sort_published_asc">Published (oldest)</option>
      <option value="title_asc" data-i18n-opt="sort_title_asc">Title (A→Z)</option>
      <option value="title_desc" data-i18n-opt="sort_title_desc">Title (Z→A)</option>
    </select>
  </label>
</div>

<!-- Tag panel -->
<div id="tagPanel" class="tag-panel">
  <div class="muted"><strong data-i18n="tagsTitle">Tags</strong></div>
  <div id="tagList" class="tag-list"></div>
</div>

<div id="searchStatus" class="muted"></div>
<ul id="results" class="content-list"></ul>

<script>
(() => {
  const INDEX_URL = "{{ '/0/contents-nested.json' | relative_url }}";

  // ---- i18n ----
  const LANG = "{{ site.lang | default: 'en' }}";

  const I18N = {
    en: {
      placeholder: 'Search titles, summaries, or tags… (tip: tag:ai-safety, content_id:ai-evaluation-structure, or "exact phrase")',{% comment %}'Search title / summary / tags… (tip: tag:ai-safety, content_id:ai-evaluation-structure, or "exact phrase")',{% endcomment %}
      onlyCurrent: "Current only",{% comment %}"Only current",{% endcomment %}
      includeArchives: "Include archived versions",{% comment %}"Include archives",{% endcomment %}
      sortLabel: "Sort:",
      tagsTitle: "Tags",
      noTags: "No tags.",
      statusType: 'Start typing to search. Use filters like tag:..., content_id:..., or phrases in quotes.)',{% comment %}'Type to search. (Filters: tag:..., content_id:..., and phrases "...")',{% endcomment %}
      statusLoading: "Loading index…",
      statusReady: "Ready.",
      statusFailed: "Failed to load search index.",
      statusNoResults: "No results.",
      statusResults: (n) => `${n} result(s).`,
      noteTags: (s) => `Tags: ${s}`,
      noteContentId: (s) => `content_id: ${s}`,
      noteOnlyCurrent: "Only current",
      noteIncludingArchives: "Including archives",
      noteSort: (s) => `Sort: ${s}`,
      badgeCurrent: "Current",
      badgeArchive: (v) => `Archive v${v}`,
      published: "Published",
      updated: "Updated",

      sort_relevance: "Relevance",
      sort_updated_desc: "Updated (newest)",
      sort_updated_asc: "Updated (oldest)",
      sort_published_desc: "Published (newest)",
      sort_published_asc: "Published (oldest)",
      sort_title_asc: "Title (A→Z)",
      sort_title_desc: "Title (Z→A)",
    },
    ja: {
      placeholder: 'タイトル・要約・タグを検索...（例：tag:AI安全性, content_id:ai-evaluation-structure, または "完全一致フレーズ"）',{% comment %}'検索（タイトル / 要約 / タグ）…（例: tag:AI安全性, content_id:ai-evaluation-structure, または "完全一致フレーズ"）',{% endcomment %}
      onlyCurrent: "現在のコンテンツのみ",{% comment %}"現行のみ",{% endcomment %}
      includeArchives: "アーカイブ版を含める",{% comment %}"アーカイブを含める",{% endcomment %}
      sortLabel: "並び替え:",
      tagsTitle: "タグ",
      noTags: "タグがまだありません。",
      statusType: '入力すると検索できます。tag:... や content_id:...、または "..." で囲んだ語句による絞り込みが可能です。',{% comment %}
'検索してください。（フィルタ: tag:..., content_id:..., フレーズ "...")',{% endcomment %}
      statusLoading: "インデックス読み込み中…",
      statusReady: "準備完了。",
      statusFailed: "検索インデックスの読み込みに失敗しました。",
      statusNoResults: "該当なし。",
      statusResults: (n) => `${n} 件`,
      noteTags: (s) => `タグ: ${s}`,
      noteContentId: (s) => `content_id: ${s}`,
      noteOnlyCurrent: "現行のみ",
      noteIncludingArchives: "アーカイブを含む",
      noteSort: (s) => `並び替え: ${s}`,
      badgeCurrent: "現行",
      badgeArchive: (v) => `アーカイブ v${v}`,
      published: "初公開",
      updated: "最終更新",

      sort_relevance: "関連度",
      sort_updated_desc: "更新日（新しい順）",
      sort_updated_asc: "更新日（古い順）",
      sort_published_desc: "公開日（新しい順）",
      sort_published_asc: "公開日（古い順）",
      sort_title_asc: "タイトル（A→Z）",
      sort_title_desc: "タイトル（Z→A）",
    }
  };

  const T = I18N[LANG] || I18N.en;
  const t = (key) => (T && Object.prototype.hasOwnProperty.call(T, key)) ? T[key] : (I18N.en[key] ?? key);

  function applyI18nStatic() {
    // placeholder
    const $q = document.getElementById('q');
    if ($q) $q.setAttribute("placeholder", t("placeholder"));

    // label text
    document.querySelectorAll("[data-i18n]").forEach(el => {
      const key = el.getAttribute("data-i18n");
      const v = t(key);
      if (typeof v === "string") el.textContent = v;
    });

    // select options
    document.querySelectorAll("[data-i18n-opt]").forEach(el => {
      const key = el.getAttribute("data-i18n-opt");
      const v = t(key);
      if (typeof v === "string") el.textContent = v;
    });
  }

  // ---- DOM ----
  const $q = document.getElementById('q');
  const $only = document.getElementById('onlyCurrent');
  const $inc = document.getElementById('includeArchives');
  const $sort = document.getElementById('sortBy');
  const $status = document.getElementById('searchStatus');
  const $results = document.getElementById('results');
  const $tagList = document.getElementById('tagList');

  const PINNED_TAGS = [
    {% if page.pinned_tags %}
      {% for t in page.pinned_tags %}
        {{ t | jsonify }}{% unless forloop.last %},{% endunless %}
      {% endfor %}
    {% endif %}
  ];

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

  function extractFilters(raw, key) {
    const values = [];
    const re = new RegExp(String.raw`(?:^|\s)${key}:(?:"([^"]+)"|'([^']+)'|([^\s]+))`, "gi");

    let remaining = raw;
    let m;
    while ((m = re.exec(raw)) !== null) {
      const v = (m[1] || m[2] || m[3] || "").trim();
      if (v) values.push(v);
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

    const t1 = extractFilters(q, "tag");
    const tagFilters = t1.values;
    const tagFiltersNorm = tagFilters.map(t => normalize(t));

    const t2 = extractFilters(t1.remainingText, "content_id");
    const contentIdFilters = t2.values;
    const contentIdFiltersNorm = contentIdFilters.map(v => normalize(v));

    const textTokens = tokenizeText(t2.remainingText);

    return {
      textTokens,
      tagFilters,
      tagFiltersNorm,
      contentIdFilters,
      contentIdFiltersNorm
    };
  }

  function buildFlat(includeArchives, onlyCurrent) {
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
          url: entry.current.url ?? "",
          last_updated: entry.current.last_updated ?? "",
          first_published: entry.current.first_published ?? ""
        });
      }

      if (!onlyCurrent && includeArchives && Array.isArray(entry.archives)) {
        for (const a of entry.archives) {
          out.push({
            kind: "archive",
            content_id: entry.content_id,
            version: a.version ?? 0,
            title: a.title ?? "",
            summary: a.summary ?? "",
            tags: a.tags ?? [],
            url: a.url ?? "",
            last_updated: a.last_updated ?? "",
            first_published: a.first_published ?? ""
          });
        }
      }
    }
    return out;
  }

  function matchesTagFilters(item, tagFiltersNorm) {
    if (!tagFiltersNorm || tagFiltersNorm.length === 0) return true;
    const itemTags = (item.tags || []).map(t => normalize(t));
    for (const tf of tagFiltersNorm) {
      if (!itemTags.some(t => t === tf)) return false;
    }
    return true;
  }

  function matchesContentIdFilters(item, contentIdFiltersNorm) {
    if (!contentIdFiltersNorm || contentIdFiltersNorm.length === 0) return true;
    const cid = normalize(item.content_id);
    for (const f of contentIdFiltersNorm) {
      if (cid !== f) return false;
    }
    return true;
  }

  function scoreItem_AND(item, textTokens) {
    if (!textTokens || textTokens.length === 0) return 0;

    const title = normalize(item.title);
    const summary = normalize(item.summary);
    const tags = normalize((item.tags || []).join(" "));

    let score = 0;

    for (const tok of textTokens) {
      const v = tok.value;
      if (!v) continue;

      let matched = false;
      if (title.includes(v)) { score += 5; matched = true; }
      if (tags.includes(v))  { score += 3; matched = true; }
      if (summary.includes(v)) { score += 1; matched = true; }

      if (!matched) return 0;
    }

    if (item.kind === "current") score += 0.2;
    score += Math.min(0.19, (item.version || 0) * 0.01);

    return score;
  }

  function normalizeToggles() {
    if ($only.checked && $inc.checked) {
      $inc.checked = false;
    }
  }

  function setQueryToURL(q, includeArchives, onlyCurrent) {
    const u = new URL(window.location.href);

    if (q) u.searchParams.set("q", q);
    else u.searchParams.delete("q");

    if (onlyCurrent) {
      u.searchParams.set("c", "1");
      u.searchParams.delete("a");
    } else {
      u.searchParams.delete("c");
      if (includeArchives) u.searchParams.set("a", "1");
      else u.searchParams.delete("a");
    }

    const s = $sort?.value || "relevance";
    if (s && s !== "relevance") u.searchParams.set("s", s);
    else u.searchParams.delete("s");

    history.replaceState(null, "", u.toString());
  }

  function getQueryFromURL() {
    const u = new URL(window.location.href);
    const q = u.searchParams.get("q") || "";
    const a = u.searchParams.get("a") === "1";
    const c = u.searchParams.get("c") === "1";
    const s = u.searchParams.get("s") || "relevance";
    return { q, includeArchives: a, onlyCurrent: c, sortBy: s };
  }

  // ---- Sorting helpers ----

  function isoToNum(s) {
    const m = String(s || "").match(/^(\d{4})-(\d{2})-(\d{2})$/);
    if (!m) return 0;
    return (parseInt(m[1], 10) * 10000) + (parseInt(m[2], 10) * 100) + parseInt(m[3], 10);
  }

  function publishedDateNum(it) {
    return isoToNum(it.first_published);
  }

  function updatedDateNum(it) {
    return isoToNum(it.last_updated) || isoToNum(it.first_published);
  }

  function sortResults(items, mode, scoreMap) {
    const arr = items.slice();

    if (mode === "updated_desc") return arr.sort((a, b) => updatedDateNum(b) - updatedDateNum(a));
    if (mode === "updated_asc")  return arr.sort((a, b) => updatedDateNum(a) - updatedDateNum(b));
    if (mode === "published_desc") return arr.sort((a, b) => publishedDateNum(b) - publishedDateNum(a));
    if (mode === "published_asc")  return arr.sort((a, b) => publishedDateNum(a) - publishedDateNum(b));
    if (mode === "title_asc") return arr.sort((a, b) => normalize(a.title).localeCompare(normalize(b.title)));
    if (mode === "title_desc") return arr.sort((a, b) => normalize(b.title).localeCompare(normalize(a.title)));

    if (scoreMap) return arr.sort((a, b) => (scoreMap.get(b) || 0) - (scoreMap.get(a) || 0));
    return arr;
  }

  // ---- Tag panel helpers ----

  function collectTagCounts(includeArchives, onlyCurrent) {
    const counts = new Map();
    const flat = buildFlat(includeArchives, onlyCurrent);

    for (const it of flat) {
      for (const t of (it.tags || [])) {
        const k = normalize(t);
        const prev = counts.get(k);
        counts.set(k, { tag: prev?.tag ?? t, n: (prev?.n ?? 0) + 1 });
      }
    }
    return counts;
  }

  function toggleTagInQuery(raw, tag) {
    const token = tag.includes(" ") ? `tag:"${tag}"` : `tag:${tag}`;
    const q = raw.trim();

    const esc = token.replace(/[.*+?^${}()|[\]\\]/g, "\\$&");
    const re = new RegExp(`(^|\\s)${esc}(?=\\s|$)`, "g");

    if (re.test(q)) {
      return q.replace(re, " ").replace(/\s+/g, " ").trim();
    }
    return q ? `${q} ${token}` : token;
  }

  function getSelectedTagSetFromQuery(raw) {
    const q = (raw ?? "").toString();
    const re = /(?:^|\s)tag:(?:"([^"]+)"|'([^']+)'|([^\s]+))/gi;

    const set = new Set();
    let m;
    while ((m = re.exec(q)) !== null) {
      const tag = (m[1] || m[2] || m[3] || "").trim();
      if (tag) set.add(normalize(tag));
    }
    return set;
  }

  function applyTagHighlighting() {
    const selected = getSelectedTagSetFromQuery($q.value);

    document.querySelectorAll("#tagList .tag-pill").forEach(btn => {
      const label = btn.textContent || "";
      const tag = label.replace(/\s*\(\d+\)\s*$/, "").trim();
      btn.classList.toggle("is-active", selected.has(normalize(tag)));
    });

    document.querySelectorAll("#results .tag-pill").forEach(btn => {
      const tag = btn.getAttribute("data-tag") || btn.textContent || "";
      btn.classList.toggle("is-active", selected.has(normalize(tag)));
    });
  }

  function renderTagPanel() {
    if (!data || !$tagList) return;

    const counts = collectTagCounts($inc.checked, $only.checked);

    const ordered = [];
    const used = new Set();

    for (const t of (PINNED_TAGS || [])) {
      const k = normalize(t);
      if (counts.has(k)) {
        ordered.push(counts.get(k));
        used.add(k);
      }
    }

    const rest = [];
    for (const [k, v] of counts.entries()) {
      if (!used.has(k)) rest.push(v);
    }

    rest.sort((a, b) => (b.n - a.n) || (normalize(a.tag) > normalize(b.tag) ? 1 : -1));

    const finalList = ordered.concat(rest);

    $tagList.innerHTML = "";

    if (finalList.length === 0) {
      const d = document.createElement("div");
      d.className = "muted";
      d.textContent = t("noTags");
      $tagList.appendChild(d);
      return;
    }

    for (const it of finalList) {
      const btn = document.createElement("button");
      btn.type = "button";
      btn.className = "tag-pill";
      btn.textContent = `${it.tag} (${it.n})`;
      btn.addEventListener("click", () => {
        $q.value = toggleTagInQuery($q.value, it.tag);
        run();
        $q.focus();
      });
      $tagList.appendChild(btn);
    }
  }

  // ---- Rendering ----

  function humanSortLabel(mode) {
    // Use translated label for the note line (not raw enum)
    const key = `sort_${mode}`;
    const v = t(key);
    if (typeof v === "string" && v !== key) return v;
    return String(mode || "").replaceAll("_", " ");
  }

  function render(items, rawQuery, parsed) {
    $results.innerHTML = "";

    if (!rawQuery.trim()) {
      $status.textContent = t("statusType");
      return;
    }

    const notes = [];
    if (parsed.tagFilters && parsed.tagFilters.length) notes.push(t("noteTags")(parsed.tagFilters.join(", ")));
    if (parsed.contentIdFilters && parsed.contentIdFilters.length) notes.push(t("noteContentId")(parsed.contentIdFilters.join(", ")));

    if ($only.checked) notes.push(t("noteOnlyCurrent"));
    else if ($inc.checked) notes.push(t("noteIncludingArchives"));

    const sortMode = $sort?.value || "relevance";
    if (sortMode && sortMode !== "relevance") notes.push(t("noteSort")(humanSortLabel(sortMode)));

    const noteText = notes.length ? `  ${notes.join(" | ")}` : "";

    if (items.length === 0) {
      $status.textContent = t("statusNoResults") + noteText;
      return;
    }

    $status.textContent = t("statusResults")(items.length) + noteText;

    for (const it of items) {
      const kindBadge = it.kind === "archive"
        ? `<span class="badge is-archive">${escapeHtml(t("badgeArchive")(it.version))}</span>`
        : `<span class="badge is-current">${escapeHtml(t("badgeCurrent"))}</span>`;

      const summary = it.summary ? `<div class="muted">${escapeHtml(it.summary)}</div>` : "";

      const fp = it.first_published ? `${escapeHtml(t("published"))}: ${escapeHtml(it.first_published)}` : "";
      const lu = it.last_updated ? `${escapeHtml(t("updated"))}: ${escapeHtml(it.last_updated)}` : "";
      const dateLine = (fp || lu)
        ? `<div class="muted"><small>${[fp, lu].filter(Boolean).join(" · ")}</small></div>`
        : "";

      const tags = (it.tags && it.tags.length)
        ? `<div class="tag-row">${it.tags.map(tag => `<button type="button" class="tag-pill" data-tag="${escapeHtml(tag)}">${escapeHtml(tag)}</button>`).join("")}</div>`
        : "";

      const li = document.createElement("li");
      li.className = "content-card";
      li.innerHTML = `
        <div class="card-top">
          <a href="${escapeHtml(it.url)}">${escapeHtml(it.title || "(untitled)")}</a>
          ${kindBadge}
        </div>
        ${summary}
        ${dateLine}
        ${tags}
      `;
      $results.appendChild(li);
    }

    $results.querySelectorAll(".tag-pill").forEach(btn => {
      btn.addEventListener("click", () => {
        const tag = btn.getAttribute("data-tag") || "";
        if (!tag) return;
        $q.value = toggleTagInQuery($q.value, tag);
        run();
        $q.focus();
      });
    });
  }

  function run() {
    normalizeToggles();

    const raw = $q.value;
    const onlyCurrent = $only.checked;
    const includeArchives = $inc.checked;

    setQueryToURL(raw.trim(), includeArchives, onlyCurrent);

    const parsed = parseQuery(raw);
    const flat = buildFlat(includeArchives, onlyCurrent);

    const cidFiltered = flat.filter(it => matchesContentIdFilters(it, parsed.contentIdFiltersNorm));
    const tagFiltered = cidFiltered.filter(it => matchesTagFilters(it, parsed.tagFiltersNorm));

    let results = [];
    const sortMode = $sort?.value || "relevance";

    if (parsed.textTokens.length > 0) {
      const scored = [];
      const scoreMap = new Map();

      for (const it of tagFiltered) {
        const s = scoreItem_AND(it, parsed.textTokens);
        if (s > 0) {
          scored.push({ it, s });
          scoreMap.set(it, s);
        }
      }

      scored.sort((a, b) => b.s - a.s);
      results = scored.slice(0, 50).map(x => x.it);

      results = sortResults(results, sortMode, sortMode === "relevance" ? scoreMap : null);
    } else {
      const current = tagFiltered.filter(x => x.kind === "current");
      const archives = tagFiltered.filter(x => x.kind === "archive")
        .sort((a, b) => (b.version || 0) - (a.version || 0));
      results = current.concat(archives).slice(0, 50);

      results = sortResults(results, sortMode, null);
    }

    renderTagPanel();
    render(results, raw, parsed);
    applyTagHighlighting();
  }

  async function init() {
    applyI18nStatic();

    $status.textContent = t("statusLoading");
    try {
      const res = await fetch(INDEX_URL, { cache: "no-cache" });
      if (!res.ok) throw new Error(`HTTP ${res.status}`);
      data = await res.json();

      const initState = getQueryFromURL();
      $q.value = initState.q;

      $only.checked = initState.onlyCurrent;
      $inc.checked = initState.onlyCurrent ? false : initState.includeArchives;

      if ($sort) $sort.value = initState.sortBy || "relevance";

      normalizeToggles();

      $status.textContent = t("statusReady");
      renderTagPanel();
      run();
    } catch (e) {
      console.error(e);
      $status.textContent = t("statusFailed");
    }
  }

  $q.addEventListener("input", () => run());

  $only.addEventListener("change", () => {
    if ($only.checked) $inc.checked = false;
    run();
  });

  $inc.addEventListener("change", () => {
    if ($inc.checked) $only.checked = false;
    run();
  });

  if ($sort) {
    $sort.addEventListener("change", () => run());
  }

  init();
})();
</script>
