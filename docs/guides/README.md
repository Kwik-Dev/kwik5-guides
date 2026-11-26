---
title: Guides
description: Install, set up, and ship a book with Kwik — and the specs behind every step
order: 0
audience: authors, agents, operators
updated: 2026-11-13
---

# Guides

These pages are the source for the future `/guides` section of kwiksher.com (see
[`../new-site-plan.md`](../new-site-plan.md) §37 — *"one landing page + pricing/checkout + downloads + a
small amount of docs"*). They are written as front-matter Markdown so they can be dropped into the
Astro content collection unchanged when `site/` exists.

They are also the **specification in human form**: every number here comes from a decision, and each
page says which one.

## Read in this order

| page | what it answers | audience |
| --- | --- | --- |
| [01 — Install](01-install.md) | what to install, what it costs in disk and time, and how to install **nothing at all** | author |
| [02 — Your first page](02-first-book.md) | one page from nothing: create, render, narrate, score, run — with the price shown before each spend | author |
| [03 — The cloud](03-cloud.md) | credits, keys, what a refusal means, and the agent surface (MCP + skill file) | author, agent |
| [04 — The local backend](04-local-backend.md) | Guaardvark + ComfyUI on your own machine, native or Docker, and what each device can and cannot do | power user |
| [05 — Media specs](05-media-specs.md) | the numbers: frames, formats, codecs, alpha, durations, sizes, retention | author, designer, agent |
| [06 — Troubleshooting](06-troubleshooting.md) | what each exit code and error body means, and the one thing to do next | everyone |
| [07 — What runs where](07-what-runs-where.md) | every tool, on your machine or in the cloud, what it costs, and whether it is built — the status column is the truth | author, agent |
| [08 — The book's seal](08-provenance.md) | a verifiable seal on a finished book: what it proves, what it does not, who signs which file, **what Kwik actually stores (one 32-byte hash — there is no seal database)**, the colophon page (奥付) that prints it, use cases for authors and for Kwik, and why no wallet is involved. **available on stage** — every verb runs, including `publish`/`withdraw` and the public member index at `/v1/club/index` | author, designer, agent |

## The one rule these pages follow

**They never claim a capability that has not been run.** Every page separates what exists from what
is decided:

| label | meaning |
| --- | --- |
| **Available** | shipped and exercised; the command printed here was run |
| **Decided, not built** | agreed in `decisions.md` (D-numbered) with no implementation yet — a spec, not a feature |
| **Open** | a real question with no answer; do not build on it |

If a page says "Available" and the command fails, that is a bug in this documentation and it should be
fixed the same day — a stale guide is worse than no guide, because an author follows it and concludes
the product is broken.

## Decisions behind these pages

| decision | what it settles |
| --- | --- |
| **D2** | everything you run is OSS and works offline; no licence gates |
| **D3 / D13 / D15 / D17** | the only paid thing is cloud GPU work: render 2 cr/image, narration 1 cr/started min, music 1 cr/started min, video 5 cr/s, Cast training 40 cr, upscale 1 cr/image; 1 credit = $0.10 |
| **D5** | publishing goes to *your* host; we host nothing |
| **D14** | local generation is first-class; the cloud is acceleration, and it never spends money without saying so |
| **D16** | 30 free credits, once per customer |
| **D18** | the pipeline runs from `kwik` with no Python and no Adobe software |

The full text is [`../decisions.md`](../decisions.md), and the media contract is
[`../media-requirements.md`](../media-requirements.md). **If a guide contradicts either, the decision
wins and the guide is wrong.**
