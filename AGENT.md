
## 1. Tech Stack

- **Language:** TypeScript 5.4.5
- **Framework:** Express.js 4.19.2 (Node.js 22.22.2)
- **ORM:** Prisma 6.0.0
- **Testing:** Vitest 2.0.0, Supertest 7.0.0
- **Linter/Formatter:** ESLint 8.57.0, Prettier 3.2.5

## 2. Dev Environment

- **Database:** PostgreSQL (Port: 5432)
- **Frontend:** Port 5173

## 3. Global DO NOT

- **Controller에서 직접 DB 접근 금지** → 반드시 UseCase와 Repository를 통해 접근
- **UseCase에서 Entity 직접 노출 금지** → 외부 계층(Presentation)으로는 반드시 DTO만 반환
- **Repository에서 Domain Model/Entity 반환 금지** → 반드시 **Infra DTO**로 변환하여 반환 (계층 오염 방지)
- **Domain Layer에서 외부 라이브러리 의존 금지** → 순수 TypeScript 클래스/객체로 비즈니스 로직 캡슐화
- **Mapper 내 비즈니스 로직 작성 금지** → 오직 필드 매핑 및 타입 변환만 담당
- **Save DTO 사용 금지** → 영속성 계층 저장(`save`) 시에는 도메인 모델을 직접 전달
- **any 타입 사용 금지** → 인터페이스 및 타입을 명확히 정의하여 타입 안정성 확보
- **Magic Number/String 금지** → 상수는 반드시 별도의 `constant.ts` 또는 설정 파일로 관리
- **리팩토링 시 테스트 코드 변경 금지** → 기존 동작이 유지됨을 보장해야 하므로 테스트 코드는 수정하지 않음
- **불필요한 전체 코드베이스 스캔 금지** → 다음의 '탐색 원칙'을 준수하여 효율적으로 탐색할 것:
    1. **의존성 기반**: 작업 대상의 `import` 및 참조 관계를 따라가며 연관 레이어만 식별.
    2. **검색 우선**: 전체 파일을 읽기 전 `grep` 등을 사용하여 키워드 기반으로 탐색 범위 압축.
    3. **규칙 기반**: 프로젝트의 폴더 구조 규칙(`docs/rules/`)에 따라 예상 경로 위주로 탐색.

## 4. Commands

- `npm install`: 의존성 설치
- `npm run dev`: 개발 서버 실행 (Nodemon/ts-node)
- `npm run build`: 프로덕션 빌드 (TSC)
- `npm run test`: 전체 테스트 실행 (Vitest)
- `npm run lint`: ESLint를 통한 코드 스타일 및 오류 체크
- `npm run format`: Prettier를 통한 코드 포맷팅

## 5. Implementation Workflow

1.  **폴더 구조 우선 확인**: 새로운 기능 구현 시, `ARCHITECTURE.md`에 정의된 4계층(`domain/`, `application/`, `infrastructure/`, `presentation/`) 구조를 확인합니다.
2.  **TDD (Test-First) 필수**:
    - 핵심 비즈니스 로직(Domain, UseCase) 구현 전에 반드시 테스트 코드를 먼저 작성합니다.
    - **위치**: 테스트 파일은 대상 코드와 **동일한 위치**에 생성합니다. (예: `book.usecase.ts` -> `book.usecase.test.ts`)
    - **순서**: `Domain Model` 정의 -> **Domain 테스트 작성** -> `UseCase/Repository` 인터페이스 정의 -> **UseCase 테스트 작성** -> 실제 구현.
3.  **DTO & Mapper 정의**: 계층 간 이동 시 필요한 DTO와 Mapper를 먼저 정의하여 데이터 흐름을 명확히 합니다.
4.  **레이어별 순차 구현**: `Domain` -> `Infrastructure` -> `Application` -> `Presentation` 순으로 구현 및 검증을 진행합니다.
5.  **검증**: 모든 구현 완료 후 `docs/rules/`의 세부 규칙 준수 여부와 전체 테스트 통과를 확인합니다.