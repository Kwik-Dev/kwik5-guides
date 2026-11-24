---
title: What runs where
description: Every tool, on your machine or in the cloud, what it costs, and whether it is built
order: 7
audience: author, agent
status: the status column is the truth — several rows are decided, not built
updated: 2026-11-23
---

# What runs where

This page is reference, not a step. Read it when you are deciding *where* something should run, or
when `kwik doctor` tells you a tool is not available and you want to know what the alternative costs.

**This is the one table of features.** 1 credit = $0.10. `Available` means shipped and exercised;
`decided` means agreed in `decisions.md` with no implementation yet — a spec, not a feature.

The **on your machine** column has two kinds of entry, and the difference is the whole point of the
install menu:

- **`kwik` itself** — works with **nothing installed**. Resizing, cropping, layers, the PSD, the Lua,
  the md5 gate, `ogv`, and the CC0 sound library are all files and code, not models.
- **backend** — needs Guaardvark and model weights. This is the generation side, and it is the only
  thing the install menu's gigabytes buy.

| feature | on your machine | cloud | cost | status |
| --- | --- | --- | --- | --- |
| project, pages, scenes, layers, asset check, Simulator (`kwik new` / `book` / `page` / `layer` / `asset check` / `run`) | **`kwik` itself** — works with nothing installed | — | **free** | **available** |
| **page art** — `cast_render` | **backend**: Z-Image Turbo + your Cast LoRAs (CUDA → MPS → CPU) | Ada-24, returns a 1536×1024 PNG | **2 cr / image** | cloud **available**; local needs a backend |
| **narration** — `tts_speak` | **backend**: Kokoro-82M, 31 voices, CPU-capable | CPU endpoint | **1 cr / started minute** | decided, not built |
| **music bed** — `music_bed` | **backend**: ACE-Step (CUDA → MPS → CPU), 240 s cap | GPU | **1 cr / started minute** | decided, not built |
| **sound effects** — *library* (the default) | **`kwik` itself** — a curated **CC0** set, a few MB, shipped or pointed at; no model | — | **free** | decided, not built |
| **sound effects** — *generated* (opt-in) | **backend**: Stable Audio Open locally: CUDA, or MPS behind `AUDIO_FOUNDRY_SAO_MPS=1`, ≤ 47 s, 44.1 kHz | **no** — never in the cloud (D15) | **free** (local only) | exists upstream; not a shipped tool |
| **clips** — `video_clip` | **backend**: ComfyUI image-to-video | 48 GB pool | **5 cr / second** (6 s default, 15 s cap, 60 s per job) | no worker yet |
| **Cast LoRA** — `cast_train` | **backend**: local trainer (CUDA) | GPU, minutes | **40 cr / LoRA** | no worker yet |
| **upscale** — `lanczos` | **`kwik` itself** — deterministic, works with nothing installed | — | **free** | decided, not built |
| **upscale** — `local` | **backend**: Guaardvark's upscaling plugin | — | **free** | exists upstream |
| **upscale** — `cloud` | — | Ada-24 | **1 cr / image** | decided, not built |
| **build**: bleed crop, per-layer PNGs, `page.psd`, Lua, md5 gate, `ogv` | **`kwik` itself** — no Python, no Adobe, no models | — | **free** | decided, not built (D18) |
| **publish** | **your** host — Pages, Netlify, itch.io (D5) | — | **free** | not built |

Three consequences worth reading before you pick a menu option:

- **The free path is real, not a trial.** Everything above marked *local* costs nothing, forever —
  that is D2. The cloud exists because a laptop without a GPU cannot do the rows that need one.
- **The rows you cannot buy yet are marked.** `video_clip`, `cast_train`, `tts_speak` and `music_bed`
  are priced and catalogued but have no worker deployed: a call must refuse with `tool_unavailable`
  and release the hold, never `500`.
- **A missing model changes the executor, not the feature.** Without the art weights, `cast_render`
  runs in the cloud for 2 credits; with them, it runs on your machine for nothing.

