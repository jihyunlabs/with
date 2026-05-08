---
name: with-chain-normal
description: 중간 깊이 with 체인 (brainstorm → plan → codex:adv → build → codex:review → with-review). 도메인은 알고 있고 접근 옵션 비교가 필요한 일반 기능용. --no-codex 로 Codex 패스.
model: opus
---

# /with-chain-normal

중간 깊이 with 체인 — 도메인은 이미 알고 있지만 접근 방식 비교 + atomic plan 이 필요한 일반 기능용. grill / office-hours 는 패스, brainstorm 부터 시작.

## 입력

`/with-chain-normal <feature 설명> [--no-codex]`

## 플래그

- `--no-codex` — Codex 단계 (adversarial-review · review) 패스. 기본은 모두 포함.

## 자동 advance vs 게이트

**자동 advance** (Claude 가 다음 스킬 호출):
- brainstorm 모든 sections OK → plan
- plan 작성 → codex:adversarial-review (`--no-codex` 면 skip)
- codex 통과 → build
- build 모든 task 완료 → codex:review (`--no-codex` 면 skip)
- codex 통과 → with-review
- with-review `clean` → ship 안내

**게이트** (사용자 입력 대기, 자동 X):
- brainstorm 옵션 선택 + 각 section OK
- build 의 Confusion Protocol 발동 시 (모르는 가정)
- codex 가 blocker / critical 반환 → 사용자 판단
- with-review 의 critical / high 위반 → 사용자 fix 결정

## 동작 순서

1. feature 설명 read · 플래그 파싱.
2. /with-brainstorm 실행 (옵션 선택 + sections OK 대기).
3. /with-plan 실행 → `.claude/plans/<file>.md` 생성.
4. `--no-codex` 아니면 /codex:adversarial-review 호출. blocker 있으면 사용자 판단 대기.
5. /with-build 실행 (TDD, atomic commit per task).
6. `--no-codex` 아니면 /codex:review 호출. blocker 있으면 사용자 판단 대기.
7. /with-review 실행.
8. clean 이면 `gh pr create` 명령 안내 — **자동 실행 X** (ship 은 사용자 결정).

## 보고 (각 stage 종료 시)

```
▸ with-chain-normal (stage <N>/6: <stage>)
  status: ✓ done · ⏸ user gate · ✗ blocked
  next: <다음 스킬 또는 사용자 액션>
```

체인 종료:

```
▸ with-chain-normal 완료
  stages: <completed list>
  artifacts: plan · commits <N>
  next: gh pr create -t "<feature title>"
```

## 언제 normal 을 쓰는가

- 도메인 용어 · 결정은 이미 명확함
- 제품 premise 는 의심할 필요 없음 (build 결정 끝)
- 그러나 접근 방식이 여러 개 — brainstorm 으로 비교 필요
- atomic plan + verify 가 정당한 규모 (3~10 파일)

벗어나면:
- 도메인 자체 불명확 → `/with-chain-deep` (grill 부터)
- 접근 명백 + plan 불필요 → `/with-chain-light` (즉시 build)

## Karpathy 4 적용

- **Think Before Coding** — brainstorm sections OK, plan verify 필수. 게이트 자동 통과 X.
- **Simplicity First** — grill / office-hours 패스. 도메인 이미 알고 있음.
- **Surgical Changes** — 각 스킬은 자기 역할만.
- **Goal-Driven Execution** — plan 의 `→ verify:` 강제.

## 안 하는 것

- grill / office-hours 자동 호출 X — 필요하면 사용자가 직접 또는 deep 사용.
- 사용자 결정 게이트 자동 통과 X.
- codex blocker / with-review critical 위반 후 ship 안내 X.
- `gh pr create` 자동 실행 X — 사용자 ship 결정.
- 같은 task 의 codex rescue 2회 이상 X.
