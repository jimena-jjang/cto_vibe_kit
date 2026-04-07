---
description: 현재 Git 상태와 브랜치 구조를 보여줍니다
---

# Git 상태 확인

사용법: `/git-status` 또는 "git 상태 알려줘"

## 워크플로우

// turbo-all
1. 현재 브랜치와 상태를 확인한다:
   ```bash
   git branch --show-current && echo "---" && git status --short
   ```
2. 전체 로컬 + 원격 브랜치를 확인한다:
   ```bash
   git branch -a
   ```
3. 최근 커밋 이력을 확인한다:
   ```bash
   git log --oneline --graph --all -n 15
   ```
4. 결과를 보기 좋게 정리하여 사용자에게 보여준다:
   - 현재 브랜치
   - 미커밋 변경사항
   - 최근 커밋 로그
   - 브랜치 목록 (Git Flow 구조별 분류)
