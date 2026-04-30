# UseCase (Application) Layer 구현 템플릿

> **참고 규칙**: `docs/rules/USECASE_RULE.md`
> **아키텍처 흐름**: `docs/PATTERN.md`

이 문서는 Application Layer의 **UseCase**, **Command/Query DTO**, **Result DTO**, **Application Mapper** 구현 시 사용하는 표준 템플릿입니다.

---

## 📁 디렉토리 구조

```
src/application/
├── usecases/
│   └── [action]-[name].usecase.ts
├── dtos/
│   ├── commands/
│   │   └── [action]-[name].command.ts
│   ├── queries/
│   │   └── [action]-[name].query.ts
│   └── results/
│       └── [action]-[name].result.dto.ts
└── mappers/
    └── [name].mapper.ts
```

---

## 1. Command DTO 템플릿 (Write 흐름)

> **목적**: Presentation Layer에서 UseCase로 전달되는 쓰기 요청 데이터. 불변 객체.

```typescript
// src/application/dtos/commands/[action]-[name].command.ts
export class [Action][Name]Command {
  constructor(
    public readonly [field1]: string,
    public readonly [field2]: string,
  ) {}
}
```

---

## 2. Query DTO 템플릿 (Read 흐름)

```typescript
// src/application/dtos/queries/[action]-[name].query.ts
export class [Action][Name]Query {
  constructor(
    public readonly [field1]: string,
  ) {}
}
```

---

## 3. Result DTO 템플릿

> **목적**: UseCase가 Presentation Layer로 반환하는 결과. Domain Model을 직접 반환 금지.

```typescript
// src/application/dtos/results/[action]-[name].result.dto.ts
export class [Action][Name]ResultDto {
  constructor(
    public readonly id: string,
    public readonly [field1]: [Type1],
    public readonly [field2]: [Type2],
  ) {}
}
```

---

## 4. Application Mapper 템플릿

> **목적**: Infra DTO ↔ Domain Model, Domain Model → Result DTO 변환. 비즈니스 로직 금지.

```typescript
// src/application/mappers/[name].mapper.ts
import { [Name] } from '../../domain/models/[name].model';
import { [Name]InfraDto } from '../../infrastructure/dtos/[name].infra-dto';
import { [Action][Name]ResultDto } from '../dtos/results/[action]-[name].result.dto';

export class App[Name]Mapper {
  /** Infra DTO → Domain Model (Write 흐름: 비즈니스 로직 실행 전) */
  static toDomain(dto: [Name]InfraDto): [Name] {
    return new [Name](dto.id, dto.[field1], dto.[field2]);
  }

  /** Domain Model → Result DTO (Write 흐름: 비즈니스 로직 실행 후) */
  static toResultFromDomain(domain: [Name]): [Action][Name]ResultDto {
    return new [Action][Name]ResultDto(domain.id, domain.get[Field1]());
  }

  /** Infra DTO → Result DTO (Query 흐름: Domain Model 생성 생략) */
  static toResult(dto: [Name]InfraDto): [Action][Name]ResultDto {
    return new [Action][Name]ResultDto(dto.id, dto.[field1]);
  }
}
```

---

## 5. UseCase 템플릿 — Write (Command) 흐름

> **목적**: 비즈니스 흐름 오케스트레이션. 트랜잭션 경계 관리. 실제 비즈니스 규칙은 Domain에 위임.

```typescript
// src/application/usecases/[action]-[name].usecase.ts
import { [Name]Repository } from '../../domain/repositories/[name].repository.interface';
import { UnitOfWork, TransactionContext } from '../../domain/types/transaction.type';
import { [Action][Name]Command } from '../dtos/commands/[action]-[name].command';
import { [Action][Name]ResultDto } from '../dtos/results/[action]-[name].result.dto';
import { App[Name]Mapper } from '../mappers/[name].mapper';

export class [Action][Name]UseCase {
  constructor(
    private readonly [name]Repository: [Name]Repository,
    private readonly unitOfWork: UnitOfWork,
  ) {}

  async execute(command: [Action][Name]Command): Promise<[Action][Name]ResultDto> {
    return await this.unitOfWork.runInTransaction(async (tx: TransactionContext) => {
      // 1. Repository에서 Infra DTO 조회 (tx 전달 필수)
      const infraDto = await this.[name]Repository.findById(command.[field1], tx);
      if (!infraDto) throw new Error('[Name] not found');

      // 2. Infra DTO → Domain Model 변환 (Application Mapper)
      const [name] = App[Name]Mapper.toDomain(infraDto);

      // 3. Domain Model의 비즈니스 로직 실행 (상태 변경)
      [name].[businessMethod](command.[field2]);

      // 4. 저장 (Domain Model 직접 전달, 동일 tx 사용)
      const savedInfraDto = await this.[name]Repository.save([name], tx);

      // 5. 저장된 Infra DTO → Domain → Result DTO 변환 후 반환
      const savedDomain = App[Name]Mapper.toDomain(savedInfraDto);
      return App[Name]Mapper.toResultFromDomain(savedDomain);
    });
  }
}
```

---

## 6. UseCase 템플릿 — Read (Query) 흐름

> **핵심**: 상태 변경이 없는 조회는 트랜잭션 불필요. Domain Model 생성 생략 가능.

```typescript
// src/application/usecases/get-[name]-detail.usecase.ts
export class Get[Name]DetailUseCase {
  constructor(private readonly [name]Repository: [Name]Repository) {}

  async execute(query: Get[Name]DetailQuery): Promise<Get[Name]DetailResultDto> {
    const infraDto = await this.[name]Repository.findById(query.[field1]);
    if (!infraDto) throw new [Name]NotFoundException(query.[field1]);

    // Query 흐름: 상태 변경 없음 → Infra DTO → Result DTO 직접 변환 (Domain Model 생략)
    return App[Name]Mapper.toResult(infraDto);
  }
}
```

---

## ⚠️ 금지 패턴 체크리스트

| 항목 | 확인 |
|------|------|
| UseCase에 비즈니스 규칙/정책 로직이 없는가? (Domain에 위임) | ☐ |
| ORM·DB를 직접 접근하지 않는가? | ☐ |
| Write 흐름에서 트랜잭션(`unitOfWork`)을 사용하는가? | ☐ |
| `tx`를 Repository에 명시적으로 전달하는가? | ☐ |
| Repository 반환값(Infra DTO)을 바로 반환하지 않는가? | ☐ |
| Domain Model을 Result DTO로 변환 후 반환하는가? | ☐ |
| Query 흐름에서 불필요한 트랜잭션을 사용하지 않는가? | ☐ |
