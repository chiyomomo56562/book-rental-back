# Infrastructure Layer 구현 템플릿

> **참고 규칙**: `docs/rules/INFRASTRUCTURE_RULE.md`
> **아키텍처 흐름**: `docs/PATTERN.md`

이 문서는 Infrastructure Layer의 **Repository**, **External API Client**, **Mapper** 구현 시 사용하는 표준 템플릿입니다.
각 섹션의 `[PlaceHolder]`를 실제 도메인 이름으로 교체하여 사용하십시오.

---

## 📁 디렉토리 구조

```
src/infrastructure/
├── repositories/
│   └── [name].repository.ts        # Repository 구현체
├── clients/
│   └── [name].client.ts            # 외부 API 클라이언트
├── mappers/
│   └── [name].mapper.ts            # Infra Mapper (Entity ↔ Infra DTO)
└── dtos/
    └── [name].infra-dto.ts         # Infra DTO 정의
```

---

## 1. Infra DTO 템플릿

> **목적**: Repository가 외부로 반환하는 데이터 전달 객체. ORM Entity를 직접 노출하지 않기 위한 방어막.

```typescript
// src/infrastructure/dtos/[name].infra-dto.ts

export class [Name]InfraDto {
  constructor(
    public readonly id: string,
    public readonly [field1]: [Type1],
    public readonly [field2]: [Type2],
    // ... 필요한 필드 추가
  ) {}
}
```

---

## 2. Infra Mapper 템플릿

> **목적**: ORM Entity(Prisma/TypeORM 객체)를 Infra DTO로 단방향 변환. 비즈니스 로직 포함 금지.

```typescript
// src/infrastructure/mappers/[name].mapper.ts

import { [Name]InfraDto } from '../dtos/[name].infra-dto';
// Prisma 예시: import { [Name] as [Name]PrismaEntity } from '@prisma/client';

export class Infra[Name]Mapper {
  /**
   * ORM Entity → Infra DTO 변환
   * 주의: 비즈니스 로직(조건문, 계산 등) 포함 금지. 순수 필드 매핑만 수행.
   */
  static toInfraDto(entity: [Name]PrismaEntity): [Name]InfraDto {
    return new [Name]InfraDto(
      entity.id,
      entity.[field1],
      entity.[field2],
      // ...
    );
  }
}
```

---

## 3. Repository 구현체 템플릿

> **목적**: Application Layer의 Repository Interface를 구현. ORM 사용은 이 내부에서만. 반드시 Infra DTO 반환.

```typescript
// src/infrastructure/repositories/[name].repository.ts

import { PrismaClient } from '@prisma/client';
import { [Name]Repository } from '../../domain/repositories/[name].repository.interface';
import { [Name]InfraDto } from '../dtos/[name].infra-dto';
import { Infra[Name]Mapper } from '../mappers/[name].mapper';
import { [Name] } from '../../domain/models/[name].model';
import { TransactionContext } from '../../domain/types/transaction.type';

export class [Name]RepositoryImpl implements [Name]Repository {
  constructor(private readonly prisma: PrismaClient) {}

  /**
   * ID로 단건 조회
   * - 반환: [Name]InfraDto | null (Domain Model 직접 반환 금지)
   * - tx 제공 시 반드시 해당 컨텍스트 사용 (암묵적 fallback 금지)
   */
  async findById(id: string, tx?: TransactionContext): Promise<[Name]InfraDto | null> {
    const client = tx ? (tx as PrismaTransactionClient) : this.prisma;

    const entity = await client.[name].findUnique({ where: { id } });
    if (!entity) return null;

    return Infra[Name]Mapper.toInfraDto(entity);
  }

  /**
   * 저장 (생성 또는 갱신)
   * - 인자: Domain Model을 직접 받음 (Save DTO 생성 금지)
   * - Domain → Entity 변환은 이 메서드 내부에서만 수행
   * - 반환: 저장된 결과를 InfraDto로 변환하여 반환
   */
  async save([name]: [Name], tx?: TransactionContext): Promise<[Name]InfraDto> {
    const client = tx ? (tx as PrismaTransactionClient) : this.prisma;

    const savedEntity = await client.[name].upsert({
      where: { id: [name].id },
      update: {
        // Domain Model의 getter를 통해 상태 추출 (직접 필드 접근 금지)
        [field1]: [name].get[Field1](),
        [field2]: [name].get[Field2](),
      },
      create: {
        id: [name].id,
        [field1]: [name].get[Field1](),
        [field2]: [name].get[Field2](),
      },
    });

    return Infra[Name]Mapper.toInfraDto(savedEntity);
  }

  /**
   * 목록 조회
   */
  async findAll(tx?: TransactionContext): Promise<[Name]InfraDto[]> {
    const client = tx ? (tx as PrismaTransactionClient) : this.prisma;

    const entities = await client.[name].findMany();
    return entities.map(Infra[Name]Mapper.toInfraDto);
  }
}
```

---

## 4. External API Client 템플릿

> **목적**: 외부 HTTP API 호출을 담당. 응답은 반드시 Mapper를 통해 Infra DTO로 변환 후 반환.

```typescript
// src/infrastructure/clients/[name].client.ts

import axios from 'axios';
import { [Name]InfraDto } from '../dtos/[name].infra-dto';
import { Infra[Name]Mapper } from '../mappers/[name].mapper';

export class [Name]ApiClient {
  constructor(private readonly baseUrl: string) {}

  /**
   * 외부 API 응답 → Infra DTO 변환 후 반환 (raw 응답 직접 반환 금지)
   */
  async fetch[Resource](id: string): Promise<[Name]InfraDto> {
    const response = await axios.get(`${this.baseUrl}/[resource]/${id}`);
    // API 응답도 반드시 Mapper를 통해 변환
    return Infra[Name]Mapper.toInfraDto(response.data);
  }
}
```

---

## ⚠️ 금지 패턴 체크리스트

구현 후 아래 항목을 반드시 확인하십시오.

| 항목 | 확인 |
|------|------|
| Repository가 ORM Entity를 직접 반환하지 않는가? | ☐ |
| Repository가 Domain Model을 직접 반환하지 않는가? | ☐ |
| `tx`가 제공된 경우 기본 클라이언트로 fallback하지 않는가? | ☐ |
| `save()` 메서드가 Save DTO 없이 Domain Model을 직접 받는가? | ☐ |
| `Domain → Entity` 변환이 Repository 내부에서만 이루어지는가? | ☐ |
| Mapper에 비즈니스 로직(조건문, 계산)이 없는가? | ☐ |
| 외부 API 응답을 Mapper 없이 raw로 반환하지 않는가? | ☐ |
| 민감 정보(비밀번호, 토큰)가 로그에 포함되지 않는가? | ☐ |
