---
description: 개발 세션 기록을 Confluence에 자동 업로드/업데이트합니다
---

# /sync-session — 개발 세션 → Confluence 동기화

> 개발 작업 완료 후 세션 기록, sub-agent 논의 내용, changelog를 Confluence에 업로드합니다.
> **카테고리별로 구분**하여 feature 브랜치 작업과 main 릴리스를 분리 관리합니다.

사용법: `/sync-session` 또는 "세션 기록 Confluence에 올려줘"

## Confluence 페이지 구조

```
Parent Page
  ├─ [Dev Sessions] b.stage Developer Portal      ← feature 브랜치 작업 기록
  │    ├─ [Dev Session] 2026-04-05 — Phase 2: Multi-Repo UI 통합
  │    ├─ [Dev Session] 2026-04-01 — 테스트 실패 수정
  │    └─ ...
  └─ [Releases] b.stage Developer Portal           ← main 머지/릴리스 기록
       ├─ [Release] v1.2.0 — Multi-Repo 아키텍처 전환
       ├─ [Release] v1.1.0 — 배포 파이프라인 고도화
       └─ ...
```

## 카테고리 구분

| 카테고리 | 트리거 | 포함 내용 | 부모 페이지 |
|---------|--------|----------|------------|
| `dev-session` | `/git-commit`, `/git-finish`, 수동 | 세션 로그, sub-agent 논의, changelog, 브랜치 커밋 내역 | `[Dev Sessions]` |
| `release` | `/git-release` | 누적 머지 커밋, 변경 통계, 테스트 결과, changelog, 릴리스 태그 | `[Releases]` |

## 워크플로우 (dev-session — 기본)

// turbo
1. 오늘 날짜의 세션이 `docs/agent_session_log.md`에 존재하는지 확인한다:

```bash
TODAY=$(date +%Y-%m-%d)
if grep -q "$TODAY" docs/agent_session_log.md; then
  echo "✅ 오늘 세션 블록이 존재합니다. 동기화를 진행합니다."
else
  echo "⚠️ 오늘 날짜의 세션 블록이 없습니다. 먼저 세션 로그를 작성해주세요."
fi
```

// turbo
2. Confluence 동기화 스크립트를 실행한다:

```bash
npx tsx scripts/sync-session-to-confluence.ts --category dev-session
```

3. 결과 URL을 사용자에게 보여준다.

## 옵션

### 기본: 오늘 개발 세션 동기화 (feature 브랜치)
```bash
npx tsx scripts/sync-session-to-confluence.ts --category dev-session
```

### 특정 날짜 개발 세션 동기화
```bash
npx tsx scripts/sync-session-to-confluence.ts --category dev-session --date 2026-04-01
```

### 릴리스 기록 동기화 (main 머지 시)
```bash
npx tsx scripts/sync-session-to-confluence.ts --category release --version 1.2.0
```

### 미리보기 (업로드 안함)
```bash
npx tsx scripts/sync-session-to-confluence.ts --dry-run
```

### 브랜치 명시
```bash
npx tsx scripts/sync-session-to-confluence.ts --category dev-session --branch feat/multi-repo
```

## 자동 트리거 연동

| Git 워크플로우 | 카테고리 | 실행 시점 |
|---------------|---------|----------|
| `/git-commit` | `dev-session` | push 직후 |
| `/git-finish` | `dev-session` | feature 브랜치 삭제 후 (최종 기록) |
| `/git-release` | `release` | release 브랜치 삭제 후 |

## ⚠️ 주의사항

- **절대 curl로 실행하지 말 것** — 반드시 `npx tsx` 사용
- `.agent/skills/confluence/SKILL.md`의 규칙을 따를 것
- 세션 로그가 비어있으면 `dev-session` 동기화가 건너뛰어짐
- `release` 카테고리는 세션 로그 없이도 Git 정보만으로 생성 가능
