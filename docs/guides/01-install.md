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
  Local backend — choose what to install (nothing is downloaded until you pick)

  ▸ 1  nothing — cloud only                    0 GB    (you'll need a Polar key)
    2  page art only         Z-Image Turbo     ~12 GB   (recommended for local use)
    3  art + narration       + Kokoro          ~13 GB
    4  art + narration + music  + ACE-Step     ~27 GB
    5  everything            + a video model   ~41 GB
```

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
  art       no local model (12 GB) — cloud only
  narration no local model (1 GB) — cloud only
  music     no local model (14 GB) — cloud only
  clips     no local model (14 GB) — cloud only
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
