---
name: with-plan
description: atomic task 분해 + verify 강제. Opus 가 청사진. 출력은 .claude/plans/<feature>.md. Karpathy Goal-Driven Execution.
model: opus
---

# /with-plan

Opus 청사진. 구현 전 단계 — 비용 거의 0. 결과를 `/codex:adversarial-review` 에 넣어 깰 수 있다.

## 입력

- `/with-brainstorm` 의 chosen approach + design sections (있다면)
- 없으면 사용자 feature 설명 + `docs/CONTEXT.md` + 관련 ADR

## 시작 시 자동

1. `.claude/plans/` 같은 ID plan 충돌 검사. 있으면 보고 + 사용자 선택 대기.
2. CONTEXT.md / 관련 ADR read.

## 분해 규칙 (Karpathy Goal-Driven)

- 각 task **atomic** — 독립적으로 commit / revert 가능.
- 각 task 에 `→ verify: <정확한 명령>` 강제. 예:
  - `→ verify: pnpm test src/auth/__tests__/login.test.ts`
  - `→ verify: curl -X POST localhost:3000/api/login | jq '.token'`
- "make it work", "should function correctly" 같은 약한 기준 금지.

## 출력 형식

`.claude/plans/<feature-kebab>.md`:

```markdown
# <feature title>

## Context
2-3 줄. CONTEXT.md / ADR / brainstorm approach 참조.

## Tasks

### 1. <task title>
- <줄 단위 행동>
- → verify: <명령>

### 2. <task title>
- ...
- → verify: <명령>

## Out of scope
요청 안 한 것 명시 (Karpathy "Simplicity First").
```

## 보고 형식

```
▸ with-plan 작성
  path: .claude/plans/<file>.md
  tasks: <N>
  next: /codex:adversarial-review .claude/plans/<file>.md
```

## 안 하는 것

- 요청 범위 밖 task 추가 X.
- "future-proofing" task X.
- 추상화·인프라 task 가 사용자 요청에 직접 trace 안 되면 X.
