# Spec Changelog

> `docs/specs.md`의 변경 이력을 누적 기록합니다.
> 스펙이 추가/수정/삭제될 때마다 아래에 항목을 추가합니다.

---

## Format

```
### [YYYY-MM-DD] — 변경 요약
- **[ADDED/MODIFIED/REMOVED]** `항목명`: 변경 내용
- **Reason**: 변경 사유
- **Impact**: 영향 범위
- **Author**: 작업자
```

---

## Changelog

### [2026-04-06] — 스테이지 연동 해제 방식 변경 (Soft Delete → Hard Delete)

- **[MODIFIED]** `src/domains/stage/stage.repository.ts`: 스테이지 연동 해제 API (`delete()`) 호출 시 데이터베이스에서 `deletedAt` 업데이트(Soft Delete) 대신 `delete()` 호출(Hard Delete)로 로직 전면 수정
- **[NEW]** `cleanup-deleted-stages.ts`: 남아있는 과거 소프트 삭제 스테이지를 강제 삭제(Clean-up)하는 DB 마이그레이터 추가
- **Reason**: 사용자 요청사항 반영. 향후 충돌 요소를 원천 차단하고 연관 데이터가 DB에 계속 쌓여 과부하 또는 레거시 로직 에러를 일으키는 문제를 예방하기 위함. GitHub 레포지토리는 삭제하지 않고 보존.
- **Impact**: 스테이지 API 호출에 관련된 모든 하위 데이터(CustomPage, StageRepository, Deployment, Project 등)
- **Author**: Backend Agent


### [2026-04-06] — DevOps Platform API Multi-Org 대응 (orgType 파라미터 추가)

- **[MODIFIED]** `src/infrastructure/bstage-api/bstage-gateway.client.ts`: `UploadTemplatePayload` 인터페이스에 `orgType?: 'internal' | 'external'` 옵셔널 필드 추가
- **[MODIFIED]** `src/domains/custom-page/custom-page.service.ts`: `triggerDeployment()`에서 GitHub URL의 org를 추출하여 `GITHUB_INTERNAL_ORG` 환경변수와 비교 → `orgType` 결정 후 DevOps API payload에 포함
- **[MODIFIED]** `src/infrastructure/bstage-api/bstage-gateway.client.test.ts`: `uploadTemplate` 테스트에 `orgType` 필드 추가
- **[MODIFIED]** `src/domains/custom-page/custom-page.test.ts`: `triggerDeployment` 테스트에 internal/external org 구분 검증 케이스 2건 추가
- **[HOTFIX]** `src/domains/deployment/deployment.service.ts`: 워크플로우 파일명 `upload-3pp-templates.yml` → `upload-templates.yml` 수정 (실제 template-liquid 레포 파일명과 일치)
- **Reason**: DevOps Platform API가 GitHub org를 `partners-bmf`로 하드코딩하여 `custom-apps-bmf` org 레포 dispatch 시 404 발생. API에 `orgType` 파라미터가 추가됨에 따라 포털에서 올바른 값을 전송하도록 대응.
- **Impact**: CustomPage 배포 API payload (`POST /api/v1/custom-pages/upload`), Phase 1 배포 워크플로우 파일명
- **Author**: Antigravity Agent

### [2026-04-06] — 파일-URI 매핑 UI + templateMappingPath 설정

- **[NEW]** `src/app/api/v1/stages/[id]/repositories/[repoId]/files/route.ts`: `GET /stages/:id/repositories/:repoId/files` — GitHub Contents API를 통해 `public/user/` 하위 디렉토리 조회 + `template.liquid` 존재 여부 확인
- **[MODIFIED]** `src/infrastructure/github/github-api.client.ts`: `listDirectoryContents()` 메서드 및 `GitHubContentItem` 인터페이스 추가
- **[MODIFIED]** `src/app/custom-pages/new/page.tsx`: 파일 브라우저 UI (📂 레포에서 선택 / ✏️ 직접 입력 모드 전환), `templateMappingPath` API payload 포함
- **[MODIFIED]** `src/features/custom-page-catalog/ui/OverviewTab.tsx`: 기존 커스텀 페이지 `templateMappingPath` 편집 가능, PATCH 요청 시 포함
- **Reason**: URI 경로와 실제 레포 파일 경로가 다를 때 매핑이 깨지는 문제 해결. 사용자가 GitHub에서 직접 파일 구조를 확인하지 않아도 됨.
- **Impact**: Frontend UX (생성/수정) + Backend API (파일 트리 조회)
- **Author**: Antigravity Agent

### [2026-04-06] — 레포지토리 생성 UX 개선 + 504 Timeout 근본 해결

- **[MODIFIED]** `src/domains/repository/repository.service.ts`: Quick Start `applyQuickStart()`를 fire-and-forget 비동기 분리. 재시도 전략 exponential backoff(8회, ~515초) → 10분×2회로 단순화. HTTP 응답 블로킹 제거로 504 근본 해결.
- **[NEW]** `src/lib/safe-fetch.ts`: `safeParseJson()` (Content-Type 확인 후 안전 JSON 파싱), `getHttpErrorMessage()` (504/403/502 등 HTTP 상태별 한글 에러 메시지)
- **[MODIFIED]** `src/app/stages/[id]/page.tsx`: 레포 생성 다이얼로그 UX 전면 개선 — 모달 즉시 닫기 + sonner Toast(loading→success/error) + Pending 카드(⏱ 경과 시간 실시간 표시) + fetchRepos 함수 추출 + window.reload 제거 + safeParseJson 에러 방어
- **Reason**: Quick Start의 동기 블로킹이 ~515초까지 HTTP를 잡아두어 504 Gateway Timeout 발생. 모달이 떠있는 동안 사용자가 아무 작업 불가.
- **Impact**: Frontend UX + Backend API 응답 시간 (~3-7초로 단축)
- **Author**: Antigravity Agent

### [2026-04-06] — DDD 아키텍처 리팩토링 (v7: 6개 도메인 + Infrastructure 계층 분리)

- **[NEW]** `src/infrastructure/github/github-api.client.ts`: GitHub API 전용 클라이언트 (createRepoFromTemplate, registerWebhook, createFileContent, checkRepoExists)
- **[NEW]** `src/infrastructure/github/github-api.client.test.ts`: 11건 단위 테스트 (fetch mock)
- **[NEW]** `src/infrastructure/bstage-api/bstage-gateway.client.ts`: b.stage Gateway API 클라이언트 (uploadTemplate, syncRoutes, getRoutes)
- **[NEW]** `src/infrastructure/bstage-api/bstage-gateway.client.test.ts`: 5건 단위 테스트
- **[MOVE]** `src/lib/rbac.ts` → `src/infrastructure/guards/rbac.guard.ts` (re-export 유지)
- **[NEW]** `src/domains/repository/`: Repository 도메인 신설 (entity, repository, service, scaffold.util, 13 tests)
- **[NEW]** `src/domains/deployment/`: Deployment 도메인 신설 (entity, repository, service, 5 tests)
- **[MODIFIED]** `stages/[id]/repositories/route.ts`: 240줄 → 50줄 thin controller
- **[MODIFIED]** `stages/[id]/deploy/route.ts`: 94줄 → 47줄 thin controller
- **[MODIFIED]** `stages/[id]/deploy/callback/route.ts`: 129줄 → 32줄 thin controller
- **[MODIFIED]** `stage.service.ts`: `ensureGithubRepo()` → `GitHubApiClient` 전환, deprecated 마킹
- **[MODIFIED]** `docs/architecture.md`, `service-flow.md`, `specs_test.md`, `context.md`, `specs.md`, `specs_api.md`: DDD 현행화
- **Reason**: Route Handler 비즈니스 로직 과집중 해소 → DDD 패턴 분리 (테스트 가능성, 유지보수성, 확장성)
- **Impact**: Domain Layer 6개 도메인, Infrastructure Layer 신설, Route Handler ~50% 코드 감소, 테스트 156→223건
- **Author**: Antigravity Agent (7-Phase 일괄)

### [2026-04-05] — 내부/외부 레포 모드 분리 + Hello World Quick Start

- **[ADDED]** `prisma/schema.prisma`: `StageRepository.source` 필드 (`INTERNAL` | `EXTERNAL`, 기본값: EXTERNAL)
- **[ADDED]** `src/lib/scaffold.ts`: Hello World Quick Start 유틸 — GitHub Contents API로 기본 템플릿 자동 커밋
- **[ADDED]** `src/lib/scaffold.test.ts`: scaffold 유틸 + source 분기 + 환경 제한 테스트 (12건)
- **[MODIFIED]** `src/app/api/v1/stages/[id]/repositories/route.ts`: source 파라미터 + quickStart 옵션, org 분기 로직
- **[MODIFIED]** `src/domains/stage/stage.service.ts`: `ensureGithubRepo`에 source 파라미터/환경변수 체계 전환
- **[MODIFIED]** `src/app/stages/[id]/page.tsx`: 레포 생성 다이얼로그 (내부/외부 선택 + Quick Start 체크박스)
- **[MODIFIED]** `src/app/custom-pages/new/page.tsx`: 레포 목록 source 뱃지 (🏢내부/🤝외부)
- **[ADDED]** 환경변수: `GITHUB_INTERNAL_ORG`, `GITHUB_EXTERNAL_ORG`, `GITHUB_TEMPLATE_OWNER`, `GITHUB_TEMPLATE_REPO`
- **Reason**: 내부 직원(devops-bmf)과 외부 파트너(partners-bmf) 레포 생성 대상 분리 + DEV/QA 테스트 환경 빠른 셋업
- **Impact**: 레포 생성 API, 스테이지 상세 UI, 환경변수 설정, 데이터 모델
- **Author**: AI Agent

### [2026-04-05] — RBAC: REAL 환경 배포 권한 제어 (Super Admin)

- **[ADDED]** `prisma/schema.prisma`: `PortalUser` 모델 (포털 사용자 역할 관리 — email, role, isActive, lastLoginAt)
- **[ADDED]** `prisma/schema.prisma`: `RoleChangeLog` 모델 (역할 변경 이력 추적 — targetEmail, previousRole, newRole, changedBy, reason)
- **[ADDED]** `src/domains/portal-user/`: Domain Layer (entity, repository, service, email.service, barrel export)
- **[ADDED]** `src/lib/rbac.ts`: RBAC 가드 유틸리티 (`requireRealDeployPermission`, `requireSuperAdmin`, `getCurrentUserRole`)
- **[ADDED]** `src/store/userRoleStore.ts`: Zustand 기반 사용자 역할 캐시 store
- **[ADDED]** `GET /api/v1/admin/users`: 전체 사용자 목록 (Super Admin 전용)
- **[ADDED]** `POST /api/v1/admin/users`: 역할 변경 + RoleChangeLog 기록 + 자기 자신 변경 차단
- **[ADDED]** `GET /api/v1/admin/users/me`: 현재 사용자 역할 조회 (dev fallback 포함)
- **[ADDED]** `POST /api/v1/admin/users/invite`: 사용자 초대 + Nodemailer SMTP 이메일 발송
- **[ADDED]** `GET /api/v1/admin/users/role-history`: 역할 변경 이력 조회 (이메일 필터링 지원)
- **[ADDED]** `src/app/admin/page.tsx`: 관리자 설정 UI (사용자 테이블, 역할 변경 다이얼로그, 초대 다이얼로그, 변경 이력 탭)
- **[ADDED]** `src/app/admin/layout.tsx`: Super Admin 접근 가드 레이아웃
- **[MODIFIED]** `src/components/ui/sidebar.tsx`: `⚙️ 관리자 설정` 메뉴 (Super Admin만 표시)
- **[MODIFIED]** `src/components/ui/global-header.tsx`: 프로필 팝오버에 역할 뱃지 (🛡️ Super Admin / 👤 Member) 추가
- **[MODIFIED]** `src/components/ui/badge.tsx`: `destructive`, `secondary` variant 추가
- **[MODIFIED]** `DeploymentsTab.tsx`: REAL 환경 + Member일 때 배포/릴리즈 버튼 비활성화 + 안내 메시지
- **[MODIFIED]** `POST /api/v1/custom-pages/:id/deploy`: RBAC 가드 적용
- **[MODIFIED]** `POST .../promote`: RBAC 가드 적용
- **[MODIFIED]** `POST /api/v1/stages/:id/deploy`: RBAC 가드 적용
- **[MODIFIED]** `POST /api/v1/stages/:id/deploy-all`: RBAC 가드 적용
- **[MODIFIED]** `prisma/seed.ts`: `INITIAL_SUPER_ADMINS` 환경변수 기반 Super Admin 시딩 (7명)
- **[ADDED]** `portal-user.test.ts`: 14건 단위 테스트 (ensureUser, isSuperAdmin, setRole, inviteUser, getRoleChangeHistory)
- **[ADDED]** `rbac-guard.test.ts`: 7건 단위 테스트 (환경별 권한 검증, 세션 부재, 역할별 접근 제어)
- **Reason**: REAL(운영) 환경에서의 배포/삭제/라우팅 등 위험 작업을 지정된 Super Admin만 수행하도록 제약. DEV/QA는 기존과 동일하게 자유 사용 가능.
- **Impact**: 인증 시스템, 배포 API 전체, 관리자 UI, Prisma Schema 2개 모델 추가, API 5개 추가, 기존 API 4개 가드 적용
- **Author**: Antigravity Agent

### [2026-04-05] — RBAC 배포 검증 + Gmail SMTP + Super Admin Seed 핫픽스

- **[MODIFIED]** `next.config.ts`: `serverExternalPackages: ["nodemailer"]` 추가 (standalone 번들링 누락 수정)
- **[MODIFIED]** `src/domains/portal-user/email.service.ts`: Gmail SMTP 기본값 전환 (smtp.gmail.com:587, sender: june.kay@bemyfriends.com)
- **[MODIFIED]** `docker-compose.yml`: `INITIAL_SUPER_ADMINS`, `SMTP_PASS` 환경변수 추가
- **[MODIFIED]** `start.sh`: RBAC 환경변수 체크 로그 추가
- **[MODIFIED]** `prisma/seed.ts`: Super Admin seed `create` → `upsert` 변경 (MEMBER→SUPER_ADMIN 레이스 컨디션 해결)
- **[ADDED]** `POST /api/v1/admin/migrations/seed-super-admins`: 배포 후 즉시 Super Admin 적용 API (멱등)
- **Reason**: 컨테이너 배포 시 nodemailer 누락, Gmail SMTP 통합, Super Admin seed 레이스 컨디션 해결
- **Impact**: Docker 빌드, 이메일 발송, Super Admin 권한 적용
- **Author**: 🎖️ PL + Backend + DevOps

### [2026-04-05] — Preview Mode: 보안 강화 프리뷰 시스템 구현

- **[ADDED]** `prisma/schema.prisma`: `PreviewSession` 모델 (프리뷰 세션 관리 — previewKey, expiresAt, allowedEmails, status, killReason)
- **[ADDED]** `prisma/schema.prisma`: `PreviewAccessLog` 모델 (접근 시도 기록 — accessorEmail, accessorIp, accessType)
- **[MODIFIED]** `prisma/schema.prisma`: `CustomPage`, `Deployment` 모델에 `previewSessions` 관계 추가
- **[ADDED]** `POST /api/v1/custom-pages/{id}/deploy/{deploymentId}/preview`: 프리뷰 생성 API (만료 시간 1분~72시간, 이메일 접근 제한, S3 라우트 자동 등록)
- **[ADDED]** `GET /api/v1/custom-pages/{id}/preview`: 프리뷰 목록 조회 + 자동 만료 처리
- **[ADDED]** `GET /api/v1/preview/{previewKey}`: 프리뷰 접근 검증 (이메일 인증, 비인가 접근 5회 시 자동 Kill)
- **[ADDED]** `DELETE /api/v1/preview/{previewKey}`: 개별 Kill Switch
- **[ADDED]** `POST /api/v1/custom-pages/{id}/preview/kill-all`: 전체 Kill Switch
- **[ADDED]** `POST /api/v1/preview/{previewKey}/share`: 공유 대상 이메일 추가
- **[ADDED]** `GET /api/v1/preview/{previewKey}/logs`: 접근 로그 및 통계 조회
- **[MODIFIED]** `route-sync.util.ts`: `SyncOptions` 인터페이스 추가 — `previewRoute`, `includePreviewRoutes` 옵션
- **[MODIFIED]** `promote/route.ts`: Promote 완료 시 해당 CustomPage의 모든 활성 프리뷰 자동 EXPIRED 처리
- **[ADDED]** `PreviewPanel.tsx`: 프리뷰 관리 패널 UI (생성 폼, 카드 리스트, Kill Switch, 공유 모달, 접근 로그 모달)
- **[MODIFIED]** `DeploymentsTab.tsx`: 🔍 프리뷰 버튼 + PreviewPanel 통합
- **[ADDED]** `route-sync.test.ts`: Preview Mode 유닛 테스트 5건 (previewRoute 옵션, seo.noindex, layout, DB fetch, 에러 핸들링)
- **Reason**: 정식 배포(Promote) 전에 고유 프리뷰 URI로 페이지를 미리 확인하고 공유하는 기능 요청. 보안 강화 요구사항(이메일 접근 제어, 비인가 감지, Kill Switch) 포함.
- **Impact**: 커스텀 페이지 배포 플로우 전체 (생성 → 프리뷰 → 확인 → Promote), Schema 2개 모델 추가, API 7개 추가
- **Author**: 🎖️ PL + Backend + Frontend + QA

### [2026-04-05] — Phase 2: 커스텀 페이지 Multi-Repo UI 통합 (Fallback 내장)


- **[MODIFIED]** `custom-page.entity.ts`: `CustomPageEntity` 및 `CreateCustomPageDTO`에 `stageRepositoryId?: string | null` 필드 추가
- **[MODIFIED]** `custom-page.repository.ts`: `findAll()`/`findById()` 쿼리에 `stageRepository` include 추가, `create()`에 `stageRepositoryId` 조건부 저장
- **[MODIFIED]** `custom-page.service.ts`:
  - `createCustomPage()`: `stageRepositoryId` 미전달 시 `resolveRepositoryInfo()` 자동 매핑 (StageRepository → Stage 레거시 fallback)
  - `triggerDeployment()`: 레거시 `stage.githubRepoUrl` 직접 참조 → `resolveRepositoryInfo()` 간접 조회 전환
- **[MODIFIED]** `custom-page-shared/types.ts`: `Project` 인터페이스에 `stageRepository` 관계 타입 추가
- **[MODIFIED]** `OverviewTab.tsx`: 커스텀 페이지 상세에 Repository 카드 추가 (타입 뱃지, 레거시 뱃지, 미연결 표시)
- **[MODIFIED]** `custom-pages/new/page.tsx`: 스테이지 선택 시 레포 목록 fetch, 레포 선택 드롭다운 UI, API payload에 `stageRepositoryId` 포함
- **[MODIFIED]** `custom-page.test.ts`: mock DB에 `stageRepository`/`stage` 모델 추가 (151/151 passed)
- **[MODIFIED]** `package.json`: `postinstall`에 `prisma generate` 추가, `build`에 `prisma generate &&` 추가 (빌드 서버 Prisma 타입 안정성 보장)
- **Reason**: Multi-Repo 아키텍처(Phase 1)가 백엔드에 구축된 이후, 프론트엔드 커스텀 페이지 라이프사이클 전체(생성 → 조회 → 배포)에서 StageRepository를 명시적으로 선택·표시·사용하도록 통합. 마이그레이션 전/후 모두 동작하는 fallback 구조를 내장하여 향후 분기 작업 시 놓침 방지.
- **Impact**: 커스텀 페이지 생성/상세/배포 전체 흐름, Repository Resolver 의존성 확대
- **Author**: Antigravity Agent

### [2026-04-01] — 테스트 실패 9건 수정 + OOM 크래시 근본 원인 해결

- **[MODIFIED]** `stage.service.ts`: `ensureGithubRepo()` 내 GitHub 레포지토리 이름 가용성 검사 `while (!isAvailable)` 무한 루프에 MAX 20회 시도 제한 추가 (프로덕션 버그 수정)
- **[MODIFIED]** `vitest.config.ts`: `pool: 'threads'` + `testTimeout: 30000` + `hookTimeout: 30000` 설정 추가
- **[MODIFIED]** `route-sync.test.ts`: spaceId 기대값 수정 ('BTS'→'bts'), consoleSpy try/finally 래핑
- **[MODIFIED]** `custom-page.test.ts`: triggerDeployment 테스트에 `global.fetch` mock 추가
- **[MODIFIED]** `stages-api.test.ts`: `github-env-parser` + `infrastructure/database/prisma` vi.mock 추가
- **[MODIFIED]** `deployment-pipeline.test.ts`: 환경변수 `DEVOPS_PLATFORM_API_URL` → `APP_GITHUB_TOKEN` 수정
- **[MODIFIED]** `stage.test.ts`: `archiveStageDeployments` mock 추가, fetch mock에 name-check 단계 반영
- **Reason**: 테스트 스위트 안정화 (9건 실패 수정), OOM 크래시 방지 (프로덕션 무한루프 버그), CI/CD 파이프라인 신뢰성 확보
- **Impact**: 테스트 인프라 전체, `stage.service.ts` 서비스 로직
- **Author**: Antigravity Agent

---

### [2026-04-01] — 파이프라인 연동 고도화, UI 버그 수정 및 API 문서 최신화

- **[MODIFIED]** `StageService.ensureGithubRepo`: 이전에 삭제된 GitHub 레포지토리를 다시 연결할 때 발생하는 404 에러를 방지하기 위해, 사전에 GitHub API로 존재 여부를 검사하고 없으면 새로 생성하도록 체크 로직을 강화.
- **[MODIFIED]** 배포 파이프라인 (Phase 2): `UPLOADED` 상태의 Deployment를 승급(promote)할 수 없던 제약사항을 API 스펙(`docs/devops-platform-api-0401-ver2.md`)과 동기화하여 성공적으로 활성화 상태로 전환 가능하도록 수정. 
- **[MODIFIED]** `DeploymentsTab` / 배포 진행 UI: 타 컴포넌트(`ReleaseWizardModal`, `DeploymentHistory`) 구현 과정을 참고하여 스크린 라우팅/배포 프로세스가 완료되기 전 "진행바(초록색 선)" UI가 비정상적으로 노출되거나 렌더링이 엇나가는 버그를 검증 및 수정, 배포 상태에 맞는 정확한 UI 제공.
- **[MODIFIED]** 로깅 및 라우팅 (`performer`): 라우팅 설정 변경 시 고정된 값이 아닌, 현재 로그인 된 Auth 사용자(june.kay@bemyfriends.com 등)의 이메일 계정을 자동으로 인식하여 `performer`(수행자) 필드에 기록토록 수정.
- **[ADDED/MODIFIED]** API 문서화: DevOps Platform API 변경점 및 최신 스펙을 반영하여 `docs/devops-platform-api-0401-ver2.md` 문서 갱신.
- **Reason**: UI 디자인 일관성 보증 및 무결성 강화, 운영 관리 시 정확한 수행자 트래킹(Audit), 대상 플랫폼의 GitHub 연동 안정화 신규 개발된 외부 API 의존성 명세 반영 및 동기화.
- **Impact**: 스테이지 레포 연동 로직, CI/CD 배포 Promote 파이프라인, 플랫폼 UI 컴포넌트 연동 상태 표기, 사용자 로깅.
- **Author**: UI/UX, DevOps & Backend Agent

---

### [2026-03-31] — Liquid 라우팅 매핑 체계 도입 (templateMappingPath + per-route tag)

- **[ADDED]** `§1.3 CustomPage`: `templateMappingPath` 필드 추가. Liquid 타입 전용으로 URI와 실제 템플릿 파일 경로 간 명시적 매핑을 관리. 미지정 시 기존 네이밍 룰(urlPath → 파일명) 적용.
- **[MODIFIED]** `route-sync.util.ts`: `syncStageRoutes` 함수 리팩터링.
  - Root payload에서 `tag` 필드 제거 → 각 route 객체 내부에 per-route `tag` 삽입.
  - `page.type === "LIQUID"` 조건부로 `templateMappingPath` 속성 동적 주입.
  - 함수 시그니처 `tag` → `defaultTag` 변경 (폴백 용도로 사용).
  - 각 CustomPage의 `version` 필드를 개별 tag 소스로 우선 참조.
- **[MODIFIED]** `route-sync.test.ts`: 테스트 9건 → 12건 확장. 신규 테스트: templateMappingPath 포함/미포함, root tag 제거 검증, defaultTag 폴백, 비-LIQUID 타입 templateMappingPath 미포함.
- **[MODIFIED]** `§2-3 타입별 Git 레포지토리 전략`: LIQUID 타입 설명에 `templateMappingPath` 매핑 가능 안내 추가.
- **Reason**: Liquid 타입은 단일 리포지토리에 여러 템플릿이 공존하므로, URI와 파일명의 암묵적 1:1 강제 매핑을 탈피하여 명시적으로 매핑을 관리하는 구조 도입. 동시에 전역 tag 이슈(모든 라우트가 동일 버전 공유)를 해결하여 라우트별 독립 버저닝 지원.
- **Impact**: `prisma/schema.prisma`, `custom-page.entity.ts`, `route-sync.util.ts`, `route-sync.test.ts`, `specs.md`
- **Author**: 🎖️ PL, Backend Agent

---

### [2026-03-31] — 스테이지 레포지토리 자동 생성 버그 수정 (Legacy 호환성)

- **[MODIFIED]** `StageService.ensureGithubRepo`: 기존 레포지토리 재사용(옵션) 시, 연동되어 있던 레포가 GitHub에서 삭제되어 조회 실패 시 이전 레포 이름을 강제 생성하지 않고 최신 Naming 규칙(`.real` 등)을 엄격하게 준수하여 생성하도록 예외처리(Fallback) 보완.
- **Reason**: 유저가 수동 삭제한 과거 스테이지를 재생성할 때 규칙이 전파되지 않던 치명적 버그 수정.
- **Impact**: Backend Webhook Secret & Repo 발급 서비스 로직
- **Author**: UI/UX & Backend Agent

---

### [2026-03-31] — 스테이지 연동 및 시크릿 키 자동 발급 프로세스 보완

- **[MODIFIED]** `StageService.ensureGithubRepo`: 레포지토리 네이밍 규칙 업데이트 (디폴트 접미사 `-real` 적용) 및 기존 레포지토리 존재 시 `webhookSecret` 부재 문제 해결을 위한 멱등성 보완 로직 추가.
- **[MODIFIED]** `src/app/api/v1/stages/[id]/github/route.ts`: 스테이지의 Webhook Secret을 신규/재발급을 위한 API 추가.
- **[MODIFIED]** Frontend Stage 상세 페이지 (`src/app/stages/[id]/page.tsx`): 구 버전(Webhook Secret 미발급) 스테이지 사용자를 위한 "시크릿 발급하기" 수동 복구 액션 UI 및 연결 로직 작성. "레포지토리 생성" 목업 버튼 실 기능 탑재.
- **Reason**: 이전 기능에서 누락되었던 자동화 생성 처리 및 구 리소스 (시크릿 미발급 레포지토리)의 보안 에러를 호환/수정하기 위함.
- **Impact**: Backend Webhook Secret 발급 프로세스, Frontend Stage/Github 연동 상세 페이지.
- **Author**: UI/UX & Backend Agent

---

### [2026-03-31] — 스테이지 코드 연동 상태 감지 로직 수정

- **[MODIFIED]** `§4.5 Stage 상세`: `CustomBuildManager`에서 "코드 푸시 감지" 진행 단계 및 문구 노출 기준을 `hasCode` (저장소 코드 존재 여부)에서 `lastCodePushedAt` (실제 웹훅 기반 푸시 시각 존재 여부)로 변경.
- **Reason**: 템플릿 저장소의 연동 직후 기본 코드 존재 여부(`hasCode=true`)와 배포 목적의 실제 코드 Push 감지를 명확히 분리하기 위함. 이전 연결 데이터 누수로 인한 UI 모순 상태를 원천 차단.
- **Impact**: Frontend Stage 상세 페이지 (`src/app/stages/[id]/page.tsx`) 내 코드 연동 상태 UI
- **Author**: UI/UX & Frontend Agent

---


### [2026-03-31] — TO-BE 커스텀 페이지 단위 독립 배포 프로세스 도입

- **[MODIFIED]** `§1.4 Deployment`: Phase 1 배포 단위 처리 로직. `project.stage.deployments`와의 의존성(공통 이력 공유) 삭제하고 커스텀페이지별 독립화
- **[MODIFIED]** `§4.3 Deployments 탭`: 3단계 파이프라인(버전 등록 → 화면 승인 → 라우팅 적용)을 2단계(템플릿 업로드 → 화면 라우팅 완료)로 단순화
- **[MODIFIED]** `§4.5 Stage 상세`: '새 버전 스탬핑' 수동 트리거 및 모달 전면 제거, 코드 연동 상태 감지로 일원화
- **[MODIFIED]** `§4.2 커스텀 페이지 생성`: 기존 `URI_MAPPING` vs `MENU_REPLACEMENT` 라우팅 옵션 및 `urlPath` (Target URI) 입력 폼 복구. 선택된 스테이지에 따른 `MenuService` 동기화 렌더링 구현
- **[MODIFIED]** 모델 확장: `Project` 인터페이스에 배포 경로를 위한 `urlPath` 속성 추가
- **Reason**: 전체 스테이지를 묶어 배포/생성하던 기존의 모호하고 복잡한 AS-IS 방식을 탈피하여, 개별 페이지별로 독립적으로 템플릿 코드만 골라 업로드하는 TO-BE 방식 안착.
- **Impact**: Frontend 배포 파이프라인 UI, Stage 및 CustomPage 프론트엔드/백엔드 라우트, API 연동 등 전반
- **Author**: DevOps & Frontend Agent

---

### [2026-03-31] — Preview 기능 전체 제거 (난수 URI 발번, Preview Wizard, Preview API)

- **[REMOVED]** `§3 Custom Pages`: `POST /api/v1/custom-pages/:id/deploy/:deploymentId/preview` API 엔드포인트 삭제 (`preview/route.ts` 파일 삭제)
- **[REMOVED]** `§4.6 Feature 모듈`: `traffic-routing` 모듈에서 `PreviewWizardModal.tsx`, `usePreviewWizard.ts` 삭제
- **[REMOVED]** `§1.4 Deployment`: `previewUrl` 필드 삭제 (데이터 모델에서 제거)
- **[REMOVED]** `§2.1 CustomPageService.createCustomPage`: 커스텀 페이지 생성 시 `p-{random6}` 난수 URI 자동 발번 로직 삭제. URI는 사용자가 직접 입력해야 함
- **[REMOVED]** `§2.1 CustomPageService.updateCustomPage`: `p-` prefix URI에서 정식 URI로의 예외적 변경 허용 로직 삭제. 비활성 상태에서는 일률적으로 URI 변경 차단
- **[REMOVED]** `traffic-routing/domain/rules.ts`: `isRandomUri` 기반 `p-` 난수 폴백 URI 생성 로직 삭제
- **[REMOVED]** `traffic-routing/infrastructure/api.ts`: `createPreviewRouteApi` 함수 삭제
- **[REMOVED]** `route-sync.util.ts`: `previewRoute` 파라미터 및 프리뷰 라우트 병합 로직 삭제
- **[MODIFIED]** `§3 Custom Pages PATCH`: URI 변경 규칙을 `p-xxxx` 예외 없이 일관된 검증으로 단순화 (비활성 → 차단, 노출 중 → 차단)
- **[MODIFIED]** `deployment-pipeline/ui/DeploymentsTab.tsx`: 프리뷰 검증(isVerified) 게이트 제거, 릴리즈 버튼이 프리뷰 확인 없이 즉시 활성화
- **[MODIFIED]** `custom-page-catalog/ui/ProjectSummaryHeader.tsx`: `isRandomUri` 분기 제거, `isRouted` 판정 단순화
- **[MODIFIED]** `promote/route.ts`: `syncStageRoutes` 호출에서 `previewRoute` 인자 제거
- **[MODIFIED]** 4개 타입 파일에서 `previewUrl` 필드 제거 (`deployment-pipeline/domain/types.ts`, `custom-page-catalog/domain/types.ts`, `custom-page-shared/types.ts`, `traffic-routing/domain/types.ts`)
- **[MODIFIED]** `custom-pages/new/page.tsx`: 난수 URI 자동 생성 `useEffect` 제거, 안내 문구를 "URI 직접 입력" 방식으로 전면 수정
- **[MODIFIED]** `custom-pages/[id]/page.tsx`: `isRandomUri` 변수 및 관련 분기 제거
- **[MODIFIED]** 테스트 파일 2건 수정 (`custom-page.test.ts`, `route-sync.test.ts`): 프리뷰/난수 URI 관련 테스트 케이스 제거 및 기대값 수정
- **Reason**: Preview 단계가 실사용에서 불필요한 것으로 판단되어, 배포 플로우 단순화를 위해 프리뷰 기능 전체를 코드베이스에서 완전 제거. 기능 히스토리는 본 문서에서만 보존.
- **Impact**: Custom Page Service(URI 생성/검증), Traffic & Routing 모듈(UI/API/도메인), 배포 파이프라인 UI, Route 동기화 유틸, Deployment 데이터 모델, 프론트엔드 Custom Page 생성/상세 페이지.
- **Author**: 🎖️ PL

---

### [2026-03-31] — 전체 코드 대조 감사 결과 반영 (13건 불일치 수정)

- **[MODIFIED]** `§1.2 Stage`: `spaceId` nullable 표기, `webhookSecret`/`lastCodePushedAt`/`deletedAt` 필드 3개 추가
- **[MODIFIED]** `§1.3 CustomPage`: `stageId` nullable 표기 수정
- **[MODIFIED]** `§1.4 Deployment`: 상태값에 `SUCCESS`, `ROUTING` 추가
- **[MODIFIED]** `§1.5 Task`: `createdAt`/`startedAt`/`completedAt`/`estimatedTimeRemaining` 필드 4개 추가
- **[ADDED]** `§3.7 External Stages`: `GET /api/v1/external-stages` API 신규 문서화
- **[ADDED]** `§3 Custom Pages`: `POST /api/v1/custom-pages/:id/deploy/:deploymentId/preview` API 신규 문서화
- **[MODIFIED]** `§3.2 Stages deploy/callback`: Body를 `tag` 기반으로 수정 (기존 `status` 기반에서 변경)
- **[MODIFIED]** `§3.6 Webhooks/github/actions`: 상태 전환 로직 상세 기술 추가 (HMAC 검증, 환경 순회)
- **[REMOVED]** `§4.2 커스텀 페이지 목록 (/custom-pages)`: 삭제된 페이지 반영 (코드에서 삭제됨)
- **[MODIFIED]** `§4.3-4.6`: 커스텀 페이지 상세를 탭 기반 구조로 갱신, Feature 모듈(`src/features/`) 6개 문서화
- **[MODIFIED]** `§7`: E2E Tests 상태를 `📋 계획` → `✅ 인프라 완료`로 수정, Integration Tests 행 추가
- **[ADDED]** `§10.3`: `deployment-pipeline.test.ts` 통합 테스트 파일 추가
- **Reason**: Prisma 스키마, API 라우트, 프론트엔드 페이지, 테스트 파일 전체와 specs.md 100% 동기화
- **Impact**: specs.md만으로 시스템을 재구현할 수 있는 정확도 복원
- **Author**: 🎖️ PL

### [2026-03-30] — Sub-Agent 4-Layer 프로토콜 강제화, Tier 2 통합 테스트, 환경 안전 장치 도입

- **[ADDED]** `agent.md` 참조하여 Section 0 `MANDATORY WORKFLOW PROTOCOL` 연동 — 4-Layer 강제 메커니즘 도입 (`CLAUDE.md`, `check-protocol.sh`)
- **[ADDED]** Tier 2 Integration Tests 31건 구현 완료 (`custom-pages-api.test.ts` 등 API 커버리지 확보)
- **[MODIFIED]** 기본 환경 설정을 DEV로 전환 및 REAL 환경 엑세스 시 2단계 안전장치(`RealModeConfirmDialog`) 추가
- **[MODIFIED]** `custom-pages` 목록 및 상세 페이지 UI/UX 개선 (빈 목록 CTA, Not Found 페이지 가이드 추가)
- **Reason**: 프로토콜 위반 원천 차단, API 신뢰성 확보, 프로덕션 환경 휴먼 에러 방지 및 사용성 개선.
- **Impact**: 프로젝트 전반적인 작업 프로세스(Git Workflow 포함), API 통합 시나리오, Frontend 환경 관리 로직.
- **Author**: 🎖️ PL, QA, Frontend, Backend Agent

---

### [2026-03-29] — DDD 리팩토링, 테스트 커버리지 90%, Playwright E2E 인프라

- **[MODIFIED]** `CustomPageService`: 중복 메서드 제거 (`findReferenceLinks`/`createReferenceLink`/`deleteReferenceLink`)
- **[ADDED]** `src/domains/custom-page/index.ts`: barrel export 추가
- **[MODIFIED]** `StageRepository`: `findActiveStageDeployment`, `createStageDeployment`, `updateDeployment` 3개 메서드 추가
- **[MODIFIED]** `StageService`: `Error` → `AppError` 전환 (6건), `dbClient.deployment` → `stageRepository` 위임
- **[ADDED]** `Section 10. 테스트 인프라` — 3-Tier 전략, 파일 구조, Playwright 구성, Coverage 제외 파일
- **[ADDED]** `custom-page.test.ts` 40 tests, `route-sync.test.ts` 9 tests, `stage.test.ts` +20 tests, `task.test.ts` +2 tests
- **[ADDED]** `playwright.config.ts`, `e2e/stages.spec.ts`, `e2e/custom-pages.spec.ts`, `e2e/deploy-flow.spec.ts`
- **[MODIFIED]** `vitest.config.ts`: entity/repository/index 파일 coverage 제외
- **[MODIFIED]** `package.json`: `test:e2e`, `test:e2e:ui`, `test:e2e:debug` 스크립트 추가
- **Reason**: DDD 아키텍처 정합성 확보, 테스트 커버리지 12.35%→92.97%, QA용 Playwright E2E 인프라 구축
- **Impact**: Backend 도메인 전체 (Service/Repository layer), 테스트 인프라 (Vitest + Playwright)
- **Author**: 🎖️ PL → Backend + QA

---
### [2026-03-27] — Custom Page Deployment Validation & History Fixes

- **[MODIFIED]** Custom Page 모델 로직: 비활성 상태에서는 타겟 URI 변경이 불가능하나, Preview 단계의 임시 난수 URI(`p-xxxx`)에서 정식 URI로 변경하는 경우는 예외적으로 허용하도록 유효성 검증 완화.
- **[MODIFIED]** 배포 파이프라인 버전 모델: 커스텀 페이지(프로젝트) 생성 시간과 무관하게, 과거의 모든 스테이지 배포 건들도 유효한 버전 이력(History)으로 취급하도록 필터링 조건 변경.
- **[MODIFIED]** 배포 파이프라인 UI: Preview Wizard 검증 완료 상태가 파이프라인 단계(Step 3)에 올바르게 반영되도록 로직 수정.
- **[MODIFIED]** UI 연동 로직: Custom Page 생성 폼이 프로젝트 폴더 그룹을 불러올 때 잘못 지정된 API(`custom-pages`) 대신 정상적인 엔드포인트(`/api/v1/projects`)를 호출하도록 수정.
- **Reason**: Preview 단계에서 정규 릴리스 승격 시 400 에러 차단, 이전 배포 버전에 대한 롤백/히스토리 트래킹 정상화 및 페이지 렌더링 검증 완료.
- **Impact**: Custom Page Service(URI 검증), 배포 파이프라인 도메인 규칙 및 UI, Custom Page 생성 프론트엔드 라우트.
- **Author**: DevOps & Frontend Agent

---

### [2026-03-27] — Browser-Based Health Check Delegation

- **[MODIFIED]** CI/CD 배포 파이프라인: 백엔드 배포 콜백 API(`POST /api/v1/stages/:id/deploy/callback`)에서 정적 Healthcheck 로직 삭제 및 `SUCCESS` 로 즉각 전환 처리. 실제 검증 주체는 프론트엔드의 Preview Wizard로 완전히 위임("Browser-Based Health Check").
- **Reason**: 외부 배포 검증을 브라우저 컨텍스트 내에서 수행하는 방식이 동적인 클라이언트 사이드 렌더링 확인에 보다 정확하고 안정적이기 때문.
- **Impact**: 배포 파이프라인(`api/v1/stages/[id]/deploy/callback/route.ts`), 환경 검증 방식.
- **Author**: CI/CD & Frontend Agent

---

### [2026-03-26] — DevOps Platform Route Overwrite & Preview Exclusion

- **[MODIFIED]** Custom Page 모델 및 라우팅: DevOps Platform API(`POST /api/v1/custom-pages/routes`)의 전체 덮어쓰기 특성을 반영하여, 해당 Stage 내 모든 활성 Custom Page의 라우트를 조회하고 병합하는 공통 유틸리티(`syncStageRoutes`) 구현 적용
- **[MODIFIED]** Custom Page 라우팅 모델: 임시 발급된 미리보기용 난수 URI(`p-xxxx`)가 정규 라우팅 페이로드에 포함되지 않도록 명시적 제외(필터링) 로직 추가
- **[MODIFIED]** `.../preview` 및 `.../promote` API: 개별 라우트 전송 방식에서 전체 덮어쓰기 병합 방식으로 완전 전환 (`preview` 시에는 임시로 병합하여 전송)
- **Reason**: 단일 라우트 전송 시 Stage 내의 다른 정상 활성 라우트들이 삭제되는 크리티컬 버그 방지 및 불필요한 미리보기 임시 경로들의 운영 찌꺼기(Lingering) 노출 차단
- **Impact**: Traffic & Routing 도메인의 Route 동기화 유틸리티, CI/CD 배포(Preview, Promote) API 전반
- **Author**: DevOps & Frontend Agent

---

### [2026-03-26] — Custom Page Deployment Automation & Layout Settings
 
- **[MODIFIED]** Custom Page 모델 및 라우팅: 기본 랜딩 페이지를 `/stages`로 리다이렉트 처리 추가
- **[MODIFIED]** Custom Page UI (Traffic & Routing): GNB 노출 설정을 모바일과 PC 각각 상/하단바 제어가 가능한 세밀한 `Layout Settings` 기능으로 전면 개편
- **[MODIFIED]** CI/CD 배포 파이프라인: 수동 버전 문자열 입력 대신 자동화된 Versioning 명명 규칙 적용 및 사전 검증을 위한 Preview 전용 API Endpoint(`.../preview`) 추가
- **[MODIFIED]** Custom Page UX: "신규 배포 시작" 버튼 제거 및 GitHub 버전 확인 로직 개선
- **Reason**: 버전 관리 일관성 제어, 커스텀 페이지 렌더링 검증, 사용자 편의성 확장
- **Impact**: Traffic & Routing 도메인 UI 전반, CI/CD 도메인 배포 API 라우트
- **Author**: Frontend & DevOps Agent
 
---

### [2026-03-26] — UI/UX Refinement & Custom Page Deployment Enhancements
 
- **[MODIFIED]** UI/UX 개선: 기본 랜딩 페이지를 홈(`/`)에서 `/stages`로 변경. `nodeGroups` 조회 시 "전체(All)" 탭 추가. Custom Page 상세 탭 이름 직관화('프로젝트 개요' → '개요').
- **[MODIFIED]** Custom Page 모델 및 목록: 커스텀 페이지 목록(`/custom-pages`)에 "Stage" 식별 컬럼 추가 및 상세 페이지 딥링크 라우팅 지원.
- **[MODIFIED]** Deployment Workflow (Phase 2): "Deploy Code" 버튼 명칭을 버전 관리 및 커스텀 페이지 렌더링 기반임을 명확히 하도록 수정. 스테이지 배포 트리거를 GitHub Actions 의존성에서 `devops-platform-api` (`/api/v1/custom-pages/upload`) 호출 방식으로 완전 전환.
- **[MODIFIED]** Stage 연동 (Integration): 스테이지 연동 확인 모달 디자인 및 사용성 최적화(GitHub 레포 자동 생성, 에셋 폴더 업로드 필요성, 커스텀 페이지 등록 절차 안내 문구 추가).
- **Reason**: 사용자 경험(UX) 향상, 배포 및 버전 관리 개념의 명확화, 신규 아키텍처(Platform API 기반 배포 파이프라인)로의 이관 구체화.
- **Impact**: Frontend 전체 라우팅 및 UI/UX, Stage / Custom Page 도메인 연동 및 UI 컴포넌트, CI/CD 배포 API 연동 스크립트.
- **Author**: Frontend & DevOps Agent
 
---

### [2026-03-26] — DevOps Platform Architecture Refactoring & Deployment Reliability Enhancements

- **[MODIFIED]** Frontend Architecture: Custom Page 상세 페이지(`custom-pages/[id]/page.tsx`)를 5개 도메인(Catalog, Resource, CI/CD, Traffic & Routing, Audit & ACL)으로 분리하는 DDD 리팩토링(Facade 패턴 등) 완료.
- **[MODIFIED]** Custom Page 모델 로직: 라우팅 URI 변경 제한 강화(활성화 상태 시 수정 불가), 과거 배포 버전(deploymentId) 선택을 통한 롤백 기능 지원 및 `promote` API 연동.
- **[MODIFIED]** CI/CD 배포 파이프라인: 배포 timeout 처리 매커니즘 구현, `TRIGGER_FAILED` 명확화, `versionHash` 필수값 지정을 통한 배포 콜백 검증 강화 및 GitHub API (`deploy.yml`) 연동. 라이브 릴리스 전 변경사항 선검증을 위한 **Preview Wizard** 기능 도입.
- **[MODIFIED]** Audit Log 모델: "system" 대신 현재 접속 중인 사용자의 이름(Actor)이 기록되도록 개선.
- **[MODIFIED]** Stage UX/API: Code Deployment 진행 시 버전 입력을 지원하는 모달 추가 및 기존 Custom Build 상태 뷰 제거. Node Group 별 노출을 위해 API 및 화면 개선. 커스텀 페이지 목록에 연결된 스테이지 정보를 표시하고 딥링크 연동. Stage Soft delete 지원.
- **[MODIFIED]** Custom Page UX: 사용자 친화적 명칭을 위해 '프로젝트 개요' 탭을 '개요'로 수정.
- **Reason**: 뷰 컴포넌트 간 결합도를 낮추어 운영 및 확장성을 크게 높이고, 배포 오동작 방지(Preview Wizard) 및 사용성, 히스토리 신뢰성(Audit) 강화.
- **Impact**: Frontend Custom Page 라우트(`src/app/custom-pages/[id]`), Stage 및 CI/CD 도메인 API 라우트 전반.
- **Author**: Fullstack Agent

---

### [2026-03-20] — Stage Repository Validation & Custom Page Management Revision

- **[MODIFIED]** Stage 연동 로직: 스테이지 연동 시 사용할 레포지토리 템플릿(LIQUID, Web Component 등) 선택 및 기존 레포지토리 재사용 여부를 결정하도록 기능 개선.
- **[MODIFIED]** Custom Page 모델: URL 식별자 제약 조건 완화(한글 및 주요 특수문자 허용).
- **[MODIFIED]** 배포 파이프라인 UX: Custom Page 배포 단계 UI 시각화 고도화(진행/완료/대기 상태 별 색상 표시 추가). GNB 및 Service open 문구 수정.
- **[ADDED]** 외부 API 연동: Stage Menu 구성을 위해 외부 스테이지 API를 직접 호출하여 동적으로 메뉴를 렌더링하도록 커스텀 페이지 라우팅 로직 연동(통합 모드 기능 고도화).
- **[MODIFIED]** 버그 수정: /api/v1/stages 500 Error 수정 및 Next.js 15 정적 빌드 캐싱 이슈/라우팅 타입 오류 해결.
- **Reason**: 스테이지 연동 및 저장소 생성 유연성 증대, 배포 프로세스 사용자 인지성 강화, 프레임워크 최신버전 호환성 에러 해결.
- **Impact**: Stage / Custom Page 도메인 엔티티, 스테이지 생성 및 연결 API, 관련 프론트엔드 UI.
- **Author**: UI/UX & Backend Agent

---

### [2026-03-19] — Custom Page 파이프라인(Phase 2) 고도화 및 UI/UX 개선

- **[MODIFIED]** Custom Page 모델: Phase 2(라우팅 배포) 기능 및 미리보기/승인 4단계 파이프라인 로직 적용
- **[MODIFIED]** 배포 흐름: `DeploymentsTab` 내 프로젝트 레벨(Phase 1)과 개별 페이지 레벨(Phase 2) 배포 이력 및 상태 노출 분리
- **[MODIFIED]** Custom Page UI: "라우팅 및 노출" 탭을 "노출 설정 관리" (Exposure Management)로 명칭 변경, 노출 설정 영역에 "GNB 노출" 선택 및 가이드 문구 추가
- **[MODIFIED]** Stage 상세 UI: "Stage Info" 탭 내 스테이지 기본 정보와 GitHub 연동 정보 카드 시각적 분리
- **[MODIFIED]** 공통 UI/UX: LNB UI 개선 및 드롭다운 네비게이션 적용, 불필요한 용어(2phase) 제거 및 툴팁 추가
- **[MODIFIED]** 버그 수정: Stage 조회 시 500 API 에러 수정, 빌드 단계의 Route Handler `context.params` 및 Prisma 타입 오류 해결
- **Reason**: 사용자 친화적인 배포 상태 추적 제공 및 서비스(프로덕션 배포) 안정성 확보
- **Impact**: Stage/Custom Page 엔티티 관련 상태 표시, 전반적인 Frontend UI(스테이지/커스텀 페이지 상세 화면) 등 전체
- **Author**: UI/UX, Frontend & Backend Agent

---

### [2026-03-18] — inhouse dev portal 개편 및 Custom Page 고도화

- **[MODIFIED]** 텍스트 전면 개편: b.stage Dev Portal -> inhouse dev portal, Custom Build -> Custom Page, 프로젝트 폴더 -> 커스텀 페이지
- **[MODIFIED]** Stage 모델: 스테이지 연동 시 자동으로 GitHub 레포지토리를 생성하도록 로직 변경 (`githubRepoUrl`, `githubRepoId`, `hasCode` 필드 추가)
- **[MODIFIED]** Custom Page 모델 및 정책: 부모 스테이지의 레포지토리를 상속받도록 결합도 상향, 개별 페이지의 레포지토리 생성 버튼 제거
- **[MODIFIED]** Custom Page UI: "노출 설정" 및 "GNB 설정" UI를 생성 화면과 동일하게 정렬, "고급 라우팅 맵" 기능 제거, 폴더 목록을 테이블 레이아웃으로 개편
- **Reason**: 사용자 경험 향상 및 스테이지-페이지 간의 레포지토리 관리 구조 일원화
- **Impact**: Stage/Custom Page 도메인 엔티티, 관련 API 서비스 로직, 프론트엔드 UI 컴포넌트 전반
- **Author**: UI/UX & Backend Agent

---

### [2026-03-18] — 환경 격리 및 프로덕션 시스템 안정화

- **[MODIFIED]** API 및 DB 설정: DEV, QA, REAL 환경 분리 시 각 시스템 상태(UI/API)에 올바른 DB가 연결되도록 격리 누락 수정
- **[MODIFIED]** CI/CD 파이프라인: 프로덕션 로그인 이슈 대응을 위해 Next.js 정적 빌드 충돌 해결 및 Dockerfile 환경변수 주입, Github Secrets 배포 로직 개선
- **[ADDED]** 애플리케이션 공통: NextAuth 기반 API 호출자 이메일 트래킹 시스템 전면 적용 및 Sonner 기반 Toast 알림 시스템 통합 추가
- **Reason**: 프로덕션 배포 안정화, 이슈 추적성 강화, 환경 분리 완성 및 사용자 경험 향상
- **Impact**: Backend API 전반, Auth 시스템, UI 공통 컴포넌트, CI/CD 스크립트
- **Author**: DevOps & Frontend Agent

---

### [2026-03-17] — 프로젝트 및 스테이지 UI/UX 개선 및 기획(PM) 에이전트 스킬 추가

- **[ADDED]** `GET /api/v1/projects`: `search` 쿼리 파라미터 추가
- **[MODIFIED]** 스테이지 목록 (`/stages`): 카드 디자인 최적화, 노드 그룹 필터링 및 이름 검색 추가, 프로젝트 그룹/프로젝트 수 노출로 정책 변경
- **[MODIFIED]** 스테이지 상세 (`/stages/:id`): '스테이지 정보' 탭 이름 변경 및 메타데이터, 도메인 하이퍼링크 UI 정렬
- **[ADDED]** 제품 기획(PM) 관련 포괄적인 에이전트 스킬 셋(`.agent/skills/pm-*`) 추가
- **Reason**: 프로젝트 검색 편의성 강화 및 스테이지 데이터 표현의 정확성 확보, 기획 단계 AI 지원 강화
- **Impact**: `/api/v1/projects`, `/stages` UI, 시스템 스킬 확장
- **Author**: Frontend Agent (jimena-jjang)

---

### [2026-03-17] — 구글 사내망 로그인 (NextAuth) 연동

- **[ADDED]** 구글 로그인을 통한 사내 이메일(`@bemyfriends.com`) 접근 제어 정책
- **[ADDED]** `src/app/api/auth/[...nextauth]`: NextAuth Provider/Callback 라우트
- **[MODIFIED]** `src/middleware.ts`: 인증 미들웨어를 통한 사내망 접근 제어
- **Reason**: 허용된 사내 인원만 Dev Portal에 접근할 수 있도록 보안 강화
- **Impact**: 애플리케이션 전반적인 접근 제어 및 UI (GNB 프로필 등)
- **Author**: Frontend Agent (jimena-jjang)

---

### [2026-03-17] — 초기 스펙 문서 생성

- **[ADDED]** `docs/specs.md`: 서비스 스펙 명세서 초기 버전 작성
  - Data Models: 9 models (Project, ProjectGroup, Stage, CustomPage, Deployment, Task, ReferenceLink, AccessControl, AuditLog)
  - API Endpoints: 8 도메인, 19개 엔드포인트
  - Frontend Pages: 6개 페이지
  - Domain Services: 3 도메인 (project, stage, task)
  - Infrastructure: MongoDB, Docker, K8s, Prisma
  - Test Status: Unit 36/36, API 21/21, Confluence 4/4
  - Backlog: Phase 2-4 미구현 기능 정리
- **Reason**: 기존 `agent.md` 기획서에서 스펙 부분을 분리하여 체계적 관리
- **Impact**: 모든 에이전트가 참조하는 기준 문서
- **Author**: DevOps Agent

---

### [2026-03-17] — agent.md 재구조화

- **[MODIFIED]** `agent.md`: 2081줄 기획서 → 원칙/규칙/역할 문서로 전환
  - 아키텍처 원칙 (DDD, Layer Rules)
  - 코딩 규칙 (TypeScript, Database, Testing, Git Flow)
  - 외부 API 규칙 (Confluence, GitHub)
  - 에이전트 역할 정의
  - 문서 업데이트 규칙
- **[ADDED]** `docs/context.md`: 에이전트 간 공유 컨텍스트 문서
- **[ADDED]** `docs/spec-changelog.md`: 이 파일
- **Reason**: 멀티 에이전트 오케스트레이션을 위한 문서 구조 표준화
- **Impact**: 모든 에이전트의 작업 시작 프로세스 변경
- **Author**: DevOps Agent

---

### [2026-03-13] — MongoDB 전환 완료

- **[REMOVED]** SQLite 관련 모든 설정 및 코드
- **[MODIFIED]** `prisma/schema.prisma`: `provider = "mongodb"` 확정
- **[MODIFIED]** `.env`: MongoDB 로컬 URL 설정
- **[ADDED]** `.env.production`: 프로덕션 MongoDB 설정
- **Reason**: SQLite → MongoDB 전환 결정
- **Impact**: Database layer 전체
- **Author**: Backend Agent

---

### [2026-03-13] — 테스트 인프라 구축

- **[ADDED]** Vitest 테스트 프레임워크 도입
- **[ADDED]** 도메인별 단위 테스트 (project 21, stage 12, task 3)
- **[ADDED]** `docs/TESTING.md`: TDD 가이드라인
- **Reason**: TDD 기반 개발 체계 수립
- **Impact**: 모든 도메인 개발 프로세스
- **Author**: QA Agent

---

### [2026-03-13] — API 버그 수정

- **[MODIFIED]** `POST /api/v1/projects`: 500 에러 수정 (optional 필드 처리)
- **[MODIFIED]** `GET /api/v1/stages/:id/pages`: 404 수정 (ObjectId/domain 자동 판별)
- **[ADDED]** `GET /api/v1/custom-pages/:id`: 405 수정 (GET 핸들러 추가)
- **Reason**: API 기능 검증 시 발견된 버그
- **Impact**: 프론트엔드 API 호출
- **Author**: Backend Agent
