# AI-Driven Development — материалы курса

Публичные материалы курса AI-Driven Development (Level 1). Все 7 модулей опубликованы.

## Структура

```
/
├── M1/   ← Промптинг и мета-промпты
├── M2/   ← AI-IDE, rules-файлы, индексация кодбейзов
├── M3/   ← Контекст и данные: RAG, CAG, MCP-серверы
├── M4/   ← UI-прототипирование: UX-first, DESIGN.md, дизайн-система
├── M5/   ← Агенты и n8n: «мозг» системы (UI ↔ Agent ↔ MCP)
├── M6/   ← AI-аудит, living docs, legacy refactoring, synthetic testing
└── M7/   ← Стратегия: безопасность, экономика, внедрение
```

Сквозной учебный проект M3→M7 — `proshop_mern` (MERN e-commerce): на нём строятся все домашки.

## Что опубликовано

### M1 — Промптинг и модели

- **[Полезные ресурсы Модуля 1](M1/resources.md)** — все ссылки в одном месте: сервисы выбора моделей (Artificial Analysis, OpenRouter, LM Arena), мета-промпты, документация, провайдеры, скиллы.
- **[Эволюция промптинга](M1/prompting-evolution-explanation.md)** — как менялся промптинг с 2022 по 2026: от "47 прилагательных" до скиллов и агентов.
- **[INoT — техника промптинга](M1/inot-prompting-technique.md)** — дебат внутри одного промпта: +8% качества при −58% токенов vs классический multi-agent.
- **[API параметры — справка](M1/api-parameters-explanation.md)** — что за Temperature / Top P / Penalty / seed / response_format, когда крутить, сравнение моделей 2026.

### M2 — AI-IDE

- **[Rules-файлы по 10 AI-IDE — справочник](M2/rules-files-per-ide.md)** — одна страница на каждую IDE (Claude Code, Cursor, Codex CLI, OpenCode, Windsurf, Gemini CLI, Antigravity, Copilot, JetBrains Junie, Warp): что создать, куда положить, как не сломать. + 10 готовых templates.
- **[Индексация и онбординг на чужой репо](M2/indexation-and-onboarding.md)** — проверенные промпты для работы с незнакомым кодбейзом: архитектурный обзор, трассировка фичи, structured bug-hunt, Mermaid-диаграммы.

### M3 — Контекст и данные: RAG, CAG, MCP

#### Домашка
- **[Домашнее задание M3](M3/assignment/homework-spec.md)** — 3 части (Feature Flags MCP + RAG над documentation + Search MCP wrap), сквозной артефакт `proshop_mern` для M4-M5.

#### Гайды (RU + EN, переключатель в начале каждого файла)
- **[Vector DB comparison 2026](M3/guides/vector-db-comparison-2026.md)** — Qdrant / Weaviate / pgvector / Pinecone / LightRAG / GraphRAG: decision tree + cost matrix + миграционные истории.
- **[Embedding models 2026](M3/guides/embedding-models-2026-guide.md)** — OpenAI 3-small/large, Cohere v3, BGE-M3, Voyage-3-large, Voyage-code-3, Gemini Embedding 2: цены, бенчмарки, мультимодальность, Matryoshka.
- **[Chunking strategies guide](M3/guides/chunking-strategies-guide.md)** — chunk size, overlap, top-K, hybrid BM25+semantic, Cohere/BGE rerankers, Anthropic Contextual Retrieval, антипаттерны.
- **[MCP design principles](M3/guides/mcp-design-principles.md)** — 10 принципов проектирования MCP-сервера: tool count, descriptions как промпт, destructive ops, eager-vs-lazy, security, типы серверов, MCP-роутеры.
- **[MCP vs CLI vs Skills decision framework](M3/guides/mcp-vs-cli-vs-skills-decision-framework.md)** — когда что выбирать: ScaleKit benchmark (17× cheaper, 4-32× tokens), Geoffrey Huntley critique, Allen Hutchison 3-interface pattern.
- **[grep + LSP + Agents.md guide](M3/guides/grep-lsp-agents-md-guide.md)** — почему лидеры AI-coding (Claude Code, Aider, Codex CLI, OpenCode, Continue) отказались от vector RAG для кода и что используют вместо.

#### Скиллы (универсальные, можно адаптировать под свой стек)
- **[notebooklm](M3/skills/notebooklm/)** — программный API NotebookLM (создание, источники, чат, генерация артефактов).
- **[telegram-cli](M3/skills/telegram-cli/)** — отправка/чтение TG-сообщений из Claude Code, gramJS-based.
- **[jira](M3/skills/jira/)** — Jira CRUD: push user stories, scan tickets, create tasks, pull bugs/history.
- **[confluence](M3/skills/confluence/)** — Confluence wiki: read pages, publish from local, search.

#### Project data (~50K слов про `proshop_mern` для RAG-домашки)
- **[Architecture, glossary, best-practices, dev-history](M3/project-data/)** — system overview, термины, паттерны MERN, история проекта.
- **[Feature flags spec + features.json](M3/project-data/feature-flags-spec.md)** — 25 флагов, контракт 3 tools, бизнес-логика валидации.
- **[Features × 6, pages × 16, API × 5, ADRs × 5, runbooks × 6, incidents × 3](M3/project-data/)** — полный production-style набор документации.

### M4 — UI-прототипирование

- **[Домашка M4](M4/homework-spec.md)** — Dashboard управления feature flags + редизайн страниц `proshop_mern` + `DESIGN.md` в корне репо.
- **[DESIGN.md как стандарт 2026](M4/guides/design-md-as-2026-standard.md)** — статически закодированный визуальный язык, который агент читает при каждой генерации UI (как `CLAUDE.md`, но для дизайна).
- **[Ландшафт UI-инструментов 2026](M4/guides/tools-landscape-2026.md)** — 5 категорий генераторов UI и когда что выбирать. + [shadcn pipeline](M4/guides/shadcn-pipeline.md), [pixel-perfect verification](M4/guides/pixel-perfect-verification.md), [component vs custom](M4/guides/component-vs-custom.md).
- **[Агенты UX/UI](M4/agents/README.md)** — UX-designer, shadcn analyzer/researcher/builder, frontend-developer, accessibility-expert, mobile-ui-specialist.
- **[Промпты](M4/prompts/)** — reverse-design (скриншот → DESIGN_SYSTEM), ascii-wireframe-lock, anti-ai-slop-guards, pixel-perfect, screenshot-fix-loop, shadcn-*.
- **[Шаблоны DESIGN_SYSTEM](M4/templates/DESIGN_SYSTEM.md)** + **[пример дизайн-пака](M4/design-system-pack-example/)** + **[12 признаков AI-look](M4/cheatsheets/12-signs-of-ai-look.md)**.

### M5 — Агенты и n8n

- **[Домашка M5](M5/homework-spec.md)** — 2 workflow (manual toggle через webhook→n8n→MCP + scheduled defensive monitor), замкнутый цикл UI ↔ Agent ↔ MCP на `proshop_mern`.
- **[CC-субагенты для n8n](M5/agents/README.md)** — связка [n8n-requirements-orchestrator](M5/agents/n8n-requirements-orchestrator.md) (идея → spec) + [n8n-workflow-builder](M5/agents/n8n-workflow-builder.md) (spec → валидный n8n JSON).
- **[CrewAI vs n8n](M5/guides/crewai-vs-n8n.md)** — когда что выбирать, гибридный паттерн.
- **[Облачный n8n + локальные сервисы](M5/guides/cloud-n8n-local-services.md)** — как пробросить локальные MCP/Dashboard публичным HTTPS через Cloudflare tunnel.
- **[Защита browser-агентов от prompt injection](M5/guides/browser-use-prompt-injection-defenses.md)** — security-reference (теория, опционально).

### M6 — AI-аудит и Legacy

- **[Домашка M6](M6/homework-spec.md)** — 4 stages (~7-9 ч): запуск агента-аудитора на своём `proshop_mern`, 3 топовых findings через safe-refactor, расширение AGENTS.md/project-index.json, опц. mutation-testing.
- **[5 mate-агентов](M6/agents/README.md)** — security / architecture / performance / legacy-auditor / test-writer (shared-библиотека универсальных аудиторов).
- **[6.1 AI Code Review](M6/6.1-ai-code-review/README.md)** — эталонная архитектура review (webhook → LLM → CI/CD), сравнение тулов, honest risks.
- **[6.2 Living Documentation](M6/6.2-living-documentation/README.md)** — AGENTS.md/CLAUDE.md стандарт, project-index.json, борьба с RAG freshness rot.
- **[6.3 Legacy-стратегии](M6/6.3-legacy-strategies/README.md)** — 4-step CC reverse engineering + 7 правил безопасного рефакторинга.
- **[6.4 Synthetic testing](M6/6.4-synthetic-testing/README.md)** — synthetic test-cases, mutation testing, coverage vs MSI, CI/CD layered. + **[глоссарий](M6/resources/glossary.md)** / **[каталог тулов](M6/resources/tools-catalog.md)**.

### M7 — Стратегия: безопасность, экономика, внедрение

- **[Домашка-капстоун M7](M7/homework/)** — приватный AI-ассистент в `proshop_mern`: роутер по чувствительности (PII → локалка, чисто → облако) + админ-дашборд трекинга, опционально prompt-инъекции и защита. Старт: **[homework/homework-spec.md](M7/homework/homework-spec.md)**.
- **[AI-readiness self-assessment](M7/ai-readiness-self-assessment.md)** — шаблон саморефлексии: карта реальности + decision-tree под твою компанию (не сдаётся).
- **[7.1 Security](M7/7.1-security/README.md)** — классификация данных, security-чеклист, локалки + инференс-стек, OWASP + guardrails, compliance.
- **[7.2 Economics](M7/7.2-economics/README.md)** — pricing 2026 + deprecation, cost-optimization (кэш/routing/self-host break-even), fine-tuning экономика.
- **[7.3 Adoption](M7/7.3-adoption/README.md)** — adoption-плейбук (70/30), роли и хайринг, построение агентов в компаниях, метрики и pitfalls.
- **[7.4 Horizons](M7/7.4-horizons/README.md)** — карта AGI-прогнозов + anti-hype toolkit. + **[глоссарий](M7/resources/glossary.md)** / **[reading-list](M7/resources/reading-list.md)** / **[каталог тулов](M7/resources/tools-catalog.md)**.

---

## Лицензия

Материалы — CC BY 4.0. Используйте в своих командах/проектах. При копировании желательна ссылка на источник.

*Автор и поддержка: инструктор курса.*
