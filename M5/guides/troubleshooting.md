# Troubleshooting — Типовые ошибки M5 + решения

> Сводка проблем, на которых обычно застревают при сборке n8n + AI Agent + MCP. Для всего что не покрыто — n8n Discord и [docs.n8n.io](https://docs.n8n.io).

---

## n8n setup

### Проблема: n8n не стартует / порт занят

```bash
docker run -p 5678:5678 n8nio/n8n
# Error: bind: address already in use
```

**Решение:** другой процесс на 5678. Проверьте:
```bash
lsof -i :5678
# kill процесс или используйте другой порт:
docker run -p 5679:5678 n8nio/n8n
```

### Проблема: n8n cloud не принимает webhook от фронтенда

CORS блокирует. См. секцию «CORS gotcha» в `dashboard-changes.md`.

### Проблема: env-переменные не подхватываются (self-host)

Docker не видит `.env`:
```bash
docker run -p 5678:5678 --env-file .env n8nio/n8n
```

Или пробросить отдельные:
```bash
docker run -p 5678:5678 -e N8N_BASIC_AUTH_ACTIVE=true -e N8N_BASIC_AUTH_USER=admin n8nio/n8n
```

---

## n8n MCP-сервер из Claude Code

### Проблема: n8n MCP-сервер не отвечает

**Чеклист:**

1. Включён ли MCP server в n8n (Settings → API → Enable Public API)?
2. Сгенерирован ли API token? Bearer token обязателен в продакшен
3. Доступен ли n8n endpoint снаружи (если CC на хосте, n8n в Docker — `http://host.docker.internal:5678`)?
4. Проверьте healthcheck: `curl https://your-n8n.com/healthz`

### Проблема: n8n MCP в queue mode рвёт SSE-соединения

**Симптом:** соединение открывается, через 30 сек обрывается без ошибки.

**Причина:** в queue mode с несколькими репликами вебхуков SSE-соединения требуют, чтобы все запросы по `/mcp*` приходили на одну реплику.

**Решение:** выделить отдельную реплику вебхуков под `/mcp*` и закрепить её на уровне ingress / load balancer.

### Проблема: nginx обрывает MCP-стрим

**Симптом:** SSE работает локально, но через nginx-прокси соединение нестабильное.

**Решение:** в nginx-конфиге для `/mcp*` пути:
```nginx
location /mcp/ {
    proxy_buffering off;
    gzip off;
    proxy_set_header Connection '';
    chunked_transfer_encoding off;
    proxy_pass http://n8n:5678;
}
```

---

## AI Agent нода

### Проблема: AI Agent зацикливается

**Симптом:** workflow исполнение идёт минутами, токены тратятся, никакого финального ответа.

**Решения по приоритету:**

1. **Установите `maxIterations`** в AI Agent ноде (5-15 для большинства случаев). Это **обязательный** guard
2. Проверьте tool descriptions — двусмысленные описания заставляют агента вызывать тулы повторно
3. Проверьте system prompt на конфликтующие инструкции
4. Включите Structured Output Parser — заставляет агента дать финальный ответ в JSON, не «продолжать раздумья»

### Проблема: AI Agent не вызывает tools

**Симптом:** агент отвечает текстом, но MCP-инструменты не дёргает.

**Возможные причины:**

- Tool descriptions слишком общие (например, «инструмент для работы с фичами» — модель не понимает когда вызвать)
- Параметры tool не понятны без context (нет `description` на параметрах)
- Modelka слабая — попробуйте Claude Sonnet 4 / GPT-5 mini вместо Haiku для tool selection
- В system prompt **нет указания** что tools нужно использовать

**Решение:** добавьте в system prompt явное:
```
You MUST use tools to interact with feature flags. Do NOT answer based on assumptions.
For status questions — call get_feature_info first. For state changes — call set_feature_state.
```

### Проблема: Hallucinated tool args

**Симптом:** агент вызывает `set_feature_state(traffic_percentage=-50)` несмотря на то что это невалидно.

**Решение:** Algorithm-before-AI (см. `algorithm-before-ai.md`):
1. Switch-нода до AI Agent — отбрасывает невалидные параметры
2. JSON Schema на MCP-инструменте с `min`, `max`, `enum`, `required`

LLM-промпт «не передавай отрицательные» **не работает** надёжно. Нужны guards в коде.

---

## Webhook trigger

### Проблема: Webhook возвращает 404

**Чеклист:**

1. Активирован ли workflow? В тестовом режиме URL только активен пока «Listen for test event» включён
2. Production URL vs Test URL — в n8n UI разные. Production URL работает только после клика «Activate workflow»
3. Полный path — `https://your-n8n.com/webhook/feature-control` (с `/webhook/` префиксом)

### Проблема: Webhook принимает запрос, но не отвечает

**Симптом:** клиент висит, потом таймаут.

**Решение:** в Webhook Trigger ноде → `Respond` параметр должен быть `Using 'Respond to Webhook' Node`. И в конце workflow обязательно нода **Respond to Webhook** с JSON-ответом.

Без `Respond to Webhook` ноды клиент не получит ответа, даже если workflow отработает.

---

## Telegram-бот для WF2 alerts

### Проблема: Telegram-нода падает с 401

**Решение:**
1. Создать бота через [@BotFather](https://t.me/BotFather) — `/newbot`
2. Скопировать **bot token** (формат `123456:ABC-DEF...`)
3. В n8n credentials → Telegram API → вставить token
4. Test connection — должно сказать success

### Проблема: бот не отправляет сообщения

**Возможная причина:** `chat_id` неправильный.

**Как найти chat_id:**

```bash
# 1. Напишите боту /start (или добавьте в группу)
# 2. Получите updates:
curl "https://api.telegram.org/bot{YOUR_TOKEN}/getUpdates"
```

В ответе найдите:
```json
{
  "message": {
    "from": {"id": 123456789, "first_name": "Sergey"},
    "chat": {"id": 123456789, "type": "private"},
    ...
  }
}
```

`chat.id` — нужное значение.

Для группы — `chat.id` будет отрицательным (например, `-1001234567890`).

---

## Имитация трафика (`simulate_traffic.py`)

### Проблема: WF2 не видит логов

**Чеклист:**

1. Где живёт `logs.json` — в **той же** файловой системе что и n8n?
2. Если n8n в Docker — нужно volume-mount: `-v $(pwd)/logs:/data/logs`
3. n8n Code Node читает по абсолютному пути или относительному к workdir?

**Альтернатива** — Postgres-таблица или Redis Stream вместо JSON файла. Тогда оба процесса (simulate + n8n) обращаются к одному backend.

### Проблема: WF2 спамит алертами

**Симптом:** каждую минуту приходит уведомление «фича деактивирована» хотя она уже Disabled.

**Решение:** в system prompt агента добавить проверку:

```
ВАЖНО: перед изменением состояния обязательно получи текущее через get_feature_info.
Если status уже Disabled — НЕ вызывай set_feature_state повторно.
В этом случае верни {action_taken: "no_op", message: "Already disabled"}.
```

И/или добавьте IF-ноду перед AI Agent:
```
Code Node: считает error_rate
  ↓
HTTP Request: get_feature_info через MCP
  ↓
IF (error_rate > 5% AND status != "Disabled") → AI Agent
   ELSE → log_only
```

---

## MCP-сервер из M3

### Проблема: AI Agent не видит инструменты MCP M3

**Чеклист:**

1. MCP-сервер запущен и доступен? Проверьте напрямую:
   ```bash
   curl https://your-mcp.com/health
   ```
2. В n8n MCP Client Tool ноде — правильный URL? Bearer token?
3. `Tools to Include` параметр — `All` или `Selected` с правильными именами?
4. Проверьте логи MCP-сервера на запросы от n8n

### Проблема: MCP отвечает но tool вызовы failure

**Симптом:** агент видит инструменты, вызывает, но в trace ошибки.

**Возможные причины:**

- JSON Schema на MCP не совпадает с тем что передаёт агент
- Parameter type mismatch (например, `traffic_percentage` ожидается `int` а агент передаёт `string`)
- MCP-сервер не обрабатывает errors грамотно (см. Anthropic «Writing Effective Tools» — ошибки как инструкции, не stacktrace)

**Решение:** в MCP-сервере на 4xx-ошибку возвращайте структурированное сообщение:
```json
{
  "error": "invalid_traffic_percentage",
  "message": "traffic_percentage must be 0-100, got -50",
  "valid_range": {"min": 0, "max": 100}
}
```

Агент это поймёт и попробует с правильными параметрами.

---

## Replicate / OpenAI / Anthropic API

### Проблема: Replicate rate limit

**Симптом:** в WF1 lite-боте генерации изображений — `429 Too Many Requests`.

**Решение:** retry с экспоненциальным backoff в HTTP Request ноде:
- Retry on Fail = ON
- Max Retries = 5
- Wait Between Tries = 2000ms (увеличивается экспоненциально если включён `Use Linear`)

Альтернатива — Redis-queue паттерн (Master-Worker), который размазывает запросы по worker'ам.

### Проблема: OpenAI API падает

**Решение:** Fallback Chat Model в AI Agent ноде:
- Primary: GPT-5 mini
- Fallback: Claude Haiku 4.5 (или Gemini 3.1 Flash)

При 5xx Primary автоматически переключится на Fallback.

---

## Сдача — типовые проблемы

### Проблема: trace screenshot не показывает «рассуждение» агента

**Решение:** в AI Agent ноде включите `Verbose` и `Return Intermediate Steps`. Тогда executions показывают каждый шаг (Thought → Action → Observation).

### Проблема: screencast слишком большой для GitHub

**Решение:** GitHub лимит на файл — 100 MB, на репо — 5 GB.

Альтернативы:
- Loom (бесплатно для 25 видео до 5 мин) — добавьте ссылку в README
- YouTube unlisted — точно не для всех публично
- Telegram-storage — файл в личке, ссылка в README

---

## Если ничего не помогает

1. Зайдите в [n8n Community Forum](https://community.n8n.io) — там 90% типовых вопросов уже разобраны
2. Проверьте [n8n Discord](https://discord.gg/n8n)
3. В чат курса — скриньте n8n executions trace + код, не «всё не работает»
4. Минимальный воспроизводимый пример (workflow JSON + log) — лучшая инвестиция времени

---

*M5 HSS AI-dev L1. Дополнения принимаются — пишите в чат курса.*
