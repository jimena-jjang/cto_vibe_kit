---
description: develop에서 release 브랜치를 생성하고 main에 머지하여 배포합니다
---

# Release (배포)

사용법: `/git-release` 또는 "릴리즈 해줘"

## 워크플로우

1. 사용자에게 버전 번호를 물어본다 (예: `1.0.0`, `1.1.0`)
// turbo
2. develop에서 release 브랜치를 생성한다:
   ```bash
   git checkout develop && git pull origin develop
   git checkout -b release/{version}
   ```
3. 버전 관련 파일을 업데이트한다 (package.json의 version 필드)
4. 커밋하고 push 한다:
   ```bash
   git add -A && git commit -m "chore: release v{version}"
   git push -u origin release/{version}
   ```
5. main으로 머지한다:
   ```bash
   git checkout main && git pull origin main
   git merge --no-ff release/{version} -m "chore: merge release v{version} into main"
   git tag -a v{version} -m "Release v{version}"
   git push origin main --tags
   ```
6. develop에도 역병합한다:
   ```bash
   git checkout develop
   git merge --no-ff release/{version} -m "chore: merge release v{version} back into develop"
   git push origin develop
   ```
7. release 브랜치를 삭제한다:
   ```bash
   git branch -d release/{version}
   git push origin --delete release/{version}
   ```
// turbo
8. **[필수]** Confluence에 릴리스 기록을 동기화한다:
   ```bash
   npx tsx scripts/sync-session-to-confluence.ts --category release --version {version}
   ```
   > 릴리스 기록은 `[Releases]` 하위에 생성됨
   > develop → main 누적 커밋, 변경 통계, 테스트 결과, changelog가 포함됨
