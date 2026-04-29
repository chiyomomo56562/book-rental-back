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

- Repository는 Domain Model을 반환해야 한다 (Entity 반환 금지)

---

### 4. External API Rules

- 외부 API 호출(fetch, axios 등)은 Infrastructure Layer에서만 수행한다

- UseCase 또는 Domain에서 외부 API 직접 호출 금지
- API 응답은 Domain Model로 변환 후 반환한다

---

### 5. Mapper Usage Rules

- DB 결과(Entity)는 반드시 Mapper를 통해 Domain Model로 변환한다
- 외부 API 응답도 Mapper를 통해 변환한다

- 변환 없이 raw 데이터 그대로 반환 금지

---

### 6. Dependency Rules

- Infrastructure Layer는 Domain 및 Application Layer의 Interface를 구현한다

- Infrastructure → Domain 참조 허용 (Interface, Model)
- Infrastructure → Application 참조는 Interface 수준에서만 허용

- 상위 Layer(UseCase, Controller)를 호출하는 코드 작성 금지

---

### 7. Error Handling Rules

- 외부 시스템에서 발생한 에러는 그대로 노출하지 않는다

- DB/네트워크 에러는 Application Layer에서 처리 가능한 형태로 변환한다
- 기술적 에러를 비즈니스 에러와 혼합하지 않는다

---

### 8. Configuration Rules

- 환경 설정(DB 연결, API URL 등)은 Infrastructure에서 관리한다

- 환경 변수 접근은 중앙 config를 통해 수행한다
- 하드코딩 금지

---

### 9. Logging Rules

- 주요 Side Effect(DB, API 호출)는 로깅 대상이다

- 요청/응답 또는 주요 상태를 기록한다
- 민감 정보(비밀번호, 토큰 등)는 로그에 포함하지 않는다

---

### 10. Prohibited Patterns

- Infrastructure에 비즈니스 로직 작성 금지
- ORM 결과를 그대로 반환 금지
- Mapper 없이 데이터 전달 금지
- UseCase / Controller 호출 금지
- 외부 API 응답을 그대로 반환 금지