---
name: with-office-hours
description: gstack 스타일 YC-partner reframe. 6 forcing questions 으로 제품 premise · 대안 · 10-star 버전 도전. whether to build 단계.
model: opus
---

# /with-office-hours

gstack `/office-hours` 변형 — 코드 짜기 전에 **whether to build** 를 도전한다. /with-grill 의 도메인 명료화 다음 단계, /with-brainstorm 의 how 탐색 직전.

## 입력

사용자 feature 설명 + (있다면) `docs/CONTEXT.md` + 최근 ADR.

## 6 forcing questions

1. **Why now?** — 지금 안 하면 안 되는 이유.
2. **Who breaks first?** — 가장 먼저 망가지는 사용자 / 시스템.
3. **What's the 10-star version?** — 야망 최대치.
4. **What's the throwaway 1-star?** — 검증용 최소.
5. **What's the cheapest experiment?** — 1주일 안 검증법.
6. **What hides as commodity?** — 외부 도구로 충분한 부분.

각 질문 답을 받으면 **풀어서 확인** + 모순 지적. 답이 코드/문서에 이미 있으면 read.

## 출력

- premise 1 줄
- 대안 N개 (ranked, build / pivot / kill 후보)
- killshot 가설 — 이게 거짓이면 전부 무너지는 1개 가정
- → `docs/adr/NNNN-<kebab>.md` (Decision: build / pivot / kill, Context, Consequences)

## 보고 형식

```
▸ with-office-hours
  premise: <한 줄>
  decision: build | pivot | kill
  killshot: <검증할 가설>
  next: /with-brainstorm (how 탐색) 또는 /with-plan
```

## 안 하는 것

- 코드 작성 X · 구현 디테일 X (그건 /with-plan 영역).
- 사용자가 이미 commit 한 결정 재도전 X (Karpathy "Surgical").
- 답이 이미 ADR / CONTEXT 에 있는 질문 다시 묻기 X.
