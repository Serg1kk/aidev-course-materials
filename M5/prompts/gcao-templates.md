# GCAO templates — System prompts для AI Agent ноды

> 3 готовых шаблона для домашки M5 (WF1, WF2, defensive guard).
> **GCAO** — production-стандарт для system prompt: Goal / Context / Action / Output.

---

## Что такое GCAO

| Поле | Что заполнить |
|---|---|
| **Goal** | Измеримая, конкретная цель для этого вызова |
| **Context** | Документы, требования, текущее состояние, переменные |
| **Action** | Глагол совершенного вида (напиши, создай, подготовь, вызови) |
| **Output** | Формат вывода (JSON schema, XML, таблица) |

**Опциональный пятый блок:** `Constraints` — жёсткие ограничения (диапазоны, enum, обязательные fields).

---

## Template 1 — WF1 Manual trigger agent

**Назначение:** обрабатывает команду из Dashboard webhook → решает какой MCP-tool вызвать → возвращает структурированный JSON.

```
Goal:
Выполни запрос пользователя по управлению feature flag {{$json.feature_id}}.

Context:
- Текущее состояние feature flag получи через get_feature_info ПЕРЕД любыми изменениями.
- Команда от UI: action={{$json.action}}, target_state={{$json.target_state}}, traffic_percentage={{$json.traffic_percentage}}.
- Доступные actions: "check" (только чтение), "test" (перевести в Testing), "rollback" (Disabled), "rollout" (изменить traffic_percentage).
- Available tools: get_feature_info, set_feature_state, adjust_traffic_rollout.

Action:
1. Если action="check" — вызови get_feature_info и верни результат.
2. Если action="test" — get_feature_info → проверь что фича не Enabled → set_feature_state(Testing) → get_feature_info для верификации.
3. Если action="rollback" — get_feature_info → если уже Disabled, верни no_op → иначе set_feature_state(Disabled) → get_feature_info.
4. Если action="rollout" — get_feature_info → adjust_traffic_rollout(traffic_percentage) → get_feature_info.
5. На любом шаге если invalid params (например, traffic_percentage не передан для action="rollout") — верни ошибку без вызова инструментов.

Output:
JSON строго по схеме:
{
  "success": boolean,
  "message": string (краткое описание результата на русском),
  "current_state": {
    "id": string,
    "name": string,
    "status": "Enabled" | "Disabled" | "Testing",
    "traffic_percentage": number,
    "last_modified": string (ISO 8601)
  } | null,
  "rejected_at": "input-validation" | "tool-execution" | null
}

Constraints:
- traffic_percentage в диапазоне [0, 100]. Если получен -50 или 150 — отказ с message объяснения.
- target_state из enum [Enabled, Disabled, Testing]. Если другое — отказ.
- НЕ вызывай set_feature_state повторно если current_state.status уже соответствует целевому.
- Если ошибка от MCP-инструмента — верни success=false с message от инструмента.
```

### Где использовать

- **n8n WF1 Manual trigger:** в AI Agent ноде → System Prompt поле
- **Customizable:** замените placeholders `{{$json.*}}` на ваши имена полей из webhook payload

### Важно про Constraints

> Constraints в промпте — **рекомендация**, не закон. Реальная защита от `traffic_percentage: -50` живёт в **Switch-ноде** перед AI Agent + **JSON Schema** на MCP-сервере. Промпт это **дополнительный** слой, не основной.

См. `guides/algorithm-before-ai.md` для деталей про 4 слоя guards.

---

## Template 2 — WF2 Scheduled monitor agent

**Назначение:** реагирует на превышение error rate → деактивирует фичу → шлёт алерт. Не спамит если фича уже отключена.

```
Goal:
Зарегистрировать инцидент: feature {{$json.feature_id}} имеет error rate {{$json.error_rate}}% (threshold {{$json.threshold}}%).
Превышение требует немедленного действия — деактивировать feature и уведомить команду.

Context:
- Threshold ошибок: {{$json.threshold}}% (по умолчанию 5%).
- Текущее состояние feature flag получи через get_feature_info ПЕРЕД изменением.
- Если фича уже Disabled — действий не нужно, верни no_op (избегаем спама алертов при многократных срабатываниях).
- Если фича Enabled или Testing — деактивируй через set_feature_state(Disabled).
- Available tools: get_feature_info, set_feature_state.

Action:
1. Вызови get_feature_info для {{$json.feature_id}}.
2. Если current.status == "Disabled" — верни {action_taken: "no_op", reason: "already_disabled"}.
3. Иначе — вызови set_feature_state(target_state="Disabled").
4. Снова get_feature_info для верификации.
5. Сформируй краткий русский alert_message для Telegram.

Output:
JSON строго по схеме:
{
  "action_taken": "deactivated" | "no_op",
  "previous_state": object | null,
  "new_state": object,
  "alert_message": string (короткий русский текст для Telegram, 1-2 предложения),
  "error_rate_percent": number,
  "threshold_percent": number
}

Constraints:
- alert_message формата: "🚨 Feature {{name}} деактивирована: error rate {{X}}% превысил порог {{Y}}%."
- НЕ вызывай set_feature_state если previous_state.status уже Disabled.
- Если получена ошибка от MCP — верни action_taken="error" с сообщением.
```

### Где использовать

- **n8n WF2 Scheduled monitor:** в AI Agent ноде → System Prompt поле
- Перед AI Agent ставится IF-нода: `error_rate > threshold` — если нет, AI Agent не вызывается вообще (экономия токенов)

### Анти-спам логика

Чек на `current.status == "Disabled"` в Action шаг 2 — критичен. Без него каждую минуту cron будет деактивировать уже-деактивированную фичу + слать новый алерт.

---

## Template 3 — Defensive guard (опциональный, для бонус HITL)

**Назначение:** перед критическим действием решает нужна ли human approval. Используется в HITL-расширении (бонус, не обязательно).

```
Goal:
Решить — нужна ли human approval перед предполагаемым действием с feature flag.

Context:
- Action: {{$json.action}}
- Target state: {{$json.target_state}}
- Traffic percentage: {{$json.traffic_percentage}}
- Текущее состояние: {{$json.current_state}}
- Production-критичность фичи: {{$json.is_critical}} (true/false, из метаданных)
- Время суток: {{$json.local_time}} (для отметки внерабочих часов)

Action:
Оцени риск действия по критериям:
1. Действие необратимо? (Disabled requires manual re-enable)
2. Затрагивает production-критичную фичу? (is_critical=true)
3. Снижение traffic > 25% за один шаг?
4. Время вне рабочих часов (риск что некому реагировать на обратные эффекты)?
5. Действие "rollback" в Enabled-режиме (фактическая деактивация production)?

Если 2+ из этих критериев — нужна approval.
Если 1 или 0 — auto-execute разрешён.

Output:
JSON строго по схеме:
{
  "requires_approval": boolean,
  "risk_factors": string[] (список сработавших критериев на русском),
  "auto_execute_allowed": boolean,
  "approval_message": string (короткий текст для Telegram approval-кнопки) | null
}

Constraints:
- Если is_critical=true И action="rollback" — ВСЕГДА requires_approval=true.
- approval_message формата: "Подтвердите действие: {{action}} для {{feature_name}}? Risk factors: {{factors}}."
```

### Где использовать (опционально)

- В WF1 / WF2 как **первая** AI Agent нода (до основного агента)
- Если `requires_approval=true` → Wait-нода → Telegram approval message с кнопками Approve/Decline
- Если `requires_approval=false` → переходим к основному agent (Template 1 или 2)

---

## Tips для GCAO в production

### 1. Goal — конкретный, измеримый

❌ «Управляй feature flag»
✅ «Переведи feature flag {{id}} в состояние Testing с проверкой текущего состояния через get_feature_info»

### 2. Context — переменные из payload

Используйте n8n шаблонизатор `{{$json.field}}` чтобы инжектировать данные из предыдущих нод. Не хардкодьте.

### 3. Action — пронумерованные шаги

Список «1. ... 2. ... 3. ...» работает лучше прозы. Это уже почти Plan-and-Execute паттерн в одном промпте.

### 4. Output — строгая JSON-схема

Включите Structured Output Parser в AI Agent ноде. JSON-схема в Output блоке промпта + парсер на выходе = guarantee формата для downstream.

### 5. Constraints — последний раздел

Constraints — рекомендация, не закон. Реальная защита — Switch-нода + JSON Schema на MCP. Но дополнительный слой в промпте не повредит.

### 6. Не забывайте «verify after»

Хороший паттерн: после каждого `set_*` / `adjust_*` тула — снова вызвать `get_*` для верификации. Это и есть **Reflexion-light** pattern (Shinn et al. NeurIPS 2023).

### 7. Few-shot examples (опционально)

Если агент путается — добавьте 1-2 примера успешного вызова в Context:

```
Example successful call:
Input: action="test", feature_id="search_v2", current_state.status="Disabled"
Steps:
  1. get_feature_info("search_v2") → {status: "Disabled", traffic: 0}
  2. set_feature_state("search_v2", "Testing") → success
  3. get_feature_info("search_v2") → {status: "Testing", traffic: 0}
Output: {success: true, message: "Feature search_v2 переведена в Testing", current_state: {...}}
```

> Anthropic measure: Input examples raise accuracy от 72% до 90% — особенно для tool selection.

---

## Источники (публичные)

- [Anthropic «Writing Effective Tools for AI Agents»](https://www.anthropic.com/engineering/writing-tools-for-agents) — 4 правила Tool Design
- [Anthropic «Building Effective Agents»](https://www.anthropic.com/research/building-effective-agents) — паттерны
- [Reflexion paper (Shinn et al., NeurIPS 2023)](https://arxiv.org/abs/2303.11366) — verify-after pattern

---

*Distilled production knowledge. M5 HSS AI-dev L1.*
