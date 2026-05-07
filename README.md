# `with` — Karpathy + gstack + Superpowers + Codex 경량 framework

4 출처를 자기모순 없이 조합한 Claude Code framework. 6 스킬, 단일 체인, CLAUDE.md ~150줄.

| 출처 | 기여 |
|---|---|
| [Karpathy guidelines](https://github.com/forrestchang/andrej-karpathy-skills) | 4 행동 원칙 (base layer) |
| [gstack](https://github.com/garrytan/gstack) | 결정층 (`/office-hours` 변형) |
| [obra/superpowers](https://github.com/obra/superpowers) | 실행층 (brainstorming + TDD + verify) |
| [codex-plugin-cc](https://github.com/openai/codex-plugin-cc) | 2nd opinion (adversarial · review · rescue) |

(Mattpocock skills 의 grill-me 컨셉도 `/with-grill` 디자인 참고)

---

## 한 문장

> **grill 로 도메인, office-hours 로 whether, brainstorm 으로 how, plan 으로 atomic, build 로 TDD, codex 로 깨기, review 로 Karpathy final.**

비용 ⅓, 버그 −50% (codex-plugin-cc 사용자 보고).

---

## 체인

```
아이디어
  ↓
/with-grill          # 도메인 + 용어 (Mattpocock)
  ↓
/with-office-hours   # YC-partner reframe (gstack)
  ↓
/with-brainstorm     # 대안 + design sections (Superpowers)
  ↓
/with-plan           # atomic + verify (Karpathy)
  ↓
/codex:adversarial-review  # plan 깨기
  ↓
/with-build          # Sonnet TDD (Karpathy + Superpowers)
  ↓
/codex:review        # diff 깨기
  ↓
/with-review         # Karpathy 4 final
  ↓
ship (gh pr create)
```

각 단계 사용자 명시 호출. 자동 chain X. Codex 단계 비용 고려해 패스 가능. 작은 task 는 grill·office-hours 패스 가능.

---

## 자기모순 X — 4 검증

1. **Karpathy "Surgical" vs gstack 23 skill bloat** → with 는 6 개로 축소. gstack 의 `/review` `/qa` `/ship` 실행층 제외, 그 자리는 Superpowers TDD + 직접 `gh`.
2. **Karpathy "Simplicity" vs Superpowers "mandatory full cycle"** → 작은 task 는 grill·office-hours 패스, plan + build 만.
3. **gstack `/office-hours` vs Mattpocock grill 중복** → grill = 도메인 모델 (용어·결정), office-hours = 제품 reframe (whether to build). 역할 분리.
4. **Codex 권고가 Karpathy 4 위반** → 버린다. 한 줄 보고.

**우선순위**: Karpathy 4 > 사용자 직접 지시 > 이 framework > Codex 권고.

---

## 설치

### 1. framework

```bash
git clone https://github.com/jihyunlabs/with.git tmp-with
cp tmp-with/CLAUDE.md ./
cp -r tmp-with/.claude ./
rm -rf tmp-with
```

### 2. Codex plugin (필수)

Claude Code 안에서:

```
/plugin marketplace add openai/codex-plugin-cc
/plugin install codex@openai-codex
/codex:setup
```

ChatGPT 계정 또는 OpenAI API key 필요.

### 3. (선택) Mattpocock skills

```bash
npx skills@latest add mattpocock/skills/grill-me
npx skills@latest add mattpocock/skills/ubiquitous-language
```

---

## 디렉토리

```
your-project/
├── CLAUDE.md                # 행동 spec (~150줄)
├── .claude/
│   ├── settings.json        # 모델 + 토큰 env
│   ├── commands/
│   │   ├── with-grill.md
│   │   ├── with-office-hours.md
│   │   ├── with-brainstorm.md
│   │   ├── with-plan.md
│   │   ├── with-build.md
│   │   └── with-review.md
│   └── plans/               # /with-plan 출력
└── docs/
    ├── CONTEXT.md
    ├── UBIQUITOUS_LANGUAGE.md
    └── adr/
```

---

## 6 스킬

| 명령 | 출처 | 모델 | 역할 |
|---|---|---|---|
| `/with-grill` | Mattpocock + Karpathy Think | opus | 도메인 인터뷰 → CONTEXT / ADR / 용어집 |
| `/with-office-hours` | gstack | opus | 6 forcing questions → premise / build·pivot·kill |
| `/with-brainstorm` | Superpowers | opus | 대안 + design sections + Confusion Protocol |
| `/with-plan` | Karpathy Goal-Driven | opus | atomic task + verify |
| `/with-build` | Karpathy + Superpowers TDD | sonnet | RED-GREEN-REFACTOR atomic commit |
| `/with-review` | Karpathy 4 | sonnet | 4 원칙 위반만 (severity) |

Codex plugin 직접 호출:
- `/codex:review` · `/codex:adversarial-review` · `/codex:codex-rescue` · `/codex:setup`

---

## 모델 라우팅

| 작업 | 모델 |
|---|---|
| grill · office-hours · brainstorm · plan | opus |
| build · review | sonnet |
| 단순 grep · test · 조회 | haiku (subagent) |
| 적대적 리뷰 · 2nd opinion | codex (외부) |

`settings.json`:
- `MAX_THINKING_TOKENS=10000`
- `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE=50`
- `CLAUDE_CODE_SUBAGENT_MODEL=claude-haiku-4-5-20251001`

---

## Codex 운영

- 호출 시점: plan 직후 · 구현 직후 · ship 직전 · rescue (모두 사용자 명시)
- 권고 평가: Karpathy 4 통과 → 채택, 위반 → 버린다. 한 줄 보고
- 큰 diff: `--background` 강제. 컨텍스트 0
- Rescue 무한루프: 같은 task 2회 이상 X. 결과는 Claude 직접 검토

---

## License

MIT
