---
description: 현재 작업 내용을 커밋합니다 (Conventional Commits 형식)
---

# Git 커밋

사용법: `/git-commit` 또는 "커밋해줘"

## 커밋 메시지 포맷 (Conventional Commits)

```
<type>(<scope>): <subject>

<body>

<footer>
```

### type (필수)

| type | 용도 | 예시 |
|------|------|------|
| `feat` | 새 기능 | `feat(project): add project search API` |
| `fix` | 버그 수정 | `fix(stage): resolve ObjectId lookup failure` |
| `refactor` | 리팩토링 (기능 변경 없음) | `refactor(repo): simplify query builder` |
| `docs` | 문서 변경 | `docs: update README with test section` |
| `test` | 테스트 추가/수정 | `test(project): add validation edge cases` |
| `chore` | 빌드, 설정, 의존성 등 | `chore: upgrade vitest to v4.2` |
| `style` | 코드 포맷팅 | `style: fix trailing whitespace` |
| `perf` | 성능 개선 | `perf(search): add index for text search` |
| `ci` | CI/CD 설정 | `ci: add GitHub Actions workflow` |

### scope (선택)

`()` 안에 변경 대상 모듈을 표기합니다:
- 도메인: `project`, `stage`, `task`
- 인프라: `db`, `prisma`, `docker`, `k8s`
- 프론트: `ui`, `page`
- 도구: `skill`, `workflow`, `script`

### subject (필수)

- **영문 소문자**로 시작
- **명령형** 사용: "add", "fix", "update" (not "added", "fixed")
- 마침표 없음
- 50자 이내

### body (선택, 여러 줄 변경 시 권장)

```
- Add Confluence skill with API patterns and rules
- Fix .env smart quotes causing auth failures
- Key rule: always use Node.js instead of curl
```

- `-` 목록으로 변경 사항 나열
- **무엇을/왜** 변경했는지 설명

### footer (선택)

```
Refs: #issue-number
BREAKING CHANGE: API response format changed
```

## 워크플로우

// turbo-all
1. **프로토콜 준수 검증** (커밋 차단 가능):
   ```bash
   bash scripts/check-protocol.sh
   ```
   - ❌ 에러가 있으면 docs/ 파일을 먼저 업데이트한 후 다시 시도
   - ⚠️ 경고만 있으면 확인 후 진행 가능
2. 변경 사항을 확인한다:
   ```bash
   git status && git diff --stat
   ```
2. 변경된 파일의 diff를 분석하여 적절한 커밋 메시지를 자동 작성한다
3. 하나의 커밋으로 스테이징하고 커밋한다:
   ```bash
   git add -A && git commit -m "<자동 생성된 메시지>"
   ```
4. 원격에 push 한다:
   ```bash
   git push origin $(git branch --show-current)
   ```
// turbo
5. **[필수]** Confluence에 개발 세션 기록을 동기화한다:
   ```bash
   npx tsx scripts/sync-session-to-confluence.ts --category dev-session
   ```
   > ⚠️ 세션 로그(`docs/agent_session_log.md`)에 오늘 날짜 블록이 없으면 건너뛰어짐
   > feature 브랜치에서의 작업 기록은 `[Dev Sessions]` 하위에 생성됨

## 예시

### 단일 변경
```
feat(skill): add Confluence skill for API integration
```

### 복합 변경
```
feat(test): add Vitest framework and domain unit tests

- Install vitest and @vitest/coverage-v8
- Add vitest.config.ts with path aliases
- Create Prisma mock factory in __tests__/setup.ts
- Add project.test.ts (21 tests), stage.test.ts (12), task.test.ts (3)
- Add npm test/test:watch/test:coverage scripts

Refs: #testing-infrastructure
```

### Squash Merge (develop → main)
```
<type>: <sprint/epic 한줄 요약> (#develop-merge)

## Summary
<무엇이 왜 변경되었는지 2-3문장 요약>

## Key Changes
- ✅ <핵심 변경 1>
- ✅ <핵심 변경 2>
- ✅ <핵심 변경 3>

## Details

### Features
- <새 기능 설명>

### Fixes
- <버그 수정 설명>

### Docs / Config
- <문서, 설정 변경>

## Test Results
- Unit Tests: N/N passed
- API Tests: N/N passed
```

## Push 메시지 포맷

push 후 팀에 공유하거나 PR description에 사용하는 요약 포맷입니다.

### 단일 커밋 Push

커밋 메시지를 그대로 사용합니다.

### 누적 커밋 Push (develop → main squash merge)

아래 포맷으로 변경 사항을 분류하여 정리합니다:

```
## 📋 Push Summary — <날짜>

### 🎯 목적
<이번 push의 전체 목적 한 문장>

### ✅ Key Changes
1. <가장 중요한 변경>
2. <두 번째 중요 변경>
3. <세 번째>

### 📦 변경 분류
| Category | Changes |
|----------|---------|
| Features | <feat 커밋들 요약> |
| Fixes    | <fix 커밋들 요약> |
| Docs     | <docs 커밋들 요약> |
| Infra    | <chore/ci 커밋들 요약> |

### 📊 Impact
- 변경 파일: N files
- 추가/삭제: +N / -N lines
- 테스트: N/N passed
```

### 분석 방법

push 전 아래 명령으로 누적 커밋을 분석합니다:

```bash
# develop에만 있는 커밋 목록
git log main..develop --oneline

# 변경 통계
git diff main..develop --stat | tail -1

# type별 분류
git log main..develop --oneline | grep -c "^.\{8\} feat"
git log main..develop --oneline | grep -c "^.\{8\} fix"
git log main..develop --oneline | grep -c "^.\{8\} docs"
```

