---
title: Your first page
description: One page from nothing — create, render, narrate, score, run — with the price shown before every spend
order: 2
audience: author
status: partly available (see per step) — D14/D18 steps are not built
updated: 2026-11-13
---

# Your first page

One page, end to end, on the cheap: 2 credits for the art, 1 for the narration, 1 for a music bed.
That is 4 credits — 40 cents — and it is the whole loop you will repeat for the other nine pages.

| step | command | cost | status |
| --- | --- | --- | --- |
| 1 | `kwik new my-book` | free | **available** |
| 2 | `kwik book set my-book p1 --tags ellie,meadow --action "@ellie walks through @meadow."` | free | **available** |
| 3 | `kwik render --book my-book --page p1` | 2 credits (cloud) / free (local) | cloud **available**, local **decided not built** |
| 4 | `kwik cloud wait <job_id> --download .kwik/out` | free | **available** |
| 5 | `kwik build` — upscale, bleed crop, layers, `page.psd`, Lua, md5 gate | free | **decided not built** (D18) |
| 6 | `kwik music --page p1 --duration 30 --loop` | 1 credit / free | **decided not built** (D15) |
| 7 | `kwik cloud tts --page p1` | 1 credit / free | **decided not built** |
| 8 | `kwik asset check && kwik run` | free | **available** |

## 1. Create the project — free

```bash
kwik new my-book
cd my-book
```

Clones the template, installs the Solar2D modules for your platform, writes `.kwik/project.json`.
No key needed. Nothing leaves the machine.

## 2. Write one page — free

```bash
kwik book set my-book p1 \
  --tags ellie,meadow \
  --action "@ellie walks through the wildflower meadow below @meadow, picking flowers."
```

Two rules, both enforced locally so you never pay to discover them:

- **every `@tag` in the action must be in `--tags`** — a render must not invent a character;
- **identity enters only through tags.** A `@tag` resolves to a cast member's `subject_id`, and the
  LoRA is applied by the backend. Writing a trigger name (`ellie_v1`) into a prompt is a bug: it
  bypasses the Cast registry.

`kwik book set` writes the scene JSON and, for a layered page, `page.layers.yaml`.

## 3. Render the art — 2 credits, or free

```bash
kwik render --book my-book --page p1              # --local / --cloud / --auto (default)
```

The scene is composed by `kwikcast` into a prompt and a **derived seed** — `sha256(book, page, layer,
prompt, model, attempt)` — so the same inputs reproduce the same image, and a new take is an explicit
`attempt` increment rather than a different picture every time.

The cloud returns a 1536×1024 PNG (the `page` preset: exactly 0.8× of the 1920×1280 page frame). The
single upscale to 1920×1280 — and the 2776×1440 bleed plate cropped from it — happens on your machine
at step 5, not in the cloud.

## 4. Wait for it — free

```bash
kwik cloud wait <job_id> --download .kwik/out
```

Exit codes are the contract: `0` ok, `4` network, `5` refused, `7` job failed with the credits
returned. A failed job is never charged.

## 5. Finish the page — free (decided, not built)

`kwik build` does the deterministic steps: one upscale (Lanczos by default, or the local backend's
upscaler, or the paid cloud one), the bleed crop from that larger file, one PNG per layer,
`page.psd` **even if you own no Adobe software**, the emitted Lua, and an md5 asset gate that fails
when an installed image no longer matches its source.

Two invariants you can rely on, because they are what the approved book was built with:

- **a bleed is a crop, never a mirror or a tile** — a reflection duplicates the picture by definition;
- **no `xScale`/`yScale` in layer props** — the plugin applies the content scale; per-axis scale is
  what made the geometry un-debuggable.

## 6. Score it — 1 credit, or free (decided, not built)

```bash
kwik music --page p1 --duration 30 --loop      # a bed, 1 credit per *started* minute
```

A 30 s bed is **1 credit**, not 2: a partial minute is a full minute, and there is no floor. Beds are
generated (ACE-Step). **Sound effects come from a library, not a model** — a curated CC0 set you choose
during setup, which is why `thunder` or `crickets` costs nothing. Generation does exist locally (Stable
Audio Open) and is an opt-in, not the default: a different-sounding door on every take is not a
feature.

```bash
kwik sfx list
kwik sfx add thunder            # from the CC0 library, or a folder you pointed at
```

## 7. Narrate it — 1 credit (decided, not built)

```bash
kwik cloud tts --page p1                       # default voice: af_heart
kwik cloud tts --page p1 --voice am_michael    # a Kokoro voice id, validated before you are charged
```

Voices are **Kokoro's own**, a curated subset of its 31: `af_heart` (default), `af_nicole`,
`am_michael`, `bf_emma`. An unknown or unshipped voice is refused with the list, **before credits are
held** — you are never charged for a voice the worker cannot speak.

## 8. Run it — free

```bash
kwik asset check      # the asset graph agrees with the sources
kwik run              # the Simulator
```

Look at the page. Layout, caption legibility and z-order are only judgeable visually; the numeric
checks cannot see a caption clipped by its plate.

## What the whole book costs

The reference model is nine spreads plus a cover and a back page: 40 images, 10 minutes of narration,
ten 4-second clips.

| | credits | ≈ |
| --- | --- | --- |
| 40 images × 2 | 80 | $8 |
| 10 min narration × 1 | 10 | $1 |
| 10 beds × 1 | 10 | $1 |
| ten 4 s clips × 5/s | 200 | $20 |
| **total** | **~300** | **~$30** |

Your first 30 credits are free (once per customer, D16) — enough for a fully rendered, narrated and
scored page, which is the point of the loop above.
