# Slide Components Reference

프레젠테이션 슬라이드 HTML을 작성할 때 참조하는 컴포넌트 목록입니다.

---

## Table of Contents

1. [전체 HTML 골격](#전체-html-골격)
2. [CSS 전체](#css-전체)
3. [슬라이드 타입별 마크업](#슬라이드-타입별-마크업)
4. [콘텐츠 컴포넌트](#콘텐츠-컴포넌트)
5. [JavaScript](#javascript)

---

## 전체 HTML 골격

```html
<!doctype html>
<html lang="ko">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>{{TITLE}}</title>
  <style>
    /* CSS 전체 (아래 섹션 참조) */
  </style>
</head>
<body>
  <div class="presentation">

    <!-- 섹션 네비게이션 바 -->
    <div class="sections-bar" id="sectionsBar">
      <button class="section-chip" data-slide="1">섹션1</button>
      <button class="section-chip" data-slide="5">섹션2</button>
      <!-- data-slide = 해당 섹션의 첫 번째 슬라이드 번호 -->
    </div>

    <div class="slides-wrap">
      <!-- 슬라이드들 -->
    </div>

    <!-- 하단 컨트롤 -->
    <div class="bottom-controls">
      <button class="nav-btn" id="prevBtn" aria-label="이전">&#8592;</button>
      <div class="slide-counter" id="slideCounter">1 / N</div>
      <button class="nav-btn" id="nextBtn" aria-label="다음">&#8594;</button>
    </div>
    <div class="progress-track"><div class="progress-bar" id="progressBar"></div></div>

  </div>
  <canvas id="pen-overlay" class="pen-overlay"></canvas>

  <script>
    /* JavaScript (아래 섹션 참조) */
  </script>
</body>
</html>
```

---

## CSS 전체

아래 CSS를 `<style>` 태그 안에 그대로 넣습니다. accent 색상을 바꾸고 싶으면 `--accent` 변수만 수정합니다.

```css
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

.cover { justify-content: center; align-items: flex-start; }
.cover-title { font-size: 64px; line-height: 1.1; letter-spacing: -.02em; max-width: 1100px; }
.cover-subtitle { color: var(--accent); font-size: 30px; font-weight: 700; }
.cover-presenter { color: var(--text-secondary); font-size: 24px; }

.slide-title { font-size: 46px; line-height: 1.15; letter-spacing: -.02em; }
.slide-subtitle { color: var(--text-secondary); font-size: 22px; line-height: 1.5; max-width: 1100px; }

.slide--section {
  align-items: center; justify-content: center; text-align: center;
  background:
    radial-gradient(circle at 15% 30%, rgba(59,130,246,.2), transparent 42%),
    radial-gradient(circle at 85% 70%, rgba(59,130,246,.18), transparent 45%);
}
.section-number { font-size: 90px; font-weight: 900; color: rgba(59,130,246,.85); text-shadow: 0 0 30px rgba(59,130,246,.32); }
.section-divider-title { font-size: 56px; letter-spacing: -.03em; }
.section-divider-desc { font-size: 24px; color: var(--text-secondary); }

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
```

---

## 슬라이드 타입별 마크업

### 커버 슬라이드

```html
<div class="slide cover active" id="slide1">
  <p class="kicker">Week 1 · 90분</p>
  <h1 class="cover-title">메인 타이틀<br>두 번째 줄</h1>
  <p class="cover-subtitle">서브타이틀</p>
  <p class="cover-presenter">발표자 · 소속</p>
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

### 데모 슬라이드

```html
<div class="slide slide--demo" id="slideN">
  <p class="demo-emoji">🎬</p>
  <h2 class="demo-title">데모 제목</h2>
  <p class="demo-desc">데모 설명 텍스트</p>
</div>
```

### 센터 콘텐츠 슬라이드

```html
<div class="slide" id="slideN">
  <div class="center-content">
    <h2 style="font-size:110px;">Q&A</h2>
    <p style="font-size:28px;color:var(--text-secondary);">질문해 주세요</p>
  </div>
</div>
```

### 일반 슬라이드 (컴포넌트 조합)

```html
<div class="slide" id="slideN">
  <h2 class="slide-title">슬라이드 제목</h2>
  <p class="slide-subtitle">설명 텍스트</p>
  <!-- 아래 콘텐츠 컴포넌트를 자유롭게 조합 -->
</div>
```

---

## 콘텐츠 컴포넌트

### 카드 그리드

```html
<div class="grid grid--3">
  <div class="card" style="text-align:center;">
    <p style="font-size:40px;margin-bottom:10px;">💡</p>
    <h3 class="card-title">카드 제목</h3>
    <p class="card-desc">카드 설명</p>
  </div>
  <div class="card">
    <h3 class="card-title">카드 제목</h3>
    <p class="card-desc">카드 설명</p>
  </div>
  <div class="card">
    <h3 class="card-title">카드 제목</h3>
    <p class="card-desc">카드 설명</p>
  </div>
</div>
```

`grid--2`, `grid--3`, `grid--4` 중 선택. 카드 안에 `.card-title`, `.card-desc`, `ul.list` 등 자유 조합.

### 하이라이트 박스

```html
<div class="highlight-box">
  <p class="text">강조할 <strong>핵심 메시지</strong>를 여기에 작성</p>
</div>
```

색상 변형:
```html
<div class="highlight-box" style="background:rgba(139,92,246,.12);border-color:rgba(139,92,246,.4);">
  <p class="text" style="color:#e0d4ff;">보라색 하이라이트 박스</p>
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

### 비교 (Good vs Bad)

```html
<div class="compare">
  <div class="compare-item bad">
    <p class="compare-label">나쁜 예</p>
    <p class="compare-text">나쁜 예시 설명</p>
  </div>
  <div class="compare-item good">
    <p class="compare-label">좋은 예</p>
    <p class="compare-text">좋은 예시 설명</p>
  </div>
</div>
```

### 코드 블록

```html
<div class="code-block">
<span class="str">문자열</span>
<span class="cmd">명령어/키워드</span>  <span class="cmt"># 주석</span>
일반 텍스트
</div>
```

### 스텝 (순서 단계)

```html
<div class="steps">
  <article class="step">
    <span class="step-no">1</span>
    <h3 class="step-title">단계 1 제목</h3>
    <p class="step-desc">단계 설명</p>
  </article>
  <article class="step">
    <span class="step-no">2</span>
    <h3 class="step-title">단계 2 제목</h3>
    <p class="step-desc">단계 설명</p>
  </article>
  <article class="step">
    <span class="step-no">3</span>
    <h3 class="step-title">단계 3 제목</h3>
    <p class="step-desc">단계 설명</p>
  </article>
</div>
```

### 테이블

```html
<div class="table-wrap">
  <table>
    <thead><tr><th>헤더1</th><th>헤더2</th><th>헤더3</th></tr></thead>
    <tbody>
      <tr><td>데이터</td><td>데이터</td><td>데이터</td></tr>
      <tr><td>데이터</td><td>데이터</td><td>데이터</td></tr>
    </tbody>
  </table>
</div>
```

### 리스트

```html
<ul class="list">
  <li><strong>굵은 텍스트</strong> -- 설명</li>
  <li>일반 항목</li>
  <li>코드 포함: <code>example</code></li>
</ul>
```

### 퍼즐 조합

```html
<div class="puzzle-row">
  <div class="puzzle-piece">A 요소</div>
  <div class="puzzle-plus">+</div>
  <div class="puzzle-piece">B 요소</div>
  <div class="puzzle-eq">=</div>
  <div class="puzzle-piece" style="border-color:rgba(59,130,246,.5);color:#d9e7ff;">결과</div>
</div>
```

### 키커 뱃지

```html
<p class="kicker">라벨 텍스트</p>
```

### 미니 노트

```html
<p class="mini-note">Tip: 작은 보조 텍스트</p>
```

---

## JavaScript

아래 JS를 `<script>` 태그에 넣습니다. `total`과 `sectionStarts`를 실제 값으로 교체합니다.

```javascript
(function(){
  var total = {{TOTAL_SLIDES}};                    // 전체 슬라이드 수
  var sectionStarts = [{{SECTION_STARTS}}];         // 각 섹션 시작 슬라이드 번호
  var chips = Array.from(document.querySelectorAll('.section-chip'));
  var counter = document.getElementById('slideCounter');
  var bar = document.getElementById('progressBar');
  var cur = 1;
  var bc = null;

  // 펜 오버레이
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

  // 스포트라이트
  function findSpotlightTarget(el) {
    if (!el) return null;
    var box = el.closest('.card, .highlight-box, article, .compare-item, .flow-item, .step, .puzzle-piece');
    if (box && !box.classList.contains('slide')) return box;
    return el.closest('.slide-title, .slide-subtitle, .section-divider-title, p, h2, h3, li, td, code') || el;
  }
  function applySpotlight(target) {
    if (!target || target.classList.contains('slide')) return;
    target.classList.remove('spotlight');
    void target.offsetWidth;
    target.classList.add('spotlight');
    target.addEventListener('animationend', function() { target.classList.remove('spotlight'); }, { once: true });
  }

  document.querySelector('.slides-wrap').addEventListener('click', function(e) {
    var target = findSpotlightTarget(e.target);
    if (!target || target.classList.contains('slide')) return;
    applySpotlight(target);
    var slide = document.getElementById('slide' + cur);
    if (slide && bc) {
      var allEls = Array.from(slide.querySelectorAll('*'));
      bc.postMessage({ type: 'elementHighlight', slideNo: cur, index: allEls.indexOf(target) });
    }
  });

  window.addEventListener('message', function(event) {
    var data = event.data || {};
    if (data.type === 'spotlightClick') {
      var el = document.elementFromPoint(data.x, data.y);
      var target = findSpotlightTarget(el);
      if (target && !target.classList.contains('slide')) applySpotlight(target);
    }
    if (data.type === 'slideChange' && data.slide !== cur) goToSlide(data.slide, true, false);
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
```
