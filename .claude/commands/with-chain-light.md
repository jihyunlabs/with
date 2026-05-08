---
name: with-chain-light
description: 가장 가벼운 with 체인 — 1-3개 명확화 질문 후 즉시 build → codex:review. 버그 fix·작은 기능·접근 명백한 task 용. plan/brainstorm 패스.
model: opus
---

# /with-chain-light

가장 가벼운 with 체인 — 짧은 명확화 Q&A 후 바로 코드 작성. 버그 fix · 작은 기능 · 접근 방식이 이미 명백한 task 용. plan / brainstorm / grill / office-hours 모두 패스. Codex review 는 그대로 포함.

## 입력

`/with-chain-light <task 설명> [--no-codex]`

## 플래그

- `--no-codex` — /codex:review 패스. 기본은 포함.

## 자동 advance vs 게이트

**자동 advance**:
- 명확화 질문 0개 (이미 명확) → 즉시 build
- 명확화 답변 받음 → build
- build 완료 → codex:review (`--no-codex` 면 skip)
- codex 통과 → ship 안내

**게이트** (사용자 입력 대기):
- 명확화 질문 1-3개 답변 (Karpathy "Think Before Coding" — 가정 명시)
- build 의 Confusion Protocol 발동 시 (모르는 가정)
- codex 가 blocker / critical 반환 → 사용자 판단

## 동작 순서

1. task 설명 read · 플래그 파싱.
2. **명확화 단계** — task 에서 모호한 부분 식별:
   - 0개면 패스 (정말 명확한 경우만, 예: "X 함수의 typo 수정")
   - 1-3개 질문 → 사용자 답변 대기
   - 4개 이상 떠오르면 → 이건 light 가 아님. `/with-chain-normal` 권장하고 정지.
3. /with-build 실행 (TDD, atomic commit).
4. `--no-codex` 아니면 /codex:review 호출. blocker 있으면 사용자 판단 대기.
5. clean 이면 `gh pr create` 안내 — **자동 실행 X**.

## 명확화 질문 가이드라인

좋은 질문 (light 에 적합):
- "X 의 입력 타입은 어떤 거?"
- "에러 처리 방식 (throw / null 반환 / 기본값)?"
- "기존 Y 함수 재사용 vs 새로 만들기?"

나쁜 질문 (light 에 부적합 — normal 로):
- "전체 아키텍처는?"
- "이 기능을 정말 만들어야 하는가?"
- "여러 접근 방식의 trade-off 는?"

## 보고

```
▸ with-chain-light (stage <N>/3: <stage>)
  status: ✓ done · ⏸ user gate · ✗ blocked
  next: <다음 액션>
```

체인 종료:

```
▸ with-chain-light 완료
  stages: clarify · build · codex:review
  artifacts: commits <N>
  next: gh pr create -t "<title>"
```

## 언제 light 를 쓰는가

- 버그 fix (재현 가능 + 원인 추정 가능)
- 작은 기능 추가 (3 파일 이하)
- 접근 방식이 코드 읽으면 명백
- 도메인 / 제품 의문 없음

벗어나면:
- 접근 옵션 여러 개 → `/with-chain-normal`
- 도메인 자체 모름 → `/with-chain-deep`

## Karpathy 4 적용

- **Think Before Coding** — 명확화 질문 단계 강제. 0개 통과는 정말 명백한 경우만.
- **Simplicity First** — light 의 핵심. plan / brainstorm 강요 X. 작은 task 는 작게.
- **Surgical Changes** — build 가 task 직접 trace 줄만 손댐. 인접 개선 X.
- **Goal-Driven Execution** — codex:review 가 verify. 통과 못하면 fix.

## 안 하는 것

- 4개 이상 명확화 질문 떠올랐는데 light 강행 X — normal 로 escalate.
- /with-plan 자동 호출 X (light 의 정의 자체).
- 사용자 명확화 답변 받기 전 build 진입 X.
- codex blocker 무시 후 ship 안내 X.
- `gh pr create` 자동 실행 X — 사용자 결정.
