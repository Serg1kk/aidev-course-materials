# Классификация данных → инструмент + Security-чеклист

> Срез: май 2026. Ядро безопасности AI-разработки: не «запретить/разрешить целиком», а матрица «класс данных → разрешённый инструмент».

## 1. Матрица «класс данных → инструмент»

| Класс | Что это | Разрешённый инструмент | Запрещено |
|---|---|---|---|
| **Public** | Документация, OSS-код, публичные спеки API | Любой frontier API (Claude / GPT), в т.ч. consumer | — |
| **Internal** | Бизнес-логика, архитектура сервисов, внутренние процессы | Корпоративный API с DPA + ZDR (Anthropic/OpenAI API, Azure OpenAI); шлюз с логированием метаданных | Consumer ChatGPT/Claude |
| **Confidential** | API-ключи, данные клиентов, IP | Managed-в-VPC (Bedrock PrivateLink / Azure OpenAI Private Link / Vertex VPC-SC) ИЛИ on-prem open-weight модель | Внешние публичные API |
| **Critical** | Секреты, PII, финансы, медицина (PHI) | Только on-prem модель + PII-маска на прокси | Любой внешний обмен (даже managed-с-BAA не всегда покрывает) |

> **ПРАВИЛО №1 (срез 2026 — политики флипаются, проверяйте):** individual/consumer-подписки тренят на ваших данных **по умолчанию (opt-out)** — и это НЕ только бесплатные тарифы. **Claude Pro $20 / Max $100 / Max $200** и **GitHub Copilot Free / Pro $10 / Pro+ $39** — все тренят by default (Copilot развернул дефолт **24 апр 2026**; Claude consumer — с окт 2025). Не тренят только **commercial с DPA: Team / Enterprise / API**. Вывод: **«платный ≠ приватный»** — решает категория (consumer vs commercial+DPA), не цена. Consumer-tier для всего выше Public — инцидент в ожидании. Детали по тарифам — §2.1.

### Почему DLP не заменяет архитектуру
DLP ищет шаблоны (номер паспорта, карты). Исходный код для DLP — просто текст: он не отличает скрипт из интернета от ядра биллинга → либо блокирует всё (паралич), либо пропускает всё. **Архитектура изоляции надёжнее DLP.**

## 2. ZDR / retention провайдеров (для выбора инструмента под Internal/Confidential)

| Провайдер | API retention | Тренит на API-данных? | Нюанс |
|---|---|---|---|
| **Anthropic** | ZERO (Messages API); Claude Code по API/commercial = zero-retention | Нет (API/commercial) | Code Execution / Tool Calling держат 30 дней; нарушение Usage Policy — до 2 лет. ⚠️ Claude Code по **Pro/Max-подписке** наследует consumer-политику и ТРЕНИТ (§2.1) |
| **OpenAI** | 30 дней abuse-monitor; ZDR для qualifying-орг по approval | Нет (API by default) | ZDR conditional + revocable; несовместим со stateful (Assistants/Batch/Vector Stores) |
| **Google Vertex AI** | Настраиваемо | Нет (Vertex) | ⚠️ Consumer Gemini до 18 мес + может тренить → использовать только Vertex, не consumer |
| **GitHub Copilot** | Business/Ent: zero-retention в IDE; Individual — см. §2.1 | Business/Ent — НЕТ (DPA); Individual (Free/Pro/Pro+) — **ДА by default** (с 24.04.2026) | «GitHub тренит свою модель» ≠ «провайдер модели тренит» — §2.1 |

> **ZDR одной фразой:** «вы платите за вычисления, а не за то, чтобы провайдер учил на вашем коде свои будущие модели».

### 2.1 Тренинг по тарифам подписок (главная ловушка 2026)

⚠️ Разводите ДВА разных вопроса (их постоянно путают):
1. **Вендор-обёртка тренит СВОЮ модель?** (Anthropic на Claude-чатах / GitHub на Copilot) — на individual-тарифах ДА by default.
2. **Провайдер базовой модели тренит?** (OpenAI / Anthropic / Google под капотом) — практически НИКОГДА (контрактные ZDR-соглашения). Даже если провайдер не тренит — сам вендор-обёртка на consumer-тарифе может.

| Тариф | Тренит by default? | Как отключить / нюанс |
|---|---|---|
| **Claude** Free / Pro $20 / Max $100 / Max $200 | ✅ ДА (retention до 5 лет, иначе 30 дн) | toggle «Help improve Claude» → off |
| **Claude** Team / Enterprise / API | ❌ НЕТ | off by default (контракт) |
| **Copilot** Free / Pro $10 / Pro+ $39 (Individual) | ✅ ДА (с 24.04.2026) | `github.com/settings/copilot/features` → Privacy → off |
| **Copilot** Business $19 / Enterprise $39 (seat) | ❌ НЕТ (DPA) | toggle не показывается |

- **Claude Code** не имеет своей политики — **наследует категорию аккаунта**: по Pro/Max ваш КОД идёт в обучение (если toggle ON); по API-ключу / Team / Enterprise — нет. 🪤 Если в окружении есть `ANTHROPIC_API_KEY` — Claude Code уходит в commercial-режим (не тренит, но биллит по API-ставкам).
- **Private-repo не спасает:** код «at rest» в приватном репо в трейнинг не идёт, НО фрагменты, которые вы руками скормили ассистенту (chat/inline), — могут.
- **Вывод:** $200 Claude Max и $39 Copilot Pro+ тренят, а $19 Copilot Business — нет. Цена не = приватность; смотрите на категорию (consumer vs commercial+DPA) и на состояние toggle.

## 3. Гибридные архитектуры (норма, а не компромисс)

Устойчивая архитектура почти всегда гибридная: качество внешних моделей + контроль над чувствительным.

- **PII-маска на прокси («таможенник»):** прокси-шлюз (например LiteLLM + Presidio) перехватывает запрос → маскирует (`<EMAIL>`, `пользователь_1`) → отправляет наружу → восстанавливает теги в ответе. Защищает от случайной утечки PII, но **не видит исходный код** (как и DLP) → не заменяет on-prem для Critical.
- **RAG с локальным эмбеддингом:** vector-store и embeddings в своём периметре, наружу уходят только обобщённые чанки. ⚠️ Чанк с захардкоженным секретом утечёт как сам код — нужна та же фильтрация.
- **Маршрутизация по уровню секретности:** классификатор направляет public-запросы во внешний API, sensitive — в локальную модель. ⚠️ Роутер должен быть консервативным и **захардкоженным**, не «router-as-a-service» (сам не должен стать точкой утечки).

## 4. TEE / Confidential Compute — «третий путь»

Между публичным cloud-API и full on-prem: GPU-сервер с аппаратной доверенной средой + аттестация, где данные не видны даже провайдеру инфраструктуры.

| Провайдер | Технология | Сценарий |
|---|---|---|
| Apple PCC | Stateless + attestation + verifiable transparency | Privacy-first |
| Azure Confidential GPU | AMD SEV-SNP + NVIDIA H100 | Банк / enterprise на Azure |
| AWS Nitro Enclaves | KMS envelope + attestation | Healthcare GenAI |
| NVIDIA Protected PCIe | TEE-режим GPU (H100/H200) | Базовый кирпич TEE-GPU |

> ⚠️ **Важно для РФ:** западные TEE-решения недоступны под санкциями. Замена — on-prem GPU **без** аппаратных TEE-гарантий аттестации. Для строгого банковского сценария «третий путь» в РФ пока не работает так, как на Западе. Это открытый вопрос, а не решённая опция.

## 5. Security-чеклист перед production

**Данные и инструмент:**
- [ ] Определены классы данных (Public / Internal / Confidential / Critical) для проекта
- [ ] Каждому классу сопоставлен разрешённый инструмент (матрица выше)
- [ ] Consumer/individual-подписки (free/Plus/Pro/Max/Copilot Pro — даже платные) НЕ используются ни для чего выше Public; training-toggle проверен (§2.1)
- [ ] Для Internal+ подтверждён ZDR / DPA с провайдером (commercial-категория, не individual)

**Архитектура (Defense-in-Depth):**
- [ ] Авторизация (RBAC) реализована В КОДЕ, не делегирована модели
- [ ] Системный промпт считается ПУБЛИЧНЫМ (секретов в нём нет)
- [ ] Есть отдельный network egress-контроль (inference-guardrail ≠ network-firewall)
- [ ] Untrusted code исполняется в sandbox (Docker), не на хосте

**Агент и действия:**
- [ ] Агент работает по allow-list действий, не «что угодно»
- [ ] Доступ к БД — read-only грант на уровне БД (не «доверие» в промпте); параметризованные запросы вместо свободного SQL
- [ ] Необратимые действия (DROP, удаление, транзакции) требуют human-in-the-loop / out-of-band подтверждения
- [ ] Dev и prod среды разделены; агент НЕ имеет prod-доступа в режиме разработки

**Supply chain:**
- [ ] Зависимости с version pinning + `--require-hashes`; есть SBOM
- [ ] LLM API-ключи трактуются как crown jewels (ротация, секрет-менеджер)
- [ ] Origin модели/датасета проверен (особенно при загрузке весов из публичных хабов)

**Compliance (если применимо):**
- [ ] Понятно, какое регулирование применяется (зависит от того, где сидит ваш ПОЛЬЗОВАТЕЛЬ, не где сидит модель)
- [ ] Для ПД в РФ: данные не уходят в зарубежные API (on-prem или маскирование)
- [ ] Для EU-пользователей: сверены обязательства EU AI Act (см. `compliance-timelines.md`)

**Бюджет безопасности (трезво):**
- [ ] Уровень защиты пропорционален риску (генератор постеров ≠ доступ к деньгам/ПД)
- [ ] Решено, что делать на 1-м этапе, а что — после доказательства ROI (но для критичных данных безопасность — не «потом»)

---

## Что почитать
- NIST SP 800-122 — Protecting PII — https://csrc.nist.gov/pubs/sp/800/122/final
- LiteLLM PII masking via Presidio — https://docs.litellm.ai/docs/proxy/guardrails/pii_masking_v2
- AWS PrivateLink — https://docs.aws.amazon.com/vpc/latest/privatelink/what-is-privatelink.html
- Azure Private Link — https://learn.microsoft.com/en-us/azure/private-link/private-link-overview
- Apple Private Cloud Compute — https://security.apple.com/blog/private-cloud-compute/
- Azure confidential GPU (SEV-SNP + H100) — https://learn.microsoft.com/en-us/azure/confidential-computing/gpu-options
- ⚠️ Политики тренинга по тарифам **меняются** — сверяйте перед проектом: Anthropic Privacy Center — https://privacy.anthropic.com/ · GitHub Copilot docs — https://docs.github.com/copilot
