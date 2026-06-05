# Модуль 4 — UI-прототипирование: UX-first и DESIGN.md

> **Тема:** От идеи до UI за 15 минут. UX-первый подход, ландшафт инструментов 2026, `DESIGN.md` как стандарт, дизайн-система как версионируемый артефакт.
> **Сквозной проект:** `proshop_mern` — продолжение M3. Домашка — в `homework-spec.md`.

## О модуле

M4 — про то, как строить интерфейс продуктово: сначала UX, потом UI. Ландшафт инструментов генерации UX за последний год прошёл несколько эпох (browser-builders, pre-code design platforms, встроенные дизайн-режимы IDE). В 2026 у разработчика есть несколько категорий инструментов под разные контексты, и выбор между ними — стратегия, а не вкус.

Ключевой артефакт модуля — `DESIGN.md`/`DESIGN_SYSTEM.md`: то же, что `CLAUDE.md` из M2, только для дизайна. Статически закодированные правила визуального языка (цвета, типографика, отступы, компоненты), которые агент читает при каждой генерации UI.

## Что в этой папке

| Файл / папка | Зачем |
|---|---|
| `homework-spec.md` | **Главный файл ТЗ домашки.** Dashboard управления feature flags + редизайн страниц `proshop_mern` + `DESIGN.md` в корне репо. **[FROZEN]** |
| `anti-slop-supplement.md` | Дополнение: как не скатиться в «AI-look», расширение к cheatsheet про признаки AI-вёрстки. |
| `agents/` | Промпт-агенты для UX/UI работы: `ux-designer`, `shadcn-requirements-analyzer`, `shadcn-component-researcher`, `shadcn-implementation-builder`, `shadcn-quick-helper`, `frontend-developer`, `accessibility-expert`, `mobile-ui-specialist` + `shadcn-components/` + `README.md` (как пользоваться). |
| `guides/` | Гайды: `design-md-as-2026-standard.md`, `tools-landscape-2026.md`, `shadcn-pipeline.md`, `pixel-perfect-verification.md`, `component-vs-custom.md`, `export-design-md-from-stitch.md`, `milestone-clear-pattern.md` (+ `images/`). |
| `prompts/` | Готовые промпты: `reverse-design-extract.md` (скриншот → DESIGN_SYSTEM), `ascii-wireframe-lock.md`, `anti-ai-slop-guards.md`, `pixel-perfect-subagent.md`, `screenshot-fix-loop.md`, `lovable-bring-your-vibe.md`, `ux-research-question-burst.md`, `shadcn-*` (mapping / ux-structure-plan / final-implementation). |
| `templates/` | Шаблоны: `DESIGN_SYSTEM.md` + 2 варианта (`-luxury-serif`, `-minimal-tech`), `globals.css`, `claude-md-with-design-ref.md`. |
| `cheatsheets/` | Quick-reference: `12-signs-of-ai-look.md`, `10-vibecoding-best-practices.md`, `pre-flight-checklist.md`, `prompt-kit-5-templates.md`. |
| `design-system-pack-example/` | Готовый пример дизайн-пака: `design-system.{md,json,css}` + `README.md`. |
| `examples/` | Разбор: `design-tokens-extraction-walkthrough.md`. |

## Как пользоваться

1. Прочитайте `homework-spec.md` полностью: там обязательные части и Q&A-блок.
2. Возьмите шаблон из `templates/DESIGN_SYSTEM.md` как основу для своего `DESIGN.md`.
3. Агенты из `agents/` — используйте как sub-agents в CC/Cursor или копируйте `prompt.md` в новый чат (старт: `agents/README.md`).
4. `prompts/reverse-design-extract.md` — если нет дизайнера: скриншот референса → `DESIGN_SYSTEM.md` за один промпт.
5. `cheatsheets/12-signs-of-ai-look.md` + `anti-slop-supplement.md` — перед отправкой промпта в генератор.

## Сквозной проект — proshop_mern

- **M3** — MCP-сервер (3 tools) + RAG на документации. `features.json` с 25 фичами в корне репо.
- **M4** — визуальная оболочка: Dashboard управления feature flags (на основе `features.json` из M3) + редизайн страниц + `DESIGN.md` в корне.
- **M5** — Dashboard оживёт: Toggle → webhook → n8n-агент → MCP меняет реальный флаг. В M4 — UI с mock-данными.
- **M6** — агент-аудитор пройдётся по коду M3 + M4.

## Ключевые ссылки

- Данные для домашки: [`../M3/project-data/features.json`](../M3/project-data/features.json)
- Спецификация фича-флагов: [`../M3/project-data/feature-flags-spec.md`](../M3/project-data/feature-flags-spec.md)

---

*M4 — HSS AI-dev L1. Следующий модуль: M5 — Агенты и n8n.*
