# 통합 테스트 명세 (Integration Test)

> **참고 규칙**: `docs/rules/TEST_RULE.md`
> **템플릿**: `docs/tests/test.template.md`

이 문서는 Controller부터 Repository까지의 연동 흐름을 검증하는 통합 테스트 명세입니다.

---

## 📁 테스트 위치
- `src/__integration__/[name].integration.test.ts`

---

## 1. Book API 통합 테스트
### ✅ Happy Path
- `GET /api/books`: 저장된 도서 목록을 `ApiResponse` 형식으로 반환하는지 확인.
- `POST /api/books`: 도서를 성공적으로 생성하고 DB에 저장되는지 확인.

### ❌ Failure Case
- `GET /api/books/:id`: 존재하지 않는 ID 요청 시 `404 Not Found`와 표준 에러 응답 반환 확인.

---

## 2. Rental API 통합 테스트
### ✅ Happy Path
- `POST /api/books/:id/rentals`: 도서 대여 시 도서 상태가 `RENTED`로 변경되고 대여 이력이 생성되는지 통합 검증.
- `PATCH /api/books/:id/rentals/return`: 반납 시 도서 상태가 `AVAILABLE`로 변경되고 이력에 반납일이 기록되는지 확인.

---

## 🛠 테스트 환경 설정
- **DB**: 인메모리 SQLite 또는 `pg-mem`을 사용하여 각 테스트 간 데이터 격리.
- **초기화**: `beforeEach`에서 테이블을 `TRUNCATE` 하거나 데이터를 초기화한다.
- **검증**: `supertest`를 사용하여 HTTP 응답 코드 및 바디 구조를 검증한다.

---

## ⚠️ 체크리스트
- [ ] 실제 운영 DB가 아닌 테스트용 격리 DB를 사용하는가?
- [ ] `ApiResponse` (status, data, error) 구조를 정확히 따르는가?
- [ ] 테스트 완료 후 DB 연결을 종료하는가?
