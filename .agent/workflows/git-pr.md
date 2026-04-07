---
description: 현재 feature 브랜치에 대해 PR(Pull Request)을 생성합니다.
---

# PR 생성

사용법: `/git-pr` 또는 "PR 만들어줘"

## PR 메시지 포맷

```markdown
## 📋 Summary
{feature 브랜치의 목적을 1-2문장으로 요약}

## 🔄 Changes
{누적된 커밋 로그를 분석하여 변경사항을 카테고리별로 정리}

### Features
- {새 기능 목록}

### Bug Fixes
- {버그 수정 목록}

### Improvements
- {리팩토링, 성능 개선 등}

### Docs / Config
- {문서, 설정 변경}

## 🧪 Test Results
{테스트 실행 결과}
- Unit Tests: {통과/전체}
- API Tests: {통과/전체}
- Manual: {수동 확인 사항}

## 📸 Screenshots
{UI 변경이 있으면 스크린샷 첨부}

## ✅ Checklist
- [ ] 코드 리뷰 요청
- [ ] 테스트 통과 확인
- [ ] 문서 업데이트
- [ ] Breaking change 없음
```

## 워크플로우

1. 현재 브랜치를 확인한다:
   ```bash
   git branch --show-current
   ```
2. 현재 브랜치가 `feat/`, `fix/`, `hotfix/` 중 하나인지 확인한다. 아니면 중단.
// turbo
3. develop 기준으로 이 브랜치의 **모든 누적 커밋**을 수집한다:
   ```bash
   git log develop..HEAD --oneline --no-merges
   ```
// turbo
4. 커밋 로그뿐 아니라 **실제 변경된 파일과 diff**도 분석한다:
   ```bash
   git diff develop...HEAD --stat
   ```
5. 위 정보를 바탕으로 PR 메시지를 자동 생성한다:
   - 커밋의 `type`별로 분류 (feat → Features, fix → Bug Fixes 등)
   - 중복되거나 연관된 커밋은 하나로 합쳐서 표현
   - 테스트 결과가 있으면 포함
6. 사용자에게 생성된 PR 메시지를 보여주고 확인을 받는다
7. GitHub CLI로 PR을 생성한다:
   ```bash
   gh pr create --base develop --title "<PR 제목>" --body "<PR 본문>"
   ```
   - `gh` CLI가 없으면 설치를 안내하거나, PR 메시지만 제공

## 예시 — 생성되는 PR 메시지

```markdown
## 📋 Summary
Confluence 연동 스킬 추가 및 테스트 인프라 구축

## 🔄 Changes

### Features
- Confluence skill 추가 (`.agent/skills/confluence/SKILL.md`)
- Vitest 테스트 프레임워크 도입 (36 unit tests)
- Git Flow 워크플로우 6개 생성

### Bug Fixes
- `.env` 스마트 따옴표 수정 (Confluence 403 해결)
- `project.repository.ts` undefined ObjectId 필드 처리

### Docs
- README.md에 테스트 섹션 및 아키텍처 다이어그램 업데이트
- TDD 가이드라인 (`docs/TESTING.md`)

## 🧪 Test Results
- Unit Tests: 36/36 passed
- API Smoke Tests: 21/21 passed

## ✅ Checklist
- [x] 테스트 통과 확인
- [x] 문서 업데이트
- [x] Breaking change 없음
```
