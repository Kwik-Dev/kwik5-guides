---
title: The cloud
description: Credits, keys, what a refusal means, and the agent surface
order: 3
audience: author, agent
status: partly available — two tools are decided, not built
updated: 2026-11-13
---

# The cloud

The cloud does one thing the local install cannot guarantee: it always has a big GPU and it needs
nothing installed. Everything else — creating a project, writing pages, building the PSD, checking
assets, running the Simulator — is free and local (D2).

## Credits and keys

| | |
| --- | --- |
| 1 credit | **$0.10** |
| free tier | **30 credits, once per customer** (D16) — not monthly, not per key |
| top-up | <https://kwiksher.com/credits> |
| identity | your **Polar licence key**. There is no account and no password: the key bought the credits, so the key is the identity |

```bash
kwik cloud login          # hidden prompt; stores the key 0600 and prints a fingerprint only
kwik cloud credits        # balance + the price list
kwik cloud status         # endpoint, key fingerprint, last job; exit 3 if not signed in
```

The key is never written to a log, a command line, a commit or a TUI frame. If you ever see a key in
one of those, treat it as compromised and rotate it.

## Prices

| tool | unit | price | notes |
| --- | --- | --- | --- |
| `cast_render` | 1 image | **2 cr** | Z-Image Turbo + your Cast LoRAs, 1536×1024 |
| `tts_speak` | started minute | **1 cr** | Kokoro voices; narration of one page ≈ 1 minute |
| `music_bed` | started minute | **1 cr** | a 30 s bed = 1 credit; no floor | **decided, not built** |
| `video_clip` | second of output | **5 cr** | 6 s default, 15 s hard cap, 60 s per job | **no worker yet** |
| `cast_train` | 1 LoRA | **40 cr** | minutes on a big card — the canonical cloud job (D12) | **no worker yet** |
| `upscale` | 1 image | **1 cr** | the *cloud* engine; `lanczos` local is free | **decided, not built** |
| `job_status`, `credits` | — | free | |

Rounding is always up: a partial unit is a full unit, which is how every metered service works and
what the CLI prints.

## What a refusal looks like

A refusal is a **decision, not an error to retry**:

```json
{ "error": "insufficient_credits",
  "message": "This job needs 2 credits; 0 available.",
  "credits_available": 0, "estimated_cost": 2,
  "top_up_url": "https://kwiksher.com/credits", "retry_after": 0 }
```

HTTP `402`. Never `403`, never `500`. Over MCP the same body arrives as a **tool result** with
`isError: true` and HTTP 200, because that is where MCP puts "the tool ran and refused".

| status | meaning | what to do |
| --- | --- | --- |
| `400` | bad request — missing `Idempotency-Key`, unknown tool, bad unit, over a duration cap | fix the request; nothing was charged |
| `401` | missing or invalid Polar key | `kwik cloud login` |
| `402` | not enough credits | read `estimated_cost` and `top_up_url`; do not loop |
| `404` | unknown job (or a job that belongs to another customer) | check the id |
| `409` | same `Idempotency-Key`, different body | use a new key for different work |
| `503` | no GPU capacity, or that tool has no worker yet | retry after `retry_after`; **the hold is already released** |

## Idempotency

Every submit needs an `Idempotency-Key`. The same key with the same body returns **the same job and
charges nothing** — that is what makes a retry safe. The same key with a *different* body is `409`,
not a silent replay, because those are different pieces of work.

Over MCP you never type a key: it is derived as `mcp:<sha256(name + arguments)>`, so a retried identical
call replays the same job instead of paying twice.

## Job states

```
reserved ──► queued ──► running ──► settled
    │           │           │
    └───────────┴───────────┴────► failed   (hold released, nothing charged)
```

A hold is real credits. It is released when the enqueue fails, when the job fails, and by the
reconciler when the API dies mid-flight — a cron pass every ten minutes that also cancels jobs still
queued after 20 minutes. **A job the runner reports as running is never cancelled by that timeout**:
the runner's own execution limit owns it.

## Agents: MCP and the skill file

| | |
| --- | --- |
| endpoint | `POST https://api.kwiksher.com/mcp` (JSON-RPC: `initialize`, `tools/list`, `tools/call`) |
| auth | the same Polar key as a bearer token |
| discovery | `GET /v1/tools.json` — machine-readable; `GET /v1/skill` — the Markdown an agent reads |
| local facade | `kwik mcp` over stdio lists the cloud tools and proxies them, so an agent sees one toolbox and does not need to know which tool is paid |

Both discovery documents are **generated from one catalogue**, so neither can describe a tool that no
longer exists. A tool with no worker deployed must answer `tool_unavailable` with the hold released,
not a `500` — that is the current known defect: nothing throws it yet, so today an unsupported tool
would be enqueued to the render endpoint.

## What the cloud never receives

Only the scene, the cast registry, a size, and an output key or a pre-signed destination. Never your
manuscript, never the book tree, never a customer key. Logs carry `job_id`, units, timings and hashes —
**no prompt, no caption, no OCR text** (plan §6). Objects expire at 30 days; uploaded sources at 24
hours; your Cast LoRAs stay while the account is active and are deleted on request within 24 hours.
Your art is never used to train anything.
