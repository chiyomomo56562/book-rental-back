## API Rules

AI는 API Layer 구현 시 아래 규칙을 반드시 준수해야 한다.

---

### 1. Controller Rules

- Controller는 UseCase 호출만 수행한다.
- 비즈니스 로직(조건문, 계산, 정책 판단) 작성 금지
- Repository 및 외부 API 직접 호출 금지
- Controller는 요청(Request)과 응답(Response) 변환 책임만 가진다

- 하나의 Controller 메서드는 하나의 UseCase만 호출해야 한다
- Controller에서 트랜잭션 처리 금지

---

### 2. DTO Rules

- 모든 요청(Request)과 응답(Response)은 반드시 DTO로 정의한다
- DTO 없이 raw object 사용 금지
- DTO는 API Layer에만 위치해야 한다 (다른 Layer로 전파 금지)

- Request DTO와 Response DTO는 명확히 분리한다
- Domain Model을 그대로 Response로 반환 금지

---

### 3. Request Handling Rules

- Controller는 Request를 반드시 Request DTO로 변환 후 UseCase에 전달한다
- Validation은 DTO 레벨에서 수행한다

- Request 객체(req, body 등)를 그대로 UseCase에 전달 금지
- 필요한 데이터만 추출하여 전달한다

---

### 4. Response Handling Rules

- UseCase 결과는 Response DTO로 변환 후 반환한다
- Domain 객체를 그대로 Response로 노출 금지

- 응답 포맷은 일관된 구조를 유지해야 한다
- null / undefined 필드는 명확히 처리한다

---

### 5. Dependency Rules (API Layer)

- Controller는 Application Layer(UseCase)만 의존할 수 있다
- Domain, Infrastructure Layer 직접 접근 금지

---

### 6. Error Handling Rules

- Controller는 예외를 직접 처리하지 않는다 (비즈니스 판단 금지)
- 발생한 예외는 공통 에러 핸들러로 전달한다

- 에러를 catch 후 무시하거나 변형 없이 재throw 금지
- HTTP 상태 코드와 에러 메시지는 일관된 정책을 따른다

---

### 7. Prohibited Patterns

- Controller에서 비즈니스 로직 작성 금지
- Controller에서 Repository 호출 금지
- DTO 없이 raw request/response 사용 금지
- Domain 객체를 그대로 반환하는 코드 생성 금지
- Request 객체를 그대로 UseCase로 전달 금지



### standard response, request structure 추가 필요

### naming rule 추가 필요