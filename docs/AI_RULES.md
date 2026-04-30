# AI Agent Global Rules

이 문서는 모든 기술적 결정 및 구현 규칙(`docs/rules/*.md`)에 우선하는 **전역 강제 규정**입니다. AI 에이전트는 작업을 시작하기 전 이 규칙을 반드시 숙지해야 합니다.

## 1. Global Strict Constraints

AI는 아래 사항을 절대 위반해서는 안 된다.

- **Type Safety:** `any` 사용 금지. 모든 데이터는 명확한 타입으로 정의한다.

- **No Silent Failure:** 에러를 무시하거나 로그 없이 처리하는 행위 금지

- **No Circular Dependency:** 순환 참조(import cycle) 생성 금지

- **No Raw Data Usage:** 외부 데이터(Request, DB, API)를 그대로 사용하지 않는다

- **Async Consistency:** 모든 비동기 로직은 `async/await` 기반으로 작성한다 (`.then` 금지)

- **No Shared Abuse:** shared 영역에 비즈니스 로직 작성 금지

- **Strict DTO Communication**: 모든 계층 간 통신 시 반드시 전용 DTO를 사용한다. **단, `save()` 경로에 한해 Domain Model을 Infrastructure로 전달하는 것만 예외적으로 허용한다.**
  - Repository는 절대 Domain Model을 반환하지 않는다. (항상 Infra DTO 반환)
  - Controller 레이어는 어떠한 경우에도 Domain Model을 직접 다루거나 생성할 수 없다.

- **Domain Model Lifecycle**: Domain Model 생성은 오직 Application Layer (UseCase/Mapper) 내부에서만 허용된다.

- **Transaction Integrity**: `tx`가 제공된 경우 반드시 사용해야 하며, 암묵적 fallback은 절대 금지한다.

- **Infrastructure Boundary**: `Domain -> Entity` 변환 로직은 오직 Repository `save()` 내부에서만 수행한다.

---

## 2. Quality & Implementation Rule

- **Single Responsibility:** 모든 함수와 모듈은 하나의 책임만 가진다

- **Explicit Naming:** 이름은 역할과 의도를 명확히 드러내야 한다

- **Simplicity First:** 불필요한 추상화 금지, 읽기 쉬운 코드 우선

- **Pure Logic Preference:** 가능한 한 순수 함수 형태로 작성한다

- **Testable Code:** 외부 의존성과 분리된 테스트 가능한 구조로 작성한다

---

## 3. Workflow & Review Rules

- **Decision Documentation**: 아키텍처 결정(계층 분리, 데이터 흐름 정책 등)을 변경하는 경우, 반드시 `docs/ADR.md`에 새로운 ADR 항목을 추가하고 **변경 사유(Context), 결정 내용(Decision), 제약 조건(Constraints), 결과(Consequences)**를 기록해야 한다.

- **Rule Update Notification**: `docs/rules/*.md` 파일을 수정하는 경우, 해당 변경이 다른 규칙 문서나 `PATTERN.md`와 충돌하지 않는지 반드시 교차 검토 후 반영한다.