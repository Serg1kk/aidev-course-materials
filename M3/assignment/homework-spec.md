# Модуль 3 — Домашнее задание

> **Тема:** Контекст и данные — RAG / MCP. Собираем «исполнительный слой»: Feature Flags MCP + RAG над документацией.
> **Сложность:** Senior+ (или middle с готовностью разобраться)
> **Время:** ~5–7 часов (Часть 1: 30 мин, Часть 2: 3–4 ч, Часть 3: 1–2 ч, Часть 4 опц: 1–2 ч)
> **Дедлайн:** объявляется отдельно перед стартом M4.
> **Куда сдавать:** ссылка на GitHub-репо. Логи MCP — в существующем `report.md` в корне репо (он у вас уже есть с M2), новая секция `## M3`.

---

## TL;DR — что нужно сделать

- **Часть 1:** написать MCP-сервер для управления feature flags `proshop_mern` — 3 обязательных + 1 рекомендуемый tool. Подключить `features.json` в runtime + добавить тестовую страницу `Dashboard Features` в админку. Входные данные уже готовы в репо курса.
- **Часть 2:** поднять RAG на ~50 K слов документации `proshop_mern`. Стек — свободный выбор. Прогнать 3 тестовых запроса. Все чанки положить в репо.
- **Часть 3:** обернуть RAG в MCP-инструмент, подключить оба сервера к одному агенту, пройти end-to-end сценарий. Логи обоих MCP — в `report.md`.
- **Часть 4 (опц):** Hybrid search + Reranker — для тех, кто хочет погонять цифры.

Сквозной артефакт — `proshop_mern`. Его MCP-слой (Часть 1) переедет в M4 как основа UI. В M5 тот же стек оборачивается в n8n-агент.

---

## Как сдавать

**Один артефакт — GitHub-репо** (форк `proshop_mern` или свой проект сопоставимого размера). **Никаких скриншотов в TG-чате** — формат сдачи: ссылка на репо + дописанная секция `## M3` в существующем `report.md` (он у вас уже есть с M2).

**Что должно лежать в репо:**

1. **Код двух MCP-серверов** — feature-flags и search-docs. Любой framework, любой язык. Папка не принципиальна.
2. **Конфиг подключения обоих MCP** в файле IDE-зависимом (см. таблицу ниже).
3. **`features.json`** — интегрирован в проект `proshop_mern`, подключен к feature-flags MCP в runtime.
4. **Тестовая страничка `Dashboard Features`** в админке — список всех фич + статус.
5. **Все чанки** для векторной БД (~300 штук) — в виде `.jsonl` или `.json` (text + metadata). Папка не принципиальна, путь упомянуть в `report.md`.
6. **Скрипты** ingestion / query для векторной БД.
7. **`report.md`** в корне — секция `## M3` с логами обоих MCP (генерируются агентом по двум готовым промптам из Части 3) + reflection (stack choice, что было сложно, что бы изменили).

**Quadrant / Weaviate / pgvector — поднимать не надо.** Я не дотянусь до твоей БД. Чанки в репо + логи search-MCP с top-K результатами в `report.md` — self-sufficient.

**MCP framework / Vector DB / Embedding — любые на выбор.** Жёсткой привязки нет:
- MCP framework: Python FastMCP / TS SDK / mcp-framework / Go / Java / .NET MCP — что удобно.
- Vector DB: Qdrant / Weaviate / Supabase pgvector / Pinecone / LightRAG.
- Embedding: BGE-M3 локально / Cohere multilingual / OpenAI / Voyage / Gemini Embed / Ollama embed.
- IDE для подключения MCP — любая.

### Где лежит конфиг подключения MCP — по IDE

| IDE | Project-level (рекомендуется для сдачи) | Global |
|---|---|---|
| **Claude Code** | `.mcp.json` в корне репо | `~/.claude/settings.json` |
| **Cursor** | `.cursor/mcp.json` в корне репо | `~/.cursor/mcp.json` |
| **VS Code (Copilot)** | `.vscode/mcp.json` в корне репо | platform-specific |
| **Codex CLI** | `.codex/config.toml` в корне (TOML, не JSON) | `~/.codex/config.toml` |
| **Windsurf (Cascade)** | нет нативного project-level — **скопируйте файл из `~/.codeium/windsurf/mcp_config.json` в корень репо** (например, как `mcp_config.json` или `windsurf-mcp.json`), чтобы я видел вашу настройку | `~/.codeium/windsurf/mcp_config.json` |
| **OpenCode** | `opencode.json` в корне | `~/.config/opencode/opencode.json` |
| **Gemini CLI** | `.gemini/settings.json` в корне | `~/.gemini/settings.json` |

**Сдавайте project-level конфиг** (если IDE поддерживает) — он попадает в репо вместе с кодом, я вижу настройку. Если ваша IDE только global (например Windsurf) — просто скопируйте этот файл и положите в корень репо, чтобы я мог проверить.

---

## Подготовка — что прочитать ДО старта

### Гайды в репо курса (`M3/guides/`)

| Файл | Когда брать |
|------|-------------|
| `vector-db-comparison-2026.md` | Перед Частью 2 — выбор Vector DB |
| `embedding-models-2026-guide.md` | Перед Частью 2 — выбор embedding-модели |
| `chunking-strategies-guide.md` | Перед Частью 2 — chunking, retrieval, reranker |
| `mcp-design-principles.md` | Перед Частями 1 и 3 — 10 принципов design MCP |
| `mcp-vs-cli-vs-skills-decision-framework.md` | Для понимания когда что выбирать |

Все гайды есть на двух языках (RU + EN). Берите удобный.

### LMS-теория

- **3.1** — RAG: chunking, embeddings, retrieval, augmented generation, CAG vs long-context.
- **3.2** — MCP: Host/Client/Server, JSON-RPC, OAuth 2.1, USB-C аналогия.
- **3.3** — Практикум: «свой MCP за 10 минут» (базовый сценарий, расширяется в Части 1).

---

## Часть 1 — Feature Flags MCP server (~30 мин)

### Контекст

LMS 3.3 даёт базовый «MCP за 10 минут». Часть 1 расширяет этот сценарий под сквозной проект курса.

В M4 эти же 3 tools становятся основой для UI-дашборда (React). В M5 агент n8n вызывает тот же MCP. Написать один раз — использовать три модуля подряд.

### Что даётся

Всё уже лежит в `aidev-course-materials/M3/project-data/`:

- **`features.json`** — 25 фичей proshop_mern. Структура каждой: `name`, `status` (Disabled / Testing / Enabled), `traffic_percentage` (0–100), `last_modified`, `depends_on` (массив зависимостей).
- **`feature-flags-spec.md`** — контракт tools + бизнес-логика валидации зависимостей. Открой если возникнут вопросы.

### Что нужно реализовать

**3 обязательных tools MCP-сервера:**

1. **`get_feature_info(feature_name: str)`** — возвращает полные данные по фиче: status, traffic_percentage, last_modified, depends_on. Включает state зависимостей.

2. **`set_feature_state(feature_name: str, state: str)`** — меняет статус. Валидация: нельзя перевести в Enabled, если хотя бы одна зависимость в Disabled. Возвращает итоговый state + список зависимостей.

3. **`adjust_traffic_rollout(feature_name: str, percentage: int)`** — устанавливает процент трафика 0–100. Жёсткий lock: `percentage > 0` невозможен при `status = Disabled`. Обновляет `last_modified`.

**4-й tool (рекомендуется):**

4. **`list_features() -> List[FeatureSummary]`** — возвращает все фичи списком (name, status, traffic_percentage). Без него агент идёт grep'ать `features.json` напрямую — диалог становится грязным. На live-практике G1 этот tool пришлось добавить по ходу. С ним лог чище и короче.

**Требования к описаниям tools** (по принципам из `mcp-design-principles.md`, п. 2):

- Что делает + когда вызывать + когда **не** вызывать.
- Явный формат входа/выхода.
- 1–3 примера вызова.
- Императив `"You MUST"` для критичных ограничений.

### Шаги

1. **Выбери любой MCP framework на удобном языке.** Python FastMCP / TypeScript SDK / mcp-framework / Go MCP SDK / Java MCP / .NET MCP — что ближе к твоему стеку. Жёсткой привязки нет. Зафиксируй выбор в `report.md`.
2. Создать MCP-сервер в одном файле (target — 40–80 строк кода).
3. Реализовать 3 обязательных tools с валидацией зависимостей (+ опционально `list_features`).
4. Подключить к Cursor / Claude Desktop / Claude Code / Codex / OpenCode / Windsurf / любой IDE с поддержкой MCP. Где лежит конфиг — см. таблицу в секции «Как сдавать» выше.
5. Пройти тестовый сценарий (см. ниже).

### Интеграция в проект

`features.json` лежит в материалах курса (`aidev-course-materials/M3/project-data/features.json`) — скачайте и положите в свой `proshop_mern`. MCP-сервер мутирует именно этот файл: `set_feature_state` / `adjust_traffic_rollout` обновляют `status`, `traffic_percentage`, `last_modified`.

**Создайте тестовую страничку `Dashboard Features` в админке проекта** — простая страница со списком всех фич + текущий статус и трафик. **Положите её именно в админку** (доступ только админ-юзеру после логина) — фичу-флаги это админская территория. На практике G1 я для скорости делал её прямо в навигационном меню одним промптом «положи сверху страничку, выведи список фич и их состояние», но в HW нормально — в админку. В **M4** мы будем делать UI-дизайн этой страницы — пока функционал минимальный (читает `features.json` + рендерит таблицу).

### Тестовый сценарий

Открой чат с агентом и дай промпт (агент сам прогонит сценарий и **сам сохранит лог в `report.md`**):

```
Проверь состояние фичи search_v2 в proshop_mern feature flags через
feature-flags MCP. Если она в статусе Disabled — переведи в Testing.
Установи трафик на 25%. Подтверди финальное состояние.

После прогона — допиши в report.md в корне репо в секцию
## M3 → ### Feature flags MCP полный лог: какие tool calls сделал
с аргументами, что MCP возвращал на каждом шаге, итоговое состояние
фичи. Markdown, чистым текстом, без скриншотов.
```

Ожидаемый результат: агент сам выстраивает цепочку из 3 tool calls — `get_feature_info` → `set_feature_state` → `adjust_traffic_rollout` → снова `get_feature_info` для подтверждения, и пишет лог в `report.md`. **Этот же лог — артефакт сдачи Части 1**, дублировать в Часть 3 не нужно.

### Что сдать

- Код MCP-сервера (один файл, 40–80 строк) в репо.
- `features.json` интегрирован в `proshop_mern`, реально мутируется сервером.
- Тестовая страничка `Dashboard Features` в админке.
- Лог тестового сценария (см. Часть 3 — генерируется агентом по готовому промпту в `report.md`).

### Подсказки

- **Python FastMCP:** декоратор `@mcp.tool()` + аннотации типов — самый быстрый путь от нуля до рабочего сервера.
- **TypeScript SDK + Zod:** описание инструментов в `z.object({})` + `.describe()` на каждом поле — модель понимает контракт без дополнительной документации.
- **mcp-framework (TS, opinionated):** ещё короче для TS-разработчиков.
- **Java MCP SDK / .NET MCP SDK / Go MCP SDK** — официальные SDK от Anthropic + community, бери если основной стек именно такой.
- Если сомневаетесь в логике зависимостей — откройте `feature-flags-spec.md`, там есть decision table.

---

## Часть 2 — RAG на documentation corpus (~3–4 часа) ⭐ центральная

### Контекст

Это основная задача модуля. Вы строите полный retrieval pipeline: ingest → embed → query. Стек — полностью ваш выбор. Обоснование выбора обязательно.

### Что даётся

`aidev-course-materials/M3/project-data/` — ~50 K слов документации proshop_mern:

| Папка / Файл | Содержимое |
|---|---|
| `architecture.md` | System overview, MERN layers, deployment |
| `glossary.md` | 62+ терминов (domain / tech / ops / business) |
| `best-practices.md` | 47 источников 2024–2026 |
| `dev-history.md` | История проекта v0.1 → v2.7 |
| `feature-flags-spec.md` | 25 флагов + контракт tools |
| `features/` | 6 файлов по фичам (auth, catalog, cart, checkout, payments, admin) |
| `pages/` | 16 файлов (страницы UI + INDEX) |
| `api/` | 5 файлов, 22 endpoints |
| `adrs/` | 5 Architecture Decision Records |
| `runbooks/` | 6 операционных гайдов |
| `incidents/` | 3 postmortem-отчёта |

Все файлы в Markdown с frontmatter. Используйте `type` из frontmatter как метаданные при инжесте.

### Свободный выбор стека

| Layer | Варианты |
|-------|----------|
| **Vector DB** | Qdrant / Weaviate / Supabase pgvector / Pinecone / LightRAG — все равны |
| **Embedding** | OpenAI text-embedding-3-small / BGE-M3 (self-host, MIT) / Cohere multilingual v3 / Voyage-3-large / Gemini Embed 2 / Ollama embed-моделями (`nomic-embed-text`, `mxbai-embed-large`) |
| **Chunking** | 300–512 токенов, overlap 20%, Recursive Markdown Splitter (не разрывает заголовки и bullet-блоки) |
| **Framework** | LangChain / LlamaIndex / прямой SDK / n8n — что удобно |

Жёсткой привязки к конкретной БД, embedding-модели или фреймворку нет. Зафиксируйте выбор в `report.md` и почему его выбрали.

**Ограничения, которых нет:**

- Cloud vs self-host — оба варианта ОК.
- OpenAI API vs open-source vs self-host — выбирайте что удобнее.
- Язык: Python, TypeScript, Node.js — всё принимается.

### Шаги

**Шаг 1 — Setup vector DB.**

Qdrant (быстрый старт):
```bash
docker run -d -p 6333:6333 qdrant/qdrant
```

Supabase pgvector — бесплатный tier на supabase.com, `CREATE EXTENSION IF NOT EXISTS vector`.

**Шаг 2 — Ingestion script.**

Логика:
1. Сканировать все `.md` файлы в `project-data/`.
2. Извлекать frontmatter: `source_file`, `type` (runbook / incident / adr / feature / api / etc.), `last_modified`.
3. Разбивать на chunks по 300–512 токенов, overlap ~20%.
4. Эмбеддировать каждый chunk.
5. Upsert в vector DB с метаданными.

Итого: ~80–100 chunks для всего corpus.

**Шаг 3 — Query script.**

Функция `search(query: str, top_k: int = 5) -> List[Chunk]`:
1. Эмбеддировать запрос той же моделью.
2. Cosine similarity поиск.
3. Опционально: pre-filter по `type` из payload.
4. Вернуть top-K с метаданными.

**Шаг 4 — Тестовые запросы (минимум 3).**

Прогоните следующие:

1. *«Какая БД используется в proshop_mern и почему именно она?»* — factual single-hop. Ожидаемый top chunk: из `adrs/adr-001-mongodb-vs-postgres.md`.

2. *«Какие фичи зависят от payment_stripe_v3?»* — multi-hop dependency. Ожидаемый top chunk: из `features/payments.md` или `feature-flags-spec.md`.

3. *«Что случилось во время последнего incident с checkout?»* — filter by type + retrieval. Ожидаемый top chunk: из `incidents/`.

Зафиксируйте top-K результаты по каждому запросу (текстом или скриншотами).

### Что сдать

- Код в репо: ingestion script + query script.
- **Все ~300 чанков в репо** в виде `.jsonl` или `.json` (text + metadata). Чанков немного — кладите целиком, не sample. Папка не принципиальна (рядом со скриптами / в `project-data/chunks/` / где удобнее), путь упомянуть в `report.md`.
- Логи 3 тестовых запросов — генерируются агентом по готовому промпту в `report.md` (см. Часть 3).
- **Reflection в `report.md`** (обязательно, 5–10 предложений): какой стек выбрали, почему, что было сложно, что бы изменили.

### Антипаттерны — учитывайте

| Антипаттерн | Что происходит | Как избежать |
|-------------|---------------|--------------|
| Английская embedding-модель на русском контенте | OpenAI 3-small MIRACL 44 vs BGE-M3 MIRACL 67.8 — пропасть | Для русских запросов: BGE-M3 или Cohere multilingual |
| Top-K > 8 в context window | Генерация разжижается нерелевантными chunks | K = 4–6 для focused задач |
| Смена embedding-модели без полного re-ingest | Разные embedding-пространства смешаны — тихая деградация | Один индекс = одна модель. Миграция: параллельный индекс |
| Chunking без учёта структуры Markdown | Заголовки разрываются, теряется контекст | Recursive Markdown Splitter — он знает `##` и `*` |
| Без метаданных в payload | Нельзя фильтровать по типу документа | Всегда кладите `source_file`, `type`, `last_modified` |

Подробнее о каждом — в `M3/guides/chunking-strategies-guide.md`.

### Подсказки

- BGE-M3 работает на M1/M2 Mac локально через `sentence-transformers` — `pip install sentence-transformers`, модель ~500 MB при первом скачивании.
- Qdrant Python client: `pip install qdrant-client`. TypeScript: `npm install @qdrant/qdrant-js`.
- Если не хочется поднимать инфраструктуру совсем — NotebookLM путь: добавьте все `.md` из `project-data/` как sources в Google NotebookLM, прогоните 3 запроса. Это не заменит полноценный RAG, но покажет суть retrieval без кода. Скилл `M3/skills/notebooklm/SKILL.md` автоматизирует добавление источников через Claude Code.

---

## Часть 3 — Search MCP wrap (~1–2 часа)

### Контекст

Обернуть RAG из Части 2 в MCP-инструмент и соединить с MCP из Части 1. Один агент — оба сервера — один диалог.

Это не просто «ещё один сервер». Это демонстрация ключевого принципа M3: инструменты retrieval и инструменты управления состоянием работают как единый слой, а агент сам решает что когда вызывать.

### Что нужно реализовать

**1 tool MCP-сервера:**

```
search_project_docs(query: str, top_k: int = 5) -> List[Chunk]
```

Под капотом — ваш retrieval pipeline из Части 2. MCP-сервер принимает запрос и возвращает top-K chunks с метаданными (`source_file`, `type`, `content_snippet`).

Описание tool по принципам из `mcp-design-principles.md`:
- Когда вызывать: «поиск информации о proshop_mern в документации — архитектура, фичи, ADR, runbooks, incidents».
- Когда не вызывать: «текущее состояние feature flags — для этого есть `get_feature_info`».

### Подключить оба MCP к одному агенту

Итоговый набор в конфиге агента:
```json
{
  "mcpServers": {
    "feature-flags": { "command": "...", "args": ["..."] },
    "search-docs": { "command": "...", "args": ["..."] }
  }
}
```

Формат конфига зависит от IDE: Claude Desktop — `~/.claude/desktop/config.json`; Cursor — `.cursor/mcp.json`; Claude Code — `.claude/settings.json`.

### End-to-end сценарий + логи MCP в `report.md`

В корне репо у вас уже есть `report.md` после M2. Допишите туда секцию `## M3` с логами обоих MCP. **Не пишите эти логи руками** — попросите своего агента (Claude Code, Cursor, Codex, OpenCode, Windsurf — любой) сам их сохранить.

> **Лог feature-flags MCP** уже сделан в Части 1 (тестовый сценарий с `search_v2`). Допишется в ту же секцию `## M3 → ### Feature flags MCP` в `report.md`.

Готовые промпты для оставшихся двух блоков:

#### Промпт 1 — search-docs MCP

```
Через search-docs MCP найди ответы на 3 вопроса по документации proshop_mern:
1) Какая БД используется и почему?
2) Какие фичи зависят от payment_stripe_v3?
3) Что случилось во время последнего incident с checkout?

После прогона — допиши в report.md в секцию ## M3 → ### Search-docs MCP
полный лог: каждый запрос, какие top-K чанки MCP вернул (id / score /
source_file / фрагмент текста), как ты на их основе сформулировал ответ.
Markdown.
```

#### Промпт 2 — end-to-end оба MCP

```
Через два MCP (feature-flags + search-docs):
1. Найди в документации proshop_mern что такое фича payment_stripe_v3
   и какие у неё зависимости (search-docs MCP).
2. Проверь текущее состояние через feature-flags MCP.
3. Если она в статусе Disabled и все зависимости не в Disabled —
   переведи в Testing, установи трафик 25%.
4. Процитируй из документации зачем эта фича нужна.

После прогона — допиши в report.md в секцию ## M3 → ### End-to-end
полный лог: цепочку tool calls обоих MCP, что вернул search, что вернул
feature-flags, итоговое состояние фичи, цитату из документации.
Markdown.
```

Ожидаемая цепочка в end-to-end: `search_project_docs` → `get_feature_info` → анализ зависимостей → `set_feature_state` → `adjust_traffic_rollout` → `get_feature_info` для подтверждения → цитата из chunks. Если агент пропустит шаги или перепутает порядок — это тоже интересный результат, зафиксируйте в `report.md`.

### Что сдать

- Код Search MCP-сервера (отдельный файл или папка рядом с Частью 1).
- Конфиг подключения **обоих MCP** в IDE-зависимом файле (см. таблицу в секции «Как сдавать»).
- Секция `## M3` в существующем `report.md` с логами обоих MCP (по 3 промптам выше) + одна строка какой IDE использовали.

---

## Часть 4 (опциональная) — Hybrid + Reranker (~1–2 часа)

Для тех, кто хочет погонять цифры и понять разницу между naive RAG и production RAG.

### Что нужно добавить поверх Части 2

**Hybrid search:**

Добавить BM25 (sparse) поверх семантического поиска. Слить результаты через RRF (Reciprocal Rank Fusion).

- Qdrant: встроенный sparse vector support (`SparseVectorParams`).
- Weaviate: hybrid search из коробки (`alpha` параметр от 0.0 pure BM25 до 1.0 pure vector).
- pgvector: `tsvector` + `to_tsquery` как sparse компонент + RRF в SQL.

**Reranker:**

После hybrid retrieval top-25 → reranker → top-5.

**Подбирайте reranker под свою embedding-модель** (важно — модели обучены парами):

- **BGE-M3** embedding → **BGE-reranker-v2-m3** — одна экосистема BAAI, обучены вместе, лучшая совместимость. Установка: `pip install sentence-transformers`, модель ~600 MB при первом скачивании. Бесплатно, локально, ~200 ms GPU / ~2 s CPU.
- **Cohere multilingual v3** embedding → **Cohere Rerank v3** — managed API, $1/1K requests, +25% accuracy, родная связка от Cohere.
- **OpenAI text-embedding-3 / Voyage-3** embedding → **BGE-reranker-v2-m3** как универсальный free вариант (работает поверх любого embedding), либо **Cohere Rerank v3** как paid managed.

Как подключить (на примере BGE-reranker-v2-m3):

```python
from sentence_transformers import CrossEncoder
reranker = CrossEncoder('BAAI/bge-reranker-v2-m3')
pairs = [[query, chunk.text] for chunk in top25_candidates]
scores = reranker.predict(pairs)
# отсортировать по scores → взять top-5
```

Для Cohere Rerank — официальный SDK `cohere`, метод `client.rerank(query=..., documents=..., model="rerank-multilingual-v3.0", top_n=5)`.

### Что замерить

Прогоните **те же 3 запроса** из Части 2 в трёх режимах:

| Режим | MRR | Recall@5 |
|-------|-----|----------|
| Naive RAG (ваш baseline из Части 2) | ? | ? |
| Hybrid BM25 + vector (RRF) | ? | ? |
| Hybrid + Reranker | ? | ? |

Reference numbers для ориентира (из `chunking-strategies-guide.md`):

```
baseline naive RAG              MRR ~0.52
  + hybrid (BM25+vector RRF) →  MRR ~0.71
  + reranker (Cohere/BGE)    →  MRR ~0.82–0.85
```

Без golden dataset точный MRR не посчитать — оценивайте subjectively: у какого режима top-1 чаще попадает в правильный chunk.

### Что сдать

- Таблица с результатами (заполненная выше).
- Reflection (1 абзац): что дала каждая ступень, какой прирост ощутили.

---

## Критерии оценки

### Часть 1

- [x] MCP-сервер запускается без ошибок.
- [x] 3 tools подключены к IDE, видны в списке tools.
- [x] Тестовый сценарий (`search_v2`: Disabled → Testing → 25%) прошёл без ручного вмешательства.
- [x] Логи 2–3 диалогов приложены.
- [x] Описания tools содержат «когда вызывать» + «когда не вызывать».

### Часть 2

- [x] Ingestion script работает на всём `project-data/`.
- [x] Все 3 тестовых запроса прогнаны, top-K зафиксированы.
- [x] Reflection написан (5–10 предложений).
- [x] Метаданные в payload: `source_file`, `type`.

### Часть 3

- [x] `search_project_docs` tool подключён рядом с feature flags MCP.
- [x] End-to-end сценарий прошёл: агент использовал оба MCP в одном диалоге.
- [x] Лог сценария приложен.

### Часть 4 (опц)

- [x] Hybrid search реализован (BM25 + vector + RRF).
- [x] Reranker добавлен.
- [x] Таблица с тремя режимами заполнена.

---

## Cross-module hooks

Зачем это важно за пределами M3:

| Модуль | Как используется то, что вы построили |
|--------|---------------------------------------|
| **M4 — UI** | React-дашборд поверх вашего Feature Flags MCP (Часть 1). Tools становятся API для UI. |
| **M5 — Агенты + n8n** | n8n-workflow с обоими MCP (Часть 1 + Часть 3) как единый автоматизированный агент. |
| **M6 — Legacy / Code review** | AI code review `proshop_mern` с M3 RAG как knowledge base для контекста о решениях. |
| **M7 — Security / Strategy** | Анализ безопасности всего получившегося стека: MCP-серверы, vector DB, embedding API. |

Чем аккуратнее сделаете Части 1–3, тем меньше придётся переделывать на M4–M7.

---

## Полезные ресурсы

### Материалы в репо курса

| Путь | Что |
|------|-----|
| `M3/guides/vector-db-comparison-2026.md` | Qdrant / Weaviate / pgvector / Pinecone — таблица trade-offs |
| `M3/guides/embedding-models-2026-guide.md` | Все embedding-модели с ценами, бенчмарками, сценариями |
| `M3/guides/chunking-strategies-guide.md` | Chunking, overlap, retrieval, reranker — числа из production |
| `M3/guides/mcp-design-principles.md` | 10 design-принципов MCP с примерами описаний tools |
| `M3/guides/mcp-vs-cli-vs-skills-decision-framework.md` | Decision tree: MCP vs CLI vs Skills |
| `M3/skills/notebooklm/SKILL.md` | No-code путь: добавить project-data как sources в NotebookLM |
| `M3/project-data/feature-flags-spec.md` | Контракт 3 tools + бизнес-логика валидации |
| `M3/project-data/best-practices.md` | 47 источников 2024–2026 по RAG, MCP, chunking |

### MCP frameworks

| Framework | URL |
|-----------|-----|
| TypeScript SDK + Zod | https://github.com/modelcontextprotocol/typescript-sdk |
| mcp-framework (TS, opinionated) | https://github.com/QuantGeekDev/mcp-framework |
| Python FastMCP | https://github.com/jlowin/fastmcp |
| MCP Inspector (debug) | https://github.com/modelcontextprotocol/inspector |
| Anthropic tools concepts | https://modelcontextprotocol.io/docs/concepts/tools |

### Vector DB

| Tool | URL |
|------|-----|
| Qdrant | https://qdrant.tech |
| Weaviate | https://weaviate.io |
| Supabase pgvector | https://supabase.com/docs/guides/ai |
| Pinecone | https://www.pinecone.io |
| LightRAG | https://github.com/HKUDS/LightRAG |

### Embedding models

| Модель | URL |
|--------|-----|
| BGE-M3 (MIT, self-host) | https://huggingface.co/BAAI/bge-m3 |
| OpenAI embeddings | https://platform.openai.com/docs/guides/embeddings |
| Cohere multilingual | https://cohere.com/models/embed |
| Voyage-3-large | https://voyageai.com |
| MTEB Leaderboard | https://huggingface.co/spaces/mteb/leaderboard |

### Rerankers

| Tool | URL |
|------|-----|
| BGE-reranker-v2-m3 (free) | https://huggingface.co/BAAI/bge-reranker-v2-m3 |
| Cohere Rerank | https://cohere.com/rerank |
| Anthropic Contextual Retrieval | https://www.anthropic.com/news/contextual-retrieval |

### MCP каталоги и ориентиры

| Ресурс | URL |
|--------|-----|
| mcp.so (10K+ серверов) | https://mcp.so |
| Smithery | https://smithery.ai |
| Anthropic reference servers | https://github.com/modelcontextprotocol/servers |
| MCP Best Practices — Philipp Schmid | https://www.philschmid.de/mcp-best-practices |

---

## FAQ

**Q: Можно ли использовать свой проект вместо proshop_mern?**

Да — если у проекта есть Markdown-документация сопоставимого объёма (≥20 файлов / ~30 K+ слов) и есть очевидные «feature flags» или конфиги с состоянием. Но вам придётся самим подготовить `features.json`-аналог для Части 1. Если сомневаетесь — уточните до старта.

**Q: Нужно ли поднимать весь стек локально? Можно cloud?**

Любой вариант. Qdrant cloud, Supabase free tier, Pinecone serverless — всё ОК. Self-host удобнее для итераций без интернет-зависимости. Зафиксируйте в `report.md` что использовали.

**Q: У меня нет OpenAI API key. Что делать?**

Альтернатив много:
- **BGE-M3** локально через `sentence-transformers` (M1/M2 Mac, ~500 MB, без API ключей, лучшее качество на русском).
- **Cohere multilingual v3** — managed API, $0.10 / 1M токенов, многоязычный.
- **OpenRouter** с любым провайдером embedding (если уже есть ключ).
- **Ollama embed-моделями** — `nomic-embed-text`, `mxbai-embed-large`. Установка `ollama pull nomic-embed-text`, локально, бесплатно.

**Q: Какой top-K правильный?**

Для focused задач: K = 4–6. При K > 8 context window разжижается нерелевантными chunks — модель теряет фокус. Для Части 4 hybrid retrieval: K = 20–25 на первом этапе (до reranker), reranker оставляет top-5.

**Q: Обязательно ли делать Часть 3?**

Да, Часть 3 — обязательная. Она короткая (~1 час) и является финальной демонстрацией соединения обоих инструментов. Без неё M4 (UI поверх MCP) начинать сложнее.

**Q: Какие IDE принимаются?**

Любые. Cursor / Claude Code / Claude Desktop / Windsurf / Codex CLI / OpenCode — всё ОК. Укажите в `report.md` что использовали.

**Q: Зачем описывать «когда не вызывать» в tool description?**

Это ключевой принцип design MCP. Без «когда НЕ вызывать» агент вызывает инструменты в неподходящих ситуациях и делает лишние roundtrips. Описание tool — это промпт для модели, не просто документация. Подробнее: `mcp-design-principles.md`, принцип 2.

---

## Таймбюджет

| Часть | Время |
|-------|-------|
| Часть 1 — Feature Flags MCP | ~30 мин |
| Часть 2 — RAG на corpus | ~3–4 часа |
| Часть 3 — Search MCP + end-to-end | ~1–2 часа |
| **Обязательное итого** | **~5–6 часов** |
| Часть 4 (опц) — Hybrid + Reranker | +1–2 часа |

Если Часть 2 затягивается: сначала сделайте минимальную работающую версию (1 embedding model + 1 vector DB + 3 запроса), зафиксируйте. Потом итерируйте.

---

## Что в итоге лежит в репо

**Обязательно:**

- [ ] **Код двух MCP-серверов** — feature-flags (Часть 1) + search-docs (Часть 3). Любой framework, любой язык.
- [ ] **Конфиг подключения обоих MCP** — `.mcp.json` / `.cursor/mcp.json` / `.codex/config.toml` / `.vscode/mcp.json` / etc. (см. таблицу в секции «Как сдавать»).
- [ ] **`features.json`** интегрирован в `proshop_mern`, мутируется feature-flags MCP.
- [ ] **Тестовая страничка `Dashboard Features`** в админке проекта.
- [ ] **Все ~300 чанков** в репо (`.jsonl` или `.json`, путь упомянут в `report.md`).
- [ ] **Скрипты** ingestion + query для векторной БД (Часть 2).
- [ ] **`report.md` в корне репо**, секция `## M3`:
  - Логи feature-flags MCP (по промпту 1)
  - Логи search-docs MCP (по промпту 2)
  - Логи end-to-end сценария (по промпту 3)
  - Reflection (5–10 предложений): какой stack выбрали, почему, что было сложно.
  - Какую IDE использовали для подключения обоих серверов.

**По желанию (Часть 4):**

- [ ] Hybrid search + reranker — таблица с тремя режимами.
- [ ] Сравнение нескольких embedding-моделей на одном запросе.
