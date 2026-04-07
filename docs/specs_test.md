# [Specs] Testing Strategy

## 7. 테스트

| Category | Framework | 상태 |
|----------|-----------|------|
| Unit Tests | Vitest | ✅ |
| Integration Tests | Vitest + Mock | ✅ |
| API Smoke | curl/fetch | ✅ |
| Confluence API | Node.js fetch | ✅ |
| E2E Tests | Playwright | ✅ 인프라 완료 |


## 10. 테스트 인프라

### 10.1 3-Tier 테스트 전략

| Tier | 종류 | 도구 | 현재 상태 |
|------|------|------|-----------|
| **Tier 1** | Unit Tests | Vitest + Mocked Repositories | ✅ 223 tests (15 files) |
| **Tier 2** | Integration Tests | Vitest + Mocked Services | ✅ 48 tests (4 files) |
| **Tier 3** | E2E Tests | Playwright (Chromium) | ✅ 인프라 완료 (9 specs) |

### 10.2 Unit Test 파일 구조

| 파일 | 테스트 수 | 대상 |
|------|-----------|------|
| `src/domains/custom-page/custom-page.test.ts` | 40 | CustomPageService 전체 비즈니스 로직 |
| `src/domains/custom-page/route-sync.test.ts` | 12 | route-sync.util 라우트 빌드/필터링 |
| `src/domains/stage/stage.test.ts` | 42 | StageService CRUD, 배포, GitHub, ensureGithubRepo, Multi-Repo |
| `src/domains/repository/repository.test.ts` | 13 | ✨ RepositoryService 레포 생성/링크/목록/멱등성 |
| `src/domains/deployment/deployment.test.ts` | 5 | ✨ DeploymentService 트리거/콜백/에러 |
| `src/domains/portal-user/portal-user.test.ts` | 14 | PortalUserService CRUD, 역할 관리 |
| `src/domains/portal-user/rbac-guard.test.ts` | 7 | RBAC 관리자 관리/데코레이터 |
| `src/domains/task/task.test.ts` | 5 | TaskService 생성/조회/시뮬레이션 |
| `src/infrastructure/github/github-api.client.test.ts` | 11 | ✨ GitHubApiClient fetch mock |
| `src/infrastructure/bstage-api/bstage-gateway.client.test.ts` | 5 | ✨ BstageGatewayClient fetch mock |
| `src/lib/scaffold.test.ts` | 12 | Quick Start 스캐폴드 + Org 라우팅 + 환경 제한 |

### 10.3 Integration Test 파일 구조

| 파일 | 테스트 수 | 대상 |
|------|-----------|------|
| `src/__tests__/integration/custom-pages-api.test.ts` | 14 | Custom Pages API (CRUD + deploy + error) |
| `src/__tests__/integration/stages-api.test.ts` | 13 | Stages API (CRUD + deploy + webhook 5종) |
| `src/__tests__/integration/projects-api.test.ts` | 5 | Projects API (list + create) |
| `src/__tests__/integration/deployment-pipeline.test.ts` | 11 | 배포 파이프라인 통합 테스트 (정적 12 + API 3) |

### 10.4 Playwright E2E 구성

**설정 파일**: `playwright.config.ts`
- 브라우저: Chromium 단독
- Reporter: HTML (`playwright-report/`)
- Dev Server: `npm run dev` 자동 시작 (로컬)

**Spec 파일**:
| 파일 | 시나리오 | 설명 |
|------|----------|------|
| `e2e/stages.spec.ts` | 3 | Stage 목록/상세/GitHub 탭 |
| `e2e/custom-pages.spec.ts` | 3 | Custom Page 목록/생성 폼/상세 |
| `e2e/deploy-flow.spec.ts` | 3 | 배포 버튼/확인 모달/환경 배지 |

**실행 명령**:
```bash
npm run test:e2e          # 헤드리스
npm run test:e2e:ui       # UI 모드 (QA용)
npm run test:e2e:debug    # 디버그 모드
```

### 10.5 Coverage 제외 파일

`vitest.config.ts`에서 다음 파일은 커버리지 측정에서 제외:
- `**/*.entity.ts` — 타입 정의 only
- `**/*.repository.ts` — Prisma 래퍼 (비즈니스 로직 없음)
- `**/index.ts` — barrel export
