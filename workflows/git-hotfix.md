---
description: 프로덕션 긴급 수정을 위한 hotfix 브랜치를 생성합니다
---

# Hotfix (긴급 수정)

사용법: `/git-hotfix` 또는 "핫픽스 해줘"

## 워크플로우

1. 사용자에게 핫픽스 설명을 물어본다
// turbo
2. main에서 hotfix 브랜치를 생성한다:
   ```bash
   git checkout main && git pull origin main
   git checkout -b hotfix/{description}
   ```
3. 수정 작업을 진행한다 (사용자의 지시에 따라)
4. 수정 완료 후, 커밋하고 push:
   ```bash
   git add -A && git commit -m "fix: {description}"
   git push -u origin hotfix/{description}
   ```
5. main에 머지하고 태그를 생성한다:
   ```bash
   git checkout main && git merge --no-ff hotfix/{description}
   git tag -a v{patch-version} -m "Hotfix: {description}"
   git push origin main --tags
   ```
6. develop에도 역병합한다:
   ```bash
   git checkout develop && git merge --no-ff hotfix/{description}
   git push origin develop
   ```
7. hotfix 브랜치를 삭제한다:
   ```bash
   git branch -d hotfix/{description}
   git push origin --delete hotfix/{description}
   ```
