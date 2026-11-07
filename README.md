<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1,viewport-fit=cover">
<meta name="theme-color" content="#006b54">
<meta name="apple-mobile-web-app-capable" content="yes">
<title>Guidepost Navigator</title>
<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css">
<link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Overpass:wght@400;600;800&family=Overpass+Mono:wght@600;700&display=swap">
<style>
/* Layout: full-bleed map; road-sign banner on top; driving cluster bottom-left; actions bottom-right; bottom sheet. */
:root{
  --bg:#e9eeeb; --panel:#ffffff; --panel-2:#f1f5f3; --ink:#13201b; --muted:#5b6a63; --line:#d3dcd8;
  --guide:#006b54; --guide-ink:#ffffff; --warn:#ffc20e; --warn-ink:#1a1a1a; --danger:#c8312b;
  --route:#1a6fe0; --route-alt:#8fa7c6; --casing:#ffffff;
  --font:'Overpass',system-ui,-apple-system,'Segoe UI',sans-serif;
  --mono:'Overpass Mono',ui-monospace,'SF Mono',Menlo,monospace;
  --bottom:180px; --shadow:0 6px 22px rgba(10,25,20,.22);
}
@media (prefers-color-scheme: dark){:root:not([data-theme="light"]){
  --bg:#0d1311; --panel:#18211e; --panel-2:#212c28; --ink:#e6eeea; --muted:#93a39c; --line:#2b3733;
  --guide:#00785e; --route:#4aa3ff; --route-alt:#4c5f78; --casing:#0b1a2e; color-scheme:dark; --shadow:0 6px 22px rgba(0,0,0,.5);
}}
:root[data-theme="dark"]{
  --bg:#0d1311; --panel:#18211e; --panel-2:#212c28; --ink:#e6eeea; --muted:#93a39c; --line:#2b3733;
  --guide:#00785e; --route:#4aa3ff; --route-alt:#4c5f78; --casing:#0b1a2e; color-scheme:dark; --shadow:0 6px 22px rgba(0,0,0,.5);
}
*{box-sizing:border-box}
html,body{height:100%;margin:0}
body{background:var(--bg);color:var(--ink);font-family:var(--font);font-size:15px;overflow:hidden;-webkit-tap-highlight-color:transparent;-webkit-font-smoothing:antialiased}
button{font:inherit;color:inherit;cursor:pointer;border:0;background:none}
button:focus-visible,input:focus-visible{outline:3px solid var(--route);outline-offset:2px}
#map{position:fixed;inset:0;z-index:0;background:var(--bg)}
#map.night .leaflet-tile-pane{filter:invert(1) hue-rotate(180deg) brightness(.9) contrast(.88) saturate(.7)}
.leaflet-bottom{bottom:var(--bottom)!important;transition:bottom .2s}
.leaflet-control-attribution{font-size:10px;background:color-mix(in srgb,var(--panel) 80%,transparent)!important;color:var(--muted)}
.leaflet-control-attribution a{color:var(--muted)}

/* Top */
#top{position:fixed;top:0;left:0;right:0;z-index:600;max-width:580px;margin:0 auto;display:flex;flex-direction:column;gap:8px;pointer-events:none;
  padding:calc(env(safe-area-inset-top,0px) + 10px) max(12px,env(safe-area-inset-right,0px)) 0 max(12px,env(safe-area-inset-left,0px))}
#top>*{pointer-events:auto}
#searchbar{display:flex;align-items:center;gap:10px;background:var(--panel);border-radius:14px;box-shadow:var(--shadow);padding:0 8px 0 14px;height:52px}
#searchbar svg{width:20px;height:20px;color:var(--muted);flex:none}
#q{flex:1;min-width:0;border:0;background:none;color:var(--ink);font:600 17px var(--font);height:100%;outline:none}
#q::placeholder{color:var(--muted);font-weight:400}
.icon-btn{width:40px;height:40px;border-radius:10px;display:grid;place-items:center;font-size:22px;color:var(--muted);flex:none}
.icon-btn:hover{background:var(--panel-2)}

#banner{background:var(--guide);color:var(--guide-ink);border-radius:14px;padding:14px 16px;
  box-shadow:inset 0 0 0 4px var(--guide),inset 0 0 0 6px rgba(255,255,255,.92),var(--shadow)}
.b-main{display:flex;align-items:center;gap:14px}
.b-icon{width:62px;height:62px;flex:none}
.b-icon svg{width:100%;height:100%}
.b-text{min-width:0;flex:1}
.b-dist{font:700 34px/1 var(--mono);font-variant-numeric:tabular-nums;letter-spacing:-.02em}
.b-dist small{font:800 15px var(--font);margin-left:4px;letter-spacing:.06em;text-transform:uppercase;opacity:.9}
.b-street{font:800 19px/1.2 var(--font);margin-top:4px;overflow:hidden;text-overflow:ellipsis;display:-webkit-box;-webkit-line-clamp:2;-webkit-box-orient:vertical;text-wrap:balance}
.b-then{display:inline-flex;align-items:center;gap:6px;margin-top:10px;padding:4px 10px 4px 12px;border-radius:999px;background:rgba(0,0,0,.22);font:800 12px var(--font);letter-spacing:.08em;text-transform:uppercase}
.b-then svg{width:22px;height:22px}

/* Driving cluster */
#hud{position:fixed;z-index:550;left:max(12px,env(safe-area-inset-left,0px));bottom:calc(var(--bottom) + 12px);display:flex;align-items:flex-end;gap:8px;transition:bottom .2s;pointer-events:none}
.speedo{width:68px;height:68px;border-radius:50%;background:var(--panel);box-shadow:var(--shadow);display:flex;flex-direction:column;align-items:center;justify-content:center;border:3px solid var(--line)}
.speedo b{font:700 26px/1 var(--mono);font-variant-numeric:tabular-nums}
.speedo span{font:800 10px var(--font);letter-spacing:.1em;text-transform:uppercase;color:var(--muted)}
.speedo.over{border-color:var(--danger);color:var(--danger)}
.limit{background:#fff;color:#111;border:2.5px solid #111;border-radius:7px;box-shadow:0 0 0 2px #fff,var(--shadow);width:50px;padding:4px 2px 3px;text-align:center;line-height:1}
.limit span{display:block;font:800 8.5px/1.05 var(--font);letter-spacing:.04em}
.limit b{display:block;font:800 25px/1 var(--font);margin-top:2px}
.limit.metric{width:56px;height:56px;border-radius:50%;border:6px solid #d52b1e;display:grid;place-items:center;padding:0}
.limit.metric span{display:none}
.limit.metric b{margin:0;font-size:22px}
.limit.over{animation:pulse 1s infinite}
@keyframes pulse{50%{box-shadow:0 0 0 2px #fff,0 0 0 8px rgba(200,49,43,.5)}}

/* Actions */
#fabs{position:fixed;z-index:550;right:max(12px,env(safe-area-inset-right,0px));bottom:calc(var(--bottom) + 12px);display:flex;flex-direction:column;gap:10px;transition:bottom .2s}
.fab{width:52px;height:52px;border-radius:50%;background:var(--panel);box-shadow:var(--shadow);display:grid;place-items:center;color:var(--ink)}
.fab svg{width:24px;height:24px}
.fab-warn{background:var(--warn);color:var(--warn-ink);width:58px;height:58px}
.fab-warn svg{width:28px;height:28px}

/* Sheets */
.sheet{position:fixed;left:0;right:0;bottom:0;z-index:600;max-width:580px;margin:0 auto;background:var(--panel);border-radius:20px 20px 0 0;box-shadow:0 -6px 26px rgba(0,0,0,.18);
  padding:6px max(16px,env(safe-area-inset-right,0px)) calc(14px + env(safe-area-inset-bottom,0px)) max(16px,env(safe-area-inset-left,0px));max-height:52vh;overflow-y:auto;overscroll-behavior:contain}
.grab{width:40px;height:5px;border-radius:3px;background:var(--line);margin:4px auto 12px}
.label{font:800 11px var(--font);letter-spacing:.12em;text-transform:uppercase;color:var(--muted);margin:16px 0 4px}
.hint{color:var(--muted);margin:10px 0 4px;line-height:1.45}
.err{color:var(--danger);margin:10px 0;font-weight:600}
.chips{display:grid;grid-template-columns:1fr 1fr;gap:10px}
.chip{display:flex;align-items:center;gap:10px;background:var(--panel-2);border-radius:12px;padding:12px;text-align:left;min-width:0}
.chip svg{width:22px;height:22px;color:var(--guide);flex:none}
.chip span{min-width:0;display:flex;flex-direction:column}
.chip b{font-weight:800}
.chip small{color:var(--muted);white-space:nowrap;overflow:hidden;text-overflow:ellipsis}
.list{list-style:none;margin:0;padding:0}
.row{display:flex;align-items:center;gap:12px;width:100%;padding:11px 2px;text-align:left;border-bottom:1px solid var(--line)}
.list li:last-child .row{border-bottom:0}
.row-ico{width:34px;height:34px;border-radius:50%;background:var(--panel-2);display:grid;place-items:center;flex:none;color:var(--muted)}
.row-ico svg{width:18px;height:18px}
.row-txt{min-width:0;flex:1;display:flex;flex-direction:column;gap:2px}
.row-txt b{font-weight:600;white-space:nowrap;overflow:hidden;text-overflow:ellipsis}
.row-txt small{color:var(--muted);white-space:nowrap;overflow:hidden;text-overflow:ellipsis}
.row-d{font:600 13px var(--mono);color:var(--muted);font-variant-numeric:tabular-nums;flex:none}
.pv-head{display:flex;align-items:flex-start;gap:8px}
.pv-name{font:800 20px/1.2 var(--font)!important;white-space:normal!important;text-wrap:balance}
.routes{list-style:none;margin:14px 0 0;padding:0;display:flex;flex-direction:column;gap:8px}
.route-opt{width:100%;display:grid;grid-template-columns:auto 1fr auto;align-items:baseline;gap:4px 12px;padding:12px 14px;border-radius:12px;border:2px solid var(--line);text-align:left}
.route-opt b{font:700 20px var(--mono);font-variant-numeric:tabular-nums}
.route-opt span{color:var(--muted);min-width:0;overflow:hidden;text-overflow:ellipsis;white-space:nowrap}
.route-opt em{font:800 10px var(--font);font-style:normal;letter-spacing:.1em;text-transform:uppercase;color:var(--guide)}
.route-opt.on{border-color:var(--route);background:color-mix(in srgb,var(--route) 8%,transparent)}
.pv-actions{display:grid;grid-template-columns:2fr 1fr;gap:10px;margin-top:14px}
.btn{height:52px;border-radius:12px;background:var(--panel-2);font-weight:800;font-size:16px}
.btn.primary{background:var(--route);color:#fff;font-size:18px;letter-spacing:.04em}
.btn.danger{background:var(--danger);color:#fff}
.pv-save{display:flex;gap:18px;margin-top:12px}
.link{color:var(--route);font-weight:600;padding:6px 0}

#navbar{display:flex;align-items:center;gap:16px}
.nb-cell{display:flex;flex-direction:column}
.nb-cell b{font:700 24px/1.05 var(--mono);font-variant-numeric:tabular-nums}
.nb-cell span{font:600 12px var(--font);color:var(--muted);letter-spacing:.04em}
.nb-cell.eta b{color:var(--guide)}
.nb-spacer{flex:1}
.pill{height:40px;padding:0 14px;border-radius:999px;background:var(--panel-2);font:700 14px var(--mono)}
#btnEnd{height:48px;padding:0 22px;border-radius:12px;background:var(--danger);color:#fff;font-weight:800;font-size:16px}

/* Modal */
#modal{position:fixed;inset:0;z-index:800;background:rgba(5,12,10,.45);display:flex;align-items:flex-end;justify-content:center}
#modal .sheet{position:relative;max-height:80vh;width:100%}
.rgrid{display:grid;grid-template-columns:repeat(3,1fr);gap:10px;margin-top:4px}
.rbtn{display:flex;flex-direction:column;align-items:center;gap:8px;padding:14px 4px;border-radius:14px;background:var(--panel-2);font-weight:700}
.rbtn i{width:52px;height:52px;border-radius:50%;display:grid;place-items:center;color:#fff}
.rbtn svg{width:26px;height:26px}
.seg{display:flex;background:var(--panel-2);border-radius:10px;padding:3px;gap:3px}
.seg button{flex:1;height:38px;border-radius:8px;font-weight:700;color:var(--muted)}
.seg button.on{background:var(--panel);color:var(--ink);box-shadow:0 1px 4px rgba(0,0,0,.15)}
.set-row{display:flex;flex-direction:column;gap:8px;margin-top:14px}
.set-row>span{font-weight:700}
.fine{font-size:12px;color:var(--muted);line-height:1.5;margin-top:18px}
.fine a{color:var(--muted)}

/* Map markers */
.car{background:none;border:0}
.car-rot{width:44px;height:44px;transition:transform .4s linear}
.car svg{width:100%;height:100%;filter:drop-shadow(0 2px 3px rgba(0,0,0,.4))}
.car .halo{fill:var(--route);opacity:.18}
.car .arrow{fill:var(--route);stroke:#fff;stroke-width:2.5;stroke-linejoin:round}
.pin{background:none;border:0}
.pin svg{width:34px;height:44px;filter:drop-shadow(0 2px 3px rgba(0,0,0,.35))}
.rep{background:none;border:0}
.rep div{width:32px;height:32px;border-radius:50%;display:grid;place-items:center;color:#fff;border:2.5px solid #fff;box-shadow:0 2px 6px rgba(0,0,0,.35)}
.rep svg{width:17px;height:17px}
.leaflet-popup-content-wrapper,.leaflet-popup-tip{background:var(--panel);color:var(--ink)}
.leaflet-popup-content{font-family:var(--font);margin:12px 14px}
.pop b{display:block;font-size:15px}
.pop small{color:var(--muted)}
.pop button{margin-top:8px;color:var(--danger);font-weight:700}

#toast{position:fixed;z-index:900;left:50%;top:calc(env(safe-area-inset-top,0px) + 76px);transform:translateX(-50%);background:var(--ink);color:var(--panel);padding:10px 16px;border-radius:12px;font-weight:600;box-shadow:var(--shadow);max-width:calc(100% - 32px);text-align:center;transition:opacity .25s}
#toast.nav{top:calc(env(safe-area-inset-top,0px) + 150px)}
@media (prefers-reduced-motion: reduce){.car-rot,#hud,#fabs,.leaflet-bottom{transition:none}.limit.over{animation:none}}
[hidden]{display:none!important}
</style>
</head>
<body>
<div id="map"></div>

<header id="top">
  <div id="searchbar">
    <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.4" stroke-linecap="round"><circle cx="10.5" cy="10.5" r="6.5"/><path d="M20 20l-4.8-4.8"/></svg>
    <input id="q" type="search" placeholder="Where to?" autocomplete="off" enterkeyhint="search" aria-label="Search for a destination">
    <button id="clearQ" class="icon-btn" aria-label="Clear search" hidden>×</button>
  </div>
  <div id="banner" hidden aria-live="polite">
    <div class="b-main">
      <div id="bIcon" class="b-icon"></div>
      <div class="b-text"><div id="bDist" class="b-dist"></div><div id="bStreet" class="b-street"></div></div>
    </div>
    <div id="bThen" class="b-then" hidden></div>
  </div>
</header>

<div id="hud">
  <div id="limit" class="limit" hidden><span>SPEED<br>LIMIT</span><b id="limitVal">35</b></div>
  <div id="speedo" class="speedo"><b id="spd">0</b><span id="spdU">mph</span></div>
</div>

<div id="fabs">
  <button id="btnSettings" class="fab" aria-label="Settings"><svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M4 7h10M18 7h2M4 17h4M12 17h8"/><circle cx="16" cy="7" r="2.2"/><circle cx="10" cy="17" r="2.2"/></svg></button>
  <button id="btnRecenter" class="fab" aria-label="Recenter on my location" hidden><svg viewBox="0 0 24 24" fill="currentColor"><path d="M12 2l7.5 19-7.5-4.5L4.5 21z"/></svg></button>
  <button id="btnReport" class="fab fab-warn" aria-label="Report something on the road"><svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.4" stroke-linecap="round" stroke-linejoin="round"><path d="M12 3L2 20h20z"/><path d="M12 10v4M12 17.2v.1"/></svg></button>
</div>

<section id="sheet" class="sheet"></section>

<section id="navbar" class="sheet" hidden>
  <div id="navbarInner" style="display:flex;align-items:center;gap:16px">
    <div class="nb-cell eta"><b id="eta">--:--</b><span>arrival</span></div>
    <div class="nb-cell"><b id="remT">--</b><span id="remD">--</span></div>
    <div class="nb-spacer"></div>
    <button id="btnSim" class="pill" hidden aria-label="Simulation speed">×1</button>
    <button id="btnEnd">End</button>
  </div>
</section>

<div id="modal" hidden><section class="sheet" id="modalSheet"></section></div>
<div id="toast" hidden role="status"></div>

<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
<script>
'use strict';
/* ---------- helpers ---------- */
const $ = s => document.querySelector(s);
const esc = s => String(s ?? '').replace(/[&<>"']/g, c => ({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;'}[c]));
const store = {
  get(k, d){ try{ const v = localStorage.getItem('gp_'+k); return v == null ? d : JSON.parse(v); }catch{ return d; } },
  set(k, v){ try{ localStorage.setItem('gp_'+k, JSON.stringify(v)); }catch{} }
};
const R_E = 6371008.8, rad = d => d*Math.PI/180, deg = r => r*180/Math.PI;
function dist(a, b){
  const dLat = rad(b[0]-a[0]), dLng = rad(b[1]-a[1]);
  const h = Math.sin(dLat/2)**2 + Math.cos(rad(a[0]))*Math.cos(rad(b[0]))*Math.sin(dLng/2)**2;
  return 2*R_E*Math.asin(Math.sqrt(h));
}
function bearing(a, b){
  const y = Math.sin(rad(b[1]-a[1]))*Math.cos(rad(b[0]));
  const x = Math.cos(rad(a[0]))*Math.sin(rad(b[0])) - Math.sin(rad(a[0]))*Math.cos(rad(b[0]))*Math.cos(rad(b[1]-a[1]));
  return (deg(Math.atan2(y, x)) + 360) % 360;
}
const css = v => getComputedStyle(document.documentElement).getPropertyValue(v).trim();

/* ---------- settings & theme ---------- */
const settings = Object.assign({ units:'imperial', voice:true, theme:'auto' }, store.get('settings', {}));
const saveSettings = () => store.set('settings', settings);
const mq = matchMedia('(prefers-color-scheme: dark)');
const isDark = () => settings.theme === 'dark' || (settings.theme === 'auto' && mq.matches);

const map = L.map('map', { zoomControl:false, attributionControl:true, zoomSnap:.25 }).setView([40.7128, -74.006], 13);
const TILE_ATTR = '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors';
let tiles = null;
function applyTheme(){
  if (settings.theme === 'auto') delete document.documentElement.dataset.theme;
  else document.documentElement.dataset.theme = settings.theme;
  map.getContainer().classList.toggle('night', isDark());
  if (!tiles) tiles = L.tileLayer('https://tile.openstreetmap.org/{z}/{x}/{y}.png', { attribution:TILE_ATTR, maxZoom:19 }).addTo(map);
  if (S.nav) drawNavRoute(); else if (S.routes.length) drawRoutes();
}
mq.addEventListener?.('change', () => settings.theme === 'auto' && applyTheme());

/* ---------- formatting ---------- */
const imperial = () => settings.units === 'imperial';
function fmtDist(m){
  if (imperial()){
    const ft = m*3.28084;
    if (ft < 528) return { v:String(Math.max(50, Math.round(ft/50)*50)), u:'ft' };
    const mi = m/1609.344; return { v: mi < 10 ? mi.toFixed(1) : String(Math.round(mi)), u:'mi' };
  }
  if (m < 950) return { v:String(Math.max(10, Math.round(m/10)*10)), u:'m' };
  const km = m/1000; return { v: km < 10 ? km.toFixed(1) : String(Math.round(km)), u:'km' };
}
const fd = m => { const d = fmtDist(m); return d.v+' '+d.u; };
function fmtDur(s){ const m = Math.max(1, Math.round(s/60)); if (m < 60) return m+' min'; return Math.floor(m/60)+' h '+(m%60)+' min'; }
const clock = s => new Date(Date.now()+s*1000).toLocaleTimeString([], { hour:'numeric', minute:'2-digit' });
const speedDisp = mps => Math.round(imperial() ? mps*2.23694 : mps*3.6);
const shortAddr = a => String(a||'').split(',').slice(1,4).join(',').trim();

/* ---------- icons ---------- */
const ico = {
  pin:'<svg viewBox="0 0 24 24" fill="currentColor"><path d="M12 2a7 7 0 0 0-7 7c0 5.2 7 13 7 13s7-7.8 7-13a7 7 0 0 0-7-7zm0 9.5A2.5 2.5 0 1 1 12 6.5a2.5 2.5 0 0 1 0 5z"/></svg>',
  clock:'<svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2" stroke-linecap="round"><circle cx="12" cy="12" r="8.5"/><path d="M12 7.5V12l3 2"/></svg>',
  home:'<svg viewBox="0 0 24 24" fill="currentColor"><path d="M12 3l9 8h-2.5v9h-5v-6h-3v6h-5v-9H3z"/></svg>',
  work:'<svg viewBox="0 0 24 24" fill="currentColor"><path d="M9 4h6a2 2 0 0 1 2 2v1h3a2 2 0 0 1 2 2v9a2 2 0 0 1-2 2H4a2 2 0 0 1-2-2V9a2 2 0 0 1 2-2h3V6a2 2 0 0 1 2-2zm0 3h6V6H9z"/></svg>'
};
const DIR_ANG = { 'straight':0, 'slight right':40, 'right':90, 'sharp right':135, 'slight left':-40, 'left':-90, 'sharp left':-135 };
function turnSVG(step){
  const m = step.maneuver, t = m.type, mod = m.modifier || 'straight';
  const open = '<svg viewBox="0 0 48 48" fill="none" stroke="currentColor" stroke-width="5" stroke-linecap="round" stroke-linejoin="round">';
  if (t === 'arrive') return '<svg viewBox="0 0 48 48"><path d="M24 3c-7.5 0-13 5.6-13 12.8C11 25.5 24 44 24 44s13-18.5 13-28.2C37 8.6 31.5 3 24 3z" fill="currentColor"/><circle cx="24" cy="16" r="5.5" fill="var(--guide)"/></svg>';
  if (t === 'roundabout' || t === 'rotary' || t === 'roundabout turn')
    return open+`<circle cx="24" cy="21" r="7"/><path d="M24 45V28"/><g transform="rotate(${DIR_ANG[mod] ?? 0} 24 21)"><path d="M24 14V4"/><path d="M18 9.5l6-6 6 6"/></g></svg>`;
  if (mod === 'uturn') return open+'<path d="M31 44V19a8.5 8.5 0 0 0-17 0v11"/><path d="M7.5 24l6.5 7.5 6.5-7.5"/></svg>';
  const a = DIR_ANG[mod] ?? 0;
  return open+`<path d="M24 45V27"/><g transform="rotate(${a} 24 27)"><path d="M24 27V7"/><path d="M15.5 15.5L24 7l8.5 8.5"/></g></svg>`;
}
const ord = n => n+(['th','st','nd','rd'][(n%100-20)%10] || ['th','st','nd','rd'][n%100] || 'th');
const CARD = ['north','northeast','east','southeast','south','southwest','west','northwest'];
function instr(step){
  const m = step.maneuver, t = m.type, mod = m.modifier || '';
  const name = step.name || step.ref || '';
  const onto = name ? ' onto '+name : '', on = name ? ' on '+name : '';
  if (mod === 'uturn' && t !== 'arrive') return 'Make a U-turn'+onto;
  switch (t){
    case 'depart': return 'Head '+CARD[Math.round((m.bearing_after||0)/45)%8]+on;
    case 'arrive': return 'Arrive at your destination'+(mod.includes('left') ? ', on the left' : mod.includes('right') ? ', on the right' : '');
    case 'turn': case 'end of road': return 'Turn '+(mod || 'ahead')+onto;
    case 'new name': case 'continue': return (!mod || mod === 'straight') ? 'Continue'+on : 'Keep '+mod.replace('slight ','')+onto;
    case 'merge': return 'Merge'+(mod ? ' '+mod.replace('slight ','') : '')+onto;
    case 'on ramp': return 'Take the ramp'+(mod.includes('left') ? ' on the left' : mod.includes('right') ? ' on the right' : '')+onto;
    case 'off ramp': return 'Take the exit'+(step.exits ? ' '+String(step.exits).split(';')[0] : '')+(mod.includes('left') ? ' on the left' : '')+onto;
    case 'fork': return 'Keep '+(mod.replace('slight ','') || 'straight')+' at the fork'+onto;
    case 'roundabout': case 'rotary': return 'At the roundabout, take the '+ord(m.exit || 1)+' exit'+onto;
    case 'roundabout turn': return 'At the roundabout, turn '+mod+onto;
    case 'exit roundabout': case 'exit rotary': return 'Exit the roundabout'+onto;
    default: return mod && mod !== 'straight' ? 'Go '+mod+onto : 'Continue'+on;
  }
}

/* ---------- state ---------- */
const S = { pos:null, acc:null, heading:0, speed:0, lastFix:0, follow:true, firstFix:false,
            dest:null, routes:[], routeIdx:0, nav:null, sim:null, simTimer:null, limitKmh:null, list:[] };

/* ---------- voice, toast, wake lock ---------- */
function say(text){
  if (!settings.voice || !('speechSynthesis' in window)) return;
  try{ speechSynthesis.cancel(); const u = new SpeechSynthesisUtterance(text); u.rate = 1.03; speechSynthesis.speak(u); }catch{}
}
let toastT;
function toast(msg, ms = 3200){
  const t = $('#toast'); t.textContent = msg; t.hidden = false; t.classList.toggle('nav', !!S.nav);
  clearTimeout(toastT); toastT = setTimeout(() => t.hidden = true, ms);
}
let wakeLock = null;
async function keepAwake(on){
  try{
    if (on && 'wakeLock' in navigator) wakeLock = await navigator.wakeLock.request('screen');
    else if (!on && wakeLock){ await wakeLock.release(); wakeLock = null; }
  }catch{}
}
document.addEventListener('visibilitychange', () => { if (document.visibilityState === 'visible' && S.nav) keepAwake(true); });

/* ---------- layout: keep floating controls above the bottom panel ---------- */
const ro = new ResizeObserver(() => {
  const el = !$('#navbar').hidden ? $('#navbar') : $('#sheet');
  document.documentElement.style.setProperty('--bottom', (el.hidden ? 0 : el.offsetHeight)+'px');
});
ro.observe($('#sheet')); ro.observe($('#navbar'));

/* ---------- position & GPS ---------- */
const carIcon = L.divIcon({ className:'car', iconSize:[44,44], iconAnchor:[22,22],
  html:'<div class="car-rot"><svg viewBox="0 0 44 44"><circle class="halo" cx="22" cy="22" r="20"/><path class="arrow" d="M22 7l11 27-11-6.5L11 34z"/></svg></div>' });
let carMarker = null, accCircle = null;
function drawCar(){
  if (!carMarker){
    carMarker = L.marker(S.pos, { icon:carIcon, interactive:false, zIndexOffset:1000 }).addTo(map);
    accCircle = L.circle(S.pos, { radius:S.acc || 10, stroke:false, fillColor:css('--route'), fillOpacity:.1, interactive:false }).addTo(map);
  }
  carMarker.setLatLng(S.pos); accCircle.setLatLng(S.pos).setRadius(Math.min(S.acc || 10, 500));
  const el = carMarker.getElement()?.querySelector('.car-rot');
  if (el) el.style.transform = `rotate(${S.heading}deg)`;
}
function onPosition(c){
  const ll = [c.latitude, c.longitude], prev = S.pos, now = Date.now();
  let spd = (typeof c.speed === 'number' && !isNaN(c.speed)) ? c.speed : null;
  if (prev){
    const d = dist(prev, ll);
    if (spd == null && S.lastFix) spd = d / Math.max(.5, (now - S.lastFix)/1000);
    if (d > 2) S.heading = (typeof c.heading === 'number' && !isNaN(c.heading) && (spd||0) > 1) ? c.heading : bearing(prev, ll);
  }
  S.pos = ll; S.acc = c.accuracy; S.speed = spd || 0; S.lastFix = now;
  drawCar(); updateSpeedo(); checkReports(); maybeSpeedLimit();
  if (S.nav) navTick();
  if (!S.firstFix){ S.firstFix = true; if (!S.nav && !S.dest) map.setView(ll, 16); }
  else if (S.follow && (S.nav || !S.dest)) followCam();
}
function startGPS(){
  if (!('geolocation' in navigator)){ toast('No location access in this browser. Use Simulate to test routes.'); return; }
  navigator.geolocation.watchPosition(p => { if (!S.sim) onPosition(p.coords); },
    err => toast(err.code === 1 ? 'Location is blocked. Allow it for this site (the page must be on HTTPS).' : 'Waiting for a GPS signal…', 4500),
    { enableHighAccuracy:true, maximumAge:1000, timeout:20000 });
}
function followCam(){
  if (!S.pos) return;
  const z = S.nav ? (S.speed > 24 ? 15 : S.speed > 12 ? 16 : 17) : Math.max(map.getZoom(), 15);
  const off = S.nav ? map.getSize().y * .2 : 0;
  const p = map.project(S.pos, z).subtract([0, off]);
  map.setView(map.unproject(p, z), z, { animate:true, duration:.6 });
}
map.on('dragstart', () => { S.follow = false; $('#btnRecenter').hidden = !S.pos; });
$('#btnRecenter').onclick = () => { S.follow = true; $('#btnRecenter').hidden = true; followCam(); };

/* ---------- speed & limit ---------- */
function updateSpeedo(){
  const v = speedDisp(S.speed);
  $('#spd').textContent = v; $('#spdU').textContent = imperial() ? 'mph' : 'km/h';
  const lim = $('#limit');
  if (S.limitKmh == null){ lim.hidden = true; $('#speedo').classList.remove('over'); return; }
  const lv = imperial() ? Math.round(S.limitKmh/1.609344/5)*5 : Math.round(S.limitKmh);
  lim.hidden = false; lim.classList.toggle('metric', !imperial());
  $('#limitVal').textContent = lv;
  const over = v > lv + (imperial() ? 4 : 6);
  lim.classList.toggle('over', over); $('#speedo').classList.toggle('over', over);
}
let slLast = 0, slBusy = false;
function parseMaxspeed(s){
  if (!s) return null; const m = String(s).match(/(\d+(?:\.\d+)?)\s*(mph)?/i);
  if (!m) return null; const n = parseFloat(m[1]); return m[2] ? n*1.609344 : n;
}
async function maybeSpeedLimit(){
  if (!S.pos || slBusy || Date.now() - slLast < 15000) return;
  if (S.speed < 2 && S.limitKmh != null) return;
  slBusy = true; slLast = Date.now();
  const q = `[out:json][timeout:10];way(around:20,${S.pos[0].toFixed(6)},${S.pos[1].toFixed(6)})[highway][maxspeed];out tags 4;`;
  try{
    const r = await fetch('https://overpass-api.de/api/interpreter', { method:'POST', body:'data='+encodeURIComponent(q),
      headers:{ 'Content-Type':'application/x-www-form-urlencoded' } });
    const j = await r.json(); const ways = j.elements || [];
    let cur = null;
    if (S.nav){ const st = S.nav.route.steps[S.nav.k || 0]; cur = st && (st.name || st.ref); }
    const w = ways.find(w => cur && (w.tags.name === cur || w.tags.ref === cur)) || ways[0];
    S.limitKmh = w ? parseMaxspeed(w.tags.maxspeed) : null;
    updateSpeedo();
  }catch{}
  slBusy = false;
}

/* ---------- search ---------- */
const q = $('#q'); let searchT = null, searchAbort = null;
q.addEventListener('input', () => {
  const v = q.value.trim(); $('#clearQ').hidden = !q.value;
  clearTimeout(searchT);
  if (v.length < 3){ if (!S.dest) renderIdle(); return; }
  searchT = setTimeout(() => search(v), 450);
});
q.addEventListener('keydown', e => { if (e.key === 'Enter'){ clearTimeout(searchT); const v = q.value.trim(); if (v) search(v); q.blur(); } });
$('#clearQ').onclick = () => { q.value = ''; $('#clearQ').hidden = true; closePreview(); q.focus(); };
async function search(text){
  searchAbort?.abort(); searchAbort = new AbortController();
  const c = S.pos || [map.getCenter().lat, map.getCenter().lng];
  const vb = [c[1]-.6, c[0]+.6, c[1]+.6, c[0]-.6].map(n => n.toFixed(4)).join(',');
  sheetHTML('<div class="grab"></div><p class="hint">Searching…</p>');
  try{
    const r = await fetch(`https://nominatim.openstreetmap.org/search?format=jsonv2&limit=8&q=${encodeURIComponent(text)}&viewbox=${vb}`,
      { signal:searchAbort.signal, headers:{ 'Accept-Language':navigator.language || 'en' } });
    if (!r.ok) throw new Error('Search service returned '+r.status+'. Try again in a moment.');
    const j = await r.json();
    S.list = j.map(x => ({ name: x.name || x.display_name.split(',')[0], addr: x.display_name, lat:+x.lat, lng:+x.lon }));
    if (S.pos) S.list.sort((a, b) => dist(S.pos, [a.lat, a.lng]) - dist(S.pos, [b.lat, b.lng]));
    sheetHTML('<div class="grab"></div>' + (S.list.length
      ? `<ul class="list">${S.list.map((p, i) => row(p, 'res', i, ico.pin)).join('')}</ul>`
      : `<p class="hint">No places match “${esc(text)}”. Try adding a city or street.</p>`));
  }catch(e){ if (e.name !== 'AbortError') sheetHTML(`<div class="grab"></div><p class="err">${esc(e.message)}</p>`); }
}
function row(p, kind, i, icon){
  const d = S.pos ? fd(dist(S.pos, [p.lat, p.lng])) : '';
  return `<li><button class="row" data-kind="${kind}" data-i="${i}"><span class="row-ico">${icon}</span>
    <span class="row-txt"><b>${esc(p.name)}</b><small>${esc(shortAddr(p.addr) || p.addr)}</small></span>
    ${d ? `<span class="row-d">${d}</span>` : ''}</button></li>`;
}

/* ---------- sheet views ---------- */
const sheet = $('#sheet');
function sheetHTML(h){ sheet.innerHTML = h; sheet.scrollTop = 0; }
function renderIdle(){
  const h = store.get('home'), w = store.get('work'), rec = store.get('recents', []);
  S.list = rec;
  sheetHTML(`<div class="grab"></div>
    <div class="chips">
      <button class="chip" data-saved="home">${ico.home}<span><b>Home</b><small>${h ? esc(h.name) : 'Not set'}</small></span></button>
      <button class="chip" data-saved="work">${ico.work}<span><b>Work</b><small>${w ? esc(w.name) : 'Not set'}</small></span></button>
    </div>
    ${rec.length ? `<h3 class="label">Recent</h3><ul class="list">${rec.map((p, i) => row(p, 'rec', i, ico.clock)).join('')}</ul>`
                 : '<p class="hint">Search for a place, or press and hold anywhere on the map to drop a pin.</p>'}`);
}
function renderPreview(state){
  const p = S.dest; if (!p) return;
  let body;
  if (state === 'loading') body = '<p class="hint">Finding routes…</p>';
  else if (state && state.error) body = `<p class="err">${esc(state.error)}</p>`;
  else body = `<ul class="routes">${S.routes.map((r, i) => `<li><button class="route-opt${i === S.routeIdx ? ' on' : ''}" data-route="${i}">
      <b>${fmtDur(r.duration)}</b><span>${fd(r.distance)} · via ${esc(r.via)}</span>${i === 0 ? '<em>Fastest</em>' : '<em></em>'}</button></li>`).join('')}</ul>
    <div class="pv-actions"><button class="btn primary" data-act="go">Go</button><button class="btn" data-act="sim">Simulate</button></div>`;
  sheetHTML(`<div class="grab"></div>
    <div class="pv-head"><div class="row-txt"><b class="pv-name">${esc(p.name)}</b><small>${esc(shortAddr(p.addr) || p.addr || '')}</small></div>
      <button class="icon-btn" data-act="close" aria-label="Close">×</button></div>
    ${body}
    <div class="pv-save"><button class="link" data-act="sethome">Set as Home</button><button class="link" data-act="setwork">Set as Work</button></div>`);
}
sheet.addEventListener('click', e => {
  const b = e.target.closest('button'); if (!b) return;
  if (b.dataset.kind) return previewPlace(S.list[+b.dataset.i]);
  if (b.dataset.saved){ const p = store.get(b.dataset.saved); return p ? previewPlace(p) : toast('Search for a place, then tap “Set as '+(b.dataset.saved === 'home' ? 'Home' : 'Work')+'”.'); }
  if (b.dataset.route){ S.routeIdx = +b.dataset.route; drawRoutes(); renderPreview(); return; }
  const act = b.dataset.act;
  if (act === 'close') closePreview();
  else if (act === 'go') startNav(false);
  else if (act === 'sim') startNav(true);
  else if (act === 'sethome' || act === 'setwork'){ store.set(act.slice(3), S.dest); toast((act === 'sethome' ? 'Home' : 'Work')+' saved'); }
});

/* ---------- destination & routing ---------- */
const destIcon = L.divIcon({ className:'pin', iconSize:[34,44], iconAnchor:[17,42],
  html:'<svg viewBox="0 0 34 44"><path d="M17 1C8.2 1 1.5 7.6 1.5 16.2 1.5 27.5 17 43 17 43s15.5-15.5 15.5-26.8C32.5 7.6 25.8 1 17 1z" fill="#d93a2b" stroke="#fff" stroke-width="2"/><circle cx="17" cy="16" r="5.5" fill="#fff"/></svg>' });
let destMarker = null;
const routeLayer = L.layerGroup().addTo(map);
let navLine = null, navCasing = null;

function addRecent(p){
  const rec = store.get('recents', []).filter(r => !(Math.abs(r.lat - p.lat) < 1e-5 && Math.abs(r.lng - p.lng) < 1e-5));
  rec.unshift({ name:p.name, addr:p.addr, lat:p.lat, lng:p.lng }); store.set('recents', rec.slice(0, 8));
}
async function previewPlace(p){
  if (!p) return;
  if (S.nav) endNav(true);
  S.dest = { ...p, ll:[p.lat, p.lng] }; S.routes = []; S.routeIdx = 0; S.follow = false;
  q.value = p.name; $('#clearQ').hidden = false; q.blur();
  if (destMarker) destMarker.setLatLng(S.dest.ll); else destMarker = L.marker(S.dest.ll, { icon:destIcon, interactive:false }).addTo(map);
  routeLayer.clearLayers(); renderPreview('loading');
  let from = S.pos;
  if (!from){ from = [map.getCenter().lat, map.getCenter().lng]; toast('No GPS fix yet, so routing starts from the map center.'); }
  S.origin = from;
  try{
    S.routes = await fetchRoutes(from, S.dest.ll, true);
    drawRoutes(); fitRoutes(); renderPreview();
  }catch(e){ map.setView(S.dest.ll, 15); renderPreview({ error:e.message }); }
}
function closePreview(){
  S.dest = null; S.routes = []; routeLayer.clearLayers();
  if (destMarker){ map.removeLayer(destMarker); destMarker = null; }
  q.value = ''; $('#clearQ').hidden = true; renderIdle();
  if (S.pos){ S.follow = true; followCam(); }
}
async function fetchRoutes(from, to, alts){
  const url = `https://router.project-osrm.org/route/v1/driving/${from[1]},${from[0]};${to[1]},${to[0]}?overview=full&geometries=geojson&steps=true&alternatives=${alts ? 'true' : 'false'}`;
  let r;
  try{ r = await fetch(url); }catch{ throw new Error('Could not reach the routing service. Check your connection.'); }
  if (!r.ok) throw new Error('Routing service returned '+r.status+'. Try again in a moment.');
  const j = await r.json();
  if (j.code !== 'Ok' || !j.routes?.length) throw new Error('No drivable route to this place.');
  return j.routes.map(prepRoute);
}
function prepRoute(r){
  const coords = r.geometry.coordinates.map(c => [c[1], c[0]]);
  if (coords.length < 2) coords.push(coords[0]);
  const cum = new Float64Array(coords.length);
  for (let i = 1; i < coords.length; i++) cum[i] = cum[i-1] + dist(coords[i-1], coords[i]);
  const total = cum[coords.length-1];
  const steps = r.legs.flatMap(l => l.steps);
  const sum = steps.reduce((a, s) => a + s.distance, 0) || 1, scale = total / sum;
  const stepStart = []; let acc = 0;
  for (const s of steps){ stepStart.push(acc*scale); acc += s.distance; }
  const byName = {};
  steps.forEach(s => { const n = s.name || s.ref; if (n) byName[n] = (byName[n] || 0) + s.distance; });
  const via = Object.entries(byName).sort((a, b) => b[1] - a[1])[0]?.[0] || 'local roads';
  return { coords, cum, total, steps, stepStart, duration:r.duration, distance:r.distance, via };
}
function drawRoutes(){
  routeLayer.clearLayers();
  const order = S.routes.map((_, i) => i).filter(i => i !== S.routeIdx).concat(S.routeIdx);
  for (const i of order){
    const r = S.routes[i], on = i === S.routeIdx;
    L.polyline(r.coords, { color:css('--casing'), weight:on ? 11 : 9, opacity:on ? 1 : .7, interactive:false }).addTo(routeLayer);
    const pl = L.polyline(r.coords, { color:on ? css('--route') : css('--route-alt'), weight:on ? 7 : 6 }).addTo(routeLayer);
    if (!on) pl.on('click', () => { S.routeIdx = i; drawRoutes(); renderPreview(); });
  }
}
function fitRoutes(){
  const b = L.latLngBounds(S.routes.flatMap(r => r.coords));
  if (S.pos) b.extend(S.pos);
  map.fitBounds(b, { paddingTopLeft:[30, $('#top').offsetHeight + 30], paddingBottomRight:[30, sheet.offsetHeight + 30] });
}

/* long-press to drop a pin */
let lpT = null, lpStart = null, lpFired = 0;
const mapEl = map.getContainer();
mapEl.addEventListener('pointerdown', e => {
  if (e.pointerType === 'mouse' && e.button !== 0) return;
  lpStart = [e.clientX, e.clientY];
  lpT = setTimeout(() => { lpFired = Date.now(); dropPin(map.mouseEventToLatLng(e)); }, 600);
});
['pointerup','pointercancel','pointerleave'].forEach(t => mapEl.addEventListener(t, () => clearTimeout(lpT)));
mapEl.addEventListener('pointermove', e => { if (lpStart && Math.hypot(e.clientX - lpStart[0], e.clientY - lpStart[1]) > 10) clearTimeout(lpT); });
map.on('contextmenu', e => { if (Date.now() - lpFired > 1000) dropPin(e.latlng); });
async function dropPin(ll){
  if (S.nav) return;
  const p = { name:'Dropped pin', addr:`${ll.lat.toFixed(5)}, ${ll.lng.toFixed(5)}`, lat:ll.lat, lng:ll.lng };
  previewPlace(p);
  try{
    const r = await fetch(`https://nominatim.openstreetmap.org/reverse?format=jsonv2&zoom=18&lat=${ll.lat}&lon=${ll.lng}`);
    const j = await r.json();
    if (j?.display_name && S.dest && S.dest.lat === ll.lat){
      const a = j.address || {};
      S.dest.name = j.name || [a.house_number, a.road].filter(Boolean).join(' ') || j.display_name.split(',')[0];
      S.dest.addr = j.display_name; q.value = S.dest.name;
      renderPreview(S.routes.length ? undefined : 'loading');
    }
  }catch{}
}

/* ---------- navigation ---------- */
function startNav(simulate){
  const route = S.routes[S.routeIdx]; if (!route) return;
  addRecent(S.dest);
  S.nav = { route, spoken:{}, off:0, lastReroute:0, along:0, k:0, rerouting:false };
  routeLayer.clearLayers(); drawNavRoute();
  $('#searchbar').hidden = true; $('#banner').hidden = false;
  sheet.hidden = true; $('#navbar').hidden = false;
  S.follow = true; $('#btnRecenter').hidden = true;
  keepAwake(true);
  const first = route.steps[0], second = route.steps[1];
  say(instr(first) + (second && second.maneuver.type !== 'arrive' ? '. Then, '+instr(second).replace(/^./, c => c.toLowerCase()) : ''));
  if (simulate) startSim();
  if (S.pos && !S.sim) navTick();
  else renderNav(Math.min(1, route.steps.length-1), route.stepStart[1] ?? route.total, route.total, route.duration);
  if (S.pos) followCam(); else map.setView(route.coords[0], 17);
}
function drawNavRoute(){
  if (navLine) map.removeLayer(navLine); if (navCasing) map.removeLayer(navCasing);
  const c = S.nav.route.coords;
  navCasing = L.polyline(c, { color:css('--casing'), weight:13, interactive:false }).addTo(map);
  navLine = L.polyline(c, { color:css('--route'), weight:8, interactive:false }).addTo(map);
  if (carMarker){ carMarker.remove(); carMarker.addTo(map); }
}
function snap(R, p, minAlong){
  const c = R.coords, cos = Math.cos(rad(p[0]));
  let gi = 0, gd = Infinity, gt = 0, fi = -1, fdd = Infinity, ft = 0;
  for (let i = 0; i < c.length - 1; i++){
    const ax = (c[i][1]-p[1])*cos, ay = c[i][0]-p[0], bx = (c[i+1][1]-p[1])*cos, by = c[i+1][0]-p[0];
    const dx = bx-ax, dy = by-ay, l2 = dx*dx + dy*dy;
    let t = l2 ? -(ax*dx + ay*dy)/l2 : 0; t = t < 0 ? 0 : t > 1 ? 1 : t;
    const x = ax + t*dx, y = ay + t*dy, d2 = x*x + y*y;
    if (d2 < gd){ gd = d2; gi = i; gt = t; }
    if (R.cum[i+1] >= minAlong && d2 < fdd){ fdd = d2; fi = i; ft = t; }
  }
  let i = fi, t = ft, d2 = fdd;
  if (fi < 0 || Math.sqrt(fdd)*111320 > 60){ i = gi; t = gt; d2 = gd; }
  const a = c[i], b = c[i+1];
  return { d:Math.sqrt(d2)*111320, i, along:R.cum[i] + t*(R.cum[i+1]-R.cum[i]), pt:[a[0]+t*(b[0]-a[0]), a[1]+t*(b[1]-a[1])] };
}
function stepAt(R, along){ let k = 0; while (k+1 < R.stepStart.length && R.stepStart[k+1] <= along) k++; return k; }
function navTick(){
  const n = S.nav, R = n.route;
  const s = snap(R, S.pos, n.along - 150);
  if (s.d > Math.max(45, (S.acc || 0)*1.3)){ if (++n.off >= 3) reroute(); } else n.off = 0;
  n.along = s.along;
  const k = stepAt(R, s.along); n.k = k;
  const last = R.steps.length - 1, next = Math.min(k+1, last);
  const toNext = (k+1 <= last ? R.stepStart[k+1] : R.total) - s.along;
  const remD = Math.max(0, R.total - s.along);
  const stepEnd = k+1 <= last ? R.stepStart[k+1] : R.total, stepLen = stepEnd - R.stepStart[k];
  let remT = R.steps[k].duration * (stepLen > 0 ? Math.max(0, Math.min(1, (stepEnd - s.along)/stepLen)) : 0);
  for (let i = k+1; i < R.steps.length; i++) remT += R.steps[i].duration;
  renderNav(next, toNext, remD, remT);
  speakGuidance(next, toNext);
  if (navLine){ const rest = [s.pt].concat(R.coords.slice(s.i+1)); navLine.setLatLngs(rest); navCasing.setLatLngs(rest); }
  if (remD < 25 && s.d < 60) arrive();
}
function renderNav(k, toNext, remD, remT){
  const R = S.nav.route, step = R.steps[k];
  const d = fmtDist(Math.max(0, toNext));
  $('#bIcon').innerHTML = turnSVG(step);
  $('#bDist').innerHTML = `${d.v}<small>${d.u}</small>`;
  $('#bStreet').textContent = S.nav.rerouting ? 'Rerouting…' : instr(step);
  const nxt = R.steps[k+1];
  if (nxt && (R.stepStart[k+1] - R.stepStart[k]) < 300){ $('#bThen').hidden = false; $('#bThen').innerHTML = 'Then '+turnSVG(nxt); }
  else $('#bThen').hidden = true;
  $('#eta').textContent = clock(remT); $('#remT').textContent = fmtDur(remT); $('#remD').textContent = fd(remD);
}
function speakGuidance(k, toNext){
  const n = S.nav, R = n.route, step = R.steps[k];
  const sp = n.spoken[k] || (n.spoken[k] = {});
  const text = instr(step), lc = text.replace(/^./, c => c.toLowerCase());
  const far = imperial() ? 1609 : 1000, near = imperial() ? 402 : 400;
  const farWord = imperial() ? '1 mile' : '1 kilometer', nearWord = imperial() ? 'a quarter mile' : '400 meters';
  const legLen = k > 0 ? R.stepStart[k] - R.stepStart[k-1] : R.stepStart[k];
  const nowAt = Math.max(40, S.speed*5);
  if (!sp.far && toNext <= far + 80 && toNext > far - 250 && legLen > far*1.3){ sp.far = 1; say(`In ${farWord}, ${lc}`); }
  else if (!sp.near && toNext <= near + 40 && toNext > nowAt + 80 && legLen > near*1.2){ sp.near = 1; say(`In ${nearWord}, ${lc}`); }
  else if (!sp.now && toNext <= nowAt){
    sp.now = 1; sp.far = sp.near = 1;
    const nxt = R.steps[k+1];
    const then = nxt && (R.stepStart[k+1] - R.stepStart[k]) < 200 ? ', then '+instr(nxt).replace(/^./, c => c.toLowerCase()) : '';
    say(step.maneuver.type === 'arrive' ? 'You have arrived' : text + then);
  }
}
async function reroute(){
  const n = S.nav;
  if (!n || n.rerouting || Date.now() - n.lastReroute < 8000) return;
  n.rerouting = true; n.lastReroute = Date.now(); $('#bStreet').textContent = 'Rerouting…'; say('Rerouting');
  try{
    const [r] = await fetchRoutes(S.pos, S.dest.ll, false);
    if (!S.nav) return;
    n.route = r; n.spoken = {}; n.off = 0; n.along = 0; drawNavRoute();
    if (S.sim) S.sim.along = 0;
  }catch(e){ toast(e.message); }
  n.rerouting = false;
}
function arrive(){
  const name = S.dest?.name || 'your destination';
  if (!S.nav.spoken.arrived) say('You have arrived at '+name);
  endNav(true);
  sheetHTML(`<div class="grab"></div><div class="pv-head"><div class="row-txt"><b class="pv-name">You’ve arrived</b><small>${esc(name)}</small></div>
    <button class="icon-btn" data-act="close" aria-label="Close">×</button></div>`);
}
function endNav(keepDest){
  stopSim(); keepAwake(false);
  if (navLine){ map.removeLayer(navLine); map.removeLayer(navCasing); navLine = navCasing = null; }
  S.nav = null;
  $('#banner').hidden = true; $('#searchbar').hidden = false; $('#navbar').hidden = true; sheet.hidden = false;
  if (!keepDest) closePreview();
  else { S.routes = []; routeLayer.clearLayers(); if (destMarker){ map.removeLayer(destMarker); destMarker = null; } S.dest = null; q.value=''; $('#clearQ').hidden = true; renderIdle(); }
}
$('#btnEnd').onclick = () => { speechSynthesis?.cancel?.(); endNav(false); };

/* ---------- simulated drive ---------- */
function pointAt(R, along){
  const c = R.coords, cum = R.cum; along = Math.max(0, Math.min(R.total, along));
  let lo = 0, hi = c.length - 1;
  while (hi - lo > 1){ const m = (lo + hi) >> 1; if (cum[m] <= along) lo = m; else hi = m; }
  const seg = (cum[hi] - cum[lo]) || 1, t = (along - cum[lo]) / seg;
  return { pt:[c[lo][0] + t*(c[hi][0]-c[lo][0]), c[lo][1] + t*(c[hi][1]-c[lo][1])], brg:bearing(c[lo], c[hi]) };
}
function startSim(){
  S.sim = { along:0, mul:1 }; $('#btnSim').hidden = false; $('#btnSim').textContent = '×1';
  S.simTimer = setInterval(() => {
    const R = S.nav?.route; if (!R){ stopSim(); return; }
    const st = R.steps[stepAt(R, S.sim.along)];
    const v = Math.max(5, Math.min(35, st.duration > 0 ? st.distance/st.duration : 13)) * S.sim.mul;
    S.sim.along += v * .5;
    const p = pointAt(R, S.sim.along);
    onPosition({ latitude:p.pt[0], longitude:p.pt[1], speed:v, heading:p.brg, accuracy:5 });
  }, 500);
}
function stopSim(){ clearInterval(S.simTimer); S.simTimer = null; S.sim = null; $('#btnSim').hidden = true; }
$('#btnSim').onclick = () => { if (!S.sim) return; S.sim.mul = S.sim.mul >= 8 ? 1 : S.sim.mul*2; $('#btnSim').textContent = '×'+S.sim.mul; };

/* ---------- road reports ---------- */
const RT = {
  police:{ label:'Police', color:'#2557c9', svg:'<svg viewBox="0 0 24 24" fill="currentColor"><path d="M12 2l8 3.2v5.6c0 5.3-3.4 9.1-8 11.2-4.6-2.1-8-5.9-8-11.2V5.2z"/></svg>' },
  crash:{ label:'Crash', color:'#c8312b', svg:'<svg viewBox="0 0 24 24" fill="currentColor"><path d="M12 1.5l2.3 5.4 5.3-2.4-2 5.5 5.4 2-5.4 2 2 5.5-5.3-2.4L12 22.5l-2.3-5.4-5.3 2.4 2-5.5-5.4-2 5.4-2-2-5.5 5.3 2.4z"/></svg>' },
  hazard:{ label:'Hazard', color:'#d99a00', svg:'<svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.6" stroke-linecap="round" stroke-linejoin="round"><path d="M12 3L2 20h20z"/><path d="M12 10v4M12 17v.1"/></svg>' },
  jam:{ label:'Traffic', color:'#d9541e', svg:'<svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.8" stroke-linecap="round"><path d="M4 7h16M4 12h16M4 17h16"/></svg>' },
  closure:{ label:'Closure', color:'#6a46c9', svg:'<svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.8" stroke-linecap="round"><circle cx="12" cy="12" r="9"/><path d="M7 12h10"/></svg>' },
  camera:{ label:'Camera', color:'#3d4a55', svg:'<svg viewBox="0 0 24 24" fill="currentColor"><path d="M4 7h3l2-2.5h6L17 7h3a1 1 0 0 1 1 1v10a1 1 0 0 1-1 1H4a1 1 0 0 1-1-1V8a1 1 0 0 1 1-1zm8 10a4 4 0 1 0 0-8 4 4 0 0 0 0 8z"/></svg>' }
};
const REPORT_TTL = 3*3600e3;
let reports = store.get('reports', []).filter(r => Date.now() - r.t < REPORT_TTL && RT[r.type]);
const reportLayer = L.layerGroup().addTo(map);
const alerted = new Set();
function drawReports(){
  reportLayer.clearLayers();
  for (const r of reports){
    const T = RT[r.type];
    const m = L.marker([r.lat, r.lng], { icon:L.divIcon({ className:'rep', iconSize:[32,32], iconAnchor:[16,16],
      html:`<div style="background:${T.color}">${T.svg}</div>` }) }).addTo(reportLayer);
    const mins = Math.round((Date.now() - r.t)/60000);
    m.bindPopup(`<div class="pop"><b>${T.label}</b><small>Reported ${mins < 1 ? 'just now' : mins+' min ago'} · saved on this device</small><br><button data-del="${r.id}">Remove</button></div>`);
  }
}
map.on('popupopen', e => {
  const b = e.popup.getElement().querySelector('[data-del]');
  if (b) b.onclick = () => { reports = reports.filter(r => r.id !== b.dataset.del); store.set('reports', reports); map.closePopup(); drawReports(); };
});
function addReport(type){
  const ll = S.pos || [map.getCenter().lat, map.getCenter().lng];
  reports.push({ id:Math.random().toString(36).slice(2, 10), type, lat:ll[0], lng:ll[1], t:Date.now() });
  store.set('reports', reports); drawReports(); closeModal();
  toast(RT[type].label+' reported. Thanks!');
}
function checkReports(){
  if (!S.pos) return;
  for (const r of reports){
    if (alerted.has(r.id) || Date.now() - r.t < 120000) continue;
    const d = dist(S.pos, [r.lat, r.lng]); if (d > 500) continue;
    if (S.speed > 3){ const diff = Math.abs(((bearing(S.pos, [r.lat, r.lng]) - S.heading + 540) % 360) - 180); if (diff > 70) continue; }
    alerted.add(r.id);
    toast(RT[r.type].label+' reported ahead · '+fd(d), 4500); say(RT[r.type].label+' reported ahead');
  }
}

/* ---------- modal: report + settings ---------- */
const modal = $('#modal'), mSheet = $('#modalSheet');
function openModal(html){ mSheet.innerHTML = '<div class="grab"></div>'+html; modal.hidden = false; }
function closeModal(){ modal.hidden = true; }
modal.addEventListener('click', e => { if (e.target === modal) closeModal(); });
$('#btnReport').onclick = () => openModal(`<h3 class="label" style="margin-top:0">Report at your location</h3>
  <div class="rgrid">${Object.entries(RT).map(([k, T]) => `<button class="rbtn" data-rep="${k}"><i style="background:${T.color}">${T.svg}</i>${T.label}</button>`).join('')}</div>
  <p class="fine">Reports stay on this device for 3 hours. Sharing them with other drivers needs a server.</p>`);
function seg(key, opts){ return `<div class="seg">${opts.map(([v, l]) => `<button data-set="${key}" data-val="${v}" class="${String(settings[key]) === String(v) ? 'on' : ''}">${l}</button>`).join('')}</div>`; }
function renderSettings(){
  openModal(`<h3 class="label" style="margin-top:0">Settings</h3>
    <div class="set-row"><span>Units</span>${seg('units', [['imperial','Miles'],['metric','Kilometers']])}</div>
    <div class="set-row"><span>Voice guidance</span>${seg('voice', [['true','On'],['false','Off']])}</div>
    <div class="set-row"><span>Map</span>${seg('theme', [['auto','Auto'],['light','Day'],['dark','Night']])}</div>
    <div class="set-row"><span>Saved data</span><div class="seg"><button data-clear="places">Clear Home, Work, recents</button><button data-clear="reports">Clear reports</button></div></div>
    <p class="fine">Map data and tiles © OpenStreetMap contributors · search by Nominatim · routing by OSRM · speed limits from Overpass. These are free public servers meant for light personal use.</p>`);
}
$('#btnSettings').onclick = renderSettings;
mSheet.addEventListener('click', e => {
  const b = e.target.closest('button'); if (!b) return;
  if (b.dataset.rep) return addReport(b.dataset.rep);
  if (b.dataset.set){
    const k = b.dataset.set; let v = b.dataset.val; if (v === 'true' || v === 'false') v = v === 'true';
    settings[k] = v; saveSettings();
    if (k === 'theme') applyTheme();
    if (k === 'units'){ updateSpeedo(); if (S.nav && S.pos) navTick(); else if (!S.nav) S.dest ? renderPreview() : renderIdle(); }
    if (k === 'voice' && !v) speechSynthesis?.cancel?.();
    renderSettings(); return;
  }
  if (b.dataset.clear === 'places'){ ['home','work','recents'].forEach(k => store.set(k, null)); if (!S.dest && !S.nav) renderIdle(); toast('Saved places cleared'); }
  if (b.dataset.clear === 'reports'){ reports = []; store.set('reports', []); drawReports(); toast('Reports cleared'); }
});

/* ---------- boot ---------- */
applyTheme(); renderIdle(); drawReports(); updateSpeedo(); startGPS();
</script>
</body>
</html>
