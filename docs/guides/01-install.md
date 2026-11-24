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

## Requirements

Short list, because it is shorter than people expect:

| what | needed for | notes |
| --- | --- | --- |
| **`kwik`** | everything that is files and code | one Go binary, ~10 MB; macOS, Linux, Windows |
| **Solar2D Simulator** | playing and building the book | **your download from [solar2d.com](https://solar2d.com/#download)** — `kwik` locates it, never installs it |
| **A GPU** | local generation, and only that | CUDA or Apple-Silicon MPS; the cloud needs none |
| **Disk** | local generation, and only that | 12–41 GB of model weights, by the choice in the setup menu — nothing downloads until you pick |
| **Python, Node, Docker, a compiler, Adobe** | **nothing** | the free path needs none of them |

The local backend is **Guaardvark**, which starts ComfyUI as a plugin and hosts the audio tools — you
do not install ComfyUI separately. Upstream is what to install; Kwik keeps a fork for one reason, and
[04 — The local backend](04-local-backend.md#the-local-backend) is where that is explained. Neither is
required.

**Kwik Cloud is not a requirement either, and not something you install.** It is the optional paid
product: credits, and the licence key that identifies them, belong to it and are read only when you ask
the cloud to render something. Nothing above needs an account, a key or a network connection.
[03 — The cloud](03-cloud.md).

## The short version

Two things exist, and they are independent:

| | you install | you download | you pay |
| --- | --- | --- | --- |
| **Cloud only** | `kwik` (~10 MB, one binary) | nothing | credits per job |
| **Local backend** | `kwik` + Guaardvark (+ ComfyUI) | model weights, by choice (12–41 GB) | nothing, forever |

**Installing nothing is a supported answer.** The default in `kwik setup` is *nothing — cloud only*.
You need cloud credits and a network connection; every media tool then runs in the cloud and costs
credits. The key that identifies those credits is Kwik Cloud's business, not a prerequisite of the
software.

Every tool, on your machine or in the cloud, with its cost and whether it is built:
[07 — What runs where](07-what-runs-where.md). This page only covers getting the software in place.

## What `kwik` needs

| | |
| --- | --- |
| OS | macOS (Intel or Apple Silicon), Linux, Windows |
| Runtime | none — `kwik` is a single Go binary that runs its own Lua |
| Also needed | the **Solar2D Simulator** (the app your book plays in) — a separate download from [solar2d.com](https://solar2d.com/#download); `kwik` finds it in this order: `--solar2d <path>` → `KWIK_SOLAR2D_PATH` → the path recorded in `.kwik/config` |
| Not needed | Python, Node, Docker, a compiler, a GPU |

`kwik doctor` reports what is present, and `kwik doctor --json` reports it for scripts and agents.

## The local backend (optional)

The local backend is **Guaardvark**, which starts **ComfyUI** as a plugin and hosts the audio tools.
It is what produced every asset in `biscuit-comes-home`.

### Who installs what

This is the question the install menu does not answer, so it is answered here. Nothing below is
installed by *you* except `kwik` itself.

| component | installed by | from | notes |
| --- | --- | --- | --- |
| `kwik` | **you** | one binary, ~10 MB | the only thing you download and place yourself |
| **Solar2D Simulator** | **you** | [solar2d.com/#download](https://solar2d.com/#download) | the engine and the Simulator; `kwik run` locates it, `kwik` never bundles or downloads it |
| Kwik **plugin** modules | **`kwik new`** | the plugin release, downloaded into the project | seconds; `--no-exporter` additionally skips the ~25 MB `.ccx` |
| **Guaardvark** | **`kwik setup`** (`--native` or `--docker`) | **[Kwik-Dev/guaardvark](https://github.com/Kwik-Dev/guaardvark), branch `cloud-plus`**, pinned to a commit | the backend Kwik is built and tested against; `kwik` never runs its own copy of it |
| **ComfyUI** — if you have none | **`kwik setup`**, as Guaardvark's own dependency | upstream's `requirements`/compose | never installed *alongside* Guaardvark — Guaardvark starts it as a plugin, and its `models/` directory is where clip weights land |
| **ComfyUI you already run** | **you**, before Kwik ever sees it | your own install, your own path or port | **detected and used as-is.** `kwik` does not install a second copy, does not move or re-index your models, does not add custom nodes to it, and does not restart it unless you say so. `kwik doctor --json` records its commit and port, so a version mismatch is visible instead of mysterious |
| model weights | **`kwik setup`**, after you pick a menu option | Hugging Face, into a cache (`KWIK_MODEL_DIR`, `HF_HOME`, ComfyUI's `models/`) | sizes are printed before anything downloads |
| Python, Docker, `nvidia-container-toolkit` | **never** | — | `--native` creates its own venv; `--docker` *detects* Docker and prints the install block instead of installing it |

Two consequences worth knowing before you read the menu:

- **`kwik setup --native`** is `venv + Guaardvark + ComfyUI as its plugin`, bound to `127.0.0.1`. It
  does not install a second backend, and it does not install ComfyUI separately — Guaardvark owns it.
- **`kwik setup --docker`** drives **the fork's own** `docker-compose.yml` +
  `docker-compose.gpu.yml`, pinned by commit. There is no third stack of ours to drift from it.
- **You may already have ComfyUI, and that is a supported answer.** Detection comes first: a ComfyUI
  you point at (port or path) or one already listening is used as it is, and only when there is none
  does the setup provide one. It is your installation, with your models and your custom nodes — the
  rule is *use*, never adopt: no second copy, no model migration, no node installs. What Kwik does
  record is the commit and the port, because "identical bytes to the bench" is a claim that only
  holds when both sides name the same ComfyUI.

**`kwik setup` installs Kwik's fork, and that is deliberate.** The backend is
[Kwik-Dev/guaardvark](https://github.com/Kwik-Dev/guaardvark) on the **`cloud-plus`** branch: upstream
Guaardvark plus the cloud chat providers upstream removed, plus the fork's own work. It is the fork,
not upstream, that `kwik` is developed and tested against, so the pin is a commit on that branch.
Installing [upstream](https://github.com/guaardvark/guaardvark) instead will look familiar and may
work for some tools, but it is not the pin — `kwik doctor --json` reports the commit and branch it
found, so the mismatch is visible rather than mysterious.
[04 — The local backend](04-local-backend.md#the-local-backend) has the reasoning, including what
happens when the fork lags upstream.

> **Not built yet.** `kwik setup` and `kwik doctor` are specified (D14, `media-requirements.md` §1b,
> REX.3–REX.7) and unimplemented; until they ship, every row above except the first two is done by
> hand — Guaardvark from its own instructions, ComfyUI with it.

`kwik setup` asks one question and installs nothing until you answer:

```
  Local backend — what should generation use? (nothing downloads until you pick)

  ▸ 1  no local models — cloud generation     0 GB    (needs cloud credits)
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
- **An existing install is used, never duplicated or adopted.** `kwik doctor` reports what it found
  (Guaardvark, ComfyUI, their commits, the port) before `kwik setup` proposes to add anything.
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
