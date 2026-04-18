---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[Feature] DB-001: Prisma Model (Part, InventoryLog) 스키마 작성"
labels: 'feature, database, priority:highest'
assignees: ''
---

## :dart: Summary
- 기능명: [DB-001] Prisma 스키마 모델링 및 마이그레이션 적용
- 목적: AS-Safe 시스템의 핵심이 되는 부품 정보(Part)와 입출고 이력(InventoryLog)의 데이터베이스 모델(SSOT)을 초기에 정의하고 확립한다.

## :link: References (Spec & Context)
> :bulb: AI Agent & Dev 단서: 작업 시작 전 아래 문서를 반드시 먼저 Read/Evaluate 할 것.
- SRS 문서: `SRS_v1.md#2-데이터-구조-설계`
- 데이터 모델 (Prisma Schema): `SRS_v1.md#코드-스니펫`

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] `schema.prisma` 파일 초기화 및 환경 설정 동기화
- [ ] `Part` 모델 데이터 필드 작성 (`partNo`, `currentQty`, `moq` 등 제약 조건 포함)
- [ ] `InventoryLog` 데이터 필드 및 외래키(Relation) 연결 작성
- [ ] 데이터베이스 마이그레이션 생성 (`prisma migrate dev` 또는 `db push`)
- [ ] 로컬 Prisma Studio 접근 및 데이터 확인

## :test_tube: Acceptance Criteria (BDD/GWT)
Scenario 1: Part 스키마 구조의 영속성 검증
- Given: 올바른 형식의 `schema.prisma` 코드가 주어짐
- When: 개발자가 로컬 터미널에서 `npx prisma db push`를 실행함
- Then: 에러 없이 Supabase에 테이블이 생성되고 `Part` 테이블에 `partNo` 유니크 속성이 반영된다.

Scenario 2: 데이터 무결성 검증 (필수값 체크)
- Given: `inventoryLog` 모델이 정의됨
- When: 코드를 통해 `worker` 값이 누락된 로그 삽입을 시도함
- Then: Prisma Client 에러를 반환하며 `NOT NULL` 제약 조건이 올바르게 동작함을 확인한다.

## :gear: Technical & Non-Functional Constraints
- 제약사항: Supabase Free Tier 환경(PostgreSQL)의 타입과 호환되는 데이터 형식 준수. 기본값(`currentQty @default(0)`) 누락 금지.
- 보안: `schema.prisma` 내 DB 접속 URL 평문 노출 금지, 즉 `.env` 파일의 `DATABASE_URL`을 통해 참조할 것.

## :checkered_flag: Definition of Done (DoD)
- [ ] 모든 Acceptance Criteria를 충족하는가?
- [ ] Supabase Cloud 대시보드에서 테이블 생성이 확인되었는가?
- [ ] Prisma Format(`prisma format`) 시 문법 에러가 없는가?

## :construction: Dependencies & Blockers
- Depends on: INF-002 (Supabase 프로젝트 및 DB 생성 이슈)
- Blocks: ACT-Q01, ACT-C01 (모든 백엔드 Query/Command 이슈)


---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[Feature] ACT-Q01: 부품 목록/상세 조회 Server Action (Query) 구현"
labels: 'feature, backend, query, priority:high'
assignees: ''
---

## :dart: Summary
- 기능명: [ACT-Q01] 재고 목록 및 상세 정보 조회를 위한 읽기 전용(Read-Only) Action
- 목적: 프론트엔드가 DB 스키마 변화에 결합되지 않도록, 데이터베이스에서 필요한 재고 데이터를 추상화된 함수로 안전하게 반환한다.

## :link: References (Spec & Context)
> :bulb: AI Agent & Dev Note: 작업 시작 전 아래 문서를 반드시 먼저 Read/Evaluate 할 것.
- 관련 TASK: `DB-001` (스키마 참조)
- 통신 규약: `CON-001` (프론트/서버 간 DTO)

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] `actions/inventory.ts` 파일 생성
- [ ] [Query] 검색어(키워드) 기반 `Part` 목록 조회 기능 구현 (`title` 및 `partNo` 텍스트 검색)
- [ ] [Query] QR 스캔을 위한 URL/품번(`partNo`) 기반 단일 `Part` 엔티티 호출 함수 구현
- [ ] 결과 값 반환 시 에러 래핑 및 타입 일치 처리 (TypeScript DTO)

## :test_tube: Acceptance Criteria (BDD/GWT)
Scenario 1: 정상적인 품번 검색
- Given: 사용자가 검색어 `'103'`을 입력함
- When: 부품 검색 Server Action 함수를 호출함
- Then: `partNo`나 `partName`에 '103'이 포함된 `Part` 데이터 배열과 200 수준의 성공 응답 객체를 반환한다.

Scenario 2: 존재하지 않는 데이터 조회
- Given: DB에 없는 잘못된 `partNo` (`INVALID-999`)를 전달함
- When: 상세 조회 Action을 호출함
- Then: 서버 에러(500) 대신 `null` 또는 '결과 없음'을 의미하는 규격화된 상태 코드가 반환된다.

## :gear: Technical & Non-Functional Constraints
- 성능: DB Query 작성 시 필요 없는 `InventoryLog` 전체 배열은 Join(`include`)하지 않도록 최적화.

## :checkered_flag: Definition of Done (DoD)
- [ ] 모든 Acceptance Criteria를 충족하는가?
- [ ] Prisma Client를 활용한 Query 로직이 안전하게 래핑되었는가?
- [ ] 서버 컴포넌트 환경(`use server`) 호환성이 지켜졌는가?

## :construction: Dependencies & Blockers
- Depends on: DB-002, CON-001
- Blocks: UI-001, AI-002


---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[Feature] ACT-C01: 부품 재고 증감 (+/-) Command 로직 및 이력 기록 트랜잭션 구현"
labels: 'feature, backend, command, priority:high'
assignees: ''
---

## :dart: Summary
- 기능명: [ACT-C01] 모바일 현장 스캔 후의 핵심 재고 조작 비즈니스 로직
- 목적: 단순 상태 변경이 아닌, `Part` 모델의 수량을 갱신함과 동시에 `InventoryLog` 데이터를 무조건 함께 기록하는 원자성(Atomicity)을 보장한다.

## :link: References (Spec & Context)
> :bulb: AI Agent & Dev Note: 작업 시작 전 아래 문서를 반드시 먼저 Read/Evaluate 할 것.
- SRS 문서: `SRS_v1.md#3.2-REQ-MOB-001-모바일-QR-입출고`
- 데이터 모델: `inventoryLog` 내 `IN`, `OUT` 타입 참조

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] `actions/mutation.ts` 작성 및 `use server` 선언
- [ ] [Command] 요청 페이로드(입출고 타입, 품번, 작업자, 수량)의 유효성 검증 로직 작성(Zod 등 도입 고려)
- [ ] Prisma Transaction 설정 (재고 Update 및 Log Insert 병렬 처리)
- [ ] 출고 시 `currentQty`가 0 미만이 되지 않도록 방어 로직 구현

## :test_tube: Acceptance Criteria (BDD/GWT)
Scenario 1: 정상적인 부품 입고 (+10) 처리
- Given: 재고가 100개인 품번 `A001`에 대해 작업자 `김길동`이 `IN`, 수량 `10`을 요청함
- When: 증감 Server Action이 실행됨
- Then: DB의 `A001` `currentQty`가 110으로 업데이트 되며, `InventoryLog` 테이블에 "+10" 작업 이력이 동시에 한 줄 추가된다.

Scenario 2: 재고 수량을 음수로 만드는 출고 (-200) 요청 방어
- Given: 재고가 `50`개인 부품에 수량 `200`개의 출고(`OUT`) 요청이 들어옴
- When: 해당 Action을 실행함
- Then: 재고 업데이트가 취소(Rollback)되며, 재고 부족 400 에러("수량이 부족합니다")를 Return한다.

## :gear: Technical & Non-Functional Constraints
- 안정성: DB 트랜잭션 도중 실패할 경우 무조건 롤백되어 이전 상태가 보존되어야 함 (`$transaction` 최적화).

## :checkered_flag: Definition of Done (DoD)
- [ ] 모든 Acceptance Criteria를 충족하는가?
- [ ] 단위 테스트를 통해 음수 재고 등록 예외가 통과 테스트(Pass)를 받았는가?
- [ ] `revalidatePath` 처리가 완료되어 프론트에 데이터가 실시간 갱신되는가?

## :construction: Dependencies & Blockers
- Depends on: DB-002, CON-001
- Blocks: UI-003


---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[Feature] AI-002: AI Agent용 DB Search Tool (Function Calling) 백엔드 구현"
labels: 'feature, backend, ai, priority:medium'
assignees: ''
---

## :dart: Summary
- 기능명: [AI-002] Gemini 전용 DB 탐색 도구 세팅
- 목적: AI가 환각 증상(Hallucination) 없이 정확한 위치와 수량(SSOT)을 안내하도록, LLM이 시스템 내부 DB 조회용 함수를 Tool로써 직접 호출할 수 있도록 제공한다.

## :link: References (Spec & Context)
> :bulb: AI Agent & Dev Note: 작업 시작 전 아래 문서를 반드시 먼저 Read/Evaluate 할 것.
- SRS 문서: `SRS_v1.md#3.1-REQ-AI-001-지능형-재고-어시스턴트`
- 라이브러리: Vercel AI SDK Tool Function 규격 명세 파악

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] `zod`를 사용하여 AI 검색 인자 스키마 정의 (예: `{ keyword: string, processType?: string }`)
- [ ] Vercel AI SDK API Route 내부에서 `tools` 프로퍼티 정의
- [ ] 프롬프트 지시어(System Prompt) 작성 ("반드시 이 Tool을 사용하여 실제 위치를 확인한 후 답하라")
- [ ] Tool Execution Block 내부에 `ACT-Q01` 조회를 매핑하는 로직 작성

## :test_tube: Acceptance Criteria (BDD/GWT)
Scenario 1: AI가 Tool을 사용하여 정확한 응답 제공
- Given: DB에 "A-01-01"창고에 적재된 "사출 103" 부품 정보가 있음
- When: 채팅 API로 "사출 103 부품 창고 어디야?" 라는 페이로드를 POST함
- Then: AI가 강제로 `SearchTool`을 사용하여 "A-01-01 창고"라는 데이터를 조회한 후 그 기반으로 텍스트를 스트리밍한다.

Scenario 2: 검색할 명사가 부족한 경우
- Given: "어딨어?" 라는 주어 없는 내용이 들어옴
- When: API를 호출함
- Then: AI가 즉시 답변하지 않고, "어떤 부품번호나 공정을 검색할까요?"라는 되물음을 반환한다.

## :gear: Technical & Non-Functional Constraints
- 보안: AI의 System Prompt는 사용자 프롬프트 Injection에 의해 노출돼서는 안 됨.
- 제약: AI가 DB를 조작(Update/Insert)하는 Tool은 원천 차단(Read-Only 한정).

## :checkered_flag: Definition of Done (DoD)
- [ ] 모든 Acceptance Criteria를 충족하는가?
- [ ] Vercel AI Tool 규격에 오류가 발생하지 않는가?
- [ ] Vercel AI SDK 가이드 상의 명세에 최적화되었는가?

## :construction: Dependencies & Blockers
- Depends on: API-001(기초 엔드포인트 세팅), ACT-Q01
- Blocks: AI-003, UI-005


---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[Feature] UI-003: QR 스캔 진입용 재고 상세 화면 및 수량 조작 버튼 UI"
labels: 'feature, frontend, priority:high'
assignees: ''
---

## :dart: Summary
- 기능명: [UI-003] 모바일 환경의 재고 디테일 및 +/- 커맨드 컴포넌트 개발
- 목적: 현장 작업자가 QR 코드를 찍고 들어왔을 때, 즉각적으로 재고 수량을 차감하거나 증가시킬 수 있는 직관적인 UI 인터페이스를 완성한다.

## :link: References (Spec & Context)
> :bulb: AI Agent & Dev Note: 작업 시작 전 아래 문서를 반드시 먼저 Read/Evaluate 할 것.
- SRS 문서: `SRS_v1.md#3.2-REQ-MOB-001`
- Mock Data: `CON-001` (화면 렌더링용)
- 참조 백엔드: `ACT-C01` (명세 확인)

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] `/part/[id] 또는 /part/[partNo]` Dynamic Route 페이지 컴포넌트 생성.
- [ ] shadcn/ui의 Card, Button 컴포넌트를 조합하여 재고 상세 섹션 렌더링
- [ ] 현장 작업용 큼지막한 [+ 입고], [- 출고] 액션 버튼 및 입력 다이얼로그(Dialog) 연결
- [ ] 버튼 클릭 시 React 훅의 `useTransition`을 활용하여 Loading State 표시 및 `ACT-C01` Server Action 실행

## :test_tube: Acceptance Criteria (BDD/GWT)
Scenario 1: 재고 페이지 렌더링
- Given: URL 파라미터로 올바른 `partNo`를 가짐
- When: 모바일 브라우저로 접속함
- Then: 부품명, 공정, 현재수량(`currentQty`), 부품사진(`imageUrl`)이 정상적으로 렌더된다.

Scenario 2: Optimistic UI 업데이트(또는 빠른 Loading) 피드백
- Given: 재고 상세페이지에서 사용자가
- When: [- 10] 출고 버튼을 클릭함
- Then: 즉각적으로 버튼이 비활성화(Disabled) 되거나 스피너가 나타나 중복 클릭을 방지한 후, 완료 시 화면의 `currentQty`가 갱신되어야 함.

## :gear: Technical & Non-Functional Constraints
- 성능: 모바일 사파리/크롬 브라우저에서 터치 민감도가 지연되지 않아야 함. 터치 타겟(Button)은 48px 이상의 크기를 지향.

## :checkered_flag: Definition of Done (DoD)
- [ ] 모든 Acceptance Criteria를 충족하는가?
- [ ] Server Action과 연동되어 실제 DB 데이터가 조작되는가?
- [ ] Linter & Prettier 기준을 모두 통과하는가?

## :construction: Dependencies & Blockers
- Depends on: ACT-C01, UI-001
- Blocks: NFR-002 (PWA 대응의 기준 화면이 됨)
