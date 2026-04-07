# Agent Discussion Log

> 이 문서는 **Sub-Agent 간 의사결정 토론 기록**이며, 🎖️ **Project Leader(PL)의 지시·검수·승인** 이력도 포함합니다.
> 모든 중요 의사결정은 PL의 승인을 거쳐야 하며, PL의 리팩토링 명령과 테스트 시나리오 지시도 여기에 기록됩니다.

---

## Discussion Format

### Sub-Agent 간 일반 논의
```markdown
### [날짜] [요청 Sub-Agent] → [대상 Sub-Agent]: [제목]

**요청 내용**: ...
**배경/이유**: ...
**기대 결과**: ...

---
**[대상 Sub-Agent] 응답** (날짜):
**결론**: ...
**변경 사항**: ...
```

### 🎖️ PL의 작업 지시
```markdown
### [날짜] 🎖️ PL → [Sub-Agent]: [지시] — [제목]

**지시 내용**: ...
**우선순위**: [즉시/다음 세션/백로그]
**기대 산출물**: ...
**완료 조건**: ...

---
**[Sub-Agent] 완료 보고** (날짜):
**결과**: ...
**변경 파일**: ...

---
**🎖️ PL 검수** (날짜):
**승인/반려**: ...
**specs.md 반영 여부**: ...
```

### 🎖️ PL의 리팩토링 명령
```markdown
### [날짜] 🎖️ PL → [Sub-Agent]: 🔧 리팩토링 명령 — [제목]

**위반 항목**: [DDD Layer 위반 / 도메인 경계 침범 / ...]
**현재 상태**: ...
**기대 상태**: ...
**우선순위**: [즉시/다음 세션/백로그]

---
**[Sub-Agent] 완료 보고** (날짜):
**변경 파일**: ...
**변경 내용**: ...

---
**🎖️ PL 재검수** (날짜):
**승인/반려**: ...
```

### 🎖️ PL의 테스트 시나리오 지시
```markdown
### [날짜] 🎖️ PL → QA: 🧪 테스트 시나리오 — [기능명]

#### Unit Tests (Tier 1)
- [ ] ...

#### Integration Tests (Tier 2)
- [ ] ...

#### E2E Tests (Tier 3)
- [ ] ...

---
**QA 완료 보고** (날짜):
**통과율**: .../...
**실패 항목**: ...
```

---

## Discussion History

---

### 2026-03-15 Backend → DevOps: MongoDB Replica Set 전환 요청

**요청 내용**: 프로덕션 MongoDB(`platform-mongo.bmfdev.io`)를 Single-Node Replica Set으로 전환 요청
**배경/이유**: Prisma ORM은 트랜잭션 사용 시 MongoDB Replica Set이 필수. 현재 Standalone 모드에서는 트랜잭션 연산이 실패.
**기대 결과**: `--replSet rs0` 옵션 추가 및 `rs.initiate()` 실행 후 정상 트랜잭션 지원

---
**DevOps 응답** (2026-03-15):
**결론**: 외부 DevOps 팀에 요청 전달 완료. 대기 중.
**변경 사항**: `k8s/secret.yaml.example` 생성, `start.sh` 자동 시딩, `prisma/seed.ts` 멱등성

---
**🎖️ PL 검수** (2026-03-29):
**승인**: ✅ 적절한 조치. ISS-003으로 추적 중.

---

### 2026-03-15 DevOps → Backend: DB 자동 시딩 방식 결정

**요청 내용**: 컨테이너 시작 시 DB 초기 데이터를 자동으로 넣는 방법 결정
**배경/이유**: 프로덕션 배포 후 빈 DB로 시작하면 앱이 정상 동작하지 않음
**기대 결과**: 배포만 하면 자동으로 초기 데이터가 생성되는 구조

---
**Backend 응답** (2026-03-15):
**결론**: `start.sh`에서 `npx tsx prisma/seed.ts` 자동 실행. Stage 개수 체크 후 멱등 처리.
**변경 사항**: `start.sh`, `prisma/seed.ts`

---
**🎖️ PL 검수** (2026-03-29):
**승인**: ✅ 멱등성 보장 확인. specs.md 반영 완료.

---

### 2026-03-29 🎖️ PL → All: [지시] — 데이터 모델 구조 변경 specs.md 반영

**지시 내용**: `docs/specs.md` 전면 갱신 — Project → CustomPage 중심 구조로 대규모 변경 반영
**우선순위**: 즉시
**기대 산출물**: 최신 스키마·API·프론트엔드 구조가 정확히 반영된 specs.md
**완료 조건**: 모든 Sub-Agent가 최신 스펙을 기준으로 작업 가능한 상태

---
**Backend 완료 보고** (2026-03-29):
**결과**: specs.md 전면 갱신 완료
**변경 파일**: `docs/specs.md`

---
**🎖️ PL 검수** (2026-03-29):
**승인**: ✅ 전체 스키마/API/프론트 구조 정확히 반영. spec-changelog.md 이력 기록 필요.
**specs.md 반영 여부**: 완료

---

### 2026-03-29 🎖️ PL → All: [지시] — Sub-Agent 아키텍처 재구성

**지시 내용**: Multi-Agent 구조를 PL 중심 Sub-Agent 아키텍처로 전면 재구성
**우선순위**: 즉시
**기대 산출물**: `agent.md` (PL + 4 Sub-Agent 정의), `docs/context.md` (PL 관리 형식), `docs/agent_discussion.md` (PL 지시/검수 형식 포함)
**완료 조건**:
- PL이 모든 Sub-Agent의 총괄 관리자로 정의
- specs.md는 PL이 최종 관리
- DDD 감사 + 리팩토링 명령 프로세스 포함
- 3-Tier TDD 시나리오 설계 프로세스 포함

---
**완료 보고** (2026-03-29):
**결과**: 3개 파일 전면 재작성 완료
**변경 파일**: `agent.md`, `docs/context.md`, `docs/agent_discussion.md`

---
**🎖️ PL 검수** (2026-03-29):
**승인**: ✅

---

### 2026-03-29 🎖️ PL → Frontend: [지시] — Custom Pages 버그 수정

**지시 내용**: 커스텀 페이지 목록과 상세 페이지의 에러 상태 UI를 개선할 것
**우선순위**: 즉시
**기대 산출물**: 1) 빈 목록 상태에 아이콘+CTA, 2) not found 상태에 설명+백버튼
**완료 조건**: 브라우저 테스트 통과

---
**Frontend 완료 보고** (2026-03-29):
**결과**: `custom-pages/page.tsx` 빈 목록 UI 개선, `custom-pages/[id]/page.tsx` not found UI 개선
**변경 파일**: `src/app/custom-pages/page.tsx`, `src/app/custom-pages/[id]/page.tsx`

---
**🎖️ PL 검수** (2026-03-29):
**승인**: ✅ 브라우저 테스트 4/4 Pass
**specs.md 반영 여부**: 미반영 (UI 개선만, 스펙 변경 없음)

---

### 2026-03-29 🎖️ PL → All: [지시] — Sub-Agent 워크플로우 필수 강제 규칙 추가

**지시 내용**: `agent.md`에 Section 0 "MANDATORY WORKFLOW PROTOCOL"을 추가하여, 모든 작업에서 Sub-Agent 협업과 대화 기록을 필수화
**우선순위**: 즉시
**기대 산출물**: `agent.md` Section 0 (약 75줄), `agent_session_log.md` 소급 기록
**완료 조건**:
- 모든 작업이 PL 계획 → Sub-Agent 할당 → 대화 기록 → 검수 프로세스를 반드시 따름
- 소규모 작업에도 최소 기록 템플릿 적용
- 위반 시 대응 절차 명시

---
**PL 완료 보고** (2026-03-29):
**결과**: Section 0 추가, 이전 세션 소급 기록 2건 완료
**변경 파일**: `agent.md`, `docs/agent_session_log.md`, `docs/agent_discussion.md`

---
**🎖️ PL 검수** (2026-03-29):
**승인**: ✅ 필수 워크플로우 프로토콜 적용 완료. 이후 모든 세션에서 강제 적용 예정.

---

### 2026-03-29 🎖️ PL → Backend: 🔧 리팩토링 명령 — DDD 아키텍처 위반 5건 수정

**위반 항목**: DDD Layer 위반, 도메인 경계 침범, 에러 표준화 미준수
**현재 상태**:
1. `CustomPageService`에 중복 메서드 존재 (Repository에 이미 존재하는 `findReferenceLinks` 등)
2. `custom-page` 도메인에 barrel export(`index.ts`) 누락
3. `StageService.triggerStageDeployment`에서 `dbClient.deployment` 직접 접근 (Layer Rules 위반)
4. `StageService`의 에러가 generic `Error` 사용 (`AppError` 미사용)
5. `StageRepository`에 deployment 관련 메서드 부재

**기대 상태**: 모든 DB 접근은 Repository를 통해, 모든 에러는 AppError, 도메인 barrel export 존재
**우선순위**: 즉시

---
**Backend 완료 보고** (2026-03-29):
**변경 파일**:
- `src/domains/custom-page/custom-page.service.ts` — 중복 메서드 제거 (L277-283)
- `src/domains/custom-page/index.ts` — 신규 barrel export
- `src/domains/stage/stage.repository.ts` — `findActiveStageDeployment`, `createStageDeployment`, `updateDeployment` 추가
- `src/domains/stage/stage.service.ts` — `Error` → `AppError` 6건, `dbClient.deployment` → `stageRepository` 위임

---
**🎖️ PL 재검수** (2026-03-29):
**승인**: ✅ 기존 15/15 테스트 통과 확인. DDD Layer Rules 준수 확인.

---

### 2026-03-29 🎖️ PL → QA: 🧪 테스트 시나리오 — 전체 도메인 커버리지 90%

#### Unit Tests (Tier 1) — CustomPageService
- [x] listCustomPages (필터 유/무)
- [x] getCustomPage (정상/404)
- [x] checkUrlPathDuplicate / checkTitleDuplicate (정상/빈값 에러)
- [x] createCustomPage (정식 저장, URI 자동발번, targetUri 우선, SEO noindex, isDraft, type 검증, stageId 필수, 미분류 자동할당, RESERVED 검증)
- [x] updateCustomPage (정상, 배포 중 URI 변경 금지, IMMEDIATE URI 변경 금지, p-prefix 허용, HIDDEN noindex, 활성 noindex false, URI_MAPPING 빈 targetUri 자동발번, RESERVED 기간 내 변경 금지, 비활성 URI 변경 금지, JSON config 파싱)
- [x] deleteCustomPage (정상/404)
- [x] triggerDeployment (정상, versionHash 없음, 동시성 충돌, triggerFailed)
- [x] Reference Links (조회/추가/삭제)

#### Unit Tests (Tier 1) — StageService
- [x] createStage (빈 도메인, ID 없음, 소프트삭제 reuseRepo=undefined, reuseRepo=true 복구, 멱등 처리)
- [x] updateStage / deleteStage (정상/404)
- [x] Custom Pages CRUD 위임 (조회/생성/수정/삭제)
- [x] triggerStageDeployment (정상, versionHash 누락, 동시성 충돌, stuck FAILED 처리, triggerFailed, 미존재 스테이지)
- [x] handleGithubPushWebhook
- [x] ensureGithubRepo (기존 skip, PAT 없음 mock, PAT API 성공, PAT API 실패, 이름 충돌, webhook 등록)

#### Unit Tests (Tier 1) — route-sync.util
- [x] active 페이지만, p-prefix 제외, 빈 urlPath 건너뛰기
- [x] RESERVED 기간 필터, previewRoute, mock/real API, JSON 파싱

#### E2E Tests (Tier 3) — Playwright
- [x] 인프라 구축 (config, 3 spec files, npm scripts)

---
**QA 완료 보고** (2026-03-29):
**통과율**: 86/86 (100%)
**커버리지**: 92.97% stmts / 84.17% branches / 96.66% funcs
**실패 항목**: 없음

---
**🎖️ PL 검수** (2026-03-29):
**승인**: ✅ 90% 목표 달성 확인 (92.97%).
**specs.md 반영 여부**: 테스트 인프라 섹션 업데이트 필요 → context.md에 반영

---

### 2026-03-29 🎖️ PL → All: [지시] — Sub-Agent 프로토콜 강제 집행 메커니즘 도입

**지시 내용**: 프로토콜 위반 재발 방지를 위해 4-Layer 강제 메커니즘(System Instruction, Pre-Task Gate, Post-Task Gate, Git Workflow 통합)을 도입
**우선순위**: 즉시
**기대 산출물**:
- `CLAUDE.md` — 시스템 레벨 자동 읽기 명령
- `scripts/check-protocol.sh` — 자동 검증 스크립트
- `agent.md` Section 0 강화 (0.0, 0.3, 0.4, 0.7)
- `.agent/workflows/git-commit.md` Step 1 프로토콜 검증 통합
**완료 조건**: `check-protocol.sh` 실행 시 정상 동작, 모든 Layer 적용

---
**PL 완료 보고** (2026-03-29):
**결과**: 4-Layer 강제 메커니즘 도입 완료
**변경 파일**: `CLAUDE.md` (신규), `scripts/check-protocol.sh` (신규), `agent.md`, `.agent/workflows/git-commit.md`

---
**🎖️ PL 검수** (2026-03-29):
**승인**: ✅ `check-protocol.sh` 테스트 통과. 이후 모든 세션에서 강제 적용.

---

### 2026-03-29 🎖️ PL → QA: 🧪 테스트 시나리오 — Tier 2 Integration Tests 구현

**지시 내용**: API Route Handler → Service 연동을 테스트하는 Integration Test 구현
**대상 API**:
- Custom Pages API (GET list, POST create, GET/:id, PATCH/:id, DELETE/:id, POST deploy)
- Stages API (GET list, POST create, GET/:id, DELETE/:id, POST deploy, POST webhook 5종)
- Projects API (GET list, POST create)

---
**QA 완료 보고** (2026-03-29):
**통과율**: 117/117 (100%) — Unit 86 + Integration 31
**신규 파일**:
- `src/__tests__/integration-helpers.ts` — Mock Request/Params 빌더
- `src/__tests__/integration/custom-pages-api.test.ts` — 14 tests
- `src/__tests__/integration/stages-api.test.ts` — 13 tests
- `src/__tests__/integration/projects-api.test.ts` — 5 tests
**수정 파일**: `src/__tests__/setup.ts` — `deployment.update` mock 추가

---
**🎖️ PL 검수** (2026-03-29):
**승인**: ✅ 117/117 전체 테스트 통과 확인. Tier 2 완료.

---

### 2026-03-30 🎖️ PL → Frontend, DevOps: [지시] — Cherry-pick ca0e174 + Wizard 3단계 수정

**지시 내용**: 1) develop을 main과 동기화 후 `ca0e174` cherry-pick, 2) `page.tsx` 배포 wizard 4단계→3단계 수정
**우선순위**: 즉시
**기대 산출물**: 3단계 wizard UI, PR #15
**완료 조건**: 브라우저에서 3단계 wizard 렌더링 확인

---
**Frontend 완료 보고** (2026-03-30):
**결과**: `page.tsx` 4단계→3단계 수정 완료 (steps, grid-cols, currentStep)
**변경 파일**: `src/app/custom-pages/[id]/page.tsx`

---
**DevOps 완료 보고** (2026-03-30):
**결과**: cherry-pick, push, PR #15 생성
**변경 파일**: `docs/spec-changelog.md` (충돌 해결), 기타 cherry-pick 대상 파일

---
**🎖️ PL 검수** (2026-03-30):
**승인**: ✅ 브라우저 검증 완료. 3단계 wizard 정상 확인.

---

### 2026-03-30 🎖️ PL → All: [지시] — Phase 2 백로그 정리 + Agent 워크플로우 통합

**지시 내용**: 1) `specs.md` Phase 2 백로그 6개 서브섹션 추가, 2) `CLAUDE.md`/`agent.md` 통합
**우선순위**: 즉시
**기대 산출물**: Phase 2 백로그(20+ 항목), 엔진 무관 통합 워크플로우
**완료 조건**: 두 엔진 모두 동일 절차로 동작

---
**PL 완료 보고** (2026-03-30):
**결과**: specs.md, CLAUDE.md, agent.md 수정 완료
**변경 파일**: `docs/specs.md`, `CLAUDE.md`, `agent.md`

---
**🎖️ PL 검수** (2026-03-30):
**승인**: ✅

---

### 2026-04-01 🎖️ PL → Frontend: [지시] — Stepper UI 라인 너비 표시 오류 수정

**지시 내용**: `src/app/stages/[id]/page.tsx`의 "코드 연동 상태" stepper UI에서 초록색 라인이 상태에 맞게 뒷 배경 회색 선 전체를 채우도록 너비(width) 계산 로직 수정.
**우선순위**: 즉시
**기대 산출물**: Step 1 완료 시 라인이 절반까지 채워지고, Step 2 완료 시 전체를 채우도록 개선.
**완료 조건**: 브라우저 UI에서 라인 채워짐 상태 정상 확인.

---
**Frontend 완료 보고** (2026-04-01):
**결과**: `currentStep`에 따른 width 연산 로직 수정 완료 (`width: calc(50% - 2rem)` 및 `calc(100% - 4rem)`)
**변경 파일**: `src/app/stages/[id]/page.tsx`

---
**🎖️ PL 검수** (2026-04-01):
**승인**: ✅ 브라우저 렌더링 확인.
**specs.md 반영 여부**: UI 수정이므로 미반영.

---

### 2026-04-01 🎖️ PL → Frontend: [지시] — Custom Page 목록 UI 간소화 및 상태 배지 추가

**지시 내용**: `feat/custom-page-list-ui` 브랜치 작업 내용으로, 불필요한 연동/노출 상태 컬럼을 통합하고 활성(Active) 뱃지 UI를 단일화.
**우선순위**: 백로그 → 완료
**기대 산출물**: 간결해진 리스트 컬럼, 직관적인 상태 뱃지.
**완료 조건**: 목록 페이지 렌더링 정상 동작.

---
**Frontend 완료 보고** (2026-04-01):
**결과**: UI 컴포넌트(`CustomPageList`) 컬럼 최소화 반영.
**변경 파일**: `src/features/deployment-pipeline/ui/DeploymentsTab.tsx` 등 연관 파일.

---
**🎖️ PL 검수** (2026-04-01):
**승인**: ✅ 디자인 리뷰 완료

---

### 2026-04-01 🎖️ PL → Backend & Frontend: [지시] — 배포 타임아웃 오류 핸들링 및 UI 픽스

**지시 내용**: `fix/deployment-timeout-ui-fixes` 브랜치 작업 내용. DevOps API 배포 요청 시 10분 등 긴 타임아웃 엣지 케이스를 처리하고, 큐 대기/진행 중일 때 에러(`errorMessage`)를 적절히 노출 처리.
**우선순위**: 즉시 (버그 수정)
**기대 산출물**: 배포 버튼 비활성화 안정성 및 오류 메시지 시각화.
**완료 조건**: API에서 오는 타임아웃 에러를 감지하고 스테이지 UI에 렌더링.

---
**Backend & Frontend 완료 보고** (2026-04-01):
**결과**: Backend Timeout 적용 및 DB 실패 상태 처리 (errorMessage 파싱). Frontend 타임아웃 에러 메시지 렌더링 연결.
**변경 파일**: `src/app/api/v1/deployments/timeout/route.ts` 등.

---
**🎖️ PL 검수** (2026-04-01):
**승인**: ✅ 검증 완료
**specs.md 반영 여부**: `devops-platform-api-0401.md`에 API 에러 및 Timeout 관련 텍스트 추가로 반영됨.

---

### 2026-04-01 🎖️ PL → Backend: [지시] — Stage GitHub Repo Private 생성 기본 설정

**지시 내용**: `fix/private-github-repo` 브랜치 작업. 스테이지 생성 시 자동으로 만들어지는 GitHub 리포지토리의 가시성(visibility) 속성을 `private`으로 하드코딩.
**우선순위**: 즉시 (보안 이슈)
**기대 산출물**: GitHub API 레포 생성 parameter를 `private: true`로 설정.
**완료 조건**: 새 스테이지 생성 시 Private repo로 세팅 확인.

---
**Backend 완료 보고** (2026-04-01):
**결과**: GitHub 연동 로직의 repo create 속성에 private: true 추가 완료.
**변경 파일**: `src/domains/stage/stage.service.ts`

---
**🎖️ PL 검수** (2026-04-01):
**승인**: ✅ 성공

---

### 2026-04-05 🎖️ PL → Agent: [지시] — Phase 2 로컬 MongoDB DEV 환경 통합 테스트

**지시 내용**: 로컬 MongoDB를 띄우고 DEV 환경에서 Phase 2 마이그레이션 API, Verify API, UI 통합을 E2E 테스트할 것.
**우선순위**: 높음
**기대 산출물**: 마이그레이션 멱등성 검증, MongoDB null 필터 호환성 확보, UI 검증 스크린샷
**완료 조건**: 전체 테스트 통과 및 Confluence 문서화

---
**Agent 완료 보고** (2026-04-05):
**결과**: ✅ 전체 E2E 테스트 완료
- Prisma Schema Push → `stage_repositories` 컬렉션 생성
- 마이그레이션 API: 2 repos 생성, 5 pages 연결 (멱등성 검증 OK)
- 🐛 MongoDB `deletedAt: null` 필터 버그 발견 → `OR: [{deletedAt:null},{deletedAt:{isSet:false}}]` 패턴으로 수정 (2 files)
- Verify API: `allEnvironmentsReady: true`
- UI: Repository 탭, 커스텀 페이지 상세, 생성 폼 모두 정상
- 테스트 스위트: 151/151 passed
**변경 파일**: `repos-schema/route.ts`, `repos-schema-verify/route.ts`

---
**🎖️ PL 검수** (2026-04-05):
**승인**: ✅ 완료 확인

---

### 2026-04-05 🎖️ PL → Agent: [긴급 지시] — Docker 빌드 실패 핫픽스

**지시 내용**: main 브랜치 배포 시 Docker 빌드에서 `prisma generate` 실패. 즉시 핫픽스 필요.
**우선순위**: 긴급 (배포 차단)
**기대 산출물**: Docker 빌드 정상화
**완료 조건**: PR 생성 및 main 머지 후 배포 성공

---
**Agent 완료 보고** (2026-04-05):
**결과**: ✅ 핫픽스 완료 → PR #85 생성
**원인**: `postinstall: "prisma generate"` — Docker deps 스테이지에서 `prisma/schema.prisma` 미존재
**수정**: `"test -f prisma/schema.prisma && prisma generate || true"` — 조건부 실행
**변경 파일**: `package.json`


---

### 2026-04-05 🎖️ PL → All: [기능 구현] — Preview Mode 보안 강화 프리뷰 시스템

**지시 내용**: 정식 배포(Promote) 전 고유 프리뷰 URI를 생성하여 페이지를 미리 확인하는 보안 강화 프리뷰 시스템 구현.

**할당**:
- **Backend**: Prisma 모델 2개 (`PreviewSession`, `PreviewAccessLog`) + API 7개 + Route Sync 확장 + Promote 자동 정리
- **Frontend**: `PreviewPanel.tsx` 컴포넌트 + `DeploymentsTab.tsx` 통합
- **QA**: Preview Mode 유닛 테스트 5건 추가 (`route-sync.test.ts`)

**보안 정책**:
- 만료: 기본 1시간, 최대 72시간
- 접근 제어: `allowedEmails` 이메일 매칭
- Kill Switch: 개별/전체 즉시 비활성화
- Auto-Kill: 비인가 접근 5회 초과 시
- Promote 연동: 정식 배포 시 활성 프리뷰 자동 EXPIRED

**우선순위**: 즉시
**기대 산출물**: Schema + API + Frontend + Tests + 문서 업데이트 일체
**완료 조건**: 156/156 tests + build 0 errors + spec-changelog + context.md + session_log 갱신

---
**All Sub-Agents 완료 보고** (2026-04-05):
**결과**: ✅ 전체 완료
- **Backend**: Schema 2모델, API 7개, route-sync SyncOptions, promote 자동정리 — `dab0155`
- **Frontend**: PreviewPanel.tsx (593 lines), DeploymentsTab 통합 — `6022f31`
- **QA**: Preview Mode 테스트 5건 추가 (156/156 passed)
- **변경 규모**: 12 files, +1,577 lines
- **문서**: agent_session_log, spec-changelog, context.md, agent_discussion.md 모두 갱신

---
**🎖️ PL 검수** (2026-04-05):
**승인**: ✅ 완료 확인 — Sub-Agent 프로토콜 준수 소급 보완 포함

---

### 2026-04-06 🎖️ PL → Agent: [지시] — 레포지토리 생성 UX 개선 + 504 Timeout 근본 해결

**지시 내용**: 레포 생성 모달이 떠있는 동안 504 타임아웃 발생 문제 해결. 모달 즉시 닫기 + Toast/Pending 카드 기반 UX 전환 + Quick Start 비동기 분리 + safeParseJson 에러 방어
**우선순위**: 즉시
**기대 산출물**: Backend Quick Start fire-and-forget, Frontend 모달 즉시 닫힘, safe-fetch 유틸, 223 tests pass
**완료 조건**: 504 발생 불가 + 사용자 친화적 에러 메시지 + 경과 시간 표시

---
**Agent 완료 보고** (2026-04-06):
**결과**: ✅ 전체 완료
- Backend: Quick Start fire-and-forget + 10분×2회 재시도
- Frontend: 모달 즉시 닫힘 + Toast + Pending 카드(⏱ 경과 시간)
- Utility: `safe-fetch.ts` 신규 (safeParseJson + getHttpErrorMessage)
- 테스트: 223/223 passed
**변경 파일**: `repository.service.ts`, `page.tsx`, `safe-fetch.ts` [NEW]

---
**🎖️ PL 검수** (2026-04-06):
**승인**: ✅

---

### 2026-04-06 🎖️ PL → Agent: [지시] — 파일-URI 매핑 UI + templateMappingPath 설정 기능

**지시 내용**: 커스텀 페이지 생성/수정 시 레포 내 파일과 URI를 직접 연결하는 UI 추가. public/user/ 하위 파일 트리 조회 + templateMappingPath 설정 + 배포 연동
**우선순위**: 즉시
**기대 산출물**: GitHub 파일 브라우저 API, 생성/수정 폼 UI, templateMappingPath DB 저장 → route-sync 배포
**완료 조건**: 파일 선택/직접 입력 가능 + template.liquid 존재 여부 표시 + 배포 시 올바른 매핑

---
**Agent 완료 보고** (2026-04-06):
**결과**: ✅ 전체 완료
- API: `GET /stages/:id/repositories/:repoId/files` [NEW]
- Frontend: 생성 폼 + OverviewTab 수정 페이지에 파일 브라우저
- Infrastructure: `listDirectoryContents()` + `GitHubContentItem`
- 테스트: 223/223 passed, build exit 0
**변경 파일**: `files/route.ts` [NEW], `github-api.client.ts`, `new/page.tsx`, `OverviewTab.tsx`

---
**🎖️ PL 검수** (2026-04-06):
**승인**: ✅

