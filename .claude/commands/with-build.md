---
name: with-build
description: plan 의 atomic task 를 TDD 로 실행. Sonnet 이 RED-GREEN-REFACTOR + atomic commit. Karpathy 4 + Superpowers TDD 적용.
model: sonnet
---

# /with-build

`.claude/plans/<file>.md` 의 task 를 하나씩 실행. Sonnet 이 main, 큰 grep · test 는 haiku subagent.

## 입력

`/with-build <plan_path>` 또는 사용자가 plan 경로 제시.

## 시작 시 자동

1. plan 파일 read.
2. 같은 plan 의 이전 commit 검사 (`git log --grep`).
3. 첫 미완료 task 시작.

## task 실행 루프 (Superpowers TDD)

각 task 마다:

1. **계획 한 줄** — 이 task 변경 파일 · 줄 수 추정.
2. **Karpathy 자기 검사**:
   - 사용자 요청 trace? (Surgical)
   - 더 단순한 방법? (Simplicity)
   - 모르는 가정? (Think) — 있으면 멈추고 질문.
3. **RED** — 실패 테스트 먼저 작성, 실행, 빨간색 확인.
4. **GREEN** — 최소 코드로 통과.
5. **REFACTOR** — 통과 유지하며 정리. 인접 코드 개선 X (Surgical).
6. **`→ verify:` 명령 실행** — 통과 못 하면 다음 task 진행 X.
7. **atomic commit** — 메시지 형식: `<plan-id>: <task-title>`.

## 절대 안 하는 것 (Karpathy)

- task 외 파일 수정 X (요청에 trace 안 됨).
- "while I'm here" 식 인접 개선 X.
- 안 망가진 거 리팩터 X.
- 일어날 수 없는 시나리오 에러 처리 X.
- 1회용 코드 추상화 X.

## 모르는 거 발견 시

멈추고 사용자에 질문 — 추측 X. 예:

> "task 3 의 'rate limit 적용' 에서 어떤 백엔드를 쓸지가 plan 에 없습니다. Redis / 메모리 / 외부 서비스 중 무엇입니까?"

## 보고 형식 (각 task 완료 시)

```
▸ with-build (task <N>/<total>)
  changed: <files>
  RED: ✓  GREEN: ✓  REFACTOR: ✓
  verify: ✓ <명령 결과>
  commit: <sha>
  next: task <N+1> 또는 /codex:review
```

## 모든 task 완료 시

```
▸ with-build 완료
  total commits: <N>
  next: /codex:review
```

사용자에 `/codex:review` 권장하고 대기 — 자동 호출 X.
