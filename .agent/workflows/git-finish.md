---
description: 현재 feature 브랜치의 작업을 커밋하고 develop에 머지합니다
---

# Feature 브랜치 완료 (develop 머지)

사용법: `/git-finish` 또는 "feature 완료해줘"

## 워크플로우

// turbo
1. 현재 브랜치와 변경 사항을 확인한다:
   ```bash
   git status && git branch --show-current
   ```
2. 현재 브랜치가 `feat/` 접두어인지 확인한다. 아니면 중단하고 사용자에게 알린다.
// turbo
3. 변경 사항이 있으면 스테이징하고 커밋한다:
   ```bash
   git add -A && git commit -m "{적절한 커밋 메시지}"
   ```
   - 커밋 메시지는 변경 내용을 분석하여 자동으로 작성한다
   - Conventional Commits 형식 사용: `feat:`, `fix:`, `refactor:`, `docs:`, `test:`, `chore:`
4. 원격에 push 한다:
   ```bash
   git push origin {current-branch}
   ```
5. develop으로 체크아웃하고 머지한다:
   ```bash
   git checkout develop && git pull origin develop && git merge --no-ff {feature-branch}
   ```
6. develop을 원격에 push 한다:
   ```bash
   git push origin develop
   ```
7. feature 브랜치를 삭제한다 (사용자에게 확인 후):
   ```bash
   git branch -d {feature-branch}
   git push origin --delete {feature-branch}
   ```
// turbo
8. **[필수]** Confluence에 feature 브랜치의 개발 세션 기록을 최종 동기화한다:
   ```bash
   npx tsx scripts/sync-session-to-confluence.ts --category dev-session --branch {feature-branch}
   ```
   > feature 브랜치 완료 시 최종 세션 기록이 `[Dev Sessions]` 하위에 업데이트됨
   > 이전에 동기화된 세션이 있으면 최신 상태로 업데이트됨
