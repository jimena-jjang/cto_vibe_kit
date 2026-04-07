---
name: confluence
description: Confluence 페이지 읽기/쓰기/수정/삭제 작업을 수행합니다. "confluence", "컨플루언스", "기획서 업로드", "PRD 동기화" 등의 요청 시 트리거됩니다.
---

# Confluence Skill

Atlassian Confluence REST API를 사용하여 페이지를 관리합니다.

## ⚠️ 핵심 규칙

### 반드시 Node.js(tsx) 스크립트로 실행할 것

**절대 `curl`이나 shell 명령으로 직접 Confluence API를 호출하지 마세요!**

| 방법 | 결과 |
|------|------|
| ❌ `curl` + `source .env` | shell escape 문제, macOS `base64` 개행 삽입 → **무한 대기/hang** |
| ✅ `npx tsx scripts/*.ts` | Node.js `Buffer.from().toString('base64')` → **정상 동작** |

### 이유
1. `.env`의 API 토큰에 `=`, `+` 등 특수문자가 포함 → shell 치환 시 깨짐
2. macOS `base64` 명령은 76자마다 개행 추가 → Authorization 헤더 오류
3. Node.js의 `Buffer.from().toString('base64')`는 개행 없이 인코딩

## 환경 변수 (.env)

```env
ATLASSIAN_EMAIL=june.kay@bemyfriends.com
ATLASSIAN_API_TOKEN=ATATT3x...
CONFLUENCE_DOMAIN=bemyfriends.atlassian.net
CONFLUENCE_SPACE_KEY=BEMYFRIEND
PARENT_PAGE_ID=2072641574
```

> **주의**: `.env` 값에 스마트 따옴표(`"…"`)를 사용하지 말 것! 반드시 직선 따옴표(`"`) 또는 따옴표 없이 작성.

## API 호출 패턴

### 인증 헤더 생성
```typescript
import dotenv from 'dotenv';
dotenv.config();

const { ATLASSIAN_EMAIL, ATLASSIAN_API_TOKEN, CONFLUENCE_DOMAIN } = process.env;
const auth = Buffer.from(`${ATLASSIAN_EMAIL}:${ATLASSIAN_API_TOKEN}`).toString('base64');
const headers = {
  'Authorization': `Basic ${auth}`,
  'Accept': 'application/json',
  'Content-Type': 'application/json',
};
const baseUrl = `https://${CONFLUENCE_DOMAIN}/wiki/rest/api/content`;
```

### 타임아웃 필수 적용
```typescript
const controller = new AbortController();
const timeout = setTimeout(() => controller.abort(), 15000); // 15초

const res = await fetch(url, { headers, signal: controller.signal });
clearTimeout(timeout);
```

### 읽기 (GET)
```typescript
const res = await fetch(`${baseUrl}/${pageId}?expand=version,body.storage`, { headers });
const page = await res.json();
```

### 쓰기 (POST)
```typescript
const res = await fetch(baseUrl, {
  method: 'POST',
  headers,
  body: JSON.stringify({
    type: 'page',
    title: '페이지 제목',
    space: { key: CONFLUENCE_SPACE_KEY },
    ancestors: [{ id: PARENT_PAGE_ID }],
    body: { storage: { value: '<p>HTML 내용</p>', representation: 'storage' } },
  }),
});
```

### 수정 (PUT) — 반드시 version number 증가 필요
```typescript
// 1. 먼저 현재 버전 조회
const current = await (await fetch(`${baseUrl}/${pageId}?expand=version`, { headers })).json();

// 2. version.number + 1로 업데이트
const res = await fetch(`${baseUrl}/${pageId}`, {
  method: 'PUT',
  headers,
  body: JSON.stringify({
    id: pageId,
    type: 'page',
    title: current.title,
    version: { number: current.version.number + 1 },
    body: { storage: { value: '<p>수정된 내용</p>', representation: 'storage' } },
  }),
});
```

### 삭제 (DELETE)
```typescript
await fetch(`${baseUrl}/${pageId}`, { method: 'DELETE', headers });
```

## 헬퍼 스크립트

프로젝트에 이미 포함된 Confluence 관련 스크립트:

| 스크립트 | 용도 |
|---------|------|
| `scripts/upload_prd.ts` | PRD 마크다운을 Confluence에 업로드 |
| `scripts/sync-prd.ts` | Git diff 기반 PRD 변경사항 동기화 |
| `scripts/update-confluence-page.ts` | 특정 페이지 조회 + 수정 (단계별 로깅) |
| `scripts/sync-specs-to-confluence.ts` | specs.md 계층적 Confluence 동기화 |
| `scripts/sync-session-to-confluence.ts` | 개발 세션 기록 → Confluence 업로드 (**매 작업 완료 시 필수 실행**) |

### 실행 방법

```bash
# 항상 npx tsx로 실행!
npx tsx scripts/upload_prd.ts
npx tsx scripts/sync-prd.ts
npx tsx scripts/update-confluence-page.ts
```

## 트러블슈팅

| 증상 | 원인 | 해결 |
|------|------|------|
| HTTP 403 | `.env`에 스마트 따옴표 사용 | 따옴표 제거 또는 직선 따옴표로 교체 |
| 무한 대기/hang | curl + shell에서 실행 | `npx tsx` 스크립트로 전환 |
| 타임아웃 | 네트워크 이슈 | `AbortController`에 15초 타임아웃 적용 |
| version conflict | 수정 시 이전 버전 번호 사용 | 먼저 GET으로 현재 버전 조회 후 +1 |
