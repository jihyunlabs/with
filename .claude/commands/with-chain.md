---
name: with-chain
description: idea → ship 전 체인을 자동 orchestrate. 사용자 결정 게이트(질문 / section OK / blocker)에서만 멈춤. --quick 으로 grill·office-hours 패스, --no-codex 로 Codex 패스.
model: opus
---

# /with-chain

전체 with 체인을 자동 orchestrate. /with-* 스킬을 순서대로 호출 + 사용자 결정 게이트에서만 정지. Karpathy "Think" 는 자동 통과 X.

## 입력

`/with-chain <feature 설명> [--quick] [--no-codex]`

## 플래그

- `--quick` — /with-grill · /with-office-hours 패스 (작은 task). /with-brainstorm 부터 시작.
- `--no-codex` — Codex 단계 (adversarial-review · review) 모두 패스.
- 둘 다 동시 사용 가능.

## 자동 advance vs 게이트

**자동 advance** (Claude 가 다음 스킬 호출):
- grill 종료 → office-hours
- office-hours decision `build` → brainstorm
- brainstorm 모든 sections OK → plan
- plan 작성 → codex:adversarial-review (`--no-codex` 면 skip)
- codex 통과 → build
- build 모든 task 완료 → codex:review (`--no-codex` 면 skip)
- codex 통과 → review
- review `clean` → ship 안내

**게이트** (사용자 입력 대기, 자동 X):
- grill 인터뷰의 모든 질문
- office-hours 6 forcing questions 답변
- office-hours decision = `pivot` / `kill` → 체인 정지 (build 안 함)
- brainstorm 옵션 선택 + 각 section OK
- build 의 Confusion Protocol 발동 시 (모르는 가정)
- codex 가 blocker / critical 반환 → 사용자 판단
- review 의 critical / high 위반 → 사용자 fix 결정

## 동작 순서

1. feature 설명 read · 플래그 파싱.
2. `--quick` 아니면 /with-grill 실행 (사용자 답변 대기).
3. `--quick` 아니면 /with-office-hours 실행. decision = pivot/kill 이면 정지.
4. /with-brainstorm 실행 (옵션 선택 + sections OK 대기).
5. /with-plan 실행 → `.claude/plans/<file>.md` 생성.
6. `--no-codex` 아니면 /codex:adversarial-review 호출. blocker 있으면 사용자 판단 대기.
7. /with-build 실행 (TDD, atomic commit per task).
8. `--no-codex` 아니면 /codex:review 호출. blocker 있으면 사용자 판단 대기.
9. /with-review 실행.
10. clean 이면 `gh pr create` 명령 안내 — **자동 실행 X** (ship 은 사용자 결정).

## 보고 (각 stage 종료 시)

```
▸ with-chain (stage <N>/<total>: <stage>)
  status: ✓ done · ⏸ user gate · ✗ blocked
  next: <다음 스킬 또는 사용자 액션>
```

체인 종료:

```
▸ with-chain 완료
  stages: <completed list>
  artifacts: ADRs <N> · plan · commits <N>
  next: gh pr create -t "<feature title>"
```

## Karpathy 4 적용

자동 체인이지만:
- **Think Before Coding** — 게이트 자동 통과 X. 답변·OK·결정 필수.
- **Simplicity First** — `--quick` 으로 작은 task 단축. 옵션 1개 강요 X.
- **Surgical Changes** — 각 스킬은 자기 역할만. 인접 단계 침범 X.
- **Goal-Driven Execution** — 각 스킬의 `→ verify:` 그대로 작동.

## 안 하는 것

- 사용자 결정 게이트 자동 통과 X.
- pivot / kill decision 후 build 진행 X.
- codex blocker / review critical 위반 후 ship 안내 X.
- `gh pr create` 자동 실행 X — 사용자 ship 결정 (Karpathy "Think").
- 같은 task 의 codex rescue 2회 이상 X.
