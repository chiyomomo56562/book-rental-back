## Domain Rules

AI는 Domain Layer 구현 시 아래 규칙을 반드시 준수해야 한다.

---

### 1. Core Responsibility

- Domain Layer는 순수 비즈니스 로직만 포함한다
- 모든 비즈니스 규칙, 정책, 상태 변경 로직은 Domain에 위치해야 한다

- UseCase 또는 다른 Layer에 비즈니스 로직을 분산시키지 않는다

---

### 2. Independence

- Domain Layer는 어떤 외부 기술에도 의존하지 않는다

금지:
- ORM (Prisma, TypeORM 등)
- Framework (Express, NestJS 등)
- 외부 API
- DB 접근 코드

---

### 3. Entity Rules

- Entity는 식별자(ID)를 가지는 객체로 정의한다
- Entity는 상태(State)와 행동(Behavior)을 함께 가진다

- 상태 변경은 반드시 Entity 내부 메서드를 통해서만 수행한다
- 외부에서 직접 상태를 변경하는 코드 금지

---

### 4. Value Object Rules

- Value Object는 불변(Immutable) 객체로 정의한다
- 동일한 값이면 동일한 객체로 취급한다

- setter 사용 금지
- 생성 시점에 모든 값이 결정되어야 한다

---

### 5. Business Logic Rules

- 모든 비즈니스 판단(조건문, 정책)은 Domain에 위치해야 한다

예:
- 상태 변경 가능 여부
- 정책 검증
- 계산 로직

- 단순 데이터 조작이 아닌 “의미 있는 로직”은 반드시 Domain으로 이동한다

---

### 6. Domain Service Rules

- 하나의 Entity로 표현하기 어려운 로직은 Domain Service로 분리한다

- Domain Service는 상태를 가지지 않는다 (Stateless)
- Domain Service는 여러 Entity를 조합하는 역할을 한다

---

### 7. Invariant Protection

- Domain은 항상 유효한 상태를 유지해야 한다

- 잘못된 상태로 생성되는 것을 허용하지 않는다
- 상태 변경 시 규칙 위반이면 예외를 발생시킨다

---

### 8. Method Design

- 메서드 이름은 비즈니스 의미를 명확히 드러내야 한다

예:
- activate()
- cancel()
- changeStatus()

금지:
- setStatus()
- updateData()

---

### 9. Exception Rules

- 비즈니스 규칙 위반 시 명확한 Domain Exception을 발생시킨다

- 의미 없는 generic error 사용 금지
- 에러 메시지는 비즈니스 의미를 포함해야 한다

---

### 10. Prohibited Patterns

- Domain에서 DB 접근 코드 작성 금지
- Domain에서 외부 API 호출 금지
- Domain에서 Framework 코드 사용 금지
- Domain을 단순 DTO처럼 사용하는 코드 금지
- getter/setter만 존재하는 빈 객체 생성 금지