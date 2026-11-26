---
title: The club
description: What the membership is, what a seat includes, and how to check the claim yourself.
layout: page
---

> **Stage page.** This is the rehearsal build of a page that does not exist in production. It sits
> behind the beta gate, ships no live numbers, and says so rather than showing a placeholder that
> looks like data.

The club is capped at **100 seats**, and the cap is not a marketing number: it is the number of
people one operator can answer within the two business days the support page already promises. Each
seat includes monthly cloud credits, a warm render endpoint that skips the cold start, Cast LoRA
training, and the book's seal on everything the member ships.

## What we claim, and how to disagree with us

Two claims are checkable from public files, with no account and no software of ours:

| Claim | Where it comes from | How to check it |
| --- | --- | --- |
| seats are capped | the number of active memberships | ask for the count; the cap is enforced by the grant job, not by a banner |
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

**No numbers yet.** No anchor has been published from this environment, so this page shows none. A
date, a count and a root will appear here once the nightly anchor has run once — and a page that
invented them would be worse than this one.

## What the membership is not

- Not a licence: everything the customer runs is free and open source (D2).
- Not required to use the cloud: a licence key and credits are enough, and an agent buying renders
  never touches a club page.
- Not a support tier: there is no SLA, no ticket system, no phone number. There is one person, a
  two-business-day promise, and 100 seats so that promise stays true.
