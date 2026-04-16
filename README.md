# mj-pt

Claude Code에서 `/mj-pt`로 호출하는 HTML 프레젠테이션 생성 스킬입니다.

"발표 자료 만들어줘", "PPT 만들어줘" 등을 요청하면 **슬라이드 + 발표자 뷰** 두 개의 HTML 파일을 자동 생성합니다. PowerPoint 없이 브라우저만으로 발표할 수 있습니다.

---

## 설치

```bash
claude mcp add-skill https://github.com/hong-minji/presentaion-script
```

또는 Claude Code 안에서:

```
/install-skill https://github.com/hong-minji/presentaion-script
```

---

## 사용법

Claude Code에서:

```
/mj-pt
```

또는 자연어로:

```
AI 에이전트 소개 발표 자료를 30분 분량으로 만들어줘
```

---

## 생성되는 파일

| 파일 | 용도 |
|------|------|
| `{topic}-presentation.html` | 청중에게 보여주는 슬라이드 |
| `{topic}-script.html` | 발표자 전용 화면 (스크립트, 타이머, 펜 등) |

두 파일을 같은 브라우저에서 각각 탭으로 열면 **자동 동기화**됩니다.

---

## 주요 기능

### 슬라이드

- 다크 테마 기반 세련된 디자인
- 5가지 슬라이드 타입: 커버, 일반, 섹션 구분, 데모, 센터
- 10+ 콘텐츠 컴포넌트: 카드 그리드, 비교(Good/Bad), 플로우, 코드 블록, 테이블, 스텝 등
- 클릭 하이라이트 (스포트라이트 효과)
- 키보드/버튼 네비게이션
- 반응형 디자인

### 발표자 뷰

- 슬라이드 미리보기 (16:9 자동 리사이즈)
- 슬라이드별 발표 스크립트
- 타이머 (시작/일시정지/리셋) + 목표 시간 설정
- 페이스 바 (시간 vs 진도 비교, 빠름/적절/느림 자동 판단)
- 펜 드로잉 (슬라이드 위에 그리기, 청중 화면에 동기화)
- 텔레프롬프터 모드 (큰 글씨)
- 스크립트만 모드 (미리보기 숨김)
- 글씨 크기 조절

### 동기화

`BroadcastChannel API`로 두 화면이 양방향 실시간 연동:

- 슬라이드 이동
- 요소 하이라이트
- 펜 드로잉

### 키보드 단축키

| 키 | 동작 |
|----|------|
| `→` / `Space` / `Enter` | 다음 슬라이드 |
| `←` | 이전 슬라이드 |
| `Home` / `End` | 처음 / 마지막 |
| `T` | 텔레프롬프터 토글 |
| `F` | 스크립트 전체화면 토글 |

---

## 파일 구조

```
presentaion-script/
├── SKILL.md                                # 스킬 정의 (Claude Code가 읽는 파일)
├── references/
│   ├── slide-components.md                 # 슬라이드 CSS + 컴포넌트 레퍼런스
│   └── presenter-view-template.md          # 발표자 뷰 HTML 템플릿
├── examples/
│   ├── week1-presentation.html             # 예시: 슬라이드
│   └── week1-script.html                   # 예시: 발표자 뷰
└── README.md
```

---

## 테마 커스터마이징

CSS `--accent` 변수를 변경하면 전체 색상이 바뀝니다:

| 테마 | `--accent` |
|------|------------|
| 블루 (기본) | `#3b82f6` |
| 퍼플 | `#8b5cf6` |
| 그린 | `#22c55e` |
| 오렌지 | `#f59e0b` |

---

## 기술 스택

HTML5 + CSS3 + Vanilla JS. 외부 라이브러리 없음. HTML 파일만 있으면 어디서든 동작합니다.

---

## 라이선스

MIT
