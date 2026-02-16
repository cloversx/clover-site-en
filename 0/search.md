---
layout: page
title: Search
permalink: /0/search/

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
    Only current
  </label>

  <label class="search-toggle">
    <input id="includeArchives" type="checkbox" />
    Include archives
  </label>
</div>

<!-- Tag panel -->
<div id="tagPanel" class="tag-panel">
  <div class="muted"><strong>Tags</strong></div>
  <div id="tagList" class="tag-list"></div>
</div>

<div id="searchStatus" class="muted"></div>
<ul id="results" class="content-list"></ul>

<script>
(() => {

  const INDEX_URL = "{{ '/0/contents-nested.json' | relative_url }}";

  const $q = document.getElementById('q');
  const $only = document.getElementById('onlyCurrent');
  const $inc = document.getElementById('includeArchives');
  const $status = document.getElementById('searchStatus');
  const $results = document.getElementById('results');
  const $tagList = document.getElementById('tagList');

  const PINNED_TAGS = [
    {% for t in page.pinned_tags %}
      {{ t | jsonify }}{% unless forloop.last %},{% endunless %}
    {% endfor %}
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
    const tagFiltersNorm = tagFilters.map(normalize);

    const t2 = extractFilters(t1.remainingText, "content_id");

    const contentIdFilters = t2.values;
    const contentIdFiltersNorm = contentIdFilters.map(normalize);

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
          url: entry.current.url ?? ""
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
            url: a.url ?? ""
          });
        }
      }
    }

    return out;
  }

  function collectTagCounts(includeArchives, onlyCurrent) {

    const counts = new Map();

    const flat = buildFlat(includeArchives, onlyCurrent);

    for (const it of flat) {

      for (const t of (it.tags || [])) {

        const k = normalize(t);

        counts.set(k, {
          tag: t,
          count: (counts.get(k)?.count || 0) + 1
        });
      }
    }

    return counts;
  }

  function toggleTag(tag) {

    const token = tag.includes(" ") ? `tag:"${tag}"` : `tag:${tag}`;

    const raw = $q.value.trim();

    const re = new RegExp(`(^|\\s)${token.replace(/[.*+?^${}()|[\]\\]/g,"\\$&")}(?=\\s|$)`,"g");

    if (re.test(raw)) {

      const next = raw.replace(re," ").replace(/\s+/g," ").trim();
      $q.value = next;

    } else {

      $q.value = raw ? raw + " " + token : token;
    }

    run();
    $q.focus();
  }

  function renderTagPanel() {

    if (!data) return;

    const counts = collectTagCounts($inc.checked, $only.checked);

    const ordered = [];
    const used = new Set();

    for (const t of PINNED_TAGS) {

      const k = normalize(t);

      if (counts.has(k)) {

        ordered.push(counts.get(k));
        used.add(k);
      }
    }

    const rest = [];

    for (const [k,v] of counts.entries()) {

      if (!used.has(k)) rest.push(v);
    }

    rest.sort((a,b)=> b.count-a.count || a.tag.localeCompare(b.tag));

    ordered.push(...rest);

    $tagList.innerHTML="";

    for (const it of ordered) {

      const btn=document.createElement("button");

      btn.className="tag-pill";
      btn.textContent=`${it.tag} (${it.count})`;

      btn.onclick=()=>toggleTag(it.tag);

      $tagList.appendChild(btn);
    }
  }

  function matchesTagFilters(item, filters) {

    if (!filters.length) return true;

    const tags=item.tags.map(normalize);

    return filters.every(f=>tags.includes(f));
  }

  function matchesContentIdFilters(item,filters){

    if(!filters.length)return true;

    return filters.includes(normalize(item.content_id));
  }

  function scoreItem_AND(item,tokens){

    if(!tokens.length)return 0;

    const title=normalize(item.title);
    const summary=normalize(item.summary);
    const tags=normalize(item.tags.join(" "));

    let score=0;

    for(const tok of tokens){

      const t=tok.value;

      let matched=false;

      if(title.includes(t)){score+=5;matched=true;}
      if(tags.includes(t)){score+=3;matched=true;}
      if(summary.includes(t)){score+=1;matched=true;}

      if(!matched)return 0;
    }

    if(item.kind==="current")score+=0.2;

    score+=Math.min(0.19,item.version*0.01);

    return score;
  }

  function normalizeToggles(){

    if($only.checked && $inc.checked){

      $inc.checked=false;
    }
  }

  function run(){

    normalizeToggles();

    renderTagPanel();

    const raw=$q.value;

    const parsed=parseQuery(raw);

    const flat=buildFlat($inc.checked,$only.checked);

    const cidFiltered=flat.filter(it=>matchesContentIdFilters(it,parsed.contentIdFiltersNorm));

    const tagFiltered=cidFiltered.filter(it=>matchesTagFilters(it,parsed.tagFiltersNorm));

    let results=[];

    if(parsed.textTokens.length){

      const scored=[];

      for(const it of tagFiltered){

        const s=scoreItem_AND(it,parsed.textTokens);

        if(s>0)scored.push({it,s});
      }

      scored.sort((a,b)=>b.s-a.s);

      results=scored.map(x=>x.it);

    }else{

      const current=tagFiltered.filter(x=>x.kind==="current");

      const archives=tagFiltered.filter(x=>x.kind==="archive")
        .sort((a,b)=>b.version-a.version);

      results=current.concat(archives);
    }

    renderResults(results,parsed);
  }

  function renderResults(items,parsed){

    $results.innerHTML="";

    if(!items.length){

      $status.textContent="No results.";
      return;
    }

    $status.textContent=`${items.length} result(s).`;

    for(const it of items){

      const li=document.createElement("li");

      li.className="content-card";

      li.innerHTML=`
        <div class="card-top">
          <a href="${escapeHtml(it.url)}">${escapeHtml(it.title)}</a>
          <span class="badge">${it.kind==="archive"?"Archive v"+it.version:"Current"}</span>
        </div>
        <div class="muted">${escapeHtml(it.summary)}</div>
      `;

      $results.appendChild(li);
    }
  }

  async function init(){

    const res=await fetch(INDEX_URL);

    data=await res.json();

    run();
  }

  $q.addEventListener("input",run);

  $only.addEventListener("change",()=>{

    if($only.checked)$inc.checked=false;
    run();
  });

  $inc.addEventListener("change",()=>{

    if($inc.checked)$only.checked=false;
    run();
  });

  init();

})();
</script>
