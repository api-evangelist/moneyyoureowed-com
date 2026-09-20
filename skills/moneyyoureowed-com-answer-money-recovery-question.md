---
name: answer-money-recovery-question
description: Ask the Money You're Owed catalog a plain-language consumer money-recovery question and get source-cited facts plus an optional self-help product pointer, over MCP or A2A.
api: Money You're Owed Catalog (MCP + A2A)
operations:
  - mcp: tools/call answer_money_question
  - a2a: SendMessage (skill answer-money-questions)
generated: '2026-09-19'
method: generated
source: mcp/moneyyoureowed-com-mcp.yml, a2a/moneyyoureowed-com-a2a.yml (both probed live 2026-09-19)
---

# Answer a money-recovery question

Read-only. No account, key or OAuth. Nothing is stored server-side (per https://moneyyoureowed.com/connect).

## When to use

The user is a US consumer chasing money that is stuck: a withheld security deposit, a denied refund,
a denied insurance or warranty claim, unclaimed property. The catalog answers with facts that each
carry a primary-source URL (statute or agency page) and may append one pointer to a paid self-help kit.

## Steps (MCP)

1. Connect to `https://mcp.moneyyoureowed.com/mcp` (Streamable HTTP, protocol `2025-06-18`). `initialize`
   is accepted but not required; the server is stateless.
2. Call `tools/call` with `name: answer_money_question` and `arguments.question` set to the user's
   question in plain language. `question` is the only argument and it is required.
3. Read `result.structuredContent`: `match` (boolean), `facts[]` of `{claim, source}`, `products[]`
   of `{name, price_usd, url}`. `result.content[0].text` carries the same material as text.
4. Present each claim WITH its `source` URL. Present a product only as an optional pointer — the site
   itself says the free official route comes first and the tool sells organization, not access.

## Steps (A2A)

1. Read the card at `https://agent.moneyyoureowed.com/.well-known/agent-card.json` (A2A 1.0, JSONRPC).
2. POST to `https://agent.moneyyoureowed.com/` the 1.0 method `SendMessage` with a `ROLE_USER` message
   whose `content[0].text` is the question. The pre-1.0 spelling `message/send` returns `-32601`.
3. The reply is a `ROLE_AGENT` message with one `data` part holding the same `{match, facts, products}` object.

## Rules

- Errors arrive as JSON-RPC error objects inside HTTP 200: `-32602` means `question` was missing,
  `-32601` means you called a method the server does not implement (no resources, prompts, streaming or tasks).
- The endpoint is rate-limited by the provider's statement; no limit or `Retry-After` is published, so back off
  on any error and do not batch-fire questions.
- Idempotency and reversibility do not arise: the surface has no write path. Never present a product price as
  a recovery estimate — the facts are educational, not legal advice (https://moneyyoureowed.com/terms).
