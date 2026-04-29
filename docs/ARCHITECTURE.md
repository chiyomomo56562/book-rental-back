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

## 3. Layered Architecture Policy

### 3.1 Domain Layer (src/domain)
- 핵심 비즈니스 로직의 중심지이며 어떠한 외부 기술이나 프레임워크에도 의존하지 않는 순수 객체 지향 영역입니다.
- 세부 구현 규칙은 `docs/rules/DOMAIN_RULE.md`를 참조하십시오.

### 3.2 Application Layer (src/application)
- Domain Layer를 조율하여 비즈니스 유스케이스를 달성하는 계층입니다.
- 세부 구현 규칙은 `docs/rules/USECASE_RULE.md`를 참조하십시오.

### 3.3 Infrastructure Layer (src/infrastructure)
- 외부 시스템(DB, API, 파일시스템 등)과의 상호작용 및 추상화된 포트의 세부 구현을 담당합니다.
- 세부 구현 규칙은 `docs/rules/INFRASTRUCTURE_RULE.md`를 참조하십시오.

### 3.4 core/ (src/core)
- 애플리케이션 전반에서 사용되는 단일 책임 기반의 기술적 공통 요소를 포함합니다 (config, logging, security).

### 3.5 shared/ (src/shared)
- 범용적으로 재사용 가능한 최소 단위(공통 타입, 순수 유틸 함수)만 포함하며, 비즈니스 로직을 포함하지 않습니다.
## 4. Data Flow

### 4.1 Request Flow

- 모든 요청은 다음 흐름을 따릅니다:

  Client → Controller → Application (UseCase) → Domain → Infrastructure → Application → Controller → Client

---

### 4.2 Flow Policy

- 계층 간 통신은 Controller → Application → Domain → Infrastructure의 단방향 흐름을 엄격히 준수합니다.
- 각 계층 간 호출 및 데이터 처리에 대한 세부 규칙은 `docs/rules/USECASE_RULE.md` 및 `docs/rules/API_RULE.md`를 참조하십시오.

---

### 4.3 Data Transformation Policy

- 모든 계층 간 데이터 통신은 반드시 전용 DTO를 통해서만 이루어져야 합니다.
- 계층 간 결합도를 최소화하기 위해 내부 도메인 모델이나 외부 스키마(DB, API)가 다른 계층에 직접 노출되지 않도록 철저히 차단합니다.
- 데이터 매핑 및 DTO 변환에 대한 구체적인 규칙은 `docs/rules/MAPPER_RULE.md`를 참조하십시오.

---

### 4.4 Transaction Boundary

- 트랜잭션은 Application Layer (UseCase) 단위에서 관리됩니다.
- 하나의 UseCase는 하나의 트랜잭션 경계를 가집니다.

---

### 4.5 Constraints

- Controller는 Domain Layer를 직접 호출할 수 없습니다.
- Domain Layer는 Infrastructure Layer를 알 수 없습니다.
- Application Layer는 Infrastructure 구현체에 직접 의존할 수 없습니다.
## 5. Dependency Policy

- 모든 의존성은 내부(Domain)를 향해 단방향으로만 흐릅니다 (Dependency Inversion Principle).
- 하위 계층은 상위 계층의 존재를 알 수 없으며, 계층 간 결합은 Interface를 통해 추상화합니다.
- 레이어 의존성에 관한 세부 규칙은 각 Layer별 규칙 문서(`docs/rules/`) 및 `docs/AI_RULES.md`를 참조하십시오.

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

- 테스트는 계층(Layer) 단위로 분리하여 독립적으로 검증 가능해야 합니다.
- 핵심 비즈니스 로직(Domain, UseCase)은 예외 없이 철저한 테스트 범위를 가져야 합니다.
- 테스트 스코프, Mocking 원칙 및 명명 규칙 등 세부적인 테스트 작성 지침은 `docs/rules/TEST_RULE.md`를 엄격히 따릅니다.

## 9. Detailed Implementation Rules & Templates

구체적인 네이밍 컨벤션, 코드 패턴 및 템플릿은 아래의 세부 규칙 문서를 참조하십시오.