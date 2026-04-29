## 1. Global Strict Constraints

AI는 아래 사항을 절대 위반해서는 안 된다.

- **Type Safety:** `any` 사용 금지. 모든 데이터는 명확한 타입으로 정의한다.

- **No Silent Failure:** 에러를 무시하거나 로그 없이 처리하는 행위 금지

- **No Circular Dependency:** 순환 참조(import cycle) 생성 금지

- **No Raw Data Usage:** 외부 데이터(Request, DB, API)를 그대로 사용하지 않는다

- **Async Consistency:** 모든 비동기 로직은 `async/await` 기반으로 작성한다 (`.then` 금지)

- **No Shared Abuse:** shared 영역에 비즈니스 로직 작성 금지

- **Strict DTO Communication:** 모든 계층 간 통신 시 원본 객체(Domain Model 등) 직접 전달을 금지하며, 반드시 전용 DTO를 통해 데이터를 주고받는다.

## 2. Quality & Implementation Rule

- **Single Responsibility:** 모든 함수와 모듈은 하나의 책임만 가진다

- **Explicit Naming:** 이름은 역할과 의도를 명확히 드러내야 한다

- **Simplicity First:** 불필요한 추상화 금지, 읽기 쉬운 코드 우선

- **Pure Logic Preference:** 가능한 한 순수 함수 형태로 작성한다

- **Testable Code:** 외부 의존성과 분리된 테스트 가능한 구조로 작성한다