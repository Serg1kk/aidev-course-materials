# AI-Driven Development — материалы курса

Публичные материалы курса AI-Driven Development (Level 1). Обновляется по ходу запуска.

## Структура

```
/
├── M1/   ← Промптинг и мета-промпты
├── M2/   ← AI-IDE, rules-файлы, индексация кодбейзов
├── M3/   ← Контекст и данные: RAG, CAG, MCP-серверы
├── M4/   ← (TBD) UI-прототипирование
├── M5/   ← (TBD) n8n, агенты
├── M6/   ← (TBD) AI-аудит, legacy refactoring
└── M7/   ← (TBD) Стратегия, ROI, безопасность
```

## Что сейчас опубликовано

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

---

## Лицензия

Материалы — CC BY 4.0. Используйте в своих командах/проектах. При копировании желательна ссылка на источник.

*Автор и поддержка: инструктор курса.*
