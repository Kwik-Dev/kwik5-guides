---
title: The local backend
description: Guaardvark + ComfyUI on your own machine, native or Docker, and what each device can actually do
order: 4
audience: power user
status: backend exists upstream; the `kwik` integration is decided, not built
updated: 2026-11-13
---

# The local backend

[Guaardvark](https://github.com/guaardvark) is the local AI backend: Cast subjects and LoRA training,
image generation, an audio foundry, video workflows, upscaling. It is MIT, it is what produced every
asset in `biscuit-comes-home`, and it starts **ComfyUI** as a plugin when a render needs it.

Nothing here is required. With no local backend, every media tool runs in the cloud and costs credits
([03 — The cloud](03-cloud.md)).

## What runs where

| tool | local backend | device order | the honest label |
| --- | --- | --- | --- |
| art (`cast_render`) | Z-Image Turbo + Cast LoRAs | CUDA → MPS → CPU | runs anywhere; a 24 GB card is ~10× a laptop |
| narration (`tts_speak`) | Kokoro-82M (31 voices, default `af_heart`) | CPU-capable | runs anywhere; it is the cheapest call |
| music (`music_bed`) | ACE-Step v1 3.5B, 240 s cap, 48 kHz | CUDA → MPS → CPU | MPS works; CPU is slow |
| clips (`video_clip`) | ComfyUI image-to-video workflows | ComfyUI's choice | small canvases locally; a long clip wants 48 GB |
| Cast training | `lora_trainer`, or `runpod_lora_trainer` | CUDA | minutes on a big card is exactly what a laptop should not do (D12) |
| sound effects | **nothing** | — | a CC0 library, never generated (§1d of the media contract) |
| upscale | Guaardvark's upscaling plugin | CUDA | better than Lanczos on hard edges; free |

`kwik doctor` reports this table **for your machine**, with the backend's own reason when something
cannot run — it does not keep a static list, because a static list goes stale and a stale capability
claim is how an author loses an afternoon.

## Native vs Docker

| | native | Docker |
| --- | --- | --- |
| macOS (Apple Silicon) | ✅ the only GPU path (Metal/MPS) | ❌ Docker cannot pass Metal; containers are CPU-only |
| macOS (Intel) | ✅ CPU-capable, slow | ❌ same |
| Linux + NVIDIA | ✅ | ✅ with `nvidia-container-toolkit` |
| Windows + WSL2 | ✅ | ✅ with `nvidia-container-toolkit` |

The Docker path drives **Guaardvark's own** `docker-compose.yml` + `docker-compose.gpu.yml`, pinned by
commit. It is not a parallel stack that will drift.

## A worker is not the app

The cloud worker wraps Guaardvark's code rather than reimplementing it, and four app-level behaviours
are switched off there, because a serverless worker has no app around it:

| override | why |
| --- | --- |
| GPU orchestrator off | it asks a main backend for VRAM over HTTP; there is none |
| registration off | the plugin POSTs every generated file back to the app as a Document |
| async jobs off | one job per invocation; background queues and retention are app features |
| command timeout ≥ 300 s | the ACE-Step daemon's watchdog kills on 30 s of silence; a 240 s bed outlives it |

That is not trivia: each of those, left on, looks like a model bug when it fails.

## What a local backend must never do

- **Bind anything but loopback.** Guaardvark has no multi-tenant auth. `kwik` must not open or tunnel
  it; the BYOC path (a token-gated tunnel registered in the `nodes` table, D11) is the deliberate
  exception.
- **Download during a job.** Models and voices are fetched at install, never mid-render.
- **Expose a workflow as a contract.** ComfyUI is an implementation detail of the local backend: no
  job envelope carries a graph, and no cloud response names one.

## Provenance, so nobody trusts the wrong number

Every artifact records where it came from:

| label | means |
| `mps/dev` | a dev render on this Mac — exercises the code path, **not** a parity result, **not** a bench number |
| `cuda/local` | the local backend on an NVIDIA card |
| `ada24/bench` | the 24 GB bench the cost model is calibrated on |
| `neon/deployed` | the real service |

MPS cannot satisfy "the same PNG hash as the bench" — parity is same-GPU-class only, because CUDA
does not promise byte identity across cards and this project does not pretend otherwise.
