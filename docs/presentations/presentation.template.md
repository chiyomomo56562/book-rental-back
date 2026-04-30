# Presentation Layer 구현 템플릿

> **참고 규칙**: `docs/rules/PRESENTATION_RULE.md`
> **아키텍처 흐름**: `docs/PATTERN.md`

이 문서는 Presentation Layer의 **Controller**, **Request/Response DTO**, **Presentation Mapper** 구현 시 사용하는 표준 템플릿입니다.

---

## 📁 디렉토리 구조

```
src/presentation/
├── controllers/
│   └── [name].controller.ts
├── dtos/
│   ├── request/
│   │   └── [action]-[name].request.dto.ts
│   └── response/
│       └── [action]-[name].response.dto.ts
└── mappers/
    └── [name].mapper.ts
```

---

## 1. Request DTO 템플릿

```typescript
// src/presentation/dtos/request/[action]-[name].request.dto.ts
import { IsString, IsNotEmpty, IsUUID } from 'class-validator';

export class [Action][Name]RequestDto {
  @IsUUID()
  @IsNotEmpty()
  readonly [field1]: string;

  @IsString()
  @IsNotEmpty()
  readonly [field2]: string;
}
```

---

## 2. Response DTO 템플릿

```typescript
// src/presentation/dtos/response/[action]-[name].response.dto.ts
export class [Action][Name]ResponseDto {
  constructor(
    public readonly id: string,
    public readonly [field1]: [Type1],
    public readonly [field2]: [Type2],
  ) {}
}
```

---

## 3. Presentation Mapper 템플릿

> **목적**: Request DTO → Command DTO / Result DTO → Response DTO 변환. 비즈니스 로직 금지.

```typescript
// src/presentation/mappers/[name].mapper.ts
export class Presentation[Name]Mapper {
  static toCommand(req: [Action][Name]RequestDto): [Action][Name]Command {
    return new [Action][Name]Command(req.[field1], req.[field2]);
  }

  static toResponse(result: [Action][Name]ResultDto): [Action][Name]ResponseDto {
    return new [Action][Name]ResponseDto(result.id, result.[field1]);
  }
}
```

---

## 4. Controller 템플릿

> **목적**: HTTP 요청 수신 → DTO 변환 → UseCase 호출 → 표준 응답 반환.
> 비즈니스 로직 금지. 하나의 메서드 = 하나의 UseCase.

```typescript
// src/presentation/controllers/[name].controller.ts
import { Request, Response } from 'express';
import { [Action][Name]UseCase } from '../../application/usecases/[action]-[name].usecase';
import { [Action][Name]RequestDto } from '../dtos/request/[action]-[name].request.dto';
import { Presentation[Name]Mapper } from '../mappers/[name].mapper';

export class [Name]Controller {
  constructor(private readonly [action][Name]UseCase: [Action][Name]UseCase) {}

  async [actionMethod](req: Request, res: Response) {
    // 1. Request DTO 변환 (Validation 포함)
    const requestDto = new [Action][Name]RequestDto(req.body);
    // 2. Request DTO → Command DTO (Presentation Mapper)
    const command = Presentation[Name]Mapper.toCommand(requestDto);
    // 3. UseCase 실행
    const resultDto = await this.[action][Name]UseCase.execute(command);
    // 4. Result DTO → Response DTO (Presentation Mapper)
    const responseDto = Presentation[Name]Mapper.toResponse(resultDto);
    // 5. 표준 ApiResponse 반환
    return res.status(200).json({ status: 200, data: responseDto, error: null });
  }
}
```

---

## 5. 표준 응답 타입

```typescript
// src/presentation/types/api-response.type.ts
export type ApiError = { message: string; code: string; status: number };
export type ApiResponse<T> = { status: number; data: T | null; error: ApiError | null };
```

- **성공**: `{ status: 200, data: responseDto, error: null }`
- **실패**: 공통 에러 핸들러가 `{ status: 4xx, data: null, error: { ... } }` 형식으로 처리

---

## ⚠️ 금지 패턴 체크리스트

| 항목 | 확인 |
|------|------|
| Controller에 비즈니스 조건문(if/else)이 없는가? | ☐ |
| Controller가 Repository·ORM을 직접 호출하지 않는가? | ☐ |
| 하나의 메서드가 하나의 UseCase만 호출하는가? | ☐ |
| `req.body`를 그대로 UseCase에 전달하지 않는가? | ☐ |
| Domain Model을 Response로 직접 반환하지 않는가? | ☐ |
| Mapper에 비즈니스 로직이 없는가? | ☐ |
| 응답이 표준 `ApiResponse<T>` 형식을 따르는가? | ☐ |
