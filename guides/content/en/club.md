---
title: The club
description: What the membership is, what a seat includes, and how to check the claim yourself.
layout: club
---

> **Stage page.** This is the rehearsal build of a page that does not exist in production. It sits
> behind the beta gate. Seats, books and member fingerprints are read from the public club index
> (`GET /v1/club/index`); the published-day counts are read from the anchor manifests. Both are fetched by
> `deploy/club-claim.py`, so nothing on the page is typed — and when there is nothing to count it says so
> instead of showing a placeholder that looks like data.

The page shows the member index itself: **seats in use** (distinct holder keys listed), **books** (one row
per listed book), and the **16-character fingerprint** of each key a member chose to publish. It is generated
from `GET /v1/club/index`, which anyone can read without an account — so the count is checkable rather than
believed. A member who runs `kwik provenance withdraw` disappears from it, and their book and its receipts
keep verifying.

The club is capped at **100 seats**, and the cap is not a marketing number: it is the number of
people one operator can answer within the two business days the support page already promises. Each
seat includes monthly cloud credits, a warm render endpoint that skips the cold start, Cast LoRA
training, and the book's seal on everything the member ships.

## What we claim, and how to disagree with us

Two claims are checkable from public files, with no account and no software of ours — the member index and the
anchor manifests:

| Claim | Where it comes from | How to check it |
| --- | --- | --- |
| seats are capped | `GET /v1/club/index` — distinct holder keys, capped at 100 | read the index yourself: `curl https://kwik5-cloud-api-stage.kwiksher.workers.dev/v1/club/index`. The 101st distinct key is refused with `409 club_full` |
| books shipped, all verifiable | the daily anchor manifests | fetch `/.well-known/kwik-anchors/<day>.json` from the cloud API and count the entries |

The second one is the one nobody can fake. Each manifest lists the 32-byte leaves submitted that
day and the Merkle root computed over them:

```bash
curl -s https://kwik5-cloud-api-stage.kwiksher.workers.dev/.well-known/kwik-anchors/2026-11-26.json
```

That document is content-free by construction: a leaf is a digest of a document that never leaves
the author's machine, so a reader learns *that* a book was sealed on a day — never which book,
whose, or what is in it. If you hold a published book, you can hash it and find its leaf; that is
the point of a timestamp.

The table below is generated, not typed, and it mixes two public sources: **seats, books and the member
fingerprints** come from `GET /v1/club/index`; the **published days and their roots** come from the anchor
manifests. The **cap of 100** is a business number in the page's own tool, not a read. If the table shows
nothing, nothing has been published — the page never fills the gap with a number it cannot show the working
for.

## What the membership is not

- Not a licence: everything the customer runs is free and open source (D2).
- Not required to use the cloud: a licence key and credits are enough, and an agent buying renders
  never touches a club page.
- Not a support tier: there is no SLA, no ticket system, no phone number. There is one person, a
  two-business-day promise, and 100 seats so that promise stays true.
