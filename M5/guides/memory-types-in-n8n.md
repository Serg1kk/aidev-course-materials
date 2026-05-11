# Типы памяти в n8n — что выбрать в production

> Memory ноды AI Agent — критический выбор для cost и stability. Этот guide — практическое сравнение 5 типов с production-сценариями.

---

## Краткая сводка

| Тип | Что | Pricing impact | Когда |
|---|---|---|---|
| **Simple Memory** | История в Python-dict внутри процесса | бесплатно | Только тесты / notebooks. **НЕ работает в queue mode** |
| **Window Buffer Memory** | Последние N сообщений в скользящем окне | константный (контролируемый) | **Default для большинства агентов** |
| **Conversation Buffer Memory** | Полная история диалога | растёт линейно с числом ходов | **Никогда в продакшен** |
| **Postgres Chat Memory** | Сохранение состояния через Postgres-таблицу | стоимость БД + токены | Многопользовательские системы, выживает рестарты |
| **Vector Store Memory** | Эмбеддинги в Pinecone/Qdrant/Supabase | стоимость embedding + БД | Семантическое извлечение, дороже в инфраструктуре |

---

## 1. Simple Memory

### Что

Хранит историю в Python-dict внутри процесса n8n. Сессионная, теряется после рестарта.

### Когда

- Тестирование на dev-машине
- Notebook-style эксперименты
- Демо без production-нагрузки

### Когда **не** использовать

- В **queue mode** (n8n с несколькими воркерами) — n8n не гарантирует, что вызов попадёт на тот же воркер, и Simple Memory у соседнего воркера будет пустой
- В production вообще — состояние теряется после рестарта пода / деплоя

### Как настроить

В AI Agent ноде → подключить Simple Memory как sub-node memory connector.

---

## 2. Window Buffer Memory ⭐

### Что

Хранит последние N сообщений в скользящем окне. Старые сообщения отсекаются автоматически. Стоимость в токенах остаётся **постоянной**.

### Когда

**Default выбор для большинства агентов.** Особенно подходит для:

- Триаж и классификация — `length=5`
- Разговорные ассистенты поддержки — `length=20`
- Single-turn задачи с контекстом — `length=3-5`

### Преимущество

Стоимость токенов **не растёт** с числом взаимодействий — окно всегда фиксированного размера.

### Как настроить

В AI Agent ноде → Window Buffer Memory sub-node → параметр `Context Window Length` = N.

---

## 3. Conversation Buffer Memory ⚠️

### Что

Полная история диалога. **Каждый запуск отправляет в LLM весь предыдущий диалог**.

### Когда

**Никогда в production.** Только для демо где число сессий <100.

### Антипаттерн (реальный кейс перерасхода)

> Команда поставила Conversation Buffer Memory без ограничения окна на агента первичной сортировки. **500 запусков в день**, средняя сессия 30 сообщений. Через месяц — счёт OpenAI **$400**.
>
> Перевод на Window Buffer Memory `length=5` — счёт упал до **$40/мес** в десять раз.

### Если уже стоит — как мигрировать

1. В AI Agent ноде заменить sub-node memory с Conversation Buffer на Window Buffer
2. Указать разумное `length` (5-20)
3. Прогнать batch tests чтобы убедиться что качество ответов не упало

---

## 4. Postgres Chat Memory

### Что

Сохранение состояния через Postgres-таблицу. Подходит для многопользовательских систем, выживает рестарты, разделяется между воркерами.

### Когда

- **Многопользовательские системы** с user_id как ключом сессии
- Long-running агенты, которые нужно возобновлять после рестарта
- Compliance-требования к persistent audit trail

### Production gotcha — n8n issue #12958

Настройка `Context Window Length` в Postgres Chat Memory **не уважается** частью узлов (на момент апреля 2026, см. [issue #12958](https://github.com/n8n-io/n8n/issues/12958)).

**Симптом:** по логам OpenAI улетает не последние N сообщений, а вся таблица истории — десятки килобайт текста, latency 4 сек → 1 мин, счёт растёт пропорционально.

**Workaround:** Code-узел перед AI Agent, который вручную обрезает историю до запроса:

```javascript
// В Code Node перед AI Agent
const fullHistory = $input.all();
const N = 10;  // желаемое окно
const trimmed = fullHistory.slice(-N);
return trimmed;
```

> **Урок:** декларативные настройки в low-code требуют **независимой проверки** реального ввода в LLM через логи исполнения. Доверяйте тому, что видите в логах LLM, а не тому, что декларировано в UI.

### Как настроить

1. Поднять Postgres (cloud / self-host)
2. В n8n создать credentials Postgres
3. AI Agent → Postgres Chat Memory sub-node → выбрать credentials → задать `Session ID Type` (например, `feature_id` или `user_id` из payload)

---

## 5. Vector Store Memory

### Что

Эмбеддинги в Pinecone, Qdrant, Weaviate, Supabase pgvector, или Chroma. Семантическое извлечение релевантных частей истории, **а не** последних N.

### Когда

- Долговременные агенты с большой базой знаний
- Семантический recall — «найди в истории что я говорил про X»
- Сценарии где Window Buffer теряет важный контекст

### Антипаттерн перерасхода

> Команда поставила Vector Store Memory на агента классификации обращений, рассчитывая на семантический поиск. **$200/мес на эмбеддинги** при том, что для классификации достаточно последних 3 сообщений. Window Buffer `length=3` закрыл задачу за **$15/мес**.

### Как выбирать между Window Buffer и Vector Store

| Сценарий | Выбор |
|---|---|
| Триаж / классификация / простые ответы | Window Buffer |
| Разговор требует помнить «5 ходов назад» | Window Buffer length=10-20 |
| Разговор требует «ты говорил про X неделю назад» | Vector Store |
| Mixed — base flow Window Buffer, deep recall Vector Store | оба + roouting |

---

## Эмпирические рекомендации (production summary)

| Use case | Memory | Length / config |
|---|---|---|
| Триаж и классификация | Window Buffer | `length=5` |
| Разговорные ассистенты поддержки | Window Buffer | `length=20` |
| Многопользовательские системы | Postgres Chat | per-user session_id |
| Долговременные агенты с базой знаний | Vector Store | + embedding model |
| Демо | Conversation Buffer | в продакшен **никогда** |

---

## Memory layer как отдельная категория (для backlog)

Кроме встроенных n8n-нод существуют **внешние memory-системы**, которые подключаются через MCP / HTTP. На 2026 год доминируют:

| Vendor | Architecture | LongMemEval | Sweet spot |
|---|---|---|---|
| **Mem0** | Vector + graph + KV | 26-49% | AWS Agent SDK exclusive provider |
| **Zep (Graphiti)** | Temporal knowledge graph | **94.8% (+15pp)** | Compliance, audit-friendly |
| **Letta** (ex-MemGPT) | OS-paged virtual context | 93.4% | Long-running enterprise |
| **LangMem** | Open SDK | n/a | LangChain teams, MIT |

**Когда брать внешний memory layer:**
- Нужна семантическая память **между сессиями** (не только внутри)
- Compliance / audit — «что агент знал в момент X» (Zep temporal queries)
- Long-running enterprise агенты с pluggable backends (Letta)
- AWS Bedrock-стек (Mem0 — exclusive)

Подробнее — [LMS 5.3 «Управление состоянием»](#) + публичный обзор [«Agent Long-Term Memory in 2026»](https://medium.com/@aviadr1/agent-long-term-memory-in-2026-letta-mem0-zep-and-langmem-compared).

---

## Применение в домашке M5

### WF1 — Manual trigger

**Window Buffer Memory `length=5`** — короткие сессии управления feature flag, не нужна долговременная память. Дёшево, предсказуемо.

### WF2 — Scheduled monitor

**Memory вообще не нужна** или Window Buffer `length=3` (на случай если агент проверяет несколько фич за один прогон). Каждый cron-tick — отдельная задача, контекст сбрасывается.

> Если в WF2 нужен «помнить что фича X была деактивирована 5 минут назад» — это **не Memory задача**, а **persistent state** в Postgres / Redis. Memory — для контекста LLM-разговора. State — для фактов о мире.

---

## Источники

- [n8n Memory docs](https://docs.n8n.io/advanced-ai/agents/memory/)
- [n8n issue #12958 — Postgres Chat Memory bug](https://github.com/n8n-io/n8n/issues/12958)
- [Mem0 LOCOMO benchmark paper](https://arxiv.org/abs/2504.19413)
- [Agent Long-Term Memory in 2026 — comparative review](https://medium.com/@aviadr1/agent-long-term-memory-in-2026-letta-mem0-zep-and-langmem-compared)

---

*Distilled production knowledge. M5 HSS AI-dev L1.*
