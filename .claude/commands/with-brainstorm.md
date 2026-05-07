---
name: with-brainstorm
description: Superpowers brainstorming — 대안 탐색, design sections 으로 검증. 코드 구현 전 마지막 결정 단계. Confusion Protocol 적용.
model: opus
---

# /with-brainstorm

Superpowers brainstorming 변형 — premise 가 잠긴 상태에서 **how to build** 를 탐색. /with-plan 직전 단계.

## 입력

- `/with-office-hours` 의 premise + decision (있다면)
- 최근 ADR + CONTEXT.md

## 행동

1. 접근 옵션 **3개 제시** (간단 / 표준 / 야심) — 옵션 1개만 제시 X (Karpathy "Think Before Coding" — 다중 해석 명시).
2. 각 옵션의 trade-off 표 — perf / 복잡도 / 시간 / lock-in.
3. 사용자 1개 선택 또는 합성 요청 대기.
4. 선택된 옵션을 **design section 으로 분해** — UI / data / control flow / failure mode.
5. 각 section 사용자 OK 받기 — **하나라도 OK 안 나면 /with-plan 진행 X**.

## Confusion Protocol (Superpowers)

모르는 부분 발견 즉시 멈추고 질문 — 추측 X. 예:

> "rate-limit 의 keying 이 user-id 인지 token-id 인지 결정 안 됐습니다. 어느 쪽?"

Karpathy "Think Before Coding" 과 동일 — 같은 행동의 다른 이름.

## 출력

- chosen approach 1 줄
- design sections (각 OK 표시)
- → `docs/adr/NNNN-<kebab>.md` (Decision: <approach>, Context, Consequences)

## 보고 형식

```
▸ with-brainstorm
  approach: <선택된 옵션>
  sections OK: UI ✓ data ✓ control ✓ failure ✓
  next: /with-plan
```

## 안 하는 것

- 코드 작성 X.
- 옵션 1개만 제시하고 진행 X.
- 사용자 OK 없는 section 으로 plan 단계 진행 X.
