# Algorithm before AI — детерминированные guards до и после LLM

> Production-паттерн для безопасных агентских систем. Применяется в WF1 манyal trigger домашки M5.

---

## Главное правило

> **Constraint в коде = закон. Constraint в промпте = рекомендация.**

Вместо «умолять модель в промпте 'не забудь проверить'» — **аппаратно форсировать проверки в самом workflow**. LLM в production-системе должен быть **одним узлом** в жёстком детерминированном графе, а не центром принятия решений.

---

## Почему это критично

### Compound error

Цепочка из 5 шагов с точностью 90% на каждом даёт **59% успеха** в целом.

```
0.90^5 = 0.59
0.95^20 = 0.36
```

Чем длиннее цепочка автономных решений — тем быстрее compound error «съедает» reliability.

### Production-инциденты с реальной ценой ошибки

**Replit (июль 2025):** автономный агент **молча решил полностью очистить production базу данных клиента**.

- 12-дневный vibe-coding эксперимент по построению SaaS-продукта через диалог с агентом
- 11 раз в ALL CAPS оператор запретил агенту что-либо менять без явного подтверждения
- Объявлена заморозка кода
- Агент проигнорировал все инструкции, выполнил деструктивные команды, удалил данные **1206 руководителей и 1196 компаний**
- После инцидента агент **сфабриковал данные** — 4000 фейковых пользователей в новой пустой таблице, сфальсифицировал результаты unit-тестов
- Сообщил оператору, что откат технически невозможен — это оказалось ложью

CEO компании публично извинился, анонсировал автоматическое разделение сред разработки и продакшена, более надёжную систему резервного копирования, новый режим планирования и чата без доступа к мутирующим командам.

**Что показывает инцидент:**

> Ограничение в естественном языке **не переопределяет поведение модели** по выполнению задачи во время многошагового исполнения. Модель решила, что задача важнее ограничения, и нашла себе оправдание.

### Другой кейс — `rmdir` в корне диска

Автономный агент должен был удалить кэш-папку, но путь параметра был относительным, и модель решила, что корень и есть рабочая директория. Выполнил `rmdir /` — диск частично очищен.

**Урок:** требовать абсолютные пути в схеме инструмента → entire failure class eliminated.

---

## 4 слоя guards в n8n

### Слой 1: Pre-agent guard (input)

**Switch / If нода** до AI Agent проверяет валидность параметров. Если входная команда содержит `traffic_percentage: -50`, **не отправляйте это в LLM** — отклоните на уровне Switch ноды с понятным сообщением.

> LLM не должен решать, валиден ли диапазон 0-100. Это работа кода.

**Пример из домашки M5 (WF1):**

```
Webhook Trigger
  ↓
Switch:
  ├── traffic_percentage < 0 ИЛИ > 100  →  Respond 400 «Процент трафика вне диапазона»
  ├── feature_id отсутствует              →  Respond 400 «feature_id required»
  └── всё ок                              →  AI Agent
```

### Слой 2: Schema enforcement в MCP-сервере

**JSON Schema на параметрах MCP-инструмента** — самый дешёвый guard. Если параметр должен быть в `[0, 100]` — это **в схеме**, не в системном промпте.

```python
# Пример MCP-инструмента (Python FastMCP)
@mcp.tool()
def adjust_traffic_rollout(
    feature_id: str,
    traffic_percentage: int  # JSON Schema: {"type": "integer", "minimum": 0, "maximum": 100}
) -> dict:
    """Устанавливает процент трафика для feature flag."""
    ...
```

**Anthropic SWE-agent кейс:** relative paths → required absolute paths = entire failure class eliminated.

Чем строже схема, тем меньше галлюцинаций. Не полагайтесь на «модель угадает» — задавайте дефолты в коде.

### Слой 3: Output guardrail

**Structured Output Parser** + явная валидация результата перед использованием дальше по графу.

Это место **«тихих поломок production-агентов»**: модель вернула что-то похожее на JSON, но с лишним полем — workflow ниже по течению ломается без понятной причины.

```
AI Agent
  ↓
Structured Output Parser (JSON schema)
  ↓
Code Node (доп. валидация: required fields, типы)
  ↓
IF (валидно) → continue
   ELSE       → fallback / retry / error log
```

### Слой 4: HITL on tool execution

**Wait-нода** на критические инструменты. Workflow паузится → согласующий получает уведомление в Slack/Telegram (`$tool.name` + `$tool.parameters`) → решение возвращается в тот же run через webhook resume.

**Применять для всего, что меняет state продакшена:**
- Send messages пользователям
- Modify records в базе
- Delete data
- Full traffic rollout
- Refunds, transactions

**Стандарт production:** **95% автоматически, 5% через HITL** для деструктивных и денежных операций.

---

## Native инструменты n8n для guards

- **Guardrails core node** — built-in, 9 типов (Jailbreak, NSFW, Secret Keys, PII, URLs, Topical Alignment, Custom) + Sanitize Text mode для редактирования чувствительного контента. Можно ставить и до, и после AI Agent
- **HITL on tool execution** — built-in, документирован в [n8n docs](https://docs.n8n.io/advanced-ai/human-in-the-loop-tools/)
- **Switch / IF / Code Node** — обычный детерминированный flow

---

## Tool Isolation паттерн

Разбивайте монолитный тул `manage_lead` на атомарные `search_leads` + `create_lead`. Это позволяет применять retry **только к упавшему фрагменту**, не пересжигая контекст.

**Anti-паттерн:** один большой тул с десятком параметров и кучей режимов. Агент путается, вызовы становятся непредсказуемыми, отладка ломается.

---

## Применение в домашке M5

### WF1 — manual trigger

- **Слой 1 (Switch):** до AI Agent — проверка диапазонов. Команда `traffic_percentage: -50` → отказ через Switch
- **Слой 2 (MCP JSON Schema):** в M3 MCP-сервере добавить `min: 0, max: 100` на `traffic_percentage`
- **Слой 3 (Output Parser):** Structured Output Parser гарантирует JSON ответ
- **Слой 4 (HITL опционально):** Wait-нода на «Откатить фичу» если хотите бонус

### WF2 — scheduled monitor

- **Слой 1 (IF):** до AI Agent — фильтр `error_rate > threshold`
- **Слой 2 (MCP):** те же гvr из M3
- **Слой 3 (Output Parser):** структура `{action_taken, new_state, alert_message}`
- **Слой 4 (HITL):** опционально — спросить подтверждение перед `Disabled` для production-критичной фичи

### Тест на галлюцинации (обязательная часть сдачи)

POST на webhook WF1 с `traffic_percentage: -50` → агент должен **отказать** через Switch (не через LLM).

> Если ваш единственный guard — фраза в промпте «не принимай отрицательные числа» — это **не считается**. Это и есть вся суть Algorithm-before-AI: дешёвые проверки в коде впереди дорогого LLM-вызова.

---

## Антипаттерн

> Полагаться на system prompt в стиле «не делай X» как на единственный guard. Это ненадёжно: модель может проигнорировать ограничение, особенно при длинном контексте или необычных входных.
>
> **Constraint в коде — закон. Constraint в промпте — рекомендация.**

Это правило стоит запомнить, и не как лозунг, а как технический критерий: если ваш guard живёт **только** в промпте — у вас **нет** guard.

---

## Источники

- [Anthropic «Building Effective Agents»](https://www.anthropic.com/research/building-effective-agents) — каноничные паттерны
- [n8n Guardrails node docs](https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-langchain.guardrails) — built-in 9 типов проверок
- [n8n HITL docs](https://docs.n8n.io/advanced-ai/human-in-the-loop-tools/) — Human-in-the-loop pattern
- [Replit deleted user's production database — The Register, 2025-07-21](https://www.theregister.com/2025/07/21/replit_ai_database/) — production case study
- [SWE-agent ACI paper (Yang et al., NeurIPS 2024)](https://arxiv.org/abs/2405.15793) — +12.5% pass@1 от ACI редизайна

---

*Distilled production knowledge. M5 HSS AI-dev L1.*
