---
description: Git Flow 기반 feature 브랜치를 생성하고 작업을 시작합니다
---

# Feature 브랜치 시작

사용법: `/git-feature` 또는 "feature 브랜치 만들어줘"

## 워크플로우

1. 현재 브랜치 상태를 확인한다
// turbo
2. `develop` 브랜치가 없으면 `main`에서 생성한다:
   ```bash
   git checkout main && git pull origin main && git checkout -b develop && git push -u origin develop
   ```
// turbo
3. `develop`을 최신으로 업데이트한다:
   ```bash
   git checkout develop && git pull origin develop
   ```
4. 사용자에게 feature 이름을 물어본다 (예: "로그인 페이지 추가")
5. feature 이름을 kebab-case로 변환하여 브랜치를 생성한다:
   ```bash
   git checkout -b feat/{feature-name} develop
   ```
6. 브랜치를 원격에 push한다:
   ```bash
   git push -u origin feat/{feature-name}
   ```
7. 생성된 브랜치 정보를 사용자에게 알려준다
