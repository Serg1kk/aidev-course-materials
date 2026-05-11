# Модуль 5 — Агенты и n8n: «Мозг» системы

> **Статус:** материалы выложены ДО занятия. Дедлайн домашки — отдельно перед стартом M6.

## О модуле

В M3 вы создали **руки** системы — MCP-сервер для управления feature flags. В M4 — **глаза**: Feature Dashboard в админке. M5 добавляет **мозг**: агентский workflow в n8n, который сам принимает решения и крутит ручки через MCP.

К концу модуля у вас будет замкнутый full-stack цикл: пользователь нажимает кнопку в Dashboard → n8n-агент через MCP меняет state → Dashboard обновляется. Параллельно — scheduled defensive monitor, который сам деактивирует фичу при всплеске ошибок.

## Что в этой папке

| Файл / папка | Зачем |
|---|---|
| `homework-spec.md` | Полное ТЗ домашки (2 workflow + тест на галлюцинации) |
| `guides/best-practices.md` | Production-правила: нода > HTTP > MCP, Algorithm-before-AI, Tool Isolation, Memory выбор, лимит итераций |
| `guides/algorithm-before-ai.md` | 4 слоя детерминированных guards до и после LLM |
| `guides/memory-types-in-n8n.md` | Window Buffer / Postgres Chat / Conversation Buffer / Vector Store — что когда |
| `guides/dashboard-changes.md` | Как расширить Feature Dashboard из M4 (sample React snippets, env переменные) |
| `guides/troubleshooting.md` | Типовые ошибки + решения (n8n MCP, webhook 404, Replicate rate limits, Telegram setup) |
| `prompts/gcao-templates.md` | 3 готовых GCAO-шаблона system prompt для AI Agent ноды |
| `agents/README.md` | Как использовать 2 CC-агента совместно (orchestrator → spec → workflow-builder → JSON) |
| `agents/n8n-requirements-orchestrator.md` | CC-агент: user story → детальный workflow spec |
| `agents/n8n-workflow-builder.md` | CC-агент: спек → валидный n8n JSON |
| `sample-workflows/README.md` | Как импортировать стартовые JSON в n8n + где placeholders для credentials |
| `sample-workflows/wf1-manual-trigger-template.json` | Стартовый шаблон для WF1 (manual webhook) |
| `sample-workflows/wf2-scheduled-monitor-template.json` | Стартовый шаблон для WF2 (scheduled cron) |
| `sample-workflows/simulate_traffic.py` | Эталонный скрипт имитации трафика для WF2 |

## Pre-read до занятия

В LMS Knowledge Base — 4 темы:
- **5.1 Анатомия агента** — ReAct, Plan-and-Execute, Reflexion, write-manage-read цикл памяти
- **5.2 Low-code оркестрация в n8n** — AI Agent node, минимальная продакшн-связка
- **5.3 Управление состоянием** — типы памяти, чекпоинтеры, time-travel
- **5.4 Связка UI + Агент + MCP** — Algorithm-before-AI, Tool Isolation, eval-цикл

На живом занятии Сергей сверху даёт продуктовую рамку: GCAO как production-промпт-каркас, виды агентов (8+1 типов), ландшафт фреймворков 2026, история n8n, CC + n8n как новый стандарт разработки workflow.

## Связь с другими модулями

```
M3 (MCP — руки)           ──┐
                              ├──▶  M5 (Agent — мозг)  ──▶  Замкнутый цикл
M4 (Dashboard — глаза)    ──┘                                full-stack
                                                              │
                                                              ▼
                                                              M6 (Auditor — контролёр)
```

В M6 мы создадим Агента-Контролёра, который пройдётся по тому же `proshop_mern` с другой ролью — аудитор кода. Тот же agentic паттерн, другая задача.

## Что вы вынесете

- Понимание **анатомии агента** в production-коде: GCAO в system prompt + Structured Output на каждом шаге
- Связку **UI ↔ Agent ↔ MCP** на собственном проекте — full-stack замкнутый цикл
- Привычку строить **детерминированные guards** до и после LLM (Algorithm-before-AI) вместо «надежды на здравый смысл модели»
- Опыт двух production-сценариев: manual (синхронный) + scheduled (асинхронный)
- Базу для M6 (Агент-Контролёр)

---

*M5 — HSS AI-dev L1. Дедлайн домашки и способ сдачи — в чате курса.*
