## Test Rules

AI는 테스트 코드 작성 시 아래 규칙을 반드시 준수해야 한다.

---

### 1. Core Principle

- **TDD (Test-Driven Development) 필수**: 핵심 비즈니스 로직(Domain, UseCase)을 개발할 때는 반드시 실제 구현 코드보다 테스트 코드를 먼저 작성하여 명세를 검증해야 한다
- 테스트는 비즈니스 로직의 정확성과 아키텍처 규칙 준수를 검증해야 한다
- 단순 구현 디테일이 아닌 "행동(Behavior)"을 검증한다

**TDD 사이클 (Red → Green → Refactor)**:

1. **Red**: 실패하는 테스트를 먼저 작성한다. 이 시점에 구현 코드는 존재하지 않는다
2. **Green**: 테스트가 통과할 최소한의 구현 코드를 작성한다. 완벽함보다 통과를 우선한다
3. **Refactor**: 테스트가 계속 통과하는 상태에서 코드 품질을 개선한다. 중복 제거 및 추상화

이 사이클은 하나의 기능(함수/메서드)을 단위로 반복한다

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

- **Repository 인터페이스**, **External Port 인터페이스**를 Mock으로 대체한다
- 구현체(Prisma Client, Axios 등)를 직접 Mock하는 것을 금지한다
- 입력 → 결과 → Side Effect(Repository 호출 횟수/순서) 여부를 검증한다

---

### 6. Test Isolation

- 테스트는 서로 독립적으로 실행되어야 한다

- 테스트 간 상태 공유 금지
- 순서 의존성 금지

**격리 방법**:
- 각 테스트 실행 전 상태 초기화는 `beforeEach`를 사용한다
- 테스트 Suite 전체 공유 자원(DB 연결 등)은 `beforeAll` / `afterAll`로 생명주기를 명시한다
- Mock은 `afterEach`에서 `vi.clearAllMocks()` 또는 `jest.clearAllMocks()`로 반드시 초기화한다

---

### 7. Naming & Location Rules

**파일 위치 및 네이밍 규칙 (계층별 통합 기준)**:

| 테스트 유형 | 위치 | 파일명 형식 | 예시 |
|------------|------|------------|------|
| Unit (Domain) | 대상 파일과 동일 폴더 | `[name].entity.test.ts` | `book.entity.test.ts` |
| Unit (UseCase) | 대상 파일과 동일 폴더 | `[action]-[name].usecase.test.ts` | `rent-book.usecase.test.ts` |
| Unit (Mapper) | 대상 파일과 동일 폴더 | `[name].mapper.test.ts` | `book.mapper.test.ts` |
| Integration | `src/__integration__/` | `[name].integration.test.ts` | `book-api.integration.test.ts` |
| E2E | `test/e2e/` | `[scenario].e2e.test.ts` | `rent-return-flow.e2e.test.ts` |
| Infrastructure | 대상 파일과 동일 폴더 | `[name].repository.test.ts` | `book.repository.test.ts` |

- 테스트 이름은 "행동 기반"으로 작성한다

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
- 테스트는 **Arrange → Act → Assert (AAA)** 구조를 따른다

```typescript
it('should_[동작]_when_[조건]', () => {
  // Arrange: 테스트 대상과 입력 데이터를 준비한다
  const input = ...;
  const mockRepo = ...;

  // Act: 테스트 대상 함수/메서드를 실행한다
  const result = sut.execute(input);

  // Assert: 결과와 Side Effect를 검증한다
  expect(result).toEqual(...);
});
```

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

---

### 13. Integration Test Rules

통합 테스트는 두 개 이상의 계층이 협력하는 흐름을 검증한다.

- **대상**: Controller ↔ UseCase ↔ Repository가 연동되는 API 엔드포인트 흐름
- **도구**: `supertest` + 실제 애플리케이션 인스턴스를 사용한다 (DB는 `sqlite` / `pg-mem` 등 인메모리 DB 또는 Docker 테스트 컨테이너를 사용한다)
- **위치**: `src/__integration__/` 디렉토리에 `[name].integration.test.ts` 파일명을 사용한다
- **범위**:
  - HTTP 요청 → 응답의 전체 흐름 (상태 코드, 응답 바디 구조)
  - 에러 케이스 시 응답 포맷(`ApiResponse`)의 일관성 검증
- **격리**: 각 테스트는 독립적인 DB 상태를 가져야 하며, `beforeEach`/`afterEach`로 데이터를 초기화한다

---

### 14. E2E Test Rules

E2E 테스트는 실제 환경에 가까운 전체 시스템 흐름을 검증한다.

- **대상**: 핵심 비즈니스 시나리오 (예: 회원가입 → 로그인 → 도서 대여 → 반납)
- **도구**: `vitest` (`globalSetup` / `globalTeardown` 활용) + `supertest`를 사용한다
  - `vitest`의 `globalSetup`으로 실제 앱 서버를 기동하고, `supertest`로 HTTP 요청을 수행한다
  - Unit/Integration과 동일한 Runner(`vitest`)를 사용하되, 별도 config(`vitest.e2e.config.ts`)로 분리한다
- **위치**: `test/e2e/` 디렉토리에 위치시키며, 파일명은 `[scenario].e2e.test.ts`를 사용한다
- **실행 시점**: CI 파이프라인의 별도 단계에서 실행하며, Unit/Integration 테스트와 분리한다
- **주의**: E2E 테스트는 실제 DB 또는 전용 스테이징 환경을 사용할 수 있으며, 실행 후 반드시 데이터를 정리(Teardown)해야 한다

---

### 15. Infrastructure Test Guidelines

Infrastructure 계층의 테스트 선택 기준은 다음과 같다.

- **테스트 필수 대상**:
  - 복잡한 쿼리 로직이 포함된 Repository 메서드 (예: 필터링, 페이지네이션, 조인)
  - Mapper 변환 결과가 예상 DTO 구조와 일치하는지 확인이 필요한 경우
- **테스트 선택 대상**:
  - 단순 CRUD Repository 메서드 (Integration Test로 대체 가능)
  - 외부 API 클라이언트 (목(Mock) 서버 또는 계약 테스트로 대체 권장)
- **도구**: 실제 DB 대신 인메모리 DB(`sqlite`, `pg-mem` 등) 또는 테스트 컨테이너(Docker)를 사용한다
- **파일 위치**: 대상 파일과 동일한 폴더에 `[name].repository.test.ts` 형식으로 위치시킨다