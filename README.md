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
.compass{width:60px;height:60px;position:relative;flex-direction:column}
.compass .rose{width:46px;height:46px;will-change:transform}
.compass .rose .n{fill:var(--danger)} .compass .rose .s{fill:var(--muted)}
.compass .rose .ring{fill:none;stroke:var(--line);stroke-width:2}
.compass .rose text{font:800 7px var(--font);fill:var(--ink);text-anchor:middle}
.compass .rose .tn{fill:var(--danger)}
.compass small{position:absolute;bottom:-17px;left:50%;transform:translateX(-50%);font:700 11px var(--mono);background:var(--panel);padding:1px 6px;border-radius:6px;box-shadow:0 1px 4px rgba(0,0,0,.2);white-space:nowrap;font-variant-numeric:tabular-nums}
.compass.off .rose{opacity:.45}
.compass.off small{color:var(--muted)}
.car .beam{fill:url(#beamGrad);opacity:0;transition:opacity .3s}
.car.compass-on .beam{opacity:1}
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
.car-rot{width:44px;height:44px;will-change:transform}
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

/* Confirm-to-place bar */
#place{position:fixed;left:0;right:0;bottom:0;z-index:700;max-width:580px;margin:0 auto;background:var(--panel);border-radius:20px 20px 0 0;box-shadow:0 -6px 26px rgba(0,0,0,.22);
  padding:16px max(16px,env(safe-area-inset-right,0px)) calc(14px + env(safe-area-inset-bottom,0px)) max(16px,env(safe-area-inset-left,0px))}
body.placing #sheet,body.placing #navbar{visibility:hidden}
.pl-head{display:flex;align-items:center;gap:12px}
.pl-badge{width:42px;height:42px;border-radius:50%;display:grid;place-items:center;color:#fff;flex:none}
.pl-badge svg{width:22px;height:22px}
.pl-title{font:800 18px/1.2 var(--font)!important;white-space:normal!important}
.pl-sub{font:600 12.5px var(--mono);color:var(--muted);font-variant-numeric:tabular-nums}
.pl-tip{color:var(--muted);font-size:13px;margin:10px 0 0}
.pl-actions{display:grid;grid-template-columns:1fr 2fr;gap:10px;margin-top:12px}
.pin.ghost svg{animation:bob 1.1s ease-in-out infinite}
@keyframes bob{50%{transform:translateY(-6px)}}
.rep div{width:100%;height:100%}
.rep.place div{box-shadow:0 0 0 7px color-mix(in srgb,var(--route) 35%,transparent),0 3px 8px rgba(0,0,0,.4)}
.rep.place svg{width:21px;height:21px}
.cat-pin svg{width:15px;height:15px}
.btn:disabled{opacity:.45;cursor:default}

/* Legend + recording pill */
#legend{display:flex;align-items:center;gap:5px;flex-wrap:wrap;background:var(--panel);border-radius:12px;box-shadow:var(--shadow);padding:7px 10px;font:700 11px var(--font)}
#legend .lg-t{font:800 10px var(--font);letter-spacing:.1em;text-transform:uppercase;color:var(--muted);margin-right:2px}
#legend .lg{display:inline-flex;align-items:center;gap:4px;padding:2px 6px;border-radius:6px;background:var(--panel-2);font-variant-numeric:tabular-nums}
#legend i{width:14px;height:4px;border-radius:2px;display:inline-block}
#legend i.dash{background:repeating-linear-gradient(90deg,var(--c) 0 3px,transparent 3px 6px)}
#legend .lg-status{flex-basis:100%;color:var(--muted);font-weight:600}
#legend .lg-status.busy::before{content:"";display:inline-block;width:8px;height:8px;margin-right:6px;border-radius:50%;border:2px solid var(--route);border-right-color:transparent;animation:spin .8s linear infinite;vertical-align:-1px}
@keyframes spin{to{transform:rotate(360deg)}}
#recPill{align-self:center;display:inline-flex;align-items:center;gap:8px;background:var(--danger);color:#fff;border-radius:999px;padding:8px 16px;font:700 13px var(--mono);box-shadow:var(--shadow);font-variant-numeric:tabular-nums;white-space:pre}
#recPill::before{content:"";width:9px;height:9px;border-radius:50%;background:#fff;animation:blink 1s infinite}
@keyframes blink{50%{opacity:.25}}

/* Elevation chip */
.alt{background:var(--panel);box-shadow:var(--shadow);border-radius:10px;padding:5px 9px;text-align:center;line-height:1.15;margin-bottom:4px}
.alt span{display:block;font:800 9px var(--font);letter-spacing:.12em;text-transform:uppercase;color:var(--muted)}
.alt b{font:700 14px var(--mono);font-variant-numeric:tabular-nums}

/* Nearby categories */
.cats{display:flex;gap:8px;overflow-x:auto;margin:14px 0 0;padding-bottom:4px;scrollbar-width:none}
.cats::-webkit-scrollbar{display:none}
.cat{flex:none;display:inline-flex;align-items:center;gap:7px;height:38px;padding:0 14px 0 8px;border-radius:999px;border:1.5px solid var(--line);font-weight:700;font-size:14px}
.cat:hover{background:var(--panel-2)}
.cat i{width:24px;height:24px;border-radius:50%;display:grid;place-items:center;color:#fff}
.cat svg{width:14px;height:14px}

/* Layers sheet */
.bases{display:grid;grid-template-columns:repeat(3,1fr);gap:10px}
.base{display:flex;flex-direction:column;gap:6px;font-weight:700;font-size:13px;text-align:center}
.base i{display:block;aspect-ratio:1.35;max-width:100%;border-radius:12px;border:3px solid transparent;box-shadow:inset 0 0 0 1px rgba(0,0,0,.12)}
.base.on{color:var(--route)} .base.on i{border-color:var(--route)}
.th-street{background:linear-gradient(155deg,transparent 44%,#fff 44% 51%,transparent 51%),linear-gradient(65deg,transparent 58%,#f4c95d 58% 64%,transparent 64%),linear-gradient(135deg,#f2efe9 0 68%,#aad3df 68%)}
.th-topo{background:repeating-radial-gradient(circle at 68% 62%,#ece6c8 0 6px,#c4b07c 6px 7px),#ece6c8}
.th-satellite{background:radial-gradient(circle at 28% 30%,#6c8a45,transparent 42%),radial-gradient(circle at 76% 72%,#3f5a2c,transparent 48%),linear-gradient(115deg,transparent 52%,#b9a77d 52% 56%,transparent 56%),linear-gradient(135deg,#6b6a48,#2c3f25)}
.tg{display:flex;align-items:center;gap:12px;width:100%;padding:11px 0;text-align:left;border-bottom:1px solid var(--line)}
.tg .row-txt small{white-space:normal}
.sw{width:46px;height:28px;border-radius:14px;background:var(--line);position:relative;flex:none;transition:background .15s}
.sw::after{content:"";position:absolute;top:3px;left:3px;width:22px;height:22px;border-radius:50%;background:#fff;box-shadow:0 1px 3px rgba(0,0,0,.3);transition:transform .15s}
.tg[aria-checked="true"] .sw{background:var(--guide)}
.tg[aria-checked="true"] .sw::after{transform:translateX(18px)}
.rec-dot{width:28px;height:28px;border-radius:50%;border:3px solid var(--danger);flex:none;position:relative}
.rec-dot::after{content:"";position:absolute;inset:4px;border-radius:50%;background:var(--danger)}
.rec-dot.on::after{inset:6px;border-radius:2px}
.stats{display:grid;grid-template-columns:repeat(3,1fr);gap:10px}
.stats div{background:var(--panel-2);border-radius:12px;padding:12px;display:flex;flex-direction:column;gap:2px}
.stats b{font:700 19px var(--mono);font-variant-numeric:tabular-nums}
.stats span{font:700 11px var(--font);letter-spacing:.08em;text-transform:uppercase;color:var(--muted)}

/* Trail popup */
.tp-grade{display:inline-block;padding:2px 8px;border-radius:6px;color:#fff;font-weight:800;font-size:11px;margin:6px 0 0;letter-spacing:.04em}
.pop dl{display:grid;grid-template-columns:auto 1fr;gap:3px 12px;margin:8px 0 0;font-size:13px}
.pop dt{color:var(--muted)} .pop dd{margin:0;font-weight:600}
.pop button.go{color:var(--route);margin-top:10px}

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
  <div id="legend" hidden></div>
  <button id="recPill" hidden aria-label="Stop recording"></button>
</header>

<div id="hud">
  <div id="limit" class="limit" hidden><span>SPEED<br>LIMIT</span><b id="limitVal">35</b></div>
  <div id="speedo" class="speedo"><b id="spd">0</b><span id="spdU">mph</span></div>
  <div id="alt" class="alt" hidden><span>Elev</span><b id="altVal"></b></div>
</div>

<div id="fabs">
  <button id="btnCompass" class="fab compass off" aria-label="Turn on live compass" style="margin-bottom:14px">
    <svg class="rose" viewBox="0 0 48 48"><circle class="ring" cx="24" cy="24" r="21"/>
      <path class="n" d="M24 7l5 17h-10z"/><path class="s" d="M24 41l5-17h-10z"/>
      <text class="tn" x="24" y="6.2">N</text><text x="24" y="47">S</text><text x="45.5" y="26.5">E</text><text x="2.5" y="26.5">W</text></svg>
    <small id="cmpTxt">Compass</small>
  </button>
  <button id="btnLayers" class="fab" aria-label="Map layers and settings"><svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2" stroke-linejoin="round"><path d="M12 3l9.5 5L12 13 2.5 8z"/><path d="M2.5 12.5L12 17.5l9.5-5"/><path d="M2.5 16.5L12 21.5l9.5-5"/></svg></button>
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

<section id="place" hidden aria-live="polite"></section>
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
const settings = Object.assign({ units:'imperial', voice:true, theme:'auto', base:'street',
  ovOffroad:true, ovHike:false, ovMtb:false, ovReports:true }, store.get('settings', {}));
const saveSettings = () => store.set('settings', settings);
const mq = matchMedia('(prefers-color-scheme: dark)');
const isDark = () => settings.theme === 'dark' || (settings.theme === 'auto' && mq.matches);

const map = L.map('map', { zoomControl:false, attributionControl:true, zoomSnap:.25 }).setView([40.7128, -74.006], 13);
const TILE_ATTR = '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors';
function applyTheme(){
  if (settings.theme === 'auto') delete document.documentElement.dataset.theme;
  else document.documentElement.dataset.theme = settings.theme;
  setBase();
  map.getContainer().classList.toggle('night', isDark() && baseKey === 'street');
  applyOverlays();
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
  const el = !$('#place').hidden ? $('#place') : !$('#navbar').hidden ? $('#navbar') : $('#sheet');
  document.documentElement.style.setProperty('--bottom', (el.hidden ? 0 : el.offsetHeight)+'px');
});
ro.observe($('#sheet')); ro.observe($('#navbar')); ro.observe($('#place'));

/* ---------- position & GPS ---------- */
const carIcon = L.divIcon({ className:'car', iconSize:[44,44], iconAnchor:[22,22],
  html:'<div class="car-rot"><svg viewBox="0 0 44 44" style="overflow:visible"><defs><radialGradient id="beamGrad" cx="22" cy="22" r="40" gradientUnits="userSpaceOnUse"><stop offset="0" stop-color="#1a6fe0" stop-opacity=".45"/><stop offset="1" stop-color="#1a6fe0" stop-opacity="0"/></radialGradient></defs><path class="beam" d="M22 22L4 -14A40 40 0 0 1 40 -14z"/><circle class="halo" cx="22" cy="22" r="20"/><path class="arrow" d="M22 7l11 27-11-6.5L11 34z"/></svg></div>' });
let carMarker = null, accCircle = null;
function drawCar(){
  if (!carMarker){
    carMarker = L.marker(S.pos, { icon:carIcon, interactive:false, zIndexOffset:1000 }).addTo(map);
    accCircle = L.circle(S.pos, { radius:S.acc || 10, stroke:false, fillColor:css('--route'), fillOpacity:.1, interactive:false }).addTo(map);
  }
  carMarker.setLatLng(S.pos); accCircle.setLatLng(S.pos).setRadius(Math.min(S.acc || 10, 500));
  rotateCar();
}
/* ---------- heading animation: one rAF loop, frame-rate independent ---------- */
// Sensor readings are applied as targets; each frame eases toward them with an exponential
// time constant. Compass uses ~45 ms (feels instant, no jitter); GPS course uses ~300 ms.
let carRot = 0, roseRot = 0, animRun = false, animLast = 0, txtLast = 0;
const wrap = d => ((d % 360) + 540) % 360 - 180;            // shortest signed angle
const compassDriving = () => CMP.on && CMP.raw != null && S.speed < CMP_SPEED;
function kickAnim(){ if (!animRun){ animRun = true; animLast = performance.now(); requestAnimationFrame(animFrame); } }
function rotateCar(){
  const root = carMarker?.getElement();
  if (root) root.classList.toggle('compass-on', compassDriving());
  kickAnim();
}
function animFrame(t){
  const dt = Math.min(.1, Math.max(0, (t - animLast) / 1000)); animLast = t;
  let busy = false;
  const carEl = carMarker?.getElement()?.querySelector('.car-rot');
  if (carEl){
    const d = wrap(S.heading - carRot);
    if (Math.abs(d) > .05){
      carRot += Math.abs(d) < .3 ? d : d * (1 - Math.exp(-dt / (compassDriving() ? .045 : .3)));
      busy = true;
    }
    carEl.style.transform = `rotate(${carRot.toFixed(2)}deg)`;
  }
  if (CMP.on && CMP.raw != null){
    const d = wrap(-CMP.raw - roseRot);
    if (Math.abs(d) > .05){ roseRot += Math.abs(d) < .3 ? d : d * (1 - Math.exp(-dt / .045)); busy = true; }
    roseEl.style.transform = `rotate(${roseRot.toFixed(2)}deg)`;
    if (t - txtLast > 120){ txtLast = t; const h = (CMP.raw + 360) % 360;
      cmpTxt.textContent = CARD16[Math.round(h / 22.5) % 16] + ' ' + Math.round(h) % 360 + '°'; }
  }
  if (busy || CMP.on) requestAnimationFrame(animFrame); else animRun = false;
}

/* ---------- live compass (device orientation) ---------- */
const CMP = { on:false, raw:null, got:false }, CMP_SPEED = 2.5; // below ~5.5 mph, trust the compass over GPS course
const CARD16 = ['N','NNE','NE','ENE','E','ESE','SE','SSE','S','SSW','SW','WSW','W','WNW','NW','NNW'];
const roseEl = $('#btnCompass .rose'), cmpTxt = $('#cmpTxt');
let scrAngle = screen.orientation?.angle ?? window.orientation ?? 0;
screen.orientation?.addEventListener?.('change', () => { scrAngle = screen.orientation.angle; });
window.addEventListener('orientationchange', () => { scrAngle = screen.orientation?.angle ?? window.orientation ?? 0; });
function onOrient(e){
  let h = null;
  if (typeof e.webkitCompassHeading === 'number' && e.webkitCompassHeading >= 0) h = e.webkitCompassHeading;   // iOS
  else if ((e.absolute || e.type === 'deviceorientationabsolute') && e.alpha != null) h = 360 - e.alpha;      // Android: alpha is CCW from north
  if (h == null) return;
  h = (h + scrAngle + 360) % 360;
  CMP.got = true;
  // Continuous (unwrapped) target so the animation never takes the long way round.
  CMP.raw = CMP.raw == null ? h : CMP.raw + wrap(h - CMP.raw);
  if (S.speed < CMP_SPEED) S.heading = (CMP.raw % 360 + 360) % 360;
  kickAnim();
}
async function compassOn(){
  try{
    if (typeof DeviceOrientationEvent !== 'undefined' && typeof DeviceOrientationEvent.requestPermission === 'function'){
      if (await DeviceOrientationEvent.requestPermission() !== 'granted'){ toast('Compass access was denied. Allow Motion & Orientation for this site.'); return; }
    }
  }catch{ toast('Compass access needs a tap on the compass button.'); return; }
  if (!('DeviceOrientationEvent' in window)){ toast('This device has no compass sensor.'); return; }
  CMP.on = true; CMP.got = false; CMP.raw = null;
  window.addEventListener('ondeviceorientationabsolute' in window ? 'deviceorientationabsolute' : 'deviceorientation', onOrient);
  const b = $('#btnCompass'); b.classList.remove('off'); b.setAttribute('aria-label', 'Turn off live compass');
  cmpTxt.textContent = '…';
  store.set('compass', true);
  kickAnim();
  setTimeout(() => { if (CMP.on && !CMP.got){ toast('No compass reading. This device may lack a magnetometer.'); compassOff(); } }, 3000);
}
function compassOff(){
  CMP.on = false; CMP.raw = null;
  window.removeEventListener('deviceorientationabsolute', onOrient); window.removeEventListener('deviceorientation', onOrient);
  const b = $('#btnCompass'); b.classList.add('off'); b.setAttribute('aria-label', 'Turn on live compass');
  roseRot = 0; roseEl.style.transform = ''; cmpTxt.textContent = 'Compass';
  store.set('compass', false); rotateCar();
}
$('#btnCompass').onclick = () => CMP.on ? compassOff() : compassOn();
function onPosition(c){
  const ll = [c.latitude, c.longitude], prev = S.pos, now = Date.now();
  let spd = (typeof c.speed === 'number' && !isNaN(c.speed)) ? c.speed : null;
  if (prev){
    const d = dist(prev, ll);
    if (spd == null && S.lastFix) spd = d / Math.max(.5, (now - S.lastFix)/1000);
    if (d > 2 && !(CMP.on && CMP.raw != null && (spd || 0) < CMP_SPEED)) S.heading =(typeof c.heading === 'number' && !isNaN(c.heading) && (spd||0) > 1) ? c.heading : bearing(prev, ll);
  }
  S.pos = ll; S.acc = c.accuracy; S.speed = spd || 0; S.lastFix = now;
  S.alt = (typeof c.altitude === 'number' && !isNaN(c.altitude)) ? c.altitude : null;
  drawCar(); updateSpeedo(); updateAlt(); checkReports(); maybeSpeedLimit();
  if (REC.on && (S.sim || (c.accuracy || 0) <= 50)) recPoint(ll, S.alt);
  if (PLACE) plSub();
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
    <span class="row-txt"><b>${esc(p.name)}</b><small>${esc(p.sub ?? (shortAddr(p.addr) || p.addr))}</small></span>
    ${d ? `<span class="row-d">${d}</span>` : ''}</button></li>`;
}

/* ---------- sheet views ---------- */
const sheet = $('#sheet');
function sheetHTML(h){ sheet.innerHTML = h; sheet.scrollTop = 0; }
function renderIdle(){
  catLayer.clearLayers();
  const h = store.get('home'), w = store.get('work'), rec = store.get('recents', []);
  S.list = rec;
  sheetHTML(`<div class="grab"></div>
    <div class="chips">
      <button class="chip" data-saved="home">${ico.home}<span><b>Home</b><small>${h ? esc(h.name) : 'Not set'}</small></span></button>
      <button class="chip" data-saved="work">${ico.work}<span><b>Work</b><small>${w ? esc(w.name) : 'Not set'}</small></span></button>
    </div>
    <div class="cats">${Object.entries(CATS).map(([k, C]) => `<button class="cat" data-cat="${k}"><i style="background:${C.color}">${C.svg}</i>${C.label}</button>`).join('')}</div>
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
  if (b.dataset.cat) return searchCat(b.dataset.cat);
  if (b.dataset.kind) return previewPlace(S.list[+b.dataset.i]);
  if (b.dataset.saved){ const p = store.get(b.dataset.saved); return p ? previewPlace(p) : toast('Search for a place, then tap “Set as '+(b.dataset.saved === 'home' ? 'Home' : 'Work')+'”.'); }
  if (b.dataset.route){ S.routeIdx = +b.dataset.route; drawRoutes(); renderPreview(); return; }
  const act = b.dataset.act;
  if (act === 'close') closePreview();
  else if (act === 'back') renderIdle();
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
  cancelPlace(); catLayer.clearLayers();
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

/* long-press to drop a pin: one finger only, cancelled by any pinch/zoom/drag */
let lpT = null, lpStart = null, lpFired = 0, lpMultiAt = 0, lpType = 'mouse';
const ptrs = new Set();
const mapEl = map.getContainer();
const lpCancel = () => { clearTimeout(lpT); lpT = null; };
mapEl.addEventListener('pointerdown', e => {
  ptrs.add(e.pointerId); lpType = e.pointerType;
  if (ptrs.size > 1){ lpMultiAt = Date.now(); return lpCancel(); }
  if (e.pointerType === 'mouse' && e.button !== 0) return;
  lpStart = [e.clientX, e.clientY];
  const ll = map.mouseEventToLatLng(e);
  lpCancel();
  lpT = setTimeout(() => {
    lpT = null;
    if (ptrs.size !== 1 || Date.now() - lpMultiAt < 800) return;
    lpFired = Date.now(); navigator.vibrate?.(15); dropPin(ll);
  }, 600);
}, true);
['pointerup','pointercancel'].forEach(t => mapEl.addEventListener(t, e => { ptrs.delete(e.pointerId); if (ptrs.size) lpMultiAt = Date.now(); lpCancel(); }, true));
mapEl.addEventListener('pointerleave', e => { if (e.pointerType === 'mouse'){ ptrs.delete(e.pointerId); lpCancel(); } }, true);
mapEl.addEventListener('touchstart', e => { if (e.touches.length > 1){ lpMultiAt = Date.now(); lpCancel(); } }, { capture:true, passive:true });
mapEl.addEventListener('pointermove', e => { if (lpT && lpStart && Math.hypot(e.clientX - lpStart[0], e.clientY - lpStart[1]) > 10) lpCancel(); }, true);
map.on('zoomstart dragstart', lpCancel);
// Right-click on desktop only; touch long-press is handled above (browsers also fire contextmenu for touch holds and two-finger taps).
map.on('contextmenu', e => {
  const oe = e.originalEvent;
  if ((oe?.pointerType && oe.pointerType !== 'mouse') || lpType !== 'mouse' || oe?.sourceCapabilities?.firesTouchEvents) return;
  if (Date.now() - lpFired > 1000 && Date.now() - lpMultiAt > 1000) dropPin(e.latlng);
});
function dropPin(ll){
  if (S.nav) return;
  if (PLACE){ PLACE.m.setLatLng(ll); plSub(); return; }
  startPlace({ ll, icon:ghostIcon, color:'#d93a2b', badge:ico.pin, title:'Drop a pin here?', okLabel:'Drop pin', onConfirm:dropPinConfirmed });
}
async function dropPinConfirmed(ll){
  const p ={ name:'Dropped pin', addr:`${ll.lat.toFixed(5)}, ${ll.lng.toFixed(5)}`, lat:ll.lat, lng:ll.lng };
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
  cancelPlace(); addRecent(S.dest);
  S.nav = { route, spoken:{}, off:0, lastReroute:0, along:0, k:0, rerouting:false };
  routeLayer.clearLayers(); drawNavRoute();
  $('#searchbar').hidden = true; $('#banner').hidden = false;
  sheet.hidden = true; $('#navbar').hidden = false; renderLegend();
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
  S.nav = null; renderLegend();
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
  closeModal();
  const T = RT[type], ll = S.pos || [map.getCenter().lat, map.getCenter().lng];
  startPlace({ ll, color:T.color, badge:T.svg, title:'Report '+T.label.toLowerCase()+' here?', okLabel:'Confirm report',
    icon:L.divIcon({ className:'rep place', iconSize:[40,40], iconAnchor:[20,20], html:`<div style="background:${T.color}">${T.svg}</div>` }),
    onConfirm: p => {
      reports.push({ id:Math.random().toString(36).slice(2, 10), type, lat:p.lat, lng:p.lng, t:Date.now() });
      store.set('reports', reports);
      if (!settings.ovReports){ settings.ovReports = true; saveSettings(); applyOverlays(); }
      drawReports(); toast(T.label+' reported. Thanks!');
    } });
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
$('#btnReport').onclick = () => openModal(`<h3 class="label" style="margin-top:0">What do you see?</h3>
  <div class="rgrid">${Object.entries(RT).map(([k, T]) => `<button class="rbtn" data-rep="${k}"><i style="background:${T.color}">${T.svg}</i>${T.label}</button>`).join('')}</div>
  <p class="fine">You’ll confirm the spot on the map next. Reports stay on this device for 3 hours.</p>`);
function seg(key, opts){ return `<div class="seg">${opts.map(([v, l]) => `<button data-set="${key}" data-val="${v}" class="${String(settings[key]) === String(v) ? 'on' : ''}">${l}</button>`).join('')}</div>`; }
function tg(k, title, sub){ return `<button class="tg" role="switch" aria-checked="${!!settings[k]}" data-tg="${k}"><span class="row-txt"><b>${title}</b><small>${sub}</small></span><span class="sw"></span></button>`; }
function renderLayers(){
  const keep = mSheet.scrollTop, wasOpen = !modal.hidden && mSheet.querySelector('.bases');
  openModal(`<h3 class="label" style="margin-top:0">Map style</h3>
    <div class="bases">${[['street','Street'],['topo','Terrain'],['satellite','Satellite']].map(([k, l]) =>
      `<button class="base${(BASES[settings.base] ? settings.base : 'street') === k ? ' on' : ''}" data-base="${k}"><i class="th-${k}"></i>${l}</button>`).join('')}</div>
    <h3 class="label">On the map</h3>
    ${tg('ovOffroad', 'Off-road tracks', '4×4, ATV and dirt tracks colored by grade. Tap a track for details.')}
    ${tg('ovHike', 'Hiking routes', 'Marked hiking routes from Waymarked Trails')}
    ${tg('ovMtb', 'Mountain bike routes', 'Marked MTB routes from Waymarked Trails')}
    ${tg('ovReports', 'Road reports', 'Police, crashes and hazards you reported')}
    <h3 class="label">Tools</h3>
    <button class="tg" data-tool="rec"><span class="row-txt"><b>${REC.on ? 'Stop recording' : 'Record track'}</b><small>${REC.on ? fd(REC.dist)+' recorded so far' : 'Draw your path on the map and save it as a GPX file'}</small></span>
      <span class="rec-dot${REC.on ? ' on' : ''}"></span></button>
    <h3 class="label">Settings</h3>
    <div class="set-row"><span>Units</span>${seg('units', [['imperial','Miles'],['metric','Kilometers']])}</div>
    <div class="set-row"><span>Voice guidance</span>${seg('voice', [['true','On'],['false','Off']])}</div>
    <div class="set-row"><span>Appearance</span>${seg('theme', [['auto','Auto'],['light','Day'],['dark','Night']])}</div>
    <div class="set-row"><span>Saved data</span><div class="seg"><button data-clear="places">Clear places</button><button data-clear="reports">Clear reports</button></div></div>
    <p class="fine">Map data © OpenStreetMap contributors · terrain © OpenTopoMap (CC-BY-SA) · imagery © Esri, Maxar · routes © Waymarked Trails · search by Nominatim · routing by OSRM · trails, places and speed limits from Overpass. These are free public servers meant for light personal use.</p>`);
  if (wasOpen) mSheet.scrollTop = keep;
}
$('#btnLayers').onclick = renderLayers;
mSheet.addEventListener('click', e => {
  const b = e.target.closest('button'); if (!b) return;
  if (b.dataset.rep) return addReport(b.dataset.rep);
  if (b.dataset.base){ settings.base = b.dataset.base; saveSettings(); applyTheme(); renderLayers(); return; }
  if (b.dataset.tg){ const k = b.dataset.tg; settings[k] = !settings[k]; saveSettings(); applyOverlays(); renderLayers(); return; }
  if (b.dataset.tool === 'rec'){ if (REC.on) recStop(); else { closeModal(); recStart(); } return; }
  if (b.dataset.rec === 'save'){ saveGPX(); return; }
  if (b.dataset.rec === 'clear'){ if (REC.line){ map.removeLayer(REC.line); REC.line = null; } REC.pts = []; closeModal(); return; }
  if (b.dataset.rec === 'done'){ closeModal(); return; }
  if (b.dataset.set){
    const k = b.dataset.set; let v = b.dataset.val; if (v === 'true' || v === 'false') v = v === 'true';
    settings[k] = v; saveSettings();
    if (k === 'theme') applyTheme();
    if (k === 'units'){ updateSpeedo(); updateAlt(); if (S.nav && S.pos) navTick(); else if (!S.nav) S.dest ? renderPreview() : renderIdle(); }
    if (k === 'voice' && !v) speechSynthesis?.cancel?.();
    renderLayers(); return;
  }
  if (b.dataset.clear === 'places'){ ['home','work','recents'].forEach(k => store.set(k, null)); if (!S.dest && !S.nav) renderIdle(); toast('Saved places cleared'); }
  if (b.dataset.clear === 'reports'){ reports = []; store.set('reports', []); drawReports(); toast('Reports cleared'); }
});

/* ---------- confirm-to-place (pins and reports) ---------- */
let PLACE = null;
const ghostIcon = L.divIcon({ className:'pin ghost', iconSize:[34,44], iconAnchor:[17,42],
  html:'<svg viewBox="0 0 34 44"><path d="M17 1C8.2 1 1.5 7.6 1.5 16.2 1.5 27.5 17 43 17 43s15.5-15.5 15.5-26.8C32.5 7.6 25.8 1 17 1z" fill="#d93a2b" stroke="#fff" stroke-width="2"/><circle cx="17" cy="16" r="5.5" fill="#fff"/></svg>' });
function startPlace(o){
  cancelPlace();
  const m = L.marker(o.ll, { icon:o.icon, draggable:true, autoPan:true, zIndexOffset:2000 }).addTo(map);
  PLACE = { ...o, m };
  $('#place').innerHTML = `<div class="pl-head"><span class="pl-badge" style="background:${o.color}">${o.badge}</span>
      <div class="row-txt"><b class="pl-title">${esc(o.title)}</b><span class="pl-sub" id="plSub"></span></div></div>
    <p class="pl-tip">Drag the marker or tap the map to move it.</p>
    <div class="pl-actions"><button class="btn" data-pl="cancel">Cancel</button><button class="btn primary" data-pl="ok">${esc(o.okLabel)}</button></div>`;
  $('#place').hidden = false; document.body.classList.add('placing');
  m.on('drag dragend', plSub); plSub();
  const sz = map.getSize(), pt = map.latLngToContainerPoint(o.ll);
  if (pt.x < 40 || pt.x > sz.x - 40 || pt.y < 140 || pt.y > sz.y - 260) map.panTo(o.ll);
}
function plSub(){
  if (!PLACE) return;
  const ll = PLACE.m.getLatLng();
  $('#plSub').textContent = `${ll.lat.toFixed(5)}, ${ll.lng.toFixed(5)}` + (S.pos ? ' · '+fd(dist(S.pos, [ll.lat, ll.lng]))+' away' : '');
}
function cancelPlace(){
  if (!PLACE) return;
  map.removeLayer(PLACE.m); PLACE = null;
  $('#place').hidden = true; document.body.classList.remove('placing');
}
$('#place').addEventListener('click', e => {
  const b = e.target.closest('[data-pl]'); if (!b || !PLACE) return;
  const p = PLACE, ll = p.m.getLatLng();
  cancelPlace();
  if (b.dataset.pl === 'ok') p.onConfirm(ll);
});
map.on('click', e => { if (PLACE){ PLACE.m.setLatLng(e.latlng); plSub(); } });
document.addEventListener('keydown', e => { if (e.key === 'Escape'){ if (PLACE) cancelPlace(); else if (!modal.hidden) closeModal(); } });

/* ---------- base maps & overlays ---------- */
map.createPane('overTiles').style.zIndex = 250; map.getPane('overTiles').style.pointerEvents = 'none';
map.createPane('trails').style.zIndex = 390;
const ESRI = 'https://server.arcgisonline.com/ArcGIS/rest/services/';
const BASES = {
  street:{ url:'https://tile.openstreetmap.org/{z}/{x}/{y}.png', o:{ maxZoom:19, attribution:TILE_ATTR } },
  topo:{ url:'https://{s}.tile.opentopomap.org/{z}/{x}/{y}.png', o:{ maxZoom:19, maxNativeZoom:17, subdomains:'abc',
    attribution:TILE_ATTR+', SRTM · style &copy; <a href="https://opentopomap.org">OpenTopoMap</a> (CC-BY-SA)' } },
  satellite:{ url:ESRI+'World_Imagery/MapServer/tile/{z}/{y}/{x}', o:{ maxZoom:19, maxNativeZoom:18, attribution:'Imagery &copy; Esri, Maxar, Earthstar Geographics' } }
};
let baseLayer = null, baseKey = null, satLabels = null;
function setBase(){
  const key = BASES[settings.base] ? settings.base : 'street';
  if (key === baseKey) return;
  if (baseLayer) map.removeLayer(baseLayer);
  if (satLabels){ map.removeLayer(satLabels); satLabels = null; }
  baseLayer = L.tileLayer(BASES[key].url, BASES[key].o).addTo(map); baseKey = key;
  if (key === 'satellite') satLabels = L.layerGroup([
    L.tileLayer(ESRI+'Reference/World_Transportation/MapServer/tile/{z}/{y}/{x}', { pane:'overTiles', maxZoom:19, maxNativeZoom:18 }),
    L.tileLayer(ESRI+'Reference/World_Boundaries_and_Places/MapServer/tile/{z}/{y}/{x}', { pane:'overTiles', maxZoom:19, maxNativeZoom:18 })
  ]).addTo(map);
}
const WM = { hike:null, mtb:null };
function wmLayer(k, on){
  if (on && !WM[k]) WM[k] = L.tileLayer(`https://tile.waymarkedtrails.org/${k === 'hike' ? 'hiking' : 'mtb'}/{z}/{x}/{y}.png`,
    { pane:'overTiles', maxZoom:19, maxNativeZoom:18, opacity:.85, attribution:'Routes &copy; <a href="https://waymarkedtrails.org">Waymarked Trails</a>' }).addTo(map);
  else if (!on && WM[k]){ map.removeLayer(WM[k]); WM[k] = null; }
}
function applyOverlays(){
  wmLayer('hike', settings.ovHike); wmLayer('mtb', settings.ovMtb);
  if (settings.ovReports) reportLayer.addTo(map); else map.removeLayer(reportLayer);
  if (settings.ovOffroad){ OR.layer.addTo(map); loadTrails(); } else { map.removeLayer(OR.layer); clearTrailHL(); }
  renderLegend();
}

/* ---------- off-road tracks (OSM via Overpass) ---------- */
const GRADES = {
  grade1:['#2f9e44','Grade 1','Solid surface, gravel or paved'],
  grade2:['#94c11f','Grade 2','Mostly solid gravel'],
  grade3:['#e6ac00','Grade 3','Mix of hard and soft ground'],
  grade4:['#f07818','Grade 4','Mostly soft dirt or grass'],
  grade5:['#d6332a','Grade 5','Soft ground, high clearance']
};
const OR_UNK = '#9a7b55', OR_4WD = '#8e44c9', OR_CLOSED = '#8d969b', OR_MIN_Z = 11.5;
const OR = { layer:L.layerGroup(), ways:new Map(), loaded:null, busy:false, status:'', hl:null, sel:null };
const trailRenderer = L.canvas({ pane:'trails', padding:.4, tolerance:10 });
const isClosed = t => /^(private|no)$/.test(t.access || '') || /^(private|no)$/.test(t.motor_vehicle || '');
function trailStyle(t){
  const closed = isClosed(t), fwd = t['4wd_only'] === 'yes';
  return { color: closed ? OR_CLOSED : fwd ? OR_4WD : (GRADES[t.tracktype]?.[0] || OR_UNK),
    weight: fwd ? 5 : 4, opacity: closed ? .75 : .95, lineCap:'round',
    dashArray: closed ? '2 8' : (t.highway === 'path' || t.highway === 'bridleway') ? '9 6' : null };
}
let trailT = null;
map.on('moveend', () => { if (!settings.ovOffroad) return; clearTimeout(trailT); trailT = setTimeout(loadTrails, 600); });
async function loadTrails(){
  if (!settings.ovOffroad) return;
  if (map.getZoom() < OR_MIN_Z){ OR.status = 'zoom'; return renderLegend(); }
  if (OR.loaded && OR.loaded.contains(map.getBounds())){ OR.status = 'ok'; return renderLegend(); }
  if (OR.busy) return;
  const pb = map.getBounds().pad(.3);
  if (OR.ways.size > 8000){ OR.layer.clearLayers(); OR.ways.clear(); }
  OR.busy = true; OR.status = 'loading'; renderLegend();
  const bb = [pb.getSouth(), pb.getWest(), pb.getNorth(), pb.getEast()].map(n => n.toFixed(5)).join(',');
  const V = '"^(yes|permissive|designated)$"';
  const ql = `[out:json][timeout:25][bbox:${bb}];(way[highway=track];way["4wd_only"=yes];`
    + `way[highway~"^(path|bridleway)$"][motor_vehicle~${V}];way[highway~"^(path|track|unclassified)$"][atv~${V}];`
    + `way[highway~"^(path|track)$"][motorcycle~${V}];);out geom 5000;`;
  try{
    const r = await fetch('https://overpass-api.de/api/interpreter', { method:'POST', body:'data='+encodeURIComponent(ql),
      headers:{ 'Content-Type':'application/x-www-form-urlencoded' } });
    if (!r.ok) throw new Error();
    const j = await r.json();
    for (const w of j.elements || []) if (w.type === 'way' && w.geometry && !OR.ways.has(w.id)) addTrail(w);
    OR.loaded = pb; OR.status = 'ok';
  }catch{ OR.status = 'error'; }
  OR.busy = false; renderLegend();
  if (OR.status === 'ok' && settings.ovOffroad && map.getZoom() >= OR_MIN_Z && !OR.loaded.contains(map.getBounds())) setTimeout(loadTrails, 300);
}
function addTrail(w){
  const ll = w.geometry.map(g => [g.lat, g.lon]), st = trailStyle(w.tags || {});
  const line = L.polyline(ll, { renderer:trailRenderer, ...st });
  line.on('click', e => { L.DomEvent.stop(e); openTrail(w, ll, e.latlng); });
  const g = L.layerGroup([L.polyline(ll, { renderer:trailRenderer, color:'#ffffff', weight:st.weight + 3, opacity:.55, interactive:false }), line]);
  g.addTo(OR.layer); OR.ways.set(w.id, g);
}
const cap = s => s ? String(s).replace(/_/g, ' ').replace(/^./, c => c.toUpperCase()) : '';
function openTrail(w, ll, at){
  if (PLACE){ PLACE.m.setLatLng(at); plSub(); return; }
  const t = w.tags || {}, g = GRADES[t.tracktype], closed = isClosed(t), fwd = t['4wd_only'] === 'yes';
  let len = 0; for (let i = 1; i < ll.length; i++) len += dist(ll[i-1], ll[i]);
  const kind = fwd ? '4×4 only track' : t.highway === 'track' ? 'Dirt track' : t.highway === 'bridleway' ? 'Bridleway' : t.highway === 'path' ? 'Trail' : 'Road';
  const veh = [['motor_vehicle','Vehicles'],['atv','ATV'],['motorcycle','Motorcycle']].filter(([k]) => t[k]).map(([k, l]) => l+': '+t[k]).join(', ');
  const rows = [['Grade', g ? g[1]+' · '+g[2] : 'Not graded'], ['Surface', cap(t.surface)], ['Condition', cap(t.smoothness)],
    ['Access', closed ? 'Closed or private' : cap(t.access) || 'Not posted'], ['Allowed', veh], ['Length', fd(len)]].filter(r => r[1]);
  const name = t.name || t.ref || 'Unnamed '+kind.toLowerCase();
  clearTrailHL();
  OR.hl = L.polyline(ll, { renderer:trailRenderer, color:'#ffe14d', weight:12, opacity:.6, interactive:false }).addTo(map);
  OR.sel = { ll, name };
  L.popup({ maxWidth:280 }).setLatLng(at).setContent(`<div class="pop"><b>${esc(name)}</b><small>${kind}</small>
    ${g || fwd || closed ? `<span class="tp-grade" style="background:${trailStyle(t).color}">${closed ? 'Closed' : fwd ? '4×4 only' : g[1]}</span>` : ''}
    <dl>${rows.map(([k, v]) => `<dt>${k}</dt><dd>${esc(v)}</dd>`).join('')}</dl>
    <button class="go" data-trail-go>Directions to trail</button></div>`).openOn(map);
}
function clearTrailHL(){ if (OR.hl){ map.removeLayer(OR.hl); OR.hl = null; } }
map.on('popupclose', clearTrailHL);
map.on('popupopen', e => {
  const b = e.popup.getElement()?.querySelector('[data-trail-go]'); if (!b) return;
  b.onclick = () => {
    const s = OR.sel; if (!s) return;
    const from = S.pos || [map.getCenter().lat, map.getCenter().lng], a = s.ll[0], z = s.ll[s.ll.length - 1];
    const end = dist(from, a) <= dist(from, z) ? a : z;
    map.closePopup();
    previewPlace({ name:s.name, addr:'Off-road trail · nearest end', lat:end[0], lng:end[1] });
  };
});
function renderLegend(){
  const el = $('#legend');
  if (!settings.ovOffroad || S.nav){ el.hidden = true; return; }
  const st = { zoom:'Zoom in to load trails', loading:'Loading trails…', error:'Trails didn’t load. Move the map to retry.' }[OR.status] || '';
  el.innerHTML = '<span class="lg-t">Off-road</span>'
    + Object.values(GRADES).map((g, i) => `<span class="lg" title="${g[1]}: ${g[2]}"><i style="background:${g[0]}"></i>${i + 1}</span>`).join('')
    + `<span class="lg"><i style="background:${OR_4WD}"></i>4×4</span><span class="lg"><i class="dash" style="--c:${OR_CLOSED}"></i>Closed</span>`
    + (st ? `<span class="lg-status${OR.status === 'loading' ? ' busy' : ''}">${st}</span>` : '');
  el.hidden = false;
}

/* ---------- nearby categories ---------- */
const CATS = {
  fuel:{ label:'Gas', color:'#c8312b', r:10000, q:'nwr[amenity=fuel]',
    svg:'<svg viewBox="0 0 24 24" fill="currentColor"><path d="M5 3h9v18H5zm2 2v5h5V5zm9.5 3.5l3 3V19a1.5 1.5 0 0 1-3 0v-4H15v-2h2a1 1 0 0 1 1 1v5h.5v-7l-3-3z"/></svg>' },
  ev:{ label:'EV charging', color:'#1f9d55', r:15000, q:'nwr[amenity=charging_station]',
    svg:'<svg viewBox="0 0 24 24" fill="currentColor"><path d="M13 2L4 14h6l-1 8 9-12h-6z"/></svg>' },
  food:{ label:'Food', color:'#e07a10', r:5000, q:'nwr[amenity~"^(restaurant|fast_food|cafe)$"]',
    svg:'<svg viewBox="0 0 24 24" fill="currentColor"><path d="M6 2v8a2 2 0 0 0 1.5 1.9V22h2V11.9A2 2 0 0 0 11 10V2H9.5v6h-1V2h-1v6h-1V2zm10 0c-1.7 0-3 2.2-3 5.5V13h2v9h2V2z"/></svg>' },
  parking:{ label:'Parking', color:'#2557c9', r:3000, q:'nwr[amenity=parking][access!~"^(private|no|customers)$"]',
    svg:'<svg viewBox="0 0 24 24" fill="currentColor"><path d="M6 3h7a5 5 0 0 1 0 10h-3v8H6zm4 4v2.8h3a1.4 1.4 0 0 0 0-2.8z"/></svg>' },
  trailhead:{ label:'Trailheads', color:'#7a5a2e', r:30000, q:'nwr[highway=trailhead]',
    svg:'<svg viewBox="0 0 24 24" fill="currentColor"><path d="M2 20L9 7l4 7 2.5-3.5L22 20z"/></svg>' },
  camp:{ label:'Camping', color:'#2f7d32', r:30000, q:'nwr[tourism~"^(camp_site|caravan_site)$"]',
    svg:'<svg viewBox="0 0 24 24" fill="currentColor"><path d="M12 3l-1.2 2.1L2 20h8l2-4 2 4h8L13.2 5.1z"/></svg>' }
};
const catLayer = L.layerGroup().addTo(map);
async function searchCat(k){
  const C = CATS[k]; if (!C) return;
  const c = S.pos || [map.getCenter().lat, map.getCenter().lng];
  catLayer.clearLayers();
  sheetHTML(`<div class="grab"></div><p class="hint">Finding ${C.label.toLowerCase()} nearby…</p>`);
  const ql = `[out:json][timeout:20];${C.q}(around:${C.r},${c[0].toFixed(5)},${c[1].toFixed(5)});out center tags 80;`;
  try{
    let r;
    try{ r = await fetch('https://overpass-api.de/api/interpreter', { method:'POST', body:'data='+encodeURIComponent(ql), headers:{ 'Content-Type':'application/x-www-form-urlencoded' } }); }
    catch{ throw new Error('Could not reach the place search. Check your connection.'); }
    if (!r.ok) throw new Error('Place search returned '+r.status+'. Try again in a moment.');
    const j = await r.json();
    S.list = (j.elements || []).map(e => {
      const t = e.tags || {}, street = [t['addr:housenumber'], t['addr:street']].filter(Boolean).join(' ');
      const where = [street, t['addr:city']].filter(Boolean).join(', ');
      return { name:t.name || t.brand || t.operator || C.label, sub:where || C.label, addr:where || C.label, lat:e.lat ?? e.center?.lat, lng:e.lon ?? e.center?.lon };
    }).filter(p => p.lat != null).sort((a, b) => dist(c, [a.lat, a.lng]) - dist(c, [b.lat, b.lng])).slice(0, 25);
    S.list.forEach(p => L.marker([p.lat, p.lng], { icon:L.divIcon({ className:'rep cat-pin', iconSize:[28,28], iconAnchor:[14,14],
      html:`<div style="background:${C.color}">${C.svg}</div>` }) }).on('click', () => previewPlace(p)).addTo(catLayer));
    sheetHTML(`<div class="grab"></div><div class="pv-head"><div class="row-txt"><b class="pv-name">${C.label} nearby</b>
        <small>${S.list.length ? S.list.length+' within '+fd(C.r) : 'None within '+fd(C.r)}</small></div>
        <button class="icon-btn" data-act="back" aria-label="Back">×</button></div>`
      + (S.list.length ? `<ul class="list">${S.list.map((p, i) => row(p, 'res', i, `<span style="color:${C.color};display:grid">${C.svg}</span>`)).join('')}</ul>`
                       : '<p class="hint">Nothing found. Move the map somewhere else and try again.</p>'));
    if (S.list.length){
      S.follow = false; $('#btnRecenter').hidden = !S.pos;
      map.fitBounds(L.latLngBounds(S.list.slice(0, 10).map(p => [p.lat, p.lng]).concat([c])),
        { paddingTopLeft:[30, $('#top').offsetHeight + 30], paddingBottomRight:[70, sheet.offsetHeight + 30], maxZoom:16 });
    }
  }catch(e){ sheetHTML(`<div class="grab"></div><p class="err">${esc(e.message)}</p><button class="link" data-act="back">Back</button>`); }
}

/* ---------- elevation ---------- */
function updateAlt(){
  const el = $('#alt');
  if (S.alt == null){ el.hidden = true; return; }
  el.hidden = false;
  $('#altVal').textContent = Math.round(imperial() ? S.alt*3.28084 : S.alt).toLocaleString() + (imperial() ? ' ft' : ' m');
}

/* ---------- track recorder ---------- */
const REC = { on:false, pts:[], line:null, t0:0, dist:0, climb:0, timer:null };
const hms = s => { s = Math.round(s); const h = Math.floor(s/3600), m = Math.floor(s%3600/60), x = s%60; return (h ? h+':'+String(m).padStart(2,'0') : m)+':'+String(x).padStart(2,'0'); };
function recStart(){
  Object.assign(REC, { on:true, pts:[], t0:Date.now(), dist:0, climb:0 });
  if (REC.line) map.removeLayer(REC.line);
  REC.line = L.polyline([], { color:'#ff6a00', weight:5, opacity:.9, dashArray:'1 9', lineCap:'round', interactive:false }).addTo(map);
  $('#recPill').hidden = false; clearInterval(REC.timer); REC.timer = setInterval(renderRec, 1000); renderRec();
  if (S.pos) recPoint(S.pos, S.alt);
  toast('Recording your track');
}
function recPoint(ll, alt){
  const last = REC.pts[REC.pts.length - 1];
  if (last){
    const d = dist(last, ll); if (d < 5) return;
    REC.dist += d;
    if (alt != null && last[2] != null && alt - last[2] > 1) REC.climb += alt - last[2];
  }
  REC.pts.push([ll[0], ll[1], alt ?? null, Date.now()]); REC.line.addLatLng(ll);
}
function renderRec(){ $('#recPill').textContent = 'REC  '+fd(REC.dist)+' · '+hms((Date.now() - REC.t0)/1000); }
function recStop(){
  REC.on = false; clearInterval(REC.timer); $('#recPill').hidden = true;
  const secs = (Date.now() - REC.t0)/1000;
  const climb = REC.pts.some(p => p[2] != null) ? Math.round(imperial() ? REC.climb*3.28084 : REC.climb).toLocaleString()+(imperial() ? ' ft' : ' m') : '—';
  openModal(`<h3 class="label" style="margin-top:0">Track recorded</h3>
    <div class="stats"><div><b>${fd(REC.dist)}</b><span>Distance</span></div><div><b>${hms(secs)}</b><span>Time</span></div><div><b>${climb}</b><span>Climb</span></div></div>
    <div class="pv-actions"><button class="btn primary" data-rec="save"${REC.pts.length < 2 ? ' disabled' : ''}>Save GPX file</button><button class="btn" data-rec="clear">Clear</button></div>
    <button class="link" data-rec="done" style="margin-top:6px">Keep on map</button>`);
}
function saveGPX(){
  const pts = REC.pts.map(p => `<trkpt lat="${p[0].toFixed(7)}" lon="${p[1].toFixed(7)}">${p[2] != null ? `<ele>${p[2].toFixed(1)}</ele>` : ''}<time>${new Date(p[3]).toISOString()}</time></trkpt>`).join('\n');
  const xml = `<?xml version="1.0" encoding="UTF-8"?>\n<gpx version="1.1" creator="Guidepost Navigator" xmlns="http://www.topografix.com/GPX/1/1">\n<trk><name>Track ${new Date(REC.t0).toLocaleString()}</name><trkseg>\n${pts}\n</trkseg></trk>\n</gpx>\n`;
  const a = document.createElement('a');
  a.href = URL.createObjectURL(new Blob([xml], { type:'application/gpx+xml' }));
  a.download = 'track-'+new Date(REC.t0).toISOString().slice(0, 16).replace(/[:T]/g, '-')+'.gpx';
  document.body.appendChild(a); a.click(); a.remove();
  setTimeout(() => URL.revokeObjectURL(a.href), 5000);
  toast('GPX file saved');
}
$('#recPill').onclick = recStop;

/* ---------- boot ---------- */
applyTheme(); renderIdle(); drawReports(); updateSpeedo(); startGPS();
// Android grants orientation without a prompt, so restore the compass; iOS needs a tap each visit.
if (store.get('compass', false) && typeof DeviceOrientationEvent !== 'undefined' && typeof DeviceOrientationEvent.requestPermission !== 'function') compassOn();
</script>
</body>
</html>
