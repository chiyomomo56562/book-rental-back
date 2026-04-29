# Architecture Decision Records (ADR)

## [ADR 001: 계층형 아키텍처 및 의존성 역전 원칙 적용]

**Status:** Accepted

### Context

- 복잡한 비즈니스 로직과 외부 인프라 기술(DB, Framework)의 강결합으로 인해 유지보수 및 테스트가 어려워짐
- 핵심 도메인 로직을 보호하고, 기술 스택 변경 시 시스템 전체에 미치는 영향을 최소화할 필요가 있음

---

### Decision

- 시스템을 Domain, Application, Infrastructure 계층으로 명확히 분리한다.
- 모든 의존성은 내부(Domain)를 향해 단방향으로만 흐른다 (의존성 역전 원칙 적용).
- 핵심 비즈니스 로직은 어떠한 외부 기술에도 의존하지 않는 순수 객체 지향 영역인 Domain Layer에 격리한다.

---

### Constraints

- 하위 계층(예: Domain)은 상위 계층(예: Infrastructure)의 존재를 알 수 없다.
- 계층 간 결합은 반드시 Interface를 통해 추상화해야 하며, 상위 레이어에서 직접 하위 레이어의 구현체를 참조해서는 안 된다.
- 외부 데이터 접근(DB 등)은 Repository Interface를 통해서만 수행된다.

---

### Consequences

- Pros: 관심사의 분리(Separation of Concerns)를 통해 핵심 로직 보호, 인프라 변경에 유연, Layer별 독립적인 테스트 용이
- Cons: Interface 선언 및 계층 분리로 인한 초기 보일러플레이트 코드 증가

---

## [ADR 002: Use Case 중심의 Application 설계]

**Status:** Accepted

### Context

- Controller에 비즈니스 로직과 라우팅, 요청 처리 로직이 혼재되어 Controller가 비대해짐
- 애플리케이션이 지원하는 비즈니스 흐름과 기능을 명확하게 파악하기 어려움

---

### Decision

- Application Layer는 Use Case 단위로 구성한다.
- Controller는 외부 요청(HTTP 등)을 DTO로 변환하여 Use Case를 호출하는 역할만 수행하며, 어떠한 비즈니스 로직도 포함하지 않는다.
- Use Case는 단일 트랜잭션 경계를 가지며, Domain Layer를 조율하여 비즈니스 흐름을 오케스트레이션(Orchestration)한다.

---

### Constraints

- Use Case 내부에 복잡한 비즈니스 정책이나 계산 로직을 직접 구현할 수 없다 (반드시 Domain 객체에 위임).
- Controller는 여러 Use Case를 엮어서 호출하거나, 트랜잭션을 관리해서는 안 된다.

---

### Consequences

- Pros: 애플리케이션의 유스케이스를 코드 레벨에서 직관적으로 파악 가능, Controller와 비즈니스 흐름의 책임 분리
- Cons: 단순한 CRUD 작업도 Use Case 클래스를 생성해야 하는 번거로움 발생

---

## [ADR 003: 계층 간 엄격한 데이터 변환 및 격리]

**Status:** Accepted

### Context

- DB의 ORM Entity나 외부 API의 응답 형태가 비즈니스 로직과 화면 응답에까지 결합되는 문제 발생
- API 스펙이나 DB 스키마 변경 시 도메인 로직 전체를 수정해야 하는 사이드 이펙트 우려

---

### Decision

- 계층 간 경계를 넘을 때는 해당 계층의 전용 모델(Request/Response DTO, Domain Model)로 데이터를 변환하여 결합도를 낮춘다.
- Controller와 Application 계층 사이는 DTO를 사용한다.
- Application과 Infrastructure 계층에서는 Domain Model을 사용하며, Infrastructure 계층의 Mapper를 통해 ORM Entity 등과 Domain Model을 변환한다.

---

### Constraints

- 외부 스키마(ORM Entity, API 구조)는 Domain Model이나 Use Case 내부에 직접 노출되어서는 안 된다.
- 응답 시 Domain 객체를 직접 Controller의 Response로 반환할 수 없다 (반드시 Response DTO 변환).

---

### Consequences

- Pros: DB 스키마나 API 스펙 변경이 내부 로직에 미치는 영향을 차단, 캡슐화 강화
- Cons: Mapper 함수 작성 및 DTO 변환 과정에 따른 구현 비용 및 성능(객체 복사) 오버헤드

---

## [ADR 004: Side Effect 격리 및 에러 처리 정책]

**Status:** Accepted

### Context

- 비즈니스 로직 중간에 DB 조회/저장, 외부 API 호출 등 Side Effect가 혼재되어 순수 로직 테스트가 불가능해짐
- 인프라 계층의 기술적 에러(예: HTTP 500, DB Connection 에러)가 비즈니스 로직에 그대로 노출되어 에러 처리 일관성 저하

---

### Decision

- 모든 Side Effect(DB, 파일, 외부 API 통신 등)의 실제 수행은 Infrastructure Layer에 위임한다.
- Domain Layer는 순수 함수처럼 동작하여 직접적인 Side Effect를 발생시키지 않는다.
- Infrastructure 에러는 Application/Domain 계층에서 이해할 수 있는 비즈니스 예외로 변환하여 상위 계층으로 전달한다.

---

### Constraints

- Domain Layer나 Application Layer에서는 외부 시스템에 대한 I/O를 직접 호출할 수 없다 (Interface 호출로 대체).
- Controller는 예외를 직접 비즈니스 로직으로 처리하지 않으며, 전역 공통 에러 핸들러를 통해 일관된 HTTP 상태 코드로 변환해야 한다.

---

### Consequences

- Pros: 핵심 도메인 로직의 단위 테스트(Mock 없는 순수 테스트) 가능, 예측 가능한 에러 응답 구조 마련
- Cons: 에러 변환 로직 및 I/O 격리를 위한 추상화 코드가 추가로 필요

---

## [ADR 005: TDD (Test-Driven Development) 기반 개발 프로세스 도입]

**Status:** Accepted

### Context

- 핵심 비즈니스 로직(Domain, UseCase)의 변경에 따른 사이드 이펙트를 사전에 방지할 필요가 있음
- 코드 작성 후 테스트를 작성하는 방식은 테스트 커버리지 하락과 결합도 높은 코드를 양산함

---

### Decision

- 새로운 기능 개발 및 버그 수정 시 TDD(Test-Driven Development) 프로세스를 엄격히 적용한다.
- 프로덕션 코드 작성 전, 실패하는 단위/통합 테스트를 먼저 작성하여 설계와 명세를 검증한다.
- 핵심 비즈니스 로직(Domain, UseCase 계층)은 철저히 격리된 테스트를 작성한다.

---

### Constraints

- 테스트가 존재하지 않는 핵심 로직(Domain, UseCase)의 코드는 병합을 허용하지 않는다.
- 테스트 작성 시 `docs/rules/TEST_RULE.md`의 명명 규칙과 Mocking 원칙을 반드시 준수해야 한다.
- 테스트는 서로 독립적이어야 하며 상태를 공유할 수 없다.

---

### Consequences

- Pros: 시스템 안정성 및 예측 가능성 극대화, 유연하고 결합도가 낮은 설계 유도, 리팩토링의 안전망 확보
- Cons: 초기 개발 속도 저하, 테스트 코드 유지보수 비용 발생

---
