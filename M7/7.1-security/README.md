# Topic 7.1 — Безопасность AI-разработки

Материалы по безопасности AI-driven разработки. Срез: май 2026.

## Что в этой папке

| Файл | Что внутри |
|---|---|
| `data-classification-and-checklist.md` | Матрица «класс данных → инструмент» (4 уровня) + ZDR-таблица провайдеров + готовый security-чеклист перед прод |
| `local-models-inference-guide.md` | Локальные open-weight модели, инференс-стек (vLLM/SGLang/Ollama), квантизация (Q-floor), 152-ФЗ + суверенность |
| `attacks-owasp-guardrails.md` | Каталог атак на LLM/агентов, OWASP Top 10 for LLM 2025, эшелоны защиты, реальные инциденты |
| `compliance-timelines.md` | EU AI Act (даты + штрафы), 152-ФЗ, суверенный ИИ РФ, HIPAA/GDPR/ISO 42001 |

## Главные идеи

- **Архитектура надёжнее политики.** Полагаться на корпоративную политику — как повесить табличку «не воруйте» на открытый сейф. Политика требует идеального поведения каждого сотрудника каждый день; архитектура работает всегда. Запрет тоже не спасает — люди уходят в shadow-AI с личных аккаунтов.
- **Классификация — это продуктовое решение.** Не «запретить ChatGPT», а матрица «класс данных → инструмент». Водораздел одним предложением: **free/Plus ChatGPT тренит на ваших данных, API/Enterprise — нет.**
- **Модель небезопасна by design.** Она не отличает инструкцию от данных — всё один поток токенов. Защита **вероятностна** (не как антивирус с фикс. сигнатурами). «99% защиты = 0% защиты» — атакующему хватает 1%.
- **Защищай ДЕЙСТВИЯ, не ОТВЕТЫ.** Allow-list действий, параметризованный инструмент вместо свободного SQL, read-only грант агенту на уровне БД (не в системном промпте!), out-of-band подтверждение транзакций.
- **Defense-in-Depth — принцип, не инструмент.** Ни один guardrail не bypass-proof. Эшелон («швейцарский сыр»), а не один слой. Каждый слой стоит +×2-3 latency — это product-trade-off.

## Что читать первым

1. `data-classification-and-checklist.md` — построить матрицу под свой проект и пройти чеклист.
2. `attacks-owasp-guardrails.md` — понять модель угроз.
3. `local-models-inference-guide.md` — если есть требование «данные не уходят за периметр».
4. `compliance-timelines.md` — если продаёте в EU или работаете с ПД в РФ.

## 🎥 Что посмотреть

- Анализ атак на LLM, техники промпт-инъекций — Евгений Кокуйкин (Raft), SafeCode — https://www.youtube.com/watch?v=b9HSmbL1VyA
- Prompt Engineering & AI Red Teaming — Sander Schulhoff (Learn Prompting / HackAPrompt), AI Engineer — https://www.youtube.com/watch?v=_BRhRh7mOX0
- Безопасность LLM: риски и защита корпоративных систем — Евгений Кокуйкин (Raft), Codex Town Club — https://www.youtube.com/watch?v=66N2MDAI7_w
- Security deep-dive: Dual LLM, lethal trifecta — Podlodka — https://www.youtube.com/watch?v=8qVp7tVLsV4
- Взлом AI-агентов — Александр Князев, Heisenbug — https://www.youtube.com/watch?v=x_gLUFL9mKo
