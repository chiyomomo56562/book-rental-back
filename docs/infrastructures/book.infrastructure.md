# Book 인프라 명세

> **참고 규칙**: `docs/rules/INFRASTRUCTURE_RULE.md`
> **아키텍처 흐름**: `docs/PATTERN.md`

이 문서는 Book 도메인의 Infrastructure Layer 구현 상세 명세입니다.

---

## 📁 디렉토리 구조

```
src/infrastructure/
├── repositories/
│   └── book.repository.ts          # Repository 구현체
├── mappers/
│   └── book.mapper.ts              # Infra Mapper (Entity ↔ Infra DTO)
└── dtos/
    └── book.infra-dto.ts           # Infra DTO 정의
```

---

## 1. Infra DTO: `BookInfraDto`
- **목적**: 저장소 계층에서 외부(Application Layer)로 반환하는 표준 데이터 객체

```typescript
export class BookInfraDto {
  constructor(
    public readonly id: string,
    public readonly title: string,
    public readonly status: string, // 'AVAILABLE' | 'RENTED'
  ) {}
}
```

---

## 2. Infra Mapper: `InfraBookMapper`
- **목적**: ORM 엔티티(Prisma 등)를 `BookInfraDto`로 변환

| 메서드 | 입력 | 출력 | 설명 |
|---|---|---|---|
| `toInfraDto` | `BookPrismaEntity` | `BookInfraDto` | 순수 필드 매핑만 수행 |

---

## 3. Repository 구현체: `BookRepositoryImpl`
- **인터페이스**: `BookRepository` (Domain Layer) 구현

### 주요 메서드 구현 로직
- `findById(id, tx?)`: `prisma.book.findUnique` 호출 후 Mapper를 통해 DTO 반환.
- `save(book, tx?)`: `book.getTitle()`, `book.getStatus()`를 사용하여 `upsert` 수행.
- `findAll(tx?)`: `prisma.book.findMany` 호출 후 목록을 DTO로 변환하여 반환.

---

## ⚠️ 금지 패턴 체크리스트
- [ ] Repository가 Prisma Entity를 직접 반환하는가? (No)
- [ ] `save()`에서 Domain Model 대신 별도 DTO를 받는가? (No)
- [ ] Mapper에 비즈니스 조건문이 포함되어 있는가? (No)
