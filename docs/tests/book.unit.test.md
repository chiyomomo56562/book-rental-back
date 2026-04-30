# Book 단위 테스트 명세

> **참고 규칙**: `docs/rules/TEST_RULE.md`
> **템플릿**: `docs/tests/test.template.md`

이 문서는 Book 도메인의 Entity 및 UseCase 단위 테스트 명세입니다.

---

## 1. Domain Entity Test: `Book`
- **위치**: `src/domain/models/book.model.test.ts` (또는 `book.entity.test.ts`)

### ✅ Happy Path
| 테스트 케이스 | 조건 | 예상 결과 |
|---|---|---|
| `should_update_title_when_valid_string` | 유효한 문자열 제목 입력 | 제목이 성공적으로 변경됨 |
| `should_change_status_to_RENTED_when_rent` | `AVAILABLE` 상태에서 `rent()` 호출 | 상태가 `RENTED`로 변경됨 |
| `should_change_status_to_AVAILABLE_when_return` | `RENTED` 상태에서 `return()` 호출 | 상태가 `AVAILABLE`로 변경됨 |

### ❌ Failure Case
| 테스트 케이스 | 조건 | 예상 결과 |
|---|---|---|
| `should_throw_when_title_is_empty` | 빈 문자열 제목 입력 | `BookDomainException` 발생 |
| `should_throw_when_renting_already_rented_book` | `RENTED` 상태에서 `rent()` 호출 | `BookDomainException` 발생 |
| `should_throw_when_returning_already_available_book` | `AVAILABLE` 상태에서 `return()` 호출 | `BookDomainException` 발생 |

---

## 2. UseCase Test: `UpdateBookTitleUseCase`
- **위치**: `src/application/usecases/update-book-title.usecase.test.ts`

### ✅ Happy Path
- `should_update_book_title_and_save`:
  - **Arrange**: Repository가 특정 도서를 반납하도록 설정.
  - **Act**: 유스케이스 실행.
  - **Assert**: Repository의 `save`가 호출되었는지, 결과 DTO의 제목이 변경되었는지 확인.

### ❌ Failure Case
- `should_throw_when_book_not_found`:
  - **Arrange**: Repository가 `null`을 반환하도록 설정.
  - **Act**: 유스케이스 실행.
  - **Assert**: 예외 발생 확인 및 `save`가 호출되지 않았음을 검증.

---

## ⚠️ 테스트 준수 사항
- Domain 객체는 Mocking 하지 않고 실제 객체를 사용한다.
- UseCase 테스트 시 `BookRepository` 인터페이스만 Mocking 한다.
- 각 테스트 후 `vi.clearAllMocks()`를 호출하여 상태를 격리한다.
