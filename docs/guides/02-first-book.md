---
title: Your first page
description: One page from nothing — create, render, narrate, score, run — with the price shown before every spend
order: 2
audience: author
status: partly available (see per step) — D14/D18 steps are not built
updated: 2026-11-24
---

# Your first page

One page, end to end, on the cheap: 2 credits for the art, 1 for the narration, 1 for a music bed.
That is 4 credits — 40 cents — and it is the whole loop you will repeat for the other nine pages.

**Steps 3, 4 and 7 talk to Kwik Cloud, so they need a licence key.** The free tier is 30 credits,
which covers this whole page, so nothing has to be bought to finish it. Steps 1, 2, 5 and 8 never
need a key, an account or a network connection. [03 — The cloud](03-cloud.md) is how you get the key
in (`kwik cloud login`) and what it costs after the free credits.

| step | command | cost | key? | status |
| --- | --- | --- | --- | --- |
| 1 | `kwik new my-book` | free | no | **available** |
| 2 | `kwik book set my-book p1 --tags ellie,meadow --action "@ellie walks through @meadow."` | free | no | **available** |
| 3 | `kwik render --book my-book --page p1` | 2 credits (cloud) / free (local) | **yes** — cloud | cloud **available**, local **decided not built** |
| 4 | `kwik cloud wait <job_id> --download .kwik/out` | free | **yes** — cloud | **available** |
| 5 | `kwik build` — upscale, bleed crop, layers, `page.psd`, Lua, md5 gate | free | no | **decided not built** (D18) |
| 6 | `kwik music --page p1 --duration 30 --loop` | 1 credit / free | only for cloud | **decided not built** (D15) |
| 7 | `kwik cloud tts --page p1` | 1 credit / free | **yes** — cloud | **decided not built** |
| 8 | `kwik asset check && kwik run` | free | no | **available** |

## 1. Create the project — free

```bash
kwik new my-book
cd my-book
```

Clones the template, installs the **Kwik plugin** modules for your platform, writes
`.kwik/project.json`. No key needed. Nothing leaves the machine.

The **Solar2D Simulator** is a separate download from [solar2d.com](https://solar2d.com/#download) —
`kwik new` does not install it. See [01 — Install](01-install.md#requirements).

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

**This is the first step that needs a key.** Sign in once — `kwik cloud login`, a hidden prompt — and
the free 30 credits pay for this render. Local art (`--local`) would be free, but it is not built yet,
so `--auto` sends step 3 to the cloud. The command prints the price and asks before it spends; a
refusal costs nothing.

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

## Where the assets land

Everything a page produces stays inside the project directory. Nothing is global except two files: the
cloud credential ([03 — The cloud](03-cloud.md#where-it-is-stored)) and `~/.kwik/venv`, which is the
only Python `kwik` ever installs into. This is `my-book/` from step 1, with the paths as they exist in
the reference book:

```
my-book/
├── .kwik/                                 state, not the book
│   ├── project.json                       `kwik new` — what makes a directory a project
│   ├── scenes/<book>/<page>.json           `kwik book set` (step 2)
│   ├── out/                                `kwik cloud wait --download` (step 4)
│   ├── logs/                              simulator.log, screenshot.png, recording.mp4
│   └── exporter/                          the panel ↔ CLI mailbox
├── Photoshop/<book>/<page>/
│   ├── <page>.psd                         the designer's master — yours, or the Exporter's
│   ├── art/                               page art as it arrived (bg.png, bg-large.png)
│   └── png/<layer>.png                    one PNG per layer, after the bleed crop
├── Solar2D/App/<book>/
│   ├── components/<page>/layers/*.lua     the emitted Lua, one file per layer
│   ├── assets/images/<page>/<layer>.png   what the app actually loads
│   ├── assets/audios/                     narration and music beds
│   └── assets/videos/                     clips, including the .ogv
├── UXP/                                   the Exporter panel (.ccx), installed by `kwik new`
└── Scripts/  BookServer/  docs/           the template's own tools
```

Which step leaves what behind:

| step | writes | where |
| --- | --- | --- |
| 1 `kwik new` | the project, `.kwik/project.json`, `UXP/` | the whole directory |
| 2 `kwik book set` | the scene JSON, and `page.layers.yaml` for a layered page | `.kwik/scenes/<book>/<page>.json` |
| 3–4 render + download | the PNG the cloud rendered | `.kwik/out/`, then the page's `Photoshop/<book>/<page>/art/` |
| 5 `kwik build` | upscaled and bleed-cropped layer PNGs, `page.psd`, one Lua file per layer, the md5 manifest | `Photoshop/<book>/<page>/png/`, `Solar2D/App/<book>/assets/images/<page>/`, `Solar2D/App/<book>/components/<page>/layers/` |
| 6 `kwik music` | the bed | `Solar2D/App/<book>/assets/audios/` |
| 7 `kwik cloud tts` | the narration | `Solar2D/App/<book>/assets/audios/` |
| 8 `kwik run` | the simulator log, recordings, screenshots | `.kwik/logs/` |

Two things worth knowing about that tree:

- **`Solar2D/App/<book>/assets/` is the book.** `Photoshop/` is source and `.kwik/` is state; neither
  ships. The reference book's installed images are ~58 MB for ten pages — check with
  `du -sh Solar2D/App/<book>/assets` when you wonder where the disk went.
- **Deleting `.kwik/` deletes your page definitions, not your pages.** The scene JSON is what
  `kwik book` and `kwik render` read, and the logs are only logs — but the art under `Photoshop/` and
  the assets the app loads are untouched.

Steps 5–7 are not built yet (D18, D15), so today those paths are written by the template's Python
scripts and the Exporter panel. They are the same paths: that is how `biscuit-comes-home` was built,
and why the layout above can be given as a fact rather than a plan.

## Real files from the reference book

Everything below was produced by the loop above, on **page 1 of `biscuit-comes-home`** — whose
Fountain scene is *“@ellie walks through the wildflower meadow below @meadow, picking flowers”*, the
same action as step 2. These are the files, not mockups, and the audio and video play.

Each one is shown with the command from this guide that produces it, and that command's status today.
The files themselves were made by the template's Python scripts and the Exporter panel, before those
commands existed; the paths are the same, which is the point of listing them together.

**Before the loop**, the scene was written in step 2. `kwik book set` records the tags, the action and
the seed, which is what makes the render repeatable. **Available** in the CLI today:

```bash
kwik book set my-book p1 --tags ellie,meadow --action "@ellie walks through @meadow"
```

{{< artifact src="img/biscuit/p1-page.jpg" caption="**The page, finished** — one 1920×1280 plate, one keyed character layer, one caption layer, composed and readable. At `/cloud-sandbox/` this page runs: caption, z-order and legibility are checked by eye, because no numeric gate can see a caption clipped by its plate." >}}

**Step 3 — the art.** One `zimage-turbo` render at 1536×1024, upscaled once to the 1920×1280 page
frame. No characters in it: they are added as layers, so the meadow can be reused and re-lit.
**Available** — `--cloud` works today; `--local` needs a backend, and `--auto` is the default:

```bash
kwik render --book my-book --page p1 --cloud    # 2 credits, printed before it spends
```

{{< artifact src="img/biscuit/p1-art-bg.jpg" caption="`Photoshop/biscuit/p1/art/bg.png` — the background plate, rendered from the `@meadow` subject only. The 2776×1440 bleed is a *crop* of a larger approved source (`bg-large.png`), never a reflection." >}}

**Step 5 — the layers.** One cut-out per character, keyed from a flat `#00B140` field, then one PNG
and one Lua file each. The props are PSD pixels verbatim — `x`/`y` is the layer centre, and there is
no per-axis scale, which is what made the geometry debuggable. **Decided, not built** (D18):

```bash
kwik build      # upscale, bleed crop, per-layer PNG, page.psd, Lua, md5 gate
```

{{< artifact src="img/biscuit/p1-layer-ellie.png" alt="Ellie, keyed out of the plate: a seven-year-old in a sage pinafore and yellow ribbon, holding a bunch of wildflowers" caption="`Solar2D/App/biscuit/assets/images/p1/ellie.png` — 500×668, RGBA, chroma-keyed. The `.lua` beside it carries exactly these pixels: `x = 1125, y = 704, width = 500, height = 668, anchorX = 0.5`." >}}

**Step 7 — the narration.** Kokoro read the caption. The voice is a Kokoro id, checked against the
shipped list **before** credits are held, so a typo costs nothing. **Decided, not built:**

```bash
kwik cloud tts --page p1                        # af_heart by default
kwik cloud tts --page p1 --voice am_michael     # a Kokoro voice id, validated first
```

{{< artifact type="audio" src="img/biscuit/p1-narration.mp3" caption="`assets/audios/narration_p1.wav` — 9.05 s of Kokoro narration for this page. The voice is Kokoro's own id (`af_heart` is the default), validated before any credit is held." >}}

**Step 6 — the music bed.** ACE-Step scores the page, and a partial minute is a full credit, so a 30 s
bed costs 1. **Decided, not built** (D15):

```bash
kwik music --page p1 --duration 30 --loop    # 1 credit, or free locally
```

A narration file and a music bed differ only in which of those two commands wrote them: both land in
`Solar2D/App/<book>/assets/audios/`, and both are metered per *started* minute.

**A clip, which has no `kwik` command yet.** A video job returns a flat-green take and the app keys it
at runtime with `filter.chromaKey` — that is the format, so this clip is green on purpose. The tool is
specified as `video_clip` (5 credits per second, 15 s cap, so 75 credits is the largest single clip the
product allows) but no CLI subcommand owns it: today it is a ComfyUI job whose output lands in `out/`,
and the app's copy is then encoded to Theora with an ffmpeg that has libtheora — which is exactly what
the template's `to_ogv.py` wraps, `-pix_fmt yuv420p` and `-ac 2` included because `plugin.movie` needs
both:

```bash
ffmpeg -i clip.mp4 -c:v libtheora -q:v 7 -pix_fmt yuv420p -c:a libvorbis -ac 2 clip.ogv
```

{{< artifact src="img/biscuit/p1-clip.mp4" poster="img/biscuit/p1-clip.jpg" caption="`out/ellie-picks-wildflowers/clip.mp4` — 1.6 s, the character on `#00B140`. A transparent `.ogv` is not a thing: Theora has no alpha channel and `plugin_movie` decodes through theoraplay, so the key is the only way to put a clip over a page." >}}

Three things this shows that the tables above cannot: the caption is a layer with its own scrim, not
text baked into the art; the character layer is genuinely separate from the plate (which is why
`kwik book set` validates every `@tag` before a render is charged); and the narration is a file in
`assets/audios/`, so re-recording one page costs one minute of metering, not a re-render of the page.
