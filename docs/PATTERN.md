# 프로젝트 아키텍처 통합 레퍼런스 (Pattern Reference)

이 문서는 AI 에이전트와 개발자가 본 프로젝트의 **PRESENTATION-USECASE-DOMAIN-INFRASTRUCTURE** 흐름을 정확히 이해하고 헷갈리지 않도록 돕기 위한 "단일 완성된 Flow 예시"입니다. 모든 계층 간 통신은 철저하게 **DTO-Only** 정책을 따르며, `Mapper`를 통해 변환됩니다.

## 🌟 핵심 요약: 계층별 DTO 및 Mapper 역할

1. **Presentation Layer**: `Request DTO` ↔ `Command/Query DTO` (입력) / `Result DTO` ↔ `Response DTO` (출력)
2. **Application Layer (UseCase)**: `Command/Query DTO` ↔ `Domain Model` ↔ `Result DTO`
3. **Domain Layer**: `Domain Model` (순수 비즈니스 로직 및 상태 캡슐화)
4. **Infrastructure Layer**: `Entity(DB/ORM)` ↔ `Infra DTO`

---

## 🚀 통합 예시: "도서 대여 (Rent a Book)" 흐름 (Flow)

이 예시는 사용자가 특정 도서를 대여하는 `POST /api/rentals` 엔드포인트의 전체 실행 흐름입니다.

### 1. Presentation Layer (Controller)
**역할**: HTTP 요청 수신, 입력 검증, UseCase 호출, 표준 HTTP 응답 반환

```typescript
// src/presentation/controllers/rental.controller.ts
export class RentalController {
  constructor(private readonly rentBookUseCase: RentBookUseCase) {}

  async rentBook(req: Request, res: Response) {
    // 1. Request DTO 변환 및 검증 (DTO 레벨에서 Validation 수행)
    const requestDto = new RentBookRequestDto(req.body);
    
    // 2. Request DTO -> Command DTO 변환 (Presentation Mapper)
    const command = PresentationRentalMapper.toCommand(requestDto);
    
    // 3. UseCase 실행 (Command 전달, Result DTO 반환)
    const resultDto = await this.rentBookUseCase.execute(command);
    
    // 4. Result DTO -> Response DTO 변환 (Presentation Mapper)
    const responseDto = PresentationRentalMapper.toResponse(resultDto);
    
    // 5. 표준 ApiResponse 포맷으로 반환
    return res.status(200).json({ 
      status: 200, 
      data: responseDto, 
      error: null 
    });
  }
}
```

### 2. Application Layer (UseCase)
**역할**: 비즈니스 흐름 오케스트레이션, 트랜잭션 경계 지정, 예외 처리 (직접적인 비즈니스 룰은 Domain에 위임)

```typescript
// src/application/usecases/rent-book.usecase.ts
export class RentBookUseCase {
  constructor(
    private readonly bookRepository: BookRepository,
    private readonly unitOfWork: UnitOfWork // 추상화된 트랜잭션 관리자
  ) {}

  async execute(command: RentBookCommand): Promise<RentBookResultDto> {
    // 트랜잭션 경계 설정 (인프라 기술에 독립적)
    return await this.unitOfWork.runInTransaction(async (tx: TransactionContext) => {
      // 1. Repository에서 Infra DTO 조회 (트랜잭션 컨텍스트 tx 전달)
      const bookInfraDto = await this.bookRepository.findById(command.bookId, tx);
      if (!bookInfraDto) throw new Error("Book not found");

      // 2. Infra DTO -> Domain Model 변환 (Application Mapper)
      const book = AppBookMapper.toDomain(bookInfraDto);

      // 3. Domain Model의 비즈니스 로직 실행 (상태 변경 등)
      book.rent(command.userId);

      // 4. Repository를 통해 저장 (Domain Model을 직접 전달, 동일 트랜잭션 tx 사용)
      // - 별도의 Save DTO 없이 도메인 객체를 그대로 영속성 계층으로 넘깁니다.
      const savedInfraDto = await this.bookRepository.save(book, tx);

      // 5. 업데이트된 Infra DTO -> Domain -> Result DTO 변환 후 반환
      const resultBook = AppBookMapper.toDomain(savedInfraDto);
      return AppBookMapper.toResult(resultBook);
    });
  }
}
```

### 3. Domain Layer (Model)
**역할**: 순수 비즈니스 상태와 행위(규칙) 캡슐화, 외부 프레임워크나 인프라에 대한 의존성 없음

```typescript
// src/domain/models/book.model.ts
export class Book {
  private status: BookStatus;
  private currentRenterId: string | null;
  
  constructor(
    public readonly id: string,
    public readonly title: string,
    status: BookStatus,
    currentRenterId: string | null = null
  ) {
    this.status = status;
    this.currentRenterId = currentRenterId;
  }

  // 비즈니스 로직: 대여 가능 여부 확인 및 상태 변경
  rent(userId: string): void {
    if (this.status !== BookStatus.AVAILABLE) {
      throw new DomainException("이 책은 현재 대여할 수 없습니다.");
    }
    this.status = BookStatus.RENTED;
    this.currentRenterId = userId;
  }

  getStatus() { return this.status; }
  getRenterId() { return this.currentRenterId; }
}
```

### 4. Infrastructure Layer (Repository)
**역할**: DB 접근 로직 (ORM 사용), 순수 DB Entity와 시스템 공통 DTO(Infra DTO) 간의 변환

```typescript
// src/infrastructure/repositories/book.repository.ts
export class BookRepositoryImpl implements BookRepository {
  constructor(private readonly prisma: PrismaClient) {}

  async findById(id: string, tx?: TransactionContext): Promise<BookInfraDto | null> {
    // tx가 있으면 반드시 사용, 없으면 기본 클라이언트 사용 (명시적 분기)
    const client = tx ? (tx as PrismaTransactionClient) : this.prisma;
    
    const entity = await client.book.findUnique({ where: { id } });
    if (!entity) return null;

    return InfraBookMapper.toInfraDto(entity);
  }

  // Domain Model을 직접 인자로 받음
  async save(book: Book, tx?: TransactionContext): Promise<BookInfraDto> {
    // tx 제공 시 반드시 해당 컨텍스트 사용 (암묵적 fallback 금지)
    const client = tx ? (tx as PrismaTransactionClient) : this.prisma;

    // 인프라 계층 내부에서 도메인 모델을 DB Entity(또는 Prisma 입력 데이터)로 변환
    const savedEntity = await client.book.update({
      where: { id: book.id },
      data: {
        status: book.getStatus(),
        renterId: book.getRenterId()
      }
    });

    return InfraBookMapper.toInfraDto(savedEntity);
  }
}
```

### 5. Mapper (계층 간 변환)
**역할**: 계층을 넘어갈 때 데이터 형태를 해당 계층에 맞게 명시적으로 변환 (Model Isolation 보장)

```typescript
// 계층별 변환을 담당하는 Mapper들의 위치 및 형태 예시

// [Presentation Mapper]: src/presentation/mappers/
class PresentationRentalMapper {
  static toCommand(req: RentBookRequestDto): RentBookCommand {
    return new RentBookCommand(req.bookId, req.userId);
  }
  static toResponse(result: RentBookResultDto): RentBookResponseDto {
    return new RentBookResponseDto(result.id, result.status);
  }
}

// [Application Mapper]: src/application/mappers/
class AppBookMapper {
  // Infra DTO -> Domain Model
  static toDomain(dto: BookInfraDto): Book {
    return new Book(dto.id, dto.title, dto.status, dto.renterId);
  }
  // Domain Model -> Result DTO
  static toResult(domain: Book): RentBookResultDto {
    return new RentBookResultDto(domain.id, domain.getStatus());
  }
}

// [Infra Mapper]: src/infrastructure/mappers/
class InfraBookMapper {
  static toInfraDto(entity: BookPrismaEntity): BookInfraDto {
    return new BookInfraDto(entity.id, entity.title, entity.status, entity.renterId);
  }
  // 필요 시 Domain -> Entity 변환 메서드를 추가할 수 있으나, Save DTO는 더 이상 사용하지 않습니다.
}
```

---

## 🔍 추가 예시: "도서 상세 조회 (Get Book Detail)" 흐름 (Query Flow)

이 예시는 `GET /api/books/:id` 엔드포인트의 **조회(Read) 흐름**입니다. Write 흐름과의 핵심 차이를 확인하십시오.

> **Write vs Query 핵심 차이**: Query 흐름에서는 상태 변경이 없으므로, **Infra DTO를 Domain Model로 변환하지 않고 바로 Result DTO로 매핑할 수 있습니다.** Domain Model 생성은 비즈니스 규칙 실행이 필요한 경우에만 수행합니다.

### Query: Application Layer (UseCase)

```typescript
// src/application/usecases/get-book-detail.usecase.ts
export class GetBookDetailUseCase {
  constructor(private readonly bookRepository: BookRepository) {}

  async execute(query: GetBookDetailQuery): Promise<GetBookDetailResultDto> {
    // 1. Repository에서 Infra DTO 조회 (트랜잭션 불필요)
    const bookInfraDto = await this.bookRepository.findById(query.bookId);
    if (!bookInfraDto) throw new BookNotFoundException(query.bookId);

    // 2. [Query 전용] Infra DTO → Result DTO 직접 변환
    //    상태 변경 없이 단순 조회이므로 Domain Model 생성을 생략할 수 있습니다.
    return AppBookMapper.toResult(bookInfraDto);
  }
}
```

### Query: Application Mapper

```typescript
// src/application/mappers/book.mapper.ts
class AppBookMapper {
  // Write 흐름용: Infra DTO → Domain Model (비즈니스 로직 실행 전 사용)
  static toDomain(dto: BookInfraDto): Book {
    return new Book(dto.id, dto.title, dto.status, dto.renterId);
  }

  // Write 흐름용: Domain Model → Result DTO
  static toResultFromDomain(domain: Book): GetBookDetailResultDto {
    return new GetBookDetailResultDto(domain.id, domain.title, domain.getStatus());
  }

  // Query 흐름용: Infra DTO → Result DTO (Domain Model 생성 생략)
  static toResult(dto: BookInfraDto): GetBookDetailResultDto {
    return new GetBookDetailResultDto(dto.id, dto.title, dto.status);
  }
}
```

### ⚖️ Write vs Query 흐름 비교

| 항목 | Write (Rent Book) | Query (Get Book) |
|------|-------------------|------------------|
| 트랜잭션 | ✅ 필요 (`unitOfWork`) | ❌ 불필요 |
| Domain Model 생성 | ✅ 필요 (비즈니스 로직 실행) | ❌ 생략 가능 |
| Repository 반환값 | Infra DTO | Infra DTO |
| 최종 반환값 | Result DTO | Result DTO |
| Mapper 경로 | Infra DTO → Domain → Result DTO | Infra DTO → Result DTO |

---

## ⚠️ 필수 주의사항 (AI 에이전트 및 개발자 체크리스트)
- **Controller**는 **절대 비즈니스 판단을 하지 않으며**, DTO 변환 후 오직 **UseCase**만 호출합니다.
- **UseCase**는 절대 ORM이나 DB Entity를 직접 다루지 않습니다. 오직 **DTO와 Domain Model**을 사용하여 흐름을 제어합니다.
- **Repository**는 DB Entity(`Prisma/TypeORM 객체 등`)를 외부로 노출하지 않으며, 반드시 **Infra DTO**로 매핑하여 반환합니다.
- **Mapper**에는 비즈니스 로직(검증, 계산, 조건부 매핑 등)을 넣지 않습니다. 순수한 **단방향 데이터 형태 변환**만 명시적으로 수행합니다.
- **Query 흐름**에서는 상태 변경이 없을 경우 Domain Model 생성을 생략하고 Infra DTO를 Result DTO로 직접 변환할 수 있습니다. 단, 이 결정은 UseCase 내에서 명시적으로 이루어져야 합니다.

