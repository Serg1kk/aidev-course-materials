# Модуль 4 — Домашнее задание

> **Тема:** Прототипирование инструментов — визуальная оболочка для сквозного проекта.
> **Сложность:** Middle+ (browser-builder путь) / Senior+ (CC + agents путь)
> **Время:** ~4–6 часов обязательного (Часть 1: ~30 мин, Часть 2: ~1.5–3 ч, Часть 3: ~1–2 ч, Часть 4: ~30 мин) + опциональные расширения.
> **Дедлайн:** объявляется отдельно перед стартом M5.
> **Куда сдавать:** PR в ваш GitHub-репо + ссылка в LMS или чат курса.

---

## Цель домашки

Взять `proshop_mern` из M3 и добавить визуальный слой: Dashboard для управления feature flags и редизайн одной страницы. Зафиксировать визуальный язык проекта в `DESIGN.md`.

Инструменты — ваш свободный выбор. Критерий — работающий результат, а не конкретный инструмент. Хотите сравнить несколько — это приветствуется.

После M4 у вас будет:
- Рабочий Dashboard с mock-данными из `features.json`
- Хотя бы одна редизайненная страница proshop_mern
- `DESIGN.md` в репо — единый источник правды о визуальном языке проекта
- База для M5 (Dashboard оживёт через webhook → n8n-агент)

---

## Выбор инструментов

Используйте любой инструмент или комбинацию:

| Категория | Примеры |
|-----------|---------|
| **Browser-builders** | Lovable, Bolt.new, v0.dev |
| **Pre-code design platforms** | Pencil Dev, Google Stitch |
| **Embedded в чате** | Claude Artifacts, Google AI Studio |
| **CC + агенты** | Claude Code + agents из `M4/agents/` + shadcn MCP |
| **Figma-based** | Figma MCP → CC (если есть доступ) |

Можно начать с browser-builder для скетча, а потом перенести в CC для доработки — это стандартный workflow.

---

## Часть 1 — Анализ существующего фронта proshop_mern (~30 мин)

### Зачем

Прежде чем генерировать UI — нужно понять что уже есть. Это не формальность: AI генерирует качественнее, если вы заранее знаете стек и существующие компоненты.

### Что сделать

Изучите фронтенд `proshop_mern` и подготовьте `homework/M4/analysis.md`:

**Страницы (перечислите все, что нашли):**
- Поиск / выдача результатов
- Страница продукта (product details)
- Корзина (cart)
- Оформление заказа (checkout)
- Личный кабинет (account)
- Административная панель (admin)
- ... и все остальные

**Стек фронта:**
- Версия React / роутер (React Router? TanStack?)
- State management (Redux? Context API? Zustand?)
- CSS-подход (CSS Modules? Tailwind? Bootstrap? Styled components?)
- Есть ли уже shadcn/ui, MUI, Chakra или другая компонентная библиотека?

**Компоненты:**
- Какие UI-компоненты уже существуют в проекте?
- Что из них можно переиспользовать при редизайне?
- Что выглядит устаревшим и требует замены?

**Скриншоты:** приложите 2–3 скриншота текущего интерфейса (home, product page, cart — на выбор).

### Что сдать

Файл `homework/M4/analysis.md` — структурированно, 1–2 страницы. Скриншоты рядом в `homework/M4/screenshots/before/`.

---

## Часть 2 — Dashboard для feature flags (~1.5–3 часа)

### Контекст

Это визуальная оболочка для MCP-сервера из M3. Dashboard читает `features.json` и позволяет визуально управлять фича-флагами. Реальное подключение к MCP — это M5. Сейчас: mock-данные из `features.json`, интерактивный UI.

### Функциональные требования

**Список фич — обязательно:**
- Таблица или карточки с фичами из `features.json`
- Для каждой фичи: название, статус-бейдж, процент трафика

**Статус-бейджи — три цвета:**
- `Enabled` — зелёный
- `Testing` — синий
- `Disabled` — серый

**Toggle для `set_feature_state`:**
- Переключатель включения/выключения фичи
- При клике меняет цвет бейджа (state в компоненте, без API-вызова)

**Slider для `adjust_traffic_rollout`:**
- Диапазон 0–100%
- При перетаскивании обновляет отображаемый процент

**Поиск и фильтр:**
- Поиск по имени фичи
- Фильтр по статусу (Enabled / Testing / Disabled / All)

**Edge cases — обязательно:**
- Loading skeleton (пока данные загружаются)
- Empty state (если фичи не найдены по фильтру)
- Error state (если `features.json` не загрузился)

**Accessibility:**
- ARIA labels на интерактивных элементах
- Keyboard navigation (Tab, Enter, Space)

### Дизайн-стиль

Используйте ваш `DESIGN.md` из Части 4 (или создайте его сначала). Если хотите ориентир — чистый минималистичный стиль, тёмный сайдбар + светлая основная область. Без cringe-градиентов, без heavy borders.

Явные запреты при использовании browser-builders (включите в промпт):
```
No gradients, no heavy borders on cards. Clean minimal style.
Font: Geist or Manrope (NOT Inter). Spacing: 8px grid only.
Use CSS variables, not hardcoded colors.
Include ARIA labels and keyboard navigation.
Include loading skeleton, empty state, error state.
```

### Компоненты — обоснование выбора

Перед кодированием решите: готовые компоненты или кастом?

| Сценарий | Рекомендация |
|----------|-------------|
| React-стек, задача внутренняя | shadcn/ui — быстро, accessible из коробки |
| Angular-стек | Angular Material или PrimeNG |
| Vue-стек | Radix Vue или PrimeVue |
| Нет опыта с компонентными библиотеками | Начните с browser-builder, он выберет сам |

Если используете shadcn/ui — агент `shadcn-component-researcher` из `M4/agents/` поможет подобрать нужные компоненты (`Table`, `Badge`, `Slider`, `Switch`, `Input`, `Select`).

Зафиксируйте решение: что взяли готовым и почему, что написали кастомно и почему готовое не подошло.

### Что сдать

- Скриншот работающего Dashboard (обязательно)
- Скриншот файлового дерева (структура компонентов)
- Секция `## Component decisions` в README (10–20 строк)

---

## Часть 3 — Редизайн одной страницы proshop_mern (~1–2 часа)

### Что сделать

Выберите любую одну страницу и отредизайните её, применив `DESIGN.md`:

- Страница поиска / выдача результатов
- Страница продукта (product details)
- Корзина (cart)
- Оформление заказа (checkout)
- Административная панель (любой раздел)

Применить `DESIGN.md` означает: цвета из палитры, типографика по scale, отступы только кратные 8, компоненты по зафиксированным паттернам.

### Подходы к редизайну

**Вариант A: browser-builder.**
Сделайте скриншот существующей страницы → дайте промпт в Lovable/Bolt/v0: "Redesign this page following the attached DESIGN.md. Keep all functionality, improve visual language." Приложите DESIGN.md и скриншот.

**Вариант B: CC + агенты.**
Запустите `ux-designer` из `M4/agents/` для анализа сценария страницы → `shadcn-requirements-analyzer` для маппинга на компоненты → `shadcn-implementation-builder` для генерации кода.

**Вариант C: Figma / Pencil / Stitch → CC.**
Набросайте дизайн в визуальном редакторе → перенесите в CC через MCP.

### Обоснование выбора компонентов

Та же механика, что в Части 2: коротко зафиксируйте что взяли готовым, что написали кастомно, почему.

### Что сдать

- Скриншот "до" (существующая страница)
- Скриншот "после" (редизайн)
- Пара предложений в README: что изменилось и почему именно так

---

## Часть 4 — DESIGN.md в корне репо (~30 мин)

### Почему это обязательно

Без `DESIGN.md` AI-инструменты генерируют default: Inter, фиолетовые градиенты, толстые рамки — узнаваемый "AI-look". С `DESIGN.md` вы получаете конкретный визуальный язык, который агент применяет при каждой генерации.

Это M2-паттерн "Kung Fu Context, тип Writing" — только для дизайна. `CLAUDE.md` определяет поведение агента, `DESIGN.md` определяет визуальный язык.

### Минимальная структура

```markdown
# DESIGN_SYSTEM.md — proshop_mern

## Color palette
- Primary: #...
- Accent: #...
- Surface: #...
- Text primary: #... / Text muted: #...
- Status: Enabled #22c55e / Testing #3b82f6 / Disabled #6b7280
(добавьте semantic roles: bg, fg, primary, muted, accent, destructive, border)

## Typography
- Font: [НЕ Inter — объясните выбор]
- Scale: H1 ..., H2 ..., body 16px, line-height 1.5
- Tracking: ...

## Spacing
- Только кратные 8: 8 / 16 / 24 / 32 / 48 / 64

## Border radius
- Карточки: ...px / Кнопки: ...px / Бейджи: ...px

## Elevation
- Подход к тени / глубине (box-shadow или через контраст фонов?)

## Component patterns
- Cards: padding, border, radius, hover
- Buttons: radius, hover state, no heavy shadows
- Badges: compact, status-color

## Interactive states
- Hover, focus, loading, empty — для каждого интерактивного элемента

## Format
- [shadcn/ui + Tailwind CSS 4 + CSS variables? Angular Material? кастом?]
```

### Как получить DESIGN.md если нет дизайнера

**Вариант A — Reverse-design (рекомендуется):**
Откройте скриншот Stripe, Linear или Vercel. Дайте CC промпт из `prompts/reverse-design.md`. CC проанализирует палитру, типографику, отступы и создаст `DESIGN_SYSTEM.md`.

**Вариант B — TweakCN:**
Если используете shadcn — откройте tweakCN.com, выберите пресет близкий к вашему бренду, подкрутите токены, экспортируйте → вставьте в `DESIGN.md`.

**Вариант C — Вручную:**
Заполните шаблон из `templates/DESIGN.md.template` под свой вкус.

### Требования

- Минимум 7 секций (color / typography / spacing / radius / elevation / components / states)
- Шрифт — не Inter (или явное обоснование почему Inter)
- Spacing scale — только числа кратные 8
- Файл лежит в **корне репо** рядом с `CLAUDE.md`
- В `CLAUDE.md` добавьте строку: `## Design rules: see ./DESIGN.md`

### Что сдать

Файл `DESIGN.md` (или `DESIGN_SYSTEM.md`) в корне репо. Скриншот с деревом файлов — чтобы было видно что он там лежит рядом с `CLAUDE.md`.

---

## Q&A блок (обязательно, минимум 4–5 ответов)

Ответьте письменно на те вопросы, которые были для вас актуальны. Не нужно отвечать на все — выберите те, что зашли. Сдать в `homework/M4/qa-reflection.md` или в README.

1. Какие инструменты пробовали на этой домашке? (можно несколько — это ОК и приветствуется)
2. На каком инструменте остановились и почему?
3. Что понравилось / не понравилось в выбранном инструменте?
4. Если пробовали несколько — чей результат лучше и почему?
5. Что хотели попробовать, но не успели или не смогли?
6. Сколько итераций / промптов потребовалось для Dashboard?
7. Какие баги или pain points встретили? Как решили (или не решили)?
8. Что изменилось в DESIGN.md после первой итерации с агентом?
9. Для QA-профиля: планируете ли писать Playwright тест для Dashboard в M5?

---

## Опциональные расширения (senior+)

### A — UI reviewer subagent

Добавить в репо `.claude/agents/ui-reviewer.md` — sub-agent на Opus, два режима:
- **review mode**: read-only аудит кода против `DESIGN.md` (находит отступления)
- **enforce mode**: рефактор под design system

Запускать после major изменений. Это production-grade паттерн: LLM дрейфуют от правил DESIGN.md после крупных правок — агент возвращает их на место.

### B — Pixel-Perfect verify

Playwright → скриншот рендера Dashboard → vision-модель diff против референса. Если diff выше порога — агент переписывает код. Верификация уровня F5 (Verify фаза pipeline).

Особенно релевантно если вы QA-профиль: это и есть производственный паттерн закрытого цикла.

### C — Selection-mode UI (design system как данные)

Реализовать паттерн из модуля: `DESIGN.md` + `design-system/tokens.json` + `design-system/globals.css`. Редактирование только выбранных секций дизайн-системы без регенерации всего файла (экономия токенов × 10).

Детали: `M4/guides/design-system-versioning.md` (появится).

### D — Скринкаст сравнения

Если пробовали несколько инструментов — короткий GIF или screencast (1–2 мин): Dashboard side-by-side в двух инструментах. Покажите группе на M5 check-in.

---

## Что НЕ требуется

Явный список — чтобы не тратить время на лишнее:

- **Нет Vercel deploy** — всё локально, у вас Docker setup (backend + БД + n8n из M3). Vercel — отдельная тема позже.
- **Нет реального подключения к MCP** из M3 — Dashboard работает на mock-данных из `features.json`. Реальный webhook → n8n-агент → MCP — это M5.
- **Нет переписывания всего proshop_mern** — редизайн одной страницы, не всего проекта.
- **Нет production-grade тестов** — тестирование (Playwright e2e, unit) разбирается в M6. Сейчас — только UI.
- **Нет бэкенд-изменений** — работаем только с фронтом. MCP-сервер из M3 не трогаем.

---

## Формат сдачи

Структура папки в репо:

```
homework/M4/
├── analysis.md              ← Часть 1: анализ существующего фронта
├── qa-reflection.md         ← Q&A блок (или встроить в README)
├── README.md                ← краткое описание что сделали + Component decisions
├── screenshots/
│   ├── before/              ← скриншоты исходного UI (2–3 штуки)
│   ├── dashboard/           ← Dashboard (обязательно)
│   ├── redesign/            ← страница "до" и "после"
│   └── file-tree.png        ← дерево файлов проекта
└── [опционально]
    └── screencast.gif       ← GIF с Dashboard в действии (тоглы, слайдер, поиск)

DESIGN.md                    ← в корне репо (не в homework/)
```

Скриншоты — обязательно не битые ссылки. Если загружаете в GitHub — через Issues (drag-and-drop) или в папку репо.

---

## Критерии приёма (бинарно)

- [ ] `homework/M4/analysis.md` создан и заполнен (стек + страницы + компоненты + скриншоты)
- [ ] Dashboard отображает фичи из `features.json`, toggle и slider работают (UI-уровень)
- [ ] Редизайн хотя бы одной страницы proshop_mern: скриншот "до" + "после"
- [ ] `DESIGN.md` в корне репо, минимум 7 секций, `CLAUDE.md` ссылается на него
- [ ] Q&A блок: минимум 4–5 ответов на актуальные вопросы

---

## Связь с курсом

```
M3 (RAG + MCP)
  └── features.json → Dashboard (M4)
                       DESIGN.md (M4) → сохраняется на M5, M6
                       Dashboard → Webhook → n8n-агент (M5)
                                             Агент-аудитор кода (M6)
```

Чем тщательнее сделаете `DESIGN.md` в M4 — тем меньше итераций потратите в M5 и M6, когда агенты будут генерировать новые компоненты поверх этого же кода.

---

## Полезные ресурсы

| Что | Где |
|-----|-----|
| Агенты UX / shadcn / a11y | `M4/agents/` |
| Шаблон DESIGN.md | `M4/templates/DESIGN.md.template` |
| Промпт reverse-design | `M4/prompts/reverse-design.md` |
| Данные: features.json | `M3/project-data/features.json` |
| Спека фич и валидация | `M3/project-data/feature-flags-spec.md` |
| Anti-AI-slop шпаргалка | `M4/cheatsheets/anti-ai-slop.md` |
| shadcn/ui MCP server | https://github.com/modelcontextprotocol/servers (shadcn/ui) |
| TweakCN (визуальный редактор shadcn-тем) | https://tweakcn.com |

---

*M4 — HSS AI-dev L1. Дедлайн и способ сдачи — в чате курса.*
