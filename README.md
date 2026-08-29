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

Underlying models/providers such as Llama, Mistral, Gemini, Groq, Cerebras, OpenRouter models, Cloudflare AI, etc. are implementation details and must **not** be exposed as model choices in the normal Ryuna UI.

The model selector is explicit: Ryuna does not silently switch the user's selected model merely because a question is difficult. Instead, when a request would benefit substantially from stronger reasoning/research, Ryuna can explain that Ritra/Yura would be more suitable. Do not add accidental one-click upgrade buttons such as "Use Ryuna Ritra" or "Use Ryuna Yura" because an accidental click could consume the user's limited usage.

## Two repositories / separation of concerns

The intended production architecture is split into:

1. **Frontend** — public Ryuna chat UI. It calls the backend and contains no provider API keys.
2. **Backend** — routing, provider pools, credentials, model orchestration, tools, usage accounting, context management, and execution routing.

Never place provider API keys in frontend code. The frontend should not need to know whether a response came from Gemini, Groq, Mistral, etc.

## Provider pool and API-key rotation

Ryuna will aggregate many providers and many accounts/keys. The backend should treat each credential as a managed resource, not as a static environment variable.

Conceptually:

```text
Ryuna Model Router
  |
  +-- Ryuna provider pool
  |     +-- Account A
  |     +-- Account B
  |     +-- Account C
  |
  +-- Ritra provider pool
  |
  +-- Vuga provider pool
  |
  +-- Yura provider pool
  |
  +-- Tool providers
        +-- Gemini
        +-- other providers
```

### Credential states

A credential should support health/state tracking such as:

- `HEALTHY`
- `DEGRADED`
- `RATE_LIMITED`
- `QUOTA_EXHAUSTED`
- `INVALID`
- `DISABLED`
- optionally `PROBING` during recovery

The router must inspect provider HTTP status, error codes/messages, retry hints, and provider-specific quota signals.

### Critical rotation rule

If Account A is known to be rate-limited or quota-exhausted, the **next request must not try Account A again**. It should immediately select a healthy account, e.g. Account B, and Account B becomes the effective primary while A is unavailable.

Do not implement slow fallback chains such as:

```text
A -> error -> retry -> timeout -> B
```

Instead:

```text
A -> quota/rate-limit signal
  -> mark A unavailable
  -> B becomes primary
  -> next request goes directly to B
```

When a cooldown expires, a previously limited key may enter `PROBING`. A small request can determine whether it is healthy again. If probing fails, keep it out of the active pool.

Do not assume every 429 means permanent quota exhaustion. Distinguish temporary rate limits from longer quota exhaustion and invalid credentials.

The system does not need exact remaining-token data when a provider does not expose it. Actual provider responses are the source of truth for health transitions.

## AI-to-AI tools

Ryuna should use other AI systems as internal tools rather than exposing every provider/model to users.

Example:

```text
User -> Ryuna
          |
          +-> Gemini (web search)
          |       |
          |       +-> search/research result
          |
          +-> Ryuna synthesizes the result
                    |
                    +-> User
```

The intermediate delegation should normally be invisible as a separate chat message. The final answer remains Ryuna's answer, informed by the tool result.

### Web search

Web search is an important tool. Ryuna may delegate a search task to a suitable provider such as Gemini. The user's conversation should not become a visible conversation with Gemini; Gemini is an internal tool/provider.

### Vision

When a user attaches an image, Ryuna should delegate visual interpretation to the strongest/most suitable vision provider available (Gemini is a likely provider).

The vision result must become part of the ongoing conversation context.

## Thinking / progress UI

Long-running AI-to-AI calls can take time. The frontend should show meaningful progress events rather than generic loading dots.

The actual wording can be generated by the application and should be natural. Do not expose internal secrets, raw provider prompts, API keys, or unnecessary implementation details.

Tool events should also have proper UI representations. For example, when a user connects GitHub, the UI can show a tool event such as **Menghubungkan tools GitHub**. The frontend should use clean SVG/iconography rather than decorative emoji in the actual product UI.

## Connectors

Ryuna is public and intended for PAGASKA users. GitHub and other powerful integrations should not be automatically available as unrestricted public tools.

Instead, integrations belong under a user-controlled **Connections/Connectors** area. A user may explicitly connect an external service and then Ryuna can use that connection according to the granted permissions.

The normal public tool set should therefore remain safe and useful without exposing the owner's private development integrations.

## Context and token efficiency

Ryuna must not blindly resend an ever-growing conversation history on every request.

A target such as roughly 50-60 recent turns may be used as a starting point, but the architecture should optimize beyond simple truncation.

Preferred strategy:

1. Keep a recent-window of raw messages for immediate continuity.
2. Maintain a compact conversation summary for older context.
3. Store important user-provided facts/decisions as structured conversation memory only when useful.
4. Store tool outputs in compact structured form rather than repeatedly embedding huge raw responses.
5. Retrieve older context selectively when relevant instead of always sending everything.
6. Keep the core system prompt stable and avoid duplicating unnecessary instructions in every layer.
7. Compress/summarize long tool outputs before putting them into durable conversation context.

The goal is to preserve continuity while minimizing token leakage and latency.

## Usage tracking

Usage tracking belongs primarily in the backend because the backend controls provider calls and actual consumption.

The frontend can display usage information returned by the backend, but it must not be the authority for quota accounting.

An internal/admin UI is planned to show things such as:

- provider/account health
- active/primary credential
- rate-limit state
- quota-exhausted state
- cooldown/reset information when available
- request counts
- provider errors
- overall usage

This admin UI can eventually live alongside the backend/admin application without exposing secrets to public users.

## Pterodactyl execution layer

A Pterodactyl-hosted Ubuntu/Universal node is planned as an execution layer for capabilities that are unsuitable for serverless environments such as Vercel or Cloudflare Workers.

Examples include:

- Playwright/browser automation
- long-running jobs
- serverful tooling
- heavier execution workloads
- tools that require a persistent process or browser runtime

Do **not** route every request through Pterodactyl. Lightweight API/routing work should stay on the appropriate serverless/backend platform. A dedicated execution router should decide where a task belongs.

Conceptually:

```text
Ryuna Backend
   |
   +-- lightweight task -> serverless/backend
   |
   +-- browser/heavy/long-running task -> Pterodactyl worker
```

## Sidebar / navigation

The public Ryuna UI should follow the familiar AI-assistant pattern. The burger/sidebar navigation should eventually contain items such as:

- New Chat
- conversation history
- Connections / Connectors
- Tools
- Settings
- Donate

The exact visual design can evolve, but the information architecture should remain simple and mobile-friendly.

## Donations / infrastructure

Ryuna is intended to be public for PAGASKA users. Infrastructure such as Pterodactyl hosting and external AI/provider usage can cost money. A Donate entry therefore belongs in the product navigation as a legitimate infrastructure-support feature.

Do not let donation UI interfere with normal chat functionality.

## Future notifications

Telegram notifications are planned for a later checkpoint, not the initial implementation.

Eventually the backend can notify the maintainer when credentials require attention, for example:

```text
Provider: Gemini
Account: A
Status: INVALID
Error: invalid API key
```

Other useful alerts may include quota exhaustion or repeated provider failures. This should be added only after the core Ryuna checkpoints are stable.

## Current implementation priority

Build in this general order and verify each checkpoint before expanding scope:

1. Frontend chat foundation.
2. Backend API foundation.
3. Ryuna / Ryuna Ritra / Ryuna Vuga / Ryuna Yura model routing.
4. Provider abstraction and credential pools.
5. Automatic health tracking and immediate key rotation.
6. Conversation/context management and token optimization.
7. Tool/event protocol and frontend progress UI.
8. Web search delegation.
9. Vision delegation and persistent tool-result context.
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
- Ryuna and Ryuna Ritra can have substantially more generous usage.
- Provider API keys stay in the backend/server environment, never in the public frontend.
- Exhausted/rate-limited credentials are removed from the active pool immediately; do not retry them on every request.
- AI providers can be used as internal tools and their intermediate calls should normally remain invisible to the user.
- Tool results must remain available as conversation context when later turns depend on them.
- Public connectors require explicit user connection/authorization.
- The frontend should show useful progress/tool events during slow operations.
- Avoid emoji in the actual frontend product UI; use SVG/icons instead.
- Optimize conversation context to control token usage and latency.
- Pterodactyl is an execution layer, not the destination for every request.
- Telegram monitoring is a later feature.
- PAGASKA is the origin and foundational domain of Ryuna, but Ryuna is a general personal assistant and must not be artificially limited to PAGASKA topics.

## Status

This file describes the intended architecture/product direction. Update this document when a major architectural decision is intentionally changed; do not silently diverge from it.