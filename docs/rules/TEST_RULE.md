## Test Rules

AI는 테스트 코드 작성 시 아래 규칙을 반드시 준수해야 한다.

---

### 1. Core Principle

- **TDD (Test-Driven Development) 필수**: 핵심 비즈니스 로직(Domain, UseCase)을 개발할 때는 반드시 실제 구현 코드보다 테스트 코드를 먼저 작성하여 명세를 검증해야 한다
- 테스트는 비즈니스 로직의 정확성과 아키텍처 규칙 준수를 검증해야 한다
- 단순 구현 디테일이 아닌 “행동(Behavior)”을 검증한다

---

### 2. Test Scope Rules

- Domain, UseCase는 반드시 테스트 대상이다
- Infrastructure는 선택적으로 테스트한다 (외부 의존성 중심)

- Controller(API)는 통합 테스트 또는 E2E로 검증한다

---

### 3. Unit Test Rules

- Unit Test는 하나의 클래스 또는 함수만을 테스트한다

- 외부 의존성(DB, API 등)은 반드시 Mock 또는 Stub으로 대체한다
- 실제 DB/API 호출 금지

---

### 4. Domain Test Rules

- 모든 비즈니스 규칙은 Domain 테스트로 검증한다

포함:
- 상태 변경
- 정책 검증
- 예외 발생

- happy path + failure case 모두 작성한다

---

### 5. UseCase Test Rules

- UseCase는 비즈니스 흐름을 검증한다

- Repository, External API는 Mock으로 대체한다
- 입력 → 결과 → Side Effect 호출 여부를 검증한다

---

### 6. Test Isolation

- 테스트는 서로 독립적으로 실행되어야 한다

- 테스트 간 상태 공유 금지
- 순서 의존성 금지

---

### 7. Naming & Location Rules

- **Location**: 테스트 파일은 별도의 폴더로 분리하지 않고, 테스트하려는 대상 파일과 **동일한 경로(같은 폴더)**에 위치시킨다
- **File Naming**: 파일명은 대상 파일명에 `.spec.ts` (또는 `.test.ts`)를 붙여 사용한다 (예: `create-user.usecase.spec.ts`)

- 테스트 이름은 “행동 기반”으로 작성한다

형식:
- should + 동작 + 조건

예:
- should_create_user_when_valid_input
- should_throw_error_when_user_not_found

금지:
- test1
- works
- success_case

---

### 8. Assertion Rules

- 하나의 테스트는 하나의 명확한 목적을 가져야 한다

- 불필요한 assertion 금지
- 핵심 결과만 검증한다

---

### 9. Mocking Rules

- Mock은 외부 의존성에만 사용한다

금지:
- Domain 객체를 Mock으로 대체
- 내부 로직까지 Mock 처리

- Mock은 최소한으로 사용한다

---

### 10. Error Case Rules

- 모든 중요한 로직은 실패 케이스 테스트를 포함해야 한다

예:
- 잘못된 입력
- 상태 불일치
- 정책 위반

---

### 11. Deterministic Rules

- 테스트 결과는 항상 동일해야 한다

금지:
- 랜덤 값 사용
- 실제 시간 의존성
- 외부 상태 의존

---

### 12. Prohibited Patterns

- 실제 DB/API 호출 테스트 금지 (Unit 기준)
- 테스트 간 의존성 생성 금지
- 구현 세부사항(private method 등) 테스트 금지
- 의미 없는 assertion 금지