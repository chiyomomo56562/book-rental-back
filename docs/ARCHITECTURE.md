# Project Architecture & Policy

이 문서는 프로젝트의 **설계 철학, 계층 간 경계(Policy), 그리고 준수해야 할 법률**을 정의합니다. 모든 세부 구현 지침(Rules)은 이 정책을 기반으로 작성되어야 합니다.

## 1. Core Principles (Policy)

- 1.1 **Layered Architecture**
  - 시스템은 Domain, Application, Infrastructure 계층으로 분리됩니다.
  - 의존성은 Domain → Application → Infrastructure 방향으로만 흐릅니다.

- 1.2 **Dependency Rule**
  - Domain Layer는 외부 기술(DB, Framework, API)에 의존하지 않습니다.
  - 모든 외부 의존성은 Interface를 통해 역전됩니다.

- 1.3 **Use Case Driven Design**
  - Application Layer는 UseCase 단위로 구성됩니다.
  - Controller는 UseCase를 호출만 하며 비즈니스 로직을 포함하지 않습니다.

- 1.4 **Domain-Driven Design**
  - 핵심 비즈니스 로직은 Domain Layer에 위치합니다.
  - Entity, Value Object, Domain Service를 활용하여 모델링합니다.

- 1.5 **Separation of Concerns**
  - 각 계층은 명확한 책임을 가지며 역할이 혼합되지 않습니다.

- 1.6 **Testability**
  - Domain / Application Layer는 순수 로직으로 구성되어야 하며 단위 테스트가 가능해야 합니다.

## 2. Global Folder Structure

```
src/
├── core/                   # 1. 핵심 공통 모듈
│   ├── config/             # 전역 설정 관리 (단일 책임, 환경변수 관리)
│   ├── logging/            # 로깅 정책 및 설정 (단일 책임)
│   └── guards/             # 인증/인가 게이트 (단일 책임)
├── domain/                 # 2. 도메인 계층 (핵심 비즈니스 로직)
├── application/            # 3. 애플리케이션 계층 (Use Cases)
├── infrastructure/         # 4. 인프라스트럭처 계층 (구현 세부사항)
├── shared/                 # 6. 공유 모듈
```

## 3. Layered Architecture (Strict)
### 3.1 Domain Layer (src/domain)
- 순수 비즈니스 로직을 포함합니다.
- 외부 기술(DB, API, Framework)에 의존하지 않습니다.

포함:
- Entity
- Value Object
- Domain Service
- Repository Interface

금지:
- ORM Entity
- HTTP / Controller 관련 코드
- 외부 API 호출
### 3.2 Application Layer (src/application)
- UseCase 단위로 비즈니스 흐름을 정의합니다.
- Domain Layer를 조합하여 실제 동작을 구성합니다.

포함:
- UseCase (Service)
- DTO (입출력 모델)
- Interface (External Port)

금지:
- 비즈니스 규칙 구현 (→ Domain으로 이동)
- DB 직접 접근
### 3.3 Infrastructure Layer (src/infrastructure)
- 외부 시스템과의 통신 및 구현 세부사항을 담당합니다.

포함:
- Repository 구현체
- ORM Entity
- 외부 API Client

금지:
- 비즈니스 로직
- UseCase 로직
### 3.4 core/ (src/core)
- 애플리케이션 전반에서 사용되는 기술적 공통 요소를 포함합니다.

포함:
- config
- logging
- security (guards, auth)

금지:
- 도메인 로직
- 특정 비즈니스 규칙
### 3.5 shared/ (src/shared)
- 범용적으로 재사용 가능한 최소 단위만 포함합니다.

포함:
- 공통 타입
- 순수 유틸 함수

금지:
- Domain 로직
- UseCase 로직
- Repository / Entity
## 4. Data Flow

### 4.1 Request Flow

- 모든 요청은 다음 흐름을 따릅니다:

  Client → Controller → Application (UseCase) → Domain → Infrastructure → Application → Controller → Client

---

### 4.2 Flow Rules

- Controller는 요청을 DTO로 변환한 후 UseCase를 호출합니다.
- Application Layer는 UseCase 단위로 동작하며, Domain Layer를 조합하여 비즈니스 흐름을 구성합니다.
- Domain Layer는 비즈니스 로직을 수행하며, 외부 시스템에 직접 접근하지 않습니다.
- 외부 데이터 접근은 Repository Interface를 통해 요청됩니다.
- Infrastructure Layer는 해당 Interface를 구현하여 DB 또는 외부 API와 통신합니다.
- 응답은 Application Layer에서 DTO로 변환되어 Controller로 전달됩니다.

---

### 4.3 Data Transformation Rules

- Controller ↔ Application 간에는 DTO를 사용합니다.
- Application ↔ Domain 간에는 Domain Model을 사용합니다.
- Infrastructure ↔ Domain 간에는 Mapper를 통해 변환합니다.
- 외부 스키마(DB, API)는 Domain Model에 직접 노출되지 않습니다.

---

### 4.4 Transaction Boundary

- 트랜잭션은 Application Layer (UseCase) 단위에서 관리됩니다.
- 하나의 UseCase는 하나의 트랜잭션 경계를 가집니다.

---

### 4.5 Constraints

- Controller는 Domain Layer를 직접 호출할 수 없습니다.
- Domain Layer는 Infrastructure Layer를 알 수 없습니다.
- Application Layer는 Infrastructure 구현체에 직접 의존할 수 없습니다.
## 5. Dependency Rules

- 모든 의존성은 단방향으로만 흐릅니다.
- Domain Layer는 외부 기술(DB, API, Framework)에 의존하지 않습니다.
- Application Layer는 Domain Layer에만 의존합니다. Infrastructure 구현체를 직접 참조하지 않습니다.
- Infrastructure Layer는 Domain Layer와 Application Layer에 의존합니다. 상위 레이어를 호출하지 않습니다.
- Controller (Interface Layer)는 Application Layer만 의존합니다.
- Interface를 통한 의존성 역전 원칙(DIP)을 준수합니다.
- 레이어 간 직접 구현체 import는 금지하며, Interface를 통해서만 접근합니다.

## 6. Side Effects Policy

- 모든 Side Effect(DB, 외부 API, 파일 시스템 등)는 Infrastructure Layer에서만 수행합니다.

- Domain Layer
  - 순수 함수처럼 동작해야 하며, Side Effect를 발생시키지 않습니다.

- Application Layer
  - Side Effect를 직접 수행하지 않고, Interface를 통해 위임합니다.

- Infrastructure Layer
  - 실제 Side Effect를 수행하는 유일한 레이어입니다.

- 비즈니스 로직은 Side Effect 없이도 테스트 가능해야 합니다.
## 7. Error Handling Policy
- Domain Layer
  - 비즈니스 규칙 위반에 대한 예외를 정의하고 발생시킵니다.

- Application Layer
  - Domain 예외를 처리하거나 그대로 전달합니다.
  - 트랜잭션 경계를 관리합니다.

- Infrastructure Layer
  - 외부 시스템 오류를 Domain/Application에서 이해 가능한 형태로 변환합니다.

- Controller
  - 모든 예외를 HTTP 응답 형태로 변환합니다.

## 8. Testing Policy
### 8.1 Test Strategy

- 테스트는 Layer 단위로 분리하여 작성합니다.
- 각 Layer는 독립적으로 테스트 가능해야 합니다.
### 8.2 Domain Layer Test
- Domain Layer는 단위 테스트(Unit Test)만 수행합니다.
- 외부 의존성 없이 순수 로직만 검증합니다.

포함:
- Entity
- Value Object
- Domain Service

금지:
- DB 접근
- Mock 남용 (순수 객체 테스트 우선)

## 8.3 Application Layer Test
- UseCase 단위의 테스트를 작성합니다.
- Domain은 실제 객체를 사용하고, 외부 의존성은 Mock으로 대체합니다.

포함:
- UseCase 흐름 검증
- 비즈니스 시나리오 테스트

금지:
- 실제 DB / 외부 API 호출

### 8.4 Infrastructure Layer Test
- 실제 구현체에 대한 통합 테스트를 수행합니다.

포함:
- Repository 구현 검증
- 외부 API 연동 테스트

허용:
- Test DB 사용
- 실제 I/O 수행

### 8.5 E2E Test
- 전체 흐름을 검증하는 End-to-End 테스트를 작성합니다.

포함:
- Controller → UseCase → Infrastructure 전체 흐름

### 8.6 Mocking Rules (중요)
- Mock은 Application Layer 이상에서만 사용합니다.
- Domain Layer에서는 Mock을 사용하지 않습니다.
- Infrastructure Layer는 Mock 대신 실제 구현을 테스트합니다.

### 8.7 Coverage & Quality Rules
- 핵심 비즈니스 로직(Domain, UseCase)은 반드시 테스트되어야 합니다.
- 테스트 없는 UseCase는 허용되지 않습니다.

## 9. Detailed Implementation Rules & Templates

구체적인 네이밍 컨벤션, 코드 패턴 및 템플릿은 아래의 세부 규칙 문서를 참조하십시오.