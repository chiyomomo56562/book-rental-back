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
- 계층 간 데이터 통신(예: Controller ↔ UseCase) 시 반드시 DTO를 사용하여 결합도를 낮춘다

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



### 8. Standard Response Structure

모든 API 응답은 프론트엔드 규격과의 호환성을 유지하기 위해 아래의 표준 래퍼 형식을 따릅니다.

```typescript
export type ApiError = {
  message: string;
  code: string;
  status: number;
};

export type ApiResponse<T> = {
  status: number;
  data: T | null;
  error: ApiError | null;
};
```

**응답 규칙:**
- **status**: HTTP Status Code를 사용합니다.
- **성공 (200~299)**: `error`는 반드시 `null`이어야 하며, `data`에 유효한 응답 페이로드가 포함되어야 합니다.
- **실패 (400~599)**: `data`는 `null`이어야 하며, 공통 에러 핸들러를 통해 `error` 객체에 상세 정보(`message`, `code`, `status`)가 포함되어야 합니다.

---

### 9. Naming Rules

- **Endpoint (URI)**: RESTful 원칙을 준수합니다.
  - 리소스는 명사의 복수형을 사용하며, 단어 구분은 kebab-case를 사용합니다. (예: `POST /api/books`, `GET /api/rental-histories/:id`)
- **Controller Method**: 수행하는 행위(동사)를 명확히 표현합니다.
  - 예: `createBook()`, `getRentalHistoryById()`
- **DTO Naming**: `[Action][Entity]RequestDto` 및 `[Action][Entity]ResponseDto` 형식을 사용합니다.
  - 예: `CreateBookRequestDto`, `GetBookResponseDto`