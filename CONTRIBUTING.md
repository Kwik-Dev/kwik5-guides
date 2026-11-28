# Contributing to the site and the guides

## Set up

```bash
git clone https://github.com/Kwik-Dev/kwik5-guides && cd kwik5-guides
cd guides && hugo server      # → http://localhost:1313/
```

Hugo 0.166 extended. Nothing else.

## Rules that bite

* **The manual is a mount.** `guides/hugo.toml` reads `../docs/guides`, so `hugo` must run from
  `guides/`; from anywhere else it renders 6 pages instead of 15 and exits 0.
* **Copy that is data stays data.** Prices and features live in `guides/data/en/*.yaml`; a price in a
  template is a bug waiting to happen.
* **Claims are generated.** `deploy/club-claim.py` in `Kwik-Dev/kwik5-siteground` writes
  `club.json`; do not hand-edit a number a reader is meant to verify.
