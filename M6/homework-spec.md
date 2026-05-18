# M6 — Домашка «Агент-Контролёр»

> **5 stages, общее время ~6-8 часов.** Сдача — перед стартом M7. Каждая стадия имеет track-выбор по сложности; берите тот, который соответствует вашему стеку и темпу.

## Главный нарратив

```
M2  поставили IDE + Claude Code
M3  написали MCP-сервер для feature flags
M4  редизайн админ-панели в Figma + код
M5  агентский workflow в n8n + GCAO + sub-agents
─────────────────────────────────────────
M6  АУДИТОР всего этого + рефактор старого
    BradTraversy слоя как тренировка
```

В M6 вы переходите от «строю систему» к «контролирую качество системы». **Агент-Контролёр** = sub-agent или agent team, которая проходит по вашему forked-проекту, находит проблемы (security, perf, arch), а вы их фиксите по safe-refactor recipe.

## Submission

Создайте папку `homework-m6/` в своём fork'е `proshop_mern` и сохраните туда артефакты каждой стадии:
- `homework-m6/stage1-audit/findings.png`
- `homework-m6/stage2-refactor/diff-before-after.md`
- `homework-m6/stage3-docs/CHANGES.md`
- `homework-m6/stage4-legacy/refactored-file.{js,ts}` + screenshot тестов
- `homework-m6/stage5-mutation/msi-report.md` (опционально)

Ссылку на PR / папку — в студ-канал с меточкой `#m6-submission` + имя.

---

## Stage 1 — Создание агента-аудитора (~1.5-2 часа)

**🎯 Цель:** запустить аудит на `proshop_mern` (или своём fork) и получить top-10 findings (security / arch / perf).

### Track-выбор (выберите ОДИН)

🟢 **Track A (basic, ~1.5 ч):** 1 универсальный агент-аудитор по baseline из LMS 6.1. Один `.claude/agents/auditor.md` файл с system prompt по 4 типам проверок (security/perf/arch/testing).

🟢 **Track B (advanced, ~2 ч):** Agent Team из 2 mate'ов через `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`. Минимум — `security-mate.md` + `architecture-mate.md`, role-locked, с непересекающимися tools.

🟢 **Track C (со ⭐, ~3 ч):** ОБА варианта (Track A + Track B) + сравнение результатов «2-vs-1». Прогоняете один и тот же PR через single-agent и agent-team — сравниваете количество найденных проблем, false positives, время выполнения.

### Подробный recipe

См. [`6.1-ai-code-review/recipe-agent-team-setup.md`](6.1-ai-code-review/recipe-agent-team-setup.md) — пошагово, с примерами `.claude/agents/*.md` файлов и `settings.json`.

### Bonus (cross-module M3)

Подключите ваш M3 RAG как **context provider** для агента — в system prompt дайте инструкцию делать MCP-запросы в ваш RAG за specific правилами / спецификой проекта перед выдачей findings. Это даёт агенту grounded context вместо «вообще-всё-что-знаю».

### Submission Stage 1

- Скриншот списка findings (top 10)
- `.claude/agents/*.md` файлы — копия в `homework-m6/stage1-audit/`
- Если Track C — таблица сравнения «2-vs-1»: метрики (TP, FP, время, tokens)

---

## Stage 2 — Рефакторинг своего legacy (~1.5-2 часа)

**🎯 Цель:** взять **3 топ-finding'а** из Stage 1 и исправить их через safe-refactor recipe из Topic 6.3.

### Целевые модули

**ВАШ СВОЙ КОД** из M3-M5: `mcp/`, `rag/`, `feature flags/`, M4-redesigned screens. **НЕ BradTraversy слой** (для него — Stage 4).

### Recipe (полный — из Topic 6.3)

См. [`6.3-legacy-strategies/recipe-risk-mitigation.md`](6.3-legacy-strategies/recipe-risk-mitigation.md). Кратко:

1. **Characterization tests ДО** изменения — зафиксируйте поведение «как оно работает сейчас» (даже если оно кривое)
2. **AI refactor с явными «Do NOT»** — пишете ограничения, а не пожелания: «Do NOT change public API. Do NOT modify error handling. Do NOT remove existing logging.»
3. **Маленькие PR** — каждый fix отдельным commit'ом, <200 строк изменений
4. **Feature flag rollout** — 10% → 50% → 100% (для production-кода)
5. **Parallel run / golden-master** — старая и новая версии работают параллельно на одинаковый input, diff outputs

### Submission Stage 2

- 3 diff'а до / после в `homework-m6/stage2-refactor/diff-before-after.md`
- Тесты прошли (CI скриншот или вывод `npm test` / `pytest`)
- Для самого важного fix — короткий explanation **почему это была ошибка** (1-2 предложения)

---

## Stage 3 — Living Documentation на разных уровнях (~1.5-2 часа)

**🎯 Цель:** превратить ваш fork `proshop_mern` в self-documenting проект, где AI-агент с первой сессии понимает структуру, conventions и архитектурные решения. Покрыть минимум **4 из 8** уровней документации (см. [`6.2-living-documentation/multi-level-docs-stack.md`](6.2-living-documentation/multi-level-docs-stack.md)).

### Что должно появиться (mandatory)

#### 1. **`project-index.json`** в корне fork'а (Layer 8) ⭐

Машиночитаемый граф структуры. Шаблон-эталон: [`6.2-living-documentation/project-index.example.json`](6.2-living-documentation/project-index.example.json).

Минимум секций:
- `name`, `type`, `description`
- `subprojects` — annotated map ваших подпроектов (backend / frontend / mcp / rag / feature-flags / прочее)
- `system_folders` — `.claude/`, `docs/`, `uploads/`
- `root_files` — root .md/.json/.yml
- `hard_rules` — минимум 5 правил specific to ваш fork
- `ai_routing` — какие MCP инструменты для каких вопросов
- `filesystem_tree` — depth 4, auto-generated
- `last_updated` — ISO timestamp UTC

**Prompt для Claude Code (копируй-вставляй):**

```
Read AGENTS.md and walk the repo structure (max depth 4, skip node_modules / .git / build).
Identify all subprojects and system folders. Read existing docs/adr/* if present.

Build a `project-index.json` in the repo root following this schema:
- name, type, description, tech_stack
- subprojects (annotated: path, tech, entry, key_paths, owner)
- system_folders (annotated: purpose)
- root_files (annotated: description)
- hard_rules (extracted from AGENTS.md + my own additions)
- ai_routing (MCP routing hints)
- filesystem_tree (dict-of-arrays, folder → children, depth 4)
- last_updated (ISO timestamp UTC)

Validate JSON with `python3 -m json.tool` before saving. Aim for 200-400 lines.
```

#### 2. **`update_project_index.py`** + Claude Code hooks (Layer 8 maintenance) ⭐

Чтобы index не отставал от реальности — настройте автообновление.

Скопируйте шаблон из [`6.2-living-documentation/keeping-docs-current.md`](6.2-living-documentation/keeping-docs-current.md) → секция «Setup — `update_project_index.py` script».

Положите в `.claude/scripts/update_project_index.py`, сделайте executable (`chmod +x`).

Добавьте в `.claude/settings.json` (или `settings.local.json`):

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit|Bash",
        "hooks": [{ "type": "command", "command": "python3 \"$CLAUDE_PROJECT_DIR/.claude/scripts/update_project_index.py\"" }]
      }
    ],
    "SessionStart": [
      {
        "hooks": [{ "type": "command", "command": "echo '✅ Read project-index.json first.'" }]
      }
    ]
  }
}
```

После настройки **проверьте**: создайте новый файл через CC → должно сразу обновиться `last_updated` в `project-index.json`.

#### 3. **Обновлённый `AGENTS.md` / `CLAUDE.md`** в корне fork'а (Layer 8)

Дополните существующий `AGENTS.md` под **ВАШИ** M3-M5 модули. Добавьте секции:

- **«⭐ START HERE — repo navigation»** — указание читать `project-index.json` FIRST
- **«⭐ Keeping project-index.json current — MANDATORY»** — правила обновления + ссылка на hook
- **«Custom additions (M3-M5)»** — описание ваших MCP / RAG / feature-flags / dashboard модулей с conventions
- **«AI routing»** — какой MCP-tool для какого вопроса (feature-flag → MCP, не features.json)

**Размер:** держите **под 200 строк**. Если больше — выносите детали в `docs/`, оставляйте указатели.

Эталон-шаблон: посмотрите на свежий `AGENTS.md` в основном `proshop_mern` (с секциями ⭐ START HERE и ⭐ Keeping project-index.json) — Сергей покажет на demo M6 BEAT 4.

#### 4. **Минимум 1 ADR** в `docs/adr/` (Layer 5)

Architecture Decision Record для самого важного архитектурного решения **в вашем коде** (не Brad Traversy слой). Кандидаты:

- ADR для вашего M3 MCP-сервера — почему выбрали именно MCP, а не REST microservice или DB layer?
- ADR для вашего RAG — какую vector DB выбрали, почему такие chunk-size + overlap?
- ADR для feature flags — почему JSON file vs ConfigDB vs LaunchDarkly?
- ADR для M5 n8n agent pipeline — почему n8n, а не CrewAI / LangGraph?

Формат — Nygard (Context / Decision / Consequences / Alternatives). ~30-80 строк.

### Что должно появиться (optional bonus, +0.5 часа)

#### 5. **Per-subproject `README.md`** в каждом из ваших M3-M5 подпроектов (Layer 2)

`mcp/README.md`, `rag/README.md`, `feature flags/README.md` (если есть). Каждый — 30-50 строк:
- Role / Owner / Quick-start
- Endpoints / API surface
- Dependencies
- Local development commands

#### 6. **Inline comments** в 2-3 critical paths (Layer 1)

Места где AI исторически ошибается — добавьте якоря по паттерну из [`6.2-living-documentation/recipe-inline-comments.md`](6.2-living-documentation/recipe-inline-comments.md). Например:

- `mcp/feature_flags_server.py` — `# CONSTRAINT: env_file IS source of truth. Do NOT add ARGS to Dockerfile.`
- `backend/controllers/userController.js` — `// SECURITY: this returns BOTH token AND user object — frontend depends on it. Don't change shape without migration.`
- `rag/server.js` — `// PERFORMANCE: chunk_size=512 chosen after benchmarking; don't lower without testing.`

#### 7. **OpenAPI auto-spec** (Layer 3) — для senior'ов кто успеет

Подключите `swagger-jsdoc` к Express routes, сгенерируйте `docs/openapi.json` через build. Это даёт typed contract для frontend и для будущих API consumers.

### Pattern «прогрессивное раскрытие»

Не пытайтесь сделать все 8 уровней. Минимум для проекта типа `proshop_mern`:

| Layer | У вас должно быть | Где |
|---|---|---|
| 1 (Code) | Inline-комментарии в critical paths | в коде |
| 2 (Repo) | README в каждом подпроекте | mcp/README.md, rag/README.md, etc. |
| 5 (Architecture) | Минимум 1 ADR | docs/adr/ |
| 8 (AI-context) | project-index.json + AGENTS.md + hooks | repo root + .claude/ |

**Не нужно:** Backstage catalog (Layer 4), production runbooks (Layer 6), Wiki (Layer 7) — это для команд от 10 человек.

### Submission Stage 3

В `homework-m6/stage3-docs/`:

- Ссылка на ваш `project-index.json` в fork (на GitHub)
- Скриншот diff'а `AGENTS.md` ДО / ПОСЛЕ ваших изменений
- Содержимое `.claude/settings.json` (только секция `hooks`, без секретов)
- Один ADR в Markdown (~30-80 строк)
- (опционально) Список путей где добавили inline-комментарии

**Optional bonus submission:**
- Per-subproject README ссылки
- OpenAPI JSON (если успели)

### Recipe-ссылки (mandatory reading)

- [`6.2-living-documentation/multi-level-docs-stack.md`](6.2-living-documentation/multi-level-docs-stack.md) — 8 уровней с таблицами «как обновляется каждый»
- [`6.2-living-documentation/keeping-docs-current.md`](6.2-living-documentation/keeping-docs-current.md) — 3 уровня enforcement (prompt → CC hooks → CI), с готовым `update_project_index.py` шаблоном
- [`6.2-living-documentation/agents-md-claude-md-standard.md`](6.2-living-documentation/agents-md-claude-md-standard.md) — формат AGENTS.md, optimal size, anti-patterns
- [`6.2-living-documentation/recipe-inline-comments.md`](6.2-living-documentation/recipe-inline-comments.md) — где ставить inline комментарии
- [`6.2-living-documentation/project-index.example.json`](6.2-living-documentation/project-index.example.json) — живой пример (из proshop_mern)

---

## Stage 4 — Reverse Engineering на чужом Legacy (~1.5-2 часа)

**🎯 Цель:** разобрать **2-3 файла чужого** legacy кода (Brad Traversy 2020 слой в `proshop_mern`) через 4-step reverse engineering pattern из Topic 6.3. На выходе — markdown-спецификации, которые становятся базой для characterization tests и автоматически индексируются в `project-index.json`.

> **Что такое reverse engineering** (обратная разработка): берёте файл legacy кода без документации, через 4 точных промпта Claude'у получаете полную спеку — описание бизнес-логики, decision table, sequence diagram, edge cases. Раньше senior сидел 2-3 часа, с AI это ~7-8 минут на файл.

### Связь со Stage 3

Stage 3 поднимал у вас `project-index.json` + hooks. Stage 4 это естественно использует: вы создадите новую папку `docs/reverse-engineering/`, hook автоматически обновит индекс. То есть Stage 3 и Stage 4 — **связанная цепочка**: сначала вы поднимаете инфру (Stage 3), потом она **сама работает** на вас (Stage 4).

### Track-выбор (выберите ОДИН — по вашему стеку и темпу)

🟢 **Basic (~1 час) — 1 файл:**
- `backend/seeder.js` — старый seeder с callbacks. Хороший для тренировки на простом файле без бизнес-логики.

🟢 **Backend ⭐ (~1.5 часа) — 2 файла рекомендуется:**
- `backend/controllers/userController.js` — Express auth с manual JWT (это файл из live demo)
- `backend/controllers/orderController.js` — orders + payment status (более сложный, есть классические уязвимости с client-supplied prices)

🟢 **Frontend (~1.5 часа) — 2 файла:**
- `frontend/src/reducers/productReducers.js` — Redux thunks 2020
- `frontend/src/actions/productActions.js` — соответствующие actions

🔴 **Advanced со ⭐ (~2.5 часа) — 3 файла:**
- Полный auth-stack: `userController.js` + `authMiddleware.js` + `generateToken.js`
- Покажет cross-file dependencies, которые при single-file подходе пропустишь

### Workflow для КАЖДОГО выбранного файла

Применяете **4 шага** из Topic 6.3 в отдельной Claude Code сессии для каждого файла. Каждый шаг = один промпт + сохранение output'а.

#### Шаг 1 — ПОНЯТЬ (~2 мин)

В CC сессии (открыта в корне fork`а):

```
Read backend/controllers/userController.js (плюс соседние файлы middleware/authMiddleware.js
и utils/generateToken.js если они есть).

Опиши бизнес-логику этого файла простыми словами:
- Какие endpoints он экспортирует?
- Какие бизнес-правила enforced?
- Какие edge cases он обрабатывает (или НЕ обрабатывает)?
- Какие предположения о данных он делает (например "user всегда есть в БД")?

Output: markdown ~300 слов. Сохрани в docs/reverse-engineering/userController-step1-understand.md
```

#### Шаг 2 — DECISION TABLE (~2 мин)

```
Для того же файла userController.js сгенерируй decision table:
все условные конструкции (if/switch/ternary).

Колонки: условие | then-action | else-action | edge case | severity

Output: markdown-таблица 10-15 строк.
Сохрани в docs/reverse-engineering/userController-step2-decisions.md
```

#### Шаг 3 — MERMAID DIAGRAM (~2 мин)

```
Сгенерируй mermaid sequenceDiagram для главного flow (например POST /api/users/login).
Включи:
- Happy path
- 1 error path (например wrong password)
- Все middleware calls
- DB queries
- Response к клиенту

Output: mermaid code block в файле docs/reverse-engineering/userController-step3-diagram.md
```

#### Шаг 4 — EDGE CASES (~1.5 мин)

```
Перечисли ВСЕ edge cases для каждого endpoint в userController.js.
Включи особенно неочевидные:
- Race conditions
- Partial failures (DB up but slow)
- Malicious input (SQL/NoSQL injection, oversized payload)
- Auth bypass scenarios
- Privilege escalation paths

Output: bullet list 15-30 пунктов в docs/reverse-engineering/userController-step4-edges.md
```

#### Шаг 5 — COMBINE → specification.md (~30 сек)

```
Combine все 4 outputs (step1-step4 files) в один файл
docs/reverse-engineering/userController-specification.md.

Структура:
1. # Overview (из step 1)
2. ## Decision Table (из step 2)
3. ## Sequence Diagram (mermaid из step 3)
4. ## Edge Cases (из step 4)
5. ## Open Questions (что осталось неясно)
6. ## Suggested Characterization Tests (минимум 10 test cases на основе edge cases)

~5K токенов финальной спеки.
```

После Шага 5 — **hook автоматически** обновит `project-index.json` (новая папка `docs/reverse-engineering/`). Если hook настроен правильно (Stage 3), это произойдёт само.

### Bonus (~30 мин если время осталось)

Из последнего шага у вас есть **«Suggested Characterization Tests»**. Попросите Claude:

```
Generate the actual test code for the first 5 characterization tests from
userController-specification.md. Use jest + supertest. Save to
__tests__/characterization/userController.test.js
```

Запустите тесты на текущем (немодифицированном) коде. Все 5 должны пройти. Это **ваша контрольная** на случай если в M7 или потом будете рефакторить файл.

### Submission Stage 4

В `homework-m6/stage4-reverse-engineering/`:

- **Per файл:** один объединённый `<filename>-specification.md` (~5K токенов markdown)
- Ссылка на коммит где появилась папка `docs/reverse-engineering/`
- Скриншот `project-index.json` где видна новая папка (= hook сработал)
- (опционально) `__tests__/characterization/<file>.test.js` + скриншот прохождения тестов
- 1-2 предложения **рефлексии**: что AI заметил в коде такого, чего вы при первом чтении пропустили

### Самопроверка качества

Хороший reverse engineering output отвечает на эти вопросы про код:

- [ ] Я понимаю **зачем** этот файл существует в продукте (не «что делает», а «зачем»)
- [ ] Я знаю **минимум 5 неочевидных** edge cases
- [ ] Я могу нарисовать flow на доске не глядя в код
- [ ] Я знаю **3 потенциальные уязвимости** в этом файле (security / performance / business logic)
- [ ] У меня есть **сценарий разговора** с автором файла (если бы он был доступен) — какие 5 вопросов я бы ему задал

Если хотя бы 3 из 5 — ✅, цель Stage 4 достигнута.

### Recipe-ссылки (mandatory reading)

- [`6.3-legacy-strategies/recipe-cc-reverse-engineering.md`](6.3-legacy-strategies/recipe-cc-reverse-engineering.md) — детальный 4-step recipe с примерами output'ов
- [`6.3-legacy-strategies/recipe-risk-mitigation.md`](6.3-legacy-strategies/recipe-risk-mitigation.md) — 7 правил безопасного refactor (используете когда будете править найденные проблемы)
- [`6.3-legacy-strategies/legacy-incidents.md`](6.3-legacy-strategies/legacy-incidents.md) — реальные production-инциденты от AI на legacy (читать чтобы понимать что НЕ делать)

---

## Stage 5 ⭐ — Synthetic Testing mini-task (опциональная, ~75 мин)

**🎯 Цель:** pre-experience того, что демонстрировалось в Topic 6.4 — mutation testing на собственном модуле.

> Эта стадия **опциональная** — для senior-эшелона и тех, кто успеет.

### 3 шага

1. **Test gen (30 мин):**
   ```
   /test src/your_python_module.py
   ```
   Используете Claude Code `/test` команду для генерации pytest-кейсов на ваш модуль.

2. **Mutmut baseline (30 мин):**
   ```bash
   pip install mutmut==3.5.0
   mutmut run
   mutmut results
   ```
   Получаете baseline MSI (mutation score index) — обычно ~50-60% после первой генерации.

3. **Improve assertions (15 мин):**
   Скопируйте топ-10 surviving mutants в Claude Code prompt, попросите улучшить assertions. Перезапустите mutmut.

### Целевая метрика

**MSI > 70%** — это означает, что 70%+ модификаций вашего кода ваши тесты ловят.

### Watch-outs (важно!)

- **mutmut 3.x НЕ мутирует код вне функций** (regression от 2.x). Если ваш код — top-level statements, mutmut пропустит их.
- **Windows → WSL обязательно** (mutmut требует `fork()`).
- **Cosmic-ray 8.4.6** — Windows-friendly альтернатива, если WSL нет.

### Recipe

См. [`6.4-synthetic-testing/recipe-mini-task.md`](6.4-synthetic-testing/recipe-mini-task.md) — пошагово, с примерами prompts для Claude Code.

### Connectivity

**Совет:** возьмите для Stage 5 тот же модуль, что в Stage 4 — получится «один файл = весь pipeline»: reverse engineering → refactor → docs → tests → mutation testing.

### Submission Stage 5

- Starting MSI vs final MSI
- Краткий analysis (1-2 параграфа): какие типы mutations выживают, почему, что добавили в assertions

---

## Cross-module hooks

| Из M6 → |  Куда |
|---|---|
| Stage 1 bonus | M3 RAG (context provider) |
| Все stages | M2 IDE setup (любая IDE с CC) |
| Stage 3 | M4 DESIGN.md (документация UI слоя) |
| Stage 1 / Stage 2 reviewer prompts | M5 GCAO + sub-agents |

## Helpful-services из этой темы

См. главный каталог в [resources/tools-catalog.md](resources/tools-catalog.md).
