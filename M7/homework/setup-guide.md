# Setup-гайд: поднять локальную модель + endpoint

> Цель — получить **OpenAI-совместимый endpoint** (`/v1/chat/completions`), в который роутер из DZ1
> будет слать «приватные» запросы. Стандарт де-факто: меняешь модель/бэкенд — код клиента не трогаешь,
> только `base_url`. Срез цен: май 2026, проверяй по ссылкам перед применением.

---

## Сначала — три движка (это часто путают)

| Движок | Что это | Где живёт | Когда брать |
|---|---|---|---|
| **Ollama** | Простейший локальный сервер моделей (1 команда). OpenAI-совместимый `/v1`. | Твой Mac/ПК **или** обычный Ubuntu-VPS/VM | Дефолт для дев/прототипа и для «локалки в облаке» |
| **LM Studio** | То же, что Ollama, но с GUI + headless-режимом | Локально (и headless на сервере) | Хочешь визуальный UI |
| **vLLM** | Прод-движок: PagedAttention, батчинг, высокий throughput, конкурентность | Машина с **NVIDIA GPU** (CUDA) | «По-взрослому», много запросов, GPU-аренда |

> Ключевое: **«облако» это не обязательно vLLM.** Можно арендовать обычный VPS на Ubuntu и поставить там
> Ollama одной командой — получится «локалка, но в облаке» (см. путь B1). vLLM — это отдельный, более
> тяжёлый прод-вариант под GPU (путь B2). Для домашки хватает Ollama где угодно.

---

## Три способа деплоя — выбери под себя

| | (A) Локально | (B) Облако-сам | (C) Endpoint преподавателя |
|---|---|---|---|
| Инструмент | Ollama / LM Studio | B1: VPS+Ollama · B2: GPU+vLLM | URL + ключ (раздаётся) |
| Стоимость | $0 (своё железо) | $0-10 (см. таблицы ниже) | $0 |
| Настройка | минимальная (1 команда) | средняя | нулевая (URL в конфиг) |
| Приватность | 100% локально | на стороне облака | на стороне преподавателя |
| Для кого | Mac 16ГБ+ / GPU 8ГБ+ | хочешь прод-опыт / слабое локальное железо | тонкий ноут, не хочешь возиться |

**Рекомендация:** дефолт — **(A) Ollama локально**. Не тянет железо и не хочется возни → **(C)**.
Хочется прод-опыт → **(B)**. Роутер (лёгкий) можно держать локально на CPU, а тяжёлый процессор вынести
в облако/на endpoint преподавателя.

---

## (A) Локально

### Ollama (проще всего)
```bash
OLLAMA_HOST=0.0.0.0:11434 ollama serve
ollama pull qwen3:8b-q6_K          # ⚠️ ЯВНО задаём квант — не дефолтный Q4!
# endpoint: http://localhost:11434/v1
```
```python
from openai import OpenAI
client = OpenAI(base_url="http://localhost:11434/v1", api_key="ollama")
print(client.chat.completions.create(model="qwen3:8b-q6_K",
      messages=[{"role":"user","content":"привет"}]).choices[0].message.content)
```

### LM Studio (GUI + headless)
```bash
lms server start --port 1234 --host 0.0.0.0 --cors
# endpoint: http://localhost:1234/v1 ; apiKey — любая непустая строка
```

---

## (B) Облако-сам

### B1 — VPS + Ollama («локалка в облаке», CPU или GPU)
Арендуешь обычную Ubuntu-машину, ставишь Ollama одной командой:
```bash
curl -fsSL https://ollama.com/install.sh | sh    # детектит x86/ARM сам
sudo systemctl edit ollama.service               # добавить: Environment="OLLAMA_HOST=0.0.0.0"
sudo systemctl restart ollama
ollama pull qwen3:8b-q6_K
```
⚠️ У Ollama нет встроенной авторизации — не открывай порт 11434 в мир голым. Варианты: SSH-туннель
(`ssh -L 11434:localhost:11434 user@host -N`) или nginx + Basic Auth, или firewall на свой IP.

**CPU-VPS** (без GPU, упор в RAM; 7-8B Q4 ~5-10 t/s — медленно, но работает для домашки):

| Провайдер | Тир | RAM / vCPU | Цена | Влезает |
|---|---|---|---|---|
| **Oracle Cloud Free** | A1 Flex (Always Free) | 24 ГБ / 4 ARM | **$0** | 4B роутер + 8B процессор |
| OVH | VPS-2/3 | 12-24 ГБ | ~$10-20/мес | 7-14B |
| Contabo | Cloud VPS 20/30 | 12-24 ГБ | ~$7-15/мес | 7-14B |
| Hetzner | CAX21/31 (ARM) | 8-16 ГБ | ~€8-17/мес | 7-8B / 14B |

> 💡 **Oracle Free Tier** реально гоняет 4B+8B бесплатно (региона может не быть в наличии — пробуй
> Frankfurt/Singapore).

### B2 — GPU-аренда + vLLM (прод-grade)
```bash
vllm serve Qwen/Qwen3-8B --host 0.0.0.0 --port 8000 --max-model-len 8192 --api-key token-abc123
# tool-calling: --enable-auto-tool-choice --tool-call-parser hermes
# против OOM: --max-model-len (режет KV-cache) + --gpu-memory-utilization
```

**GPU-сервисы** (RTX 4090 24ГБ тянет любой процессор до 32B; бюджет $2-10 = многие часы):

| Сервис | Биллинг | vLLM-шаблон | RTX 4090 | Заметка |
|---|---|---|---|---|
| **RunPod** | per-sec, **serverless scale-to-zero** | ✅ официальный | Pod $0.34/ч (serverless: L40S ~$1.9/ч) | Лучший «нажми-готово» vLLM, платишь за реальные секунды |
| **Vast.ai** | per-sec, marketplace | ✅ serverless | от $0.29-0.40/ч | Дешевле всех; хост может забрать interruptible-машину |
| **TensorDock** | per-hour / spot | ручками | $0.37/ч ($0.20 spot) | Простая VM-аренда |
| **Together AI** | per-token API | — (managed) | — ($25 free) | Если можно «просто endpoint» без своего vLLM: $5 = ~50М токенов |

> Для нерегулярной ДЗ-нагрузки **RunPod Serverless** = реально центы. Если ТЗ не требует «подними vLLM
> сам», а нужен просто endpoint — **Together AI free $25** закрывает всё бесплатно.

---

## (C) Endpoint преподавателя

Если железо не тянет и облако влом — **попроси `base_url` + ключ в личке у преподавателя в Telegram** и
впиши в конфиг роутера. Ничего ставить не надо. Это CPU-хостед, общий на группу — **медленнее своего и
под нагрузкой может ждать очередь**, держи это в уме. Подходит как fallback, не как дефолт.

```python
client = OpenAI(base_url="<выданный_base_url>/v1", api_key="<выданный_ключ>")
```

---

---

## Опционально (НЕ нужно для домашки) — для тех, кто хочет глубже

Две штуки ниже **в домашке от тебя не требуются**. Это альтернативы для любопытных и для прод-варианта.
Если ты выбрал путь A/B/C выше и роутер делаешь в n8n/коде — можешь спокойно их пропустить.

### llama.cpp — ещё один движок локальной модели (вместо Ollama)
**Что это:** то же, что Ollama/LM Studio (поднять модель как OpenAI-endpoint), но без Python-зависимостей,
заточен под CPU/Mac/edge. **Когда брать:** если по какой-то причине не хочешь Ollama. Для домашки Ollama
проще и достаточно — это просто другой движок того же назначения.
```bash
./llama-server -m qwen3-8b-Q6_K.gguf --host 0.0.0.0 --port 8080 -c 4096 -t 8   # -ngl 0 = чистый CPU
```

### LiteLLM — «прод-способ» сделать сам РОУТЕР (вместо n8n/кода)
**Что это НЕ:** это не ещё одна локальная модель. **Что это:** шлюз/прокси — один OpenAI-совместимый
endpoint поверх всех твоих моделей (Ollama + vLLM + облачные API), который сам умеет роутить и навешивать
PII-guardrail. **Где используется:** это **альтернативный, «взрослый» способ сделать роутер из DZ1** —
вместо того чтобы собирать его в n8n или писать кодом. Большинству для домашки хватает n8n/кода; LiteLLM
бери, только если хочешь сделать роутер по-проду (это идёт как бонус к оценке).
```yaml
# config.yaml
model_list:
  - model_name: router                 # лёгкая модель для решения о маршруте
    litellm_params: {model: openai/qwen3:4b, api_base: http://localhost:11434/v1, api_key: "x"}
  - model_name: processor              # модель, которая отвечает
    litellm_params: {model: openai/Qwen/Qwen3-8B, api_base: http://vllm-host:8000/v1, api_key: "x"}
```
```bash
litellm --config config.yaml          # поднимет endpoint http://0.0.0.0:4000
```
Грабли LiteLLM: `api_base` обязан кончаться на `/v1`; `api_key` обязателен даже если бэкенд его не требует.

---

## Подключить кодинг-агента к локальной модели (для DZ2, если делаешь)
- **Claude Code → Ollama:** `ANTHROPIC_BASE_URL=http://localhost:11434` + `ANTHROPIC_AUTH_TOKEN=ollama`
  → `claude --model qwen3` (Claude Code требует контекст ≥64K).
- **OpenCode → Ollama:** провайдер `@ai-sdk/openai-compatible`, `baseURL: http://localhost:11434/v1`.
