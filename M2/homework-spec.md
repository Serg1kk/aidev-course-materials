# Модуль 2 — Домашнее задание

**Тема:** AI-IDE как полноценный напарник. Ты приходишь в незнакомый проект и за вечер делаешь его своим — rules-файл, индексация, документация, мелкие фиксы.

**Дедлайн:** понедельник **2026-04-27, 23:59**.
**Формат сдачи:** ссылка на публичный форк в чат курса + отчёт `report.md` внутри форка.

---

## Легенда

Представь: ты зашёл в проект от уволившегося стартапера. Код "на коленке", документации нет, стилей по-разному, типов нет, половина вещей захардкожена. **Задача — за вечер привести в порядок, почти не печатая код руками. Только rules, промпты, команды IDE.**

---

## Репозиторий

### Вариант А — Дефолтный (рекомендован)

**https://github.com/bradtraversy/proshop_mern** — MERN e-commerce магазин.
Стек: Node.js + Express (Mongoose 5), React 17 + Redux, Bootstrap.
Автор сам пометил "THIS PROJECT IS DEPRECATED" — то есть мы работаем с намеренно "грязной" версией.

**Запуск (15 мин):**
1. Форкнуть репо себе на GitHub.
2. `git clone` своего форка.
3. Создать `.env` (переменные в README репо).
4. **MongoDB — любой вариант на ваш вкус:**
   - **Docker** (быстро): `docker run -d -p 27017:27017 --name mongo mongo:7`
   - **Локальная установка:** [MongoDB Community](https://www.mongodb.com/try/download/community)
   - **Atlas free tier:** [cloud.mongodb.com](https://cloud.mongodb.com)
5. PayPal — [developer.paypal.com](https://developer.paypal.com) sandbox (бесплатно, 2 мин).
6. `npm install && cd frontend && npm install && cd .. && npm run dev`.

### Вариант Б — Свой проект

Можно взять свой pet-project или open-source fork, если он проходит **7 из 8 критериев**:

1. **Средний размер** — 30-500 файлов прикладного кода (без node_modules).
2. **Frontend и Backend** — или fullstack (Next.js / Nuxt / Remix).
3. **JS/TS или Python** — другие языки, если уверенно владеете.
4. **Запускается локально за 30 мин**.
5. **Есть куда улучшать** — нет rules-файла, слабый README, есть TODO в коде.
6. **3-5 мест под feature-flags** на будущее (можно не размечать сейчас).
7. **Нет NDA / секретов**.
8. **(Желательно)** есть DB с состоянием.

Когда определитесь — киньте в чат курса ссылку на свой репо или напишите «беру дефолтный». Желательно до практик (чт/пт), чтобы было время помочь на старте, если появятся вопросы.

Отполированный проект с готовыми rules-файлами тоже ОК — в этом случае делаете **ревью-апдейт под best practices**, в отчёте описываете что обновили и почему.

---

## Что делаем — блоками

**Система оценки:**
- 🟢 **MUST** (обязательно, все сдают) — без этого не засчитывается.
- 🟡 **NICE-TO-HAVE** (по желанию, для сертификата "с отличием") — минимум один из них.
- 🟣 **EXTRA** — если совсем интересно. Не влияет на оценку, но сильно помогает на M3-M7.

---

## 🟢 БЛОК 1 (MUST). Rules-файл под твою IDE

**Что сделать:** создать и отпилить rules-файл под IDE, в которой работаешь. Файл кладётся в корень форка, коммитится в git.

**Какой именно файл — зависит от IDE:**

| Primary IDE | Файл |
|---|---|
| Claude Code | `CLAUDE.md` |
| Cursor | `.cursor/rules/*.mdc` (legacy `.cursorrules` тоже ок) |
| Codex CLI / Codex App / OpenCode / OpenClaw | `AGENTS.md` (опционально, см. ниже) |
| VS Code + Copilot | `.github/copilot-instructions.md` |
| JetBrains + Junie | `.junie/AGENTS.md` или `AGENTS.md` в корне |
| Gemini CLI | `GEMINI.md` |
| Antigravity (Google) | `GEMINI.md` и/или `AGENTS.md` в корне (с v1.20.3 читает оба) |
| Warp | `AGENTS.md` (заглавными — обязательно) |
| Windsurf | `.windsurf/rules/*.md` (Wave 8+, legacy `.windsurfrules` тоже ок) |

**Не уверен какой у тебя формат?**
- На **Miro-доске Модуля 2** есть фрейм "Rules-файлы — сравнение" (открывали на теории).
- Подробный гайд по всем 10 IDE + 10 готовых templates: **https://github.com/Serg1kk/aidev-course-materials/blob/main/M2/rules-files-per-ide.md**

### Как делать

**Шаг 1 — автоген.**

> ⚠️ **Важно для сдачи домашки:** rules-файл (`AGENTS.md` / `CLAUDE.md` / `GEMINI.md` / `.cursor/rules/*.mdc` — что подходит для твоей IDE) **должен лежать в репо**. Даже если сам инструмент технически работает без него (например, Codex App или Antigravity) — для оценки M2 файл обязателен. Это ядро домашки.

В разных IDE автоген запускается по-разному. Главный split — **CLI** (есть slash-команда `/init`) vs **App / cloud-агент** (slash-команды нет, нужно попросить агента текстом):

#### CLI-инструменты (есть встроенный `/init`)
- **Claude Code (CLI):** `/init` в корне проекта → `CLAUDE.md`.
- **Codex CLI:** `/init` в корне → `AGENTS.md`. Для домашки — обязательно запусти.
- **Gemini CLI:** `/init` в корне → `GEMINI.md` (команда добавлена в июле 2025).
- **Warp:** `/init` в корне → индексирует кодбазу + создаёт `AGENTS.md` (заглавные обязательны). При первом открытии git-репо Warp сам предложит запустить init flow.
- **Cursor:** `/create-rule` в Agent chat (старая `/rules` уже не работает) → `.cursor/rules/*.mdc` с frontmatter. Не классический `/init`, но рабочий аналог.

#### App / cloud-агенты (slash-команды `/init` нет — нужно попросить текстом)

В этих IDE нет встроенной `/init`. Открываешь проект, отправляешь агенту текстовый запрос — он читает репо и создаёт файл. Для сдачи домашки этот шаг **обязателен**:

- **Codex App / ChatGPT cloud Codex:** *«Посмотри проект и создай AGENTS.md в корне со следующими секциями: Overview, Tech Stack, Architecture, Commands, Conventions, What NOT to do»* → Codex сам прочитает репо и создаст файл.
- **Antigravity (Google):** аналогично — *«Посмотри проект и создай GEMINI.md в корне с секциями…»* (или `AGENTS.md` — Antigravity с v1.20.3 читает оба, `GEMINI.md` приоритетнее при конфликте).
- **JetBrains Junie:** кнопка "Create project guidelines" в IDE → `.junie/AGENTS.md`. Есть каталог готовых шаблонов по стекам: github.com/JetBrains/junie-guidelines
- **Copilot:** спроси агента *"Generate copilot-instructions.md for this repo"* → `.github/copilot-instructions.md`. При первом PR агент сам предложит.
- **Windsurf:** нет встроенного `/init`, используй промпт ниже.

Если у твоей IDE нет встроенного автогена ИЛИ ты в App-режиме без slash-команд — дай агенту такой промпт:

```
Проанализируй весь репо. Создай файл [AGENTS.md / CLAUDE.md / .cursor/rules/main.mdc] со следующими секциями:
1. Overview (1 абзац: что делает проект и зачем)
2. Tech Stack (языки, фреймворки, версии)
3. Architecture (структура папок + entry points с реальными путями)
4. Commands (build / test / lint / dev)
5. Conventions (наблюдаемые паттерны naming, imports, error handling)
6. What NOT to do (anti-patterns, которые вижу в коде)

Формат: markdown. ≤200 строк. Английский (русский только для прямых цитат).
```

**Шаг 2 — ручная доработка.** Auto-output — это **30% работы**. Дальше прочитай и **добавь минимум 3 unwritten rules**, которые AI не мог инферить:
- Командные конвенции (как делается code review, как именуются PR).
- Локальные gotchas (например: "после добавления новой модели запустить seed-скрипт вручную").
- Deployment quirks.
- Твои личные предпочтения по стилю работы с AI.

**Шаг 3 — ограничение.** Держи файл **≤150-200 строк** (Anthropic-рекомендация, больше → AI начинает игнорировать части).

**Шаг 4 — несколько IDE?** Нормально. Можно иметь несколько rules-файлов — я буду смотреть все, что есть в репо. Два важных момента:

- **В `report.md` явно укажи**, с какой IDE ты работал как с **основной** (её rules-файл ревьюю в первую очередь). Если работал с несколькими — перечисли и пометь какая primary.
- **Недоделанные rules-файлы либо удали, либо явно пометь** как черновик в `report.md` (например «`.windsurfrules` — просто auto-генерация, не дорабатывал»). Так я не буду считать их за финальную работу.
- **Симлинки — только если реально работаешь с несколькими IDE и хочешь один источник правды.** Паттерн из production-проектов (Next.js, Payload):
  ```bash
  ln -s AGENTS.md CLAUDE.md
  ln -s AGENTS.md .cursorrules
  ```
  Симлинки не обязательны — это одна из рабочих стратегий, не требование.

### Что сдаёшь

1. **Rules-файл(ы)** закоммичены в форк — в том месте, где их ожидает твоя IDE. `CLAUDE.md`, `AGENTS.md`, `GEMINI.md` — в корне. `.cursor/rules/*.mdc`, `.junie/AGENTS.md`, `.github/copilot-instructions.md` — в соответствующих папках. **Главное — файл должен быть в репо, а не в `.gitignore`.**
2. В `report.md` — секция **"Rules diff"**: 3-5 bullets о том, что ты добавил руками поверх auto-generated, и **зачем**. Плюс одной строкой — какая IDE primary.

---

## 🟢 БЛОК 2 (MUST). Запуск проекта локально + обновлённый README

**Что сделать:** сначала **сам запусти проект локально** (на своей машине). Как только убедился, что всё поднимается и открывается в браузере — документируй процесс в обновлённом README. Студент после тебя должен поднять проект по этому README за ≤30 минут.

### Шаг 1 — Запустить у себя

**Это обязательная часть** — без реально работающего у тебя локально проекта дальнейшие блоки (findings, рефакторинг) делать вслепую. Если в процессе запуска вылезут нюансы — разбирайся (через AI тоже можно, это нормальная часть онбординга):

- Зависимости (Node version, npm/yarn/pnpm) — поставить то, что нужно.
- База данных — поднять любым способом (`docker run -d -p 27017:27017 --name mongo mongo:7` / локальная установка MongoDB / Atlas free tier).
- `.env` — сформировать (в dev-репе он обычно не описан полностью, придётся реверсить из кода).
- Внешние сервисы — PayPal sandbox и т.п. — зарегаться где надо.
- Если ошибки на запуске — фиксить (AI-IDE отлично справляется с "Debug with AI" / "Fix with AI" в терминале).

Проверь, что `proshop_mern` реально открывается у тебя в `http://localhost:3000` (или у своего проекта — соответствующий URL), и фронт ходит в бэк.

### Шаг 2 — Документировать запуск в README

Только после того как у тебя проект запустился — обнови `README.md` в форке, чтобы **следующий человек повторил твой путь** без твоей помощи.

Промпт AI для обновления:

```
Я запустил этот проект локально и привожу его в порядок. Помоги обновить README.md так, чтобы следующий человек поднял всё за 30 минут по инструкции.

Текущий README — неполный. Нужно:
1. Краткое описание проекта (1 абзац — что делает, для кого).
2. Tech stack (явно, с версиями из package.json).
3. Структура папок (дерево верхнего уровня + что в каждой).
4. Установка и запуск — пошагово:
   - prerequisites (Node version, MongoDB — любой способ)
   - env vars (все, что реально читаются в коде — сверь с кодом)
   - команды установки зависимостей (обоих частей — root и frontend)
   - команды запуска dev-режима
5. Troubleshooting — типовые проблемы, с которыми я сам столкнулся при запуске (PayPal sandbox, MongoDB connection, CORS и т.д.).

Формат: markdown. Не пиши "a modern/robust/scalable", говори конкретно.
Сравни с реальным кодом, не фантазируй.
```

Проверь результат. Пересобери те блоки, где AI нафантазировал, добавь реальные подводные камни, с которыми столкнулся сам.

### Что сдаёшь

- **Проект реально запускается** у тебя локально (в `report.md` одной строкой подтверди: «запустил локально через docker mongo + npm run dev»).
- Обновлённый `README.md` в форке — можно по нему пройти с нуля и поднять.
- `.env.example` в корне с реальными переменными (без секретов — `MONGO_URI=mongodb://localhost:27017/proshop`, `PAYPAL_CLIENT_ID=your_paypal_sandbox_id`). В `.gitignore` — сам `.env` (не пример).

---

## 🟢 БЛОК 3 (MUST). FINDINGS — что не так с проектом

**Что сделать:** найти через AI минимум **3 реальные проблемы** в проекте и зафиксировать. **Одну из них** — исправить.

**Зачем это надо:** это главное упражнение M2 — ты учишься видеть проект глазами внимательного ревьюера. На M6 (агент-аудитор) будет ровно это, но на масштабе.

### Как делать

**Шаг 1 — запустить structured bug-hunt.** Промпт:

```
Действуй как senior quality reviewer этого репо. Найди:

1. HOTSPOTS — функции >50 строк, вложенные callbacks, высокая сложность.
2. EDGE CASES — места где нет проверки на empty / null / invalid input.
3. OUTDATED DEPS — зависимости далеко позади (Mongoose 5 → 8, React 17 → 19, …).
4. HARDCODED VALUES — магические числа, URLы, секреты в коде.
5. DEAD CODE — закомментированные блоки, неиспользуемые экспорты.

Для каждой находки:
- Где (file:function — без line numbers, их ты галлюцинируешь)
- Что не так (1-2 предложения)
- Уровень: 🔴 critical / 🟡 medium / 🟢 cosmetic
- Как исправить (1 предложение — НЕ исправляй сам сейчас)

Топ-5 сортированных по риску. Markdown-таблица.
```

**Шаг 2 — сохранить в FINDINGS.md.** В корне форка.

Формат:
```markdown
# FINDINGS — proshop_mern

| # | Риск | Где | Что | Как фиксить | Статус |
|---|------|-----|-----|-------------|--------|
| 1 | 🔴 | backend/controllers/orderController.js::calcPrices | при пустых orderItems возвращает NaN | early-return для empty array | ✅ fixed in commit a3f2c1 |
| 2 | 🟡 | frontend/screens/CartScreen.jsx | localStorage без try/catch | обернуть в try/catch | 🔴 not yet |
| 3 | 🟡 | package.json | mongoose 5.x — 4 major позади | upgrade + test | 🔴 not yet |
```

**Шаг 3 — исправить 1 находку через AI.** Выбирай 🔴 или 🟡 (не косметику).

Промпт:
```
Исправь FINDING #1 из FINDINGS.md. Правило:
- Минимальный diff (только то что нужно для фикса)
- Не ломай существующее поведение (если нужно — сначала напиши тест, потом фикс)
- Внятный commit message: "fix(orderController): guard calcPrices against empty items"
- Если не уверен в подходе — остановись и спроси, не додумывай
```

Закоммить, проставь ✅ + ссылку на commit в `FINDINGS.md`.

### Что сдаёшь

- `FINDINGS.md` с ≥3 находками, минимум одна — ✅ исправлена с ссылкой на commit.

Подробный гайд по паттернам (включая промпты для Cursor / Claude Code / Windsurf / Codex) — **https://github.com/Serg1kk/aidev-course-materials/blob/main/M2/indexation-and-onboarding.md**

---

## 🟡 NICE-TO-HAVE (минимум 1 из них для сертификата "с отличием")

Если 3 базовых блока закончил быстро и есть время — берёшь один или несколько из следующих.

### NH-1. Mermaid-диаграмма архитектуры

**Зачем:** GitHub нативно рендерит Mermaid прямо в markdown → документация становится "живой", не просто текст. Видно как всё связано.

**Как:** добавить блок `docs/architecture.md` с Mermaid C4-container-диаграммой. Промпт — в публичном репо (`indexation-and-onboarding.md`, §7).

**Что сдаёшь:** файл `docs/architecture.md`. Проверь что на GitHub он рендерится как **картинка схемы**, а не просто код (открой файл в браузере у себя на github.com).

### NH-2. ADR × 3

**Зачем:** Architecture Decision Records фиксируют implicit решения, которые сейчас просто "как-то так сложилось". Это нужно на M6 (аудитор захочет знать почему что-то сделано именно так).

**Как:** попросить AI найти 3 ключевых implicit-решения в коде и оформить по MADR-шаблону. Каждый с маркером уверенности `HIGH` / `MEDIUM` / `LOW`.

Промпт:
```
Найди 3 implicit архитектурных решения в этом репо — то, что в коде есть, но нигде не задокументировано. Для каждого создай `docs/adr/NNNN-short-title.md` по MADR-шаблону:

- Status: Accepted
- Context: какую проблему решали
- Decision: что именно выбрано
- Alternatives: что ещё рассматривали (инферится из commented-out, removed deps)
- Consequences: что получили (+/-)

Каждый ADR пометь confidence: HIGH (явно в коде) / MEDIUM (инферится) / LOW (спекуляция).
Не выдумывай — только то, что видно в коде.
```

**Что сдаёшь:** `docs/adr/0001-*.md`, `docs/adr/0002-*.md`, `docs/adr/0003-*.md`.

### NH-3. Characterization tests на 1 функцию

**Зачем:** ключевой safety-net паттерн для рефакторинга legacy. На M6 без этого не обойтись, хочешь заранее прочувствовать — делай сейчас.

**Как:** берёшь одну "грязную" функцию (20-50 строк с условными ветвлениями). Примеры в `proshop_mern`: `calcPrices`, `orderItemsReducer`.

1. AI генерит характеризующие тесты, **захватывающие текущее (даже багованное) поведение**. Тесты зелёные на оригинале.
2. AI рефакторит функцию. Все тесты должны оставаться зелёными (или явно задокументировать какие упали и почему — если вы сознательно "пофиксили" баг).

Промпт для генерации тестов:
```
Прочитай эту функцию. Напиши characterization tests, которые захватывают ТЕКУЩЕЕ поведение, включая баги.

НЕ:
- Не чини очевидные баги
- Не добавляй валидацию
- Не улучшай error messages

ДА:
- Покрой все ветки (happy / empty / null / malformed)
- Ассерть на реальные возвращаемые значения (даже если выглядят странно)
- Добавь комментарий к 2-3 тестам: "# This asserts current buggy behavior. Correct would be: X"

Формат: runnable test file (Jest).
```

Потом промпт для рефакторинга:
```
Рефактори эту функцию на современный стиль (async/await, pure functions, consistent error handling).

CONSTRAINTS:
- Все characterization-тесты ДОЛЖНЫ оставаться зелёными
- Не "улучшай" поведение — только форму
- Если думаешь что тест неверный, остановись и спроси
```

**Что сдаёшь:**
- `original.js`, `characterization.test.js`, `refactored.js` в отдельной папке (`experiments/` или `docs/m2-char-tests/`).
- `reflection.md` — 1-2 абзаца "что узнал, какие тесты упали и почему".

---

### NH-4. `/init` в разных IDE — сравнительный разбор

**Зачем:** `/init` в Claude Code — не уникальная команда, у каждой IDE свой аналог. Понять как устроен автоген в другой IDE — это понять, что он делает под капотом, и почему вывод всегда требует ручной доработки.

**Что сделать:** запусти автоген rules в своей основной IDE (или в дополнительной). Посмотри на результат и ответь в `report.md` на три вопроса:

1. **Что автоген нашёл правильно?** (реальный стек, реальные команды, структура папок)
2. **Что он упустил или нафантазировал?** (implicit правила, конвенции, gotchas, которые видны только из опыта)
3. **Что ты добавил руками?** (твоих 3 unwritten rules из Блока 1)

**Справочник: аналоги `/init` в разных IDE**

> ⚠️ **Для сдачи M2:** rules-файл **обязательно лежит в репо**, независимо от того, обязателен ли он самой IDE. Codex и Antigravity технически могут работать без него — но для оценки нужно показать, что ты прошёл через автоген + ручную доработку.

#### CLI-инструменты (есть встроенный `/init`)

| IDE | Как запустить | Файл | Особенности |
|-----|---------------|------|-------------|
| **Claude Code (CLI)** | `/init` в корне | `CLAUDE.md` | Иерархия: глобальный `~/.claude/CLAUDE.md` → проектный → подпапки |
| **Codex CLI** | `/init` в корне | `AGENTS.md` | Иерархия: `~/.codex/AGENTS.md` (глобальный) → repo `AGENTS.md` → подпапки. `AGENTS.override.md` для временного override. Fallback-имена через `project_doc_fallback_filenames` в `~/.codex/config.toml`. Технически не требует AGENTS.md — но для домашки обязательно |
| **Gemini CLI** | `/init` в корне | `GEMINI.md` | Команда добавлена в июле 2025. `/memory show` — посмотреть что загружено. `/memory add` — добавить на лету. Идемпотентен: не перезаписывает существующий `GEMINI.md` |
| **Warp** | `/init` в корне (или авто-flow при первом открытии git-репо) | `AGENTS.md` (CAPS!) | Имя файла — строго заглавными, иначе Warp не найдёт. Legacy `WARP.md` тоже работает. `/init` умеет линковать существующие `CLAUDE.md`, `.cursorrules`, `GEMINI.md`, `.windsurfrules`, `.github/copilot-instructions.md` в `AGENTS.md` |
| **Cursor** | `/create-rule` в Agent chat (или Settings → Rules → + Add Rule) | `.cursor/rules/*.mdc` | Не классический `/init`, но рабочий аналог. Несколько файлов с frontmatter: `alwaysApply`, `globs`, `description`. Legacy `.cursorrules` deprecated |

#### App / cloud-агенты (slash-команды нет — просим агента текстом)

| IDE | Как запустить | Файл | Особенности |
|-----|---------------|------|-------------|
| **Codex App / ChatGPT cloud Codex** | Текстовый запрос агенту: *«Посмотри проект и создай AGENTS.md в корне с секциями: Overview, Tech Stack, Architecture, Commands, Conventions, What NOT to do»* | `AGENTS.md` | Codex сам читает репо и генерит файл. В cloud-режиме окружение задаётся через UI: setup scripts, env vars, package versions. Для домашки — файл должен оказаться в репо |
| **Antigravity (Google)** | Текстовый запрос агенту: *«Посмотри проект и создай GEMINI.md в корне с секциями: …»* (или `AGENTS.md`) | `GEMINI.md` и/или `AGENTS.md` | С v1.20.3 (март 2026) читает **оба** файла. При конфликте `GEMINI.md` побеждает. Глобальные: `~/.gemini/GEMINI.md` или `~/.gemini/AGENTS.md`. Workspace rules: `.agent/rules/*.md`. Nested files в подпапках включаются в Settings → Agent → Load nested AGENTS.md |
| **JetBrains Junie** | Кнопка "Create project guidelines" в IDE | `.junie/AGENTS.md` | Официальный каталог готовых гайдлайнов по стекам: [junie-guidelines](https://github.com/JetBrains/junie-guidelines) |
| **Copilot** | Текстовый запрос: *"Generate copilot-instructions.md for this repo"* | `.github/copilot-instructions.md` | + path-specific `.github/instructions/*.instructions.md` с `applyTo` glob. При первом PR агент сам предложит создать |
| **Windsurf** | Нет встроенного, дай промпт ниже | `.windsurf/rules/*.md` (Wave 8+) | GUI-редактор в IDE. Legacy `.windsurfrules` |

**Ключевой инсайт про AGENTS.md:** большинство IDE (Codex, Copilot, Cursor, Windsurf, Amp, Devin) читают `AGENTS.md` нативно. Claude Code — нет (использует только `CLAUDE.md`). Если работаешь в нескольких IDE — можно держать один `AGENTS.md` как источник правды и симлинковать:
```bash
ln -s AGENTS.md CLAUDE.md        # Claude Code читает CLAUDE.md
ln -s AGENTS.md .cursorrules     # Cursor (legacy)
```

**Промпт для автогена (если нет встроенного):**
```
Проанализируй весь репо. Создай файл [AGENTS.md / CLAUDE.md / .windsurf/rules/main.md] со следующими секциями:
1. Overview (1 абзац: что делает проект)
2. Tech Stack (языки, фреймворки, версии из package.json)
3. Architecture (структура папок + entry points с реальными путями)
4. Commands (build / test / lint / dev)
5. Conventions (наблюдаемые паттерны naming, imports, error handling)
6. What NOT to do (anti-patterns, которые вижу в коде)

Формат: markdown. ≤200 строк. Английский.
```

**Что сдаёшь:** секцию "IDE autogen comparison" в `report.md` — 3-5 bullet points ответы на три вопроса выше.

---

## 🟣 EXTRA (не для оценки, но критично понимать)

- **Docker-compose для проекта** — добавь `docker-compose.yml`, чтобы Mongo + бэк + фронт поднимались одной командой `docker compose up`. Это альтернатива ручному запуску — проект и без compose работает нормально (что было в Блоке 2), но compose делает опыт воспроизводимым и убирает "а у меня Mongo не поставился".
- **Upgrade deps** — в proshop_mern реально старые версии (на 2026-04: **mongoose 5.10**, express 4.17, react 16.13, jsonwebtoken 8.5). Сценарий: через AI мигрировать одну из них на актуальную (mongoose 5 → 8, или react 16 → 19). Промпт: *"upgrade mongoose from 5.x to 8.x in this project, adjust all usage sites to new API (callbacks → promises, strictQuery etc), preserve behavior. Break into small commits so I can review step by step."*
- **Попробуй другую IDE** — основную работу сделал в Cursor, попробуй те же шаги в Claude Code (или наоборот). Сравни в отчёте: что лучше, что хуже, где удобнее.

---

## Как сдавать

**Публичный форк на GitHub** — не локальные файлы, не zip.

### Пошаговый флоу (для дефолтного proshop_mern)

```bash
# 1. Форкнуть в GitHub UI (кнопка Fork → в свой аккаунт). Форк публичный (по умолчанию так и есть).

# 2. Clone локально
git clone https://github.com/ВАШ_USERNAME/proshop_mern.git
cd proshop_mern

# 3. Проверь имя дефолтной ветки — у proshop_mern это 'master' (старый репо, 2020 года), не 'main'
git branch --show-current
# → master

# 4. Baseline — "я ничего ещё не менял"
git commit --allow-empty -m "chore: baseline before M2 AI work"
git push origin master   # <-- master, НЕ main

# ... далее работаешь, коммитишь каждый логический шаг отдельно ...

# В конце
git push origin master
```

**Опционально — переименовать master → main** (если хочется по современным конвенциям):
```bash
git branch -m master main
git push -u origin main
# Потом в GitHub UI: Settings → Branches → сменить Default branch на main → удалить master
```

> **Почему так:** репо `bradtraversy/proshop_mern` создан в 2020, когда GitHub ещё дефолтил `master`. Для новых репо (после ~2021) дефолт — `main`. Команда `git push origin main` выдаст `src refspec main does not match any`, если ты работаешь в `master` — вот и весь секрет.

### Если берёшь свой проект

У тебя уже есть репозиторий → не нужно ничего форкать. Просто создаёшь отдельную ветку под эту домашку в своём же репо и делаешь там первый пустой baseline-коммит:

```bash
# в корне своего проекта
git checkout -b m2-hss-homework          # новая ветка — не важно как называется дефолтная (main/master)
git commit --allow-empty -m "chore: baseline before M2 AI work"
git push origin m2-hss-homework          # пушим ветку с тем же именем
# сдаёшь ссылку на эту ветку в чат курса
```

### Зачем этот baseline-коммит?

Это **пустой** коммит — в нём нет никаких правок кода. Он нужен как «точка отсчёта» до того как ты начнёшь что-то менять через AI. Дальше, когда домашка сделана, ты открываешь на GitHub кнопку **Compare** → выбираешь baseline-коммит слева и финальный коммит справа → видишь в одном экране **полный diff** того, что ты (точнее AI под твоим руководством) сделал за вечер. Это нужно для самопроверки и для меня при ревью — сразу видно весь объём работы, ничего не потеряется в истории.

### Работаешь в любой IDE / любом языке

Нет привязки к конкретному инструменту. Используй что комфортно:
- **Одну IDE** — Cursor / Claude Code / Windsurf / Copilot / Codex / OpenCode / Gemini CLI / Warp / JetBrains AI.
- **Несколько параллельно** — ок, укажи в `report.md`.
- **Главное — осмысленные коммиты.** Каждый шаг = отдельный коммит с вменяемым message. Примеры: `feat: add CLAUDE.md`, `docs: mermaid architecture`, `fix: guard calcPrices`. Плохие: `update`, `wip`, `.`. AI сам умеет писать нормальные — попросите.

### Требования к форку

- Публичный (Settings → General → Change visibility).
- Есть baseline-коммит до первой AI-правки.
- `.env` в `.gitignore` — секреты не коммитим.
- `.env.example` — есть.
- Mermaid (если делал NH-1) — корректно рендерится на GitHub. Суть: GitHub умеет превращать код mermaid внутри markdown-файла в настоящую картинку-схему прямо на странице. Чтобы проверить — открой в браузере свой файл на github.com (например `…/blob/main/docs/architecture.md`). Если видишь нарисованную диаграмму — всё отлично. Если видишь «простыню кода со стрелочками и скобками» — значит не хватает обёртки ` ```mermaid ` / ` ``` ` вокруг блока, и её надо добавить. Это нужно, чтобы документация реально помогала (картинку ревьюер видит сразу, а сырой код никто читать не будет).
- `report.md` в корне.

### Куда сдавать

**Чат курса** — скинь ссылку на свой форк, когда готов. Преподавателю удобно иметь все ссылки в одном месте перед практиками.

---

## Что в итоге лежит в форке

**Обязательно (MUST):**
- [x] Rules-файл под вашу IDE (корень).
- [x] Обновлённый `README.md` + `.env.example`.
- [x] `FINDINGS.md` (≥3 находки, ≥1 исправлена).
- [x] `report.md` (см. ниже).

**По желанию (NICE-TO-HAVE):**
- [ ] `docs/architecture.md` (Mermaid).
- [ ] `docs/adr/*.md` × 3.
- [ ] Characterization tests (в `experiments/` или аналогично).

### Что пишешь в `report.md`

Короткий отчёт (≤200 слов):

```markdown
# M2 — Report

## IDE
- Primary (с ней я работал основной): Cursor → rules в `.cursor/rules/main.mdc`
- Secondary: Claude Code (CLAUDE.md — чёрновик, руки не дошли дотюнить)

## Rules diff (что добавил руками поверх auto-generated)
- Add "always use functional React components" — в репо есть class components, AI сам не инферил стандарт
- Add "mongodb = Atlas sandbox, не production" — контекстное предупреждение
- Add "don't touch Procfile" — legacy от Heroku, но деплой уже не туда

## 3 вопроса
- Сколько заняло бы вручную: ~6-8 часов (только findings + рефакторинг + README)
- Самая магическая функция IDE: `@Codebase` + `/generate docs` — за 40 сек получил mermaid-диаграмму всего бэкенда
- Где AI сломал и как пофиксил: после миграции mongoose сгенерил `await User.findOneAndUpdate(…)` без `{ new: true }` — старое значение возвращалось. Нашёл через characterization tests, добавил флаг, тесты снова зелёные.
```

---

## Полезные ссылки

- 🔗 **Репо с материалами курса:** https://github.com/Serg1kk/aidev-course-materials
  - [Rules-файлы — справочник по 10 IDE](https://github.com/Serg1kk/aidev-course-materials/blob/main/M2/rules-files-per-ide.md)
  - [Индексация и онбординг — best practices + промпты](https://github.com/Serg1kk/aidev-course-materials/blob/main/M2/indexation-and-onboarding.md)
- 🔗 **Miro-доска Модуля 2** — фрейм "Rules-файлы — сравнение" (на теории разбирали).
- **Чат курса** — вопросы, ссылки на свои форки, помощь друг другу.
- **M2-P1 (четверг)** / **M2-P2 (пятница)** — разбираем сдачи, отвечаем на вопросы.

---

## Таймбюджет (прикидка)

| Блок | Время |
|---|---|
| 🟢 1. Rules-файл | 30-45 мин |
| 🟢 2. README + .env.example | 45-60 мин |
| 🟢 3. FINDINGS (3 находки + 1 фикс) | 60-90 мин |
| **🟢 MUST — итого** | **~2-3 часа** |
| 🟡 NH-1 Mermaid | +20 мин |
| 🟡 NH-2 ADR × 3 | +60 мин |
| 🟡 NH-3 Characterization tests | +60-90 мин |
| **🟡 MUST + один NH** | **~3-4 часа** |
