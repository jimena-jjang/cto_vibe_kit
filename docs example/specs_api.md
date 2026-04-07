# [Specs] API & Business Logic

## 2. Business Logic (Service Layer)

> [!NOTE]
> v7 DDD 리팩토링으로 모든 비즈니스 로직이 Domain Service에 집중됩니다.
> Route Handler는 request/response 변환만 담당합니다.

### 2.1 CustomPageService

#### `createCustomPage(dto: CreateCustomPageDTO)`

**정식 저장 (`isDraft: false`)**:
1. `title` 검증: 필수
2. `urlPath` 검증: 필수 (비어있으면 `integrationConfig.targetUri` 사용)
3. `type` 검증: Phase 1에서는 `LIQUID`만 허용 → `400`
4. `stageId` + `projectId` 필수 → 없으면 `400`
5. `projectId` 미지정 시: 해당 Stage의 `미분류` 프로젝트 자동 할당 (없으면 자동 생성)
6. `exposureType === 'RESERVED'` → `exposureStartDate`, `exposureEndDate` 필수, `start < end` 검증

**임시 저장 (`isDraft: true`)**:
1. `title` 존재 여부만 확인

#### `updateCustomPage(id, data)`
- 기존 페이지 조회 → 없으면 `404`
- `integrationConfig` 업데이트 시 URI_MAPPING 모드이면 `targetUri` 자동 발번

#### `deleteCustomPage(id)`
- Hard delete

#### `triggerDeployment(customPageId, payload)`
- Concurrency Control: `QUEUED` | `IN_PROGRESS` | `PENDING` | `ROUTING` 상태 배포가 이미 있으면 `409` 반환
- `versionHash` 필수
- 새 Deployment를 `QUEUED` 상태로 생성
- `triggerFailed: true` 시 즉시 `TRIGGER_FAILED` 상태로 전환
- **(v8)** Multi-Org `orgType` 자동 결정: GitHub Repository URL에서 org를 추출 → `GITHUB_INTERNAL_ORG` 환경변수(`custom-apps-bmf`)와 비교 → 일치하면 `"internal"`, 불일치하면 `"external"` → DevOps Platform API 업로드 페이로드에 포함

### 2.2 StageService

#### `createStage(dto)`
- `domain` 검증: 반드시 `.` 포함 → 없으면 에러
- `spaceId` 자동 추출: `domain.split('.')[0]`

#### `triggerStageDeployment(stageId, payload)`
- Stage 단위의 Phase 1 배포 트리거
- 새 Deployment를 `stageId` + `phase: 1` + `QUEUED` 상태로 생성

#### Stage 조회
- `getStageById(id)`: ObjectId로 조회
- `getStageByDomain(domain)`: `findUnique({ where: { domain } })`
- API route에서 ID 형태를 확인하여 자동 분기

---

### 2.3 RepositoryService ✨ NEW (v7)

> **파일**: `src/domains/repository/repository.service.ts`
> **Infrastructure**: `GitHubApiClient`, `scaffold.util`

#### `listRepositories(stageId, dbClient)`
- StageRepository 조회 + 레거시 Stage fallback

#### `createRepository(dto)`
- GitHub 레포 생성 (template fork) + Webhook 등록
- Source 분기: INTERNAL(devops-bmf) / EXTERNAL(partners-bmf)
- Quick Start: DEV/QA 환경만 허용

#### `linkRepository(dto)`
- 기존 레포 연결 (멱등성 보장)

---

### 2.4 DeploymentService ✨ NEW (v7)

> **파일**: `src/domains/deployment/deployment.service.ts`
> **Infrastructure**: `GitHubApiClient`

#### `triggerStageDeploy(stageId, payload, dbClient)`
- Concurrency Control: 진행 중인 배포가 있으면 409
- Deployment를 QUEUED 상태로 생성
- GitHub Actions `workflow_dispatch` 트리거

#### `handleCallback(stageId, payload, dbClient)`
- `tag` 기반으로 PENDING/QUEUED/IN_PROGRESS 배포를 SUCCESS로 전환
- 모든 환경(REAL/QA/DEV) DB 순회

---

## 3. API Endpoints (상세)

### 3.1 Custom Pages

#### `GET /api/v1/custom-pages`
- **Query**: `type`, `status`, `stageId`, `projectId`, `search` (모두 선택)
- **Response**: `200` → CustomPage[]

#### `POST /api/v1/custom-pages`
- **Body**: CustomPage 생성 DTO (title, urlPath, stageId, type, integrationMode 등)
- **Response**: `201` → `{ customPage, customPageId, status }`

#### `GET /api/v1/custom-pages/check-name?name={name}`
- 커스텀 페이지 제목 중복 체크
- **Response**: `200` → `{ isDuplicate: boolean }`

#### `GET /api/v1/custom-pages/:id`
- **Response**: `200` → CustomPage (full include)

#### `PATCH /api/v1/custom-pages/:id`
- **Body**: `Partial<CustomPage fields>`
- **규칙**: 비활성 상태에서는 타겟 URI 변경이 차단됨. 활성 상태이며 현재 노출 중(IMMEDIATE 또는 RESERVED 기간 내)인 경우에도 URI 변경 불가.
- **Response**: `200`

#### `DELETE /api/v1/custom-pages/:id`
- Hard delete → `200`

#### `POST /api/v1/custom-pages/:id/deploy`
- **Body**: `{ versionHash: string, versionName?: string }`
- **Internal**: DevOps Platform API 호출 시 `orgType` (`"internal"` | `"external"`) 자동 결정하여 페이로드에 포함
- **Response**: `201` → Deployment 객체
- **Error**: `400` (versionHash 누락), `409` (배포 진행 중)

> [RESTORED] `POST /api/v1/custom-pages/:id/deploy/:deploymentId/preview` — **[2026-04-05 재구현]** 보안 강화 프리뷰 시스템으로 재도입.

### 3.10 Preview Mode (🔍 프리뷰)

#### `POST /api/v1/custom-pages/:id/deploy/:deploymentId/preview`
- 프리뷰 세션 생성
- **Body**: `{ expiresInMinutes?: number, allowedEmails?: string[], memo?: string }`
- **규칙**: 만료 시간 1분~4320분(72시간), 기본 60분
- **동작**: 고유 `previewKey` 생성 → S3 라우트 자동 등록 (`/preview/{key}/{urlPath}`)
- **Response**: `201` → `{ preview: PreviewSession }`

#### `GET /api/v1/custom-pages/:id/preview`
- 해당 CustomPage의 모든 프리뷰 세션 목록 조회
- **동작**: 만료된 세션은 자동으로 EXPIRED 업데이트
- **Response**: `200` → `{ previews: PreviewSession[] }`

#### `GET /api/v1/preview/:previewKey`
- 프리뷰 접근 검증 (미들웨어 역할)
- **동작**: `allowedEmails` 매칭, 접근 로그 기록, 비인가 5회 초과 시 자동 Kill
- **Response**: `200` (인가) / `403` (비인가) / `410` (만료/killed)

#### `DELETE /api/v1/preview/:previewKey`
- 개별 Kill Switch — 프리뷰 즉시 비활성화
- **동작**: status → KILLED, S3 라우트 제거, AuditLog 기록
- **Response**: `200`

#### `POST /api/v1/custom-pages/:id/preview/kill-all`
- 전체 Kill Switch — 해당 CustomPage의 모든 활성 프리뷰 비활성화
- **Response**: `200` → `{ killedCount: number }`

#### `POST /api/v1/preview/:previewKey/share`
- 공유 대상 이메일 추가
- **Body**: `{ email: string }`
- **Response**: `200` → 업데이트된 `allowedEmails[]`

#### `GET /api/v1/preview/:previewKey/logs`
- 접근 로그 및 통계 조회
- **Response**: `200` → `{ logs: PreviewAccessLog[], stats: { total, authorized, unauthorized } }`

#### `POST /api/v1/custom-pages/:id/deploy/:deploymentId/promote`
- Phase 2 Routing 배포 승격 (READY → ACTIVE)
- 기존 ACTIVE 배포를 INACTIVE로 전환 후 새 Phase 2 Deployment 생성
- **Body**: `{ scheduledAt?: string }` (예약 배포 지원)

#### `GET /api/v1/custom-pages/:id/reference-links`
- **Response**: `200` → ReferenceLink[]

#### `POST /api/v1/custom-pages/:id/reference-links`
- **Body**: `{ type, title, url, description?, icon?, createdBy? }`
- **Response**: `201`

#### `DELETE /api/v1/custom-pages/:id/reference-links/:slug`
- **Response**: `200`

### 3.2 Stages

#### `GET /api/v1/stages`
- **Response**: `200` → Stage[]

#### `GET /api/v1/stages/:id`
- **Parameter**: ObjectId 또는 domain 문자열 (자동 판별)
- **Response**: `200` → Stage (with `customPages`, `projects`)

#### `GET /api/v1/stages/:id/pages`
- **Parameter**: ObjectId 또는 domain (자동 판별)
- **Response**: `200` → CustomPage[]

#### `GET /api/v1/stages/:id/menus`
- Stage 메뉴 정보 조회

#### `POST /api/v1/stages/:id/deploy`
- **Phase 1 (Stage-level) 배포 트리거**
- **Body**: `{ versionHash?, versionName? }`
- **Response**: `201` → Deployment 객체

#### `POST /api/v1/stages/:id/deploy/callback`
- GitHub Actions Runner에서 S3 업로드 완료 후 호출하는 배포 콜백
- `tag`(versionHash) 기반으로 PENDING/QUEUED/IN_PROGRESS 배포를 검색하여 `SUCCESS`로 전환
- 모든 환경(REAL/QA/DEV) DB를 순회하여 매칭
- **Body**: `{ tag: string, commitHash?: string, artifactUrl?: string }`
- **Response**: `200` → `{ message, status: "SUCCESS" }`

#### `POST /api/v1/stages/:id/deploy-all`
- **Safeguarded Bulk Deploy**: 해당 Stage의 활성화된(isActive: true) 모든 CustomPage에 대해 배포 큐 일괄 추가
- **Body**: `{ versionHash?, versionName? }`
- **Response**: `201` → `{ targetPaths, deployments[] }`

#### `POST /api/v1/stages/:id/webhook`
- GitHub Webhook 수신 (push, workflow_run 이벤트)
- Phase 1 Deployment의 상태를 자동 업데이트

### 3.3 Projects (Node Groups)

#### `GET /api/v1/projects`
- **Response**: `200` → Project[] (노드 그룹 목록)

#### `GET /api/v1/projects/:id`
- **Response**: `200` → Project

### 3.4 Tasks

#### `GET /api/v1/tasks/:id`
- **Response**: `200` → Task (steps는 JSON.parse 후 반환)

### 3.5 Search

#### `GET /api/v1/search?q={query}`
- 전체 리소스 통합 검색: Projects, Stages, CustomPages에서 name/domain/title 매칭
- **Response**: `200` → `{ projects: [], stages: [], customPages: [] }`

### 3.6 Webhooks

#### `POST /api/v1/webhooks/github`
- GitHub push 이벤트 수신
- `repository.html_url`로 매칭 → `hasCode=true`, 상태 전환

#### `POST /api/v1/webhooks/github/actions`
- GitHub Actions `workflow_run` 이벤트 수신
- HMAC-SHA256 시그니처 검증 (`GITHUB_WEBHOOK_SECRET`)
- 모든 환경(REAL/QA/DEV) DB를 순회하여 해당 repo의 활성 Phase 1 배포를 검색
- 상태 전환: `requested` → `QUEUED`, `in_progress` → `IN_PROGRESS`, `completed+success` → `SUCCESS`, `completed+failure` → `FAILED`

### 3.7 External Stages

#### `GET /api/v1/external-stages`
- 외부 b.stage 플랫폼에서 스테이지 목록을 프록시 조회
- `x-environment` 헤더에 따라 DEV/QA/REAL 콘솔 API로 분기
- **Response**: `200` → `{ id, spaceId, domain, name, ogTagImgPath }[]`
- 60초 캐싱(`next.revalidate`)

### 3.8 Deployments

#### `POST /api/v1/deployments/timeout`
- 타임아웃된 배포 처리

### 3.9 Authentication (NextAuth)

#### `GET/POST /api/auth/[...nextauth]`
- **Provider**: Google OAuth
- **Access Control**: `@bemyfriends.com` 도메인 이메일만 로그인 허용
- **Middleware**: 비로그인 사용자는 인증된 세션이 없을 경우 접근 차단 (`NODE_ENV=development`일 때 인증 우회)
- **Tracking**: NextAuth 세션을 활용하여 주요 API 액션에 대한 사용자 이메일 트래킹 (AuditLog 및 생성자 추적)

### 3.10 RBAC Admin API (Super Admin 전용)

#### `GET /api/v1/admin/users`
- 전체 포털 사용자 목록 조회
- **Guard**: Super Admin 전용 (`requireSuperAdmin`)
- **Response**: `200` → `{ users: PortalUser[] }`

#### `POST /api/v1/admin/users`
- 사용자 역할 변경
- **Guard**: Super Admin 전용
- **Body**: `{ email, role, reason? }`
- **규칙**: 자기 자신 역할 변경 불가, RoleChangeLog 자동 기록
- **Response**: `200` → `{ user: PortalUser }`

#### `GET /api/v1/admin/users/me`
- 현재 로그인 사용자의 역할 정보
- **Response**: `200` → `{ email, role, name, isSuperAdmin }`
- 개발 환경에서는 세션 없이도 `SUPER_ADMIN` 반환

#### `POST /api/v1/admin/users/invite`
- 신규 사용자 초대 + 이메일 발송
- **Guard**: Super Admin 전용
- **Body**: `{ email, role?, message? }`
- **규칙**: `@bemyfriends.com` 도메인만 초대 가능, 중복 초대 차단
- **Response**: `201` → `{ user: PortalUser, emailSent: boolean }`

#### `GET /api/v1/admin/users/role-history`
- 역할 변경 이력 조회
- **Guard**: Super Admin 전용
- **Query**: `?email=xxx` (선택, 특정 사용자 필터링)
- **Response**: `200` → `{ history: RoleChangeLog[] }`

### 3.11 RBAC 가드 적용 API

아래 API들은 `x-environment: REAL`일 때만 Super Admin 검증:

| API | 가드 |
|-----|------|
| `POST /api/v1/stages/:id/deploy` | `requireRealDeployPermission` |
| `POST /api/v1/stages/:id/deploy-all` | `requireRealDeployPermission` |
| `POST /api/v1/custom-pages/:id/deploy` | `requireRealDeployPermission` |
| `POST .../deploy/:deploymentId/promote` | `requireRealDeployPermission` |