<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
<title>ANILLO: Última Guardia</title>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Chakra+Petch:wght@400;500;600;700&display=swap" rel="stylesheet">
<style>
:root{
  --ink:#070d13; --panel:rgba(10,20,29,.82); --line:rgba(95,211,232,.45);
  --holo:#5fd3e8; --holo-dim:#2a7f92; --amber:#ffb347; --alert:#ff5a4d; --ok:#7dffb0; --text:#e6f4f8;
  box-sizing:border-box;
  padding-top:env(safe-area-inset-top,0px);
  padding-bottom:env(safe-area-inset-bottom,0px);
}
@media (prefers-color-scheme: dark){:root:not([data-theme="light"]){--ink:#070d13;--text:#e6f4f8}}
:root[data-theme="dark"]{--ink:#070d13;--text:#e6f4f8}
html{scroll-padding-top:env(safe-area-inset-top,0px);height:100%}
*{box-sizing:border-box}
body{margin:0;height:100%;overflow:hidden;background:var(--ink);color:var(--text);
  font-family:'Chakra Petch','Segoe UI',Roboto,Helvetica,Arial,sans-serif;user-select:none;-webkit-user-select:none}
#game{position:fixed;inset:0}
#game canvas{display:block;width:100%;height:100%;cursor:crosshair}
.hidden{display:none!important}

/* HUD */
#hud{position:fixed;inset:0;pointer-events:none;
  padding:calc(env(safe-area-inset-top,0px) + 14px) 18px calc(env(safe-area-inset-bottom,0px) + 14px)}
#vignette{position:fixed;inset:0;opacity:0;transition:opacity .12s;
  background:radial-gradient(ellipse at center,rgba(255,40,30,0) 45%,rgba(255,40,30,.55) 100%)}
#vignette.low{animation:pulse 1s infinite}
@keyframes pulse{0%,100%{box-shadow:inset 0 0 90px rgba(255,40,30,.25)}50%{box-shadow:inset 0 0 150px rgba(255,40,30,.5)}}
.lbl{font-size:11px;letter-spacing:.14em;color:var(--holo);margin:0 0 3px;font-weight:600}
.lbl.small{font-size:10px;color:var(--holo-dim);margin-top:2px}
#topLeft{position:absolute;left:18px;top:calc(env(safe-area-inset-top,0px) + 14px);width:260px;
  background:var(--panel);border:1px solid var(--line);padding:10px 12px;clip-path:polygon(0 0,100% 0,100% calc(100% - 10px),calc(100% - 10px) 100%,0 100%)}
.bar{height:11px;background:rgba(255,255,255,.08);margin-bottom:9px;position:relative;overflow:hidden}
.bar:last-child{margin-bottom:0}
.fill{height:100%;width:100%;transition:width .08s linear}
.fill.shield{background:linear-gradient(90deg,#2b8fff,#7fe4ff);box-shadow:0 0 10px #4fc3ff}
.fill.health{background:linear-gradient(90deg,#3fdc7b,#b9ff8a)}
.fill.health.low{background:linear-gradient(90deg,#ff4d3d,#ff9a5c)}
.fill.boss{background:linear-gradient(90deg,#ff3b30,#ffb347)}
#topCenter{position:absolute;left:50%;top:calc(env(safe-area-inset-top,0px) + 14px);transform:translateX(-50%);text-align:center;min-width:280px}
#waveInfo{font-size:14px;letter-spacing:.12em;color:var(--text);background:var(--panel);border:1px solid var(--line);padding:6px 16px;display:inline-block}
#bossWrap{margin-top:8px;width:340px;max-width:60vw;margin-left:auto;margin-right:auto}
#bossWrap .bar{height:9px}
#topRight{position:absolute;right:18px;top:calc(env(safe-area-inset-top,0px) + 14px);text-align:right;background:var(--panel);border:1px solid var(--line);padding:8px 14px;min-width:150px}
#score{font-size:30px;font-weight:700;line-height:1;color:var(--text)}
#banner{position:absolute;left:0;right:0;top:24%;text-align:center;opacity:0;transition:opacity .35s}
#banner.show{opacity:1}
#bannerTitle{font-size:clamp(28px,6vw,58px);font-weight:700;letter-spacing:.2em;text-shadow:0 0 24px rgba(95,211,232,.7)}
#bannerSub{font-size:clamp(13px,2vw,18px);color:var(--amber);margin-top:6px;letter-spacing:.06em}
#crosshair{position:absolute;left:50%;top:50%;width:0;height:0}
#crosshair i{position:absolute;background:rgba(230,250,255,.9);box-shadow:0 0 3px #000}
#crosshair i:nth-child(1){left:-1px;top:-14px;width:2px;height:8px}
#crosshair i:nth-child(2){left:-1px;top:6px;width:2px;height:8px}
#crosshair i:nth-child(3){top:-1px;left:-14px;height:2px;width:8px}
#crosshair i:nth-child(4){top:-1px;left:6px;height:2px;width:8px}
#hitm{position:absolute;left:-12px;top:-12px;width:24px;height:24px;opacity:0}
#hitm::before,#hitm::after{content:"";position:absolute;left:11px;top:-2px;width:2px;height:28px;background:#fff;transform:rotate(45deg)}
#hitm::after{transform:rotate(-45deg)}
#hitm.on{opacity:1}
#hitm.head::before,#hitm.head::after{background:var(--amber)}
#hitm.kill::before,#hitm.kill::after{background:var(--alert)}
#radar{position:absolute;left:18px;bottom:calc(env(safe-area-inset-bottom,0px) + 14px);width:150px;height:150px}
#bottomRight{position:absolute;right:18px;bottom:calc(env(safe-area-inset-bottom,0px) + 14px);text-align:right;background:var(--panel);border:1px solid var(--line);padding:10px 16px;min-width:240px}
#wname{font-size:12px;letter-spacing:.14em;color:var(--holo)}
#ammo{font-size:38px;font-weight:700;line-height:1.1}
#ammo small{font-size:18px;color:var(--holo-dim);font-weight:500}
#heatWrap{height:8px;background:rgba(255,255,255,.08);margin:6px 0 4px}
#heatFill{height:100%;width:0;background:linear-gradient(90deg,#5fd3e8,#ffb347,#ff3b30)}
#gren{font-size:15px;letter-spacing:.2em;color:var(--ok);margin-top:4px}
#reloadTxt{position:absolute;left:50%;top:58%;transform:translateX(-50%);color:var(--amber);letter-spacing:.2em;font-size:14px}
#hintLook{position:absolute;left:50%;bottom:calc(env(safe-area-inset-bottom,0px) + 14px);transform:translateX(-50%);font-size:12px;color:var(--holo-dim);text-align:center;max-width:36vw}

/* Overlay */
#overlay{position:fixed;inset:0;display:flex;align-items:center;justify-content:center;
  background:radial-gradient(ellipse at 50% 40%,rgba(7,13,19,.25),rgba(7,13,19,.88));padding:20px;overflow:auto}
.card{width:min(620px,100%);background:var(--panel);border:1px solid var(--line);padding:28px 30px;
  clip-path:polygon(0 0,calc(100% - 18px) 0,100% 18px,100% 100%,18px 100%,0 calc(100% - 18px));backdrop-filter:blur(6px)}
.card h1{margin:0;font-size:clamp(44px,9vw,76px);letter-spacing:.32em;line-height:1;font-weight:700;padding-left:.32em}
.card h2{margin:6px 0 16px;font-size:clamp(15px,3vw,22px);font-weight:500;letter-spacing:.42em;color:var(--holo)}
.card p{margin:0 0 16px;line-height:1.55;max-width:60ch;color:#c9dde4}
#ovStats{display:flex;gap:26px;margin-bottom:16px}
#ovStats div{font-size:12px;color:var(--holo)}
#ovStats b{display:block;font-size:26px;color:var(--text)}
button.play{font-family:inherit;font-size:18px;font-weight:700;letter-spacing:.2em;padding:13px 34px;color:#04141a;background:var(--holo);border:0;cursor:pointer;
  clip-path:polygon(0 0,calc(100% - 12px) 0,100% 12px,100% 100%,12px 100%,0 calc(100% - 12px));transition:background .15s,transform .1s}
button.play:hover{background:#9be8f6}
button.play:active{transform:translateY(1px)}
button.play:focus-visible{outline:2px solid var(--amber);outline-offset:3px}
.controls{display:grid;grid-template-columns:repeat(auto-fit,minmax(170px,1fr));gap:6px 20px;margin-top:20px;font-size:13px;color:#a9c3cc}
.controls span{display:flex;gap:10px;align-items:baseline}
.controls kbd{font-family:inherit;font-weight:700;color:var(--text);background:rgba(95,211,232,.14);border:1px solid var(--line);padding:1px 7px;min-width:42px;text-align:center;font-size:12px}
#ovNote{font-size:12px;color:var(--amber);margin-top:14px}
#pauseActions{display:flex;gap:10px;flex-wrap:wrap;margin-top:10px}
button.secondary{font-family:inherit;font-size:13px;font-weight:700;letter-spacing:.12em;padding:10px 16px;color:var(--text);background:rgba(95,211,232,.08);border:1px solid var(--line);cursor:pointer}
button.secondary:hover{background:rgba(95,211,232,.2)}
#interactPrompt{position:absolute;left:50%;bottom:calc(env(safe-area-inset-bottom,0px) + 92px);transform:translateX(-50%);padding:7px 13px;background:rgba(5,14,20,.86);border:1px solid rgba(95,211,232,.55);color:var(--text);font-size:12px;letter-spacing:.09em;text-align:center;white-space:nowrap;opacity:0;transition:opacity .15s}
#interactPrompt.on{opacity:1}
#err{position:fixed;inset:0;display:none;align-items:center;justify-content:center;background:var(--ink);color:var(--alert);padding:30px;text-align:center}
#menuOpts{margin-bottom:14px}
.opts{display:flex;gap:8px;flex-wrap:wrap;margin:6px 0 14px}
.opt{font-family:inherit;color:var(--text);background:rgba(95,211,232,.1);border:1px solid var(--line);padding:8px 14px;cursor:pointer;font-size:13px;letter-spacing:.1em;text-align:left;font-weight:600}
.opt small{display:block;font-size:10px;color:var(--holo-dim);letter-spacing:.04em;font-weight:400}
.opt:hover{background:rgba(95,211,232,.25)} .opt.on{background:var(--holo);color:#04141a} .opt.on small{color:#04303a}
#mpPanel{display:flex;gap:8px;flex-direction:column;align-items:stretch;margin-bottom:6px}
.mpRow{display:flex;gap:8px;flex-wrap:wrap}
#mpPanel .mpRow > .opt{flex:0 1 auto}
#mpSignal{width:100%;min-height:92px;resize:vertical;font-family:Consolas,monospace;font-size:11px;line-height:1.35;padding:9px;background:rgba(0,0,0,.42);color:var(--text);border:1px solid var(--line);user-select:text;-webkit-user-select:text}
#mpPanel input{font-family:inherit;font-size:13px;padding:8px 10px;background:rgba(0,0,0,.35);color:var(--text);border:1px solid var(--line);width:140px;user-select:text;-webkit-user-select:text}
#mpStatus{flex-basis:100%;font-size:12px;color:var(--amber)}
#scope{position:absolute;inset:0;opacity:0;transition:opacity .1s;background:radial-gradient(circle at center,transparent 0,transparent 26vmin,#000 27vmin)}
#scope.on{opacity:1}
#scope::before,#scope::after{content:"";position:absolute;background:rgba(0,0,0,.75)}
#scope::before{left:50%;top:calc(50% - 27vmin);width:1px;height:54vmin}
#scope::after{top:50%;left:calc(50% - 27vmin);height:1px;width:54vmin}
@media (max-width:720px){#topLeft{width:190px}#topRight{min-width:110px}#score{font-size:22px}#radar{width:110px;height:110px}#bottomRight{min-width:170px}#ammo{font-size:28px}}
@media (prefers-reduced-motion:reduce){#vignette.low{animation:none}}
</style>
</head>
<body>
<div id="game"></div>

<div id="hud" class="hidden">
  <div id="vignette"></div>
  <div id="topLeft">
    <div class="lbl">ESCUDO</div><div class="bar"><div id="shieldFill" class="fill shield"></div></div>
    <div class="lbl">SALUD</div><div class="bar"><div id="healthFill" class="fill health"></div></div>
  </div>
  <div id="topCenter">
    <div id="waveInfo">OLEADA 1</div>
    <div id="bossWrap" class="hidden"><div class="lbl">TITÁN VORAK</div><div class="bar"><div id="bossFill" class="fill boss"></div></div></div>
  </div>
  <div id="topRight"><div class="lbl">PUNTAJE</div><div id="score">0</div><div class="lbl small">RÉCORD <span id="best">0</span></div></div>
  <div id="banner"><div id="bannerTitle"></div><div id="bannerSub"></div></div>
  <div id="scope"></div>
  <div id="crosshair"><i></i><i></i><i></i><i></i><b id="hitm"></b></div>
  <div id="reloadTxt"></div>
  <canvas id="radar" width="150" height="150"></canvas>
  <div id="bottomRight">
    <div id="wname">RIFLE DE ASALTO</div>
    <div id="ammo">32 <small>/ 128</small></div>
    <div id="heatWrap" class="hidden"><div id="heatFill"></div></div>
    <div id="gren">●●●</div>
  </div>
  <div id="hintLook" class="hidden">Cursor libre: mueve el mouse hacia los bordes de la pantalla para girar, o usa las flechas.</div>
  <div id="interactPrompt"></div>
</div>

<div id="overlay">
  <div class="card">
    <h1>ANILLO</h1>
    <h2>ÚLTIMA GUARDIA</h2>
    <p id="ovText">Los Vorak abrieron una brecha en el Anillo y solo queda tu pelotón en la Base Alfa. Aguanta oleada tras oleada, sobrevive a los Titanes y no dejes que caiga tu escudo.</p>
    <div id="ovStats" class="hidden"></div>
    <div id="menuOpts">
      <div class="lbl">MAPA</div>
      <div class="opts" id="mapOpts"><button class="opt on" data-map="base">BASE ALFA<small>Campo abierto</small></button><button class="opt" data-map="city">CIUDAD<small>Calles y rascacielos</small></button><button class="opt" data-map="desert">DESIERTO<small>Mesetas y ruinas</small></button></div>
      <div class="lbl">MODO</div>
      <div class="opts" id="modeOpts"><button class="opt on" data-mode="solo">UN JUGADOR</button><button class="opt" data-mode="multi">MULTIJUGADOR</button></div>
      <div id="mpPanel" class="hidden">
        <div class="mpRow"><input id="mpName" maxlength="12" placeholder="Tu apodo" value="Soldado"><input id="mpCode" maxlength="12" placeholder="Código local" value="alfa"></div>
        <div class="mpRow"><button class="opt" id="mpJoin">SALA LOCAL</button><button class="opt" id="mpOffer">CREAR OFERTA P2P</button><button class="opt" id="mpAnswer">RESPONDER OFERTA</button><button class="opt" id="mpApply">APLICAR RESPUESTA</button><button class="opt" id="mpCopy">COPIAR CÓDIGO</button></div>
        <textarea id="mpSignal" spellcheck="false" placeholder="Para jugar entre dos computadores: Jugador 1 crea una oferta, Jugador 2 la pega aquí y responde, luego Jugador 1 pega la respuesta."></textarea>
        <div id="mpStatus">Sin conexión. Sala local funciona entre pestañas del mismo navegador; P2P conecta dos equipos sin usar la API del juego.</div>
      </div>
    </div>
    <button class="play" id="btnPlay">JUGAR</button>
    <div id="pauseActions" class="hidden"><button class="secondary" id="btnMainMenu">MENÚ PRINCIPAL</button></div>
    <div class="controls">
      <span><kbd>WASD</kbd> moverse</span>
      <span><kbd>Mouse</kbd> apuntar</span>
      <span><kbd>Clic</kbd> disparar</span>
      <span><kbd>Espacio</kbd> saltar</span>
      <span><kbd>Shift</kbd> correr</span>
      <span><kbd>R</kbd> recargar</span>
      <span><kbd>G</kbd> granada</span>
      <span><kbd>1 · 4</kbd> o rueda: arma</span><span><kbd>Q</kbd> dash</span><span><kbd>Clic der.</kbd> zoom (francotirador)</span>
      <span><kbd>P</kbd> pausa</span>
    </div>
    <div id="ovNote" class="hidden"></div>
  </div>
</div>
<div id="err">No se pudo cargar el motor 3D. Revisa tu conexión y vuelve a abrir la página.</div>

<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
<script>
(() => {
'use strict';
if (typeof THREE === 'undefined') { document.getElementById('err').style.display = 'flex'; return; }

const $ = id => document.getElementById(id);
const clamp = (v, a, b) => Math.max(a, Math.min(b, v));
const rand = (a, b) => a + Math.random() * (b - a);
const V3 = (x, y, z) => new THREE.Vector3(x, y, z);

/* ------------------------------------------------------------------ */
/* Renderer, escena, cámara                                            */
/* ------------------------------------------------------------------ */
const lowEndHint = (!!navigator.hardwareConcurrency && navigator.hardwareConcurrency <= 4) || (!!navigator.deviceMemory && navigator.deviceMemory <= 4) || innerWidth < 900;
const renderer = new THREE.WebGLRenderer({ antialias: !lowEndHint, powerPreference: 'high-performance', alpha: false });
const nativeDpr = Math.max(1, window.devicePixelRatio || 1);
let renderScale = lowEndHint ? 1.0 : 1.12;
const maxDpr = lowEndHint ? 1.15 : 1.5;
function applyRenderQuality(){
  renderer.setPixelRatio(Math.min(nativeDpr * renderScale, maxDpr));
  renderer.setSize(innerWidth, innerHeight, false);
}
applyRenderQuality();
renderer.outputEncoding = THREE.sRGBEncoding;
renderer.toneMapping = THREE.ACESFilmicToneMapping;
renderer.toneMappingExposure = 1.08;
renderer.shadowMap.enabled = true;
renderer.shadowMap.type = THREE.PCFShadowMap;
renderer.shadowMap.autoUpdate = false;
$('game').appendChild(renderer.domElement);
let perfT = 0, perfFrames = 0, perfGoodT = 0;
function monitorPerformance(dt){
  perfT += dt; perfFrames++;
  if(perfT < 0.75) return;
  const fps = perfFrames / perfT; perfT = 0; perfFrames = 0;
  if(fps < 50 && renderScale > 0.82){ renderScale = Math.max(0.82, renderScale - 0.1); applyRenderQuality(); perfGoodT = 0; }
  else if(fps > 60){ perfGoodT += 0.75; if(perfGoodT >= 2.25 && renderScale < 1.12){ renderScale = Math.min(1.12, renderScale + 0.06); applyRenderQuality(); perfGoodT = 0; } }
  else perfGoodT = Math.max(0, perfGoodT - 0.35);
}

const scene = new THREE.Scene();
scene.fog = new THREE.Fog(0xa9c7d6, 70, 300);
const camera = new THREE.PerspectiveCamera(75, innerWidth / innerHeight, 0.05, 3000);
camera.rotation.order = 'YXZ';
scene.add(camera);

// Cielo con degradado
const sky = new THREE.Mesh(
  new THREE.SphereGeometry(1500, 24, 16),
  new THREE.ShaderMaterial({
    side: THREE.BackSide, depthWrite: false, fog: false,
    uniforms: { top: { value: new THREE.Color(0x15346e) }, mid: { value: new THREE.Color(0x6ea9d8) }, bot: { value: new THREE.Color(0xa9c7d6) } },
    vertexShader: 'varying vec3 vP; void main(){ vP = normalize(position); gl_Position = projectionMatrix * modelViewMatrix * vec4(position,1.0); }',
    fragmentShader: 'uniform vec3 top; uniform vec3 mid; uniform vec3 bot; varying vec3 vP; void main(){ float h = vP.y; vec3 c = h > 0.0 ? mix(mid, top, pow(h, 0.55)) : mix(mid, bot, clamp(-h * 3.0, 0.0, 1.0)); gl_FragColor = vec4(c, 1.0); }'
  })
);
scene.add(sky);

// Anillo-mundo en el cielo
function ringTexture() {
  const c = document.createElement('canvas'); c.width = 1024; c.height = 128;
  const g = c.getContext('2d');
  g.fillStyle = '#3d7448'; g.fillRect(0, 0, 1024, 128);
  const pal = ['#2e5f8c', '#4d8c55', '#8b7c4f', '#2f6a44', '#6d9a5c', '#345f86'];
  for (let i = 0; i < 420; i++) {
    g.fillStyle = pal[(Math.random() * pal.length) | 0];
    g.globalAlpha = rand(0.5, 0.95);
    g.beginPath(); g.ellipse(Math.random() * 1024, Math.random() * 128, rand(6, 60), rand(4, 22), 0, 0, 7); g.fill();
  }
  for (let i = 0; i < 90; i++) {
    g.fillStyle = '#ffffff'; g.globalAlpha = rand(0.12, 0.4);
    g.beginPath(); g.ellipse(Math.random() * 1024, Math.random() * 128, rand(20, 90), rand(3, 9), 0, 0, 7); g.fill();
  }
  g.globalAlpha = 1;
  const t = new THREE.CanvasTexture(c);
  t.wrapS = THREE.RepeatWrapping; t.repeat.set(3, 1);
  return t;
}
const ring = new THREE.Mesh(
  new THREE.TorusGeometry(900, 40, 14, 120),
  new THREE.MeshBasicMaterial({ map: ringTexture(), fog: false })
);
ring.scale.z = 4.5;
ring.rotation.set(1.15, 0.55, 0);
scene.add(ring);

// Sol
const sunDir = V3(-0.5, 0.55, -0.65).normalize();
const sun = new THREE.Mesh(new THREE.SphereGeometry(48, 20, 14), new THREE.MeshBasicMaterial({ color: 0xfff3c4, fog: false }));
sun.position.copy(sunDir).multiplyScalar(1250);
scene.add(sun);
const sunGlow = new THREE.Mesh(new THREE.SphereGeometry(110, 20, 14), new THREE.MeshBasicMaterial({ color: 0xfff0b0, transparent: true, opacity: 0.22, fog: false, depthWrite: false }));
sunGlow.position.copy(sun.position);
scene.add(sunGlow);

// Luces
scene.add(new THREE.HemisphereLight(0xc4dcff, 0x3f4b3f, 0.75));
const dirLight = new THREE.DirectionalLight(0xfff0d8, 0.95);
dirLight.position.copy(sunDir).multiplyScalar(90);
dirLight.castShadow = true;
dirLight.shadow.mapSize.set(1024, 1024);
Object.assign(dirLight.shadow.camera, { left: -155, right: 155, top: 155, bottom: -155, near: 10, far: 240 });
dirLight.shadow.bias = -0.0006;
scene.add(dirLight); scene.add(dirLight.target);
dirLight.position.copy(sunDir).multiplyScalar(90);
dirLight.target.position.set(0, 0, 0);

const flashLight = new THREE.PointLight(0xffd9a0, 0, 14);
camera.add(flashLight); flashLight.position.set(0.2, -0.1, -1.0);
const boomLight = new THREE.PointLight(0xff9040, 0, 45);
scene.add(boomLight);

// Suelo
function groundTexture() {
  const S = 512, c = document.createElement('canvas'); c.width = c.height = S; const g = c.getContext('2d');
  g.fillStyle = '#55633d'; g.fillRect(0, 0, S, S);
  const pal = ['#6b7a45', '#4a5535', '#7a6a4a', '#3f4a30', '#857456'];
  for (let i = 0; i < 260; i++) { g.globalAlpha = rand(0.15, 0.4); g.fillStyle = pal[(Math.random() * pal.length) | 0]; g.beginPath(); g.ellipse(Math.random() * S, Math.random() * S, rand(8, 48), rand(6, 30), rand(0, 3), 0, 7); g.fill(); }
  g.globalAlpha = 1;
  for (let i = 0; i < 5000; i++) { g.strokeStyle = 'rgba(' + (Math.random() < 0.5 ? '20,40,10' : '140,160,80') + ',' + rand(0.1, 0.35) + ')'; const x = Math.random() * S, y = Math.random() * S; g.beginPath(); g.moveTo(x, y); g.lineTo(x + rand(-2, 2), y - rand(2, 7)); g.stroke(); }
  for (let i = 0; i < 300; i++) { g.fillStyle = 'rgba(60,50,35,' + rand(0.2, 0.5) + ')'; g.fillRect(Math.random() * S, Math.random() * S, rand(1, 3), rand(1, 3)); }
  const t = new THREE.CanvasTexture(c); t.wrapS = t.wrapT = THREE.RepeatWrapping; t.repeat.set(150, 150); t.anisotropy = 8; return t;
}
const ground = new THREE.Mesh(new THREE.PlaneGeometry(1200, 1200), (() => { const t = groundTexture(); return new THREE.MeshStandardMaterial({ map: t, bumpMap: t, bumpScale: 1.2, roughness: 0.95, metalness: 0 }); })());
ground.rotation.x = -Math.PI / 2; ground.receiveShadow = true;
scene.add(ground);

const decal = new THREE.Mesh(new THREE.RingGeometry(17.2, 18.2, 64), new THREE.MeshBasicMaterial({ color: 0x5fd3e8, transparent: true, opacity: 0.75, side: THREE.DoubleSide }));
decal.rotation.x = -Math.PI / 2; decal.position.y = 0.03; scene.add(decal);
const decal2 = new THREE.Mesh(new THREE.RingGeometry(40, 40.5, 96), new THREE.MeshBasicMaterial({ color: 0x5fd3e8, transparent: true, opacity: 0.25, side: THREE.DoubleSide }));
decal2.rotation.x = -Math.PI / 2; decal2.position.y = 0.03; scene.add(decal2);
const padT = noiseTex('#7c8790', '#3a4650', 256, true); padT.repeat.set(10, 10);
const pad = new THREE.Mesh(new THREE.CircleGeometry(21, 48), new THREE.MeshStandardMaterial({ map: padT, bumpMap: padT, roughness: 0.8 }));
pad.rotation.x = -Math.PI / 2; pad.position.y = 0.02; pad.receiveShadow = true; scene.add(pad);

/* ------------------------------------------------------------------ */
/* Mapa: estructuras y colisiones                                      */
/* ------------------------------------------------------------------ */
const colliders = [], mapObjs = [];
const wallMeshes = [];
const interactables = [];
let interactClock = 0;
let nearestInteractable = null;
function noiseTex(base, speck, size, lines) {
  const c = document.createElement('canvas'); c.width = c.height = size; const g = c.getContext('2d');
  g.fillStyle = base; g.fillRect(0, 0, size, size);
  for (let i = 0; i < size * 14; i++) { g.fillStyle = 'rgba(' + (i % 2 ? '255,255,255' : '0,0,0') + ',' + rand(0.02, 0.09) + ')'; g.fillRect(Math.random() * size, Math.random() * size, rand(1, 5), rand(1, 5)); }
  for (let i = 0; i < 40; i++) { g.fillStyle = speck; g.globalAlpha = rand(0.05, 0.14); g.beginPath(); g.ellipse(Math.random() * size, Math.random() * size, rand(10, 50), rand(8, 30), rand(0, 3), 0, 7); g.fill(); }
  g.globalAlpha = 1;
  if (lines) { g.strokeStyle = 'rgba(0,0,0,.4)'; g.lineWidth = 2; g.strokeRect(0, 0, size, size); g.beginPath(); g.moveTo(size / 2, 0); g.lineTo(size / 2, size); g.moveTo(0, size / 2); g.lineTo(size, size / 2); g.stroke(); g.strokeStyle = 'rgba(255,255,255,.1)'; g.strokeRect(3, 3, size - 6, size - 6); }
  const t = new THREE.CanvasTexture(c); t.wrapS = t.wrapT = THREE.RepeatWrapping; t.anisotropy = 8; return t;
}
function texMat(base, speck, rough, metal, lines) { const t = noiseTex(base, speck, 256, lines); return new THREE.MeshStandardMaterial({ map: t, bumpMap: t, bumpScale: 1.4, roughness: rough, metalness: metal }); }
const matStruct = texMat('#8b98a4', '#4a5560', 0.78, 0.15, true);
const matDark = texMat('#4f5c69', '#20262c', 0.55, 0.55, true);
const matWarm = texMat('#a89b84', '#6b5a3c', 0.85, 0.05, false);
const matAccent = new THREE.MeshStandardMaterial({ color: 0x5fd3e8, emissive: 0x5fd3e8, emissiveIntensity: 1.3 });
const boxGeoCache = {};
function boxGeo(w, h, d) {
  const k = w + ',' + h + ',' + d; if (boxGeoCache[k]) return boxGeoCache[k];
  const g = new THREE.BoxGeometry(w, h, d), uv = g.attributes.uv, dm = [[d, h], [d, h], [w, d], [w, d], [w, h], [w, h]];
  for (let f = 0; f < 6; f++) for (let v = 0; v < 4; v++) { const i = f * 4 + v; uv.setXY(i, uv.getX(i) * dm[f][0] / 4, uv.getY(i) * dm[f][1] / 4); }
  return (boxGeoCache[k] = g);
}

function addBox(cx, cz, w, d, h, y0, mat, accent) {
  y0 = y0 || 0; mat = mat || matStruct;
  const m = new THREE.Mesh(boxGeo(w, h, d), mat);
  m.position.set(cx, y0 + h / 2, cz);
  m.castShadow = !!accent || h >= 5;
  m.receiveShadow = h >= 2.4;
  scene.add(m); wallMeshes.push(m); mapObjs.push(m);
  colliders.push({ minX: cx - w / 2, maxX: cx + w / 2, minZ: cz - d / 2, maxZ: cz + d / 2, top: y0 + h, bottom: y0 });
  if (accent) {
    const s = new THREE.Mesh(boxGeo(w + 0.08, 0.1, d + 0.08), matAccent);
    s.position.set(cx, y0 + h - 0.15, cz); scene.add(s); mapObjs.push(s);
  }
  return m;
}
function stairs(x, z, dx, dz, n, sh, wid, dep, mat) {
  for (let j = 0; j < n; j++) {
    const h = sh * (n - j);
    const cx = x + dx * dep * (j + 0.5), cz = z + dz * dep * (j + 0.5);
    addBox(cx, cz, dx !== 0 ? dep : wid, dz !== 0 ? dep : wid, h, 0, mat || matDark);
  }
}

function clearInteractables() {
  interactables.forEach(o => { if (o.group) scene.remove(o.group); });
  interactables.length = 0; nearestInteractable = null;
  $('interactPrompt').classList.remove('on');
}
function addInteractable(type, name, x, z, color) {
  const g = new THREE.Group();
  const base = new THREE.Mesh(new THREE.CylinderGeometry(0.58,0.7,0.22,16), new THREE.MeshStandardMaterial({color:0x222b32,roughness:0.45,metalness:0.75}));
  base.position.y=0.12; g.add(base);
  const core = new THREE.Mesh(new THREE.CylinderGeometry(0.16,0.24,0.85,12), new THREE.MeshStandardMaterial({color,emissive:color,emissiveIntensity:1.8,roughness:0.3,metalness:0.55}));
  core.position.y=0.63; g.add(core);
  const ring = new THREE.Mesh(new THREE.TorusGeometry(0.72,0.035,8,32), new THREE.MeshBasicMaterial({color,transparent:true,opacity:0.85}));
  ring.rotation.x=Math.PI/2; ring.position.y=0.18; g.add(ring);
  const light = new THREE.PointLight(color,1.6,6); light.position.y=1.1; g.add(light);
  g.position.set(x, groundAt(x,z,0,0.2,0.6), z); scene.add(g);
  const o={type,name,x,z,color,group:g,ring,core,light,cooldown:0,phase:Math.random()*6.28}; interactables.push(o); return o;
}
function buildInteractables(map) {
  clearInteractables();
  const sets={
    base:[['shield','NÚCLEO DE ESCUDO',-52,0,0x5fd3e8],['ammo','ARSENAL DE CAMPO',52,0,0xffc040],['health','MÓDULO MÉDICO',0,-54,0x40ff80]],
    city:[['ammo','ARSENAL URBANO',0,28,0xffc040],['shield','RELÉ DE ENERGÍA',56,0,0x5fd3e8],['health','MED-BOT',-56,0,0x40ff80]],
    desert:[['health','ESTACIÓN MÉDICA',0,58,0x40ff80],['ammo','CAJA DE SUMINISTROS',-58,-12,0xffc040],['shield','RELÉ SOLAR',58,-12,0x5fd3e8]]
  };
  (sets[map]||sets.base).forEach(a=>addInteractable(...a));
}
function getNearestInteractable() {
  let best=null, bd=4.2;
  for(const o of interactables){ const d=Math.hypot(o.x-P.x,o.z-P.z); if(d<bd && o.cooldown<=0){bd=d;best=o;} }
  return best;
}
function useInteractable(o) {
  if(!o || o.cooldown>0 || state!=='playing') return;
  o.cooldown=18; let gain='';
  if(o.type==='shield'){P.shield=Math.min(100,P.shield+55);gain='+55 ESCUDO';}
  else if(o.type==='health'){P.hp=Math.min(100,P.hp+45);gain='+45 SALUD';}
  else {WP.rifle.reserve=Math.min(240,WP.rifle.reserve+90);WP.shotgun.reserve=Math.min(60,WP.shotgun.reserve+12);WP.sniper.reserve=Math.min(30,WP.sniper.reserve+8);WP.grenades=Math.min(5,WP.grenades+1);gain='+MUNICIÓN · +1 GRANADA';}
  o.core.material.emissiveIntensity=4; spark(o.group.position.clone().setY(1),o.color,22,5,0.55); tone('sine',520,980,0.14,0.18);
  banner(o.name,gain+' · disponible de nuevo en 18 s',1800);
}
function updateInteractables(dt) {
  interactClock-=dt;
  for(const o of interactables){
    o.cooldown=Math.max(0,o.cooldown-dt); const pulse=1+Math.sin(gt*4+o.phase)*0.08; o.ring.scale.setScalar(pulse); o.core.rotation.y+=dt*1.6;
    o.core.material.emissiveIntensity=o.cooldown>0?0.35:1.8; o.light.intensity=o.cooldown>0?0.18:1.5+Math.sin(gt*5+o.phase)*0.45;
  }
  if(state!=='playing'){ $('interactPrompt').classList.remove('on'); return; }
  if(interactClock>0) return; interactClock=0.08; nearestInteractable=getNearestInteractable();
  const el=$('interactPrompt'); if(nearestInteractable){el.textContent='[E] '+nearestInteractable.name;el.classList.add('on');} else el.classList.remove('on');
}

function buildBase() {
// Muros perimetrales
addBox(0, -141, 286, 2, 14, 0, matDark);
addBox(0, 141, 286, 2, 14, 0, matDark);
addBox(-141, 0, 2, 286, 14, 0, matDark);
addBox(141, 0, 2, 286, 14, 0, matDark);
// Zonas exteriores: edificios, torres, rocas y muros (mundo abierto, generado con semilla fija)
(function () {
  let sd = 1337; const r = () => ((sd = (sd * 16807) % 2147483647) / 2147483647), mats = [matStruct, matDark, matWarm];
  for (let i = 0; i < 90; i++) {
    const a = r() * 6.283, rad = 64 + r() * 72, x = Math.cos(a) * rad, z = Math.sin(a) * rad, k = r();
    if (Math.abs(x) > 130 || Math.abs(z) > 130) continue;
    if (k < 0.35) addBox(x, z, 10 + r() * 12, 10 + r() * 12, 4 + r() * 6, 0, mats[(r() * 3) | 0], r() < 0.4);
    else if (k < 0.6) { const w = 2 + r() * 3; addBox(x, z, w, w, 1.5 + r() * 2.5, 0, matWarm); }
    else if (k < 0.8) addBox(x, z, 3, 3, 10 + r() * 8, 0, matDark, true);
    else addBox(x, z, 8 + r() * 10, 1.5, 2.5, 0, matWarm);
  }
})();
// Torre central
addBox(0, 0, 14, 14, 3, 0, matStruct, true);
stairs(0, 7, 0, 1, 5, 0.5, 5, 1.2);
stairs(0, -7, 0, -1, 5, 0.5, 5, 1.2);
addBox(-4.5, 0, 1, 7, 1.2, 3, matDark);
addBox(4.5, 0, 1, 7, 1.2, 3, matDark);
addBox(0, 0, 2, 2, 5, 3, matWarm, true);
// Plataformas elevadas
[[-40, -38, 1], [40, -38, -1], [-40, 38, 1], [40, 38, -1]].forEach(([px, pz, dir]) => {
  addBox(px, pz, 12, 12, 2, 0, matStruct, true);
  stairs(px + dir * -6, pz, dir, 0, 3, 0.5, 4, 1.2);
  addBox(px - dir * 4, pz, 1, 8, 1, 2, matDark);
});
// Cobertura
[[-22, -12, 6, 2, 2.4], [22, -12, 6, 2, 2.4], [-22, 12, 6, 2, 2.4], [22, 12, 6, 2, 2.4],
 [-14, -27, 3, 3, 3], [14, -27, 3, 3, 3], [-14, 27, 3, 3, 3], [14, 27, 3, 3, 3],
 [0, -32, 12, 1.5, 2], [0, 32, 12, 1.5, 2], [-42, 0, 2, 11, 3], [42, 0, 2, 11, 3],
 [-28, 0, 1.5, 6, 1.4], [28, 0, 1.5, 6, 1.4]].forEach(a => addBox(a[0], a[1], a[2], a[3], a[4], 0, matWarm));
// Pilares altos
[[-30, -30], [30, -30], [-30, 30], [30, 30]].forEach(p => addBox(p[0], p[1], 3, 3, 9, 0, matDark, true));


}
function walls() { addBox(0, -141, 286, 2, 14, 0, matDark); addBox(0, 141, 286, 2, 14, 0, matDark); addBox(-141, 0, 2, 286, 14, 0, matDark); addBox(141, 0, 2, 286, 14, 0, matDark); }
function winTex(base, lit) {
  const c = document.createElement('canvas'); c.width = c.height = 256; const g = c.getContext('2d');
  g.fillStyle = base; g.fillRect(0, 0, 256, 256);
  for (let i = 0; i < 3000; i++) { g.fillStyle = 'rgba(0,0,0,' + rand(0.03, 0.1) + ')'; g.fillRect(Math.random() * 256, Math.random() * 256, rand(1, 4), rand(1, 4)); }
  for (let y = 0; y < 2; y++) for (let x = 0; x < 2; x++) { g.fillStyle = Math.random() < 0.35 ? lit : '#1c2a36'; g.fillRect(x * 128 + 24, y * 128 + 22, 80, 84); g.fillStyle = 'rgba(255,255,255,.14)'; g.fillRect(x * 128 + 24, y * 128 + 22, 80, 8); g.strokeStyle = 'rgba(0,0,0,.5)'; g.strokeRect(x * 128 + 24, y * 128 + 22, 80, 84); }
  const t = new THREE.CanvasTexture(c); t.wrapS = t.wrapT = THREE.RepeatWrapping; t.anisotropy = 8; return t;
}
function buildCity() {
  const bm = [winTex('#8a5a48', '#ffd27a'), winTex('#9aa3ab', '#ffe6a8'), winTex('#4a5866', '#9fe6ff')].map(t => new THREE.MeshStandardMaterial({ map: t, roughness: 0.8, metalness: 0.1 }));
  const car = [0xb02a2a, 0x2a5fb0, 0xd8d8d8, 0x2d2d30, 0xc9a534].map(c => new THREE.MeshStandardMaterial({ color: c, roughness: 0.35, metalness: 0.6 }));
  walls(); let sd = 77; const r = () => ((sd = (sd * 16807) % 2147483647) / 2147483647);
  for (let bx = -2; bx <= 2; bx++) for (let bz = -2; bz <= 2; bz++) {
    const cx = bx * 42, cz = bz * 42;
    if (!bx && !bz) { addBox(0, 0, 8, 8, 1, 0, matStruct, true); addBox(0, 0, 2.4, 2.4, 7, 1, matWarm, true); continue; }
    if (r() < 0.12) { addBox(cx, cz, 26, 26, 1.2, 0, matWarm); addBox(cx, cz, 8, 8, 3, 1.2, matStruct); continue; }
    if (r() < 0.5) { addBox(cx - 7, cz, 12, 26, 8 + r() * 22, 0, bm[(r() * 3) | 0], r() < 0.3); addBox(cx + 7.5, cz, 11, 26, 6 + r() * 18, 0, bm[(r() * 3) | 0]); }
    else addBox(cx, cz, 26, 26, 10 + r() * 26, 0, bm[(r() * 3) | 0], r() < 0.4);
  }
  const lanes = [-105, -63, -21, 21, 63, 105];
  for (let i = 0; i < 60; i++) {
    const hz = r() < 0.5, ln = lanes[(r() * 6) | 0] + (r() - 0.5) * 6, al = (r() * 2 - 1) * 125, x = hz ? al : ln, z = hz ? ln : al;
    if (Math.hypot(x, z - 24) < 9) continue;
    addBox(x, z, hz ? 4.5 : 2, hz ? 2 : 4.5, 1.5, 0, car[(r() * 5) | 0]);
  }
}
const matSand = texMat('#b89a66', '#7a5c32', 0.95, 0, false);
function buildDesert() {
  walls(); let sd = 4242; const r = () => ((sd = (sd * 16807) % 2147483647) / 2147483647);
  addBox(0, 0, 10, 10, 2, 0, matSand, true); addBox(0, 0, 3, 3, 6, 2, matWarm, true);
  for (let i = 0; i < 120; i++) {
    const a = r() * 6.283, rad = 18 + r() * 115, x = Math.cos(a) * rad, z = Math.sin(a) * rad;
    if (Math.abs(x) > 132 || Math.abs(z) > 132 || Math.hypot(x, z - 24) < 10) continue;
    if (i % 9 === 0) addBox(x, z, 8 + r() * 10, 1.4, 2.6, 0, matWarm); else { const w = 4 + r() * 14; addBox(x, z, w, w * (0.6 + r() * 0.8), 3 + r() * 12, 0, matSand); }
  }
}
const grassT = ground.material.map;
const asphaltT = noiseTex('#2d3136', '#14171a', 512, false), sandT = noiseTex('#c9a86a', '#8a6a3a', 512, false);
[asphaltT, sandT].forEach(t => t.repeat.set(150, 150));
let curMap = 'base';
function loadMap(name) {
  mapObjs.forEach(m => scene.remove(m)); mapObjs.length = 0; colliders.length = 0; wallMeshes.length = 0;
  holes.forEach(m => scene.remove(m)); holes.length = 0;
  curMap = name; const base = name === 'base';
  pad.visible = decal.visible = decal2.visible = base;
  const t = name === 'city' ? asphaltT : name === 'desert' ? sandT : grassT;
  ground.material.map = t; ground.material.bumpMap = t; ground.material.needsUpdate = true;
  scene.fog.color.setHex(name === 'city' ? 0x8a97a6 : name === 'desert' ? 0xd8b98a : 0xa9c7d6);
  (name === 'city' ? buildCity : name === 'desert' ? buildDesert : buildBase)();
  buildInteractables(name);
  renderer.shadowMap.needsUpdate = true;
}

/* ------------------------------------------------------------------ */
/* Física simple                                                       */
/* ------------------------------------------------------------------ */
const _pos = { x: 0, z: 0 };
function collide(x, z, y, r, h, stepUp) {
  for (const c of colliders) {
    if (c.top <= y + stepUp) continue;
    if (c.bottom >= y + h) continue;
    const cx = clamp(x, c.minX, c.maxX), cz = clamp(z, c.minZ, c.maxZ);
    const dx = x - cx, dz = z - cz, d2 = dx * dx + dz * dz;
    if (d2 < r * r) {
      if (d2 > 1e-8) { const d = Math.sqrt(d2), push = r - d; x += dx / d * push; z += dz / d * push; }
      else {
        const l = x - c.minX, rr = c.maxX - x, t = z - c.minZ, b = c.maxZ - z, m = Math.min(l, rr, t, b);
        if (m === l) x = c.minX - r; else if (m === rr) x = c.maxX + r; else if (m === t) z = c.minZ - r; else z = c.maxZ + r;
      }
    }
  }
  _pos.x = x; _pos.z = z; return _pos;
}
function groundAt(x, z, y, r, stepUp) {
  let g = 0;
  for (const c of colliders) {
    if (c.top <= y + stepUp && c.top > g && x + r > c.minX && x - r < c.maxX && z + r > c.minZ && z - r < c.maxZ) g = c.top;
  }
  return g;
}
function insideAny(x, y, z, r) {
  for (const c of colliders) {
    if (x > c.minX - r && x < c.maxX + r && z > c.minZ - r && z < c.maxZ + r && y > c.bottom - r && y < c.top + r) return c;
  }
  return null;
}

/* ------------------------------------------------------------------ */
/* Audio sintetizado                                                   */
/* ------------------------------------------------------------------ */
let AC = null, master = null, noiseBuf = null;
function ensureAudio() {
  try {
    if (!AC) {
      AC = new (window.AudioContext || window.webkitAudioContext)();
      master = AC.createGain(); master.gain.value = 0.5; master.connect(AC.destination);
      noiseBuf = AC.createBuffer(1, AC.sampleRate * 1.6, AC.sampleRate);
      const d = noiseBuf.getChannelData(0); for (let i = 0; i < d.length; i++) d[i] = Math.random() * 2 - 1;
    }
    if (AC.state === 'suspended') AC.resume();
  } catch (e) { AC = null; }
}
function tone(type, f0, f1, dur, gain, delay) {
  if (!AC) return;
  const t = AC.currentTime + (delay || 0), o = AC.createOscillator(), g = AC.createGain();
  o.type = type; o.frequency.setValueAtTime(f0, t); o.frequency.exponentialRampToValueAtTime(Math.max(20, f1), t + dur);
  g.gain.setValueAtTime(gain, t); g.gain.exponentialRampToValueAtTime(0.0001, t + dur);
  o.connect(g); g.connect(master); o.start(t); o.stop(t + dur + 0.03);
}
function noise(dur, ftype, f0, f1, gain) {
  if (!AC) return;
  const t = AC.currentTime, s = AC.createBufferSource(), f = AC.createBiquadFilter(), g = AC.createGain();
  s.buffer = noiseBuf; f.type = ftype;
  f.frequency.setValueAtTime(f0, t); f.frequency.exponentialRampToValueAtTime(Math.max(20, f1), t + dur);
  g.gain.setValueAtTime(gain, t); g.gain.exponentialRampToValueAtTime(0.0001, t + dur);
  s.connect(f); f.connect(g); g.connect(master); s.start(t, Math.random() * 0.5); s.stop(t + dur + 0.03);
}
const sfx = {
  rifle() { noise(0.12, 'bandpass', 2600, 400, 0.6); noise(0.3, 'lowpass', 900, 80, 0.35); tone('square', 190, 50, 0.09, 0.22); tone('sine', 80, 35, 0.14, 0.4); },
  plasma() { tone('sawtooth', 950, 180, 0.2, 0.22); tone('sine', 1900, 380, 0.16, 0.15); },
  overheat() { noise(0.4, 'highpass', 3000, 6000, 0.25); tone('square', 300, 100, 0.3, 0.12); },
  reload() { tone('square', 260, 200, 0.05, 0.12); tone('square', 420, 300, 0.05, 0.12, 0.55); tone('square', 320, 480, 0.06, 0.14, 1.25); },
  empty() { tone('square', 140, 120, 0.04, 0.12); },
  hit() { tone('triangle', 1100, 700, 0.05, 0.22); },
  head() { tone('triangle', 1700, 1200, 0.08, 0.25); },
  kill() { tone('sawtooth', 500, 90, 0.22, 0.2); noise(0.18, 'lowpass', 1800, 200, 0.25); },
  boom() { noise(0.9, 'lowpass', 1400, 60, 1.0); tone('sine', 110, 28, 0.6, 0.6); },
  throw() { noise(0.12, 'bandpass', 700, 300, 0.2); },
  enemy(d) { const v = clamp(1 - d / 55, 0.04, 0.6) * 0.35; tone('sawtooth', 620, 240, 0.12, v); },
  hurt() { noise(0.14, 'lowpass', 900, 200, 0.5); },
  shieldBreak() { tone('sawtooth', 900, 120, 0.5, 0.3); noise(0.3, 'highpass', 2500, 5000, 0.2); },
  alarm() { tone('square', 880, 880, 0.1, 0.1); tone('square', 660, 660, 0.1, 0.1, 0.14); },
  pickup() { tone('sine', 600, 900, 0.1, 0.2); tone('sine', 900, 1400, 0.12, 0.2, 0.09); },
  wave() { tone('sawtooth', 220, 330, 0.5, 0.18); tone('sawtooth', 330, 440, 0.6, 0.18, 0.3); },
  swap() { tone('square', 500, 700, 0.05, 0.1); },
  shotgun() { noise(0.25, 'bandpass', 1800, 200, 0.9); noise(0.5, 'lowpass', 700, 50, 0.6); tone('sine', 70, 28, 0.25, 0.7); },
  sniper() { noise(0.15, 'highpass', 2000, 6000, 0.5); noise(0.7, 'lowpass', 1200, 50, 0.8); tone('sine', 60, 25, 0.4, 0.8); },
  dash() { noise(0.25, 'bandpass', 400, 2500, 0.3); }
};

/* ------------------------------------------------------------------ */
/* Estado                                                              */
/* ------------------------------------------------------------------ */
let state = 'menu';       // menu | playing | paused | over
let fallback = false;     // sin captura de cursor
let locked = false;
let gt = 0;               // tiempo de juego
const keys = {};
let mouseDown = false;
const mouse = { x: innerWidth / 2, y: innerHeight / 2 };
const EYE = 1.65;
const P = { x: 0, z: 24, y: 0, camY: 0, vx: 0, vz: 0, vy: 0, yaw: 0, pitch: 0, onGround: true, hp: 100, shield: 100, lastHit: -99 };
const WP = {
  cur: 'rifle', grenades: 3, gcd: 0,
  rifle: { mag: 32, magMax: 32, reserve: 128, cd: 0, reloading: 0 },
  plasma: { heat: 0, over: false, cd: 0 },
  shotgun: { mag: 6, magMax: 6, reserve: 36, cd: 0, reloading: 0 },
  sniper: { mag: 5, magMax: 5, reserve: 25, cd: 0, reloading: 0 }
};
let score = 0, best = 0, wave = 0, waveState = 'intermission', waveTimer = 2.5, toSpawn = [], spawnT = 0;
let kick = 0, shake = 0, dmgFlash = 0, alarmT = 0, bobT = 0, hitmT = 0, orbitT = 0;
try { best = parseInt(localStorage.getItem('anillo_best') || '0', 10) || 0; } catch (e) { best = 0; }

const enemies = [], eBolts = [], grenades = [], pickups = [], parts = [], tracers = [], booms = [];

/* ------------------------------------------------------------------ */
/* Armas (modelos en primera persona)                                  */
/* ------------------------------------------------------------------ */
const vm = new THREE.Group(); camera.add(vm);
const gunMat = texMat('#3a4550', '#11151a', 0.4, 0.75, true);
const gunMat2 = texMat('#232b33', '#05070a', 0.5, 0.6, true);
const gunGlow = new THREE.MeshStandardMaterial({ color: 0x5fd3e8, emissive: 0x5fd3e8, emissiveIntensity: 1.4 });
const plasmaGlow = new THREE.MeshStandardMaterial({ color: 0x5fd3e8, emissive: 0x5fd3e8, emissiveIntensity: 1.6 });
function gpart(parent, geo, mat, x, y, z, rx) { const m = new THREE.Mesh(geo, mat); m.position.set(x, y, z); if (rx) m.rotation.x = rx; parent.add(m); return m; }
const rifleM = new THREE.Group();
gpart(rifleM, new THREE.BoxGeometry(0.07, 0.11, 0.5), gunMat, 0, 0, 0);
gpart(rifleM, new THREE.CylinderGeometry(0.018, 0.018, 0.36, 8), gunMat2, 0, 0.02, -0.4, Math.PI / 2);
gpart(rifleM, new THREE.BoxGeometry(0.03, 0.02, 0.3), gunGlow, 0, 0.066, -0.02);
gpart(rifleM, new THREE.BoxGeometry(0.05, 0.16, 0.08), gunMat2, 0, -0.12, 0.06);
gpart(rifleM, new THREE.BoxGeometry(0.06, 0.09, 0.2), gunMat2, 0, -0.01, 0.34);
gpart(rifleM, new THREE.BoxGeometry(0.02, 0.05, 0.03), gunMat2, 0, 0.09, -0.2);
const pistolM = new THREE.Group();
gpart(pistolM, new THREE.BoxGeometry(0.08, 0.11, 0.26), gunMat, 0, 0, 0);
gpart(pistolM, new THREE.CylinderGeometry(0.03, 0.04, 0.22, 10), plasmaGlow, 0, 0.02, -0.22, Math.PI / 2);
gpart(pistolM, new THREE.BoxGeometry(0.06, 0.14, 0.07), gunMat2, 0, -0.11, 0.09);
gpart(pistolM, new THREE.BoxGeometry(0.11, 0.025, 0.1), gunGlow, 0, 0.07, -0.02);
const woodT = (() => { const c = document.createElement('canvas'); c.width = c.height = 128; const g = c.getContext('2d'); g.fillStyle = '#6b4526'; g.fillRect(0, 0, 128, 128);
  for (let i = 0; i < 220; i++) { g.strokeStyle = 'rgba(' + (i % 2 ? '40,22,8' : '150,100,55') + ',' + rand(0.08, 0.3) + ')'; g.lineWidth = rand(0.5, 2); const y = Math.random() * 128; g.beginPath(); g.moveTo(0, y); g.bezierCurveTo(40, y + rand(-4, 4), 80, y + rand(-4, 4), 128, y + rand(-3, 3)); g.stroke(); }
  const t = new THREE.CanvasTexture(c); t.wrapS = t.wrapT = THREE.RepeatWrapping; return t; })();
const woodM = new THREE.MeshStandardMaterial({ map: woodT, bumpMap: woodT, roughness: 0.7 });
const steelM = texMat('#5a6672', '#1a1f24', 0.3, 0.9, false), blackM = texMat('#1c2127', '#000000', 0.55, 0.6, true);
const shotgunM = new THREE.Group();
gpart(shotgunM, new THREE.CylinderGeometry(0.028, 0.028, 0.78, 12), steelM, 0, 0.03, -0.36, Math.PI / 2);
gpart(shotgunM, new THREE.CylinderGeometry(0.022, 0.022, 0.6, 12), blackM, 0, -0.02, -0.3, Math.PI / 2);
gpart(shotgunM, new THREE.BoxGeometry(0.07, 0.1, 0.26), blackM, 0, 0, 0.06);
gpart(shotgunM, new THREE.BoxGeometry(0.06, 0.06, 0.22), woodM, 0, -0.04, -0.36);
gpart(shotgunM, new THREE.BoxGeometry(0.06, 0.11, 0.3), woodM, 0, -0.02, 0.33);
gpart(shotgunM, new THREE.BoxGeometry(0.02, 0.02, 0.02), gunGlow, 0, 0.065, -0.74);
const sniperM = new THREE.Group();
gpart(sniperM, new THREE.CylinderGeometry(0.02, 0.024, 1.0, 12), steelM, 0, 0.02, -0.5, Math.PI / 2);
gpart(sniperM, new THREE.BoxGeometry(0.06, 0.1, 0.45), blackM, 0, 0, 0.05);
gpart(sniperM, new THREE.BoxGeometry(0.055, 0.12, 0.34), woodM, 0, -0.03, 0.4);
gpart(sniperM, new THREE.CylinderGeometry(0.034, 0.034, 0.32, 14), blackM, 0, 0.1, -0.05, Math.PI / 2);
gpart(sniperM, new THREE.CylinderGeometry(0.036, 0.036, 0.02, 14), gunGlow, 0, 0.1, -0.22, Math.PI / 2);
vm.add(rifleM); vm.add(pistolM); vm.add(shotgunM); vm.add(sniperM); shotgunM.visible = sniperM.visible = false;
const gunModels = { rifle: rifleM, plasma: pistolM, shotgun: shotgunM, sniper: sniperM };
const VM_BASE = V3(0.24, -0.24, -0.6);
vm.position.copy(VM_BASE);
pistolM.visible = false;

function glowTex() {
  const c = document.createElement('canvas'); c.width = c.height = 64;
  const g = c.getContext('2d'), gr = g.createRadialGradient(32, 32, 0, 32, 32, 32);
  gr.addColorStop(0, 'rgba(255,255,255,1)'); gr.addColorStop(0.35, 'rgba(255,255,255,.6)'); gr.addColorStop(1, 'rgba(255,255,255,0)');
  g.fillStyle = gr; g.fillRect(0, 0, 64, 64);
  return new THREE.CanvasTexture(c);
}
const muzzle = new THREE.Mesh(new THREE.PlaneGeometry(0.4, 0.4), new THREE.MeshBasicMaterial({ map: glowTex(), color: 0xffd58a, transparent: true, blending: THREE.AdditiveBlending, depthWrite: false }));
muzzle.position.set(0.24, -0.22, -1.05); muzzle.visible = false; camera.add(muzzle);
let muzzleT = 0;
vm.visible = false;

/* ------------------------------------------------------------------ */
/* Partículas, trazadoras                                              */
/* ------------------------------------------------------------------ */
const partGeo = new THREE.BoxGeometry(0.07, 0.07, 0.07), partMats = {};
function pmat(c) { return partMats[c] || (partMats[c] = new THREE.MeshBasicMaterial({ color: c })); }
function spark(pos, color, n, speed, life) {
  for (let i = 0; i < n && parts.length < 380; i++) {
    const m = new THREE.Mesh(partGeo, pmat(color)); m.position.copy(pos); scene.add(m);
    parts.push({ m, vx: rand(-1, 1) * speed, vy: rand(0, 1) * speed, vz: rand(-1, 1) * speed, t: life * rand(0.6, 1.2), l: life });
  }
}
function tracer(a, b, color) {
  const geo = new THREE.BufferGeometry().setFromPoints([a.clone(), b.clone()]);
  const mat = new THREE.LineBasicMaterial({ color, transparent: true, opacity: 0.9 });
  const m = new THREE.Line(geo, mat); scene.add(m); tracers.push({ m, geo, mat, t: 0.07 });
}

/* ------------------------------------------------------------------ */
/* Enemigos                                                            */
/* ------------------------------------------------------------------ */
const TYPES = {
  grunt: { name: 'Vorak', hp: 45, speed: 4.3, dmg: 7, rate: 0.6, range: 26, pspeed: 24, scale: 1, score: 100, color: 0x7a3fc0, glow: 0xff5cf0, radar: '#ff6a5c' },
  brute: { name: 'Vorak Blindado', hp: 230, speed: 2.7, dmg: 16, rate: 1.15, range: 22, pspeed: 20, scale: 1.5, score: 300, color: 0xb8442a, glow: 0xff9a30, radar: '#ff9a30' },
  drone: { name: 'Dron', hp: 70, speed: 5.6, dmg: 6, rate: 0.75, range: 30, pspeed: 26, scale: 1, score: 200, fly: true, color: 0x2f8f9a, glow: 0x50fff0, radar: '#50fff0' },
  titan: { name: 'Titán', hp: 1400, speed: 2.3, dmg: 14, rate: 1.4, range: 34, pspeed: 22, scale: 2.6, score: 2000, boss: true, color: 0xc9a534, glow: 0xff3030, radar: '#ffffff' }
};
Object.values(TYPES).forEach(t => {
  t.matBody = new THREE.MeshStandardMaterial({ color: t.color, roughness: 0.5, metalness: 0.3 });
  t.matDark = new THREE.MeshStandardMaterial({ color: 0x1f242c, roughness: 0.6, metalness: 0.4 });
  t.matGlow = new THREE.MeshStandardMaterial({ color: t.glow, emissive: t.glow, emissiveIntensity: 1.8 });
  t.matBolt = new THREE.MeshBasicMaterial({ color: t.glow });
});

function buildEnemy(key) {
  const t = TYPES[key], g = new THREE.Group();
  const part = (geo, mat, x, y, z, head) => { const m = new THREE.Mesh(geo, mat); m.position.set(x, y, z); m.castShadow = false; if (head) m.userData.head = true; g.add(m); return m; };
  if (t.fly) {
    const body = part(new THREE.SphereGeometry(0.42, 14, 10), t.matBody, 0, 0, 0); body.scale.y = 0.65;
    const rg = part(new THREE.TorusGeometry(0.58, 0.05, 8, 24), t.matGlow, 0, 0, 0); rg.rotation.x = Math.PI / 2;
    part(new THREE.SphereGeometry(0.15, 10, 8), t.matGlow, 0, 0, 0.4);
    part(boxGeo(0.18, 0.18, 0.5), t.matDark, 0.5, -0.05, 0.1);
    part(boxGeo(0.18, 0.18, 0.5), t.matDark, -0.5, -0.05, 0.1);
  } else {
    part(boxGeo(0.28, 0.9, 0.3), t.matDark, 0.2, 0.45, 0);
    part(boxGeo(0.28, 0.9, 0.3), t.matDark, -0.2, 0.45, 0);
    part(boxGeo(0.9, 0.9, 0.55), t.matBody, 0, 1.35, 0);
    part(boxGeo(0.46, 0.46, 0.46), t.matBody, 0, 2.05, 0, true);
    part(boxGeo(0.4, 0.12, 0.06), t.matGlow, 0, 2.08, 0.24, true);
    part(boxGeo(0.22, 0.75, 0.26), t.matBody, 0.62, 1.4, 0.05);
    part(boxGeo(0.22, 0.75, 0.26), t.matBody, -0.62, 1.4, 0.05);
    part(boxGeo(0.16, 0.16, 0.8), t.matDark, 0.62, 1.35, 0.5);
    part(boxGeo(0.1, 0.1, 0.1), t.matGlow, 0.62, 1.35, 0.95);
    part(boxGeo(0.3, 0.3, 0.3), t.matGlow, 0, 1.4, -0.3);
    if (key === 'brute' || key === 'titan') {
      part(boxGeo(0.5, 0.35, 0.65), t.matDark, 0.62, 1.9, 0);
      part(boxGeo(0.5, 0.35, 0.65), t.matDark, -0.62, 1.9, 0);
    }
    if (key === 'titan') {
      for (let i = -1; i <= 1; i++) part(boxGeo(0.14, 0.7, 0.14), t.matGlow, i * 0.3, 2.0, -0.42);
    }
  }
  g.scale.setScalar(t.scale);
  return g;
}

function spawnPos() {
  for (let k = 0; k < 50; k++) {
    const a = Math.random() * 6.283, r = rand(32, 58), x = P.x + Math.cos(a) * r, z = P.z + Math.sin(a) * r;
    if (Math.abs(x) > 135 || Math.abs(z) > 135) continue;
    let bad = false;
    for (const c of colliders) if (x > c.minX - 1.5 && x < c.maxX + 1.5 && z > c.minZ - 1.5 && z < c.maxZ + 1.5) { bad = true; break; }
    if (!bad) return { x, z };
  }
  return { x: -50, z: -50 };
}
function spawnEnemy(key) {
  const t = TYPES[key], p = spawnPos(), g = buildEnemy(key), hp = t.hp * (1 + 0.09 * (wave - 1));
  const e = { key, t, group: g, hp, maxhp: hp, x: p.x, z: p.z, y: t.fly ? rand(3, 5) : 0, fireT: rand(0.9, 2.2), strafe: Math.random() < 0.5 ? -1 : 1, strafeT: rand(1, 3), los: false, losT: 0, detourT: 0, detourDir: 1, stuckT: 0, phase: Math.random() * 6.28, hover: rand(2.6, 4.6), r: 0.5 * t.scale, h: 2.2 * t.scale };
  if (t.fly) { e.r = 0.55; e.h = 0.9; }
  g.userData.enemy = e; g.position.set(e.x, e.y, e.z); scene.add(g); enemies.push(e);
  spark(V3(e.x, e.y + 1, e.z), t.glow, 14, 4, 0.5);
}
function findEnemy(o) { while (o) { if (o.userData && o.userData.enemy) return o.userData.enemy; o = o.parent; } return null; }

const losRay = new THREE.Raycaster();
const _a = V3(0, 0, 0), _b = V3(0, 0, 0);
function hasLOS(e) {
  _a.set(e.x, e.y + (e.t.fly ? 0 : 1.6 * e.t.scale), e.z);
  _b.set(P.x, P.y + 1.3, P.z);
  const len = _a.distanceTo(_b); _b.sub(_a).normalize();
  losRay.set(_a, _b); losRay.far = len;
  return losRay.intersectObjects(wallMeshes, false).length === 0;
}

const boltGeo = new THREE.SphereGeometry(0.16, 8, 6);
function enemyFire(e) {
  const t = e.t, org = V3(e.x, e.y + (t.fly ? 0 : 1.5 * t.scale), e.z);
  const dist = Math.hypot(P.x - e.x, P.z - e.z), tt = dist / t.pspeed * 0.5;
  const tgt = V3(P.x + P.vx * tt, P.y + 1.1, P.z + P.vz * tt);
  const n = t.boss ? 3 : 1;
  for (let i = 0; i < n; i++) {
    const d = tgt.clone().sub(org).normalize(), sp = 0.025 + dist * 0.0016;
    d.x += rand(-sp, sp) + (t.boss ? (i - 1) * 0.09 : 0); d.y += rand(-sp, sp) * 0.6; d.z += rand(-sp, sp); d.normalize();
    const m = new THREE.Mesh(boltGeo, t.matBolt);
    m.position.copy(org).addScaledVector(d, 0.9 * t.scale); m.scale.setScalar(t.scale > 1 ? 1.5 : 1);
    scene.add(m); eBolts.push({ m, v: d.multiplyScalar(t.pspeed), dmg: t.dmg, t: 4, color: t.glow });
  }
  sfx.enemy(dist);
}

function damageEnemy(e, dmg, head, pt) {
  e.hp -= dmg;
  spark(pt, e.t.glow, 5, 3, 0.3);
  if (e.hp <= 0) { killEnemy(e, head); return; }
  showHit(head ? 'head' : '');
  if (head) sfx.head(); else sfx.hit();
}
function killEnemy(e, head) {
  const i = enemies.indexOf(e); if (i < 0) return;
  enemies.splice(i, 1); scene.remove(e.group);
  const c = V3(e.x, e.y + 1.2 * e.t.scale, e.z);
  spark(c, e.t.color, e.t.boss ? 70 : 24, 6, 0.8); spark(c, e.t.glow, e.t.boss ? 40 : 12, 7, 0.6);
  score += e.t.score + (head ? 50 : 0);
  sfx.kill(); showHit('kill');
  const r = Math.random();
  if (e.t.boss) { dropLoot(e, 'health'); dropLoot(e, 'ammo'); dropLoot(e, 'grenade'); }
  else if (r < 0.3) dropLoot(e, 'ammo'); else if (r < 0.46) dropLoot(e, 'health'); else if (r < 0.54) dropLoot(e, 'grenade');
}
function dropLoot(e, type) {
  const col = type === 'ammo' ? 0xffc040 : type === 'health' ? 0x40ff80 : 0xa8ff60;
  const geo = type === 'grenade' ? new THREE.SphereGeometry(0.22, 10, 8) : boxGeo(0.45, 0.45, 0.45);
  const m = new THREE.Mesh(geo, new THREE.MeshStandardMaterial({ color: col, emissive: col, emissiveIntensity: 0.9 }));
  const ox = rand(-1, 1), oz = rand(-1, 1);
  const x = clamp(e.x + ox, -137, 137), z = clamp(e.z + oz, -137, 137);
  const y = groundAt(x, z, e.y, 0.3, 0.6);
  m.position.set(x, y + 0.6, z); scene.add(m);
  pickups.push({ m, type, x, z, y, t: 30, ph: Math.random() * 6 });
}

/* ------------------------------------------------------------------ */
/* Jugador: daño y armas                                               */
/* ------------------------------------------------------------------ */
function hurt(n) {
  if (state !== 'playing') return;
  P.lastHit = gt; let d = n;
  if (P.shield > 0) { const a = Math.min(P.shield, d); P.shield -= a; d -= a; if (P.shield <= 0) sfx.shieldBreak(); }
  if (d > 0) { P.hp -= d; sfx.hurt(); dmgFlash = 1; } else dmgFlash = Math.max(dmgFlash, 0.5);
  shake = Math.max(shake, 0.12);
  if (P.hp <= 0) { P.hp = 0; endGame(); }
}

const raycaster = new THREE.Raycaster();
const _dir = V3(0, 0, -1), _org = V3(0, 0, 0), _muz = V3(0, 0, 0);
function shoot(dmg, spread, color) {
  camera.updateMatrixWorld(true);
  _dir.set(rand(-spread, spread), rand(-spread, spread), -1).normalize().applyQuaternion(camera.quaternion);
  camera.getWorldPosition(_org);
  raycaster.set(_org, _dir); raycaster.far = 250;
  const targets = [ground].concat(wallMeshes);
  for (const e of enemies) targets.push(e.group);
  const hits = raycaster.intersectObjects(targets, true);
  let end;
  if (hits.length) {
    const h = hits[0]; end = h.point.clone();
    const en = findEnemy(h.object);
    if (en) { const head = !!h.object.userData.head; damageEnemy(en, head ? dmg * 2 : dmg, head, h.point); }
    else impact(h, color);
  } else end = _org.clone().addScaledVector(_dir, 120);
  _muz.set(0.24, -0.2, -0.95); camera.localToWorld(_muz);
  tracer(_muz, end, color); netShot(_muz, end, color);
}
const holes = [], holeGeo = new THREE.PlaneGeometry(0.28, 0.28), _n = V3(0, 1, 0);
const holeMat = new THREE.MeshBasicMaterial({ map: (() => { const c = document.createElement('canvas'); c.width = c.height = 64; const g = c.getContext('2d'), gr = g.createRadialGradient(32, 32, 0, 32, 32, 32); gr.addColorStop(0, 'rgba(10,10,10,.95)'); gr.addColorStop(0.35, 'rgba(25,25,25,.55)'); gr.addColorStop(1, 'rgba(0,0,0,0)'); g.fillStyle = gr; g.fillRect(0, 0, 64, 64); return new THREE.CanvasTexture(c); })(), transparent: true, depthWrite: false, polygonOffset: true, polygonOffsetFactor: -4 });
function impact(h, color) {
  spark(h.point, color, 5, 3, 0.25); spark(h.point, 0xa8a8a0, 4, 1.4, 0.9);
  if (!h.face) return;
  _n.copy(h.face.normal).transformDirection(h.object.matrixWorld);
  const m = new THREE.Mesh(holeGeo, holeMat); m.position.copy(h.point).addScaledVector(_n, 0.012); m.lookAt(h.point.clone().add(_n)); m.rotateZ(rand(0, 6.28)); scene.add(m);
  holes.push(m); if (holes.length > 70) scene.remove(holes.shift());
}
function doFlash(color) { muzzle.material.color.setHex(color); muzzle.visible = true; muzzleT = 0.05; muzzle.rotation.z = rand(0, 6); muzzle.scale.setScalar(rand(1.1, 1.8)); flashLight.color.setHex(color); flashLight.intensity = 2.4; }

function startReload() {
  const w = WP[WP.cur];
  if (w.mag === undefined || w.reloading > 0 || w.mag >= w.magMax || w.reserve <= 0) return;
  w.reloading = RT[WP.cur]; sfx.reload();
}
function fireRifle() {
  const w = WP.rifle;
  w.mag--; w.cd = 0.085; kick = 1;
  const moving = Math.hypot(P.vx, P.vz) > 1;
  P.yaw += rand(-0.003, 0.003); casing();
  shoot(13, 0.008 + (moving ? 0.006 : 0) + (P.onGround ? 0 : 0.01), 0xffe0a0);
  doFlash(0xffd58a); sfx.rifle();
  P.pitch += 0.005 + Math.random() * 0.004;
  if (w.mag === 0) startReload();
}
function firePlasma() {
  const w = WP.plasma;
  w.cd = 0.33; w.heat += 0.16; kick = 1.4;
  shoot(34, 0.004, 0x6fe6ff); doFlash(0x6fe6ff); sfx.plasma();
  if (w.heat >= 1) { w.heat = 1; w.over = true; sfx.overheat(); }
}
const RT = { rifle: 1.7, shotgun: 2.2, sniper: 2.6 };
let dashT = 0, dashCd = 0, zoom = false;
function casing() { _muz.set(0.3, -0.18, -0.5); camera.localToWorld(_muz); spark(_muz, 0xd9ab3a, 1, 2.2, 1.4); }
function fireShotgun() {
  const w = WP.shotgun; w.mag--; w.cd = 0.85; kick = 2.4;
  for (let i = 0; i < 8; i++) shoot(11, 0.045, 0xffd08a);
  doFlash(0xffb060); sfx.shotgun(); casing(); P.pitch += 0.03; P.yaw += rand(-0.01, 0.01); shake = Math.max(shake, 0.25);
  if (w.mag === 0) startReload();
}
function fireSniper() {
  const w = WP.sniper; w.mag--; w.cd = 1.1; kick = 2.8;
  shoot(110, 0.0005, 0xfff2c0); doFlash(0xffe6b0); sfx.sniper(); casing(); P.pitch += 0.025; shake = Math.max(shake, 0.2);
  if (w.mag === 0) startReload();
}
function doDash() {
  if (dashCd > 0 || state !== 'playing') return;
  let ix = 0, iz = 0; if (keys.KeyW) iz -= 1; if (keys.KeyS) iz += 1; if (keys.KeyA) ix -= 1; if (keys.KeyD) ix += 1;
  if (!ix && !iz) iz = -1;
  const l = Math.hypot(ix, iz), cy = Math.cos(P.yaw), sy = Math.sin(P.yaw); ix /= l; iz /= l;
  P.vx = (ix * cy + iz * sy) * 26; P.vz = (-ix * sy + iz * cy) * 26; dashT = 0.2; dashCd = 1.6; shake = Math.max(shake, 0.15); sfx.dash();
}
function throwGrenade() {
  if (WP.grenades <= 0 || WP.gcd > 0) return;
  WP.grenades--; WP.gcd = 0.7; sfx.throw();
  const fwd = V3(-Math.sin(P.yaw) * Math.cos(P.pitch), Math.sin(P.pitch), -Math.cos(P.yaw) * Math.cos(P.pitch));
  const m = new THREE.Mesh(new THREE.SphereGeometry(0.14, 10, 8), new THREE.MeshStandardMaterial({ color: 0x4d6b3c, emissive: 0x7dff9a, emissiveIntensity: 0.6 }));
  m.position.set(P.x + fwd.x * 0.6, P.camY + EYE - 0.15 + fwd.y * 0.6, P.z + fwd.z * 0.6); m.castShadow = true; scene.add(m);
  grenades.push({ m, vx: fwd.x * 17 + P.vx * 0.5, vy: fwd.y * 17 + 4.5, vz: fwd.z * 17 + P.vz * 0.5, t: 1.7 });
}
const sphereGeo = new THREE.SphereGeometry(1, 16, 12);
function boom(pos) {
  sfx.boom(); shake = 0.7; boomLight.position.copy(pos); boomLight.intensity = 9;
  const m = new THREE.Mesh(sphereGeo, new THREE.MeshBasicMaterial({ color: 0xffa838, transparent: true, opacity: 0.85 }));
  m.position.copy(pos); m.scale.setScalar(0.5); scene.add(m); booms.push({ m, t: 0 });
  spark(pos, 0xffb040, 30, 9, 0.7); spark(pos, 0x9a9a9a, 12, 5, 0.9);
  const R = 8;
  for (const e of enemies.slice()) {
    const d = Math.hypot(e.x - pos.x, e.y + 1 - pos.y, e.z - pos.z);
    if (d < R) damageEnemy(e, 230 * (1 - d / R) + 20, false, V3(e.x, e.y + 1, e.z));
  }
  const dp = Math.hypot(P.x - pos.x, P.y + 1 - pos.y, P.z - pos.z);
  if (dp < R) hurt(70 * (1 - dp / R));
}
function switchWeapon(w) {
  if (WP.cur === w || state !== 'playing') return;
  WP.cur = w; ['rifle', 'shotgun', 'sniper'].forEach(k => WP[k].reloading = 0); zoom = false; sfx.swap();
  Object.keys(gunModels).forEach(k => gunModels[k].visible = k === w);
  muzzle.position.z = -1.0;
}

/* ------------------------------------------------------------------ */
/* Oleadas                                                             */
/* ------------------------------------------------------------------ */
let bannerTimer = null;
function banner(title, sub, ms) {
  $('bannerTitle').textContent = title; $('bannerSub').textContent = sub || '';
  $('banner').classList.add('show'); clearTimeout(bannerTimer);
  bannerTimer = setTimeout(() => $('banner').classList.remove('show'), ms || 2600);
}
function startWave() {
  wave++;
  const n = wave, list = [];
  let grunts = 3 + n * 2, brutes = n >= 2 ? Math.floor(n / 2) : 0, drones = n >= 3 ? Math.floor((n - 1) / 2) : 0;
  const boss = n % 5 === 0;
  if (boss) { list.push('titan'); grunts = Math.ceil(grunts / 2); }
  for (let i = 0; i < grunts; i++) list.push('grunt');
  for (let i = 0; i < brutes; i++) list.push('brute');
  for (let i = 0; i < drones; i++) list.push('drone');
  for (let i = list.length - 1; i > 0; i--) { const j = (Math.random() * (i + 1)) | 0; [list[i], list[j]] = [list[j], list[i]]; }
  toSpawn = list; waveState = 'active'; spawnT = 0.5;
  banner('OLEADA ' + n, boss ? 'ALERTA: TITÁN VORAK EN EL SECTOR' : 'Los Vorak se aproximan', 2800);
  sfx.wave();
}
function endWave() {
  waveState = 'intermission'; waveTimer = 6;
  const bonus = 250 * wave; score += bonus;
  P.hp = Math.min(100, P.hp + 25);
  WP.rifle.reserve = Math.min(240, WP.rifle.reserve + 50);
  WP.grenades = Math.min(5, WP.grenades + 1);
  banner('OLEADA ' + wave + ' SUPERADA', '+' + bonus + ' puntos · suministros recibidos', 3200);
}
function updateWaves(dt) {
  if (waveState === 'intermission') { waveTimer -= dt; if (waveTimer <= 0) startWave(); return; }
  const maxAlive = Math.min(22, 8 + Math.floor(wave * 1.5));
  if (toSpawn.length && enemies.length < maxAlive) {
    spawnT -= dt;
    if (spawnT <= 0) { spawnEnemy(toSpawn.pop()); spawnT = Math.max(0.5, 1.4 - wave * 0.05); }
  }
  if (!toSpawn.length && enemies.length === 0) endWave();
}

/* ------------------------------------------------------------------ */
/* Actualización                                                       */
/* ------------------------------------------------------------------ */
function updatePlayer(dt) {
  // Mirada
  if (fallback) {
    const dz = 0.14, nx = (mouse.x / innerWidth - 0.5) * 2, ny = (mouse.y / innerHeight - 0.5) * 2;
    if (Math.abs(nx) > dz) P.yaw -= Math.sign(nx) * (Math.abs(nx) - dz) / (1 - dz) * 2.6 * dt;
    if (Math.abs(ny) > dz) P.pitch -= Math.sign(ny) * (Math.abs(ny) - dz) / (1 - dz) * 1.5 * dt;
  }
  if (keys.ArrowLeft) P.yaw += 2.2 * dt;
  if (keys.ArrowRight) P.yaw -= 2.2 * dt;
  if (keys.ArrowUp) P.pitch += 1.5 * dt;
  if (keys.ArrowDown) P.pitch -= 1.5 * dt;
  P.pitch = clamp(P.pitch, -1.45, 1.45);

  // Movimiento
  let ix = 0, iz = 0;
  if (keys.KeyW) iz -= 1; if (keys.KeyS) iz += 1; if (keys.KeyA) ix -= 1; if (keys.KeyD) ix += 1;
  const len = Math.hypot(ix, iz); if (len > 0) { ix /= len; iz /= len; }
  const sp = (keys.ShiftLeft || keys.ShiftRight) ? 10.5 : 7;
  const cy = Math.cos(P.yaw), sy = Math.sin(P.yaw);
  const tx = (ix * cy + iz * sy) * sp, tz = (-ix * sy + iz * cy) * sp;
  const k = 1 - Math.exp(-(P.onGround ? 14 : 3) * dt);
  if (dashT > 0) dashT -= dt; else { P.vx += (tx - P.vx) * k; P.vz += (tz - P.vz) * k; }
  let nx = P.x + P.vx * dt, nz = P.z + P.vz * dt;
  const c = collide(nx, nz, P.y, 0.4, 1.8, 0.6);
  P.x = clamp(c.x, -138.5, 138.5); P.z = clamp(c.z, -138.5, 138.5);

  if (keys.Space && P.onGround) { P.vy = 9; P.onGround = false; }
  P.vy -= 24 * dt; P.y += P.vy * dt;
  const g = groundAt(P.x, P.z, P.y, 0.3, 0.6);
  if (P.y <= g) { P.y = g; P.vy = 0; P.onGround = true; } else P.onGround = false;
  const dcy = P.y - P.camY; P.camY += dcy * Math.min(1, dt * 22);

  // Escudo
  if (gt - P.lastHit > 4 && P.shield < 100) P.shield = Math.min(100, P.shield + 28 * dt);
  if (P.shield <= 0 && P.hp > 0) { alarmT -= dt; if (alarmT <= 0) { sfx.alarm(); alarmT = 1.1; } }

  // Armas
  const R = WP.rifle, PL = WP.plasma;
  R.cd -= dt; PL.cd -= dt; WP.gcd -= dt; dashCd -= dt;
  ['shotgun', 'sniper'].forEach(k => { const w = WP[k]; w.cd -= dt; if (w.reloading > 0) { w.reloading -= dt; if (w.reloading <= 0) { w.reloading = 0; const tk = Math.min(w.magMax - w.mag, w.reserve); w.mag += tk; w.reserve -= tk; } } });
  if (R.reloading > 0) { R.reloading -= dt; if (R.reloading <= 0) { R.reloading = 0; const take = Math.min(R.magMax - R.mag, R.reserve); R.mag += take; R.reserve -= take; } }
  PL.heat = Math.max(0, PL.heat - (PL.over ? 0.28 : 0.4) * dt);
  if (PL.over && PL.heat < 0.35) PL.over = false;
  if (mouseDown && (WP.cur === 'shotgun' || WP.cur === 'sniper')) {
    const w = WP[WP.cur];
    if (w.reloading <= 0 && w.cd <= 0) { if (w.mag > 0) (WP.cur === 'shotgun' ? fireShotgun : fireSniper)(); else if (w.reserve > 0) startReload(); else { w.cd = 0.3; sfx.empty(); } }
  } else if (mouseDown) {
    if (WP.cur === 'rifle') {
      if (R.reloading <= 0 && R.cd <= 0) { if (R.mag > 0) fireRifle(); else if (R.reserve > 0) startReload(); else { R.cd = 0.3; sfx.empty(); } }
    } else if (PL.cd <= 0 && !PL.over) firePlasma();
    else if (PL.over && PL.cd <= 0) { PL.cd = 0.3; sfx.empty(); }
  }
  if (muzzleT > 0) { muzzleT -= dt; if (muzzleT <= 0) muzzle.visible = false; }
  flashLight.intensity = Math.max(0, flashLight.intensity - 40 * dt);
  kick = Math.max(0, kick - dt * 9);

  // Cámara
  const moveAmt = P.onGround ? Math.min(1, Math.hypot(P.vx, P.vz) / 7) : 0;
  bobT += dt * moveAmt * 9;
  shake = Math.max(0, shake - dt * 2.5);
  const sk = shake * 0.12;
  camera.position.set(P.x + rand(-sk, sk), P.camY + EYE + Math.sin(bobT * 2) * 0.03 * moveAmt + rand(-sk, sk), P.z + rand(-sk, sk));
  camera.rotation.set(P.pitch + rand(-sk, sk) * 0.4, P.yaw, 0, 'YXZ');

  // Arma en pantalla
  let dip = 0; const cw = WP[WP.cur]; if (cw.reloading > 0) dip = Math.sin((1 - cw.reloading / RT[WP.cur]) * Math.PI);
  vm.position.set(VM_BASE.x + Math.sin(bobT) * 0.008 * moveAmt, VM_BASE.y + Math.abs(Math.cos(bobT)) * -0.008 * moveAmt - dip * 0.22, VM_BASE.z + kick * 0.05);
  vm.rotation.set(kick * 0.05 + dip * 0.7, 0, dip * -0.2);
  plasmaGlow.emissive.setHex(PL.over ? 0xff3010 : PL.heat > 0.6 ? 0xffa030 : 0x5fd3e8);
  const scoped = zoom && WP.cur === 'sniper', zt = scoped ? 22 : 75 + (dashT > 0 ? 10 : 0);
  if (Math.abs(camera.fov - zt) > 0.1) { camera.fov += (zt - camera.fov) * Math.min(1, dt * 14); camera.updateProjectionMatrix(); }
  vm.visible = !(scoped && camera.fov < 45); $('scope').classList.toggle('on', scoped && camera.fov < 45);
}

function updateEnemies(dt) {
  for (let i = 0; i < enemies.length; i++) {
    const e = enemies[i], t = e.t;
    const dx = P.x - e.x, dz = P.z - e.z, dist = Math.hypot(dx, dz) || 0.001;
    const nx = dx / dist, nz = dz / dist;
    e.strafeT -= dt; if (e.strafeT <= 0) { e.strafe = Math.random() < 0.5 ? -1 : 1; e.strafeT = rand(1.2, 3.2); }
    e.losT -= dt; if (e.losT <= 0) { e.los = hasLOS(e); e.losT = 0.3; }
    const pref = t.range * 0.55;
    let fwd = dist > pref ? 1 : dist < pref * 0.5 ? -1 : 0;
    if (!e.los && dist > 4) fwd = 1;
    let vx = nx * fwd + (-nz) * e.strafe * 0.7, vz = nz * fwd + nx * e.strafe * 0.7;
    if (e.detourT > 0) { e.detourT -= dt; const a = 1.2 * e.detourDir, ca = Math.cos(a), sa = Math.sin(a); const rx = vx * ca - vz * sa, rz = vx * sa + vz * ca; vx = rx; vz = rz; }
    const vl = Math.hypot(vx, vz); if (vl > 1) { vx /= vl; vz /= vl; }
    const ox = e.x, oz = e.z;
    const c = collide(e.x + vx * t.speed * dt, e.z + vz * t.speed * dt, e.y, e.r, e.h, 0.6);
    e.x = clamp(c.x, -137, 137); e.z = clamp(c.z, -137, 137);
    // atasco
    const want = vl * t.speed * dt, moved = Math.hypot(e.x - ox, e.z - oz);
    if (want > 0.001 && moved < want * 0.3) { e.stuckT += dt; if (e.stuckT > 0.5) { e.detourT = 1.5; e.detourDir = Math.random() < 0.5 ? -1 : 1; e.stuckT = 0; } } else e.stuckT = 0;
    // separación
    for (let j = i + 1; j < enemies.length; j++) {
      const o = enemies[j], sx = e.x - o.x, sz = e.z - o.z, sd = Math.hypot(sx, sz), min = (e.r + o.r) * 1.2;
      if (sd > 0.001 && sd < min && Math.abs(e.y - o.y) < 2) { const push = (min - sd) * 0.5; e.x += sx / sd * push; e.z += sz / sd * push; o.x -= sx / sd * push; o.z -= sz / sd * push; }
    }
    // altura
    if (t.fly) e.y += (e.hover + Math.sin(gt * 2 + e.phase) * 0.6 - e.y) * Math.min(1, dt * 2);
    else e.y = groundAt(e.x, e.z, e.y, e.r * 0.5, 0.6);
    e.group.position.set(e.x, e.y, e.z);
    e.group.rotation.y = Math.atan2(dx, dz);
    // disparo
    e.fireT -= dt;
    if (e.fireT <= 0 && dist < t.range * 1.25 && e.los) { enemyFire(e); e.fireT = t.rate * rand(0.8, 1.4); }
  }
}

function updateProjectiles(dt) {
  // balas enemigas
  for (let i = eBolts.length - 1; i >= 0; i--) {
    const b = eBolts[i]; b.t -= dt; b.m.position.addScaledVector(b.v, dt);
    const p = b.m.position; let remove = false;
    if (b.t <= 0 || p.y < 0.05 || Math.abs(p.x) > 142 || Math.abs(p.z) > 142) remove = true;
    else if (insideAny(p.x, p.y, p.z, 0.12)) { spark(p, b.color, 5, 3, 0.25); remove = true; }
    else {
      const dx = p.x - P.x, dz = p.z - P.z;
      if (dx * dx + dz * dz < 0.36 && p.y > P.y - 0.1 && p.y < P.y + 1.9) { hurt(b.dmg); remove = true; }
    }
    if (remove) { scene.remove(b.m); eBolts.splice(i, 1); }
  }
  // granadas
  for (let i = grenades.length - 1; i >= 0; i--) {
    const g = grenades[i], p = g.m.position; g.t -= dt; g.vy -= 22 * dt;
    let nx = p.x + g.vx * dt; if (Math.abs(nx) > 140 || insideAny(nx, p.y, p.z, 0.14)) g.vx *= -0.45; else p.x = nx;
    let nz = p.z + g.vz * dt; if (Math.abs(nz) > 140 || insideAny(p.x, p.y, nz, 0.14)) g.vz *= -0.45; else p.z = nz;
    let ny = p.y + g.vy * dt; const c = insideAny(p.x, ny, p.z, 0.14);
    if (ny < 0.14 || c) {
      if (ny < 0.14) ny = 0.14; else if (g.vy < 0) ny = c.top + 0.14; else ny = p.y;
      g.vy = -g.vy * 0.4; if (Math.abs(g.vy) < 1.5) g.vy = 0; g.vx *= 0.82; g.vz *= 0.82;
    }
    p.y = ny;
    if (g.t <= 0) { boom(p.clone()); scene.remove(g.m); grenades.splice(i, 1); }
  }
  // explosiones
  for (let i = booms.length - 1; i >= 0; i--) {
    const b = booms[i]; b.t += dt; const k = b.t / 0.4;
    b.m.scale.setScalar(0.5 + k * 7); b.m.material.opacity = 0.85 * (1 - k);
    if (k >= 1) { scene.remove(b.m); b.m.material.dispose(); booms.splice(i, 1); }
  }
  boomLight.intensity = Math.max(0, boomLight.intensity - 30 * dt);
  // trazadoras
  for (let i = tracers.length - 1; i >= 0; i--) {
    const tr = tracers[i]; tr.t -= dt; tr.mat.opacity = Math.max(0, tr.t / 0.07);
    if (tr.t <= 0) { scene.remove(tr.m); tr.geo.dispose(); tr.mat.dispose(); tracers.splice(i, 1); }
  }
  // partículas
  for (let i = parts.length - 1; i >= 0; i--) {
    const q = parts[i]; q.t -= dt; q.vy -= 15 * dt;
    q.m.position.x += q.vx * dt; q.m.position.y += q.vy * dt; q.m.position.z += q.vz * dt;
    if (q.m.position.y < 0.04) { q.m.position.y = 0.04; q.vy = 0; q.vx *= 0.5; q.vz *= 0.5; }
    q.m.scale.setScalar(Math.max(0.05, q.t / q.l));
    if (q.t <= 0) { scene.remove(q.m); parts.splice(i, 1); }
  }
  // pickups
  for (let i = pickups.length - 1; i >= 0; i--) {
    const k = pickups[i]; k.t -= dt;
    k.m.rotation.y += dt * 2; k.m.position.y = k.y + 0.6 + Math.sin(gt * 3 + k.ph) * 0.12;
    const near = Math.hypot(k.x - P.x, k.z - P.z) < 1.6 && Math.abs(k.y - P.y) < 2;
    if (near) {
      if (k.type === 'ammo') { WP.rifle.reserve = Math.min(240, WP.rifle.reserve + 40); WP.shotgun.reserve = Math.min(60, WP.shotgun.reserve + 8); WP.sniper.reserve = Math.min(30, WP.sniper.reserve + 5); }
      else if (k.type === 'health') { P.hp = Math.min(100, P.hp + 35); }
      else WP.grenades = Math.min(5, WP.grenades + 1);
      sfx.pickup(); scene.remove(k.m); pickups.splice(i, 1);
    } else if (k.t <= 0) { scene.remove(k.m); pickups.splice(i, 1); }
  }
}

/* ------------------------------------------------------------------ */
/* HUD                                                                 */
/* ------------------------------------------------------------------ */
const el = { shield: $('shieldFill'), health: $('healthFill'), score: $('score'), best: $('best'), wave: $('waveInfo'), ammo: $('ammo'), wname: $('wname'), heat: $('heatFill'), heatWrap: $('heatWrap'), gren: $('gren'), reload: $('reloadTxt'), vig: $('vignette'), boss: $('bossWrap'), bossFill: $('bossFill'), hitm: $('hitm') };
const rctx = $('radar').getContext('2d');
function showHit(kind) { el.hitm.className = 'on ' + kind; hitmT = 0.14; }
function updateHUD(dt) {
  el.shield.style.width = P.shield + '%'; el.health.style.width = P.hp + '%';
  el.health.classList.toggle('low', P.hp < 35);
  el.vig.classList.toggle('low', P.hp < 35);
  dmgFlash = Math.max(0, dmgFlash - dt * 2.2);
  el.vig.style.opacity = P.hp < 35 ? '' : String(dmgFlash);
  if (P.hp >= 35) el.vig.style.opacity = String(dmgFlash);
  el.score.textContent = score; el.best.textContent = Math.max(best, score);
  const left = enemies.length + toSpawn.length;
  el.wave.textContent = waveState === 'intermission' ? (wave === 0 ? 'PREPÁRATE' : 'SIGUIENTE OLEADA EN ' + Math.ceil(waveTimer)) : 'OLEADA ' + wave + '  ·  HOSTILES ' + left;
  if (WP.cur === 'shotgun' || WP.cur === 'sniper') {
    const w = WP[WP.cur]; el.wname.textContent = WP.cur === 'shotgun' ? 'ESCOPETA' : 'FRANCOTIRADOR'; el.heatWrap.classList.add('hidden');
    el.ammo.innerHTML = w.mag + ' <small>/ ' + w.reserve + '</small>';
    el.reload.textContent = w.reloading > 0 ? 'RECARGANDO' : (w.mag === 0 && w.reserve === 0 ? 'SIN MUNICIÓN' : '');
  } else if (WP.cur === 'rifle') {
    el.wname.textContent = 'RIFLE DE ASALTO'; el.heatWrap.classList.add('hidden');
    el.ammo.innerHTML = WP.rifle.mag + ' <small>/ ' + WP.rifle.reserve + '</small>';
    el.reload.textContent = WP.rifle.reloading > 0 ? 'RECARGANDO' : (WP.rifle.mag === 0 && WP.rifle.reserve === 0 ? 'SIN MUNICIÓN · CAMBIA DE ARMA' : '');
  } else {
    el.wname.textContent = 'PISTOLA DE PLASMA'; el.heatWrap.classList.remove('hidden');
    el.ammo.innerHTML = WP.plasma.over ? 'SOBRECALENTADA' : 'CARGA <small>∞</small>';
    el.ammo.style.fontSize = WP.plasma.over ? '20px' : '';
    el.heat.style.width = (WP.plasma.heat * 100) + '%';
    el.reload.textContent = WP.plasma.over ? 'ENFRIANDO' : '';
  }
  if (WP.cur !== 'plasma') el.ammo.style.fontSize = '';
  if (remote.size) el.wave.textContent += '  ·  ALIADOS ' + remote.size;
  el.gren.textContent = ('● '.repeat(WP.grenades).trim() || '—') + '   DASH ' + (dashCd > 0 ? Math.ceil(dashCd) + 's' : 'LISTO');
  const boss = enemies.find(e => e.t.boss);
  el.boss.classList.toggle('hidden', !boss); if (boss) el.bossFill.style.width = clamp(boss.hp / boss.maxhp, 0, 1) * 100 + '%';
  if (hitmT > 0) { hitmT -= dt; if (hitmT <= 0) el.hitm.className = ''; }
  drawRadar();
}
function drawRadar() {
  const s = 150, c = s / 2, R = 68, range = 45;
  rctx.clearRect(0, 0, s, s);
  rctx.fillStyle = 'rgba(6,20,26,.62)'; rctx.beginPath(); rctx.arc(c, c, R + 4, 0, 7); rctx.fill();
  rctx.strokeStyle = 'rgba(95,211,232,.55)'; rctx.lineWidth = 1.5; rctx.beginPath(); rctx.arc(c, c, R + 4, 0, 7); rctx.stroke();
  rctx.strokeStyle = 'rgba(95,211,232,.22)'; rctx.lineWidth = 1;
  rctx.beginPath(); rctx.arc(c, c, R * 0.5, 0, 7); rctx.moveTo(c - R, c); rctx.lineTo(c + R, c); rctx.moveTo(c, c - R); rctx.lineTo(c, c + R); rctx.stroke();
  const cy = Math.cos(P.yaw), sy = Math.sin(P.yaw);
  for (const e of enemies) {
    const dx = e.x - P.x, dz = e.z - P.z, lx = dx * cy - dz * sy, lf = -(dx * sy + dz * cy);
    let rx = lx / range * R, ry = -lf / range * R; const d = Math.hypot(rx, ry);
    if (d > R) { rx *= R / d; ry *= R / d; }
    rctx.fillStyle = e.t.radar; rctx.beginPath(); rctx.arc(c + rx, c + ry, e.t.boss ? 5.5 : 3.5, 0, 7); rctx.fill();
  }
  rctx.fillStyle = '#e6f4f8'; rctx.beginPath(); rctx.moveTo(c, c - 7); rctx.lineTo(c + 5, c + 5); rctx.lineTo(c - 5, c + 5); rctx.closePath(); rctx.fill();
}

/* ------------------------------------------------------------------ */
/* Flujo del juego                                                     */
/* ------------------------------------------------------------------ */
function clearWorld() {
  [enemies, eBolts, grenades, pickups, parts, tracers, booms].forEach(arr => {
    arr.forEach(o => { const m = o.group || o.m; if (m) scene.remove(m); });
    arr.length = 0;
  });
}
function resetGame() {
  clearWorld();
  Object.assign(P, { x: 0, z: 24, y: 0, camY: 0, vx: 0, vz: 0, vy: 0, yaw: 0, pitch: 0, onGround: true, hp: 100, shield: 100, lastHit: -99 });
  Object.assign(WP.rifle, { mag: 32, reserve: 128, cd: 0, reloading: 0 });
  Object.assign(WP.plasma, { heat: 0, over: false, cd: 0 });
  Object.assign(WP.shotgun, { mag: 6, reserve: 36, cd: 0, reloading: 0 }); Object.assign(WP.sniper, { mag: 5, reserve: 25, cd: 0, reloading: 0 });
  WP.grenades = 3; WP.gcd = 0; WP.cur = 'rifle'; Object.keys(gunModels).forEach(k => gunModels[k].visible = k === 'rifle'); muzzle.position.z = -1.05; dashT = dashCd = 0; zoom = false;
  score = 0; wave = 0; waveState = 'intermission'; waveTimer = 2.5; toSpawn = []; kick = 0; shake = 0; dmgFlash = 0;
  Object.keys(keys).forEach(k => keys[k] = false); mouseDown = false;
}
function requestLock() {
  try {
    const r = renderer.domElement.requestPointerLock();
    if (r && r.catch) r.catch(() => { fallback = true; syncHint(); });
  } catch (e) { fallback = true; syncHint(); }
}
function syncHint() { $('hintLook').classList.toggle('hidden', !(fallback && state === 'playing')); }
function showOverlay(kind) {
  const ov = $('overlay'), txt = $('ovText'), st = $('ovStats'), btn = $('btnPlay'), note = $('ovNote');
  ov.classList.remove('hidden'); $('hud').classList.add('hidden'); $('menuOpts').classList.toggle('hidden', kind === 'pause');
  $('pauseActions').classList.toggle('hidden', kind !== 'pause');
  if (kind === 'menu') {
    txt.textContent = 'Los Vorak abrieron una brecha en el Anillo y solo queda tu pelotón en la Base Alfa. Aguanta oleada tras oleada, sobrevive a los Titanes y no dejes que caiga tu escudo.';
    st.classList.add('hidden'); btn.textContent = 'JUGAR';
  } else if (kind === 'pause') {
    txt.textContent = 'Partida en pausa.'; st.classList.add('hidden'); btn.textContent = 'CONTINUAR';
  } else {
    txt.textContent = 'La Base Alfa ha caído. Otro pelotón tomará tu lugar.';
    st.classList.remove('hidden');
    st.innerHTML = '<div><b>' + score + '</b>puntaje</div><div><b>' + wave + '</b>oleada alcanzada</div><div><b>' + best + '</b>récord</div>';
    btn.textContent = 'JUGAR DE NUEVO';
  }
  note.classList.toggle('hidden', !fallback);
  if (fallback) note.textContent = 'Tu navegador no permite capturar el cursor aquí: gira moviendo el mouse hacia los bordes de la pantalla o con las flechas del teclado.';
  btn.focus();
}
function startGame() {
  ensureAudio(); if (curMap !== selMap) loadMap(selMap); resetGame();
  state = 'playing'; $('overlay').classList.add('hidden'); $('hud').classList.remove('hidden'); vm.visible = true;
  requestLock(); syncHint();
}
function pauseGame() {
  if (state !== 'playing') return;
  state = 'paused'; mouseDown = false; syncHint();
  try { document.exitPointerLock(); } catch (e) {}
  showOverlay('pause');
}
function resumeGame() {
  ensureAudio(); state = 'playing'; $('overlay').classList.add('hidden'); $('hud').classList.remove('hidden'); requestLock(); syncHint();
}
function goToMainMenu() {
  state='menu'; mouseDown=false; vm.visible=false;
  try { document.exitPointerLock(); } catch(e) {}
  Object.keys(keys).forEach(k=>keys[k]=false);
  leaveRoom(); clearWorld(); loadMap(selMap); showOverlay('menu');
}
function endGame() {
  state = 'over'; mouseDown = false; vm.visible = false; syncHint();
  if (score > best) { best = score; try { localStorage.setItem('anillo_best', String(best)); } catch (e) {} }
  try { document.exitPointerLock(); } catch (e) {}
  setTimeout(() => showOverlay('over'), 700);
}

/* ------------------------------------------------------------------ */
/* Entrada                                                             */
/* ------------------------------------------------------------------ */
$('btnPlay').addEventListener('click', () => { if (state === 'paused') resumeGame(); else startGame(); });
$('btnMainMenu').addEventListener('click', goToMainMenu);
document.addEventListener('pointerlockchange', () => {
  locked = document.pointerLockElement === renderer.domElement;
  if (!locked && state === 'playing' && !fallback) pauseGame();
});
document.addEventListener('pointerlockerror', () => { fallback = true; syncHint(); });
window.addEventListener('keydown', ev => {
  if (['Space', 'ArrowUp', 'ArrowDown', 'ArrowLeft', 'ArrowRight'].includes(ev.code) && state === 'playing') ev.preventDefault();
  if (ev.repeat) { keys[ev.code] = true; return; }
  keys[ev.code] = true;
  if (state === 'playing') {
    if (ev.code === 'KeyR') startReload();
    else if (ev.code === 'KeyG') throwGrenade();
    else if (ev.code === 'Digit1') switchWeapon('rifle');
    else if (ev.code === 'Digit2') switchWeapon('plasma');
    else if (ev.code === 'Digit3') switchWeapon('shotgun');
    else if (ev.code === 'Digit4') switchWeapon('sniper');
    else if (ev.code === 'KeyQ') doDash();
    else if (ev.code === 'KeyE') useInteractable(getNearestInteractable());
    else if (ev.code === 'KeyP' || (ev.code === 'Escape' && fallback)) pauseGame();
  } else if (state === 'paused' && ev.code === 'KeyP') resumeGame();
});
window.addEventListener('keyup', ev => { keys[ev.code] = false; });
window.addEventListener('blur', () => { Object.keys(keys).forEach(k => keys[k] = false); mouseDown = false; if (state === 'playing') pauseGame(); });
window.addEventListener('mousemove', ev => {
  mouse.x = ev.clientX; mouse.y = ev.clientY;
  if (locked && state === 'playing') { const sn = 0.0022 * (camera.fov / 75); P.yaw -= ev.movementX * sn; P.pitch = clamp(P.pitch - ev.movementY * sn, -1.45, 1.45); }
});
renderer.domElement.addEventListener('mousedown', ev => { if (state !== 'playing') return; if (ev.button === 0) mouseDown = true; else if (ev.button === 2) zoom = true; });
window.addEventListener('mouseup', ev => { if (ev.button === 0) mouseDown = false; if (ev.button === 2) zoom = false; });
window.addEventListener('wheel', ev => { if (state === 'playing') { const o = ['rifle', 'plasma', 'shotgun', 'sniper']; switchWeapon(o[(o.indexOf(WP.cur) + (ev.deltaY > 0 ? 1 : 3)) % 4]); }; }, { passive: true });
window.addEventListener('contextmenu', ev => ev.preventDefault());
window.addEventListener('resize', () => { camera.aspect = innerWidth / innerHeight; camera.updateProjectionMatrix(); applyRenderQuality(); renderer.shadowMap.needsUpdate = true; });

/* ------------------------------------------------------------------ */
/* Bucle principal                                                     */
/* ------------------------------------------------------------------ */
/* Multijugador sin API externa: BroadcastChannel local + WebRTC P2P manual */
let netT=0, myName='Soldado';
const remote=new Map();
const netId=(window.crypto&&crypto.randomUUID)?crypto.randomUUID():'p'+Math.random().toString(36).slice(2);
let bcRoom=null, p2pPC=null, p2pDC=null;
const RTC_CONFIG={iceServers:[
  {urls:'stun:stun.l.google.com:19302'},
  {urls:'stun:stun1.l.google.com:19302'}
]};
const avBody=new THREE.MeshStandardMaterial({color:0x2f7fd0,roughness:0.5,metalness:0.3});
const avGlow=new THREE.MeshStandardMaterial({color:0x7dffb0,emissive:0x7dffb0,emissiveIntensity:1.6});
function buildAvatar(label){
  const g=new THREE.Group(),dk=TYPES.grunt.matDark;
  const add=(geo,mat,x,y,z)=>{const m=new THREE.Mesh(geo,mat);m.position.set(x,y,z);g.add(m);};
  add(boxGeo(0.3,0.9,0.3),dk,0.2,0.45,0);add(boxGeo(0.3,0.9,0.3),dk,-0.2,0.45,0);add(boxGeo(0.9,0.9,0.55),avBody,0,1.35,0);add(boxGeo(0.46,0.46,0.46),avBody,0,2.05,0);
  add(boxGeo(0.4,0.12,0.06),avGlow,0,2.08,-0.24);add(boxGeo(0.2,0.2,0.8),dk,0.6,1.35,-0.45);
  const c=document.createElement('canvas');c.width=256;c.height=64;const x=c.getContext('2d');x.font='700 34px sans-serif';x.textAlign='center';x.fillStyle='#7dffb0';x.strokeStyle='#000';x.lineWidth=5;x.strokeText(label,128,44);x.fillText(label,128,44);
  const sp=new THREE.Sprite(new THREE.SpriteMaterial({map:new THREE.CanvasTexture(c),depthTest:false,transparent:true}));sp.scale.set(2.4,0.6,1);sp.position.y=2.9;g.add(sp);return g;
}
function setStatus(t){$('mpStatus').textContent=t;}
function removeRemote(id){const r=remote.get(id);if(r){scene.remove(r.g);remote.delete(id);}}
function applyRemotePeer(id,pr){
  if(!id||id===netId||!pr||pr.m!==curMap||typeof pr.x!=='number')return;
  if(pr.on!==1){removeRemote(id);return;}
  let r=remote.get(id);
  if(!r){const g=buildAvatar(String(pr.n||'ALIADO').toUpperCase());g.position.set(pr.x,pr.y||0,pr.z);scene.add(g);r={g};remote.set(id,r);}
  r.x=pr.x;r.y=pr.y||0;r.z=pr.z;r.ty=pr.yaw||0;r.last=performance.now();
}
async function closeBroadcast(){if(!bcRoom)return;try{bcRoom.postMessage({type:'bye',id:netId});}catch(e){}try{bcRoom.close();}catch(e){}bcRoom=null;}
function closeP2P(){
  if(p2pDC){try{p2pDC.close();}catch(e){}p2pDC=null;}
  if(p2pPC){try{p2pPC.close();}catch(e){}p2pPC=null;}
  removeRemote('rtc');
}
async function leaveRoom(){await closeBroadcast();closeP2P();for(const id of Array.from(remote.keys()))removeRemote(id);setStatus('Desconectado.');}
function openBroadcast(code){
  if(!('BroadcastChannel' in window))return false;
  try{
    bcRoom=new BroadcastChannel('anillo-room-'+code);
    bcRoom.onmessage=ev=>{const m=ev.data||{};if(m.id===netId)return;
      if(m.type==='hello'){bcRoom.postMessage({type:'state',id:netId,state:{on:state==='playing'?1:0,x:P.x,y:P.y,z:P.z,yaw:P.yaw,m:curMap,n:myName}});}
      else if(m.type==='state')applyRemotePeer('bc:'+m.id,m.state);
      else if(m.type==='shot'&&m.a&&m.b)tracer(V3(m.a[0],m.a[1],m.a[2]),V3(m.b[0],m.b[1],m.b[2]),m.c||0xffe0a0);
      else if(m.type==='bye')removeRemote('bc:'+m.id);
    };
    bcRoom.postMessage({type:'hello',id:netId});
    setStatus('Sala local activa · abre el HTML en otra pestaña del mismo navegador y usa el mismo código.');
    return true;
  }catch(e){bcRoom=null;return false;}
}
async function joinRoom(){
  await leaveRoom();
  myName=($('mpName').value||'Soldado').trim().slice(0,12)||'Soldado';
  const code=($('mpCode').value||'alfa').toLowerCase().replace(/[^a-z0-9]/g,'').slice(0,12)||'alfa';
  if(!openBroadcast(code))setStatus('Sala local no disponible. Usa P2P para conectar dos equipos.');
}
function waitForIce(pc){
  if(pc.iceGatheringState==='complete')return Promise.resolve();
  return new Promise(resolve=>{
    const started=Date.now();
    const done=()=>{if(pc.iceGatheringState==='complete'||Date.now()-started>8000){pc.removeEventListener('icegatheringstatechange',done);clearInterval(timer);resolve();}};
    const timer=setInterval(done,120);
    pc.addEventListener('icegatheringstatechange',done);
  });
}
function signalText(){return ($('mpSignal').value||'').trim();}
function setupDataChannel(ch){
  p2pDC=ch;
  ch.onopen=()=>{
    setStatus('P2P conectado · los jugadores ya pueden verse. Elige el mismo mapa y pulsa JUGAR en ambos.');
    try{ch.send(JSON.stringify({t:'state',d:{on:state==='playing'?1:0,x:P.x,y:P.y,z:P.z,yaw:P.yaw,m:curMap,n:myName}}));}catch(e){}
  };
  ch.onclose=()=>{if(p2pDC===ch){p2pDC=null;removeRemote('rtc');setStatus('Conexión P2P cerrada.');}};
  ch.onerror=()=>setStatus('Error en el canal P2P. Repite la conexión.');
  ch.onmessage=ev=>{
    try{
      const m=JSON.parse(ev.data);
      if(m.t==='state')applyRemotePeer('rtc',m.d);
      else if(m.t==='shot'&&m.a&&m.b)tracer(V3(m.a[0],m.a[1],m.a[2]),V3(m.b[0],m.b[1],m.b[2]),m.c||0xffe0a0);
    }catch(e){}
  };
}
async function createOffer(){
  closeP2P();
  if(!window.RTCPeerConnection){setStatus('Este navegador no soporta WebRTC P2P.');return;}
  try{
    p2pPC=new RTCPeerConnection(RTC_CONFIG);
    p2pPC.ondatachannel=e=>setupDataChannel(e.channel);
    p2pPC.onconnectionstatechange=()=>{if(p2pPC.connectionState==='connected')setStatus('P2P conectado.');else if(['failed','disconnected','closed'].includes(p2pPC.connectionState))removeRemote('rtc');};
    setupDataChannel(p2pPC.createDataChannel('anillo-game',{ordered:true}));
    const offer=await p2pPC.createOffer();
    await p2pPC.setLocalDescription(offer); await waitForIce(p2pPC);
    $('mpSignal').value=JSON.stringify({type:p2pPC.localDescription.type,sdp:p2pPC.localDescription.sdp});
    setStatus('OFERTA P2P creada. Copia el texto y envíalo al segundo jugador.');
  }catch(e){closeP2P();setStatus('No se pudo crear la oferta P2P.');}
}
async function answerOffer(){
  const raw=signalText();if(!raw){setStatus('Pega primero la oferta del jugador 1.');return;}
  closeP2P();
  if(!window.RTCPeerConnection){setStatus('Este navegador no soporta WebRTC P2P.');return;}
  try{
    const offer=JSON.parse(raw);if(offer.type!=='offer'||!offer.sdp)throw new Error('oferta');
    p2pPC=new RTCPeerConnection(RTC_CONFIG);
    p2pPC.ondatachannel=e=>setupDataChannel(e.channel);
    p2pPC.onconnectionstatechange=()=>{if(p2pPC.connectionState==='connected')setStatus('Respuesta P2P conectada.');};
    await p2pPC.setRemoteDescription(offer);
    const answer=await p2pPC.createAnswer(); await p2pPC.setLocalDescription(answer); await waitForIce(p2pPC);
    $('mpSignal').value=JSON.stringify({type:p2pPC.localDescription.type,sdp:p2pPC.localDescription.sdp});
    setStatus('RESPUESTA creada. Copia el texto y devuélveselo al jugador 1.');
  }catch(e){closeP2P();setStatus('La oferta no es válida o expiró.');}
}
async function applyAnswer(){
  const raw=signalText();if(!raw||!p2pPC){setStatus('En el jugador 1: pega aquí la respuesta con la oferta P2P ya creada.');return;}
  try{
    const answer=JSON.parse(raw);if(answer.type!=='answer'||!answer.sdp)throw new Error('respuesta');
    await p2pPC.setRemoteDescription(answer);setStatus('Respuesta aplicada. Esperando conexión P2P…');
  }catch(e){setStatus('La respuesta no es válida.');}
}
async function copySignal(){
  const raw=signalText();if(!raw)return;
  try{if(navigator.clipboard&&navigator.clipboard.writeText){await navigator.clipboard.writeText(raw);setStatus('Código copiado al portapapeles.');return;}}catch(e){}
  $('mpSignal').focus();$('mpSignal').select();setStatus('Código seleccionado. Pulsa Ctrl+C para copiarlo.');
}
let selMap='base',mode='solo';

const pick=(sel,fn)=>document.querySelectorAll(sel).forEach(b=>b.addEventListener('click',()=>{document.querySelectorAll(sel).forEach(x=>x.classList.toggle('on',x===b));fn(b);}));
pick('#mapOpts .opt',b=>{selMap=b.dataset.map;if(state==='menu'||state==='over')loadMap(selMap);});
pick('#modeOpts .opt',b=>{mode=b.dataset.mode;$('mpPanel').classList.toggle('hidden',mode!=='multi');if(mode==='solo')leaveRoom();});
$('mpJoin').addEventListener('click',joinRoom);
$('mpOffer').addEventListener('click',createOffer);
$('mpAnswer').addEventListener('click',answerOffer);
$('mpApply').addEventListener('click',applyAnswer);
$('mpCopy').addEventListener('click',copySignal);
function netSend(dt){
  netT-=dt;if(netT>0)return;netT=0.08;
  const payload={on:state==='playing'?1:0,x:P.x,y:P.y,z:P.z,yaw:P.yaw,m:curMap,n:myName};
  if(bcRoom){try{bcRoom.postMessage({type:'state',id:netId,state:payload});}catch(e){}}
  if(p2pDC&&p2pDC.readyState==='open'){try{p2pDC.send(JSON.stringify({t:'state',d:payload}));}catch(e){}}
}
function netShot(a,b,c){
  const msg={t:'shot',a:[a.x,a.y,a.z],b:[b.x,b.y,b.z],c};
  if(bcRoom){try{bcRoom.postMessage({type:'shot',id:netId,a:msg.a,b:msg.b,c:msg.c});}catch(e){}}
  if(p2pDC&&p2pDC.readyState==='open'){try{p2pDC.send(JSON.stringify(msg));}catch(e){}}
}
function updateRemote(dt){const k=1-Math.exp(-12*dt);remote.forEach(r=>{const p=r.g.position;p.x+=(r.x-p.x)*k;p.y+=(r.y-p.y)*k;p.z+=(r.z-p.z)*k;let dy=(r.ty||0)-r.g.rotation.y;dy=Math.atan2(Math.sin(dy),Math.cos(dy));r.g.rotation.y+=dy*k;});}

let last = performance.now();
function frame(now) {
  requestAnimationFrame(frame);
  const dt = Math.min(0.05, (now - last) / 1000); last = now;
  sky.position.copy(camera.position);
  updateRemote(dt); netSend(dt);
  monitorPerformance(dt);
  updateInteractables(dt);
  if (state === 'playing') {
    gt += dt;
    updatePlayer(dt);
    updateWaves(dt);
    updateEnemies(dt);
    updateProjectiles(dt);
    updateHUD(dt);
  } else if (state === 'menu' || state === 'over') {
    orbitT += dt * 0.12;
    const OR = curMap === 'city' ? 100 : 46, OY = curMap === 'city' ? 55 : 11;
    camera.position.set(Math.cos(orbitT) * OR, OY + Math.sin(orbitT * 2) * 2, Math.sin(orbitT) * OR);
    camera.lookAt(0, 3, 0);
    updateProjectiles(dt);
    if (state === 'over') { updateEnemies(dt * 0.0001); }
  }
  renderer.render(scene, camera);
}
loadMap('base'); showOverlay('menu');
requestAnimationFrame(frame);
})();
</script>
</body>
</html>
