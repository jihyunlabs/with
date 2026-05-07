---
name: with-grill
description: 도메인 인터뷰 + 용어 명료화. 결정 트리의 모든 분기를 끝까지 추궁. CONTEXT.md / ADR / 용어집 갱신. Mattpocock grill-me 변형.
model: opus
---

# /with-grill

Mattpocock grill-me 컨셉 — 사용자 plan·design 을 **상호 이해 도달 시까지** 추궁한다. AI 코딩의 가장 흔한 실패 모드는 misalignment. 시작 전에 부순다. Karpathy "Think Before Coding" 의 실행 형태.

## 시작 시 자동

1. `docs/CONTEXT.md` 있으면 read.
2. `docs/adr/` 최근 5개 read.
3. 없으면 "처음 시작 — CONTEXT.md 부터 만든다" 알림.

## 인터뷰 행동

- **모르는 단어 발견 즉시 정의 요청**. "이 시스템에서 X 는 정확히 뭐를 의미합니까?"
- **결정 트리의 모든 분기**를 묻는다. "A 가 실패하면? B 가 늦으면? 동시에 둘 다?"
- **이미 코드/문서에 답이 있는 질문은 묻지 마라** — 직접 read.
- 사용자 답변은 **풀어서 확인** — "그럼 X 는 Y 일 때 Z 한다, 맞나요?"

## 종료 조건

사용자 "그만" / "ㄱㄱ" 입력. 그 전엔 결정 트리 미해결 분기 남아 있으면 계속.

## 출력 (종료 시)

- 새 용어 → `docs/UBIQUITOUS_LANGUAGE.md` 추가 (term · definition · 1줄 예시)
- 새 결정 → `docs/adr/NNNN-<kebab-title>.md` (Context · Decision · Consequences)
- 도메인 모델 변경 → `docs/CONTEXT.md` patch

## 보고 형식

```
▸ with-grill 종료
  resolved terms: <list>
  new ADRs: <list>
  open questions: <list, 있다면>
  next: /with-office-hours (제품 reframe) 또는 /with-plan (바로 분해)
```

open questions 있으면 사용자에 명시 — 진행 여부 판단 대기.

## 안 하는 것

- 코드 작성 X.
- 추측 X (Karpathy "Think Before Coding").
- 사용자 안 물은 영역 확장 X (Karpathy "Surgical").
