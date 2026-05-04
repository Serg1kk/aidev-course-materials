# Pre-flight checklist M4

> Пройдитесь по этому списку ПЕРЕД тем как открывать PR. Каждый пункт — бинарный: сделано или нет.

---

## Часть 1: Анализ существующего UI

- [ ] `homework/M4/analysis.md` создан
- [ ] Перечислены страницы (search / product / cart / checkout / account / admin)
- [ ] Указан стек фронта (React версия, роутер, state management)
- [ ] Перечислены компоненты: что уже есть, что можно переиспользовать
- [ ] Скриншоты текущей UI добавлены в `homework/M4/screenshots/before/` (2–3 штуки)

---

## Часть 2: Dashboard для feature flags

- [ ] Список фич из `features.json` отображается (карточки или таблица)
- [ ] Статус-бейджи трёх цветов: Enabled зелёный / Testing синий / Disabled серый
- [ ] Toggle для `set_feature_state` — меняет цвет бейджа при клике
- [ ] Slider 0–100 для `adjust_traffic_rollout` — обновляет отображаемый процент
- [ ] Поиск по имени фичи работает
- [ ] Фильтр по статусу работает
- [ ] Loading skeleton присутствует
- [ ] Empty state присутствует (пустой результат фильтра)
- [ ] Error state присутствует (если данные не загрузились)
- [ ] ARIA labels на интерактивных элементах
- [ ] Keyboard navigation: Tab фокусирует элементы, Enter/Space активирует
- [ ] Скриншот работающего Dashboard добавлен в `homework/M4/screenshots/dashboard/`
- [ ] Скриншот файлового дерева (структура компонентов) добавлен

---

## Часть 3: Редизайн страницы

- [ ] Выбрана и редизайнена минимум одна страница proshop_mern
- [ ] Скриншот "до" добавлен в `homework/M4/screenshots/redesign/`
- [ ] Скриншот "после" добавлен в `homework/M4/screenshots/redesign/`
- [ ] Решение по компонентам зафиксировано в README (что взяли готовым, что написали кастомно)

---

## Часть 4: DESIGN.md

- [ ] `DESIGN.md` (или `DESIGN_SYSTEM.md`) лежит в **корне репо** рядом с `CLAUDE.md`
- [ ] Минимум 7 секций: color / typography / spacing / radius / elevation / components / states
- [ ] Шрифт — не Inter (или есть явное обоснование в файле почему Inter)
- [ ] Spacing scale содержит только числа кратные 8
- [ ] В `CLAUDE.md` добавлена строка `## Design rules: see ./DESIGN.md`
- [ ] Скриншот дерева корня репо показывает `DESIGN.md` рядом с `CLAUDE.md`

---

## Q&A

- [ ] Файл `homework/M4/qa-reflection.md` создан (или Q&A встроен в README)
- [ ] Ответы на минимум 4–5 вопросов из Q&A-блока

---

## Anti-AI-slop check

Быстрый визуальный аудит перед сдачей — посмотрите на Dashboard и редизайн:

- [ ] Нет cringe-градиентов (фиолетовые / радужные переходы)
- [ ] Нет 2-column comparison blocks как дефолтный layout
- [ ] Нет heavy borders на карточках (тонкая линия или убрать совсем)
- [ ] Hover state есть на кнопках и интерактивных элементах
- [ ] Focus state виден при keyboard navigation (не скрыт через `outline: none`)
- [ ] Loading state (skeleton) работает, не просто спиннер
- [ ] Все отступы кратны 8 (8 / 16 / 24 / 32 / 48 / 64 — нет 13px, 17px, 25px)
- [ ] Шрифт соответствует `DESIGN.md` (не Inter, если не обосновали)

---

## Финал

- [ ] PR создан в вашем репо
- [ ] Ссылка на PR отправлена в LMS или чат курса
- [ ] Все скриншоты загружены и не битые ссылки (проверьте в preview)
- [ ] Структура папки соответствует дереву из `homework-spec.md`:
  ```
  homework/M4/
  ├── analysis.md
  ├── qa-reflection.md
  ├── README.md
  └── screenshots/
      ├── before/
      ├── dashboard/
      └── redesign/
  DESIGN.md   ← в корне репо
  ```

---

*Если все пункты закрыты — готово к сдаче. Если что-то не закрыто и вы не понимаете почему — задайте вопрос в чате курса до дедлайна.*
