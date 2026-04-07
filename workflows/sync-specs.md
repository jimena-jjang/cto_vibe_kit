---
description: specs.md 내용을 Confluence에 업로드/업데이트합니다
---

# Specs → Confluence 동기화

사용법: `/sync-specs` 또는 "스펙 문서 Confluence에 올려줘"

## 워크플로우

1. `docs/specs.md` 파일의 현재 내용을 읽는다
2. `docs/spec-changelog.md`에서 최신 변경 이력을 읽는다
// turbo
3. Confluence에 `[Specs] b.stage Developer Portal` 페이지가 존재하는지 확인한다:
   - **scripts/sync-specs-to-confluence.ts** 스크립트를 실행한다
   - 스크립트가 자동으로:
     a. parent page 하위에서 제목으로 기존 페이지 검색
     b. 있으면 → PUT (업데이트), 없으면 → POST (생성)
     c. specs.md를 HTML로 변환 후 업로드
     d. 마지막에 changelog 요약을 페이지 하단에 추가
4. 결과 URL을 사용자에게 보여준다

## ⚠️ 주의사항

- **절대 curl로 실행하지 말 것** — 반드시 `npx tsx` 사용
- `.agent/skills/confluence/SKILL.md`의 규칙을 따를 것
