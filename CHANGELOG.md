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

### Changed
- **License upgraded to MIT with OpenAI/Anthropic Rider** -- restricts use by OpenAI, Anthropic,
  and their affiliates without express written permission from Jeffrey Emanuel.
  Replaces the plain MIT license added in January.
  ([`74d5f2f`](https://github.com/Dicklesworthstone/beads_viewer-pages/commit/74d5f2f51d33e2034829ce6a132132fb49737133))

### Added
- **GitHub social preview image** (1280x640 OG image) for consistent link previews when sharing
  the repository URL on social media.
  ([`2154e85`](https://github.com/Dicklesworthstone/beads_viewer-pages/commit/2154e854f5b76733521bb64e59cddf3d2da4aaff))

---

## [2026-01-21] Add MIT license

### Added
- Initial MIT license file (later replaced with the OpenAI/Anthropic Rider variant).
  ([`84fef22`](https://github.com/Dicklesworthstone/beads_viewer-pages/commit/84fef2212bd7c24a4b04ba16e333721fa8512326))

---

## [2025-12-18] Initial deploy -- beads_viewer v0.11.1

First deployment of the Beads Viewer static dashboard to GitHub Pages.
Corresponds to source release
[v0.11.1](https://github.com/Dicklesworthstone/beads_viewer/releases/tag/v0.11.1)
(published 2025-12-19).

([`664da72`](https://github.com/Dicklesworthstone/beads_viewer-pages/commit/664da72a00b1582e732ccbf88f2c66a34e5965a8))

### Deployed capabilities

All features below shipped in this single initial commit (28,760 lines across 31 files).

#### Core architecture
- **Client-side SQLite via sql.js WASM** -- the entire issue database (`beads.sqlite3`, 2.3 MB)
  is loaded and queried in the browser using WebAssembly-compiled SQLite.
- **OPFS caching** -- database bytes are cached to the Origin Private File System for
  offline/repeat-visit performance; cache key is hash-based for automatic invalidation.
- **Cross-Origin Isolation service worker** (`coi-serviceworker.js`) -- injects
  `Cross-Origin-Embedder-Policy: credentialless` and `Cross-Origin-Opener-Policy: same-origin`
  headers to enable `SharedArrayBuffer` on GitHub Pages (which does not allow custom headers).
- **Fully self-contained** -- all vendor libraries (Alpine.js, Tailwind CSS, Chart.js, D3 v7,
  force-graph, Mermaid, DOMPurify, marked, sql-wasm) are bundled locally; no CDN calls.
  Content Security Policy enforces `default-src 'self'`.

#### Issue viewer (`viewer.js` -- 3,472 lines)
- **FTS5 full-text search** with BM25 ranking, snippet highlighting, and fallback LIKE queries.
- **Hybrid search scoring** (`hybrid_scorer.js`) -- multi-signal ranking combining text relevance,
  PageRank, status weight, blocker impact, priority, and recency. Five presets: default,
  bug-hunting, sprint-planning, impact-first, text-only.
- **WASM hybrid scorer** (`wasm_loader.js`) -- optional Rust WASM accelerated scoring for
  datasets above 5,000 issues, with automatic JS fallback.
- **Materialized view queries** (`issue_overview_mv`) for fast filtered/sorted issue listing.
- **Filter system** -- status, priority, type, label, has-blockers, assignee; persisted to URL
  query parameters for shareable deep links.
- **Hash-based routing** -- SPA navigation (`/`, `/issues`, `/issue/:id`) with browser
  history integration and URL-encoded filter state.
- **Issue detail modal** -- full issue view with markdown rendering (marked + DOMPurify),
  Mermaid dependency graphs, blocker/dependent lists, and JSON metadata display with
  syntax highlighting.

#### Graph analysis engine
- **WASM graph engine** (`vendor/bv_graph.js` + `bv_graph_bg.wasm`, 214 KB) -- Rust-compiled
  graph algorithms: PageRank, betweenness centrality, eigenvector centrality, k-core
  decomposition, HITS (hub/authority), critical path heights, slack values, articulation
  points, and cycle detection.
- **What-if analysis** -- simulate closing an issue to see cascade impact on the dependency graph;
  `topWhatIf()` ranks all issues by unblocking potential.
- **Actionable issue detection** -- identifies issues whose blockers are all closed.
- **Cycle break suggestions** -- recommends edges to remove for DAG restoration.
- **Top-K set computation** -- finds the minimal set of issues to close for maximum unblocking.

#### Force-graph visualization (`graph.js` -- 3,847 lines)
- **Interactive force-directed graph** powered by `force-graph` (WebGL) + D3 v7 physics.
- **Five view modes**: Force-Directed, Hierarchy (top-down DAG), Radial (from selected node),
  Status Clusters, and Label Galaxy (label-based clustering with convex hulls).
- **Six layout presets**: Force-Directed, Compact DAG, Spread, Orthogonal (grid-snapped),
  Radial, Status Clusters -- each with tuned simulation parameters.
- **Heatmap overlay** -- color/size nodes by PageRank, betweenness, critical path depth, or
  in-degree; togglable via keyboard or UI.
- **Critical path animation** -- animated highlighting of the longest dependency chain.
- **What-if mode** -- shift+click a node to visualize cascade unblocking; affected nodes
  glow and statistics update live.
- **Connected subgraph highlighting** -- click a node for gold-glow BFS expansion of its
  dependency neighborhood.
- **Dependency path tracing** -- ctrl/meta+click to highlight the full dependency path.
- **Cycle navigator** -- step through detected cycles with zoom-to-fit.
- **Node search and filtering** -- real-time search within the graph; filter by status/type/label.
- **Precomputed layout** (`data/graph_layout.json`, 92 KB) -- cached node positions for
  instant initial render of 487 nodes.
- **Time-travel animation** -- replay project history commit-by-commit, watching the graph
  evolve over time from `data/history.json`.
- **Dracula-inspired theme** with colorblind-friendly palette.

#### Charts dashboard (`charts.js` -- 761 lines)
- **Burndown/burnup progress chart** -- tracks issue closure over time.
- **Label dependency heatmap** -- custom canvas-rendered matrix showing cross-label dependencies.
- **Priority distribution pie chart**.
- **Type breakdown bar chart**.
- All charts use the Dracula color palette and are responsive.

#### Keyboard navigation (v0.11.1 feature)
- **Vim-style shortcuts**: `/` to focus search, `?` for help modal, `d` for diagnostics,
  `c` for critical path toggle.
- **Dependency traversal**: `h` navigates to first blocker, `l` to first dependent.
- **Arrow key issue navigation** with wrapping; `o` opens the first issue from list view.

#### Mobile and responsive design
- **Mobile-first CSS** (`styles.css` -- 2,374 lines) with touch-optimized buttons, bottom
  sheet modals, horizontal-scrolling stat cards, and mobile navigation bar.
- **Mobile hamburger menu** and collapsible filter panel.
- **PWA metadata** -- `mobile-web-app-capable`, theme-color, viewport-fit cover.
- **Dark mode** -- default-on, persisted to localStorage, respects `prefers-color-scheme`.

#### Data files
- `beads.sqlite3` (2.3 MB) -- full issue database with 487 issues and FTS5 index.
- `data/meta.json` -- generation timestamp and issue count.
- `data/triage.json` -- robot triage recommendations and quick wins.
- `data/project_health.json` -- velocity metrics, weekly closure rates, graph density.
- `data/history.json` (64 KB) -- commit-level history for time-travel visualization.
- `data/graph_layout.json` (92 KB) -- precomputed force-graph node positions.

#### Bundled vendor libraries
| Library | Purpose |
|---|---|
| `sql-wasm.js` + `sql-wasm.wasm` (640 KB) | Client-side SQLite engine |
| `bv_graph.js` + `bv_graph_bg.wasm` (214 KB) | Rust WASM graph algorithms |
| `force-graph.min.js` v1.43.5 | WebGL force-directed graph renderer |
| `d3.v7.min.js` | Physics simulation and data utilities |
| `chart.umd.min.js` (Chart.js) | Bar/pie/line charts |
| `mermaid.min.js` (3.2 MB) | Dependency diagram rendering |
| `alpine.min.js` + `alpine-collapse.min.js` | Reactive UI framework |
| `tailwindcss.js` | Utility-first CSS (JIT in browser) |
| `marked.min.js` | Markdown-to-HTML conversion |
| `dompurify.min.js` | XSS-safe HTML sanitization |
| `inter-variable.woff2` / `jetbrains-mono-regular.woff2` | Typography |

---

## Repository metadata

- **GitHub Pages URL**: <https://dicklesworthstone.github.io/beads_viewer-pages/>
- **Source repo**: <https://github.com/Dicklesworthstone/beads_viewer>
- **Created**: 2025-12-17
- **License**: MIT with OpenAI/Anthropic Rider
- **Dataset**: 487 issues, 629 dependencies, 0 cycles (100% closed at time of deploy)
