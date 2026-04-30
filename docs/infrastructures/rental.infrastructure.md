# Rental 인프라 명세

> **참고 규칙**: `docs/rules/INFRASTRUCTURE_RULE.md`
> **아키텍처 흐름**: `docs/PATTERN.md`

이 문서는 Rental 도메인의 Infrastructure Layer 구현 상세 명세입니다.

---

## 📁 디렉토리 구조

```
src/infrastructure/
├── repositories/
│   └── rental-history.repository.ts # Repository 구현체
├── mappers/
│   └── rental-history.mapper.ts     # Infra Mapper (Entity ↔ Infra DTO)
└── dtos/
    └── rental-history.infra-dto.ts  # Infra DTO 정의
```

---

## 1. Infra DTO: `RentalHistoryInfraDto`
- **목적**: 대여 이력 정보를 담아 전달하는 객체

```typescript
export class RentalHistoryInfraDto {
  constructor(
    public readonly id: string,
    public readonly bookId: string,
    public readonly rentedAt: Date,
    public readonly returnedAt: Date | null,
  ) {}
}
```

---

## 2. Infra Mapper: `InfraRentalHistoryMapper`
- **목적**: ORM 데이터를 `RentalHistoryInfraDto`로 변환

| 메서드 | 입력 | 출력 | 설명 |
|---|---|---|---|
| `toInfraDto` | `RentalPrismaEntity` | `RentalHistoryInfraDto` | 순수 필드 매핑 |

---

## 3. Repository 구현체: `RentalHistoryRepositoryImpl`
- **인터페이스**: `RentalHistoryRepository` (Domain Layer) 구현

### 주요 메서드 구현 로직
- `findByBookId(bookId, tx?)`: 특정 도서의 모든 이력을 최신순으로 조회.
- `findLatestByBookId(bookId, tx?)`: `returnedAt`이 null인 가장 최신 데이터를 조회하여 반환.
- `save(history, tx?)`: `history.bookId`, `history.rentedAt`, `history.getReturnedAt()` 등을 사용하여 저장.

---

## ⚠️ 금지 패턴 체크리스트
- [ ] 저장소 외부로 도메인 모델을 직접 노출하는가? (No)
- [ ] 트랜잭션(`tx`) 전달 시 기본 클라이언트를 사용하는가? (No)
- [ ] Mapper에서 날짜 계산 로직을 수행하는가? (No)
