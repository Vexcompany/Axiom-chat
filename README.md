# Ryuna

Ryuna is a public AI assistant born from PAGASKA. Its primary identity and knowledge domain are PAGASKA, but it is **not** restricted to PAGASKA topics. It should behave as a general personal AI assistant for each user while retaining PAGASKA as its foundational context.

This repository is the source of truth for the product direction. Future AI coding agents must read this document before implementing features and must preserve the architecture and product decisions below.

## Product identity

The frontend exposes four model choices:

- **Ryuna** — the normal/default assistant. Fast, practical, broad usage allowance.
- **Ryuna Ritra** — stronger reasoning/research-oriented assistant. More capable for difficult analysis and research.
- **Ryuna Vuga** — a distinct model variant for specialized workloads.
- **Ryuna Yura** — the most expensive/heavy reasoning option. Usage must be tightly controlled. It may eventually have a small free allowance, but it must never be treated as unlimited.

The variant names are product identities and should be treated as names in their own right. Their private/personal naming origins must not be exposed or documented in the public product or repository.

Underlying models/providers are implementation details and must not be exposed as model choices in the normal Ryuna UI.

## Provider discovery checkpoint

The current provider strategy is still in the **discovery/research phase**. Do not prematurely build a large provider router or commit to every service below. First verify which services are genuinely usable, free, stable, and suitable for a public application.

### Provider/platform candidates already identified

- FreeLLMAPI
- XKiro
- No-Cost-AI (provider/service discovery catalog, not itself a provider)
- BAI
- Caven / Caventra
- Google AI Studio / Gemini API
- Groq
- Cerebras
- OpenRouter
- Cloudflare Workers AI
- NVIDIA NIM
- Hugging Face Router
- NavyAI
- Mistral
- Cohere
- Z.ai / Zhipu
- OpenCode Zen
- ModelScope
- LLM7
- Puter
- Pollinations
- AnyAPI
- UncloseAI
- Ollama / OllamaFreeAPI
- DeepInfra
- Together AI
- Replicate
- Fireworks AI
- IBM
- Anthropic API
- OpenAI API

This is a **research inventory**, not a claim that every entry is free, production-ready, or approved for Ryuna. Availability, pricing, quotas, terms, model catalogs, and API compatibility must be verified before integration.

### Free-model discovery shortlist

The following is a research shortlist of models/endpoints that are currently free, free-tier, or explicitly free-labeled. **Free does not mean unlimited**, and some entries require signup, have small quotas, are temporary, or are intended only for testing. Re-check provider pricing before integration.

| Provider | Free / free-tier model candidates | API-key farming priority |
|---|---|---|
| **FreeLLMAPI** | Free catalog/router; model availability changes dynamically | **High — verify one key's actual quota first** |
| **XKiro** | Models shown as $0/free on its lowest-price dashboard; exact free set must be rechecked dynamically | **High** |
| **BAI** | Free models exposed through its key/API service; exact catalog/limits still need verification | **Medium** |
| **Caven / Caventra** | Free model/API candidates; exact model catalog and quota still need verification | **Medium** |
| **Google AI Studio / Gemini** | Gemini 3.7 Flash, Gemini 3.6 Flash, Gemini 3.5 Flash-Lite, Gemini 3.1 Flash-Lite, Gemini 2.5 Pro, Gemini 2.5 Flash, Gemini 2.5 Flash-Lite and other explicitly free-tier entries | **Very High** |
| **Groq** | Free-plan hosted models include GPT-OSS 120B/20B, Qwen 3.6/3.8 27B, Compound/Compound Mini and other eligible models; limits are per account/model | **Very High** |
| **Cerebras** | Free API tier for selected hosted open models; exact current free catalog must be checked from the live model list | **Very High** |
| **OpenRouter** | Large rotating `:free` catalog plus `openrouter/free`; examples include Hy3, Nemotron 3 Ultra, Laguna M.1 and many other free endpoints | **Very High** |
| **Cloudflare Workers AI** | Free Workers allocation: 10,000 neurons/day; examples currently available on Free include GLM-4.7-Flash, Gemma 4 26B and Nemotron 3 120B; some newer frontier models require Paid | **High** |
| **NVIDIA NIM** | Free endpoints/downloadable free endpoints include Nemotron 3.5 Lightning 30B, Muse Glimmer 30B, Inkling, Laguna XS 2.1, GLM-5.2, MiniMax M3, Gemma 4 31B and others | **Very High** |
| **Hugging Face Router** | Free monthly inference credit is currently $0.10 for Free users; many models/providers are available but this is a tiny free pool | **Low** |
| **NavyAI** | Free/non-premium models can be discovered dynamically from `/v1/models`; API metadata exposes `premium` and `required_plan` | **High** |
| **Mistral** | Free Studio/API mode with no card required; free usage is rate-limited and applies to eligible Mistral models | **High** |
| **Cohere** | Free trial key; all Cohere models/APIs can be tested, but trial usage is rate-limited | **Medium** |
| **Z.ai / Zhipu** | Free/promotional model access may appear through Z.ai, OpenRouter and other gateways; verify current direct-API free quota before farming | **Medium** |
| **OpenCode Zen** | Big Pickle, MiMo-V2.5 Free, Hy3 Free, Nemotron 3 Ultra Free, Nemotron 3.5 Lightning Free, Muse Spark 1.2 Contributor Free | **Very High** |
| **ModelScope** | Free hosted/inference model catalog exists; exact public API quota/model availability needs live verification | **Medium** |
| **LLM7** | Free API catalog with multiple model families; exact current limits/models need live verification | **High** |
| **Puter** | Hundreds of LLMs through Puter.js without API keys; includes GPT, Claude, Gemini, Grok, DeepSeek, Qwen, Mistral, Gemma, Llama and more | **Do not farm keys — keyless** |
| **Pollinations** | Free/public model catalog and OpenAI-compatible endpoints; model availability and zero-price status vary by model/provider | **High, but verify auth model** |
| **AnyAPI** | Free API access reported for multiple open models such as Llama, Qwen Coder, QwQ, Gemma, Nemotron, Mistral, DeepSeek and GPT-OSS; quota must be verified | **High** |
| **UncloseAI** | Hermes AI, Qwen 3 Coder and TTS Speech listed as free, unlimited, OpenAI-compatible in the no-cost catalog; independently verify reliability/terms | **High** |
| **Ollama / OllamaFreeAPI** | Huge/local model catalog; Ollama is primarily local/self-hosted, while OllamaFreeAPI is a separate free endpoint with uncertain public quota | **Low for keys; high for self-hosting** |
| **DeepInfra** | Usually trial/credit-based rather than a durable unlimited-free tier; verify current promotional/free credits before use | **Low** |
| **Together AI** | Promotional/free credits may be available for new accounts; not treated as a permanent free provider until verified | **Low** |
| **Replicate** | Promotional/free credits may be available depending on account; usage is generally paid after credits | **Low** |
| **Fireworks AI** | Promotional/free credits may be available for new accounts; not treated as permanent free quota | **Low** |
| **IBM / watsonx** | Free trial/sandbox credits may exist, but not treated as a permanent free API pool | **Low** |
| **Anthropic API** | No permanent public free API pool; trial/promotional access may vary | **Do not farm** |
| **OpenAI API** | No permanent public free API pool; promotional credits may vary | **Do not farm** |

### Important interpretation

This table is for **provider discovery and capacity planning**, not a recommendation to create accounts in violation of provider terms. Do not mass-create accounts, bypass identity/payment controls, evade rate limits, or rotate credentials to defeat quotas. If a provider permits multiple legitimate keys/accounts for a project or team, use that supported mechanism.

For Ryuna, the useful number is not simply "how many API keys can we farm?". First measure the **quota per legitimate account/key**, then estimate how many legitimately obtained credentials are needed for the desired traffic.

A rough planning formula is:

```text
required credentials ≈ peak required capacity / usable capacity per credential
```

Use a safety margin rather than trying to consume 100% of a provider's quota.

### Current high-value verification targets

For legitimate account creation and testing, prioritize:

1. **Google AI Studio / Gemini**
2. **Groq**
3. **Cerebras**
4. **OpenRouter free models**
5. **NVIDIA NIM free endpoints**
6. **OpenCode Zen free-labeled models**
7. **Mistral free API mode**
8. **Cloudflare Workers AI free allocation**
9. **NavyAI free/non-premium models**
10. **LLM7 / AnyAPI / UncloseAI** after verifying current limits and public-use terms
11. **Puter / Pollinations** as keyless or alternative capacity sources where their terms permit the intended use

### Discovery sources

Useful discovery sources currently include:

- FreeLLMAPI model catalog
- XKiro model dashboard
- `zebbern/no-cost-ai` GitHub repository
- BAI API-key page
- Caven / Caventra
- OpenCode Zen model/pricing documentation

The `no-cost-ai` repository should be treated as a discovery list. It contains a mixture of chat services, APIs, platforms, and other free AI resources; entries must be independently verified before use.

### Evaluation checklist

For each candidate provider, verify:

1. Whether access is genuinely free and whether a payment card is required.
2. Request, token, daily, monthly, and concurrency limits.
3. Available models and model families.
4. Whether the API is OpenAI-compatible or requires a custom adapter.
5. Streaming support.
6. Vision/multimodal support.
7. Tool/function-calling support.
8. Reliability and rate-limit behavior.
9. Terms that allow use by a public application.
10. Credential/security requirements.

Only after this discovery checkpoint should Ryuna select a smaller set of providers for actual integration.

## Provider pool and API-key rotation

Ryuna will aggregate many providers and many accounts/keys. The backend should eventually treat each credential as a managed resource, not as a static environment variable.

Conceptually:

```text
Ryuna Model Router
  |
  +-- Ryuna provider pool
  +-- Ritra provider pool
  +-- Vuga provider pool
  +-- Yura provider pool
  +-- Tool providers
```

A credential should support health/state tracking such as `HEALTHY`, `DEGRADED`, `RATE_LIMITED`, `QUOTA_EXHAUSTED`, `INVALID`, `DISABLED`, and optionally `PROBING` during recovery.

If a credential is known to be rate-limited or quota-exhausted, the next request must not retry that credential. It should be removed from the active pool until it is eligible for recovery/probing.

## AI-to-AI tools

Ryuna should use other AI systems as internal tools rather than exposing every provider/model to users. The final response remains Ryuna's response, informed by internal tool results.

## Context and token efficiency

Ryuna must not blindly resend an ever-growing conversation history. Use recent raw context, compact summaries for older context, selective retrieval, and compact tool-result storage to control token usage and latency.

## Usage tracking

Usage tracking belongs primarily in the backend because the backend controls provider calls and actual consumption. An internal/admin UI is planned for provider/account health, quota/rate-limit state, errors, and usage.

## Pterodactyl execution layer

A Pterodactyl-hosted node is planned as an execution layer for capabilities unsuitable for serverless environments, such as browser automation, long-running jobs, and heavier execution workloads. Do not route every request through Pterodactyl.

## Current implementation priority

1. Frontend chat foundation.
2. Backend API foundation.
3. Ryuna / Ryuna Ritra / Ryuna Vuga / Ryuna Yura model routing.
4. **Provider discovery and verification checkpoint.**
5. Provider abstraction and credential pools.
6. Automatic health tracking and immediate key rotation.
7. Conversation/context management and token optimization.
8. Tool/event protocol and frontend progress UI.
9. Web search and vision delegation.
10. Connector architecture.
11. Backend usage tracking and admin monitoring.
12. Pterodactyl execution worker/router.
13. Telegram alerting.

Do not prematurely implement future infrastructure if an earlier checkpoint is unstable.

## Non-negotiable product rules

- Only Ryuna, Ryuna Ritra, Ryuna Vuga, and Ryuna Yura are visible as model choices.
- Underlying provider/model names are implementation details.
- Ryuna should not silently switch the user's selected model because a request is difficult.
- Yura is expensive/heavy and must have a strict usage limit.
- Provider API keys stay in the backend/server environment, never in the public frontend.
- Exhausted/rate-limited credentials are removed from the active pool immediately; do not retry them on every request.
- AI providers can be used as internal tools and their intermediate calls should normally remain invisible to the user.
- Public connectors require explicit user connection/authorization.
- Optimize conversation context to control token usage and latency.
- Pterodactyl is an execution layer, not the destination for every request.
- PAGASKA is the origin and foundational domain of Ryuna, but Ryuna is a general personal assistant and must not be artificially limited to PAGASKA topics.