# DataVerse — Pharma Data Capture Architecture

An interactive, single-file visualization that maps how pharmaceutical commercial data is generated across the drug supply and engagement ecosystem, and shows **which datasets capture which interactions** (and where they have blind spots).

It's a teaching / concept tool: pick a dataset and the diagram highlights exactly the flows that dataset "sees." Compare two datasets and it shows the overlap and the gaps.

## What it is

- **A single `index.html`** — no build step, no `package.json`, no server. Open the file in a browser and it runs.
- React 18 + Babel Standalone + Tailwind, all loaded from CDNs at runtime. The app source lives in a `<script type="text/plain" id="appSource">` block and is transformed in-browser with Babel's **classic** JSX runtime (deliberately — the automatic runtime emits `import` statements that break in a plain inline script). A small bootstrap at the bottom does the `Babel.transform(...)` + `eval` and shows a friendly error panel if a CDN fails to load.
- Icons are inline SVG (lucide paths) so there's no icon dependency.

> Because everything is CDN-loaded, the page needs an internet connection to render.

## How to run

Just open [index.html](index.html) in a browser. (Optionally serve the folder with any static server, e.g. `python -m http.server`, but it's not required.)

## What it does

The UI has two stacked "screens":

### Screen 1 — Interactive map (fills the viewport)
- A **node-and-edge diagram** drawn as an SVG. **Nodes** are actors (Manufacturer, Wholesaler, Pharmacy, Clinic, Patient, Payer/PBM, Rep, HCP, etc.). **Edges** are captured interactions or relationships between them (e.g. "Bulk", "Dispensed", "Rx Claim", "Detail", "Chargeback").
- **Dataset buttons** below the map, grouped by category. Interaction model:
  - **Hover** a dataset button → the edges that dataset captures light up in the dataset's type color; everything else fades.
  - **Click** to **pin** a dataset (📌) so it stays highlighted.
  - **Pin one + hover another** → **comparison mode**: edges are colored as *A only* (indigo), *B only* (pink), or *captured by both* (green), with a caption legend.
  - Some datasets declare `missed` edges — drawn as dashed red lines with an ✕ marker to call out **blind spots** (e.g. DDD misses warehouse-to-warehouse transfers).
- Switching tabs resets the page's pinned/hovered selection.

### Screen 2 — Dataset Architecture (scroll down)
A static hierarchy for the active tab: a titled set of **branches** (one per data category), each with chips listing the datasets, an HTML description, and a "notes" callout box covering caveats (projection error, coverage limits, compliance separation, etc.).

## The four tabs (pages)

The whole app is **one component driven by data** — each tab is just a different config object (`VENDOR`, `INTERNAL`, `PRIMARY`, `CHANNEL`) in the `PAGES` array. Each page defines its own `nodes`, `edges`, `datasets`, button groups, legend, and architecture branches.

| # | Tab | Theme | Example datasets |
|---|-----|-------|------------------|
| 1 | **Vendor-Supplied** | Third-party secondary data bought from data vendors | Xponent, PlanTrak, NPA, DDD, NSP, LRx, Dx, APLD, MMIT, Formulary |
| 2 | **Internal** | The brand's own systems (CRM, ops, master data) | Calls, Samples, Digital, Speaker, MSL, Alignment, Targeting, Affiliation, IDN Hierarchy |
| 3 | **Primary** | Research commissioned directly with stakeholders | ATU, Message Testing, Promo Response, Patient Survey, PRO/QoL, Patient Journey, Payer Survey, Access Research, Advisory Boards, Chart Audit |
| 4 | **Channel** | Direct trade feeds — wholesaler EDI + specialty pharmacy | 852 Inventory, 867 Resale, 844 Chargeback, DDD, NSP, Specialty 867, SP Dispense, Hub/Status |

## Code structure (all inside `index.html`)

The app source (the `appSource` script) is organized top-to-bottom as:

1. **Inline SVG icon components** (`Factory`, `Warehouse`, `Store`, `Stethoscope`, … built on a shared `Icon`).
2. **Shared geometry & palettes** — `VIEW_W`/`VIEW_H` (1000×500 SVG viewBox), arrow-marker colors, the comparison palette `CMP`, and the Tailwind class maps `ACCENTS` / `BTN` / `TONE`.
3. **Page definitions** — the four config objects (`VENDOR`, `INTERNAL`, `PRIMARY`, `CHANNEL`) collected into `PAGES`. This is where the actual content lives; editing these changes the diagrams.
4. **`App`** — holds state (`pageId`, `pinned`, `hovered`), computes `activeId` / `compare`, and the `edgeStyle(id)` function that decides each edge's color/width/visibility. Renders the sticky navbar, the SVG map (edges are sorted so highlighted lines paint on top), the node overlay, the comparison caption, the dataset buttons, and Screen 2.
5. **Presentational helpers** — `LegendItem`, `Branch`, `Chips`, `Note`.

### Editing notes
- **To change what a dataset captures**, edit its `edges` (and optional `missed`) array under that page's `datasets`.
- **To move a node**, change its `x`/`y` (in the 1000×500 coordinate space); nodes are positioned by percentage so they stay aligned with the SVG edges.
- **Edge geometry** is a raw SVG path `d`, with `mx`/`my` giving the label/marker position. `selfLoop: true` marks the curved self-referencing edges (e.g. W2W transfer, IDN).
- `desc` and note `items` are rendered with `dangerouslySetInnerHTML`, so they accept inline HTML entities/tags.

## Status

Illustrative concept (the footer says as much): **node = actor, edge = a captured interaction or relationship.** It's a presentation/education artifact, not wired to any live data source.
