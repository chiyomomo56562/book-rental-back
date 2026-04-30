이 프로젝트는 Harness 프레임워크를 사용한다. 아래 워크플로우에 따라 작업을 진행하라.

---

## 워크플로우

### A. 탐색

`/docs/` 하위 문서(ARCHITECTURE, ADR 등)를 읽고 프로젝트의 기획·아키텍처·설계 의도를 파악한다. 필요시 Gemini의 탐색 기능을 활용한다.

### B. 논의

구현을 위해 구체화하거나 기술적으로 결정해야 할 사항이 있으면 사용자에게 제시하고 논의한다.

### C. Step 설계

사용자가 구현 계획 작성을 지시하면 여러 step으로 나뉜 초안을 작성해 피드백을 요청한다.

설계 원칙:

1. **Scope 최소화** — 하나의 step에서 하나의 레이어 또는 모듈만 다룬다. 여러 모듈을 동시에 수정해야 하면 step을 쪼갠다.
2. **자기완결성** — 각 step 파일은 독립된 Gemini 세션에서 실행된다. "이전 대화에서 논의한 바와 같이" 같은 외부 참조는 금지한다. 필요한 정보는 전부 파일 안에 적는다.
3. **사전 준비 강제** — 관련 문서 경로와 이전 step에서 생성/수정된 파일 경로를 명시한다. 세션이 코드를 읽고 맥락을 파악한 뒤 작업하도록 유도한다.
4. **시그니처 수준 지시** — 함수/클래스의 인터페이스만 제시하고 내부 구현은 에이전트 재량에 맡긴다. 단, 설계 의도에서 벗어나면 안 되는 핵심 규칙(멱등성, 보안, 데이터 무결성 등)은 반드시 명시한다.
5. **AC는 실행 가능한 커맨드** — "~가 동작해야 한다" 같은 추상적 서술이 아닌 `npm run build && npm test` 같은 실제 실행 가능한 검증 커맨드를 포함한다.
6. **주의사항은 구체적으로** — "조심해라" 대신 "X를 하지 마라. 이유: Y" 형식으로 적는다.
7. **네이밍** — step name은 kebab-case slug로, 해당 step의 핵심 모듈/작업을 한두 단어로 표현한다 (예: `project-setup`, `api-layer`, `auth-flow`).
8. **TDD 우선** — 기능을 설계할 때 `/docs/tests/`의 명세와 연동하여, 테스트 작성을 독립된 step으로 두거나 구현 step의 필수 선행 작업으로 명시한다.

### D. 파일 생성

사용자가 승인하면 아래 파일들을 생성한다.

#### D-1. `phases/index.json` (전체 현황)

여러 task를 관리하는 top-level 인덱스. 이미 존재하면 `phases` 배열에 새 항목을 추가한다.

```json
{
  "phases": [
    {
      "dir": "0-mvp",
      "status": "pending"
    }
  ]
}
```

- `dir`: task 디렉토리명.
- `status`: `"pending"` | `"completed"` | `"error"` | `"blocked"`. execute.py가 실행 중 자동으로 업데이트한다.
- 타임스탬프(`completed_at`, `failed_at`, `blocked_at`)는 execute.py가 상태 변경 시 자동 기록한다. 생성 시 넣지 않는다.

#### D-2. `phases/{task-name}/index.json` (task 상세)

```json
{
  "project": "<프로젝트명>",
  "phase": "<task-name>",
  "steps": [
    { "step": 0, "name": "project-setup", "status": "pending" },
    { "step": 1, "name": "core-types", "status": "pending" },
    { "step": 2, "name": "api-layer", "status": "pending" }
  ]
}
```

필드 규칙:

- `project`: 프로젝트명 (AGENT.md 참조).
- `phase`: task 이름. 디렉토리명과 일치시킨다.
- `steps[].step`: 0부터 시작하는 순번.
- `steps[].name`: kebab-case slug.
- `steps[].status`: 초기값은 모두 `"pending"`.

상태 전이와 자동 기록 필드:

| 전이 | 기록되는 필드 | 기록 주체 |
|------|-------------|----------|
| → `completed` | `completed_at`, `summary` | Gemini 세션 (summary), execute.py (timestamp) |
| → `error` | `failed_at`, `error_message` | Gemini 세션 (message), execute.py (timestamp) |
| → `blocked` | `blocked_at`, `blocked_reason` | Gemini 세션 (reason), execute.py (timestamp) |

`summary`는 step 완료 시 산출물을 한 줄로 요약한 것으로, execute.py가 다음 step 프롬프트에 컨텍스트로 누적 전달한다. 따라서 다음 step에 유용한 정보(생성된 파일, 핵심 결정 등)를 담아야 한다.

`created_at`은 execute.py가 최초 실행 시 task 레벨에 한 번만 기록한다. step 레벨의 `started_at`도 execute.py가 각 step 시작 시 자동 기록한다. 생성 시 넣지 않는다.

#### D-3. `phases/{task-name}/step{N}.md` (각 step마다 1개)

```markdown
# Step {N}: {이름}

## 읽어야 할 파일

먼저 아래 파일들을 읽고 프로젝트의 아키텍처와 설계 의도를 파악하라:

- `/docs/ARCHITECTURE.md`
- `/docs/ADR.md`
- `/docs/tests/{관련_테스트_명세}.md`
- {이전 step에서 생성/수정된 파일 경로}

이전 step에서 만들어진 코드를 꼼꼼히 읽고, 설계 의도를 이해한 뒤 작업하라.

## 작업 (TDD 원칙 준수 및 자율 개선 루프)

에이전트는 사용자 개입 없이 다음의 **계획 -> 실행 -> 평가 -> 개선 (Plan-Execute-Evaluate-Improve)** 루프를 자율적으로 반복하여 완벽한 코드를 작성해야 한다.

1. **Plan (계획)**: `/docs/tests/` 하위의 명세를 읽고 실패하는 테스트 코드를 먼저 작성한다.
2. **Execute (실행)**: 테스트를 통과시키기 위한 실제 로직을 구현하고 필요한 경우 리팩토링한다.
3. **Evaluate (평가)**: 하단의 AC 커맨드와 검증 절차(체크리스트)를 스스로 점검한다.
4. **Improve (개선)**: 테스트 실패, 린트 에러, 아키텍처/보안 위반이 하나라도 발견되면 스스로 코드를 수정하고 다시 3번(Evaluate)으로 돌아가 점검한다. 이상이 없을 때까지 이 루프를 돈다.

## Acceptance Criteria

```bash
npm run build   # 컴파일 에러 없음
npm run lint    # 린트 에러 없음
npm run test    # 테스트 통과
```

## 검증 절차 (Evaluate Phase)

1. 위 AC 커맨드를 실행하여 에러가 없는지 점검한다.
2. 아키텍처 및 보안 체크리스트를 스스로 확인한다:
   - ARCHITECTURE.md 디렉토리 구조를 따르는가?
   - ADR 기술 스택을 벗어나지 않았는가?
   - AGENT.md CRITICAL 규칙을 위반하지 않았는가?
   - **SECURITY_RULE.md의 OWASP 및 Rule of Two 보안 체크리스트를 모두 만족하는가?**
3. **루프 종료 및 상태 업데이트**: 위의 모든 평가(Evaluate)를 완벽하게 통과했을 때만 `phases/{task-name}/index.json`의 해당 step을 업데이트하고 루프를 종료한다.
   - 모든 조건 만족 시 (성공) → `"status": "completed"`, `"summary": "산출물 한 줄 요약"`
   - 스스로 코드를 개선(Improve)하며 3회 이상 재시도했음에도 실패하는 경우 → `"status": "error"`, `"error_message": "구체적 에러 내용 및 해결 시도 내역"`
   - 사용자 개입이 불가피한 물리적 제약 (API 키, 외부 인증, 수동 설정 등) → `"status": "blocked"`, `"blocked_reason": "구체적 사유"` 후 루프 즉시 중단

## 금지사항

- {이 step에서 하지 말아야 할 것. "X를 하지 마라. 이유: Y" 형식}
- 기존 테스트를 깨뜨리지 마라

#### D-4. `docs/commands/REVIEW.md` (회고 및 피드백)

각 task 또는 중요한 step 완료 후, 작업 과정에서의 핵심 결정 사항, 발생했던 이슈, 그리고 AI 에이전트의 자가 피드백을 기록하는 문서입니다. 이는 다음 작업의 품질을 높이는 컨텍스트로 활용됩니다.

### E. 실행

```bash
python3 scripts/execute.py {task-name}               # 대화형으로 브랜치 확인 후 순차 실행
python3 scripts/execute.py {task-name} --branch {b}  # 특정 브랜치에서 실행
python3 scripts/execute.py {task-name} --push        # 실행 후 push
```

execute.py가 자동으로 처리하는 것:

- **브랜치 관리** — `--branch` 옵션이 없으면 대화형으로 확인하며, 기본값은 `feat-{task-name}`입니다.
- 가드레일 주입 — AGENT.md + docs/*.md 내용을 매 step 프롬프트에 포함
- 컨텍스트 누적 — 완료된 step의 summary를 다음 step 프롬프트에 전달
- 자가 교정 — 실패 시 최대 3회 재시도하며, 이전 에러 메시지를 프롬프트에 피드백
- 2단계 커밋 — 코드 변경(`feat`)과 메타데이터(`chore`)를 분리 커밋
- 타임스탬프 — started_at, completed_at, failed_at, blocked_at 자동 기록

에러 복구:

- **error 발생 시**: `phases/{task-name}/index.json`에서 해당 step의 `status`를 `"pending"`으로 바꾸고 `error_message`를 삭제한 뒤 재실행한다.
- **blocked 발생 시**: `blocked_reason`에 적힌 사유를 해결한 뒤, `status`를 `"pending"`으로 바꾸고 `blocked_reason`을 삭제한 뒤 재실행한다.
