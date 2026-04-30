## Mapper Rules

AI는 데이터 변환(Mapper) 구현 시 아래 규칙을 반드시 준수해야 한다.

---

### 1. Core Responsibility

- Mapper는 서로 다른 Layer의 데이터 모델을 변환하는 역할만 수행한다

예:
- Presentation Layer: Request DTO ↔ Command/Query DTO, Result DTO ↔ Response DTO
- Application Layer: Infra DTO → Domain Model, Domain Model → Result DTO
- Infrastructure Layer: Entity(DB)/외부 응답 ↔ Infra DTO

- Mapper는 비즈니스 로직을 포함하지 않는다

---

### 2. Mandatory Mapping

- 모든 Layer 간 데이터 전달은 반드시 Mapper를 통해 이루어져야 한다

금지:
- DTO → Domain 매핑 생략하고 DTO를 Domain 로직에 직접 사용
- DB Entity ↔ Domain 간의 직접 변환 (반드시 DTO를 거쳐야 함)
- **Domain → Entity 매핑**: 이 변환은 오직 Repository 내부에서만 수행하며, 매퍼가 담당하지 않는다.
- Domain → Response 변환 없이 반환

---

### 3. Mapper Simplification Policy

- **인라인(Inline) 허용**: 단순한 1:1 필드 매핑이고 변환 로직(타입 변환 등)이 전혀 없는 경우에 한해, Mapper 클래스를 별도로 만들지 않고 UseCase 내부에서 직접 매핑하는 것을 허용한다.
- 단, 이는 생산성 향상을 위한 선택이며, 일관성이 깨지거나 매핑 로직이 복잡해질 경우 반드시 Mapper로 분리해야 한다.

---

### 4. No Direct Exposure

- 외부 데이터(DB, API, Request)를 그대로 다른 Layer로 전달 금지

- DB Entity를 그대로 반환 금지
- API 응답을 그대로 UseCase에 전달 금지
- Request 객체를 그대로 Domain에 전달 금지

---

### 5. Explicit Mapping

- 모든 필드 매핑은 명시적으로 작성한다

금지:
- spread operator (`...`) 남용
- 자동 매핑에 의존

- 필드 이름이 동일하더라도 명시적으로 변환한다

---

### 6. One-Way Responsibility

- Mapper는 변환만 수행하며, 상태를 가지지 않는다

- 로직, 검증, 조건 분기 포함 금지
- Side Effect 발생 금지

---

### 7. Model Isolation

- 각 Layer의 모델은 서로 독립적이어야 한다

- DB 스키마 변경이 Domain에 영향을 주지 않아야 한다
- API 스펙 변경이 Domain에 영향을 주지 않아야 한다

---

### 8. Naming Convention

- Mapper는 대상 및 소속 계층 기준으로 명확하게 정의한다

- 파일명은 `[name].mapper.ts` 포맷을 사용한다. (예: `user.mapper.ts`)
- 메서드 명은 변환 대상 기준으로 명확하게 정의한다

예:
- Presentation Layer: `PresentationUserMapper.toCommand()`, `PresentationUserMapper.toResponse()`
- Application Layer: `AppUserMapper.toDomain()`, `AppUserMapper.toResult()`
- Infrastructure Layer: `InfraUserMapper.toInfraDto()`

> ⚠️ **주의**: `Domain → Entity` 변환(`toEntity()` 등)은 Mapper의 책임이 아니다. 이 변환은 오직 Repository `save()` 메서드 내부에서만 수행하며, Mapper 클래스에 해당 메서드를 노출하지 않는다. (참조: `INFRASTRUCTURE_RULE.md` Section 3)

- 의미 없는 이름 사용 금지

금지:
- map()
- convert()
- transformData()

---

### 9. Data Integrity Rules

- Mapper는 변환 과정에서 데이터 정합성을 보장해야 한다

- null / undefined 값은 명확히 처리한다
- 필요한 기본값 설정 가능

---

### 10. Type Conversion Rules

- 외부 타입은 Domain 타입으로 변환해야 한다

예:
- string → Date
- string → Enum
- number → Value Object

- Domain 내부 표현과 외부 표현을 분리한다

---

### 11. No Partial Mapping

- 일부 필드만 매핑하는 불완전 변환 금지

- Domain Model은 항상 완전한 상태로 생성되어야 한다

---

### 12. Prohibited Patterns

- Mapper에 비즈니스 로직 포함 금지
- Mapper에서 DB/API 호출 금지
- Mapper 없이 데이터 전달 금지
- raw object 그대로 반환 금지