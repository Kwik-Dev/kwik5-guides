---
title: Privacy Policy
description: What kwiksher.com, the Kwik apps and Kwik Cloud collect, what they never receive, and how to have it deleted.
layout: page
updated: 2026-11-15
---



**Last updated: 15 November 2026.**

This policy covers four things, which collect very different amounts of data: this **website**, the
**Kwik apps**, **Kwik Cloud** (the paid rendering service), and the **archive** of the previous site.
It describes what the software actually does, not what a template says software usually does.

**Who we are.** Naoya Yamamoto, doing business as **Kwiksher**. Contact for anything on this page:
**support@kwiksher.com**.

## The short version

- The website sets **no cookies** and runs **no JavaScript**, so there is no analytics, no tracking
  pixel and no consent banner to click.
- Payments are handled by **Polar** as merchant of record. We never see or store your card.
- Your **licence key is your identity** in the cloud. There is no account, no password, and no email
  required to use a key you already own.
- Kwik Cloud stores **credit balances and job metadata** — never your manuscript, never a prompt,
  never a caption.
- **Your art is never used to train anything.**

## 1. This website

The site is static HTML and one stylesheet. It contains **no JavaScript file at all**, sets no
cookies, and has no analytics, advertising or social embed.

What is still collected, because any web server must:

| data | why | who holds it | kept |
|---|---|---|---|
| IP address, user agent, requested URL | serving the page, security, abuse prevention | our hosting provider's access logs | as long as the host's log retention allows |

We do not build profiles, and there is nothing to opt out of because there is nothing being tracked.

## 2. Payments, and what Polar sees

Credits are bought through **Polar**, which acts as **merchant of record**. That means Polar — not
we — is the seller of record for the transaction, handles card details, and handles sales tax and VAT
in the applicable jurisdiction. Polar receives your name, email, country and payment details.

**We receive from Polar only:** an order reference, the credits it bought, and the **customer id**
that Polar assigns to you. We do not receive your card, and we do not store your name or email.

See [Polar's privacy policy](https://polar.sh/legal/privacy) for what the merchant of record holds.

## 3. Kwik Cloud — what is stored

Opting in to cloud rendering means asking a machine we rent to render your page. The service is keyed
by your **Polar licence key**, which we never store: each request is validated against Polar and
answered with a customer id.

What the database holds, exactly:

| stored | example | notes |
|---|---|---|
| a customer id | `cus_…` (Polar's identifier) | there is no row with your name or email in it |
| credit balance | granted 300, used 84 | the meter |
| per-job metadata | tool `cast_render`, units, estimated and actual credits, state, an error code | the receipt |
| a request hash | a hash of your request | so a retried request cannot run twice; **not** the request itself |
| an output object key | `art/<job>.png` in our object store | not the image data in the database |

**What the cloud never receives.** Only the scene, the cast registry, a size, and an output key or a
pre-signed destination. Never your manuscript, never the book tree, never a customer key. Logs carry
`job_id`, units, timings and hashes — **no prompt, no caption, no OCR text**.

## 4. How long it is kept

| item | kept |
|---|---|
| Rendered images, narration, clips, book bundles (object storage) | **30 days** |
| Uploaded source files | **24 hours** |
| Cast LoRAs you trained | while your account is active; **deleted within 24 hours of a request** |
| Credit ledger and job rows | while the account exists — these are the receipts for money |
| Website access logs | the hosting provider's retention period |

**No training.** Your art, your manuscript and your renders are never used to train a model — ours or
anyone else's. This is a decision recorded in the project's decision log, not a promise that can be
quietly dropped.

## 5. The Kwik apps

The Photoshop panels, the editor and the runtime are local programs:

- They **do not collect personal information** and do not send telemetry home. Local work runs
  offline.
- If you choose to use cloud tools, only what §3 describes leaves your machine, and only when you
  invoke a cloud tool.
- Where an app integrates a third-party service (for example an app store or a platform SDK), that
  service's own policy applies.
- **Payments inside published apps** are processed by the app store (Apple, Google). We never receive
  payment details from an in-app purchase.

## 6. The archive

The previous site — a WordPress installation with its tutorials, forum and showcase — is archived
rather than deleted, because a decade of tutorials is still the best documentation for Kwik 3 and 4.

**The archive is the old stack, unchanged**: it runs WordPress with its own plugins and sets its own
cookies (session, anti-spam, and the plugins the forum used). Its cookie and privacy behaviour is the
one described by the previous version of this policy, and it is separate from everything above. If
you interact with the archive — logging in, posting, commenting — that stack collects what WordPress
normally collects.

We are not migrating that data into this site.

## 7. Children

Kwik is a tool for making children's books; it is not directed at children, and we do not knowingly
collect personal information from anyone under 13 (or the applicable age in your jurisdiction).

- The **website** collects nothing, from anyone.
- **Purchases** require the buyer to be an adult with a payment method.
- **Books you make about children** are your content, and remain yours: we do not read them, and the
  cloud receives no manuscript (§3).

## 8. Your rights

Depending on where you live you may have the right to access, correct, delete, port or object to the
processing of your personal data, and to complain to a supervisory authority.

Because we hold almost nothing, most requests are short:

- **See what we have:** email us; we will tell you which customer id is yours and what the ledger
  holds for it.
- **Delete it:** email us. Objects and uploads go immediately, and the ledger row for the customer is
  deleted with it. **Cast LoRAs are deleted within 24 hours** of the request.
- **Correct it:** we hold no name, address or email to correct. Correcting those means changing them
  with Polar, who holds them.

Write to **support@kwiksher.com**. We answer within two business days and complete deletion requests
within 24 hours for model data.

## 9. Who processes data on our behalf

| provider | role | what it can see |
|---|---|---|
| **Polar** | merchant of record; licence keys | your payment and contact details, orders |
| **Cloudflare** | Workers (the API) and R2 (object storage) | request metadata, stored objects |
| **Neon** | Postgres (the credit ledger) | the rows in §3 |
| **Runpod** | GPU capacity that runs a render | the scene and output key for a job |
| **SiteGround** | static hosting for this site and the archive | HTTP access logs |
| **GitHub** | source code and issue tracker | whatever you post there publicly |

We do not sell personal data, and we do not share it for advertising.

## 10. Security

Transport is TLS everywhere. Payment credentials never touch our systems. Cloud access is by licence
key, which is validated on every call and never stored. If a breach affects personal data, we will
notify affected people and the relevant authority as required by law.

## 11. Changes

If this policy changes materially — a new processor, a new category of data, an analytics tool — the
date at the top changes and the change is described here rather than silently edited in.

## 12. Contact

**support@kwiksher.com** — support, privacy requests, or a correction to anything on this page.
