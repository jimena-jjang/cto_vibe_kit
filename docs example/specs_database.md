# [Specs] Database Models

## 1. Data Models (Prisma / MongoDB)

### 1.1 Project (Node Group)

스테이지 내 커스텀 페이지를 그룹핑하는 경량 엔티티. 이전 버전의 복잡한 Project 모델에서 **그룹핑 전용 모델**로 단순화됨.

| Field | Type | Required | Default | 설명 |
|-------|------|----------|---------|------|
| `id` | ObjectId | auto | `@default(auto())` | PK |
| `name` | String | ✅ | — | 그룹 이름 (e.g. `미분류`, `이벤트 페이지`) |
| `description` | String? | — | null | 그룹 설명 |
| `stageId` | String | ✅ | — | 연결 Stage ID |
| `color` | String | — | `#3b82f6` | UI 표시 색상 |
| `order` | Int | — | `0` | 정렬 순서 |
| `createdAt` | DateTime | auto | `now()` | 생성일시 |
| `updatedAt` | DateTime | auto | `@updatedAt` | 수정일시 |

**Relations**: `stage`, `customPages[]`

### 1.2 Stage

b.stage 플랫폼의 고객 인스턴스.

| Field | Type | Required | Default | 설명 |
|-------|------|----------|---------|------|
| `id` | String | ✅ | manual | PK (auto가 아닌 수동 지정) |
| `domain` | String | ✅ unique | — | e.g. `test.bstage.in` (반드시 `.` 포함) |
| `spaceId` | String? | — | — | 도메인에서 자동 추출 (`test.bstage.in` → `test`), nullable |
| `name` | String | ✅ | — | 표시 이름 |
| `customBuild` | Boolean | — | `false` | 커스텀 빌드 활성화 여부 |
| `gnbConfig` | String? | — | null | GNB 설정 JSON |
| `basePath` | String? | — | null | 기본 경로 |
| `githubRepoUrl` | String? | — | null | GitHub repo URL (스테이지 연동 시 자동 생성) |
| `githubRepoId` | Int? | — | null | GitHub repo ID |
| `webhookSecret` | String? | — | null | GitHub Webhook 시크릿 (Webhook 검증용) |
| `hasCode` | Boolean | — | `false` | 코드 업로드 여부 |
| `lastCodePushedAt` | DateTime? | — | null | 마지막 코드 push 시각 |
| `deletedAt` | DateTime? | — | null | soft delete 마커 |

**Relations**: `customPages[]`, `projects[]`, `deployments[]`, `repositories[]`

### 1.2.1 StageRepository (v4+)

스테이지에 연결된 GitHub 레포지토리. 멀티 레포지토리 아키텍처 지원.

| Field | Type | Required | Default | 설명 |
|-------|------|----------|---------|------|
| `id` | ObjectId | auto | — | PK |
| `stageId` | String | ✅ | — | 연결 Stage ID |
| `url` | String | ✅ | — | GitHub Repository URL |
| `githubRepoId` | Int? | — | null | GitHub API Repository ID |
| `webhookSecret` | String? | — | null | Webhook 검증용 시크릿 |
| `type` | String | — | `"LIQUID"` | LIQUID / SDK_FULL / SDK_INPAGE |
| `source` | String | — | `"EXTERNAL"` | **(v6)** `INTERNAL`(devops-bmf) / `EXTERNAL`(partners-bmf) |
| `label` | String? | — | null | 사용자 구분용 별칭 |
| `hasCode` | Boolean | — | `false` | 코드 push 여부 |
| `lastCodePushedAt` | DateTime? | — | null | 마지막 코드 push 시각 |
| `isDefault` | Boolean | — | `false` | 해당 타입의 기본 레포 여부 |

**Relations**: `stage`, `customPages[]`

### 1.3 CustomPage

Stage별 커스텀 페이지 — **이전 Project 모델의 비즈니스 필드를 흡수**하여 커스텀 빌드의 핵심 엔티티로 승격됨.

| Field | Type | Required | Default | 설명 |
|-------|------|----------|---------|------|
| `id` | ObjectId | auto | — | PK |
| `projectId` | ObjectId? | — | null | 연결 Project(노드 그룹), nullable |
| `stageId` | String? | — | null | 연결 Stage (nullable) |
| `urlPath` | String | ✅ | — | URL 경로 (Stage + urlPath로 유니크 제약) |
| `title` | String | ✅ | — | 페이지 제목 |
| `githubFile` | String? | — | null | GitHub 파일 경로 |
| `projectName` | String? | — | null | 프로젝트명 (denormalized) |
| `version` | String? | — | null | 배포 버전 |
| `isActive` | Boolean | — | `true` | 활성화 여부 |
| `type` | String | — | `SDK_INPAGE` | `LIQUID` / `SDK_FULL` / `SDK_INPAGE` |
| `integrationMode` | String | — | `URI_MAPPING` | `URI_MAPPING` / `PLACEHOLDER` / `MENU_ITEM` |
| `integrationConfig` | String? | — | null | JSON 문자열. URI_MAPPING 시 `{targetUri: "p-xxxxx"}` |
| `ownerEmail` | String? | — | null | 소유자 이메일 |
| `description` | String? | — | null | 설명 |
| `status` | String | — | `CODE_WAITING` | `CODE_WAITING` → `CODE_UPLOADED` → `DEPLOY_REQUESTED` → `QA_REVIEW` → `ACTIVE` → `FAILED` |
| `isDraft` | Boolean | — | `false` | 임시저장 여부 |
| `exposureType` | String | — | `HIDDEN` | `IMMEDIATE` / `HIDDEN` / `RESERVED` |
| `exposureStartDate` | DateTime? | — | null | `RESERVED` 시 필수 |
| `exposureEndDate` | DateTime? | — | null | `RESERVED` 시 필수, start < end 검증 |
| `templateMappingPath` | String? | — | null | **Liquid 타입 전용**: URI와 실제 템플릿 파일 경로 간 명시적 매핑. 미지정 시 기존 네이밍 룰(urlPath → 파일명) 적용 |
| `activeDeploymentId` | String? | — | null | 현재 활성 배포 ID |
| `deletedAt` | DateTime? | — | null | soft delete 마커 |
| `scheduleStart` | DateTime? | — | null | 스케줄 시작 |
| `scheduleEnd` | DateTime? | — | null | 스케줄 종료 |
| `timezone` | String? | — | `Asia/Seoul` | 타임존 |
| `gnbDesktopTop` | Boolean | — | `true` | GNB 데스크탑 상단 표시 |
| `gnbDesktopBottom` | Boolean | — | `true` | GNB 데스크탑 하단 표시 |
| `gnbMobileTop` | Boolean | — | `true` | GNB 모바일 상단 표시 |
| `gnbMobileBottom` | Boolean | — | `true` | GNB 모바일 하단 표시 |
| `createdBy` | String? | — | null | 생성자 |
| `updatedBy` | String? | — | null | 마지막 수정자 |

**Unique Constraint**: `@@unique([stageId, urlPath])`

**Relations**: `project?`, `stage`, `deployments[]`, `accessControl[]`, `auditLogs[]`, `tasks[]`, `referenceLinks[]`

### 1.4 Deployment

2-Phase 배포 모델 — Stage 종속(Phase 1) 또는 CustomPage 종속(Phase 2).

| Field | Type | Required | Default | 설명 |
|-------|------|----------|---------|------|
| `id` | ObjectId | auto | — | PK |
| `stageId` | String? | — | null | Phase 1 배포 시 연결 Stage |
| `customPageId` | ObjectId? | — | null | Phase 2 배포 시 연결 CustomPage |
| `phase` | Int | — | `1` | `1`: GitHub Build 배포 (Stage 종속), `2`: Routing 배포 (CustomPage 종속) |
| `versionName` | String? | — | null | 사용자 노출 버전 (e.g. `1.0`) |
| `versionHash` | String | ✅ | — | 내부 식별 해시 / S3 경로용 |
| `dockerImage` | String? | — | null | Docker 이미지 경로 |
| `commitHash` | String? | — | null | Git commit SHA |
| `deployedAt` | DateTime | auto | `now()` | 배포 시각 |
| `deployedBy` | String? | — | null | 배포자 |
| `environment` | String | ✅ | — | `DEV` / `STAGING` / `PROD` |
| `routingPath` | String? | — | null | 라우팅 경로 |
| `status` | String | — | `PENDING` | `PENDING` → `QUEUED` → `IN_PROGRESS` → `SUCCESS` → `READY` → `ACTIVE` / `INACTIVE` / `SCHEDULED` / `FAILED` / `TRIGGER_FAILED` / `ROUTING` |
| `artifactUrl` | String? | — | null | 배포 아티팩트 URL |

| `scheduledAt` | DateTime? | — | null | 예약 배포 시각 |

**Relations**: `customPage?`, `stage?`

### 1.5 Task

자동화 태스크 추적.

| Field | Type | Required | Default | 설명 |
|-------|------|----------|---------|------|
| `id` | ObjectId | auto | — | PK |
| `customPageId` | ObjectId? | — | null | 연결 CustomPage |
| `type` | String | ✅ | — | `PROJECT_CREATE` / `DEPLOYMENT` / `ROUTING_UPDATE` |
| `status` | String | — | `PENDING` | `PENDING` → `IN_PROGRESS` → `COMPLETED` / `FAILED` / `CANCELLED` |
| `progressCurrent` | Int | — | `0` | 완료된 스텝 수 |
| `progressTotal` | Int | — | `7` | 전체 스텝 수 |
| `progressPercentage` | Int | — | `0` | 진행률 (0~100) |
| `steps` | String | — | `[]` | JSON: `TaskStep[]` |
| `createdAt` | DateTime | auto | `now()` | 생성일시 |
| `startedAt` | DateTime? | — | null | 작업 시작 시각 |
| `completedAt` | DateTime? | — | null | 작업 완료 시각 |
| `estimatedTimeRemaining` | Int? | — | null | 예상 남은 시간(초) |
| `createdBy` | String | ✅ | — | 생성자 |
| `metadata` | String? | — | null | JSON 메타데이터 |

**Relations**: `customPage?`

### 1.6 기타 모델

| Model | 설명 | 핵심 필드 |
|-------|------|-----------|
| `ReferenceLink` | **CustomPage별** 참조 링크 | customPageId, type(`GITHUB_REPO`\|`CONFLUENCE`\|...), title, url |
| `AccessControl` | **CustomPage별** RBAC | customPageId, userEmail, role(`OWNER`\|`ADMIN`\|`DEVELOPER`\|`VIEWER`) |
| `AuditLog` | 감사 로그 | customPageId?, action, actor, targetResource, details(JSON), ipAddress |

## 2. 스키마 변경 및 무중단 마이그레이션 원칙 (Zero-Downtime Migration Guidelines)

본 프로젝트는 운영 중인 시스템(특히 Portal)에서 불가피하게 데이터베이스 스키마(테이블 신설, 필드 분리 등)를 변경해야 할 때, 단 1초의 서비스 다운타임이나 데이터 유실을 방지하기 위해 **반드시 아래의 "3단계 무중단 마이그레이션 및 멱등성 보장 원칙"을 준수하여 아키텍처를 설계하고 배포**해야 합니다.

### 원칙 1. 하위 호환성(Backward Compatibility) 유지 선 배포 (Phase 1)
- **기존 필드 보존**: 신규 모델/필드를 추가하더도 기존의 레거시 필드는 즉시 삭제(Drop)하지 말고 `@deprecated` 주석 처리만 한 채 남겨둡니다.
- **Fallback 로직 필수**: 코드 레벨에서는 신규 아키텍처를 조회하되, 값이 존재하지 않는 경우(마이그레이션 전 데이터) 기존 레거시 필드를 Fallback으로 읽도록 방어 로직을 씌워 먼저 배포합니다.

### 원칙 2. 철저한 멱등성(Idempotency)이 보장된 데이터 동기화 (Phase 2)
- **1회성 마이그레이션 실행**: 코드가 배포되어 하위 호환 환경이 갖춰진 뒤, 구 데이터를 신규 테이블로 적재(Backfill)하는 One-off 스크립트(또는 Admin API)를 별도로 실행합니다.
- **[핵심] 멱등성 100% 보장**: 
  - 네트워크 오류나 관리자 실수로 인해 마이그레이션 스크립트가 여러 번 중복 실행되더라도 데이터베이스 상태가 꼬이거나 중복 레코드가 발생해서는 안 됩니다.
  - 마이그레이션 로직은 반드시 **"이미 타겟 데이터가 생성되었거나 연결(Mapping)이 완료되었는지 확인하고 참이라면 건너뛰는(Skip) 방어 코드"**를 선제로 담아야 합니다. 이를 통해 프로세스가 항상 예측 가능한 단일 상태(Deterministic State)로 수렴하도록 강제합니다.

### 원칙 3. 레거시 코드와 필드의 완전한 청소 (Phase 3)
- 데이터 동기화가 모두 안전하게 완료된 이후의 다음 정기 릴리즈에서 `Prisma Schema`상의 레거시 필드를 완전히 제거하고, 코드에 남겨둔 Phase 1의 Fallback 방어 로직 찌꺼기를 안전하게 정리(Drop)하여 신규 구조로 100% 전환합니다.

### 검증 및 추적 도구 (Verification & Tracking)

- **Phase 2 완료 검증 API**: 각 마이그레이션에는 대응하는 `verify` 엔드포인트를 함께 제공합니다. 이 API가 모든 환경에서 `ready: true`를 반환해야만 Phase 3에 진입할 수 있습니다.
- **포탈 관리자 알림 UI**: `GlobalHeader`에 `MigrationAlertBanner` 컴포넌트가 통합되어 있어, 포탈에 접속하는 관리자는 미완료 마이그레이션을 즉시 인지할 수 있습니다. 마이그레이션이 모두 완료되면 Phase 3 진행 가능 안내로 자동 전환됩니다.
- **마이그레이션 레지스트리**: [`docs/migrations.md`](./migrations.md)에 모든 마이그레이션 이력이 ORM 마이그레이션 파일처럼 누적 기록됩니다. 신규 스키마 변경 시 반드시 해당 문서에 항목을 추가해야 합니다.

### 1.9 PreviewSession

배포 전 프리뷰 세션 관리. 고유 `previewKey`로 임시 라우팅을 생성하여 페이지를 미리 확인.

| Field | Type | Required | Default | 설명 |
|-------|------|----------|---------|------|
| `id` | ObjectId | auto | `@default(auto())` | PK |
| `previewKey` | String | ✅ unique | — | 고유 프리뷰 키 (`crypto.randomUUID()`) |
| `previewUrl` | String | ✅ | — | 완전한 프리뷰 URL |
| `urlPath` | String | ✅ | — | 원본 CustomPage의 URL 경로 |
| `status` | String | ✅ | `ACTIVE` | `ACTIVE` / `EXPIRED` / `KILLED` |
| `allowedEmails` | String[] | — | `[]` | 접근 허용 이메일 목록 (빈 배열 = 공개) |
| `expiresAt` | DateTime | ✅ | — | 만료 일시 |
| `memo` | String? | — | null | 관리용 메모 |
| `createdBy` | String? | — | null | 생성자 이메일 |
| `killedAt` | DateTime? | — | null | Kill 실행 일시 |
| `killedBy` | String? | — | null | Kill 실행자 |
| `killReason` | String? | — | null | Kill 사유 |
| `customPageId` | ObjectId | ✅ | — | FK → CustomPage |
| `deploymentId` | ObjectId | ✅ | — | FK → Deployment |
| `createdAt` | DateTime | auto | `now()` | 생성일시 |
| `updatedAt` | DateTime | auto | `@updatedAt` | 수정일시 |

**Relations**: `customPage` (CustomPage), `deployment` (Deployment), `accessLogs[]` (PreviewAccessLog)
**Collection**: `preview_sessions`
**Unique Index**: `preview_key`

### 1.10 PreviewAccessLog

프리뷰 접근 시도 기록. 인가/비인가 접근을 모두 추적.

| Field | Type | Required | Default | 설명 |
|-------|------|----------|---------|------|
| `id` | ObjectId | auto | `@default(auto())` | PK |
| `accessorEmail` | String? | — | null | 접근자 이메일 |
| `accessorIp` | String? | — | null | 접근자 IP |
| `accessType` | String | ✅ | — | `AUTHORIZED` / `UNAUTHORIZED` |
| `previewSessionId` | ObjectId | ✅ | — | FK → PreviewSession |
| `createdAt` | DateTime | auto | `now()` | 접근 시각 |

**Relations**: `previewSession` (PreviewSession)
**Collection**: `preview_access_logs`

---

### 1.11 PortalUser

포털 사용자 역할 관리. Google 로그인 시 자동 등록(MEMBER), Super Admin이 역할 승격.

| Field | Type | Required | Default | 설명 |
|-------|------|----------|---------|------|
| `id` | ObjectId | auto | `@default(auto())` | PK |
| `email` | String | ✅ unique | — | 사용자 이메일 |
| `name` | String? | — | null | 표시 이름 |
| `role` | String | ✅ | `MEMBER` | `SUPER_ADMIN` / `MEMBER` |
| `isActive` | Boolean | ✅ | `true` | 활성화 여부 |
| `lastLoginAt` | DateTime? | — | null | 마지막 로그인 |
| `createdAt` | DateTime | auto | `now()` | 생성일시 |
| `updatedAt` | DateTime | auto | `@updatedAt` | 수정일시 |
| `createdBy` | String? | — | null | 역할 변경/초대한 admin |

**Collection**: `portal_users`
**Unique Index**: `email`
**저장 위치**: masterDbClient (REAL DB) — 환경 독립적 글로벌 모델

### 1.12 RoleChangeLog

역할 변경 이력 추적. 누가 언제 어떤 역할로 변경했는지 기록.

| Field | Type | Required | Default | 설명 |
|-------|------|----------|---------|------|
| `id` | ObjectId | auto | `@default(auto())` | PK |
| `targetEmail` | String | ✅ | — | 변경 대상 이메일 |
| `previousRole` | String | ✅ | — | 이전 역할 |
| `newRole` | String | ✅ | — | 새 역할 |
| `changedBy` | String | ✅ | — | 변경 수행자 |
| `reason` | String? | — | null | 변경 사유 |
| `createdAt` | DateTime | auto | `now()` | 변경일시 |

**Collection**: `role_change_logs`
**저장 위치**: masterDbClient (REAL DB)

---

### 개발자 체크리스트 (스키마 변경 시 필수)

스키마를 변경하는 모든 개발 작업은 아래를 확인합니다:

1. [ ] `docs/migrations.md`에 MIG-XXX 항목을 등록했는가?
2. [ ] Phase 1 Fallback 로직을 작성했는가?
3. [ ] Phase 2 마이그레이션 API를 멱등성 보장으로 작성했는가?
4. [ ] Phase 2 verify API를 함께 작성했는가?
5. [ ] Phase 3 체크리스트를 verify API의 JSDoc 주석에 기록했는가?
6. [ ] `MigrationAlertBanner`가 신규 마이그레이션도 감지하도록 조정했는가?

