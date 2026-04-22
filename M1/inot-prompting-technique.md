# INoT — Introspection of Thought

> **TL;DR:** техника промптинга, в которой внутри **одного** запроса к LLM запускается внутренний дебат между несколькими виртуальными агентами. Они аргументируют, критикуют, ревизируют ответы друг друга и приходят к консенсусу — всё за один API-вызов. Токенов тратится в ~2 раза меньше, чем у обычных multi-agent фреймворков, при росте качества на ~8%.

---

## Источник

- **Paper:** «Introspection of Thought Helps AI Agents» (Haoran Sun, Shaoning Zeng) — [arXiv:2507.08664](https://arxiv.org/abs/2507.08664), 11 июля 2025
- **HTML-версия с рисунками:** [arxiv.org/html/2507.08664v1](https://arxiv.org/html/2507.08664v1)
- **Практический разбор + FSM-вариант:** [INoT Meets FSM — Eugenii Shevchenko, Medium](https://medium.com/@eugenesh4work/introspection-of-thought-inot-meets-fsm-a-practical-single-shot-prompting-pattern-9b4aa81406fa)
- **Разбор концепции:** [Inner Critics, Better Agents — Cognaptus](https://cognaptus.com/blog/2025-07-14-inner-critics-better-agents-the-rise-of-introspective-ai/)

---

## Ключевая идея

Классический multi-agent debate (ChatEval, Society of Minds и др.) делает так:

```
LLM call 1 → Agent A отвечает
LLM call 2 → Agent B критикует
LLM call 3 → Agent A ревизирует
LLM call 4 → Agent B соглашается
...
```

Каждая итерация = отдельный API-запрос. Дорого и медленно.

**INoT переворачивает:** дебат моделируется **внутри одного промпта** через специальный псевдо-код (`PromptCode`), который LLM читает как «программу» и проходит все раунды сам.

```
1 LLM call → [внутри модели: Agent_A debate Agent_B × N раундов → консенсус]
```

Всё происходит в одной «голове». Модель сама проигрывает роль и агента А, и агента B, и проверяющего.

---

## Метрики (из paper)

Авторы прогнали INoT на 6 бенчмарках (GSM8K, HumanEval, SQuAD и др.) на GPT-4o, DeepSeek, Claude, LLaMA 3.2:

- **+7.95%** средний прирост качества reasoning vs baseline
- **−58.3%** экономия токенов vs классический multi-turn debate
- Работает и для multimodal задач (изображения + текст)

---

## Как устроен PromptCode (канонический шаблон)

Из оригинальной статьи (Listing 3):

```python
if image in task:
    import <vision_module>

# Assume agents with independent reasoning
Agent_A, Agent_B = DebateAgent(task), DebateAgent(task)
result_A, thought_A = Agent_A.reason()
result_B, thought_B = Agent_B.reason()

# Set debate parameters
MaxRounds = 10
Counter = 0
agreement = False

while not agreement and Counter < MaxRounds:
    Counter += 1
    # Каждый агент критикует ответ другого
    critique_A = Agent_A.critique(result_B, thought_B)
    critique_B = Agent_B.critique(result_A, thought_A)
    # Ревизия с учётом критики
    result_A, thought_A = Agent_A.rebut_and_adjust(critique_B)
    result_B, thought_B = Agent_B.rebut_and_adjust(critique_A)
    # Проверка консенсуса
    agreement = semantic_match(result_A, result_B)

final_answer = result_A if agreement else arbitrate(result_A, result_B)
```

Модель **не выполняет код** — она **читает его как инструкцию** и симулирует каждый шаг в натуральном языке внутри своего ответа.

> В статье — 2 агента и до 10 раундов. Вариант с 4 персонами и 5 раундами — это адаптация/обобщение (ближе к ChatEval или Council). Паттерн один и тот же.

---

## Готовый к копированию шаблон (compact version)

Вставь это в любой chat (ChatGPT, Claude, Gemini), поменяв `{TASK}`:

```
<PromptCode>
<Config>
  task = "{TASK}"
  max_rounds = 5
  agents = ["Agent_A: логик", "Agent_B: скептик"]
</Config>

<Execute>
# Step 1 — Initial reasoning
Agent_A produces draft_A with step-by-step reasoning.
Agent_B independently produces draft_B.

# Step 2 — Debate loop (до max_rounds или до consensus)
For round in 1..max_rounds:
  Agent_A critiques draft_B (конкретные ошибки, пропущенные шаги).
  Agent_B critiques draft_A (то же самое).
  Agent_A revises → new draft_A.
  Agent_B revises → new draft_B.
  If draft_A ≡ draft_B semantically: break (consensus reached).

# Step 3 — Output
Return final consensus answer + 1-line rationale.
Если consensus не достигнут за max_rounds — вернуть лучший draft + где осталось расхождение.
</Execute>

<Task>
{TASK}
</Task>
```

---

## Расширенный вариант: 4 персоны + 5 раундов

Если хочется больше перспектив — добавь ролей. Пример для продуктового/стратегического решения:

```
<PromptCode>
<Config>
  task = "Стоит ли запускать платный курс по AI для PM?"
  max_rounds = 5
  agents = [
    "Agent_A: оптимист-маркетолог (видит спрос, рост рынка)",
    "Agent_B: скептик-финансист (считает unit-economics, риски)",
    "Agent_C: опытный ученик (смотрит глазами ЦА)",
    "Agent_D: арбитр (взвешивает аргументы, ищет blind spots)"
  ]
</Config>

<Execute>
Round 0 — INITIAL:
  Каждый из A, B, C независимо даёт позицию (5-7 пунктов, с обоснованием).

For round in 1..5:
  DEBATE:
    A критикует B и C. B критикует A и C. C критикует A и B.
    (фокус на конкретных ошибках рассуждения, не на тоне)
  REVISE:
    Каждый обновляет свою позицию с учётом критики.
  CHECK:
    Если 2+ агента сходятся в ключевом выводе → зафиксировать пункт как consensus.
    Если остаются разногласия → следующий раунд.

FINAL — Agent_D (арбитр) пишет:
  1. Consensus points (где все согласны)
  2. Open questions (где остались расхождения + почему)
  3. Рекомендация с условиями (if X then YES, if Y then NO)
</Execute>

<Task>
{TASK}
</Task>
```

---

## Где реально полезно

Проверенные области из paper + практики:

| Задача | Почему помогает |
|--------|------------------|
| Математика, логические задачи | Один агент ошибается → второй ловит ошибку → консенсус корректный |
| Код-ревью, debugging | Agent A предлагает фикс, Agent B ищет edge cases, которые фикс ломает |
| Стратегические решения с trade-off | Оптимист vs скептик не дают модели «влюбиться» в первую идею |
| Факт-чекинг и QA | Самопроверка через дебат снижает галлюцинации |
| Оценка качества текста (LLM-as-judge) | ChatEval показал: разные роли > один и тот же промпт |

---

## Когда НЕ работать будет

- **Простые задачи** — лишний overhead, обычный CoT быстрее и дешевле
- **Слабые модели** (~7B open-source) — часто не держат ролевой игры на протяжении 5+ раундов, сливаются в один голос
- **Нужен детерминированный ответ** — debate внутри модели всё равно стохастичен. Ставь `temperature=0-0.2`
- **Task с чёткой внешней метрикой** — лучше реальный multi-agent через API (Karpathy-loop style), где можно мерить
- **Когда длины контекста не хватает** — 5 раундов × 4 агента = много текста, модель может «забыть» начало

---

## Пример-сравнение (простой → INoT)

**Обычный промпт:**
```
Посчитай: если x + 3 = 2x − 4, чему равен x?
```
Модель отвечает сразу, иногда ошибается в знаках.

**INoT-версия:**
```
<PromptCode>
  Agent_A и Agent_B решают независимо. Затем критикуют шаги друг друга.
  Max 3 раунда или до консенсуса. Verify: подставить x в исходное уравнение.
</PromptCode>

Task: x + 3 = 2x − 4, чему равен x?
```
Теперь даже если Agent_A ошибётся с переносом знака, Agent_B при verify подставит ответ в уравнение и поймает ошибку. Консенсус: x = 7.

---

## Связанные техники (для контекста)

- **CoT (Chain-of-Thought)** — «давай подумаем шаг за шагом», один голос, линейное рассуждение
- **ToT (Tree-of-Thought)** — ветвление гипотез, но обычно multi-turn, дорогой
- **Self-Consistency** — несколько независимых прогонов + голосование, без debate
- **ChatEval** — [arXiv:2308.07201](https://arxiv.org/abs/2308.07201) — внешний multi-agent debate, больше агентов, но дороже
- **The Council** — [github.com/dubs3c/council](https://github.com/dubs3c/council) — готовая CLI-реализация multi-agent debate (Architect, Critic, Pragmatist, Security + moderator)
- **DSPy** — автооптимизация промптов, иногда включает debate-паттерны

---

## TL;DR в 30 секунд

> «INoT — это когда ты в одном промпте описываешь игру двух (или больше) виртуальных агентов внутри модели. Agent_A предлагает ответ, Agent_B критикует, оба ревизируют, повторяется 5 раундов или до консенсуса — всё в одном API-вызове.
> По бенчмаркам из paper: +8% качества, −58% токенов vs обычный multi-agent через несколько API-запросов.
> Фишка: модель проигрывает дебат сама с собой внутри своей "головы". Работает, потому что одна и та же LLM при смене роли ловит слабые места собственного рассуждения.»
