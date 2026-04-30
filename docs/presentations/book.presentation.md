# Book 프레젠테이션 명세

> **참고 규칙**: `docs/rules/PRESENTATION_RULE.md`
> **아키텍처 흐름**: `docs/PATTERN.md`

이 문서는 Book 도메인의 Presentation Layer(Controller, DTO, Mapper) 구현 상세 명세입니다.

---

## 📁 디렉토리 구조

```
src/presentation/
├── controllers/
│   └── book.controller.ts
├── dtos/
│   ├── request/
│   │   ├── create-book.request.dto.ts
│   │   └── update-book-title.request.dto.ts
│   └── response/
│       ├── book.response.dto.ts
│       └── book-list.response.dto.ts
└── mappers/
    └── book.mapper.ts
```

---

## 1. Request DTO 명세

### 1.1 `CreateBookRequestDto`
- `title`: string (NotEmpty)

### 1.2 `UpdateBookTitleRequestDto`
- `title`: string (NotEmpty)

---

## 2. Response DTO 명세

### 2.1 `BookResponseDto`
- `id`: string
- `title`: string
- `status`: 'AVAILABLE' | 'RENTED'

---

## 3. Presentation Mapper: `PresentationBookMapper`
- `toCreateCommand(req)`: `CreateBookRequestDto` -> `CreateBookCommand`
- `toUpdateTitleCommand(id, req)`: `id`, `UpdateBookTitleRequestDto` -> `UpdateBookTitleCommand`
- `toResponse(result)`: `BookResultDto` -> `BookResponseDto`

---

## 4. Controller: `BookController`

| Method | Endpoint | UseCase |
|---|---|---|
| `getBooks` | `GET /api/books` | `GetBooksUseCase` |
| `createBook` | `POST /api/books` | `CreateBookUseCase` |
| `getBookById` | `GET /api/books/:id` | `GetBookByIdUseCase` |
| `updateBookTitle` | `PATCH /api/books/:id/title` | `UpdateBookTitleUseCase` |
| `deleteBook` | `DELETE /api/books/:id` | `DeleteBookUseCase` |

---

## ⚠️ 금지 패턴 체크리스트
- [ ] Controller에서 `if (!req.body.title)`와 같은 로직을 수행하는가? (No, DTO Validation 사용)
- [ ] UseCase의 결과를 그대로 반환하는가? (No, `PresentationBookMapper`를 통해 변환)
- [ ] `ApiResponse<T>` 형식을 준수하는가? (Yes)
