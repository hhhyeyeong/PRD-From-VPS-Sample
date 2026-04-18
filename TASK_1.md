# AS-Safe (단산 부품 재고 관리 시스템) 개발 TASK List

**문서명:** `TASK.md`
**작성자:** Senior Technical Project Manager & System Architect
**분석 대상:** `SRS_v1.md`
**작성 기준:** 단일 진실 공급원(Contract/Data) 우선, 상태 변경(Mutation) 기반 컨텍스트 분리, 테스트 주도 확인, NFR 및 의존성 매핑 포함.

## 1. Task 추출 근거 및 전략 매핑
- **Step 1 (Data & Protocol):** [DB], [Contract], [Mock] 단위 도출. (Supabase Schema, TypeScript DTO)
- **Step 2 (Logic & Mutation):** 데이터베이스 쓰기(Command)와 읽기(Query)의 엄격한 분리로 오버엔지니어링 방지.
- **Step 3 (Test & AC):** 재고 증감 무결성, QR 스캔 예외 통제, AI 응답 규격 점검을 `[Test]` 코드로 전환.
- **Step 4 (NFR & Dependency):** PWA(오프라인), Storage, 비용 최적화(Free Tier Vercel limits) 등을 `[NFR]`과 인프라 티켓으로 분리.

## 2. 전체 TASK 리스트 (Epic & Feature 단위)

| Task ID | Epic (도메인) | Feature (기능명) | 관련 SRS 섹션 | 선행 태스크 (Dependencies) | 복잡도 (H/M/L) |
|---|---|---|---|---|---|
| **INF-001** | Infra Setup | `[Infra]` Next.js 14 App Router, Tailwind 및 Vercel 계정 연동 기본 세팅 | 1.1, 4.1 | None | L |
| **INF-002** | Infra Setup | `[Infra]` Supabase 프로젝트 생성 및 Postgres DB / 연결 URL 확보 | 1.2, 4.1 | None | L |
| **DB-001** | Data Model | `[DB]` Prisma Model (Part, InventoryLog) 스키마 작성 및 파일 정의 | 2. 코트 스니펫 | INF-001 | M |
| **DB-002** | Data Model | `[DB]` Prisma DB Push 자동화 및 Seed(초기 Mock Data) 스크립트 작성 | 5. 개발 환경 | DB-001, INF-002 | L |
| **CON-001** | Protocol | `[Contract]` 프론트/백엔드간 통신용 Type/Interface (DTO) 정의 | 2. 데이터 구조 | None | L |
| **NFR-001** | NFR & Storage | `[NFR/Infra]` 부품 사진 (imageUrl) 저장을 위한 Supabase Storage 설정 및 보안 룰 세팅 | 수정 제안 | INF-002 | M |
| **ACT-Q01** | Inventory Core | `[Feature/Query]` 키워드(`partNo`, `process` 등)기반 부품 목록/상세 조회 Server Action 구현 | 3. 핵심 기능 | DB-002, CON-001 | L |
| **ACT-C01** | Inventory Core | `[Feature/Command]` 재고 증감(+/-) 처리 및 `InventoryLog` Insert 트랜잭션 로직 구현 | 3.2 REQ-MOB-001 | DB-002, CON-001 | M |
| **AI-001** | AI Assistant | `[Mock]` 프론트엔드 UI용 Gemini 응답 (streamText) Mocking API 구축 | 3.1 REQ-AI-001 | INF-001 | L |
| **AI-002** | AI Assistant | `[Feature/Query]` AI Agent가 `Part` 테이블을 탐색할 수 있는 `Search Tool`(Function Calling) 구현 | 3.1 REQ-AI-001 | ACT-Q01 | H |
| **AI-003** | AI Assistant | `[Feature/Command]` Vercel AI SDK 기반 Gemini 1.5 Pro 시스템 프롬프트 및 API 연동 | 3.1 REQ-AI-001 | AI-002 | M |
| **UI-001** | Web UI | `[Feature/Query]` 메인 대시보드 화면 및 검색바 컴포넌트 (shadcn/ui) 구현 | 1.2 상세 기술 | ACT-Q01 | M |
| **UI-002** | Web UI/QR | `[Feature/Command]` 모바일 QR 스캐너 연동 컴포넌트 구현 및 `partNo` 딥링크 연결 | 3.2 REQ-MOB-001 | UI-001 | M |
| **UI-003** | Web UI/QR | `[Feature/Command]` 스캔 진입용 재고 상세 화면 구현 및 `ACT-C01` 증감 액션 바인딩 | 3.2 REQ-MOB-001 | UI-002, ACT-C01 | M |
| **UI-004** | Web UI/AI | `[Feature/Command]` 자연어 채팅 UI 컴포넌트(입력창 및 스크롤)와 모킹 API(AI-001) 연동 | 3.1 REQ-AI-001 | AI-001 | M |
| **UI-005** | Web UI/AI | `[Feature/Command]` 실서버용 AI 라우트(AI-003)로 통신 타겟 스위칭 및 스트림 렌더링 검증 | 3.1 REQ-AI-001 | UI-004, AI-003 | M |
| **TST-001** | Testing & QA | `[Test]` 입출고 트랜잭션에서 `currentQty` 음수 변환 시 예외 스로우 검증 (Unit Test) | 2. 모델 검증 | ACT-C01 | M |
| **TST-002** | Testing & QA | `[Test]` 잘못된 형식의 QR 인식 시 에러 바운더리 동작 및 안내 메시지 표출 (E2E Test) | 3.2 REQ-MOB-001 | UI-002 | M |
| **TST-003** | Testing & QA | `[Test]` AI 검색 Tool에 없는 부품 번호를 질의했을 경우의 거절 메세지 규격 확인 (Integration) | 3.1 REQ-AI-001 | AI-003 | M |
| **NFR-002** | NFR/DevOps | `[NFR/Feature]` 오프라인 통신(Wi-fi 단절) 대응용 Next.js PWA 매니페스트 및 서비스워커 설정 | 수정 제안 | UI-003 | H |
| **DEP-001** | DevOps | `[Infra]` GitHub 원격 브랜치 배포 트리거 및 생성된 모든 환경변수(.env) Vercel 주입 자동화 | 5. 인프라 배포 | INF-002 | L |

## 3. GitHub Project / Jira 보드 운영 가이드 (Manager's Note)

1. **에이전트에게 지시할 때의 규칙:** 
   - 절대 `UI-003`을 진행하기 전에 `ACT-C01`과 `DB-002`가 완료되었는지 **의존성(`Depends on`)** 필드를 확인하게 통제하십시오.
2. **SSOT(Single Source of Truth) 확보:**
   - 처음 시작할 때 `DB-X`, `CON-X`를 최우선 Sprint로 배치하십시오. 에이전트가 "데이터 모델이 없어서 임의로 만들었습니다"라고 보고하는 상황을 원천 차단해야 합니다.
3. **CQRS 분리 검증:**
   - 읽기 전용 티켓(`ACT-Q01`)을 진행할 때, 에이전트가 불필요하게 쓰기/트랜잭션 코드를 주입하지 않도록 PR 단위로 엄격히 리뷰(DoD 점검)하십시오.
