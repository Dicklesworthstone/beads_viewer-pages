# Changelog

All notable changes to the **beads_viewer-pages** deployment repository are documented here.

This repo hosts the GitHub Pages site for [beads_viewer](https://github.com/Dicklesworthstone/beads_viewer),
serving a static, fully offline-capable issue tracker dashboard at
<https://dicklesworthstone.github.io/beads_viewer-pages/>.

The application itself is built in the [beads_viewer](https://github.com/Dicklesworthstone/beads_viewer)
source repo (Go + Rust WASM). This pages repo contains the compiled/deployed artifacts.
Source-side version tags (e.g., v0.11.1) are noted where applicable.

---

## [2026-02-21] License and social preview updates

Two metadata commits on the same day. No application code changed.

### License

- **Upgraded to MIT with OpenAI/Anthropic Rider.**
  The new rider restricts use by OpenAI, Anthropic, and their affiliates without
  express written permission from Jeffrey Emanuel. Replaces the plain MIT license
  added one month earlier.
  ([`74d5f2f`](https://github.com/Dicklesworthstone/beads_viewer-pages/commit/74d5f2f51d33e2034829ce6a132132fb49737133))

### Repository branding

- **Added GitHub social preview image** (`gh_og_share_image.png`, 1280x640) for
  consistent link previews when the repository URL is shared on social media.
  ([`2154e85`](https://github.com/Dicklesworthstone/beads_viewer-pages/commit/2154e854f5b76733521bb64e59cddf3d2da4aaff))

---

## [2026-01-21] Add MIT license

- Initial MIT license file (`LICENSE`). Later replaced by the OpenAI/Anthropic
  Rider variant above.
  ([`84fef22`](https://github.com/Dicklesworthstone/beads_viewer-pages/commit/84fef2212bd7c24a4b04ba16e333721fa8512326))

---

## [2025-12-18] Initial deploy -- beads_viewer v0.11.1

First deployment of the Beads Viewer static dashboard to GitHub Pages.
Corresponds to source release
[v0.11.1](https://github.com/Dicklesworthstone/beads_viewer/releases/tag/v0.11.1)
(published 2025-12-19 UTC).

([`664da72`](https://github.com/Dicklesworthstone/beads_viewer-pages/commit/664da72a00b1582e732ccbf88f2c66a34e5965a8))

31 files, 28,760 lines added. Every feature below shipped in this single commit.

---

### Client-side SQLite engine

- **sql.js WASM** (`vendor/sql-wasm.js` + `sql-wasm.wasm`, 640 KB) loads the
  entire issue database (`beads.sqlite3`, 2.3 MB) in the browser.
- **OPFS caching** stores the database in the Origin Private File System on
  repeat visits; a hash-based cache key triggers automatic invalidation when the
  database changes.
- **FTS5 full-text search** with BM25 ranking and `snippet()` highlighting.
  Falls back to `LIKE` queries when FTS is unavailable.
- **Materialized view queries** (`issue_overview_mv`) for fast filtered/sorted
  issue listing.
- **Chunk reassembly** support for databases split into multiple files, controlled
  by `beads.sqlite3.config.json`.

### Hybrid search and scoring

- **Multi-signal ranking** (`hybrid_scorer.js`, 88 lines) combines six
  dimensions: text relevance, PageRank, status weight, blocker impact, priority,
  and recency.
- **Five search presets**: default, bug-hunting, sprint-planning, impact-first,
  text-only -- each with tuned weight vectors.
- **Optional WASM accelerator** (`wasm_loader.js`, 154 lines) loads a Rust-compiled
  hybrid scorer for datasets above 5,000 issues. Falls back to the JS
  implementation automatically.

### Cross-Origin Isolation

- **Service worker** (`coi-serviceworker.js`, 136 lines) injects
  `Cross-Origin-Embedder-Policy: credentialless` and
  `Cross-Origin-Opener-Policy: same-origin` headers, enabling `SharedArrayBuffer`
  on GitHub Pages (which does not allow custom response headers).

### SPA routing and navigation

- **Hash-based router** with five routes: `/` (dashboard), `/issues` (list),
  `/issue/:id` (detail), `/insights` (graph analytics), `/graph` (visualization).
- **URL-encoded filter state** -- status, priority, type, labels, assignee,
  has-blockers, is-blocking, sort order, and search query are all persisted to
  the URL for shareable deep links.
- **Browser history integration** via `history.replaceState`.

### Issue viewer (`viewer.js`, 3,472 lines)

- **Issue list** with multi-criteria filtering: status, priority, type, label,
  assignee, has-blockers, is-blocking. Sort by priority, triage score, or text
  relevance.
- **Issue detail modal** with markdown rendering (marked + DOMPurify), Mermaid
  dependency diagrams, blocker/dependent lists, and JSON metadata with syntax
  highlighting.
- **Quick wins detection** -- surfaces actionable issues that unblock the most
  downstream items.
- **Top-K set computation** -- greedy algorithm that finds the minimal set of
  issues to close for maximum unblocking.
- **Top what-if ranking** (`topWhatIf`) -- ranks all issues by cascade
  unblocking potential.
- **Cycle break suggestions** -- recommends dependency edges to remove for DAG
  restoration.
- **Triage data integration** -- loads `data/triage.json` for robot-generated
  recommendations, quick wins, and blocker-clearing suggestions.
- **Toast notifications** for non-blocking status messages.
- **Error display** with optional action buttons and dismissible state.
- **Diagnostics panel** (toggle with `d`) showing WASM status, OPFS
  availability, database source, load time, query count, and memory statistics.

### Graph analysis engine (Rust WASM)

- **Compiled WASM module** (`vendor/bv_graph.js` + `bv_graph_bg.wasm`, 214 KB)
  implementing eight graph algorithms:
  1. **PageRank** -- node importance in the dependency network.
  2. **Betweenness centrality** -- bottleneck detection.
  3. **Eigenvector centrality** -- influence propagation.
  4. **K-core decomposition** -- structural cohesion analysis.
  5. **HITS** (hub/authority scores) -- identifying hub coordinators vs. leaf
     deliverables.
  6. **Critical path heights** -- longest dependency chain depth.
  7. **Slack values** -- schedule flexibility (0 = on critical path).
  8. **Articulation points** -- cut vertices whose removal disconnects the graph.
- **Cycle detection** with enumeration (up to 100 cycles).
- **What-if simulation** -- simulate closing an issue to see cascade impact.
- **Subgraph extraction** with RAII-style `withSubgraph()` wrapper ensuring
  WASM memory cleanup.
- **Fallback mode** -- when WASM is unavailable, uses pre-computed metrics from
  `data/graph_layout.json`.

### Force-graph visualization (`graph.js`, 3,847 lines)

- **Interactive WebGL renderer** powered by `force-graph` v1.43.5 + D3 v7
  physics simulation.
- **Dracula-inspired theme** with colorblind-friendly palette (10 distinct
  label colors).

#### Five view modes

1. **Force-Directed** -- standard physics-based layout.
2. **Hierarchy** -- top-down DAG layout with vertical depth alignment.
3. **Radial** -- circular layout emanating from a selected node.
4. **Status Clusters** -- groups nodes by status (open, in-progress, blocked).
5. **Label Galaxy** -- label-based clustering with convex hull outlines
   computed via `d3.polygonHull`.

#### Six layout presets

Each preset configures force simulation parameters (link distance, charge
strength, center force, warmup/cooldown ticks):

1. Force-Directed (balanced readability)
2. Compact DAG (tight spacing, emphasizes structure)
3. Spread (maximum spacing, ideal for exports)
4. Orthogonal (grid-snapped with grid-strength forces)
5. Radial (circular from center/selection)
6. Status Clusters (grouped by status)

#### Heatmap overlay

- Color and size nodes by one of four metrics: PageRank, betweenness
  centrality, critical path depth, or in-degree.
- Toggle via `h` key or UI button.

#### Interactive analysis

- **What-if mode** -- shift+click a node to simulate closing it. Affected
  nodes glow cyan, statistics update live, and a cascade animation plays.
- **Connected subgraph highlighting** -- click a node for gold-glow BFS
  expansion of its dependency neighborhood.
- **Dependency path tracing** -- ctrl/meta+click highlights the full
  blocker and dependent chains of a node.
- **Critical path animation** -- animated red highlighting of the longest
  dependency chain. Toggle with `c` key.
- **Cycle navigator** -- step through detected cycles with previous/next
  (`[`/`]` keys), highlight cycle edges, and zoom-to-fit (`y` to toggle).

#### Time-travel animation

- **History replay** from `data/history.json` (64 KB). Plays back project
  evolution commit-by-commit with a timeline scrubber, play/pause, speed
  control (0.5x--10x), and skip-to-start/end.
- Toggle with `t` key; space to play/pause.

#### Precomputed layout

- `data/graph_layout.json` (92 KB) provides cached node positions for instant
  initial render of 487 nodes.

#### Node search and filtering

- Real-time text search within the graph view.
- Filter by status, type, and label.

### Insights dashboard

The `/insights` route presents eight expandable metric panels powered by
the WASM graph engine, each with an educational tooltip explaining the
algorithm, its significance, and recommended actions:

1. **Bottlenecks** -- betweenness centrality top-N.
2. **Keystones** -- highest in-degree / most depended-upon.
3. **Influencers** -- PageRank and eigenvector centrality top-N.
4. **Most Blocking** -- issues blocking the most downstream work.
5. **HITS Hubs** -- hub scores (link-out coordinators).
6. **HITS Authorities** -- authority scores (leaf deliverables).
7. **K-Core Cohesion** -- k-core decomposition tiers.
8. **Cut Points** -- articulation vertices on the undirected graph view.

Additional insights sections:

- **AI Priority Picks** -- top-K greedy set with marginal unblock counts.
- **What-If Cascade Impact** -- top issues ranked by `topWhatIf`.
- **Cycle Navigator** -- step through detected cycles with highlight, zoom,
  and path display.
- **Cycle Break Suggestions** -- edge removal recommendations with cycle-count
  impact.
- **Health stats** -- active nodes, graph engine status, cycle count,
  actionable count, blocked count.

### Charts dashboard (`charts.js`, 761 lines)

All charts use the Dracula color palette and are responsive.

- **Burndown/burnup progress chart** -- tracks issue closure over time.
- **Label dependency heatmap** -- custom canvas-rendered matrix showing
  cross-label dependency density.
- **Priority distribution pie chart** (P0--P4).
- **Type breakdown bar chart** (bug, feature, task, epic, chore).

### Keyboard navigation

#### Global shortcuts (viewer.js)

| Key | Action |
|-----|--------|
| `/` | Focus search input |
| `?` | Open help & shortcuts modal |
| `d` | Toggle diagnostics panel |
| `c` | Toggle critical path highlighting |
| `h` / `l` | Navigate to first blocker / first dependent (issue modal open) |
| `o` | Open first issue from list view |
| `Esc` | Close modal or reset selection |

#### Graph shortcuts (graph.js)

| Key | Action |
|-----|--------|
| `1`--`5` | Switch view mode (Force / Hierarchy / Radial / Cluster / Label Galaxy) |
| `h` | Toggle heatmap overlay |
| `w` | What-if on selected node |
| `c` | Toggle critical path |
| `y` | Toggle cycle navigator |
| `[` / `]` | Previous / next cycle |
| `r` | Reset view |
| `t` | Toggle time-travel mode |
| `Space` | Play/pause time-travel |
| `Left` / `Right` | Navigate by priority (P0 first, then PageRank within tier) |
| `Up` / `Down` | Navigate by k-core clique (higher/lower cohesion) |
| `Ctrl/Cmd+F` | Search request |

#### Mouse and touch gestures

- Click = select node, Shift+click = what-if, Ctrl/Meta+click = dependency path.
- Hover = tooltip with gold-glow BFS, Drag = pan or move node.
- Double-click = open issue detail modal.
- Touch: tap = select, drag = pan, pinch = zoom, long-press = info.
- Mobile detail pane: swipe down to dismiss. Desktop: swipe right.

### Mobile and responsive design

- **Mobile-first CSS** (`styles.css`, 2,374 lines) with touch-optimized buttons,
  bottom-sheet modals, horizontal-scrolling stat cards, and a mobile navigation
  bar.
- **Mobile hamburger menu** and collapsible filter panel.
- **PWA metadata** -- `mobile-web-app-capable`, theme-color (light and dark),
  `viewport-fit: cover`.
- **Dark mode** -- on by default, persisted to `localStorage`, respects
  `prefers-color-scheme`.
- **Graph detail pane** -- bottom sheet (60% viewport) on mobile, 400px side
  panel on desktop, with swipe-to-dismiss on both.

### Security

- **Content Security Policy** enforces `default-src 'self'` with narrow
  exceptions (`'unsafe-eval'` and `'unsafe-inline'` for Tailwind JIT and
  Alpine.js).
- **DOMPurify** sanitizes all rendered markdown.
- Fully self-contained: no CDN calls, no external network requests.

### Data files

| File | Size | Contents |
|------|------|----------|
| `beads.sqlite3` | 2.3 MB | Full issue database (487 issues, FTS5 index) |
| `data/meta.json` | 157 B | Generation timestamp and issue count |
| `data/triage.json` | 2.1 KB | Robot triage recommendations and quick wins |
| `data/project_health.json` | 1.3 KB | Velocity metrics, weekly closure rates, graph density |
| `data/history.json` | 64 KB | Commit-level history for time-travel visualization |
| `data/graph_layout.json` | 92 KB | Precomputed force-graph node positions |

### Bundled vendor libraries

| Library | File(s) | Size | Purpose |
|---------|---------|------|---------|
| sql.js | `sql-wasm.js` + `sql-wasm.wasm` | 689 KB | Client-side SQLite engine |
| bv_graph | `bv_graph.js` + `bv_graph_bg.wasm` | 249 KB | Rust WASM graph algorithms |
| force-graph | `force-graph.min.js` | 158 KB | WebGL force-directed graph renderer (v1.43.5) |
| D3 | `d3.v7.min.js` | 273 KB | Physics simulation and data utilities |
| Chart.js | `chart.umd.min.js` | 201 KB | Bar/pie/line charts |
| Mermaid | `mermaid.min.js` | 3.2 MB | Dependency diagram rendering |
| Alpine.js | `alpine.min.js` + `alpine-collapse.min.js` | 45 KB | Reactive UI framework |
| Tailwind CSS | `tailwindcss.js` | 398 KB | Utility-first CSS (JIT in browser) |
| marked | `marked.min.js` | 36 KB | Markdown-to-HTML conversion |
| DOMPurify | `dompurify.min.js` | 20 KB | XSS-safe HTML sanitization |
| Inter | `inter-variable.woff2` | 23 KB | Sans-serif typeface |
| JetBrains Mono | `jetbrains-mono-regular.woff2` | 90 KB | Monospace typeface |

---

## Repository metadata

- **GitHub Pages URL**: <https://dicklesworthstone.github.io/beads_viewer-pages/>
- **Source repo**: <https://github.com/Dicklesworthstone/beads_viewer>
- **Created**: 2025-12-17
- **License**: MIT with OpenAI/Anthropic Rider
- **Dataset**: 487 issues, 629 dependencies, 0 cycles (100% closed at time of deploy)
