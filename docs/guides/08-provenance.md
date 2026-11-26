---
title: The book's seal
description: A verifiable seal on a finished book — what it proves, what it does not, what Kwik actually stores (one hash), and what it is for on both sides
order: 8
audience: author, designer, agent
status: available on stage — build, verify, link, colophon, attest, the leaf submission and the daily anchor all run against the stage cloud
updated: 2026-11-25
---

# The book's seal

> **Available on stage.** `kwik provenance build`, `verify`, `link`, `unlink`, `colophon`, `attest`
> and `anchor` all exist; two real projects are sealed (`kwik5-project-template`,
> `biscuit-comes-home`), and the cloud's half runs against the stage deployment with an ES256 key.
> The daily anchor runs too: two days are published (`2026-11-25`, `2026-11-26`), and the
> [club page](https://kwiksher.com/cloud-sandbox/club/) counts them from the manifests themselves.
> The bucket rule that keeps a published manifest past seven days is set (2026-11-26), and the nightly
> job writes an OpenTimestamps proof beside each manifest — it ran for the first time that night, so #17 is
> closed. The one dated loose end: objects written before the rule changed keep the old 7-day header until
> R2's migration sweep finishes (by 2026-11-27T06:21Z). The design is
> [`../provenance-plan.md`](../provenance-plan.md), the reasoning
> [`../membership-model.md`](../membership-model.md) §7.

A finished book is a pile of files: page images rendered somewhere, crops, a layered `page.psd`, and
an `ogv` per animation. Today nothing binds those files together, and nothing lets anyone check where
they came from. **The book's seal** is that binding: one small file, `book.provenance.json`, that
travels inside the book bundle, signed, and verifiable on a machine that is offline, has no account,
and has no software of ours installed beyond `kwik`.

## What it proves

| Proves | Does not prove |
| --- | --- |
| these exact bytes are the bytes that were produced (`sha256` per artifact) | that a human wrote or approved the book |
| which pipeline version and which tool produced each one | authorship, originality, or who owns the copyright |
| for a cloud render: that Kwik Cloud's GPU produced that file, on that job, at that time | that the input cast was licensed, or that a person in it consented |
| the assembled files came from those renders, not from somewhere else | that the book is any good |
| the set existed at or before a date (once it is anchored — see below) | anything about your manuscript, prompts, or scene text |

The one line to remember, and the one `kwik verify` prints: **provenance is not authorship.**

## Two issuers, and why that matters

Every artifact is signed by whoever actually produced it, so the seal says *who stands behind
what*:

| File | Produced by | Signed by |
| --- | --- | --- |
| page image (`art/<job>.png`) | the GPU in the cloud | **Kwik Cloud** — it can check its own record of that render |
| upscale, bleed crop, `page.psd`, `ogv`, `mp4` | your machine, running `kwik` | **you** — a device key created at `kwik setup` and never uploaded |

Your copy is the one that holds the whole graph, because your machine is what assembled the book. The
cloud's copy only claims what the cloud did. Neither can silently stand in for the other.

That split also makes a book *partly* verifiable, honestly: if you touch up a page in Photoshop after
the build, the chain breaks at that page and the seal says so instead of pretending otherwise.
Re-running the build records the new bytes and marks the edge as edited outside the pipeline.

## Three levels, and none of them needs a wallet

| Level | What you get | Needs a wallet? |
| --- | --- | --- |
| **seal only** | the bytes are the bytes, and who signed them. The date is *claimed* | no |
| **+ daily anchor** | the same, plus "this set existed at or before <date>": the nightly job Merkelises the day's leaves and publishes the manifest. Costs one 32-byte submission (`kwik provenance anchor`) | **no** |
| **+ public memo** | the same proof, in a place a wallet-native reader looks first | **no** |

An anchor is checked, not trusted: `--anchor` fetches the day's manifest, **recomputes** the Merkle
root from the manifest's own leaf list, and only then looks for your leaf. It exits `0` when your book
is in a published manifest and **`4`** when the day has no manifest yet — never `0` for a timestamp it
could not check.

"On-chain" here means **a timestamp, not a token**. A batch of seal hashes is fingerprinted into a
public timestamp once a day — by us, on a schedule — and the proof is published as a plain JSON file
anyone can fetch. You never hold, buy, stake, or pay gas for anything; there is no wallet in the
instructions because there is no wallet in the design. If every chain disappeared tomorrow, your
seal would still verify, and would still be signed.

## The page a reader actually sees: the colophon (奥付)

A JSON file is not something a reader opens. The seal's visible face is a **colophon page** — the
credits page at the back of the book, which can carry the seal:

```
        Biscuit Comes Home
        text and pictures · A. Author
        imprint · Kwik Studio, 2026

        rendered with the Kwik pipeline 0.9.3
        seal bk_9f3c… · 2026-12-01 · verify with `kwik verify`
```

Two things make this more than decoration:

- **Every word is yours.** Title, credit name, imprint, edition — you supply them. We never fill a
  page from our records, because we do not have your name.
- **The page is part of the seal.** `page/colophon` is an artifact in the provenance graph, so editing
  the colophon after the build breaks the chain exactly like editing a page image does.

It is also the one part of this feature that is **irreversible**: the page ships inside the book, to
your host, to a reader. Opting in prints a name; the only way back is a rebuild, and only for copies
not yet distributed. So it is a separate choice from linking, and the prompt says so.

In Japanese publishing terms the two are different objects, and the page shows the mark:
**奥付** is the colophon page, **落款** is the maker's seal on the work.

## What you run

All six exist. `build`, `verify`, `link`, `unlink` and `colophon` are local and offline; `attest`, `anchor`
and `verify --anchor` are the only ones that talk to the network, and they send hashes, never a file:

```bash
kwik provenance build .            # hash the tree, write book.provenance.json (your device key)
kwik provenance build . --include 'Photoshop/biscuit/**' --include 'Solar2D/**'
kwik verify book.provenance.json   # re-hash the files, check the signature
kwik verify book.provenance.json --no-files   # a reader on another machine: signature only
kwik provenance link .             # let the book count towards a membership (a public key)
kwik provenance unlink .           # withdraw it; the document is re-issued and re-signed

kwik provenance attest .           # the cloud's half: render hashes only, never a file
kwik provenance anchor .           # submit the 32-byte leaf for the daily timestamp
kwik verify book.provenance.json --anchor --base-url <deployment>   # find this leaf in a published manifest
kwik provenance colophon . --name "A. Author" --imprint "Kwik Studio"   # the credits page (奥付)
```

Verification is local and offline by default, and it prints the split rather than a single verdict:

```
signature   ok   (kwik cloud, key published at …/provenance-keys.json)
artifacts   43/43  (40 render · 3 assembly)
chain       intact (every assembly artifact derives from an attested render)
note        provenance is not authorship
```

## Why "seal" and not "stamp" or "receipt"

| word | why not |
| --- | --- |
| *receipt* | a receipt asserts a transaction between two parties. Your local signature has no counterparty, and 領収書 already means a payment receipt — the wrong idea, and in the wrong direction |
| *stamp* | postage, date stamps, and "rubber stamp" (unthinking approval). It is also what the timestamping tool does (`ots stamp`), so the word is taken by the *act*, not the artifact |
| **seal** | authentication without a transaction, and it has the exact counterpart in 落款 |

The technical names, if you are writing code against it: the file is the **provenance document**, each
signature is an **attestation** (local or cloud), and *seal* is what we call the result.

## Are seals collected?

**No. There is no seal database, and there must not be one.** The document lives in your book bundle;
a cloud signature comes back to you and the payload is not filed anywhere. What survives contact with
Kwik is a **32-byte hash** — which the daily timestamp publishes anyway, because that *is* what a
timestamp is.

The complete list of everything Kwik holds, without exception:

| What | Where it lives | How long | Why |
| --- | --- | --- | --- |
| **Your seal** (`book.provenance.json`) | **your book bundle only** | forever — it is yours | verification must work offline, and keep working |
| **The cloud's part** (`book.provenance.cloud.json`) | returned by the attest call, stored by your CLI | forever — yours | a signature is handed back, not filed |
| **One leaf hash per anchored book** (32 bytes) | the day's **public** manifest | forever, and public | it is simultaneously the anchor's input and its proof |
| The artifact hash we verified (`jobs.output_sha256`) | our ledger, beside the job row | with the row | it describes files we already hold for 30 days — no new exposure |
| An attestation counter (`customer`, `day`, `count`) | our ledger | 90 days | rate limiting. **A number, not content** |
| A **`holder` public key** in your document — only if you run `kwik provenance link` | **your document**, not a server | until you unlink | it is what makes your books countable, and it is a key, never your name. **No club index is built**: we hold no roster of members |
| A membership claim (`customer`, book hash) — only for member books | our ledger | until you revoke | a roster row, not a book |

### What Kwik cannot reconstruct, ever

Your manuscript, your artwork, your prompts, your scene text, your cast or subject names, your
licence key — none of it is in the seal, so none of it can be re-derived from anything we hold. The
leaf hash is a digest of a *content-free* document: knowing it tells you nothing about the book.

### What that still reveals — said plainly

Anyone who holds your **published** book can hash it and match it to a day's manifest, and can see how
many documents were anchored that day. That is the point of a timestamp, and it does not tell them
whose book it is — unless you opted into a holder key, in which case the link is yours, deliberate,
and per book.

### Opting out, and deleting

| You want | Do this | Effect |
| --- | --- | --- |
| no trace at all | never run `attest`, never run `anchor` | nothing exists anywhere but your disk — the service keeps no copy of the document, only the 32-byte leaf you choose to submit |
| stop linking to a membership | `kwik provenance unlink book.yaml` | the holder key is removed and the document re-issued; already-anchored days keep their hash and prove nothing. (There is no club index yet, so nothing is removed from one) |
| delete what we hold about you | ask | the roster/index entry goes; the public leaf cannot be un-anchored **and does not need to be** — it is a hash of a document we never had |

None of this requires an account, and none of it is a support ticket you have to justify.

## Use cases — for you, and for Kwik

### For you (author, illustrator, studio)

| Situation | What the seal does |
| --- | --- |
| **Delivering to a client or publisher** | proves the file they received is the file the pipeline produced, and which pipeline version made it — without sending them your project |
| **"This page looks wrong"** | one file instead of a job-id lookup: the seeds, the tool versions and the artifact hashes are all in it, so a take can be reproduced deliberately |
| **A designer touches up pages** | the chain says which pages came from the pipeline and which were edited outside it, so a mixed book is *honest* rather than unverifiable |
| **Credits** | the colophon page (奥付) puts your name, imprint and the seal in the book itself — attribution a reader can see |
| **Coming back in two years** | tells you which pipeline version, toolchain and seeds produced the book, so a reprint or a new edition starts from known ground |
| **Schools, libraries, tenders** | a procurement file can be checked by the buyer without trusting you or us |
| **Portfolio / club eligibility** | optionally link books to a public key so your body of work is countable — and withdrawable |

What you give up: nothing, by default. A published book is already public; a hash of it adds no
information to anyone who has the book.

### For Kwik

| Situation | What it buys |
| --- | --- |
| **Support** | the customer's own evidence replaces our lookup. Cheaper triage, and the answer to "is that really what we rendered" is checkable in both directions |
| **The club's claim** | *"100 seats · 340 books shipped · every one verifiable"* — countable from public manifests, unfakeable, and it costs us the storage of a hash |
| **Eligibility without surveillance** | a membership gate that wants "has shipped work" gets evidence (book hash + live URL) instead of storing anyone's books (`../membership-model.md` §2b) |
| **Trust as a feature** | a commodity image API cannot cheaply say "here is what our pipeline did, check it yourself" |
| **Overclaim discipline** | the "does not prove" table protects both sides: we never imply authorship, so a seal cannot become a legal promise we did not intend |
| **Retention risk** | we hold nothing about a child's book, so the worst-case leak is a hash of a document we never had |

What we give up: we cannot say who made what. That is the trade the design chooses — and it is why
every sentence on this page is about the *document*, not the person.

## What it is not

- **Not a copyright registration.** It is evidence about production, not ownership.
- **Not a DRM or an anti-copy measure.** It does not stop anyone doing anything.
- **Not a copy of your book on our servers.** There is nothing to leak and nothing to delete.
- **The colophon is content, not pixels.** `colophon` writes `page/colophon.json` and `.txt` and seals
  them; drawing that text as a page is the book's own job, done by the runtime, and deliberately outside
  this command.
- **Not a wallet, a token, or an investment.** There is nothing to buy and nothing to hold.
- **Not required.** A book without a seal is a book. Nothing in the build, the publish path, or the
  simulator depends on one existing.
- **Not reproducible-by-rebuild.** The seal says where the bytes came from; it does not promise that
  a second build produces identical bytes (PSDs carry timestamps, for one).

## Open questions

Recorded here rather than answered in a doc, because these decide what ships:

1. Whether the seal lives beside `book.yaml` as its own file, or as a section inside `book.yaml`
   (a YAML rewrite would break the signature, so the separate file is favoured).
2. Whether `kwik publish` ships the seal to your host automatically.
3. Whether a partly hand-edited book can count towards a membership, or only a fully intact chain.
4. Whether the seal pins the toolchain versions (psd-tools, Pillow, ffmpeg) or only the scripts.
5. Whether the colophon page is generated by default (with the seal line optional) or only on request —
   it is an irreversible choice, so defaulting it on would be the wrong answer.

## Where the design lives

| | |
| --- | --- |
| the design and the staged plan | [`../provenance-plan.md`](../provenance-plan.md) |
| why it exists, and the membership it belongs to | [`../membership-model.md`](../membership-model.md) §2b, §7 |
| the rules the payload obeys | [`../kwik-cloud-plan.md`](../kwik-cloud-plan.md) §6 (C8) |
| the pricing and credit model it sits inside | [`../decisions.md`](../decisions.md) D13 |
