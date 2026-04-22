# Rules-файлы по 10 AI-IDE — справочник для M2 homework

> **Кому:** студенты курса AI-Driven Development, M2 — AI-IDE.
> **Зачем:** домашка требует настроить rules-файл для твоей IDE. Этот справочник — одна страница на каждую IDE: что создать, куда положить, как не сломать.
> **Дата:** 2026-04-22. 2025-2026 sources, все ссылки верифицированы.

---

## 1. Overview — картина целиком (читать 2 минуты)

**Rules-файл = system prompt твоего проекта.** Без него AI-IDE пишет «generic» код: не знает твой стек, твои имена, твои конвенции. С ним ты перестаёшь каждый раз объяснять «я маркетолог, я использую Next.js 15, у нас camelCase, а эти тесты нельзя трогать».

**Что происходит в индустрии (начало 2026):**

- **`AGENTS.md` оформляется как кросс-IDE стандарт.** Родился в OpenAI Codex в 2025, в декабре 2025 отдан в Agentic AI Foundation (Linux Foundation). Поддерживают: Codex, OpenCode, Amp, Jules (Google), Cursor, Factory. Claude Code читает его как fallback, Antigravity с v1.20.3 (март 2026) — нативно. 60K+ репозиториев на GitHub используют. Next.js, Payload, OpenAI main repo (88 AGENTS.md файлов!) — production references. ([agents.md][1], [augmentcode][2])
- **Два лагеря по поиску по коду:**
  - **RAG-с-embeddings** (Cursor, Windsurf, Copilot Workspace) — индексирует репо в vector DB, быстрый fuzzy-поиск, но индекс «протухает» при активном редактировании.
  - **Agentic search** (Claude Code, Codex CLI, OpenCode, Gemini CLI) — никакого индекса, просто `grep`+`read`+`glob`. Anthropic: «мы попробовали RAG рано — agentic выиграл by a lot.» ([Cherny / Zerofilter][3])
- **Почему это важно для тебя:** формат rules-файла отражает, как IDE читает код. RAG-IDE хотят компактные rules (контекст дорогой → token budget жёсткий). Agentic-IDE терпят чуть больше текста, но деградируют после ~200 строк («context rot»).

**Главное правило, применимое ко ВСЕМ IDE:**
- Держи ≤150-200 строк (Anthropic official: «худой» CLAUDE.md).
- Секции: **WHAT** (стек, архитектура) + **HOW** (команды, конвенции) + **NOT** (анти-паттерны).
- Примеры > описания: 5 строк примера > 50 строк объяснения.
- Обновляй при каждом regress: поймал себя на повторении одного и того же промпта — это сигнал добавить в rules.

---

## 2. Decision matrix — одной таблицей

| # | IDE | Rules-файл | Формат | Soft limit | Ключевая фишка / gotcha |
|---|---|---|---|---|---|
| 1 | **Claude Code** | `CLAUDE.md` + `~/.claude/CLAUDE.md` + `.claude/rules/*.md` | Markdown + `@import` | ≤150 строк (Anthropic) | Hierarchy: Enterprise → User → Project → Local; `@import` до 5 hops глубины |
| 2 | **Cursor** | `.cursor/rules/*.mdc` (+ legacy `.cursorrules`) | Markdown + YAML frontmatter | Контекст-бюджет Cursor | `alwaysApply` / `globs` / `description` — 4 режима активации |
| 3 | **OpenAI Codex CLI** | `AGENTS.md` (+ `AGENTS.override.md`) | Markdown | 32 KiB (hard) | `project_doc_max_bytes` в конфиге; override-файл для временных правок |
| 4 | **OpenCode** | `AGENTS.md` + `~/.config/opencode/AGENTS.md` | Markdown | Не указан | Читает `CLAUDE.md` как fallback; `opencode.json` с `instructions[]` |
| 5 | **Windsurf (Cascade)** | `.windsurfrules` (global + project) | Markdown | Не указан | Memories auto-save между сессиями; Fast Context subagent |
| 6 | **Gemini CLI** | `GEMINI.md` (global + project + subfolder) | Markdown | Не указан | `/memory show`, `/memory refresh`; `context.fileName` в `settings.json` расширяет до `AGENTS.md` |
| 7 | **Google Antigravity** | `.agents/rules/*.md` + `GEMINI.md` / `AGENTS.md` | Markdown + activation mode | Не указан | 4 режима: Manual / Always / Model Decision / Glob; `AGENTS.md` с v1.20.3 |
| 8 | **GitHub Copilot** | `.github/copilot-instructions.md` + `.github/instructions/*.instructions.md` | Markdown + YAML frontmatter (`applyTo`) | Не указан | Priority: Personal > Repo > Org; читает и `AGENTS.md` / `CLAUDE.md` / `GEMINI.md` в coding agent mode |
| 9 | **JetBrains Junie** | `.junie/AGENTS.md` (новое, 2026) или `AGENTS.md` root; legacy `.junie/guidelines.md` | Markdown | Не указан | Сам настраивает путь в Settings → Tools → Junie; legacy `.junie/guidelines/` как папка |
| 10 | **Warp Terminal AI** | `AGENTS.md` (caps!) + Warp Drive Rules | Markdown | «Lean» — prepended to prompt | Auto-link от `CLAUDE.md` / `.cursorrules` / `GEMINI.md` / `.windsurfrules` через `/init` |

**Portability шкала (у кого AGENTS.md работает «из коробки»):**
✅ Codex, OpenCode, Warp, Antigravity (v1.20.3+), Cursor, Copilot coding agent, Junie (через `.junie/AGENTS.md`)
⚠️ Claude Code (reads as fallback, нативный — `CLAUDE.md`; симлинк стандартная практика)
❌ Windsurf (есть свой `.windsurfrules`, AGENTS.md не читает нативно по состоянию на март 2026)

---

## 3. По IDE — детальные разделы

Ниже — по одной секции на каждую IDE. Структура одинаковая: **файл** → **формат** → **load order** → **scoping** → **что писать** → **что НЕ писать** → **лимиты** → **bootstrap** → **portability** → **real examples** → **gotchas**.

---

### 3.1. Claude Code (Anthropic)

**Файлы:**
- `CLAUDE.md` — project memory, в корне репо. Коммитится в git, шарится командой.
- `~/.claude/CLAUDE.md` — user memory, applies ко всем проектам. Твои личные preferences.
- `CLAUDE.local.md` — local memory, **в `.gitignore`**. Твои private overrides для этого проекта (API keys-заметки, эксперименты).
- `.claude/rules/*.md` — path-scoped rules. Рекурсивно дискавернется, можно подпапки.
- **Managed (Enterprise)** — системный уровень, разворачивается IT-службой, read-only.

**Формат:** обычный Markdown, опционально с `@import` директивами:
```markdown
@./docs/coding-standards.md
@~/shared-rules.md
@/absolute/path/file.md
```

**Load order (сверху = highest priority):**
1. Managed (Enterprise) — corporate policy
2. Local (`CLAUDE.local.md`) — твои private
3. Project (`./CLAUDE.md` или `./.claude/CLAUDE.md`) — командные
4. User (`~/.claude/CLAUDE.md`) — глобальные твои

**Важно:** более специфичное бьёт более общее. User `CLAUDE.md` говорит «2 пробела», project — «4 пробела» → побеждают 4. ([Developer Toolkit][4])

**Scoping / hierarchy:**
- Recurses **вверх** по дереву от cwd до (не включая) `/`
- Subdirectory `CLAUDE.md` подгружаются **on-demand** (когда Claude переходит в папку)
- Идеально для monorepo: корневой shared + per-package specific

**Что писать (секции, которые работают):**
```markdown
# Project Name

## Overview (WHY)
1 абзац — что за продукт, для кого, зачем.

## Tech Stack (WHAT)
- Languages + versions
- Framework, DB, build tool
- Package manager (pnpm/yarn/bazel)

## Architecture
- Monorepo layout
- Key entry points с путями: `src/cli/next-dev.ts → src/server/dev/next-dev-server.ts`
- Data flow

## Commands (HOW)
- Build / test / lint / typecheck
- Dev server launch

## Conventions
- Naming (camelCase/PascalCase/kebab-case, где что)
- Import style
- Error handling
- Logger usage

## Testing
- Framework, fixtures
- Tests MUST self-clean (как в Payload)

## What NOT to do
- Запрет default exports
- Не использовать floating-point без Decimal.js
- Не re-run test suite с разными grep — capture once, grep the log

## Skills / Subagents
@./.claude/skills/my-skill/SKILL.md
```

**Что НЕ писать:**
- **Пароли, API keys, tokens** — `CLAUDE.md` уходит в облако Anthropic при каждой сессии. Для секретов — `CLAUDE.local.md` (gitignored).
- README-копипаста — только уникальная, AI-не-инферимая информация.
- Устаревшие инструкции — каждое regress сигнал обновить.
- Длинные motivational абзацы, вежливости («пожалуйста, не мог бы ты») — принцип «контекст минимален, сигнал/шум максимален».

**Лимиты:**
- **Soft:** 150 строк (Anthropic official recommendation). HumanLayer держит <60 строк.
- **Hard:** контекст 200K у Sonnet / 1M beta — физически можно 50K строк, но deep context rot.
- `@import` — **5 hops максимум** (recursive).

**Dynamic vs static:**
- `CLAUDE.md` в cwd — **всегда** загружается в начало сессии.
- Subdirectory `CLAUDE.md` — **on-demand**, когда Claude входит в папку.
- `.claude/rules/*.md` — условно по path-pattern.
- Imports (`@`) — **at load time**, рекурсивно до 5 hops.

**Bootstrap:**
```
/init          # Claude генерит CLAUDE.md, анализируя файлы репо
# ...          # префикс # в чате — запомнить строку как memory
/memory        # показать, какие файлы сейчас в контексте
```
Практический принцип: «любая работа в новой папке с файлами — с `/init`.»

**Portability:**
- **Симлинк на AGENTS.md** — production pattern. Next.js, Payload так делают:
  ```bash
  ln -s AGENTS.md CLAUDE.md
  ```
  В `AGENTS.md` — `> Note: CLAUDE.md is a symlink to AGENTS.md.`
- Или короткий `CLAUDE.md` с `See @AGENTS.md` + Claude-specific додатки.

**Real-world examples:**
1. **Vercel Next.js `AGENTS.md`** ([github.com/vercel/next.js/blob/canary/AGENTS.md][5]) — 138K stars. Паттерны: tree-диаграмма packages/, точный маппинг entry points, Secrets and Env Safety секция, pointer на `.agents/skills/*`.
2. **Payload CMS `CLAUDE.md`** ([github.com/payloadcms/payload/blob/main/CLAUDE.md][6]) — 42K stars. «Tests MUST self-clean up» — эталон «что НЕ делать».
3. **LobeHub `CLAUDE.md`** ([github.com/lobehub/lobe-chat/blob/next/CLAUDE.md][7]) — 75K stars.

**Gotchas:**
- Claude **confidently hallucinates line numbers** в `file.ts:123`. Не проси его их генерить. Имена функций и файлов — OK.
- `CLAUDE.local.md` **не** покрывается `@import` из shared-файлов (по дизайну).
- Hooks (`.claude/settings.json` `hooks`) — это отдельный механизм, не CLAUDE.md. Для «каждый раз когда X» — hooks, не memory.
- После правки `CLAUDE.md` применить: **Ctrl+C Ctrl+C** (рестарт сессии) или `/clear`.

**Migration:**
- Из `.cursorrules` — просто переименовать и дополнить секциями. ([thepromptshelf][8])
- Из `AGENTS.md` — `ln -s AGENTS.md CLAUDE.md`.

---

### 3.2. Cursor

**Файлы:**
- `.cursor/rules/*.mdc` — **новый** формат (с февраля 2025 — default). YAML-frontmatter + markdown тело.
- `.cursorrules` — legacy, в root проекта. **Deprecated** с февраля 2025, но ещё работает. Новые проекты — только `.mdc`.
- User Rules — в настройках Cursor, глобальные для аккаунта.
- Team Rules — Enterprise, admin-managed.

**Формат `.mdc`:**
```mdc
---
description: "TypeScript conventions for this project"
globs: "**/*.ts,**/*.tsx"
alwaysApply: false
---

# TypeScript Rules

- Use `import type` for type-only imports
- Prefer `unknown` over `any`
- ...
```

**Важно:** это **не** чистый YAML. Не ставь кавычки на globs там, где агент их не ставит — иначе frontmatter парсится криво. ([forum.cursor.com][9])

**4 режима активации (определяются frontmatter):**

| Режим | `description` | `globs` | `alwaysApply` | Когда применяется |
|---|---|---|---|---|
| **Always** | empty | empty | `true` | Каждая сессия, автоматически |
| **Auto Attached** | empty | `"*.ts,*.tsx"` | `false` | Когда в chat упомянут/открыт файл по globs |
| **Agent Requested** | describable text | empty | `false` | Агент сам решает по description |
| **Manual** | empty | empty | `false` | Только при `@rule-name` в chat |

**Load order (priority top-down):**
1. **Team Rules** (Enterprise) — admin-managed
2. **Project Rules** (`.cursor/rules/`)
3. **User Rules** (settings)
4. **Legacy** `.cursorrules`
5. **`AGENTS.md`** (fallback, если ничего выше нет)

**Scoping:**
- Glob-scoped per rule файл
- Subdirectories: `.cursor/rules/frontend/`, `.cursor/rules/backend/` — рекурсивный discovery

**Что писать:**
- Разбивать на **модульные `.mdc` файлы** по темам: `typescript.mdc`, `react.mdc`, `security.mdc`, `testing.mdc`
- `alwaysApply: true` — только для CORE conventions (стек, общие правила)
- `globs:` — для per-language / per-framework rules
- `description:` + `alwaysApply: false` + `globs: empty` — для ситуативных («security-review for auth code»)

**Что НЕ писать:**
- Не клади всё в один `.mdc` с `alwaysApply: true` — теряешь всю экономию контекста
- Не описывай abstractly («Use Tailwind») — пиши конкретно («Use `cn()` from `@/lib/utils` for conditional classes»)
- Не пихай команды / setup — они живут в `README.md` (для людей) или `AGENTS.md` (для всех AI-tools)

**Лимиты:**
- Cursor контекст-бюджет жёсткий (у каждого tier свой). `alwaysApply: true` файлы суммарно съедают budget.
- Нет формального soft-limit, но community consensus: каждый `.mdc` ≤100 строк.

**Bootstrap:**
- В Cursor chat: `New Cursor Rule` → UI с выбором режима
- Или руками создать `.cursor/rules/my-rule.mdc`
- Auto-migration из `.cursorrules` — Cursor предложит

**Portability:**
- **`AGENTS.md`** Cursor читает как fallback (с 2026). Но без frontmatter features.
- Для команд с Cursor + Claude Code: `AGENTS.md` как канонический + `.cursor/rules/*.mdc` для Cursor-specific поведения.
- Symlink workaround: `ln -s ../../AGENTS.md .cursor/rules/rules.mdc` (aihackers.net)

**Real-world examples:**
1. **PatrickJS/awesome-cursorrules** ([github.com/PatrickJS/awesome-cursorrules][10]) — 39K stars, сотни rules для разных стеков. Бери как референс.
2. **Payload CMS templates** — с декабря 2025 ship с `.cursor/rules/` + `AGENTS.md`.

**Gotchas:**
- **`.cursorignore`** — отдельная концерна. Большие generated файлы засоряют embeddings → всегда исключать `dist/`, `build/`, `.next/`, `node_modules/`, `*.lock`.
- **Stale index:** активное редактирование → embeddings отстают. Для больших репо — 5-min sync, drift до 20%. (Morph)
- Frontmatter с `globs` в fancy-braces (`{js,ts}`) — **не работает** по состоянию на 2026. Пиши плоский список: `*.ts,*.tsx`.
- Индексы удаляются после 6 недель неактивности.

**Migration:**
- `.cursorrules` → `.cursor/rules/main.mdc` с `alwaysApply: true` — не потеряешь поведение.
- `CLAUDE.md` → можешь просто скопировать содержимое в `.cursor/rules/main.mdc`, `alwaysApply: true`.

---

### 3.3. OpenAI Codex CLI / Codex App

**Файл:** `AGENTS.md` — нативный. Codex — один из родоначальников формата (плюс Amp, Jules, Cursor, Factory).

**Опционально:** `AGENTS.override.md` — для **временных** отклонений. Пример: «на этой неделе форсим TypeScript strict, потом уберём».

**Формат:** обычный Markdown. Никакого frontmatter, никаких особых директив.

**Load order (Codex делает discovery-chain на старте run):**
1. **Global:** `~/.codex/AGENTS.override.md` → если нет, `~/.codex/AGENTS.md` (первый non-empty)
2. **Project scope:** от project root (git root) вниз до cwd. В каждой папке: `AGENTS.override.md` → `AGENTS.md` → `project_doc_fallback_filenames`. **Максимум один файл на папку.**
3. **Merge:** concatenate от root вниз. Файлы ближе к cwd **перезаписывают** (позднее в итоговом prompt = выше приоритет).

**Scoping:**
- Hierarchical по директориям (like OpenAI main repo — 88 AGENTS.md файлов)
- Монорепо-friendly: корень → packages/X → src/feature

**Что писать:**
Тот же универсальный шаблон: Overview / Stack / Architecture / Commands / Conventions / What NOT.

Для монорепо:
- Корневой `AGENTS.md` — shared (стек целиком, топ-команды)
- Per-package `AGENTS.md` — package-specific (локальные commands, deviations)

**Что НЕ писать:**
- Секреты (они идут в OpenAI cloud).
- Огромные copy-paste из README — если README нужен, pointer «See README.md for X».

**Лимиты:**
- **`project_doc_max_bytes`** — **32 KiB** по умолчанию (hard). Можно поднять в конфиге.
- Codex **пропускает пустые файлы** и **останавливается** когда combined size достигает лимита.

**Dynamic vs static:**
- Load chain строится **один раз на run** (в TUI — один раз на сессию).
- Изменил `AGENTS.md` в середине сессии — нужен новый run.

**Bootstrap:**
- Codex сам может сгенерить: «Create an AGENTS.md for this repo». См. [agents.md][1] FAQ.
- Или руками.

**Portability:**
- **Канонический стандарт.** Если пишешь `AGENTS.md` — он нативно работает в Codex, OpenCode, Warp, Cursor (fallback), Copilot coding agent, Antigravity v1.20.3+, Junie (через `.junie/AGENTS.md`).
- Claude Code — reads as fallback + симлинк пацнерн.

**Real-world examples:**
1. **Vercel Workflow SDK AGENTS.md** ([github.com/vercel/workflow/blob/main/AGENTS.md][11]) — production SDK. Packages table + пример env vars для CI.
2. **Vercel Next.js AGENTS.md** (см. 3.1) — эталон монорепо-уровня.
3. **OpenAI main repo** — 88 AGENTS.md файлов (каждый пакет).

**Gotchas:**
- **Нет `@include`** (пока — есть issue [#17401][12], community requests). Если нужен modular — раскидывай по папкам через hierarchical discovery.
- Максимум **один файл на папку** — если и `AGENTS.md`, и `TEAM_GUIDE.md`, Codex возьмёт только первый по priority.
- `AGENTS.override.md` **не коммитить** в git — это для локальных temporary tweaks.

**Migration:**
- Из `CLAUDE.md` → `mv CLAUDE.md AGENTS.md && ln -s AGENTS.md CLAUDE.md` (symlink для взаимной совместимости).

---

### 3.4. OpenCode

> Примечание: Дмитрий Добротворский активно использует OpenCode через BYOK (bring-your-own-keys) с Z.AI GLM и OpenRouter.

**Файл:** `AGENTS.md` — нативный.

**Локации:**
- Project: `./AGENTS.md` (коммитится)
- Global: `~/.config/opencode/AGENTS.md`
- Fallback: OpenCode **читает `CLAUDE.md`** — из cwd и из `~/.claude/CLAUDE.md`, если `OPENCODE_DISABLE_CLAUDE_CODE_PROMPT` не установлен.

**Формат:** обычный Markdown.

**Load order (first-match-wins в каждой категории):**
1. Local traversal из cwd вверх: `AGENTS.md` → `CLAUDE.md` (первый найденный)
2. Global: `~/.config/opencode/AGENTS.md` → `~/.claude/CLAUDE.md`

Т.е. если в репо лежат и `AGENTS.md`, и `CLAUDE.md` — OpenCode возьмёт **только `AGENTS.md`** (first match).

**Scoping:**
- Traverse вверх от cwd
- Плюс `opencode.json` с `instructions[]` — можно указать дополнительные файлы с glob-паттернами:
  ```json
  {
    "instructions": ["packages/*/AGENTS.md", "docs/rules/*.md"]
  }
  ```

**Что писать:** тот же универсальный шаблон + про то, какие tools / skills агенту разрешено использовать (OpenCode имеет свою permission-систему в `opencode.json`).

**Что НЕ писать:** то же самое — секреты нет, копипасты из README нет.

**Лимиты:** в документации не указан hard limit. Community — keep lean, как везде.

**Bootstrap:**
```
/init          # в TUI — OpenCode сгенерит AGENTS.md
```
Потом закоммитить в git.

**Portability:**
- `AGENTS.md` + fallback на `CLAUDE.md` = OpenCode «из коробки» работает в любой экосистеме.
- Через `opencode.json` `instructions` — можно reuse любые rules-файлы команды без дублирования.

**Real-world examples:**
1. **MichaelFisher1997/opencode-config** ([github.com/MichaelFisher1997/opencode-config][13]) — personal config с multi-provider, MCP, AGENTS.md + skills.

**Gotchas:**
- **Bug Jan 2026** ([issue #11534][14]): `OPENCODE_CONFIG_DIR` с локальным `AGENTS.md` игнорировался, если глобальный есть. Проверь, что у тебя свежая версия (fixed весной 2026).
- OpenCode — open-source community продукт, API меняется быстрее чем Codex / Claude Code. Фиксируй версию, с которой работаешь.
- Custom agents имеют **свой YAML frontmatter** (`permission:`, `tools:`) — это про конфиг кастом-агентов, не про AGENTS.md.

---

### 3.5. Windsurf (Codeium Cascade)

**Файлы:**
- `.windsurfrules` — project-level, в корне. Коммитится в git.
- Global Windsurf rules — в настройках IDE (привязаны к аккаунту).
- **Memories** — auto-generated, сохраняются между сессиями (уникальная фича Windsurf).

**Формат:** обычный Markdown. Без frontmatter.

**Load order:**
1. Global rules (user account)
2. Project `.windsurfrules`
3. Memories подкачиваются по relevance (ИИ сам решает)

**Scoping:** только root-level, иерархии нет. Для monorepo — один файл, разные секции.

**Что писать:**
- Coding conventions
- Anti-patterns
- Framework preferences
- Testing/lint commands

Опирайся на официальный Windsurf getting-started template.

**Что НЕ писать:**
- Секреты.
- Команды для ad-hoc задач — Windsurf Memories подхватит после первого раза.

**Лимиты:** hard-лимит не опубликован. Community — ≤150 строк.

**Dynamic vs static:**
- `.windsurfrules` — static, подгружается при открытии workspace.
- Memories — dynamic, агент запоминает по ходу и использует в будущих сессиях.

**Bootstrap:**
- В IDE: Settings → AI → Workflow → Custom rules
- Или руками `.windsurfrules` в root

**Portability:**
- **Windsurf НЕ читает `AGENTS.md` нативно** (по состоянию на март 2026). Если хочешь cross-IDE rules — копируй содержимое `AGENTS.md` в `.windsurfrules` руками. Single source of truth — невозможно.

**Real-world examples:**
- Большинство открытых `.windsurfrules` в GitHub поиске (`filename:.windsurfrules`) — небольшие файлы с framework-specific правилами.

**Gotchas:**
- **Cascade понимает static structure, не runtime/intent** — дыра, куда прячутся баги. Если нужна runtime-логика — объясняй её явно в rules.
- Memories auto-save — проверяй, что туда попало: иногда запоминает temporary hacks как permanent rules.
- Fast Context subagent (SWE-grep / SWE-grep-mini) — отдельный механизм, работает поверх rules.

---

### 3.6. Gemini CLI

**Файлы:**
- `GEMINI.md` — project-level, в корне
- `~/.gemini/GEMINI.md` — global, user-level
- Субдиректорные `GEMINI.md` — per-folder context

**Формат:** обычный Markdown.

**Load order (hierarchical, concatenated, sent to model each prompt):**
1. **Global:** `~/.gemini/GEMINI.md`
2. **Project root + ancestors:** from cwd up to project root (detected by `.git`)
3. **Subdirectories:** scans below cwd; respects `.gitignore` and `.geminiignore`

Все найденные файлы конкатенируются и отправляются. С февраля 2026 ([issue #16999][15]) — есть PR для структурированного hierarchy с XML-тегами для explicit conflict resolution.

**Scoping:** monorepo-friendly — корневой + per-folder.

**Что писать:**
- Persona: «You are a senior [language] engineer specialized in [domain]»
- Coding style
- Testing framework
- Tech stack constraints
- Domain knowledge

**Что НЕ писать:** секреты.

**Лимиты:**
- `context.discoveryMaxDirs` — **200 директорий** (default) для scan
- 1M токенов контекста Gemini 3 Pro — физически много, но context rot всё равно.

**Dynamic vs static:**
- GEMINI.md файлы — static, подгружаются при старте
- `/memory show` — увидеть собранный концатенированный контекст
- `/memory refresh` — force reload после правки
- `/memory add <text>` — append в global `~/.gemini/GEMINI.md` на лету

**Bootstrap:**
- Нет `/init`, который бы генерил GEMINI.md автоматически — пишешь руками или через промпт «Create GEMINI.md for this repo».

**Portability:**
- **Через `settings.json`** расширяемо до любых имён:
  ```json
  {
    "context": {
      "fileName": ["AGENTS.md", "CONTEXT.md", "GEMINI.md"]
    }
  }
  ```
  Это прямой способ заставить Gemini CLI читать `AGENTS.md`.

**Real-world examples:**
- Github search `filename:GEMINI.md` — сотни репо с примерами (в основном Python/Go проекты).

**Gotchas:**
- Concatenation **без приоритетов** (пока PR #18350 не в main) — ambiguity при конфликтах.
- `.geminiignore` — отдельный файл для исключения из scan (аналог `.cursorignore`).
- Если правишь в работающей сессии — сессия не видит, `/memory refresh` обязателен.

---

### 3.7. Google Antigravity

**Файлы (v1.20.3+, март 2026):**
- **Global rules:** `~/.gemini/GEMINI.md` (Antigravity-only) или `~/.gemini/AGENTS.md` (cross-tool)
- **Workspace rules:** `GEMINI.md` или `AGENTS.md` в корне + `.agents/rules/*.md` (note: раньше было `.agent/rules/`, с v1.20.3 default `.agents/rules/` с backward compat).
- **Nested:** `.agents/rules/**/*.md` + per-subfolder `AGENTS.md`

**Формат:** Markdown. Для rules в `.agents/rules/` — есть activation mode, настраивается в IDE UI (не через frontmatter!).

**4 режима активации (как Cursor):**

| Mode | Trigger |
|---|---|
| **Manual** | @ mention в agent input |
| **Always On** | Каждая сессия |
| **Model Decision** | Модель решает по description |
| **Glob** | По glob pattern (`src/**/*.ts`) |

**Load order:**
1. Global: `~/.gemini/` — applies ко всем workspaces
2. Workspace: `GEMINI.md` / `AGENTS.md` в workspace root
3. `.agents/rules/*.md`
4. Nested `AGENTS.md` (если включено в Settings → Agent → Load nested AGENTS.md files)

`GEMINI.md` берет приоритет над `AGENTS.md` при конфликте.

**Scoping:** nested AGENTS.md, path-scoped rules.

**Что писать:** универсальный шаблон + стек-специфичные rules в `.agents/rules/` с glob-активацией.

**Что НЕ писать:** секреты.

**Лимиты:** не опубликован. Community — lean.

**Bootstrap:**
- Customizations panel → + Global / + Workspace → Rule name → содержимое.
- Или руками.

**Portability:**
- С v1.20.3 — `AGENTS.md` нативный ✅
- Shared rules → `AGENTS.md`, Antigravity-specific → `GEMINI.md`

**Real-world examples:**
- Antigravity Codes community ([antigravity.codes/blog/user-rules][16]) — блог с template и примерами.

**Gotchas:**
- **Две папки путают:** `.agent/rules/` (legacy) vs `.agents/rules/` (v1.20.3+ default). Обе работают, но пиши в новую.
- Workflows (`.agents/workflows/`) — **отдельная концерна**: reusable sequences команд, не rules.
- Skills (`.agents/skills/`) — тоже отдельно, progressive disclosure как в Claude Code.

---

### 3.8. GitHub Copilot (VS Code / JetBrains / Visual Studio)

**Файлы:**
- `.github/copilot-instructions.md` — **repository-wide**, main file.
- `.github/instructions/*.instructions.md` — **path-scoped**, с YAML frontmatter.
- Personal instructions — user-level, в Copilot settings.
- Organization instructions — enterprise, admin-managed.

**Формат `.instructions.md`:**
```markdown
---
applyTo: "**/*.tsx,**/*.jsx"
excludeAgent: "code-review"
---

# React Component Rules
- Use functional components with hooks
- ...
```

**Frontmatter fields:**
- `applyTo` — glob pattern (обязательно, если файл в `.github/instructions/`)
- `excludeAgent` — `"code-review"` or `"coding-agent"` — исключить из контекста определённого агента

**Load order (priority top-down):**
1. **Personal** (user-level settings)
2. **Repository** (`.github/copilot-instructions.md` + `.github/instructions/*.instructions.md`)
3. **Organization** (admin-managed)

Copilot Cloud Agent **также читает:**
- `**/AGENTS.md`
- `/CLAUDE.md`
- `/GEMINI.md`

Т.е. Copilot coding agent нативно поддерживает все три популярных формата.

**Scoping:**
- Repository-wide → `.github/copilot-instructions.md`
- Path-scoped через `applyTo` glob

**Что писать:**
- **`copilot-instructions.md`:** overview + build/test commands + coding standards
- **`.github/instructions/playwright-tests.instructions.md`:** Playwright-specific rules

Официальный example от GitHub docs:
```markdown
# Project: my-awesome-app

We're building a [description].

## Commands
- `npm run build`
- `npm test`
- `npm run lint`

## Conventions
- ESLint config in `.eslintrc.json`
- Use React functional components only
```

**Что НЕ писать:**
- Conflicting instructions между personal / repo / org — если боишься проблем, временно отключи repo instructions в Settings.

**Лимиты:** не опубликован. Custom instructions включаются для всех типов Copilot (Chat, code review, coding agent).

**Dynamic vs static:**
- `.github/copilot-instructions.md` — всегда в контексте
- `.instructions.md` с `applyTo` — только когда работаешь с matching files

**Bootstrap:**
- Руками. Нет команды.
- Шаблоны — в GitHub docs + GitHub blog changelog от 2025-07.

**Portability:**
- Copilot cloud agent с 2025-07 читает `AGENTS.md` / `CLAUDE.md` / `GEMINI.md` — т.е. если уже есть любой из этих файлов, Copilot его подхватит.
- Для **inline chat** (VS Code / JetBrains plugins) — всё ещё нужен `.github/copilot-instructions.md`.

**Real-world examples:**
- GitHub Enterprise Cloud docs → раздел Best Practices — show canonical `copilot-instructions.md`.
- Большинство active open-source Next.js / TypeScript проектов имеют этот файл в `.github/`.

**Gotchas:**
- **Директория `.github/instructions/` vs `.github/prompts/`** — разные вещи! Instructions = правила для агента, prompts = reusable prompt templates.
- **`excludeAgent`** — полезно, чтобы code-review бот не получал instructions для coding-agent (и наоборот).
- Repo instructions можно **отключить** в Repo Settings → Copilot — если agent ведёт себя странно.

**Migration:**
- Из `AGENTS.md` → `cp AGENTS.md .github/copilot-instructions.md` или просто оставить оба (Copilot coding agent читает оба).

---

### 3.9. JetBrains AI Assistant / Junie

**Разные продукты:**
- **AI Assistant** — classic autocomplete + chat, встроен в IDE. Project context — через built-in settings, **нет** стандартного rules-файла.
- **Junie** — agentic mode (с 2025), это тот, что использует rules.

**Файлы (Junie, updated 2026):**
- **Preferred:** `.junie/AGENTS.md` — **стандартная локация** в корне проекта.
- **Fallback:** `AGENTS.md` в корне проекта.
- **Legacy (deprecated):** `.junie/guidelines.md`, `.junie/guidelines/*.md` (папка — конкатенация всех `.md`).
- **Custom path:** Settings → Tools → Junie → Project Settings → указать любой путь.

**Формат:** стандартный AGENTS.md Markdown.

**Load order Junie:**
1. Custom path из Settings (если указан) — **всегда побеждает**
2. `.junie/AGENTS.md`
3. `AGENTS.md` в project root
4. Legacy `.junie/guidelines.md`
5. Legacy `.junie/guidelines/` directory (concatenates)

**Scoping:** для monorepo — Junie **рекомендует** явно указывать `Guidelines path` в Settings (не надеяться на default).

**Что писать:**
- Spec-driven артефакты: Junie promotes подход `docs/requirements.md` + `docs/plan.md` + `docs/tasks.md` + `.junie/guidelines.md` (linking them).
- Tech stack, testing framework, build commands.
- Guidance for running tests, executing scripts.
- Pointers to existing guidelines: «See `docs/testing-guide.md`».

**Что НЕ писать:** секреты, дубли README.

**Лимиты:** не опубликован.

**Bootstrap:**
- **Автоматически через Junie:** промпт «Analyze the project structure and tech stack, and create a `.junie/guidelines.md` file...». Junie сам пройдёт по файлам и создаст.
- Или руками.

**Portability:**
- `AGENTS.md` в root — Junie нативно его берёт ✅
- `.junie/AGENTS.md` — более специфично, приоритет выше.

**Real-world examples:**
- **JetBrains/junie-guidelines** ([github.com/JetBrains/junie-guidelines][17]) — 406 stars, официальный catalog. Java / Spring Boot / Django / Gin / Nuxt шаблоны.

**Gotchas:**
- AI Assistant ≠ Junie. Если используешь **только** AI Assistant — rules-файлов нет, project context настраивается в Settings > Tools > AI Assistant (prompt hints).
- **`.junie/guidelines/` как папка** (legacy) — Junie конкатенирует все `.md` внутри. Если миграционишь — сначала слить в один `.junie/AGENTS.md`.
- **Ignore-файл:** `.aiignore` в корне — исключает файлы/папки из видимости Junie (аналог `.cursorignore` / `.geminiignore`).
- Junie CLI (headless mode) и GitHub Action — те же правила, та же иерархия.

**Migration:**
- Из `.junie/guidelines.md` → `.junie/AGENTS.md` (буквально `mv`).
- Из `CLAUDE.md` → `mv CLAUDE.md .junie/AGENTS.md` + symlink.

---

### 3.10. Warp Terminal AI

**Файлы:**
- **`AGENTS.md`** (caps!) в репо — **default since 2026**. Backward compat: `WARP.md` ещё работает.
- **Global Rules** в Warp Drive → Personal → Rules (не файл, а entity в UI).
- Можно **линковать внешние rules-файлы** (через `/init`): `CLAUDE.md`, `.cursorrules`, `AGENT.md`, `GEMINI.md`, `.clinerules`, `.windsurfrules`, `.github/copilot-instructions.md` — все эти форматы Warp распознаёт и может использовать.

**Формат:** обычный Markdown.

**Load order:**
1. Global Rules (Warp Drive, user-level)
2. Project Root `AGENTS.md` / `WARP.md`
3. Current directory `AGENTS.md` (если отличается от root)
4. Best-effort: subdirectory `AGENTS.md` (когда редактируешь файлы там)

**Scoping:**
- Root + current directory — автоматически
- Other subdirectories — best-effort (когда touching files там)
- Для monorepo: разные `AGENTS.md` в `ui/`, `api/`, и т.д.

**Что писать:**
- Environment preferences (package manager, Python env, build tool) — Warp особенно это ценит:
  ```
  Rule: Environment Preferences
  - Always use pnpm for Node.js projects
  - Default to miniconda for Python environments
  - Use the Tauri CLI when building desktop apps
  ```
- Coding standards
- Branching / PR guidelines

**Что НЕ писать:**
- Секреты. Warp может запускать команды на твоей машине — всё, что в rules, попадает в контекст сессии.
- Огромные тексты — «everything in warp.md is prepended to your prompt. A longer file consumes more tokens and can increase compute cost. Keep only what truly matters.» ([Warp docs][18])

**Лимиты:** «keep lean» — нет явного hard-лимита, но каждая строка = токен в каждом prompt.

**Dynamic vs static:**
- `AGENTS.md` — static, prepended to every agent prompt
- Warp Drive Notebooks / Prompts / Workflows — dynamic, referenced agent'ом по требованию

**Bootstrap:**
```
/init                    # сгенерит AGENTS.md, проиндексирует codebase
/open-project-rules      # откроет AGENTS.md в боковом редакторе
```

Или из UI: Settings > AI > Knowledge > Manage Rules.

**Portability:**
- `AGENTS.md` — nativа для Warp ✅
- **Linking mode:** `/init` может слинковать существующий `CLAUDE.md` / `.cursorrules` / `GEMINI.md` / `.windsurfrules` — Warp его распознает и будет использовать напрямую, без дублирования.

**Real-world examples:**
- Warp docs собственные templates ([docs.warp.dev/guides/configuration][18]).
- Любой OSS-репо с `WARP.md` или `AGENTS.md` в GitHub поиске.

**Gotchas:**
- **Filename case-sensitive:** `AGENTS.md` (caps), не `agents.md`. Warp docs особо это подчёркивают.
- **Warp Drive** — shared config layer для команд. Если команда шарит Drive, Rules оттуда применяются ко всем её членам.
- Warp — **терминал**, не IDE. Агенты работают с shell + text editor panel. Rules важны для shell commands («какой package manager»), не для UI-конвенций.
- Codebase indexing Warp делает локально, ничего не уходит на сервера — но AGENTS.md при каждом prompt **отправляется** в LLM-провайдер (OpenAI / Anthropic / Google).

**Migration:**
- `WARP.md` → `AGENTS.md` — можно переименовать без миграции.
- Внешний формат → `/init` с linking (Warp не копирует, использует напрямую).

---

## 4. Cross-IDE portability — как писать один rules-файл для всех

**Большая идея:** `AGENTS.md` — single source of truth. Остальные rules-файлы либо симлинки, либо короткие тул-специфичные ноты.

### 4.1. Паттерн «Next.js / Payload» — симлинк

```bash
# В корне репо
echo "# Agents guide" > AGENTS.md
ln -s AGENTS.md CLAUDE.md
```

В `AGENTS.md` первой строкой:
```markdown
> **Note:** CLAUDE.md is a symlink to AGENTS.md. They are the same file.
```

Работает нативно для: Codex, OpenCode, Warp, Antigravity, Cursor (fallback), Copilot coding agent, Junie (если назвать `.junie/AGENTS.md` или положить в root).

### 4.2. Паттерн «Claude Code + Cursor»

```bash
# AGENTS.md = канонический shared
# CLAUDE.md — symlink
# .cursor/rules/main.mdc — symlink (Cursor ignores frontmatter if absent; works as plain Cursor rule)

ln -s AGENTS.md CLAUDE.md
mkdir -p .cursor/rules
ln -s ../../AGENTS.md .cursor/rules/main.mdc
```

Cursor-specific фичи (per-language, agent-requested) — отдельными файлами `.cursor/rules/typescript.mdc`, `.cursor/rules/security.mdc`.

### 4.3. Паттерн «Windsurf — отдельно»

Windsurf не читает `AGENTS.md`. Но `.windsurfrules` можно сгенерить скриптом из `AGENTS.md` при каждом merge в main:
```bash
cp AGENTS.md .windsurfrules
# или в Makefile:
# build-rules: .windsurfrules
# .windsurfrules: AGENTS.md
#   cp $< $@
```

### 4.4. Паттерн «Monorepo»

```
project-root/
  AGENTS.md                    # shared (stack, top-level architecture)
  CLAUDE.md -> AGENTS.md
  packages/
    api/
      AGENTS.md                # api-specific (DB, routing, tests)
    ui/
      AGENTS.md                # ui-specific (components, styling)
    shared/
      AGENTS.md                # shared utilities rules
```

Claude Code автоматом подтянет иерархически (вверх по cwd). Codex — тоже. Cursor — через subdirectory rules.

### 4.5. Антипаттерны, которые не надо делать

❌ **Copypaste везде** — через месяц файлы drift, разные правила в разных местах.
❌ **Один огромный AGENTS.md на 500 строк** — context rot, Claude начинает ignore parts.
❌ **Rules с секретами** — они улетают в облако при каждой сессии.
❌ **Instructions в READMEs для людей** — агент обычно их не приоритизирует. Дублируй в AGENTS.md.

---

## 5. Главные принципы написания CLAUDE.md / AGENTS.md — distilled

### 5.1. Правила написания

1. **Не больше 150 строк на файл** (Anthropic-рекомендация). Больше → AI начинает игнорировать части из-за context rot.

2. **Минимальный контекст + максимальный signal/noise.** Убери всё, что не решает конкретную задачу.

3. **Наибольшая ценность контекста — ближе к его началу.** Самое важное — в первых ~30 строк файла.

4. **Писать rules-файлы на английском.** Русский — третий язык в LLM-данных, английский доминирует (~70%+). Измеримый прирост качества на английском. Рабочий подход: инструкции на английском, прямые цитаты и контакты — на русском.

5. **Без вежливостей в промптах.** «Пожалуйста, не мог бы ты…», «извини за беспокойство» — пропускай. Это контекстный шум.

### 5.2. Что КЛАСТЬ

- **Профиль / персона** — если проект делается от конкретной роли (маркетолог X, junior dev в команде Y, support engineer). Выходы персонализируются, а не шаблонные.
- **Конкретика, не абстракции.** «Использовать библиотеку Prisma» лучше, чем «использовать ORM». «Генерация через NanoBanana» лучше «AI-генератор».
- **Pointers на скиллы/правила** вместо копипасты: `@./.claude/skills/weekly-report/SKILL.md` — загрузится on-demand.
- **Минимум contextual-friction:** основные факты о проекте и себе — чтобы не писать в 50-й раз.

### 5.3. Что НЕ класть

- **Пароли, логины, ключи, токены, доступы.** CLAUDE.md уходит в облако при каждой сессии. Для секретов — `CLAUDE.local.md` (gitignored по дефолту).
- **Раздутый content** — ухудшает качество за счёт context rot.
- **Слишком глубокая иерархия файлов правил** — легко заблудиться и искать концы. 2-3 уровня максимум.

### 5.4. Workflow

- **`/init` — с любой работы в новой папке.** Claude сам сгенерит стартовый CLAUDE.md, проанализировав репо.
- **`/init` при крупных изменениях в коде,** не каждую сессию. Claude сам следит за актуальностью CLAUDE.md.
- **Перезапуск для применения CLAUDE.md:** `Ctrl+C` дважды (рестарт сессии) или `/clear`.
- **`#` префикс в чате** — добавляет строку в memory на лету:
  ```
  # Всегда говори со мной на русском
  ```
- **`/compact` с фокусом:** `/compact важная_информация` — при компрессии контекста Claude сохранит эту область с приоритетом.
- **Тест CLAUDE.md через канонический промпт:** подготовь один типовой промпт для своего проекта. Запусти без CLAUDE.md → шаблонное клише. Запусти с CLAUDE.md → персонализированный ответ. Разница показывает, работает ли файл.

### 5.5. Skills — связаны с rules-файлами

- **Description во frontmatter скила — главный триггер активации.** В описании — глаголы конкретных действий. На основании description AI решает, запускать скилл или нет.
- **Progressive disclosure:** при scanе загружается только frontmatter (~10% размера скила), полное тело — по требованию. В CLAUDE.md достаточно pointer, не копипаста скила.
- **Не запускать скачанные скилы без аудита** — скилл = исполняемый код + промпт, читай перед подключением.

---

## 6. Templates — копи-пейст стартеры по IDE

Все шаблоны ≤100 строк, на английском (для лучшего качества LLM), production-quality. Замени `[placeholder]` на твои значения.

### 6.1. Claude Code `CLAUDE.md`

```markdown
# [Project Name]

A [one-sentence description] for [target users].

## Stack
- Language: TypeScript 5.4 (strict mode)
- Framework: Next.js 15 (App Router)
- DB: Postgres 16 via Drizzle ORM
- Package manager: pnpm 9

## Architecture
- `src/app/` — Next.js App Router pages
- `src/lib/` — pure utility functions (no DB, no fetch)
- `src/db/` — Drizzle schema + queries
- `src/components/` — React components (functional only)

Key entry points:
- `src/app/api/[...path]/route.ts` — catch-all API handler
- `src/middleware.ts` — auth + i18n

## Commands
- `pnpm dev` — dev server
- `pnpm test` — Vitest
- `pnpm lint` — ESLint + prettier check
- `pnpm db:migrate` — Drizzle migration

## Conventions
- Components: functional + hooks. No class components.
- Imports: `import type` for type-only; alias `@/` for `src/`
- Errors: throw typed errors from `src/errors/`; catch in route handlers
- Naming: camelCase for variables, PascalCase for components, kebab-case for files

## Testing
- Vitest for units, Playwright for e2e
- Tests MUST self-clean up (reset DB fixtures after each)
- Never re-run test suite with different grep filters; capture once, grep the log

## Secrets & Env Safety
- Never print secret values (even truncated) in code or logs
- Stop and ask if a task seems to require new env vars
- Secrets live in `.env.local` (gitignored)

## What NOT to do
- No default exports (breaks refactoring)
- No floating-point math without Decimal.js
- No `any` — use `unknown` + narrow

## Skills
@./.claude/skills/code-review/SKILL.md
@./.claude/skills/db-migration/SKILL.md
```

### 6.2. Cursor `.cursor/rules/main.mdc`

```mdc
---
description: "Core project conventions — always applied"
alwaysApply: true
---

# [Project Name] — Core Rules

TypeScript strict, Next.js 15 App Router, pnpm. See `AGENTS.md` for full context.

## Conventions (apply everywhere)
- Functional components only
- Named exports (no default exports)
- `import type` for type-only imports
- Alias `@/` for `src/`

## When editing code
- Always run `pnpm lint` mentally before suggesting
- Prefer small, focused functions over large ones
- Never use `any` — use `unknown` + narrow

## When editing tests
- Vitest for units, Playwright for e2e
- Tests MUST self-clean up

## What NOT to do
- No `console.log` in committed code
- No floating-point math without Decimal.js
- No silent catch blocks
```

Дополни по necessity:
- `.cursor/rules/typescript-strict.mdc` — `globs: "**/*.ts,**/*.tsx"`, `alwaysApply: false`
- `.cursor/rules/security-review.mdc` — `description: "Security review checklist for auth code"`, `alwaysApply: false`

### 6.3. Codex CLI `AGENTS.md` (canonical, portable)

```markdown
# [Project Name]

> Note: CLAUDE.md is a symlink to this file. Same content, all agents.

[One-paragraph project description.]

## Tech Stack
- TypeScript 5.4 (strict)
- Next.js 15 App Router
- Postgres 16 + Drizzle ORM
- pnpm 9

## Project Structure
- `src/app/` — Next.js routes
- `src/lib/` — pure utils
- `src/db/` — DB schema + queries
- `src/components/` — React (functional)

## Commands
```bash
pnpm dev
pnpm test
pnpm lint
pnpm db:migrate
```

## Coding Conventions
- Functional components + hooks; no classes
- Named exports only
- `import type` for type-only
- camelCase (vars) / PascalCase (components) / kebab-case (files)
- Throw typed errors from `src/errors/`; catch in route handlers

## Testing
- Vitest (units) + Playwright (e2e)
- Tests MUST self-clean
- Never re-run suite with different greps — capture once, grep the log

## Secrets & Env
- Never print secret values
- Stop and ask if the task needs new env vars
- `.env.local` is gitignored

## What NOT to do
- No default exports
- No floating-point without Decimal.js
- No `any`

## CI reproduction
- `IS_WEBPACK_TEST=1` reproduces webpack-lane failures
- Do NOT set `NEXT_SKIP_ISOLATE=1` when running full suite
```

### 6.4. OpenCode `AGENTS.md`

Идентичен 6.3 — OpenCode читает стандартный AGENTS.md. Опционально в `opencode.json`:
```json
{
  "instructions": ["AGENTS.md", "docs/rules/*.md"]
}
```

### 6.5. Windsurf `.windsurfrules`

```markdown
# [Project Name] — Windsurf Rules

TypeScript strict + Next.js 15 + pnpm.

## Conventions
- Functional components + hooks only
- Named exports
- `import type` for types
- Alias `@/` → `src/`
- camelCase / PascalCase / kebab-case

## Testing
- Vitest (units) + Playwright (e2e)
- Tests self-clean

## Framework preferences
- Tailwind via `cn()` from `@/lib/utils`
- Drizzle ORM for all DB access
- Zod for runtime validation

## Anti-patterns (reject these)
- Default exports
- Floating-point math without Decimal.js
- `any` type
- Silent `catch { }` blocks

## Commands
- `pnpm dev` — start
- `pnpm test` — run tests
- `pnpm lint` — ESLint + prettier
```

### 6.6. Gemini CLI `GEMINI.md`

```markdown
# [Project Name] Context

You are a senior TypeScript engineer specializing in Next.js App Router and Postgres.

## Stack
- TypeScript 5.4 strict, Next.js 15 (App Router), Postgres 16 + Drizzle, pnpm 9

## Project layout
- `src/app/` routes, `src/lib/` utils, `src/db/` schema, `src/components/` React

## Conventions
- Functional React + hooks only; no classes
- Named exports; no default exports
- `import type` for type-only imports
- camelCase / PascalCase / kebab-case
- Errors: throw typed from `src/errors/`

## Testing
- Vitest + Playwright
- Tests MUST self-clean

## Commands
- `pnpm dev` / `pnpm test` / `pnpm lint` / `pnpm db:migrate`

## What NOT to do
- No default exports
- No floating-point math without Decimal.js
- No `any`
- Never print secret values

## Workflow preferences
- When I ask for a refactor, propose a plan first; wait for approval before editing
- When I paste an error, trace it 2-3 files deep before suggesting fixes
```

Bonus — в `settings.json`:
```json
{
  "context": {
    "fileName": ["AGENTS.md", "GEMINI.md"]
  }
}
```

### 6.7. Antigravity `AGENTS.md` (workspace) + `.agents/rules/code-style.md`

**AGENTS.md** — используй 6.3 канонический.

**`.agents/rules/code-style.md`** (activation: Glob, pattern `**/*.ts,**/*.tsx`):
```markdown
# TypeScript / React Style

- Functional components only
- Named exports
- `import type` for type-only imports
- Prefer `unknown` + narrow over `any`
- Tailwind via `cn()` from `@/lib/utils`
- Props typed as `interface ComponentNameProps`
```

### 6.8. GitHub Copilot `.github/copilot-instructions.md`

```markdown
# Project: [Project Name]

[One-paragraph description.]

## Tech Stack
- TypeScript 5.4 strict, Next.js 15 App Router, Postgres 16 + Drizzle ORM, pnpm 9

## Commands
- `pnpm dev` — start dev server
- `pnpm test` — Vitest
- `pnpm lint` — ESLint + prettier

## Conventions
- Functional React components + hooks only
- Named exports (no default exports)
- `import type` for type-only imports
- camelCase / PascalCase / kebab-case
- Errors: throw typed from `src/errors/`

## What NOT to do
- No default exports
- No floating-point math without Decimal.js
- No `any`
```

Плюс `.github/instructions/playwright-tests.instructions.md`:
```markdown
---
applyTo: "**/*.spec.ts,**/e2e/**"
---

# Playwright Tests
- Use `test.describe` blocks to group
- Tests MUST self-clean (reset DB fixtures after)
- Use `data-testid` attributes, not CSS selectors
- Run via `pnpm test:e2e` (sets `PLAYWRIGHT_BASE_URL`)
```

### 6.9. JetBrains Junie `.junie/AGENTS.md`

Используй 6.3. Junie его нативно прочитает.

Дополнительно (spec-driven workflow, как рекомендует JetBrains blog):
- `docs/requirements.md` — high-level спек
- `docs/plan.md` — implementation план
- `docs/tasks.md` — `[ ]` / `[x]` checklist
- Правило в `.junie/AGENTS.md`: «Mark tasks as `[x]` when completed. Link every new task to a requirement and plan item.»

### 6.10. Warp Terminal `AGENTS.md`

```markdown
# [Project Name]

TypeScript + Next.js + Postgres project.

## Environment Preferences
- Always use pnpm for Node.js (never npm/yarn)
- Python: use miniconda environments
- Build desktop apps with Tauri CLI

## Commands
- `pnpm dev` / `pnpm test` / `pnpm lint` / `pnpm db:migrate`

## Coding Conventions
- TypeScript strict; no `any`
- Functional React + hooks only
- Named exports
- `import type` for types

## Shell / Git guidelines
- Branch names: `feat/...`, `fix/...`, `chore/...`
- Commits: conventional style (`feat(scope): summary`)
- Before pushing: `pnpm lint && pnpm test`

## Don't
- Don't print secret values in terminal output
- Don't commit `.env*` files
- Don't run `db:migrate` on prod without explicit confirmation
```

---

## 7. Sources — verified 2026-04-22

### Official docs
1. **AGENTS.md spec** — https://agents.md/
2. **Claude Code memory** — https://docs.anthropic.com/en/docs/claude-code/claude-md
3. **Cursor rules (new `.mdc`)** — https://design.dev/guides/cursor-rules/
4. **Cursor mdc vs .cursorrules 2026** — https://thepromptshelf.dev/blog/cursorrules-vs-mdc-format-guide-2026
5. **OpenAI Codex AGENTS.md** — https://developers.openai.com/codex/guides/agents-md
6. **OpenAI Codex onboarding** — https://developers.openai.com/codex/use-cases/codebase-onboarding
7. **OpenCode rules** — https://open-code.ai/en/docs/rules
8. **OpenCode agent config** — https://opencodeguide.com/en/docs/configure/agents/
9. **Windsurf Cascade docs** — https://docs.windsurf.com/context-awareness/fast-context
10. **Gemini CLI context** — https://google-gemini.github.io/gemini-cli/docs/cli/gemini-md.html
11. **Gemini CLI memory mgmt** — https://geminicli.com/docs/cli/tutorials/memory-management/
12. **Antigravity rules/workflows** — https://antigravity.google/docs/rules-workflows
13. **Antigravity AGENTS.md guide** — https://antigravity.codes/blog/user-rules
14. **GitHub Copilot repo instructions** — https://docs.github.com/en/copilot/customizing-copilot/adding-repository-custom-instructions-for-github-copilot
15. **GitHub Copilot coding-agent best practices** — https://docs.github.com/enterprise-cloud@latest/copilot/tutorials/coding-agent/get-the-best-results
16. **Junie IDE plugin** — https://junie.jetbrains.com/docs/junie-ide-plugin.html
17. **Junie project settings** — https://junie.jetbrains.com/docs/junie-plugin-project-settings.html
18. **Junie Playbook** — https://www.jetbrains.com/guide/ai/article/junie/intellij-idea/
19. **Warp rules** — https://docs.warp.dev/agent-platform/capabilities/rules
20. **Warp guide: create project rules** — https://docs.warp.dev/guides/configuration

### Real repo examples
21. **Vercel Next.js AGENTS.md** — https://github.com/vercel/next.js/blob/canary/AGENTS.md
22. **Vercel Workflow AGENTS.md** — https://github.com/vercel/workflow/blob/main/AGENTS.md
23. **Payload CMS CLAUDE.md** — https://github.com/payloadcms/payload/blob/main/CLAUDE.md
24. **LobeHub CLAUDE.md** — https://github.com/lobehub/lobe-chat/blob/next/CLAUDE.md
25. **PatrickJS/awesome-cursorrules** — https://github.com/PatrickJS/awesome-cursorrules
26. **JetBrains/junie-guidelines** — https://github.com/JetBrains/junie-guidelines
27. **MichaelFisher1997/opencode-config** — https://github.com/MichaelFisher1997/opencode-config

### Best-practice guides 2025-2026
28. **Augment Code — How to Build Your AGENTS.md** — https://www.augmentcode.com/guides/how-to-build-agents-md
29. **aihackers — AGENTS.md: What Actually Works** — https://aihackers.net/posts/agents-md-practical-guide/
30. **Developer Toolkit — Claude Code memory system** — https://developertoolkit.ai/en/claude-code/advanced-techniques/memory-system/
31. **Claude Code Handbook — Use CLAUDE.md** — https://nikiforovall.github.io/claude-code-rules/fundamentals/use-claude-md
32. **Buildcamp — Ultimate Guide to CLAUDE.md** — https://www.buildcamp.io/guides/the-ultimate-guide-to-claudemd
33. **Amit Ray — Best Practices for CLAUDE.md** — https://amitray.com/best-practices-for-claude-md/
34. **Morph — Agentic Search: How Coding Agents Find Code** — https://www.morphllm.com/agentic-search
35. **Cursor Blog — Securely Indexing Large Codebases** — https://cursor.com/blog/secure-codebase-indexing


<!-- Reference links inline -->
[1]: https://agents.md/
[2]: https://www.augmentcode.com/guides/how-to-build-agents-md
[3]: https://medium.com/@zerofilter/why-claude-code-is-special-for-not-doing-rag-vector-search-agent-search-tool-calling-versus-41b9a6c0f4d9
[4]: https://developertoolkit.ai/en/claude-code/advanced-techniques/memory-system/
[5]: https://github.com/vercel/next.js/blob/canary/AGENTS.md
[6]: https://github.com/payloadcms/payload/blob/main/CLAUDE.md
[7]: https://github.com/lobehub/lobe-chat/blob/next/CLAUDE.md
[8]: https://thepromptshelf.dev/blog/migrate-cursorrules-to-claude-md
[9]: https://forum.cursor.com/t/my-take-on-cursor-rules/67535
[10]: https://github.com/PatrickJS/awesome-cursorrules
[11]: https://github.com/vercel/workflow/blob/main/AGENTS.md
[12]: https://github.com/openai/codex/issues/17401
[13]: https://github.com/MichaelFisher1997/opencode-config
[14]: https://github.com/anomalyco/opencode/issues/11534
[15]: https://github.com/google-gemini/gemini-cli/issues/16999
[16]: https://antigravity.codes/blog/user-rules
[17]: https://github.com/JetBrains/junie-guidelines
[18]: https://docs.warp.dev/guides/configuration

---

*Последнее обновление: 2026-04-22. Материал курса AI-Driven Development, M2.*
