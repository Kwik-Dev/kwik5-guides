---
title: Install
description: What to install, what it costs in disk and time, and how to install nothing at all
order: 1
audience: author
status: decided (D14, D18) — not built
updated: 2026-11-13
---

# Install

> **Status: decided, not built.** `kwik setup` and `kwik doctor` are specified in D14 /
> `media-requirements.md` §1b (`REX.3`–`REX.7`), and no implementation exists yet. Until they ship,
> the local backend is installed by hand from
> [Guaardvark's own instructions](https://github.com/guaardvark) and the cloud needs nothing at all.
> This page is the spec the command has to satisfy.

## The short version

Two things exist, and they are independent:

| | you install | you download | you pay |
| --- | --- | --- | --- |
| **Cloud only** | `kwik` (~10 MB, one binary) | nothing | credits per job |
| **Local backend** | `kwik` + Guaardvark (+ ComfyUI) | model weights, by choice (12–41 GB) | nothing, forever |

**Installing nothing is a supported answer.** The default in `kwik setup` is *nothing — cloud only*.
You need a Polar key and a network connection; every media tool then runs in the cloud and costs
credits.

## What you get, and where it runs

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

## What `kwik` needs

| | |
| --- | --- |
| OS | macOS (Intel or Apple Silicon), Linux, Windows |
| Runtime | none — `kwik` is a single Go binary that runs its own Lua |
| Also needed | Solar2D (the runtime your book plays in), fetched by `kwik new` on first use (~40 MB) |
| Not needed | Python, Node, Docker, a compiler, a GPU |

`kwik doctor` reports what is present, and `kwik doctor --json` reports it for scripts and agents.

## The local backend (optional)

The local backend is **Guaardvark**, which starts **ComfyUI** as a plugin and hosts the audio tools.
It is what produced every asset in `biscuit-comes-home`.

`kwik setup` asks one question and installs nothing until you answer:

```
  Local backend — what should generation use? (nothing downloads until you pick)

  ▸ 1  no local models — cloud generation     0 GB    (needs a Polar key)
    2  page art only         Z-Image Turbo     ~12 GB   (recommended for local use)
    3  art + narration       + Kokoro          ~13 GB
    4  art + narration + music  + ACE-Step     ~27 GB
    5  everything            + a video model   ~41 GB
```

The gigabytes are **model weights only**. `kwik` itself, the CC0 sound library and every
files-and-code step above are a few megabytes and are installed regardless — option 1 removes
*models*, not features. With option 1 you still get, offline and free:

| still works with option 1 | needs the models, so option 1 sends it to the cloud |
| --- | --- |
| `kwik new` / `book` / `page` / `layer` / `asset check` / `run` | page art — 2 credits |
| **Lanczos upscale**, the bleed crop, per-layer PNGs | the local Guaardvark upscaler (a quality option, not a feature) |
| `page.psd`, the emitted Lua, the md5 gate | narration — 1 credit/started min |
| `ogv` encoding (needs an ffmpeg with libtheora, not a model) | music beds — 1 credit/started min |
| the **CC0 sound-effect library**, and `kwik sfx` | generated SFX (local-only in v1 — with option 1, not at all; use the library) |
| publish to your own host | clips, Cast LoRA training |

So: **option 1 is "generate in the cloud, assemble locally"** — not "cloud only".

Rules the command must honour:

- **Program and weights are separate decisions.** `kwik setup --native --with none` installs the app
  and downloads no model; `--with art` adds one.
- **Nothing downloads before its size and time are printed.** A silent 20 GB pull is the same class
  of bug as a silent charge.
- **No `sudo`, ever.** `kwik setup --docker` detects Docker and `nvidia-container-toolkit`; if either
  is missing it prints the exact per-OS install block and exits 1. `--print-install` prints that same
  block, so what you paste is what we test.
- **`--cloud-only`** is the non-interactive "install nothing"; `KWIK_NO_DOWNLOADS=1` fails instead of
  prompting on a locked-down machine.
- **`kwik setup --remove`** stops services and deletes the cache only after you confirm. It never
  touches a project or a trained LoRA.

### Docker

| host | GPU in a container | supported for media work |
| --- | --- | --- |
| Linux + `nvidia-container-toolkit` | yes | ✅ |
| Windows + WSL2 + `nvidia-container-toolkit` | yes | ✅ |
| macOS (Intel or Apple Silicon) | **no** — Docker cannot pass Metal | ❌ use the native install |

`kwik setup --docker` drives **Guaardvark's own** compose stack (`docker-compose.yml` +
`docker-compose.gpu.yml`), pinned by commit. It is not a fourth stack that will drift from upstream.

## A missing model is not a broken install

With nothing installed you will see this, and it is correct:

```
$ kwik doctor
  art       no local model (12 GB) — cloud: 2 credits/image
  narration no local model (1 GB)  — cloud: 1 credit/started min
  music     no local model (14 GB) — cloud: 1 credit/started min
  clips     no local model (14 GB) — cloud: 5 credits/second
  local     lanczos upscale ✓  build ✓  psd ✓  ogv ✓  sfx library ✓
  cloud     signed in, 30 credits

$ kwik render --page p1
  no local model for images → cloud, 2 credits each · 30 available · continue? [y/N]
```

`--auto` (the default) prefers local when the backend is healthy and can run that tool on that
device, and it prints which executor it chose and why. **It never silently spends money**: falling
back to the cloud needs a key, a printed price and your confirmation. Falling back to local needs
nothing, because it is free.

## Requirements the backend is held to

| | |
| --- | --- |
| loopback only | Guaardvark binds `127.0.0.1`; it has no multi-tenant auth and must never be exposed |
| no runtime downloads | a render never fetches a model or a voice; install-time only |
| version receipt | `kwik doctor --json` reports Guaardvark's version, the ComfyUI commit, torch, diffusers and the model ids — the same fields a cloud receipt carries, so "identical bytes" is checkable |
| provenance | every artifact records the device it came from: `mps/dev`, `cuda/local`, `ada24/bench`, `neon/deployed` |
