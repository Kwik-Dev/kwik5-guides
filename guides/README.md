# kwiksher.com — one static build, built with Hugo

This project renders **the whole new site** so it can be reviewed and deployed from one command
while the stack question is still open:

| path | source | status |
|---|---|---|
| `/` | `layouts/home.html` + `data/*.yaml` | **landing page, first version** — copy is draft, two blocks are placeholders |
| `/guides/` and `/guides/NN-*/` | **mounted** from `../docs/guides/*.md` | real content, six pages |
| `/404.html` | `layouts/404.html` | real |

`docs/new-site-plan.md` planned Astro for the landing page and, later, Hugo only for `/guides`. This
project builds both surfaces in Hugo so the structure is visible today. Splitting later is a copy
step, not a rewrite: the landing page is one template plus four data files.

## Build

```bash
cd guides
hugo                # → guides/public/         (baseURL https://kwiksher.com/)
hugo server         # → http://localhost:1313/
```

Hugo ≥ 0.146 (built on **0.166.0 extended**). No theme, no Hugo modules, no network at build time.

## Deploy

`public/` **is the domain root**, so it goes to `public_html/` — but the old WordPress site lives
there. Read `docs/new-site-plan.md` §7 before the first real rsync; that runbook owns the archive
move, redirects and DNS. For a review deploy, a subdirectory is safer:

```bash
hugo --source guides
rsync -a --delete guides/public/ <siteground>:/home/<user>/public_html/preview/
```

## How the guides are wired

`docs/guides/` is **mounted**, not copied — editing the planning repo edits the site.

```toml
[[module.mounts]]
  source = '../docs/guides'
  target = 'content/guides'
  files = ['[0-9][0-9]-*.md']   # README.md is the repo's index, not a page
```

The numbered glob is the one place filenames matter: a new guide must be `NN-name.md` to be
published. Everything after that is front-matter driven (`order` sorts index, sidebar and prev/next).

| in `docs/guides/*.md` | rendered as |
|---|---|
| `title`, `description` | `<h1>`, lede, `<title>`, `<meta name=description>` |
| `order` | index order, sidebar order, prev/next |
| `audience` | chip under the title |
| `status` | the amber banner (e.g. *"decided, not built"*) |
| `updated` | "updated …" beside the chip |

Links between guides resolve through Hugo's content graph (`layouts/_markup/render-link.html`), so a
renamed guide breaks the build instead of the link. A link to a file this site does not publish
(`../decisions.md`) renders as a plain reference, never a 404.

## Editing the landing page

| file | what it holds |
|---|---|
| `data/features.yaml` | the three "how it works" steps and the six features |
| `data/pricing.yaml` | credit price (D13), free tier (D16), per-tool costs, the ten-page-book figure |
| `data/faq.yaml` | the FAQ — answers cite decisions and never promise an unbuilt feature |
| `hugo.toml` `[params]` | tagline, description, GitHub/archive URLs, `preview` |
| `layouts/home.html` | section order and markup |

`params.preview = true` renders the strip at the top saying the copy is draft. Turn it off when the
copy is final.

## Deliberately not done

- **Polar checkout.** `params.checkoutURL` is empty, so no buy button renders. Copy is real; the
  button is not, and a dead buy button is worse than none.
- **Proof strip and testimonials.** Placeholders, visibly marked `TODO` in `layouts/home.html`.
  They need real covers from `kwik5-sample-books` and the verbatim credit lines from the archive.
- **`/pricing`, `/downloads`, `/updates` as pages.** Pricing, downloads and FAQ are sections of the
  single-page landing, which is what `new-site-plan.md` §6 describes. Separate routes arrive with
  the Astro build if they are still wanted.
- **Search, i18n (`ja`).** Six pages; nothing is translated yet, so there is no routing pretending
  otherwise.
- **The old manual.** `kwik-visual-code/document/src` is 309 pages of a **separate Hugo site**
  (`hugo-theme-relearn`, `uglyURLs`, 532 image shortcodes, 323 mermaid pages). It is not merged here.
  `docs/guides/` and its `get_started`/`kwik_CLI` sections overlap — reconcile before publishing
  both as separate manuals.

## Verified

`hugo` builds 13 pages with **no warnings**; `/`, `/guides/`, all six guides and the 404 answer
correctly under `hugo server`; the guides index lists six cards in `order`; pricing shows six tool
rows and two price cards; the FAQ renders seven `<details>`; nav anchors resolve to `/ #how`,
`#features`, `#pricing`, `#faq` and `/guides/`; CSS is fingerprinted and served.

**Not verified visually** — no browser automation was reachable in the session that built this, so
layout is verified structurally and by HTTP status, not by eye.

## Internal notes — the About and Privacy pages

Both are content pages (`content/about.md`, `content/privacy.md`, `layout: page`). The privacy policy
was rewritten for the new stack; every technical claim in it is checkable:

| claim | how to check |
|---|---|
| "no cookies, no JavaScript" | `find public -name '*.js'` → empty; the site ships HTML + one CSS file |
| stored columns (`customer`, balance, job metadata) | `kwik5-cloud/api/sql/*.sql` — `credit_accounts`, `jobs` |
| retention (30 d objects, 24 h uploads, LoRAs on request) | `kwik5-cloud/docs/SECURITY.md` §4 |
| "never your manuscript / prompt / caption" | same section, quoted verbatim |

**Still needed before it is legally complete** (do not delete this list):

- [ ] the **legal entity and postal address** — the previous policy named only a person
- [ ] confirm **support@kwiksher.com** exists and is monitored (the runbook in
      `docs/ops-admin-runbook.md` §3 assumes one address)
- [ ] decide what the **archive** does: if `archive.kwiksher.com` keeps the old WordPress install, this
      policy must say so and carry the old cookie table — §6 currently only says "the old stack,
      unchanged"
- [ ] if an **analytics** tool is ever added, §1 becomes false
- [ ] a lawyer reads it. The technical statements are accurate; whether they satisfy GDPR/CCPA
      wording is not something a repository can decide
