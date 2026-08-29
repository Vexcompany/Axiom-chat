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

### OpenCode Zen — currently free-labeled model candidates

OpenCode Zen currently lists these models with **Free** input/output pricing. They are available for a limited time, so their availability must be re-verified before production integration:

- **Big Pickle** — `big-pickle`
- **MiMo-V2.5 Free** — `mimo-v2.5-free`
- **Hy3 Free** — `hy3-free`
- **Nemotron 3 Ultra Free** — `nemotron-3-ultra-free`
- **Nemotron 3.5 Lightning Free** — `nemotron-3.5-lightning-free`
- **Muse Spark 1.2 Contributor Free** — `muse-spark-1.2-contributor-free`

Most of these use the OpenAI-compatible Chat Completions endpoint; Muse Spark 1.2 Contributor Free uses the Responses API. OpenCode Zen's overall service is pay-as-you-go, so only the models explicitly marked Free should be treated as free candidates. Free status is temporary and may change. Do not send personal or confidential data to free models where OpenCode's privacy notes warn that data may be logged or used for improvement.

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