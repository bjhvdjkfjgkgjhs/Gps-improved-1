<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1,viewport-fit=cover">
<meta name="theme-color" content="#006b54">
<meta name="apple-mobile-web-app-capable" content="yes">
<title>Guidepost Navigator</title>
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/mapbox-gl@3.8.0/dist/mapbox-gl.css">
<link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Overpass:wght@400;600;800&family=Overpass+Mono:wght@600;700&display=swap">
<style>
/* Layout: full-bleed map; road-sign banner on top; driving cluster bottom-left; actions bottom-right; bottom sheet. */
:root{
  --bg:#e9eeeb; --panel:#ffffff; --panel-2:#f1f5f3; --ink:#13201b; --muted:#5b6a63; --line:#d3dcd8;
  --guide:#006b54; --guide-ink:#ffffff; --warn:#ffc20e; --warn-ink:#1a1a1a; --danger:#c8312b;
  --route:#1a9be6; --route-alt:#8fa7c6; --casing:#ffffff; --sky:#d7e4ec; --wz:#33ccff;
  --font:'Overpass',system-ui,-apple-system,'Segoe UI',sans-serif;
  --mono:'Overpass Mono',ui-monospace,'SF Mono',Menlo,monospace;
  --bottom:180px; --shadow:0 6px 22px rgba(10,25,20,.22);
}
@media (prefers-color-scheme: dark){:root:not([data-theme="light"]){
  --bg:#0d1311; --panel:#18211e; --panel-2:#212c28; --ink:#e6eeea; --muted:#93a39c; --line:#2b3733;
  --guide:#00785e; --route:#33ccff; --route-alt:#4c5f78; --casing:#0b3d52; --sky:#0a121c; color-scheme:dark; --shadow:0 6px 22px rgba(0,0,0,.5);
}}
:root[data-theme="dark"]{
  --bg:#0d1311; --panel:#18211e; --panel-2:#212c28; --ink:#e6eeea; --muted:#93a39c; --line:#2b3733;
  --guide:#00785e; --route:#33ccff; --route-alt:#4c5f78; --casing:#0b3d52; --sky:#0a121c; color-scheme:dark; --shadow:0 6px 22px rgba(0,0,0,.5);
}
*{box-sizing:border-box}
html,body{height:100%;margin:0}
body{background:var(--bg);color:var(--ink);font-family:var(--font);font-size:15px;overflow:hidden;-webkit-tap-highlight-color:transparent;-webkit-font-smoothing:antialiased}
button{font:inherit;color:inherit;cursor:pointer;border:0;background:none}
button:focus-visible,input:focus-visible{outline:3px solid var(--route);outline-offset:2px}
#mapWrap{position:fixed;inset:0;z-index:0;overflow:hidden;background:var(--sky)}
#map{position:absolute;inset:0;background:var(--bg)}
.mapboxgl-ctrl-bottom-left,.mapboxgl-ctrl-bottom-right{bottom:var(--bottom)!important;transition:bottom .2s}
.mapboxgl-ctrl-attrib{font-size:10px}
.gp-icon{position:absolute;top:0;left:0}
.gp-pop .mapboxgl-popup-content{background:var(--panel);color:var(--ink);border-radius:14px;padding:12px 14px;box-shadow:var(--shadow);font-family:var(--font)}
.gp-pop .mapboxgl-popup-close-button{color:var(--muted);font-size:20px;padding:2px 8px}
.gp-pop.mapboxgl-popup-anchor-bottom .mapboxgl-popup-tip,.gp-pop.mapboxgl-popup-anchor-bottom-left .mapboxgl-popup-tip,.gp-pop.mapboxgl-popup-anchor-bottom-right .mapboxgl-popup-tip{border-top-color:var(--panel)}
.gp-pop.mapboxgl-popup-anchor-top .mapboxgl-popup-tip,.gp-pop.mapboxgl-popup-anchor-top-left .mapboxgl-popup-tip,.gp-pop.mapboxgl-popup-anchor-top-right .mapboxgl-popup-tip{border-bottom-color:var(--panel)}
.gp-pop.mapboxgl-popup-anchor-left .mapboxgl-popup-tip{border-right-color:var(--panel)}
.gp-pop.mapboxgl-popup-anchor-right .mapboxgl-popup-tip{border-left-color:var(--panel)}
/* first-run Mapbox token screen */
#mbSetup{position:fixed;inset:0;z-index:2000;display:grid;place-items:center;background:var(--bg);padding:24px max(16px,env(safe-area-inset-right,0px)) 24px max(16px,env(safe-area-inset-left,0px));overflow:auto}
.mbs-card{width:min(460px,100%);display:flex;flex-direction:column;gap:12px}
.mbs-card h1{margin:0;font:800 28px var(--font);text-wrap:balance}
.mbs-card p,.mbs-card ol{margin:0;line-height:1.5;color:var(--muted)}
.mbs-card ol{padding-left:20px;display:flex;flex-direction:column;gap:4px}
.mbs-card a{color:var(--route)}
.mbs-card code{font-family:var(--mono);background:var(--panel-2);padding:0 4px;border-radius:4px}
.mbs-card label{font-weight:700;margin-top:6px}
.mbs-card input{height:48px;border-radius:12px;border:1.5px solid var(--line);background:var(--panel);color:var(--ink);padding:0 14px;font:600 14px var(--mono);min-width:0}
.mbs-err{color:var(--danger)!important;font-weight:600;min-height:1.2em}
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
#legend{display:flex;flex-direction:column;align-items:stretch;gap:6px;background:var(--panel);border-radius:12px;box-shadow:var(--shadow);padding:7px 10px;font:700 11px var(--font)}
#legend .lg-t{font:800 10px var(--font);letter-spacing:.1em;text-transform:uppercase;color:var(--muted);margin-right:2px}
#legend .lg{display:inline-flex;align-items:center;gap:4px;padding:2px 6px;border-radius:6px;background:var(--panel-2);font-variant-numeric:tabular-nums}
#legend i{width:14px;height:4px;border-radius:2px;display:inline-block}
#legend i.dash{background:repeating-linear-gradient(90deg,var(--c) 0 3px,transparent 3px 6px)}
#legend .lg-status{color:var(--muted);font-weight:600;min-width:0;overflow:hidden;text-overflow:ellipsis;white-space:nowrap}
#legend .lg-head{display:flex;align-items:center;gap:8px;width:100%;text-align:left;font:inherit;min-height:22px}
#legend .lg-car{margin-left:auto;flex:none;color:var(--route);font-weight:700}
#legend .lg-row{display:flex;flex-wrap:wrap;align-items:center;gap:5px}
#legend .lg-k{font:800 9.5px var(--font);letter-spacing:.08em;text-transform:uppercase;color:var(--muted);width:72px}
.subs{margin-left:4px;padding-left:14px;border-left:3px solid var(--line)}
.subs .tg b{font-weight:600}
#legend .lg-status.busy::before{content:"";display:inline-block;width:8px;height:8px;margin-right:6px;border-radius:50%;border:2px solid var(--route);border-right-color:transparent;animation:spin .8s linear infinite;vertical-align:-1px}
@keyframes spin{to{transform:rotate(360deg)}}
#recPill{align-self:center;display:inline-flex;align-items:center;gap:8px;background:var(--danger);color:#fff;border-radius:999px;padding:8px 16px;font:700 13px var(--mono);box-shadow:var(--shadow);font-variant-numeric:tabular-nums;white-space:pre}
#recPill::before{content:"";width:9px;height:9px;border-radius:50%;background:#fff;animation:blink 1s infinite}
@keyframes blink{50%{opacity:.25}}

#tkey{display:flex;align-items:center;gap:6px;flex-wrap:wrap;background:var(--panel);border-radius:12px;box-shadow:var(--shadow);padding:6px 10px;font:700 11px var(--font)}
#tkey .lg-t{font:800 10px var(--font);letter-spacing:.1em;text-transform:uppercase;color:var(--muted);margin-right:2px}
#tkey .tk{display:inline-flex;align-items:center;gap:4px}
#tkey i{width:14px;height:5px;border-radius:3px;display:inline-block}
.keyrow{display:flex;flex-direction:column;gap:6px;padding:10px 0}
.keyrow label{font-weight:700}
.keyin{display:flex;gap:8px}
.keyin input{flex:1;min-width:0;height:44px;border-radius:10px;border:1.5px solid var(--line);background:var(--panel-2);color:var(--ink);padding:0 12px;font:600 14px var(--mono)}
.keyin .btn{height:44px;padding:0 18px}
.keyrow small{color:var(--muted);font-size:12px;line-height:1.45}

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

/* Lanes */
.b-lanes{display:flex;gap:2px;margin-top:10px;padding:4px 6px;border-radius:10px;background:rgba(0,0,0,.24);width:max-content;max-width:100%;overflow:hidden}
.lane{width:24px;height:32px;opacity:.38;display:grid;place-items:center}
.lane+.lane{border-left:1.5px dashed rgba(255,255,255,.45)}
.lane svg{width:20px;height:28px}
.lane.on{opacity:1}

/* Nav bar controls */
.trip-btn{display:flex;align-items:center;gap:14px;text-align:left;padding:4px 6px;margin-left:-6px;border-radius:12px}
.trip-btn:hover{background:var(--panel-2)}
.nb-ico{width:44px;height:44px;border-radius:12px;background:var(--panel-2);display:grid;place-items:center;flex:none}
.nb-ico svg{width:22px;height:22px}
.nb-ico.off{color:var(--danger)}
#btnEnd{padding:0 18px}
.trip-head{display:flex;align-items:flex-end;gap:22px;flex-wrap:wrap}
.trip-head div{display:flex;flex-direction:column}
.trip-head b{font:700 26px/1.05 var(--mono);font-variant-numeric:tabular-nums}
.trip-head div:first-child b{color:var(--guide)}
.trip-head span{font:600 12px var(--font);color:var(--muted)}
.delay{font:800 12px var(--font);letter-spacing:.02em}
.delay.bad{color:var(--danger)} .delay.ok{color:var(--guide)}
.route-opt small.delay{grid-column:1/-1}
.steps{list-style:none;margin:0;padding:0}
.st{display:flex;align-items:center;gap:12px;padding:9px 0;border-bottom:1px solid var(--line)}
.st:last-child{border-bottom:0}
.st-ico{width:30px;height:30px;flex:none;color:var(--guide);display:grid;place-items:center}
.st-ico svg{width:28px;height:28px}
.st .row-txt b{white-space:normal;font-weight:600}
.share-txt{width:100%;min-height:96px;border-radius:12px;border:1.5px solid var(--line);background:var(--panel-2);color:var(--ink);padding:10px;font:500 14px var(--font);resize:vertical}

/* Search mic */
#micBtn.listening{color:#fff;background:var(--danger);animation:pulse2 1.2s infinite}
@keyframes pulse2{50%{box-shadow:0 0 0 6px color-mix(in srgb,var(--danger) 30%,transparent)}}

/* Weather */
.wxchip{display:flex;align-items:center;gap:5px;background:var(--panel);box-shadow:var(--shadow);border-radius:10px;padding:5px 9px;margin-bottom:4px;pointer-events:auto;color:var(--ink)}
.wxchip svg{width:20px;height:20px;color:var(--muted)}
.wxchip b{font:700 15px var(--mono);font-variant-numeric:tabular-nums}
.wxchip span{font:800 9px var(--font);letter-spacing:.1em;text-transform:uppercase;color:var(--muted)}
.wxchip em{font:700 11px var(--mono);font-style:normal;color:var(--route);margin-left:2px;font-variant-numeric:tabular-nums}
#wxAlert{display:flex;align-items:center;gap:10px;background:var(--panel);border-radius:12px;box-shadow:var(--shadow),inset 4px 0 0 var(--wc);padding:9px 12px 9px 14px;text-align:left;font-size:13px;color:var(--ink)}
#wxAlert i{width:12px;height:12px;border-radius:50%;background:var(--wc);flex:none;animation:blink 1s infinite}
#wxAlert b{font-weight:800}
.wx-route{display:flex;align-items:center;gap:10px;margin-top:12px;padding:10px 12px;border-radius:12px;background:var(--panel-2)}
.wx-route i{width:12px;height:12px;border-radius:50%;flex:none}
.wx-route span{display:flex;flex-direction:column;min-width:0}
.wx-route small{color:var(--muted)}
.pv-wx{display:flex;align-items:center;gap:10px;margin-top:10px}
.pv-wx:empty{display:none}
.pvw-i{width:34px;height:34px;border-radius:50%;background:var(--panel-2);display:grid;place-items:center;flex:none;color:var(--muted)}
.pvw-i svg{width:22px;height:22px}
.pv-wx span:last-child{display:flex;flex-direction:column}
.pv-wx small{color:var(--muted)}
.wx-head{margin:8px 0 6px;font-weight:600;font-size:13px}
.wx-desc{max-height:140px;overflow:auto;white-space:pre-wrap;font-size:12px;line-height:1.45;color:var(--muted);background:var(--panel-2);border-radius:8px;padding:8px}
.wx-ins{margin-top:6px;color:var(--ink)}

/* Parking */
.park div{width:32px;height:32px;border-radius:8px;background:#2557c9;color:#fff;display:grid;place-items:center;font:800 18px var(--font);border:2.5px solid #fff;box-shadow:0 2px 6px rgba(0,0,0,.35)}
.park-badge{width:30px;height:30px;border-radius:8px;background:#2557c9;color:#fff;display:grid;place-items:center;font:800 16px var(--font);flex:none}
.chev{font-size:24px;color:var(--muted);flex:none}

/* Route bubbles */
.rt-bub-wrap{background:none;border:0}
.rt-bub{position:absolute;left:0;bottom:10px;transform:translateX(-14px);display:flex;flex-direction:column;align-items:flex-start;white-space:nowrap;
  padding:6px 11px 7px;border-radius:12px;background:var(--ink);color:var(--panel);box-shadow:0 3px 10px rgba(0,0,0,.3);cursor:pointer;line-height:1.1}
.rt-bub::after{content:"";position:absolute;left:10px;bottom:-7px;border:7px solid transparent;border-bottom:0;border-top-color:var(--ink)}
.rt-bub b{font:800 17px var(--font)}
.rt-bub span{font:800 11px var(--font);letter-spacing:.06em;text-transform:uppercase;opacity:.85}
.rt-bub.on{background:var(--route);color:#fff}
.rt-bub.on::after{border-top-color:var(--route)}
/* Stops */
.stop-pin div{width:30px;height:30px;border-radius:50%;background:#7a3fd1;color:#fff;display:grid;place-items:center;font:800 14px var(--font);border:3px solid #fff;box-shadow:0 2px 6px rgba(0,0,0,.35)}
.cats.wrap{flex-wrap:wrap;overflow:visible}
.cat.on{border-color:var(--route);background:color-mix(in srgb,var(--route) 10%,transparent)}
.pv-actions.three{grid-template-columns:1.3fr 1fr 1fr}
/* Compass in heading-up mode */
.compass.hdg{box-shadow:var(--shadow),inset 0 0 0 2.5px var(--route)}

/* Trip details: weather strip + elevation */
.pv-trip:empty{display:none}
.ws{display:grid;grid-auto-flow:column;grid-auto-columns:minmax(52px,1fr);gap:6px;overflow-x:auto;padding-bottom:2px}
.ws-c{display:flex;flex-direction:column;align-items:center;gap:2px;background:var(--panel-2);border-radius:10px;padding:7px 4px}
.ws-c small{font:700 10.5px var(--font);color:var(--muted);white-space:nowrap;font-variant-numeric:tabular-nums}
.ws-c .ws-p{color:var(--route)}
.ws-i svg{width:22px;height:22px;color:var(--muted)}
.ws-c b{font:700 15px var(--mono);font-variant-numeric:tabular-nums}
.ws-haz{display:flex;align-items:center;gap:8px;margin:0 0 8px;font-weight:700;color:var(--danger)}
.ws-haz i{width:10px;height:10px;border-radius:50%;background:var(--danger);flex:none}
.elev{display:block;width:100%;height:56px;max-width:100%}
.elev-a{fill:color-mix(in srgb,var(--guide) 22%,transparent)}
.elev-l{fill:none;stroke:var(--guide);stroke-width:2;vector-effect:non-scaling-stroke}
.elev-t{display:flex;gap:16px;flex-wrap:wrap;margin:6px 0 0;font-size:13px;color:var(--muted)}
.elev-t b{color:var(--ink);font-family:var(--mono);font-variant-numeric:tabular-nums}
.route-opt .rt-extra{grid-column:1/-1;font:700 12px var(--font);color:var(--muted)}
/* Vehicle inputs */
.num-row{display:grid;grid-template-columns:1fr 1fr;gap:10px}
.num-row label{display:flex;flex-direction:column;gap:6px;font-weight:700;font-size:13px}
.num-row span{display:flex;align-items:center;gap:6px;color:var(--muted);font-weight:600}
.num-row input{width:100%;min-width:0;height:42px;border-radius:10px;border:1.5px solid var(--line);background:var(--panel-2);color:var(--ink);padding:0 10px;font:600 15px var(--mono)}
.cam-pin svg{width:13px;height:13px}
/* Windshield HUD: deliberately one dark look, high contrast */
#hudv{position:fixed;inset:0;z-index:950;background:#000;color:#5cf2b0;display:flex;flex-direction:column;
  padding:calc(env(safe-area-inset-top,0px) + 16px) max(16px,env(safe-area-inset-right,0px)) calc(env(safe-area-inset-bottom,0px) + 12px) max(16px,env(safe-area-inset-left,0px))}
#hudv .h-in{flex:1;display:flex;flex-direction:column;justify-content:center;align-items:center;gap:2vh;text-align:center}
#hudv .h-in.mirror{transform:scaleY(-1)}
#hudv .h-turn{display:flex;align-items:center;gap:4vw}
#hudv .h-icon{width:min(30vw,26vh);height:min(30vw,26vh)}
#hudv .h-icon svg{width:100%;height:100%}
#hudv .h-icon svg circle[fill="var(--guide)"]{fill:#000}
#hudv .h-dist{font:700 min(16vw,15vh)/1 var(--mono);font-variant-numeric:tabular-nums}
#hudv .h-dist small{font:800 .35em var(--font);margin-left:.15em}
#hudv .h-street{font:800 min(7vw,6vh)/1.15 var(--font);max-width:90vw;text-wrap:balance}
#hudv .h-row{display:flex;align-items:baseline;gap:6vw}
#hudv .h-spd b{font:700 min(20vw,18vh)/1 var(--mono);font-variant-numeric:tabular-nums}
#hudv .h-spd small{font:800 min(5vw,4vh) var(--font);margin-left:.3em}
#hudv.over .h-spd{color:#ff4d4d}
#hudv .h-eta{font:700 min(6vw,5vh) var(--mono);color:#9fb8ad}
#hudv .h-bar{display:flex;gap:10px;justify-content:center}
#hudv .h-bar button{height:44px;padding:0 18px;border-radius:12px;background:#151a18;color:#cfe;font-weight:700;border:1px solid #2a3330}

/* Estimated limit, parking chip, faster alt */
.limit.est{opacity:.82}
.limit.est::after{content:"EST";position:absolute;left:50%;bottom:-15px;transform:translateX(-50%);font:800 8.5px var(--font);letter-spacing:.1em;color:var(--muted);background:var(--panel);padding:0 4px;border-radius:4px}
.limit{position:relative}
.b-park{display:inline-flex;align-items:center;gap:8px;margin-top:10px;margin-left:8px;padding:5px 12px 5px 5px;border-radius:999px;background:rgba(255,255,255,.95);color:#13201b;font:800 13px var(--font)}
.b-park span{width:24px;height:24px;border-radius:50%;background:#2557c9;color:#fff;display:grid;place-items:center;font-size:13px}
.route-opt em.faster{color:var(--guide)}
div.route-opt{cursor:default}

/* ===== Waze-style driving UI ===== */
body:not(.navigating) .nav-only{display:none}
.sim-pill{font:800 14px var(--mono);width:52px;height:40px;border-radius:12px}
/* instruction card: one dark look in both themes, like Waze */
#banner{background:#0b0d0f;color:#fff;border-radius:16px;padding:0;overflow:hidden;box-shadow:0 8px 26px rgba(0,0,0,.45)}
#banner .b-main{padding:12px 16px 12px 14px;gap:14px}
#banner .b-icon{width:58px;height:58px;color:#fff}
#banner .b-dist{font:800 24px/1.1 var(--font);letter-spacing:0}
#banner .b-dist small{font:800 24px var(--font);text-transform:none;letter-spacing:0;opacity:1;margin-left:0}
#banner .b-street{font:800 26px/1.15 var(--font);color:var(--wz);margin-top:2px}
#banner .b-lanes{display:flex;margin:0;padding:10px 12px 0;background:none;border-radius:0;gap:0;width:auto;max-width:none}
#banner .lane{flex:1;max-width:58px;width:auto;height:34px;opacity:.35;color:#fff;position:relative}
#banner .lane+.lane{border-left:0}
#banner .lane+.lane::before{content:"";position:absolute;left:0;bottom:2px;width:2px;height:9px;border-radius:1px;background:#6b7378}
#banner .lane.on{opacity:1}
#banner .b-then{display:flex;width:100%;margin:0;border-radius:0;padding:9px 16px;background:#1c2024;gap:12px;text-transform:none;letter-spacing:0;font:700 17px var(--font);color:var(--wz);align-items:center}
#banner .bt-i{width:26px;height:26px;color:#fff;display:grid;place-items:center;flex:none}
#banner .bt-i svg{width:26px;height:26px}
#banner .bt-l{min-width:0;overflow:hidden;text-overflow:ellipsis;white-space:nowrap}
#banner .b-park{margin:0 16px 12px}
.shields{display:inline-flex;gap:6px;vertical-align:middle;align-items:center}
.shield{display:block;flex:none}
#banner .b-street .shield{height:34px;width:auto}
.shield-txt{display:inline-block;padding:1px 7px;border-radius:5px;background:#fff;color:#111;font:800 .8em var(--font)}
/* bottom bar */
#navbar{display:block;padding-top:8px}
.wz-bar{display:grid;grid-template-columns:52px minmax(0,1fr) 52px;align-items:center;gap:10px}
.wz-round{width:52px;height:52px;border-radius:50%;background:var(--panel-2);display:grid;place-items:center;color:var(--ink)}
.wz-round svg{width:24px;height:24px}
.wz-mid{display:flex;flex-direction:column;align-items:center;gap:8px;padding:0 0 4px;min-width:0}
.wz-grab{width:40px;height:5px;border-radius:3px;background:var(--line)}
.wz-row{display:grid;grid-template-columns:auto auto;justify-content:center;align-items:baseline;column-gap:0}
#eta{grid-column:1/-1;text-align:center;font:800 32px/1.05 var(--font);font-variant-numeric:tabular-nums;color:var(--ink)}
.wz-sub{font:600 17px var(--font);color:var(--muted);font-variant-numeric:tabular-nums}
#remD::before{content:"•";margin:0 9px}
.wz-row #eta{grid-row:1}
.wz-row #remT,.wz-row #remD{grid-row:2}
/* speedometer ring + overlapping limit sign */
.speedo{position:relative;width:78px;height:78px;border:0;background:color-mix(in srgb,var(--panel) 94%,transparent)}
.sp-ring{position:absolute;inset:0;width:100%;height:100%;transform:rotate(135deg)}
.sp-track,.sp-arc{fill:none;stroke-width:5;stroke-linecap:round}
.sp-track{stroke:var(--line);stroke-dasharray:146.1 999}
.sp-arc{stroke:var(--ink);stroke-dasharray:0 999;transition:stroke-dasharray .4s}
.sp-arc.over{stroke:var(--danger)}
.speedo b{position:relative;font:800 30px/1 var(--font);font-variant-numeric:tabular-nums}
.speedo span{position:relative;letter-spacing:.02em;text-transform:none;font-size:12px}
.speedo.over{border:0}
#limit{align-self:flex-start;margin:-8px 0 0 -14px;z-index:1}
/* current street */
#streetPill{position:fixed;z-index:545;left:50%;transform:translateX(-50%);bottom:calc(var(--bottom) + 16px);max-width:max(140px,calc(100vw - 280px));
  padding:9px 18px;border-radius:999px;background:#0b0d0f;color:#fff;font:800 17px var(--font);white-space:nowrap;overflow:hidden;text-overflow:ellipsis;box-shadow:0 4px 14px rgba(0,0,0,.35);pointer-events:none}
#streetPill .shield{height:24px;width:auto}
/* car + route, Waze colors */
.car .halo{fill:#5d6b78;opacity:.32}
.car .arrow{fill:var(--wz);stroke:#effcff;stroke-width:2}
.car .arrow-hl{fill:#a6ecff}
.sig{background:none;border:0}
.sig svg{width:16px;height:30px;filter:drop-shadow(0 1px 2px rgba(0,0,0,.5))}
.trip-links{display:flex;align-items:center;justify-content:space-between;gap:10px;margin-top:10px}
.trip-links .btn{height:44px;padding:0 20px}
/* landscape phone: card top-left, bar centered (Waze landscape) */
@media (orientation: landscape) and (max-height: 600px){
  #top{left:max(10px,env(safe-area-inset-left,0px));right:auto;width:min(46vw,500px);margin:0;padding-left:0;padding-right:0;padding-top:calc(env(safe-area-inset-top,0px) + 8px)}
  #sheet,#place{left:max(10px,env(safe-area-inset-left,0px));right:auto;width:min(46vw,500px);margin:0;max-height:calc(100% - 84px);border-radius:20px 20px 0 0}
  #navbar{left:50%;right:auto;transform:translateX(-50%);width:min(78vw,860px);margin:0;padding-top:6px;padding-bottom:calc(6px + env(safe-area-inset-bottom,0px))}
  .wz-mid{gap:4px}
  .wz-row{display:flex;gap:26px;align-items:baseline}
  #eta{font-size:30px}
  .wz-sub{font-size:19px}
  #remD::before{content:none}
  #banner .b-main{padding:8px 14px}
  #banner .b-icon{width:46px;height:46px}
  #banner .b-dist,#banner .b-dist small{font-size:20px}
  #banner .b-street{font-size:22px}
  #banner .b-lanes{padding-top:6px}
  #banner .lane{height:28px}
  #banner .b-then{padding:6px 14px;font-size:15px}
  .fab{width:46px;height:46px}
  #fabs{gap:8px}
  .compass{width:52px;height:52px}
  #streetPill{left:calc(max(12px,env(safe-area-inset-left,0px)) + 150px);transform:none;max-width:24vw}
  #modal .sheet{max-height:92vh}
}

.limit.unk{opacity:.6}
.limit.unk b{color:#666}

body.navigating .car-rot{width:58px;height:58px;margin:-7px}

#toast{position:fixed;z-index:900;left:50%;top:calc(env(safe-area-inset-top,0px) + 76px);transform:translateX(-50%);background:var(--ink);color:var(--panel);padding:10px 16px;border-radius:12px;font-weight:600;box-shadow:var(--shadow);max-width:calc(100% - 32px);text-align:center;transition:opacity .25s}
#toast.nav{top:calc(env(safe-area-inset-top,0px) + 150px)}
@media (prefers-reduced-motion: reduce){.car-rot,#hud,#fabs,.leaflet-bottom{transition:none}.limit.over{animation:none}}
[hidden]{display:none!important}
</style>
</head>
<body>
<div id="mapWrap"><div id="map"></div></div>

<header id="top">
  <div id="searchbar">
    <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.4" stroke-linecap="round"><circle cx="10.5" cy="10.5" r="6.5"/><path d="M20 20l-4.8-4.8"/></svg>
    <input id="q" type="search" placeholder="Where to?" autocomplete="off" enterkeyhint="search" aria-label="Search for a destination">
    <button id="clearQ" class="icon-btn" aria-label="Clear search" hidden>×</button>
    <button id="micBtn" class="icon-btn" aria-label="Search by voice" hidden><svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2" stroke-linecap="round"><rect x="9" y="3" width="6" height="11" rx="3" fill="currentColor"/><path d="M5.5 11a6.5 6.5 0 0 0 13 0M12 17.5V21"/></svg></button>
  </div>
  <div id="banner" hidden aria-live="polite">
    <div id="bLanes" class="b-lanes" hidden></div>
    <div class="b-main">
      <div id="bIcon" class="b-icon"></div>
      <div class="b-text"><div id="bDist" class="b-dist"></div><div id="bStreet" class="b-street"></div></div>
    </div>
    <div id="bThen" class="b-then" hidden></div>
    <button id="bPark" class="b-park" hidden><span>P</span>Parking near destination</button>
  </div>
  <button id="wxAlert" hidden></button>
  <div id="tkey" hidden></div>
  <div id="legend" hidden></div>
  <button id="recPill" hidden aria-label="Stop recording"></button>
</header>

<div id="hud">
  <div id="speedo" class="speedo"><svg class="sp-ring" viewBox="0 0 72 72" aria-hidden="true"><circle class="sp-track" cx="36" cy="36" r="31"/><circle id="spArc" class="sp-arc" cx="36" cy="36" r="31"/></svg><b id="spd">0</b><span id="spdU">mph</span></div>
  <div id="limit" class="limit" hidden><span>SPEED<br>LIMIT</span><b id="limitVal">35</b></div>
  <div id="alt" class="alt" hidden><span>Elev</span><b id="altVal"></b></div>
  <button id="wx" class="wxchip" hidden aria-label="Current weather"></button>
</div>

<div id="fabs">
  <button id="btnCompass" class="fab compass off" aria-label="Turn on live compass" style="margin-bottom:14px">
    <svg class="rose" viewBox="0 0 48 48"><circle class="ring" cx="24" cy="24" r="21"/>
      <path class="n" d="M24 7l5 17h-10z"/><path class="s" d="M24 41l5-17h-10z"/>
      <text class="tn" x="24" y="6.2">N</text><text x="24" y="47">S</text><text x="45.5" y="26.5">E</text><text x="2.5" y="26.5">W</text></svg>
    <small id="cmpTxt">Compass</small>
  </button>
  <button id="btnMute" class="fab nav-only" aria-label="Mute voice"></button>
  <button id="btnSim" class="fab sim-pill" hidden aria-label="Simulation speed">×1</button>
  <button id="btnLayers" class="fab" aria-label="Map layers and settings"><svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2" stroke-linejoin="round"><path d="M12 3l9.5 5L12 13 2.5 8z"/><path d="M2.5 12.5L12 17.5l9.5-5"/><path d="M2.5 16.5L12 21.5l9.5-5"/></svg></button>
  <button id="btnRecenter" class="fab" aria-label="Recenter on my location" hidden><svg viewBox="0 0 24 24" fill="currentColor"><path d="M12 2l7.5 19-7.5-4.5L4.5 21z"/></svg></button>
  <button id="btnReport" class="fab fab-warn" aria-label="Report something on the road"><svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.4" stroke-linecap="round" stroke-linejoin="round"><path d="M12 3L2 20h20z"/><path d="M12 10v4M12 17.2v.1"/></svg></button>
</div>

<div id="streetPill" hidden></div>
<section id="sheet" class="sheet"></section>

<section id="navbar" class="sheet" hidden>
  <div class="wz-bar">
    <button id="btnNavSearch" class="wz-round" aria-label="Find a stop along the way"><svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.4" stroke-linecap="round"><circle cx="10.5" cy="10.5" r="6.5"/><path d="M20 20l-4.8-4.8"/></svg></button>
    <button id="tripBtn" class="wz-mid" aria-label="Trip details and end drive">
      <span class="wz-grab"></span>
      <span class="wz-row"><span id="remT" class="wz-sub">--</span><b id="eta">--:--</b><span id="remD" class="wz-sub">--</span></span>
    </button>
    <button id="btnAlts" class="wz-round" aria-label="Other routes from here"><svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.3" stroke-linecap="round" stroke-linejoin="round"><path d="M12 21v-7l-6-6V4M12 14l6-6V4"/><path d="M3.5 6.5L6 4l2.5 2.5M15.5 6.5L18 4l2.5 2.5"/></svg></button>
  </div>
  <button id="btnOverview" hidden aria-label="Route overview"></button>
  <button id="btnEnd" hidden>End</button>
</section>

<section id="place" hidden aria-live="polite"></section>
<div id="hudv" hidden>
  <div class="h-in">
    <div class="h-turn"><div id="hIcon" class="h-icon"></div><div id="hDist" class="h-dist"></div></div>
    <div id="hStreet" class="h-street"></div>
    <div class="h-row"><span class="h-spd"><b id="hSpd">0</b><small id="hSpdU">mph</small></span><span id="hEta" class="h-eta"></span></div>
  </div>
  <div class="h-bar"><button data-h="mirror">Flip for windshield</button><button data-h="exit">Exit HUD</button></div>
</div>
<div id="mbSetup" hidden>
  <div class="mbs-card">
    <h1>Connect Mapbox</h1>
    <p>Guidepost draws its map, traffic and live-traffic routes with Mapbox. Paste your Mapbox <b>public token</b> to start.</p>
    <ol>
      <li>Sign in or sign up free at <a href="https://account.mapbox.com/" target="_blank" rel="noopener">account.mapbox.com</a>.</li>
      <li>Copy the <b>Default public token</b> (it starts with <code>pk.</code>).</li>
      <li>Paste it below. It’s saved on this device only.</li>
    </ol>
    <label for="mbTok">Mapbox public token</label>
    <input id="mbTok" type="text" autocomplete="off" spellcheck="false" placeholder="pk.eyJ1Ijoi…">
    <p id="mbSetupErr" class="mbs-err"></p>
    <button id="mbSave" class="btn primary">Start driving</button>
  </div>
</div>
<div id="modal" hidden><section class="sheet" id="modalSheet"></section></div>
<div id="toast" hidden role="status"></div>

<script src="https://cdn.jsdelivr.net/npm/mapbox-gl@3.8.0/dist/mapbox-gl.js"></script>
<script>
/* Paste a Mapbox public token (starts with pk.) here to skip the setup screen, or enter it in the app. */
var MAPBOX_TOKEN = '';
document.getElementById('mbSave').onclick = () => {
  const v = document.getElementById('mbTok').value.trim();
  if (!/^pk\./.test(v)){ document.getElementById('mbSetupErr').textContent = 'That doesn’t look like a public token. It should start with pk.'; return; }
  try{ localStorage.setItem('gp_mbtoken', JSON.stringify(v)); }catch{}
  location.reload();
};
</script>
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
/* OpenStreetMap Overpass queries, with mirror servers when the main one is busy or rate-limiting */
const OVERPASS = ['https://overpass-api.de/api/interpreter', 'https://overpass.private.coffee/api/interpreter', 'https://maps.mail.ru/osm/tools/overpass/api/interpreter'];
let opFirst = 0;
async function opFetch(opts){
  let last = null;
  for (let i = 0; i < OVERPASS.length; i++){
    const k = (opFirst + i) % OVERPASS.length;
    try{
      const r = await fetch(OVERPASS[k], opts);
      if (r.ok){ opFirst = k; return r; }
      last = r; if (r.status !== 429 && r.status < 500) return r;
    }catch(e){ last = null; }
  }
  if (last) return last;
  throw new Error('overpass');
}

/* ---------- settings & theme ---------- */
const settings = Object.assign({ units:'imperial', voice:true, theme:'auto', base:'street',
  ovOffroad:true, ovHike:false, ovMtb:false, ovReports:true,
  srcOsm:true, srcUsfs:true, srcBlm:true, legendOpen:true, ovTraffic:false, trafficInc:true,
  trafficRouting:true, avoidTolls:false, avoidHighways:false, avoidFerries:false, avoidUnpaved:false, ovRadar:false, radarAnim:true, ovWarnings:true,
  orient:'heading', useCompass:false, speedAlert:true,
  camAlerts:true, view3d:true, showSignals:true, mpg:28, priceGal:3.1, l100:8, priceL:1.8, currency:'$', hudMirror:true }, store.get('settings', {}));
const saveSettings = () => store.set('settings', settings);
const mq = matchMedia('(prefers-color-scheme: dark)');
const isDark = () => {
  if (settings.theme === 'sun'){ const p = S.pos || [map.getCenter().lat, map.getCenter().lng]; return sunAlt(p[0], p[1]) < -2; }
  return settings.theme === 'dark' || (settings.theme === 'auto' && mq.matches);
};

/* ---------- Mapbox GL engine behind a small Leaflet-style API ----------
   The rest of the app talks to "L" and "map" the way it did with Leaflet; this layer turns that into
   Mapbox GL sources, layers, markers and popups. Zoom numbers keep Leaflet's 256-px scale (Mapbox zoom + 1). */
const MB_TOKEN = (typeof MAPBOX_TOKEN === 'string' && MAPBOX_TOKEN) || store.get('mbtoken', '') || '';
if (!MB_TOKEN || typeof mapboxgl === 'undefined'){
  document.getElementById('mbSetup').hidden = false;
  if (typeof mapboxgl === 'undefined' && MB_TOKEN) document.getElementById('mbSetupErr').textContent = 'The Mapbox map library didn’t load. Check your connection and reload.';
  throw new Error('Mapbox token needed');
}
mapboxgl.accessToken = MB_TOKEN;

const L = (() => {
  let mb = null, shim = null, seq = 0;
  const GL = { ready:false, items:new Set(), order:[], objects:new Map(), popup:null };
  const Z_LABELS = 500;

  class LatLng { constructor(lat, lng){ this.lat = lat; this.lng = lng; } }
  const toLL = p => p instanceof LatLng ? p : Array.isArray(p) ? new LatLng(+p[0], +p[1]) : new LatLng(+p.lat, +(p.lng ?? p.lon));
  const lnglat = p => { const q = toLL(p); return [q.lng, q.lat]; };
  const isPt = p => p instanceof LatLng || (Array.isArray(p) && typeof p[0] === 'number') || (p && typeof p === 'object' && 'lat' in p && !Array.isArray(p));

  class Bounds {
    constructor(pts){ this.s = Infinity; this.w = Infinity; this.n = -Infinity; this.e = -Infinity; (pts || []).forEach(p => this.extend(p)); }
    extend(p){
      if (p instanceof Bounds){ if (p.isValid()){ this.extend([p.s, p.w]); this.extend([p.n, p.e]); } return this; }
      if (!isPt(p)){ (p || []).forEach(x => this.extend(x)); return this; }
      const q = toLL(p); this.s = Math.min(this.s, q.lat); this.n = Math.max(this.n, q.lat); this.w = Math.min(this.w, q.lng); this.e = Math.max(this.e, q.lng); return this;
    }
    isValid(){ return isFinite(this.s); }
    getSouth(){ return this.s; } getNorth(){ return this.n; } getWest(){ return this.w; } getEast(){ return this.e; }
    getCenter(){ return new LatLng((this.s + this.n) / 2, (this.w + this.e) / 2); }
    pad(r){ const dl = (this.n - this.s) * r, dw = (this.e - this.w) * r; return new Bounds([[this.s - dl, this.w - dw], [this.n + dl, this.e + dw]]); }
    contains(b){
      if (b instanceof Bounds) return b.s >= this.s && b.n <= this.n && b.w >= this.w && b.e <= this.e;
      const q = toLL(b); return q.lat >= this.s && q.lat <= this.n && q.lng >= this.w && q.lng <= this.e;
    }
    intersects(b){ return !(b.s > this.n || b.n < this.s || b.w > this.e || b.e < this.w); }
    toMB(){ return [[this.w, this.s], [this.e, this.n]]; }
  }

  const Evented = {
    on(types, fn){ this._ev = this._ev || {}; String(types).split(/\s+/).forEach(t => (this._ev[t] = this._ev[t] || []).push(fn)); return this; },
    off(types, fn){ String(types).split(/\s+/).forEach(t => { if (this._ev?.[t]) this._ev[t] = fn ? this._ev[t].filter(f => f !== fn) : []; }); return this; },
    fire(t, e = {}){ (this._ev?.[t] || []).slice().forEach(f => f.call(this, e)); return this; }
  };

  /* layer ordering among our own GL layers; everything below the style's labels unless z >= 500 */
  function labelLayer(){ const ls = mb.getStyle()?.layers || []; return ls.find(l => l.type === 'symbol')?.id; }
  function beforeFor(z){
    const next = GL.order.filter(o => o.z > z && mb.getLayer(o.id)).sort((a, b) => a.z - b.z)[0];
    if (next) return next.id;
    return z < Z_LABELS ? labelLayer() : undefined;
  }
  function addGLLayer(spec, z){ if (mb.getLayer(spec.id)) return; mb.addLayer(spec, beforeFor(z)); GL.order.push({ id:spec.id, z }); }
  const paneZ = pane => pane ? (Number(PANES[pane]?.style.zIndex) || 400) : null;
  const PANES = {};

  /* ----- batched vector overlays: one GeoJSON source per batch, data-driven style ----- */
  class Batch {
    constructor(id, z, kind){ this.id = id; this.z = z; this.kind = kind; this.items = new Map(); this.dash = new Map(); this.dirty = false; GL.items.add(this); }
    set(o){ this.items.set(o._id, o); this.touch(); }
    del(o){ if (this.items.delete(o._id)) this.touch(); }
    touch(){ if (!this.dirty){ this.dirty = true; requestAnimationFrame(() => this.flush()); } }
    layerIds(){ const base = this.kind === 'line' ? [this.id+'-l'] : [this.id+'-f', this.id+'-fl']; return base.concat([...this.dash.values()]).filter(id => mb.getLayer(id)); }
    attach(){
      if (!mb.getSource(this.id)) mb.addSource(this.id, { type:'geojson', data:{ type:'FeatureCollection', features:[] }, tolerance:.2 });
      const line = id => ({ id, type:'line', source:this.id, layout:{ 'line-cap':'round', 'line-join':'round' },
        paint:{ 'line-color':['get','c'], 'line-width':['get','w'], 'line-opacity':['get','o'] } });
      if (this.kind === 'line') addGLLayer({ ...line(this.id+'-l'), filter:['==', ['get','dk'], ''] }, this.z);
      else {
        addGLLayer({ id:this.id+'-f', type:'fill', source:this.id, paint:{ 'fill-color':['get','fc'], 'fill-opacity':['get','fo'] } }, this.z);
        addGLLayer({ ...line(this.id+'-fl'), filter:['>', ['get','w'], 0] }, this.z);
      }
      const keys = [...this.dash.keys()]; this.dash.clear(); keys.forEach(k => this.addDash(k));
      this.dirty = false; this.flush();
    }
    addDash(k){
      if (this.dash.has(k)) return;
      const [d, w] = k.split('|'), id = this.id+'-d'+(++seq);
      this.dash.set(k, id);
      if (!GL.ready || !mb.getSource(this.id)) return;
      addGLLayer({ id, type:'line', source:this.id, filter:['==', ['get','dk'], k], layout:{ 'line-cap':'butt', 'line-join':'round' },
        paint:{ 'line-color':['get','c'], 'line-width':['get','w'], 'line-opacity':['get','o'], 'line-dasharray':d.split(/[ ,]+/).map(Number).map(v => Math.max(.1, v / Math.max(1, +w))) } }, this.z);
    }
    flush(){
      this.dirty = false;
      if (!GL.ready) return;
      if (!mb.getSource(this.id)) return this.attach();      // first use after the style loaded
      const fs = [];
      for (const o of this.items.values()){ const f = o._feature(); if (!f) continue; if (f.properties.dk && !this.dash.has(f.properties.dk)) this.addDash(f.properties.dk); fs.push(f); }
      mb.getSource(this.id).setData({ type:'FeatureCollection', features:fs });
    }
  }
  const BATCH = {};
  const batchFor = (kind, pane) => {
    const key = kind + ':' + (pane || '');
    return BATCH[key] || (BATCH[key] = new Batch('gp-'+kind+'-'+(pane || 'base'), paneZ(pane) ?? (kind === 'line' ? 400 : 380), kind));
  };

  /* ----- base layer object ----- */
  class Layer {
    constructor(){ this._id = 'o'+(++seq); this._map = null; this._groups = new Set(); }
    addTo(t){ t.addLayer(this); return this; }
    remove(){ if (this._map) this._map.removeLayer(this); return this; }
    bindPopup(c, o){ this._popup = { c, o }; return this; }
    _openPopup(at){ if (!this._popup) return; const c = typeof this._popup.c === 'function' ? this._popup.c(this) : this._popup.c; shim.openPopup(c, at, this._popup.o); }
  }
  Object.assign(Layer.prototype, Evented);

  const lineCoords = ll => {
    if (!ll || !ll.length) return null;
    if (isPt(ll[0])) return ll.length > 1 ? { type:'LineString', coordinates:ll.map(lnglat) } : null;
    const parts = ll.filter(p => p && p.length > 1).map(p => p.map(lnglat));
    return parts.length ? { type:'MultiLineString', coordinates:parts } : null;
  };
  class Polyline extends Layer {
    constructor(ll, o = {}){ super(); this._ll = ll; this.options = o; }
    _batch(){ return batchFor('line', this.options.renderer?.pane || this.options.pane); }
    _feature(){
      const g = lineCoords(this._ll); if (!g) return null;
      const o = this.options, w = o.weight ?? 3;
      return { type:'Feature', geometry:g, properties:{ id:this._id, c:o.color || '#3388ff', w, o:o.opacity ?? 1, dk:o.dashArray ? o.dashArray+'|'+w : '', ia:o.interactive === false ? 0 : 1 } };
    }
    _add(m){ this._map = m; GL.objects.set(this._id, this); this._batch().set(this); }
    _rm(){ GL.objects.delete(this._id); this._batch().del(this); this._map = null; }
    setLatLngs(ll){ this._ll = ll; if (this._map) this._batch().touch(); return this; }
    addLatLng(p){ (this._ll = this._ll || []).push(p); if (this._map) this._batch().touch(); return this; }
    getLatLngs(){ return this._ll; }
    getBounds(){ return new Bounds(this._ll); }
  }
  class Polygon extends Polyline {
    _batch(){ return batchFor('fill', this.options.pane); }
    _feature(){
      const polys = isPt(this._ll[0]) ? [[this._ll]] : isPt(this._ll[0][0]) ? [this._ll] : this._ll;
      const o = this.options, close = r => { const c = r.map(lnglat); if (c.length && (c[0][0] !== c[c.length-1][0] || c[0][1] !== c[c.length-1][1])) c.push(c[0]); return c; };
      return { type:'Feature', geometry:{ type:'MultiPolygon', coordinates:polys.map(p => p.map(close)) },
        properties:{ id:this._id, fc:o.fillColor || o.color || '#3388ff', fo:o.fillOpacity ?? .2, c:o.color || '#3388ff', w:o.stroke === false ? 0 : (o.weight ?? 2), o:o.opacity ?? 1, ia:o.interactive === false ? 0 : 1 } };
    }
  }
  class Circle extends Polygon {
    constructor(c, o = {}){ super([], o); this._c = toLL(c); this._r = o.radius || 10; }
    setLatLng(c){ this._c = toLL(c); if (this._map) this._batch().touch(); return this; }
    setRadius(r){ this._r = r; if (this._map) this._batch().touch(); return this; }
    _feature(){
      const pts = [], c = this._c, dLat = this._r / 111320, dLng = dLat / Math.cos(c.lat * Math.PI / 180);
      for (let i = 0; i <= 40; i++){ const a = i / 40 * 2 * Math.PI; pts.push([c.lat + dLat * Math.sin(a), c.lng + dLng * Math.cos(a)]); }
      this._ll = pts; return super._feature();
    }
  }

  /* ----- raster tile overlays (radar, trails, traffic tiles) ----- */
  class TileLayer extends Layer {
    constructor(url, o = {}){ super(); this.url = url; this.options = o; this.z = paneZ(o.pane) ?? 200; this.gid = 'gp-t'+(++seq); }
    _tiles(){
      const o = this.options, subs = o.subdomains ? String(o.subdomains).split('') : ['a','b','c'];
      let u = this.url.replace('{r}', '');
      u = u.replace(/\{(\w+)\}/g, (m, k) => ['x','y','z','s'].includes(k) ? m : (typeof o[k] === 'function' ? o[k]() : o[k] ?? ''));
      return u.includes('{s}') ? subs.map(s => u.replace('{s}', s)) : [u];
    }
    attach(){
      if (!mb.getSource(this.gid)) mb.addSource(this.gid, { type:'raster', tiles:this._tiles(), tileSize:256, maxzoom:this.options.maxNativeZoom ?? Math.min(19, this.options.maxZoom ?? 19), attribution:this.options.attribution || '' });
      addGLLayer({ id:this.gid, type:'raster', source:this.gid, paint:{ 'raster-opacity':this.options.opacity ?? 1, 'raster-fade-duration':0 } }, this.z);
    }
    _add(m){ this._map = m; GL.items.add(this); if (GL.ready) this.attach(); }
    _rm(){ GL.items.delete(this); if (GL.ready){ if (mb.getLayer(this.gid)) mb.removeLayer(this.gid); if (mb.getSource(this.gid)) mb.removeSource(this.gid); } this._map = null; }
    setOpacity(v){ this.options.opacity = v; if (GL.ready && mb.getLayer(this.gid)) mb.setPaintProperty(this.gid, 'raster-opacity', v); return this; }
    redraw(){ if (GL.ready && mb.getSource(this.gid)?.setTiles) mb.getSource(this.gid).setTiles(this._tiles()); return this; }
  }

  /* ----- custom GL layers (Mapbox traffic, 3D buildings) ----- */
  class GLLayer extends Layer {
    constructor(build, z){ super(); this.build = build; this.z = z; }
    attach(){ const sp = this.build(mb); if (!sp) return; this.ids = []; for (const [srcId, src] of sp.sources || []) if (!mb.getSource(srcId)) mb.addSource(srcId, src); for (const l of sp.layers){ addGLLayer(l, this.z); this.ids.push(l.id); } }
    _add(m){ this._map = m; GL.items.add(this); if (GL.ready) this.attach(); }
    _rm(){ GL.items.delete(this); if (GL.ready) (this.ids || []).forEach(id => mb.getLayer(id) && mb.removeLayer(id)); this._map = null; }
  }

  /* ----- markers ----- */
  class Marker extends Layer {
    constructor(ll, o = {}){
      super(); this.options = o; this._ll = toLL(ll);
      const ic = o.icon?.options || { className:'', html:'', iconSize:[24,24] };
      const el = document.createElement('div');
      el.className = 'gp-icon ' + (ic.className || '');
      el.innerHTML = ic.html || '';
      const [w, h] = ic.iconSize || [0, 0], [ax, ay] = ic.iconAnchor || [w / 2, h / 2];
      el.style.width = w+'px'; el.style.height = h+'px';
      if (o.interactive === false) el.style.pointerEvents = 'none';
      this._el = el;
      this._m = new mapboxgl.Marker({ element:el, anchor:'top-left', offset:[-ax, -ay], draggable:!!o.draggable,
        ...(o.alignMap ? { pitchAlignment:'map', rotationAlignment:'map' } : {}) });   // alignMap: lies flat on the road and turns with the map
      this._m.setLngLat(lnglat(this._ll));
      el.addEventListener('click', ev => { ev.stopPropagation(); this.fire('click', { latlng:this.getLatLng(), originalEvent:ev }); this._openPopup(this.getLatLng()); });
      if (o.draggable){ this._m.on('drag', () => this.fire('drag')); this._m.on('dragend', () => this.fire('dragend')); }
    }
    _add(m){ this._map = m; this._m.addTo(mb); const wrap = this._m.getElement(); wrap.style.zIndex = String(10 + (this.options.zIndexOffset || 0)); }
    _rm(){ this._m.remove(); this._map = null; }
    setLatLng(ll){ this._ll = toLL(ll); this._m.setLngLat(lnglat(this._ll)); return this; }
    getLatLng(){ const p = this._m.getLngLat(); return new LatLng(p.lat, p.lng); }
    getElement(){ return this._el; }
  }

  /* ----- groups ----- */
  class LayerGroup extends Layer {
    constructor(ls){ super(); this._layers = new Set(); (ls || []).forEach(l => this.addLayer(l)); }
    addLayer(l){ this._layers.add(l); l._groups.add(this); if (this._map) this._map.addLayer(l); return this; }
    removeLayer(l){ if (this._layers.delete(l)){ l._groups.delete(this); if (this._map) this._map.removeLayer(l); } return this; }
    clearLayers(){ [...this._layers].forEach(l => this.removeLayer(l)); return this; }
    getLayers(){ return [...this._layers]; }
    _add(m){ this._map = m; this._layers.forEach(l => m.addLayer(l)); }
    _rm(){ const m = this._map; this._map = null; this._layers.forEach(l => m.removeLayer(l)); }
  }

  /* ----- popups ----- */
  class Popup {
    constructor(o = {}){ this.o = o; }
    setLatLng(ll){ this.ll = toLL(ll); return this; }
    setContent(c){ this.c = c; return this; }
    openOn(m){ m.openPopup(this.c, this.ll, this.o); return this; }
  }

  /* ----- the map ----- */
  class MapShim {
    constructor(id, o = {}){
      shim = this; this._layers = new Set(); this._ev = {}; this._style = o.style;
      mb = new mapboxgl.Map({ container:id, style:o.style, center:o.center ? lnglat(o.center) : [-74.006, 40.7128], zoom:(o.zoom ?? 13) - 1,
        attributionControl:true, logoPosition:'bottom-left', pitchWithRotate:true, cooperativeGestures:false, fadeDuration:150, projection:'mercator' });
      this._mb = mb;
      this.attributionControl = { getContainer:() => mb.getContainer().querySelector('.mapboxgl-ctrl-attrib-inner') };
      mb.on('style.load', () => {
        GL.ready = true; GL.order = [];
        const items = [...GL.items].sort((a, b) => a.z - b.z);
        items.forEach(it => { try{ it.attach(); }catch(e){ console.warn('layer', e); } });
        this.fire('styleload');
      });
      const toEv = e => ({ latlng:new LatLng(e.lngLat.lat, e.lngLat.lng), originalEvent:e.originalEvent, _stopped:false });
      mb.on('click', e => {
        if (e.originalEvent?.target?.closest?.('.mapboxgl-marker,.mapboxgl-popup')) return;
        const ev = toEv(e);
        const ids = Object.values(BATCH).flatMap(b => b.layerIds());
        if (ids.length){
          const p = e.point, hits = mb.queryRenderedFeatures([[p.x - 10, p.y - 10], [p.x + 10, p.y + 10]], { layers:ids });
          const hit = hits.map(f => GL.objects.get(f.properties.id)).find(o => o && o.options.interactive !== false && (o._ev?.click?.length || o._popup));
          if (hit){ hit.fire('click', ev); if (hit._popup) hit._openPopup(ev.latlng); if (ev._stopped || hit._popup) return; }
        }
        this.fire('click', ev);
      });
      mb.on('contextmenu', e => this.fire('contextmenu', toEv(e)));
      ['dragstart', 'rotatestart', 'pitchstart'].forEach(t => mb.on(t, e => { if (e.originalEvent) this.fire('dragstart', e); }));
      mb.on('zoomstart', e => this.fire('zoomstart', e));
      mb.on('moveend', e => this.fire('moveend', e));
      mb.on('move', e => this.fire('move', e));
    }
    addLayer(l){ if (l._map === this) return this; this._layers.add(l); l._add(this); return this; }
    removeLayer(l){ if (l._map !== this) return this; this._layers.delete(l); l._rm(); return this; }
    hasLayer(l){ return l._map === this; }
    setView(c, z, o = {}){
      const opts = { center:lnglat(c), zoom:(z ?? this.getZoom()) - 1 };
      if (o.animate === false || !GL.ready) mb.jumpTo(opts); else mb.easeTo({ ...opts, duration:(o.duration ?? .5) * 1000 });
      return this;
    }
    camera(o){
      mb.easeTo({ center:lnglat(o.center), zoom:o.zoom - 1, bearing:o.bearing ?? mb.getBearing(), pitch:o.pitch ?? mb.getPitch(),
        padding:o.padding, duration:o.duration ?? 1000, easing:t => t, essential:true });
    }
    panTo(c){ mb.easeTo({ center:lnglat(c), duration:400 }); return this; }
    fitBounds(b, o = {}){
      const bb = b instanceof Bounds ? b : new Bounds(b); if (!bb.isValid()) return this;
      const [tlx, tly] = o.paddingTopLeft || o.padding || [20, 20], [brx, bry] = o.paddingBottomRight || o.padding || [20, 20];
      const sz = this.getSize(), pad = { top:Math.min(tly, sz.y * .45), left:Math.min(tlx, sz.x * .4), bottom:Math.min(bry, sz.y * .45), right:Math.min(brx, sz.x * .4) };
      mb.fitBounds(bb.toMB(), { padding:pad, maxZoom:(o.maxZoom ?? 18) - 1, bearing:0, pitch:0, duration:700 });
      return this;
    }
    getBounds(){ const b = mb.getBounds(); return new Bounds([[b.getSouth(), b.getWest()], [b.getNorth(), b.getEast()]]); }
    getCenter(){ const c = mb.getCenter(); return new LatLng(c.lat, c.lng); }
    getZoom(){ return mb.getZoom() + 1; }
    getBearing(){ return mb.getBearing(); }
    getSize(){ const c = mb.getContainer(); return { x:c.clientWidth, y:c.clientHeight }; }
    getContainer(){ return mb.getContainer(); }
    latLngToContainerPoint(ll){ const p = mb.project(lnglat(ll)); return { x:p.x, y:p.y }; }
    containerPointToLatLng(p){ const q = mb.unproject(Array.isArray(p) ? p : [p.x, p.y]); return new LatLng(q.lat, q.lng); }
    mouseEventToLatLng(e){ const r = mb.getContainer().getBoundingClientRect(); return this.containerPointToLatLng([e.clientX - r.left, e.clientY - r.top]); }
    invalidateSize(){ mb.resize(); return this; }
    createPane(name){ return PANES[name] = PANES[name] || { style:{} }; }
    getPane(name){ return this.createPane(name); }
    openPopup(html, at, o = {}){
      this.closePopup();
      const p = new mapboxgl.Popup({ maxWidth:(o.maxWidth || 300)+'px', className:'gp-pop', focusAfterOpen:false, offset:12 })
        .setLngLat(lnglat(at)).setHTML(html).addTo(mb);
      GL.popup = p;
      p.on('close', () => { if (GL.popup === p) GL.popup = null; this.fire('popupclose', { popup:p }); });
      this.fire('popupopen', { popup:{ getElement:() => p.getElement() } });
    }
    closePopup(){ if (GL.popup){ const p = GL.popup; GL.popup = null; p.remove(); } return this; }
    setStyle(url){ if (this._style === url) return; this._style = url; GL.ready = false; mb.setStyle(url); }
  }
  Object.assign(MapShim.prototype, Evented);

  const DivIcon = function(o){ this.options = o; };
  return {
    map:(id, o) => new MapShim(id, o),
    marker:(ll, o) => new Marker(ll, o),
    divIcon:o => new DivIcon(o),
    polyline:(ll, o) => new Polyline(ll, o),
    polygon:(ll, o) => new Polygon(ll, o),
    circle:(c, o) => new Circle(c, o),
    tileLayer:(u, o) => new TileLayer(u, o),
    glLayer:(build, z) => new GLLayer(build, z),
    layerGroup:ls => new LayerGroup(ls),
    popup:o => new Popup(o),
    latLngBounds:pts => new Bounds(pts),
    latLng:(a, b) => new LatLng(a, b),
    canvas:o => ({ pane:o?.pane || 'trails' }),
    DomEvent:{ stop:e => { if (e) e._stopped = true; } }
  };
})();

const MB_STYLES = {
  street:dark => dark ? 'mapbox://styles/mapbox/navigation-night-v1' : 'mapbox://styles/mapbox/navigation-day-v1',
  topo:() => 'mapbox://styles/mapbox/outdoors-v12',
  satellite:() => 'mapbox://styles/mapbox/satellite-streets-v12'
};
const startDark = settings.theme === 'dark' || (settings.theme === 'auto' && mq.matches) || (settings.theme === 'sun' && (new Date().getHours() < 7 || new Date().getHours() >= 18));
const lastView = store.get('lastView', null);
const map = L.map('map', { style:(MB_STYLES[settings.base] || MB_STYLES.street)(startDark), center:lastView ? [lastView[0], lastView[1]] : [39.5, -98.35], zoom:lastView ? lastView[2] : 5 });
map.on('styleload', () => { if (S.nav) drawNavRoute(); });
map.on('moveend', () => { const c = map.getCenter(); store.set('lastView', [+c.lat.toFixed(5), +c.lng.toFixed(5), +map.getZoom().toFixed(1)]); });
const TILE_ATTR = '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors';
function applyTheme(){
  const th = settings.theme === 'sun' ? ((sunDark = isDark()) ? 'dark' : 'light') : settings.theme;
  if (th === 'auto') delete document.documentElement.dataset.theme;
  else document.documentElement.dataset.theme = th;
  setBase();
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
  star:'<svg viewBox="0 0 24 24" fill="currentColor"><path d="M12 2.5l2.9 6 6.6.8-4.9 4.5 1.3 6.5L12 17l-5.9 3.3 1.3-6.5L2.5 9.3l6.6-.8z"/></svg>',
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
    case 'arrive': return (step.waypoint ? 'Arrive at your stop' : 'Arrive at your destination')+(mod.includes('left') ? ', on the left' : mod.includes('right') ? ', on the right' : '');
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
  html:'<div class="car-rot"><svg viewBox="0 0 44 44" style="overflow:visible"><defs><radialGradient id="beamGrad" cx="22" cy="22" r="40" gradientUnits="userSpaceOnUse"><stop offset="0" stop-color="#1a6fe0" stop-opacity=".45"/><stop offset="1" stop-color="#1a6fe0" stop-opacity="0"/></radialGradient></defs><path class="beam" d="M22 22L4 -14A40 40 0 0 1 40 -14z"/><ellipse class="halo" cx="22" cy="23" rx="20" ry="19"/><path class="arrow" d="M22 6.5L34 32.5 22 27 10 32.5z"/><path class="arrow-hl" d="M22 6.5L22 27 10 32.5z"/></svg></div>' });
let carMarker = null, accCircle = null;
function drawCar(){
  if (!carMarker){
    carMarker = L.marker(S.pos, { icon:carIcon, interactive:false, zIndexOffset:1000, alignMap:true }).addTo(map);
    accCircle = L.circle(S.pos, { radius:S.acc || 10, stroke:false, fillColor:css('--route'), fillOpacity:.1, interactive:false }).addTo(map);
  }
  carMarker.setLatLng(S.pos); accCircle.setLatLng(S.pos).setRadius(Math.min(S.acc || 10, 500));
  rotateCar();
}
/* ---------- map rotation, car arrow and compass: one rAF loop, frame-rate independent ---------- */
// Heading-up: Mapbox turns the map so your direction of travel points up (see followCam).
// The car arrow eases toward the newest heading each frame: compass ~45 ms, GPS ~300 ms.
const canRotate = true;
let carRot = 0, animRun = false, animLast = 0, txtLast = 0;
S.mapHdg = 0;
const wrap = d => ((d % 360) + 540) % 360 - 180;            // shortest signed angle
const compassDriving = () => CMP.on && CMP.raw != null && S.speed < CMP_SPEED;
const bearingNow = () => -map.getBearing();          // screen turn of the map (negative of Mapbox bearing)
const headingUp = () => canRotate && settings.orient === 'heading';
function northUp(){}                                   // fitBounds already resets bearing and pitch
function kickAnim(){ if (!animRun){ animRun = true; animLast = performance.now(); requestAnimationFrame(animFrame); } }
function rotateCar(){
  const root = carMarker?.getElement();
  if (root) root.classList.toggle('compass-on', compassDriving());
  kickAnim();
}
function mapHeading(){
  const n = S.nav;
  if (n && n.routeBrg != null && !n.off) return n.routeBrg;       // follow the road ahead, not GPS jitter
  if (compassDriving()) return S.heading;
  if (S.speed > 1.5) return S.heading;
  return S.mapHdg;                                                 // stopped: hold the last direction
}
function animFrame(t){
  const dt = Math.min(.1, Math.max(0, (t - animLast) / 1000)); animLast = t;
  let busy = false;
  const b = bearingNow();
  const carEl = carMarker?.getElement()?.querySelector('.car-rot');
  if (carEl){
    const d = wrap(S.heading - carRot);
    if (Math.abs(d) > .05){ carRot += Math.abs(d) < .3 ? d : d * (1 - Math.exp(-dt / (compassDriving() ? .045 : .3))); busy = true; }
    carEl.style.transform = `rotate(${carRot.toFixed(2)}deg)`;          // the car marker lies on the map, so world heading is right
  }
  roseEl.style.transform = `rotate(${b.toFixed(2)}deg)`;            // the red needle always points at north on screen
  if (t - txtLast > 150){
    txtLast = t;
    const h = ((headingUp() ? S.mapHdg : (CMP.raw ?? 0)) % 360 + 360) % 360;
    cmpTxt.textContent = headingUp() ? CARD16[Math.round(h / 22.5) % 16] : CMP.on && CMP.raw != null ? CARD16[Math.round(h / 22.5) % 16]+' '+Math.round(h)+'°' : 'N up';
  }
  if (busy || CMP.on) requestAnimationFrame(animFrame); else animRun = false;
}
function renderOrient(){
  const b = $('#btnCompass');
  b.classList.remove('off'); b.classList.toggle('hdg', headingUp());
  b.setAttribute('aria-label', headingUp() ? 'Switch to north up' : 'Turn the map with your direction of travel');
}

/* ---------- phone compass (device orientation) ---------- */
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
  CMP.raw = CMP.raw == null ? h : CMP.raw + wrap(h - CMP.raw);
  if (S.speed < CMP_SPEED) S.heading = (CMP.raw % 360 + 360) % 360;
  kickAnim();
  if (compassDriving() && S.follow && headingUp() && Date.now() - (S.camAt || 0) > 250) followCam(300);
}
async function compassOn(){
  try{
    if (typeof DeviceOrientationEvent !== 'undefined' && typeof DeviceOrientationEvent.requestPermission === 'function'){
      if (await DeviceOrientationEvent.requestPermission() !== 'granted'){ toast('Compass access was denied. Allow Motion & Orientation for this site.'); return compassOff(); }
    }
  }catch{ toast('Compass access needs a tap on the switch.'); return compassOff(); }
  if (!('DeviceOrientationEvent' in window)){ toast('This device has no compass sensor.'); return compassOff(); }
  CMP.on = true; CMP.got = false; CMP.raw = null;
  window.addEventListener('ondeviceorientationabsolute' in window ? 'deviceorientationabsolute' : 'deviceorientation', onOrient);
  settings.useCompass = true; saveSettings(); store.set('compass', true);
  kickAnim();
  setTimeout(() => { if (CMP.on && !CMP.got){ toast('No compass reading. This device may lack a magnetometer.'); compassOff(); } }, 3000);
}
function compassOff(){
  CMP.on = false; CMP.raw = null;
  window.removeEventListener('deviceorientationabsolute', onOrient); window.removeEventListener('deviceorientation', onOrient);
  settings.useCompass = false; saveSettings(); store.set('compass', false); rotateCar();
  if (!modal.hidden && mSheet.dataset.view === 'layers') renderLayers();
}
$('#btnCompass').onclick = () => {
  settings.orient = headingUp() ? 'north' : 'heading'; saveSettings();
  if (headingUp() && S.pos){ S.follow = true; $('#btnRecenter').hidden = true; }
  followCam(500); renderOrient(); kickAnim();
  toast(headingUp() ? 'Map turns with you' : 'North up', 1500);
};
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
  checkWx(); refreshConditions();
  if (S.nav) navTick();
  if (!S.firstFix){ S.firstFix = true; if (!S.nav && !S.dest) followCam(900); kickAnim(); }
  else if (S.follow && (S.nav || !S.dest)) followCam();
}
function startGPS(){
  if (!('geolocation' in navigator)){ toast('No location access in this browser. Use Simulate to test routes.'); return; }
  navigator.geolocation.watchPosition(p => { if (!S.sim) onPosition(p.coords); },
    err => toast(err.code === 1 ? 'Location is blocked. Allow it for this site (the page must be on HTTPS).' : 'Waiting for a GPS signal…', 4500),
    { enableHighAccuracy:true, maximumAge:1000, timeout:20000 });
}
// One camera move per GPS fix, eased linearly over about a second, so the map glides continuously.
function followCam(dur){
  if (!S.pos) return;
  S.camAt = Date.now();
  is3D = want3D() && S.follow;
  const turning = headingUp() && (S.nav || !S.dest);
  if (turning) S.mapHdg = mapHeading();
  const H = map.getSize().y, navH = S.nav ? $('#navbar').offsetHeight : 0;
  const z = S.nav ? (S.speed > 24 ? 15.2 : S.speed > 12 ? 16 : 16.8) + (is3D ? .4 : 0) : Math.max(map.getZoom(), 15.5);
  map.camera({ center:S.pos, zoom:z, bearing:turning ? S.mapHdg : 0, pitch:is3D ? 60 : 0,
    padding:{ top:S.nav ? Math.round(H * (is3D ? .48 : .34)) : 0, bottom:navH, left:0, right:0 }, duration:dur ?? 1000 });
}
function destPoint(p, brg, m){
  const d = m / R_E, b = rad(brg), la = rad(p[0]), lo = rad(p[1]);
  const la2 = Math.asin(Math.sin(la) * Math.cos(d) + Math.cos(la) * Math.sin(d) * Math.cos(b));
  return [deg(la2), deg(lo + Math.atan2(Math.sin(b) * Math.sin(d) * Math.cos(la), Math.cos(d) - Math.sin(la) * Math.sin(la2)))];
}
map.on('dragstart', () => { S.follow = false; is3D = false; $('#btnRecenter').hidden = !S.pos; });
map.on('move', kickAnim);
$('#btnRecenter').onclick = () => { S.follow = true; $('#btnRecenter').hidden = true; followCam(700); kickAnim(); };

/* ---------- speed & limit ---------- */
function updateSpeedo(){
  const v = speedDisp(S.speed);
  $('#spd').textContent = v; $('#spdU').textContent = imperial() ? 'mph' : 'km/h';
  const lim = $('#limit');
  if (S.limitKmh == null){ lim.hidden = !S.pos; lim.classList.add('unk'); lim.classList.remove('est', 'over'); lim.classList.toggle('metric', !imperial()); $('#limitVal').textContent = '–'; lim.title = 'Looking up the speed limit'; $('#speedo').classList.remove('over'); renderSpeedRing(v, 0, false); if (!$('#hudv').hidden){ $('#hSpd').textContent = v; $('#hSpdU').textContent = imperial() ? 'mph' : 'km/h'; } return; }
  const lv = imperial() ? Math.round(S.limitKmh/1.609344/5)*5 : Math.round(S.limitKmh);
  lim.hidden = false; lim.classList.remove('unk'); lim.classList.toggle('metric', !imperial()); lim.classList.toggle('est', !!S.limitEst);
  lim.title = S.limitEst ? 'Estimated from the road type. No posted limit is mapped here.' : 'Posted speed limit';
  $('#limitVal').textContent = lv;
  const over = v > lv + (imperial() ? 4 : 6);
  lim.classList.toggle('over', over && !S.limitEst); $('#speedo').classList.toggle('over', over);
  renderSpeedRing(v, lv, over && !S.limitEst);
  if (over && !S.wasOver && S.speed > 4 && !S.limitEst) chime();
  if (!$('#hudv').hidden){ $('#hSpd').textContent = v; $('#hSpdU').textContent = imperial() ? 'mph' : 'km/h'; $('#hudv').classList.toggle('over', over); }
  S.wasOver = over;
}
/* Speed limits on every road:
   1) the route's own limits (TomTom speed-limit sections, or OSRM maxspeed annotations from OSM),
   2) otherwise the nearest road from OpenStreetMap: its posted maxspeed tag,
   3) otherwise a typical limit for that road type (shown as an estimate). */
let slLast = 0, slBusy = false, slPos = null;
const SL_ZONES = { urban:50, rural:90, motorway:120, trunk:100, living_street:20, walk:7, school:30, zone30:30 };
function parseMaxspeed(s){
  if (!s) return null;
  s = String(s).split(';')[0].trim();
  if (/^none$/i.test(s) || /^signals$/i.test(s)) return null;
  if (/^walk$/i.test(s)) return 7;
  const m = s.match(/(\d+(?:\.\d+)?)\s*(mph|knots)?/i);
  if (m){ const n = parseFloat(m[1]); return m[2] && /mph/i.test(m[2]) ? n*1.609344 : n; }
  const z = s.split(':')[1]; return z && SL_ZONES[z.toLowerCase()] || null;
}
const inUS = ([la, lo]) => (la > 24.4 && la < 49.4 && lo > -124.9 && lo < -66.8) || (la > 51 && la < 71.5 && lo > -170 && lo < -129.9) || (la > 18.8 && la < 22.4 && lo > -160.4 && lo < -154.7);
const US_MPH = { motorway:65, trunk:55, primary:45, secondary:45, tertiary:35, unclassified:35, road:35, residential:25, living_street:15, service:15,
  motorway_link:45, trunk_link:35, primary_link:35, secondary_link:35, tertiary_link:25 };
const INTL_KMH = { motorway:120, trunk:100, primary:80, secondary:80, tertiary:60, unclassified:60, road:60, residential:50, living_street:20, service:30,
  motorway_link:80, trunk_link:60, primary_link:60, secondary_link:50, tertiary_link:50 };
function defaultLimit(t, ll){
  const h = t.highway || '';
  if (inUS(ll)) return US_MPH[h] ? US_MPH[h] * 1.609344 : null;
  return INTL_KMH[h] || null;
}
function segDist(p, a, b){
  const cos = Math.cos(rad(p[0])), ax = (a[1]-p[1])*cos, ay = a[0]-p[0], bx = (b[1]-p[1])*cos, by = b[0]-p[0];
  const dx = bx-ax, dy = by-ay, l2 = dx*dx + dy*dy; let t = l2 ? -(ax*dx + ay*dy)/l2 : 0; t = t < 0 ? 0 : t > 1 ? 1 : t;
  return Math.hypot(ax + t*dx, ay + t*dy) * 111320;
}
function setLimit(kmh, est){
  if (kmh !== S.limitKmh || est !== S.limitEst){ S.limitKmh = kmh; S.limitEst = est; updateSpeedo(); }
}
const ROAD_RE = '^(motorway|trunk|primary|secondary|tertiary|unclassified|residential|living_street|service|road|motorway_link|trunk_link|primary_link|secondary_link|tertiary_link)$';
async function maybeSpeedLimit(){
  if (!S.pos) return;
  const n = S.nav;
  if (n && n.route.limits && n.si != null && !n.off){
    const v = n.route.limits[Math.min(n.si, n.route.limits.length - 1)];
    if (v != null){ setLimit(v, false); return; }
  }
  const moved = !slPos || dist(slPos, S.pos) > 120;
  if (slBusy || Date.now() - slLast < (moved ? 6000 : 30000)) return;
  if (!moved && S.limitKmh != null) return;
  slBusy = true; slLast = Date.now();
  const at = S.pos, hdg = S.heading, moving = S.speed > 2;
  const ql = `[out:json][timeout:10];way(around:30,${at[0].toFixed(6)},${at[1].toFixed(6)})[highway~"${ROAD_RE}"];out geom 12;`;
  try{
    const r = await opFetch({ method:'POST', body:'data='+encodeURIComponent(ql), headers:{ 'Content-Type':'application/x-www-form-urlencoded' } });
    const j = await r.json();
    let cur = null;
    if (n){ const st = n.route.steps[n.k || 0]; cur = st && (st.name || st.ref); }
    let best = null, bestScore = Infinity;
    for (const w of j.elements || []){
      const g = (w.geometry || []).map(p => [p.lat, p.lon]); if (g.length < 2) continue;
      for (let i = 0; i < g.length - 1; i++){
        let sc = segDist(at, g[i], g[i+1]);
        if (moving){ const b = bearing(g[i], g[i+1]); sc += Math.min(Math.abs(wrap(b - hdg)), Math.abs(wrap(b + 180 - hdg))) * .4; }
        if (cur && (w.tags.name === cur || w.tags.ref === cur)) sc -= 12;
        if (w.tags.highway === 'service' && !w.tags.name) sc += 15;      // parking aisles, driveways
        if (sc < bestScore){ bestScore = sc; best = w; }
      }
    }
    if (best){
      const t = best.tags;
      let kmh = parseMaxspeed(t.maxspeed || t['maxspeed:forward'] || t['maxspeed:backward']), est = false;
      if (kmh == null && !/^none$/i.test(t.maxspeed || '')){ kmh = defaultLimit(t, at); est = true; }
      slPos = at; setLimit(kmh, est);
    }
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
    S.list = null;
    try{
      const r = await fetch(`https://api.mapbox.com/search/geocode/v6/forward?q=${encodeURIComponent(text)}&proximity=${c[1].toFixed(4)},${c[0].toFixed(4)}&limit=8&language=${(navigator.language || 'en').slice(0, 2)}&access_token=${encodeURIComponent(MB_TOKEN)}`, { signal:searchAbort.signal });
      if (r.ok){
        const j = await r.json();
        S.list = (j.features || []).map(f => ({ name:f.properties.name || f.properties.full_address, addr:f.properties.full_address || f.properties.place_formatted || '',
          sub:f.properties.place_formatted || '', lat:f.geometry.coordinates[1], lng:f.geometry.coordinates[0] }));
      }
    }catch(e){ if (e.name === 'AbortError') throw e; }
    if (!S.list || !S.list.length){
      const r = await fetch(`https://nominatim.openstreetmap.org/search?format=jsonv2&limit=8&q=${encodeURIComponent(text)}&viewbox=${vb}`,
        { signal:searchAbort.signal, headers:{ 'Accept-Language':navigator.language || 'en' } });
      if (!r.ok) throw new Error('Search service returned '+r.status+'. Try again in a moment.');
      const j = await r.json();
      S.list = j.map(x => ({ name: x.name || x.display_name.split(',')[0], addr: x.display_name, lat:+x.lat, lng:+x.lon }));
    }
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
    ${favs().length ? `<h3 class="label">Saved</h3><ul class="list">${favs().map((p, i) => row(p, 'fav', i, ico.star)).join('')}</ul>` : ''}
    ${rec.length ? `<h3 class="label">Recent</h3><ul class="list">${rec.map((p, i) => row(p, 'rec', i, ico.clock)).join('')}</ul>`
                 : '<p class="hint">Search for a place, or press and hold anywhere on the map to drop a pin.</p>'}`);
}
function renderPreview(state){
  const p = S.dest; if (!p) return;
  let body;
  if (state === 'loading') body = '<p class="hint">Finding routes…</p>';
  else if (state && state.error) body = `<p class="err">${esc(state.error)}</p>`;
  else {
    const R0 = S.routes[S.routeIdx];
    body = `<ul class="routes">${S.routes.map((r, i) => `<li><button class="route-opt${i === S.routeIdx ? ' on' : ''}" data-route="${i}">
      <b>${fmtDur(r.duration)}</b><span>${fd(r.distance)} · via ${esc(r.via)}</span>${i === 0 ? '<em>Fastest</em>' : '<em></em>'}
      ${routeExtra(r)}</button></li>`).join('')}</ul>
    ${S.stop ? `<p class="hint">Stopping at <b>${esc(S.stop.name)}</b> on the way</p>` : ''}
    ${R0?.note ? `<p class="hint">${esc(R0.note)}</p>` : ''}
    ${wxRouteHTML(R0)}
    <div class="pv-actions"><button class="btn primary" data-act="go">Go</button><button class="btn" data-act="sim">Simulate</button></div>
    <div id="pvTrip" class="pv-trip"></div>`;
  }
  sheetHTML(`<div class="grab"></div>
    <div class="pv-head"><div class="row-txt"><b class="pv-name">${esc(p.name)}</b><small>${esc(shortAddr(p.addr) || p.addr || '')}</small></div>
      <button class="icon-btn" data-act="close" aria-label="Close">×</button></div>
    ${body}
    <div class="pv-save"><button class="link" data-act="fav">${isFav(p) ? '★ Saved' : '☆ Save'}</button><button class="link" data-act="sethome">Set as Home</button><button class="link" data-act="setwork">Set as Work</button><button class="link" data-act="opts">Route options</button><button class="link" data-act="addstop">${S.stop ? 'Change stop' : 'Add a stop'}</button></div>`);
  const R0 = S.routes[S.routeIdx];
  if (R0 && !state) fillTripDetails(R0);
}
sheet.addEventListener('click', e => {
  const b = e.target.closest('button'); if (!b) return;
  if (b.dataset.cat) return searchCat(b.dataset.cat);
  if (b.dataset.kind === 'fav') return previewPlace(favs()[+b.dataset.i]);
  if (b.dataset.kind) return previewPlace(S.list[+b.dataset.i]);
  if (b.dataset.saved){ const p = store.get(b.dataset.saved); return p ? previewPlace(p) : toast('Search for a place, then tap “Set as '+(b.dataset.saved === 'home' ? 'Home' : 'Work')+'”.'); }
  if (b.dataset.route){ S.routeIdx = +b.dataset.route; drawRoutes(); renderPreview(); return; }
  const act = b.dataset.act;
  if (act === 'close') closePreview();
  else if (act === 'back') renderIdle();
  else if (act === 'go') startNav(false);
  else if (act === 'sim') startNav(true);
  else if (act === 'opts') renderRouteOpts();
  else if (act === 'addstop') renderAddStop();
  else if (act === 'fav'){ toggleFav(S.dest); renderPreview(); }
  else if (act === 'park') askPark();
  else if (act === 'sethome' || act === 'setwork'){ store.set(act.slice(3), S.dest); toast((act === 'sethome' ? 'Home' : 'Work')+' saved'); }
});

/* ---------- destination & routing ---------- */
const destIcon = L.divIcon({ className:'pin', iconSize:[34,44], iconAnchor:[17,42],
  html:'<svg viewBox="0 0 34 44"><defs><pattern id="chk" width="6" height="6" patternUnits="userSpaceOnUse"><rect width="6" height="6" fill="#fff"/><rect width="3" height="3" fill="#111"/><rect x="3" y="3" width="3" height="3" fill="#111"/></pattern></defs><path d="M17 43l-6-9h12z" fill="#fff"/><circle cx="17" cy="16" r="15" fill="#fff" stroke="rgba(0,0,0,.25)"/><circle cx="17" cy="16" r="12" fill="url(#chk)"/></svg>' });
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
  if (S.stop && !(S.dest && S.dest.lat === p.lat && S.dest.lng === p.lng)){ S.stop = null; drawStop(); }
  S.dest = { ...p, ll:[p.lat, p.lng] }; S.routes = []; S.routeIdx = 0; S.follow = false; layoutView();
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
  S.stop = null; drawStop();
  S.dest = null; S.routes = []; routeLayer.clearLayers();
  if (destMarker){ map.removeLayer(destMarker); destMarker = null; }
  q.value = ''; $('#clearQ').hidden = true; renderIdle();
  if (S.pos){ S.follow = true; followCam(); }
}
function prepRoute(r){
  const coords = r.geometry.coordinates.map(c => [c[1], c[0]]);
  if (coords.length < 2) coords.push(coords[0]);
  const cum = new Float64Array(coords.length);
  for (let i = 1; i < coords.length; i++) cum[i] = cum[i-1] + dist(coords[i-1], coords[i]);
  const total = cum[coords.length-1];
  const steps = r.legs.flatMap((l, li) => l.steps.map(st => (st.maneuver?.type === 'arrive' && li < r.legs.length - 1) ? Object.assign(st, { waypoint:true }) : st));
  const sum = steps.reduce((a, s) => a + s.distance, 0) || 1, scale = total / sum;
  const stepStart = []; let acc = 0;
  for (const s of steps){ stepStart.push(acc*scale); acc += s.distance; }
  const byName = {};
  steps.forEach(s => { const n = s.name || s.ref; if (n) byName[n] = (byName[n] || 0) + s.distance; });
  const via = Object.entries(byName).sort((a, b) => b[1] - a[1])[0]?.[0] || 'local roads';
  const ann = (r.legs || []).flatMap(l => l.annotation?.maxspeed || []);
  const limits = ann.length ? ann.map(a => a && a.speed ? (a.unit === 'mph' ? a.speed * 1.609344 : a.speed) : null) : null;
  return { coords, cum, total, steps, stepStart, duration:r.duration, distance:r.distance, via, limits };
}
function drawRoutes(){
  routeLayer.clearLayers();
  const order = S.routes.map((_, i) => i).filter(i => i !== S.routeIdx).concat(S.routeIdx);
  for (const i of order){
    const r = S.routes[i], on = i === S.routeIdx;
    L.polyline(r.coords, { color:css('--casing'), weight:on ? 11 : 9, opacity:on ? 1 : .7, interactive:false }).addTo(routeLayer);
    const pl = L.polyline(r.coords, { color:on ? css('--route') : css('--route-alt'), weight:on ? 7 : 6 }).addTo(routeLayer);
    if (!on) pl.on('click', () => { S.routeIdx = i; drawRoutes(); renderPreview(); });
    if (on) for (const t of trafficSegs(r)) L.polyline(t.ll, { color:t.color, weight:7, interactive:false }).addTo(routeLayer);
  }
  drawBubbles();
}
function fitRoutes(){
  northUp();
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
  S.nav = { route, spoken:{}, off:0, lastReroute:0, along:0, k:0, rerouting:false, remT:route.duration, remD:route.total, t0:Date.now(), driven:0, lastPos:S.pos };
  renderMute(); setTimeout(apply3D, 0); WX.told.forEach(id => { if (String(id).startsWith('ahead:')) WX.told.delete(id); });
  routeLayer.clearLayers(); drawNavRoute();
  $('#searchbar').hidden = true; $('#banner').hidden = false;
  sheet.hidden = true; $('#navbar').hidden = false; renderLegend(); document.body.classList.add('navigating');
  S.follow = true; $('#btnRecenter').hidden = true;
  keepAwake(true);
  const first = route.steps[0], second = route.steps[1];
  say(instr(first) + (second && second.maneuver.type !== 'arrive' ? '. Then, '+instr(second).replace(/^./, c => c.toLowerCase()) : ''));
  if (simulate) startSim();
  if (S.pos && !S.sim) navTick();
  else renderNav(Math.min(1, route.steps.length-1), route.stepStart[1] ?? route.total, route.total, route.duration);
  if (S.pos) followCam(); else map.setView(route.coords[0], 17);
  startFasterChecks(); checkWx(true);
}
function drawNavRoute(){
  if (CAM.route !== S.nav.route){ CAM.route = S.nav.route; loadCameras(S.nav.route); }
  if (navLine) map.removeLayer(navLine); if (navCasing) map.removeLayer(navCasing);
  const c = S.nav.route.coords;
  navCasing = L.polyline(c, { color:css('--casing'), weight:13, interactive:false }).addTo(map);
  navLine = L.polyline(c, { color:css('--route'), weight:8, interactive:false }).addTo(map);
  if (S.nav.tLayer) map.removeLayer(S.nav.tLayer);
  S.nav.tSegs = trafficSegs(S.nav.route);
  S.nav.tLayer = L.layerGroup(S.nav.tSegs.map(t => (t.layer = L.polyline(t.ll, { color:t.color, weight:8, interactive:false })))).addTo(map);
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
  n.remT = remT; n.remD = remD; n.si = s.i;
  $('#bPark').hidden = !(remD < 1300 && S.dest && !S.dest.isParking);
  if (n.lastPos){ const dd = dist(n.lastPos, S.pos); if (dd < 300) n.driven += dd; } n.lastPos = S.pos;
  checkCameras(s.along);
  n.routeBrg = s.d < 35 ? bearing(s.pt, pointAt(R, s.along + 35).pt) : null;
  if (S.stop){ const w = R.steps.findIndex(st => st.waypoint); if (w >= 0 && s.along > R.stepStart[w] + 40){ S.stop = null; drawStop(); toast('Stop reached'); } }
  renderNav(next, toNext, remD, remT);
  speakGuidance(next, toNext);
  for (const t of n.tSegs || []) if (!t.gone && t.b <= s.i){ t.gone = true; n.tLayer.removeLayer(t.layer); }
  if (navLine){ const rest = [s.pt].concat(R.coords.slice(s.i+1)); navLine.setLatLngs(rest); navCasing.setLatLngs(rest); }
  if (remD < 25 && s.d < 60) arrive();
}
function renderNav(k, toNext, remD, remT){
  const R = S.nav.route, step = R.steps[k];
  const d = fmtDist(Math.max(0, toNext));
  $('#bIcon').innerHTML = turnSVG(step);
  $('#bDist').innerHTML = `${d.v} <small>${d.u === 'mi' ? (d.v === '1.0' || d.v === '1' ? 'mile' : 'miles') : d.u === 'ft' ? 'feet' : d.u}</small>`;
  $('#bStreet').innerHTML = S.nav.rerouting ? 'Rerouting…' : stepLabelHTML(step);
  $('#bStreet').setAttribute('aria-label', instr(step));
  const nxt = R.steps[k+1];
  if (nxt){ $('#bThen').hidden = false; $('#bThen').innerHTML = `<span class="bt-i">${turnSVG(nxt)}</span><span class="bt-l">${stepLabelHTML(nxt)}</span>`; }
  else $('#bThen').hidden = true;
  const rn = roadNowHTML(), pill = $('#streetPill'); pill.hidden = !rn; if (rn && pill.innerHTML !== rn) pill.innerHTML = rn;
  renderLanes(step, toNext);
  $('#eta').textContent = clock(remT); $('#remT').textContent = fmtDur(remT); $('#remD').textContent = fd(remD);
  if (!$('#hudv').hidden){ $('#hIcon').innerHTML = turnSVG(step); $('#hDist').innerHTML = `${d.v}<small>${d.u}</small>`; $('#hStreet').textContent = instr(step); $('#hEta').textContent = clock(remT)+' · '+fd(remD); }
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
    sp.now = 1; sp.far = sp.near = 1; navigator.vibrate?.([60, 60, 60]);
    const nxt = R.steps[k+1];
    const then = nxt && (R.stepStart[k+1] - R.stepStart[k]) < 200 ? ', then '+instr(nxt).replace(/^./, c => c.toLowerCase()) : '';
    say(step.maneuver.type === 'arrive' ? (step.waypoint ? 'You have reached your stop' : 'You have arrived') : text + then);
  }
}
async function reroute(force){
  const n = S.nav;
  if (!n || n.rerouting || (!force && Date.now() - n.lastReroute < 8000)) return;
  n.rerouting = true; n.lastReroute = Date.now(); $('#bStreet').textContent = force ? 'Updating route…' : 'Rerouting…'; say(force ? 'Updating your route' : 'Rerouting');
  try{
    const [r] = await fetchRoutes(S.pos, S.dest.ll, false);
    if (!S.nav) return;
    n.route = r; n.spoken = {}; n.off = 0; n.along = 0; drawNavRoute(); startFasterChecks();
    if (S.sim) S.sim.along = 0;
  }catch(e){ toast(e.message); }
  n.rerouting = false;
}
function arrive(){
  const name = S.dest?.name || 'your destination';
  if (!S.nav.spoken.arrived) say('You have arrived at '+name);
  S.nav.arrived = true;
  if (S.dest?.isParking){ store.set('parked', { lat:S.pos?.[0] ?? S.dest.lat, lng:S.pos?.[1] ?? S.dest.lng, t:Date.now() }); drawPark(); toast('Parking spot saved'); }
  endNav(true);
  sheetHTML(`<div class="grab"></div><div class="pv-head"><div class="row-txt"><b class="pv-name">You’ve arrived</b><small>${esc(name)}</small></div>
    <button class="icon-btn" data-act="close" aria-label="Close">×</button></div>
    <div class="pv-actions"><button class="btn primary" data-act="park">Save parking spot</button><button class="btn" data-act="close">Done</button></div>`);
}
function endNav(keepDest){
  logTrip(S.nav); hudClose(); altLayer.clearLayers(); $('#bPark').hidden = true;
  CAM.list = []; CAM.layer.clearLayers(); CAM.route = null; SIG.layer.clearLayers(); $('#streetPill').hidden = true;
  stopSim(); keepAwake(false); clearInterval(fasterT);
  S.stop = null; drawStop();
  if (S.nav?.tLayer) map.removeLayer(S.nav.tLayer);
  $('#bLanes').hidden = true; WX.ahead = []; renderWxAlert();
  if (navLine){ map.removeLayer(navLine); map.removeLayer(navCasing); navLine = navCasing = null; }
  S.nav = null; renderLegend(); apply3D(); document.body.classList.remove('navigating');
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
function openModal(html){ mSheet.innerHTML = '<div class="grab"></div>'+html; mSheet.dataset.view = ''; modal.hidden = false; }
function closeModal(){
  if (mSheet.dataset.view === 'alts') altLayer.clearLayers();
  modal.hidden = true; mSheet.dataset.view = '';
  if (S.routesStale){ S.routesStale = false; if (S.dest && !S.nav) previewPlace(S.dest); }
}
modal.addEventListener('click', e => { if (e.target === modal) closeModal(); });
$('#btnReport').onclick = () => openModal(`<h3 class="label" style="margin-top:0">What do you see?</h3>
  <div class="rgrid">${Object.entries(RT).map(([k, T]) => `<button class="rbtn" data-rep="${k}"><i style="background:${T.color}">${T.svg}</i>${T.label}</button>`).join('')}</div>
  <p class="fine">You’ll confirm the spot on the map next. Reports stay on this device for 3 hours.</p>`);
function seg(key, opts){ return `<div class="seg">${opts.map(([v, l]) => `<button data-set="${key}" data-val="${v}" class="${String(settings[key]) === String(v) ? 'on' : ''}">${l}</button>`).join('')}</div>`; }
function tg(k, title, sub){ return `<button class="tg" role="switch" aria-checked="${!!settings[k]}" data-tg="${k}"><span class="row-txt"><b>${title}</b><small>${sub}</small></span><span class="sw"></span></button>`; }
function renderLayers(){
  const keep = mSheet.scrollTop, wasOpen = !modal.hidden && mSheet.querySelector('.bases');
  mSheet.dataset.view = 'layers';
  openModal(`<h3 class="label" style="margin-top:0">Map style</h3>
    <div class="bases">${[['street','Street'],['topo','Terrain'],['satellite','Satellite']].map(([k, l]) =>
      `<button class="base${(BASES[settings.base] ? settings.base : 'street') === k ? ' on' : ''}" data-base="${k}"><i class="th-${k}"></i>${l}</button>`).join('')}</div>
    <h3 class="label">On the map</h3>
    ${tg('ovTraffic', 'Live traffic', 'Slow and stopped roads colored orange to dark red, from Mapbox')}
    ${settings.ovTraffic ? `<div class="subs">${tg('trafficInc', 'Incident icons', 'Crashes, closures and roadwork from TomTom. Needs the optional key below.')}
      <div class="keyrow"><label for="ttKey">TomTom API key (optional)</label>
        <div class="keyin"><input id="ttKey" type="password" autocomplete="off" spellcheck="false" placeholder="Paste your key" value="${esc(trafficKey())}"><button class="btn" data-ttsave>Save</button></div>
        <small>Get one free at developer.tomtom.com (sign up, then copy the key from your dashboard). It stays on this device.</small></div></div>` : ''}
    ${tg('ovRadar', 'Weather radar', 'NEXRAD reflectivity for the US, updated every 5 minutes')}
    ${settings.ovRadar ? `<div class="subs">${tg('radarAnim', 'Animate last hour', 'Loop the past 50 minutes of radar')}</div>` : ''}
    ${tg('ovWarnings', 'Storm warnings', 'NWS tornado, severe storm and flash flood warnings, with alerts when you or your route is inside one')}
    ${tg('ovOffroad', 'Off-road trails', 'Dirt roads, 4×4, ATV and motorcycle routes. Tap one for details.')}
    ${settings.ovOffroad ? `<div class="subs">${SOURCES.map(x => tg(x.key, x.label, x.sub)).join('')}</div>` : ''}
    ${tg('ovHike', 'Hiking routes', 'Marked hiking routes from Waymarked Trails')}
    ${tg('ovMtb', 'Mountain bike routes', 'Marked MTB routes from Waymarked Trails')}
    ${tg('ovReports', 'Road reports', 'Police, crashes and hazards you reported')}
    <h3 class="label">Tools</h3>
    <button class="tg" data-tool="rec"><span class="row-txt"><b>${REC.on ? 'Stop recording' : 'Record track'}</b><small>${REC.on ? fd(REC.dist)+' recorded so far' : 'Draw your path on the map and save it as a GPX file'}</small></span>
      <span class="rec-dot${REC.on ? ' on' : ''}"></span></button>
    ${store.get('parked') ? `<button class="tg" data-tool="parkgo"><span class="row-txt"><b>Go to parked car</b><small>Route back to your saved spot</small></span><span class="park-badge">P</span></button>
      <button class="tg" data-tool="parkclear"><span class="row-txt"><b>Clear parking spot</b><small>Remove the P marker</small></span></button>`
      : `<button class="tg" data-tool="park"><span class="row-txt"><b>Save parking spot</b><small>Drop a P marker so you can find your car</small></span><span class="park-badge">P</span></button>`}
    <button class="tg" data-tool="trips"><span class="row-txt"><b>Trip history</b><small>Your recent drives and weekly totals</small></span><span class="chev">›</span></button>
    <h3 class="label">Routing</h3>
    <button class="tg" data-tool="opts"><span class="row-txt"><b>Route options</b><small>${[settings.trafficRouting ? 'Live traffic' : 'No traffic', settings.avoidTolls && 'no tolls', settings.avoidHighways && 'no highways', settings.avoidFerries && 'no ferries', settings.avoidUnpaved && 'no unpaved'].filter(Boolean).join(' · ')}</small></span><span class="chev">›</span></button>
    <h3 class="label">Vehicle</h3>
    <div class="num-row">
      ${imperial() ? `<label>Fuel economy<span><input type="number" inputmode="decimal" step="0.1" min="0" id="nMpg" data-num="mpg" value="${settings.mpg || ''}"> mpg</span></label>
        <label>Fuel price<span>${esc(settings.currency || '$')}<input type="number" inputmode="decimal" step="0.01" min="0" id="nPriceGal" data-num="priceGal" value="${settings.priceGal || ''}"> /gal</span></label>`
      : `<label>Fuel use<span><input type="number" inputmode="decimal" step="0.1" min="0" id="nL100" data-num="l100" value="${settings.l100 || ''}"> L/100 km</span></label>
        <label>Fuel price<span>${esc(settings.currency || '$')}<input type="number" inputmode="decimal" step="0.01" min="0" id="nPriceL" data-num="priceL" value="${settings.priceL || ''}"> /L</span></label>`}
    </div>
    <h3 class="label">Settings</h3>
    <div class="set-row"><span>Units</span>${seg('units', [['imperial','Miles'],['metric','Kilometers']])}</div>
    <div class="set-row"><span>Voice guidance</span>${seg('voice', [['true','On'],['false','Off']])}</div>
    <div class="set-row"><span>Day and night</span>${seg('theme', [['auto','System'],['sun','Sunset'],['light','Day'],['dark','Night']])}</div>
    <div class="set-row"><span>Map direction</span>${seg('orient', [['heading','Turns with you'],['north','North up']])}</div>
    ${tg('useCompass', 'Phone compass', 'Point the arrow, and the map, the way your phone faces when you’re stopped')}
    ${tg('speedAlert', 'Speeding chime', 'Play a tone when you go over the speed limit')}
    ${tg('view3d', '3D driving view', 'Tilt the map toward the horizon while you navigate')}
    ${tg('showSignals', 'Traffic lights on route', 'Show traffic signals along your route while navigating')}
    ${tg('camAlerts', 'Speed camera alerts', 'Warn before speed and red light cameras mapped on OpenStreetMap')}
    <div class="set-row"><span>Mapbox token</span><div class="keyin"><input id="mbTok2" type="password" autocomplete="off" spellcheck="false" value="${esc(MB_TOKEN)}"><button class="btn" data-mbsave>Save</button></div></div>
    <div class="set-row"><span>Saved data</span><div class="seg"><button data-clear="places">Clear places</button><button data-clear="reports">Clear reports</button></div></div>
    <p class="fine">Map, traffic, search and live-traffic routes © Mapbox © OpenStreetMap contributors · routes © Waymarked Trails · incidents © TomTom · radar: NWS NEXRAD via Iowa Environmental Mesonet · warnings: National Weather Service · weather: Open-Meteo · trails, places and speed limits from Overpass and OpenStreetMap · public lands from USFS and BLM</p>`);
  mSheet.dataset.view = 'layers';
  if (wasOpen) mSheet.scrollTop = keep;
}
const rerenderModal = () => mSheet.dataset.view === 'routeopts' ? renderRouteOpts() : mSheet.dataset.view === 'stops' ? null : renderLayers();
$('#btnLayers').onclick = renderLayers;
mSheet.addEventListener('click', e => {
  const b = e.target.closest('button'); if (!b) return;
  if (b.dataset.rep) return addReport(b.dataset.rep);
  if ('ttsave' in b.dataset){ saveTrafficKey(); return; }
  if ('mbsave' in b.dataset){ const v = $('#mbTok2').value.trim(); if (!/^pk\./.test(v)){ toast('A Mapbox public token starts with pk.'); return; } store.set('mbtoken', v); location.reload(); return; }
  if (b.dataset.base){ settings.base = b.dataset.base; saveSettings(); applyTheme(); renderLayers(); return; }
  if (b.dataset.tg){ const k = b.dataset.tg; settings[k] = !settings[k]; saveSettings(); if (k.startsWith('src')) resetTrails();
    if (k === 'useCompass'){ settings[k] ? compassOn() : compassOff(); }
    if (k === 'view3d'){ map.removeLayer(buildings3d); map.addLayer(buildings3d); apply3D(); }
    if (/^(avoid|trafficRouting)/.test(k)) S.routesStale = true;
    applyOverlays(); rerenderModal(); return; }
  if (b.dataset.tool === 'park'){ askPark(); return; }
  if (b.dataset.tool === 'parkgo'){ const p = store.get('parked'); closeModal(); if (p) previewPlace({ name:'Parked car', addr:'Your saved parking spot', lat:p.lat, lng:p.lng }); return; }
  if (b.dataset.tool === 'parkclear'){ store.set('parked', null); drawPark(); renderLayers(); toast('Parking spot cleared'); return; }
  if (b.dataset.tool === 'opts'){ renderRouteOpts(); return; }
  if (b.dataset.tool === 'trips'){ renderTrips(); return; }
  if (b.dataset.tool === 'rec'){ if (REC.on) recStop(); else { closeModal(); recStart(); } return; }
  if (b.dataset.rec === 'save'){ saveGPX(); return; }
  if (b.dataset.rec === 'clear'){ if (REC.line){ map.removeLayer(REC.line); REC.line = null; } REC.pts = []; closeModal(); return; }
  if (b.dataset.rec === 'done'){ closeModal(); return; }
  if (b.dataset.set){
    const k = b.dataset.set; let v = b.dataset.val; if (v === 'true' || v === 'false') v = v === 'true';
    settings[k] = v; saveSettings();
    if (k === 'theme') applyTheme();
    if (k === 'orient'){ renderOrient(); if (settings.orient === 'heading' && S.pos){ S.follow = true; $('#btnRecenter').hidden = true; } followCam(500); kickAnim(); }
    if (k === 'units'){ updateSpeedo(); updateAlt(); if (S.nav && S.pos) navTick(); else if (!S.nav) S.dest ? renderPreview() : renderIdle(); }
    if (k === 'voice' && !v) speechSynthesis?.cancel?.();
    if (k === 'voice') renderMute();
    if (k === 'units'){ WX.condAt = 0; refreshConditions(true); S.routes.forEach(r => { r._wx = null; r._elev = null; }); }
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
  $('#place').hidden = false; document.body.classList.add('placing'); apply3D();
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
  $('#place').hidden = true; document.body.classList.remove('placing'); apply3D();
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
const BASES = { street:1, topo:1, satellite:1 };
let baseKey = null;
function setBase(){
  const key = BASES[settings.base] ? settings.base : 'street';
  baseKey = key;
  map.setStyle(MB_STYLES[key](isDark()));
}
/* 3D buildings, drawn from the style's own building data */
const buildings3d = L.glLayer(mbm => {
  if (!mbm.getSource('composite') || !settings.view3d) return null;
  const dark = isDark() && baseKey !== 'satellite';
  return { layers:[{ id:'gp-buildings', type:'fill-extrusion', source:'composite', 'source-layer':'building', minzoom:14.5,
    filter:['==', ['get', 'extrude'], 'true'],
    paint:{ 'fill-extrusion-color':dark ? '#2a3442' : '#d9dde3', 'fill-extrusion-opacity':.82,
      'fill-extrusion-height':['interpolate', ['linear'], ['zoom'], 14.5, 0, 15.5, ['get', 'height']],
      'fill-extrusion-base':['get', 'min_height'] } }] };
}, 470);
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
  applyTraffic(); applyRadar(); applyWarnings(); map.addLayer(buildings3d); renderLegend();
}

/* ---------- off-road trails: OpenStreetMap + USFS Motor Vehicle Use Map + BLM GTLF ---------- */
const GRADES = {
  grade1:['#2f9e44','Grade 1','Solid surface, gravel or paved'],
  grade2:['#94c11f','Grade 2','Mostly solid gravel'],
  grade3:['#e6ac00','Grade 3','Mix of hard and soft ground'],
  grade4:['#f07818','Grade 4','Mostly soft dirt or grass'],
  grade5:['#d6332a','Grade 5','Soft ground, high clearance']
};
const OR_UNK = '#9a7b55', OR_4WD = '#8e44c9', OR_CLOSED = '#8d969b', OR_MIN_Z = 11.5;
// Official access classes shared by USFS MVUM and BLM travel plans.
const ACC = {
  all:{ color:'#0b72d9', label:'Open to all vehicles', short:'All' },
  hwy:{ color:'#4a5a8c', label:'Street-legal vehicles only', short:'Street legal' },
  ohv50:{ color:'#d81b60', label:'Vehicles 50″ wide or less (ATV/UTV)', short:'≤50″' },
  moto:{ color:'#00897b', label:'Motorcycles only', short:'Moto' },
  special:{ color:'#3b3b3b', label:'Special designation, check rules', short:'Special' },
  limited:{ color:'#0b72d9', label:'Limited public use, check rules', short:'Limited', dash:'12 5 2 5' }
};
const USFS_SYM = { 1:['all',0], 2:['all',1], 3:['hwy',0], 4:['hwy',1], 5:['all',0], 6:['all',1], 7:['ohv50',0], 8:['ohv50',1],
  9:['moto',0], 10:['moto',1], 11:['special',0], 12:['special',1], 16:['ohv50',0], 17:['ohv50',1] };
const USFS_URL = 'https://apps.fs.usda.gov/arcx/rest/services/EDW/EDW_MVUM_02/MapServer/';
const BLM_URL = 'https://gis.blm.gov/arcgis/rest/services/transportation/BLM_Natl_GTLF_Public_Display/MapServer/';
const SOURCES = [
  { key:'srcOsm', label:'OpenStreetMap', sub:'Community-mapped dirt and 4×4 tracks, worldwide' },
  { key:'srcUsfs', label:'National Forests (USFS)', sub:'Official Motor Vehicle Use Map roads and trails, with vehicle types and open dates' },
  { key:'srcBlm', label:'BLM public lands', sub:'Official BLM roads and trails managed for motorized use' }
];
const OR = { layer:L.layerGroup(), ways:new Map(), loaded:null, busy:false, status:'', failed:[], hl:null, sel:null, gen:0 };
const trailRenderer = L.canvas({ pane:'trails', padding:.4, tolerance:10 });
const isClosed = t => /^(private|no)$/.test(t.access || '') || /^(private|no)$/.test(t.motor_vehicle || '');
const cap = s => s ? String(s).replace(/_/g, ' ').replace(/^./, c => c.toUpperCase()) : '';
const tidy = s => { if (s == null || String(s).trim() === '') return ''; s = String(s).trim(); return cap(s === s.toUpperCase() ? s.toLowerCase() : s); };
function trailStyle(t){
  const closed = isClosed(t), fwd = t['4wd_only'] === 'yes';
  return { color: closed ? OR_CLOSED : fwd ? OR_4WD : (GRADES[t.tracktype]?.[0] || OR_UNK),
    weight: fwd ? 5 : 4, opacity: closed ? .75 : .95, lineCap:'round',
    dashArray: closed ? '2 8' : (t.highway === 'path' || t.highway === 'bridleway') ? '9 6' : null };
}
function accStyle(cls, seasonal, trail){
  const A = ACC[cls] || ACC.special;
  return { color:A.color, weight: trail ? 3.5 : 4.5, opacity:.95, lineCap:'round', dashArray: A.dash || (seasonal ? '8 6' : null) };
}
function addLine(key, lines, style, info){
  if (OR.ways.has(key)) return;
  const line = L.polyline(lines, { renderer:trailRenderer, ...style });
  line.on('click', e => { L.DomEvent.stop(e); openTrail(info(), lines, e.latlng); });
  const g = L.layerGroup([L.polyline(lines, { renderer:trailRenderer, color:'#ffffff', weight:style.weight + 3, opacity:.55, interactive:false }), line]);
  g.addTo(OR.layer); OR.ways.set(key, g);
}

/* OpenStreetMap via Overpass */
async function loadOSM(pb, gen){
  const bb = [pb.getSouth(), pb.getWest(), pb.getNorth(), pb.getEast()].map(n => n.toFixed(5)).join(',');
  const V = '"^(yes|permissive|designated)$"';
  const ql = `[out:json][timeout:25][bbox:${bb}];(way[highway=track];way["4wd_only"=yes];`
    + `way[highway~"^(path|bridleway)$"][motor_vehicle~${V}];way[highway~"^(path|track|unclassified)$"][atv~${V}];`
    + `way[highway~"^(path|track)$"][motorcycle~${V}];);out geom 5000;`;
  const r = await opFetch({ method:'POST', body:'data='+encodeURIComponent(ql),
    headers:{ 'Content-Type':'application/x-www-form-urlencoded' } });
  if (!r.ok) throw new Error('osm');
  const j = await r.json();
  if (gen !== OR.gen) return;
  for (const w of j.elements || []){
    if (w.type !== 'way' || !w.geometry) continue;
    const t = w.tags || {};
    addLine('osm:'+w.id, [w.geometry.map(g => [g.lat, g.lon])], trailStyle(t), () => osmInfo(t));
  }
}
function osmInfo(t){
  const g = GRADES[t.tracktype], closed = isClosed(t), fwd = t['4wd_only'] === 'yes';
  const kind = fwd ? '4×4 only track' : t.highway === 'track' ? 'Dirt track' : t.highway === 'bridleway' ? 'Bridleway' : t.highway === 'path' ? 'Trail' : 'Road';
  const veh = [['motor_vehicle','Vehicles'],['atv','ATV'],['motorcycle','Motorcycle']].filter(([k]) => t[k]).map(([k, l]) => l+': '+t[k]).join(', ');
  return { name:t.name || t.ref || 'Unnamed '+kind.toLowerCase(), kind, src:'OpenStreetMap',
    badge: g || fwd || closed ? [trailStyle(t).color, closed ? 'Closed' : fwd ? '4×4 only' : g[1]] : null,
    rows:[['Grade', g ? g[1]+' · '+g[2] : 'Not graded'], ['Surface', tidy(t.surface)], ['Condition', tidy(t.smoothness)],
      ['Access', closed ? 'Closed or private' : tidy(t.access) || 'Not posted'], ['Allowed', veh]] };
}

/* USFS + BLM ArcGIS feature services (GeoJSON queries by map extent) */
async function loadArc(src, base, layers, pb, gen){
  const z = map.getZoom(), tol = (360 / (256 * 2 ** z)).toFixed(7);   // ~1 screen pixel, simplifies geometry server-side
  const env = [pb.getWest(), pb.getSouth(), pb.getEast(), pb.getNorth()].map(n => n.toFixed(5)).join(',');
  await Promise.all(layers.map(async lyr => {
    const u = `${base}${lyr}/query?where=1%3D1&geometry=${env}&geometryType=esriGeometryEnvelope&inSR=4326`
      + `&spatialRel=esriSpatialRelIntersects&outFields=*&returnGeometry=true&outSR=4326&maxAllowableOffset=${tol}&geometryPrecision=6&f=geojson`;
    const r = await fetch(u);
    if (!r.ok) throw new Error(src);
    const j = await r.json();
    if (j.error) throw new Error(src);
    if (gen !== OR.gen) return;
    for (const f of j.features || []) addArcFeature(src, lyr, f);
  }));
}
function addArcFeature(src, lyr, f){
  const g = f.geometry; if (!g) return;
  const flip = l => l.map(c => [c[1], c[0]]);
  const lines = g.type === 'LineString' ? [flip(g.coordinates)] : g.type === 'MultiLineString' ? g.coordinates.map(flip) : null;
  if (!lines || !lines.length) return;
  const p = f.properties || {}, id = f.id ?? p.objectid ?? p.OBJECTID ?? p.globalid ?? p.GlobalID;
  if (src === 'usfs'){
    const [cls, seas] = USFS_SYM[p.symbol] || ['special', 0];
    addLine(`usfs${lyr}:${id}`, lines, accStyle(cls, seas, lyr === 2), () => usfsInfo(p, lyr, cls, seas));
  } else {
    const cls = blmClass(p, lyr), seas = /^(y|yes|seasonal)/i.test(String(p.PLAN_SEASON_RSTRCT_CODE || ''));
    addLine(`blm${lyr}:${id}`, lines, accStyle(cls, seas, lyr >= 2), () => blmInfo(p, lyr, cls, seas));
  }
}
const USFS_VEH = [['passengervehicle','Passenger car'],['highclearancevehicle','High-clearance'],['truck','Truck'],
  ['fourwd_gt50inches','4WD over 50″'],['twowd_gt50inches','2WD over 50″'],['atv','ATV'],['other_ohv_lt50inches','UTV/OHV 50″ or less'],
  ['motorcycle','Motorcycle'],['otherwheeled_ohv','Other wheeled OHV']];
const isOpen = v => v != null && String(v).trim() !== '' && !/^(no|n|null|closed|none|n\/a)$/i.test(String(v).trim());
function usfsInfo(p, lyr, cls, seas){
  const road = lyr === 1;
  const open = USFS_VEH.filter(([k]) => isOpen(p[k])).map(([k, l]) => {
    const d = String(p[k.replace('inches', '')+'_datesopen'] || '').trim();
    return d && !/^0?1\/0?1\s*-\s*12\/31$/.test(d) ? `${l} (${d})` : l;
  }).join(', ');
  return { name: tidy(p.name) || (road ? 'Forest road ' : 'Forest trail ')+(p.id || ''), kind: (road ? 'Forest road' : 'Forest trail')+(p.id ? ' '+p.id : ''),
    src:'USFS Motor Vehicle Use Map', badge:[ACC[cls].color, ACC[cls].short+(seas ? ' · seasonal' : '')],
    rows:[['Access', ACC[cls].label+(seas ? ', seasonal' : ', yearlong')], ['Open to', open], ['Maintenance', tidy(p.operationalmaintlevel)],
      ['Surface', tidy(p.surfacetype)], ['Trail class', tidy(p.trailclass)], ['Forest', [tidy(p.forestname), tidy(p.districtname)].filter(Boolean).join(' · ')]] };
}
function blmClass(p, lyr){
  if (lyr === 1 || lyr === 3) return 'limited';
  const m = String(p.PLAN_ALLOW_MODE_TRNSPRT || p.PLAN_MODE_TRNSPRT || '');
  if (/motorcycle|single.?track/i.test(m) && !/atv|utv|4wd|4x4|full.?size|all/i.test(m)) return 'moto';
  if (/atv|utv|side.?by|less than 50|<\s*50|50"?\s*or less/i.test(m) && !/full.?size|4wd|4x4|all/i.test(m)) return 'ohv50';
  return 'all';
}
function blmInfo(p, lyr, cls, seas){
  const road = lyr <= 1, mode = tidy(p.PLAN_MODE_TRNSPRT), allow = tidy(p.PLAN_ALLOW_MODE_TRNSPRT);
  return { name: tidy(p.ROUTE_PRMRY_NM) || (road ? 'BLM road' : 'BLM trail'), kind: (road ? 'BLM road' : 'BLM trail')+(p.ADMIN_ST ? ' · '+p.ADMIN_ST : ''),
    src:'BLM Ground Transportation Linear Features', badge:[ACC[cls].color, ACC[cls].short+(seas ? ' · seasonal' : '')],
    rows:[['Access', ACC[cls].label], ['Designation', tidy(p.PLAN_OHV_ROUTE_DSGNTN)], ['Allowed', allow], ['Mode', mode !== allow ? mode : ''],
      ['Limits', tidy(p.OHV_DSGNTN_LIM_EXPLAIN || p.OHV_ROUTE_DSGNTN_LIM)], ['Seasonal', tidy(p.PLAN_SEASON_RSTRCT_CODE)],
      ['Restriction', tidy(p.PLAN_ACCESS_RSTRCT)], ['Surface', tidy(p.OBSRVE_SRFCE_TYPE)]] };
}

/* loading + popups + legend */
let trailT = null;
map.on('moveend', () => { if (!settings.ovOffroad) return; clearTimeout(trailT); trailT = setTimeout(loadTrails, 600); });
function resetTrails(){ OR.gen++; OR.layer.clearLayers(); OR.ways.clear(); OR.loaded = null; OR.failed = []; clearTrailHL(); map.closePopup(); }
async function loadTrails(){
  if (!settings.ovOffroad) return;
  if (!SOURCES.some(s => settings[s.key])){ OR.status = 'none'; return renderLegend(); }
  if (map.getZoom() < OR_MIN_Z){ OR.status = 'zoom'; return renderLegend(); }
  if (OR.loaded && OR.loaded.contains(map.getBounds())){ OR.status = 'ok'; return renderLegend(); }
  if (OR.busy) return;
  const pb = map.getBounds().pad(.3), gen = OR.gen;
  if (OR.ways.size > 12000){ OR.layer.clearLayers(); OR.ways.clear(); }
  OR.busy = true; OR.status = 'loading'; renderLegend();
  const jobs = [];
  if (settings.srcOsm) jobs.push(['OpenStreetMap', loadOSM(pb, gen)]);
  if (settings.srcUsfs) jobs.push(['Forest Service', loadArc('usfs', USFS_URL, [1, 2], pb, gen)]);
  if (settings.srcBlm) jobs.push(['BLM', loadArc('blm', BLM_URL, [0, 1, 2, 3], pb, gen)]);
  const res = await Promise.allSettled(jobs.map(j => j[1]));
  OR.busy = false;
  if (gen !== OR.gen) return loadTrails();          // sources changed mid-load
  OR.failed = jobs.filter((_, i) => res[i].status === 'rejected').map(j => j[0]);
  OR.status = OR.failed.length === jobs.length ? 'error' : 'ok';
  if (OR.status === 'ok') OR.loaded = pb;
  renderLegend();
  if (OR.status === 'ok' && settings.ovOffroad && map.getZoom() >= OR_MIN_Z && !OR.loaded.contains(map.getBounds())) setTimeout(loadTrails, 300);
}
function openTrail(info, lines, at){
  if (PLACE){ PLACE.m.setLatLng(at); plSub(); return; }
  let len = 0; for (const l of lines) for (let i = 1; i < l.length; i++) len += dist(l[i-1], l[i]);
  clearTrailHL();
  OR.hl = L.polyline(lines, { renderer:trailRenderer, color:'#ffe14d', weight:12, opacity:.6, interactive:false }).addTo(map);
  OR.sel = { ends:lines.flatMap(l => [l[0], l[l.length - 1]]), name:info.name };
  const rows = info.rows.filter(r => r[1]).concat([['Length', fd(len)], ['Source', info.src]]);
  L.popup({ maxWidth:290 }).setLatLng(at).setContent(`<div class="pop"><b>${esc(info.name)}</b><small>${esc(info.kind)}</small>
    ${info.badge ? `<span class="tp-grade" style="background:${info.badge[0]}">${esc(info.badge[1])}</span>` : ''}
    <dl>${rows.map(([k, v]) => `<dt>${k}</dt><dd>${esc(v)}</dd>`).join('')}</dl>
    <button class="go" data-trail-go>Directions to trail</button></div>`).openOn(map);
}
function clearTrailHL(){ if (OR.hl){ map.removeLayer(OR.hl); OR.hl = null; } }
map.on('popupclose', clearTrailHL);
map.on('popupopen', e => {
  const b = e.popup.getElement()?.querySelector('[data-trail-go]'); if (!b) return;
  b.onclick = () => {
    const s = OR.sel; if (!s) return;
    const from = S.pos || [map.getCenter().lat, map.getCenter().lng];
    const end = s.ends.reduce((a, b) => dist(from, b) < dist(from, a) ? b : a);
    map.closePopup();
    previewPlace({ name:s.name, addr:'Off-road trail · nearest end', lat:end[0], lng:end[1] });
  };
});
function renderLegend(){
  renderTrafficKey();
  const el = $('#legend');
  if (!settings.ovOffroad || S.nav){ el.hidden = true; return; }
  const st = { zoom:'Zoom in to load trails', loading:'Loading trails…', error:'Trails didn’t load. Move the map to retry.',
    none:'Turn on a trail source in Layers' }[OR.status] || (OR.failed.length ? OR.failed.join(' and ')+' didn’t load' : '');
  const open = settings.legendOpen !== false;
  let h = `<button class="lg-head" data-lg aria-expanded="${open}"><span class="lg-t">Off-road</span>`
    + (st ? `<span class="lg-status${OR.status === 'loading' ? ' busy' : ''}">${st}</span>` : '')
    + `<span class="lg-car">${open ? 'Hide key' : 'Key'}</span></button>`;
  if (open && settings.srcOsm) h += '<div class="lg-row"><span class="lg-k">OSM grade</span>'
    + Object.values(GRADES).map((g, i) => `<span class="lg" title="${g[1]}: ${g[2]}"><i style="background:${g[0]}"></i>${i + 1}</span>`).join('')
    + `<span class="lg"><i style="background:${OR_4WD}"></i>4×4</span><span class="lg"><i class="dash" style="--c:${OR_CLOSED}"></i>Closed</span></div>`;
  if (open && (settings.srcUsfs || settings.srcBlm)) h += '<div class="lg-row"><span class="lg-k">Public land</span>'
    + ['all','hwy','ohv50','moto','special'].map(k => `<span class="lg" title="${ACC[k].label}"><i style="background:${ACC[k].color}"></i>${ACC[k].short}</span>`).join('')
    + `<span class="lg"><i class="dash" style="--c:${ACC.all.color}"></i>Seasonal</span></div>`;
  el.innerHTML = h; el.hidden = false;
}
$('#legend').addEventListener('click', e => {
  if (!e.target.closest('[data-lg]')) return;
  settings.legendOpen = settings.legendOpen === false; saveSettings(); renderLegend();
});

/* ---------- live traffic (TomTom raster flow + incidents, user's own key) ---------- */
const TT = { flow:null, inc:null, timer:null, sig:null, warned:false };
const TT_COLORS = [['#f7a21b','Slow'],['#e8382d','Heavy'],['#8b1a1a','Stopped']];
const trafficKey = () => String(store.get('ttkey', '') || '').trim();
const mbTraffic = L.glLayer(() => ({
  sources:[['gp-mbtraffic', { type:'vector', url:'mapbox://mapbox.mapbox-traffic-v1' }]],
  layers:[{ id:'gp-mbtraffic', type:'line', source:'gp-mbtraffic', 'source-layer':'traffic',
    filter:['in', ['get', 'congestion'], ['literal', ['moderate', 'heavy', 'severe']]],
    layout:{ 'line-cap':'round', 'line-join':'round' },
    paint:{ 'line-color':['match', ['get', 'congestion'], 'moderate', '#f7a21b', 'heavy', '#e8382d', 'severe', '#8b1a1a', '#2fbf4a'],
      'line-width':['interpolate', ['linear'], ['zoom'], 8, 1.2, 13, 2.6, 17, 6], 'line-offset':['interpolate', ['linear'], ['zoom'], 10, .5, 17, 3], 'line-opacity':.95 } }]
}), 300);
function applyTraffic(){
  const key = trafficKey(), dark = (isDark() && baseKey === 'street') || baseKey === 'satellite';
  if (settings.ovTraffic) map.addLayer(mbTraffic); else map.removeLayer(mbTraffic);
  const sig = settings.ovTraffic && key && settings.trafficInc ? [key, dark].join('|') : '';
  if (sig !== TT.sig){
    TT.sig = sig;
    if (TT.inc){ map.removeLayer(TT.inc); TT.inc = null; }
    clearInterval(TT.timer);
    if (sig){
      TT.inc = L.tileLayer(`https://api.tomtom.com/traffic/map/4/tile/incidents/${dark ? 'night' : 's1'}/{z}/{x}/{y}.png?key=${encodeURIComponent(key)}&ts={ts}`,
        { pane:'overTiles', maxZoom:19, ts:() => Math.floor(Date.now() / 120000), attribution:'Incidents &copy; TomTom' }).addTo(map);
      TT.timer = setInterval(() => { if (document.visibilityState === 'visible') TT.inc?.redraw(); }, 120000);
    }
  }
  renderTrafficKey();
}
function renderTrafficKey(){
  const el = $('#tkey');
  if (!settings.ovTraffic || S.nav){ el.hidden = true; return; }
  el.innerHTML = '<span class="lg-t">Traffic</span>' + TT_COLORS.map(([c, l]) => `<span class="tk"><i style="background:${c}"></i>${l}</span>`).join('');
  el.hidden = false;
}
function saveTrafficKey(){
  const v = ($('#ttKey')?.value || '').trim();
  store.set('ttkey', v); TT.warned = false; TT.sig = null;
  if (v && !settings.ovTraffic){ settings.ovTraffic = true; saveSettings(); }
  applyTraffic(); renderLayers();
  toast(v ? 'Traffic key saved' : 'Traffic key removed');
}
mSheet.addEventListener('keydown', e => { if (e.key === 'Enter' && e.target.id === 'ttKey'){ e.preventDefault(); saveTrafficKey(); } });

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
    try{ r = await opFetch({ method:'POST', body:'data='+encodeURIComponent(ql), headers:{ 'Content-Type':'application/x-www-form-urlencoded' } }); }
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
      S.follow = false; $('#btnRecenter').hidden = !S.pos; northUp();
      map.fitBounds(L.latLngBounds(S.list.slice(0, 10).map(p => [p.lat, p.lng]).concat([c])),
        { paddingTopLeft:[30, $('#top').offsetHeight + 30], paddingBottomRight:[70, sheet.offsetHeight + 30], maxZoom:16 }); northUp();
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

/* ---------- routing engines: TomTom (live traffic) or OSRM ---------- */
/* Mapbox Directions with live traffic: same response shape as OSRM, plus congestion and speed limits */
async function fetchMapbox(from, to, alts){
  const pts = [from, ...(S.stop ? [[S.stop.lat, S.stop.lng]] : []), to];
  const ex = [settings.avoidTolls && 'toll', settings.avoidHighways && 'motorway', settings.avoidFerries && 'ferry', settings.avoidUnpaved && 'unpaved'].filter(Boolean);
  const u = `https://api.mapbox.com/directions/v5/mapbox/driving-traffic/${pts.map(p => p[1].toFixed(6)+','+p[0].toFixed(6)).join(';')}`
    + `?alternatives=${alts && !S.stop ? 'true' : 'false'}&geometries=geojson&overview=full&steps=true&annotations=congestion,maxspeed`
    + (ex.length ? '&exclude='+ex.join(',') : '') + `&access_token=${encodeURIComponent(MB_TOKEN)}`;
  const r = await fetch(u);
  const j = await r.json().catch(() => ({}));
  if (!r.ok || j.code !== 'Ok' || !j.routes?.length){
    if (j.code === 'NoRoute' || j.code === 'NoSegment'){ const e = new Error('No drivable route to this place.'); e.noRoute = true; throw e; }
    throw new Error('mapbox');
  }
  const LVL = { moderate:1, heavy:2, severe:3 };
  return j.routes.map(rt => {
    const R = prepRoute(rt);
    R.engine = 'mapbox';
    R.delay = rt.duration_typical ? Math.max(0, rt.duration - rt.duration_typical) : 0;
    const cg = (rt.legs || []).flatMap(l => l.annotation?.congestion || []);
    R.traffic = [];
    for (let i = 0; i < cg.length;){
      const m = LVL[cg[i]] || 0; let k = i + 1;
      while (k < cg.length && (LVL[cg[k]] || 0) === m) k++;
      if (m) R.traffic.push({ a:i, b:Math.min(k, R.coords.length - 1), mag:m, cat:'JAM', delay:0 });
      i = k;
    }
    return R;
  });
}
async function fetchRoutes(from, to, alts){
  if (settings.trafficRouting){
    try{ return await fetchMapbox(from, to, alts); }
    catch(e){ if (e.noRoute) throw e; }
    if (trafficKey()){
      try{ return await fetchTomTom(from, to, alts); }
      catch(e){ if (e.noRoute) throw e; }
    }
    toast('Live-traffic routing didn’t respond, so this route ignores traffic.', 4000);
  }
  return fetchOSRM(from, to, alts);
}
async function fetchOSRM(from, to, alts){
  const ex = [settings.avoidTolls && 'toll', settings.avoidHighways && 'motorway', settings.avoidFerries && 'ferry'].filter(Boolean);
  const pts = [from, ...(S.stop ? [[S.stop.lat, S.stop.lng]] : []), to];
  const base = `https://router.project-osrm.org/route/v1/driving/${pts.map(p => p[1].toFixed(6)+','+p[0].toFixed(6)).join(';')}?overview=full&geometries=geojson&steps=true&alternatives=${alts && !S.stop ? 'true' : 'false'}`;
  const get = async url => {
    let r; try{ r = await fetch(url); }catch{ throw new Error('Could not reach the routing service. Check your connection.'); }
    const j = await r.json().catch(() => ({}));
    return { ok:r.ok && j.code === 'Ok' && j.routes?.length, j, status:r.status };
  };
  let res = await get(base + (ex.length ? '&exclude='+ex.join(',') : '')), note = '';
  if (!res.ok && ex.length){ res = await get(base); note = 'The free router can’t apply your avoid settings to this trip.'; }
  if (!res.ok && res.j.code === 'InvalidQuery' && /alternatives=true/.test(base)) res = await get(base.replace('alternatives=true', 'alternatives=false'));
  if (!res.ok){
    if (res.j.code === 'NoRoute' || res.j.code === 'Ok') throw new Error('No drivable route to this place.');
    throw new Error('Routing service returned '+(res.j.code || res.status)+'. Try again in a moment.');
  }
  return res.j.routes.map(r => Object.assign(prepRoute(r), { engine:'osrm', note }));
}
async function fetchTomTom(from, to, alts){
  const av = [settings.avoidTolls && 'tollRoads', settings.avoidHighways && 'motorways', settings.avoidFerries && 'ferries', settings.avoidUnpaved && 'unpavedRoads'].filter(Boolean);
  const locs = [from, ...(S.stop ? [[S.stop.lat, S.stop.lng]] : []), to].map(p => p[0].toFixed(6)+','+p[1].toFixed(6)).join(':');
  if (S.stop) alts = false;
  const u = `https://api.tomtom.com/routing/1/calculateRoute/${locs}/json`
    + `?key=${encodeURIComponent(trafficKey())}&traffic=true&travelMode=car&routeType=fastest&instructionsType=coded&sectionType=traffic&sectionType=speedLimit&computeTravelTimeFor=all`
    + (alts ? '&maxAlternatives=2' : '') + av.map(a => '&avoid='+a).join('');
  const r = await fetch(u);
  const j = await r.json().catch(() => ({}));
  if (!r.ok || !j.routes?.length){
    const msg = j.error?.description || j.detailedError?.message || '';
    if (/no route|NO_ROUTE_FOUND|unreachable/i.test(msg)){ const e = new Error('No drivable route to this place.'); e.noRoute = true; throw e; }
    throw new Error('tomtom');
  }
  return j.routes.map(tt => {
    const R = prepRoute(ttToOsrm(tt));
    R.engine = 'tomtom'; R.delay = tt.summary.trafficDelayInSeconds || 0;
    const sl = (tt.sections || []).filter(x => x.sectionType === 'SPEED_LIMIT' && x.maxSpeedLimitInKmh);
    if (sl.length){ R.limits = new Array(R.coords.length).fill(null); for (const x of sl) for (let i = x.startPointIndex; i < x.endPointIndex && i < R.limits.length; i++) R.limits[i] = x.maxSpeedLimitInKmh; }
    R.traffic = (tt.sections || []).filter(s => s.sectionType === 'TRAFFIC')
      .map(s => ({ a:s.startPointIndex, b:s.endPointIndex, mag:s.magnitudeOfDelay ?? 0, cat:s.simpleCategory, delay:s.delayInSeconds || 0 }));
    return R;
  });
}
function ttToOsrm(tt){
  const pts = tt.legs.flatMap(l => l.points).map(p => [p.longitude, p.latitude]);
  const ins = (tt.guidance?.instructions || []).filter(i => i.maneuver !== 'WAYPOINT_AHEAD');
  let steps = ins.map((i, n) => {
    const nx = ins[n + 1];
    return { distance: nx ? nx.routeOffsetInMeters - i.routeOffsetInMeters : 0, duration: nx ? nx.travelTimeInSeconds - i.travelTimeInSeconds : 0,
      name: i.street || '', ref: (i.roadNumbers || [])[0] || '', exits: i.exitNumber || '', maneuver: ttManeuver(i, pts), waypoint: /^WAYPOINT/.test(i.maneuver || '') };
  });
  if (steps.length < 2) steps = [
    { distance:tt.summary.lengthInMeters, duration:tt.summary.travelTimeInSeconds, name:'', maneuver:ttManeuver({ maneuver:'DEPART', pointIndex:0 }, pts) },
    { distance:0, duration:0, name:'', maneuver:{ type:'arrive' } }];
  return { geometry:{ coordinates:pts }, legs:[{ steps }], duration:tt.summary.travelTimeInSeconds, distance:tt.summary.lengthInMeters };
}
function ttManeuver(i, pts){
  const m = i.maneuver || '', ang = i.turnAngleInDecimalDegrees || 0;
  const side = /LEFT/.test(m) ? 'left' : /RIGHT/.test(m) ? 'right' : ang < -10 ? 'left' : ang > 10 ? 'right' : '';
  if (m === 'DEPART'){ const k = i.pointIndex || 0, a = pts[k], b = pts[Math.min(k + 1, pts.length - 1)]; return { type:'depart', bearing_after: bearing([a[1], a[0]], [b[1], b[0]]) }; }
  if (/^ARRIVE|^WAYPOINT/.test(m)) return { type:'arrive', modifier:side };
  if (m === 'STRAIGHT' || m === 'FOLLOW' || /^SWITCH_/.test(m) || m === 'TAKE_FERRY') return { type:'continue', modifier:'straight' };
  if (/^KEEP_/.test(m)) return { type:'fork', modifier:'slight '+side };
  if (/^BEAR_/.test(m)) return { type:'turn', modifier:'slight '+side };
  if (/^TURN_/.test(m)) return { type:'turn', modifier:side };
  if (/^SHARP_/.test(m)) return { type:'turn', modifier:'sharp '+side };
  if (/UTURN/.test(m)) return { type:'turn', modifier:'uturn' };
  if (/^ROUNDABOUT/.test(m)) return { type:'roundabout', exit:i.roundaboutExitNumber || 1,
    modifier: m === 'ROUNDABOUT_LEFT' ? 'left' : m === 'ROUNDABOUT_RIGHT' ? 'right' : m === 'ROUNDABOUT_BACK' ? 'sharp left' : 'straight' };
  if (/^ENTER_|ENTRANCE_RAMP/.test(m)) return { type:'on ramp', modifier: side ? 'slight '+side : 'straight' };
  if (/EXIT/.test(m)) return { type:'off ramp', modifier:'slight '+(side || 'right') };
  return { type:'continue', modifier: side ? 'slight '+side : 'straight' };
}
const TRAFFIC_SEG = [null, '#f7a21b', '#e8382d', '#a3171a', '#7a0f12'];
function trafficSegs(R){
  return (R.traffic || []).map(t => ({ ...t, color: t.cat === 'ROAD_CLOSURE' ? '#2b2b2b' : TRAFFIC_SEG[t.mag] || (t.delay > 30 ? '#f7a21b' : null) }))
    .filter(t => t.color && t.b > t.a).map(t => ({ ...t, ll:R.coords.slice(t.a, t.b + 1) }));
}

/* ---------- faster-route checks while driving (TomTom only) ---------- */
let fasterT = null;
function startFasterChecks(){
  clearInterval(fasterT);
  if (!['tomtom', 'mapbox'].includes(S.nav?.route.engine)) return;
  fasterT = setInterval(async () => {
    const n = S.nav; if (!n || !S.pos || n.rerouting || S.sim || n.remT == null || n.remT < 300) return;
    try{
      const [r] = await (S.nav.route.engine === 'mapbox' ? fetchMapbox : fetchTomTom)(S.pos, S.dest.ll, false);
      if (S.nav !== n) return;
      const save = n.remT - r.duration;
      if (save > Math.max(180, n.remT * .08)){
        n.route = r; n.spoken = {}; n.off = 0; n.along = 0; drawNavRoute();
        const mins = Math.round(save / 60);
        say(`Faster route found. Saving ${mins} minute${mins === 1 ? '' : 's'}.`); toast(`Faster route found · saves ${mins} min`, 4500);
      }
    }catch{}
  }, 180000);
}

/* ---------- lane guidance (OSRM intersections) ---------- */
function laneSVG(ind){
  const a = ind === 'uturn' ? null : DIR_ANG[ind] ?? 0;
  if (a === null) return '<svg viewBox="0 0 24 32" fill="none" stroke="currentColor" stroke-width="3.2" stroke-linecap="round" stroke-linejoin="round"><path d="M16 30V14a5 5 0 0 0-10 0v6"/><path d="M2.5 16.5L6 21l3.5-4.5"/></svg>';
  return `<svg viewBox="0 0 24 32" fill="none" stroke="currentColor" stroke-width="3.2" stroke-linecap="round" stroke-linejoin="round"><path d="M12 30V20"/><g transform="rotate(${a} 12 20)"><path d="M12 20V5"/><path d="M6.5 10.5L12 5l5.5 5.5"/></g></svg>`;
}
function renderLanes(step, toNext){
  const el = $('#bLanes'), lanes = step?.intersections?.[0]?.lanes;
  if (!lanes || lanes.length < 2 || toNext > 1600 || S.nav?.rerouting){ el.hidden = true; return; }
  const mod = step.maneuver.modifier || 'straight';
  el.innerHTML = lanes.map(l => {
    const ind = (l.valid && l.indications.includes(mod)) ? mod : (l.indications[0] || 'straight');
    return `<span class="lane${l.valid ? ' on' : ''}">${laneSVG(ind === 'none' ? 'straight' : ind)}</span>`;
  }).join('');
  el.hidden = false;
}

/* ---------- trip controls: mute, overview, trip sheet, share ETA ---------- */
const ICON_VOL = '<svg viewBox="0 0 24 24" fill="currentColor"><path d="M4 9h4l5-4v14l-5-4H4z"/><path d="M16 8.5a5 5 0 0 1 0 7M18.5 6a8.5 8.5 0 0 1 0 12" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round"/></svg>';
const ICON_MUTE = '<svg viewBox="0 0 24 24" fill="currentColor"><path d="M4 9h4l5-4v14l-5-4H4z"/><path d="M16.5 9.5l5 5M21.5 9.5l-5 5" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round"/></svg>';
function renderMute(){ const b = $('#btnMute'); b.innerHTML = settings.voice ? ICON_VOL : ICON_MUTE; b.setAttribute('aria-label', settings.voice ? 'Mute voice' : 'Unmute voice'); b.classList.toggle('off', !settings.voice); }
$('#btnMute').onclick = () => { settings.voice = !settings.voice; saveSettings(); if (!settings.voice) speechSynthesis?.cancel?.(); renderMute(); toast(settings.voice ? 'Voice on' : 'Voice muted', 1500); };
$('#btnOverview').onclick = () => {
  const n = S.nav; if (!n) return;
  S.follow = false; $('#btnRecenter').hidden = false; apply3D();
  const rest = n.route.coords.slice(Math.max(0, snap(n.route, S.pos || n.route.coords[0], n.along - 150).i));
  northUp();
  map.fitBounds(L.latLngBounds(rest.concat(S.pos ? [S.pos] : [])), { paddingTopLeft:[40, $('#top').offsetHeight + 30], paddingBottomRight:[80, $('#navbar').offsetHeight + 40] });
};
function shareText(){
  const n = S.nav, d = S.dest; if (!d) return '';
  const eta = n?.remT != null ? clock(n.remT) : '';
  return `I’m heading to ${d.name}${eta ? ', arriving around '+eta : ''}. https://www.openstreetmap.org/?mlat=${d.lat.toFixed(5)}&mlon=${d.lng.toFixed(5)}#map=16/${d.lat.toFixed(5)}/${d.lng.toFixed(5)}`;
}
async function shareETA(){
  const text = shareText(); if (!text) return;
  try{ if (navigator.share){ await navigator.share({ title:'My ETA', text }); return; } }catch(e){ if (e.name === 'AbortError') return; }
  try{ await navigator.clipboard.writeText(text); toast('ETA copied. Paste it in a message.'); }
  catch{ openModal(`<h3 class="label" style="margin-top:0">Share your ETA</h3><textarea class="share-txt" id="shareTxt" readonly>${esc(text)}</textarea><p class="fine">Copy this text and send it.</p>`); $('#shareTxt').select(); }
}
function renderTrip(){
  const n = S.nav; if (!n) return;
  const R = n.route, k = n.k || 0;
  const delay = R.engine === 'tomtom' || R.engine === 'mapbox' ? (R.delay > 60 ? `<span class="delay bad">+${fmtDur(R.delay)} traffic</span>` : '<span class="delay ok">Light traffic</span>') : '';
  const items = R.steps.slice(k + 1).map((s, i) => {
    const idx = k + 1 + i, d = Math.max(0, R.stepStart[idx] - n.along);
    return `<li class="st"><span class="st-ico">${turnSVG(s)}</span><span class="row-txt"><b>${esc(instr(s))}</b></span><span class="row-d">${fd(d)}</span></li>`;
  }).join('');
  openModal(`<div class="trip-head"><div><b>${n.remT != null ? clock(n.remT) : '--'}</b><span>arrival</span></div>
      <div><b>${n.remT != null ? fmtDur(n.remT) : '--'}</b><span>${n.remD != null ? fd(n.remD) : ''}</span></div>${delay}</div>
    <div class="pv-actions three"><button class="btn primary" data-trip="share">Share ETA</button><button class="btn" data-trip="stop">${S.stop ? 'Change stop' : 'Add stop'}</button><button class="btn" data-trip="overview">Overview</button></div>
    <div class="trip-links"><button class="link" data-trip="hud">Windshield HUD</button><button class="btn danger" data-trip="end">End drive</button></div>
    ${CAM.list.length ? `<p class="hint">${CAM.list.filter(c => c.along > n.along).length} speed or red light cameras on this route</p>` : ''}
    ${S.stop ? `<p class="hint">Stop: <b>${esc(S.stop.name)}</b></p>` : ''}
    ${wxRouteHTML(R)}
    <h3 class="label">Turns ahead</h3><ol class="steps">${items}</ol>`);
}
$('#tripBtn').onclick = renderTrip;
$('#btnNavSearch').onclick = renderAddStop;
mSheet.addEventListener('click', e => {
  const b = e.target.closest('[data-trip]'); if (!b) return;
  if (b.dataset.trip === 'share') shareETA();
  if (b.dataset.trip === 'stop') renderAddStop();
  if (b.dataset.trip === 'hud') hudOpen();
  if (b.dataset.trip === 'end'){ closeModal(); $('#btnEnd').click(); }
  if (b.dataset.trip === 'overview'){ closeModal(); $('#btnOverview').click(); }
});

/* ---------- route options (avoid) ---------- */
function renderRouteOpts(){
  openModal(`<h3 class="label" style="margin-top:0">Route options</h3>
    ${tg('trafficRouting', 'Use live traffic', 'Route around jams with Mapbox, and check for faster routes while you drive')}
    ${tg('avoidTolls', 'Avoid tolls', 'Skip toll roads where there’s another way')}
    ${tg('avoidHighways', 'Avoid highways', 'Stay off motorways and freeways')}
    ${tg('avoidFerries', 'Avoid ferries', 'Never route onto a ferry')}
    ${tg('avoidUnpaved', 'Avoid unpaved roads', settings.trafficRouting ? 'Stay on paved roads' : 'Works only with live-traffic routing')}`);
  mSheet.dataset.view = 'routeopts';
}

/* ---------- weather: NEXRAD radar, NWS storm warnings, conditions ---------- */
map.createPane('radar').style.zIndex = 240; map.getPane('radar').style.pointerEvents = 'none';
map.createPane('warnings').style.zIndex = 395;
const RADAR = { layers:[], labels:[], timer:null, refresh:null, frame:0, sig:null, hold:0 };
function applyRadar(){
  const sig = settings.ovRadar ? (settings.radarAnim ? 'anim' : 'still') : '';
  if (sig === RADAR.sig) return;
  RADAR.sig = sig;
  RADAR.layers.forEach(l => map.removeLayer(l)); RADAR.layers = [];
  clearInterval(RADAR.timer); clearInterval(RADAR.refresh);
  if (sig){
    const frames = sig === 'anim' ? ['-m50m','-m45m','-m40m','-m35m','-m30m','-m25m','-m20m','-m15m','-m10m','-m05m',''] : [''];
    const opt = { pane:'radar', maxZoom:19, ts:() => Math.floor(Date.now() / 300000),
      attribution:'Radar: NWS NEXRAD via <a href="https://mesonet.agron.iastate.edu">Iowa Environmental Mesonet</a>' };
    RADAR.layers = frames.map((f, i) => L.tileLayer(`https://mesonet.agron.iastate.edu/cache/tile.py/1.0.0/nexrad-n0q-900913${f}/{z}/{x}/{y}.png?t={ts}`,
      { ...opt, opacity: i === frames.length - 1 ? .7 : 0, attribution: i ? '' : opt.attribution }).addTo(map));
    RADAR.labels = frames.map(f => f ? '−'+parseInt(f.slice(2), 10)+' min' : 'Now');
    RADAR.frame = frames.length - 1;
    if (sig === 'anim') RADAR.timer = setInterval(() => {
      if (RADAR.frame === RADAR.layers.length - 1 && RADAR.hold++ < 3) return;   // linger on the latest frame
      RADAR.hold = 0;
      RADAR.layers[RADAR.frame].setOpacity(0);
      RADAR.frame = (RADAR.frame + 1) % RADAR.layers.length;
      RADAR.layers[RADAR.frame].setOpacity(.7);
      renderWxChip();
    }, 450);
    RADAR.refresh = setInterval(() => RADAR.layers.forEach(l => l.redraw()), 300000);
  }
  renderWxChip();
}
const WARN_EVENTS = { 'Tornado Warning':'#ff0000', 'Severe Thunderstorm Warning':'#ffa500', 'Flash Flood Warning':'#8b0000',
  'Extreme Wind Warning':'#ff8c00', 'Snow Squall Warning':'#c71585', 'Dust Storm Warning':'#c8a165', 'Special Marine Warning':'#ffa500' };
const WX = { layer:L.layerGroup(), alerts:[], timer:null, told:new Set(), lastCheck:0, cond:null, condAt:0, condPos:null, condBusy:false };
function applyWarnings(){
  if (settings.ovWarnings){
    WX.layer.addTo(map);
    if (!WX.timer){ loadWarnings(); WX.timer = setInterval(loadWarnings, 120000); }
  } else {
    map.removeLayer(WX.layer); clearInterval(WX.timer); WX.timer = null; WX.alerts = []; WX.layer.clearLayers(); renderWxAlert();
  }
}
async function loadWarnings(){
  try{
    const r = await fetch('https://api.weather.gov/alerts/active?status=actual&event='+encodeURIComponent(Object.keys(WARN_EVENTS).join(',')),
      { headers:{ Accept:'application/geo+json' } });
    if (!r.ok) return;
    const j = await r.json();
    WX.alerts = (j.features || []).filter(f => f.geometry && /Polygon/.test(f.geometry.type)).map(f => {
      const polys = (f.geometry.type === 'Polygon' ? [f.geometry.coordinates] : f.geometry.coordinates).map(p => p.map(ring => ring.map(c => [c[1], c[0]])));
      return { id:f.id, p:f.properties, polys, color:WARN_EVENTS[f.properties.event] || '#ff5a00' };
    });
    WX.layer.clearLayers();
    for (const a of WX.alerts){
      L.polygon(a.polys, { pane:'warnings', color:a.color, weight:2.5, opacity:.95, fillColor:a.color, fillOpacity:.13 })
        .bindPopup(() => warnPopup(a), { maxWidth:300 }).addTo(WX.layer);
    }
    checkWx(true);
    if (S.dest && !S.nav && S.routes.length) renderPreview();
  }catch{}
}
const untilTxt = iso => iso ? new Date(iso).toLocaleTimeString([], { hour:'numeric', minute:'2-digit' }) : '';
function warnPopup(a){
  const p = a.p;
  return `<div class="pop"><b style="color:${a.color === '#ffa500' ? '#c77700' : a.color}">${esc(p.event)}</b><small>Until ${untilTxt(p.expires || p.ends)} · ${esc(p.senderName || 'NWS')}</small>
    <p class="wx-head">${esc(p.headline || '')}</p><div class="wx-desc">${esc((p.description || '').trim())}</div>
    ${p.instruction ? `<div class="wx-desc wx-ins">${esc(p.instruction.trim())}</div>` : ''}</div>`;
}
function inRing(pt, ring){
  let inside = false;
  for (let i = 0, j = ring.length - 1; i < ring.length; j = i++){
    const [yi, xi] = ring[i], [yj, xj] = ring[j];
    if ((yi > pt[0]) !== (yj > pt[0]) && pt[1] < (xj - xi) * (pt[0] - yi) / (yj - yi) + xi) inside = !inside;
  }
  return inside;
}
const inAlert = (pt, a) => a.polys.some(poly => inRing(pt, poly[0]) && !poly.slice(1).some(h => inRing(pt, h)));
function alertsOnRoute(R, fromIdx = 0){
  if (!WX.alerts.length || !R) return [];
  const pts = []; for (let i = fromIdx; i < R.coords.length; i += 6) pts.push(R.coords[i]);
  return WX.alerts.filter(a => pts.some(p => inAlert(p, a)));
}
function wxRouteHTML(R){
  const hits = alertsOnRoute(R, S.nav ? snap(R, S.pos || R.coords[0], S.nav.along - 150).i : 0);
  if (!hits.length) return '';
  const names = [...new Set(hits.map(a => a.p.event))];
  return `<div class="wx-route"><i style="background:${hits[0].color}"></i><span><b>Route crosses ${names.length === 1 ? 'a '+esc(names[0]) : esc(names.join(', '))}</b>
    <small>${hits.length} active NWS warning${hits.length === 1 ? '' : 's'} along the way</small></span></div>`;
}
function checkWx(force){
  if (!settings.ovWarnings || !S.pos || (!force && Date.now() - WX.lastCheck < 15000)) return;
  WX.lastCheck = Date.now();
  const here = WX.alerts.filter(a => inAlert(S.pos, a));
  for (const a of here) if (!WX.told.has(a.id)){
    WX.told.add(a.id);
    say(`Weather alert. You are inside a ${a.p.event}, until ${untilTxt(a.p.expires || a.p.ends)}.`);
    toast(`You are inside a ${a.p.event}`, 6000);
  }
  if (S.nav){
    const ahead = alertsOnRoute(S.nav.route, snap(S.nav.route, S.pos, S.nav.along - 150).i).filter(a => !here.includes(a));
    for (const a of ahead) if (!WX.told.has('ahead:'+a.id)){ WX.told.add('ahead:'+a.id); say(`${a.p.event} on your route ahead.`); }
    WX.ahead = ahead;
  } else WX.ahead = [];
  WX.here = here; renderWxAlert();
}
function renderWxAlert(){
  const el = $('#wxAlert'), a = (WX.here || [])[0], b = (WX.ahead || [])[0];
  const pick = a || b;
  if (!pick || !settings.ovWarnings){ el.hidden = true; return; }
  el.style.setProperty('--wc', pick.color);
  el.innerHTML = `<i></i><span><b>${esc(pick.p.event)}</b> ${a ? 'here' : 'on your route'} · until ${untilTxt(pick.p.expires || pick.p.ends)}</span>`;
  el.onclick = () => { S.follow = false; $('#btnRecenter').hidden = !S.pos; northUp(); map.fitBounds(L.polygon(pick.polys).getBounds(), { padding:[40, 40] }); L.popup({ maxWidth:300 }).setLatLng(L.polygon(pick.polys).getBounds().getCenter()).setContent(warnPopup(pick)).openOn(map); };
  el.hidden = false;
}
/* current conditions + forecast at arrival (Open-Meteo, keyless) */
const WMO = c => c === 0 ? ['Clear','sun'] : c <= 2 ? ['Partly cloudy','part'] : c === 3 ? ['Cloudy','cloud'] : c <= 48 ? ['Fog','fog']
  : c <= 57 ? ['Drizzle','rain'] : c <= 67 ? ['Rain','rain'] : c <= 77 ? ['Snow','snow'] : c <= 82 ? ['Showers','rain'] : c <= 86 ? ['Snow showers','snow'] : ['Thunderstorms','storm'];
const WXI = {
  sun:'<svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round"><circle cx="12" cy="12" r="4.5" fill="currentColor"/><path d="M12 2v2.5M12 19.5V22M2 12h2.5M19.5 12H22M4.9 4.9l1.8 1.8M17.3 17.3l1.8 1.8M4.9 19.1l1.8-1.8M17.3 6.7l1.8-1.8"/></svg>',
  part:'<svg viewBox="0 0 24 24" fill="currentColor"><circle cx="9" cy="8" r="4" opacity=".55"/><path d="M8 20a4.5 4.5 0 0 1 .5-9 6 6 0 0 1 11.4 2.2A3.5 3.5 0 0 1 19 20z"/></svg>',
  cloud:'<svg viewBox="0 0 24 24" fill="currentColor"><path d="M6.5 19a4.5 4.5 0 0 1 .5-9 6 6 0 0 1 11.4 2.2A3.5 3.5 0 0 1 17.5 19z"/></svg>',
  fog:'<svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2" stroke-linecap="round"><path d="M4 9h16M3 13h18M5 17h14"/></svg>',
  rain:'<svg viewBox="0 0 24 24" fill="currentColor"><path d="M6.5 14a4.5 4.5 0 0 1 .5-9 6 6 0 0 1 11.4 2.2A3.5 3.5 0 0 1 17.5 14z"/><path d="M8 17l-1 3M12 17l-1 3M16 17l-1 3" stroke="currentColor" stroke-width="2" stroke-linecap="round"/></svg>',
  snow:'<svg viewBox="0 0 24 24" fill="currentColor"><path d="M6.5 14a4.5 4.5 0 0 1 .5-9 6 6 0 0 1 11.4 2.2A3.5 3.5 0 0 1 17.5 14z"/><circle cx="8" cy="18" r="1.3"/><circle cx="12" cy="20" r="1.3"/><circle cx="16" cy="18" r="1.3"/></svg>',
  storm:'<svg viewBox="0 0 24 24" fill="currentColor"><path d="M6.5 14a4.5 4.5 0 0 1 .5-9 6 6 0 0 1 11.4 2.2A3.5 3.5 0 0 1 17.5 14z"/><path d="M12.5 13L9 18.5h3l-1 4 4-6h-3z"/></svg>'
};
const tempU = () => imperial() ? 'fahrenheit' : 'celsius';
async function refreshConditions(force){
  const p = S.pos || [map.getCenter().lat, map.getCenter().lng];
  if (WX.condBusy || (!force && WX.condPos && dist(WX.condPos, p) < 8000 && Date.now() - WX.condAt < 600000)) return;
  WX.condBusy = true;
  try{
    const r = await fetch(`https://api.open-meteo.com/v1/forecast?latitude=${p[0].toFixed(3)}&longitude=${p[1].toFixed(3)}&current=temperature_2m,weather_code,wind_speed_10m,is_day&temperature_unit=${tempU()}&wind_speed_unit=${imperial() ? 'mph' : 'kmh'}`);
    const j = await r.json();
    if (j.current){ WX.cond = j.current; WX.condAt = Date.now(); WX.condPos = p; renderWxChip(); }
  }catch{}
  WX.condBusy = false;
}
function renderWxChip(){
  const el = $('#wx'), c = WX.cond;
  const radar = RADAR.sig === 'anim' ? `<em>${RADAR.labels[RADAR.frame] || ''}</em>` : '';
  if (!c){ el.hidden = !radar; el.innerHTML = radar ? `<span>Radar</span>${radar}` : ''; return; }
  const [label, icon] = WMO(c.weather_code);
  el.innerHTML = `${WXI[icon === 'sun' && !c.is_day ? 'cloud' : icon]}<b>${Math.round(c.temperature_2m)}°</b>${radar}`;
  el.title = `${label}, wind ${Math.round(c.wind_speed_10m)} ${imperial() ? 'mph' : 'km/h'}`;
  el.hidden = false;
}
$('#wx').onclick = () => { const c = WX.cond; if (c) toast(`${WMO(c.weather_code)[0]} · ${Math.round(c.temperature_2m)}° · wind ${Math.round(c.wind_speed_10m)} ${imperial() ? 'mph' : 'km/h'}`); };
async function destWeather(d, secs){
  try{
    const r = await fetch(`https://api.open-meteo.com/v1/forecast?latitude=${d.lat.toFixed(3)}&longitude=${d.lng.toFixed(3)}&hourly=temperature_2m,weather_code,precipitation_probability&forecast_hours=48&timeformat=unixtime&temperature_unit=${tempU()}`);
    const j = await r.json(), h = j.hourly; if (!h?.time?.length) return null;
    const t = Date.now() / 1000 + (secs || 0);
    let k = 0; for (let i = 0; i < h.time.length; i++) if (Math.abs(h.time[i] - t) < Math.abs(h.time[k] - t)) k = i;
    const [label, icon] = WMO(h.weather_code[k]);
    return { label, icon, temp:Math.round(h.temperature_2m[k]), pop:h.precipitation_probability?.[k] };
  }catch{ return null; }
}

/* ---------- parked car ---------- */
const parkIcon = L.divIcon({ className:'park', iconSize:[32,32], iconAnchor:[16,16], html:'<div>P</div>' });
let parkMarker = null;
function drawPark(){
  const p = store.get('parked');
  if (parkMarker){ map.removeLayer(parkMarker); parkMarker = null; }
  if (!p) return;
  parkMarker = L.marker([p.lat, p.lng], { icon:parkIcon, zIndexOffset:500 }).addTo(map)
    .bindPopup(() => { const m = Math.round((Date.now() - p.t) / 60000);
      return `<div class="pop"><b>Your parked car</b><small>Parked ${m < 1 ? 'just now' : m < 60 ? m+' min ago' : fmtDur(m * 60)+' ago'}</small><br>
        <button class="go" data-park="go">Directions</button> &nbsp; <button data-park="clear">Clear</button></div>`; });
}
function askPark(){
  const ll = S.pos || [map.getCenter().lat, map.getCenter().lng];
  closeModal();
  startPlace({ ll, icon:parkIcon, color:'#2557c9', badge:'<b style="font:800 18px var(--font)">P</b>', title:'Save parking spot here?', okLabel:'Save spot',
    onConfirm: p => { store.set('parked', { lat:p.lat, lng:p.lng, t:Date.now() }); drawPark(); toast('Parking spot saved'); } });
}
map.on('popupopen', e => {
  const el = e.popup.getElement(); if (!el) return;
  el.querySelectorAll('[data-park]').forEach(b => b.onclick = () => {
    const p = store.get('parked'); map.closePopup(); if (!p) return;
    if (b.dataset.park === 'go') previewPlace({ name:'Parked car', addr:'Your saved parking spot', lat:p.lat, lng:p.lng });
    else { store.set('parked', null); drawPark(); toast('Parking spot cleared'); }
  });
});

/* ---------- voice search ---------- */
const SR = window.SpeechRecognition || window.webkitSpeechRecognition;
if (SR){
  const mic = $('#micBtn'); mic.hidden = false;
  let rec = null;
  mic.onclick = () => {
    if (rec){ rec.stop(); return; }
    try{
      rec = new SR(); rec.lang = navigator.language || 'en-US'; rec.interimResults = true; rec.maxAlternatives = 1;
      mic.classList.add('listening'); q.placeholder = 'Listening…';
      rec.onresult = e => { const t = Array.from(e.results).map(r => r[0].transcript).join(''); q.value = t; $('#clearQ').hidden = !t;
        if (e.results[e.results.length - 1].isFinal && t.trim()) search(t.trim()); };
      rec.onerror = e => { if (e.error === 'not-allowed') toast('Microphone access is blocked for this site.'); };
      rec.onend = () => { mic.classList.remove('listening'); q.placeholder = 'Where to?'; rec = null; };
      rec.start();
    }catch{ rec = null; mic.classList.remove('listening'); toast('Voice search isn’t available in this browser.'); }
  };
}

/* ---------- add a stop on the way ---------- */
const STOP_CATS = { fuel:CATS.fuel, food:CATS.food,
  cafe:{ label:'Coffee', color:'#7b4a2e', q:'nwr[amenity=cafe]', svg:'<svg viewBox="0 0 24 24" fill="currentColor"><path d="M4 8h13v5a6 6 0 0 1-6 6H10a6 6 0 0 1-6-6zm13 1h1.5a2.5 2.5 0 0 1 0 5H17v-2h1.5a.5.5 0 0 0 0-1H17zM7 3c0 1.5 1.5 1.5 1.5 3M11 3c0 1.5 1.5 1.5 1.5 3" stroke="currentColor" stroke-width="1.4" fill-rule="evenodd"/></svg>' },
  ev:CATS.ev, parking:CATS.parking,
  rest:{ label:'Rest area', color:'#4a6b8a', q:'nwr[highway~"^(rest_area|services)$"]', svg:'<svg viewBox="0 0 24 24" fill="currentColor"><path d="M4 10h16v3H4zM6 13h2v7H6zM16 13h2v7h-2zM12 3l7 5H5z"/></svg>' } };
let stopMarker = null;
function drawStop(){
  if (stopMarker){ map.removeLayer(stopMarker); stopMarker = null; }
  if (S.stop) stopMarker = L.marker([S.stop.lat, S.stop.lng], { interactive:false, zIndexOffset:600,
    icon:L.divIcon({ className:'stop-pin', iconSize:[30,30], iconAnchor:[15,15], html:'<div>1</div>' }) }).addTo(map);
}
function renderAddStop(){
  openModal(`<h3 class="label" style="margin-top:0">Add a stop on the way</h3>
    <div class="cats wrap">${Object.entries(STOP_CATS).map(([k, C]) => `<button class="cat" data-stopcat="${k}"><i style="background:${C.color}">${C.svg}</i>${C.label}</button>`).join('')}</div>
    <div id="stopList"><p class="hint">Pick a category to find places close to your route.</p></div>
    ${S.stop ? `<button class="link" data-stop="remove">Remove stop: ${esc(S.stop.name)}</button>` : ''}`);
  mSheet.dataset.view = 'stops';
}
async function findStops(k){
  const C = STOP_CATS[k], R = S.nav ? S.nav.route : S.routes[S.routeIdx]; if (!C || !R) return;
  mSheet.querySelectorAll('[data-stopcat]').forEach(b => b.classList.toggle('on', b.dataset.stopcat === k));
  const here = S.nav ? S.nav.along : 0, from = S.nav ? snap(R, S.pos || R.coords[0], here - 150).i : 0, pts = [];
  for (let i = from; i < R.coords.length && R.cum[i] - R.cum[from] < 60000; i++)
    if (!pts.length || dist(pts[pts.length - 1], R.coords[i]) > 1500) pts.push(R.coords[i]);
  pts.push(R.coords[Math.min(R.coords.length - 1, from + 1)]);
  const box = $('#stopList'); box.innerHTML = '<p class="hint">Searching along your route…</p>';
  const ql = `[out:json][timeout:25];${C.q}(around:1200,${pts.map(p => p[0].toFixed(5)+','+p[1].toFixed(5)).join(',')});out center tags 80;`;
  try{
    const r = await opFetch({ method:'POST', body:'data='+encodeURIComponent(ql), headers:{ 'Content-Type':'application/x-www-form-urlencoded' } });
    if (!r.ok) throw new Error();
    const j = await r.json();
    S.stopList = (j.elements || []).map(e => {
      const t = e.tags || {}, lat = e.lat ?? e.center?.lat, lng = e.lon ?? e.center?.lon;
      if (lat == null) return null;
      const sn = snap(R, [lat, lng], here);
      return { name:t.name || t.brand || C.label, lat, lng, addr:[[t['addr:housenumber'], t['addr:street']].filter(Boolean).join(' '), t['addr:city']].filter(Boolean).join(', ') || C.label, off:sn.d, ahead:sn.along - here };
    }).filter(p => p && p.ahead > 100).sort((a, b) => (a.ahead + a.off * 4) - (b.ahead + b.off * 4)).slice(0, 15);
    if (!$('#stopList')) return;
    $('#stopList').innerHTML = S.stopList.length
      ? `<ul class="list">${S.stopList.map((p, i) => `<li><button class="row" data-stopi="${i}"><span class="row-ico" style="color:${C.color}">${C.svg}</span>
          <span class="row-txt"><b>${esc(p.name)}</b><small>${fd(p.off)} off route · ${fd(p.ahead)} ahead</small></span></button></li>`).join('')}</ul>`
      : '<p class="hint">Nothing close to your route in the next 37 miles.</p>';
  }catch{ if ($('#stopList')) $('#stopList').innerHTML = '<p class="err">Couldn’t search along the route. Try again in a moment.</p>'; }
}
async function rerouteWithStop(){
  if (S.nav) await reroute(true);
  else if (S.dest) previewPlace(S.dest);
}
mSheet.addEventListener('click', e => {
  const b = e.target.closest('[data-stopcat],[data-stopi],[data-stop]'); if (!b) return;
  if (b.dataset.stopcat) return findStops(b.dataset.stopcat);
  if (b.dataset.stopi){ const p = S.stopList[+b.dataset.stopi]; if (!p) return; S.stop = p; drawStop(); closeModal(); toast('Stop added: '+p.name); rerouteWithStop(); return; }
  if (b.dataset.stop === 'remove'){ S.stop = null; drawStop(); closeModal(); toast('Stop removed'); rerouteWithStop(); }
});

/* ---------- speeding chime (WebAudio, unlocked on first tap) ---------- */
let audioCtx = null;
function unlockAudio(){ try{ audioCtx = audioCtx || new (window.AudioContext || window.webkitAudioContext)(); if (audioCtx.state === 'suspended') audioCtx.resume(); }catch{} }
document.addEventListener('pointerdown', unlockAudio, { capture:true });
function chime(){
  if (!audioCtx || !settings.speedAlert) return;
  const t = audioCtx.currentTime;
  [0, .17].forEach((o, i) => {
    const os = audioCtx.createOscillator(), g = audioCtx.createGain();
    os.type = 'sine'; os.frequency.value = i ? 1320 : 990;
    g.gain.setValueAtTime(.0001, t + o); g.gain.exponentialRampToValueAtTime(.22, t + o + .02); g.gain.exponentialRampToValueAtTime(.0001, t + o + .15);
    os.connect(g).connect(audioCtx.destination); os.start(t + o); os.stop(t + o + .17);
  });
}

/* ---------- sunset-based day/night ---------- */
function sunAlt(lat, lng, when = Date.now()){
  const d = when / 86400000 - 10957.5;                               // days since J2000.0
  const g = rad((357.529 + 0.98560028 * d) % 360), q = (280.459 + 0.98564736 * d) % 360;
  const Lm = rad(q + 1.915 * Math.sin(g) + 0.020 * Math.sin(2 * g)), e = rad(23.439 - 0.00000036 * d);
  const dec = Math.asin(Math.sin(e) * Math.sin(Lm)), ra = Math.atan2(Math.cos(e) * Math.sin(Lm), Math.cos(Lm));
  const ha = rad(((18.697374558 + 24.06570982441908 * d) % 24) * 15 + lng) - ra;
  return deg(Math.asin(Math.sin(rad(lat)) * Math.sin(dec) + Math.cos(rad(lat)) * Math.cos(dec) * Math.cos(ha)));
}
let sunDark = null;
setInterval(() => { if (settings.theme === 'sun' && isDark() !== sunDark) applyTheme(); }, 60000);

/* ---------- route time bubbles in the preview ---------- */
function bubblePoint(r, others){
  const n = r.coords.length;
  if (!others.length) return r.coords[Math.floor(n * .5)];
  const step = Math.max(1, Math.floor(n / 60));
  let best = r.coords[Math.floor(n / 2)], bestD = -1;
  for (let i = Math.floor(n * .2); i < Math.floor(n * .8); i += step){
    let md = Infinity;
    for (const o of others){ const os = Math.max(1, Math.floor(o.coords.length / 80)); for (let j = 0; j < o.coords.length; j += os){ const dd = dist(r.coords[i], o.coords[j]); if (dd < md) md = dd; } }
    if (md > bestD){ bestD = md; best = r.coords[i]; }
  }
  return best;
}
function drawBubbles(){
  S.routes.forEach((r, i) => {
    const on = i === S.routeIdx;
    L.marker(bubblePoint(r, S.routes.filter(o => o !== r)), { zIndexOffset: on ? 900 : 800, keyboard:false,
      icon:L.divIcon({ className:'rt-bub-wrap', iconSize:[0,0], html:`<div class="rt-bub${on ? ' on' : ''}"><b>${fmtDur(r.duration)}</b>${i === 0 ? '<span>Best</span>' : ''}</div>` }) })
      .on('click', () => { S.routeIdx = i; drawRoutes(); renderPreview(); }).addTo(routeLayer);
  });
}

/* ---------- trip details in the preview: weather along the route + elevation profile ---------- */
const hazLabel = w => [56,57,66,67].includes(w.code) ? 'Freezing rain' : w.label;
async function routeWeather(R){
  const n = R.total > 20000 ? 6 : 2, pts = [];
  for (let i = 0; i < n; i++){ const f = i / (n - 1); pts.push({ ll:pointAt(R, f * R.total).pt, secs:f * R.duration, along:f * R.total }); }
  const r = await fetch(`https://api.open-meteo.com/v1/forecast?latitude=${pts.map(p => p.ll[0].toFixed(3)).join(',')}&longitude=${pts.map(p => p.ll[1].toFixed(3)).join(',')}`
    + `&hourly=temperature_2m,weather_code,precipitation_probability,wind_gusts_10m&forecast_hours=48&timeformat=unixtime&temperature_unit=${tempU()}&wind_speed_unit=${imperial() ? 'mph' : 'kmh'}`);
  let j = await r.json(); if (!Array.isArray(j)) j = [j];
  return pts.map((p, i) => {
    const h = j[i]?.hourly; if (!h?.time?.length) return null;
    const t = Date.now() / 1000 + p.secs; let k = 0;
    for (let x = 0; x < h.time.length; x++) if (Math.abs(h.time[x] - t) < Math.abs(h.time[k] - t)) k = x;
    const [label, icon] = WMO(h.weather_code[k]);
    return { ...p, label, icon, code:h.weather_code[k], temp:Math.round(h.temperature_2m[k]), pop:h.precipitation_probability?.[k] ?? null, gust:Math.round(h.wind_gusts_10m?.[k] ?? 0) };
  }).filter(Boolean);
}
function wxStripHTML(wx){
  if (!wx?.length) return '';
  const gl = imperial() ? 40 : 65, wu = imperial() ? 'mph' : 'km/h';
  const haz = wx.find(w => w.code >= 95 || (w.code >= 71 && w.code <= 77) || w.code === 85 || w.code === 86 || [56,57,66,67].includes(w.code) || w.gust >= gl);
  return `<h3 class="label">Weather on the way</h3>
    ${haz ? `<p class="ws-haz"><i></i>${esc(hazLabel(haz))}${haz.gust >= gl ? (haz.code >= 51 ? ' with ' : '') + 'gusts to '+haz.gust+' '+wu : ''} ${haz.along < 500 ? 'where you start' : 'about '+fd(haz.along)+' in'}</p>` : ''}
    <div class="ws">${wx.map((w, i) => `<div class="ws-c" title="${esc(w.label)}"><small>${i === 0 ? 'Start' : i === wx.length - 1 ? 'Arrive' : fd(w.along)}</small>
      <span class="ws-i">${WXI[w.icon]}</span><b>${w.temp}°</b><small class="ws-p">${w.pop != null && w.pop >= 20 ? w.pop+'%' : '&nbsp;'}</small></div>`).join('')}</div>`;
}
async function routeElevation(R){
  if (R.total < 2000) return null;
  const N = Math.min(100, Math.max(24, Math.round(R.total / 400))), pts = [];
  for (let i = 0; i < N; i++) pts.push(pointAt(R, i / (N - 1) * R.total).pt);
  const r = await fetch(`https://api.open-meteo.com/v1/elevation?latitude=${pts.map(p => p[0].toFixed(4)).join(',')}&longitude=${pts.map(p => p[1].toFixed(4)).join(',')}`);
  const j = await r.json(), e = j.elevation; if (!e?.length) return null;
  let climb = 0, desc = 0;
  for (let i = 1; i < e.length; i++){ const d = e[i] - e[i-1]; if (d > 0) climb += d; else desc -= d; }
  return { e, climb, desc, min:Math.min(...e), max:Math.max(...e) };
}
function elevHTML(x){
  if (!x) return '';
  const u = v => Math.round(imperial() ? v * 3.28084 : v).toLocaleString() + (imperial() ? ' ft' : ' m');
  const W = 300, H = 56, span = Math.max(15, x.max - x.min), n = x.e.length;
  const pts = x.e.map((v, i) => `${(i / (n - 1) * W).toFixed(1)},${(H - 4 - (v - x.min) / span * (H - 10)).toFixed(1)}`);
  return `<h3 class="label">Elevation</h3>
    <svg class="elev" viewBox="0 0 ${W} ${H}" preserveAspectRatio="none" role="img" aria-label="Elevation profile">
      <path d="M0,${H} L${pts.join(' L')} L${W},${H} Z" class="elev-a"/><path d="M${pts.join(' L')}" class="elev-l"/></svg>
    <p class="elev-t"><span>Climb <b>${u(x.climb)}</b></span><span>Descent <b>${u(x.desc)}</b></span><span>Highest <b>${u(x.max)}</b></span></p>`;
}
function fillTripDetails(R){
  const box = $('#pvTrip'); if (!box || !R) return;
  R._id = R._id || Math.random().toString(36).slice(2);
  box.dataset.rid = R._id;
  if (!R._wx) R._wx = routeWeather(R).catch(() => null);
  if (!R._elev) R._elev = routeElevation(R).catch(() => null);
  Promise.all([R._wx, R._elev]).then(([wx, el]) => {
    const b = $('#pvTrip'); if (!b || b.dataset.rid !== R._id) return;
    b.innerHTML = wxStripHTML(wx) + elevHTML(el);
  });
}

/* ---------- fuel cost ---------- */
function fuelCost(m){
  const eco = +(imperial() ? settings.mpg : settings.l100), price = +(imperial() ? settings.priceGal : settings.priceL);
  if (!eco || !price) return '';
  const c = imperial() ? (m / 1609.344) / eco * price : (m / 1000) * eco / 100 * price;
  return (settings.currency || '$') + c.toFixed(2);
}
function routeExtra(r){
  const bits = [];
  if (r.engine === 'tomtom' || r.engine === 'mapbox') bits.push(`<span class="delay ${r.delay > 60 ? 'bad' : 'ok'}">${r.delay > 60 ? '+'+fmtDur(r.delay)+' traffic' : 'Light traffic'}</span>`);
  const fc = fuelCost(r.distance); if (fc) bits.push(`<span>~${fc} fuel</span>`);
  return bits.length ? `<small class="rt-extra">${bits.join(' · ')}</small>` : '';
}
mSheet.addEventListener('change', e => {
  const k = e.target.dataset?.num; if (!k) return;
  const v = parseFloat(String(e.target.value).replace(',', '.'));
  settings[k] = isFinite(v) && v > 0 ? v : 0; saveSettings();
  if (S.dest && !S.nav && S.routes.length) renderPreview();
  toast('Saved', 1000);
});

/* ---------- saved places ---------- */
const favs = () => store.get('favs', []) || [];
const isFav = p => favs().some(f => Math.abs(f.lat - p.lat) < 1e-5 && Math.abs(f.lng - p.lng) < 1e-5);
function toggleFav(p){
  let f = favs();
  if (isFav(p)){ f = f.filter(x => !(Math.abs(x.lat - p.lat) < 1e-5 && Math.abs(x.lng - p.lng) < 1e-5)); toast('Removed from Saved'); }
  else { f.unshift({ name:p.name, addr:p.addr, lat:p.lat, lng:p.lng }); toast('Saved'); }
  store.set('favs', f.slice(0, 30));
}

/* ---------- speed cameras along the route (OpenStreetMap) ---------- */
const CAM = { list:[], layer:L.layerGroup().addTo(map), told:new Set(), route:null };
async function loadCameras(R){
  CAM.list = []; CAM.layer.clearLayers(); CAM.told.clear();
  const pts = [];
  for (let i = 0; i < R.coords.length && R.cum[i] < 250000; i++) if (!pts.length || dist(pts[pts.length - 1], R.coords[i]) > 400) pts.push(R.coords[i]);
  pts.push(R.coords[R.coords.length - 1]);
  const line = pts.map(p => p[0].toFixed(5)+','+p[1].toFixed(5)).join(',');
  if (settings.showSignals) loadSignals(R, line);
  try{
    const r = await opFetch({ method:'POST', headers:{ 'Content-Type':'application/x-www-form-urlencoded' },
      body:'data='+encodeURIComponent(`[out:json][timeout:25];(node[highway=speed_camera](around:70,${line});node[enforcement~"^(maxspeed|traffic_signals)$"](around:70,${line}););out tags 400;`) });
    if (!r.ok || CAM.route !== R || !settings.camAlerts) return;
    const j = await r.json();
    for (const e of j.elements || []){
      const sn = snap(R, [e.lat, e.lon], 0); if (sn.d > 70) continue;
      const red = e.tags?.enforcement === 'traffic_signals';
      CAM.list.push({ id:e.id, along:sn.along, red, max:parseMaxspeed(e.tags?.maxspeed) });
      L.marker([e.lat, e.lon], { interactive:false, icon:L.divIcon({ className:'rep cam-pin', iconSize:[24,24], iconAnchor:[12,12],
        html:`<div style="background:${red ? '#c8312b' : RT.camera.color}">${RT.camera.svg}</div>` }) }).addTo(CAM.layer);
    }
  }catch{}
}
function checkCameras(along){
  if (!settings.camAlerts) return;
  const warnAt = Math.max(300, S.speed * 15);
  for (const c of CAM.list){
    const ahead = c.along - along;
    if (ahead <= 0 || ahead > warnAt || CAM.told.has(c.id)) continue;
    CAM.told.add(c.id);
    const lim = c.max ? (imperial() ? Math.round(c.max / 1.609344 / 5) * 5 + ' miles per hour' : Math.round(c.max) + ' kilometers per hour') : '';
    say((c.red ? 'Red light camera ahead' : 'Speed camera ahead') + (lim ? ', limit ' + lim : ''));
    toast((c.red ? 'Red light camera' : 'Speed camera') + ' in ' + fd(ahead), 4000);
  }
}

/* ---------- trip history ---------- */
function logTrip(n){
  if (!n || !(n.driven > 300)) return;
  const trips = store.get('trips', []) || [];
  trips.unshift({ name:S.dest?.name || 'Trip', t0:n.t0, t1:Date.now(), dist:n.driven, arrived:!!n.arrived });
  store.set('trips', trips.slice(0, 60));
}
function renderTrips(){
  const trips = store.get('trips', []) || [], wk = Date.now() - 7 * 864e5;
  const week = trips.filter(t => t.t0 > wk), wd = week.reduce((a, t) => a + t.dist, 0), wt = week.reduce((a, t) => a + (t.t1 - t.t0) / 1000, 0);
  const day = t => new Date(t).toLocaleDateString([], { weekday:'short', month:'short', day:'numeric' });
  openModal(`<h3 class="label" style="margin-top:0">Last 7 days</h3>
    <div class="stats"><div><b>${week.length}</b><span>Trips</span></div><div><b>${fd(wd)}</b><span>Driven</span></div><div><b>${fmtDur(wt)}</b><span>Drive time</span></div></div>
    <h3 class="label">Trip history</h3>
    ${trips.length ? `<ul class="list">${trips.slice(0, 30).map(t => `<li><div class="row"><span class="row-ico">${ico.clock}</span>
      <span class="row-txt"><b>${esc(t.name)}</b><small>${day(t.t0)} · ${new Date(t.t0).toLocaleTimeString([], { hour:'numeric', minute:'2-digit' })} · ${fmtDur((t.t1 - t.t0) / 1000)}${t.arrived ? '' : ' · ended early'}</small></span>
      <span class="row-d">${fd(t.dist)}</span></div></li>`).join('')}</ul>
      <button class="link" data-trips="clear">Clear history</button>` : '<p class="hint">Trips you navigate show up here.</p>'}`);
}
mSheet.addEventListener('click', e => {
  const b = e.target.closest('[data-trips]'); if (!b) return;
  store.set('trips', []); renderTrips(); toast('Trip history cleared');
});

/* ---------- windshield HUD ---------- */
function hudOpen(){
  closeModal(); $('#hudv').hidden = false;
  try{ document.documentElement.requestFullscreen?.().catch(() => {}); }catch{}
  if (S.nav && S.pos) navTick(); updateSpeedo();
}
function hudClose(){ $('#hudv').hidden = true; try{ if (document.fullscreenElement) document.exitFullscreen(); }catch{} }
$('#hudv').addEventListener('click', e => {
  const b = e.target.closest('[data-h]'); if (!b) return;
  if (b.dataset.h === 'mirror'){ settings.hudMirror = !settings.hudMirror; saveSettings(); $('#hudv .h-in').classList.toggle('mirror', settings.hudMirror); }
  else hudClose();
});

/* ---------- driving camera (Mapbox): heading-up bearing, 3D pitch, car low on screen ---------- */
let is3D = false;
function want3D(){ return !!(settings.view3d && S.nav); }
function apply3D(){ is3D = want3D() && S.follow; if (S.pos && S.follow) followCam(600); }
function layoutView(){ apply3D(); }

/* ---------- alternate routes while driving ---------- */
const altLayer = L.layerGroup().addTo(map);
async function showAlts(){
  const n = S.nav; if (!n || !S.pos) return;
  openModal('<h3 class="label" style="margin-top:0">Routes from here</h3><p class="hint">Finding routes…</p>');
  mSheet.dataset.view = 'alts';
  try{
    const rs = await fetchRoutes(S.pos, S.dest.ll, true);
    if (S.nav !== n || mSheet.dataset.view !== 'alts') return;
    S.altRoutes = rs; altLayer.clearLayers();
    rs.forEach(r => { L.polyline(r.coords, { color:css('--casing'), weight:9, opacity:.8, interactive:false }).addTo(altLayer);
      L.polyline(r.coords, { color:css('--route-alt'), weight:6, interactive:false }).addTo(altLayer); });
    const cur = n.remT ?? n.route.duration;
    mSheet.innerHTML = `<div class="grab"></div><h3 class="label" style="margin-top:0">Routes from here</h3>
      <ul class="routes"><li><div class="route-opt on"><b>${fmtDur(cur)}</b><span>Current route</span><em>Now</em></div></li>
      ${rs.map((r, i) => { const d = Math.round((r.duration - cur) / 60);
        return `<li><button class="route-opt" data-alt="${i}"><b>${fmtDur(r.duration)}</b><span>${fd(r.distance)} · via ${esc(r.via)}</span>
          <em class="${d < 0 ? 'faster' : ''}">${d === 0 ? 'Same' : d < 0 ? d+' min' : '+'+d+' min'}</em>${routeExtra(r)}</button></li>`; }).join('')}</ul>
      <p class="fine">Pick a route to switch to it. Gray lines on the map show the options.</p>`;
  }catch(e){ if (mSheet.dataset.view === 'alts') mSheet.innerHTML = `<div class="grab"></div><p class="err">${esc(e.message === 'tomtom' ? 'Couldn’t find other routes right now.' : e.message)}</p>`; }
}
mSheet.addEventListener('click', e => {
  const b = e.target.closest('[data-alt]'); if (!b) return;
  const r = S.altRoutes?.[+b.dataset.alt], n = S.nav; if (!r || !n) return;
  n.route = r; n.spoken = {}; n.off = 0; n.along = 0; if (S.sim) S.sim.along = 0;
  drawNavRoute(); startFasterChecks(); closeModal(); toast('Route changed'); say('Route changed');
});
$('#btnAlts').onclick = showAlts;

/* ---------- parking near the destination ---------- */
async function showDestParking(){
  const d = S.dest; if (!d) return;
  openModal(`<h3 class="label" style="margin-top:0">Parking near ${esc(d.name)}</h3><p class="hint">Looking for parking…</p>`);
  mSheet.dataset.view = 'dpark';
  const ql = `[out:json][timeout:20];nwr[amenity=parking][access!~"^(private|no|customers)$"](around:800,${d.lat.toFixed(5)},${d.lng.toFixed(5)});out center tags 40;`;
  try{
    const r = await opFetch({ method:'POST', body:'data='+encodeURIComponent(ql), headers:{ 'Content-Type':'application/x-www-form-urlencoded' } });
    if (!r.ok) throw new Error();
    const j = await r.json();
    S.dParks = (j.elements || []).map(e => {
      const t = e.tags || {}, lat = e.lat ?? e.center?.lat, lng = e.lon ?? e.center?.lon; if (lat == null) return null;
      const bits = [t.parking === 'multi-storey' ? 'Garage' : t.parking === 'underground' ? 'Underground' : t.parking === 'street_side' ? 'Street' : 'Lot',
        t.fee === 'no' ? 'Free' : t.fee === 'yes' ? 'Paid' : '', t.capacity ? t.capacity+' spaces' : ''].filter(Boolean).join(' · ');
      return { name:t.name || 'Parking', lat, lng, info:bits, walk:dist([lat, lng], [d.lat, d.lng]) };
    }).filter(Boolean).sort((a, b) => a.walk - b.walk).slice(0, 12);
    if (mSheet.dataset.view !== 'dpark') return;
    mSheet.innerHTML = `<div class="grab"></div><h3 class="label" style="margin-top:0">Parking near ${esc(d.name)}</h3>` + (S.dParks.length
      ? `<ul class="list">${S.dParks.map((p, i) => `<li><button class="row" data-dpark="${i}"><span class="park-badge">P</span>
          <span class="row-txt"><b>${esc(p.name)}</b><small>${esc(p.info)}</small></span><span class="row-d">${Math.max(1, Math.round(p.walk / 80))} min walk</span></button></li>`).join('')}</ul>`
      : '<p class="hint">No public parking mapped within half a mile.</p>');
  }catch{ if (mSheet.dataset.view === 'dpark') mSheet.innerHTML = '<div class="grab"></div><p class="err">Couldn’t search for parking. Try again in a moment.</p>'; }
}
mSheet.addEventListener('click', e => {
  const b = e.target.closest('[data-dpark]'); if (!b) return;
  const p = S.dParks?.[+b.dataset.dpark]; if (!p || !S.dest) return;
  const orig = S.dest.name;
  S.dest = { name:p.name === 'Parking' ? 'Parking near '+orig : p.name, addr:'Parking near '+orig, lat:p.lat, lng:p.lng, ll:[p.lat, p.lng], isParking:true };
  destMarker?.setLatLng(S.dest.ll);
  closeModal(); toast('Heading to parking'); reroute(true);
});
$('#bPark').onclick = showDestParking;

/* ---------- road shields (I-75, US 42, KY 18 …) ---------- */
function shieldSVG(ref){
  const r = String(ref).trim().replace(/\s+/g, ' ');
  let m;
  if ((m = r.match(/^(?:I|IH|Interstate)[- ]?(\d{1,3}[A-Z]?)$/i))){
    const n = m[1], w = n.length > 2 ? 34 : 28;
    return `<svg class="shield" viewBox="0 0 ${w} 28" width="${w}" height="28"><path d="M2 3c4 1 8 0 ${w/2-2}-2 ${w/2-6} 2 ${w/2-2} 3 ${w/2-2} 2 0 9-2 17-${w/2-2} 23C4 20 2 12 2 3z" fill="#1d4fa8" stroke="#fff" stroke-width="1.5"/><path d="M2.6 3.4c4 1 8 0 ${w/2-2.6}-1.9 ${w/2-6} 1.9 ${w/2-2.6} 2.9 ${w/2-2.6} 1.9L${w-2.3} 9H2.6z" fill="#c8312b"/><text x="${w/2}" y="20.5" text-anchor="middle" font-family="Overpass,system-ui,sans-serif" font-weight="800" font-size="${n.length > 2 ? 10 : 11.5}" fill="#fff">${esc(n)}</text></svg>`;
  }
  if ((m = r.match(/^US[- ]?(\d{1,3}[A-Z]?)$/i))){
    const n = m[1], w = n.length > 2 ? 32 : 26;
    return `<svg class="shield" viewBox="0 0 ${w} 28" width="${w}" height="28"><path d="M3 2h${w-6}c0 3 2 4 2 7 0 6-2 8-2 11 0 3-4 5-${w/2-3} 6C${w/2-4} 27 3 25 3 20c0-3-2-5-2-11 0-3 2-4 2-7z" fill="#fff" stroke="#111" stroke-width="1.6"/><text x="${w/2}" y="18.5" text-anchor="middle" font-family="Overpass,system-ui,sans-serif" font-weight="800" font-size="${n.length > 2 ? 10.5 : 12.5}" fill="#111">${esc(n)}</text></svg>`;
  }
  if ((m = r.match(/^([A-Z]{1,3})[- ]?(\d{1,4}[A-Z]?)$/))){
    const n = m[2], w = Math.max(26, 9 + n.length * 7.5);
    return `<svg class="shield" viewBox="0 0 ${w} 22" width="${w}" height="22"><rect x="1" y="1" width="${w-2}" height="20" rx="10" fill="#fff" stroke="#111" stroke-width="1.6"/><text x="${w/2}" y="15.5" text-anchor="middle" font-family="Overpass,system-ui,sans-serif" font-weight="800" font-size="12" fill="#111">${esc(n)}</text></svg>`;
  }
  return `<span class="shield-txt">${esc(r)}</span>`;
}
const shieldsHTML = ref => String(ref || '').split(';').map(x => x.trim()).filter(Boolean).slice(0, 2).map(shieldSVG).join('');
// What the card shows for a maneuver: the street (or its shields), or a plain instruction when there's no name.
function stepLabelHTML(step){
  const t = step.maneuver.type, mod = step.maneuver.modifier || 'straight';
  if (t === 'arrive') return step.waypoint ? 'Your stop' : esc(S.dest?.name || 'Destination');
  if ((t === 'continue' || t === 'new name' || t === 'notification') && mod === 'straight') return 'Continue straight';
  if (t === 'roundabout' || t === 'rotary') return `${ord(step.maneuver.exit || 1)} exit${step.name ? ' · '+esc(step.name) : ''}`;
  if (step.name) return esc(step.name);
  if (step.ref) return `<span class="shields">${shieldsHTML(step.ref)}</span>`;
  return esc(instr(step));
}
function roadNowHTML(){
  const n = S.nav; if (!n) return '';
  const st = n.route.steps[n.k || 0]; if (!st) return '';
  if (st.name) return esc(st.name);
  if (st.ref) return `<span class="shields">${shieldsHTML(st.ref)}</span>`;
  return '';
}

/* ---------- traffic signals along the route ---------- */
const SIG = { layer:L.layerGroup().addTo(map) };
const sigIcon = L.divIcon({ className:'sig', iconSize:[16,30], iconAnchor:[8,28],
  html:'<svg viewBox="0 0 16 30"><rect x="1.5" y="1" width="13" height="25" rx="4" fill="#1b1f22" stroke="#fff" stroke-width="1.4"/><circle cx="8" cy="7" r="2.8" fill="#ff4b3e"/><circle cx="8" cy="13.5" r="2.8" fill="#ffc21a"/><circle cx="8" cy="20" r="2.8" fill="#3ddc6a"/></svg>' });
async function loadSignals(R, line){
  SIG.layer.clearLayers();
  try{
    const r = await opFetch({ method:'POST', headers:{ 'Content-Type':'application/x-www-form-urlencoded' },
      body:'data='+encodeURIComponent(`[out:json][timeout:25];node[highway=traffic_signals](around:25,${line});out 1500;`) });
    if (!r.ok || CAM.route !== R) return;
    const j = await r.json(), kept = [];
    for (const e of j.elements || []){
      const p = [e.lat, e.lon];
      if (kept.some(k => dist(k, p) < 45)) continue;                  // one light per intersection
      kept.push(p); L.marker(p, { icon:sigIcon, interactive:false, keyboard:false }).addTo(SIG.layer);
    }
  }catch{}
}

/* ---------- Waze-style speedometer ring ---------- */
const SP_LEN = 2 * Math.PI * 31 * .75;     // 270° arc on r=31
function renderSpeedRing(v, lv, over){
  const arc = $('#spArc'); if (!arc) return;
  const max = lv ? lv * 1.5 : (imperial() ? 80 : 130);
  arc.style.strokeDasharray = `${(Math.min(1, v / max) * SP_LEN).toFixed(1)} 999`;
  arc.classList.toggle('over', !!over);
}

/* ---------- boot ---------- */
applyTheme(); renderIdle(); drawReports(); drawPark(); renderMute(); updateSpeedo(); startGPS(); refreshConditions(true);
// Android grants orientation without a prompt, so restore the compass; iOS needs a tap each visit.
renderOrient(); kickAnim();
if (store.get('compass', false) && typeof DeviceOrientationEvent !== 'undefined' && typeof DeviceOrientationEvent.requestPermission !== 'function') compassOn();
</script>
</body>
</html>
