# Rental 단위 테스트 명세

> **참고 규칙**: `docs/rules/TEST_RULE.md`
> **템플릿**: `docs/tests/test.template.md`

이 문서는 Rental 도메인의 Entity 및 UseCase 단위 테스트 명세입니다.

---

## 1. Domain Entity Test: `RentalHistory`
- **위치**: `src/domain/models/rental-history.model.test.ts`

### ✅ Happy Path
| 테스트 케이스 | 조건 | 예상 결과 |
|---|---|---|
| `should_complete_return_when_valid_date` | 대여일 이후의 날짜로 반납 완료 호출 | `returnedAt`이 성공적으로 기록됨 |

### ❌ Failure Case
| 테스트 케이스 | 조건 | 예상 결과 |
|---|---|---|
| `should_throw_when_return_date_is_before_rent_date` | 대여일보다 과거 날짜로 반납 호출 | `RentalDomainException` 발생 |
| `should_throw_when_already_returned` | 이미 반납된 이력에 대해 다시 반납 호출 | `RentalDomainException` 발생 |

---

## 2. UseCase Test: `RentBookUseCase`
- **위치**: `src/application/usecases/rent-book.usecase.test.ts`

### ✅ Happy Path
- `should_rent_book_successfully`:
  - **Arrange**: `Book`이 `AVAILABLE` 상태로 조회되도록 Mock 설정.
  - **Act**: 대여 유스케이스 실행.
  - **Assert**: `book.rent()`가 실행되어 상태가 변경되고, `BookRepository.save`와 `RentalHistoryRepository.save`가 모두 호출됨.

### ❌ Failure Case
- `should_rollback_when_history_save_fails`:
  - **Arrange**: 도서 저장 후 이력 저장 시 에러 발생하도록 설정.
  - **Act**: 유스케이스 실행.
  - **Assert**: 트랜잭션 롤백 확인 (UnitOfWork 기능 검증).

---

## ⚠️ 테스트 준수 사항
- 복합 유스케이스(`Rent`, `Return`)의 경우 두 가지 Repository가 모두 적절히 호출되는지 검증한다.
- 날짜(Date) 객체는 고정된 값을 사용하여 테스트의 결정성(Determinism)을 보장한다.
