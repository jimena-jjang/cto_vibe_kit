---
name: stitch-design
description: Google Stitch(AI UI 디자인 도구)를 Browser Subagent로 제어하여 UI 디자인 생성, 수정, Export를 수행합니다. "stitch", "디자인 만들어줘", "UI 생성", "화면 디자인", "스티치" 등의 요청 시 트리거됩니다.
---

# Google Stitch Design Skill

Google Stitch (stitch.withgoogle.com)를 Browser Subagent를 통해 직접 제어하여 UI 디자인을 생성하고 관리하는 스킬입니다.

## 사전 조건

- **Google 로그인 필요**: Stitch는 Google 계정 로그인이 필수입니다.
  - 처음 사용 시 "로그인이 필요합니다"라고 사용자에게 알려주세요.
  - Browser Subagent로 `stitch.withgoogle.com` 접속 후 로그인 화면이 보이면 사용자에게 직접 로그인을 요청하세요.
  - 로그인 완료 후 다시 작업을 이어가세요.

## 핵심 규칙

1. **항상 Browser Subagent 사용** — Stitch는 브라우저에서만 동작합니다.
2. **스크린샷 필수** — 모든 단계에서 스크린샷을 캡처하여 사용자에게 보여주세요.
3. **단계별 확인** — 생성 결과를 반드시 사용자에게 보여주고 피드백을 받으세요.
4. **Export 시 HTML/CSS 우선** — 코드 변환이 목적이면 HTML/CSS Export를 선택하세요.

## 작업 흐름

### 1. Stitch 접속 및 준비

```
Browser Subagent Task:
1. stitch.withgoogle.com 접속
2. 로그인 상태 확인
   - 로그인 안 됨 → 사용자에게 로그인 요청, 완료될 때까지 대기
   - 로그인 됨 → 다음 단계로 진행
3. 메인 화면 스크린샷 캡처
```

### 2. 디자인 생성 (New Design)

```
Browser Subagent Task:
1. "Create" 또는 새 디자인 생성 버튼 클릭
2. 모드 선택:
   - Standard Mode (Gemini 2.5 Flash) — 빠른 생성
   - Experimental Mode (Gemini 2.5 Pro) — 고품질
3. 프롬프트 입력란에 디자인 설명 입력
4. 생성 버튼 클릭
5. 생성 완료 대기 (최대 60초)
6. 결과 스크린샷 캡처하여 사용자에게 반환
```

### 3. 디자인 수정 (Iterate)

```
Browser Subagent Task:
1. 현재 디자인 캔버스에서 수정 프롬프트 입력
2. 수정 요청 전송
3. 결과 대기 및 스크린샷 캡처
4. 사용자에게 before/after 보여주기
```

### 4. 디자인 Export

```
Browser Subagent Task:
1. Export 버튼 찾기 (보통 상단 툴바 또는 메뉴)
2. Export 형식 선택:
   - "HTML/CSS" — 코드로 변환할 때
   - "Figma" — Figma에서 추가 편집할 때
   - "PNG/SVG" — 이미지로 저장할 때
3. 다운로드 완료 대기
4. 다운로드된 파일 경로 반환
```

## 프롬프트 작성 가이드

Stitch에 입력할 프롬프트를 잘 작성하면 더 좋은 결과를 얻습니다.

### 좋은 프롬프트 예시

```
"A modern admin dashboard for a developer portal.
Left sidebar with navigation: Projects, Stages, Deployments, Settings.
Main area shows project cards in a grid layout.
Each card has: project name, status badge, stage domain, last deploy date.
Dark theme with blue accent colors.
Top header with search bar and user avatar."
```

### 프롬프트 구성 요소

| 요소 | 설명 | 예시 |
|------|------|------|
| **What** | 어떤 화면인지 | "admin dashboard", "login page" |
| **Layout** | 레이아웃 구조 | "sidebar + main content", "full-width" |
| **Components** | 포함할 컴포넌트 | "cards, tables, buttons, forms" |
| **Style** | 시각 스타일 | "dark theme", "minimal", "glassmorphism" |
| **Details** | 세부 데이터 | "each card shows name and status" |

## 프로젝트 컨텍스트

현재 프로젝트(b.stage Dev Portal)의 디자인 작업 시 아래 컨텍스트를 프롬프트에 포함하세요:

- **서비스**: b.stage Developer Portal — 커스텀 빌드 관리 포털
- **사용자**: 내부 DevOps 팀, 외부 파트너 개발자
- **기술**: Next.js + Tailwind CSS + shadcn/ui
- **페이지**: 프로젝트 목록, 프로젝트 상세, 스테이지 관리, 배포 관리
- **스타일**: 다크 테마, 모던, 프로페셔널

## Export → 코드 반영 워크플로우

1. Stitch에서 HTML/CSS Export
2. Export된 코드를 분석
3. 프로젝트의 Tailwind CSS + shadcn/ui 체계에 맞게 변환
4. `src/app/` 또는 `src/components/` 에 반영
5. 불필요한 인라인 스타일 → Tailwind 클래스로 변환

## 트러블슈팅

| 문제 | 해결 |
|------|------|
| 로그인 화면이 계속 나옴 | 사용자에게 직접 로그인 요청 |
| 생성이 60초 이상 걸림 | Standard Mode로 전환 |
| Export 버튼을 못 찾음 | 상단 메뉴 또는 우클릭 메뉴 확인 |
| 빈 캔버스만 보임 | 페이지 새로고침 후 재시도 |
| 원하는 결과가 안 나옴 | 프롬프트를 더 구체적으로 수정 |

## 제한사항

- Google 로그인은 사용자가 직접 해야 합니다 (자동화 불가)
- Stitch UI가 업데이트되면 셀렉터 조정이 필요할 수 있습니다
- 복잡한 캔버스 조작(드래그 등)은 제한적입니다
- 생성 품질은 프롬프트 품질에 크게 의존합니다
