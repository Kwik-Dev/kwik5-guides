---
title: Media specs
description: The numbers — frames, formats, codecs, alpha, durations, sizes, retention
order: 5
audience: author, designer, agent
status: decided (D13–D18, media-requirements.md) — partly built
updated: 2026-11-13
---

# Media specs

Everything on this page is a constraint you can design against. Each one comes from a decision or
from a decoder; none of them is a style preference.

## Geometry

| | value | why |
| page frame | **1920 × 1280** (3:2) | every layer coordinate is in these pixels; changing it invalidates every PSD |
| device plate | **2776 × 1440** (1.928:1) | the Simulator/canvas aspect. The bleed exists only to cover this difference |
| render preset `page` | **1536 × 1024** | exactly 0.8× of the frame — the single upscale to 1920×1280 happens on your machine |
| design space | **480 × 320 at 2×** | `setProps` applies ×0.25 plus a content-centre offset; layer props are PSD pixels |
| presets | `page` 1536×1024 · `square` 1024² · `portrait` 1024×1536 · `sprite` 1024² | sizes snap to 16 px and are clamped to 2688 px / 2048² |

Two invariants, because breaking them is invisible until someone looks at the page:

- **a bleed is a crop of a larger source** (2776×1852), never a mirror, tile or outpaint. A reflection
  duplicates the picture by definition; mirror-*tiling* reflects the opposite edge, which is how a
  second cottage appeared on page 1.
- **no `xScale` / `yScale` in layer props.** Per-axis scale is what made the geometry un-debuggable;
  the plugin applies the content scale.

## Images

| | |
| --- | --- |
| delivered | PNG, RGBA (alpha preserved) |
| layers | one PNG per layer, one Lua file per layer, both from the same `page.layers.yaml` |
| PSD | **PSD v1 (8BPS)**, flattened composite + one layer record per YAML entry, in PSD z-order; names, visibility, opacity and blend survive; no 16/32-bit channels in v1 |
| Adobe needed | **none** — the PSD is generated, and CI opens it with `psd-tools` (a dev-only dependency) |
| upscale engines | `lanczos` (default, deterministic, free) · `local` (Guaardvark plugin) · `cloud` (1 credit/image) |

## Audio

| | |
| --- | --- |
| narration | **WAV, mono PCM** — no lossy step; Solar2D loads it directly |
| music beds | **MP3**, stereo 44.1 kHz, up to 240 s, loopable |
| sound effects | **WAV** one-shots, ≤ 3 s, from a curated CC0 library — never generated |
| billing unit | minutes of produced audio, rounded up: 30 s = 1 credit, 61 s = 2 |
| a declared file that does not exist | a **build error**, not silence — plus an explicit `ungenerated: true` if you mean it |

## Video

| | |
| --- | --- |
| master | **MP4**, H.264, `yuv420p` |
| for the app | **OGV**, Theora, `yuv420p` — the only format `plugin.movie` decodes |
| alpha | **a transparent video is not possible**: `plugin.movie` renders through theoraplay, which maps one pixel format (`TH_PF_420`), and Theora has no alpha plane |
| transparency, the working way | render on a flat **`#00B140`** field, then `filter.chromaKey` at runtime on the object that holds the MovieTexture |
| do not do this | key a clip to `.webm` — Solar2D cannot play it |
| aspect | **16:9 only** unless a model declares otherwise; the backend reshapes to the nearest declared ratio within 6 %, so a 3:2 request silently returns 16:9 cropped |
| duration | 6 s default, **15 s hard** per clip, **60 s per job** (a 4 s clip ≈ 90 GPU-seconds at the plan's own rate; 30 s would exceed the endpoint's 600 s execution limit) |
| decoder constraints | even dimensions; ≤ 2 audio channels; fps comfortably under `config.lua`'s fps, because playback advances one decoded frame per `enterFrame` |
| encoder prerequisite | an **ffmpeg built with libtheora + libvorbis**. Stock Homebrew `ffmpeg` no longer ships libtheora; `ffmpeg-full` does. `kwik` reports which encoder it will use and refuses rather than emit an unloadable file |
| real alpha motion | an optional **alpha bundle** from the same job: RGBA PNG sequence (`video/<id>@alpha/`), or VP9 `.webm` / ProRes 4444 `.mov` for editors — **not** playable by `plugin.movie`, but a frame animation is |

## Storage and retention

| prefix | content | lifetime |
| --- | --- | --- |
| `art/` | renders | 30 days |
| `up/` | cloud upscales | 30 days |
| `tts/`, `audio/` | narration, beds, effects | 30 days |
| `video/` | clips, optional alpha bundle | 30 days |
| `in/` | **your uploads** | **24 hours** |
| `lora/` | Cast LoRAs | while the account is active; deleted on request within 24 h |

Retention is a **lifecycle rule on the bucket, not a cron** — a delete-cron is a way to lose someone's
book. `output_url` is a stable CDN URL; nothing you are shown expires in 15 minutes.

## Uploads

| | |
| --- | --- |
| used for | `upscale` of an image the service does not already have |
| limit | **≤ 40 MB**, **≤ 50 megapixels** (decompression bombs), PNG / JPEG / WebP, single frame |
| scoping | an asset belongs to your customer; a job may only reference your own |
| free | yes — the upload costs nothing, the upscale job is what is metered |
| expired asset | `400 asset_expired`, nothing charged |

## What is never in a log

No prompt, no caption, no OCR text, no narration text, no music prompt, no licence key. Logs carry
`job_id`, units, durations, sizes, seeds, model ids and short hashes. That is a requirement (plan §6),
not a policy that may drift with a convenient `print`.
