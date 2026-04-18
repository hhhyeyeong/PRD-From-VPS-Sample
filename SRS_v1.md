# [SRS-001] AS-Safe: 단산 부품 재고 관리 시스템
Date: 2026-04-18 | Revision: 1.1 (Final Review)
Tech Stack: Next.js 14, Prisma, Supabase, Vercel AI SDK (Gemini)

## 1. 기술적 시스템 설계 (Technical System Context)
### 1.1 Full-Stack Architecture
본 시스템은 인프라 관리 부담을 최소화하기 위해 Serverless 아키텍처를 채택한다. 별도의 API 서버 없이 Next.js의 기능을 활용하여 데이터와 AI를 처리한다.

### 1.2 상세 기술 스택 (Explicit Tech Stack)
- **Framework**: Next.js 14 (App Router) - Server Actions로 백엔드 대체.
- **Language**: TypeScript - 정적 타입을 통해 초보자의 런타임 에러 방지.
- **Database**: Supabase (PostgreSQL) - 인증(Auth)과 DB를 한 번에 해결.
- **ORM**: Prisma - SQL 없이 JS 객체로 데이터 조작 (러닝커브 완화).
- **AI**: Vercel AI SDK + Google Gemini 1.5 Pro - 자연어 재고 검색 및 분석.
- **UI**: Tailwind CSS + shadcn/ui - 디자인 고민 없이 고품질 컴포넌트 사용.

## 2. 데이터 구조 설계 (Data Structural Design)
초급 개발자가 가장 어려워하는 것이 DB 설계입니다. Prisma 스키마를 통해 구조를 정의합니다.

### 코드 스니펫
```prisma
// schema.prisma (핵심 구조)
model Part {
  id           String   @id @default(cuid())
  partNo       String   @unique // 품번
  partName     String           // 품명
  process      String           // 사출, 프레스 등
  moldId       String?          // 금형번호
  binLocation  String           // 창고 위치 (A-01-01)
  currentQty   Int      @default(0)
  moq          Int      @default(0)
  imageUrl     String?          // 부품 사진
  createdAt    DateTime @default(now())
  logs         InventoryLog[]
}

model InventoryLog {
  id        String   @id @default(cuid())
  partId    String
  type      String   // IN(입고), OUT(출고)
  quantity  Int
  worker    String   // 작업자
  part      Part     @relation(fields: [partId], references: [id])
  createdAt DateTime @default(now())
}
```

## 3. 핵심 기능 요구사항 (Specific Functional Requirements)
### 3.1 REQ-AI-001: 지능형 재고 어시스턴트
- **설명**: Gemini API를 연동하여 채팅창에 "사출 금형 103번으로 찍는 부품 재고 어디 있어?"라고 물으면 위치와 수량을 즉시 답변함.
- **구현**: Vercel AI SDK의 `streamText` 함수를 사용하여 실시간 답변 구현.

### 3.2 REQ-MOB-001: 모바일 QR 입출고
- **설명**: 스마트폰 카메라로 QR 코드를 스캔하면 해당 `partNo`의 상세 페이지로 즉시 이동하여 -, + 버튼으로 재고를 수정함.

## 4. 운영 및 리소스 분석 (Operational Perspective)
### 4.1 리소스 비용 (Cost Analysis)
초기 운영 비용은 **월 0원(Free Tier)**을 목표로 한다.
- **Vercel**: Hobby Plan (무료 - 개인/소규모 프로젝트)
- **Supabase**: Free Tier (500MB DB - 수만 건의 재고 데이터 수용 가능)
- **Gemini API**: 무료 할당량 범위 내 사용 (분당 요청 수 제한 내)

### 4.2 개발 난이도 및 러닝커브 (Learning Curve)
- **Python 경험자라면**: TypeScript의 문법은 Python과 유사하나, **'비동기 처리(async/await)'**와 'React Hook(useState)' 개념을 익히는 데 약 1~2주의 집중 학습이 필요함.
- **지원 방안**: Cursor AI(IDE)를 활용하여 "Prisma로 특정 품번 재고를 업데이트하는 Server Action 코드를 짜줘"와 같은 방식으로 구현 가능.

## 5. 인프라 및 배포 전략 (Deployment)
- **개발 환경**: 로컬에서 `npx prisma studio`를 통해 GUI로 데이터 관리.
- **배포 환경**: Vercel과 GitHub을 연동하여 코드 수정 후 `git push` 시 자동 반영.
- **데이터 백업**: Supabase의 일일 자동 백업 기능 활용.

---
### 시니어 개발자의 최종 검토 의견
> "구조적으로 완벽합니다. 특히 Next.js 단일 프레임워크 선택은 관리 포인트가 없어 1인 개발자에게 최적입니다."

### 수정 제안 및 보완점
- **오프라인 대응**: 공장 창고 내부에서 Wi-Fi가 끊길 경우를 대비해, Next.js의 PWA(Progressive Web App) 기능을 나중에 추가하여 아이콘으로 앱처럼 설치하게 만드는 것을 추천합니다.
- **이미지 스토리지**: 부품 사진을 DB에 직접 넣지 말고, Supabase Storage를 활용하여 이미지 URL만 저장하는 방식을 명시했습니다.
