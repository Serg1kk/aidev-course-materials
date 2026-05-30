# Готовый промпт: роутер в n8n (DZ1, шаги 2-3) + ASCII + пример workflow

> Роутер по умолчанию собираем в **n8n** (ты его уже поднимал). Ниже: схема, как собрать ноды (можно
> руками по схеме, можно скормить промпт AI-ассистенту n8n), 3 варианта PII-детекта на выбор и тулы
> агента. Готовый экспорт — в `workflow.placeholder.json` (замени на свой после сборки).
>
> Кто хочет — делает роутер **кодом** на любом фреймворке, схема та же.

---

## ASCII workflow

```
                            ┌───────────────────────────┐
   chat из proshop_mern ───►│  Webhook (POST /chat)     │  вход: { userId, message }  [+ JWT]
                            └─────────────┬─────────────┘
                                          ▼
                            ┌───────────────────────────┐
                            │  PII-детект (1 из 3):     │
                            │   (а) HTTP → Presidio     │
                            │   (б) AI-нода (LLM)       │
                            │   (в) Code-нода (regex)   │
                            └─────────────┬─────────────┘
                                          │ entities[]
                                          ▼
                            ┌───────────────────────────┐
                            │  IF: PII найден?          │
                            └──────┬─────────────┬──────┘
                            PII →  │             │  ← чисто
                                   ▼             ▼
                      ┌────────────────┐   ┌────────────────┐
                      │ AI Agent       │   │ AI Agent       │   тулы агента (scoped!):
                      │ модель=ЛОКАЛКА │   │ модель=ОБЛАКО  │    • getProducts()      (каталог)
                      │ + DB-тулы      │   │ + DB-тулы      │    • getMyOrders()      (только свои)
                      └───────┬────────┘   └───────┬────────┘    • getMyProfile()     (только свой)
                              └───────┬────────────┘
                                      ▼
                            ┌───────────────────────────┐
                            │  MongoDB-нода → chatlogs  │  { createdAt, userId, message, piiEntities,
                            │  (трекинг)                │    route, model, reply, latencyMs, costUsd }
                            └─────────────┬─────────────┘
                                          ▼
                            ┌───────────────────────────┐
                            │  Respond to Webhook       │  → ответ в чат
                            └───────────────────────────┘
```

---

## PII-детект — выбери ОДИН вариант

### (а) Presidio в Docker + HTTP-нода — точнее всего (ловит имена через NER)
```bash
docker run -d -p 5002:3000 mcr.microsoft.com/presidio-analyzer:latest
```
HTTP Request нода: `POST` на `http://presidio-analyzer:3000/analyze` (если n8n и Presidio в одном
docker-compose — по имени сервиса; иначе `http://host.docker.internal:5002/analyze`).
Тело:
```json
{ "text": "{{ $json.body.message }}", "language": "en",
  "entities": ["PERSON","EMAIL_ADDRESS","PHONE_NUMBER","CREDIT_CARD"] }
```
Ответ — массив сущностей (пустой `[]` = чисто). Дальше IF: `{{ $json.length > 0 }}`.

### (б) LLM в AI-ноде — проще, без инфры (недетерминированно)
AI-нода с system-промптом:
```
Ты PII-детектор. Верни ТОЛЬКО JSON-массив найденных типов из
["PERSON","EMAIL_ADDRESS","PHONE_NUMBER","CREDIT_CARD"]. Если ничего нет — []. Без пояснений.
```

### (в) regex в Code-ноде — детерминированно, но имена НЕ ловит
```javascript
const text = $json.body.message;
const has = [];
if (/[\w.%+-]+@[\w.-]+\.[A-Za-z]{2,}/.test(text)) has.push("EMAIL_ADDRESS");
if (/\+?\d[\d\s().-]{7,}\d/.test(text)) has.push("PHONE_NUMBER");
if (/\b(?:\d[ -]?){13,16}\b/.test(text)) has.push("CREDIT_CARD");
return [{ json: { piiEntities: has, hasPii: has.length > 0 } }];
```

---

## Промпт для AI-ассистента n8n (если строишь не руками)

> 💡 **Лучше всего собрать этот роутер двумя субагентами из M5** ([`M5/agents/`](https://github.com/Serg1kk/aidev-course-materials/tree/main/M5/agents)) — они дадут
> заметно более качественный и валидный workflow, чем сборка руками или одним промптом:
> 1. **[`n8n-requirements-orchestrator`](https://github.com/Serg1kk/aidev-course-materials/blob/main/M5/agents/n8n-requirements-orchestrator.md)** — скорми ему задачу ниже, он развернёт её в детальный
>    workflow-spec (ноды, ветки, connections, scoped-тулы, поля chatlogs).
> 2. **[`n8n-workflow-builder`](https://github.com/Serg1kk/aidev-course-materials/blob/main/M5/agents/n8n-workflow-builder.md)** — отдай ему получившийся spec, он соберёт валидный n8n JSON
>    (знает каноны нод, credentials, связи sub-nodes AI Agent + Memory + Tools).
>
> Цепочка: orchestrator → spec → builder → JSON → финальный review и импорт. Как подключить
> агентов (`cp … ~/.claude/agents/`) и полные промпты — см. [`M5/homework-spec.md`](https://github.com/Serg1kk/aidev-course-materials/blob/main/M5/homework-spec.md) (раздел D).
> Промпт ниже годится и как самостоятельный (для любого AI-ассистента n8n), и как вход для
> [`n8n-requirements-orchestrator`](https://github.com/Serg1kk/aidev-course-materials/blob/main/M5/agents/n8n-requirements-orchestrator.md).

```
Собери workflow «приватный AI-роутер для чата интернет-магазина»:
1. Webhook (POST, путь /chat), принимает JSON { userId, message }.
2. PII-детект [ВЫБЕРИ: HTTP к Presidio на http://presidio-analyzer:3000/analyze | AI-нода-классификатор | Code-нода с regex],
   результат — массив типов PII (пустой = чисто).
3. IF: если PII найден → ветка LOCAL, иначе → ветка CLOUD.
4. В обеих ветках — AI Agent нода с инструментами к MongoDB магазина, НО строго scoped:
   getProducts() (весь каталог), getMyOrders(userId) и getMyProfile(userId), где userId берётся
   ТОЛЬКО из входа webhook ({{ $json.body.userId }}), не из текста модели. LOCAL-ветка использует
   локальную модель (OpenAI-совместимый base_url), CLOUD-ветка — облачную (OpenRouter/API).
5. MongoDB-нода: записать в коллекцию chatlogs документ { createdAt, userId, message, piiEntities,
   route, model, reply, latencyMs, costUsd }.
6. Respond to Webhook: вернуть { reply }.
Сделай так, чтобы решение роутера было видно (reason), и латентность/стоимость считались.
```

> ⚠️ Тулы агента **обязательно scoped по userId из webhook**, а не по тому, что просит модель. Это и есть
> базовая детерминированная защита (понадобится в DZ2). См. [`THEORY-injection-defenses.md`](https://github.com/Serg1kk/aidev-course-materials/blob/main/M7/homework/THEORY-injection-defenses.md), слой 3.

После сборки: экспортируй workflow (⋯ → Download) и положи в `homework-m7/router/workflow.json`.
