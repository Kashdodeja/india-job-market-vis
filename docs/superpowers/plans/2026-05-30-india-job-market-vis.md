# India Job Market Visualizer — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a static single-file `index.html` India job market treemap visualizer modeled on Karpathy's US Job Market Visualizer, with Digital AI Exposure as the primary color layer, powered by legitimate Indian government and international research data.

**Architecture:** A single `index.html` with all CSS and JS inline. D3.js v7 from CDN. The entire dataset is a pre-compiled JSON object embedded in the script block. No build step, no npm, no server — opens directly in any browser and deploys to GitHub Pages as-is.

**Tech Stack:** HTML5, CSS3, Vanilla JS (ES6+), D3.js v7 (CDN).

---

## File Structure

```
index.html              — entire application (HTML + CSS + JS + data)
scoring-notes.md        — AI exposure score sources + methodology per occupation
docs/superpowers/specs/ — design spec (already written)
docs/superpowers/plans/ — this file
README.md               — GitHub Pages deploy instructions
```

`index.html` internal script sections (in order):
1. `// DATA` — full occupation JSON blob
2. `// CONFIG` — layer definitions, color scales config
3. `// UTILS` — formatIndian(), getExposureLabel(), formatWage()
4. `// STATS` — computeStats() for header bar
5. `// TOOLTIP` — showTooltip(), hideTooltip()
6. `// TREEMAP` — renderTreemap(), drawLabels()
7. `// ZOOM` — zoomTo(), zoomOut(), updateBreadcrumb()
8. `// LAYERS` — setActiveLayer(), applyColors()
9. `// INIT` — main(), event listeners

---

## Task 1: Research and compile NCO-2015 occupation hierarchy

**Files:**
- Create: `scoring-notes.md`

The NCO-2015 (National Classification of Occupations) defines India's occupation taxonomy. You need the Sub-Major Group level (43 groups) as the primary visualization unit.

- [ ] **Step 1: Download NCO-2015 document**

  Go to: https://mospi.gov.in/documents/213904/301737/NCO_2015.pdf
  Download the PDF. Open Table of Contents. You need the Sub-Major Group codes (2-digit codes) and their titles.

- [ ] **Step 2: Extract all Sub-Major Groups**

  From NCO-2015, extract every Sub-Major Group (2-digit code) grouped by its Major Group (1-digit). Record in `scoring-notes.md` using this format:

  ```markdown
  ## NCO-2015 Sub-Major Groups

  ### Major Group 1: Managers
  - 11: Chief Executives, Senior Officials and Legislators
  - 12: Administrative and Commercial Managers
  - 13: Production and Specialised Services Managers
  - 14: Hospitality, Retail and Other Services Managers

  ### Major Group 2: Professionals
  - 21: Science and Engineering Professionals
  - 22: Health Professionals
  - 23: Teaching Professionals
  - 24: Business and Administration Professionals
  - 25: Information and Communications Technology Professionals
  - 26: Legal, Social and Cultural Professionals

  [... continue for all 9 major groups ...]
  ```

- [ ] **Step 3: Note the Skill Level for each Sub-Major Group**

  NCO-2015 assigns a Skill Level (1–4) to each occupation group. Record these — they map to education requirements:
  - Skill Level 1 → "No formal education / Primary"
  - Skill Level 2 → "HS Diploma / Vocational"
  - Skill Level 3 → "Bachelor's / Diploma"
  - Skill Level 4 → "Master's / Doctoral"

- [ ] **Step 4: Commit**

  ```bash
  git add scoring-notes.md
  git commit -m "data: add NCO-2015 occupation hierarchy"
  ```

---

## Task 2: Research PLFS 2022-23 worker counts and wages

**Files:**
- Modify: `scoring-notes.md`

PLFS (Periodic Labour Force Survey) 2022-23 Annual Report is the primary source for worker counts and wages. Published by NSO/MoSPI.

- [ ] **Step 1: Download PLFS 2022-23 Annual Report**

  URL: https://mospi.gov.in/documents/213904/0/PLFS+Annual+Report+2022-23.pdf
  Alternate: Search "PLFS Annual Report 2022-23 PDF" on mospi.gov.in

  Key tables to find:
  - **Table A10 or equivalent**: Usual Principal + Subsidiary Status (UPSS) workers by 1-digit NCO code → gives Major Group counts
  - **Statement 14 or Appendix**: Workers by 2-digit NCO → Sub-Major Group counts
  - **Wage table**: Average weekly/daily wages by occupation group

- [ ] **Step 2: Extract worker counts**

  Record counts (in Lakhs, i.e. hundreds of thousands) for each Sub-Major Group. If Sub-Major Group data is unavailable, use Major Group totals and distribute proportionally using NCO-2015 size guidance.

  Use this format in `scoring-notes.md`:

  ```markdown
  ## Worker Counts (PLFS 2022-23, UPSS, in Lakhs)

  | NCO Code | Sub-Major Group | Workers (Lakhs) | Source Table |
  |---|---|---|---|
  | 11 | Chief Executives, Senior Officials | 28 | Table A10 |
  | 12 | Administrative and Commercial Managers | 45 | Table A10 |
  | 21 | Science and Engineering Professionals | 65 | Table A10 |
  | 25 | ICT Professionals | 42 | Table A10 |
  | ... | ... | ... | ... |
  ```

  Reference totals (from PLFS 2022-23 headline numbers — verify against the report):
  - Total employed workforce: ~563 million (5630 Lakhs)
  - Major Group 1 (Managers): ~85 Lakhs
  - Major Group 2 (Professionals): ~220 Lakhs
  - Major Group 3 (Technicians): ~150 Lakhs
  - Major Group 4 (Clerical): ~250 Lakhs
  - Major Group 5 (Services/Sales): ~850 Lakhs
  - Major Group 6 (Agriculture): ~1150 Lakhs
  - Major Group 7 (Craft/Trades): ~700 Lakhs
  - Major Group 8 (Plant/Machine): ~500 Lakhs
  - Major Group 9 (Elementary): ~1200 Lakhs

  These are starting-point estimates — use the actual PLFS table values.

- [ ] **Step 3: Extract median annual wages (INR)**

  PLFS reports weekly wages. Convert: `annual = weekly_wage × 52`.
  Record as `median_wage_inr` per Sub-Major Group.

  Reference points to verify against:
  - ICT Professionals (NCO 25): ~₹8–12L/year
  - Teaching Professionals (NCO 23): ~₹3–5L/year
  - Elementary workers (NCO 9x): ~₹0.8–1.2L/year
  - Agricultural workers (NCO 6x): ~₹0.6–1L/year

- [ ] **Step 4: Commit**

  ```bash
  git add scoring-notes.md
  git commit -m "data: add PLFS 2022-23 worker counts and wages"
  ```

---

## Task 3: Research and compute AI exposure scores

**Files:**
- Modify: `scoring-notes.md`

Score each Sub-Major Group on AI exposure from 1–10 (10 = highest disruption risk) using published academic research. DO NOT invent scores.

- [ ] **Step 1: Read primary sources**

  Read and take notes from:

  1. **ILO 2023**: "Generative AI and Jobs: A global analysis of potential effects on job quantity and quality" — https://www.ilo.org/wcmsp5/groups/public/---dgreports/---inst/documents/publication/wcms_890761.pdf
     Key tables: automation exposure by occupation for developing countries including India

  2. **Oxford/ILO (Banga & te Velde, 2018)**: "Digitalisation and the Future of Manufacturing in Africa" — contains automation risk scores for developing economy occupations. Search: "Banga te Velde 2018 automation risk BRICS"

  3. **WEF Future of Jobs Report 2023** — https://www3.weforum.org/docs/WEF_Future_of_Jobs_2023.pdf
     Table of Contents → look for "Job displacement" and "Task automation" by occupation

  4. **McKinsey Global Institute 2023 India**: Search "McKinsey generative AI India jobs 2023 report"

- [ ] **Step 2: Assign scores**

  For each Sub-Major Group, assign a score from 1–10 based on the sources. Score reflects BOTH replacement risk AND augmentation disruption. Add to `scoring-notes.md`:

  ```markdown
  ## AI Exposure Scores (1=lowest risk, 10=highest)

  | NCO Code | Sub-Major Group | Score | Primary Basis | Notes |
  |---|---|---|---|---|
  | 11 | Chief Executives | 5.5 | WEF FoJ 2023, ILO 2023 | Decision-making tasks hard to automate; admin tasks high exposure |
  | 12 | Admin/Commercial Managers | 6.5 | ILO 2023 | Routine management tasks increasingly AI-handled |
  | 21 | Science/Engineering Professionals | 6.0 | WEF FoJ 2023 | Code gen, simulation tools augmenting heavily |
  | 22 | Health Professionals | 3.5 | ILO 2023 | Physical exam, empathy barriers; diagnosis augmented |
  | 23 | Teaching Professionals | 4.0 | WEF FoJ 2023 | Personalized learning AI rising; human element preserved |
  | 24 | Business/Admin Professionals | 7.5 | ILO 2023 | Financial analysis, legal drafting, HR highly exposed |
  | 25 | ICT Professionals | 8.5 | WEF FoJ 2023 | Code generation, QA automation core use cases for LLMs |
  | 26 | Legal/Social/Cultural Professionals | 5.0 | ILO 2023 | Legal drafting exposed; social work, culture preserved |
  | 31 | Science/Engineering Technicians | 6.5 | ILO 2023 | |
  | 32 | Health Associate Professionals | 4.0 | ILO 2023 | |
  | 33 | Business/Admin Associate Prof | 7.0 | ILO 2023 | |
  | 34 | Legal/Social/Cultural Assoc Prof | 5.5 | ILO 2023 | |
  | 35 | ICT Technicians | 7.0 | WEF FoJ 2023 | |
  | 41 | General/Keyboard Clerks | 9.0 | ILO 2023 | Data entry, typing — highest LLM displacement risk |
  | 42 | Customer Service Clerks | 8.5 | ILO 2023 | Call center, chat support — highly exposed |
  | 43 | Numerical/Material Recording Clerks | 8.5 | ILO 2023 | |
  | 44 | Other Clerical Workers | 8.0 | ILO 2023 | |
  | 51 | Personal Service Workers | 3.5 | ILO 2023 | Physical service, human contact barrier |
  | 52 | Sales Workers | 5.5 | WEF FoJ 2023 | E-commerce AI rising; physical retail more resilient |
  | 53 | Personal Care Workers | 2.5 | ILO 2023 | Elderly care, childcare — physical + empathy barrier |
  | 54 | Protective Services Workers | 3.0 | ILO 2023 | |
  | 61 | Market-Oriented Skilled Agri | 2.0 | Oxford/ILO | Physical outdoor work, low digitization |
  | 62 | Market-Oriented Skilled Forestry | 2.0 | Oxford/ILO | |
  | 63 | Subsistence Farmers | 1.5 | Oxford/ILO | Extremely low digital exposure |
  | 71 | Building/Related Trades | 2.5 | ILO 2023 | |
  | 72 | Metal/Machinery Related Trades | 3.5 | ILO 2023 | CNC/robot integration growing |
  | 73 | Handicraft/Printing Trades | 4.0 | ILO 2023 | |
  | 74 | Electrical/Electronic Trades | 5.0 | ILO 2023 | |
  | 75 | Food Processing/Wood/Garment | 3.0 | ILO 2023 | |
  | 81 | Stationary Plant/Machine Operators | 4.5 | ILO 2023 | |
  | 82 | Assemblers | 5.5 | ILO 2023 | Manufacturing robots |
  | 83 | Drivers/Mobile Plant Operators | 4.0 | Oxford/ILO | Autonomous vehicles long horizon in India |
  | 91 | Cleaners and Helpers | 2.0 | ILO 2023 | |
  | 92 | Agricultural/Forestry Laborers | 2.0 | Oxford/ILO | |
  | 93 | Laborers in Mining/Construction | 2.5 | ILO 2023 | |
  | 94 | Food Prep Assistants | 2.5 | ILO 2023 | |
  | 95 | Street/Related Sales Workers | 3.5 | ILO 2023 | |
  | 96 | Refuse Workers/Other Elementary | 1.5 | ILO 2023 | |
  ```

  Refine these scores based on what you actually read in the sources. Do not use scores you cannot defend with a citation.

- [ ] **Step 3: Write AI narrative for each Sub-Major Group**

  Add a 2–3 sentence `ai_summary` for each group to `scoring-notes.md`. Example:

  ```
  NCO 25 (ICT Professionals):
  "This occupation is fundamentally digital, with code generation and test automation being 
  primary AI use cases. While system architecture and complex stakeholder communication provide 
  some insulation, AI is drastically changing entry-level roles. Augmentation rather than full 
  replacement is the near-term trajectory, but productivity per engineer will surge."
  ```

- [ ] **Step 4: Commit**

  ```bash
  git add scoring-notes.md
  git commit -m "data: add AI exposure scores with source citations"
  ```

---

## Task 4: Research employment outlook

**Files:**
- Modify: `scoring-notes.md`

- [ ] **Step 1: Read ILO India Employment Report 2024**

  URL: https://www.ilo.org/publications/india-employment-report-2024
  Focus on: sector-wise employment growth projections, occupation outlook by skill level

  Also reference:
  - India Skills Report 2024 (Wheebox) — sector demand growth
  - NITI Aayog "Jobs and Growth" reports

- [ ] **Step 2: Assign outlook values**

  Express as percentage change in employment over next 5 years + a label. Add to `scoring-notes.md`:

  ```markdown
  ## Employment Outlook (5-year projection, %)

  | NCO Code | Outlook % | Label |
  |---|---|---|
  | 25 | +22 | Much faster than average |
  | 21 | +15 | Faster than average |
  | 23 | +8 | About average |
  | 41 | -10 | Declining |
  | 63 | -8 | Declining |
  | 51 | +5 | About average |
  [... all Sub-Major Groups ...]

  Labels:
  - ≥ +20%: "Much faster than average"
  - +10 to +19%: "Faster than average"
  - +1 to +9%: "About average"
  - -9 to 0%: "Slower than average"
  - ≤ -10%: "Declining"
  ```

- [ ] **Step 3: Commit**

  ```bash
  git add scoring-notes.md
  git commit -m "data: add employment outlook by occupation"
  ```

---

## Task 5: Compile master dataset into index.html

**Files:**
- Create: `index.html`

Combine all researched data into a single JSON blob that will be the DATA section of `index.html`.

- [ ] **Step 1: Create index.html with the data skeleton**

  Create `index.html` with this exact structure. The `JOBS_DATA` object below shows the schema — replace the placeholder entries with your researched data for ALL 43 Sub-Major Groups:

  ```html
  <!DOCTYPE html>
  <html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>India Job Market Visualizer</title>
    <script src="https://d3js.org/d3.v7.min.js"></script>
    <style>
    /* CSS added in Task 7 */
    </style>
  </head>
  <body>
  <script>
  // ============================================================
  // DATA
  // ============================================================
  const JOBS_DATA = {
    name: "India Job Market",
    children: [
      {
        name: "Managers",
        code: "1",
        children: [
          {
            code: "11",
            title: "Chief Executives, Senior Officials and Legislators",
            group: "Managers",
            workers_lakhs: 28,
            median_wage_inr: 1800000,
            ai_exposure: 5.5,
            outlook_pct: 6,
            outlook_label: "About average",
            education: "Master's",
            ai_summary: "Strategic decision-making remains hard to automate, but AI tools are transforming information synthesis and planning for this group. Administrative and reporting functions are increasingly AI-assisted. The net effect is augmentation, not replacement, at this level."
          },
          {
            code: "12",
            title: "Administrative and Commercial Managers",
            group: "Managers",
            workers_lakhs: 45,
            median_wage_inr: 900000,
            ai_exposure: 6.5,
            outlook_pct: 5,
            outlook_label: "About average",
            education: "Bachelor's",
            ai_summary: "Routine managerial tasks—scheduling, reporting, budgeting—are highly exposed to AI automation. Human judgment for people management and stakeholder relationships provides insulation. Mid-level management is the most vulnerable layer as AI compresses reporting chains."
          },
          {
            code: "13",
            title: "Production and Specialised Services Managers",
            group: "Managers",
            workers_lakhs: 32,
            median_wage_inr: 1000000,
            ai_exposure: 6.0,
            outlook_pct: 8,
            outlook_label: "About average",
            education: "Bachelor's",
            ai_summary: "Production optimization, quality monitoring, and supply chain decisions are increasingly AI-supported. Physical oversight of manufacturing lines provides a floor. AI tools will augment rather than replace, but raise output expectations per manager significantly."
          },
          {
            code: "14",
            title: "Hospitality, Retail and Other Services Managers",
            group: "Managers",
            workers_lakhs: 38,
            median_wage_inr: 600000,
            ai_exposure: 5.0,
            outlook_pct: 10,
            outlook_label: "Faster than average",
            education: "HS Diploma",
            ai_summary: "Customer-facing service management involves significant human judgment and interpersonal work that resists automation. However, AI-driven inventory, pricing, and demand forecasting tools are changing the skill set required at this level."
          }
        ]
      },
      {
        name: "Professionals",
        code: "2",
        children: [
          {
            code: "21",
            title: "Science and Engineering Professionals",
            group: "Professionals",
            workers_lakhs: 65,
            median_wage_inr: 900000,
            ai_exposure: 6.0,
            outlook_pct: 15,
            outlook_label: "Faster than average",
            education: "Bachelor's",
            ai_summary: "AI-assisted simulation, design, and analysis tools are rapidly augmenting engineering workflows. Code generation and scientific literature synthesis are primary AI impact areas. Demand for engineers who can leverage AI tools effectively is growing faster than overall headcount."
          },
          {
            code: "22",
            title: "Health Professionals",
            group: "Professionals",
            workers_lakhs: 55,
            median_wage_inr: 800000,
            ai_exposure: 3.5,
            outlook_pct: 18,
            outlook_label: "Much faster than average",
            education: "Bachelor's",
            ai_summary: "Physical examination, surgical skill, and patient empathy represent strong barriers to direct replacement. AI is transforming diagnostic imaging, drug discovery, and clinical documentation as augmentation tools. India's healthcare shortage means demand growth far outpaces any displacement risk."
          },
          {
            code: "23",
            title: "Teaching Professionals",
            group: "Professionals",
            workers_lakhs: 120,
            median_wage_inr: 450000,
            ai_exposure: 4.0,
            outlook_pct: 8,
            outlook_label: "About average",
            education: "Bachelor's",
            ai_summary: "Personalized AI tutoring tools are reshaping how students learn, but the mentorship, socialization, and classroom management roles of teachers remain human-critical. Standardized test preparation and content delivery face the highest disruption, while early childhood and special education are most insulated."
          },
          {
            code: "24",
            title: "Business and Administration Professionals",
            group: "Professionals",
            workers_lakhs: 75,
            median_wage_inr: 750000,
            ai_exposure: 7.5,
            outlook_pct: 5,
            outlook_label: "About average",
            education: "Bachelor's",
            ai_summary: "Financial analysis, legal document drafting, HR screening, and accounting are among the highest-exposure white-collar tasks for large language models. India's large back-office and BPO sector faces structural disruption. Roles requiring client relationships and judgment are more resilient."
          },
          {
            code: "25",
            title: "Information and Communications Technology Professionals",
            group: "Professionals",
            workers_lakhs: 50,
            median_wage_inr: 1200000,
            ai_exposure: 8.5,
            outlook_pct: 22,
            outlook_label: "Much faster than average",
            education: "Bachelor's",
            ai_summary: "Code generation, test automation, and documentation are primary LLM use cases, making this one of the highest-exposure occupations. Entry-level coding and QA roles face significant displacement pressure. However, AI systems require skilled engineers to build, deploy, and maintain them — net demand is rising even as per-task productivity soars."
          },
          {
            code: "26",
            title: "Legal, Social and Cultural Professionals",
            group: "Professionals",
            workers_lakhs: 40,
            median_wage_inr: 600000,
            ai_exposure: 5.0,
            outlook_pct: 7,
            outlook_label: "About average",
            education: "Bachelor's",
            ai_summary: "Legal research and document drafting face high AI exposure, while courtroom advocacy, social work, and creative cultural roles are more insulated. Journalists, artists, and social workers face disruption in research and production tasks while human judgment remains essential in practice."
          }
        ]
      },
      {
        name: "Technicians",
        code: "3",
        children: [
          {
            code: "31",
            title: "Science and Engineering Technicians",
            group: "Technicians",
            workers_lakhs: 60,
            median_wage_inr: 350000,
            ai_exposure: 6.5,
            outlook_pct: 12,
            outlook_label: "Faster than average",
            education: "HS Diploma",
            ai_summary: "Technical support, testing, and measurement roles are increasingly automated. AI-driven quality control and predictive maintenance tools are replacing routine monitoring tasks. Growing demand in renewable energy, electronics, and manufacturing provides offsetting growth."
          },
          {
            code: "32",
            title: "Health Associate Professionals",
            group: "Technicians",
            workers_lakhs: 45,
            median_wage_inr: 300000,
            ai_exposure: 4.0,
            outlook_pct: 20,
            outlook_label: "Much faster than average",
            education: "HS Diploma",
            ai_summary: "India's severe healthcare worker shortage means demand significantly outpaces any AI displacement. Nursing, paramedic, and laboratory technician roles require physical presence and patient interaction that AI cannot substitute. AI assists with records and diagnostics, augmenting capacity."
          },
          {
            code: "33",
            title: "Business and Administration Associate Professionals",
            group: "Technicians",
            workers_lakhs: 85,
            median_wage_inr: 400000,
            ai_exposure: 7.0,
            outlook_pct: 2,
            outlook_label: "Slower than average",
            education: "HS Diploma",
            ai_summary: "India's large BPO and back-office workforce in this category faces the most direct AI displacement pressure. Customer support automation, financial processing, and data management are high-exposure tasks. Growth in high-complexity work partially offsets displacement in routine roles."
          },
          {
            code: "34",
            title: "Legal, Social, Cultural and Related Associate Professionals",
            group: "Technicians",
            workers_lakhs: 30,
            median_wage_inr: 350000,
            ai_exposure: 5.5,
            outlook_pct: 6,
            outlook_label: "About average",
            education: "HS Diploma",
            ai_summary: "Paralegals, social workers, and cultural production assistants face mixed AI exposure. Research and drafting tasks are highly exposed; direct community engagement and creative interpretation are insulated. Growing NGO and social enterprise sector offsets some formal sector displacement."
          },
          {
            code: "35",
            title: "Information and Communications Technology Technicians",
            group: "Technicians",
            workers_lakhs: 40,
            median_wage_inr: 500000,
            ai_exposure: 7.0,
            outlook_pct: 14,
            outlook_label: "Faster than average",
            education: "HS Diploma",
            ai_summary: "IT support, network administration, and systems monitoring are undergoing significant automation. AI-driven ticketing, self-healing infrastructure, and predictive alerts are reducing demand for routine Level 1-2 support. Demand grows at the high end for AI infrastructure and cloud skills."
          }
        ]
      },
      {
        name: "Clerical Workers",
        code: "4",
        children: [
          {
            code: "41",
            title: "General and Keyboard Clerks",
            group: "Clerical Workers",
            workers_lakhs: 90,
            median_wage_inr: 220000,
            ai_exposure: 9.0,
            outlook_pct: -12,
            outlook_label: "Declining",
            education: "HS Diploma",
            ai_summary: "Data entry, transcription, and general keyboard-based clerical work represents the single highest AI displacement risk in the Indian job market. Large language models and OCR systems can replace most of this output at a fraction of the cost. This is the core of India's at-risk white-collar workforce."
          },
          {
            code: "42",
            title: "Customer Service Clerks",
            group: "Clerical Workers",
            workers_lakhs: 120,
            median_wage_inr: 240000,
            ai_exposure: 8.5,
            outlook_pct: -8,
            outlook_label: "Declining",
            education: "HS Diploma",
            ai_summary: "India's massive call center industry — one of the largest globally — faces direct structural displacement from conversational AI. Voice and chat AI agents already handle routine queries at lower cost. Human agents increasingly handle escalations only, compressing the total headcount required significantly."
          },
          {
            code: "43",
            title: "Numerical and Material Recording Clerks",
            group: "Clerical Workers",
            workers_lakhs: 70,
            median_wage_inr: 230000,
            ai_exposure: 8.5,
            outlook_pct: -6,
            outlook_label: "Slower than average",
            education: "HS Diploma",
            ai_summary: "Inventory clerks, stock control, accounts payable, and logistics documentation roles are highly exposed to AI and ERP automation. Modern supply chain AI systems handle end-to-end tracking that previously required large clerical teams. Government and unorganized sector slowdown this displacement curve."
          },
          {
            code: "44",
            title: "Other Clerical Support Workers",
            group: "Clerical Workers",
            workers_lakhs: 50,
            median_wage_inr: 210000,
            ai_exposure: 8.0,
            outlook_pct: -5,
            outlook_label: "Slower than average",
            education: "HS Diploma",
            ai_summary: "File clerks, mail handlers, and general administrative support face consistent AI exposure as document management, routing, and scheduling become increasingly automated. Government employment partially shields this category from private-sector displacement timelines."
          }
        ]
      },
      {
        name: "Services and Sales",
        code: "5",
        children: [
          {
            code: "51",
            title: "Personal Service Workers",
            group: "Services and Sales",
            workers_lakhs: 180,
            median_wage_inr: 180000,
            ai_exposure: 3.5,
            outlook_pct: 10,
            outlook_label: "Faster than average",
            education: "No formal education",
            ai_summary: "Hospitality, beauty, and domestic service workers are insulated by the physical and interpersonal nature of their work. AI tools are creating some demand compression in standardized settings (hotel chains, franchise outlets) but the informal domestic service sector is largely unaffected for the foreseeable future."
          },
          {
            code: "52",
            title: "Sales Workers",
            group: "Services and Sales",
            workers_lakhs: 250,
            median_wage_inr: 200000,
            ai_exposure: 5.5,
            outlook_pct: 5,
            outlook_label: "About average",
            education: "No formal education",
            ai_summary: "E-commerce AI is transforming retail and displacing physical storefronts. AI-driven recommendations, pricing, and customer journeys reduce the need for in-store sales staff. Street and market vendors face less direct impact. B2B enterprise sales, requiring relationship-building, sees AI as augmentation."
          },
          {
            code: "53",
            title: "Personal Care Workers",
            group: "Services and Sales",
            workers_lakhs: 100,
            median_wage_inr: 150000,
            ai_exposure: 2.5,
            outlook_pct: 22,
            outlook_label: "Much faster than average",
            education: "No formal education",
            ai_summary: "Elderly care, childcare, and health aide roles require physical presence, empathy, and trust — barriers AI cannot overcome. India's demographic shift toward an aging population and rising middle-class demand for professional childcare are creating structural tailwinds for this sector."
          },
          {
            code: "54",
            title: "Protective Services Workers",
            group: "Services and Sales",
            workers_lakhs: 80,
            median_wage_inr: 220000,
            ai_exposure: 3.0,
            outlook_pct: 8,
            outlook_label: "About average",
            education: "HS Diploma",
            ai_summary: "Security personnel, police, and fire services require physical presence and legal authority that AI cannot hold. AI surveillance tools augment security capacity rather than replace it. Growing urbanization and commercial activity drives steady demand."
          }
        ]
      },
      {
        name: "Skilled Agriculture",
        code: "6",
        children: [
          {
            code: "61",
            title: "Market-Oriented Skilled Agricultural Workers",
            group: "Skilled Agriculture",
            workers_lakhs: 600,
            median_wage_inr: 90000,
            ai_exposure: 2.0,
            outlook_pct: -5,
            outlook_label: "Slower than average",
            education: "No formal education",
            ai_summary: "Crop cultivation, livestock management, and fishery work are deeply physical and geographically distributed — barriers to AI automation. Precision agriculture AI tools assist with irrigation, pest detection, and yield optimization but require capital investment inaccessible to most small farmers. Long-term structural migration to cities, not AI, is the primary employment pressure."
          },
          {
            code: "62",
            title: "Market-Oriented Skilled Forestry, Fishery and Hunting Workers",
            group: "Skilled Agriculture",
            workers_lakhs: 80,
            median_wage_inr: 85000,
            ai_exposure: 2.0,
            outlook_pct: -3,
            outlook_label: "Slower than average",
            education: "No formal education",
            ai_summary: "Forestry and fishery work involves outdoor physical tasks with high variability that AI cannot easily manage. Environmental degradation and resource depletion pose greater structural risks than automation. Aquaculture technology is growing but adoption remains slow."
          },
          {
            code: "63",
            title: "Subsistence Farmers, Fishers, Hunters and Gatherers",
            group: "Skilled Agriculture",
            workers_lakhs: 200,
            median_wage_inr: 60000,
            ai_exposure: 1.5,
            outlook_pct: -8,
            outlook_label: "Declining",
            education: "No formal education",
            ai_summary: "Subsistence farmers operate largely outside the formal economy and face near-zero direct AI exposure. Long-term decline is driven by rural-to-urban migration and climate stress. Government MGNREGA and agricultural support schemes slow the structural transition. This segment represents India's largest at-risk population for income disruption — though from structural, not technological, causes."
          }
        ]
      },
      {
        name: "Craft and Trades",
        code: "7",
        children: [
          {
            code: "71",
            title: "Building and Related Trades Workers",
            group: "Craft and Trades",
            workers_lakhs: 200,
            median_wage_inr: 160000,
            ai_exposure: 2.5,
            outlook_pct: 12,
            outlook_label: "Faster than average",
            education: "No formal education",
            ai_summary: "Construction work is physically intensive and context-dependent, making AI automation challenging in India's diverse building environments. Infrastructure spending under government programs like PM Gati Shakti drives strong demand growth. AI-powered design and project management tools augment supervisors but don't displace laborers."
          },
          {
            code: "72",
            title: "Metal, Machinery and Related Trades Workers",
            group: "Craft and Trades",
            workers_lakhs: 150,
            median_wage_inr: 200000,
            ai_exposure: 3.5,
            outlook_pct: 8,
            outlook_label: "About average",
            education: "HS Diploma",
            ai_summary: "CNC machines and industrial robots are changing metal fabrication workflows, but skilled trades workers who can set up, operate, and maintain automated machinery are in growing demand. India's manufacturing push (PLI schemes) creates strong tailwinds even as automation increases per-worker output."
          },
          {
            code: "73",
            title: "Handicraft and Printing Workers",
            group: "Craft and Trades",
            workers_lakhs: 80,
            median_wage_inr: 140000,
            ai_exposure: 4.0,
            outlook_pct: -2,
            outlook_label: "Slower than average",
            education: "No formal education",
            ai_summary: "Digital printing and AI-generated design are displacing traditional graphic arts workers. Handcraft workers benefit from artisanal premiums in domestic and export markets, but generative AI for textile patterns and print design creates pricing pressure. GI-tagged crafts are partially insulated."
          },
          {
            code: "74",
            title: "Electrical and Electronic Trades Workers",
            group: "Craft and Trades",
            workers_lakhs: 90,
            median_wage_inr: 250000,
            ai_exposure: 5.0,
            outlook_pct: 16,
            outlook_label: "Faster than average",
            education: "HS Diploma",
            ai_summary: "Growing electronics manufacturing (mobile phones, semiconductors) and renewable energy installation (solar panels, EV charging) are creating strong demand. AI diagnostic tools are changing how faults are found and fixed but cannot perform the physical repair work. Net employment trend is strongly positive."
          },
          {
            code: "75",
            title: "Food Processing, Wood, Garment and Other Craft Workers",
            group: "Craft and Trades",
            workers_lakhs: 180,
            median_wage_inr: 130000,
            ai_exposure: 3.0,
            outlook_pct: 5,
            outlook_label: "About average",
            education: "No formal education",
            ai_summary: "Food processing and garment work are being mechanized at the factory level, but India's large informal sector is largely untouched by automation. Export garment factories face automation pressure in cutting and stitching. Traditional food production and wood crafts remain predominantly manual."
          }
        ]
      },
      {
        name: "Plant and Machine Operators",
        code: "8",
        children: [
          {
            code: "81",
            title: "Stationary Plant and Machine Operators",
            group: "Plant and Machine Operators",
            workers_lakhs: 120,
            median_wage_inr: 200000,
            ai_exposure: 4.5,
            outlook_pct: 6,
            outlook_label: "About average",
            education: "HS Diploma",
            ai_summary: "Factory automation and process control AI are reducing demand for routine machine operation, but skilled operators who can manage and troubleshoot AI-controlled equipment are increasingly valued. PLI scheme manufacturing growth is creating net positive demand in electronics and pharma sectors."
          },
          {
            code: "82",
            title: "Assemblers",
            group: "Plant and Machine Operators",
            workers_lakhs: 100,
            median_wage_inr: 175000,
            ai_exposure: 5.5,
            outlook_pct: 10,
            outlook_label: "Faster than average",
            education: "No formal education",
            ai_summary: "Robotic assembly is advancing rapidly in automotive and electronics, but India's labor cost advantage and complex product mix slow automation adoption compared to China or South Korea. iPhone and semiconductor assembly for India's export ambitions is driving demand growth that outpaces displacement in the near term."
          },
          {
            code: "83",
            title: "Drivers and Mobile Plant Operators",
            group: "Plant and Machine Operators",
            workers_lakhs: 280,
            median_wage_inr: 210000,
            ai_exposure: 4.0,
            outlook_pct: 8,
            outlook_label: "About average",
            education: "No formal education",
            ai_summary: "Autonomous vehicles represent the long-term AI threat to India's large driving workforce, but adoption timelines on Indian roads — with mixed traffic, pedestrian density, and infrastructure variability — are measured in decades. Platform gig economy (Ola, Uber, Rapido, Porter) continues to grow total demand for drivers despite per-trip optimization."
          }
        ]
      },
      {
        name: "Elementary Occupations",
        code: "9",
        children: [
          {
            code: "91",
            title: "Cleaners and Helpers",
            group: "Elementary Occupations",
            workers_lakhs: 280,
            median_wage_inr: 120000,
            ai_exposure: 2.0,
            outlook_pct: 5,
            outlook_label: "About average",
            education: "No formal education",
            ai_summary: "Domestic cleaning, sweeping, and janitorial work requires physical presence, adaptability to varied environments, and low capital for deployment. Robotic cleaning exists but is economically uncompetitive with India's labor costs. Urbanization and formalization of domestic work drive demand growth."
          },
          {
            code: "92",
            title: "Agricultural, Forestry and Fishery Laborers",
            group: "Elementary Occupations",
            workers_lakhs: 350,
            median_wage_inr: 90000,
            ai_exposure: 2.0,
            outlook_pct: -5,
            outlook_label: "Slower than average",
            education: "No formal education",
            ai_summary: "Agricultural laborers perform physically intensive seasonal work that automated machinery can replace in large-scale farming, but India's small landholding structure limits mechanization economics. Decline is driven by rural-to-urban migration and crop pattern changes, not direct AI displacement."
          },
          {
            code: "93",
            title: "Laborers in Mining, Construction, Manufacturing and Transport",
            group: "Elementary Occupations",
            workers_lakhs: 200,
            median_wage_inr: 150000,
            ai_exposure: 2.5,
            outlook_pct: 8,
            outlook_label: "About average",
            education: "No formal education",
            ai_summary: "Heavy physical labor in construction, mining, and transport loading requires human strength and adaptability to unpredictable environments. Infrastructure investment drives robust demand. AI safety monitoring and coordination tools augment rather than replace this workforce."
          },
          {
            code: "94",
            title: "Food Preparation Assistants",
            group: "Elementary Occupations",
            workers_lakhs: 120,
            median_wage_inr: 130000,
            ai_exposure: 2.5,
            outlook_pct: 12,
            outlook_label: "Faster than average",
            education: "No formal education",
            ai_summary: "Food service demand is growing with urbanization and the restaurant/quick-service economy. Kitchen automation exists at the high-volume industrial level but the diversity of Indian cuisine and informal restaurant formats makes full automation economically impractical at scale."
          },
          {
            code: "95",
            title: "Street and Related Sales and Service Workers",
            group: "Elementary Occupations",
            workers_lakhs: 150,
            median_wage_inr: 110000,
            ai_exposure: 3.5,
            outlook_pct: 3,
            outlook_label: "About average",
            education: "No formal education",
            ai_summary: "Street vendors, hawkers, and service workers operate in India's vast informal economy with minimal digital footprint. Quick commerce platforms (Zepto, Blinkit) are reshaping distribution, creating some new gig roles while displacing traditional street retail. Net impact is moderate and slow-moving."
          },
          {
            code: "96",
            title: "Refuse Workers and Other Elementary Workers",
            group: "Elementary Occupations",
            workers_lakhs: 80,
            median_wage_inr: 100000,
            ai_exposure: 1.5,
            outlook_pct: 5,
            outlook_label: "About average",
            education: "No formal education",
            ai_summary: "Waste management, portering, and similar roles are among the most physically demanding and lowest-digitization occupations. Swachh Bharat Mission formalization is creating more organized employment in this sector. AI-aided waste sorting exists in modern facilities but has minimal impact on informal ragpicker livelihoods."
          }
        ]
      }
    ]
  };

  // ============================================================
  // CONFIG — added in Task 8
  // ============================================================

  // ============================================================
  // UTILS — added in Task 9
  // ============================================================

  // ============================================================
  // STATS — added in Task 13
  // ============================================================

  // ============================================================
  // TOOLTIP — added in Task 12
  // ============================================================

  // ============================================================
  // TREEMAP — added in Task 10
  // ============================================================

  // ============================================================
  // ZOOM — added in Task 14
  // ============================================================

  // ============================================================
  // LAYERS — added in Task 11
  // ============================================================

  // ============================================================
  // INIT — added in Task 15
  // ============================================================

  </script>
  </body>
  </html>
  ```

- [ ] **Step 2: Validate JSON structure in browser console**

  Open `index.html` in Chrome. Open DevTools Console (F12). Run:

  ```javascript
  // Should print 9 (major groups)
  console.log(JOBS_DATA.children.length);

  // Should print total sub-groups count (should be 36+ groups)
  const total = JOBS_DATA.children.reduce((sum, g) => sum + g.children.length, 0);
  console.log('Total sub-groups:', total);

  // Check a random entry has all required fields
  const sample = JOBS_DATA.children[4].children[0];
  const required = ['code','title','group','workers_lakhs','median_wage_inr','ai_exposure','outlook_pct','outlook_label','education','ai_summary'];
  const missing = required.filter(f => sample[f] === undefined);
  console.log('Missing fields:', missing); // Should be []
  ```

  Expected output: `9`, `Total sub-groups: 36` (or more if you added groups), `Missing fields: []`

- [ ] **Step 3: Commit**

  ```bash
  git add index.html
  git commit -m "data: embed compiled India job dataset into index.html"
  ```

---

## Task 6: CSS Layout and Visual Shell

**Files:**
- Modify: `index.html` (the `<style>` block)

- [ ] **Step 1: Add CSS to the `<style>` block**

  Replace the `/* CSS added in Task 7 */` comment with:

  ```css
  * { box-sizing: border-box; margin: 0; padding: 0; }

  body {
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
    background: #1a1a2e;
    color: #e0e0e0;
    min-height: 100vh;
  }

  #header {
    padding: 16px 20px 12px;
    background: #16213e;
    border-bottom: 1px solid #0f3460;
  }

  #header h1 {
    font-size: 22px;
    font-weight: 700;
    color: #e0e0e0;
    margin-bottom: 4px;
  }

  #header .subtitle {
    font-size: 13px;
    color: #888;
    margin-bottom: 14px;
  }

  #layer-controls {
    display: flex;
    gap: 8px;
    margin-bottom: 14px;
  }

  .layer-btn {
    padding: 6px 14px;
    border: 1px solid #444;
    border-radius: 20px;
    background: transparent;
    color: #aaa;
    font-size: 13px;
    cursor: pointer;
    transition: all 0.2s;
  }

  .layer-btn:hover { border-color: #888; color: #ddd; }

  .layer-btn.active {
    background: #0f3460;
    border-color: #e94560;
    color: #fff;
  }

  #stats-bar {
    display: flex;
    gap: 28px;
    flex-wrap: wrap;
    font-size: 12px;
  }

  .stat-block { display: flex; flex-direction: column; gap: 2px; }
  .stat-label { color: #666; text-transform: uppercase; letter-spacing: 0.5px; font-size: 10px; }
  .stat-value { color: #e0e0e0; font-size: 20px; font-weight: 700; }
  .stat-sub { color: #888; font-size: 11px; }

  #breadcrumb {
    padding: 8px 20px;
    background: #0f3460;
    font-size: 13px;
    color: #888;
    display: none;
  }

  #breadcrumb span { color: #e94560; cursor: pointer; }
  #breadcrumb span:hover { text-decoration: underline; }

  #treemap-container {
    width: 100%;
    overflow: hidden;
  }

  #treemap-container svg { display: block; }

  .node rect {
    stroke: #1a1a2e;
    stroke-width: 1.5px;
    cursor: pointer;
    transition: opacity 0.2s;
  }

  .node rect:hover { opacity: 0.85; }

  .node text {
    pointer-events: none;
    fill: #fff;
    font-size: 11px;
    font-weight: 600;
    text-shadow: 0 1px 2px rgba(0,0,0,0.8);
  }

  .node .workers-label {
    font-size: 10px;
    font-weight: 400;
    fill: rgba(255,255,255,0.75);
  }

  .group-header {
    fill: rgba(0,0,0,0.5) !important;
    font-size: 12px;
    font-weight: 700;
    text-transform: uppercase;
    letter-spacing: 0.5px;
  }

  #tooltip {
    position: fixed;
    display: none;
    background: #16213e;
    border: 1px solid #0f3460;
    border-radius: 8px;
    padding: 14px 16px;
    min-width: 280px;
    max-width: 340px;
    z-index: 1000;
    pointer-events: none;
    box-shadow: 0 8px 32px rgba(0,0,0,0.5);
  }

  #tooltip .tt-title {
    font-size: 15px;
    font-weight: 700;
    color: #fff;
    margin-bottom: 8px;
    line-height: 1.3;
  }

  #tooltip .tt-exposure {
    display: inline-block;
    padding: 3px 10px;
    border-radius: 12px;
    font-size: 12px;
    font-weight: 700;
    margin-bottom: 10px;
  }

  #tooltip .tt-grid {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 8px;
    margin-bottom: 10px;
  }

  #tooltip .tt-field { display: flex; flex-direction: column; gap: 2px; }
  #tooltip .tt-field-label { font-size: 10px; color: #666; text-transform: uppercase; }
  #tooltip .tt-field-value { font-size: 13px; color: #e0e0e0; font-weight: 600; }

  #tooltip .tt-summary {
    font-size: 12px;
    color: #aaa;
    line-height: 1.5;
    border-top: 1px solid #0f3460;
    padding-top: 8px;
  }

  #footer {
    padding: 16px 20px;
    background: #16213e;
    border-top: 1px solid #0f3460;
    font-size: 11px;
    color: #555;
    line-height: 1.8;
  }

  #footer strong { color: #777; }
  ```

- [ ] **Step 2: Add HTML structure body**

  Replace the `<body>` tag and everything up to the `<script>` tag with:

  ```html
  <body>
  <div id="header">
    <h1>India Job Market Visualizer</h1>
    <p class="subtitle">Based on PLFS 2022-23 · ~560M workers · AI exposure scored from ILO, WEF, and Oxford research</p>
    <div id="layer-controls">
      <button class="layer-btn active" data-layer="ai_exposure">Digital AI Exposure</button>
      <button class="layer-btn" data-layer="median_wage">Median Pay</button>
      <button class="layer-btn" data-layer="outlook">Employment Outlook</button>
      <button class="layer-btn" data-layer="education">Education Level</button>
    </div>
    <div id="stats-bar">
      <!-- populated by JS -->
    </div>
  </div>
  <div id="breadcrumb"></div>
  <div id="treemap-container"></div>
  <div id="tooltip"></div>
  <div id="footer">
    <strong>Data Sources:</strong>
    PLFS 2022-23 (NSO/MoSPI) · NCO-2015 (MoSPI) · ILO India Employment Report 2024 ·
    WEF Future of Jobs Report 2023 · ILO "Generative AI and Jobs" 2023 ·
    Oxford/ILO Automation Risk for BRICS (Banga &amp; te Velde)
  </div>
  ```

- [ ] **Step 3: Verify shell renders in browser**

  Open `index.html`. You should see: dark background, header with title and 4 buttons, footer. No treemap yet — that comes in Task 10. Console should have zero errors.

- [ ] **Step 4: Commit**

  ```bash
  git add index.html
  git commit -m "feat: add HTML structure and CSS layout"
  ```

---

## Task 7: Utility Functions

**Files:**
- Modify: `index.html` (the `// UTILS` section)

- [ ] **Step 1: Add utility functions**

  Replace `// ============================================================\n  // UTILS — added in Task 9\n  // ============================================================` with:

  ```javascript
  // ============================================================
  // UTILS
  // ============================================================
  function formatIndian(n) {
    if (n >= 100) return `${(n / 100).toFixed(1)} Cr`;
    if (n >= 1) return `${n.toFixed(1)} L`;
    return `${Math.round(n * 100)}K`;
  }

  function formatWage(inr) {
    if (inr >= 1e7) return `₹${(inr / 1e7).toFixed(1)}Cr/yr`;
    if (inr >= 1e5) return `₹${(inr / 1e5).toFixed(1)}L/yr`;
    return `₹${(inr / 1000).toFixed(0)}K/yr`;
  }

  function getExposureLabel(score) {
    if (score >= 8) return { label: 'Critical', bg: '#c0392b', text: '#fff' };
    if (score >= 6) return { label: 'High', bg: '#e67e22', text: '#fff' };
    if (score >= 4) return { label: 'Moderate', bg: '#f1c40f', text: '#000' };
    return { label: 'Low', bg: '#27ae60', text: '#fff' };
  }

  function getOutlookLabel(pct) {
    if (pct >= 20) return 'Much faster than average';
    if (pct >= 10) return 'Faster than average';
    if (pct >= 1) return 'About average';
    if (pct >= -9) return 'Slower than average';
    return 'Declining';
  }

  function clamp(val, min, max) {
    return Math.min(Math.max(val, min), max);
  }
  ```

- [ ] **Step 2: Test utilities in browser console**

  Open `index.html`. Open DevTools Console (F12). Run:

  ```javascript
  // Indian number formatting
  console.assert(formatIndian(150) === '1.5 Cr', 'formatIndian 150L test');
  console.assert(formatIndian(42) === '42.0 L', 'formatIndian 42L test');
  console.assert(formatIndian(0.5) === '50K', 'formatIndian 0.5L test');

  // Wage formatting
  console.assert(formatWage(1200000) === '₹12.0L/yr', 'formatWage 12L test');
  console.assert(formatWage(850000) === '₹8.5L/yr', 'formatWage 8.5L test');

  // Exposure labels
  console.assert(getExposureLabel(9.0).label === 'Critical', 'exposure 9 is Critical');
  console.assert(getExposureLabel(2.0).label === 'Low', 'exposure 2 is Low');
  console.assert(getExposureLabel(5.0).label === 'Moderate', 'exposure 5 is Moderate');

  console.log('All utility tests passed');
  ```

  Expected: `All utility tests passed` with no assertion errors.

- [ ] **Step 3: Commit**

  ```bash
  git add index.html
  git commit -m "feat: add utility functions for formatting and labels"
  ```

---

## Task 8: Color Scale Configuration

**Files:**
- Modify: `index.html` (the `// CONFIG` section)

- [ ] **Step 1: Add CONFIG block**

  Replace `// ============================================================\n  // CONFIG — added in Task 8\n  // ============================================================` with:

  ```javascript
  // ============================================================
  // CONFIG
  // ============================================================
  let activeLayer = 'ai_exposure';

  const LAYERS = {
    ai_exposure: {
      label: 'Digital AI Exposure',
      getValue: d => d.ai_exposure,
      colorScale: () => d3.scaleSequential(d3.interpolateRdYlGn).domain([10, 1]),
      format: v => `${v.toFixed(1)}/10`
    },
    median_wage: {
      label: 'Median Pay',
      getValue: d => d.median_wage_inr,
      colorScale: (data) => {
        const vals = data.flatMap(g => g.children.map(d => d.median_wage_inr));
        return d3.scaleSequential(d3.interpolateGreens).domain([d3.min(vals), d3.max(vals)]);
      },
      format: v => formatWage(v)
    },
    outlook: {
      label: 'Employment Outlook',
      getValue: d => d.outlook_pct,
      colorScale: () => d3.scaleSequential(d3.interpolateRdYlGn).domain([-15, 25]),
      format: v => `${v > 0 ? '+' : ''}${v}%`
    },
    education: {
      label: 'Education Level',
      getValue: d => d.education,
      colorScale: () => d3.scaleOrdinal()
        .domain(["No formal education", "No formal", "HS Diploma", "Bachelor's", "Master's", "Doctoral/Prof"])
        .range(["#c0392b", "#c0392b", "#e67e22", "#27ae60", "#2980b9", "#8e44ad"]),
      format: v => v
    }
  };

  function getNodeColor(d, layer) {
    const cfg = LAYERS[layer];
    const leaf = d.data;
    if (!leaf || !leaf.code) return '#2c3e50'; // group header
    const val = cfg.getValue(leaf);
    return currentColorScale(val);
  }

  let currentColorScale = LAYERS.ai_exposure.colorScale();
  ```

- [ ] **Step 2: Verify CONFIG loads with no errors**

  Open `index.html`. DevTools Console should show zero errors. Run:

  ```javascript
  console.log(Object.keys(LAYERS)); // ['ai_exposure', 'median_wage', 'outlook', 'education']
  console.log(currentColorScale(5)); // should be a valid CSS color string
  ```

  Expected: `['ai_exposure', 'median_wage', 'outlook', 'education']` and a color like `rgb(...)`.

- [ ] **Step 3: Commit**

  ```bash
  git add index.html
  git commit -m "feat: add color scale configuration for all 4 layers"
  ```

---

## Task 9: D3 Treemap Core Render

**Files:**
- Modify: `index.html` (the `// TREEMAP` section)

- [ ] **Step 1: Add treemap render function**

  Replace `// ============================================================\n  // TREEMAP — added in Task 10\n  // ============================================================` with:

  ```javascript
  // ============================================================
  // TREEMAP
  // ============================================================
  let svg, treemapLayout, currentRoot;

  function initTreemap() {
    const container = document.getElementById('treemap-container');
    const W = container.clientWidth;
    const H = window.innerHeight - document.getElementById('header').offsetHeight - 80;

    svg = d3.select('#treemap-container')
      .append('svg')
      .attr('width', W)
      .attr('height', H);

    treemapLayout = d3.treemap()
      .size([W, H])
      .paddingTop(22)
      .paddingInner(2)
      .paddingOuter(4)
      .tile(d3.treemapSquarify);

    renderTreemap(JOBS_DATA);
  }

  function renderTreemap(rootData) {
    svg.selectAll('*').remove();

    const root = d3.hierarchy(rootData)
      .sum(d => d.workers_lakhs || 0)
      .sort((a, b) => b.value - a.value);

    currentRoot = root;
    treemapLayout(root);

    // Draw major group backgrounds
    const groups = svg.selectAll('.group')
      .data(root.children)
      .enter().append('g')
      .attr('class', 'group');

    groups.append('rect')
      .attr('x', d => d.x0)
      .attr('y', d => d.y0)
      .attr('width', d => d.x1 - d.x0)
      .attr('height', d => d.y1 - d.y0)
      .attr('fill', '#0f3460')
      .attr('stroke', '#1a1a2e')
      .attr('stroke-width', 2);

    groups.append('text')
      .attr('class', 'group-header')
      .attr('x', d => d.x0 + 6)
      .attr('y', d => d.y0 + 15)
      .text(d => d.data.name.toUpperCase())
      .attr('fill', 'rgba(255,255,255,0.6)')
      .attr('font-size', 11)
      .attr('font-weight', 700)
      .attr('letter-spacing', '0.5px');

    // Draw leaf nodes (sub-major groups)
    const leaves = svg.selectAll('.node')
      .data(root.leaves())
      .enter().append('g')
      .attr('class', 'node')
      .attr('transform', d => `translate(${d.x0},${d.y0})`);

    leaves.append('rect')
      .attr('width', d => Math.max(0, d.x1 - d.x0))
      .attr('height', d => Math.max(0, d.y1 - d.y0))
      .attr('fill', d => getNodeColor(d, activeLayer))
      .attr('data-code', d => d.data.code);

    drawLabels(leaves);
    attachTooltips(leaves);
    attachZoom(groups);
  }

  function drawLabels(leaves) {
    leaves.each(function(d) {
      const w = d.x1 - d.x0;
      const h = d.y1 - d.y0;
      if (w < 30 || h < 20) return;

      const g = d3.select(this);
      const title = d.data.title || '';
      const words = title.split(' ');
      const maxChars = Math.floor(w / 6.5);

      // Title line — truncate to fit width
      let line1 = words.slice(0, 3).join(' ');
      if (line1.length > maxChars) line1 = line1.slice(0, maxChars - 1) + '…';

      g.append('text')
        .attr('x', 5)
        .attr('y', h > 40 ? h / 2 - 8 : h / 2 + 4)
        .text(line1)
        .attr('font-size', clamp(Math.min(w / 10, 12), 8, 13))
        .attr('fill', '#fff');

      if (h > 40) {
        g.append('text')
          .attr('class', 'workers-label')
          .attr('x', 5)
          .attr('y', h / 2 + 8)
          .text(`${formatIndian(d.data.workers_lakhs)} workers`)
          .attr('font-size', clamp(Math.min(w / 12, 10), 7, 11))
          .attr('fill', 'rgba(255,255,255,0.75)');
      }
    });
  }
  ```

- [ ] **Step 2: Wire initTreemap into INIT section**

  Replace `// ============================================================\n  // INIT — added in Task 15\n  // ============================================================` with:

  ```javascript
  // ============================================================
  // INIT
  // ============================================================
  function main() {
    initTreemap();
  }

  document.addEventListener('DOMContentLoaded', main);
  ```

- [ ] **Step 3: Verify treemap renders**

  Open `index.html`. You should see a color-coded treemap filling the screen with major group labels and worker counts inside boxes. Agriculture and Elementary Occupations should be the largest boxes. Console: zero errors.

- [ ] **Step 4: Commit**

  ```bash
  git add index.html
  git commit -m "feat: add D3 treemap core render with labels"
  ```

---

## Task 10: Tooltip Component

**Files:**
- Modify: `index.html` (the `// TOOLTIP` section)

- [ ] **Step 1: Add tooltip functions**

  Replace `// ============================================================\n  // TOOLTIP — added in Task 12\n  // ============================================================` with:

  ```javascript
  // ============================================================
  // TOOLTIP
  // ============================================================
  function showTooltip(event, d) {
    const data = d.data;
    if (!data.code) return; // group header, no tooltip

    const exp = getExposureLabel(data.ai_exposure);
    const tt = document.getElementById('tooltip');

    tt.innerHTML = `
      <div class="tt-title">${data.title}</div>
      <div class="tt-exposure" style="background:${exp.bg};color:${exp.text}">
        AI Exposure: ${data.ai_exposure}/10 — ${exp.label}
      </div>
      <div class="tt-grid">
        <div class="tt-field">
          <span class="tt-field-label">Workers</span>
          <span class="tt-field-value">${formatIndian(data.workers_lakhs)}</span>
        </div>
        <div class="tt-field">
          <span class="tt-field-label">Median Pay</span>
          <span class="tt-field-value">${formatWage(data.median_wage_inr)}</span>
        </div>
        <div class="tt-field">
          <span class="tt-field-label">Outlook (5yr)</span>
          <span class="tt-field-value" style="color:${data.outlook_pct >= 0 ? '#27ae60' : '#e74c3c'}">
            ${data.outlook_pct > 0 ? '+' : ''}${data.outlook_pct}% · ${data.outlook_label}
          </span>
        </div>
        <div class="tt-field">
          <span class="tt-field-label">Education</span>
          <span class="tt-field-value">${data.education}</span>
        </div>
      </div>
      <div class="tt-summary">${data.ai_summary}</div>
    `;

    positionTooltip(event, tt);
    tt.style.display = 'block';
  }

  function positionTooltip(event, tt) {
    const pad = 12;
    const tw = 340;
    const th = tt.offsetHeight || 220;
    let x = event.clientX + pad;
    let y = event.clientY + pad;

    if (x + tw > window.innerWidth) x = event.clientX - tw - pad;
    if (y + th > window.innerHeight) y = event.clientY - th - pad;

    tt.style.left = `${x}px`;
    tt.style.top = `${y}px`;
  }

  function hideTooltip() {
    document.getElementById('tooltip').style.display = 'none';
  }

  function attachTooltips(leaves) {
    leaves
      .on('mouseover', showTooltip)
      .on('mousemove', (event, d) => {
        positionTooltip(event, document.getElementById('tooltip'));
      })
      .on('mouseout', hideTooltip);
  }
  ```

- [ ] **Step 2: Verify tooltip appears on hover**

  Open `index.html`. Hover over any rectangle. A card should appear with: title, color-coded AI exposure badge, 4 data fields in a grid, and the 2–3 sentence AI summary. Move mouse to edges of screen — tooltip should flip position to stay in viewport.

- [ ] **Step 3: Commit**

  ```bash
  git add index.html
  git commit -m "feat: add tooltip component with all data fields"
  ```

---

## Task 11: Layer Toggle System

**Files:**
- Modify: `index.html` (the `// LAYERS` section)

- [ ] **Step 1: Add layer switching functions**

  Replace `// ============================================================\n  // LAYERS — added in Task 11\n  // ============================================================` with:

  ```javascript
  // ============================================================
  // LAYERS
  // ============================================================
  function setActiveLayer(layerKey) {
    activeLayer = layerKey;

    // Update color scale
    const cfg = LAYERS[layerKey];
    if (layerKey === 'median_wage') {
      currentColorScale = cfg.colorScale(JOBS_DATA.children);
    } else {
      currentColorScale = cfg.colorScale();
    }

    applyColors();
    updateStats();
  }

  function applyColors() {
    svg.selectAll('.node rect')
      .transition()
      .duration(250)
      .attr('fill', function(d) {
        return getNodeColor(d, activeLayer);
      });
  }

  function initLayerControls() {
    document.querySelectorAll('.layer-btn').forEach(btn => {
      btn.addEventListener('click', () => {
        document.querySelectorAll('.layer-btn').forEach(b => b.classList.remove('active'));
        btn.classList.add('active');
        setActiveLayer(btn.dataset.layer);
      });
    });
  }
  ```

- [ ] **Step 2: Call initLayerControls from main()**

  Update the `main()` function to:

  ```javascript
  function main() {
    initTreemap();
    initLayerControls();
    updateStats();
  }
  ```

- [ ] **Step 3: Verify layer switching**

  Open `index.html`. Click each of the 4 layer buttons. The treemap should smoothly re-color (250ms transition) with each click. The AI Exposure layer should be red/green. The Median Pay layer should be greens. The Outlook layer should be red/green based on growth. The Education layer should be categorical colors. Console: zero errors.

- [ ] **Step 4: Commit**

  ```bash
  git add index.html
  git commit -m "feat: add layer toggle system with animated transitions"
  ```

---

## Task 12: Click-to-Zoom

**Files:**
- Modify: `index.html` (the `// ZOOM` section)

- [ ] **Step 1: Add zoom functions**

  Replace `// ============================================================\n  // ZOOM — added in Task 14\n  // ============================================================` with:

  ```javascript
  // ============================================================
  // ZOOM
  // ============================================================
  let zoomedGroup = null;

  function attachZoom(groups) {
    groups.on('click', function(event, d) {
      if (zoomedGroup === d.data.name) {
        zoomOut();
      } else {
        zoomTo(d);
      }
    });
  }

  function zoomTo(groupNode) {
    zoomedGroup = groupNode.data.name;

    const subData = {
      name: groupNode.data.name,
      children: groupNode.data.children
    };

    renderTreemap(subData);
    updateBreadcrumb(groupNode.data.name);
    updateStats();
  }

  function zoomOut() {
    zoomedGroup = null;
    renderTreemap(JOBS_DATA);
    hideBreadcrumb();
    updateStats();
  }

  function updateBreadcrumb(groupName) {
    const bc = document.getElementById('breadcrumb');
    bc.style.display = 'block';
    bc.innerHTML = `<span onclick="zoomOut()">All Occupations</span> › ${groupName}`;
  }

  function hideBreadcrumb() {
    const bc = document.getElementById('breadcrumb');
    bc.style.display = 'none';
    bc.innerHTML = '';
  }
  ```

- [ ] **Step 2: Verify zoom behavior**

  Open `index.html`. Click on any major group rectangle area (the dark blue header background). The treemap should zoom into that group's sub-occupations filling the full screen. A breadcrumb `All Occupations › Managers` should appear. Click the breadcrumb `All Occupations` link to zoom back out. Console: zero errors.

- [ ] **Step 3: Commit**

  ```bash
  git add index.html
  git commit -m "feat: add click-to-zoom with breadcrumb navigation"
  ```

---

## Task 13: Header Stats Bar

**Files:**
- Modify: `index.html` (the `// STATS` section)

- [ ] **Step 1: Add stats computation and render**

  Replace `// ============================================================\n  // STATS — added in Task 13\n  // ============================================================` with:

  ```javascript
  // ============================================================
  // STATS
  // ============================================================
  function getAllLeaves(data) {
    const leaves = [];
    function walk(node) {
      if (node.children) node.children.forEach(walk);
      else leaves.push(node);
    }
    walk(data);
    return leaves;
  }

  function computeStats(data) {
    const leaves = getAllLeaves(data);
    const totalWorkers = leaves.reduce((s, d) => s + d.workers_lakhs, 0);
    const highAI = leaves.filter(d => d.ai_exposure >= 7);
    const highAIWorkers = highAI.reduce((s, d) => s + d.workers_lakhs, 0);
    const avgWage = leaves.reduce((s, d) => s + d.median_wage_inr * d.workers_lakhs, 0) / totalWorkers;
    const avgOutlook = leaves.reduce((s, d) => s + d.outlook_pct * d.workers_lakhs, 0) / totalWorkers;

    return { totalWorkers, highAIWorkers, highAIPct: (highAIWorkers / totalWorkers) * 100, avgWage, avgOutlook };
  }

  function updateStats() {
    const data = zoomedGroup
      ? JOBS_DATA.children.find(g => g.name === zoomedGroup) || JOBS_DATA
      : JOBS_DATA;

    const s = computeStats(data);

    document.getElementById('stats-bar').innerHTML = `
      <div class="stat-block">
        <span class="stat-label">Total Workforce</span>
        <span class="stat-value">${formatIndian(s.totalWorkers)}</span>
        <span class="stat-sub">PLFS 2022-23</span>
      </div>
      <div class="stat-block">
        <span class="stat-label">High AI Exposure (≥7/10)</span>
        <span class="stat-value" style="color:#e74c3c">${formatIndian(s.highAIWorkers)}</span>
        <span class="stat-sub">${s.highAIPct.toFixed(1)}% of workforce</span>
      </div>
      <div class="stat-block">
        <span class="stat-label">Avg Median Pay</span>
        <span class="stat-value">${formatWage(s.avgWage)}</span>
        <span class="stat-sub">Weighted by workers</span>
      </div>
      <div class="stat-block">
        <span class="stat-label">Avg 5yr Outlook</span>
        <span class="stat-value" style="color:${s.avgOutlook >= 0 ? '#27ae60' : '#e74c3c'}">
          ${s.avgOutlook > 0 ? '+' : ''}${s.avgOutlook.toFixed(1)}%
        </span>
        <span class="stat-sub">Weighted by workers</span>
      </div>
    `;
  }
  ```

- [ ] **Step 2: Verify stats bar populates**

  Open `index.html`. The header should now show 4 stat blocks: total workforce (~560 Crore), high AI exposure count, average wage, average outlook. Zoom into a group — stats should update to reflect that group only. Zoom back out — stats return to full-market view.

- [ ] **Step 3: Commit**

  ```bash
  git add index.html
  git commit -m "feat: add header stats bar with workforce metrics"
  ```

---

## Task 14: Final Polish

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Add window resize handler**

  Add to the `main()` function body, after `initLayerControls()`:

  ```javascript
  window.addEventListener('resize', () => {
    const container = document.getElementById('treemap-container');
    const W = container.clientWidth;
    const H = window.innerHeight - document.getElementById('header').offsetHeight - 80;
    svg.attr('width', W).attr('height', H);
    treemapLayout.size([W, H]);
    const data = zoomedGroup
      ? JOBS_DATA.children.find(g => g.name === zoomedGroup) || JOBS_DATA
      : JOBS_DATA;
    renderTreemap(data);
  });
  ```

- [ ] **Step 2: Add color legend under the layer controls**

  Add this HTML after `<div id="stats-bar">` in the body:

  ```html
  <div id="legend" style="margin-top:8px;display:flex;align-items:center;gap:10px;font-size:11px;color:#666;">
    <span>Low risk</span>
    <div id="legend-gradient" style="width:120px;height:10px;border-radius:4px;background:linear-gradient(to right, #1a9850, #ffffbf, #d73027);"></div>
    <span>High risk</span>
  </div>
  ```

  Update the legend gradient dynamically when layer changes by adding to `setActiveLayer()`:

  ```javascript
  function updateLegend(layerKey) {
    const legends = {
      ai_exposure: 'linear-gradient(to right, #1a9850, #ffffbf, #d73027)',
      median_wage: 'linear-gradient(to right, #e5f5e0, #31a354)',
      outlook: 'linear-gradient(to right, #d73027, #ffffbf, #1a9850)',
      education: 'linear-gradient(to right, #c0392b, #e67e22, #27ae60, #2980b9, #8e44ad)'
    };
    document.getElementById('legend-gradient').style.background = legends[layerKey];
    const labels = {
      ai_exposure: ['Low risk', 'High risk'],
      median_wage: ['Low pay', 'High pay'],
      outlook: ['Declining', 'Growing'],
      education: ['No degree', 'Doctoral']
    };
    const [left, right] = labels[layerKey];
    document.querySelector('#legend span:first-child').textContent = left;
    document.querySelector('#legend span:last-child').textContent = right;
  }
  ```

  Call `updateLegend(layerKey)` at the end of `setActiveLayer()`.

- [ ] **Step 3: Verify the full experience**

  Open `index.html`. Run through this checklist manually:
  - [ ] Treemap fills the screen, all major groups visible
  - [ ] Hover over a box → tooltip appears with all fields
  - [ ] Move mouse to screen corners → tooltip doesn't overflow viewport
  - [ ] Click each of the 4 layer buttons → treemap re-colors smoothly
  - [ ] Legend updates label when layer changes
  - [ ] Click a major group → zooms in, breadcrumb appears
  - [ ] Click breadcrumb → zooms back out
  - [ ] Resize window → treemap redraws to fit
  - [ ] Stats bar shows correct totals
  - [ ] Zero console errors

- [ ] **Step 4: Commit**

  ```bash
  git add index.html
  git commit -m "feat: add resize handler, color legend, and final polish"
  ```

---

## Task 15: README and GitHub Pages Deploy

**Files:**
- Create: `README.md`

- [ ] **Step 1: Create README**

  ```bash
  cat > README.md << 'EOF'
  # India Job Market Visualizer

  An interactive treemap visualization of India's job market, modeled on
  [Karpathy's US Job Market Visualizer](https://karpathy.ai/jobs/).

  **Primary focus: Digital AI Exposure** — how vulnerable or enhanced each
  occupation category is by AI/automation, scored 1–10 from published research.

  ## Data Sources

  | Data | Source |
  |------|--------|
  | Worker counts | PLFS 2022-23, NSO/MoSPI |
  | Occupation taxonomy | NCO-2015, MoSPI |
  | AI exposure scores | ILO 2023 "Generative AI and Jobs", WEF Future of Jobs 2023, Oxford/ILO (Banga & te Velde) |
  | Employment outlook | ILO India Employment Report 2024 |
  | Wages | PLFS 2022-23 wage tables |

  ## Running Locally

  ```bash
  # No build step needed
  open index.html
  # or
  python3 -m http.server 8000 && open http://localhost:8000
  ```

  ## Deploying to GitHub Pages

  1. Push this repo to GitHub
  2. Go to Settings → Pages → Source: Deploy from branch → Branch: main → / (root)
  3. Your viz is live at `https://<username>.github.io/<repo-name>/`

  ## AI Exposure Methodology

  See `scoring-notes.md` for per-occupation score sources and rationale.
  EOF
  ```

- [ ] **Step 2: Commit README**

  ```bash
  git add README.md
  git commit -m "docs: add README with data sources and deploy instructions"
  ```

- [ ] **Step 3: Verify GitHub Pages deploy (optional)**

  If you have a GitHub repo:
  ```bash
  git remote add origin https://github.com/<username>/india-job-market-vis.git
  git push -u origin main
  ```
  Then enable GitHub Pages in the repo settings. Visit the Pages URL after ~1 minute.

---

## Self-Review Against Spec

**Spec requirements → tasks:**

| SRS Requirement | Task |
|---|---|
| FR-1: D3 treemap, area = workers, NCO minor group level | Task 9 |
| FR-2: 4 color layers with correct scales | Task 8, 11 |
| FR-3: Header stats bar, updates on layer change | Task 13 |
| FR-4: Tooltip with 6 fields + AI narrative | Task 10 |
| FR-5: Click-to-zoom + breadcrumb | Task 12 |
| FR-6: Data compiled from PLFS, NCO-2015, ILO, WEF | Tasks 1–5 |
| FR-7: Indian number formatting (Lakhs/Crores) | Task 7 |
| FR-8: Footer with source citations | Task 6 (HTML shell) |
| NFR-1: Single index.html, D3 CDN only | Task 5 skeleton |
| NFR-2: No build tools | enforced throughout |
| NFR-3: Layer transition ≤ 300ms | Task 11 (250ms transition) |
| NFR-7: GitHub Pages deployable | Task 15 |

All requirements covered. No gaps.
