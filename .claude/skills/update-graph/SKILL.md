---
name: update-graph
description: "Build or refresh the user's interactive idea graph — connects their ideas from meeting notes to other people's ideas in docs, email and comms, and to active projects. Saves a self-contained HTML file. Run weekly or on demand."
---

# Update Idea Graph

> **Tools:** steps call role operations (`pm.*`, `email.*`, `calendar.*`, `docs.*`, `files.*`, `comms.*`). Resolve each through `Context/agent-config.md` → **Tool Stack** and the adapter it names (`.claude/adapters/README.md`). A role set to `none` → skip that step.

Regenerate `Context/Document Hub/idea-graph.html` — a fully self-contained interactive graph connecting the user's ideas, other people's related ideas in docs, email and comms, and active projects and items.

**Design principle:** Only draw edges when the connection is OBVIOUS and SPECIFIC. A shared vague theme (both mention "AI") is NOT an edge. A shared specific concept ("bottom-sheet", "guest checkout", "ENG-123") IS an edge. When in doubt, leave the edge out. A sparse, accurate graph is better than a dense, noisy one.

---

## Instructions

Run all steps in sequence. This will take several tool calls — work through them fully before writing the output.

---

### Step 1 — Load the user's context

Read all of these:
- `GOALS.md` — active goals, milestones, and priorities
- `Context/Memory/brain.md` — how the user thinks, their active mental models

From these, extract a **keyword list**: 15–25 specific terms that represent the user's active ideas right now. These must be specific (product names, feature names, issue keys, initiative names) — not generic words like "help", "AI", "users".

Good keywords are drawn directly from the user's goals, milestones, and meeting notes — not invented here.

This keyword list drives all searches in Step 3.

---

### Step 2 — Extract the user's idea nodes (PURPLE)

**Source A: Meeting notes (last 30 days)**
Read all files in `Context/Meeting Notes/` modified in the last 30 days.
For each file, extract:
- Ideas, positions, or proposals the user expressed
- Decisions they made or drove
- Concepts they're actively exploring

For each idea node:
```json
{
  "id": "unique-slug",
  "type": "my-idea",
  "label": "Short label (5 words max)",
  "content": "1–2 sentence description of the idea",
  "source": "filename",
  "date": "YYYY-MM-DD",
  "keywords": ["specific", "terms", "from", "this", "idea"]
}
```

**Source B: Active projects and items (ORANGE)**
`pm.projects(mine)` (not completed or cancelled) and `pm.my_items(open)`.

For each project or issue, create:
```json
{
  "id": "KEY-123",
  "type": "project",
  "label": "Short title",
  "content": "Project or issue summary",
  "date": "last updated date",
  "url": "item link (pm.link)",
  "keywords": ["extracted from title and description"]
}
```

**Source C: Strategy themes (GREEN)**
Identify 3–5 recurring themes that span multiple of the user's ideas. These are meta-nodes representing broader strategic directions. Examples: "AI-First Support", "Mobile Experience", "Content Automation", "Activation". Create these only if they genuinely tie together 3+ other nodes.

```json
{
  "id": "theme-ai-first-support",
  "type": "strategy-theme",
  "label": "AI-First Support",
  "content": "Overarching theme connecting the AI support work",
  "date": null,
  "keywords": ["AI", "support", "self-serve"]
}
```

---

### Step 3 — Find other people's ideas (BLUE)

For your top 10–15 most specific keywords, search:
- **Docs** — `docs.search(keyword)`. Keep pages authored or last edited by someone other than the user
- **Email** — `email.search(keyword, 30 days ago)`, excluding threads only the user wrote
- **Comms** — `comms.search(keyword, 30 days ago)`, if configured

Skip generic words.

For each result:
- Read the content (`docs.read` / `email.thread` / `comms.thread`)
- Check: does this discuss the SAME SPECIFIC concept as one of the user's ideas, or is it just coincidentally using a similar word?
- **Include only if:** the connection is unambiguous — clearly about the same feature, initiative, or decision the user is working on
- **Exclude if:** the match is loose, generic, or would require interpretation to connect

For included results, create a **person node** (GREY) for the author if not already in the graph:
```json
{
  "id": "person-[name-slug]",
  "type": "person",
  "label": "[Display Name]",
  "content": "[Name] — [docs / email / comms] contributor",
  "date": null,
  "keywords": []
}
```

And an **others' idea node** (BLUE):
```json
{
  "id": "other-[source-id]",
  "type": "others-idea",
  "label": "Short label of the idea",
  "content": "Quoted or paraphrased content (1–2 sentences)",
  "author": "[Display Name]",
  "channel": "page title, email subject or channel",
  "date": "YYYY-MM-DD",
  "url": "source link if available",
  "keywords": ["specific", "terms"]
}
```

---

### Step 4 — Find edges (connections)

Go through all nodes and identify edges. Only create an edge when the connection is **OBVIOUS AND SPECIFIC**:

**Valid reasons for an edge:**
- Both nodes reference the same item key or project
- Both nodes use the same specific feature name (e.g., "bottom-sheet", "guest checkout")
- Both nodes mention the same person in a decision-making context
- A doc or email is directly discussing a topic from a meeting note
- A project is the direct home of an idea from a meeting

**Invalid reasons (do NOT create edge):**
- Both mention "AI" (too generic)
- Both are about the same product area broadly
- They're from the same week
- They involve the same team generally

Each edge:
```json
{
  "source": "node-id-1",
  "target": "node-id-2",
  "reason": "Specific shared concept: 'bottom-sheet interaction model'",
  "strength": "strong"
}
```

Also create edges:
- Person node → each of their idea nodes
- Strategy theme node → each idea that belongs to that theme
- Project node → each idea that was discussed in the context of that project

---

### Step 5 — Assemble the graph JSON

Build the final data object:
```json
{
  "metadata": {
    "generated": "YYYY-MM-DD",
    "window_days": 30,
    "node_counts": { "my-idea": N, "others-idea": N, "project": N, "strategy-theme": N, "person": N },
    "edge_count": N
  },
  "nodes": [...all nodes from Steps 2 and 3...],
  "edges": [...all edges from Step 4...]
}
```

---

### Step 6 — Write the HTML file

Write the complete file to `Context/Document Hub/idea-graph.html` using the template below. Replace `__GRAPH_DATA__` with the actual JSON object from Step 5. Serialise it with a JSON encoder and escape `</` as `<\/` so text from notes or email can't close the script tag.

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Idea Graph</title>
<script src="https://cdn.jsdelivr.net/npm/d3@7.9.0/dist/d3.min.js"></script>
<style>
  * { box-sizing: border-box; margin: 0; padding: 0; }
  body { font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif; background: #0f1117; color: #e0e0e0; overflow: hidden; }
  #app { display: flex; height: 100vh; }
  #sidebar { width: 300px; min-width: 300px; background: #1a1d27; border-right: 1px solid #2d3147; display: flex; flex-direction: column; z-index: 10; }
  #sidebar-header { padding: 16px; border-bottom: 1px solid #2d3147; }
  #sidebar-header h1 { font-size: 14px; font-weight: 600; color: #fff; margin-bottom: 4px; }
  #sidebar-header .meta { font-size: 11px; color: #666; }
  #search-box { margin: 12px; }
  #search-box input { width: 100%; background: #252836; border: 1px solid #2d3147; border-radius: 6px; padding: 8px 10px; color: #e0e0e0; font-size: 13px; outline: none; }
  #search-box input:focus { border-color: #9B59B6; }
  #filters { padding: 0 12px 12px; display: flex; flex-wrap: wrap; gap: 6px; }
  .filter-btn { padding: 4px 10px; border-radius: 20px; border: 1px solid; background: transparent; cursor: pointer; font-size: 11px; font-weight: 500; transition: all 0.2s; }
  .filter-btn.active { color: #fff !important; }
  .filter-btn[data-type="my-idea"] { border-color: #9B59B6; color: #9B59B6; }
  .filter-btn[data-type="my-idea"].active { background: #9B59B6; }
  .filter-btn[data-type="others-idea"] { border-color: #3498DB; color: #3498DB; }
  .filter-btn[data-type="others-idea"].active { background: #3498DB; }
  .filter-btn[data-type="project"] { border-color: #E67E22; color: #E67E22; }
  .filter-btn[data-type="project"].active { background: #E67E22; }
  .filter-btn[data-type="strategy-theme"] { border-color: #27AE60; color: #27AE60; }
  .filter-btn[data-type="strategy-theme"].active { background: #27AE60; }
  .filter-btn[data-type="person"] { border-color: #7f8c8d; color: #7f8c8d; }
  .filter-btn[data-type="person"].active { background: #7f8c8d; }
  #node-detail { flex: 1; overflow-y: auto; padding: 12px; border-top: 1px solid #2d3147; }
  #node-detail .placeholder { color: #444; font-size: 12px; text-align: center; margin-top: 40px; }
  #node-detail .detail-type { font-size: 10px; font-weight: 600; text-transform: uppercase; letter-spacing: 0.08em; margin-bottom: 6px; }
  #node-detail .detail-label { font-size: 15px; font-weight: 600; color: #fff; margin-bottom: 8px; line-height: 1.3; }
  #node-detail .detail-content { font-size: 12px; color: #aaa; line-height: 1.6; margin-bottom: 12px; }
  #node-detail .detail-meta { font-size: 11px; color: #555; }
  #node-detail .detail-connections { margin-top: 12px; }
  #node-detail .detail-connections h4 { font-size: 11px; font-weight: 600; color: #666; text-transform: uppercase; letter-spacing: 0.06em; margin-bottom: 8px; }
  #node-detail .connection-item { padding: 6px 8px; background: #252836; border-radius: 6px; margin-bottom: 4px; cursor: pointer; }
  #node-detail .connection-item:hover { background: #2d3147; }
  #node-detail .connection-item .conn-label { font-size: 12px; color: #ccc; }
  #node-detail .connection-item .conn-reason { font-size: 10px; color: #555; margin-top: 2px; }
  #graph-container { flex: 1; position: relative; }
  svg { width: 100%; height: 100%; }
  .node circle { cursor: pointer; transition: opacity 0.3s; }
  .node text { pointer-events: none; font-size: 11px; fill: #ccc; }
  .link { stroke: #2d3147; stroke-width: 1.5; opacity: 0.6; transition: opacity 0.3s; }
  .link.highlighted { stroke: #9B59B6; stroke-width: 2.5; opacity: 1; }
  .node.dimmed circle { opacity: 0.15; }
  .node.dimmed text { opacity: 0.15; }
  .link.dimmed { opacity: 0.05; }
  #legend { position: absolute; bottom: 16px; right: 16px; background: rgba(26,29,39,0.9); border: 1px solid #2d3147; border-radius: 8px; padding: 12px 14px; }
  #legend h4 { font-size: 10px; font-weight: 600; color: #555; text-transform: uppercase; letter-spacing: 0.08em; margin-bottom: 8px; }
  .legend-item { display: flex; align-items: center; gap: 8px; margin-bottom: 5px; font-size: 11px; color: #888; }
  .legend-dot { width: 10px; height: 10px; border-radius: 50%; flex-shrink: 0; }
  #meta-bar { position: absolute; top: 12px; right: 16px; font-size: 11px; color: #444; }
  .tooltip { position: absolute; background: rgba(26,29,39,0.95); border: 1px solid #2d3147; border-radius: 6px; padding: 8px 10px; font-size: 11px; color: #ccc; pointer-events: none; max-width: 200px; line-height: 1.4; }
</style>
</head>
<body>
<div id="app">
  <div id="sidebar">
    <div id="sidebar-header">
      <h1>Idea Graph</h1>
      <div class="meta" id="meta-text">Loading...</div>
    </div>
    <div id="search-box"><input type="text" placeholder="Search ideas..." id="search-input" /></div>
    <div id="filters">
      <button class="filter-btn active" data-type="my-idea">My ideas</button>
      <button class="filter-btn active" data-type="others-idea">Others</button>
      <button class="filter-btn active" data-type="project">Projects</button>
      <button class="filter-btn active" data-type="strategy-theme">Strategy</button>
      <button class="filter-btn active" data-type="person">People</button>
    </div>
    <div id="node-detail"><div class="placeholder">Click a node to explore</div></div>
  </div>
  <div id="graph-container">
    <svg id="graph-svg"></svg>
    <div id="legend">
      <h4>Node types</h4>
      <div class="legend-item"><div class="legend-dot" style="background:#9B59B6"></div>My ideas</div>
      <div class="legend-item"><div class="legend-dot" style="background:#3498DB"></div>Others' ideas</div>
      <div class="legend-item"><div class="legend-dot" style="background:#E67E22"></div>Projects</div>
      <div class="legend-item"><div class="legend-dot" style="background:#27AE60"></div>Strategy theme</div>
      <div class="legend-item"><div class="legend-dot" style="background:#7f8c8d"></div>People</div>
    </div>
    <div id="meta-bar"></div>
    <div class="tooltip" id="tooltip" style="display:none"></div>
  </div>
</div>
<script>
const RAW = __GRAPH_DATA__;

// Escape all text that came from notes, docs or email before it touches innerHTML
const esc = v => String(v ?? '').replace(/[&<>"']/g, c => ({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;'}[c]));
const safeUrl = u => /^https?:\/\//i.test(u || '') ? esc(u) : '';

const COLOR = {
  'my-idea': '#9B59B6',
  'others-idea': '#3498DB',
  'project': '#E67E22',
  'strategy-theme': '#27AE60',
  'person': '#7f8c8d'
};

const RADIUS = {
  'my-idea': 9,
  'others-idea': 7,
  'project': 11,
  'strategy-theme': 13,
  'person': 6
};

// Freshness: nodes older than 21 days at reduced opacity
function freshnessOpacity(date) {
  if (!date) return 1;
  const days = (Date.now() - new Date(date)) / 86400000;
  if (days < 14) return 1;
  if (days < 21) return 0.75;
  return 0.5;
}

// Meta bar
const m = RAW.metadata;
document.getElementById('meta-text').textContent =
  `Generated ${m.generated} · ${m.window_days}-day window · ${RAW.nodes.length} nodes · ${RAW.edges.length} connections`;
document.getElementById('meta-bar').textContent =
  `${RAW.nodes.length} nodes · ${RAW.edges.length} connections · ${m.generated}`;

const svg = d3.select('#graph-svg');
const container = document.getElementById('graph-container');
let W = container.clientWidth, H = container.clientHeight;

const g = svg.append('g');

svg.call(d3.zoom().scaleExtent([0.2, 4]).on('zoom', e => g.attr('transform', e.transform)));

let activeFilters = new Set(['my-idea','others-idea','project','strategy-theme','person']);
let selectedNode = null;

const simulation = d3.forceSimulation()
  .force('link', d3.forceLink().id(d => d.id).distance(d => {
    if (d.source.type === 'strategy-theme' || d.target.type === 'strategy-theme') return 140;
    if (d.source.type === 'person' || d.target.type === 'person') return 60;
    return 110;
  }))
  .force('charge', d3.forceManyBody().strength(d => d.type === 'strategy-theme' ? -600 : -250))
  .force('center', d3.forceCenter(W / 2, H / 2))
  .force('collision', d3.forceCollide(d => RADIUS[d.type] + 14));

let linkSel, nodeSel;

function render() {
  const visibleIds = new Set(RAW.nodes.filter(n => activeFilters.has(n.type)).map(n => n.id));
  const nodes = RAW.nodes.filter(n => visibleIds.has(n.id));
  const edges = RAW.edges.filter(e => {
    const sid = typeof e.source === 'object' ? e.source.id : e.source;
    const tid = typeof e.target === 'object' ? e.target.id : e.target;
    return visibleIds.has(sid) && visibleIds.has(tid);
  });

  g.selectAll('*').remove();

  linkSel = g.append('g').selectAll('line')
    .data(edges).join('line')
    .attr('class', 'link')
    .attr('stroke', '#2d3147');

  nodeSel = g.append('g').selectAll('g')
    .data(nodes, d => d.id).join('g')
    .attr('class', 'node')
    .call(d3.drag()
      .on('start', (e, d) => { if (!e.active) simulation.alphaTarget(0.3).restart(); d.fx = d.x; d.fy = d.y; })
      .on('drag', (e, d) => { d.fx = e.x; d.fy = e.y; })
      .on('end', (e, d) => { if (!e.active) simulation.alphaTarget(0); d.fx = null; d.fy = null; }));

  nodeSel.append('circle')
    .attr('r', d => RADIUS[d.type] || 8)
    .attr('fill', d => COLOR[d.type] || '#888')
    .attr('opacity', d => freshnessOpacity(d.date))
    .attr('stroke', '#1a1d27')
    .attr('stroke-width', 2);

  nodeSel.append('text')
    .text(d => d.label.length > 22 ? d.label.slice(0, 22) + '…' : d.label)
    .attr('x', d => (RADIUS[d.type] || 8) + 5)
    .attr('y', 4)
    .attr('fill', '#bbb');

  nodeSel.on('click', (e, d) => selectNode(d, edges))
    .on('mouseover', (e, d) => showTooltip(e, d))
    .on('mousemove', (e) => moveTooltip(e))
    .on('mouseout', () => hideTooltip());

  simulation.nodes(nodes).on('tick', () => {
    linkSel
      .attr('x1', d => d.source.x).attr('y1', d => d.source.y)
      .attr('x2', d => d.target.x).attr('y2', d => d.target.y);
    nodeSel.attr('transform', d => `translate(${d.x},${d.y})`);
  });
  simulation.force('link').links(edges);
  simulation.alpha(0.8).restart();
}

function selectNode(d, edges) {
  selectedNode = d;
  const connectedIds = new Set();
  const connectedEdges = [];
  edges.forEach(e => {
    const sid = typeof e.source === 'object' ? e.source.id : e.source;
    const tid = typeof e.target === 'object' ? e.target.id : e.target;
    if (sid === d.id || tid === d.id) {
      connectedIds.add(sid === d.id ? tid : sid);
      connectedEdges.push(e);
    }
  });
  connectedIds.add(d.id);

  nodeSel.classed('dimmed', n => !connectedIds.has(n.id));
  linkSel.classed('dimmed', e => {
    const sid = typeof e.source === 'object' ? e.source.id : e.source;
    const tid = typeof e.target === 'object' ? e.target.id : e.target;
    return sid !== d.id && tid !== d.id;
  });
  linkSel.classed('highlighted', e => {
    const sid = typeof e.source === 'object' ? e.source.id : e.source;
    const tid = typeof e.target === 'object' ? e.target.id : e.target;
    return sid === d.id || tid === d.id;
  });

  const allNodes = RAW.nodes;
  const connNodes = [...connectedIds].filter(id => id !== d.id).map(id => ({
    node: allNodes.find(n => n.id === id),
    edge: connectedEdges.find(e => {
      const s = typeof e.source === 'object' ? e.source.id : e.source;
      const t = typeof e.target === 'object' ? e.target.id : e.target;
      return s === id || t === id;
    })
  })).filter(x => x.node);

  const typeColor = COLOR[d.type] || '#888';
  const detail = document.getElementById('node-detail');
  detail.innerHTML = `
    <div class="detail-type" style="color:${typeColor}">${d.type.replace('-', ' ').replace('pri idea', 'my idea')}</div>
    <div class="detail-label">${esc(d.label)}</div>
    <div class="detail-content">${esc(d.content)}</div>
    <div class="detail-meta">${d.date ? `📅 ${esc(d.date)}` : ''} ${d.author ? `· 👤 ${esc(d.author)}` : ''} ${d.channel ? `· ${esc(d.channel)}` : ''} ${d.source ? `· 📄 ${esc(d.source)}` : ''}</div>
    ${safeUrl(d.url) ? `<div class="detail-meta" style="margin-top:4px"><a href="${safeUrl(d.url)}" target="_blank" rel="noopener" style="color:#3498DB;font-size:11px">Open source ↗</a></div>` : ''}
    ${connNodes.length ? `
      <div class="detail-connections">
        <h4>${connNodes.length} connection${connNodes.length > 1 ? 's' : ''}</h4>
        ${connNodes.map(({node, edge}) => `
          <div class="connection-item" data-node-id="${esc(node.id)}">
            <div class="conn-label" style="color:${COLOR[node.type]}">${esc(node.label)}</div>
            ${edge ? `<div class="conn-reason">${esc(edge.reason)}</div>` : ''}
          </div>
        `).join('')}
      </div>
    ` : ''}
  `;
  detail.querySelectorAll('.connection-item').forEach(el => el.addEventListener('click', () => highlightNode(el.dataset.nodeId)));
}

window.highlightNode = (id) => {
  const node = RAW.nodes.find(n => n.id === id);
  if (node) {
    const visibleNodes = simulation.nodes();
    const visibleEdges = simulation.force('link').links();
    const found = visibleNodes.find(n => n.id === id);
    if (found) selectNode(found, visibleEdges);
  }
};

const tooltip = document.getElementById('tooltip');
function showTooltip(e, d) {
  tooltip.style.display = 'block';
  tooltip.innerHTML = `<strong style="color:${COLOR[d.type]}">${esc(d.label)}</strong><br>${d.content ? esc(d.content.slice(0, 80)) + (d.content.length > 80 ? '…' : '') : ''}`;
}
function moveTooltip(e) {
  const rect = container.getBoundingClientRect();
  tooltip.style.left = (e.clientX - rect.left + 12) + 'px';
  tooltip.style.top = (e.clientY - rect.top - 10) + 'px';
}
function hideTooltip() { tooltip.style.display = 'none'; }

// Filters
document.querySelectorAll('.filter-btn').forEach(btn => {
  btn.addEventListener('click', () => {
    const t = btn.dataset.type;
    if (activeFilters.has(t)) { activeFilters.delete(t); btn.classList.remove('active'); }
    else { activeFilters.add(t); btn.classList.add('active'); }
    render();
  });
});

// Search
document.getElementById('search-input').addEventListener('input', e => {
  const q = e.target.value.toLowerCase().trim();
  if (!q) { nodeSel.classed('dimmed', false); linkSel.classed('dimmed', false); return; }
  const matchIds = new Set(RAW.nodes.filter(n =>
    n.label.toLowerCase().includes(q) || (n.content || '').toLowerCase().includes(q)
  ).map(n => n.id));
  nodeSel.classed('dimmed', n => !matchIds.has(n.id));
  linkSel.classed('dimmed', e => {
    const s = typeof e.source === 'object' ? e.source.id : e.source;
    const t = typeof e.target === 'object' ? e.target.id : e.target;
    return !matchIds.has(s) && !matchIds.has(t);
  });
});

// Reset on background click
svg.on('click', (e) => {
  if (e.target.tagName === 'svg' || e.target.tagName === 'g') {
    nodeSel.classed('dimmed', false);
    linkSel.classed('dimmed', false).classed('highlighted', false);
    selectedNode = null;
    document.getElementById('node-detail').innerHTML = '<div class="placeholder">Click a node to explore</div>';
  }
});

window.addEventListener('resize', () => {
  W = container.clientWidth; H = container.clientHeight;
  simulation.force('center', d3.forceCenter(W / 2, H / 2)).alpha(0.3).restart();
});

render();
</script>
</body>
</html>
```

---

### Step 7 — Confirm output

After writing the file, say:

> **Graph updated** → `Context/Document Hub/idea-graph.html`
> Open it in any browser: `open "Context/Document Hub/idea-graph.html"`
>
> **[N] nodes:** [N] my ideas · [N] others' ideas · [N] projects · [N] strategy themes · [N] people
> **[N] connections** (specific matches only)
>
> Tip: Click any node to see its connections and the reason they're linked.

---

## Edge cases

- **No results for a keyword**: skip silently, don't create an empty node
- **Doc or email too vague to connect**: exclude it — when in doubt, leave it out
- **Duplicate ideas** (same concept in two meeting notes): merge into one node, cite both sources in `content`
- **More than 20 projects/items**: limit to the 20 most recently updated
- **No connections found for a node**: still include the node — an isolated idea is valid, it just has no edges yet
