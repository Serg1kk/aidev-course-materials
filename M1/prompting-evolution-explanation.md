# Эволюция промптинга — справка

> Как промптинг менялся с 2022 по 2026: от "47 прилагательных" до скиллов и агентов. Каждый этап — один абзац + ссылки.

---

## Этап 1 (2022–2023): Ручное копирование

Эпоха «скопировал промпт из твиттера → вставил в ChatGPT». Никаких инструментов, никаких фреймворков — только коллективные находки сообщества. Ключевая ментальная модель: *промпт это заклинание, чем длиннее и пафоснее — тем лучше*.

### «47 прилагательных»

Мем/собирательный образ ранних промптов эпохи GPT-3.5. Пользователи верили, что если навалить в начало запроса побольше качественных прилагательных («профессиональный, детальный, креативный, точный, вдумчивый, экспертный, аналитический, инновационный…»), модель станет отвечать лучше. Отсюда виральные шаблоны типа:

> *«Act as a world-class expert with 20 years of experience. You are meticulous, detail-oriented, creative, analytical, innovative, thoughtful, professional, engaging, compelling…»*

Число 47 — условно-мемное (ср. «25 adjectives for ChatGPT», «50 tone adjectives»). Реальных бенчмарков, что это работает, не было — но паттерн был повсеместным. Современные модели этот стиль уже не требуют: достаточно указать роль и контекст один раз.

Источники для контекста:
- [25 Adjectives you can try with ChatGPT — Lauren Selley](https://www.laurenselley.com/blog/25-adjectives-for-chat-gpt) — типичный образец жанра
- [100 Best ChatGPT Prompts — Gridfiti](https://gridfiti.com/best-chatgpt-prompts/) — раздел «Prompt Adjectives»

### awesome-chatgpt-prompts

[github.com/f/awesome-chatgpt-prompts](https://github.com/f/awesome-chatgpt-prompts) — репозиторий от Fatih Kadir Akın, на который «все смотрели» в 2023-м. Десятки ролевых промптов вида «Act as a Linux Terminal», «Act as a Translator», «Act as a Stand-up Comedian». На пике — 100k+ звёзд на GitHub, стал де-факто стандартом для «чего можно попросить у ChatGPT».

Ключевое — показал, что **роль + контекст** работают лучше, чем просто вопрос. Практически весь паттерн `Act as a [role], I will [input], you will [output]` в культуру промптинга пришёл оттуда.

### PromptBase

[promptbase.com](https://promptbase.com/) — первый маркетплейс промптов, запущен в 2022. Люди **продавали промпты** как цифровой товар за $1.99–$9.99. На пике: 270k+ промптов для ChatGPT, Midjourney, DALL-E, Gemini, Veo. 450k+ пользователей.

Что показало: (1) у качественного промпта есть коммерческая ценность, (2) на рынке огромный спрос на готовые решения, (3) большинство пользователей не умели и не хотели учиться писать промпты сами. По сути — ранний прототип «магазина ИИ-приложений» до появления GPT Store.

---

## Этап 2 (2023–2024): Шаблоны и библиотеки

Люди начали систематизировать: не копипаст единичного промпта, а **структура** (Role → Context → Task → Constraints → Output). Появились внутренние корпоративные библиотеки, публичные гайды, фреймворки типа CRISPE, RTF, RISEN.

### Ролевые промпты

Паттерн `You are [role] with [experience]. Your task is [task]...` стал стандартом. Шаблоны вроде CO-STAR (Context, Objective, Style, Tone, Audience, Response) или GCAO (Goal, Context, Audience, Output) — этой же эпохи.

Полезные ссылки:
- [Prompt Engineering Guide (DAIR.AI)](https://www.promptingguide.ai/) — самый цитируемый open-source гайд
- [OpenAI Prompt Engineering docs](https://platform.openai.com/docs/guides/prompt-engineering) — официальная дока

### Библиотеки промптов

Организованные коллекции по темам: маркетинг, код, аналитика, обучение. Примеры:
- [PromptHero](https://prompthero.com/)
- [FlowGPT](https://flowgpt.com/) — соцсеть промптов
- [DocsBot AI Prompt Library](https://docsbot.ai/prompts)
- Внутренние Notion-базы в командах (у многих компаний появились свои)

### Первые «повторители»

Сервисы-обёртки, которые автоматизировали повторяющиеся цепочки промптов: вбиваешь вход → получаешь готовый многошаговый результат. Ранние версии Jasper, Copy.ai, Notion AI — всё это «запакованные шаблоны с UI». Пользователь не видит промпта, но получает детерминированный результат.

---

## Этап 3 (2024–2025): Мета-промпты

Переломный момент: **AI пишет промпт для AI**. Вместо того чтобы человек страдал над формулировками — он описывает задачу простыми словами, а мета-промпт-агент превращает её в продакшн-готовый промпт с ролью, контекстом, примерами, структурой вывода.

### AI пишет промпт для AI (meta-prompting)

Базовая идея: берём задачу → скармливаем её специализированному промпту-архитектору → получаем оптимизированный промпт под конкретную модель. Экономит часы ручной итерации.

Теоретическая основа: [Meta-Prompt Revolution — Prompt Bestie](https://promptbestie.com/en/meta-prompt-template-prompt-engineering-mastery/)

### ELITE PROMPT ENGINEER AGENT

Конкретный виральный шаблон мета-промпта — превращает любую LLM в «архитектора промптов». Протокол: извлечь intent → определить домен → выбрать формат → вложить ограничения → собрать few-shot примеры → выдать модульный шаблон.

Полный текст и разбор: [Elite Prompt Engineer — PromptDocker](https://www.promptdocker.com/community/prompts/952bf01c-407f-4387-859e-2b7d35e2830c)

Открытый набор XML-шаблонов мета-промптинга: [github.com/Liad0205/PromptEngineering](https://github.com/Liad0205/PromptEngineering)

### Anthropic Console Generator

Официальная фича Anthropic — кнопка «Generate a prompt» в [console.anthropic.com](https://console.anthropic.com/). Описываешь задачу одной фразой, Claude сам создаёт production-ready промпт со structured XML-тегами, ролью, chain-of-thought секцией и {{variables}} в handlebars-нотации.

- Анонс 2024-05-20: [Generate better prompts in the developer console](https://anthropic.com/news/prompt-generator)
- Дополнение 2024-10-14: [Improve your prompts in the developer console](https://anthropic.com/news/prompt-improver) — кнопка «Improve» улучшает существующие промпты (CoT, example enrichment, +30% точности в тестах).

---

## Этап 4 (2025–2026): Скиллы

Следующий уровень: промпт перестал быть текстом. Теперь это **пакет**: инструкции + файлы с данными + логика + триггеры. Один «скилл» = маленькая автономная единица, которую агент подтягивает по контексту.

### Промпт + файлы + логика

Скилл содержит:
- `SKILL.md` — инструкции для модели
- вспомогательные ресурсы (JSON/CSV/код/шаблоны)
- правила активации (какие триггеры включают скилл)
- optional: исполняемая логика (Python, bash)

Модель сама решает, какой скилл применить под конкретный запрос — не нужно копировать промпт вручную.

### Claude Code skills

Реализация концепции в Claude Code. Скиллы лежат в `~/.claude/skills/` и активируются автоматически по description-матчингу. Типичные примеры — обработка митингов, ревью кода, генерация отчётов, работа с YouTube / Telegram и пр.

- Документация: [Claude Skills — docs.claude.com](https://docs.claude.com/en/docs/claude-code/skills) (и в Claude Agent SDK)
- Коллекция готовых скиллов: [github.com/Serg1kk/skills](https://github.com/Serg1kk/skills)

### Custom GPTs, Gems

Тот же паттерн в продуктовой обёртке:
- **Custom GPTs** (OpenAI) — магазин персонализированных GPT-ассистентов, [openai.com/gpts](https://chatgpt.com/gpts)
- **Gems** (Google Gemini) — аналогичная фича от Google, [gemini.google/advanced/gems](https://gemini.google.com/gems)

Пользователь без кода создаёт «своего ассистента»: промпт + файлы + инструменты (search, code, DALL-E). Публикует в магазине, другие им пользуются. По сути это skills для массового пользователя.

---

## Этап 5 (2026+): Self-improving агенты

Переход от «человек пишет промпт» к «агент сам переписывает свои промпты, оценивает результат, оставляет лучшие версии». Человек задаёт только метрику — всё остальное цикл делает сам.

### Karpathy Loop (autoresearch)

Март 2026. Андрей Карпатый выложил [github.com/karpathy/autoresearch](https://github.com/karpathy/autoresearch) — 630 строк Python + один файл инструкций `program.md`. Агент (Claude Code/Codex) сам:
1. читает `train.py`
2. формирует гипотезу («что если увеличить глубину attention?»)
3. правит код
4. запускает 5-минутный эксперимент
5. смотрит метрику — коммитит или откатывает
6. **goto 1**

За 2 дня агент прогнал 700 экспериментов и нашёл 20 реальных улучшений nanochat. 71k+ звёзд на GitHub за недели. Fortune назвал это «The Karpathy Loop».

Главное — **паттерн переносим**: та же петля применима к оптимизации промптов, маркетинговых офферов, CUDA-кернелов, пайплайнов preprocessing. Одна правка + фиксированная метрика + быстрый прогон + keep-or-revert.

Разбор: [The Karpathy Loop — GoPenAI](https://blog.gopenai.com/the-karpathy-loop-how-a-630-line-script-is-rewriting-the-rules-of-ai-research-21190138f253) | [NextBigFuture обзор](https://www.nextbigfuture.com/2026/03/andrej-karpathy-on-code-agents-autoresearch-and-the-self-improvement-loopy-era-of-ai.html)

### Binary adoptions

Отбор улучшений по бинарному критерию: метрика выросла → коммит, не выросла → откат. Никаких «чуть-чуть лучше», никакого субъективного суждения модели. Тот самый приём, который делает Karpathy loop воспроизводимым: шум отсекается фиксированным датасетом + повторными прогонами, бинарное решение принимается автоматически.

### Агенты переписывают промпты

Логическое следствие: агент сам правит свой системный промпт, прогоняет на тестовом наборе, сравнивает метрики, оставляет лучшую версию. Цикл работает ночью, утром у тебя промпт, который ты сам бы месяцами доводил вручную.

Связанные концепты для углубления:
- **DSPy** (Stanford) — [dspy.ai](https://dspy.ai/) — фреймворк для автооптимизации промптов через обучаемые «сигнатуры»
- **TextGrad** — [github.com/zou-group/textgrad](https://github.com/zou-group/textgrad) — «градиентный спуск для текста»
- **APE** (Automatic Prompt Engineering) — [arxiv.org/abs/2211.01910](https://arxiv.org/abs/2211.01910) — ранняя академическая работа

---

## Ключевой тезис для устной подачи

> «За 4 года промпт прошёл путь: **строка → структура → пакет → цикл**.
> Сначала человек копировал магию из твиттера. Потом появились шаблоны. Потом AI начал писать промпты за нас. Потом промпт стал скиллом с файлами и логикой. Сейчас агент сам переписывает свои промпты и оставляет лучшие версии.
> Главное — на каждом этапе менялась не «магия слов», а **единица работы**.»
