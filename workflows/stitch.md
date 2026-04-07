---
description: Google Stitch로 UI 디자인 생성/수정/Export를 진행합니다
---

# Stitch 디자인 워크플로우

사용법: `/stitch` 또는 "스티치로 디자인 만들어줘"

## 사전 준비

- `.agent/skills/stitch-design/SKILL.md` 스킬을 먼저 읽어 규칙을 숙지합니다.

## 워크플로우

### 1. Stitch 접속
// turbo
- Browser Subagent로 `https://stitch.withgoogle.com` 접속
- 로그인 상태 확인
  - 로그인 필요 시: 사용자에게 "Google 로그인이 필요합니다. 브라우저에서 로그인해주세요." 안내
  - 로그인 완료 시: 다음 단계로

### 2. 디자인 요청 확인
- 사용자에게 어떤 화면을 디자인할지 확인
- 필요한 정보 수집:
  - 화면 종류 (대시보드, 폼, 목록 등)
  - 스타일 (다크/라이트, 미니멀/리치)
  - 포함할 컴포넌트 (카드, 테이블, 사이드바 등)
  - 참고할 기존 페이지나 레퍼런스

### 3. 프롬프트 작성 및 생성
// turbo
- 수집된 정보를 영문 프롬프트로 변환
- Browser Subagent로 Stitch에 프롬프트 입력
- 생성 완료 대기 (최대 60초)
- 결과 스크린샷 캡처

### 4. 사용자 피드백
- 결과 스크린샷을 사용자에게 보여줌
- 수정이 필요한 부분 확인
- 필요 시 3번으로 돌아가서 반복

### 5. Export (선택)
- 사용자가 만족하면 Export 진행
- Export 형식 선택: HTML/CSS, Figma, PNG
// turbo
- Browser Subagent로 Export 실행
- 다운로드된 파일을 프로젝트에 반영 (필요 시)

### 6. 코드 반영 (선택)
- HTML/CSS Export를 Tailwind + shadcn/ui 체계로 변환
- `src/app/` 또는 `src/components/`에 적용
- 커밋 및 push

## 빠른 사용 예시

```
사용자: /stitch
에이전트: 어떤 화면을 디자인하시겠어요?
사용자: 스테이지 관리 대시보드를 다크 테마로 만들어줘
에이전트: (Stitch 접속 → 프롬프트 입력 → 생성 → 스크린샷 반환)
사용자: 좋은데 사이드바를 좀 더 넓게 해줘
에이전트: (수정 프롬프트 → 결과 반환)
사용자: 이걸로 확정. 코드에 반영해줘
에이전트: (Export → Tailwind 변환 → 커밋)
```
