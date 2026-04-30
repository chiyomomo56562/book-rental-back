# Test 구현 템플릿

> **참고 규칙**: `docs/rules/TEST_RULE.md`
> **아키텍처 흐름**: `docs/PATTERN.md`

이 문서는 **Unit / Integration / E2E / Infrastructure** 테스트 작성 시 사용하는 표준 템플릿입니다.
테스트를 작성하기 전에 아래 체크리스트와 템플릿 코드를 따르십시오.

---

## 📋 테스트 유형 선택

작성하려는 테스트의 유형을 먼저 결정한다.

| 테스트 유형 | 대상 계층 | 위치 | 파일명 형식 |
|------------|-----------|------|------------|
| **Unit** | Domain Entity, UseCase, Mapper | 대상 파일과 동일 폴더 | `[name].[layer].test.ts` |
| **Integration** | Controller ↔ UseCase ↔ Repository | `src/__integration__/` | `[name].integration.test.ts` |
| **E2E** | 전체 시스템 비즈니스 시나리오 | `test/e2e/` | `[scenario].e2e.test.ts` |
| **Infrastructure** | Repository 복잡 쿼리, Mapper 변환 | 대상 파일과 동일 폴더 | `[name].repository.test.ts` |

---

## 🔴🟢🔵 TDD 사이클 체크리스트

Domain / UseCase 구현 시 반드시 아래 순서를 따른다.

- [ ] **Red**: 실패하는 테스트를 먼저 작성했는가? (구현 코드 없음)
- [ ] **Green**: 테스트가 통과하는 최소한의 구현을 작성했는가?
- [ ] **Refactor**: 테스트가 계속 통과하면서 코드 품질을 개선했는가?

---

## 1. Unit Test — Domain Entity 템플릿

> **목적**: 비즈니스 규칙, 상태 변경, 정책 위반 예외를 검증한다
> **위치**: `src/domain/[name].entity.test.ts`

```typescript
// src/domain/[name]/[name].entity.test.ts
import { [Name] } from './[name].entity';
import { [Name]DomainException } from './[name].exception';

describe('[Name] Entity', () => {
  // ── happy path ───────────────────────────────────────────────────────────
  describe('should_[동작]_when_[정상 조건]', () => {
    it('should_[동작]_when_[정상 조건]', () => {
      // Arrange
      const [name] = new [Name](/* 유효한 초기값 */);

      // Act
      [name].[businessMethod](/* 인자 */);

      // Assert
      expect([name].get[Field]()).toBe(/* 예상값 */);
    });
  });

  // ── failure case ─────────────────────────────────────────────────────────
  describe('should_throw_when_[위반 조건]', () => {
    it('should_throw_[ExceptionName]_when_[위반 조건]', () => {
      // Arrange
      const [name] = new [Name](/* 초기값 */);

      // Act & Assert
      expect(() => [name].[businessMethod](/* 위반 인자 */)).toThrow(
        [Name]DomainException,
      );
    });
  });
});
```

**체크리스트**:
- [ ] happy path 케이스를 포함했는가?
- [ ] 모든 실패 케이스(정책 위반, 잘못된 입력)를 포함했는가?
- [ ] Domain 객체를 Mock으로 대체하지 않았는가?
- [ ] 외부 의존성(DB, API)을 사용하지 않았는가?
- [ ] 테스트 결과가 랜덤 값·시간에 의존하지 않는가?

---

## 2. Unit Test — UseCase 템플릿

> **목적**: 비즈니스 흐름(Orchestration), 입력→결과, Side Effect 호출 여부를 검증한다
> **위치**: `src/application/usecases/[action]-[name].usecase.test.ts`

```typescript
// src/application/usecases/[action]-[name].usecase.test.ts
import { vi, describe, it, expect, beforeEach, afterEach } from 'vitest';
import { [Action][Name]UseCase } from './[action]-[name].usecase';
import { [Name]Repository } from '../../domain/repositories/[name].repository.interface';
import { [Action][Name]Command } from '../dtos/commands/[action]-[name].command';

describe('[Action][Name]UseCase', () => {
  let sut: [Action][Name]UseCase;
  let mock[Name]Repository: [Name]Repository; // 구현체가 아닌 인터페이스를 Mock

  beforeEach(() => {
    // Arrange (공통 준비): 인터페이스 기반 Mock 생성
    mock[Name]Repository = {
      findById: vi.fn(),
      save: vi.fn(),
      // ... 인터페이스에 선언된 메서드만
    };
    sut = new [Action][Name]UseCase(mock[Name]Repository, /* unitOfWork */);
  });

  afterEach(() => {
    vi.clearAllMocks(); // Mock 상태 초기화
  });

  // ── happy path ───────────────────────────────────────────────────────────
  it('should_return_result_dto_when_[정상 조건]', async () => {
    // Arrange
    const command = new [Action][Name]Command(/* 유효한 입력 */);
    vi.mocked(mock[Name]Repository.findById).mockResolvedValue(/* InfraDto */);
    vi.mocked(mock[Name]Repository.save).mockResolvedValue(/* savedInfraDto */);

    // Act
    const result = await sut.execute(command);

    // Assert — 결과 검증
    expect(result).toEqual(/* 예상 ResultDto */);
    // Assert — Side Effect 검증 (호출 횟수, 인자)
    expect(mock[Name]Repository.findById).toHaveBeenCalledOnce();
    expect(mock[Name]Repository.save).toHaveBeenCalledOnce();
  });

  // ── failure case ─────────────────────────────────────────────────────────
  it('should_throw_when_[name]_not_found', async () => {
    // Arrange
    const command = new [Action][Name]Command(/* 존재하지 않는 ID */);
    vi.mocked(mock[Name]Repository.findById).mockResolvedValue(null);

    // Act & Assert
    await expect(sut.execute(command)).rejects.toThrow(/* Exception */);
    expect(mock[Name]Repository.save).not.toHaveBeenCalled();
  });
});
```

**체크리스트**:
- [ ] Repository / External Port의 **인터페이스**를 Mock했는가? (구현체 직접 Mock 금지)
- [ ] `afterEach`에서 `vi.clearAllMocks()`를 호출했는가?
- [ ] 결과(ResultDto) 구조를 검증했는가?
- [ ] Side Effect(Repository 호출 횟수/순서)를 검증했는가?
- [ ] Domain Model 또는 InfraDto를 그대로 반환하지 않는가?
- [ ] 실패 케이스에서 `save`가 호출되지 않음을 검증했는가?

---

## 3. Integration Test 템플릿

> **목적**: Controller ↔ UseCase ↔ Repository 실제 연동 흐름을 검증한다
> **위치**: `src/__integration__/[name].integration.test.ts`
> **도구**: `supertest` + 실제 앱 인스턴스, 인메모리 DB (`sqlite` / `pg-mem`)

```typescript
// src/__integration__/[name].integration.test.ts
import { describe, it, expect, beforeAll, afterAll, beforeEach } from 'vitest';
import request from 'supertest';
import { createApp } from '../app'; // 실제 앱 팩토리

describe('[Name] API Integration', () => {
  let app: Express.Application;

  beforeAll(async () => {
    // 실제 앱 인스턴스 생성 (인메모리 DB 연결)
    app = await createApp({ db: 'memory' });
  });

  afterAll(async () => {
    // DB 연결 해제
    await app.close();
  });

  beforeEach(async () => {
    // 각 테스트 전 DB 데이터 초기화
    await clearTestDatabase();
  });

  // ── happy path ───────────────────────────────────────────────────────────
  it('should_return_[status]_when_[정상 조건]', async () => {
    // Arrange
    const requestBody = { /* 요청 데이터 */ };

    // Act
    const response = await request(app)
      .post('/[endpoint]')
      .send(requestBody);

    // Assert — HTTP 상태 코드
    expect(response.status).toBe(/* 예상 상태 코드 */);
    // Assert — 응답 바디 구조 (ApiResponse 형식 준수)
    expect(response.body).toMatchObject({
      success: true,
      data: { /* 예상 필드 */ },
    });
  });

  // ── failure case ─────────────────────────────────────────────────────────
  it('should_return_[error_status]_when_[실패 조건]', async () => {
    // Arrange
    const invalidBody = { /* 잘못된 요청 */ };

    // Act
    const response = await request(app)
      .post('/[endpoint]')
      .send(invalidBody);

    // Assert
    expect(response.status).toBe(/* 예상 에러 상태 코드 */);
    expect(response.body).toMatchObject({
      success: false,
      error: { message: expect.any(String) },
    });
  });
});
```

**체크리스트**:
- [ ] 인메모리 DB 또는 Docker 테스트 컨테이너를 사용하는가?
- [ ] `beforeEach`에서 DB 상태를 초기화하는가?
- [ ] HTTP 상태 코드와 응답 바디(`ApiResponse`) 구조를 함께 검증하는가?
- [ ] `afterAll`에서 DB 연결을 해제하는가?

---

## 4. E2E Test 템플릿

> **목적**: 핵심 비즈니스 시나리오의 전체 흐름을 검증한다
> **위치**: `test/e2e/[scenario].e2e.test.ts`
> **도구**: `vitest` (`vitest.e2e.config.ts`) + `supertest`

```typescript
// test/e2e/[scenario].e2e.test.ts
import { describe, it, expect, beforeAll, afterAll } from 'vitest';
import request from 'supertest';

// globalSetup에서 기동된 앱 URL을 참조
const BASE_URL = process.env.E2E_BASE_URL ?? 'http://localhost:3000';

describe('[Scenario] E2E', () => {
  beforeAll(async () => {
    // 시나리오 초기 데이터 세팅 (필요한 경우)
  });

  afterAll(async () => {
    // 테스트 데이터 Teardown (반드시 수행)
    await cleanupE2EData();
  });

  it('should_complete_[scenario]_flow', async () => {
    // Step 1: [첫 번째 행동]
    const step1Response = await request(BASE_URL)
      .post('/[step1-endpoint]')
      .send({ /* 데이터 */ });
    expect(step1Response.status).toBe(/* 상태 코드 */);

    // Step 2: [두 번째 행동] (Step 1 결과 활용)
    const { [field] } = step1Response.body.data;
    const step2Response = await request(BASE_URL)
      .post('/[step2-endpoint]')
      .send({ [field], /* 추가 데이터 */ });
    expect(step2Response.status).toBe(/* 상태 코드 */);

    // Final Assert: 최종 상태 검증
    expect(step2Response.body.data).toMatchObject({ /* 예상 결과 */ });
  });
});
```

**체크리스트**:
- [ ] `vitest.e2e.config.ts` 별도 설정을 사용하는가?
- [ ] `afterAll`에서 모든 테스트 데이터를 정리(Teardown)하는가?
- [ ] Unit/Integration 테스트와 CI 단계가 분리되어 있는가?
- [ ] 시나리오 각 단계의 성공 여부를 중간에 검증하는가?

---

## 5. Infrastructure Test 템플릿 (Repository)

> **목적**: 복잡한 쿼리 로직, Mapper 변환 결과를 검증한다
> **위치**: `src/infrastructure/[name]/[name].repository.test.ts`
> **도구**: 인메모리 DB (`sqlite` / `pg-mem`) 또는 Docker 테스트 컨테이너

```typescript
// src/infrastructure/[name]/[name].repository.test.ts
import { describe, it, expect, beforeAll, afterAll, beforeEach } from 'vitest';
import { [Name]PrismaRepository } from './[name].prisma.repository';
import { createTestPrismaClient } from '../../test/helpers/prisma-test-client';

describe('[Name]PrismaRepository', () => {
  let repository: [Name]PrismaRepository;
  let prisma: PrismaClient; // 인메모리 DB 클라이언트

  beforeAll(async () => {
    prisma = await createTestPrismaClient(); // 인메모리 DB 연결
    repository = new [Name]PrismaRepository(prisma);
  });

  afterAll(async () => {
    await prisma.$disconnect();
  });

  beforeEach(async () => {
    // 각 테스트 전 테이블 초기화
    await prisma.[name].deleteMany();
  });

  it('should_return_filtered_results_when_[조건]', async () => {
    // Arrange: 테스트 데이터 삽입
    await prisma.[name].createMany({ data: [/* 테스트 레코드 */] });

    // Act
    const result = await repository.findBy[Condition](/* 필터 인자 */);

    // Assert
    expect(result).toHaveLength(/* 예상 개수 */);
    expect(result[0]).toMatchObject(/* 예상 InfraDto 구조 */);
  });
});
```

**체크리스트**:
- [ ] 실제 운영 DB가 아닌 인메모리 DB 또는 테스트 컨테이너를 사용하는가?
- [ ] `beforeEach`에서 테이블 데이터를 초기화하는가?
- [ ] `afterAll`에서 DB 연결을 해제하는가?
- [ ] 반환값이 예상 InfraDto 구조와 일치하는가?

---

## ⚠️ 공통 금지 패턴 체크리스트

| 항목 | 확인 |
|------|------|
| Unit Test에서 실제 DB/API를 호출하지 않는가? | ☐ |
| Domain 객체를 Mock으로 대체하지 않았는가? | ☐ |
| Repository 구현체(Prisma Client 등)를 직접 Mock하지 않았는가? | ☐ |
| 테스트 간 상태를 공유하지 않는가? (순서 의존성 없음) | ☐ |
| 랜덤 값·실제 시간·외부 상태에 의존하지 않는가? | ☐ |
| private method를 직접 테스트하지 않는가? | ☐ |
| 테스트 이름이 `should_[동작]_when_[조건]` 형식을 따르는가? | ☐ |
| `afterEach`에서 Mock을 초기화했는가? | ☐ |
