## Infrastructure Rules

AI는 Infrastructure Layer 구현 시 아래 규칙을 반드시 준수해야 한다.

---

### 1. Core Responsibility

- Infrastructure Layer는 외부 시스템과의 상호작용만 담당한다

포함:
- DB 접근 (ORM, Query)
- 외부 API 호출
- 파일 시스템
- 캐시, 메시지 큐 등

- 비즈니스 로직을 포함하지 않는다

---

### 2. Side Effect Rules

- 모든 Side Effect(DB, 외부 API 등)는 Infrastructure Layer에서만 발생해야 한다

- UseCase, Domain에서 직접 Side Effect 수행 금지
- 모든 I/O는 반드시 Infrastructure를 통해서만 이루어져야 한다

---

### 3. Repository Implementation Rules

- Repository Interface의 구현체는 Infrastructure Layer에 위치해야 한다

- ORM (Prisma, TypeORM 등)은 Repository 구현 내부에서만 사용한다
- UseCase에서 ORM 직접 사용 금지

- Repository는 결과를 반드시 DTO로 변환하여 반환해야 한다. **어떠한 경우에도 Domain Model을 직접 반환해서는 안 된다.**
- **Save DTO 금지**: 저장 시 별도의 DTO를 생성하지 않고 Domain Model을 직접 인자로 받는다.
- **엔티티 변환 책임**: `Domain -> Entity` 변환은 Repository `save()` 메서드 내부에서만 수행한다. 매퍼가 도메인을 엔티티로 변환하도록 책임을 넘기는 것을 금지한다.

---

### 4. Transaction Handling Rules

- Repository 메서드는 선택적 매개변수로 `tx: TransactionContext`를 받을 수 있어야 한다.
- **트랜잭션 강제**: `tx`가 제공된 경우 **반드시** 해당 트랜잭션 컨텍스트를 사용하여 작업을 수행해야 한다.
- **Fallback 금지**: 트랜잭션 컨텍스트가 전달되었음에도 기본 클라이언트로 암묵적인 fallback을 수행하는 것을 엄격히 금지한다.
- **혼용 금지**: 하나의 작업 단위 내에서 트랜잭션 클라이언트와 기본 클라이언트를 혼용하여 사용하는 것을 금지한다.

**구현 예시 (Prisma):**
```typescript
async save(book: Book, tx?: TransactionContext): Promise<BookInfraDto> {
  // tx가 있으면 반드시 사용, 없으면 기본 클라이언트 사용 (명시적 분기)
  const client = tx ? (tx as PrismaTransactionClient) : this.prisma;
  
  // 도메인 모델의 상태를 인프라 엔티티에 직접 매핑하여 저장 (Repository 내부 책임)
  const savedEntity = await client.book.update({
    where: { id: book.id },
    data: {
      status: book.getStatus(),
      renterId: book.getRenterId()
    }
  });
  
  return InfraBookMapper.toInfraDto(savedEntity);
}
```

---

### 5. External API Rules

- 외부 API 호출(fetch, axios 등)은 Infrastructure Layer에서만 수행한다

- UseCase 또는 Domain에서 외부 API 직접 호출 금지
- API 응답은 반드시 DTO로 변환 후 반환한다

---

### 6. Mapper Usage Rules

- DB 결과(Entity) 및 외부 API 응답은 반드시 Mapper를 통해 DTO로 변환한다
- 외부 API 응답도 Mapper를 통해 변환한다

- 변환 없이 raw 데이터 그대로 반환 금지

---

### 7. Dependency Rules

- Infrastructure Layer는 Domain 및 Application Layer의 Interface를 구현한다

- Infrastructure → Domain 참조 허용 (Interface, Model)
- Infrastructure → Application 참조는 Interface 수준에서만 허용

- 상위 Layer(UseCase, Controller)를 호출하는 코드 작성 금지

---

### 8. Error Handling Rules

- 외부 시스템에서 발생한 에러는 그대로 노출하지 않는다

- DB/네트워크 에러는 Application Layer에서 처리 가능한 형태로 변환한다
- 기술적 에러를 비즈니스 에러와 혼합하지 않는다

---

### 9. Configuration Rules

- 환경 설정(DB 연결, API URL 등)은 Infrastructure에서 관리한다

- 환경 변수 접근은 중앙 config를 통해 수행한다
- 하드코딩 금지

---

### 10. Logging Rules

- 주요 Side Effect(DB, API 호출)는 로깅 대상이다

- 요청/응답 또는 주요 상태를 기록한다
- 민감 정보(비밀번호, 토큰 등)는 로그에 포함하지 않는다

---

### 11. Prohibited Patterns

- Infrastructure에 비즈니스 로직 작성 금지
- ORM 결과를 그대로 반환 금지
- Mapper 없이 데이터 전달 금지
- UseCase / Controller 호출 금지
- 외부 API 응답을 그대로 반환 금지

---

### 12. Naming Rules

- **Repository**: 파일명 `[name].repository.ts`, 클래스명 `[Name]Repository` (예: `book.repository.ts`)
- **External API Client**: 파일명 `[name].client.ts` 또는 `[name].api.ts`
- **Mapper**: 파일명 `[name].mapper.ts` (예: `book.mapper.ts`)