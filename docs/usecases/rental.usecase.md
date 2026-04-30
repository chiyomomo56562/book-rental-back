# Rental 유스케이스 명세

> **참고 규칙**: `docs/rules/USECASE_RULE.md`
> **아키텍처 흐름**: `docs/PATTERN.md`

이 문서는 Rental 도메인의 Application Layer(UseCase) 구현 상세 명세입니다.

---

## 📁 디렉토리 구조

```
src/application/
├── usecases/
│   ├── rent-book.usecase.ts
│   ├── return-book.usecase.ts
│   └── get-rental-histories.usecase.ts
├── dtos/
│   ├── commands/
│   │   ├── rent-book.command.ts
│   │   └── return-book.command.ts
│   └── results/
│       └── rental-history.result.dto.ts
└── mappers/
    └── rental.mapper.ts
```

---

## 1. DTO 명세

### Command DTO
- `RentBookCommand`: `bookId` (string)
- `ReturnBookCommand`: `bookId` (string)

### Result DTO: `RentalHistoryResultDto`
- `id` (string), `bookId` (string), `rentedAt` (string/ISO), `returnedAt` (string | null)

---

## 2. UseCase 구현 계획

### 2.1 RentBookUseCase (Write - 복합 도메인)
- **흐름**:
  1. `unitOfWork.runInTransaction` 시작.
  2. `bookRepository.findById(bookId, tx)` 조회.
  3. `book.rent()` 호출 (상태를 `RENTED`로 변경).
  4. `bookRepository.save(book, tx)` 저장.
  5. 새로운 `RentalHistory` 엔티티 생성.
  6. `rentalHistoryRepository.save(history, tx)` 저장.

### 2.2 ReturnBookUseCase (Write - 복합 도메인)
- **흐름**:
  1. `unitOfWork.runInTransaction` 시작.
  2. `bookRepository.findById(bookId, tx)` 조회 및 `book.return()` 호출.
  3. `rentalHistoryRepository.findLatestByBookId(bookId, tx)` 조회.
  4. `history.completeReturn(now)` 호출.
  5. `bookRepository.save` 및 `rentalHistoryRepository.save` 호출.

---

## ⚠️ 금지 패턴 체크리스트
- [ ] 도서 대여 시 `Book` 상태만 바꾸고 `RentalHistory` 생성을 누락했는가? (No, 트랜잭션 내 동시 수행)
- [ ] `new Date()` 같은 비결정적 값을 도메인이 아닌 유스케이스에서 생성하여 넘기는가? (Yes, 현재 시점은 유스케이스가 결정)
- [ ] 예외 발생 시 트랜잭션 롤백이 보장되는가? (Yes, `UnitOfWork` 사용)
