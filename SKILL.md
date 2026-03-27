---
name: cap-table
version: 0.1.0
description: |
  Plan and visualize a startup cap table from founding through Series B.
  Generates a single self-contained HTML file with adjustable sliders,
  live recalculation, dilution waterfall, Chart.js visualizations, and
  Excel/PDF export. Use when the user mentions "cap table", "equity",
  "dilution", "ownership", "fundraising rounds", "SAFE", "convertible note",
  or "option pool".
allowed-tools:
  - Bash
  - Read
  - Edit
  - Write
  - Glob
  - Grep
  - Agent
  - AskUserQuestion
---

# Cap Table Planner

You generate a **single self-contained HTML file** (`cap-table.html`) that lets
early-stage founders model their cap table from founding through Series B.

> No build step, no framework, no backend. One file, open in any browser.

---

## When to use this skill

- User asks to plan or visualize a cap table
- User wants to model dilution across funding rounds
- User asks about SAFE / convertible note / priced round mechanics
- User wants to export equity breakdown to Excel

---

## What to generate

Create a file called `cap-table.html` (or whatever the user requests) using the
complete template below. Then customize it based on the user's company details
(founder names, round sizes, valuations, etc.).

### Customization checklist

Before delivering, ask the user for (or use defaults):

1. Company name
2. Number of founders + names + equity split
3. Seed round details (instrument, amount, valuation cap)
4. Series A details (pre-money, amount)
5. Series B details (pre-money, amount)

If the user says "just use defaults" or doesn't specify, use the template values.

---

## Complete HTML Template

Generate **exactly** this HTML file, then apply user customizations on top.

````html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Cap Table Planner</title>
<script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.7/dist/chart.umd.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/xlsx@0.18.5/dist/xlsx.full.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
<style>
*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
body { font-family: 'Inter', system-ui, sans-serif; background: #f8f9fa; color: #1a1a2e; line-height: 1.5; }

.header { background: linear-gradient(135deg, #1a1a2e 0%, #2d2b55 100%); color: #fff; padding: 2.5rem 2rem 2rem; text-align: center; }
.header h1 { font-size: 1.8rem; font-weight: 700; margin-bottom: 0.25rem; }
.header p { color: #a5b4fc; font-size: 0.95rem; }

.container { max-width: 1200px; margin: 0 auto; padding: 1.5rem; }

/* Controls */
.controls { background: linear-gradient(135deg, #1a1a2e 0%, #2d2b55 100%); border-radius: 12px; padding: 1.5rem; margin-bottom: 1.5rem; }
.controls h2 { color: #fff; font-size: 1.1rem; margin-bottom: 1rem; border-bottom: 1px solid rgba(255,255,255,0.1); padding-bottom: 0.5rem; }
.control-section { margin-bottom: 1.25rem; }
.control-section h3 { color: #f59e0b; font-size: 0.85rem; text-transform: uppercase; letter-spacing: 0.05em; margin-bottom: 0.75rem; display: flex; align-items: center; gap: 0.5rem; }
.control-grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(220px, 1fr)); gap: 0.75rem; }
.control-item { display: flex; flex-direction: column; gap: 0.25rem; }
.control-item label { color: #cbd5e1; font-size: 0.8rem; display: flex; justify-content: space-between; align-items: center; }
.control-item .val { color: #f59e0b; font-weight: 600; cursor: pointer; padding: 1px 4px; border-radius: 3px; transition: background 0.15s; }
.control-item .val:hover { background: rgba(245,158,11,0.15); }
.control-item .val-input { background: rgba(245,158,11,0.15); color: #f59e0b; border: 1px solid #f59e0b; border-radius: 3px; padding: 1px 4px; font: inherit; font-weight: 600; font-size: inherit; width: 90px; text-align: right; outline: none; }
.control-item input[type=range] { width: 100%; accent-color: #f59e0b; height: 6px; }
.control-item select { background: #16162a; color: #f59e0b; border: 1px solid rgba(245,158,11,0.3); border-radius: 4px; padding: 4px 8px; font-size: 0.8rem; cursor: pointer; outline: none; }
.control-item input[type=text] { background: #16162a; color: #f59e0b; border: 1px solid rgba(245,158,11,0.3); border-radius: 4px; padding: 4px 8px; font-size: 0.8rem; outline: none; width: 100%; }

/* SAFE cards */
.safe-list { display: flex; flex-direction: column; gap: 0.75rem; }
.safe-card { background: rgba(255,255,255,0.04); border: 1px solid rgba(255,255,255,0.08); border-radius: 8px; padding: 0.75rem; position: relative; }
.safe-card.yc-card { border-color: rgba(245,158,11,0.3); background: rgba(245,158,11,0.05); }
.safe-card .safe-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 0.5rem; }
.safe-card .safe-name { color: #e2e8f0; font-weight: 600; font-size: 0.85rem; }
.safe-card .safe-name.yc-badge { color: #f59e0b; }
.safe-card .safe-name.yc-badge::before { content: 'YC '; }
.safe-remove { background: none; border: none; color: #ef4444; cursor: pointer; font-size: 1.1rem; padding: 2px 6px; border-radius: 4px; opacity: 0.6; transition: opacity 0.15s; }
.safe-remove:hover { opacity: 1; background: rgba(239,68,68,0.1); }

/* Buttons */
.btn-row { display: flex; gap: 0.5rem; flex-wrap: wrap; margin-top: 0.5rem; }
.btn-add { background: rgba(99,102,241,0.15); color: #a5b4fc; border: 1px dashed rgba(99,102,241,0.4); border-radius: 6px; padding: 0.4rem 0.85rem; font-size: 0.8rem; font-weight: 500; cursor: pointer; transition: background 0.15s; }
.btn-add:hover { background: rgba(99,102,241,0.25); }
.yc-toggle { display: flex; align-items: center; gap: 0.4rem; cursor: pointer; background: rgba(245,158,11,0.08); border: 1px solid rgba(245,158,11,0.25); border-radius: 6px; padding: 0.35rem 0.75rem; transition: background 0.15s; }
.yc-toggle:hover { background: rgba(245,158,11,0.18); }
.yc-toggle input[type=checkbox] { accent-color: #f59e0b; width: 15px; height: 15px; cursor: pointer; }
.yc-toggle-label { color: #f59e0b; font-size: 0.8rem; font-weight: 600; }

/* Export bar */
.export-bar { display: flex; gap: 0.75rem; margin-bottom: 1.5rem; flex-wrap: wrap; }
.export-bar button { padding: 0.6rem 1.25rem; border: none; border-radius: 8px; font-weight: 600; font-size: 0.85rem; cursor: pointer; transition: transform 0.1s, box-shadow 0.15s; }
.export-bar button:active { transform: scale(0.97); }
.btn-excel { background: #22c55e; color: #fff; }
.btn-excel:hover { box-shadow: 0 4px 12px rgba(34,197,94,0.4); }
.btn-pdf { background: #ef4444; color: #fff; }
.btn-pdf:hover { box-shadow: 0 4px 12px rgba(239,68,68,0.4); }

/* KPI Cards */
.kpi-grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(180px, 1fr)); gap: 0.75rem; margin-bottom: 1.5rem; }
.kpi-card { background: #fff; border-radius: 10px; padding: 1rem; border-left: 4px solid #6366f1; box-shadow: 0 1px 4px rgba(0,0,0,0.06); }
.kpi-card .kpi-label { font-size: 0.75rem; color: #64748b; text-transform: uppercase; letter-spacing: 0.04em; margin-bottom: 0.25rem; }
.kpi-card .kpi-value { font-size: 1.35rem; font-weight: 700; color: #1e293b; }
.kpi-card .kpi-sub { font-size: 0.75rem; color: #94a3b8; margin-top: 0.15rem; }
.kpi-card.green { border-left-color: #22c55e; }
.kpi-card.amber { border-left-color: #f59e0b; }
.kpi-card.purple { border-left-color: #8b5cf6; }
.kpi-card.blue { border-left-color: #3b82f6; }
.kpi-card.red { border-left-color: #ef4444; }

/* Tables */
.table-wrap { background: #fff; border-radius: 10px; box-shadow: 0 1px 4px rgba(0,0,0,0.06); overflow-x: auto; margin-bottom: 1.5rem; }
.table-wrap h2 { padding: 1rem 1.25rem 0.5rem; font-size: 1rem; color: #1e293b; }
table { width: 100%; border-collapse: collapse; font-size: 0.82rem; }
thead th { background: #f1f5f9; padding: 0.6rem 0.75rem; text-align: right; font-weight: 600; color: #475569; white-space: nowrap; position: sticky; top: 0; }
thead th:first-child { text-align: left; }
tbody td { padding: 0.5rem 0.75rem; text-align: right; border-top: 1px solid #f1f5f9; }
tbody td:first-child { text-align: left; font-weight: 500; }
tbody tr:hover { background: #f8fafc; }
tbody tr.total-row { font-weight: 700; background: #f1f5f9; }
.pct { color: #6366f1; font-weight: 600; }
.shares { color: #64748b; font-size: 0.78rem; }

/* Charts */
.charts-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 1.5rem; margin-bottom: 1.5rem; }
.chart-card { background: #fff; border-radius: 10px; padding: 1rem; box-shadow: 0 1px 4px rgba(0,0,0,0.06); }
.chart-card h3 { font-size: 0.9rem; color: #1e293b; margin-bottom: 0.75rem; }
.chart-card canvas { width: 100% !important; height: 280px !important; }

.footer { text-align: center; padding: 1.5rem; color: #94a3b8; font-size: 0.75rem; }

@media (max-width: 768px) {
  .charts-grid { grid-template-columns: 1fr; }
  .control-grid { grid-template-columns: 1fr; }
}
@media print {
  .controls, .export-bar { display: none !important; }
  body { background: #fff; }
}
</style>
</head>
<body>

<div class="header">
  <h1 id="companyName">Cap Table Planner</h1>
  <p>Founding &rarr; SAFEs &rarr; Series A &rarr; Series B</p>
</div>

<div class="container">

<div class="controls" id="controlPanel">
  <h2>Assumptions</h2>

  <!-- Founding -->
  <div class="control-section">
    <h3>Founding</h3>
    <div class="control-grid">
      <div class="control-item">
        <label>Founders <span class="val" data-slider="founders">2</span></label>
        <input type="range" id="founders" min="1" max="5" value="2" step="1">
      </div>
      <div class="control-item">
        <label>Authorized Shares <span class="val" data-slider="authShares">10,000,000</span></label>
        <input type="range" id="authShares" min="1000000" max="20000000" value="10000000" step="1000000">
      </div>
      <div class="control-item">
        <label>Initial Option Pool <span class="val" data-slider="initPool">10%</span></label>
        <input type="range" id="initPool" min="0" max="25" value="10" step="1">
      </div>
    </div>
  </div>

  <!-- SAFEs / Convertible Notes -->
  <div class="control-section">
    <h3>SAFEs &amp; Notes</h3>
    <div class="safe-list" id="safeList"></div>
    <div class="btn-row">
      <button class="btn-add" onclick="addSafe()">+ Add SAFE</button>
      <button class="btn-add" onclick="addNote()">+ Add Note</button>
      <label class="yc-toggle" title="$125K post-money SAFE for 7% + $375K uncapped MFN SAFE">
        <input type="checkbox" id="ycDealToggle" onchange="toggleYCDeal(this.checked)">
        <span class="yc-toggle-label">YC Deal ($500K)</span>
      </label>
    </div>
  </div>

  <!-- Series A -->
  <div class="control-section">
    <h3>Series A</h3>
    <div class="control-grid">
      <div class="control-item">
        <label>Pre-Money Valuation <span class="val" data-slider="aPreMoney">$25M</span></label>
        <input type="range" id="aPreMoney" min="5000000" max="100000000" value="25000000" step="1000000">
      </div>
      <div class="control-item">
        <label>Amount Raised <span class="val" data-slider="aAmount">$8M</span></label>
        <input type="range" id="aAmount" min="1000000" max="30000000" value="8000000" step="500000">
      </div>
      <div class="control-item">
        <label>Option Pool (post) <span class="val" data-slider="aPool">10%</span></label>
        <input type="range" id="aPool" min="0" max="20" value="10" step="1">
      </div>
    </div>
  </div>

  <!-- Series B -->
  <div class="control-section">
    <h3>Series B</h3>
    <div class="control-grid">
      <div class="control-item">
        <label>Pre-Money Valuation <span class="val" data-slider="bPreMoney">$100M</span></label>
        <input type="range" id="bPreMoney" min="20000000" max="500000000" value="100000000" step="5000000">
      </div>
      <div class="control-item">
        <label>Amount Raised <span class="val" data-slider="bAmount">$30M</span></label>
        <input type="range" id="bAmount" min="5000000" max="100000000" value="30000000" step="1000000">
      </div>
      <div class="control-item">
        <label>Option Pool (post) <span class="val" data-slider="bPool">10%</span></label>
        <input type="range" id="bPool" min="0" max="20" value="10" step="1">
      </div>
    </div>
  </div>
</div>

<!-- Export -->
<div class="export-bar">
  <button class="btn-excel" onclick="exportExcel()">Download Excel</button>
  <button class="btn-pdf" onclick="exportPDF()">Export PDF</button>
</div>

<!-- KPIs -->
<div class="kpi-grid" id="kpiGrid"></div>

<!-- Cap Table -->
<div class="table-wrap">
  <h2>Ownership Waterfall</h2>
  <table id="capTable"><thead></thead><tbody></tbody></table>
</div>

<!-- Charts -->
<div class="charts-grid">
  <div class="chart-card">
    <h3>Ownership by Stage</h3>
    <canvas id="ownershipChart"></canvas>
  </div>
  <div class="chart-card">
    <h3>Valuation &amp; Share Price</h3>
    <canvas id="valuationChart"></canvas>
  </div>
</div>

<!-- Detailed Table -->
<div class="table-wrap">
  <h2>Detailed Share Count</h2>
  <table id="shareTable"><thead></thead><tbody></tbody></table>
</div>

<div class="footer">
  For planning purposes only. Consult legal and financial advisors for actual cap table management.
</div>

</div>

<script>
// ───────────────── HELPERS ─────────────────
const $ = id => document.getElementById(id);
const fmt = (n) => {
  if (Math.abs(n) >= 1e9) return '$' + (n/1e9).toFixed(1) + 'B';
  if (Math.abs(n) >= 1e6) return '$' + (n/1e6).toFixed(1) + 'M';
  if (Math.abs(n) >= 1e3) return '$' + (n/1e3).toFixed(0) + 'K';
  return '$' + n.toFixed(0);
};
const fmtPct = n => (n * 100).toFixed(1) + '%';
const fmtShares = n => n.toLocaleString('en-US');
const parseHuman = s => {
  s = s.replace(/[$,%\s]/g, '');
  const m = s.match(/^([0-9.]+)\s*([kmb])?$/i);
  if (!m) return NaN;
  let v = parseFloat(m[1]);
  if (m[2]) { const u = m[2].toLowerCase(); if (u==='k') v*=1e3; if (u==='m') v*=1e6; if (u==='b') v*=1e9; }
  return v;
};

let ownershipChart, valuationChart;

// ───────────────── SAFE STATE ─────────────────
let safeIdCounter = 0;
let safes = []; // { id, name, type:'safe'|'note', safeType:'pre'|'post', amount, valCap, uncapped, discount, interest, termMonths, isYC }

function addSafe(opts = {}) {
  const id = ++safeIdCounter;
  safes.push({
    id,
    name: opts.name || 'SAFE ' + id,
    type: opts.type || 'safe',
    safeType: opts.safeType || 'post',
    amount: opts.amount || 1000000,
    valCap: opts.valCap || 10000000,
    uncapped: opts.uncapped || false,
    discount: opts.discount || 0,
    interest: opts.interest || 0,
    termMonths: opts.termMonths || 18,
    isYC: opts.isYC || false,
  });
  renderSafes();
  recalc();
  return id;
}

function addNote(opts = {}) {
  return addSafe({ ...opts, type: 'note', interest: opts.interest || 0.05, termMonths: opts.termMonths || 18, name: opts.name || 'Note ' + (safeIdCounter + 1) });
}

function toggleYCDeal(on) {
  // Remove existing YC SAFEs
  safes = safes.filter(s => !s.isYC);
  if (on) {
    // $125K post-money SAFE for 7%: cap = $125K / 0.07 = $1,785,714
    addSafe({ name: '$125K for 7%', safeType: 'post', amount: 125000, valCap: 1785714, isYC: true });
    // $375K uncapped MFN SAFE — converts at lowest cap of other SAFEs
    addSafe({ name: '$375K MFN (uncapped)', safeType: 'post', amount: 375000, valCap: 0, uncapped: true, isYC: true });
  } else {
    renderSafes();
    recalc();
  }
}

function removeSafe(id) {
  safes = safes.filter(s => s.id !== id);
  renderSafes();
  recalc();
}

function updateSafe(id, field, value) {
  const s = safes.find(s => s.id === id);
  if (!s) return;
  s[field] = value;
  // If MFN uncapped SAFE, resolve cap to lowest cap among other SAFEs
  // (handled in calculate)
  recalc();
}

function renderSafes() {
  const list = $('safeList');
  list.innerHTML = safes.map(s => {
    const isNote = s.type === 'note';
    const ycClass = s.isYC ? ' yc-card' : '';
    const ycBadge = s.isYC ? ' yc-badge' : '';

    // YC SAFEs are fixed terms — show read-only summary
    if (s.isYC) {
      const capLabel = s.uncapped ? 'Uncapped MFN (converts at lowest cap of other SAFEs)' : 'Post-money cap: ' + fmt(s.valCap) + ' (7% ownership)';
      return `
      <div class="safe-card yc-card" data-safe-id="${s.id}">
        <div class="safe-header">
          <span class="safe-name yc-badge">${s.name}</span>
          <span style="color:#94a3b8;font-size:0.75rem">Fixed terms</span>
        </div>
        <div style="color:#cbd5e1;font-size:0.8rem;display:flex;gap:1.5rem;flex-wrap:wrap">
          <span>Amount: <b style="color:#f59e0b">${fmt(s.amount)}</b></span>
          <span>${capLabel}</span>
        </div>
      </div>`;
    }

    return `
    <div class="safe-card${ycClass}" data-safe-id="${s.id}">
      <div class="safe-header">
        <span class="safe-name${ycBadge}">${s.name}</span>
        <button class="safe-remove" onclick="removeSafe(${s.id})" title="Remove">&times;</button>
      </div>
      <div class="control-grid">
        <div class="control-item">
          <label>Amount <span class="val" data-safe="${s.id}" data-field="amount">${fmt(s.amount)}</span></label>
          <input type="range" min="25000" max="10000000" step="25000" value="${s.amount}"
            oninput="updateSafe(${s.id},'amount',+this.value)">
        </div>
        ${!s.uncapped ? `
        <div class="control-item">
          <label>${s.safeType === 'post' ? 'Post-Money' : 'Pre-Money'} Cap <span class="val" data-safe="${s.id}" data-field="valCap">${fmt(s.valCap)}</span></label>
          <input type="range" min="500000" max="50000000" step="250000" value="${s.valCap}"
            oninput="updateSafe(${s.id},'valCap',+this.value)">
        </div>` : `
        <div class="control-item">
          <label style="color:#f59e0b">Uncapped MFN</label>
          <div style="color:#94a3b8;font-size:0.75rem;margin-top:2px">Converts at lowest cap of other SAFEs</div>
        </div>`}
        <div class="control-item">
          <label>Type</label>
          <select onchange="updateSafe(${s.id},'safeType',this.value)">
            <option value="post" ${s.safeType==='post'?'selected':''}>Post-Money SAFE</option>
            <option value="pre" ${s.safeType==='pre'?'selected':''}>Pre-Money SAFE</option>
            ${isNote ? `<option value="note" selected>Conv. Note</option>` : ''}
          </select>
        </div>
        <div class="control-item">
          <label>Discount <span class="val" data-safe="${s.id}" data-field="discount">${(s.discount*100).toFixed(0)}%</span></label>
          <input type="range" min="0" max="35" step="1" value="${s.discount*100}"
            oninput="updateSafe(${s.id},'discount',+this.value/100)">
        </div>
        ${isNote ? `
        <div class="control-item">
          <label>Interest <span class="val" data-safe="${s.id}" data-field="interest">${(s.interest*100).toFixed(1)}%</span></label>
          <input type="range" min="0" max="12" step="0.5" value="${s.interest*100}"
            oninput="updateSafe(${s.id},'interest',+this.value/100)">
        </div>
        <div class="control-item">
          <label>Term (mo) <span class="val" data-safe="${s.id}" data-field="termMonths">${s.termMonths}</span></label>
          <input type="range" min="6" max="36" step="1" value="${s.termMonths}"
            oninput="updateSafe(${s.id},'termMonths',+this.value)">
        </div>` : ''}
      </div>
    </div>`;
  }).join('');
}

// ───────────────── CLICK-TO-EDIT (sliders) ─────────────────
function setupClickToEdit() {
  document.querySelectorAll('.val[data-slider]').forEach(el => {
    el.addEventListener('click', function() {
      const slider = $(this.dataset.slider);
      if (!slider) return;
      const inp = document.createElement('input');
      inp.className = 'val-input';
      inp.value = this.textContent;
      const self = this;
      const commit = () => {
        let v = parseHuman(inp.value);
        if (isNaN(v)) { inp.replaceWith(self); return; }
        v = Math.max(parseFloat(slider.min), Math.min(parseFloat(slider.max), v));
        slider.value = v;
        inp.replaceWith(self);
        slider.dispatchEvent(new Event('input'));
      };
      inp.addEventListener('blur', commit);
      inp.addEventListener('keydown', e => { if (e.key==='Enter') commit(); if (e.key==='Escape') inp.replaceWith(self); });
      this.replaceWith(inp);
      inp.focus(); inp.select();
    });
  });
}
setupClickToEdit();

// ───────────────── CORE CALCULATION ─────────────────
function getInputs() {
  return {
    founders: parseInt($('founders').value),
    authShares: parseInt($('authShares').value),
    initPool: parseInt($('initPool').value) / 100,
    aPreMoney: parseFloat($('aPreMoney').value),
    aAmount: parseFloat($('aAmount').value),
    aPool: parseInt($('aPool').value) / 100,
    bPreMoney: parseFloat($('bPreMoney').value),
    bAmount: parseFloat($('bAmount').value),
    bPool: parseInt($('bPool').value) / 100,
  };
}

function calculate(inp) {
  // ── FOUNDING ──
  const founderSharesEach = Math.floor(inp.authShares / inp.founders);
  const totalFounderShares = founderSharesEach * inp.founders;

  // ── OPTION POOL ──
  const poolShares = Math.round(inp.authShares * inp.initPool / (1 - inp.initPool));
  const postPoolTotal = totalFounderShares + poolShares;

  // ── SAFE / NOTE CONVERSION ──
  // Resolve MFN uncapped SAFEs: find lowest cap among capped non-YC SAFEs
  // (YC's own 7% SAFE uses an internal cap mechanism, not a market cap)
  const cappedSafes = safes.filter(s => !s.uncapped && s.valCap > 0 && !s.isYC);
  const lowestCap = cappedSafes.length > 0 ? Math.min(...cappedSafes.map(s => s.valCap)) : 0;

  let totalSeedShares = 0;
  const safeResults = [];
  let totalSeedAmount = 0;

  for (const s of safes) {
    let principal = s.amount;
    // Accrue interest for notes
    if (s.type === 'note') {
      principal = s.amount * (1 + s.interest * (s.termMonths / 12));
    }
    totalSeedAmount += s.amount;

    let effectiveCap = s.uncapped ? lowestCap : s.valCap;
    if (effectiveCap <= 0) effectiveCap = inp.aPreMoney; // fallback: convert at Series A price

    let shares;
    if (s.safeType === 'post') {
      // Post-money SAFE: ownership% = amount / post-money cap
      // shares = ownership% * pre_money_shares / (1 - total_post_money_ownership)
      // But for simplicity in multi-SAFE scenario, we compute shares individually
      // then adjust totals. Post-money cap means: amount / cap = guaranteed ownership
      const ownershipPct = principal / effectiveCap;
      // shares such that shares / (postPoolTotal + allSafeShares) = ownershipPct
      // We'll do a two-pass: first compute implied shares, then true up
      shares = Math.round(ownershipPct * postPoolTotal / (1 - ownershipPct));
    } else {
      // Pre-money SAFE: PPS = cap / existing shares
      const pps = effectiveCap / postPoolTotal;
      shares = Math.round(principal / pps);
    }

    totalSeedShares += shares;
    safeResults.push({ ...s, shares, principal, effectiveCap });
  }

  // Post-money SAFE adjustment: for post-money SAFEs, shares should reflect
  // guaranteed ownership of the TOTAL post-conversion company (including other SAFEs).
  // We iterate to converge since post-money SAFEs are interdependent.
  if (safes.some(s => s.safeType === 'post')) {
    for (let iter = 0; iter < 10; iter++) {
      let newTotal = 0;
      const totalPostConversion = postPoolTotal + totalSeedShares;
      for (const sr of safeResults) {
        if (sr.safeType === 'post') {
          const ownershipPct = sr.principal / sr.effectiveCap;
          sr.shares = Math.round(ownershipPct * totalPostConversion);
        }
        newTotal += sr.shares;
      }
      if (Math.abs(newTotal - totalSeedShares) < 1) break;
      totalSeedShares = newTotal;
    }
  }

  const postSeedTotal = postPoolTotal + totalSeedShares;

  // Compute effective seed PPS (blended)
  const seedPPS = totalSeedAmount > 0 ? totalSeedAmount / totalSeedShares : 0;
  const seedEffectiveVal = totalSeedShares > 0 ? seedPPS * postPoolTotal : 0;

  // ── SERIES A (with option pool shuffle) ──
  const aPostMoney = inp.aPreMoney + inp.aAmount;
  const aTargetPoolShares = Math.round((inp.aPool * aPostMoney) / (inp.aPreMoney / postSeedTotal));
  const aNewPoolShares = Math.max(0, aTargetPoolShares - poolShares);
  const preAShares = postSeedTotal + aNewPoolShares;
  const aPPS = inp.aPreMoney / preAShares;
  const aShares = Math.round(inp.aAmount / aPPS);
  const postATotal = preAShares + aShares;
  const aPoolTotal = poolShares + aNewPoolShares;

  // ── SERIES B ──
  const bPostMoney = inp.bPreMoney + inp.bAmount;
  const bTargetPoolShares = Math.round((inp.bPool * bPostMoney) / (inp.bPreMoney / postATotal));
  const bNewPoolShares = Math.max(0, bTargetPoolShares - aPoolTotal);
  const preBShares = postATotal + bNewPoolShares;
  const bPPS = inp.bPreMoney / preBShares;
  const bShares = Math.round(inp.bAmount / bPPS);
  const postBTotal = preBShares + bShares;
  const bPoolTotal = aPoolTotal + bNewPoolShares;

  // ── BUILD ROWS ──
  const stages = ['Founding', 'Post-Pool', 'Post-Seed', 'Post-A', 'Post-B'];
  const rows = [];

  for (let i = 0; i < inp.founders; i++) {
    rows.push({
      name: 'Founder ' + (i + 1),
      shares: [founderSharesEach, founderSharesEach, founderSharesEach, founderSharesEach, founderSharesEach],
      category: 'founder'
    });
  }

  rows.push({
    name: 'Option Pool',
    shares: [0, poolShares, poolShares, aPoolTotal, bPoolTotal],
    category: 'pool'
  });

  // Individual SAFE rows
  for (const sr of safeResults) {
    const label = sr.isYC ? 'YC: ' + sr.name : sr.name;
    rows.push({
      name: label,
      shares: [0, 0, sr.shares, sr.shares, sr.shares],
      category: 'seed'
    });
  }

  rows.push({
    name: 'Series A Investors',
    shares: [0, 0, 0, aShares, aShares],
    category: 'seriesA'
  });

  rows.push({
    name: 'Series B Investors',
    shares: [0, 0, 0, 0, bShares],
    category: 'seriesB'
  });

  const totals = [totalFounderShares, postPoolTotal, postSeedTotal, postATotal, postBTotal];

  for (const r of rows) {
    r.pct = r.shares.map((s, i) => totals[i] > 0 ? s / totals[i] : 0);
  }

  return {
    stages, rows, totals, safeResults,
    seedPPS, aPPS, bPPS,
    seedEffectiveVal,
    aPostMoney, bPostMoney,
    totalSeedAmount,
    totalRaised: totalSeedAmount + inp.aAmount + inp.bAmount,
    founderDilution: 1 - (founderSharesEach / postBTotal),
    aPoolTotal, bPoolTotal,
    totalSeedShares, aShares, bShares,
  };
}

// ───────────────── RENDER ─────────────────
function recalc() {
  const inp = getInputs();
  const data = calculate(inp);
  updateLabels(inp, data);
  renderKPIs(inp, data);
  renderCapTable(data);
  renderShareTable(data);
  renderCharts(data);
}

function updateLabels(inp, data) {
  const sliderLabels = {
    founders: inp.founders,
    authShares: fmtShares(inp.authShares),
    initPool: (inp.initPool * 100).toFixed(0) + '%',
    aPreMoney: fmt(inp.aPreMoney),
    aAmount: fmt(inp.aAmount),
    aPool: (inp.aPool * 100).toFixed(0) + '%',
    bPreMoney: fmt(inp.bPreMoney),
    bAmount: fmt(inp.bAmount),
    bPool: (inp.bPool * 100).toFixed(0) + '%',
  };
  for (const [k, v] of Object.entries(sliderLabels)) {
    const el = document.querySelector(`.val[data-slider="${k}"]`);
    if (el) el.textContent = v;
  }
  // Update SAFE card labels
  for (const s of safes) {
    const amtEl = document.querySelector(`.val[data-safe="${s.id}"][data-field="amount"]`);
    if (amtEl) amtEl.textContent = fmt(s.amount);
    const capEl = document.querySelector(`.val[data-safe="${s.id}"][data-field="valCap"]`);
    if (capEl) capEl.textContent = fmt(s.valCap);
    const discEl = document.querySelector(`.val[data-safe="${s.id}"][data-field="discount"]`);
    if (discEl) discEl.textContent = (s.discount * 100).toFixed(0) + '%';
    const intEl = document.querySelector(`.val[data-safe="${s.id}"][data-field="interest"]`);
    if (intEl) intEl.textContent = (s.interest * 100).toFixed(1) + '%';
    const termEl = document.querySelector(`.val[data-safe="${s.id}"][data-field="termMonths"]`);
    if (termEl) termEl.textContent = s.termMonths;
  }
}

function renderKPIs(inp, data) {
  const cards = [
    { label: 'Total Raised', value: fmt(data.totalRaised), sub: 'SAFEs + A + B', cls: 'green' },
    { label: 'Post-B Valuation', value: fmt(data.bPostMoney), sub: 'Pre-money + round', cls: 'purple' },
    { label: 'Founder Dilution', value: fmtPct(data.founderDilution), sub: 'Per founder, founding \u2192 B', cls: 'amber' },
    { label: 'SAFEs/Notes', value: safes.length.toString(), sub: fmt(data.totalSeedAmount) + ' total', cls: 'blue' },
    { label: 'Series A Price', value: '$' + data.aPPS.toFixed(4), sub: fmtShares(data.aShares) + ' shares', cls: 'blue' },
    { label: 'Series B Price', value: '$' + data.bPPS.toFixed(4), sub: fmtShares(data.bShares) + ' shares', cls: 'blue' },
  ];
  $('kpiGrid').innerHTML = cards.map(c => `
    <div class="kpi-card ${c.cls}">
      <div class="kpi-label">${c.label}</div>
      <div class="kpi-value">${c.value}</div>
      <div class="kpi-sub">${c.sub}</div>
    </div>`).join('');
}

function renderCapTable(data) {
  const thead = `<tr><th>Stakeholder</th>${data.stages.map(s => `<th>${s}</th>`).join('')}</tr>`;
  let tbody = '';
  for (const r of data.rows) {
    tbody += '<tr>';
    tbody += `<td>${r.name}</td>`;
    for (let i = 0; i < data.stages.length; i++) {
      if (r.shares[i] === 0) {
        tbody += '<td style="color:#cbd5e1">&mdash;</td>';
      } else {
        tbody += `<td><span class="pct">${fmtPct(r.pct[i])}</span></td>`;
      }
    }
    tbody += '</tr>';
  }
  tbody += `<tr class="total-row"><td>Total</td>${data.totals.map(t => `<td>${fmtShares(t)}</td>`).join('')}</tr>`;
  $('capTable').querySelector('thead').innerHTML = thead;
  $('capTable').querySelector('tbody').innerHTML = tbody;
}

function renderShareTable(data) {
  const thead = `<tr><th>Stakeholder</th>${data.stages.map(s => `<th>${s}</th>`).join('')}</tr>`;
  let tbody = '';
  for (const r of data.rows) {
    tbody += '<tr>';
    tbody += `<td>${r.name}</td>`;
    for (let i = 0; i < data.stages.length; i++) {
      if (r.shares[i] === 0) {
        tbody += '<td style="color:#cbd5e1">&mdash;</td>';
      } else {
        tbody += `<td><span class="shares">${fmtShares(r.shares[i])}</span><br><span class="pct">${fmtPct(r.pct[i])}</span></td>`;
      }
    }
    tbody += '</tr>';
  }
  tbody += `<tr class="total-row"><td>Total</td>${data.totals.map(t => `<td>${fmtShares(t)}</td>`).join('')}</tr>`;
  $('shareTable').querySelector('thead').innerHTML = thead;
  $('shareTable').querySelector('tbody').innerHTML = tbody;
}

function renderCharts(data) {
  const categoryColors = { founder: '#6366f1', pool: '#f59e0b', seed: '#22c55e', seriesA: '#3b82f6', seriesB: '#8b5cf6' };
  // For multiple seed rows, vary the green
  const seedGreens = ['#22c55e','#16a34a','#15803d','#166534','#14532d'];
  let seedIdx = 0;
  const datasets = data.rows.map(r => {
    let bg = categoryColors[r.category] || '#94a3b8';
    if (r.category === 'seed') { bg = seedGreens[seedIdx % seedGreens.length]; seedIdx++; }
    return {
      label: r.name,
      data: r.pct.map(p => +(p * 100).toFixed(1)),
      backgroundColor: bg,
    };
  });

  if (ownershipChart) ownershipChart.destroy();
  ownershipChart = new Chart($('ownershipChart'), {
    type: 'bar',
    data: { labels: data.stages, datasets },
    options: {
      responsive: true, maintainAspectRatio: false,
      scales: { x: { stacked: true }, y: { stacked: true, max: 100, ticks: { callback: v => v + '%' } } },
      plugins: {
        tooltip: { callbacks: { label: ctx => ctx.dataset.label + ': ' + ctx.parsed.y.toFixed(1) + '%' } },
        legend: { position: 'bottom', labels: { boxWidth: 12, padding: 8, font: { size: 11 } } }
      }
    }
  });

  // Valuation chart
  const valStages = ['Post-Seed', 'Post-A', 'Post-B'];
  const valData = [data.seedEffectiveVal || 0, data.aPostMoney, data.bPostMoney];
  const ppsData = [data.seedPPS || 0, data.aPPS, data.bPPS];

  if (valuationChart) valuationChart.destroy();
  valuationChart = new Chart($('valuationChart'), {
    type: 'line',
    data: {
      labels: valStages,
      datasets: [
        { label: 'Post-Money Valuation', data: valData, borderColor: '#6366f1', backgroundColor: 'rgba(99,102,241,0.1)', fill: true, yAxisID: 'y', tension: 0.3 },
        { label: 'Price per Share', data: ppsData, borderColor: '#f59e0b', backgroundColor: 'rgba(245,158,11,0.1)', fill: false, yAxisID: 'y1', tension: 0.3 }
      ]
    },
    options: {
      responsive: true, maintainAspectRatio: false,
      scales: {
        y: { type: 'linear', position: 'left', ticks: { callback: v => fmt(v) } },
        y1: { type: 'linear', position: 'right', grid: { drawOnChartArea: false }, ticks: { callback: v => '$' + v.toFixed(2) } }
      },
      plugins: {
        legend: { position: 'bottom', labels: { boxWidth: 12, padding: 8 } },
        tooltip: { callbacks: { label: ctx => ctx.dataset.label + ': ' + (ctx.datasetIndex === 0 ? fmt(ctx.parsed.y) : '$' + ctx.parsed.y.toFixed(4)) } }
      }
    }
  });
}

// ───────────────── EXPORT: EXCEL ─────────────────
function exportExcel() {
  const inp = getInputs();
  const data = calculate(inp);
  const wb = XLSX.utils.book_new();

  const assumptions = [
    ['Cap Table Assumptions', ''],
    ['', ''],
    ['Founding', ''],
    ['Founders', inp.founders],
    ['Authorized Shares', inp.authShares],
    ['Initial Option Pool', inp.initPool],
    ['', ''],
  ];
  for (const s of safes) {
    assumptions.push([s.name + ' (' + (s.isYC ? 'YC ' : '') + s.safeType.toUpperCase() + ' ' + s.type.toUpperCase() + ')', '']);
    assumptions.push(['Amount', s.amount]);
    assumptions.push(['Valuation Cap', s.uncapped ? 'Uncapped (MFN)' : s.valCap]);
    assumptions.push(['Discount', s.discount]);
    if (s.type === 'note') {
      assumptions.push(['Interest Rate', s.interest]);
      assumptions.push(['Term (months)', s.termMonths]);
    }
    assumptions.push(['', '']);
  }
  assumptions.push(['Series A', '']);
  assumptions.push(['Pre-Money Valuation', inp.aPreMoney]);
  assumptions.push(['Amount Raised', inp.aAmount]);
  assumptions.push(['Option Pool (post-money)', inp.aPool]);
  assumptions.push(['', '']);
  assumptions.push(['Series B', '']);
  assumptions.push(['Pre-Money Valuation', inp.bPreMoney]);
  assumptions.push(['Amount Raised', inp.bAmount]);
  assumptions.push(['Option Pool (post-money)', inp.bPool]);
  assumptions.push(['', '']);
  assumptions.push(['Key Metrics', '']);
  assumptions.push(['Total Raised', data.totalRaised]);
  assumptions.push(['Post-B Valuation', data.bPostMoney]);
  assumptions.push(['Founder Dilution', data.founderDilution]);

  const ws1 = XLSX.utils.aoa_to_sheet(assumptions);
  ws1['!cols'] = [{ wch: 32 }, { wch: 18 }];
  XLSX.utils.book_append_sheet(wb, ws1, 'Assumptions');

  const header = ['Stakeholder', ...data.stages.map(s => s + ' (Shares)'), ...data.stages.map(s => s + ' (%)')];
  const capRows = data.rows.map(r => [r.name, ...r.shares, ...r.pct]);
  capRows.push(['Total', ...data.totals, ...data.totals.map(() => 1)]);
  const ws2 = XLSX.utils.aoa_to_sheet([header, ...capRows]);
  ws2['!cols'] = Array(header.length).fill({ wch: 16 });
  XLSX.utils.book_append_sheet(wb, ws2, 'Cap Table');

  XLSX.writeFile(wb, 'Cap_Table_Model.xlsx');
}

// ───────────────── EXPORT: PDF ─────────────────
function exportPDF() {
  const el = document.querySelector('.container');
  const controls = $('controlPanel');
  const exportBar = document.querySelector('.export-bar');
  controls.style.display = 'none';
  exportBar.style.display = 'none';
  html2canvas(el, { scale: 2, useCORS: true }).then(canvas => {
    controls.style.display = '';
    exportBar.style.display = '';
    const { jsPDF } = window.jspdf;
    const pdf = new jsPDF('p', 'mm', 'a4');
    const pageW = 210, pageH = 297, imgW = pageW - 20;
    const imgH = canvas.height * imgW / canvas.width;
    let y = 10;
    const imgData = canvas.toDataURL('image/jpeg', 0.92);
    while (y < imgH + 10) {
      if (y > 10) pdf.addPage();
      pdf.addImage(imgData, 'JPEG', 10, 10 - (y - 10), imgW, imgH);
      y += pageH - 20;
    }
    pdf.save('Cap_Table_Model.pdf');
  }).catch(() => {
    controls.style.display = '';
    exportBar.style.display = '';
    alert('PDF export failed. Try Cmd/Ctrl + P to print.');
  });
}

// ───────────────── INIT ─────────────────
document.querySelectorAll('input[type=range]').forEach(el => {
  el.addEventListener('input', recalc);
});

// Start with one default SAFE
addSafe({ name: 'Seed SAFE', safeType: 'post', amount: 2000000, valCap: 10000000 });
</script>
</body>
</html>

````

---

## Customization Guide

When the user provides company-specific details, modify the template as follows:

### Company name
Change the `<h1 id="companyName">` text and the `<title>`.

### Founder names
In the `calculate()` function, replace `'Founder ' + (i + 1)` with actual names.
Or, better: add a `founderNames` array at the top of the script and reference it.

### Default values
Change the `value` attributes on the sliders and the corresponding `.val` span text.

### Adding SAFEs programmatically
Call `addSafe({ name, safeType, amount, valCap })` in the init section.
- `safeType`: `'pre'` for pre-money SAFE, `'post'` for post-money SAFE
- Set `uncapped: true` for MFN/uncapped SAFEs
- Set `type: 'note'` with `interest` and `termMonths` for convertible notes

### YC Deal
The YC Deal checkbox adds two fixed-term SAFEs per YC's standard deal:
- $125K post-money SAFE for 7% equity (cap = $1,785,714)
- $375K uncapped MFN SAFE (converts at lowest cap of other SAFEs)

To start with the YC deal enabled, add this to the init section:
```js
$('ycDealToggle').checked = true;
toggleYCDeal(true);
```

### Removing a round
To remove Series B, delete the Series B control section, remove the B columns from
the stages array, and remove B calculations from `calculate()`.

### Colors / branding
Update CSS in the `<style>` block. The main colors:
- Control panel: `#1a1a2e`, `#2d2b55`
- Accent: `#f59e0b` (amber)
- Charts: `#6366f1` (indigo), `#22c55e` (green), `#3b82f6` (blue), `#8b5cf6` (purple)

---

## Deployment

### Local
Just open `cap-table.html` in any browser. No server needed.

### Vercel / Netlify
1. Create a repo with just the HTML file
2. Set build command to empty / output directory to `.`
3. Deploy

### As part of a larger project
Drop `cap-table.html` into the `public/` folder of any web project.

---

## Key Formulas Reference

| Concept | Formula |
|---------|---------|
| Post-money SAFE ownership | `investment / post_money_cap` (guaranteed %) |
| Pre-money SAFE shares | `investment / (cap / existing_shares)` |
| Convertible note accrued | `principal * (1 + rate * term_in_years)` |
| MFN resolution | Converts at lowest valuation cap among non-YC SAFEs |
| Priced round new shares | `amount / (pre_money / existing_shares)` |
| Option pool shuffle | Pool sized as % of post-money, allocated pre-money so existing holders bear dilution |
| Dilution per founder | `1 - (founder_shares / total_shares_post_round)` |
