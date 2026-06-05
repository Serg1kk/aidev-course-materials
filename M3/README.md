# Модуль 3 — Контекст и данные: RAG, CAG, MCP

> **Статус:** материалы выложены ДО занятия. Домашка — в `assignment/homework-spec.md`.

## О модуле

M3 собирает «исполнительный слой» системы: MCP-сервер для управления feature flags (руки) + RAG над документацией проекта. Сквозной учебный проект — `proshop_mern` (MERN e-commerce), на котором строятся все домашки M3→M7. Здесь же лежит полный production-style корпус документации этого проекта для RAG-практики.

## Что в этой папке

| Файл / папка | Зачем |
|---|---|
| `assignment/homework-spec.md` | **Главный файл ТЗ домашки** (3 части + опц. 4-я): Feature Flags MCP + RAG над documentation + Search MCP wrap. **[FROZEN]** |
| `guides/` | 6 гайдов, каждый в паре RU + EN (переключатель языка в начале файла): vector DB comparison, embedding models 2026, chunking strategies, MCP design principles, MCP vs CLI vs Skills, grep + LSP + AGENTS.md. |
| `project-data/` | ~50K-словный корпус про `proshop_mern` для RAG-домашки: architecture, glossary, best-practices, dev-history, feature-flags-spec + features.json (25 флагов), features × 6, pages × 16, API × 5, ADRs × 5, runbooks × 6, incidents × 3. |
| `skills/` | 4 универсальных скилла (можно адаптировать под свой стек): `notebooklm` (API NotebookLM), `telegram-cli` (TG из Claude Code), `jira` (Jira CRUD), `confluence` (wiki read/publish/search). |

## Домашка

ТЗ домашки M3 → `assignment/homework-spec.md`. Сквозной артефакт `proshop_mern` переходит в M4→M5.

---

*M3 — HSS AI-dev L1. Следующий модуль: M4 — UI-прототипирование.*
