# Домашка M7 — Приватный AI-ассистент с роутером (+ инъекции)

Практический капстоун модуля 7 «Стратегия: безопасность, экономика, внедрение». Ты встраиваешь в свой
`proshop_mern` AI-ассистента, который **маршрутизирует данные по чувствительности** (приватность через
архитектуру) и **логирует всё в админ-дашборд**, а опционально — ломаешь его prompt-инъекцией и защищаешь.

## С чего начать

1. Прочитай [`homework-spec.md`](https://github.com/Serg1kk/aidev-course-materials/blob/main/M7/homework/homework-spec.md) — само задание (Шаг 0 → Часть 0 → DZ1 → DZ2).
2. Выбери способ поднять локальную модель: [`setup-guide.md`](https://github.com/Serg1kk/aidev-course-materials/blob/main/M7/homework/setup-guide.md) + [`hardware-model-mapping.md`](https://github.com/Serg1kk/aidev-course-materials/blob/main/M7/homework/hardware-model-mapping.md).
3. Бери готовые промпты из [`prompts/`](https://github.com/Serg1kk/aidev-course-materials/tree/main/M7/homework/prompts) и скармливай своему кодинг-агенту.

## Файлы

| Файл | Что внутри |
|---|---|
| **[`homework-spec.md`](https://github.com/Serg1kk/aidev-course-materials/blob/main/M7/homework/homework-spec.md)** | Задание: Шаг 0 (карта форка) → Часть 0 (локалка) → **DZ1** (чат + роутер + ассистент с БД + дашборд, обязательно) → **DZ2** (инъекции, опционально). Деливераблы + критерии. |
| **[`setup-guide.md`](https://github.com/Serg1kk/aidev-course-materials/blob/main/M7/homework/setup-guide.md)** | Как поднять локальную модель: (A) локально, (B) облако-сам, (C) endpoint преподавателя. Движки Ollama / LM Studio / vLLM. |
| **[`hardware-model-mapping.md`](https://github.com/Serg1kk/aidev-course-materials/blob/main/M7/homework/hardware-model-mapping.md)** | Какая модель влезет на твоё железо, нужен ли GPU, квант по задаче, цены. |
| **[`prompts/`](https://github.com/Serg1kk/aidev-course-materials/tree/main/M7/homework/prompts)** | Готовые промпты для кодинг-агента: чат-виджет, админ-дашборд, n8n-роутер (+ ASCII + пример workflow), DZ2-инъекция. |
| **[`THEORY-privacy-routing.md`](https://github.com/Serg1kk/aidev-course-materials/blob/main/M7/homework/THEORY-privacy-routing.md)** | 🧠 Out of scope: куда ещё утекают персданные кроме чата и как это решают по-взрослому. |
| **[`THEORY-injection-defenses.md`](https://github.com/Serg1kk/aidev-course-materials/blob/main/M7/homework/THEORY-injection-defenses.md)** | 🧠 Out of scope: полная таксономия защит от prompt-инъекций. |

## Связь с модулем

- **Безопасность:** роутер = «архитектура вместо политики» (PII физически не уходит в облако); защита от
  инъекций = «защищай действия, не ответы» + убери ногу lethal trifecta.
- **Экономика:** приватные запросы на локалке = $0, маршрутизация экономит на облаке; щупаешь руками
  стоимость локалки vs облака.
- **Внедрение:** приватность и безопасность как продуктовые требования.

> Делать на своём форке `proshop_mern`. Бюджет на облако (если выберешь его) — $2-10 хватает с запасом.
> Кодинг-агента для сборки (Claude Code / Cursor / OpenCode и т.п.) — на твой выбор; готовые промпты в `prompts/`.
