# Модуль 3 — Домашнее задание

> **Тема:** Контекст и данные — RAG / MCP. Собираем «исполнительный слой»: Feature Flags MCP + RAG над документацией.
> **Сложность:** Senior+ (или middle с готовностью разобраться)
> **Время:** ~5–7 часов (Часть 1: 30 мин, Часть 2: 3–4 ч, Часть 3: 1–2 ч, Часть 4 опц: 1–2 ч)
> **Дедлайн:** объявляется отдельно перед стартом M4.
> **Куда сдавать:** ссылка на GitHub-репо + `report.md` внутри — в LMS или чат курса.

---

## TL;DR — что нужно сделать

- **Часть 1:** написать MCP-сервер (3 tools) для управления feature flags `proshop_mern`. Входные данные уже готовы — лежат в репо курса.
- **Часть 2:** поднять RAG на ~50 K слов документации `proshop_mern`. Стек — свободный выбор. Прогнать 3 тестовых запроса.
- **Часть 3:** обернуть RAG в MCP-инструмент, подключить оба сервера к одному агенту, пройти end-to-end сценарий.
- **Часть 4 (опц):** Hybrid search + Reranker — для тех, кто хочет погонять цифры.

Сквозной артефакт — `proshop_mern`. Его MCP-слой (Часть 1) переедет в M4 как основа UI. В M5 тот же стек оборачивается в n8n-агент.

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

### Запись занятия

Ссылка на запись доступна в после занятия. Там разобраны 4 подхода к retrieval (file-based / vector DB / LightRAG / NotebookLM) на одном корпусе — полезный контекст перед Частью 2.

---

## Часть 1 — Feature Flags MCP server (~30 мин)

### Контекст

LMS 3.3 даёт базовый «MCP за 10 минут». Часть 1 расширяет этот сценарий под сквозной проект курса.

В M4 эти же 3 tools становятся основой для UI-дашборда (React). В M5 агент n8n вызывает тот же MCP. Написать один раз — использовать три модуля подряд.

### Что даётся

Всё уже лежит в `aidev-course-materials/M3/project-data/`:

- **`features.json`** — 25 фичей proshop_mern. Структура каждой: `name`, `status` (Disabled / Testing / Enabled), `traffic_percentage` (0–100), `last_modified`, `depends_on` (массив зависимостей).
- **`feature-flags-spec.md`** — ~5 K слов: что делает каждая фича, какие состояния допустимы, логика зависимостей, контракт 3 tools.

Прочитайте `feature-flags-spec.md` перед тем как писать код — там описана бизнес-логика валидации.

### Что нужно реализовать

**3 tools MCP-сервера:**

1. **`get_feature_info(feature_name: str)`** — возвращает полные данные по фиче: status, traffic_percentage, last_modified, depends_on. Включает state зависимостей.

2. **`set_feature_state(feature_name: str, state: str)`** — меняет статус. Валидация: нельзя перевести в Enabled, если хотя бы одна зависимость в Disabled. Возвращает итоговый state + список зависимостей.

3. **`adjust_traffic_rollout(feature_name: str, percentage: int)`** — устанавливает процент трафика 0–100. Жёсткий lock: `percentage > 0` невозможен при `status = Disabled`. Обновляет `last_modified`.

**Требования к описаниям tools** (по принципам из `mcp-design-principles.md`, п. 2):

- Что делает + когда вызывать + когда **не** вызывать.
- Явный формат входа/выхода.
- 1–3 примера вызова.
- Императив `"You MUST"` для критичных ограничений.

### Шаги

1. Выбрать стек: **TypeScript SDK + Zod** (рекомендуется если ваш основной стек JS/TS) или **Python FastMCP** (быстрее старт для Python-разработчиков). Зафиксировать выбор в `README.md`.
2. Создать MCP-сервер в одном файле (target — 40–60 строк кода).
3. Реализовать 3 tools с валидацией зависимостей.
4. Подключить к Cursor / Claude Desktop / Claude Code / любой IDE.
5. Пройти тестовый сценарий (см. ниже).

### Тестовый сценарий

Открыть чат с агентом и дать промпт:

```
Проверь состояние фичи search_v2 в proshop_mern feature flags.
Если она в статусе Disabled — переведи в Testing.
Установи трафик на 25%.
Подтверди финальное состояние.
```

Ожидаемый результат: агент сам выстраивает цепочку из 3 tool calls — `get_feature_info` → `set_feature_state` → `adjust_traffic_rollout` → снова `get_feature_info` для подтверждения.

### Что сдать

- Код MCP-сервера (1 файл, 40–60 строк) в GitHub-репо.
- Скриншот списка tools в IDE (MCP Inspector или UI).
- Лог 2–3 успешных диалогов с цепочкой tool calls (текстовый вывод или скриншот).

### Подсказки

- TypeScript + Zod: описание инструментов в `z.object({})` + `.describe()` на каждом поле — модель понимает контракт без дополнительной документации.
- FastMCP: декоратор `@mcp.tool()` + аннотации типов — самый быстрый путь от нуля до рабочего сервера.
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

| Layer | Варианты | Рекомендация |
|-------|----------|--------------|
| **Vector DB** | Qdrant / Weaviate / Supabase pgvector / Pinecone / LightRAG | Qdrant (default — Docker или cloud free tier); Supabase pgvector если уже есть Postgres |
| **Embedding** | OpenAI text-embedding-3-small / BGE-M3 / Cohere multilingual v3 / Voyage-3-large / Gemini Embed 2 | OpenAI 3-small для старта ($0.02/1M, быстро); BGE-M3 если нужен self-host или нет ключей OpenAI |
| **Chunking** | 300–512 токенов, overlap 20% | Recursive Markdown Splitter — не разрывает заголовки и bullet-блоки |
| **Framework** | LangChain / LlamaIndex / прямой SDK / n8n | Прямой SDK если хочется меньше абстракций; LangChain если нужны цепочки |

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

- GitHub-репо с ingestion script + query script.
- 2–3 примера запросов с top-K результатами (текст, лог команды или скриншот).
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

### End-to-end сценарий

Ключевая часть сдачи. Дать агенту промпт:

```
Найди в документации proshop_mern что такое фича payment_stripe_v3
и какие у неё зависимости.
Проверь текущее состояние этой фичи через feature flags.
Если она в статусе Disabled и все её зависимости не в Disabled,
переведи в Testing и установи трафик 25%.
Процитируй из документации зачем эта фича нужна.
```

Ожидаемая цепочка tool calls:
1. `search_project_docs("payment_stripe_v3 dependencies")` — получает chunks из документации.
2. `get_feature_info("payment_stripe_v3")` — смотрит текущий статус + зависимости.
3. Агент анализирует зависимости.
4. `set_feature_state("payment_stripe_v3", "Testing")`.
5. `adjust_traffic_rollout("payment_stripe_v3", 25)`.
6. `get_feature_info("payment_stripe_v3")` — подтверждение.
7. Агент цитирует из chunks что возвращал `search_project_docs`.

Если агент пропустит шаги или перепутает порядок — это тоже интересный результат, зафиксируйте в `report.md`.

### Что сдать

- Код Search MCP-сервера (отдельный файл или папка рядом с Частью 1).
- Лог end-to-end сценария: агент использует оба MCP в одном диалоге.
- Одна строка в `report.md`: какой IDE использовали для подключения обоих серверов.

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

После hybrid retrieval top-25 → reranker → top-5:

- **BGE-reranker-v2-m3** (free, HuggingFace): `pip install sentence-transformers`. Работает локально, ~200 ms GPU / ~2 s CPU.
- **Cohere Rerank v3**: $1/1K requests, +25% accuracy, managed API.

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

Берите BGE-M3: работает локально на M1/M2 Mac без API ключей. Установка: `pip install sentence-transformers`, модель скачивается автоматически при первом вызове (~500 MB). Качество на русском контенте — лучше OpenAI 3-small.

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

- [ ] MCP-сервер feature flags (Часть 1) — код + скриншот tools + лог диалогов.
- [ ] Ingestion + query scripts (Часть 2) — код + примеры запросов с top-K.
- [ ] Search MCP-сервер (Часть 3) — код + лог end-to-end сценария.
- [ ] `report.md` — reflection + stack choice + IDE.

**По желанию:**

- [ ] Hybrid search + reranker (Часть 4) — таблица с тремя режимами.
- [ ] Сравнение нескольких embedding-моделей на одном запросе.
