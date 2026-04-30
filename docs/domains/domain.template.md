# Domain Layer 구현 템플릿

> **참고 규칙**: `docs/rules/DOMAIN_RULE.md`
> **아키텍처 흐름**: `docs/PATTERN.md`

이 문서는 Domain Layer의 **관계도**, **Entity(Model)**, **Value Object**, **Domain Service**, **Repository Interface**, **Domain Exception** 구현 시 사용하는 표준 템플릿입니다.

---

## 📁 디렉토리 구조

```
src/domain/
├── models/
│   └── [name].model.ts             # Entity (Domain Model)
├── value-objects/
│   └── [name].vo.ts                # Value Object
├── services/
│   └── [name].domain-service.ts   # Domain Service
├── repositories/
│   └── [name].repository.interface.ts  # Repository Interface (Port)
├── exceptions/
│   └── [name].exception.ts        # Domain Exception
└── types/
    └── transaction.type.ts        # 트랜잭션 추상화 타입
```

---

## 0. 도메인 모델 관계도 (Relationships)

> **목적**: 도메인 간의 관계(1:1, 1:N, N:M)를 시각화하고 비즈니스적 의존 관계를 정의합니다. (물리적 테이블 관계와 매핑됨)

### 📊 관계 다이어그램
```mermaid
erDiagram
    [ENTITY_A] ||--o{ [ENTITY_B] : "관계를 설명하는 동사 (예: 소유한다)"
    [ENTITY_A] {
        string id PK "설명"
        string field "설명"
    }
    [ENTITY_B] {
        string id PK "설명"
        string entityAId FK "외래키 설명"
    }
```

### 📋 관계 상세 설명
| 소스 도메인 | 타겟 도메인 | 관계 유형 | 설명 |
|---|---|---|---|
| `[EntityA]` | `[EntityB]` | `1:N` | `EntityA` 하나에 여러 `EntityB`가 존재함 |
| `[EntityB]` | `[EntityA]` | `N:1` | `EntityB`는 반드시 하나의 `EntityA`에 속함 |

---

## 1. Entity (Domain Model) 템플릿

> **규칙**: 상태(State) + 행동(Behavior) 포함. 외부에서 직접 상태 변경 금지.
> 비즈니스 규칙 위반 시 Domain Exception 발생. ORM·프레임워크 의존성 금지.

```typescript
// src/domain/models/[name].model.ts
import { [Name]Exception } from '../exceptions/[name].exception';

export enum [Name]Status {
  [STATUS_A] = '[STATUS_A]',
  [STATUS_B] = '[STATUS_B]',
}

export class [Name] {
  private [mutableField]: [Type];

  constructor(
    public readonly id: string,
    public readonly [immutableField]: [Type],
    [mutableField]: [Type],
  ) {
    this.[mutableField] = [mutableField];
  }

  // ✅ 비즈니스 의미를 드러내는 메서드명 사용 (setXxx 금지)
  [businessAction]([param]: [Type]): void {
    if (this.[mutableField] !== [Name]Status.[REQUIRED_STATUS]) {
      throw new [Name]Exception('비즈니스 규칙 위반 메시지');
    }
    this.[mutableField] = [Name]Status.[NEW_STATUS];
  }

  // Getter (상태 노출용)
  get[MutableField](): [Type] { return this.[mutableField]; }
}
```

---

## 2. Value Object 템플릿

> **규칙**: 불변(Immutable). setter 금지. 동일 값 = 동일 객체. 생성 시점에 모든 값 결정.

```typescript
// src/domain/value-objects/[name].vo.ts
export class [Name]Vo {
  public readonly [field]: [Type];

  private constructor([field]: [Type]) {
    // 생성 시점 불변 조건 검증
    if (![validationCondition]) {
      throw new Error('[Name] 유효성 검증 실패: ...');
    }
    this.[field] = [field];
  }

  static of([field]: [Type]): [Name]Vo {
    return new [Name]Vo([field]);
  }

  equals(other: [Name]Vo): boolean {
    return this.[field] === other.[field];
  }
}
```

---

## 3. Repository Interface (Port) 템플릿

> **목적**: Infrastructure의 구현체에서 이 인터페이스를 구현. UseCase는 이 인터페이스에만 의존.
> 반환 타입은 반드시 Infra DTO (Domain Model 직접 반환 금지).

```typescript
// src/domain/repositories/[name].repository.interface.ts
import { [Name]InfraDto } from '../../infrastructure/dtos/[name].infra-dto';
import { [Name] } from '../models/[name].model';
import { TransactionContext } from '../types/transaction.type';

export interface [Name]Repository {
  findById(id: string, tx?: TransactionContext): Promise<[Name]InfraDto | null>;
  findAll(tx?: TransactionContext): Promise<[Name]InfraDto[]>;
  /** save()는 Domain Model을 직접 인자로 받음 (Save DTO 금지) */
  save([name]: [Name], tx?: TransactionContext): Promise<[Name]InfraDto>;
  delete(id: string, tx?: TransactionContext): Promise<void>;
}
```

---

## 4. Domain Exception 템플릿

> **목적**: 비즈니스 규칙 위반 시 발생. 의미 있는 에러 메시지 포함. generic Error 사용 금지.

```typescript
// src/domain/exceptions/[name].exception.ts
export class [Name]DomainException extends Error {
  constructor(message: string) {
    super(message);
    this.name = '[Name]DomainException';
  }
}
```

---

## 5. Domain Service 템플릿

> **목적**: 단일 Entity로 표현하기 어려운 도메인 로직. Stateless. 여러 Entity 조합.

```typescript
// src/domain/services/[name].domain-service.ts
export class [Name]DomainService {
  /**
   * 여러 Entity를 조합한 비즈니스 규칙
   * - 외부 I/O 금지, 순수 도메인 로직만 포함
   */
  [businessMethod]([entity1]: [Entity1], [entity2]: [Entity2]): [ReturnType] {
    // 순수 비즈니스 로직
  }
}
```

---

## 6. 트랜잭션 추상화 타입

```typescript
// src/domain/types/transaction.type.ts
export type TransactionContext = unknown; // 구현체(PrismaClient 등)에서 타입 캐스팅

export interface UnitOfWork {
  runInTransaction<T>(work: (tx: TransactionContext) => Promise<T>): Promise<T>;
}
```

---

## ⚠️ 금지 패턴 체크리스트

| 항목 | 확인 |
|------|------|
| Domain Model에 ORM(Prisma, TypeORM) import가 없는가? | ☐ |
| Domain Model에 Framework(Express, NestJS) 의존성이 없는가? | ☐ |
| 상태 변경이 반드시 내부 메서드를 통해서만 이루어지는가? | ☐ |
| setter가 없고 비즈니스 의미 있는 메서드명을 사용하는가? | ☐ |
| 비즈니스 규칙 위반 시 Domain Exception을 발생시키는가? | ☐ |
| Value Object가 불변 객체로 구현되었는가? | ☐ |
| Repository Interface 반환 타입이 Infra DTO인가? | ☐ |
| Domain Model을 단순 DTO처럼 getter/setter만으로 사용하지 않는가? | ☐ |
