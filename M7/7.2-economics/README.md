# Topic 7.2 — Экономика и выбор стека

Материалы по экономике LLM-приложений. Срез: май 2026. **Все цены волатильны** — проверяйте по официальным ссылкам.

## Что в этой папке

| Файл | Что внутри |
|---|---|
| `pricing-and-deprecation-cheatsheet.md` | Карта моделей 2026 (4 тира) + deprecation cliff + как читать blended cost |
| `cost-optimization-playbook.md` | Кэширование (3 движка), routing/gateway, self-host TCO (LCOAI), BYOK |
| `frameworks-and-finetuning.md` | Framework landscape (5 осей), fine-tuning экономика (когда оно вообще окупается) |

## Главные идеи

- **Цена флагмана больше не аргумент.** Топовая модель сегодня стоит как топовая год назад, а полтора года назад тот же уровень интеллекта стоил в ~240× дороже. Средний тир решает 80-95% задач при стоимости в 5-20× ниже флагмана.
- **Смотри на blended cost, не на прайс-лист.** Реальная стоимость = профиль нагрузки (соотношение input/output, доля кэша, доля batch), а не цена за токен.
- **Экономика — мостик к adoption, не самоцель.** Если не понимаешь стоимость своих LLM-вызовов — не можешь принимать архитектурные решения, только надеешься.
- **Self-host редко окупается ниже определённого объёма.** GPU — лишь ~40% TCO. Считай по LCOAI = (CapEx + OpEx) / 1000 инференсов, а не «счёт за токены vs цена стойки».
- **Fine-tuning — про форму, не про факты.** Порядок решений: Prompt → RAG → Fine-tune → Distill. 96% стоимости fine-tuning — это данные, эвал и люди, сам train-run почти бесплатен.

## Что читать первым
1. `pricing-and-deprecation-cheatsheet.md` — сориентироваться в моделях и ценах.
2. `cost-optimization-playbook.md` — снизить счёт (кэш → routing → потом self-host).
3. `frameworks-and-finetuning.md` — выбрать фреймворк по 5 осям; решить, нужен ли fine-tuning.

## 🎥 Что посмотреть

- Mastering AI Economics (FinOps) — Finout — https://www.youtube.com/watch?v=8L-HOYhMi0s
- LLM Inference Optimization — Chip Huyen, ScyllaDB P99 Conf — https://www.youtube.com/watch?v=-MIv3mWAlBc
- The Economics of Enterprise LLMs: AI Break-Even Point — Tim Barnes — https://www.youtube.com/watch?v=Kj3eQkZsCpM
- AI-агенты, производительность и здравый смысл (токеномика) — подкаст «Свободный слот» / AvitoTech — https://www.youtube.com/watch?v=UG2a6i2lHko
