# Модуль 5 — Домашнее задание

> **Тема:** Агенты и n8n. Замыкаем full-stack: M3 (руки / MCP) + M4 (глаза / Dashboard) + M5 (мозг / Agent).
> **Сложность:** Middle+ (n8n визуально) / Senior+ (полный стек с self-hosted n8n + Telegram + MCP-интеграция).
> **Время:** ~3–5 часов (WF1 manual: 1.5–2 ч, WF2 scheduled: 1–2 ч, тест на галлюцинации: 30 мин, README: 30 мин).
> **Дедлайн:** объявляется отдельно перед стартом M6.
> **Куда сдавать:** PR в ваш форк `proshop_mern` — папка `homework/M5/` с JSON-экспортами + README. Ссылка в LMS / чат курса.

---

## TL;DR — что нужно сделать

Построить **два n8n-workflow** на сквозном проекте `proshop_mern`:

- **WF1 — Manual trigger из Dashboard.** Расширяете Feature Dashboard из M4: добавляете кнопки управления → клик отправляет POST на webhook n8n → AI Agent через MCP-сервер из M3 принимает решение и крутит ручки → UI обновляется.
- **WF2 — Scheduled defensive monitor.** Раз в минуту cron-trigger n8n проверяет имитацию логов ошибок: если error_rate > 5% — агент сам деактивирует фичу через MCP и шлёт алерт в Telegram.

Плюс **тест на галлюцинации**: команда с `traffic_percentage: -50` должна отвергаться **на уровне Switch-ноды и JSON Schema**, а не «на здравом смысле LLM». Это и есть Algorithm-before-AI.

**Без advanced со звёздочкой** — все делают одну и ту же базу. Бонусы (HITL, Langfuse, multi-agent) — для самопроверки, не для ранжирования.

---

## Как сдавать

**Один артефакт — PR в `proshop_mern`** с папкой `homework/M5/`:

```
proshop_mern/
├── frontend/src/screens/FeatureDashboardScreen.js   ← расширен «Auto-Pilot Controls» блоком
└── homework/M5/
    ├── README.md                       ← краткий отчёт + архитектурное описание
    ├── wf1-manual-trigger.json         ← n8n workflow JSON-экспорт
    ├── wf2-scheduled-monitor.json      ← n8n workflow JSON-экспорт
    ├── simulate_traffic.py             ← скрипт имитации трафика для WF2
    ├── logs.json                       ← пример имитации трафика (последние N событий)
    ├── trace-wf1.png                   ← скриншот executions из n8n с «рассуждением» агента
    ├── trace-wf2.png                   ← скриншот executions WF2 (срабатывание на > 5% errors)
    └── screencast.mp4 (или ссылка)     ← 3–5 минут: команда из UI → агент → MCP → UI обновляется
```

**Скринкаст** — Loom / YouTube unlisted / Telegram-storage / любой удобный хостинг. В README приложите ссылку, если файл не помещается в репо.

---

## Подготовка — что прочитать ДО старта

### LMS-теория (Knowledge Base)

- **5.1 Анатомия агента** — отличие LLM-вызова от агентского цикла. ReAct, Plan-and-Execute, Reflexion, write-manage-read цикл памяти. Мы будем пользоваться **GCAO** для system prompt каждого агента.
- **5.2 Low-code оркестрация в n8n** — AI Agent node, минимальная продакшн-связка (Chat Model + Tools + Memory + System Prompt), правило выбора интеграции (нода > HTTP > MCP), Guardrails node.
- **5.3 Управление состоянием** — типы памяти в n8n (Window Buffer / Postgres Chat / Conversation Buffer — что использовать, что **никогда** в продакшен).
- **5.4 Связка UI + Агент + MCP** — Algorithm-before-AI, Tool Isolation, structured output, eval-цикл.

### Pre-requisites — что должно работать ДО M5

- ✅ **MCP-сервер из M3** работает локально с тремя tools: `get_feature_info`, `set_feature_state`, `adjust_traffic_rollout`. JSON Schema на параметрах с `enum` / `min` / `max` / `required`. Если у вас в M3 валидации диапазонов нет — добавьте сейчас (нам это нужно для теста на галлюцинации).
- ✅ **Feature Dashboard из M4** работает: список фич, status badges, toggle, slider.
- ✅ **n8n запущен** — на выбор: cloud free tier (https://app.n8n.io), self-host docker (`docker run -p 5678:5678 n8nio/n8n`) или n8n-install (https://github.com/kossakovsky/n8n-install).
- ✅ **Telegram-бот** для алертов WF2 — создан через @BotFather, токен и chat_id в credentials n8n.

### Гайды в репо курса (`aidev-course-materials/M5/`)

| Файл | Когда брать |
|---|---|
| `agents/n8n-workflow-builder.md` | CC-агент, который строит workflow JSON из спека. Используйте если хочется ускориться — описание подбираете руками, JSON генерит агент |
| `agents/n8n-requirements-orchestrator.md` | CC-агент, который превращает user story в детальный спек workflow. Полезен если запутались с тем, что именно надо сделать |
| `gcao-prompt-templates.md` | 3 готовых GCAO-шаблона system prompt для AI Agent ноды (для WF1 manual, для WF2 monitor, для defensive guard) |
| `best-practices.md` | Сводка production-правил: нода > HTTP > MCP, Algorithm-before-AI 4 слоя guards, Tool Isolation, выбор Memory, лимит итераций |
| `dashboard-changes.md` | Как расширить Dashboard из M4 — sample snippets под React (proshop_mern stack), env-переменная `VITE_N8N_WEBHOOK_URL` |
| `sample-workflows/wf1-manual-trigger-template.json` | Стартовый JSON для WF1 — импортируете в свой n8n, настраиваете credentials под себя, дальше расширяете под задачу |
| `sample-workflows/wf2-scheduled-monitor-template.json` | Стартовый JSON для WF2 |
| `sample-workflows/simulate_traffic.py` | Эталонный скрипт имитации трафика. Можете использовать как есть или переписать под свой стек |

> ⚠️ Sample workflows — это **стартовые шаблоны, не готовое решение**. Промпт пустой, MCP-подключение placeholder, валидация не настроена. Ваша задача — допилить под себя и добиться чтобы прошёл тест на галлюцинации.

---

## WF1 — Manual trigger из Dashboard (~1.5–2 ч)

### Что делает

Пользователь нажимает кнопку в Dashboard → fetch на webhook n8n → AI Agent через MCP M3 принимает решение → возврат JSON в UI → Dashboard обновляет состояние feature flag.

### Шаги

#### 1. Расширить Feature Dashboard из M4

В `FeatureDashboardScreen.js` добавьте блок **«Auto-Pilot Controls»** — отдельная карточка с тремя кнопками для **выбранной** фичи:

- **«Запустить ручную проверку»** — POST `{webhook_url}/feature-control` с body `{"feature_id": "search_v2", "action": "check"}`
- **«Включить тестовый режим»** — `{"feature_id": "search_v2", "action": "test", "target_state": "Testing"}`
- **«Откатить фичу»** — `{"feature_id": "search_v2", "action": "rollback", "target_state": "Disabled"}`

`webhook_url` берёте из env-переменной `VITE_N8N_WEBHOOK_URL` (см. `dashboard-changes.md`). На ответ от webhook — обновляете UI: badge меняет цвет, slider обновляет процент.

#### 2. Собрать workflow в n8n

```
Webhook Trigger (POST /feature-control)
  ↓
Switch (input validation)
  ├── traffic_percentage < 0 или > 100  →  Respond 400 + сообщение
  ├── feature_id отсутствует              →  Respond 400 + сообщение
  └── всё ок                              →  следующий шаг
  ↓
AI Agent (Tools Agent)
  ├── Chat Model: ваш выбор (Claude / GPT / Gemini / OpenRouter)
  ├── Memory: Window Buffer Memory length=5
  ├── Tools: MCP Client Tool → подключение к вашему MCP M3 серверу
  │            (3 tools: get_feature_info / set_feature_state / adjust_traffic_rollout)
  └── System Prompt: GCAO (см. gcao-prompt-templates.md → Template 1)
  ↓
Structured Output Parser (JSON schema гарантирует формат)
  ↓
Respond to Webhook (JSON ответ Dashboard'у с обновлённым состоянием)
```

#### 3. System prompt по GCAO (в Agent ноде)

```
Goal:    Выполни запрос пользователя по управлению feature flag {{$json.feature_id}}.
Context: Текущее состояние получи через get_feature_info. Команда от UI: {{$json.action}}, target_state: {{$json.target_state}}.
Action:  Используй MCP-тулы (get_feature_info / set_feature_state / adjust_traffic_rollout)
         для выполнения. Проверь валидность параметров перед вызовом.
Output:  JSON: {success: bool, message: string, current_state: object}.

Constraints:
- traffic_percentage в диапазоне [0, 100]
- target_state из enum [Enabled, Disabled, Testing]
- Если параметр невалиден — вернуть {success: false, message: "..."} без вызова инструмента.
```

#### 4. Валидация в Dashboard

UI показывает ошибку если `success: false`. Если `success: true` — обновляет UI без перезагрузки страницы.

---

## WF2 — Scheduled defensive monitor (~1–2 ч)

### Что делает

Раз в минуту cron-trigger n8n проверяет имитацию логов ошибок. Если error_rate > 5% за окно — агент деактивирует фичу через MCP + шлёт алерт в Telegram.

### Шаги

#### 1. Подготовить имитацию трафика

Создать `homework/M5/simulate_traffic.py` (или возьмите эталон из `aidev-course-materials/M5/sample-workflows/`). Скрипт:

- Запускается раз в N секунд (CLI флаг `--interval`)
- Дописывает fake-events в `logs.json`:
  ```json
  {"timestamp": "2026-05-15T10:00:00Z", "feature_id": "search_v2", "status": "success"}
  {"timestamp": "2026-05-15T10:00:05Z", "feature_id": "search_v2", "status": "error"}
  ```
- Конфигурируемый `--error-rate` (0.0–1.0) — для теста срабатывания на спайке ошибок

CLI пример:
```bash
python3 simulate_traffic.py --feature-id search_v2 --error-rate 0.07 --interval 5 --duration 600
```

> ⚠️ **Логи можно хранить в Postgres / Redis Stream / любом другом storage** — JSON-файл выбран для простоты. Используйте свой стек если хочется (отметьте в README что выбрали и почему).

#### 2. Собрать workflow в n8n

```
Cron Trigger (every minute)
  ↓
Code Node (или Read Binary File + Function)
  → читает logs.json за последние N минут
  → считает error_rate = errors / total
  ↓
IF (error_rate > 5%)
  ├── да →
  │   AI Agent (Tools Agent)
  │     ├── System Prompt: GCAO (см. gcao-prompt-templates.md → Template 2)
  │     ├── Tools: get_feature_info, set_feature_state (через MCP Client Tool)
  │     └── Cвязь с MCP M3 — те же credentials что в WF1
  │   ↓
  │   Telegram Send Message
  │     → "🚨 Обнаружен всплеск ошибок: feature {{$json.feature_id}} деактивирована
  │        (error rate {{$json.error_rate}}%). Текущее состояние: {{$json.new_state}}"
  │   ↓
  │   (опционально) Postgres Insert в historical-таблицу events
  └── нет → ничего не делать (или log в historical-таблицу для аналитики)
```

#### 3. System prompt по GCAO для WF2

```
Goal:    Зарегистрировать инцидент: feature {{$json.feature_id}} имеет error rate {{$json.error_rate}}%.
Context: Threshold 5%. Превышение требует немедленного действия — деактивировать feature.
         Текущее состояние получи через get_feature_info перед изменением.
Action:  Если состояние уже Disabled — не делай повторного действия (избежать спама алертов).
         Иначе — вызови set_feature_state с target_state='Disabled'.
         Получи новое состояние через get_feature_info для верификации.
Output:  JSON: {action_taken: string, new_state: object, alert_message: string}.
         alert_message — краткий русский текст для Telegram уведомления.
```

#### 4. Telegram-бот настройка

- Создать `@your_alerts_bot` через @BotFather → получить bot token
- Найти chat_id (личный или группа) через `/start` боту + `https://api.telegram.org/bot{TOKEN}/getUpdates`
- Добавить в n8n credentials Telegram API
- Привязать к Telegram-ноде в WF2

---

## Тест на галлюцинации (~30 мин)

Это **обязательная часть сдачи**. Демонстрирует что вы поняли Algorithm-before-AI принцип.

### Что нужно показать

POST на webhook WF1 с противоречивой командой:
```bash
curl -X POST https://your-n8n.example.com/webhook/feature-control \
  -H "Content-Type: application/json" \
  -d '{"feature_id": "search_v2", "action": "test", "traffic_percentage": -50}'
```

**Ожидаемый ответ** (status 400 или 200 с `success: false`):
```json
{
  "success": false,
  "message": "Процент трафика должен быть в диапазоне 0-100. Получено: -50",
  "rejected_at": "input-validation"
}
```

### Где должна стоять защита

В **двух местах** (defense in depth):

1. **Switch-нода ДО AI Agent** в n8n — отбрасывает невалидные параметры до того как они попадут в LLM. Это и есть Algorithm-before-AI: дешёвый детерминированный guard впереди дорогого LLM-вызова.
2. **JSON Schema в MCP-сервере M3** — `min: 0, max: 100, required: true` на параметре `traffic_percentage`. Это «ремни безопасности» если Switch не отработал по какой-то причине.

> ⚠️ **Защита НЕ должна быть только в system prompt.** Если ваш единственный guard — фраза в промпте «не принимай отрицательные числа» — это **не считается**. LLM может проигнорировать ограничение, особенно при длинном контексте или необычных входных. **Constraint в коде = закон. Constraint в промпте = рекомендация.**

### Что приложить в сдаче

- Скриншот / curl-лог с попыткой отправить `-50` и отказом
- В README отметить **где именно** стоит проверка (Switch-нода или JSON Schema или обе)

---

## Что лежит в `homework/M5/README.md`

```markdown
# M5 Homework — n8n Agentic Workflow

## Архитектура
[опишите 2-3 предложениями: как WF1 + WF2 работают вместе, как они связаны с M3 MCP и M4 Dashboard]

## Стек
- n8n: cloud / self-hosted Docker / n8n-install (укажите выбор)
- Chat Model: Claude / GPT / Gemini / другое
- Storage логов: JSON / Postgres / Redis Stream
- Telegram bot: @your_bot_handle (для проверки можно прислать chat_id личным сообщением Сергею)

## WF1 — Manual trigger
- Webhook URL: [для проверки укажите тестовый URL — мы дёрнем оттуда]
- Что нового в Dashboard: [перечислите кнопки + где живут]

## WF2 — Scheduled monitor
- Threshold: [5% или ваше значение, объясните выбор]
- Logs storage: [logs.json / другое]
- Telegram chat для алертов: [ID или handle]

## Тест на галлюцинации
- Где стоит защита: Switch-нода / JSON Schema / обе
- Лог отказа на `-50%`: [приложите вывод]

## Что было сложно
[2-3 предложения. Что не получилось с первого раза, как разобрались]

## Бонусы (если делали)
- [ ] HITL Wait-нода на критическое действие
- [ ] Langfuse / LangSmith трейсинг
- [ ] Multi-agent supervisor + worker
- [ ] Replay-логика (если фича снова стабильна — re-enable через N минут)
```

---

## Чеклист (бинарно — пройдитесь ПЕРЕД PR)

### WF1 — Manual trigger

- [ ] Feature Dashboard из M4 расширен блоком «Auto-Pilot Controls» с 2–3 кнопками
- [ ] Webhook trigger в n8n принимает POST `/feature-control`
- [ ] Switch-нода валидирует параметры до AI Agent (диапазон, required fields)
- [ ] AI Agent ноде подключены: Chat Model + Memory (Window Buffer length=5) + Tools (MCP Client Tool с подключением к M3 MCP)
- [ ] System prompt написан по GCAO (Goal / Context / Action / Output)
- [ ] Structured Output Parser гарантирует JSON формат
- [ ] Respond to Webhook возвращает JSON с обновлённым состоянием
- [ ] UI Dashboard рендерит обновлённое состояние feature flag по ответу

### WF2 — Scheduled monitor

- [ ] `simulate_traffic.py` скрипт работает, генерит fake-events в `logs.json` (или другой storage)
- [ ] Cron Trigger в n8n запускается раз в минуту
- [ ] Code Node читает логи и считает error_rate
- [ ] IF-нода фильтрует error_rate > threshold (5% или ваше)
- [ ] AI Agent деактивирует фичу через MCP `set_feature_state(Disabled)`
- [ ] Перед изменением проверяется текущее состояние (избегаем повторных действий)
- [ ] Telegram алерт отправляется при срабатывании
- [ ] WF2 не спамит алертами если фича уже Disabled

### Тест на галлюцинации

- [ ] Команда с `traffic_percentage: -50` **отвергается до AI Agent** (Switch-нода)
- [ ] JSON Schema в MCP-сервере M3 имеет `min: 0, max: 100` на `traffic_percentage`
- [ ] В README указано **где именно** стоит защита

### Сдача

- [ ] PR в `proshop_mern` создан
- [ ] Папка `homework/M5/` содержит: `README.md`, 2 JSON workflow, `simulate_traffic.py`, `logs.json` пример, 2 trace screenshots, screencast (или ссылка)
- [ ] Screencast 3–5 мин показывает: команду из Dashboard → агент в n8n → цепочку tool-вызовов → возврат в UI + срабатывание WF2 на спайке ошибок
- [ ] Ссылка на PR отправлена в LMS / чат курса

---

## Бонусы (опционально, не обязательно)

Эти расширения **не влияют на оценку**. Делаете для себя и для портфолио.

- **HITL Wait-нода** на критическое действие (например, перед `set_feature_state='Disabled'` в WF2 — спросить подтверждение через Telegram кнопку Approve / Decline). Production-паттерн 95/5: 95% действий автомат, 5% через human approval.
- **Langfuse / LangSmith трейсинг** — observability для ваших агентов. Покажет реальный flow вызовов tool, токены, latency.
- **Multi-agent supervisor + worker** — supervisor читает sentiment / context из логов, worker делает конкретное действие. На двух нодах AI Agent в n8n.
- **Replay-логика** — если фича снова стабильна (error_rate < 1% в течение N минут после Disabled) — re-enable через MCP. Минимальная adaptive logic.
- **Postgres Chat Memory** вместо Window Buffer — для долгоживущих сессий с множеством событий.

---

## Полезные ресурсы

| Что | Где |
|---|---|
| Sample workflows + simulate_traffic.py | `aidev-course-materials/M5/sample-workflows/` |
| GCAO шаблоны для system prompt | `aidev-course-materials/M5/gcao-prompt-templates.md` |
| Best practices (нода>HTTP>MCP, Algorithm-before-AI, Tool Isolation, Memory) | `aidev-course-materials/M5/best-practices.md` |
| Dashboard расширение (React snippets) | `aidev-course-materials/M5/dashboard-changes.md` |
| 2 CC-агента для разработки workflow в n8n | `aidev-course-materials/M5/agents/` |
| Troubleshooting типовых ошибок | `aidev-course-materials/M5/troubleshooting.md` |
| n8n официальная документация | https://docs.n8n.io |
| AI Agent node документация | https://docs.n8n.io/integrations/builtin/cluster-nodes/root-nodes/n8n-nodes-langchain.agent |
| MCP Client Tool node | https://docs.n8n.io/integrations/builtin/cluster-nodes/sub-nodes/n8n-nodes-langchain.mcpclienttool |
| Anthropic «Building Effective Agents» | https://www.anthropic.com/research/building-effective-agents |
| n8n templates marketplace | https://n8n.io/workflows |
| n8n-install (стек для self-host) | https://github.com/kossakovsky/n8n-install |
| Telegram Bot API | https://core.telegram.org/bots/api |

---

## Что вы вынесете из M5

- Понимание **анатомии агента** в production-коде: GCAO в system prompt + Structured Output на каждом шаге
- Связку **UI ↔ Agent ↔ MCP** на собственном проекте — full-stack замкнутый цикл
- Привычку строить **детерминированные guards** до и после LLM (Algorithm-before-AI) вместо «надежды на здравый смысл модели»
- Опыт двух production-сценариев: manual (синхронный) + scheduled (асинхронный)
- Базу для M6, где мы создадим Агента-Контролёра, который пройдётся по тому же `proshop_mern` с другой ролью — аудитор кода

---

*M5 — HSS AI-dev L1. Дедлайн и способ сдачи — в чате курса.*
