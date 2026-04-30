## UseCase Rules

AI는 Application Layer(UseCase) 구현 시 아래 규칙을 반드시 준수해야 한다.

---

### 1. UseCase Responsibility

- UseCase는 하나의 명확한 비즈니스 목적만 수행한다
- 여러 책임을 하나의 UseCase에 혼합하지 않는다

- UseCase는 비즈니스 흐름(Orchestration)만 담당한다
- 실제 비즈니스 규칙은 Domain Layer에 위임한다

---

### 2. Business Logic Rules

- 복잡한 비즈니스 로직은 Domain(Entity, Domain Service)으로 이동한다
- UseCase 내부에 정책/규칙/계산 로직 작성 금지

- 단순 흐름 제어(if, try/catch)는 허용되지만,
  비즈니스 의미를 가지는 조건문은 Domain으로 이동해야 한다

---

### 3. Dependency Rules

- UseCase는 Domain Layer와 Interface(Repository, External Port)에만 의존한다

- Infrastructure 구현체를 직접 import 금지
- ORM, 외부 API 클라이언트 직접 사용 금지

---

### 4. Data Flow Rules

- 계층 간 통신 원칙에 따라 입력은 반드시 DTO로 받는다
- **도메인 모델 생성**: Domain Model 생성은 Application Layer (UseCase 또는 연관된 Mapper) 내부에서만 수행한다. Presentation이나 Infrastructure에서 도메인 모델을 생성하여 전달하는 것을 금지한다.
- 외부 데이터(DB/API)는 반드시 Domain Model로 변환 후 사용한다

---

### 5. Side Effect Rules

- UseCase는 Side Effect를 직접 수행하지 않는다
- 모든 Side Effect(DB, 외부 API)는 Interface를 통해 위임한다

- Side Effect 호출 순서는 명확하고 예측 가능해야 한다

---

### 6. Transaction Rules

- 트랜잭션은 UseCase 단위에서 관리한다
- 하나의 UseCase는 하나의 트랜잭션 경계를 가진다
- UseCase 내부에서 트랜잭션을 분산시키지 않는다

**추상화된 트랜잭션 사용 예시:**
```typescript
async execute(command: RentBookCommand): Promise<RentBookResultDto> {
  // UseCase는 추상화된 UnitOfWork를 주입받아 트랜잭션 경계를 관리한다.
  return await this.unitOfWork.runInTransaction(async (tx: TransactionContext) => {
    // 1. Repository에 추상화된 트랜잭션 컨텍스트(tx)를 전달하여 작업 수행
    const bookInfraDto = await this.bookRepository.findById(command.bookId, tx);
    
    // ... 비즈니스 로직 수행 ...
    
    // 2. 저장 작업도 동일한 트랜잭션 컨텍스트 내에서 도메인 객체를 직접 전달
    await this.bookRepository.save(book, tx);
    
    return resultDto;
  });
}
```

---

### 7. Error Handling Rules

- Domain에서 발생한 예외는 그대로 전달하거나 명확히 처리한다

- Infrastructure 에러는 UseCase에서 이해 가능한 형태로 변환한다

- 에러를 무시하거나 흐름 제어용으로 사용 금지

---

### 8. Output Rules

- UseCase는 결과를 반드시 Output DTO로 변환하여 반환한다.
- **도메인 모델 노출 금지**: Domain Model을 그대로 반환하는 것을 엄격히 금지한다.
- **저장소 반환값 처리**: Repository는 도메인 모델을 반환하지 않으므로, Repository의 반환값(Infra DTO)을 반드시 도메인 모델로 변환한 후 다시 Result DTO로 변환하여 반환한다.
- Response 형태로 직접 반환하지 않는다 (HTTP 개념 금지)

---

### 9. Naming Rules

- UseCase 이름은 반드시 동사 기반으로 작성한다
- 파일명은 `[name].usecase.ts` 포맷을 사용한다. (예: `create-user.usecase.ts`)

예:
- CreateUserUseCase
- UpdateOrderStatusUseCase
- CancelPaymentUseCase

금지:
- UserService
- OrderManager
- ProcessData

---

### 10. Prohibited Patterns

- UseCase에서 비즈니스 로직 구현 금지
- Repository 구현체 직접 사용 금지
- ORM/DB 직접 접근 금지
- 외부 API 직접 호출 금지
- Domain 없이 raw 데이터 처리 금지