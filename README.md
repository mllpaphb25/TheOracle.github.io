<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width,initial-scale=1,viewport-fit=cover">
<meta name="description" content="The Oracle — a real-time audio-reactive sphere that bounces on bass, grows spikes on treble and shifts colour with the tone of the music.">
<meta property="og:title" content="The Oracle">
<meta property="og:description" content="Real-time audio-reactive visuals. Play a song, share a tab or use a mic.">
<meta name="theme-color" content="#0b0c10">
<link rel="icon" href="data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 64 64'%3E%3Cdefs%3E%3CradialGradient id='g' cx='35%25' cy='30%25'%3E%3Cstop offset='0' stop-color='%23fff6c8'/%3E%3Cstop offset='.5' stop-color='%23e46aa8'/%3E%3Cstop offset='1' stop-color='%233a2a8c'/%3E%3C/radialGradient%3E%3C/defs%3E%3Crect width='64' height='64' rx='12' fill='%230b0c10'/%3E%3Ccircle cx='32' cy='32' r='20' fill='url(%23g)'/%3E%3C/svg%3E">
<title>The Oracle</title>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=IBM+Plex+Mono:wght@400;500&family=IBM+Plex+Sans:wght@400;500;600&display=swap">
<style>
/* Layout: a 16:9 output screen centred above one control strip. The live data HUD sits bottom-centre inside the screen and scales with it. Single dark look on purpose (stage/VJ use). */
:root {
  --stage: #0b0c10;
  --panel: rgba(18, 20, 28, 0.78);
  --line: rgba(220, 214, 200, 0.14);
  --ink: #ece7dc;
  --ink-dim: #9a968c;
  --accent: #f2b45a;
  --font-ui: "IBM Plex Sans", system-ui, -apple-system, "Segoe UI", sans-serif;
  --font-data: "IBM Plex Mono", ui-monospace, Menlo, Consolas, monospace;
  color-scheme: dark;
  --void: #050608;
}
html, body { height: 100%; overflow: hidden; }
body { background: var(--void); color: var(--ink); font-family: var(--font-ui); display: flex; flex-direction: column; }

.wrap { flex: 1; min-height: 0; padding: 16px 16px 12px; display: grid; place-items: center; container-type: size; }
.output { --aw: 16; --ah: 9; width: min(100cqw, (100cqh - 22px) * var(--aw) / var(--ah)); display: grid; gap: 6px; }
.output.portrait { --aw: 9; --ah: 16; }
.wrap:fullscreen { padding: 0; background: var(--void); }
.wrap:fullscreen .output { width: min(100cqw, 100cqh * var(--aw) / var(--ah)); }
.wrap:fullscreen .caption { display: none; }
.wrap:fullscreen .screen { border: 0; border-radius: 0; }
.caption { display: flex; justify-content: space-between; gap: 12px; font-family: var(--font-data); font-size: 11px; color: var(--ink-dim); letter-spacing: 0.08em; text-transform: uppercase; font-variant-numeric: tabular-nums; }
.screen { --u: 1cqw; position: relative; width: 100%; aspect-ratio: var(--aw) / var(--ah); background: var(--stage); border: 1px solid var(--line); border-radius: 4px; overflow: hidden; container-type: size; }
canvas.stage { position: absolute; inset: 0; width: 100%; height: 100%; display: block; }

/* in-screen typography is sized in container units so the output looks identical at any size */
.title { position: absolute; top: 3.2cqh; left: calc(2.4 * var(--u)); pointer-events: none; display: grid; gap: 0.5cqh; }
.title h1 { margin: 0; font-size: max(10px, calc(1.6 * var(--u))); font-weight: 600; letter-spacing: 0.04em; }
.title .src { font-family: var(--font-data); font-size: max(8px, calc(0.85 * var(--u))); color: var(--ink-dim); text-transform: uppercase; letter-spacing: 0.08em; }

.hud {
  position: absolute; left: 50%; bottom: 4cqh; transform: translateX(-50%);
  display: flex; align-items: stretch; pointer-events: none;
  background: rgba(11, 12, 16, 0.55); border: 1px solid var(--line); border-radius: calc(0.6 * var(--u));
  backdrop-filter: blur(6px); -webkit-backdrop-filter: blur(6px);
  font-family: var(--font-data); font-size: max(8px, calc(0.85 * var(--u))); color: var(--ink-dim); font-variant-numeric: tabular-nums; letter-spacing: 0.06em;
}
.cell { display: grid; gap: 0.7cqh; padding: 1.2cqh calc(1.4 * var(--u)); min-width: calc(9 * var(--u)); }
.cell + .cell { border-left: 1px solid var(--line); }
.cell .k { display: flex; justify-content: space-between; gap: calc(1 * var(--u)); }
.cell .k b { font-weight: 500; color: var(--ink); }
.bar { height: max(3px, 0.45cqh); background: var(--line); border-radius: 2px; overflow: hidden; position: relative; }
.bar i { display: block; height: 100%; width: 0; }
#mBass { background: #8f7cff; } #mMid { background: #e6a1c8; } #mHigh { background: #f7f0b0; }
.cell.tone-cell { min-width: calc(14 * var(--u)); }
.tone { height: max(4px, 0.6cqh); border-radius: 3px; background: linear-gradient(90deg, hsl(250 60% 18%), hsl(300 70% 40%), hsl(335 85% 62%), hsl(30 95% 72%), hsl(60 100% 82%)); position: relative; }
.tone b { position: absolute; top: -0.5cqh; width: 2px; height: calc(100% + 1cqh); background: var(--ink); left: 0; transform: translateX(-1px); }
.swatch { width: max(8px, calc(0.8 * var(--u))); height: max(8px, calc(0.8 * var(--u))); border-radius: 50%; border: 1px solid var(--line); background: #fff; display: inline-block; vertical-align: middle; }
.beat { width: max(6px, calc(0.6 * var(--u))); height: max(6px, calc(0.6 * var(--u))); border-radius: 50%; background: var(--line); transition: background 80ms; align-self: center; }
.beat.on { background: var(--accent); }

.portrait .screen { --u: 2.6cqw; }
.portrait .hud { display: grid; grid-template-columns: repeat(3, 1fr); width: 86cqw; }
.portrait .cell { min-width: 0; }
.portrait .cell:nth-child(4) { grid-column: span 2; border-left: 0; }
.portrait .cell:nth-child(n+4) { border-top: 1px solid var(--line); }

.controls {
  flex: none;
  padding: 12px 16px calc(12px + env(safe-area-inset-bottom, 0px));
  display: flex; flex-wrap: wrap; gap: 10px 18px; align-items: center;
  background: var(--panel); backdrop-filter: blur(10px); -webkit-backdrop-filter: blur(10px);
  border-top: 1px solid var(--line);
  transition: opacity 300ms;
}
.controls.hidden { display: none; }
.group { display: flex; flex-wrap: wrap; gap: 6px; align-items: center; min-width: 0; }
.label { font-size: 11px; color: var(--ink-dim); letter-spacing: 0.08em; text-transform: uppercase; margin-right: 4px; }
button, .file {
  font: 500 13px var(--font-ui); color: var(--ink);
  background: transparent; border: 1px solid var(--line); border-radius: 6px;
  padding: 6px 12px; cursor: pointer; white-space: nowrap;
}
button:hover, .file:hover { border-color: var(--ink-dim); }
button:focus-visible, .file:focus-within, input:focus-visible { outline: 2px solid var(--accent); outline-offset: 2px; }
button[aria-pressed="true"] { background: var(--ink); color: var(--stage); border-color: var(--ink); }
.file input { position: absolute; opacity: 0; width: 1px; height: 1px; }
input[type=range] { width: 90px; accent-color: var(--accent); }
input[type=range]:disabled { opacity: 0.35; }
input[type=url] { font: 13px var(--font-data); color: var(--ink); background: transparent; border: 1px solid var(--line); border-radius: 6px; padding: 6px 10px; width: min(320px, 60vw); min-width: 0; }
input[type=url]::placeholder { color: var(--ink-dim); }
.msg { font-size: 12px; color: var(--accent); min-height: 1em; flex-basis: 100%; }
.msg:empty { display: none; }
.keys { font-family: var(--font-data); font-size: 11px; color: var(--ink-dim); margin-left: auto; }
@media (max-width: 640px) { .keys { display: none; } }
</style>
</head>
<body style="margin:0">
<main class="wrap" id="wrap">
  <div class="output" id="output">
    <div class="caption"><span id="fmtLabel">Output · 16:9</span><span id="resLabel">— × —</span></div>
    <div class="screen" id="screen">
      <canvas class="stage" id="gl" aria-label="The Oracle — audio-reactive sphere"></canvas>
      <canvas class="stage" id="stage" aria-label="Audio-reactive 2D visuals" hidden></canvas>
      <div class="title">
        <h1>The Oracle</h1>
        <div class="src" id="srcLabel">idle · no audio input</div>
      </div>
      <div class="hud" aria-hidden="true">
        <div class="cell"><div class="k"><span>BASS</span><b id="vBass">0.00</b></div><div class="bar"><i id="mBass"></i></div></div>
        <div class="cell"><div class="k"><span>MID</span><b id="vMid">0.00</b></div><div class="bar"><i id="mMid"></i></div></div>
        <div class="cell"><div class="k"><span>HIGH</span><b id="vHigh">0.00</b></div><div class="bar"><i id="mHigh"></i></div></div>
        <div class="cell tone-cell"><div class="k"><span>TONE <span id="vTone">LOW</span></span><span class="swatch" id="swatch"></span></div><div class="tone"><b id="toneMark"></b></div></div>
        <div class="cell"><div class="k"><span>BEAT</span><b id="vBpm">— bpm</b></div><div class="beat" id="beatDot"></div></div>
      </div>
    </div>
  </div>
</main>

<div class="controls" id="controls">
  <div class="group">
    <span class="label">Source</span>
    <button id="btnDemo" type="button">▶ Demo track</button>
    <label class="file" for="fileIn">Open audio file<input id="fileIn" type="file" accept="audio/*"></label>
    <button id="btnTab" type="button">Tab audio</button>
    <button id="btnMic" type="button">Mic / line in</button>
    <button id="btnStop" type="button">Stop</button>
  </div>
  <form class="group" id="urlForm">
    <label class="label" for="urlIn">Link</label>
    <input id="urlIn" type="url" placeholder="https://…/track.mp3" autocomplete="off">
    <button type="submit">Play</button>
  </form>
  <div class="group" role="group" aria-label="Output format">
    <span class="label">Format</span>
    <button id="fmtL" type="button" aria-pressed="true">16:9</button>
    <button id="fmtP" type="button" aria-pressed="false">9:16</button>
  </div>
  <div class="group" role="group" aria-label="Visual mode">
    <span class="label">Visual</span>
    <button id="mode0" type="button" aria-pressed="true">Oracle</button>
    <button id="mode1" type="button" aria-pressed="false">Dots</button>
    <button id="mode2" type="button" aria-pressed="false">Waves</button>
    <button id="mode3" type="button" aria-pressed="false">Color</button>
  </div>
  <div class="group" id="oracleCtl">
    <span class="label">Oracle</span>
    <label class="label" for="kBounce">Bounce</label>
    <input id="kBounce" type="range" min="0" max="2" step="0.05" value="1">
    <label class="label" for="kSpike">Spike</label>
    <input id="kSpike" type="range" min="0" max="2" step="0.05" value="1">
    <label class="label" for="kFold">Fold</label>
    <input id="kFold" type="range" min="0" max="2" step="0.05" value="1">
  </div>
  <div class="group">
    <label class="label" for="sens">Gain</label>
    <input id="sens" type="range" min="0.3" max="2.5" step="0.05" value="1">
    <label class="label" for="trail">Trail</label>
    <input id="trail" type="range" min="0" max="0.95" step="0.01" value="0.75" disabled>
  </div>
  <span class="keys">1–4 visual · R rotate 16:9 / 9:16 · H hide bar · F full-screen output</span>
  <div class="msg" id="msg" role="status"></div>
</div>

<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
<script>
(() => {
  const $ = id => document.getElementById(id);
  const msg = t => { $('msg').textContent = t || ''; };
  const reduced = matchMedia('(prefers-reduced-motion: reduce)').matches;
  const clamp = (x, a = 0, b = 1) => Math.min(b, Math.max(a, x));

  // ---------- 2D canvas ----------
  const cv = $('stage');
  const cx = cv.getContext('2d');
  let W = 0, H = 0, DPR = 1;
  function resize2d() {
    DPR = Math.min(window.devicePixelRatio || 1, 2);
    W = $('screen').clientWidth; H = $('screen').clientHeight;
    cv.width = W * DPR; cv.height = H * DPR;
    cx.setTransform(DPR, 0, 0, DPR, 0, 0);
    cx.fillStyle = '#0b0c10'; cx.fillRect(0, 0, W, H);
  }

  // ---------- audio graph ----------
  let ac = null, analyser = null, master = null, current = null;
  let freq = new Uint8Array(1024), wave = new Uint8Array(2048);
  function ensureAudio() {
    if (ac) return;
    const AC = window.AudioContext || window.webkitAudioContext;
    try { ac = new AC({ latencyHint: 'playback' }); } catch { ac = new AC(); } // larger buffer = fewer dropouts
    analyser = ac.createAnalyser();
    analyser.fftSize = 2048; analyser.smoothingTimeConstant = 0.8;
    freq = new Uint8Array(analyser.frequencyBinCount);
    wave = new Uint8Array(analyser.fftSize);
    master = ac.createGain(); master.gain.value = 0.9;
    // Only demo/file/url go to the speakers. Tab and mic connect to the analyser alone,
    // otherwise their sound would play a second time, slightly delayed (heard as echo / stutter).
    master.connect(analyser); master.connect(ac.destination);
  }
  function stopCurrent() { if (current) { current.stop(); current = null; } $('srcLabel').textContent = 'idle · no audio input'; }

  // ---------- demo: a small generative track built from oscillators ----------
  function startDemo() {
    ensureAudio(); ac.resume(); stopCurrent();
    const bus = ac.createGain(); bus.gain.value = 0.8; bus.connect(master);
    const bpm = 112, step = 60 / bpm / 4;
    const chords = [[57,60,64],[53,57,60],[48,52,55],[55,59,62]]; // Am F C G
    const mtof = m => 440 * Math.pow(2, (m - 69) / 12);
    const noiseBuf = ac.createBuffer(1, ac.sampleRate * 0.5, ac.sampleRate);
    const nd = noiseBuf.getChannelData(0); for (let i = 0; i < nd.length; i++) nd[i] = Math.random() * 2 - 1;
    let n = 0, next = ac.currentTime + 0.05;
    const env = (g, t, a, peak, d) => { g.gain.setValueAtTime(0.0001, t); g.gain.exponentialRampToValueAtTime(peak, t + a); g.gain.exponentialRampToValueAtTime(0.0001, t + a + d); };
    function kick(t) { const o = ac.createOscillator(), g = ac.createGain(); o.frequency.setValueAtTime(150, t); o.frequency.exponentialRampToValueAtTime(42, t + 0.18); env(g, t, 0.003, 1.0, 0.35); o.connect(g).connect(bus); o.start(t); o.stop(t + 0.4); }
    function hat(t, v) { const s = ac.createBufferSource(), f = ac.createBiquadFilter(), g = ac.createGain(); s.buffer = noiseBuf; f.type = 'highpass'; f.frequency.value = 7000; env(g, t, 0.002, v, 0.06); s.connect(f).connect(g).connect(bus); s.start(t); s.stop(t + 0.1); }
    function snare(t) { const s = ac.createBufferSource(), f = ac.createBiquadFilter(), g = ac.createGain(); s.buffer = noiseBuf; f.type = 'bandpass'; f.frequency.value = 1800; env(g, t, 0.002, 0.5, 0.18); s.connect(f).connect(g).connect(bus); s.start(t); s.stop(t + 0.25); }
    function bass(t, m) { const o = ac.createOscillator(), f = ac.createBiquadFilter(), g = ac.createGain(); o.type = 'sawtooth'; o.frequency.value = mtof(m); f.type = 'lowpass'; f.frequency.setValueAtTime(900, t); f.frequency.exponentialRampToValueAtTime(160, t + 0.25); env(g, t, 0.005, 0.35, 0.3); o.connect(f).connect(g).connect(bus); o.start(t); o.stop(t + 0.35); }
    function pad(t, ch, dur) { ch.forEach(m => { const o = ac.createOscillator(), g = ac.createGain(); o.type = 'triangle'; o.frequency.value = mtof(m); o.detune.value = (Math.random() - 0.5) * 12; g.gain.setValueAtTime(0.0001, t); g.gain.exponentialRampToValueAtTime(0.06, t + 0.4); g.gain.exponentialRampToValueAtTime(0.0001, t + dur); o.connect(g).connect(bus); o.start(t); o.stop(t + dur + 0.05); }); }
    function lead(t, m) { const o = ac.createOscillator(), g = ac.createGain(); o.type = 'sine'; o.frequency.value = mtof(m); env(g, t, 0.01, 0.12, 0.25); o.connect(g).connect(bus); o.start(t); o.stop(t + 0.3); }
    function schedule() {
      while (next < ac.currentTime + 0.12) {
        const s = n % 16, bar = Math.floor(n / 16) % 4, ch = chords[bar];
        const section = Math.floor(n / 64) % 4; // 0: bass-heavy, 1: full, 2: bright/airy, 3: full
        const drums = section !== 2, airy = section >= 1;
        if (drums && (s % 4 === 0 || (section === 3 && s === 10))) kick(next);
        if (drums && (s === 4 || s === 12)) snare(next);
        if (airy) hat(next, (s % 2 ? 0.08 : 0.18) * (section === 2 ? 2.2 : 1));
        if (section !== 2 && [0, 3, 6, 10, 14].includes(s)) bass(next, ch[0] - 24 + (s === 14 ? 7 : 0));
        if (s === 0) pad(next, section === 2 ? ch.map(m => m + 12) : ch, step * 16);
        if (airy && s % 2 === 0 && Math.random() < 0.75) lead(next, ch[(s / 2) % 3] + 12 + (section === 2 || Math.random() < 0.3 ? 12 : 0));
        next += step; n++;
      }
    }
    const timer = setInterval(schedule, 25); schedule();
    current = { stop() { clearInterval(timer); bus.gain.setTargetAtTime(0, ac.currentTime, 0.05); setTimeout(() => bus.disconnect(), 300); } };
    $('srcLabel').textContent = 'demo · 112 bpm · Am–F–C–G';
    msg('');
  }

  // ---------- local file ----------
  function startFile(file) {
    ensureAudio(); ac.resume(); stopCurrent();
    const el = new Audio(); el.src = URL.createObjectURL(file); el.loop = true;
    const src = ac.createMediaElementSource(el); src.connect(master);
    el.play().catch(() => msg('This file could not be played. Try an .mp3, .wav, .m4a or .ogg file.'));
    current = { stop() { el.pause(); src.disconnect(); URL.revokeObjectURL(el.src); } };
    $('srcLabel').textContent = 'file · ' + file.name;
    msg('');
  }

  // ---------- direct audio URL (server must allow CORS) ----------
  const isStreamingPage = u => /(^|\.)(youtube\.com|youtu\.be|spotify\.com|soundcloud\.com|music\.apple\.com|tiktok\.com|deezer\.com|tidal\.com)$/i.test(u.hostname);
  function startUrl(raw) {
    let u; try { u = new URL(raw.trim()); } catch { msg('That link is not valid. It should start with https://'); return; }
    if (isStreamingPage(u)) { msg('Streaming links (YouTube, Spotify, SoundCloud…) can’t be read directly. Play the song in another tab, then press “Tab audio” and tick “Share tab audio”.'); return; }
    ensureAudio(); ac.resume(); stopCurrent();
    const el = new Audio(); el.crossOrigin = 'anonymous'; el.loop = true; el.src = u.href;
    const src = ac.createMediaElementSource(el); src.connect(master);
    el.onerror = () => { stopCurrent(); msg('Could not load that link. It must point straight to an audio file (.mp3, .wav, .ogg) on a server that allows cross-origin access. Otherwise download the file and use “Open audio file”.'); };
    el.play().catch(() => {});
    current = { stop() { el.onerror = null; el.pause(); el.removeAttribute('src'); src.disconnect(); } };
    $('srcLabel').textContent = 'url · ' + decodeURIComponent(u.pathname.split('/').pop() || u.hostname);
    msg('');
  }

  // ---------- tab audio: play YouTube etc. in another tab and share that tab's sound ----------
  async function startTab() {
    ensureAudio(); ac.resume();
    // Only Chromium browsers (Chrome, Edge, Brave, Arc) can capture a tab's sound. Safari and Firefox share picture only.
    const ua = navigator.userAgent;
    const chromium = !!navigator.userAgentData || (/Chrome\//.test(ua) && !/Firefox\//.test(ua));
    if (!chromium) {
      msg('Tab audio needs Chrome or Edge. This browser (Safari / Firefox) only shares picture, not sound. Open this file in Chrome, or use BlackHole with “Mic / line in”.');
      return;
    }
    let stream;
    try {
      stream = await navigator.mediaDevices.getDisplayMedia({
        video: { displaySurface: 'browser', frameRate: { ideal: 1, max: 5 }, width: { ideal: 320 }, height: { ideal: 180 } }, // picker opens on “Chrome Tab”; tiny video keeps CPU free for audio
        audio: { echoCancellation: false, noiseSuppression: false, autoGainControl: false, suppressLocalAudioPlayback: false },
        preferCurrentTab: false, selfBrowserSurface: 'exclude', surfaceSwitching: 'include', systemAudio: 'include'
      });
    } catch (e) {
      msg(window.top === window
        ? 'Tab sharing was cancelled. Press “Tab audio” again, pick the tab playing music and tick “Share tab audio”.'
        : 'Tab audio is blocked on this hosted page. Open the downloaded HTML file in Chrome or Edge to use it.');
      return;
    }
    if (!stream.getAudioTracks().length) {
      const kind = stream.getVideoTracks()[0]?.getSettings?.().displaySurface;
      stream.getTracks().forEach(t => t.stop());
      msg(kind && kind !== 'browser'
        ? 'No sound came through: a window or screen was shared. Press “Tab audio” again and choose the “Chrome Tab” section, pick the tab playing music, and turn on “Also share tab audio”.'
        : 'No sound came through. Press “Tab audio” again and turn on “Also share tab audio” at the bottom of the picker before pressing Share.');
      return; }
    stopCurrent();
    const src = ac.createMediaStreamSource(new MediaStream(stream.getAudioTracks()));
    src.connect(analyser); // the tab already plays its own sound
    stream.getTracks().forEach(t => t.addEventListener('ended', () => { if (current && current.tab === stream) stopCurrent(); }));
    current = { tab: stream, stop() { src.disconnect(); stream.getTracks().forEach(t => t.stop()); } };
    $('srcLabel').textContent = 'tab · ' + (stream.getVideoTracks()[0]?.label || 'shared audio');
    msg('');
  }

  // ---------- mic / line in (also BlackHole / Loopback for system audio) ----------
  async function startMic() {
    ensureAudio(); ac.resume();
    try {
      const stream = await navigator.mediaDevices.getUserMedia({ audio: { echoCancellation: false, noiseSuppression: false, autoGainControl: false } });
      stopCurrent();
      const src = ac.createMediaStreamSource(stream);
      src.connect(analyser);
      current = { stop() { src.disconnect(); stream.getTracks().forEach(t => t.stop()); } };
      $('srcLabel').textContent = 'mic · ' + (stream.getAudioTracks()[0]?.label || 'live input');
      msg('');
    } catch (e) {
      msg(window.top === window
        ? 'Microphone access was refused. Allow it in the browser’s site settings and try again.'
        : 'The microphone is blocked on this hosted page. Open the downloaded HTML file in your browser to use it.');
    }
  }

  $('urlForm').onsubmit = e => { e.preventDefault(); const v = $('urlIn').value; if (v) startUrl(v); };
  $('btnTab').onclick = startTab;
  $('btnDemo').onclick = startDemo;
  $('fileIn').onchange = e => { const f = e.target.files[0]; if (f) startFile(f); e.target.value = ''; };
  $('btnMic').onclick = startMic;
  $('btnStop').onclick = stopCurrent;

  // ---------- analysis ----------
  const A = { bass: 0, mid: 0, high: 0, level: 0, centroid: 0.3, tone: 0.5, beat: 0, bpm: 0, presence: 0 };
  let bassAvg = 0, lastBeat = 0; const beatTimes = [];
  const binHz = () => (ac ? ac.sampleRate : 44100) / 2 / freq.length;
  function band(lo, hi) { const b = binHz(); let s = 0, c = 0; for (let i = Math.max(1, Math.floor(lo / b)); i <= Math.min(freq.length - 1, Math.ceil(hi / b)); i++) { s += freq[i]; c++; } return c ? s / c / 255 : 0; }

  let lastAT = 0;
  function analyse(t) {
    const live = !!current;
    const fr = clamp((t - (lastAT || t - 16.7)) / 16.7, 0.2, 6); lastAT = t; // frame-rate independent easing
    const ease = r => 1 - Math.pow(1 - r, fr);
    if (live) { analyser.getByteFrequencyData(freq); analyser.getByteTimeDomainData(wave); }
    else {
      for (let i = 0; i < freq.length; i++) freq[i] = 255 * Math.max(0, 0.32 * Math.exp(-i / 140) + 0.06 * Math.sin(t * 0.0012 + i * 0.03) + 0.05);
      for (let i = 0; i < wave.length; i++) wave[i] = 128 + 20 * Math.sin(i * 0.02 + t * 0.002) * Math.sin(t * 0.0007);
    }
    const sens = +$('sens').value;
    const k = x => Math.min(1, x * sens);
    A.bass = k(band(20, 150)); A.mid = k(band(150, 2000)); A.high = k(band(2000, 12000) * 1.8);
    A.level = (A.bass + A.mid + A.high) / 3;
    let num = 0, den = 0; for (let i = 1; i < freq.length; i++) { num += i * freq[i]; den += freq[i]; }
    A.centroid += ((den ? num / den / freq.length : 0.2) - A.centroid) * 0.05;

    // tone: 0 = deep / bass-heavy, 1 = high / bright
    if (live && den > 0) {
      // energy-weighted mean pitch on a log scale (squared magnitudes, so quiet hiss doesn't drag it up)
      let lw = 0, ls = 0; const bh = binHz();
      for (let i = 2; i < freq.length; i++) { const w = freq[i] * freq[i]; lw += w; ls += w * Math.log2(i * bh); }
      const logHz = lw ? ls / lw : Math.log2(300);
      const fromCentroid = clamp((logHz - Math.log2(120)) / (Math.log2(3000) - Math.log2(120)));
      const sum = A.bass + A.mid + A.high + 1e-4;
      const brightness = clamp((A.high * 1.2 + A.mid * 0.3 - A.bass * 1.2) / sum * 0.8 + 0.45);
      A.tone += ((fromCentroid * 0.4 + brightness * 0.6) - A.tone) * ease(0.035);
    }
    // presence: how much the sphere takes on colour (white when silent / idle)
    A.presence += ((live ? clamp(A.level * 6) : 0) - A.presence) * ease(0.05);

    bassAvg = bassAvg * 0.95 + A.bass * 0.05;
    A.beat *= 0.9;
    if (live && A.bass > bassAvg * 1.25 && A.bass > 0.35 && t - lastBeat > 250) {
      A.beat = 1; lastBeat = t; beatTimes.push(t); if (beatTimes.length > 9) beatTimes.shift();
      if (beatTimes.length > 3) { const iv = []; for (let i = 1; i < beatTimes.length; i++) iv.push(beatTimes[i] - beatTimes[i - 1]); iv.sort((a, b) => a - b); let bpm = 60000 / iv[iv.length >> 1]; while (bpm > 165) bpm /= 2; A.bpm = Math.round(bpm); }
    }
    if (!live) { A.bpm = 0; beatTimes.length = 0; }
  }

  // tone → colour. Low: deep indigo, dark. High: light, vivid (pink → peach → lemon).
  function toneHSL(tone) {
    const t = tone * tone * (3 - 2 * tone); // smoothstep: push toward the ends
    return { h: (250 + t * 170) % 360, s: 0.6 + t * 0.4, l: 0.1 + t * 0.72 };
  }
  function hslToRgb(h, s, l) {
    h /= 360; const f = n => { const k = (n + h * 12) % 12; return l - s * Math.min(l, 1 - l) * Math.max(-1, Math.min(k - 3, 9 - k, 1)); };
    return [f(0), f(8), f(4)];
  }
  function sphereRGB() {
    const c = toneHSL(A.tone), rgb = hslToRgb(c.h, c.s, c.l), p = A.presence;
    return rgb.map(v => 1 + (v - 1) * p); // mix from white
  }

  // ---------- Sphere (Three.js, deformed in the vertex shader) ----------
  let gl = null;
  function initSphere() {
    if (!window.THREE) return null;
    try {
      const renderer = new THREE.WebGLRenderer({ canvas: $('gl'), antialias: true });
      renderer.setPixelRatio(Math.min(devicePixelRatio || 1, 1.5));
      renderer.setClearColor(0x0b0c10, 1);
      const scene = new THREE.Scene();
      const camera = new THREE.PerspectiveCamera(35, 1, 0.1, 100);
      camera.position.set(0, 0, 6);
      const SPIKES = 40;
      // spike directions spread evenly over the sphere (Fibonacci lattice)
      const dirs = [];
      for (let i = 0; i < SPIKES; i++) { const y = 1 - 2 * (i + 0.5) / SPIKES, r = Math.sqrt(1 - y * y), ph = (i + 0.5) * 2.399963; dirs.push(new THREE.Vector3(Math.cos(ph) * r, y, Math.sin(ph) * r)); }
      const uniforms = {
        uTime: { value: 0 }, uBass: { value: 0 }, uMid: { value: 0 }, uHigh: { value: 0 },
        uSpike: { value: new Array(SPIKES).fill(0) }, uDir: { value: dirs },
        uC1: { value: new THREE.Color(1, 1, 1) }, uC2: { value: new THREE.Color(1, 1, 1) },
        uC3: { value: new THREE.Color(1, 1, 1) }, uC4: { value: new THREE.Color(1, 1, 1) },
        uRim: { value: new THREE.Color(1, 1, 1) }
      };
      const NOISE = `
        vec3 mod289(vec3 x){return x-floor(x*(1.0/289.0))*289.0;}
        vec4 mod289(vec4 x){return x-floor(x*(1.0/289.0))*289.0;}
        vec4 permute(vec4 x){return mod289(((x*34.0)+1.0)*x);}
        vec4 taylorInvSqrt(vec4 r){return 1.79284291400159-0.85373472095314*r;}
        float snoise(vec3 v){
          const vec2 C=vec2(1.0/6.0,1.0/3.0); const vec4 D=vec4(0.0,0.5,1.0,2.0);
          vec3 i=floor(v+dot(v,C.yyy)); vec3 x0=v-i+dot(i,C.xxx);
          vec3 g=step(x0.yzx,x0.xyz); vec3 l=1.0-g; vec3 i1=min(g.xyz,l.zxy); vec3 i2=max(g.xyz,l.zxy);
          vec3 x1=x0-i1+C.xxx; vec3 x2=x0-i2+C.yyy; vec3 x3=x0-D.yyy;
          i=mod289(i);
          vec4 p=permute(permute(permute(i.z+vec4(0.0,i1.z,i2.z,1.0))+i.y+vec4(0.0,i1.y,i2.y,1.0))+i.x+vec4(0.0,i1.x,i2.x,1.0));
          float n_=0.142857142857; vec3 ns=n_*D.wyz-D.xzx;
          vec4 j=p-49.0*floor(p*ns.z*ns.z); vec4 x_=floor(j*ns.z); vec4 y_=floor(j-7.0*x_);
          vec4 x=x_*ns.x+ns.yyyy; vec4 y=y_*ns.x+ns.yyyy; vec4 h=1.0-abs(x)-abs(y);
          vec4 b0=vec4(x.xy,y.xy); vec4 b1=vec4(x.zw,y.zw);
          vec4 s0=floor(b0)*2.0+1.0; vec4 s1=floor(b1)*2.0+1.0; vec4 sh=-step(h,vec4(0.0));
          vec4 a0=b0.xzyw+s0.xzyw*sh.xxyy; vec4 a1=b1.xzyw+s1.xzyw*sh.zzww;
          vec3 p0=vec3(a0.xy,h.x); vec3 p1=vec3(a0.zw,h.y); vec3 p2=vec3(a1.xy,h.z); vec3 p3=vec3(a1.zw,h.w);
          vec4 norm=taylorInvSqrt(vec4(dot(p0,p0),dot(p1,p1),dot(p2,p2),dot(p3,p3)));
          p0*=norm.x; p1*=norm.y; p2*=norm.z; p3*=norm.w;
          vec4 m=max(0.6-vec4(dot(x0,x0),dot(x1,x1),dot(x2,x2),dot(x3,x3)),0.0); m=m*m;
          return 42.0*dot(m*m,vec4(dot(p0,x0),dot(p1,x1),dot(p2,x2),dot(p3,x3)));
        }`;
      const vertexShader = `
        #define SPIKES 40
        uniform float uTime, uBass, uMid, uHigh;
        uniform float uSpike[SPIKES];
        uniform vec3 uDir[SPIKES];
        varying vec3 vN; varying vec3 vV; varying vec3 vObj; varying float vD; varying float vS;
        ${NOISE}
        // HIGH: sharp cones, one per treble frequency
        float spikes(vec3 n){
          float s = 0.0;
          for (int i = 0; i < SPIKES; i++) {
            float h = uSpike[i];
            if (h > 0.002) {
              float a = acos(clamp(dot(n, uDir[i]), -1.0, 1.0));
              float w = 0.30;
              if (a < w) s = max(s, h * pow(1.0 - a / w, 1.7));
            }
          }
          return s;
        }
        float body(vec3 n){
          float d = snoise(n*1.1 + vec3(uTime*0.22)) * (0.03 + uBass*0.10);          // BASS: soft swell (the thump is the scale bounce)
          d += snoise(n*2.6 + vec3(uTime*0.5, 0.0, -uTime*0.4)) * (uMid*0.22);      // MID: folds
          d += snoise(n*9.0 + vec3(uTime*1.4)) * (uHigh*0.025);                      // HIGH: fine grain
          return d;
        }
        vec3 shape(vec3 n){ return n * (1.0 + body(n) + spikes(n)); }
        void main(){
          vec3 n = normalize(position);
          vec3 t1 = normalize(abs(n.y) < 0.99 ? cross(n, vec3(0.0,1.0,0.0)) : cross(n, vec3(1.0,0.0,0.0)));
          vec3 t2 = cross(n, t1);
          float e = 0.008;
          vec3 p0 = shape(n);
          vec3 p1 = shape(normalize(n + t1*e));
          vec3 p2 = shape(normalize(n + t2*e));
          vec3 nn = normalize(cross(p1 - p0, p2 - p0));
          if (dot(nn, n) < 0.0) nn = -nn;
          vD = body(n); vS = length(p0) - 1.0 - vD; vObj = n;
          vec4 mv = modelViewMatrix * vec4(p0, 1.0);
          vV = mv.xyz; vN = normalize(normalMatrix * nn);
          gl_Position = projectionMatrix * mv;
        }`;
      const fragmentShader = `
        uniform float uTime;
        uniform vec3 uC1, uC2, uC3, uC4, uRim;
        varying vec3 vN; varying vec3 vV; varying vec3 vObj; varying float vD; varying float vS;
        ${NOISE}
        void main(){
          // multi-colour gradient: a slow-moving flow field mixes 3 tone colours across the body, spike tips take the 4th
          float g1 = 0.5 + 0.5 * snoise(vObj*1.2 + vec3(0.0, uTime*0.18, uTime*0.07));
          float g2 = 0.5 + 0.5 * snoise(vObj*2.1 - vec3(uTime*0.11, 0.0, 0.0));
          float band = vObj.y * 0.5 + 0.5;
          vec3 base = mix(uC1, uC2, smoothstep(0.1, 0.9, band*0.55 + g1*0.45));
          base = mix(base, uC3, smoothstep(0.5, 0.9, g2) * 0.85);
          base = mix(base, uC4, smoothstep(0.02, 0.35, vS));
          base *= 0.82 + 0.36 * smoothstep(-0.15, 0.2, vD);
          vec3 N = normalize(vN); vec3 V = normalize(-vV);
          vec3 L1 = normalize(vec3(0.6, 0.8, 0.7)); vec3 L2 = normalize(vec3(-0.7, -0.3, 0.4));
          float dif = max(dot(N, L1), 0.0)*0.8 + max(dot(N, L2), 0.0)*0.25 + 0.22;
          float spec = pow(max(dot(N, normalize(L1 + V)), 0.0), 40.0) * 0.3;
          float fr = pow(1.0 - max(dot(N, V), 0.0), 2.5);
          gl_FragColor = vec4(base*dif + vec3(spec) + uRim*fr*0.45, 1.0);
        }`;
      const mat = new THREE.ShaderMaterial({ uniforms, vertexShader, fragmentShader });
      const mesh = new THREE.Mesh(new THREE.SphereGeometry(1, 200, 200), mat);
      scene.add(mesh);
      function size() { const sc = $('screen'), w = Math.max(1, sc.clientWidth), h = Math.max(1, sc.clientHeight); renderer.setSize(w, h, false); camera.aspect = w / h; camera.position.z = camera.aspect < 1 ? 6.8 / camera.aspect : 7; camera.updateProjectionMatrix(); }
      size();
      // each spike listens to one treble frequency (1.8–14 kHz); shuffled so neighbours differ
      const order = dirs.map((_, i) => i); let seed = 7;
      for (let i = order.length - 1; i > 0; i--) { seed = (seed * 16807) % 2147483647; const j = seed % (i + 1); [order[i], order[j]] = [order[j], order[i]]; }
      const spikeHz = order.map(k => 1800 * Math.pow(14000 / 1800, k / (SPIKES - 1)));
      return { renderer, scene, camera, mesh, uniforms, size, spikeHz, SPIKES };
    } catch (e) { console.error(e); return null; }
  }
  gl = initSphere();
  if (!gl) msg('The Oracle needs WebGL, which this browser doesn’t provide. Showing the 2D visuals instead.');

  const sm = { bass: 0, mid: 0, high: 0 };
  const pal = [[1,1,1],[1,1,1],[1,1,1],[1,1,1]];
  const spring = { s: 1, v: 0, prevBass: 0 };
  let sphereTime = 0;
  function oraclePalette() {
    const c = toneHSL(A.tone), p = A.presence;
    const sp = 20 + A.tone * 45; // low tones keep a tight, deep hue family; high tones fan out
    const set = [
      [c.h, c.s, c.l * 0.75],                                  // deep body
      [c.h + sp, c.s, Math.min(0.9, c.l + 0.04)],              // neighbour hue
      [c.h - sp * 1.3, c.s * 0.95, Math.min(0.92, c.l + 0.1)], // counter hue
      [c.h + 130, 1, Math.min(0.94, c.l + 0.32)]               // spike tips: bright accent
    ];
    return set.map(([h, s, l]) => hslToRgb(((h % 360) + 360) % 360, Math.min(1, s), l).map(v => 1 + (v - 1) * p));
  }
  function drawSphere(dt) {
    const f = reduced ? 0.4 : 1, live = current ? 1 : 0;
    const bounce = +$('kBounce').value, spikeK = +$('kSpike').value, foldK = +$('kFold').value, sens = +$('sens').value;
    sm.bass += (A.bass - sm.bass) * 0.3; sm.mid += (A.mid - sm.mid) * 0.2; sm.high += (A.high - sm.high) * 0.3;
    sphereTime += dt * 0.001 * (0.3 + A.level * 1.4) * f;
    const u = gl.uniforms;
    u.uTime.value = sphereTime;
    u.uBass.value = live ? sm.bass * f : 0.3 + 0.2 * Math.sin(sphereTime * 2);
    u.uMid.value = sm.mid * f * live * foldK;
    u.uHigh.value = sm.high * f * live * spikeK;

    // BASS: spring bounce. Kick transients punch the scale outward, the spring pulls it back (thump, thump).
    const onset = Math.max(0, A.bass - spring.prevBass); spring.prevBass = A.bass;
    const target = 1 + (live ? A.bass * 0.10 * bounce : 0);
    spring.v += (target - spring.s) * 0.28 + live * f * bounce * (onset * 0.9 + (A.beat > 0.99 ? 0.07 : 0));
    spring.v *= 0.68; spring.s += spring.v;
    const k = spring.s - 1;
    gl.mesh.scale.set(1 + k * 1.1, 1 + k * 0.8, 1 + k * 1.1); // a little squash on each hit

    // HIGH: spike heights from individual treble bins (fast attack, slower release)
    const b = binHz(), arr = u.uSpike.value;
    for (let i = 0; i < gl.SPIKES; i++) {
      let v = 0;
      if (live) {
        const bi = Math.round(gl.spikeHz[i] / b);
        const raw = (freq[bi - 1] + freq[bi] * 2 + freq[bi + 1]) / 4 / 255 * 1.6 * sens;
        v = clamp((raw - 0.25) / 0.75) * 0.6 * spikeK * f;
      }
      arr[i] += (v - arr[i]) * (v > arr[i] ? 0.6 : 0.12);
    }

    // COLOUR: four tone-derived colours, eased
    const tgt = oraclePalette();
    for (let c = 0; c < 4; c++) for (let i = 0; i < 3; i++) pal[c][i] += (tgt[c][i] - pal[c][i]) * 0.08;
    u.uC1.value.setRGB(...pal[0]); u.uC2.value.setRGB(...pal[1]); u.uC3.value.setRGB(...pal[2]); u.uC4.value.setRGB(...pal[3]);
    u.uRim.value.setRGB(...pal[1].map(v => 0.08 + v * 0.85));

    gl.mesh.rotation.y += dt * 0.00012 * (1 + A.level * 2) * f;
    gl.mesh.rotation.x = Math.sin(sphereTime * 0.15) * 0.25;
    gl.renderer.render(gl.scene, gl.camera);
  }

  // ---------- 2D visuals ----------
  const hueOf = off => (200 + A.centroid * 420 + off) % 360;
  const N = 900, parts = [];
  for (let i = 0; i < N; i++) { const f = i / N; parts.push({ a: Math.random() * 6.283, f, bin: Math.floor(2 * Math.pow(400, f)), r0: 0.12 + f * 0.3 + (Math.random() - 0.5) * 0.04, spin: (Math.random() < 0.5 ? -1 : 1) * (0.2 + Math.random() * 0.8) }); }
  let kick = 0;
  function drawDots(dt) {
    const R = Math.min(W, H) * 0.9, cxm = W / 2, cym = H / 2;
    kick = Math.max(kick * 0.9, A.beat);
    cx.globalCompositeOperation = 'lighter';
    for (const p of parts) {
      const m = freq[Math.min(p.bin, freq.length - 1)] / 255 * +$('sens').value;
      p.a += p.spin * dt * 0.0002 * (0.3 + A.level * 3);
      const r = R * (p.r0 + m * 0.18 + kick * 0.05 * (1 - p.f));
      cx.fillStyle = `hsla(${hueOf(p.f * 140)},85%,${45 + m * 30}%,${0.35 + m * 0.6})`;
      cx.beginPath(); cx.arc(cxm + Math.cos(p.a) * r, cym + Math.sin(p.a) * r * 0.92, 0.6 + m * 3.2, 0, 6.283); cx.fill();
    }
    cx.globalCompositeOperation = 'source-over';
  }
  function drawWaves(t) {
    const L = 9; cx.lineWidth = 1.6; cx.lineCap = 'round';
    for (let i = 0; i < L; i++) {
      const lo = 30 * Math.pow(2, i * 1.1), e = Math.min(1, band(lo, lo * 2.1) * +$('sens').value);
      const y0 = H * (0.12 + 0.76 * i / (L - 1)), amp = H * 0.06 * (0.15 + e * 1.6);
      const fx = 0.004 + i * 0.0018, sp = 0.0008 + i * 0.0003;
      cx.strokeStyle = `hsla(${hueOf(i * 22)},80%,${50 + e * 25}%,${0.35 + e * 0.6})`;
      cx.beginPath();
      for (let x = 0; x <= W; x += 5) {
        const w = (wave[Math.floor(x / W * (wave.length - 1))] - 128) / 128;
        const y = y0 + Math.sin(x * fx + t * sp + i) * amp + w * amp * 1.2 + Math.sin(x * 0.02 - t * 0.003) * A.beat * 10;
        x ? cx.lineTo(x, y) : cx.moveTo(x, y);
      }
      cx.stroke();
    }
  }
  function drawField(t) {
    const S = Math.max(W, H);
    const blobs = [
      { e: A.bass, h: hueOf(0), x: 0.5 + 0.18 * Math.cos(t * 0.0002), y: 0.55 + 0.12 * Math.sin(t * 0.00027), r: 0.25 + A.bass * 0.45 + A.beat * 0.1 },
      { e: A.mid, h: hueOf(110), x: 0.3 + 0.2 * Math.sin(t * 0.00031), y: 0.4 + 0.2 * Math.cos(t * 0.00023), r: 0.18 + A.mid * 0.4 },
      { e: A.high, h: hueOf(220), x: 0.72 + 0.15 * Math.cos(t * 0.00041), y: 0.35 + 0.22 * Math.sin(t * 0.00037), r: 0.12 + A.high * 0.35 }
    ];
    cx.globalCompositeOperation = 'lighter';
    for (const b of blobs) {
      const x = b.x * W, y = b.y * H, r = b.r * S, g = cx.createRadialGradient(x, y, 0, x, y, r);
      g.addColorStop(0, `hsla(${b.h},90%,${40 + b.e * 25}%,${0.18 + b.e * 0.5})`);
      g.addColorStop(1, `hsla(${b.h},90%,30%,0)`);
      cx.fillStyle = g; cx.fillRect(0, 0, W, H);
    }
    if (A.beat > 0.6) { cx.fillStyle = `hsla(${hueOf(40)},60%,70%,${(A.beat - 0.6) * 0.12})`; cx.fillRect(0, 0, W, H); }
    cx.globalCompositeOperation = 'source-over';
  }

  // ---------- modes & UI ----------
  let mode = gl ? 0 : 1;
  function setMode(m) {
    if (m === 0 && !gl) return;
    mode = m;
    for (let i = 0; i < 4; i++) $('mode' + i).setAttribute('aria-pressed', String(i === m));
    $('gl').hidden = m !== 0; cv.hidden = m === 0;
    $('trail').disabled = m === 0;
    $('oracleCtl').hidden = m !== 0;
    if (m !== 0) resize2d();
  }
  [0, 1, 2, 3].forEach(i => $('mode' + i).onclick = () => setMode(i));
  if (!gl) $('mode0').disabled = true;
  setMode(mode);
  function onScreenResize() {
    const sc = $('screen'), r = Math.min(devicePixelRatio || 1, 1.5);
    if (gl) gl.size(); if (mode !== 0) resize2d();
    $('resLabel').textContent = Math.round(sc.clientWidth * r) + ' × ' + Math.round(sc.clientHeight * r) + ' px';
  }
  if (window.ResizeObserver) new ResizeObserver(onScreenResize).observe($('screen')); else addEventListener('resize', onScreenResize);
  onScreenResize();

  function setFormat(portrait) {
    $('output').classList.toggle('portrait', portrait);
    $('fmtL').setAttribute('aria-pressed', String(!portrait));
    $('fmtP').setAttribute('aria-pressed', String(portrait));
    $('fmtLabel').textContent = 'Output · ' + (portrait ? '9:16' : '16:9');
    try { localStorage.setItem('oracle-format', portrait ? 'p' : 'l'); } catch {}
  }
  $('fmtL').onclick = () => setFormat(false);
  $('fmtP').onclick = () => setFormat(true);
  try { if (localStorage.getItem('oracle-format') === 'p') setFormat(true); } catch {}
  addEventListener('keydown', e => {
    if (e.target.tagName === 'INPUT' && e.target.type !== 'range') return;
    if (e.key >= '1' && e.key <= '4') setMode(+e.key - 1);
    else if (e.key === 'r' || e.key === 'R') setFormat(!$('output').classList.contains('portrait'));
    else if (e.key === 'h' || e.key === 'H') $('controls').classList.toggle('hidden');
    else if (e.key === 'f' || e.key === 'F') { const d = document; const p = d.fullscreenElement ? d.exitFullscreen() : $('wrap').requestFullscreen?.(); p?.catch?.(() => {}); }
  });

  // ---------- loop ----------
  let prev = performance.now(), frame = 0;
  function loop(t) {
    const dt = Math.min(50, t - prev); prev = t;
    analyse(t);
    if (mode === 0) drawSphere(dt);
    else {
      const trail = reduced ? 0.9 : +$('trail').value;
      cx.fillStyle = `rgba(11,12,16,${1 - trail * (mode === 3 ? 0.6 : 1)})`;
      cx.fillRect(0, 0, W, H);
      if (mode === 1) drawDots(dt); else if (mode === 2) drawWaves(t); else drawField(t);
    }
    if ((frame++ & 3) === 0) {
      [['Bass', A.bass], ['Mid', A.mid], ['High', A.high]].forEach(([k, v]) => { $('m' + k).style.width = (v * 100).toFixed(0) + '%'; $('v' + k).textContent = v.toFixed(2); });
      $('toneMark').style.left = (A.tone * 100).toFixed(1) + '%';
      $('vTone').textContent = A.tone < 0.33 ? 'LOW' : A.tone < 0.66 ? 'MID' : 'HIGH';
      const c = sphereRGB().map(v => Math.round(clamp(v) * 255));
      $('swatch').style.background = `rgb(${c[0]},${c[1]},${c[2]})`;
      $('beatDot').classList.toggle('on', A.beat > 0.5);
      $('vBpm').textContent = A.bpm ? A.bpm + ' bpm' : '— bpm';
    }
    requestAnimationFrame(loop);
  }
  requestAnimationFrame(loop);
})();
</script>
</body>
</html>
