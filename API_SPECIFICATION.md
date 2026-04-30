# API Specification

이 문서는 도서 대여 시스템의 백엔드 API 명세서입니다. 모든 API는 `docs/rules/PRESENTATION_RULE.md`에 정의된 계층형 아키텍처 및 DTO 정책을 준수해야 합니다.

## 1. 공통 사항

### Base URL
- 모든 API의 기본 경로는 `/api`입니다.

### 공통 응답 포맷 (Standard Response)
모든 API 응답은 `PRESENTATION_RULE.md`에 정의된 `ApiResponse` 구조를 따릅니다.

```typescript
interface ApiResponse<T> {
  status: number;         // HTTP 상태 코드
  data: T | null;         // 실제 데이터 (성공 시)
  error: ApiError | null; // 에러 정보 (성공 시 null)
}

interface ApiError {
  message: string;        // 사용자 에러 메시지
  code: string;           // 에러 코드 (예: 'NOT_FOUND')
  status: number;         // HTTP 상태 코드
}
```

---

## 2. 도서 관리 (Book Management)

### [GET] 도서 목록 조회
- **Endpoint**: `/books`
- **Controller Method**: `getBooks()`
- **Response `data`**: `BookResponseDto[]`

### [POST] 도서 등록
- **Endpoint**: `/books`
- **Controller Method**: `createBook()`
- **Request Body**:
  ```json
  { "title": "도서 제목" }
  ```
- **Response `data`**: `BookResponseDto`

### [GET] 도서 상세 조회
- **Endpoint**: `/books/:id`
- **Controller Method**: `getBookById()`
- **Response `data`**: `BookResponseDto`

### [PATCH] 도서 제목 수정
- **Endpoint**: `/books/:id/title`
- **Controller Method**: `updateBookTitle()`
- **Request Body**:
  ```json
  { "title": "변경할 도서 제목" }
  ```
- **Response `data`**: `boolean` (성공 여부)

### [DELETE] 도서 삭제
- **Endpoint**: `/books/:id`
- **Controller Method**: `deleteBook()`
- **Response `data`**: `boolean` (성공 여부)

---

## 3. 대여 관리 (Rental Management)

### [GET] 대여 이력 조회
- **Endpoint**: `/books/:id/rentals`
- **Controller Method**: `getRentalHistoriesByBookId()`
- **Response `data`**: `RentalHistoryResponseDto[]`

### [POST] 도서 대여 신청
- **Endpoint**: `/books/:id/rentals`
- **Controller Method**: `rentBook()`
- **Response `data`**: `boolean` (성공 여부)
- **Note**: 성공 시 도서의 상태가 `RENTED`로 변경됩니다. (UseCase 트랜잭션 보장)

### [PATCH] 도서 반납 신청
- **Endpoint**: `/books/:id/rentals/return`
- **Controller Method**: `returnBook()`
- **Response `data`**: `boolean` (성공 여부)
- **Note**: 성공 시 도서의 상태가 `AVAILABLE`로 변경됩니다. (UseCase 트랜잭션 보장)

---

## 4. 데이터 모델 (DTOs)

### BookResponseDto
```typescript
interface BookResponseDto {
  id: string;          // 도서 고유 ID
  title: string;       // 도서 제목
  status: 'AVAILABLE' | 'RENTED'; // 대여 상태
}
```

### RentalHistoryResponseDto
```typescript
interface RentalHistoryResponseDto {
  id: string;          // 이력 고유 ID
  rentedAt: string;    // 대여 일시 (ISO 8601)
  returnedAt: string | null; // 반납 일시 (반납 전이면 null)
}
```

---

## 5. 에러 처리 가이드

- **400 Bad Request**: 잘못된 요청 (예: 이미 대여 중인 도서 대여 시도) - `ALREADY_RENTED`
- **404 Not Found**: 존재하지 않는 리소스 (예: 잘못된 도서 ID) - `NOT_FOUND`
- **500 Internal Server Error**: 서버 내부 오류 - `INTERNAL_SERVER_ERROR`
