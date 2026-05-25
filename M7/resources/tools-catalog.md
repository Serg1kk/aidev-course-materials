# Каталог инструментов M7

> Единый список инструментов модуля со ссылками. Срез: май 2026. Статусы и цены меняются — сверяйтесь по официальным ссылкам.

## Инференс-стек (self-host)
| Инструмент | Назначение | Ссылка |
|---|---|---|
| vLLM | Прод-инференс по умолчанию | https://docs.vllm.ai/en/latest/ |
| SGLang | Prefix-heavy (RAG/agent-loops) | https://github.com/sgl-project/sglang |
| TensorRT-LLM | NVIDIA-only, пиковый throughput | https://github.com/NVIDIA/TensorRT-LLM |
| Ollama | Локальный прототип (не прод) | https://ollama.com/ |
| llama.cpp | CPU / edge / Mac | https://github.com/ggerganov/llama.cpp |

## Безопасность / guardrails
| Инструмент | Назначение | Ссылка |
|---|---|---|
| Presidio | PII detection / masking | https://github.com/microsoft/presidio |
| LiteLLM (PII masking) | Прокси-маскирование | https://docs.litellm.ai/docs/proxy/guardrails/pii_masking_v2 |
| NVIDIA NeMo Guardrails | Dialog/topical rails | https://github.com/NVIDIA/NeMo-Guardrails |
| Meta LlamaFirewall | Agent-firewall (OSS) | https://arxiv.org/html/2505.03574v1 |
| Garak | Red-team фаззер для LLM | https://github.com/NVIDIA/garak |
| TruffleHog / git-secrets | Secret-scanning | https://github.com/trufflesecurity/trufflehog |

## Стандарты / compliance
| Документ | Ссылка |
|---|---|
| OWASP Top 10 for LLM 2025 | https://genai.owasp.org/resource/owasp-top-10-for-llm-applications-2025/ |
| NIST AI 100-2 (Adversarial ML) | https://nvlpubs.nist.gov/nistpubs/ai/NIST.AI.100-2e2025.pdf |
| MITRE ATLAS | https://atlas.mitre.org/ |
| EU AI Act (timeline + penalties) | https://artificialintelligenceact.eu/ |
| NIST SP 800-122 (PII) | https://csrc.nist.gov/pubs/sp/800/122/final |

## Экономика / pricing
| Инструмент | Назначение | Ссылка |
|---|---|---|
| Artificial Analysis | Intelligence Index + blended cost 300+ моделей | https://artificialanalysis.ai/leaderboards/models |
| Anthropic Pricing | актуальные цены | https://www.anthropic.com/pricing |
| OpenAI Pricing | актуальные цены | https://developers.openai.com/api/docs/pricing |
| Gemini API Pricing | актуальные цены | https://ai.google.dev/gemini-api/docs/pricing |

## AI-gateway / routing
| Инструмент | Лицензия / хост | Ссылка |
|---|---|---|
| LiteLLM | MIT, self-host | https://github.com/BerriAI/litellm |
| OpenRouter | hosted SaaS | https://openrouter.ai/ |
| Portkey | Apache 2.0 / hosted | https://portkey.ai/ |
| Cloudflare AI Gateway | SaaS edge | https://developers.cloudflare.com/ai-gateway/ |
| RouteLLM | Apache 2.0 OSS (роутер) | https://github.com/lm-sys/RouteLLM |

> Helicone — Apache 2.0, но на момент составления в maintenance-режиме (для новых проектов проверяйте актуальность).

## Agent frameworks
| Framework | Лагерь | Ссылка |
|---|---|---|
| Claude Agent SDK | provider-native | https://docs.anthropic.com/ |
| OpenAI Agents SDK | provider-native | https://platform.openai.com/docs/ |
| Google ADK | provider-native | https://google.github.io/adk-docs/ |
| LangGraph | independent | https://langchain-ai.github.io/langgraph/ |
| CrewAI | independent | https://www.crewai.com/ |
| Pydantic AI | independent | https://ai.pydantic.dev/ |

## Fine-tuning
| Инструмент | Назначение | Ссылка |
|---|---|---|
| Unsloth | быстрый LoRA/QLoRA | https://github.com/unslothai/unsloth |
| QLoRA (paper) | база дешёвого дообучения | https://arxiv.org/abs/2305.14314 |
| DSPy | prompt-optimization (альтернатива дообучению) | https://github.com/stanfordnlp/dspy |
