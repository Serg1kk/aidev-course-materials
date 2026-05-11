# Claude Code agents для разработки workflow в n8n

> Два специализированных Claude Code агента для построения n8n-workflow из user story. Используются совместно или по отдельности.

---

## Зачем

Ручная сборка workflow в n8n — это:
1. Выяснить требования (триггер, ноды, integrations, edge cases)
2. Превратить требования в spec
3. Кликать в UI или писать JSON руками
4. Подключать credentials, проверять connections
5. Валидировать схему

Если шаг 1-2 делает один CC-агент, а шаг 3-5 — второй, у вас остаётся только финальный review JSON и импорт в n8n.

---

## Два агента

### `n8n-requirements-orchestrator.md`

**Что делает:** превращает абстрактную user story в детальный workflow spec.

**Input:**
- User story: «Хочу чтобы при mention в Slack #ideas записывалось в Google Sheet и приходило уведомление»

**Output (structured spec):**
```yaml
trigger:
  type: slack_event
  channel: "#ideas"
  filter: "message contains @YourBot"

steps:
  - read_message_metadata
  - extract_text_and_author
  - append_to_sheet:
      sheet_id: ENV.IDEAS_SHEET_ID
      columns: [timestamp, author, content, slack_url]
  - send_telegram_dm:
      chat_id: ENV.TG_CHAT_ID
      template: "💡 Новая идея от {author}: {preview}"

edge_cases:
  - empty_message: skip
  - bot_message: skip (avoid loops)
  - sheet_unavailable: log error, retry 3x

credentials_needed: [slack, google_sheets, telegram]
```

**Запрашивает уточнения** если что-то неясно («Какой формат preview? Сколько символов?»).

### `n8n-workflow-builder.md`

**Что делает:** превращает детальный spec в валидный n8n JSON workflow.

**Input:** spec из orchestrator (или вручную написанный)
**Output:** JSON-файл готовый к импорту в n8n

**Знает:**
- Каноны нод n8n (точные имена, типы, версии)
- Валидацию connections
- Систему credentials
- Различия между nodes vs sub-nodes (особенно AI Agent + Memory + Tools)

---

## Как использовать совместно

### Вариант 1: Orchestrator → Builder

```bash
# 1. Запустите orchestrator с user story
claude --skill n8n-requirements-orchestrator < story.md > spec.yaml

# 2. Передайте spec в builder
claude --skill n8n-workflow-builder < spec.yaml > workflow.json

# 3. Импортируйте JSON в n8n (Import from File)
# 4. Настройте credentials под себя
```

### Вариант 2: Только Orchestrator (для сложных задач)

Если задача требует много уточнений — используйте только orchestrator в интерактивном режиме:
```bash
claude --skill n8n-requirements-orchestrator
> Расскажи задачу, и я задам уточняющие вопросы.
> ...
```

В конце получите spec.yaml который можно скопировать в n8n руками или передать builder'у.

### Вариант 3: Только Builder (если spec уже есть)

Если у вас уже есть детальный spec (из документации, ADR, ticket в Jira):
```bash
claude --skill n8n-workflow-builder < existing-spec.yaml > workflow.json
```

---

## Установка

```bash
# 1. Скопируйте файлы агентов в локальную CC skills папку
mkdir -p ~/.claude/skills/n8n-workflow-builder
mkdir -p ~/.claude/skills/n8n-requirements-orchestrator

cp n8n-workflow-builder.md ~/.claude/skills/n8n-workflow-builder/SKILL.md
cp n8n-requirements-orchestrator.md ~/.claude/skills/n8n-requirements-orchestrator/SKILL.md

# 2. Перезапустите CC. Skills должны автоматически появиться.

# 3. Проверьте что skills загружены:
claude --list-skills | grep n8n
```

---

## Применение в домашке M5

### WF1 — Manual trigger workflow

Spec для orchestrator:
> «Студент в Dashboard нажимает кнопку → POST /webhook/feature-control с {feature_id, action, target_state, traffic_percentage} → AI Agent через MCP M3 (3 tools: get_feature_info, set_feature_state, adjust_traffic_rollout) меняет state → возврат JSON в UI».

Orchestrator уточнит:
- Какие именно actions поддерживаются?
- Какой формат ответа?
- Что делать при invalid params (`-50%`)?
- Какая Memory? (Window Buffer length=5 — рекомендация)
- Нужен ли HITL? (опционально для бонуса)

Builder сгенерирует JSON workflow.

### WF2 — Scheduled monitor

Spec для orchestrator:
> «Cron каждую минуту → читает logs.json за последнюю минуту → считает error_rate → если >5% → AI Agent через MCP M3 деактивирует фичу + Telegram alert».

Orchestrator уточнит:
- Где живёт logs.json? (storage path)
- Threshold ровно 5%?
- Telegram chat_id где брать?
- Как избежать спама алертов? (check current state перед disable)

Builder сгенерирует JSON.

---

## Антипаттерн использования агентов

❌ **Не используйте оба агента подряд без review.** Builder может галлюцинировать имена нод (`AI Agent` vs `LangChain Agent` vs `Tools Agent`) — после генерации обязательно проверьте JSON в n8n UI до того как залить в production.

❌ **Не доверяйте orchestrator на edge cases.** Он хорошо собирает happy path, но edge cases требуют вашего внимания. Прочитайте spec перед передачей builder'у.

❌ **Не хардкодьте credentials.** В сгенерированном JSON credentials должны быть placeholders (`{{credentials.MCP_SERVER_TOKEN}}`), а не реальные значения. Builder это знает, но проверьте.

---

## Что в файлах агентов

> Файлы `n8n-workflow-builder.md` и `n8n-requirements-orchestrator.md` будут добавлены в эту папку перед стартом домашки. Они содержат YAML-frontmatter (description, version) + полный system prompt + примеры использования + ограничения.
>
> Если CC у вас уже работает с другими skills — формат знакомый.

---

*M5 HSS AI-dev L1. Skills built and tested by course author.*
