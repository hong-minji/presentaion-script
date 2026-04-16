# Presentation Script Skill

AI 에이전트(Claude Code, Gemini CLI 등)가 **발표 자료(PPT)를 HTML로 자동 생성**할 수 있게 해주는 스킬입니다.

단일 HTML 파일 두 개로 구성된 프레젠테이션 시스템을 만들어내며, PowerPoint나 Google Slides 없이 브라우저만으로 발표할 수 있습니다.

---

## 주요 특징

### 프레젠테이션 슬라이드

- 다크 테마 기반의 세련된 디자인
- 다양한 슬라이드 타입: 커버, 일반, 섹션 구분, 데모, 센터 콘텐츠
- 풍부한 UI 컴포넌트: 카드 그리드, 비교(Good/Bad), 플로우 다이어그램, 코드 블록, 테이블, 스텝 등
- 키보드/버튼 네비게이션
- 섹션 바로가기 탭
- 프로그레스 바
- 반응형 디자인

### 발표자 뷰 (Presenter View)

| 기능 | 설명 |
|------|------|
| 슬라이드 미리보기 | 16:9 비율로 자동 리사이즈되는 iframe 프리뷰 |
| 발표 스크립트 | 슬라이드별 대본 (강조, 무대지시, 참고정보 구분) |
| 타이머 | 발표 시간 측정 (시작/일시정지/리셋) |
| 목표 시간 설정 | 분 단위로 목표 발표 시간 입력 가능 |
| 페이스 바 | 시간 vs 슬라이드 진행률 비교 (빠름/적절/느림 자동 판단) |
| 클릭 하이라이트 | 미리보기에서 요소 클릭 시 스포트라이트 효과 |
| 펜 드로잉 | 슬라이드 위에 자유롭게 그리기 (청중 화면에 동기화) |
| 텔레프롬프터 | 스크립트를 큰 글씨로 읽을 수 있는 모드 |
| 스크립트 전체화면 | 미리보기를 숨기고 스크립트만 표시 |
| 글씨 크기 조절 | 스크립트 폰트 사이즈 +/- 조절 |
| 실시간 동기화 | BroadcastChannel로 두 화면 양방향 연동 |

---

## 파일 구조

```
presentaion-script/
├── README.md                          # 이 문서
├── skills/
│   ├── presentation-slide.md          # 슬라이드 생성 스킬 정의
│   └── presenter-view.md              # 발표자 뷰 생성 스킬 정의
└── examples/
    ├── week1-presentation.html        # 예시: 프레젠테이션 슬라이드
    └── week1-script.html              # 예시: 발표자 뷰
```

---

## 사용 방법

### 1. AI에게 발표 자료 생성 요청

```
이 내용으로 HTML 프레젠테이션을 만들어줘:
- 주제: [발표 주제]
- 발표 시간: [N]분
- 섹션: [섹션1], [섹션2], ...
- 내용: [발표 내용 또는 참고 자료]
```

AI가 `skills/` 폴더의 스킬을 참조하여 두 개의 HTML 파일을 생성합니다:
- `{topic}-presentation.html` (청중용 슬라이드)
- `{topic}-script.html` (발표자 뷰)

### 2. 브라우저에서 발표

1. **프레젠테이션 파일**을 브라우저 탭 1에서 열기 (청중에게 공유하는 화면)
2. **발표자 뷰 파일**을 브라우저 탭 2에서 열기 (발표자만 보는 화면)
3. 두 화면이 `BroadcastChannel`로 **자동 동기화**됨
4. 발표자 뷰에서 슬라이드 넘기면 청중 화면도 자동으로 따라감

### 3. 키보드 단축키

| 키 | 동작 |
|----|------|
| `→` / `Space` / `Enter` | 다음 슬라이드 |
| `←` | 이전 슬라이드 |
| `Home` | 첫 슬라이드로 |
| `End` | 마지막 슬라이드로 |
| `T` | 텔레프롬프터 모드 토글 |
| `F` | 스크립트 전체화면 토글 |

---

## 동기화 원리

두 HTML 파일은 **[BroadcastChannel API](https://developer.mozilla.org/en-US/docs/Web/API/BroadcastChannel)**를 사용하여 같은 브라우저의 탭/윈도우 간에 실시간 메시지를 주고받습니다.

```
[발표자 뷰]                    [프레젠테이션]
     │                              │
     ├──slideChange──────────────► │  슬라이드 이동
     ├──elementHighlight─────────►│  요소 하이라이트
     ├──penStart/penMove/penEnd──►│  펜 드로잉
     ├──penClear─────────────────►│  펜 지우기
     │                              │
     │◄──slideChange────────────── │  역방향 동기화
     │◄──elementHighlight────────  │
```

채널 이름: `presentation-sync`

---

## 슬라이드 컴포넌트 목록

### 슬라이드 타입

| 타입 | CSS 클래스 | 용도 |
|------|-----------|------|
| 커버 | `.slide.cover` | 첫 슬라이드. 제목 + 부제 + 발표자 |
| 일반 | `.slide` | 제목 + 다양한 콘텐츠 조합 |
| 섹션 구분 | `.slide.slide--section` | 새 섹션 시작. 번호 + 제목 + 설명 |
| 데모 | `.slide.slide--demo` | 라이브 데모. 이모지 + 제목 + 설명 + 맥동 테두리 |
| 센터 | `.center-content` (내부) | 중앙 메시지 (Q&A, 인용문 등) |

### 콘텐츠 컴포넌트

| 컴포넌트 | CSS 클래스 | 예시 |
|----------|-----------|------|
| 카드 그리드 | `.grid.grid--2/3/4` + `.card` | 2~4열 정보 카드 |
| 하이라이트 박스 | `.highlight-box` | 핵심 메시지 강조 |
| 플로우 | `.flow` + `.flow-item` + `.flow-arrow` | INPUT → PROCESS → OUTPUT |
| 비교 | `.compare` + `.compare-item.good/.bad` | 좋은 예 vs 나쁜 예 |
| 코드 블록 | `.code-block` | 코드/터미널 표시 |
| 스텝 | `.steps` + `.step` | 순서가 있는 단계 |
| 테이블 | `.table-wrap` + `table` | 데이터 테이블 |
| 리스트 | `.list` | 불릿 포인트 |
| 키커 | `.kicker` | 작은 뱃지/라벨 |
| 퍼즐 | `.puzzle-row` + `.puzzle-piece` | A + B = C 조합 |
| 타임 뱃지 | `.time-badge` | 시간 표시 뱃지 |

---

## 스크립트 서식

발표자 뷰의 스크립트에서 사용하는 서식:

| 서식 | HTML | 용도 |
|------|------|------|
| 강조 | `<span class="emphasis">텍스트</span>` | 핵심 키워드 파란색 강조 |
| 무대 지시 | `<span class="action">텍스트</span>` | 화면 전환, 데모 시작 등 |
| 쉼표 | `<span class="pause">[잠깐]</span>` | 잠깐 멈추는 지점 |
| 참고 | `<div class="script-ref">텍스트</div>` | 수치, 출처 등 보조 정보 |

---

## 커스터마이징

### 테마 색상 변경

CSS 변수를 수정하면 전체 테마가 바뀝니다:

```css
:root {
  --accent: #3b82f6;   /* 주요 강조색 (기본: 파랑) */
  --bg: #0a0a0a;       /* 배경색 */
  --surface: #141414;  /* 카드 배경 */
  --danger: #ef4444;   /* 경고/나쁜 예 */
  --success: #22c55e;  /* 성공/좋은 예 */
  --warning: #f59e0b;  /* 주의 */
  --purple: #8b5cf6;   /* 보라 강조 */
}
```

### 테마 프리셋 예시

| 테마 | `--accent` 값 |
|------|---------------|
| 블루 (기본) | `#3b82f6` |
| 퍼플 | `#8b5cf6` |
| 그린 | `#22c55e` |
| 오렌지 | `#f59e0b` |
| 레드 | `#ef4444` |

---

## 기술 스택

- **HTML5** + **CSS3** (CSS Variables, Grid, Flexbox, Animations)
- **Vanilla JavaScript** (외부 라이브러리 없음)
- **BroadcastChannel API** (탭 간 동기화)
- **Canvas API** (펜 드로잉)
- **Google Fonts** (Noto Sans KR, JetBrains Mono)

외부 의존성이 전혀 없어서 HTML 파일만 있으면 어디서든 동작합니다.

---

## 예제 실행

```bash
# examples 폴더의 예제를 브라우저에서 열기
open examples/week1-presentation.html  # 프레젠테이션
open examples/week1-script.html        # 발표자 뷰 (같은 브라우저에서!)
```

---

## 라이선스

MIT License
