# Security

## Reporting a vulnerability

Use GitHub's **Private vulnerability reporting** on this repository (Security → Report a
vulnerability), or email **support@kwiksher.com**. An advisory is preferred: it stays private from
the first message and we can fix, release and disclose in one place.

Please do not open a public issue for a vulnerability. We aim to answer within a week.

## What this repository holds, and what it cannot leak

* **Static HTML, no server code.** A leak here is a published page or an asset that should not be
  public — report it and it is unpublished, not patched.
* **The claim pages are generated, not typed.** The club numbers come from a script that reads
  manifests a stranger can fetch; if a number looks wrong, say which manifest disagrees rather than
  editing the page.
* **No analytics, no third-party script** is embedded on purpose; a PR that adds one is a decision,
  not a detail.

## Supported versions

The latest release of each channel is supported. Kwik is pre-1.0 where it is pre-1.0 — the
`version.txt` a release stamps and `kwik version` are the truth; older tags are not patched.

## Not a security boundary

Kwik's paid product is **Kwik Cloud** (GPU time and the credit ledger). Everything in this
repository runs on the user's machine and talks to no server of ours except where the user configures
one — so a bug here is a local bug, not a leak of someone else's data.
