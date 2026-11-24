---
title: Install
description: What to install, what it costs in disk and time, and how to install nothing at all
order: 1
audience: author
status: decided (D14, D18) — not built
updated: 2026-11-24
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
| **Adobe Photoshop** (2022+) | refining page images by hand, and running the Exporter panel | **not mandatory — recommended.** The only application the **Kwik Exporter** panel runs in; see [Photoshop](#photoshop-recommended-not-required) |
| **ffmpeg** | `kwik record`, and encoding a clip to `ogv` | **only where you use those two.** `kwik` and every image step need none of it; the `ogv` encoder must be built with libtheora, which stock Homebrew `ffmpeg` is not — see [ffmpeg](#ffmpeg-installed-where-you-use-it) |
| **Python 3** + `psd-tools`, `pillow`, `pyyaml`, `numpy` | `kwik psd new`, `kwik icon` — authoring a PSD and an icon set | **not needed to build, render or run a book.** `kwik setup` offers to put it in its own venv, `~/.kwik/venv`, and never touches the system Python; see [the toolchain](#the-toolchain-kwik-setup-asks-about) |
| **Node, Docker, a compiler** | **nothing** | the free path needs none of them |

The local backend is **Guaardvark**, which starts ComfyUI as a plugin and hosts the audio tools — you
do not install ComfyUI separately. Upstream is what to install; Kwik keeps a fork for one reason, and
[04 — The local backend](04-local-backend.md#the-local-backend) is where that is explained. Neither is
required.

**Kwik Cloud is not a requirement either, and not something you install.** It is the optional paid
product: credits, and the licence key that identifies them, belong to it and are read only when you ask
the cloud to render something. Nothing above needs an account, a key or a network connection.
[03 — The cloud](03-cloud.md).

## Machine spec

**The cloud path has no machine spec.** `kwik` is a single binary and a project is text and PNGs, so
any laptop that runs a browser runs the cloud path. Everything below is about **local generation**, and
the numbers are not ours: they are the ones the backend itself enforces, from
[`guaardvark/docs/HARDWARE.md`](https://github.com/Kwik-Dev/guaardvark/blob/cloud-plus/docs/HARDWARE.md),
calibrated on a 16 GB card with a 32 GB machine. `kwik doctor` reports what *your* machine will
actually do — a static table goes stale, and a stale capability claim costs an afternoon.

### By platform

| platform | local backend | GPU path | notes |
| --- | --- | --- | --- |
| **macOS, Apple Silicon** | native | **Metal / MPS** | the only GPU path a Mac has; Docker cannot pass Metal into a container, so a container is CPU-only |
| **macOS, Intel** | native | **CPU only** | it works and it is slow; treat local generation as "images, eventually" |
| **Windows 11** | **WSL2** | CUDA inside WSL2 | there is no native Windows backend: Guaardvark's Windows support *is* WSL2, and it is still being verified — the `kwik` binary, the project and the Simulator stay on the Windows side |
| **Linux** | native or Docker | CUDA (Docker needs `nvidia-container-toolkit`) | the primary target, and the only place everything is tested |

The Simulator runs on all four. `kwik` itself runs on all four. It is the *backend* that is
platform-dependent, and the cloud is the answer when yours is not in the comfortable column.

### By workload

| | floor | comfortable | why |
| --- | --- | --- | --- |
| **system RAM** | **32 GB** | 64 GB | a Z-Image or Krea 2 still is 21–24 GB of resident memory per job; SDXL is ~10 GB |
| **NVIDIA VRAM**, page art only | **8–12 GB** | 16 GB | the 8–12 GB tier does SDXL-class images, upscaling and music |
| **NVIDIA VRAM**, everything | **16 GB** | 20–24 GB | clips refuse below 16 GB — the preflight wants a 16 GB-class card, ~0.5 GB of grace |
| **disk** | 12–41 GB | + tens of GB | the weights you pick in the setup menu; a video model is 9–20 GB on its own |
| **swap** (Linux, WSL2) | ≥ 16 GB, low swappiness | — | so a spike degrades instead of freezing the desktop |

Two traps in that table, both measured:

- **RAM decides, not VRAM.** Z-Image and Krea 2 always run with CPU offload, so their weights live in
  system memory between stages *even on a 24 GB card*. A 16 GB-RAM machine is refused with
  `system RAM too low (not VRAM)` no matter what is in the slot, and swap does not count toward the gate.
- **Under ~8 GB of RAM** the backend installs a small text-only model tier instead of the real one. The
  install succeeds, the page renders, and the results are not what the guide's examples show.

### On a Mac, read "RAM" as unified memory

Apple Silicon has no separate VRAM: the weights and the offload live in the same pool, which is why the
RAM column matters more there, not less. What Guaardvark has actually verified on Metal is **Z-Image and
Krea 2 image generation and LoRA training** (slow). **Not** verified on a Mac, by the project's own
statement: clips through ComfyUI on Metal — there is no render on file — so treat video as a cloud job,
and treat every Mac number as "images, yes; video, unproven".

### On Windows, the split is WSL2

Install the NVIDIA driver on Windows and let WSL2 see it; the backend, its Python and its models then
live inside the distro, where `kwik setup --native` can put them. Give the distro the RAM: WSL2 takes
roughly half of the host's by default, which on a 32 GB machine is 16 GB — under the floor above. Add to
`%UserProfile%\.wslconfig`:

```ini
[wsl2]
memory=24GB
swap=8GB
```

If you want Docker instead of the native install, `nvidia-container-toolkit` goes inside the distro
([Docker](#docker)), not on Windows.

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
| Not needed to run a book | Python, Node, Docker, a compiler, a GPU — and Python is needed only by `kwik psd new` and `kwik icon`, which `kwik setup` can put in their own venv |

`kwik doctor` reports what is present, and `kwik doctor --json` reports it for scripts and agents.

## Photoshop: recommended, not required

Nothing in the pipeline needs an Adobe application. The layered `page.psd` is generated by
`kwik build`, so it opens in Affinity Photo, GIMP, Photopea or Krita just as well
([05 — Media specs](05-media-specs.md)).

**Adobe Photoshop (2022 or later) is recommended anyway**, because it is the one application the
**Kwik Exporter** panel runs in. The panel is a UXP plugin: `kwik new` drops its `.ccx` (~25 MB) into
the project's `UXP/` directory, and it requires Photoshop `23.0.0` or newer. A designer uses it to
refine page images by hand and to publish them into the book — the panel walks the `.psd`, exports
layers and images, renders the Solar2D `components/`/`models/` Lua, and keeps `index.lua` in step.

So the split is worth stating plainly:

| who | needs Photoshop? |
| --- | --- |
| an author writing pages, rendering art, building and running the book | **no** |
| a designer refining and manually publishing page images for the book | **yes** — and only Photoshop; the panel declares `host: PS` and will not load in another application |

Skipping it is supported: `kwik new --no-exporter` leaves the `.ccx` out of the project entirely, and
nothing else changes.

## ffmpeg: installed where you use it

No image step shells out to an external tool. Resizing, the bleed crop, per-layer PNGs and the layered
`page.psd` are `kwik` itself, so **ImageMagick is not used anywhere** — not by the CLI, not by the
Exporter panel, not by the project pipeline. `kwik icon` writes the `.ico` directly, and only the
`.icns` needs `iconutil`, which macOS already has (`kwik doctor` warns when it is missing).

Two features do reach for the **ffmpeg** family, and neither is on the path to a finished book:

| command | needs | when it is missing |
| --- | --- | --- |
| `kwik record` | **ffmpeg** — it assembles the Simulator's frames into a video | exit 3, and the frames are kept with the directory named |
| `kwik asset` (the HTML report) | **ffprobe**, only to show durations | a warning, and durations are omitted — not an error |
| `ogv` output | **ffmpeg built with libtheora + libvorbis** | the step refuses, rather than emitting a file `plugin.movie` cannot decode |

The libtheora part is the one people get wrong: stock Homebrew `ffmpeg` no longer ships it
(`ffmpeg-full` does). Probe before assuming:

```bash
ffmpeg -encoders 2>/dev/null | grep -c theora   # 0 means this ffmpeg cannot produce ogv
```

`ffprobe` ships with `ffmpeg`, so one install covers the first two rows. `kwik doctor` must report
which encoder it will use (D14, `media-requirements.md` RVC.3b) rather than leave `kwik build` to
discover it.

## The toolchain `kwik setup` asks about

Four tools sit outside Guaardvark and outside a book build, and none of them is a prerequisite. They
are still needed by *some* command, so both commands name them rather than letting a command fail
later:

| tool | what it gates | what happens without it |
| --- | --- | --- |
| **ffmpeg** | `kwik record`, `ogv` output | record exits 3 with the frames kept; `ogv` refuses |
| **ffprobe** | durations in the `kwik asset` HTML report | the report omits durations, with a warning |
| **python3** + `psd-tools pillow pyyaml numpy` | `kwik psd new`, `kwik icon` | both exit 8 with the pip line to run |
| **iconutil** (macOS only) | the `.icns` half of `kwik icon` | the iconset is kept and the `.icns` is skipped |

**`kwik doctor` reports, `kwik setup` asks.** Doctor is read-only and prints each tool next to the
command it gates. Setup asks about each one that is missing, and runs nothing until you answer:

```
  ffmpeg is not installed — `kwik record` and `ogv` output need it.

  ▸ 1  install it now    brew install ffmpeg-full          (no sudo)
    2  print the command, I'll run it
    3  skip — record and ogv stay unavailable; everything else works
```

What each answer may do is bounded by the same rule as everything else here:

- **An install is offered only where it needs no privileges** — Homebrew, `winget`, `pip --user`.
  Where it would need root — `apt`, `dnf`, `pacman` — setup **prints the command and stops**, and
  there is no menu entry that runs it:

  ```
  ffmpeg is not installed, and installing it here needs root.
  Run this, then `kwik doctor` will see it:

      sudo apt install ffmpeg
  ```

  `kwik` never asks for your password, never pipes a script to a shell, and never retries the check
  on its own — re-run `kwik doctor` when you have.
- **Python goes into `~/.kwik/venv`**, never the system interpreter, so accepting the offer cannot
  break a distro's Python. `kwik doctor` prefers that venv over `PATH` and says which it used.
- **`--print-install` covers these too**, so the block you would paste is the block we test.
- **Non-interactive:** `--tools=install|print|skip`, with `ask` as the default; on a locked-down
  machine `KWIK_NO_DOWNLOADS=1` means `skip`.
- **The asymmetry is deliberate.** A missing *model weight* fails a locked-down setup, because the
  tool that was asked for cannot run at all. A missing *tool* only narrows what the CLI can do, so
  it is skippable: no tool here is needed to build, render or run a book.
- **A present-but-wrong tool is reported as wrong.** An `ffmpeg` without a theora encoder is not
  "ffmpeg ✓" — doctor says so, and setup offers `ffmpeg-full`.

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
| **Adobe Photoshop** (2022+) — hosting the Exporter panel | **you**, and only if you or your designer wants it | adobe.com | not installed, not required and not detectable by `kwik`; the panel needs Photoshop `23.0.0`+, and `kwik` works without it |
| **ffmpeg / ffprobe** | **`kwik setup`**, after you pick an option | `brew install ffmpeg-full`, `apt install ffmpeg`, or the vendor build | printed, never executed, where it needs root; `kwik` needs it only for `kwik record`, the asset report and `ogv` |
| **Python 3** + `psd-tools pillow pyyaml numpy` | **`kwik setup`**, into `~/.kwik/venv` | PyPI, at your answer | only for `kwik psd new` and `kwik icon`; the system interpreter is never touched |
| **ImageMagick** | **never** | — | not used at all: every image step is `kwik` itself, and `kwik icon` writes the `.ico` directly |
| **Guaardvark** | **`kwik setup`** (`--native` or `--docker`) | **[Kwik-Dev/guaardvark](https://github.com/Kwik-Dev/guaardvark), branch `cloud-plus`**, pinned to a commit | the backend Kwik is built and tested against; `kwik` never runs its own copy of it |
| **ComfyUI** — if you have none | **`kwik setup`**, as Guaardvark's own dependency | upstream's `requirements`/compose | never installed *alongside* Guaardvark — Guaardvark starts it as a plugin, and its `models/` directory is where clip weights land |
| **ComfyUI you already run** | **you**, before Kwik ever sees it | your own install, your own path or port | **detected and used as-is.** `kwik` does not install a second copy, does not move or re-index your models, does not add custom nodes to it, and does not restart it unless you say so. `kwik doctor --json` records its commit and port, so a version mismatch is visible instead of mysterious |
| model weights | **`kwik setup`**, after you pick a menu option | Hugging Face, into a cache (`KWIK_MODEL_DIR`, `HF_HOME`, ComfyUI's `models/`) | sizes are printed before anything downloads |
| Python, Docker, `nvidia-container-toolkit` | **never** | — | `--native` creates its own venv; `--docker` *detects* Docker and prints the install block instead of installing it. The one Python `kwik` will install is its own `~/.kwik/venv`, with your answer |

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
  block, so what you paste is what we test. The same block discipline covers the toolchain
  ([above](#the-toolchain-kwik-setup-asks-about)), with one addition: where an install needs no
  privileges, setup offers to run it, after asking.
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
  tools     ffmpeg ✗ (record, ogv)  python3 3.12.4 ✓  psd-tools ✗ (kwik psd)
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

## Uninstalling

This page is a list of things you add, so uninstalling is a list of things you remove — nothing was
installed into a place you cannot see. Two rules: **`kwik setup` uninstalls only what `kwik setup`
installed** (a tool or a model you installed yourself stays yours), and **stop services before you
delete anything**, or a running Guaardvark holds the files open.

Do this first, because it is the only part `kwik` can do for you:

```bash
kwik setup --stop        # stop Guaardvark and its ComfyUI plugin
kwik setup --remove      # after you confirm: delete the model cache kwik downloaded
```

`--remove` never touches a project, a trained LoRA, an ffmpeg you installed, or a ComfyUI you already
ran. Then remove the rest by hand:

| what | where it is | how to remove it |
| --- | --- | --- |
| model weights | the cache `kwik setup` chose (`KWIK_MODEL_DIR`, `HF_HOME`, ComfyUI's `models/`) | `kwik setup --remove`. Weights you downloaded yourself, into your own ComfyUI, are yours — delete those from your own tree |
| Python packages | `~/.kwik/venv` — the only Python `kwik` ever installs | `rm -rf ~/.kwik/venv` |
| Guaardvark in Docker | the fork's own compose stack | `docker compose down -v` in that directory — the `-v` is the part that deletes the volumes |
| ffmpeg, ffprobe, python3 | your package manager, where you installed them | `brew uninstall ffmpeg-full`, `sudo apt remove ffmpeg`, `winget uninstall ...`. `kwik` does not uninstall these, because it did not install them |
| the Exporter panel | inside each project, `UXP/` | delete `UXP/`, or remove **Kwik Exporter** in Photoshop's plugin manager. It is a panel, not a system install |
| the cloud key | `~/Library/Application Support/kwik/credentials` (macOS), `~/.config/kwik/credentials` (Linux), `%AppData%\kwik\credentials` (Windows) | `rm` that file. There is no logout command: the file is the session ([03 — The cloud](03-cloud.md#where-it-is-stored)) |
| the Solar2D Simulator | your own download from [solar2d.com](https://solar2d.com/#download) | delete the application. `kwik` never installed it |
| your book | the project directory | `rm -rf` it. Everything a project owns is inside that directory, including `.kwik/` |
| `kwik` itself | wherever you placed it | `rm $(command -v kwik)` |

There is no uninstaller to run and nothing to clean up afterwards: `kwik` starts no daemon, adds no
login item, keeps no registry of your projects, and writes nothing outside `~/.kwik` and the project's
own `.kwik/`. The only long-lived processes it ever creates are Guaardvark and ComfyUI.

A half-removed install is not a broken one — it is the same state this page started from. `kwik doctor`
says exactly which tools and models are gone, and which commands still work without them.

