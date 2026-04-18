# [SRS-001] AS-Safe: 상세 TASK 전체 리스트
Date: 2026-04-18
Target Document: `SRS_v1.md`

## 1. 프로젝트 WBS (Work Breakdown Structure) 개요
본 문서는 AS-Safe (단산 부품 재고 관리 시스템)의 SRS(요구사항 정의서)를 기반으로, 실제 구현을 위한 전체 작업 단위(TASK)를 정의한 문서입니다. 각 작업은 기능적 분리에 따른 유형(Type)과 선행되어야 하는 작업(Dependency)을 기준으로 분류되어 있으며, 실질적인 개발 스프린트(Sprint)의 티켓으로 즉시 활용할 수 있습니다.

## 2. 상세 TASK 리스트 차트

| Task ID | Task 유형 | Task 명칭 | 간략 설명 | 선행 작업 (Dependency) |
| :--- | :--- | :--- | :--- | :--- |
| **TSK-INF-001** | 인프라 설정 | Next.js 프로젝트 초기화 | `npx create-next-app` (App Router, TS, Tailwind CSS) 환경 구성 | 없음 |
| **TSK-INF-002** | 인프라 설정 | Supabase 클라우드 셋업 | Supabase 프로젝트 생성 및 DB 접근 권한/키 등 프로비저닝 | 없음 |
| **TSK-INF-003** | 인프라 설정 | 뷰 단 디자인 계층 설정 | shadcn/ui 코어 설치 및 Button, Input 등 필수 UI 컴포넌트 세팅 | TSK-INF-001 |
| **TSK-DB-001** | 데이터베이스 | Prisma ORM & 스키마 설계 | `schema.prisma` 구성 (Part, InventoryLog 모델 데이터 정의) | TSK-INF-001, 002 |
| **TSK-DB-002** | 데이터베이스 | DB Schema 마이그레이션 | `npx prisma db push`를 통한 Supabase 물리 테이블 동기화 | TSK-DB-001 |
| **TSK-DB-003** | 데이터베이스 | DB Client 환경 구성 | Next.js 서버리스 한계 극복을 위한 Prisma Client 전역 객체 세팅 | TSK-DB-002 |
| **TSK-BE-001** | 백엔드 모델 | 기본 입출고 Server Actions 구현 | 부품 목록 조회, 상세 조회 및 초기 부품 등록 로직 작성 | TSK-DB-003 |
| **TSK-BE-002** | 백엔드 모델 | 재고 증감 트랜잭션 구현 | +, - 처리 시 `currentQty` 값 연산 및 이력(InventoryLog) 저장 | TSK-DB-003 |
| **TSK-BE-003** | 백엔드 (AI) | Gemini AI API 라우트 연동 | Vercel AI SDK 설치 및 `streamText` 기반 대화형 API 라우트 구성 | TSK-INF-001 |
| **TSK-BE-004** | 백엔드 (AI) | AI DB 탐색 함수(Tool) 구현 | AI가 DB에서 질문된 부품 재고를 검색하도록 Function Calling 연동 | TSK-DB-003, BE-003 |
| **TSK-FE-001** | 프론트엔드 (UI) | 메인 대시보드 화면 렌더링 | 부품 리스트 테이블/카드, 전체 검색 UI 개발 | TSK-INF-003, BE-001 |
| **TSK-FE-002** | 프론트엔드 (UI) | 모바일 상세 및 수량 조작 화면 | 개별 품번 정보 렌더링 및 입출고 증감(+ / -) 인터랙션 페이지 개발 | TSK-FE-001, BE-002 |
| **TSK-FE-003** | 프론트엔드 (UI) | 웹 기반 모바일 QR 스캐너 | `html5-qrcode` 또는 유사 툴 활용을 통한 모바일 QR 인식 및 자동 진입 | TSK-FE-002 |
| **TSK-FE-004** | 프론트엔드 (UI) | AI 지능형 챗봇 화면 개발 | `useChat` 모듈을 활용하여 현장 작업자의 자연어 질문/답변 뷰 구현 | TSK-INF-003, BE-004 |
| **TSK-DEP-001** | 배포 및 환경 | 시스템 환경 변수 일원화 | 로컬(`.env`) 및 Vercel 상에 DB 환경변수, Gemini API Key 세팅 | 누적 |
| **TSK-DEP-002** | 배포 및 운영 | Vercel 프로덕션 CI/CD 구축 | GitHub Repository 연동 및 코드 Push 트리거 자동 배포(런칭) 설정 | TSK-DEP-001 |

## 3. TASK 의존성(선후관계) 기반 권장 개발 파이프라인
효율적인 1인 풀스택 개발 진행을 위하여 다음의 Phase(단계별 흐름) 순서로 작업을 추진하는 것을 권장합니다.

1. **Phase 1: 데이터 백본 구축 (인프라 & DB)**  
   *`TSK-INF-001` → `TSK-INF-002` → `TSK-DB-001` → `TSK-DB-002` → `TSK-DB-003`*  
   _(목표: 로컬 코드와 원격 데이터베이스 연결 및 조작 준비 시점)_
2. **Phase 2: 핵심 비즈니스 로직(Core UX) 구현**  
   *`TSK-INF-003` → `TSK-BE-001 / BE-002` → `TSK-FE-001 / FE-002`*  
   _(목표: AI 및 카메라 제외, 수동으로 재고 검색 및 조정이 완벽히 동작하는 MVP 완성)_
3. **Phase 3: 부가 가치(Mobile/AI) 피처 통합**  
   *`TSK-FE-003` (QR 모바일 스캔) → `TSK-BE-003 / BE-004 / FE-004` (AI Gemini 기능 적용)*  
   _(목표: 지능형 재고 어시스턴트 적용 및 스캐너 연동을 통한 기획 의도 완성)_
4. **Phase 4: 서버 배포 및 필드 테스트**  
   *`TSK-DEP-001` → `TSK-DEP-002`*  
   _(목표: 스마트폰 브라우저 상에서 실제 필드 동작 테스트 및 배포 최종화)_

---
*💡 개발 팁: 위 명시된 Task ID(`TSK-XXX-000`)를 Github 브랜치(Branch) 생성명(예: `feat/TSK-INF-001`)이나 커밋(Commit) 메시지의 헤더로 적극 활용하면, 나중에도 어떤 작업을 위한 파트인지 명확한 추적관리가 가능합니다.*
