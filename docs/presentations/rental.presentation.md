# Rental 프레젠테이션 명세

> **참고 규칙**: `docs/rules/PRESENTATION_RULE.md`
> **아키텍처 흐름**: `docs/PATTERN.md`

이 문서는 Rental 도메인의 Presentation Layer(Controller, DTO, Mapper) 구현 상세 명세입니다.

---

## 📁 디렉토리 구조

```
src/presentation/
├── controllers/
│   └── rental.controller.ts
├── dtos/
│   ├── request/
│   │   ├── rent-book.request.dto.ts
│   │   └── return-book.request.dto.ts
│   └── response/
│       └── rental-history.response.dto.ts
└── mappers/
    └── rental.mapper.ts
```

---

## 1. Request DTO 명세

### 1.1 `RentBookRequestDto`
- `bookId`: string (NotEmpty, UUID)

### 1.2 `ReturnBookRequestDto`
- `bookId`: string (NotEmpty, UUID)

---

## 2. Response DTO 명세

### 2.1 `RentalHistoryResponseDto`
- `id`: string
- `bookId`: string
- `rentedAt`: string (ISO 8601)
- `returnedAt`: string | null (ISO 8601)

---

## 3. Presentation Mapper: `PresentationRentalMapper`
- `toRentCommand(req)`: `RentBookRequestDto` -> `RentBookCommand`
- `toReturnCommand(req)`: `ReturnBookRequestDto` -> `ReturnBookCommand`
- `toResponse(result)`: `RentalHistoryResultDto` -> `RentalHistoryResponseDto`

---

## 4. Controller: `RentalController`

| Method | Endpoint | UseCase |
|---|---|---|
| `getRentalHistoriesByBookId` | `GET /api/books/:id/rentals` | `GetRentalHistoriesUseCase` |
| `rentBook` | `POST /api/books/:id/rentals` | `RentBookUseCase` |
| `returnBook` | `PATCH /api/books/:id/rentals/return` | `ReturnBookUseCase` |

---

## ⚠️ 금지 패턴 체크리스트
- [ ] Controller에서 직접 트랜잭션을 시작하는가? (No, UseCase 담당)
- [ ] `req.params.id`를 직접 UseCase에 전달하는가? (No, DTO 또는 명시적 Command 필드로 변환)
- [ ] 도메인 모델(RentalHistory)을 클라이언트에 노출하는가? (No, `RentalHistoryResponseDto` 사용)
