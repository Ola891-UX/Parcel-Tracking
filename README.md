# Parcel-Tracking
<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Japan Delivery — Parcel Tracking</title>

<!-- Leaflet CSS (CDN) -->
<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css">

<style>
  :root{
    --bg:#f4f6f8; --card:#fff; --accent:#c62828; --muted:#6b7280;
    --glass: rgba(255,255,255,0.7);
  }
  html,body{height:100%; margin:0; font-family: Inter, system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial;}
  body{background:linear-gradient(180deg,#f7fafc,#eef2f7); color:#111;}

  /* Center auth */
  #authWrap{min-height:100vh; display:flex; align-items:center; justify-content:center; padding:24px;}
  .card{background:var(--card); border-radius:14px; box-shadow:0 12px 40px rgba(12,24,40,0.08); max-width:980px; width:100%; overflow:hidden; display:flex; gap:0;}
  .left{flex:1.05; padding:28px 26px;}
  .right{flex:1.15; background:linear-gradient(180deg, rgba(198,40,40,0.06), rgba(198,40,40,0.02)); padding:26px; display:flex; flex-direction:column; gap:12px;}

  h1{margin:0; font-size:20px;}
  p.lead{margin:6px 0 18px 0; color:var(--muted); font-size:14px;}

  label.small{display:block; font-size:13px; color:var(--muted); margin-bottom:6px;}
  input[type="text"]{width:100%; padding:12px 14px; border-radius:8px; border:1px solid #e5e7eb; font-size:15px; outline:none;}
  .btn{display:inline-block; padding:10px 14px; border-radius:10px; background:#111; color:#fff; border:none; font-weight:600; cursor:pointer;}
  .btn.ghost{background:transparent; color:#111; border:1px solid #e5e7eb;}
  .muted{color:var(--muted); font-size:13px;}

  .error{color:#b91c1c; margin-top:8px; display:none;}

  /* Map container (hidden until valid) */
  #app{display:none; min-height:100vh; background:var(--bg);}

  .topbar{
    position:sticky; top:12px; z-index:999;
    display:flex; gap:12px; align-items:center; justify-content:space-between;
    padding:12px; background:var(--card); border-radius:12px; margin:16px auto; width:calc(100% - 32px);
    max-width:1200px; box-shadow:0 8px 20px rgba(12,24,40,0.06);
  }

  .infoBlock{display:flex; gap:14px; align-items:center;}
  .stamp{width:58px; height:58px; border-radius:10px; background:linear-gradient(135deg,#fff,#fff); display:flex; align-items:center; justify-content:center; border:2px dashed rgba(0,0,0,0.06); font-weight:700; color:var(--accent);}
  .meta{display:flex; flex-direction:column;}
  .meta small{color:var(--muted); font-size:12px;}
  .meta strong{font-size:15px;}

  .actions{display:flex; gap:8px; align-items:center;}

  /* layout */
  .content{max-width:1200px; margin:18px auto; display:grid; grid-template-columns: 440px 1fr; gap:18px; padding:0 16px;}
  @media(max-width:980px){ .content{grid-template-columns:1fr; } .card{flex-direction:column;} .left,.right{padding:16px;} .topbar{flex-direction:column; align-items:flex-start;} }

  /* left column: timeline + history */
  .panel{background:var(--card); border-radius:12px; padding:14px; box-shadow:0 8px 20px rgba(12,24,40,0.04);}
  .timeline{display:flex; flex-direction:column; gap:12px; margin-top:8px;}
  .step{display:flex; gap:12px; align-items:flex-start;}
  .dot{width:12px; height:12px; border-radius:50%; background:#e5e7eb; margin-top:4px;}
  .dot.active{background:var(--accent);}
  .step .body{flex:1;}
  .time{font-size:12px; color:var(--muted);}

  .history{margin-top:12px; display:flex; flex-direction:column; gap:8px;}
  .histItem{padding:10px; border-radius:8px; background:linear-gradient(180deg,#fff,#fcfcfd); border:1px solid #f0f2f5;}
  .histItem small{color:var(--muted); font-size:12px;}

  /* right column: map */
  #mapWrap{height:72vh; border-radius:12px; overflow:hidden; position:relative; border:1px solid #e8edf3;}
  #map{height:100%; width:100%;}
  .controlsRow{display:flex; gap:8px; margin-top:10px; align-items:center;}

  /* progress + countdown */
  .progressbar{height:10px; background:#eef2f6; border-radius:999px; overflow:hidden; margin-top:8px;}
  .progress{height:100%; background:linear-gradient(90deg,#f97316,#c62828); width:0%; transition:width 0.6s linear;}

  /* small responsive */
  .k{font-weight:700; font-size:16px;}

  /* loading overlay */
  .loaderOverlay{position:absolute; inset:0; display:flex; align-items:center; justify-content:center; background:linear-gradient(180deg, rgba(255,255,255,0.7), rgba(255,255,255,0.6)); z-index:30;}
  .spinner{width:64px;height:64px;border-radius:12px;display:flex;align-items:center;justify-content:center; font-size:28px; transform:rotate(0)}
  .dots{display:flex; gap:8px;}
  .dotPulse{width:12px;height:12px;border-radius:50%; background:var(--accent); animation:pulse 1s infinite;}
  .dotPulse:nth-child(2){animation-delay:.2s} .dotPulse:nth-child(3){animation-delay:.4s}
  @keyframes pulse { 0%{transform:scale(.6);opacity:.6}50%{transform:scale(1);opacity:1}100%{transform:scale(.6);opacity:.6} }

  /* language toggle */
  .langToggle{display:flex; gap:8px; align-items:center;}
  .langBtn{padding:6px 8px; border-radius:8px; border:1px solid #e5e7eb; cursor:pointer; background:transparent;}
</style>
</head>
<body>

<!-- AUTH / TRACKING ENTRY -->
<div id="authWrap">
  <div class="card" role="main" aria-labelledby="title">
    <div class="left">
      <h1 id="title">Track your parcel</h1>
      <p class="lead" id="subtitle">Enter your tracking number to view live progress & map.</p>

      <label class="small" id="labelTracking">Tracking number</label>
      <input id="trackingInput" type="text" inputmode="text" placeholder="EA778899001JP" autofocus>

      <div style="display:flex; gap:8px; margin-top:12px;">
        <button class="btn" id="btnTrack">Track</button>
        <button class="btn ghost" id="btnDemo">Demo</button>
      </div>

      <div class="error" id="errorMsg">Invalid tracking number</div>

      <div style="margin-top:18px;">
        <strong>Secure:</strong> Only the generated tracking number will unlock this shipment.
        <div class="muted" style="margin-top:8px;">Receiver: <strong id="receiverInfo">Hamabe Teruhiko</strong></div>
        <div class="muted">Delivery type: <strong id="deliveryType">Miscellaneous Delivery</strong></div>
      </div>
    </div>

    <div class="right">
      <div style="display:flex; justify-content:space-between; align-items:center;">
        <div style="display:flex; gap:12px; align-items:center;">
          <div class="stamp">JP</div>
          <div>
            <div class="muted">Carrier</div>
            <div class="k">Japan Delivery</div>
          </div>
        </div>

        <div class="langToggle">
          <button class="langBtn" id="btnEN">EN</button>
          <button class="langBtn" id="btnJP">日本語</button>
        </div>
      </div>

      <div style="margin-top:16px; color:var(--muted); font-size:13px;">
        This demo is offline — map tiles use OpenStreetMap. No backend required.
      </div>

      <div style="margin-top:auto; display:flex; gap:8px;">
        <div style="flex:1;">
          <small class="muted">Quick help</small>
          <div style="font-size:13px; margin-top:6px;" class="muted">Enter <strong>EA778899001JP</strong> to unlock.</div>
        </div>
      </div>
    </div>
  </div>
</div>

<!-- APP (hidden until auth) -->
<div id="app">
  <div class="topbar">
    <div class="infoBlock">
      <div class="stamp">JP</div>
      <div class="meta">
        <small id="labelTN">Tracking Number</small>
        <strong id="showTN">EA778899001JP</strong>
      </div>
      <div style="width:1px; height:26px; background:#eef2f6; margin:0 12px;"></div>
      <div class="meta">
        <small id="labelReceiver">Receiver</small>
        <strong id="showReceiver">Hamabe Teruhiko</strong>
      </div>
      <div style="width:1px; height:26px; background:#eef2f6; margin:0 12px;"></div>
      <div class="meta">
        <small id="labelType">Delivery Type</small>
        <strong id="showType">Miscellaneous Delivery</strong>
      </div>
    </div>

    <div class="actions">
      <div class="muted" id="etaText">ETA: —</div>
      <div style="width:10px;"></div>
      <button class="btn ghost" id="btnCopy">Copy</button>
      <button class="btn" id="btnSignOut">Sign out</button>
    </div>
  </div>

  <div class="content">
    <!-- left: timeline + history -->
    <div>
      <div class="panel">
        <strong id="timelineTitle">Shipment Timeline</strong>
        <div class="timeline" id="timeline"></div>

        <div style="margin-top:12px;">
          <div class="progressbar"><div class="progress" id="progressBar"></div></div>
        </div>
      </div>

      <div class="panel" style="margin-top:12px;">
        <strong id="historyTitle">Shipment History</strong>
        <div class="history" id="historyList"></div>
      </div>
    </div>

    <!-- right: map -->
    <div>
      <div class="panel" style="padding:0; overflow:hidden;">
        <div id="mapWrap">
          <div id="map"></div>
          <div class="loaderOverlay" id="mapLoader" style="display:flex;">
            <div class="spinner">
              <div class="dots" aria-hidden="true">
                <div class="dotPulse"></div>
                <div class="dotPulse"></div>
                <div class="dotPulse"></div>
              </div>
            </div>
          </div>
        </div>

        <div style="padding:12px; display:flex; justify-content:space-between; align-items:center;">
          <div>
            <small class="muted">Started</small>
            <div id="startTime">—</div>
          </div>
          <div>
            <small class="muted">Estimated delivery</small>
            <div id="endTime">—</div>
          </div>
          <div style="min-width:160px;">
            <small class="muted">Countdown</small>
            <div id="countdown">—</div>
          </div>
        </div>

      </div>

      <div style="display:flex; gap:8px; margin-top:10px;">
        <button class="btn" id="btnFast">Fast demo (2m)</button>
        <button class="btn ghost" id="btnReset">Reset 48h</button>
        <button class="btn ghost" id="btnCenter">Center map</button>
      </div>
    </div>
  </div>
</div>

<!-- Leaflet JS -->
<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>

<script>
/* ======================
   Configuration / State
   ====================== */
const VALID_TRACK = "EA778899001JP";
const RECEIVER = "Hamabe Teruhiko";
const DELIVERY_TYPE = "Miscellaneous Delivery";

/* Coordinates */
const TOKYO = { lat: 35.6762, lng: 139.6503 };
const SORTING = { lat: 35.6895, lng: 139.7000 }; // Tokyo sorting (approx)
const NIIGATA = { lat: 37.9162, lng: 139.0364 };
const PORT = { lat: 37.9150, lng: 139.0387 };
const SADO = { lat: 38.0206, lng: 138.3525 };

/* Checkpoints array - timestamps will be generated along start->end */
const CHECKPOINTS = [
  { key:"dispatch", labelEN:"Tokyo Dispatch", labelJP:"東京出荷", coord:TOKYO },
  { key:"sorting", labelEN:"Sorting Center", labelJP:"仕分けセンター", coord:SORTING },
  { key:"niigata", labelEN:"Niigata Processing", labelJP:"新潟処理", coord:NIIGATA },
  { key:"port", labelEN:"Ferry / Port", labelJP:"フェリー港", coord:PORT },
  { key:"arrival", labelEN:"Sado Arrival", labelJP:"佐渡到着", coord:SADO }
];

/* DOM elements */
const authWrap = document.getElementById('authWrap');
const btnTrack = document.getElementById('btnTrack');
const btnDemo = document.getElementById('btnDemo');
const trackingInput = document.getElementById('trackingInput');
const errorMsg = document.getElementById('errorMsg');

const app = document.getElementById('app');
const showTN = document.getElementById('showTN');
const showReceiver = document.getElementById('showReceiver');
const showType = document.getElementById('showType');
const btnCopy = document.getElementById('btnCopy');
const btnSignOut = document.getElementById('btnSignOut');

const timelineEl = document.getElementById('timeline');
const historyList = document.getElementById('historyList');
const progressBar = document.getElementById('progressBar');
const startTimeEl = document.getElementById('startTime');
const endTimeEl = document.getElementById('endTime');
const countdownEl = document.getElementById('countdown');
const etaText = document.getElementById('etaText');

const btnFast = document.getElementById('btnFast');
const btnReset = document.getElementById('btnReset');
const btnCenter = document.getElementById('btnCenter');
const mapLoader = document.getElementById('mapLoader');

const btnEN = document.getElementById('btnEN');
const btnJP = document.getElementById('btnJP');

/* Localization */
let LANG = 'en'; // 'en' or 'jp'
const T = {
  en: {
    trackingNumber: "Tracking Number",
    receiver: "Receiver",
    deliveryType: "Delivery Type",
    eta: "ETA",
    timeline: "Shipment Timeline",
    history: "Shipment History",
    start: "Started",
    estimated: "Estimated delivery",
    countdown: "Countdown",
    statusWaiting: "Waiting to depart",
    statusTransit: "In transit",
    statusDelivered: "Delivered"
  },
  jp: {
    trackingNumber: "お問い合わせ番号",
    receiver: "受取人",
    deliveryType: "配送種別",
    eta: "到着予定",
    timeline: "配送スケジュール",
    history: "配送履歴",
    start: "開始",
    estimated: "到着予定",
    countdown: "残り時間",
    statusWaiting: "出発待ち",
    statusTransit: "配送中",
    statusDelivered: "配達完了"
  }
};

/* Time state */
let startTimestamp = null;
let endTimestamp = null;
let loopInterval = null;
let map = null, marker = null, routeLine = null, movingLine = null;

/* ======================
   Helpers
   ====================== */
function fmtDate(ts){
  return new Date(ts).toLocaleString(LANG === 'jp' ? 'ja-JP' : undefined);
}
function clamp(v,a,b){ return Math.max(a,Math.min(b,v)); }
function lerp(a,b,t){ return a + (b-a)*t; }

/* Automatic language detection (if browser/phone in Japanese) */
(function detectLang(){
  const userLang = navigator.language || navigator.userLanguage || 'en';
  if(userLang.startsWith('ja')) LANG = 'jp';
  applyLang();
})();

function applyLang(){
  // top bar labels
  document.getElementById('labelTN').textContent = T[LANG].trackingNumber;
  document.getElementById('labelReceiver').textContent = T[LANG].receiver;
  document.getElementById('labelType').textContent = T[LANG].deliveryType;
  document.getElementById('etaText').textContent = T[LANG].eta + ': —';
  document.getElementById('timelineTitle').textContent = T[LANG].timeline;
  document.getElementById('historyTitle').textContent = T[LANG].history;
  document.getElementById('startTime').previousElementSibling.textContent = T[LANG].start;
  document.getElementById('endTime').previousElementSibling.textContent = T[LANG].estimated;
  document.getElementById('countdown').previousElementSibling.textContent = T[LANG].countdown;
  // buttons - visual only
  btnEN.style.opacity = LANG==='en'? '1':'0.6';
  btnJP.style.opacity = LANG==='jp'? '1':'0.6';
}

/* Language toggle events */
btnEN.addEventListener('click', ()=>{ LANG='en'; applyLang(); renderTimeline(); renderHistory(); updateTimesUI(); });
btnJP.addEventListener('click', ()=>{ LANG='jp'; applyLang(); renderTimeline(); renderHistory(); updateTimesUI(); });

/* ================
   Auth handlers
   ================ */
btnTrack.addEventListener('click', attemptUnlock);
trackingInput.addEventListener('keydown', (e)=>{ if(e.key==='Enter') attemptUnlock(); });

btnDemo.addEventListener('click', ()=>{ trackingInput.value = VALID_TRACK; attemptUnlock(true); });

function attemptUnlock(demo=false){
  const v = (trackingInput.value || '').trim();
  if(v === VALID_TRACK){
    // show app
    authWrap.style.display = 'none';
    app.style.display = 'block';
    showTN.textContent = VALID_TRACK;
    showReceiver.textContent = RECEIVER;
    showType.textContent = DELIVERY_TYPE;
    initShipment(demo);
  } else {
    errorMsg.style.display = 'block';
    setTimeout(()=> errorMsg.style.display = 'none', 2500);
  }
}

btnCopy.addEventListener('click', ()=>{
  navigator.clipboard?.writeText(VALID_TRACK).then(()=> {
    btnCopy.textContent = LANG==='jp' ? 'コピー済み' : 'Copied';
    setTimeout(()=> btnCopy.textContent = 'Copy', 1300);
  }).catch(()=> {});
});
btnSignOut.addEventListener('click', ()=>{
  // reset to auth
  app.style.display = 'none';
  authWrap.style.display = 'flex';
  trackingInput.value = '';
});

/* ===============
   Shipment logic
   =============== */

function initShipment(demo=false){
  // set times
  const now = Date.now();
  if(demo){
    startTimestamp = now;
    endTimestamp = now + 2*60*1000; // 2 minutes demo
  } else {
    startTimestamp = now;
    endTimestamp = now + 48*3600*1000; // 48 hours
  }

  // Build timeline/hx + map
  renderTimeline();
  renderHistory(true); // initial
  initMap(); // map will show loader briefly
  startLoop();
}

/* Timeline: display steps and fill progress based on time */
function renderTimeline(){
  timelineEl.innerHTML = '';
  const total = CHECKPOINTS.length;
  for(let i=0;i<total;i++){
    const cp = CHECKPOINTS[i];
    const el = document.createElement('div');
    el.className = 'step';
    const dot = document.createElement('div');
    dot.className = 'dot';
    dot.id = 'dot-'+cp.key;
    const body = document.createElement('div');
    body.className = 'body';
    const label = document.createElement('div');
    label.innerHTML = `<strong>${LANG==='jp'?cp.labelJP:cp.labelEN}</strong>`;
    const eta = document.createElement('div');
    eta.className = 'time';
    eta.id = 'eta-'+cp.key;
    body.appendChild(label);
    body.appendChild(eta);
    el.appendChild(dot);
    el.appendChild(body);
    timelineEl.appendChild(el);
  }
}

/* Shipment history: realistic fake entries using timestamps spaced along route */
function renderHistory(initial=false){
  historyList.innerHTML = '';
  const events = generateHistoryEvents();
  for(const ev of events){
    const item = document.createElement('div');
    item.className = 'histItem';
    item.innerHTML = `<div style="display:flex;justify-content:space-between;align-items:start;">
        <div>
          <div><strong>${LANG==='jp'?ev.titleJP:ev.titleEN}</strong></div>
          <small class="muted">${LANG==='jp'?ev.locJP:ev.locEN}</small>
        </div>
        <div style="text-align:right;"><small class="muted">${fmtDate(ev.ts)}</small></div>
      </div>
      <div style="margin-top:6px;" class="muted">${LANG==='jp'?ev.noteJP:ev.noteEN}</div>`;
    historyList.appendChild(item);
    if(initial && historyList.children.length>4) break;
  }
}

/* Create history events spaced between start and end */
function generateHistoryEvents(){
  const events = [];
  const now = Date.now();
  const total = CHECKPOINTS.length;
  for(let i=0;i<total;i++){
    // assign timestamp proportional
    const frac = i/(total-1);
    const ts = Math.round( startTimestamp + (endTimestamp - startTimestamp) * frac );
    const cp = CHECKPOINTS[i];
    events.push({
      key: cp.key,
      ts,
      titleEN: cp.labelEN,
      titleJP: cp.labelJP,
      locEN: cp.labelEN,
      locJP: cp.labelJP,
      noteEN: i===0? 'Parcel scanned and dispatched.' : (i===total-1? 'Arrived at destination.' : 'In transit to next hub.'),
      noteJP: i===0? '荷物は出荷されました。' : (i===total-1? '目的地に到着しました。' : '次の中継所へ移動中です。')
    });
  }
  // Add final "Delivered" event slightly after endTimestamp
  events.push({
    key: 'delivered',
    ts: endTimestamp + 5*60*1000,
    titleEN: 'Delivered',
    titleJP: '配達完了',
    locEN: 'Recipient address',
    locJP: '受取人住所',
    noteEN: 'Package handed to recipient.',
    noteJP: '荷物は受取人に渡されました。'
  });
  // sort desc
  events.sort((a,b)=> b.ts - a.ts);
  return events;
}

/* =================
   Map + animation
   ================= */
function initMap(){
  // small delay to simulate loading tiles
  mapLoader.style.display = 'flex';
  setTimeout(()=> {
    mapLoader.style.display = 'none';
    if(!map){
      map = L.map('map', { zoomControl:true }).setView([36.2, 139.0], 6);
      L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', { maxZoom:18 }).addTo(map);

      // route polyline (curved approximation via multiple points)
      const latlngs = [
        [TOKYO.lat, TOKYO.lng],
        [SORTING.lat, SORTING.lng],
        [NIIGATA.lat, NIIGATA.lng],
        [PORT.lat, PORT.lng],
        [SADO.lat, SADO.lng]
      ];
      routeLine = L.polyline(latlngs, { color:'#c62828', weight:4, opacity:0.8 }).addTo(map);

      // marker (emoji + rotates slightly)
      marker = L.marker([TOKYO.lat, TOKYO.lng], {
        icon: L.divIcon({ html:'<div style="font-size:20px;">🚚</div>', className:'', iconSize:[28,28], iconAnchor:[14,14] })
      }).addTo(map);

      map.fitBounds(routeLine.getBounds().pad(0.6));
    }
  }, 800);
}

/* Move marker along route based on progress (linear along segments) */
function getPositionAlongRoute(t){
  // break route into segments and interpolate
  const pts = [TOKYO, SORTING, NIIGATA, PORT, SADO];
  if(t<=0) return pts[0];
  if(t>=1) return pts[pts.length-1];
  const seg = 1/(pts.length-1);
  const idx = Math.floor(t/seg);
  const localT = (t - seg*idx) / seg;
  const a = pts[idx], b = pts[idx+1];
  return { lat: lerp(a.lat, b.lat, localT), lng: lerp(a.lng, b.lng, localT) };
}

function updateUI(){
  const now = Date.now();
  const total = endTimestamp - startTimestamp;
  const raw = (now - startTimestamp) / total;
  const p = clamp(raw, 0, 1);

  // set progress bar
  progressBar.style.width = (p*100) + '%';

  // update checkpoint states & ETA labels
  for(let i=0;i<CHECKPOINTS.length;i++){
    const cp = CHECKPOINTS[i];
    const dot = document.getElementById('dot-'+cp.key);
    const etaEl = document.getElementById('eta-'+cp.key);
    // timestamp at fraction
    const frac = i/(CHECKPOINTS.length-1);
    const ts = Math.round( startTimestamp + (endTimestamp - startTimestamp) * frac );
    etaEl.textContent = fmtDate(ts);
    if(p >= frac) dot.classList.add('active'); else dot.classList.remove('active');
  }

  // update history list periodically
  renderHistory();

  // marker movement
  const pos = getPositionAlongRoute(p);
  if(marker) marker.setLatLng([pos.lat, pos.lng]);

  // countdown
  const remaining = endTimestamp - now;
  countdownEl.textContent = remaining <= 0 ? (LANG==='jp'?'到着済':'Arrived') : formatCountdown(remaining);

  // ETA text
  etaText.textContent = T[LANG].eta + ': ' + fmtDate(endTimestamp);

  // Start/end times
  startTimeEl.textContent = fmtDate(startTimestamp);
  endTimeEl.textContent = fmtDate(endTimestamp);
}

/* Pretty countdown */
function formatCountdown(ms){
  if(ms<=0) return LANG==='jp' ? '到着済' : 'Arrived';
  const s = Math.floor(ms/1000);
  const days = Math.floor(s/86400); let rem = s%86400;
  const hrs = Math.floor(rem/3600); rem = rem%3600;
  const mins = Math.floor(rem/60); const secs = rem%60;
  if(days>0) return `${days}d ${hrs}h ${mins}m`;
  if(hrs>0) return `${hrs}h ${mins}m`;
  if(mins>0) return `${mins}m ${secs}s`;
  return `${secs}s`;
}

/* Start update loop */
function startLoop(){
  if(loopInterval) clearInterval(loopInterval);
  updateUI();
  loopInterval = setInterval(updateUI, 1000);
}

/* Controls */
btnFast.addEventListener('click', ()=>{
  startTimestamp = Date.now();
  endTimestamp = Date.now() + 2*60*1000; // 2 minutes
  startLoop();
});
btnReset.addEventListener('click', ()=>{
  startTimestamp = Date.now();
  endTimestamp = Date.now() + 48*3600*1000;
  startLoop();
});
btnCenter.addEventListener('click', ()=>{
  if(routeLine) map.fitBounds(routeLine.getBounds().pad(0.5));
});

/* Utility: update times (after lang change) */
function updateTimesUI(){
  startTimeEl.textContent = fmtDate(startTimestamp);
  endTimeEl.textContent = fmtDate(endTimestamp);
}

/* Initialize once app loaded */
window.addEventListener('load', ()=>{
  // prefill tracking input as hint (user asked earlier)
  trackingInput.value = "";
});

</script>
</body>
</html>
