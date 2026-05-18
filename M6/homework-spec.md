# M6 — Домашка «Агент-Контролёр»

> **4 stages, ~7-9 часов общее время, дедлайн перед M7.** Применяешь на своём fork `proshop_mern`.

---


## TL;DR — что нужно сделать

**4 stages, ~7-9 часов общее время, дедлайн перед M7.**

| # | Stage | Что делаем | Время | Что сдаём |
|---|---|---|---|---|
| 1 | **Multi-Agent Code Review** | Запускаем 3 sub-agents последовательно (security → performance → architecture) + final synthesizer → получаем `synthesis.md` | 1.5-2 ч | Все промежуточные review + `synthesis.md` |
| 2 | **Fix Top-3** | Из `synthesis.md` берём 3 самых критичных findings, чиним через safe-refactor recipe | 1.5-2 ч | 3 diff'а + characterization tests прошли |
| 3 | **Legacy Audit + Living Documentation** | В CC запускаем `/plan` brainstorm: «спланируй audit и living docs для моего fork`а». Plan включает: project-index.json + 4-step reverse engineering на M3-M5 + новая docs/ + Python скрипт автообновления + AGENTS.md секции про поддержку. Старую docs/ архивируем. | 2.5-3.5 ч | project-index.json + docs/ specs + update_project_index.py + AGENTS.md diff + (опц.) hook |
| 4 | **Tests Agent** | Создаём `test-writer-mate.md`, прогоняем на 2 сервисах из своего кода | 1-1.5 ч | Agent definition + сгенерированные тесты + прогон |

**Все ссылки на агентов и шаблоны** — в студ-репо `aidev-course-materials/M6/`.

---

## Pre-requisites (ДО старта домашки)

- [ ] Свой fork `proshop_mern` с твоим кодом M3-M5 (MCP / RAG / feature flags / dashboard)
- [ ] Claude Code установлен, `claude --version` ≥ v2.1.32
- [ ] `ANTHROPIC_API_KEY` в env или login сделан (`claude /login`)
- [ ] Скопируй 3 mate-агентов из `aidev-course-materials/M6/6.1-ai-code-review/cloud-agent-team/agents/` в свой fork в `.claude/agents/`
- [ ] Свободный slot времени 7-9 часов на следующие 2 недели

---

## Карта папки сдачи

```
proshop_mern/homework-m6/
├── stage1-code-review/
│   ├── security-review.md           ← output security-mate
│   ├── performance-review.md        ← output performance-mate
│   ├── architecture-review.md       ← output architecture-mate
│   └── synthesis.md                 ← final synthesizer output
├── stage2-fix-top3/
│   ├── fix-1-<topic>.md             ← description + diff + reasoning
│   ├── fix-2-<topic>.md
│   ├── fix-3-<topic>.md
│   └── tests/                       ← characterization tests (ДО фиксов)
├── stage3-living-docs/
│   ├── 00-plan.md                   ← output /plan brainstorm — что и в каком порядке
│   ├── project-index.json           ← готовый файл (копия из корня fork)
│   ├── update_project_index.py      ← скрипт обновления (копия из .claude/scripts/)
│   ├── docs-new/                    ← новая структура docs/ (копия из корня fork)
│   │   ├── README.md                ← индекс
│   │   ├── specs/                   ← *-spec.md per module
│   │   ├── adr/                     ← перенесённые из archive важные ADR
│   │   └── architecture/            ← high-level overview
│   ├── docs-archived/               ← старая docs/ как была до stage 3
│   ├── AGENTS-md-diff.md            ← diff AGENTS.md ДО/ПОСЛЕ (новые секции про project-index)
│   └── hook-screenshot.png          ← (опц.) hook сработал [update-index hook] ✅
└── stage4-tests-agent/
    ├── test-writer-mate.md          ← новое agent definition (копия из .claude/agents/)
    ├── service-1-tests/             ← сгенерированные тесты для service 1
    ├── service-2-tests/             ← сгенерированные тесты для service 2
    └── coverage-report.png          ← скриншот прохождения тестов
```

---

## ⭐ Stage 1 — Multi-Agent Code Review (~1.5-2 часа)

### 🎯 Цель

Прогнать **3 специализированных sub-agents** последовательно по своему forked-проекту и получить **synthesis.md** с топ-findings. Это контрольный документ для Stage 2.

### Почему 3 sub-agents, а не один универсальный

- Универсальный агент **расфокусирован** — найдёт ~5-7 проблем
- 3 специалиста дают **15-30 findings** (security + perf + arch)
- Final synthesizer группирует, убирает дубли, выставляет приоритеты
- В live demo вы видели Agent Team — это **production-уровень** этого паттерна

### Как именно запускать — 3 опции (выбери одну)

#### Опция A: Sequential через Claude Code (рекомендуется)

В CC сессии запускаешь по очереди:

**Шаг 1.1 — Security review:**

```
Use the Agent tool to spawn security-mate (from .claude/agents/security-mate.md).
Scope: all backend/ files in my M3-M5 modules (mcp/, rag/, controllers/featureFlagController.js).
Output findings as JSON to homework-m6/stage1-code-review/security-review.md
in markdown format with severity grouping.
```

Получишь `security-review.md`. **Не переходи дальше пока не прочитаешь.**

**Шаг 1.2 — Performance review:**

```
Use the Agent tool to spawn performance-mate (from .claude/agents/performance-mate.md).
Same scope as security. Also check N+1 patterns, missing pagination, blocking I/O.
Output to homework-m6/stage1-code-review/performance-review.md.
Reference findings from security-review.md if there's overlap (e.g. ReDoS).
```

**Шаг 1.3 — Architecture review:**

```
Use the Agent tool to spawn architecture-mate (from .claude/agents/architecture-mate.md).
Same scope. Read docs/adr/* first if it exists in my fork.
Cross-reference security-review.md and performance-review.md.
Output to homework-m6/stage1-code-review/architecture-review.md.
Propose 1-2 new ADRs if you find undocumented architectural decisions.
```

**Шаг 1.4 — Final synthesizer:**

```
Read all 3 review files (security-review.md, performance-review.md, architecture-review.md).
Synthesize into homework-m6/stage1-code-review/synthesis.md:

1. Group findings by SEVERITY (HIGH / MEDIUM / LOW)
2. Within severity, group related findings (e.g. all auth-related)
3. De-duplicate cross-mate findings (if security and architecture flagged same issue)
4. Add "Recommended fix order" — top 5 to fix first
5. Add "Top-3 for Stage 2 homework" — explicit list with file:line + suggested fix approach
```

#### Опция B: Agent Team (если включил experimental hook)

Если у тебя `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` в `.claude/settings.local.json`:

```
Spawn an agent team from .claude/agents/ to review my M3-M5 modules.
3 teammates: security-mate, architecture-mate, performance-mate.
Each writes to homework-m6/stage1-code-review/<mate>-review.md.
After all 3 finish, synthesize to synthesis.md (same structure as Option A).
Use peer-to-peer mailbox for cross-category findings.
```

#### Опция C: Skills через коммандную последовательность

Если работаешь с другим coding agent (Cursor / Codex) или хочешь чтобы это запускалось в CI:

1. Запакуй каждого mate'а в shell скрипт: `scripts/run-security-review.sh`, `run-performance-review.sh`, `run-architecture-review.sh`
2. Каждый скрипт вызывает CC headless: `claude -p "$(cat .claude/agents/security-mate.md) Review M3-M5 modules. Output JSON to security-review.md"`
3. Final скрипт `synthesize.sh` объединяет все 3 reviews
4. Запускай в нужном порядке: `./scripts/full-review.sh`

### Что должно появиться в `synthesis.md`

```markdown
# Code Review Synthesis — proshop_mern (homework M6 Stage 1)

**Date:** 2026-MM-DD
**Reviewer:** 3-agent team (security + performance + architecture)
**Scope:** M3-M5 модули (MCP / RAG / feature flags)

## HIGH severity (N findings)

1. **backend/mcp/feature_flags_server.py:42** — Hardcoded JWT secret
   - Source: security-mate
   - Fix approach: move to env var + rotate
   - Estimated effort: 30 min

2. **rag/server.js:78** — N+1 query in /search endpoint
   - Source: performance-mate
   - Fix approach: batch .populate('vector_id')
   - Estimated effort: 1 hour

## MEDIUM severity (N findings)
...

## Top-3 для Stage 2

| # | File:line | Issue | Recommended fix | Effort |
|---|---|---|---|---|
| 1 | mcp/feature_flags_server.py:42 | Hardcoded JWT | Move to env | 30m |
| 2 | rag/server.js:78 | N+1 query | populate batch | 1h |
| 3 | backend/controllers/featureFlagController.js:23 | Direct file write violates AGENTS.md rule | Route through MCP | 30m |

## Cross-mate observations
...
```

### 📤 Output Checklist Stage 1 (бинарно — пройдитесь ПЕРЕД переходом на Stage 2)

#### Файлы

- [ ] `homework-m6/stage1-code-review/security-review.md` существует
- [ ] `homework-m6/stage1-code-review/performance-review.md` существует
- [ ] `homework-m6/stage1-code-review/architecture-review.md` существует
- [ ] `homework-m6/stage1-code-review/synthesis.md` существует

#### Security review

- [ ] ≥ 5 findings в `security-review.md`
- [ ] У каждого finding указан `file:line` (не общие фразы «в backend есть проблемы»)
- [ ] У каждого finding указан severity (HIGH / MEDIUM / LOW)
- [ ] У каждого finding указан OWASP category (если применимо)
- [ ] У каждого finding указан рекомендуемый fix approach

#### Performance review

- [ ] ≥ 5 findings в `performance-review.md`
- [ ] У каждого finding есть estimated impact (`+200ms p95` или `+5MB memory`)
- [ ] Минимум 1 finding касается N+1 / blocking I/O / memory
- [ ] У каждого finding указан рекомендуемый fix approach

#### Architecture review

- [ ] ≥ 5 findings в `architecture-review.md`
- [ ] Прочитаны существующие `docs/adr/*.md` (если есть) и упомянуты в reviews
- [ ] Найдены минимум 1-2 предложения новых ADR
- [ ] У каждого finding указан criticality (C1 / C2 / C3)

#### Synthesis.md

- [ ] Findings сгруппированы по SEVERITY (HIGH / MEDIUM / LOW секции)
- [ ] HIGH severity содержит **≥ 2 finding'а**
- [ ] Явная секция **«Top-3 для Stage 2»** с таблицей: file:line / issue / fix approach / effort estimate
- [ ] Cross-mate observations отмечены (минимум 1 finding флагнули ≥ 2 разных mate'а)
- [ ] Total token usage estimate (для cost awareness)

### Recipe-ссылки

- [`6.1-ai-code-review/claude-code-review-setup.md`](https://github.com/Serg1kk/aidev-course-materials/blob/main/M6/6.1-ai-code-review/claude-code-review-setup.md) — все 4 способа запуска review
- [`6.1-ai-code-review/cloud-agent-team/agents/`](https://github.com/Serg1kk/aidev-course-materials/tree/main/M6/6.1-ai-code-review/cloud-agent-team/agents) — готовые mate-агенты (security/architecture/performance)
- [`6.1-ai-code-review/honest-risks.md`](https://github.com/Serg1kk/aidev-course-materials/blob/main/M6/6.1-ai-code-review/honest-risks.md) — что не делать

---

## ⭐ Stage 2 — Fix Top-3 из synthesis.md (~1.5-2 часа)

### 🎯 Цель

Из `homework-m6/stage1-code-review/synthesis.md` берём **3 самых критичных findings** (из секции «Top-3») и чиним через safe-refactor recipe из Topic 6.3.

### Workflow для каждого из 3 fix'ов

#### Шаг 2.1 — Characterization tests ПЕРЕД фиксом

В CC сессии:

```
Read homework-m6/stage1-code-review/synthesis.md. Take finding #1 (top of list).
For the affected file (e.g. mcp/feature_flags_server.py), generate
characterization tests that pin down CURRENT behavior — even if it's wrong.

Use pytest if Python, jest if JS. Save to homework-m6/stage2-fix-top3/tests/
test-<finding-1-short-name>.py (or .js).

Run the tests. They MUST pass on current code. If they don't pass — fix the test,
not the code (we're capturing current behavior).
```

**Важно:** тесты должны проходить ДО фикса. Это «контрольная» что после фикса вы ничего не сломали.

#### Шаг 2.2 — Применить фикс с явными «Do NOT»

```
Now apply the fix for finding #1. Use this approach:

[paste recommended_fix from synthesis.md]

Constraints (Do NOT):
- Do NOT change the public API of this file
- Do NOT modify error handling logic
- Do NOT remove existing logging
- Do NOT touch any other files unless explicitly required
- Do NOT add new dependencies

After applying the fix, re-run the characterization tests from Step 2.1.
ALL tests must still pass. If any fail — STOP and explain why.
```

#### Шаг 2.3 — Документация фикса

```
Create homework-m6/stage2-fix-top3/fix-1-<short-name>.md with:
- Original finding (copy from synthesis.md)
- What I changed (diff)
- Why this approach (explain trade-offs)
- Test status (all tests passing screenshot or output)
- Lessons learned (1-2 sentences)
```

#### Повторить шаги 2.1-2.3 для findings #2 и #3

### Recipe-ссылки

- [`6.3-legacy-strategies/recipe-risk-mitigation.md`](https://github.com/Serg1kk/aidev-course-materials/blob/main/M6/6.3-legacy-strategies/recipe-risk-mitigation.md) — 7 правил безопасного рефактора
- [`6.3-legacy-strategies/legacy-incidents.md`](https://github.com/Serg1kk/aidev-course-materials/blob/main/M6/6.3-legacy-strategies/legacy-incidents.md) — что бывает когда не пишешь характеризационные тесты

### 📤 Output Checklist Stage 2 (бинарно)

#### Файлы

- [ ] `homework-m6/stage2-fix-top3/fix-1-<short-name>.md` существует
- [ ] `homework-m6/stage2-fix-top3/fix-2-<short-name>.md` существует
- [ ] `homework-m6/stage2-fix-top3/fix-3-<short-name>.md` существует
- [ ] `homework-m6/stage2-fix-top3/tests/` папка содержит тесты для **всех 3** finding'ов

#### Per fix-N.md содержит

- [ ] **Original finding** — copy-paste из `synthesis.md`
- [ ] **What I changed** — git diff (или скриншот diff'а)
- [ ] **Why this approach** — 2-3 предложения trade-offs (почему так, а не иначе)
- [ ] **Test status** — скриншот / output `pytest` / `npm test` показывает все тесты passing
- [ ] **Lessons learned** — 1-2 предложения про неочевидное (что AI пропустил / что я понял)

#### Characterization tests

- [ ] Тесты написаны **ДО** фикса (в commit history видно что test commit раньше fix commit)
- [ ] Все тесты проходят на **исходном** коде (фиксируют текущее поведение)
- [ ] Все тесты проходят на **исправленном** коде (фикс ничего не сломал)
- [ ] Минимум 3 теста на каждый finding (happy path + 1 edge case + 1 error path)

#### Качество fix'ов

- [ ] Ни один fix не нарушил публичный API (signature функций, response shapes)
- [ ] Ни один fix не добавил новые dependencies
- [ ] Каждый fix < 200 строк изменений
- [ ] Каждый fix имеет отдельный git commit с conventional message: `fix(<scope>): ...`

#### Финал

- [ ] Все 3 fix'а закоммичены и запушены
- [ ] Все характеризационные тесты в CI зелёные (если CI настроен) или локально passing

---

## ⭐ Stage 3 — Legacy Audit + Living Documentation (через `/plan`) (~2.5-3.5 часа)

### 🎯 Цель

В CC сессии запустить `/plan` brainstorm и через него получить **полный living documentation pack** для своего fork`а:

1. `project-index.json` в корне (Layer 8 — машиночитаемая карта)
2. Новая структура `docs/` со spec'ами твоих M3-M5 модулей (Layer 5 — Architecture / Specs)
3. `update_project_index.py` для автообновления (Layer 8 maintenance)
4. (опц.) Claude Code hook на PostToolUse
5. Обновлённый `AGENTS.md` с секциями про project-index + 4-step pattern
6. Старая `docs/` папка заархивирована (`docs-archived-YYYY-MM-DD/`)

**Главная идея:** `project-index.json` не делается отдельно — он **рождается** из планирования audit'а проекта. План main agent'а **включает** создание индекса как один из шагов.

### Почему planning mode

На занятии я объяснил: для multi-step analysis НЕЛЬЗЯ просто говорить «проанализируй всё». Главный агент:

1. Сначала **планирует** что и в каком порядке делать
2. Создаёт TODO с конкретными подзадачами
3. Прогоняет 4-step паттерн на каждом модуле по очереди
4. Параллельно собирает project-index.json
5. В конце создаёт новую docs/ структуру и обновляет AGENTS.md

Это и есть `/plan` mode в Claude Code (Shift+Tab+Tab или `/plan` команда).

### Workflow

#### Шаг 3.1 — Архивировать старую `docs/`

```bash
cd <your-fork>
git mv docs/ docs-archived-2026-MM-DD/
git commit -m "chore(docs): archive old docs before M6 living docs setup"
```

Старая docs/ не пропадает — она остаётся в `docs-archived-...` для истории.

#### Шаг 3.2 — Запустить `legacy-auditor-mate` как orchestrator

⭐ **Главное отличие от других стейджей:** здесь мы используем **специального orchestrator-агента** (`legacy-auditor-mate`) который САМ планирует и запускает sub-agents.

Сначала **скопируй legacy-auditor** в свой fork:

```bash
cp aidev-course-materials/M6/6.1-ai-code-review/cloud-agent-team/agents/legacy-auditor-mate.md \
   .claude/agents/legacy-auditor-mate.md
```

Verify что у тебя в `.claude/agents/` теперь 4 файла:
- `security-mate.md`
- `performance-mate.md`
- `architecture-mate.md`
- `legacy-auditor-mate.md` ⭐ новый orchestrator

Запусти CC в plan mode (Shift+Tab+Tab или `/plan` команда):

```
/plan

Act as legacy-auditor-mate (read .claude/agents/legacy-auditor-mate.md for your role).

This is M6 homework Stage 3. Goal: set up living documentation for my proshop_mern fork.

Follow your 5-phase workflow:
- Phase 1 (DISCOVERY): read AGENTS.md/CLAUDE.md/README.md/docs/adr/* (if exist),
  walk repo structure, identify subprojects, find legacy markers
- Phase 2 (PLAN): write homework-m6/stage3-living-docs/00-plan.md with full TODO list
  for Phases 3-5. Include time estimates and risks.
- Phase 3 (DISPATCH): use Task tool to spawn security-mate, performance-mate, architecture-mate
  in parallel on my M3-M5 modules (mcp/, rag/, backend/controllers/featureFlagController.js).
  Then apply 4-step reverse engineering per module yourself.
- Phase 4 (AGGREGATE): synthesize 3 mate reports + reverse-eng specs into:
  - homework-m6/stage1-code-review/synthesis.md
  - project-index.json in repo root
  - new docs/ structure (README, specs/, adr/, architecture/)
- Phase 5 (AUTOMATE): copy update_project_index.py, configure hook,
  update AGENTS.md with maintenance sections.

CRITICAL: Stay in Plan mode for Phase 1 and Phase 2. Wait for my approval before
executing Phase 3-5. Explicitly announce phase transitions.

Reference materials (read these first):
- aidev-course-materials/M6/6.2-living-documentation/multi-level-docs-stack.md
- aidev-course-materials/M6/6.2-living-documentation/keeping-docs-current.md
- aidev-course-materials/M6/6.2-living-documentation/project-index.example.json
- aidev-course-materials/M6/6.3-legacy-strategies/recipe-cc-reverse-engineering.md
- aidev-course-materials/M6/6.2-living-documentation/example-hooks/update_project_index.py
```

`legacy-auditor-mate` выполнит **Phase 1 (Discovery) + Phase 2 (Plan)** и **остановится** на запросе approval. Прочитай `00-plan.md`, дай feedback если что-то пропустил, потом скажи:

```
Plan approved. Switch to EXECUTE mode and proceed with Phase 3-5.
Update 00-plan.md with checkbox progress as you complete each TODO.
```

Auditor сам:
- Спавнит security-mate / performance-mate / architecture-mate **через Task tool**
- Получает их reports
- Применяет 4-step reverse engineering на каждом модуле
- Синтезирует в `synthesis.md`
- Строит `project-index.json`
- Создаёт новую `docs/` структуру
- Настраивает script + hook
- Обновляет AGENTS.md

**Если что-то пошло не так** — auditor обновит `00-plan.md` с пометкой проблемы и спросит тебя. Не пытайся всё делать руками — позволь orchestrator работать.

#### Шаг 3.3 — Per-module reverse engineering (4-step pattern из 6.3)

CC будет делать это в рамках плана. Для каждого модуля 4 шага:

**Промпт-шаблон для каждого модуля (CC делает это автоматически, но проверь output):**

```
Apply 4-step reverse engineering on <module-path> (e.g. mcp/feature_flags_server.py).
Read AGENTS.md first for project conventions.

Step 1 — UNDERSTAND (~2 min):
Describe business logic in plain English:
- What endpoints/functions does it expose?
- What business rules?
- What edge cases?
- What assumptions about data?
Output: ~300 words markdown.

Step 2 — DECISION TABLE (~2 min):
Generate decision table for all conditional constructs.
Columns: condition, then-action, else-action, edge_case.
Output: 10-15 row markdown table.

Step 3 — MERMAID DIAGRAM (~2 min):
Generate mermaid sequenceDiagram for main flow + 1 error path.
Include all middleware, DB queries, response.

Step 4 — EDGE CASES (~1.5 min):
List ALL edge cases: race conditions, partial failures, malicious input,
auth bypass, privilege escalation.
Output: 15-30 bullet list.

Combine all 4 outputs into docs/specs/<module>-spec.md with sections:
1. # Overview (Step 1)
2. ## Decision Table (Step 2)
3. ## Sequence Diagram (Step 3 — mermaid block)
4. ## Edge Cases (Step 4)
5. ## Open Questions (что осталось неясно)
6. ## Suggested Characterization Tests (на основе edge cases)
```

#### Шаг 3.4 — project-index.json и update script (CC делает в рамках плана)

CC сам создаст `project-index.json` на основе walked структуры репо. Проверь что включено:

- `name`, `type`, `description`, `tech_stack`
- `subprojects` — annotated (mcp, rag, backend, frontend, feature flags)
- `system_folders` — `.claude/`, `docs/`, `uploads/`
- `root_files` — annotated
- `hard_rules` — минимум 5 (включая «ALWAYS read project-index.json FIRST»)
- `ai_routing` — какие MCP-tools для каких вопросов
- `filesystem_tree` — depth 4
- `last_updated` — ISO timestamp

Скрипт `update_project_index.py` копируется из:

```bash
cp aidev-course-materials/M6/6.2-living-documentation/example-hooks/update_project_index.py \
   .claude/scripts/update_project_index.py
chmod +x .claude/scripts/update_project_index.py
```

**Адаптируй `WATCH_PATHS`** в скрипте под свои критичные директории:

```python
WATCH_PATHS = ("backend/", "frontend/src/", "mcp/", "rag/", "feature flags/")
```

Тест standalone:

```bash
python3 .claude/scripts/update_project_index.py
# Должно вывести: [update-index manual] ✅ updated project-index.json
# Или: [update-index manual] no structural change, last_updated NOT bumped
```

#### Шаг 3.5 (опционально) — Подключить Claude Code hook

В `.claude/settings.json` (или `settings.local.json`) добавь:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit|Bash",
        "hooks": [
          {
            "type": "command",
            "command": "python3 \"$CLAUDE_PROJECT_DIR/.claude/scripts/update_project_index.py\""
          }
        ]
      }
    ],
    "SessionStart": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "echo '✅ Read project-index.json first for repo structure.'"
          }
        ]
      }
    ]
  }
}
```

**Тест:** в свежей CC сессии попроси «создай test файл в backend/services/`. На stderr должно появиться:

```
[update-index hook] triggered by Write on backend/services/test.js
[update-index hook] ✅ updated project-index.json (tree + last_updated)
```

Скриншот этого момента → `homework-m6/stage3-living-docs/hook-screenshot.png`.

#### Шаг 3.6 — Обновить AGENTS.md (CC делает в рамках плана)

CC добавит две секции в начало `AGENTS.md`:

```markdown
## ⭐ START HERE — repo navigation

**ALWAYS read `project-index.json` FIRST** at the start of every session.

It contains:
- subprojects — annotated map of all subprojects
- system_folders — .claude/, docs/, uploads/ with purpose
- hard_rules — project-wide rules
- filesystem_tree — full directory tree (auto-updated, depth 4)

This file is faster than find / tree / ls, accurate, and machine-readable.

## ⭐ Keeping project-index.json current — MANDATORY

**ALWAYS** update `project-index.json` when:
- Creating a new file or folder
- Deleting / renaming files or folders
- Changing subproject purpose / description

How: `python3 .claude/scripts/update_project_index.py`
Or: auto-fires via PostToolUse hook (see .claude/settings.local.json)

For 4-step legacy analysis on new modules: see docs/specs/ for past examples.
```

#### Шаг 3.7 — Скопировать всё в submission папку

```bash
mkdir -p homework-m6/stage3-living-docs
cp project-index.json homework-m6/stage3-living-docs/
cp -r .claude/scripts/update_project_index.py homework-m6/stage3-living-docs/
cp -r docs/ homework-m6/stage3-living-docs/docs-new/
mv docs-archived-* homework-m6/stage3-living-docs/docs-archived/  # если ещё не было
# AGENTS.md diff:
git diff HEAD~ AGENTS.md > homework-m6/stage3-living-docs/AGENTS-md-diff.md
```

### Recipe-ссылки

- [`6.2-living-documentation/multi-level-docs-stack.md`](https://github.com/Serg1kk/aidev-course-materials/blob/main/M6/6.2-living-documentation/multi-level-docs-stack.md) — 8 уровней docs
- [`6.2-living-documentation/keeping-docs-current.md`](https://github.com/Serg1kk/aidev-course-materials/blob/main/M6/6.2-living-documentation/keeping-docs-current.md) — 3 уровня enforcement
- [`6.2-living-documentation/example-hooks/`](https://github.com/Serg1kk/aidev-course-materials/tree/main/M6/6.2-living-documentation/example-hooks) — готовый скрипт + settings
- [`6.2-living-documentation/hooks-reference/`](https://github.com/Serg1kk/aidev-course-materials/tree/main/M6/6.2-living-documentation/hooks-reference) — все 8 типов hooks + 7 примеров
- [`6.2-living-documentation/project-index.example.json`](https://github.com/Serg1kk/aidev-course-materials/blob/main/M6/6.2-living-documentation/project-index.example.json) — живой пример
- [`6.3-legacy-strategies/recipe-cc-reverse-engineering.md`](https://github.com/Serg1kk/aidev-course-materials/blob/main/M6/6.3-legacy-strategies/recipe-cc-reverse-engineering.md) — 4-step паттерн

### 📤 Output Checklist Stage 3 (бинарно)

#### Файлы в submission папке

- [ ] `homework-m6/stage3-living-docs/00-plan.md` (с галочками выполненных подзадач)
- [ ] `homework-m6/stage3-living-docs/project-index.json`
- [ ] `homework-m6/stage3-living-docs/update_project_index.py`
- [ ] `homework-m6/stage3-living-docs/docs-new/` (содержит README + specs + adr + architecture)
- [ ] `homework-m6/stage3-living-docs/docs-archived/` (старая docs)
- [ ] `homework-m6/stage3-living-docs/AGENTS-md-diff.md`
- [ ] (опц.) `homework-m6/stage3-living-docs/hook-screenshot.png`

#### project-index.json содержит

- [ ] Валидный JSON (`python3 -m json.tool < project-index.json` без ошибок)
- [ ] `last_updated` — сегодняшний ISO timestamp
- [ ] `subprojects` — ≥ 3 annotated entries (mcp, rag, backend минимум)
- [ ] `system_folders` — `.claude/`, `docs/`, `uploads/` отмечены
- [ ] `hard_rules` — ≥ 5 правил, включая «ALWAYS read project-index.json FIRST»
- [ ] `ai_routing` — минимум 1 routing entry (например feature_flag_questions → MCP)
- [ ] `filesystem_tree` — содержит все ключевые папки fork`а

#### Per-module specs

- [ ] Минимум **2 module spec файла** в `docs/specs/`
- [ ] Каждый spec имеет 4-6 секций: Overview / Decision Table / Sequence Diagram / Edge Cases / Open Questions / Suggested Tests
- [ ] Mermaid диаграммы рендерятся (проверить в GitHub preview)
- [ ] Edge Cases section содержит **≥ 10 пунктов** на модуль

#### update_project_index.py

- [ ] Файл существует в `.claude/scripts/`
- [ ] Executable (`-rwxr-xr-x`)
- [ ] `WATCH_PATHS` адаптирован под мой fork
- [ ] Standalone запуск работает (вывод `[update-index manual] ...`)

#### Hook (опционально)

- [ ] `.claude/settings.json` (или `.local.json`) содержит `hooks.PostToolUse`
- [ ] Hook сработал хотя бы 1 раз (screenshot в submission)

#### AGENTS.md

- [ ] Секция «⭐ START HERE» добавлена
- [ ] Секция «⭐ Keeping project-index.json current» добавлена
- [ ] Размер AGENTS.md по-прежнему ≤ 250 строк (не разрослось)

#### Архив

- [ ] `docs-archived-YYYY-MM-DD/` создан в корне fork (старая docs/ цела)
- [ ] git history показывает archive commit ДО создания новой docs/


## ⭐ Stage 4 — Tests Agent (~1-1.5 часа)

### 🎯 Цель

Создать **отдельного sub-agent для написания тестов** + прогнать его на 2 сервисах из своего кода. Это естественное продолжение Stage 4: ты уже знаешь что код делает (через specs), теперь пиши тесты которые это проверяют.

### Почему отдельный test-агент

- security-mate / performance-mate / architecture-mate — это **review-агенты** (find, not fix)
- test-writer-mate — это **write-агент** (генерирует код тестов)
- Разделение: review-агенты тестов **не пишут**, иначе они находят проблемы под свои же тесты (см. Topic 6.4 «Fake test coverage»)

### Шаг 4.1 — Создать `test-writer-mate.md`

В `.claude/agents/test-writer-mate.md` создай новый agent definition. Шаблон (адаптируй под свой стек):

```markdown
---
name: test-writer-mate
description: Test-writing specialist. Generates unit + integration tests with strong assertions (no `assert not None`). Aims for high MSI, not just coverage.
model: claude-opus-4-7
tools: [Read, Grep, Glob, Write]
when_to_use: Writing tests for new or refactored code. NOT for reviewing tests (use security/architecture mate).
category: testing
---

# Test Writer Mate

You are a Senior Test Engineer. Your role: write STRONG tests that catch real bugs,
not weak tests that just hit coverage gates.

## ROLE-LOCK

- You ONLY write test code. You don't write production code.
- You don't modify production source files.
- You don't run mutmut yourself (that's mutation-tester-mate's job).

## Strong test principles

- **Assertions must check VALUES**, not just `not None`. Bad: `assert response is not None`. Good: `assert response.status == 200 and response.body['user_id'] == 42`
- **One test = one behavior** — don't pile 5 assertions on different behaviors in one test
- **Edge cases first** — happy path is obvious, edge cases catch real bugs
- **No try-catch wrappers** that swallow exceptions
- **No `assert 2 + 2 == 4`** trivial tests just for coverage
- **Use realistic test data** — not just `"foo"` / `"bar"`, use values that mimic production

## Test types

- **Unit tests**: pure functions, single-file logic. Fast (< 1 sec each).
- **Integration tests**: with DB / HTTP / file system. Slower (1-5 sec).

For each function/endpoint you test, write:
1. 1 happy path test
2. 2-3 edge case tests (boundary values, empty input, large input)
3. 1-2 error path tests (auth fail, validation fail, DB timeout)
4. 1 security test if endpoint has security implications (injection, oversized payload)

## Output format

For each tested file, create:
- `<service>/__tests__/<file>.test.js` (or .py / .ts based on stack)

Use the existing test framework (jest for JS, pytest for Python).
Match the existing test style if there are existing tests in the project.

After writing, list the tests with one-line descriptions in markdown:

```
- test_login_happy_path: valid creds → 200 + user object
- test_login_wrong_password: invalid creds → 401
- test_login_missing_email: missing field → 400 with validation error
- ...
```

## Anti-patterns to AVOID

- Mocking everything (then you're testing the mocks, not the code)
- Tests that depend on test order
- Tests with hard-coded timestamps that fail on Tuesdays
- Tests that delete production data (use test DB!)
- `assert response is not None` — replace with value checks
```

### Шаг 4.2 — Запустить test-writer на 2 сервисах

Выбери 2 модуля из своих M3-M5 — лучше всего те, что прошли через Stage 4 reverse engineering (у тебя уже есть spec файлы для них).

В CC:

```
Use the Agent tool to spawn test-writer-mate (from .claude/agents/test-writer-mate.md).

Service 1: mcp/feature_flags_server.py
Reference spec: homework-m6/stage3-living-docs/docs-new/specs/mcp-spec.md

Write tests covering:
- All public functions/endpoints from spec
- Top 10 edge cases from spec's Edge Cases section
- 2-3 security tests if applicable

Output to mcp/__tests__/test_feature_flags_server.py

Use pytest. Match the project's existing test style.
```

Повтори для service 2 (например `rag/server.js`).

### Шаг 4.3 — Прогнать тесты

```bash
# Python
pytest mcp/__tests__/ -v

# Node
npm test rag/__tests__/
```

Если что-то падает — НЕ исправляй код. Прочитай каждый fail:
- Если **тест неправильный** (не отражает реальное поведение) — попроси agent его исправить
- Если **тест прав, а код кривой** — это твой next finding для Stage 2 (но это уже не вашу домашку)

### Шаг 4.4 (опционально, для senior) — MSI замер через mutmut

```bash
pip install mutmut==3.5.0
cd mcp/
mutmut run --paths-to-mutate feature_flags_server.py
mutmut results
```

Целевая метрика: **MSI > 70%**. Если меньше — попроси test-writer-mate улучшить assertions:

```
Read mutmut survived mutants for feature_flags_server.py.
For each surviving mutant, identify which test SHOULD have caught it but didn't.
Strengthen the assertions in that test. Output as patch to existing test file.
```

### Recipe-ссылки

- [`6.4-synthetic-testing/recipe-mini-task.md`](https://github.com/Serg1kk/aidev-course-materials/blob/main/M6/6.4-synthetic-testing/recipe-mini-task.md) — детальный 75-минутный workflow
- [`6.4-synthetic-testing/coverage-vs-msi.md`](https://github.com/Serg1kk/aidev-course-materials/blob/main/M6/6.4-synthetic-testing/coverage-vs-msi.md) — что такое MSI
- [`6.4-synthetic-testing/tools-landscape.md`](https://github.com/Serg1kk/aidev-course-materials/blob/main/M6/6.4-synthetic-testing/tools-landscape.md) — mutmut / cosmic-ray / Diffblue / PromptFuzz

### 📤 Output Checklist Stage 4 (бинарно)

#### Файлы

- [ ] `.claude/agents/test-writer-mate.md` создан в моём fork
- [ ] `homework-m6/stage4-tests-agent/test-writer-mate.md` (копия для submission)
- [ ] `homework-m6/stage4-tests-agent/service-1-tests/` — тесты для сервиса 1
- [ ] `homework-m6/stage4-tests-agent/service-2-tests/` — тесты для сервиса 2
- [ ] `homework-m6/stage4-tests-agent/coverage-report.png` — скриншот прохождения тестов

#### test-writer-mate.md содержит

- [ ] YAML frontmatter (`name`, `description`, `model: claude-opus-4-7`, `tools`)
- [ ] **ROLE-LOCK** секция (только Write test code, никогда production code)
- [ ] **Strong test principles** секция (не `assert not None`)
- [ ] Шаблон 4 типов тестов per функционал: happy / edges / error / security
- [ ] **Anti-patterns to AVOID** секция (no try-catch wrappers, no mocking everything, etc.)
- [ ] Размер ≤ 250 строк (compact, не разрастающийся)

#### Сгенерированные тесты — качество

- [ ] **2 сервиса** покрыты (минимум) из моих M3-M5 модулей
- [ ] Per service ≥ **5 test cases**: 1 happy path + 2-3 edge cases + 1-2 error paths + (опц.) 1 security
- [ ] Assertions проверяют **VALUES**: `assert response.status == 200 and body['id'] == 42` — НЕ `assert response is not None`
- [ ] **Ни одного** теста с try-catch wrapper'ом который swallow'ит exceptions
- [ ] **Ни одного** trivial теста `assert 2+2 == 4`
- [ ] Test data реалистичная (не `"foo"` / `"bar"` — а production-like values)

#### Прогон

- [ ] Все тесты **проходят** на текущем коде
- [ ] Test runner (pytest / jest) показывает stats в `coverage-report.png`
- [ ] Если есть failing tests — explain в submission README **почему** (баг в коде vs test wrong assumption)

#### Bonus (для senior — опционально)

- [ ] `mutmut` запущен на тестируемом модуле
- [ ] **Starting MSI** замерен (`starting_msi.txt`)
- [ ] Assertions улучшены на survived mutants (повторный прогон)
- [ ] **Final MSI > 70%** (`final_msi.txt`)
- [ ] Краткий analysis 1-2 параграфа: какие mutations выживали чаще всего, какие assertions добавили

---

## Cross-stage связи (важно понять)

Stage'ы **не изолированы**, они **цепочка**:

```
Stage 1 (multi-agent review)  → synthesis.md с Top-3 findings
        ↓
Stage 2 (fix Top-3)           → исправили критичное, characterization tests прошли
        ↓
Stage 3 (living docs)         → project-index.json + новая docs/ + автообновление
        │  (через legacy-auditor-mate как orchestrator)
        ↓
Stage 4 (tests agent)         → test-writer-mate + защитили тестами 2 сервиса
```

К концу домашки у тебя:
- ✅ Понимаешь что нашли AI-агенты ревью (Stage 1)
- ✅ Исправил критичное (Stage 2)
- ✅ Задокументировал систему (project-index + specs) + автообновление (Stage 3)
- ✅ Защитил тестами (Stage 4)

Это **полный цикл качества** который ты применишь в production-проектах.

---

## Что сдавать (общая папка)

```
proshop_mern/homework-m6/
├── stage1-code-review/
├── stage2-fix-top3/
├── stage3-living-docs/
└── stage4-tests-agent/
```

Ссылку на папку / PR — в студенческом Telegram-канале с хэштегом `#m6-submission` + твоё имя.

---

## Дедлайн

**Перед стартом M7** (~2 недели после M6).

Если успеешь — раньше. Если нужно дольше — напиши в чат, согласуем.

---

## Cross-module hooks (что используется откуда)

| Из | В Stage | Что используется |
|---|---|---|
| M2 (IDE setup) | все | Claude Code установлен + настроен |
| M3 (MCP) | Stage 1, 4 | review своего MCP сервера, его спецификация |
| M4 (DESIGN.md) | Stage 4 | архитектурный контекст для reverse eng |
| M5 (sub-agents) | Stage 1, 5 | паттерн запуска нескольких sub-agents |
| M6 — текущий | все | recipes из 6.1-6.4 |

---

## Полезные ссылки (студ-репо M6)

- [README.md](https://github.com/Serg1kk/aidev-course-materials/blob/main/M6/README.md) — overview модуля
- [6.1 AI Code Review](https://github.com/Serg1kk/aidev-course-materials/tree/main/M6/6.1-ai-code-review) — recipes, agents, tools-catalog
- [6.2 Living Documentation](https://github.com/Serg1kk/aidev-course-materials/tree/main/M6/6.2-living-documentation) — multi-level stack, hooks, project-index
- [6.3 Legacy Strategies](https://github.com/Serg1kk/aidev-course-materials/tree/main/M6/6.3-legacy-strategies) — reverse engineering, risk mitigation, incidents
- [6.4 Synthetic Testing](https://github.com/Serg1kk/aidev-course-materials/tree/main/M6/6.4-synthetic-testing) — recipes, tools, MSI / coverage

---

## ❓ FAQ

**Q: Можно делать стейджи не по порядку?**
A: Stage 2 требует Stage 1 (synthesis.md). Stage 4 (tests) проще делать после Stage 3 (есть specs модулей). Остальные — в любом порядке.

**Q: Что если у меня в M3-M5 мало кода / простой код?**
A: Возьми чужой код Brad Traversy слоя как backup: `backend/controllers/userController.js`, `orderController.js`. Они достаточно сложные.

**Q: Можно использовать GPT / Cursor вместо Claude Code?**
A: Можно, но recipes написаны под Claude Code (Agent tool, hooks). Адаптируй: Cursor — `Cmd+K` + chat mode, GPT — через ChatGPT app. Submission должна быть та же — папки + файлы.

**Q: Что если hook не срабатывает?**
A: Проверь `claude --version` ≥ v2.1.32, путь к скрипту, `chmod +x`, settings.json валидный. Если не получается — пропусти Stage 3 hook часть (это опционально), запускай `update_project_index.py` руками.

**Q: Сколько времени реально потратишь?**
A: Минимум **7 часов** если делаешь честно. Если поверхностно (без characterization tests, без MSI) — 4-5 часов, но reviewer (я) увидит и поставит ниже.

---

*Этот файл — DRAFT v2. После твоего ревью закоммитим в `aidev-course-materials/M6/homework-spec.md`.*
