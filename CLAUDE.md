# CLAUDE.md

`with` — Karpathy 4원칙 base + gstack 결정층 + Superpowers 실행층 + Codex 2-model 결합 경량 framework.

---

## 0. 우선순위 (충돌 시)

1. **Karpathy 4** (base, 절대)
2. 이번 턴 사용자 직접 지시
3. 이 파일 + `.claude/commands/`
4. Codex 외부 권고 (평가 후 채택/기각)

규칙끼리 충돌하면 1→4 순서. 채택/기각 사유 한 줄 보고.

---

## 1. Karpathy 4 — Base Layer

- **Think Before Coding** — 가정 명시 · 모르면 질문 · 해석 다중이면 제시 · 막히면 멈추고 이름 짓는다.
- **Simplicity First** — 요청 외 기능 X · 1회용 추상화 X · 일어날 수 없는 시나리오 에러 X · 200→50줄 가능하면 다시.
- **Surgical Changes** — 요청에 직접 trace 되는 줄만 · 인접 개선 X · 기존 스타일 유지 · 네 변경이 만든 orphan 만 정리.
- **Goal-Driven Execution** — 모든 task 를 `→ verify: <명령>` 으로 변환 · "make it work" 금지.

---

## 2. 체인 워크플로우 (idea → ship)

```
아이디어
  ↓
/with-grill          # 도메인 + 용어 (Mattpocock)
  ↓
/with-office-hours   # YC-partner reframe (gstack)
  ↓
/with-brainstorm     # 대안 + design sections (Superpowers)
  ↓
/with-plan           # atomic plan + verify (Karpathy)
  ↓
/codex:adversarial-review   # plan 깨기 (사용자 명시)
  ↓
/with-build          # Sonnet TDD 실행 (Karpathy + Superpowers TDD)
  ↓
/codex:review        # diff 깨기 (사용자 명시)
  ↓
/with-review         # Karpathy 4 final check
  ↓
ship (gh pr create — 직접)
```

각 단계 **사용자 명시 호출**. 자동 chain X. Codex 단계 비용 고려해 패스 가능. 작은 task 는 grill·office-hours 패스 — Karpathy Simplicity 우선.

---

## 3. 스킬 매핑 (6개, `with-` 접두어)

| 명령 | 출처 | 단계 | 모델 | 출력 |
|---|---|---|---|---|
| `/with-grill` | Mattpocock + Karpathy Think | 도메인화 | opus | `docs/CONTEXT.md` · `UBIQUITOUS_LANGUAGE.md` · `adr/NNNN-*.md` |
| `/with-office-hours` | gstack | reframe | opus | 6 forcing questions · premise · build/pivot/kill |
| `/with-brainstorm` | Superpowers | 대안 + 검증 | opus | 접근 옵션 · design sections · 사용자 OK |
| `/with-plan` | Karpathy Goal-Driven | atomic 분해 | opus | `.claude/plans/<feature>.md` (각 task 에 `→ verify:`) |
| `/with-build` | Karpathy + Superpowers TDD | 구현 | sonnet | RED-GREEN-REFACTOR atomic commit |
| `/with-review` | Karpathy 4 | final check | sonnet | 4원칙 위반 보고 (severity) |

Codex 명령은 plugin 직접 — wrapper X: `/codex:setup` · `/codex:review` · `/codex:adversarial-review` · `/codex:codex-rescue`.

---

## 4. 자기모순 해결 (4 출처)

1. **Karpathy "Surgical" vs gstack 23 skill bloat** → with 는 6 개로 축소. gstack 의 `/review` `/qa` `/ship` 등 실행층 제외 — 그 자리는 Superpowers TDD + 직접 `gh`.
2. **Karpathy "Simplicity" vs Superpowers "mandatory full cycle"** → 작은 task 는 grill·office-hours 패스, plan + build 만. Karpathy 우선.
3. **Mattpocock grill vs gstack office-hours 중복** → grill = 도메인 모델 (용어·결정), office-hours = 제품 reframe (whether to build). 역할 분리, 둘 다 유지.
4. **Codex 권고가 Karpathy 4 위반** → 버린다. 사용자에 한 줄 보고.

---

## 5. Codex 통합

호출 시점 (사용자 명시):
- plan 직후 → `/codex:adversarial-review .claude/plans/<file>.md`
- 구현 직후 → `/codex:review`
- ship 직전 → `/codex:adversarial-review` (마지막 도전)
- 큰 task 위임 → `/codex:codex-rescue --background`

권고 평가:
1. Karpathy 4 위반? → 버린다
2. trade-off? → 사용자 판단 대기
3. align? → 채택

큰 diff → `--background` 강제, Claude 컨텍스트 0 추가. Rescue 무한루프 방지: 같은 task 2회 이상 X. 결과는 Claude 가 직접 검토 후 적용.

---

## 6. 모델 라우팅

| 작업 | 모델 |
|---|---|
| grill · office-hours · brainstorm · plan | **opus** |
| build · review | **sonnet** |
| grep · test · 단순 조회 | **haiku** (subagent) |
| 적대적 리뷰 · 2nd opinion | **codex** (외부) |

`.claude/settings.json` 적용:
- `model: claude-opus-4-7`
- `MAX_THINKING_TOKENS=10000`
- `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE=50`
- `CLAUDE_CODE_SUBAGENT_MODEL=claude-haiku-4-5-20251001`

---

## 7. 메모리

```
docs/
├── CONTEXT.md              # /with-grill 도메인 모델
├── UBIQUITOUS_LANGUAGE.md  # 용어집
└── adr/                    # /with-grill·office-hours·brainstorm 결정

.claude/
├── plans/                  # /with-plan 출력
└── commands/with-*.md      # 6 스킬
```

자동 read: grill → CONTEXT + 최근 ADR 5개 / plan → 충돌 검사 / build → 해당 plan. 자동 write 는 명시 호출된 명령 안에서만.

---

## 8. 토큰 위생

| 컨텍스트 | 행동 |
|---|---|
| 40% | subagent 위임 |
| 55% | 새 세션 권장 |
| 70% | 즉시 중단 + hand-off doc |
| 큰 codex 리뷰 | `--background` 강제 |

---

## 9. 자체 모순 발견 시

이 spec 안에서 모순 발견 → Karpathy 4 우선 → 사용자에 보고 → 판단 대기. 자체 해결 X.
