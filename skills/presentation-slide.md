---
name: presentation-slide
description: HTML 프레젠테이션 슬라이드 생성 스킬. 발표자 뷰(스크립트+자막)와 BroadcastChannel 동기화, 하이라이트, 펜 그리기, 타이머, 페이스 바 등 발표 보조 기능이 포함된 단일 HTML 프레젠테이션을 생성합니다.
---

# HTML Presentation Slide Generator

사용자의 발표 주제와 내용을 기반으로 **두 개의 연동 HTML 파일**을 생성하는 스킬입니다:
1. **프레젠테이션 슬라이드** (청중에게 보여주는 화면)
2. **발표자 뷰** (스크립트, 슬라이드 미리보기, 타이머 등 발표 보조 도구)

두 파일은 `BroadcastChannel`로 실시간 동기화됩니다.

---

## 사용 시점

- 사용자가 "발표 자료 만들어줘", "PPT 만들어줘", "프레젠테이션 만들어줘" 등을 요청할 때
- 사용자가 슬라이드와 발표 스크립트를 함께 원할 때
- HTML 기반 슬라이드쇼가 필요할 때

---

## 생성 절차

### Step 1: 콘텐츠 구조화

사용자의 발표 내용을 아래 구조로 정리합니다:

```
발표 제목: ...
발표자: ...
목표 시간: ... 분
섹션 구성:
  - 섹션 1: [이름] (슬라이드 N~M)
  - 섹션 2: [이름] (슬라이드 N~M)
  ...
```

### Step 2: 슬라이드 타입 선택

각 슬라이드에 적합한 타입을 선택합니다:

| 타입 | 클래스 | 용도 |
|------|--------|------|
| 커버 | `.cover` | 첫 슬라이드. 제목, 부제, 발표자 |
| 일반 | (기본) | 제목 + 콘텐츠 조합 |
| 섹션 구분 | `.slide--section` | 새 섹션 시작. 번호 + 제목 + 설명 |
| 데모 | `.slide--demo` | 라이브 데모 안내. 이모지 + 제목 + 설명 |
| 센터 | `.center-content` | 중앙 정렬 메시지 (Q&A, 핵심 메시지 등) |

### Step 3: 콘텐츠 컴포넌트 조합

슬라이드 내부에 아래 컴포넌트를 조합합니다:

| 컴포넌트 | 클래스 | 설명 |
|----------|--------|------|
| 카드 그리드 | `.grid.grid--2/3/4` + `.card` | 2~4열 카드 레이아웃 |
| 하이라이트 박스 | `.highlight-box` | 강조 메시지 영역 |
| 플로우 | `.flow` + `.flow-item` + `.flow-arrow` | INPUT → PROCESS → OUTPUT 흐름도 |
| 비교 | `.compare` + `.compare-item.good/.bad` | 좋은 예 / 나쁜 예 비교 |
| 코드 블록 | `.code-block` | 코드 또는 터미널 출력 표시 |
| 스텝 | `.steps` + `.step` | 순서가 있는 단계 (번호 포함) |
| 테이블 | `.table-wrap` + `table` | 데이터 테이블 |
| 리스트 | `.list` | 불릿 리스트 |
| 키커 | `.kicker` | 작은 라벨/뱃지 |
| 퍼즐 조합 | `.puzzle-row` + `.puzzle-piece` | A + B = C 형태의 조합 표현 |
| 타임 뱃지 | `.time-badge` / `.time-badge.green/.orange/.purple` | 시간 표시 |

---

## 프레젠테이션 슬라이드 HTML 템플릿

아래 템플릿을 기반으로 슬라이드를 생성합니다. `{{PLACEHOLDER}}`는 실제 콘텐츠로 교체합니다.

```html
<!doctype html>
<html lang="ko">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>{{PRESENTATION_TITLE}}</title>
  <style>
    @import url('https://fonts.googleapis.com/css2?family=Noto+Sans+KR:wght@400;500;700;900&family=JetBrains+Mono:wght@400;500;600&display=swap');

    :root {
      --bg: #0a0a0a;
      --surface: #141414;
      --surface-light: #1a1a1a;
      --border: #2a2a2a;
      --text: #ffffff;
      --text-secondary: #888;
      --text-muted: #555;
      --accent: #3b82f6;
      --accent-light: #60a5fa;
      --danger: #ef4444;
      --success: #22c55e;
      --warning: #f59e0b;
      --purple: #8b5cf6;
    }

    * { box-sizing: border-box; margin: 0; padding: 0; }

    html, body {
      width: 100%; height: 100%;
      background: var(--bg);
      color: var(--text);
      font-family: 'Noto Sans KR', sans-serif;
      overflow: hidden;
    }

    .presentation {
      position: relative;
      width: 100vw; height: 100vh;
      background: linear-gradient(160deg, #0d0d0d 0%, #090909 50%, #050505 100%);
    }

    /* ── Top Section Bar ── */
    .sections-bar {
      position: fixed; top: 0; left: 0; right: 0;
      display: flex; gap: 6px; justify-content: center; align-items: center;
      padding: 12px 16px;
      background: rgba(10,10,10,0.92); border-bottom: 1px solid var(--border);
      backdrop-filter: blur(8px); z-index: 30;
    }
    .section-chip {
      border: 1px solid var(--border); background: var(--surface);
      color: var(--text-secondary); padding: 7px 12px; border-radius: 999px;
      font-size: 13px; font-weight: 500; cursor: pointer; transition: all .2s; white-space: nowrap;
    }
    .section-chip:hover { color: var(--text); border-color: var(--accent); }
    .section-chip.active {
      color: var(--text); background: rgba(59,130,246,.22);
      border-color: var(--accent); box-shadow: 0 0 0 1px rgba(59,130,246,.35) inset;
    }

    /* ── Slides ── */
    .slides-wrap { width: 100%; height: 100%; padding-top: 58px; padding-bottom: 80px; }

    .slide {
      width: 100%; height: 100%; display: none;
      padding: 60px 120px; overflow: auto; position: relative;
    }
    .slide.active {
      display: flex; flex-direction: column; gap: 28px;
      animation: fadeIn .3s ease;
    }
    @keyframes fadeIn {
      from { opacity: 0; transform: translateY(10px); }
      to { opacity: 1; transform: translateY(0); }
    }

    /* ── Cover Slide ── */
    .cover { justify-content: center; align-items: flex-start; }
    .cover-title { font-size: 64px; line-height: 1.1; letter-spacing: -.02em; max-width: 1100px; }
    .cover-subtitle { color: var(--accent); font-size: 30px; font-weight: 700; }
    .cover-presenter { color: var(--text-secondary); font-size: 24px; }

    /* ── Slide Content ── */
    .slide-title { font-size: 46px; line-height: 1.15; letter-spacing: -.02em; }
    .slide-subtitle { color: var(--text-secondary); font-size: 22px; line-height: 1.5; max-width: 1100px; }

    /* ── Section Divider ── */
    .slide--section {
      align-items: center; justify-content: center; text-align: center;
      background:
        radial-gradient(circle at 15% 30%, rgba(59,130,246,.2), transparent 42%),
        radial-gradient(circle at 85% 70%, rgba(59,130,246,.18), transparent 45%);
    }
    .section-number { font-size: 90px; font-weight: 900; color: rgba(59,130,246,.85); text-shadow: 0 0 30px rgba(59,130,246,.32); }
    .section-divider-title { font-size: 56px; letter-spacing: -.03em; }
    .section-divider-desc { font-size: 24px; color: var(--text-secondary); }

    /* ── Demo Slide ── */
    .slide--demo {
      justify-content: center; align-items: center; text-align: center;
      position: relative; overflow: hidden;
    }
    .slide--demo::before {
      content: ''; position: absolute; inset: 40px; border-radius: 26px;
      border: 1px solid rgba(59,130,246,.55);
      box-shadow: 0 0 35px rgba(59,130,246,.2), inset 0 0 35px rgba(59,130,246,.12);
      animation: pulseBorder 3.2s ease-in-out infinite; pointer-events: none;
    }
    @keyframes pulseBorder {
      0%,100% { opacity: .55; transform: scale(1); }
      50% { opacity: .95; transform: scale(1.01); }
    }
    .demo-emoji { font-size: 100px; filter: drop-shadow(0 0 16px rgba(59,130,246,.45)); }
    .demo-title { font-size: 56px; line-height: 1.15; letter-spacing: -.02em; }
    .demo-desc { font-size: 28px; line-height: 1.4; color: #d7e6ff; max-width: 1020px; }

    /* ── Layout Helpers ── */
    .grid { display: grid; gap: 20px; }
    .grid--2 { grid-template-columns: repeat(2, 1fr); }
    .grid--3 { grid-template-columns: repeat(3, 1fr); }
    .grid--4 { grid-template-columns: repeat(4, 1fr); }

    .card {
      background: linear-gradient(160deg, rgba(255,255,255,.04), rgba(255,255,255,.02));
      border: 1px solid var(--border); border-radius: 16px; padding: 24px;
    }
    .card-title { font-size: 24px; margin-bottom: 12px; line-height: 1.3; }
    .card-desc { color: var(--text-secondary); font-size: 18px; line-height: 1.6; }

    .highlight-box {
      background: linear-gradient(120deg, rgba(59,130,246,.18), rgba(59,130,246,.08));
      border: 1px solid rgba(59,130,246,.48); border-radius: 18px; padding: 22px 26px;
    }
    .highlight-box .text { font-size: 20px; line-height: 1.6; color: #e6f0ff; font-weight: 500; }

    .flow { display: flex; align-items: stretch; gap: 10px; flex-wrap: wrap; }
    .flow-item {
      flex: 1; min-width: 140px; background: var(--surface); border: 1px solid var(--border);
      border-radius: 14px; padding: 16px; display: flex; flex-direction: column;
      justify-content: center; text-align: center;
    }
    .flow-label { font-size: 20px; line-height: 1.4; font-weight: 700; }
    .flow-arrow { align-self: center; font-size: 30px; color: var(--accent); padding: 0 2px; }

    .kicker {
      display: inline-flex; width: fit-content; padding: 5px 11px; border-radius: 999px;
      border: 1px solid rgba(59,130,246,.45); color: #d8e7ff;
      background: rgba(59,130,246,.16); font-size: 14px; font-weight: 600;
    }

    .mini-note { color: var(--text-muted); font-size: 16px; line-height: 1.6; }

    .list { padding-left: 24px; display: flex; flex-direction: column; gap: 12px; }
    .list li { font-size: 20px; line-height: 1.6; color: var(--text); }
    .list li::marker { color: var(--accent); }

    .compare { display: grid; grid-template-columns: repeat(2, 1fr); gap: 16px; }
    .compare-item { border-radius: 14px; border: 1px solid var(--border); padding: 20px; background: var(--surface); }
    .compare-label { font-size: 16px; color: var(--text-secondary); margin-bottom: 8px; }
    .compare-text { font-size: 22px; line-height: 1.45; }
    .compare-item.good { border-color: rgba(34,197,94,.45); background: linear-gradient(140deg, rgba(34,197,94,.14), rgba(20,20,20,.92)); }
    .compare-item.bad { border-color: rgba(239,68,68,.42); background: linear-gradient(140deg, rgba(239,68,68,.13), rgba(20,20,20,.92)); }

    .code-block {
      background: #111; border: 1px solid var(--border); border-radius: 14px;
      padding: 20px 24px; font-family: 'JetBrains Mono', monospace; font-size: 15px;
      line-height: 1.7; color: #ccc; overflow-x: auto;
    }
    .code-block .cmd { color: var(--success); }
    .code-block .cmt { color: #666; }
    .code-block .str { color: var(--warning); }

    .steps { display: grid; grid-template-columns: repeat(3, 1fr); gap: 18px; }
    .step { border: 1px solid var(--border); border-radius: 16px; background: var(--surface); padding: 20px; }
    .step-no {
      display: inline-flex; width: 34px; height: 34px; border-radius: 50%;
      align-items: center; justify-content: center;
      background: rgba(59,130,246,.25); color: var(--accent); font-weight: 700; margin-bottom: 10px;
    }
    .step-title { font-size: 22px; margin-bottom: 8px; line-height: 1.35; }
    .step-desc { color: var(--text-secondary); font-size: 18px; line-height: 1.55; }

    .table-wrap { border: 1px solid var(--border); border-radius: 14px; overflow: hidden; }
    table { width: 100%; border-collapse: collapse; background: var(--surface); }
    th, td { font-size: 18px; text-align: left; vertical-align: top; padding: 14px 16px; border-bottom: 1px solid var(--border); line-height: 1.5; }
    th { background: var(--surface-light); color: #d6d6d6; font-weight: 700; }
    tr:last-child td { border-bottom: 0; }

    .time-badge {
      display: inline-block; padding: 4px 10px; border-radius: 8px;
      background: rgba(59,130,246,.2); color: var(--accent-light);
      font-size: 14px; font-weight: 600; margin-bottom: 8px;
    }
    .time-badge.green { background: rgba(34,197,94,.2); color: #86efac; }
    .time-badge.orange { background: rgba(245,158,11,.2); color: #fbbf24; }
    .time-badge.purple { background: rgba(139,92,246,.2); color: #c4b5fd; }

    .center-content {
      display: flex; justify-content: center; align-items: center;
      text-align: center; width: 100%; height: 100%; flex-direction: column; gap: 20px;
    }

    .puzzle-row { display: flex; flex-wrap: wrap; gap: 12px; align-items: center; }
    .puzzle-piece {
      padding: 14px 18px; border-radius: 12px; border: 1px solid var(--border);
      background: var(--surface); font-size: 20px; font-weight: 600;
    }
    .puzzle-plus, .puzzle-eq { font-size: 28px; color: var(--accent); font-weight: 700; }

    /* ── Bottom Controls ── */
    .bottom-controls {
      position: fixed; bottom: 14px; left: 0; right: 0;
      display: flex; justify-content: center; align-items: center; gap: 16px;
      z-index: 40; pointer-events: none;
    }
    .nav-btn, .slide-counter { pointer-events: auto; }
    .nav-btn {
      width: 50px; height: 50px; border-radius: 50%;
      border: 1px solid var(--border); background: rgba(20,20,20,.85);
      color: var(--text); font-size: 26px; cursor: pointer; transition: .2s;
    }
    .nav-btn:hover { transform: translateY(-2px); border-color: var(--accent); }
    .slide-counter {
      border: 1px solid var(--border); background: rgba(20,20,20,.85);
      padding: 8px 18px; border-radius: 999px; color: var(--text-secondary);
      font-size: 16px; min-width: 110px; text-align: center;
    }
    .progress-track { position: fixed; left: 0; right: 0; bottom: 0; height: 5px; background: #111; z-index: 45; }
    .progress-bar {
      height: 100%; width: 0%;
      background: linear-gradient(90deg, #2563eb, #3b82f6 60%, #60a5fa);
      box-shadow: 0 0 18px rgba(59,130,246,.6); transition: width .24s ease;
    }

    /* ── Spotlight ── */
    @keyframes spotlightGlow {
      0% { box-shadow: 0 0 0 rgba(59,130,246,0); transform: scale(1); }
      40% { box-shadow: 0 0 32px rgba(59,130,246,0.6), inset 0 0 16px rgba(59,130,246,0.15); transform: scale(1.03); }
      100% { box-shadow: 0 0 0 rgba(59,130,246,0); transform: scale(1); }
    }
    .spotlight {
      animation: spotlightGlow 1.5s ease;
      border-radius: 12px;
      position: relative;
      z-index: 10;
    }

    .pen-overlay {
      position: fixed; inset: 0; z-index: 100; pointer-events: none;
    }

    /* ── Responsive ── */
    @media (max-width: 1400px) {
      .slide { padding: 50px 60px; }
      .cover-title { font-size: 52px; }
      .slide-title { font-size: 40px; }
      .grid--3, .steps { grid-template-columns: repeat(2, 1fr); }
    }
    @media (max-width: 980px) {
      .slide { padding: 40px 24px 70px; }
      .grid--2, .grid--3, .grid--4, .compare, .steps { grid-template-columns: 1fr; }
      .cover-title { font-size: 42px; }
      .slide-title { font-size: 36px; }
      .sections-bar { justify-content: flex-start; overflow-x: auto; padding: 8px; }
    }
  </style>
</head>
<body>
  <div class="presentation">

    <!-- ══════ Section Navigation ══════ -->
    <div class="sections-bar" id="sectionsBar">
      <!-- {{SECTION_CHIPS}} -->
      <!-- 예시:
      <button class="section-chip" data-slide="1">Opening</button>
      <button class="section-chip" data-slide="5">본론</button>
      -->
    </div>

    <div class="slides-wrap">

      <!-- ══════ 슬라이드 영역 ══════ -->

      <!--
        [커버 슬라이드 예시]
        <div class="slide cover" id="slide1">
          <p class="kicker">Week 1 · 90분</p>
          <h1 class="cover-title">발표 제목</h1>
          <p class="cover-subtitle">부제목</p>
          <p class="cover-presenter">발표자 이름 · 소속</p>
        </div>
      -->

      <!--
        [일반 슬라이드 예시]
        <div class="slide" id="slide2">
          <h2 class="slide-title">슬라이드 제목</h2>
          <p class="slide-subtitle">설명 텍스트</p>
          <div class="grid grid--2">
            <div class="card">
              <h3 class="card-title">카드 제목</h3>
              <p class="card-desc">카드 설명</p>
            </div>
            <div class="card">
              <h3 class="card-title">카드 제목</h3>
              <p class="card-desc">카드 설명</p>
            </div>
          </div>
        </div>
      -->

      <!--
        [섹션 구분 슬라이드 예시]
        <div class="slide slide--section" id="slide3">
          <p class="section-number">01</p>
          <h2 class="section-divider-title">섹션 제목</h2>
          <p class="section-divider-desc">섹션 설명</p>
        </div>
      -->

      <!--
        [데모 슬라이드 예시]
        <div class="slide slide--demo" id="slide4">
          <p class="demo-emoji">🎬</p>
          <h2 class="demo-title">데모 제목</h2>
          <p class="demo-desc">데모 설명</p>
        </div>
      -->

      <!--
        [센터 콘텐츠 슬라이드 예시]
        <div class="slide" id="slide5">
          <div class="center-content">
            <h2 style="font-size:110px;">Q&A</h2>
            <p style="font-size:28px;color:var(--text-secondary);">질문해 주세요</p>
          </div>
        </div>
      -->

      <!-- {{SLIDES}} -->

    </div>

    <!-- ══════ Bottom Controls ══════ -->
    <div class="bottom-controls">
      <button class="nav-btn" id="prevBtn" aria-label="이전 슬라이드">&#8592;</button>
      <div class="slide-counter" id="slideCounter">1 / {{TOTAL_SLIDES}}</div>
      <button class="nav-btn" id="nextBtn" aria-label="다음 슬라이드">&#8594;</button>
    </div>
    <div class="progress-track"><div class="progress-bar" id="progressBar"></div></div>

  </div>

  <canvas id="pen-overlay" class="pen-overlay"></canvas>

  <script>
  (function(){
    var total = {{TOTAL_SLIDES}};
    var sectionStarts = [{{SECTION_START_SLIDES}}]; // 예: [1, 5, 10, 15]
    var chips = Array.from(document.querySelectorAll('.section-chip'));
    var counter = document.getElementById('slideCounter');
    var bar = document.getElementById('progressBar');
    var cur = 1;
    var bc = null;

    // Pen overlay
    var penOverlay = document.getElementById('pen-overlay');
    var penOvCtx = penOverlay.getContext('2d');
    function initPenOverlay() {
      penOverlay.width = window.innerWidth;
      penOverlay.height = window.innerHeight;
      penOvCtx.strokeStyle = '#ff4444';
      penOvCtx.lineWidth = 4;
      penOvCtx.lineCap = 'round';
      penOvCtx.lineJoin = 'round';
    }
    initPenOverlay();
    window.addEventListener('resize', initPenOverlay);

    // BroadcastChannel 동기화
    if ('BroadcastChannel' in window) {
      bc = new BroadcastChannel('presentation-sync');
      bc.onmessage = function(event) {
        var data = event.data || {};
        if (data.type === 'slideChange' && Number.isInteger(data.slide)) {
          if (data.slide !== cur) goToSlide(data.slide, true, false);
        }
        if (data.type === 'elementHighlight' && Number.isInteger(data.index) && Number.isInteger(data.slideNo)) {
          var slide = document.getElementById('slide' + data.slideNo);
          if (!slide) return;
          var allEls = Array.from(slide.querySelectorAll('*'));
          var target = allEls[data.index];
          applySpotlight(target);
        }
        if (data.type === 'penStart') {
          penOvCtx.beginPath();
          penOvCtx.moveTo(data.x / 1920 * penOverlay.width, data.y / 1080 * penOverlay.height);
        }
        if (data.type === 'penMove') {
          penOvCtx.lineTo(data.x / 1920 * penOverlay.width, data.y / 1080 * penOverlay.height);
          penOvCtx.stroke();
        }
        if (data.type === 'penClear') {
          penOvCtx.clearRect(0, 0, penOverlay.width, penOverlay.height);
        }
      };
    }

    function clamp(n){ return Math.min(total, Math.max(1, Math.floor(n))); }

    function secIdx(n){
      var idx=0;
      for(var i=0;i<sectionStarts.length;i++) if(n>=sectionStarts[i]) idx=i;
      return idx;
    }

    function goToSlide(n, hash, broadcast){
      if(broadcast === undefined) broadcast = true;
      var t = clamp(n);
      var old = document.getElementById('slide'+cur);
      var next = document.getElementById('slide'+t);
      if(!next) return;
      if(old) old.classList.remove('active');
      next.classList.add('active');
      cur = t;
      penOvCtx.clearRect(0, 0, penOverlay.width, penOverlay.height);
      counter.textContent = cur + ' / ' + total;
      bar.style.width = ((cur-1)/(total-1)*100).toFixed(2)+'%';
      chips.forEach(function(c,i){ c.classList.toggle('active', i===secIdx(cur)); });
      if(hash) history.replaceState(null,'','#'+cur);
      if(broadcast && bc) bc.postMessage({ type: 'slideChange', slide: cur });
    }

    // 스포트라이트 (클릭 하이라이트)
    function findSpotlightTarget(el) {
      if (!el) return null;
      var box = el.closest('.card, .highlight-box, article, .compare-item, .flow-item, .step, .puzzle-piece');
      if (box && !box.classList.contains('slide')) return box;
      var styledDiv = el.closest('div[style]');
      if (styledDiv && styledDiv.style.background && !styledDiv.classList.contains('slide')) {
        if (!styledDiv.querySelector('.card, article, .highlight-box')) return styledDiv;
      }
      return el.closest('.slide-title, .slide-subtitle, .section-divider-title, .section-divider-desc, p, h2, h3, li, td, code') || el;
    }

    function applySpotlight(target) {
      if (!target || target.classList.contains('slide')) return;
      target.classList.remove('spotlight');
      void target.offsetWidth;
      target.classList.add('spotlight');
      target.addEventListener('animationend', function() {
        target.classList.remove('spotlight');
      }, { once: true });
    }

    document.querySelector('.slides-wrap').addEventListener('click', function(e) {
      var target = findSpotlightTarget(e.target);
      if (!target || target.classList.contains('slide')) return;
      applySpotlight(target);
      var slide = document.getElementById('slide' + cur);
      if (slide && bc) {
        var allEls = Array.from(slide.querySelectorAll('*'));
        var idx = allEls.indexOf(target);
        bc.postMessage({ type: 'elementHighlight', slideNo: cur, index: idx });
      }
    });

    window.addEventListener('message', function(event) {
      var data = event.data || {};
      if (data.type === 'spotlightClick') {
        var el = document.elementFromPoint(data.x, data.y);
        var target = findSpotlightTarget(el);
        if (!target || target.classList.contains('slide')) return;
        applySpotlight(target);
        var slide = document.getElementById('slide' + cur);
        if (slide && bc) {
          var allEls = Array.from(slide.querySelectorAll('*'));
          var idx = allEls.indexOf(target);
          bc.postMessage({ type: 'elementHighlight', slideNo: cur, index: idx });
        }
      }
      if (data.type === 'slideChange' && data.slide !== cur) {
        goToSlide(data.slide, true, false);
      }
    });

    document.getElementById('prevBtn').addEventListener('click', function(){ goToSlide(cur-1,true); });
    document.getElementById('nextBtn').addEventListener('click', function(){ goToSlide(cur+1,true); });
    chips.forEach(function(c){ c.addEventListener('click', function(){ goToSlide(+c.dataset.slide,true); }); });

    document.addEventListener('keydown', function(e){
      if(e.key==='ArrowRight'||e.key===' '||e.key==='Enter'){ e.preventDefault(); goToSlide(cur+1,true); }
      else if(e.key==='ArrowLeft'){ e.preventDefault(); goToSlide(cur-1,true); }
    });

    window.addEventListener('hashchange', function(){
      var h = parseInt(location.hash.replace('#',''),10);
      if(h && h!==cur) goToSlide(h, false, false);
    });

    var h = parseInt(location.hash.replace('#',''),10);
    goToSlide(h||1, true, false);
  })();
  </script>
</body>
</html>
```

---

## 슬라이드 컴포넌트 레퍼런스

### 커버 슬라이드
```html
<div class="slide cover" id="slide1">
  <p class="kicker">Week 1 · 90분</p>
  <h1 class="cover-title">메인 타이틀<br>두 번째 줄</h1>
  <p class="cover-subtitle">서브타이틀 (accent color)</p>
  <p class="cover-presenter">발표자 · 소속 · 행사명</p>
</div>
```

### 카드 그리드 (2/3/4열)
```html
<div class="grid grid--3">
  <div class="card" style="text-align:center;">
    <p style="font-size:40px;margin-bottom:10px;">💡</p>
    <h3 class="card-title">카드 제목</h3>
    <p class="card-desc">카드 설명 텍스트</p>
  </div>
  <!-- 반복 -->
</div>
```

### 비교 (Good vs Bad)
```html
<div class="compare">
  <div class="compare-item bad">
    <p class="compare-label">나쁜 예</p>
    <p class="compare-text">나쁜 예시 텍스트</p>
  </div>
  <div class="compare-item good">
    <p class="compare-label">좋은 예</p>
    <p class="compare-text">좋은 예시 텍스트</p>
  </div>
</div>
```

### 플로우 (화살표 연결)
```html
<div class="flow">
  <div class="flow-item">
    <p class="flow-label">INPUT</p>
    <p style="font-size:15px;color:var(--text-secondary);">설명</p>
  </div>
  <div class="flow-arrow">&rarr;</div>
  <div class="flow-item">
    <p class="flow-label">PROCESS</p>
    <p style="font-size:15px;color:var(--text-secondary);">설명</p>
  </div>
  <div class="flow-arrow">&rarr;</div>
  <div class="flow-item">
    <p class="flow-label">OUTPUT</p>
    <p style="font-size:15px;color:var(--text-secondary);">설명</p>
  </div>
</div>
```

### 하이라이트 박스
```html
<div class="highlight-box">
  <p class="text">강조할 <strong>핵심 메시지</strong>를 여기에 작성합니다.</p>
</div>
```

### 스텝 (순서 단계)
```html
<div class="steps">
  <article class="step">
    <span class="step-no">1</span>
    <h3 class="step-title">단계 제목</h3>
    <p class="step-desc">단계 설명</p>
  </article>
  <!-- 반복 -->
</div>
```

### 코드 블록
```html
<div class="code-block">
<span class="cmd">명령어</span>  <span class="cmt"># 주석</span>
<span class="str">"문자열"</span>
</div>
```

### 테이블
```html
<div class="table-wrap">
  <table>
    <thead><tr><th>헤더1</th><th>헤더2</th></tr></thead>
    <tbody>
      <tr><td>데이터1</td><td>데이터2</td></tr>
    </tbody>
  </table>
</div>
```

### 섹션 구분 슬라이드
```html
<div class="slide slide--section" id="slideN">
  <p class="section-number">01</p>
  <h2 class="section-divider-title">섹션 타이틀</h2>
  <p class="section-divider-desc">섹션 설명</p>
</div>
```

### 퍼즐 조합
```html
<div class="puzzle-row">
  <div class="puzzle-piece">A 요소</div>
  <div class="puzzle-plus">+</div>
  <div class="puzzle-piece">B 요소</div>
  <div class="puzzle-eq">=</div>
  <div class="puzzle-piece" style="border-color:rgba(59,130,246,.5);">결과</div>
</div>
```

---

## 생성 시 규칙

1. **모든 슬라이드에 고유 ID 부여**: `id="slide1"`, `id="slide2"`, ...
2. **섹션 칩(section-chip)의 `data-slide` 속성**: 해당 섹션의 첫 번째 슬라이드 번호
3. **`total` 변수**: 전체 슬라이드 수와 일치
4. **`sectionStarts` 배열**: 각 섹션의 시작 슬라이드 번호 배열
5. **첫 슬라이드에 `.active` 클래스 추가**: `<div class="slide cover active" id="slide1">`
6. **파일명 규칙**: `{topic}-presentation.html` (발표자 뷰에서 참조)
7. **컴포넌트 조합은 자유**: 하나의 슬라이드 안에 grid + highlight-box + list 등 자유롭게 섞어 사용 가능
8. **accent 색상 커스텀 가능**: CSS 변수 `--accent`를 변경하여 테마 색상 변경

---

## 색상 커스터마이징

`:root`의 CSS 변수를 변경하면 전체 테마가 바뀝니다:

| 변수 | 기본값 | 용도 |
|------|--------|------|
| `--accent` | `#3b82f6` (blue) | 주요 강조색 |
| `--bg` | `#0a0a0a` | 배경색 |
| `--surface` | `#141414` | 카드/패널 배경 |
| `--danger` | `#ef4444` | 경고/나쁜 예 |
| `--success` | `#22c55e` | 성공/좋은 예 |
| `--warning` | `#f59e0b` | 주의/오렌지 |
| `--purple` | `#8b5cf6` | 보라 강조 |
