<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Handheld Gaming Benchmarks — Wulff Den</title>
<link href="https://fonts.googleapis.com/css2?family=Barlow+Condensed:wght@400;600;700;800;900&family=Barlow:wght@400;500;600&display=swap" rel="stylesheet">
<style>
  :root {
    --bg: #000000;
    --surface: #161922;
    --surface2: #1e222d;
    --border: #2a2f3d;
    --win-accent: #00c2ff;
    --win-glow: rgba(0,194,255,0.18);
    --android-accent: #3ddc84;
    --android-glow: rgba(61,220,132,0.18);
    --text: #e8eaf0;
    --text-muted: #6b7280;
    --text-dim: #9ca3af;
    --highlight: #ff9f1c;
  }
  * { box-sizing: border-box; margin: 0; padding: 0; }
  body {
    background: var(--bg);
    color: var(--text);
    font-family: 'Barlow', sans-serif;
    min-height: 100vh;
    overflow-x: hidden;
  }
  body::before { display: none; }

  /* ── Film grain ─────────────────────────────────────────────────────────── */
  #grain {
    position: fixed; inset: 0; z-index: 9999;
    pointer-events: none; opacity: 0.055; will-change: transform;
  }

  /* ── Bloom wrap ──────────────────────────────────────────────────────────── */
  .bloom-wrap { filter: url(#bloom-filter); }

  .wrapper {
    position: relative; z-index: 1;
    max-width: 1300px; margin: 0 auto;
    padding: 40px 24px 80px;
  }

  header { margin-bottom: 40px; }
  .header-eyebrow {
    font-family: 'Barlow Condensed', sans-serif;
    font-size: 11px; font-weight: 700;
    letter-spacing: 4px; text-transform: uppercase;
    color: var(--text-muted); margin-bottom: 8px;
  }
  h1 {
    font-family: 'Barlow Condensed', sans-serif;
    font-size: clamp(32px, 5vw, 56px);
    font-weight: 900; text-transform: uppercase; line-height: 1; color: #fff;
  }
  h1 span { transition: color 0.4s; color: var(--win-accent); }
  .header-sub { font-size: 14px; color: var(--text-muted); margin-top: 10px; }

  /* ── Tabs ─────────────────────────────────────────────────────────────────── */
  .tab-bar {
    display: flex; gap: 4px; margin-bottom: 32px;
    background: var(--surface); padding: 4px;
    border-radius: 10px; border: 1px solid var(--border);
    width: fit-content;
  }
  .tab-btn {
    font-family: 'Barlow Condensed', sans-serif;
    font-size: 15px; font-weight: 800; letter-spacing: 2px;
    text-transform: uppercase; padding: 10px 28px;
    border: none; border-radius: 7px; cursor: pointer;
    background: transparent; color: var(--text-muted); transition: all 0.25s;
  }
  .tab-btn.win-tab.active,     .tab-btn.win-tab:hover     { color: var(--win-accent);     background: var(--win-glow); }
  .tab-btn.android-tab.active, .tab-btn.android-tab:hover { color: var(--android-accent); background: var(--android-glow); }

  /* ── Controls ──────────────────────────────────────────────────────────────── */
  .controls-row { display: flex; align-items: flex-start; flex-wrap: wrap; gap: 24px; margin-bottom: 28px; }
  .control-group { display: flex; flex-direction: column; }
  .section-label {
    font-family: 'Barlow Condensed', sans-serif;
    font-size: 11px; font-weight: 700;
    letter-spacing: 3px; text-transform: uppercase;
    color: var(--text-muted); margin-bottom: 8px;
  }
  .device-selector { display: flex; flex-wrap: wrap; gap: 6px; max-width: 1100px; }
  .device-chip {
    font-family: 'Barlow Condensed', sans-serif;
    font-size: 12px; font-weight: 700; letter-spacing: 0.5px;
    padding: 5px 12px; border-radius: 20px;
    border: 1px solid var(--border); background: var(--surface);
    color: var(--text-dim); cursor: pointer;
    transition: all 0.18s; user-select: none; position: relative;
  }
  .device-chip.highlighted::after {
    content: '★'; font-size: 9px;
    position: absolute; top: -4px; right: -3px;
    color: var(--highlight); text-shadow: 0 0 6px var(--highlight);
  }
  #tab-windows .device-chip.active  { background: var(--win-accent);     border-color: var(--win-accent);     color: #000; }
  #tab-android .device-chip.active  { background: var(--android-accent); border-color: var(--android-accent); color: #000; }
  .device-chip.highlighted { border-color: var(--highlight) !important; box-shadow: 0 0 8px rgba(255,159,28,0.4); }

  .selector-actions { display: flex; gap: 8px; margin-top: 8px; }
  .action-btn {
    font-family: 'Barlow Condensed', sans-serif;
    font-size: 11px; font-weight: 700; letter-spacing: 1px; text-transform: uppercase;
    padding: 4px 12px; border: 1px solid var(--border); border-radius: 4px;
    background: transparent; color: var(--text-muted); cursor: pointer; transition: all 0.18s;
  }
  #tab-windows .action-btn:hover { border-color: var(--win-accent);     color: var(--win-accent); }
  #tab-android .action-btn:hover { border-color: var(--android-accent); color: var(--android-accent); }
  .highlight-hint {
    font-family: 'Barlow Condensed', sans-serif;
    font-size: 10px; letter-spacing: 1.5px; text-transform: uppercase;
    color: var(--text-muted); margin-top: 10px;
    display: flex; align-items: center; gap: 6px;
  }
  .highlight-hint span { color: var(--highlight); text-shadow: 0 0 6px rgba(255,159,28,0.5); }

  /* ── Chart panel ───────────────────────────────────────────────────────────── */
  .chart-panel {
    background: var(--surface); border: 1px solid var(--border);
    border-radius: 12px; padding: 36px 32px 28px;
    position: relative; overflow: hidden;
  }
  .chart-panel::before {
    content: ''; position: absolute; top: 0; left: 0; right: 0; height: 3px;
  }
  .win-panel::before     { background: linear-gradient(90deg, var(--win-accent),     transparent); }
  .android-panel::before { background: linear-gradient(90deg, var(--android-accent), transparent); }

  .chart-heading {
    font-family: 'Barlow Condensed', sans-serif;
    font-size: 22px; font-weight: 900; text-transform: uppercase;
    letter-spacing: 1px; color: #fff; margin-bottom: 4px;
  }
  .chart-subheading { font-size: 12px; color: var(--text-muted); margin-bottom: 32px; }

  /* ── Bar rows ──────────────────────────────────────────────────────────────── */
  .bar-row {
    display: grid;
    grid-template-columns: 220px 1fr;
    align-items: center; gap: 14px; margin-bottom: 11px;
    opacity: 0; transform: translateX(-12px);
    animation: rowIn 0.4s ease forwards;
    transition: opacity 0.2s;
  }
  @keyframes rowIn { to { opacity: 1; transform: translateX(0); } }

  .bar-row.dimmed        { opacity: 0.28 !important; }
  .bar-row.is-highlighted { opacity: 1  !important; }

  .bar-label {
    font-family: 'Barlow Condensed', sans-serif;
    font-size: 13px; font-weight: 700; color: var(--text-dim);
    text-align: right; white-space: nowrap;
    overflow: hidden; text-overflow: ellipsis; transition: color 0.2s;
  }
  .bar-row.is-highlighted .bar-label {
    color: var(--highlight);
    text-shadow: 0 0 12px rgba(255,159,28,0.6);
  }

  .bar-track {
    height: 30px; background: var(--surface2);
    border-radius: 5px;
    /* NO overflow:hidden — lets the value sit at the very tip */
    position: relative;
  }

  .bar-fill {
    height: 100%; border-radius: 5px;
    width: 0%;
    transition: width 1s cubic-bezier(0.16, 1, 0.3, 1);
    position: relative;
    /* clip the colored bar itself but not its children */
    overflow: visible;
  }
  /* right-edge highlight line */
  .bar-fill::after {
    content: ''; position: absolute;
    top: 0; right: 0; width: 2px; height: 100%;
    background: rgba(255,255,255,0.3); border-radius: 0 5px 5px 0;
  }

  /* The value rides on the inside-right tip of the bar */
  .bar-value {
    font-family: 'Barlow Condensed', sans-serif;
    font-size: 14px; font-weight: 800; letter-spacing: 0.5px;
    color: rgba(0,0,0,0.8);
    position: absolute;
    right: 8px;         /* padding from the right edge of bar-fill */
    top: 50%;
    transform: translateY(-50%);
    white-space: nowrap;
    pointer-events: none;
    text-shadow: 0 1px 2px rgba(0,0,0,0.15);
    transition: text-shadow 0.2s;
  }
  .bar-row.is-highlighted .bar-value {
    text-shadow: 0 0 10px rgba(255,159,28,0.5);
  }

  /* ── Status / loading ──────────────────────────────────────────────────────── */
  .status-msg {
    display: flex; flex-direction: column; align-items: center;
    justify-content: center; padding: 80px 20px; gap: 16px;
    color: var(--text-muted); font-family: 'Barlow Condensed', sans-serif;
    font-size: 13px; letter-spacing: 2px; text-transform: uppercase;
    text-align: center; line-height: 1.8;
  }
  .spinner {
    width: 32px; height: 32px; border: 2px solid var(--border);
    border-radius: 50%; animation: spin 0.8s linear infinite;
  }
  .spinner.win     { border-top-color: var(--win-accent); }
  .spinner.android { border-top-color: var(--android-accent); }
  @keyframes spin { to { transform: rotate(360deg); } }

  .tab-content { display: none; }
  .tab-content.active { display: block; }

  /* ── Floating play button ──────────────────────────────────────────────────── */
  #play-btn {
    position: fixed; bottom: 32px; right: 32px; z-index: 10000;
    width: 64px; height: 64px; border-radius: 50%;
    border: 2px solid rgba(255,255,255,0.15);
    background: rgba(12,14,20,0.88);
    backdrop-filter: blur(14px); -webkit-backdrop-filter: blur(14px);
    cursor: pointer; display: flex; align-items: center; justify-content: center;
    transition: transform 0.2s, box-shadow 0.2s, border-color 0.2s;
    box-shadow: 0 4px 24px rgba(0,0,0,0.6);
  }
  #play-btn:hover {
    transform: scale(1.1); border-color: rgba(255,255,255,0.35);
    box-shadow: 0 6px 32px rgba(0,0,0,0.6), 0 0 22px var(--play-glow, rgba(0,194,255,0.4));
  }
  #play-btn:active { transform: scale(0.95); }
  #play-btn .btn-icon { width: 26px; height: 26px; fill: #fff; transition: fill 0.2s; pointer-events: none; }
  #play-btn:hover .btn-icon { fill: var(--play-color, var(--win-accent)); }
  #play-btn .btn-label {
    position: absolute; bottom: -22px; left: 50%; transform: translateX(-50%);
    font-family: 'Barlow Condensed', sans-serif;
    font-size: 10px; font-weight: 700; letter-spacing: 2px; text-transform: uppercase;
    color: var(--text-muted); white-space: nowrap; pointer-events: none; transition: color 0.2s;
  }
  #play-btn:hover .btn-label { color: var(--play-color, var(--win-accent)); }
  #play-btn.playing {
    border-color: var(--play-color, var(--win-accent));
    animation: playPulse 0.7s ease infinite alternate;
  }
  @keyframes playPulse {
    from { box-shadow: 0 0 10px var(--play-glow, rgba(0,194,255,0.3)); }
    to   { box-shadow: 0 0 30px var(--play-glow, rgba(0,194,255,0.7)); }
  }

  @media (max-width: 640px) {
    .bar-row { grid-template-columns: 130px 1fr; }
    .chart-panel { padding: 20px 14px 16px; }
    #play-btn { width: 52px; height: 52px; bottom: 20px; right: 20px; }
    #play-btn .btn-label { display: none; }
  }
</style>
</head>
<body>

<!-- ── Hidden SVG bloom filter ─────────────────────────────────────────────── -->
<svg width="0" height="0" style="position:absolute">
  <defs>
    <filter id="bloom-filter" x="-10%" y="-10%" width="120%" height="120%" color-interpolation-filters="sRGB">
      <feColorMatrix type="saturate" values="1.07" result="sat"/>
      <feGaussianBlur in="sat" stdDeviation="3.5" result="blur1"/>
      <feGaussianBlur in="sat" stdDeviation="9"   result="blur2"/>
      <feBlend in="blur1" in2="blur2" mode="screen" result="blended"/>
      <feComposite in="blended" in2="blended" operator="arithmetic" k1="0" k2="0.5" k3="0" k4="0" result="halfBloom"/>
      <feBlend in="sat" in2="halfBloom" mode="screen" result="bloomed"/>
      <feColorMatrix in="bloomed" type="matrix"
        values="1.02 0.005 0.005 0 0.009
                0.005 1.02 0.005 0 0.009
                0.005 0.005 1.02 0 0.009
                0 0 0 1 0" result="final"/>
      <feComponentTransfer in="final">
        <feFuncR type="linear" slope="0.965" intercept="0.0125"/>
        <feFuncG type="linear" slope="0.965" intercept="0.0125"/>
        <feFuncB type="linear" slope="0.965" intercept="0.0125"/>
      </feComponentTransfer>
    </filter>
  </defs>
</svg>

<!-- ── Animated film grain ───────────────────────────────────────────────────── -->
<canvas id="grain"></canvas>

<div class="bloom-wrap">
<div class="wrapper">
  <header>
    <div class="header-eyebrow">Wulff Den — 3DMark Performance Tests</div>
    <h1>Handheld Gaming <span id="header-accent">Benchmarks</span></h1>
    <p class="header-sub">Live data from Google Sheets · Time Spy Average · Select devices to compare</p>
  </header>

  <div class="tab-bar">
    <button class="tab-btn win-tab active" onclick="switchTab('windows')">⊞ Windows</button>
    <button class="tab-btn android-tab" onclick="switchTab('android')">◎ Android</button>
  </div>

  <!-- Windows Tab -->
  <div class="tab-content active" id="tab-windows">
    <div class="controls-row">
      <div class="control-group">
        <div class="section-label">Filter Devices</div>
        <div class="device-selector" id="win-chips"></div>
        <div class="selector-actions">
          <button class="action-btn" onclick="selectAll('windows')">All</button>
          <button class="action-btn" onclick="selectNone('windows')">None</button>
          <button class="action-btn" onclick="clearHighlights('windows')">Clear Highlights</button>
        </div>
        <div class="highlight-hint"><span>★</span> Right-click a device to highlight it</div>
      </div>
    </div>
    <div class="chart-panel win-panel" id="win-chart-panel">
      <div class="status-msg" id="win-loading"><div class="spinner win"></div>Loading benchmark data…</div>
    </div>
  </div>

  <!-- Android Tab -->
  <div class="tab-content" id="tab-android">
    <div class="controls-row">
      <div class="control-group">
        <div class="section-label">Filter Devices</div>
        <div class="device-selector" id="android-chips"></div>
        <div class="selector-actions">
          <button class="action-btn" onclick="selectAll('android')">All</button>
          <button class="action-btn" onclick="selectNone('android')">None</button>
          <button class="action-btn" onclick="clearHighlights('android')">Clear Highlights</button>
        </div>
        <div class="highlight-hint"><span>★</span> Right-click a device to highlight it</div>
      </div>
    </div>
    <div class="chart-panel android-panel" id="android-chart-panel">
      <div class="status-msg" id="android-loading"><div class="spinner android"></div>Loading benchmark data…</div>
    </div>
  </div>
</div>
</div><!-- end bloom-wrap -->

<!-- ── Floating play button ───────────────────────────────────────────────────── -->
<button id="play-btn" onclick="replayAnimation()" title="Replay animation">
  <svg class="btn-icon" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
    <path d="M8 5v14l11-7z"/>
  </svg>
  <span class="btn-label">Replay</span>
</button>

<script>
// ── Sheet URLs ──────────────────────────────────────────────────────────────
const PUB_BASE    = 'https://docs.google.com/spreadsheets/d/e/2PACX-1vQBAd926glCKt96V0vcwLhk2MX9y-skPq3v2kO0HvBKXeP6Oxgbwv94rNSKcrGYGl11oSpQ5NmrqPbY/pub';
const WIN_URL     = PUB_BASE + '?gid=0&single=true&output=csv';
const ANDROID_URL = PUB_BASE + '?gid=1032283982&single=true&output=csv';

// ── State ───────────────────────────────────────────────────────────────────
let winData = null, androidData = null;
let winSelected      = new Set(), androidSelected      = new Set();
let winHighlighted   = new Set(), androidHighlighted   = new Set();
let currentPlatform  = 'windows';

// ── Colors ──────────────────────────────────────────────────────────────────
const WIN_COLORS = [
  '#00c2ff','#33cfff','#0099d6','#66d9ff','#0077b3','#00eaff','#4dd9f5',
  '#007fa8','#00b3e6','#aaeeff','#0055aa','#3399cc','#1188bb','#66bbdd',
  '#0044aa','#22aadd','#88ccee','#3377aa','#55aacc','#99ddee',
];
const ANDROID_COLORS = [
  '#3ddc84','#5de69a','#29b36a','#a3f0c4','#1f9956','#6ef0a8','#82e8b6',
  '#138847','#0d6633','#aaedcc','#3bc47a','#55d890','#27a060','#1b8049',
  '#7ae8ae','#107040','#44cc80','#9ef2c8','#2aaa68','#66dda0',
];
const HIGHLIGHT_COLOR = '#ff9f1c';

// ── Film grain ───────────────────────────────────────────────────────────────
(function initGrain() {
  const canvas = document.getElementById('grain');
  const ctx    = canvas.getContext('2d');
  let w, h, frame = 0;
  function resize() { w = canvas.width = window.innerWidth; h = canvas.height = window.innerHeight; }
  window.addEventListener('resize', resize);
  resize();
  function drawGrain() {
    frame++;
    if (frame % 2 === 0) {
      const img = ctx.createImageData(w, h);
      const d   = img.data;
      for (let i = 0; i < d.length; i += 4) {
        const v = (Math.random() * 255) | 0;
        d[i] = d[i+1] = d[i+2] = v; d[i+3] = 255;
      }
      ctx.putImageData(img, 0, 0);
    }
    requestAnimationFrame(drawGrain);
  }
  drawGrain();
})();

// ── Fetch with proxy fallback ────────────────────────────────────────────────
async function fetchCSV(url) {
  const proxies = [
    u => 'https://api.allorigins.win/raw?url=' + encodeURIComponent(u),
    u => 'https://api.codetabs.com/v1/proxy?quest=' + encodeURIComponent(u),
    u => 'https://corsproxy.io/?url=' + encodeURIComponent(u),
    u => u,
  ];
  for (const make of proxies) {
    try {
      const resp = await Promise.race([
        fetch(make(url)),
        new Promise((_, r) => setTimeout(() => r(new Error('timeout')), 8000))
      ]);
      if (!resp.ok) continue;
      const text = await resp.text();
      if (!text || text.trim().startsWith('<') || text.trim().startsWith('{')) continue;
      return text;
    } catch(e) { /* try next */ }
  }
  throw new Error('All fetch attempts failed.');
}

// ── CSV parser ───────────────────────────────────────────────────────────────
function parseCSV(text) {
  const lines = text.split('\n').map(l => l.trim()).filter(Boolean);
  return lines.map(line => {
    const cells = []; let inQ = false, cur = '';
    for (const c of line) {
      if (c === '"') { inQ = !inQ; }
      else if (c === ',' && !inQ) { cells.push(cur.trim()); cur = ''; }
      else cur += c;
    }
    cells.push(cur.trim());
    return cells;
  });
}

// ── Data parser ──────────────────────────────────────────────────────────────
function parseData(rows) {
  const header = rows[0].map(h => h.trim());
  let avgCol = -1;
  for (let c = 0; c < header.length; c++) {
    const h = header[c].toLowerCase();
    if (h.includes('time spy') && h.includes('average')) { avgCol = c; break; }
  }
  if (avgCol === -1) for (let c = 0; c < header.length; c++) {
    if (header[c].toLowerCase().includes('time spy')) { avgCol = c; break; }
  }
  if (avgCol === -1) for (let c = 0; c < header.length; c++) {
    if (header[c].toLowerCase().includes('average')) { avgCol = c; break; }
  }
  if (avgCol === -1) throw new Error('Could not find "Time Spy Average" column. Headers: ' + header.join(', '));
  const entries = [];
  for (let r = 1; r < rows.length; r++) {
    const row = rows[r];
    const device = (row[0] || '').trim();
    if (!device) continue;
    const score = parseFloat((row[avgCol] || '').replace(/,/g, '').trim());
    if (!isNaN(score) && score > 0) entries.push({ name: device, score });
  }
  if (entries.length === 0) throw new Error('No valid data rows found.');
  entries.sort((a, b) => b.score - a.score);
  return entries;
}

// ── Chips ─────────────────────────────────────────────────────────────────────
function renderChips(platform) {
  const data        = platform === 'windows' ? winData : androidData;
  const selected    = platform === 'windows' ? winSelected    : androidSelected;
  const highlighted = platform === 'windows' ? winHighlighted : androidHighlighted;
  const el          = document.getElementById(platform === 'windows' ? 'win-chips' : 'android-chips');
  el.innerHTML      = '';
  data.forEach(({ name }) => {
    const chip = document.createElement('button');
    let cls = 'device-chip';
    if (selected.has(name))    cls += ' active';
    if (highlighted.has(name)) cls += ' highlighted';
    chip.className   = cls;
    chip.textContent = name;
    chip.title       = 'Left-click to show/hide · Right-click to highlight';
    chip.onclick = (e) => {
      e.preventDefault();
      selected.has(name) ? selected.delete(name) : selected.add(name);
      renderChips(platform); renderChart(platform);
    };
    chip.oncontextmenu = (e) => {
      e.preventDefault();
      if (!highlighted.has(name)) { highlighted.add(name); selected.add(name); }
      else { highlighted.delete(name); }
      renderChips(platform); renderChart(platform);
    };
    el.appendChild(chip);
  });
}

function selectAll(p) {
  const data = p === 'windows' ? winData : androidData;
  const sel  = p === 'windows' ? winSelected : androidSelected;
  data.forEach(({ name }) => sel.add(name));
  renderChips(p); renderChart(p);
}
function selectNone(p) {
  (p === 'windows' ? winSelected : androidSelected).clear();
  renderChips(p); renderChart(p);
}
function clearHighlights(p) {
  (p === 'windows' ? winHighlighted : androidHighlighted).clear();
  renderChips(p); renderChart(p);
}

// ── Number counter animation ─────────────────────────────────────────────────
function animateCounters(panel, duration = 950) {
  panel.querySelectorAll('.bar-value[data-target]').forEach(el => {
    const target  = parseFloat(el.dataset.target);
    const start   = performance.now();
    function tick(now) {
      const t = Math.min((now - start) / duration, 1);
      // same easing as the bar: cubic-bezier(0.16, 1, 0.3, 1) approximated
      const ease = 1 - Math.pow(1 - t, 3);
      el.textContent = Math.round(ease * target).toLocaleString();
      if (t < 1) requestAnimationFrame(tick);
      else el.textContent = Math.round(target).toLocaleString();
    }
    requestAnimationFrame(tick);
  });
}

// ── Chart ─────────────────────────────────────────────────────────────────────
function renderChart(platform, animate = false) {
  const data        = platform === 'windows' ? winData        : androidData;
  const selected    = platform === 'windows' ? winSelected    : androidSelected;
  const highlighted = platform === 'windows' ? winHighlighted : androidHighlighted;
  const panel       = document.getElementById(platform === 'windows' ? 'win-chart-panel' : 'android-chart-panel');
  const colors      = platform === 'windows' ? WIN_COLORS     : ANDROID_COLORS;

  const visible = data.filter(d => selected.has(d.name));
  if (visible.length === 0) {
    panel.innerHTML = `<div class="status-msg">Select at least one device above.</div>`;
    return;
  }

  const anyHL    = visible.some(d => highlighted.has(d.name));
  const maxScore = Math.max(...visible.map(d => d.score), 1);

  let html = `
    <div class="chart-heading">Time Spy Average</div>
    <div class="chart-subheading">Higher is better &nbsp;·&nbsp; Sorted by score (high to low)${anyHL ? ' &nbsp;·&nbsp; <span style="color:var(--highlight)">★ Highlighted</span>' : ''}</div>
  `;

  visible.forEach(({ name, score }, i) => {
    const pct      = (score / maxScore) * 100;
    const isHL     = highlighted.has(name);
    const color    = isHL ? HIGHLIGHT_COLOR : colors[i % colors.length];
    const rowClass = anyHL ? (isHL ? 'bar-row is-highlighted' : 'bar-row dimmed') : 'bar-row';
    const barGrad  = isHL
      ? `linear-gradient(90deg, ${HIGHLIGHT_COLOR}88, ${HIGHLIGHT_COLOR})`
      : `linear-gradient(90deg, ${color}77, ${color})`;
    const delay    = animate ? i * 45 : 0;
    // Start counter at 0 if animating, else show final value immediately
    const initVal  = animate ? '0' : Math.round(score).toLocaleString();

    html += `
      <div class="${rowClass}" style="animation-delay:${delay}ms">
        <div class="bar-label" title="${esc(name)}">${isHL ? '★ ' : ''}${esc(name)}</div>
        <div class="bar-track">
          <div class="bar-fill" data-pct="${pct}" style="background:${barGrad}">
            <span class="bar-value" data-target="${score}">${initVal}</span>
          </div>
        </div>
      </div>`;
  });

  panel.innerHTML = html;

  // Trigger bar grow + counter animate
  requestAnimationFrame(() => requestAnimationFrame(() => {
    panel.querySelectorAll('.bar-fill').forEach(el => { el.style.width = el.dataset.pct + '%'; });
    if (animate) animateCounters(panel);
  }));
}

// ── Tab switch ────────────────────────────────────────────────────────────────
function switchTab(platform) {
  currentPlatform = platform;
  document.querySelectorAll('.tab-content').forEach(el => el.classList.remove('active'));
  document.querySelectorAll('.tab-btn').forEach(el => el.classList.remove('active'));
  document.getElementById('tab-' + platform).classList.add('active');
  document.querySelector('.' + (platform === 'windows' ? 'win' : 'android') + '-tab').classList.add('active');
  document.querySelector('h1 span').style.color =
    platform === 'windows' ? 'var(--win-accent)' : 'var(--android-accent)';
  // Update play button glow colour
  const btn = document.getElementById('play-btn');
  if (platform === 'windows') {
    btn.style.setProperty('--play-color', '#00c2ff');
    btn.style.setProperty('--play-glow',  'rgba(0,194,255,0.4)');
  } else {
    btn.style.setProperty('--play-color', '#3ddc84');
    btn.style.setProperty('--play-glow',  'rgba(61,220,132,0.4)');
  }
}

// ── Replay animation ──────────────────────────────────────────────────────────
function replayAnimation() {
  const btn      = document.getElementById('play-btn');
  const platform = currentPlatform;
  const panel    = document.getElementById(platform === 'windows' ? 'win-chart-panel' : 'android-chart-panel');

  // Collapse bars and reset counters to 0
  panel.querySelectorAll('.bar-fill').forEach(el => {
    el.style.transition = 'none';
    el.style.width = '0%';
  });
  panel.querySelectorAll('.bar-value').forEach(el => { el.textContent = '0'; });
  panel.querySelectorAll('.bar-row').forEach(el => {
    el.style.animation = 'none';
    el.style.opacity   = '0';
    el.style.transform = 'translateX(-12px)';
  });

  btn.classList.add('playing');
  setTimeout(() => {
    renderChart(platform, true);
    setTimeout(() => btn.classList.remove('playing'), 1200);
  }, 80);
}

// ── Error display ─────────────────────────────────────────────────────────────
function showError(id, msg) {
  document.getElementById(id).innerHTML =
    `<div style="color:#f87171;font-family:'Barlow Condensed',sans-serif;text-align:center;padding:20px">
      <div style="font-size:28px;margin-bottom:10px">⚠</div>
      <div style="font-size:13px;letter-spacing:1px;margin-bottom:8px">FAILED TO LOAD DATA</div>
      <div style="font-size:11px;color:var(--text-muted);max-width:500px;margin:0 auto;
                  line-height:1.8;text-transform:none;letter-spacing:0">${msg}</div>
    </div>`;
}

function esc(s) {
  return String(s).replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;').replace(/"/g,'&quot;');
}

// ── Init ──────────────────────────────────────────────────────────────────────
async function init() {
  try {
    const text = await fetchCSV(WIN_URL);
    winData = parseData(parseCSV(text));
    winData.forEach(({ name }) => winSelected.add(name));
    renderChips('windows'); renderChart('windows');
  } catch(e) { console.error('Windows:', e); showError('win-loading', e.message); }

  try {
    const text = await fetchCSV(ANDROID_URL);
    androidData = parseData(parseCSV(text));
    androidData.forEach(({ name }) => androidSelected.add(name));
    renderChips('android'); renderChart('android');
  } catch(e) { console.error('Android:', e); showError('android-loading', e.message); }
}

init();
</script>
</body>
</html>

