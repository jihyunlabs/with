---
name: with-chain-deep
description: idea → ship 까지 풀 사이클 (grill → office-hours → brainstorm → plan → codex:adv → build → codex:review → with-review). 새 시스템·큰 기능·도메인 불명확. 사용자 결정 게이트에서만 정지. --no-codex 로 Codex 패스.
model: opus
---

# /with-chain-deep

가장 깊은 with 체인 — 도메인부터 ship 까지 전 단계 자동 orchestrate. 새 시스템·큰 기능·도메인 모델 불명확한 작업용. 작은 task 는 `/with-chain-normal` 또는 `/with-chain-light` 사용.

## 입력

`/with-chain-deep <feature 설명> [--no-codex]`

## 플래그

- `--no-codex` — Codex 단계 (adversarial-review · review) 패스. 기본은 모두 포함.

## 자동 advance vs 게이트

**자동 advance** (Claude 가 다음 스킬 호출):
- grill 종료 → office-hours
- office-hours decision `build` → brainstorm
- brainstorm 모든 sections OK → plan
- plan 작성 → codex:adversarial-review (`--no-codex` 면 skip)
- codex 통과 → build
- build 모든 task 완료 → codex:review (`--no-codex` 면 skip)
- codex 통과 → with-review
- with-review `clean` → ship 안내

**게이트** (사용자 입력 대기, 자동 X):
- grill 인터뷰의 모든 질문
- office-hours 6 forcing questions 답변
- office-hours decision = `pivot` / `kill` → 체인 정지 (build 안 함)
- brainstorm 옵션 선택 + 각 section OK
- build 의 Confusion Protocol 발동 시 (모르는 가정)
- codex 가 blocker / critical 반환 → 사용자 판단
- with-review 의 critical / high 위반 → 사용자 fix 결정

## 동작 순서

1. feature 설명 read · 플래그 파싱.
2. /with-grill 실행 (사용자 답변 대기).
3. /with-office-hours 실행. decision = pivot/kill 이면 정지.
4. /with-brainstorm 실행 (옵션 선택 + sections OK 대기).
5. /with-plan 실행 → `.claude/plans/<file>.md` 생성.
6. `--no-codex` 아니면 /codex:adversarial-review 호출. blocker 있으면 사용자 판단 대기.
7. /with-build 실행 (TDD, atomic commit per task).
8. `--no-codex` 아니면 /codex:review 호출. blocker 있으면 사용자 판단 대기.
9. /with-review 실행.
10. clean 이면 `gh pr create` 명령 안내 — **자동 실행 X** (ship 은 사용자 결정).

## 보고 (각 stage 종료 시)

```
▸ with-chain-deep (stage <N>/8: <stage>)
  status: ✓ done · ⏸ user gate · ✗ blocked
  next: <다음 스킬 또는 사용자 액션>
```

체인 종료:

```
▸ with-chain-deep 완료
  stages: <completed list>
  artifacts: ADRs <N> · plan · commits <N>
  next: gh pr create -t "<feature title>"
```

## 언제 deep 을 쓰는가

- 새로운 시스템 / 모듈 처음 만들 때
- 도메인 용어 · 결정 트리가 불명확
- 제품 premise 자체를 의심해야 할 때 (build/pivot/kill)
- 영향 범위 큰 리팩터 (>10 파일, 여러 모듈)

작은 task 는:
- `/with-chain-normal` — brainstorm + plan + build (도메인은 알고 있음)
- `/with-chain-light` — 명확화 질문 후 즉시 build (버그 fix · 작은 기능)

## Karpathy 4 적용

자동 체인이지만:
- **Think Before Coding** — 게이트 자동 통과 X. 답변·OK·결정 필수.
- **Simplicity First** — deep 은 정말 큰 task 에만. 작은 task 는 light/normal.
- **Surgical Changes** — 각 스킬은 자기 역할만. 인접 단계 침범 X.
- **Goal-Driven Execution** — 각 스킬의 `→ verify:` 그대로 작동.

## 안 하는 것

- 사용자 결정 게이트 자동 통과 X.
- pivot / kill decision 후 build 진행 X.
- codex blocker / with-review critical 위반 후 ship 안내 X.
- `gh pr create` 자동 실행 X — 사용자 ship 결정 (Karpathy "Think").
- 같은 task 의 codex rescue 2회 이상 X.
