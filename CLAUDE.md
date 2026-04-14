# CLAUDE.md — Developer Guide for PackFlow AI

This file tells Claude (and any developer) exactly how to understand, modify, and extend the PackFlow AI demo. Everything lives in a single `index.html` file — no build tools, no frameworks, no dependencies beyond Google Fonts.

---

## Architecture Overview

```
index.html
├── <style>       CSS custom properties + all component styles
├── .sidebar      Left nav — module switching
├── .main         Topbar + scrollable content area  
├── .panel        Slide-out project detail panel (overlay)
└── <script>
    ├── DATA      PROJECTS, ECOS, APPROVALS, DIES, MATERIALS, COMPLIANCE
    ├── STATE     currentModule, panelProject, panelTab, activeFilters
    ├── HELPERS   badge(), statusBadge(), projectCard(), etc.
    └── MODULES   renderPortfolio(), renderDesign(), renderMaterials(), ...
```

---

## How to Add a New Module

### Step 1 — Add nav item in the sidebar HTML

Find the sidebar nav section and add a new div:

```html
<div class="nav-item" onclick="showModule('mymodule')">
  <span class="nav-icon">◆</span>My Module
  <span class="nav-badge teal">New</span>
</div>
```

### Step 2 — Register the title

In showModule(), add to the titles object:

```js
mymodule: ['My Module Title', 'Subtitle text here'],
```

### Step 3 — Write the render function

```js
function renderMyModule(el) {
  el.innerHTML = `
    <div class="stat-row" style="grid-template-columns:repeat(3,1fr)">
      <div class="stat-card teal">
        <div class="stat-val" style="color:var(--teal)">42</div>
        <div class="stat-label">My Metric</div>
      </div>
    </div>
  `;
}
```

### Step 4 — Register in renderModule()

```js
const fn = {
  // ... existing ...
  mymodule: renderMyModule,
};
```

---

## Data Structures

### PROJECTS — one object per packaging job

```js
{
  id: 'LB001',
  customer: 'Lumiere Beauty',
  type: 'rigid',               // rigid | corrugated | flexible
  name: 'Eco-Friendly Cosmetic Box',
  status: 'Under Review',      // Under Review | Approved | In Design | Market-Ready
  statusColor: 'amber',        // teal | amber | blue | green | coral
  pcr: 74,                     // actual PCR%
  pcrTarget: 80,               // target PCR%
  co2: 1.2,                    // CO2e kg per run
  waste: 3.8,                  // press waste %
  signals: 3,                  // number of AI signals
  tags: ['PCR', 'Rigid'],
  plant: 'Mumbai Plant 1',
  lifecycle: [                 // array of stage objects
    { stage: 'Concept', date: 'Jan 10', done: true, detail: 'Brief approved.' },
    { stage: 'Artwork', date: 'Feb 28', done: false, blocked: true, detail: 'Blocked.' },
  ],
  bom: [                       // array of BOM line objects
    { cat: 'Substrate', mat: '18pt SBS Board', spec: '250gsm FSC',
      qty: '1,000 sheets', cost: 'Rs0.48', pcr: '60%', co2: '0.31 kg' },
  ],
  bomTotal: 'Rs1,060',
  insights: [                  // AI signal objects
    { icon: 'x', title: 'Signal title', priority: 'high',
      delta: 'down 12%', desc: 'Description.', action: 'Action text' },
  ],
  gate: [                      // release gate checklist
    { label: 'Structural specs approved', status: 'done', detail: 'Confirmed.' },
    { label: 'Artwork pre-flighted',      status: 'fail', detail: 'Pending.' },
  ]
}
```

### ECOS — Engineering Change Orders

```js
{
  id: 'ECO-2024-041',
  sku: 'LB001',
  skuName: 'Eco-Friendly Cosmetic Box',
  customer: 'Lumiere Beauty',
  type: 'Artwork Change',      // Artwork Change | Material Change | Structural Change | Regulatory Update
  raised: 'Mar 18, 2025',
  priority: 'high',            // high | medium | low
  status: 'In Review',         // In Review | Approved | Pending | Closed
  reason: 'Brand refresh — new logo.',
  affectedSkus: 3,
  impactLevel: 'Medium',       // High | Medium | Low
  owner: 'Priya Mehta'
}
```

### APPROVALS — Customer proof approvals

```js
{
  id: 'APR-001',
  project: 'Eco-Friendly Cosmetic Box',
  customer: 'Lumiere Beauty',
  version: 'v3',
  type: 'Colour Proof',
  sent: 'Feb 20, 2025',
  dueIn: '2 days',
  status: 'Awaiting',          // Awaiting | In Review | Approved
  urgent: true,
  proofNotes: 'Pantone 485 gradient check required',
  comments: [
    { author: 'Sarah Chen (Lumiere)', text: 'Looks slightly off.', time: 'Feb 22, 10:41 AM' }
  ]
}
```

### DIES — Die library entries

```js
{
  id: 'DIE-LB-001',
  name: 'Cosmetic Box Tuck-End',
  sku: 'LB001',
  customer: 'Lumiere Beauty',
  type: 'Steel-Rule',          // Steel-Rule | Rotary | Flat-Bed
  size: '320x240mm',
  crease: 4,
  status: 'In Use',            // In Use | Available | Maintenance
  plant: 'Mumbai P1',
  cost: 'Rs42,000',
  made: 'Jan 2025',
  runs: 3,
  shape: 'rect'                // rect | rect-wide | tall | square | slim
}
```

### MATERIALS — Approved materials database

```js
{
  name: '18pt SBS Board',
  supplier: 'ITC Paperboards',
  stock: 'in stock',           // in stock | low | critical
  pcr: '60%',
  co2: '0.31 kg',
  lead: '7d',
  score: 74,                   // sustainability score 0-100
  type: 'substrate',           // substrate | ink | coating | adhesive | bio-plastic
  color: 'teal'                // badge color: teal | green | amber | coral
}
```

### COMPLIANCE — Regulatory compliance records

```js
{
  id: 'CP-LB001',
  project: 'Eco-Friendly Cosmetic Box',
  customer: 'Lumiere Beauty',
  type: 'FMCG / Cosmetics',
  certReq: ['ISO 9001', 'FSC-C', 'BRC IOP'],
  scores: { artwork: 92, material: 78, labeling: 85, regulatory: 90 },
  issues: ['Pantone 485 approval pending'],
  status: 'partial'            // pass | partial | fail
}
```

---

## CSS Design Tokens

```css
/* Accent colours */
--teal: #12b89a;       /* primary accent, active nav, progress */
--amber: #f59e0b;      /* warning, pending, medium priority */
--coral: #ef4444;      /* danger, blocked, high priority */
--blue: #3b82f6;       /* info, In Design status */
--purple: #a78bfa;     /* ECO type badges */
--green: #22c55e;      /* success, approved, market-ready */

/* Backgrounds */
--bg: #0b0f1a;         /* page background */
--bg2: #111827;        /* sidebar + cards */
--bg3: #1a2235;        /* hover, inner cards */
--bg4: #1f2a3d;        /* deepest nested */

/* Text */
--text: #e2e8f0;       /* primary */
--text2: #94a3b8;      /* secondary */
--text3: #64748b;      /* muted, labels */

/* Borders */
--border: #1e2d45;
--border2: #243447;
```

---

## Reusable Components (copy-paste HTML)

### Stat card
```html
<div class="stat-card teal">
  <div class="stat-val" style="color:var(--teal)">42</div>
  <div class="stat-label">Metric Name</div>
  <div class="stat-delta delta-up">up 12% vs last month</div>
</div>
```
Colour options for stat-card class: teal | amber | coral | blue | purple | green

### AI insight card
```html
<div class="ai-card high">
  <div class="ai-icon" style="background:#2a1200">!</div>
  <div class="ai-body">
    <div class="ai-title-row">
      <div class="ai-title">Signal title here</div>
      <span class="badge coral">HIGH</span>
    </div>
    <div class="ai-desc">Detailed description text.</div>
    <div class="ai-action">Action recommended</div>
  </div>
</div>
```
Priority class options: high | medium | low (sets the left border colour)

### Data table
```html
<div class="tbl-wrap">
  <table>
    <thead>
      <tr><th>Column 1</th><th>Column 2</th></tr>
    </thead>
    <tbody>
      <tr>
        <td><div class="td-main">Bold primary value</div></td>
        <td class="td-mono">Monospace value</td>
      </tr>
    </tbody>
  </table>
</div>
```

### Badge
```js
// In JS: 
badge('Approved', 'teal')
// Outputs: <span class="badge teal">Approved</span>
// Colour options: teal | amber | coral | blue | purple | green | gray
```

### Progress bar
```html
<div class="progress-wrap">
  <div class="progress-fill" style="width:74%;background:var(--amber)"></div>
</div>
```

---

## Deploying Changes

**Option A — Edit directly on GitHub (easiest):**
1. Open `index.html` in the repo
2. Click the pencil (edit) icon
3. Make your changes
4. Click Commit changes
5. GitHub Pages auto-redeploys in ~2 minutes

**Option B — Clone and push:**
```bash
git clone https://github.com/AbhiRa11/packflow-plm-demo.git
cd packflow-plm-demo
# edit index.html
git add index.html
git commit -m "describe your change"
git push origin main
```

**Live URL:** https://AbhiRa11.github.io/packflow-plm-demo

---

## Enable GitHub Pages (one-time setup)

1. Go to: https://github.com/AbhiRa11/packflow-plm-demo/settings/pages
2. Source: Deploy from a branch
3. Branch: main / Folder: / (root)
4. Click Save
5. Wait ~2 minutes — live at https://AbhiRa11.github.io/packflow-plm-demo

---

## Contact

Built by devx labs · devxlabs.ai · abhishek.rawal@devxlabs.ai
