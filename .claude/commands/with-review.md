---
name: with-review
description: 현재 diff 를 Karpathy 4원칙으로 검사. 위반만 보고 (severity). Codex 와 별도 internal final check.
model: sonnet
---

# /with-review

`/codex:review` 와 분리 — Codex 는 **설계 · tradeoff** 도전, /with-review 는 **Karpathy 4 위반** 만 본다.

## 입력

`git diff` (uncommitted) 또는 `git diff <base>..HEAD` (브랜치 비교).

## 검사 항목 (각 hunk)

### 1. Surgical Changes 위반
- 사용자 요청 · plan task 에 직접 trace 안 되는 줄?
- 인접 코드 "개선" (변수명 · 포맷 · 주석)?
- 안 망가진 함수 리팩터?
- 기존 dead code 삭제?
→ 위반이면 **critical**, 해당 hunk 명시.

### 2. Simplicity First 위반
- 1회용 코드 추상화 (단 한 번 호출되는 인터페이스 · base class)?
- 요청 안 한 "유연성" · 옵션 파라미터?
- 일어날 수 없는 시나리오 에러 처리?
- 50줄 가능한 게 200줄?
→ 위반이면 **high**.

### 3. Think Before Coding 위반
- 명시 안 된 가정으로 침묵 결정?
- ambiguity 있었는데 안 묻고 한 쪽으로 결정?
→ 위반이면 **high**.

### 4. Goal-Driven Execution 위반
- 새 코드인데 verify 명령 없음?
- 테스트 없는 새 분기?
→ 위반이면 **medium**.

## 출력 형식

```
▸ with-review

[critical]
  src/auth/login.ts:42-58
  Surgical Changes 위반 — login 핸들러 외 logger 포맷 변경.
  요청 trace: 없음.
  권고: logger 변경 revert.

[high]
  src/utils/cache.ts (전체)
  Simplicity First 위반 — 1회 호출 함수에 generic factory + adapter 패턴.
  권고: 직접 호출로 단순화 (~30줄 절감 추정).

[medium]
  src/api/orders.ts:120
  Goal-Driven Execution 위반 — 새 분기 'partial fulfillment' 에 테스트 없음.
  권고: → verify: pnpm test src/api/__tests__/orders.partial.test.ts

위반 없으면: "▸ with-review · clean"
```

## 안 하는 것

- 스타일 코멘트 X (린터 영역).
- 성능 권고 X (`/codex:adversarial-review` 영역).
- 보안 권고 X (별도 단계).
- "더 좋게 쓸 수 있는 방법" 일반론 X — 4 원칙 위반만.
