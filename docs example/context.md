# Agent Shared Context

> 이 문서는 **🎖️ Project Leader(PL)가 관리**하고, 모든 Sub-Agent가 참조하는 공유 컨텍스트입니다.
> 각 Sub-Agent는 작업 시작 전 반드시 이 문서를 읽고, 작업 완료 시 "최근 완료 작업"에 기록합니다.
> PL이 매 세션 종료 시 "현재 상태"를 최종 검수·갱신합니다.

**Last Updated**: 2026-04-06T14:40:00+09:00
**Updated By**: 🎖️ PL (Antigravity Agent)

---

## 현재 상태

### 브랜치
- **main**: v8 릴리즈 완료 (Multi-Org orgType)
- **develop**: main과 동기화 완료
- **활성 feature**: 없음
- **병합 완료**: `feat/multi-org-upload` (#112 → develop, #113 → main)

### 환경
- **로컬 DB**: MongoDB `localhost:27017/inhouse_dev_portal_dev`
- **프로덕션 DB**: `platform-mongo.bmfdev.io:27017` (Single-Node Replica Set 구성 요청 중)
- **Dev Server**: `npm run dev` → `http://localhost:3000`

### 빌드 / 테스트 상태
- **npm run build**: ✅ 성공 (2026-04-06)
- **npm test**: ✅ 223/223 passed (15 files) — DDD 리팩토링 후 신규 테스트 67건 추가
- **커버리지**: 77.11% stmts / 67.86% branches / 88.57% funcs
- **Playwright**: ✅ 인프라 구축 완료 (9 E2E specs)

### 아키텍처 (DDD 리팩토링 적용)
```
src/
├── app/api/v1/           ← Presentation (얇은 Controller, ~30~50줄/handler)
├── domains/              ← Domain Layer (비즈니스 로직 100%)
│   ├── stage/            # StageService (CRUD, GitHub, Webhook)
│   ├── custom-page/      # CustomPageService (CRUD, 라우트 동기화, 배포)
│   ├── repository/       # ✨ RepositoryService (생성/링크, Quick Start, Multi-Repo)
│   ├── deployment/       # ✨ DeploymentService (트리거, 콜백, 상태 전이)
│   ├── portal-user/      # PortalUserService (RBAC, 역할 관리)
│   └── task/             # TaskService (비동기 작업 추적)
├── infrastructure/       ← Infrastructure Layer
│   ├── database/         # Prisma (DEV/QA/REAL 멀티 DB)
│   ├── github/           # ✨ GitHubApiClient (11 tests)
│   ├── bstage-api/       # ✨ BstageGatewayClient (5 tests)
│   └── guards/           # ✨ RBAC Guard
└── lib/                  ← 순수 유틸 (비즈니스 로직 금지)
    └── safe-fetch.ts     # ✨ safeParseJson, getHttpErrorMessage
```

---

## 최근 완료 작업

| 날짜 | 작업 | 담당 | PL 검수 |
|------|------|------|---------|
| 2026-04-05 | **DDD Phase 1**: Infrastructure 계층 정립 (GitHubApiClient, BstageGatewayClient, RBAC Guard) | Backend | ✅ |
| 2026-04-05 | **DDD Phase 2**: Repository 도메인 신설 (route handler 240줄 → 50줄) | Backend | ✅ |
| 2026-04-05 | **DDD Phase 3**: Deployment 도메인 신설 (deploy 94→47줄, callback 129→32줄) | Backend | ✅ |
| 2026-04-05 | **DDD Phase 4**: stage.service → GitHubApiClient 전환, deprecated 마킹 | Backend | ✅ |
| 2026-04-05 | **DDD Phase 5**: architecture.md, specs_test.md DDD 현행화 | Docs | ✅ |
| 2026-04-05 | **DDD Phase 6**: service-flow.md v5 DDD 반영 | Docs | ✅ |
| 2026-04-05 | **DDD Phase 7**: context.md, specs.md, specs_api.md, spec-changelog.md 전체 업데이트 | Docs | ✅ |
| 2026-04-05 | Preview Mode: Schema + API 7개 + Frontend + Tests 5건 | PL + All | ✅ |
| 2026-04-05 | RBAC Super Admin: REAL 배포 권한 제어 시스템 | PL + All | ✅ |
| 2026-04-05 | Phase 2: 커스텀 페이지 Multi-Repo UI 통합 | Antigravity | ✅ |
| 2026-04-06 | **레포 생성 UX 개선**: 504 Timeout 근본 해결 + 즉시 닫힘 모달 + Toast + Pending 카드(경과시간) | Antigravity | ✅ |
| 2026-04-06 | **파일-URI 매핑 UI**: 레포 파일 브라우저 + templateMappingPath 설정 (생성/수정) | Antigravity | ✅ |
| 2026-04-06 | **Multi-Org orgType**: DevOps Platform API 업로드 페이로드에 `orgType` 파라미터 추가 (internal/external 자동 결정) | Antigravity | ✅ |

---

## 진행 중인 작업

| 브랜치 | 작업 | 상태 |
|--------|------|------|
| `feat/custom-page-preview` | 커스텀 페이지 프리뷰 화면 확인 | ⏳ 진행 중 |
| `feat/route-latest-template-url` | 기존 스테이지 route-latest.json templateURL 설정 | ⏳ 진행 중 |

---

## Sub-Agent 간 공유 정보

### 도메인-Route 매핑 (DDD 리팩토링 후)

| API Route | 담당 Domain Service | Infrastructure 의존 |
|-----------|--------------------|--------------------|
| `POST /stages/:id/repositories` | `RepositoryService.createRepository()` | GitHubApiClient |
| `GET /stages/:id/repositories` | `RepositoryService.listRepositories()` | — |
| `POST /stages/:id/deploy` | `DeploymentService.triggerStageDeploy()` | GitHubApiClient |
| `POST /stages/:id/deploy/callback` | `DeploymentService.handleCallback()` | — |
| `POST /stages/:id/deploy-all` | `DeploymentService.deployAll()` | — |
| `POST /custom-pages/:id/deploy` | `CustomPageService.triggerDeployment()` | BstageGatewayClient |
| `POST .../promote` | `CustomPageService` + route-sync | BstageGatewayClient |
| `GET /stages/:id/repos/:repoId/files` | `GitHubApiClient.listDirectoryContents()` | GitHubApiClient |
| `GET/POST /admin/users` | `PortalUserService` | RBAC Guard |
| `POST /admin/users/invite` | `PortalUserService.inviteUser()` | EmailService |

### Backend → Frontend
- **CustomPage API**: `/api/v1/custom-pages` (GET, POST, PATCH, DELETE)
- **Stage API**: `/api/v1/stages/:id/deploy`, `/api/v1/stages/:id/deploy-all`, `/api/v1/stages/:id/webhook`
- **Repository API**: `/api/v1/stages/:id/repositories` (GET, POST)
- **Admin API**: `/api/v1/admin/users`, `/api/v1/admin/users/me`, `/api/v1/admin/users/invite`
- **환경 헤더**: `x-environment` 헤더로 DEV/QA/REAL 환경 분기

### Backend → DevOps
- **DB 요구사항**: MongoDB Replica Set 필수 (Prisma 트랜잭션)
- **시드 스크립트**: `npx tsx prisma/seed.ts` — 컨테이너 시작 시 자동 실행
- **K8s Secret 필요**: `dev-portal-secrets` (MONGODB_USERNAME, MONGODB_PASSWORD)

### DevOps → All
- **프로덕션 URL**: `platform-mongo.bmfdev.io:27017`
- **배포 파이프라인**: K8s Deployment → `start.sh` (prisma db push → seed → server)
- **Confluence**: `bemyfriends.atlassian.net` / Space: `BEMYFRIEND`

### 🎖️ PL → QA (테스트 커버리지 현황)
| Tier | 종류 | 상태 | 상세 |
|------|------|------|------|
| Tier 1 | Unit Tests (Vitest) | ✅ 223 tests | stage 42, custom-page 40, route-sync 12, repository 13, deployment 5, portal-user 14, rbac-guard 7, task 5, github-api 11, bstage-gw 5, scaffold 12 등 |
| Tier 2 | Integration Tests | ✅ 48 tests | 4 API route test files |
| Tier 3 | E2E Tests (Playwright) | ✅ 인프라 완료 | 3 spec files (9 tests) |

---

## 알려진 이슈 / 주의사항

| ID | 내용 | 심각도 | 상태 | 담당 |
|----|------|--------|------|------|
| ISS-001 | `.env`에 스마트 따옴표 사용 시 API 인증 실패 | Critical | ✅ 해결됨 | DevOps |
| ISS-002 | curl로 Confluence API 호출 시 hang | High | ✅ Node.js 스크립트로 대체 | DevOps |
| ISS-003 | MongoDB Replica Set 미설정 시 Prisma 트랜잭션 실패 | High | ⏳ DevOps 팀 요청 중 | DevOps |
| ISS-004 | K8s `dev-portal-secrets` Secret 미생성 | Medium | ⏳ DevOps 팀 요청 중 | DevOps |
| ISS-005 | Prisma 타입 lint 워닝 (generate 필요) | Low | ℹ️ `postinstall`에 generate 추가로 완화 | Backend |

---

## 다음 작업 후보 (Backlog) — 🎖️ PL 관리

| 우선순위 | 작업 | 담당 Sub-Agent | PL 승인 |
|----------|------|----------------|---------|
| 🔴 High | **Preview 도메인 분리** (별도 feature 브랜치) | Backend | 📋 DDD PR 완료 직후 |
| 🔴 High | Phase 2: Stage 중심 IA 개편 (specs.md §8) | Frontend / Backend | 📋 계획 |
| 🔴 High | Phase 2: 배포 대상 가시성 + URI 상태 | Frontend / Backend | 📋 계획 |
| 🔴 High | Phase 2: Web Component/SDK 배포 구조 | DevOps / Backend | 📋 계획 |
| 🔴 High | Phase 2: Component Inject | Frontend / Backend | 📋 계획 |
| 🟡 Medium | GitHub Actions CI/CD 파이프라인 | DevOps | 대기 |
| 🟡 Medium | SSE 실시간 업데이트 구현 | Backend / Frontend | 대기 |

---

## 파일 변경 시 알림 규칙

| 파일 변경 | 알림 대상 | PL 검수 필요 |
|-----------|-----------|-------------|
| `prisma/schema.prisma` | Backend, QA | ✅ 필수 |
| `src/app/api/**` | Backend, Frontend, QA | ✅ 필수 |
| `src/domains/**` | Backend, QA | ✅ 필수 |
| `src/infrastructure/**` | Backend, QA | ✅ 필수 |
| `src/app/**/*.tsx` | Frontend | 선택 |
| `k8s/`, `Dockerfile`, `start.sh` | DevOps | ✅ 필수 |
| `docs/specs.md` | All | ✅ PL 직접 수행 |
| `docs/context.md` (이 파일) | All | ✅ PL 최종 검수 |
| `docs/agent_discussion.md` | All | 자동 기록 |
