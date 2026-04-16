# Presenter View Template

발표자 뷰 HTML을 작성할 때 참조하는 템플릿입니다.

---

## Table of Contents

1. [전체 HTML 구조](#전체-html-구조)
2. [스크립트 카드 작성법](#스크립트-카드-작성법)
3. [스크립트 서식](#스크립트-서식)

---

## 전체 HTML 구조

아래 템플릿에서 `{{PLACEHOLDER}}`를 실제 값으로 교체합니다.

교체할 항목:
- `{{TITLE}}` — 발표 제목
- `{{TOTAL_SLIDES}}` — 전체 슬라이드 수 (2곳: `.slide-total` span, JS `totalSlides`)
- `{{TARGET_MINUTES}}` — 목표 발표 시간(분) (2곳: input value, JS fallback)
- `{{PRESENTATION_FILENAME}}` — 슬라이드 HTML 파일명 (2곳: iframe src, JS 변수)
- `{{SCRIPT_CARDS}}` — 슬라이드별 스크립트 카드 (아래 섹션 참조)

```html
<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>발표 스크립트 - {{TITLE}}</title>
  <style>
    @import url('https://fonts.googleapis.com/css2?family=Noto+Sans+KR:wght@300;400;500;600;700&display=swap');
    * { margin: 0; padding: 0; box-sizing: border-box; }
    :root {
      --bg: #0a0a0a; --surface: #141414; --surface-light: #1a1a1a;
      --border: #2a2a2a; --text: #ffffff; --text-secondary: #888888;
      --text-muted: #555555; --accent: #3b82f6;
    }
    body { font-family: 'Noto Sans KR', -apple-system, BlinkMacSystemFont, sans-serif; background: var(--bg); color: var(--text); height: 100vh; overflow: hidden; }
    .header { position: fixed; top: 0; left: 0; right: 0; height: 56px; background: rgba(10,10,10,0.95); backdrop-filter: blur(20px); border-bottom: 1px solid var(--border); display: flex; align-items: center; justify-content: space-between; padding: 0 32px; z-index: 100; }
    .header-left { display: flex; align-items: center; gap: 16px; }
    .header-title { font-size: 14px; font-weight: 600; }
    .header-badge { background: var(--accent); color: white; padding: 4px 12px; border-radius: 12px; font-size: 11px; font-weight: 600; }
    .slide-indicator { display: flex; align-items: center; gap: 12px; }
    .slide-num { font-size: 24px; font-weight: 700; color: var(--accent); font-variant-numeric: tabular-nums; }
    .slide-total { font-size: 14px; color: var(--text-muted); }
    .header-right { display: flex; align-items: center; gap: 16px; }
    .pace-bars { position: fixed; top: 56px; left: 0; right: 0; z-index: 50; background: var(--bg); border-bottom: 1px solid var(--border); padding: 8px 32px; display: flex; flex-direction: column; gap: 6px; }
    .pace-row { display: flex; align-items: center; gap: 12px; }
    .pace-label { font-size: 11px; font-weight: 600; color: var(--text-muted); min-width: 80px; text-transform: uppercase; letter-spacing: 0.05em; }
    .pace-track { flex: 1; height: 8px; background: var(--surface-light); border-radius: 4px; overflow: hidden; }
    .pace-fill-time { height: 100%; background: #f59e0b; border-radius: 4px; transition: width 1s linear; width: 0%; }
    .pace-fill-slide { height: 100%; background: #3b82f6; border-radius: 4px; transition: width 0.3s; width: 0%; }
    .pace-value { font-size: 11px; font-weight: 700; font-variant-numeric: tabular-nums; min-width: 42px; text-align: right; }
    .pace-value.time { color: #f59e0b; } .pace-value.slide { color: #3b82f6; }
    .pace-status { font-size: 11px; font-weight: 700; min-width: 70px; text-align: center; padding: 2px 8px; border-radius: 4px; }
    .pace-status.on-pace { color: #22c55e; background: rgba(34,197,94,0.1); }
    .pace-status.too-slow { color: #ef4444; background: rgba(239,68,68,0.1); }
    .pace-status.too-fast { color: #f59e0b; background: rgba(245,158,11,0.1); }
    .main-container { margin-top: 112px; height: calc(100vh - 112px); display: flex; }
    .preview-panel { width: 75%; height: 100%; border-right: 1px solid var(--border); display: flex; flex-direction: column; background: var(--surface); }
    .preview-header { padding: 12px 20px; border-bottom: 1px solid var(--border); display: flex; align-items: center; justify-content: space-between; background: var(--bg); }
    .preview-label { font-size: 11px; font-weight: 600; color: var(--text-muted); text-transform: uppercase; letter-spacing: 0.1em; }
    .preview-controls { display: flex; gap: 8px; }
    .preview-btn { background: var(--surface); border: 1px solid var(--border); border-radius: 6px; padding: 6px 12px; color: var(--text-muted); font-size: 11px; cursor: pointer; transition: all 0.2s; }
    .preview-btn:hover { background: var(--surface-light); color: var(--text); }
    .preview-btn.active { background: var(--accent); border-color: var(--accent); color: white; }
    .preview-iframe-container { flex: 1; position: relative; overflow: hidden; display: flex; align-items: center; justify-content: center; background: #000; }
    .preview-iframe-wrapper { position: relative; overflow: hidden; }
    .preview-iframe { width: 1920px; height: 1080px; border: none; background: var(--bg); transform-origin: 0 0; }
    .preview-overlay { position: absolute; inset: 0; z-index: 10; cursor: pointer; }
    .preview-overlay:hover { background: rgba(59,130,246,0.05); }
    .pen-canvas { position: absolute; top: 0; left: 0; width: 100%; height: 100%; z-index: 15; pointer-events: none; }
    .pen-canvas.active { pointer-events: auto; cursor: crosshair; }
    .script-panel { width: 25%; height: 100%; display: flex; flex-direction: column; overflow: hidden; }
    .script-header-bar { padding: 12px 20px; border-bottom: 1px solid var(--border); display: flex; align-items: center; justify-content: space-between; background: var(--bg); }
    .script-label { font-size: 11px; font-weight: 600; color: var(--text-muted); text-transform: uppercase; letter-spacing: 0.1em; }
    .font-controls { display: flex; align-items: center; gap: 4px; }
    .font-btn { width: 28px; height: 28px; background: var(--surface); border: 1px solid var(--border); border-radius: 6px; color: var(--text-muted); font-size: 13px; font-weight: 700; cursor: pointer; display: flex; align-items: center; justify-content: center; font-family: inherit; transition: all 0.2s; }
    .font-btn:hover { background: var(--surface-light); color: var(--text); border-color: var(--accent); }
    .font-size-label { font-size: 10px; color: var(--text-muted); font-weight: 600; min-width: 28px; text-align: center; }
    .script-content { flex: 1; overflow-y: auto; padding: 20px; }
    .script-card { display: none; flex-direction: column; gap: 16px; }
    .script-card.active { display: flex; }
    .script-meta { display: flex; align-items: center; gap: 10px; padding-bottom: 12px; border-bottom: 1px solid var(--border); }
    .script-number { width: 36px; height: 36px; background: var(--accent); border-radius: 8px; display: flex; align-items: center; justify-content: center; font-size: 14px; font-weight: 700; flex-shrink: 0; }
    .script-info { flex: 1; min-width: 0; }
    .script-section { font-size: 9px; color: var(--accent); font-weight: 600; text-transform: uppercase; letter-spacing: 0.1em; margin-bottom: 2px; }
    .script-title { font-size: 18px; font-weight: 700; letter-spacing: -0.02em; line-height: 1.3; }
    .script-body { flex: 1; }
    .script-text { font-size: 16px; line-height: 1.8; }
    .script-text p { margin-bottom: 14px; }
    .script-text .emphasis { color: var(--accent); font-weight: 600; }
    .script-text .pause { display: inline-block; color: var(--text-muted); font-size: 11px; margin: 0 4px; }
    .script-text .action { display: block; background: var(--surface); border-left: 3px solid var(--accent); padding: 8px 12px; margin: 10px 0; font-size: 12px; color: var(--text-secondary); border-radius: 0 8px 8px 0; }
    .script-ref { margin-top: 16px; padding: 10px 14px; background: rgba(59,130,246,0.06); border-left: 2px solid rgba(59,130,246,0.3); border-radius: 0 6px 6px 0; font-size: 12px; color: var(--text-muted); line-height: 1.6; }
    .script-ref strong { color: var(--text-secondary); font-weight: 600; }
    .nav-buttons { position: fixed; bottom: 24px; right: 24px; display: flex; gap: 8px; z-index: 100; }
    .nav-btn { width: 48px; height: 48px; background: var(--surface); border: 1px solid var(--border); border-radius: 12px; color: var(--text-muted); font-size: 18px; cursor: pointer; display: flex; align-items: center; justify-content: center; transition: all 0.2s; }
    .nav-btn:hover { background: var(--surface-light); border-color: var(--accent); color: var(--text); }
    .bottom-progress-bar { position: fixed; bottom: 0; left: 0; height: 3px; background: var(--accent); transition: width 0.3s; z-index: 101; }
    .sync-status { display: flex; align-items: center; gap: 8px; font-size: 11px; color: var(--text-muted); }
    .sync-dot { width: 8px; height: 8px; background: #22c55e; border-radius: 50%; animation: pulse 2s infinite; }
    @keyframes pulse { 0%,100% { opacity: 1; } 50% { opacity: 0.5; } }
    .timer-container { display: flex; align-items: center; gap: 12px; padding-right: 16px; border-right: 1px solid var(--border); margin-right: 8px; }
    .current-time { font-size: 14px; font-weight: 600; color: var(--text-secondary); font-variant-numeric: tabular-nums; }
    .timer-controls { display: flex; align-items: center; gap: 8px; }
    .timer-display { font-size: 18px; font-weight: 700; color: var(--accent); font-variant-numeric: tabular-nums; min-width: 60px; text-align: center; }
    .timer-display.running { color: #22c55e; }
    .timer-display.warning { color: #f59e0b; }
    .timer-display.overtime { color: #ef4444; animation: blink 1s infinite; }
    @keyframes blink { 0%,100% { opacity: 1; } 50% { opacity: 0.5; } }
    .timer-btn { width: 28px; height: 28px; background: var(--surface); border: 1px solid var(--border); border-radius: 6px; color: var(--text-muted); font-size: 12px; cursor: pointer; display: flex; align-items: center; justify-content: center; transition: all 0.2s; }
    .timer-btn:hover { background: var(--surface-light); color: var(--text); border-color: var(--accent); }
    .timer-btn.active { background: var(--accent); border-color: var(--accent); color: white; }
    .time-setting { display: flex; align-items: center; gap: 4px; padding-right: 12px; border-right: 1px solid var(--border); margin-right: 4px; }
    .time-input { width: 42px; height: 24px; background: var(--surface); border: 1px solid var(--border); border-radius: 4px; color: var(--text); font-size: 12px; font-weight: 600; text-align: center; font-family: inherit; }
    .time-input:focus { outline: none; border-color: var(--accent); }
    .time-unit { font-size: 10px; color: var(--text-muted); }
    .mode-toggle { background: var(--surface); border: 1px solid var(--border); border-radius: 6px; padding: 6px 12px; color: var(--text-secondary); font-size: 11px; cursor: pointer; transition: all 0.2s; }
    .mode-toggle:hover { background: var(--surface-light); color: var(--text); }
    .mode-toggle.active { background: var(--accent); border-color: var(--accent); color: white; }
    .teleprompter-mode .script-text { font-size: 26px; line-height: 2; }
    .teleprompter-mode .script-meta { display: none; }
    .teleprompter-mode .script-text .action { font-size: 14px; }
    .teleprompter-mode .script-ref { display: none; }
    .fullscreen-script .preview-panel { display: none; }
    .fullscreen-script .script-panel { width: 100%; }
    .fullscreen-script .script-content { max-width: 900px; margin: 0 auto; padding: 40px; }
    .fullscreen-script .script-text { font-size: 26px; line-height: 1.9; }
    .fullscreen-script .script-title { font-size: 24px; }
    .script-content::-webkit-scrollbar { width: 4px; }
    .script-content::-webkit-scrollbar-track { background: transparent; }
    .script-content::-webkit-scrollbar-thumb { background: var(--border); border-radius: 2px; }
    @media (max-width: 1400px) { .preview-panel { width: 70%; } .script-panel { width: 30%; } }
    @media (max-width: 1100px) { .preview-panel { width: 60%; } .script-panel { width: 40%; } }
    @media (max-width: 768px) { .main-container { flex-direction: column; } .preview-panel { width: 100%; height: 40%; border-right: none; border-bottom: 1px solid var(--border); } .script-panel { width: 100%; height: 60%; } .header { padding: 0 16px; } }
  </style>
</head>
<body>
<header class="header">
  <div class="header-left">
    <span class="header-title">발표 스크립트</span>
    <span class="header-badge">PRESENTER VIEW</span>
  </div>
  <div class="slide-indicator">
    <span class="slide-num" id="current-slide">1</span>
    <span class="slide-total">/ {{TOTAL_SLIDES}}</span>
  </div>
  <div class="header-right">
    <div class="time-setting">
      <input type="number" class="time-input" id="time-input" value="{{TARGET_MINUTES}}" min="1" max="300" title="목표 시간(분)">
      <span class="time-unit">분</span>
    </div>
    <div class="timer-container">
      <div class="current-time" id="current-time">00:00:00</div>
      <div class="timer-controls">
        <div class="timer-display" id="timer-display">00:00</div>
        <button type="button" class="timer-btn" id="timer-start" onclick="toggleTimer()">&#9654;</button>
        <button type="button" class="timer-btn" onclick="resetTimer()">&#8634;</button>
      </div>
    </div>
    <button type="button" class="mode-toggle" id="fullscreen-toggle" onclick="toggleFullscreenScript()">스크립트만</button>
    <button type="button" class="mode-toggle" id="teleprompter-toggle" onclick="toggleTeleprompter()">텔레프롬프터</button>
    <div class="sync-status"><div class="sync-dot"></div><span>동기화</span></div>
  </div>
</header>
<div class="pace-bars">
  <div class="pace-row">
    <span class="pace-label">시간</span>
    <div class="pace-track"><div class="pace-fill-time" id="pace-time-fill"></div></div>
    <span class="pace-value time" id="pace-time-value">0%</span>
    <span class="pace-status on-pace" id="pace-status">대기중</span>
  </div>
  <div class="pace-row">
    <span class="pace-label">진도</span>
    <div class="pace-track"><div class="pace-fill-slide" id="pace-slide-fill"></div></div>
    <span class="pace-value slide" id="pace-slide-value">0%</span>
  </div>
</div>
<div class="main-container">
  <div class="preview-panel">
    <div class="preview-header">
      <span class="preview-label">슬라이드 미리보기 (16:9)</span>
      <div class="preview-controls">
        <button type="button" class="preview-btn" id="pen-toggle" onclick="togglePen()">&#9998; 펜</button>
        <button type="button" class="preview-btn" id="pen-clear" onclick="clearPen()">&#128465; 지우기</button>
        <button type="button" class="preview-btn" onclick="openInNewTab()">새 탭에서 열기 &#8599;</button>
      </div>
    </div>
    <div class="preview-iframe-container">
      <div class="preview-iframe-wrapper" id="preview-wrapper">
        <iframe id="preview-iframe" class="preview-iframe" src="{{PRESENTATION_FILENAME}}#1" title="슬라이드 미리보기"></iframe>
        <div class="preview-overlay" onclick="handlePreviewClick(event)"></div>
        <canvas id="pen-canvas" class="pen-canvas"></canvas>
      </div>
    </div>
  </div>
  <div class="script-panel">
    <div class="script-header-bar">
      <span class="script-label">발표 스크립트</span>
      <div class="font-controls">
        <button type="button" class="font-btn" onclick="changeFontSize(-2)">A-</button>
        <span class="font-size-label" id="font-size-label">16px</span>
        <button type="button" class="font-btn" onclick="changeFontSize(2)">A+</button>
      </div>
    </div>
    <div class="script-content">
      <!-- {{SCRIPT_CARDS}} -->
    </div>
  </div>
</div>
<div class="nav-buttons">
  <button type="button" class="nav-btn" onclick="prevSlide()">&#8592;</button>
  <button type="button" class="nav-btn" onclick="nextSlide()">&#8594;</button>
</div>
<div class="bottom-progress-bar" id="progress-bar"></div>
<script>
  const scripts = document.querySelectorAll('.script-card');
  const totalSlides = scripts.length;
  let currentSlide = 1;
  const previewIframe = document.getElementById('preview-iframe');
  const PRESENTATION_FILE = '{{PRESENTATION_FILENAME}}';
  let timerSeconds = 0, timerInterval = null, isTimerRunning = false;
  function getTargetMinutes() { return parseInt(document.getElementById('time-input').value) || {{TARGET_MINUTES}}; }
  document.addEventListener('DOMContentLoaded', function() { document.getElementById('time-input').addEventListener('change', function() { updateTimerDisplay(); updatePaceBars(); }); });
  const channel = new BroadcastChannel('presentation-sync');
  channel.onmessage = (event) => { if (event.data.type === 'slideChange') showSlide(event.data.slide, false); };
  function updateUI(n) { document.getElementById('current-slide').textContent = n; document.getElementById('progress-bar').style.width = (n / totalSlides) * 100 + '%'; }
  function updatePreview(n) { previewIframe.src = PRESENTATION_FILE + '#' + n; }
  function showSlide(n, broadcast = true) {
    if (n < 1) n = 1; if (n > totalSlides) n = totalSlides;
    scripts.forEach(s => s.classList.remove('active'));
    const target = document.querySelector('.script-card[data-slide="' + n + '"]');
    if (target) target.classList.add('active');
    currentSlide = n; updateUI(n); updatePreview(n); updatePaceBars();
    var pc = document.getElementById('pen-canvas');
    if (pc && pc.getContext) pc.getContext('2d').clearRect(0, 0, pc.width, pc.height);
    if (broadcast) channel.postMessage({ type: 'slideChange', slide: n });
  }
  function nextSlide() { showSlide(currentSlide + 1); }
  function prevSlide() { showSlide(currentSlide - 1); }
  function toggleTeleprompter() { document.body.classList.toggle('teleprompter-mode'); document.getElementById('teleprompter-toggle').classList.toggle('active'); }
  function toggleFullscreenScript() { document.body.classList.toggle('fullscreen-script'); document.getElementById('fullscreen-toggle').classList.toggle('active'); }
  function openInNewTab() { window.open(PRESENTATION_FILE + '#' + currentSlide, '_blank'); }
  function handlePreviewClick(event) {
    var overlay = event.currentTarget, rect = overlay.getBoundingClientRect();
    var x = (event.clientX - rect.left) * (1920 / rect.width), y = (event.clientY - rect.top) * (1080 / rect.height);
    previewIframe.contentWindow.postMessage({ type: 'spotlightClick', x: x, y: y }, '*');
  }
  document.addEventListener('keydown', e => {
    if (e.target.tagName === 'INPUT') return;
    if (e.key === 'ArrowRight' || e.key === ' ' || e.key === 'Enter') { e.preventDefault(); nextSlide(); }
    else if (e.key === 'ArrowLeft') { e.preventDefault(); prevSlide(); }
    else if (e.key === 'Home') { e.preventDefault(); showSlide(1); }
    else if (e.key === 'End') { e.preventDefault(); showSlide(totalSlides); }
    else if (e.key === 't' || e.key === 'T') toggleTeleprompter();
    else if (e.key === 'f' || e.key === 'F') toggleFullscreenScript();
  });
  showSlide(1, true);
  const IFRAME_W = 1920, IFRAME_H = 1080;
  function resizePreview() {
    const c = document.querySelector('.preview-iframe-container'), w = document.getElementById('preview-wrapper'), f = document.getElementById('preview-iframe');
    if (!c||!w||!f) return;
    const cW = c.clientWidth, cH = c.clientHeight, ar = IFRAME_W/IFRAME_H;
    let pw, ph; if (cW/cH > ar) { ph = cH; pw = ph*ar; } else { pw = cW; ph = pw/ar; }
    w.style.width = pw+'px'; w.style.height = ph+'px'; f.style.transform = 'scale('+(pw/IFRAME_W)+')';
  }
  window.addEventListener('resize', resizePreview); window.addEventListener('load', resizePreview);
  setTimeout(resizePreview, 100); setTimeout(resizePreview, 500);
  function updateTimerDisplay() {
    const m = Math.floor(timerSeconds/60), s = timerSeconds%60, d = document.getElementById('timer-display');
    d.textContent = String(m).padStart(2,'0')+':'+String(s).padStart(2,'0');
    const t = getTargetMinutes(); d.classList.remove('running','warning','overtime');
    if (isTimerRunning) { if (timerSeconds >= t*60) d.classList.add('overtime'); else if (timerSeconds >= (t-5)*60) d.classList.add('warning'); else d.classList.add('running'); }
  }
  function toggleTimer() {
    const btn = document.getElementById('timer-start');
    if (isTimerRunning) { clearInterval(timerInterval); isTimerRunning = false; btn.textContent = '\u25B6'; btn.classList.remove('active'); }
    else { isTimerRunning = true; btn.textContent = '\u23F8'; btn.classList.add('active'); timerInterval = setInterval(() => { timerSeconds++; updateTimerDisplay(); updatePaceBars(); }, 1000); }
    updateTimerDisplay();
  }
  function resetTimer() { clearInterval(timerInterval); isTimerRunning = false; timerSeconds = 0; document.getElementById('timer-start').textContent = '\u25B6'; document.getElementById('timer-start').classList.remove('active'); updateTimerDisplay(); updatePaceBars(); }
  function updateCurrentTime() { const n = new Date(); document.getElementById('current-time').textContent = String(n.getHours()).padStart(2,'0')+':'+String(n.getMinutes()).padStart(2,'0')+':'+String(n.getSeconds()).padStart(2,'0'); }
  function updatePaceBars() {
    const t = getTargetMinutes(), tp = Math.min((timerSeconds/(t*60))*100,100), sp = (currentSlide/totalSlides)*100;
    document.getElementById('pace-time-fill').style.width = tp+'%'; document.getElementById('pace-slide-fill').style.width = sp+'%';
    document.getElementById('pace-time-value').textContent = Math.round(tp)+'%'; document.getElementById('pace-slide-value').textContent = Math.round(sp)+'%';
    const st = document.getElementById('pace-status');
    if (!isTimerRunning && timerSeconds === 0) { st.textContent = '대기중'; st.className = 'pace-status on-pace'; return; }
    const diff = sp - tp;
    if (diff > 10) { st.textContent = '빠름'; st.className = 'pace-status too-fast'; }
    else if (diff < -10) { st.textContent = '느림'; st.className = 'pace-status too-slow'; }
    else { st.textContent = '적절'; st.className = 'pace-status on-pace'; }
  }
  let currentFontSize = 16;
  function changeFontSize(delta) { currentFontSize = Math.max(10, Math.min(36, currentFontSize + delta)); document.querySelectorAll('.script-text').forEach(el => { el.style.fontSize = currentFontSize + 'px'; }); document.getElementById('font-size-label').textContent = currentFontSize + 'px'; }
  let penActive = false, penDrawing = false;
  const penCanvas = document.getElementById('pen-canvas'), penCtx = penCanvas.getContext('2d');
  function initPenCanvas() { penCanvas.width = 1920; penCanvas.height = 1080; penCtx.strokeStyle = '#ff4444'; penCtx.lineWidth = 4; penCtx.lineCap = 'round'; penCtx.lineJoin = 'round'; }
  initPenCanvas();
  function togglePen() { penActive = !penActive; penCanvas.classList.toggle('active', penActive); document.getElementById('pen-toggle').classList.toggle('active', penActive); }
  function clearPen() { penCtx.clearRect(0, 0, penCanvas.width, penCanvas.height); channel.postMessage({ type: 'penClear' }); }
  function getPenCoords(e) { var r = penCanvas.getBoundingClientRect(); return { x: (e.clientX-r.left)/r.width*1920, y: (e.clientY-r.top)/r.height*1080 }; }
  penCanvas.addEventListener('mousedown', function(e) { if (!penActive) return; penDrawing = true; var p = getPenCoords(e); penCtx.beginPath(); penCtx.moveTo(p.x, p.y); channel.postMessage({ type: 'penStart', x: p.x, y: p.y }); });
  penCanvas.addEventListener('mousemove', function(e) { if (!penDrawing) return; var p = getPenCoords(e); penCtx.lineTo(p.x, p.y); penCtx.stroke(); channel.postMessage({ type: 'penMove', x: p.x, y: p.y }); });
  window.addEventListener('mouseup', function() { if (penDrawing) { penDrawing = false; channel.postMessage({ type: 'penEnd' }); } });
  setInterval(updateCurrentTime, 1000); updateCurrentTime(); updateTimerDisplay(); updatePaceBars();
</script>
</body>
</html>
```

---

## 스크립트 카드 작성법

각 슬라이드마다 하나의 `.script-card`를 작성합니다. 첫 번째 카드에만 `class="script-card active"`를 줍니다.

```html
<div class="script-card active" data-slide="1">
  <div class="script-meta">
    <div class="script-number">1</div>
    <div class="script-info">
      <div class="script-section">Opening</div>
      <h1 class="script-title">슬라이드 제목</h1>
    </div>
  </div>
  <div class="script-body">
    <div class="script-text">
      <p>실제로 말할 내용을 자연스러운 구어체로 작성합니다.</p>
      <p><span class="emphasis">강조할 키워드</span>는 이렇게 표시합니다.</p>
      <span class="action">(화면 전환: 데모 시작)</span>
    </div>
    <div class="script-ref">
      <strong>참고:</strong> 보조 정보, 수치, 출처 등
    </div>
  </div>
</div>
```

---

## 스크립트 서식

| 서식 | HTML | 용도 |
|------|------|------|
| 강조 | `<span class="emphasis">텍스트</span>` | 핵심 키워드 (파란색) |
| 무대 지시 | `<span class="action">텍스트</span>` | 화면 전환, 데모 시작 등 |
| 쉼표 | `<span class="pause">[2초]</span>` | 잠깐 멈추는 지점 |
| 참고 | `<div class="script-ref">` | 수치, 출처 등 보조 정보 |
