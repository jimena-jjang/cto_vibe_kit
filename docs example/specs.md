# Service Specifications

> 이 문서는 inhouse dev portal의 **서비스 스펙 명세서**입니다.
> 다른 에이전트가 이 문서만 보고도 **동일한 시스템을 재구현**할 수 있는 수준의 상세 내용을 포함합니다.
> 변경 시 반드시 `spec-changelog.md`에 이력을 남기세요.

**Last Updated**: 2026-04-06 (v8: Multi-Org orgType — DevOps Platform API 업로드 시 internal/external 자동 결정)

---


> [!IMPORTANT]
> **Sub-Agent 지시사항 (문서 참조 가이드)**
> 이 문서는 inhouse-dev-portal 전체 아키텍처의 인덱스입니다. 구체적인 개발 스펙이 필요하면 아래 하위 문서를 반드시 선 참조하세요:
> - **[데이터 모델 (Prisma/MongoDB) 추가/수정 시 📚](./specs_database.md)**
> - **[API Endpoint 또는 Business Logic 개발/검증 시 ⚙️](./specs_api.md)**
> - **[테스트 환경 설정, 커버리지, E2E 스펙 확인 시 🧪](./specs_test.md)**
> - **[DDD 아키텍처, 계층 구조, 의존성 규칙 확인 시 🏛️](./architecture.md)**
 
---

## 3. DDD 아키텍처 (v7)

> [!IMPORTANT]
> v7 리팩토링으로 모든 비즈니스 로직이 Route Handler에서 Domain Service로 이동했습니다.
> Route Handler는 **얇은 Controller**(~30~50줄)로만 유지됩니다.

### 도메인-Route 매핑

| API Route | 담당 Domain Service | Infrastructure 의존 |
|-----------|--------------------|--------------------|  
| `GET/POST /stages/:id/repositories` | `RepositoryService` | `GitHubApiClient` |
| `POST /stages/:id/deploy` | `DeploymentService.triggerStageDeploy()` | `GitHubApiClient` |
| `POST /stages/:id/deploy/callback` | `DeploymentService.handleCallback()` | — |
| `POST /stages/:id/deploy-all` | `DeploymentService.deployAll()` | — |
| `POST /custom-pages/:id/deploy` | `CustomPageService.triggerDeployment()` | `BstageGatewayClient` |
| `POST .../promote` | `CustomPageService` + `route-sync` | `BstageGatewayClient` |
| `GET/POST /admin/users` | `PortalUserService` | `RBAC Guard` |

### 계층 구조

```
Presentation (Route Handlers) → Domain Services → Domain Repositories → Prisma/MongoDB
                               ↘ Infrastructure Clients (GitHub, b.stage GW, RBAC Guard)
```

**의존성 규칙**: Route → Domain ✅ | Domain → Infrastructure ✅ | Route → Infrastructure ❌

→ 상세: [`architecture.md`](./architecture.md)

---

## 4. Frontend Pages

### 4.1 홈 (`/`)
- 대시보드 또는 리다이렉트

### 4.2 커스텀 페이지 생성 (`/custom-pages/new`)
- 단일 폼 (위저드가 아닌 단일 페이지)
- 필드: title, type(select), stageId(select), ownerEmail, description, integrationMode, urlPath
- isDraft 지원 (임시저장 버튼)
- 생성 후 `/custom-pages/:id`로 리다이렉트

### 4.3 커스텀 페이지 상세 (`/custom-pages/:id`)
- **탭 기반 구조** — 아래 Feature 모듈에서 각 탭 컴포넌트 import
- **Overview 탭** (`custom-page-catalog`): 페이지 기본 정보, 프로젝트 요약 헤더
- **Deployments 탭** (`deployment-pipeline`): 배포 이력, 2단계 배포 wizard (템플릿 업로드 → 화면 라우팅 완료), **Preview Mode 관리 패널**
  - **Preview Mode**: 배포 전 프리뷰 URI 생성/관리 (만료 시간 선택, 이메일 접근 제한, Kill Switch, 접근 로그)
- **Resources 탭** (`resource-references`): GITHUB_REPO, CONFLUENCE, JIRA 등 9가지 타입 참조 링크
- **Routing 탭** (`traffic-routing`): Release Wizard, 라우팅 관리
- **Access 탭** (`audit-acl`): 접근 제어 및 감사 로그

### 4.4 스테이지 목록 (`/stages`)
- 카드 기반 형식
- 통합 노드 그룹(Project) 필터링 및 이름 검색 지원
- 각 스테이지: domain, name, 프로젝트 그룹(Node Group) 수, 커스텀 페이지 수 표기

### 4.5 스테이지 상세 (`/stages/:id`)
- **Stage Basic Info**: 스테이지 기본 메타데이터 및 도메인 하이퍼링크 UI 구성
- **GitHub Info → Repositories**: (**v4 아키텍처 변경**) 기존 단일 레포지토리 표시에서 **다중 레포지토리 목록 관리**로 전환
  - 연결된 StageRepository 목록 (타입별: LIQUID / SDK_FULL / SDK_INPAGE)
  - 레거시(Stage 1:1) 데이터도 Fallback으로 표시 (마이그레이션 완료 전까지)
  - 각 레포지토리별 코드 동기화 상태, Webhook Secret, 마지막 Push 시점 표시
  - **(v6)** **Source 모드 선택**: 레포 생성 시 사용자가 **내부(🏢 devops-bmf)** 또는 **외부(🤝 partners-bmf)** 선택 (기본값: EXTERNAL)
  - **(v6)** **Quick Start (DEV/QA 전용)**: 레포 생성 후 Hello World 템플릿 코드 자동 커밋 + 배포 트리거 (REAL 환경 차단)
  - **(v6)** 레포 목록에 source 뱃지 표시 (🏢내부 / 🤝외부)
- **Custom Build Manager**: 커스텀 빌드 배포 관리 (Phase 1 빌드 트리거, 일괄 배포 등)
- **Custom Pages**: 테이블 레이아웃 기반 커스텀 페이지 목록 관리 및 그룹 정렬

### 4.7 Not Found (`/not-found`)
- 404 에러 페이지

---

## 5. Infrastructure

### 5.1 Database
- **Provider**: MongoDB (Replica Set 필수 — Prisma 트랜잭션 지원)
- **Connection**: `DATABASE_URL` 환경 변수
- **로컬**: `mongodb://localhost:27017/inhouse_dev_portal_dev`
- **프로덕션**: `platform-mongo.bmfdev.io` (Single-Node Replica Set)

### 5.2 Prisma Client
- `src/lib/db-factory.ts`에서 환경별 DB 클라이언트 관리
- `getDatabaseClient(env)` 함수로 `DEV` / `QA` / `REAL` 환경 분기

### 5.3 Docker
- `Dockerfile`: Node.js base, `prisma generate` 포함 (비루트 유저 `nextjs`로 실행)
- `docker-compose.yml`: app + MongoDB 컨테이너
- `start.sh`: 컨테이너 시작 스크립트 (`prisma db push` → DB 자동 시딩 → `node server.js`)

### 5.4 Kubernetes
```
k8s/
├── configmap.yaml       # 환경 변수 (non-secret)
├── deployment.yaml      # Deployment spec
├── service.yaml         # Service (ClusterIP/NodePort)
└── secret.yaml.example  # Secret 예시 (git 추적 안 함)
```
- **Secret**: `dev-portal-secrets` (MONGODB_USERNAME, MONGODB_PASSWORD 등)

### 5.5 Seed Data (`prisma/seed.ts`)
- 3개 Stage: `fanmaum.bstage.in`, `test.bstage.in`, `demo.bstage.in`
- 샘플 Project(노드 그룹), Deployment, CustomPage, ReferenceLink 생성
- **멱등성 보장**: 기존 Stage 데이터가 있으면 시딩 스킵
- **자동 실행**: `start.sh`에서 컨테이너 시작 시 자동 실행

---

## 6. 환경 변수

### .env (로컬)
```env
NODE_ENV=development
DATABASE_URL=mongodb://localhost:27017/inhouse_dev_portal_dev
MOCK_GITHUB_API=true
MOCK_AWS_ECR=true
MOCK_K8S_API=true
GITHUB_INTERNAL_ORG=custom-apps-bmf
GITHUB_EXTERNAL_ORG=partners-bmf
ATLASSIAN_EMAIL=june.kay@bemyfriends.com
ATLASSIAN_API_TOKEN=ATATT3x...
CONFLUENCE_DOMAIN=bemyfriends.atlassian.net
CONFLUENCE_SPACE_KEY=BEMYFRIEND
PARENT_PAGE_ID=2072641574
GEMINI_API_KEY=AIzaSy...
```

### .env.production
```env
NODE_ENV=production
DATABASE_URL=mongodb://root:...@platform-mongo.bmfdev.io:27017/...
CONFLUENCE_DOMAIN=bemyfriends.atlassian.net
CONFLUENCE_SPACE_KEY=BEMYFRIEND
MONGODB_USERNAME=root
MONGODB_PASSWORD=***
MONGODB_URL=platform-mongo.bmfdev.io
MONGODB_PORT=27017
```

> ⚠️ 절대 스마트 따옴표(`"…"`) 사용 금지, 직선 따옴표(`"`)나 따옴표 없이 작성

---

## 8. Backlog

### Phase 2: Stage 중심 IA 개편 & 배포 고도화 (🔴 High Priority)

> **목표**: Stage 하위에 CustomPage들이 구성되는 직관적인 메뉴/IA 구조로 전환하고,
> 배포 시 대상 Stage의 URI별 상태 가시성을 확보하여 운영 효율을 극대화한다.

#### 2-1. Stage 중심 메뉴/IA 구조 개편

| 항목 | 설명 | 상태 |
|------|------|------|
| 메뉴 구조 변경 | 현재 `커스텀 페이지 관리` + `스테이지 관리` 병렬 → **Stage가 최상위 네비게이션**, 하위에 CustomPage 목록이 트리 형태로 구성 | 📋 계획 |
| Stage 대시보드 | Stage 진입 시 해당 Stage에 속한 모든 CustomPage의 상태/배포 현황을 한눈에 확인할 수 있는 대시보드 | 📋 계획 |
| Breadcrumb 개선 | `Stage > CustomPage > 상세` 형태의 계층적 네비게이션 | 📋 계획 |
| 사이드바 트리뷰 | Stage별 하위 CustomPage를 트리 구조로 펼쳐볼 수 있는 사이드바 | 📋 계획 |

#### 2-2. 배포 대상 가시성 & 배포 지정 구조

| 항목 | 설명 | 상태 |
|------|------|------|
| Stage 배포 현황 뷰 | 배포하려는 대상 Stage에 어떤 CustomPage가 이미 배포되어 있는지 **URI별 상태**(Active/Inactive/Pending)를 한눈에 표시 | 📋 계획 |
| 배포 대상 지정 | CustomPage를 배포할 때 **어디에(어떤 Stage)** 배포할지 명시적으로 지정하는 UI/로직 구현 | 📋 계획 |
| URI 충돌 검증 | 같은 Stage 내에서 동일 URI에 중복 배포가 되지 않도록 사전 검증 | 📋 계획 |
| 배포 이력 비교 | Stage 내 각 URI의 현재 활성 버전과 배포 대기 버전을 시각적으로 비교 | 📋 계획 |

#### 2-3. CustomPage 타입별 Git 레포지토리 전략

> ⚠️ **아직 실제 구현 내용이 확정되지 않은 상태**. 타입별로 다른 전략이 필요하다는 점은 명확.

| 타입 | 레포 구조 (예상) | 설명 |
|------|-----------------|------|
| `LIQUID` | 1 Stage = 1 Repo | Stage 단위로 단일 레포에서 Liquid 템플릿 관리. `templateMappingPath`로 각 URI에 매핑할 파일 경로를 명시적으로 지정 가능 |
| `SDK_FULL` | 1 CustomPage = 1 Repo | 각 커스텀 페이지가 독립 레포로 빌드/배포 |
| `SDK_INPAGE` | N CustomPage = 1 Repo (가능) | 여러 인페이지 컴포넌트를 하나의 레포에서 관리 가능 |

**향후 확인 필요사항**:
- 각 타입별 실제 빌드 파이프라인 구조 파악
- 레포 자동 생성 시 타입에 따른 분기 로직
- 모노레포 vs 멀티레포 결정 기준

#### 2-4. Web Component / SDK 배포 구조

| 항목 | 설명 | 상태 |
|------|------|------|
| SDK 빌드 파이프라인 | Web Component/SDK를 빌드하여 CDN(S3+CloudFront)에 배포하는 구조 설계/구현 | 📋 계획 |
| SDK 버저닝 | Semantic Versioning 기반 SDK 아티팩트 버전 관리 | 📋 계획 |
| SDK 레지스트리 | 배포된 SDK 버전 목록을 조회/관리하는 내부 레지스트리 | 📋 계획 |
| SDK 로딩 전략 | `<script>` 태그 또는 ES Module import 등 로딩 방식 정의 | 📋 계획 |

#### 2-5. Component Inject (기존 페이지 내 컴포넌트 주입)

| 항목 | 설명 | 상태 |
|------|------|------|
| Placeholder 주입 | 기존 b.stage 페이지의 특정 위치에 Custom Component를 inject하는 구조 | 📋 계획 |
| Inject Target 정의 | 어떤 페이지의 어떤 영역에 주입할지 Target Selector/Slot 정의 | 📋 계획 |
| Inject 라이프사이클 | Mount/Unmount/Update 이벤트 처리 및 호스트 페이지와의 통신 | 📋 계획 |
| Shadow DOM 격리 | 주입 컴포넌트의 스타일/스크립트 격리 전략 | 📋 계획 |

#### 2-6. CI/CD & Infrastructure

| 항목 | 설명 | 상태 |
|------|------|------|
| GitHub Actions 워크플로우 권한/환경변수 주입 | 템플릿 레포의 워크플로우가 동작하도록 Repository Secrets/Variables 동적 주입 | 📋 계획 |
| S3/CDN 배포 파이프라인 | 아티팩트 버저닝 + CloudFront 배포 | 📋 계획 |
| ~~PR Preview 환경~~ | ~~PR별 자동 프리뷰 + TTL 자동 정리~~ | ✅ 구현 완료 (Preview Mode로 대체) |
| Preview Mode | 배포 전 고유 프리뷰 URI 생성, 이메일 접근 제어, Kill Switch, 접근 로그, 비인가 감지 | ✅ 구현 완료 |
| **RBAC Super Admin** | **REAL 환경 배포 권한 제어 (PortalUser, RoleChangeLog, 관리자 UI, 초대 메일)** | **✅ 구현 완료** |

---

### Phase 3: Real-time & Security (🟡 Medium)

| Feature | 설명 |
|---------|------|
| Kill Switch (3-layer) | 긴급 라우팅 차단 (즉시/예약/롤백) |
| SSE 실시간 업데이트 | `EventSource` 기반 태스크 진행 스트리밍 |
| Monitoring & Alerting | ELK, Prometheus, Slack 알림 |

### Phase 4: External Integrations (🟡 Medium)

| Feature | 설명 |
|---------|------|
| GitHub Repository 자동 생성 | 타입별 전략에 따른 템플릿 레포 복제. 네이밍: `bstage-custom-{stage}-{name}` |
| b.stage Gateway 연동 | URI 매핑 API, Placeholder 주입, 메뉴 추가 |
| AWS ECR/S3 연동 | 컨테이너 레지스트리 + 아티팩트 스토리지 |

---

## 9. Confluence 동기화

- **워크플로우**: `/sync-specs` → `npx tsx scripts/sync-specs-to-confluence.ts`
- **Space**: `BEMYFRIEND`
- **Parent Page ID**: `2072641574`
- **Page Title**: `[Specs] b.stage Developer Portal` (ID: `2089943041`)
- 스크립트가 자동으로 기존 페이지 검색 → 있으면 PUT (업데이트), 없으면 POST (생성)

---

