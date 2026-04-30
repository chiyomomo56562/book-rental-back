# Book 유스케이스 명세

> **참고 규칙**: `docs/rules/USECASE_RULE.md`
> **아키텍처 흐름**: `docs/PATTERN.md`

이 문서는 Book 도메인의 Application Layer(UseCase) 구현 상세 명세입니다.

---

## 📁 디렉토리 구조

```
src/application/
├── usecases/
│   ├── create-book.usecase.ts
│   ├── update-book-title.usecase.ts
│   ├── delete-book.usecase.ts
│   ├── get-books.usecase.ts
│   └── get-book-by-id.usecase.ts
├── dtos/
│   ├── commands/
│   │   ├── create-book.command.ts
│   │   └── update-book-title.command.ts
│   ├── queries/
│   │   └── get-book-by-id.query.ts
│   └── results/
│       └── book.result.dto.ts
└── mappers/
    └── book.mapper.ts
```

---

## 1. DTO 명세

### Command/Query DTO
- `CreateBookCommand`: `title` (string)
- `UpdateBookTitleCommand`: `id` (string), `title` (string)
- `GetBookByIdQuery`: `id` (string)

### Result DTO: `BookResultDto`
- `id` (string), `title` (string), `status` (string)

---

## 2. UseCase 구현 계획

### 2.1 CreateBookUseCase (Write)
- **흐름**:
  1. `unitOfWork.runInTransaction` 시작.
  2. 새로운 `Book` 엔티티 생성.
  3. `bookRepository.save(book, tx)` 호출.
  4. 결과를 `BookResultDto`로 변환하여 반환.

### 2.2 UpdateBookTitleUseCase (Write)
- **흐름**:
  1. `bookRepository.findById(id, tx)` 조회.
  2. `AppBookMapper.toDomain(infraDto)` 변환.
  3. `book.updateTitle(newTitle)` 호출 (도메인 로직).
  4. `bookRepository.save(book, tx)` 호출.

### 2.3 GetBooksUseCase (Read)
- **흐름**:
  1. `bookRepository.findAll()` 호출.
  2. `AppBookMapper.toResult(infraDto)`를 통해 목록 변환 및 반환.

---

## ⚠️ 금지 패턴 체크리스트
- [ ] UseCase 내부에서 `if (title === "")` 같은 유효성 검사를 직접 하는가? (No, Domain에 위임)
- [ ] Read 흐름에서 트랜잭션을 사용하는가? (No)
- [ ] Repository에서 받은 `InfraDto`를 그대로 반환하는가? (No, `ResultDto`로 변환)
