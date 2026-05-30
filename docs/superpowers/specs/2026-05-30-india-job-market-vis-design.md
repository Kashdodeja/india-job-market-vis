# India Job Market Visualizer — SRS + PRD
**Date:** 2026-05-30
**Status:** Approved

---

## 1. Product Requirements Document (PRD)

### 1.1 Overview
An interactive, browser-based treemap visualization of the Indian job market — modeled closely on Andrej Karpathy's US Job Market Visualizer (karpathy.ai/jobs). The primary lens is **Digital AI Exposure**: how vulnerable or enhanced each occupation is by AI/automation. All data sourced from legitimate, citable Indian government and international research sources.

### 1.2 Goals
- Give Indians a visual, intuitive map of the job market shaped around AI disruption risk
- Surface median wages, employment outlook, and education requirements per occupation
- Be fully self-contained: one HTML file, zero server, runs offline or on GitHub Pages
- Every data point must be traceable to a named, credible source

### 1.3 Non-Goals
- Real-time data (data is baked in at build time, refreshed manually per PLFS cycle)
- User accounts, authentication, or backend
- Mobile-first (desktop-primary, responsive is a bonus)
- Job search or application features

### 1.4 Target Users
- Indian students and early-career professionals evaluating career paths
- Researchers and journalists covering India's AI economy
- Policy audiences tracking workforce AI readiness

### 1.5 Success Criteria
- Treemap renders all ~80+ NCO major/minor group occupations with correct worker counts
- AI Exposure score visible per occupation, defensibly sourced
- Layer toggle re-colors treemap in < 300ms
- Tooltip shows all 6 data fields correctly on hover
- Page loads in < 2 seconds on a standard connection
- All data sources cited in footer

---

## 2. Software Requirements Specification (SRS)

### 2.1 Functional Requirements

#### FR-1: Treemap Visualization
- Render a D3.js treemap where rectangle **area** is proportional to number of workers (from PLFS 2022-23)
- Organize occupations at **NCO-2015 Minor Group level** (~144 occupations across 9 Major Groups) — granular enough to be meaningful, coarse enough for PLFS data to be reliable
- Two-level hierarchy visible in the treemap: Major Group (labeled header) → Minor Groups (individual rectangles)
- Display occupation title and worker count (formatted in Lakhs/Crores) inside each rectangle
- Rectangles too small to label should show title on hover only

#### FR-2: Color Layers
Four toggle-able color layers. Active layer colors the entire treemap:

| Layer | Color Scale | Source |
|---|---|---|
| Digital AI Exposure (default) | Red (high) → Green (low), 1–10 | WEF FoJ India 2023 + Oxford/ILO research |
| Median Pay | Dark green (high) → Light (low), INR | PLFS Wage Tables + 6th Economic Census |
| Employment Outlook | Green (+) → Red (−) | ILO India Employment Report 2024 |
| Education Level | Color by tier: No degree / HS / Bach / Masters / PhD | NCO-2015 skill level definitions |

#### FR-3: Header Stats Bar
Displays aggregate statistics that update when layer changes:
- Total workforce (in Crores)
- % of jobs in each education tier
- Average pay by education tier
- Average outlook by education tier
- Count of high AI exposure jobs (score ≥ 7)

#### FR-4: Tooltip on Hover
On hover over any occupation rectangle, show a floating card with:
- Occupation title
- AI Exposure score (x/10) with label (Low / Moderate / High / Critical)
- Worker count (PLFS 2022-23, in Lakhs)
- Median annual wage (INR, formatted with Indian number system)
- Employment Outlook (% change, qualitative label)
- Education requirement
- 2–3 sentence AI narrative (written, not generated at runtime)

#### FR-5: Click-to-Zoom
- Clicking a Major Group rectangle zooms into its sub-occupations
- Breadcrumb trail at top shows current zoom level
- Clicking breadcrumb navigates back up

#### FR-6: Data Sourcing (Offline Pre-processing)
All data compiled into a single `data` JSON object embedded in the HTML file. No runtime API calls. Fields per occupation unit:
```json
{
  "code": "NCO-2015 code",
  "title": "Occupation title",
  "group": "Major group name",
  "workers_lakhs": 42.0,
  "median_wage_inr": 850000,
  "ai_exposure": 8.5,
  "outlook_pct": 12,
  "outlook_label": "Much faster than average",
  "education": "Bachelor's",
  "ai_summary": "...",
  "sources": ["PLFS 2022-23", "WEF FoJ India 2023"]
}
```

#### FR-7: Indian Number Formatting
All numeric outputs use Indian number system (Lakhs, Crores) — not US thousands/millions.

#### FR-8: Footer
Cites all data sources with year: PLFS 2022-23, NCO-2015, WEF Future of Jobs India 2023, ILO India Employment Report 2024, Oxford/ILO automation risk research.

### 2.2 Non-Functional Requirements

| ID | Requirement |
|---|---|
| NFR-1 | Single `index.html` file. External dependency: D3.js v7 via CDN only. |
| NFR-2 | No build tools, no npm, no bundler. Open in browser directly. |
| NFR-3 | Layer color transition ≤ 300ms |
| NFR-4 | Initial page load ≤ 2s on 10 Mbps connection |
| NFR-5 | Tested in Chrome and Firefox |
| NFR-6 | All data defensibly sourced — no estimated/hallucinated numbers |
| NFR-7 | GitHub Pages deployable (static, no server-side code) |

### 2.3 Data Sources

| Source | Data Used | URL / Reference |
|---|---|---|
| PLFS 2022-23 (NSO, MoSPI) | Worker counts by occupation, wages | mospi.gov.in |
| NCO-2015 (MoSPI) | Occupation taxonomy, skill level definitions | mospi.gov.in |
| ILO India Employment Report 2024 | Outlook by sector, youth employment trends | ilo.org |
| WEF Future of Jobs Report 2023 | AI exposure by occupation category | weforum.org |
| Oxford/ILO: "The Risk of Automation for Jobs in BRICS" | Automation risk scores for Indian occupations | academic paper |
| 6th Economic Census (MoSPI) | Supplementary wage data by sector | mospi.gov.in |

### 2.4 AI Exposure Scoring Methodology
- Base scores from Oxford/ILO automation risk research for Indian occupations (Banga & te Velde, 2018; ILO 2023 update)
- Cross-referenced with WEF Future of Jobs India 2023 task-level automation estimates
- Normalized to a 1–10 scale (1 = lowest AI disruption risk, 10 = highest)
- Score assigned at NCO Minor Group level; inherited by Unit Groups within
- Score reflects both **replacement risk** (job automated away) and **augmentation impact** (job fundamentally changed by AI tools)
- All scores documented in a companion `scoring-notes.md` for auditability

---

## 3. Architecture

```
index.html
├── <head>        CDN link: D3.js v7
├── <style>       All CSS inline
└── <script>
    ├── DATA      Embedded JSON (all occupations)
    ├── CONFIG    Layer definitions, color scales
    ├── TREEMAP   D3 treemap layout + render
    ├── TOOLTIP   Hover card logic
    ├── ZOOM      Click-to-zoom + breadcrumb
    ├── LAYERS    Toggle logic + color transitions
    └── STATS     Header stat bar computation
```

No modules. No imports. One `<script>` block, organized by section comments.

---

## 4. Component Breakdown

### 4.1 Header Stats Bar
Mirrors Karpathy's stat panel. Shows aggregate numbers for the active layer. Recalculates on layer switch.

### 4.2 Layer Toggle Controls
4 buttons. Active button highlighted. On click: update color scale, re-color treemap (animated), update stats bar.

### 4.3 Treemap (D3)
- `d3.treemap()` with `d3.hierarchy()` from nested JSON
- `d3.treemapSquarify` tiling algorithm (same as Karpathy's)
- Color mapped via `d3.scaleSequential` per layer
- Text clipped to rectangle bounds with `foreignObject` or SVG `clipPath`

### 4.4 Tooltip
Absolutely positioned `<div>`, hidden by default. On `mouseover`: populate fields + position near cursor. On `mouseout`: hide.

### 4.5 Zoom
On click: filter hierarchy to clicked node, re-run treemap layout on sub-tree. Breadcrumb updates. Back button restores root.

---

## 5. Out of Scope (v1)
- Mobile layout
- Search/filter by occupation name
- Download data as CSV
- State-level breakdown
- Real-time data updates

These are natural v2 features.
