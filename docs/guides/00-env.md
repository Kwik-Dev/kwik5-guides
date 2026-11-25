---
title: Your machine
description: What local generation needs — RAM, VRAM, disk, platform — and the four external tools kwik shells out to
order: 0
audience: author
status: hardware numbers are the backend's, not ours; kwik setup is decided (D14, REX.3c) — not built
updated: 2026-11-25
---

# Your machine

**Nothing on this page is required to use Kwik Cloud.** The cloud path has no machine spec: `kwik` is
a single binary, a project is text and PNGs, and every media step happens somewhere else. Install is
[01 — Install](01-install.md).

This page is about the other case — the machine that generates art, narration, music and clips itself —
and about the four external tools `kwik` shells out to, which are not part of the software either.

## Minimum spec, if you never generate locally

The 32 GB / 16 GB-VRAM floor further down is **not** a Kwik requirement. It is what the local backend
enforces on a machine that runs a 21 GB image model. If every media step is a cloud job, none of that
applies, and the honest minimum is the one for "run a Go binary and open a window":

| | minimum | why that is the whole story |
| --- | --- | --- |
| **system RAM** | **4 GB** | `kwik` is one ~10 MB binary; a project is text and PNGs; the Solar2D Simulator is a small SDL window. Nothing loads model weights |
| **GPU** | **none** | no step renders on your machine — the CUDA/Metal paths only exist for generation |
| **disk** | **~1 GB free** | the binary, the Solar2D Simulator, and one book's page art. Not 12–41 GB: that is weights, and you download none |
| **display** | any windowed desktop | the Simulator shows the book at 320×480 and scales it; page art is authored large but never displayed at full size here |

In practice: any 64-bit Mac (Intel or Apple Silicon, including an 8 GB M1 Air), any Windows 10/11
machine that can run the Simulator, any Linux desktop. **A Mac does not need 32 GB, and does not need
Apple Silicon, to use Kwik** — it needs Apple Silicon (or a lot of patience on CPU) only to *generate*,
which is the cloud's job in this case.

Two things worth knowing about this row of the table:

- **It is not an enforced gate.** Guaardvark refuses a job when the hardware is too small; nothing
  refuses `kwik new`, `kwik build` or the Simulator, because there is nothing to refuse. These numbers
  are what the local path actually loads, not a measured threshold, and `kwik doctor` is the only
  place that will ever print them.
- **Cloud-only is the setup default anyway.** `kwik setup` offers *no local models — cloud generation*
  first, at 0 GB ([01 — Install](01-install.md#the-short-version)). The spec below is the other answer
  to the same question.

## Machine spec

The numbers here are not ours: they are the ones the backend itself enforces, from
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
([01 — Install, Docker](01-install.md#docker)), not on Windows.

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

Removing any of these is a package-manager job, not a `kwik` one:
[01 — Install, Uninstalling](01-install.md#uninstalling).
