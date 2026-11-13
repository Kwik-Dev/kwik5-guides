---
title: Configuration and environment variables
description: Every setting that matters for a Kwik install — where it lives, what it defaults to, and what is deliberately not yours
order: 7
audience: author, power user
status: mixed — CLI config is available; `kwik setup` keys are decided, not built
updated: 2026-11-13
---

# Configuration and environment variables

Three scopes, and they should not be mixed up:

| scope | configured by | you touch it |
| --- | --- | --- |
| **`kwik`** — the CLI | a `0600` JSON file, a few `KWIK_*` variables, flags | yes |
| **the local backend** — Guaardvark + ComfyUI + Audio Foundry | Guaardvark's own `.env` and `AUDIO_FOUNDRY_*` variables | only if you install it |
| **the operator side** — the cloud worker and its database | secrets in Cloudflare / Neon / RunPod | **no** — see the last section |

Precedence, highest first: **explicit flag → environment variable → config file → default.**
Nothing reads a licence key from an environment variable (see "What must never be in an
environment variable").

## 1. `kwik`

### Config file

| | |
| --- | --- |
| macOS | `~/Library/Application Support/kwik/cloud.json` |
| Windows | `%APPDATA%\kwik\cloud.json` |
| Linux | `~/.config/kwik/cloud.json` |
| mode | **`0600`** — created by `kwik cloud login`, never by hand |
| contains | the endpoint, the stored Polar key, and the key's fingerprint |
| printed | the **fingerprint only** (`key:…a1b2`) — never the key, in any output, frame or log |

### Environment

| variable | default | what it does |
| --- | --- | --- |
| `KWIK_CLOUD_URL` | the stored endpoint | overrides the endpoint for this shell — which is how you point the same stored key at a local dry run and at production without logging out |
| `KWIK_NO_DOWNLOADS` | unset | `1` fails instead of prompting for a model download (locked-down machines, CI) |
| `KWIK_MODEL_DIR` | `/models/zimage-turbo` | where the renderer looks for weights (worker/dev) |
| `KWIK_LORA_DIR` | unset | where Cast LoRAs live (`<lora_dir>/<subject_id>.safetensors`) |
| `KWIK_DEVICE` | `cuda` | `cuda` \| `mps` \| `cpu` — the device profile. **Unset means the deployed behaviour**: CUDA or an explicit refusal |
| `KWIK_ALLOW_CPU` | unset | a second opt-in required for `KWIK_DEVICE=cpu`; a 40-image book on CPU is a day |
| `KWIK_LOG_LEVEL` | `INFO` | worker log level |

**`KWIK_DEVICE` is the one that changes what a result *means*.** `mps` exercises the code path and
the seed determinism on this Mac; it is not a parity result and its timings are not bench numbers.
Every artifact carries the label (`mps/dev`, `cuda/local`, `ada24/bench`, `neon/deployed`) for exactly
that reason.

### Exit codes are configuration too

`0` ok · `1` usage / missing prerequisite · `2` bad key or argument · `3` not signed in · `4` network ·
`5` refused · `6` unknown job · `7` job failed (credits returned). `status` with no key exits `3`,
which is what makes it safe in a script. Full table: [06 — Troubleshooting](06-troubleshooting.md).

## 2. The local backend (only if you install it)

Guaardvark keeps its own `.env` at the project root; its plugin launcher sources it. The media-relevant
settings are a small subset of a large app — these are the ones that change a *render*:

| variable | default | what it does |
| --- | --- | --- |
| `FLASK_PORT` | `5000` | the backend's HTTP port. It moves off 5000 on macOS, where AirPlay Receiver claims it — the reference book used `5055` |
| `GUAARDVARK_BACKEND_URL` | `http://localhost:$FLASK_PORT` | where a plugin reports its outputs back to; set it when the port moved |
| `GUAARDVARK_UPLOAD_DIR` | the app's data dir | where generated files land |
| `GUAARDVARK_COMFYUI_URL` | `127.0.0.1:8188` | the ComfyUI instance the plugin starts and drives |
| `GUAARDVARK_COMFYUI_DIR`, `_VENV`, `_IDLE_TIMEOUT`, `COMFYUI_OUTPUT_DIR` | app defaults | where ComfyUI lives, which Python it uses, when it unloads, where it writes |
| `AUDIO_FOUNDRY_DISABLE_BACKENDS` | none | comma-separated backends to switch off — `audio_fx` turns **Stable Audio Open** off, which is the default policy (D15) |
| `AUDIO_FOUNDRY_SAO_MPS` | unset | `1` opts Stable Audio Open into Apple-Silicon MPS. Unset, it refuses with a reason instead of half-working |
| `HF_HOME`, `TORCH_HOME` | framework defaults | model caches; point these at a large disk before the first download |

The Audio Foundry runs as its own service on **port 8206** (`plugins/audio_foundry/plugin.json`), with
its own virtualenv — the app's way of keeping music dependencies away from torch and ComfyUI.

Device order is not an environment variable: ACE-Step picks `CUDA → MPS → CPU`, and the art pipeline
picks per `KWIK_DEVICE`. **`kwik doctor --json` reports what your machine actually chose**, with the
backend's own reason when it cannot — a static table would go stale and a stale capability claim costs
an afternoon.

## 3. Development and CI (this repo)

Only relevant if you are working on `kwik5-cloud` itself. Every harness documents its own variables in
`kwik5-cloud/Makefile`; the ones you are most likely to set:

| variable | used by | what it does |
| --- | --- | --- |
| `DATABASE_URL` | `api/dev.ts`, `make e2e` | the local Postgres (`postgresql://kwik:kwik@localhost:55432/kwik_test`) |
| `DEV_KEY`, `KWIK_DEV_KEYS`, `ADMIN_TOKEN` | `api/dev.ts` | the static keys the dry run accepts (`key:customer` pairs) |
| `FAKE_RENDER_MS` | `api/dev.ts` | how long the fake worker pretends to render |
| `RECONCILE_MS` | `api/dev.ts` | **off unless set**, because a forgotten dev server once reconciled a shared database |
| `ARTIFACT_DIR` | `api/dev.ts` | where the fake worker writes, served at `/art/<key>.png` |

## 4. What must never be in an environment variable

- **The Polar licence key.** It is read from the `0600` file or from a hidden prompt
  (`kwik cloud login`, `--key-stdin`). An env variable lands in shell history, in `ps`, and in every
  child process — and a key that bought credits is money.
- **A prompt, a caption, or a manuscript.** Not configuration, and never logged (plan §6).
- **A customer's storage credential.** The worker writes with an endpoint-scoped credential; nothing
  long-lived is handed to a job.

## 5. Not part of your setup

The cloud side has its own configuration — database, licence-key validation, the GPU queue, the
bucket, and an admin token — and **none of it is something a user sets**. If you are running the
service rather than using it, those variables are documented once, in
[`kwik5-cloud/docs/DEPLOY.md`](../../../kwik5-cloud/docs/DEPLOY.md), next to the command that proves
each one. They are listed there and not here on purpose: a setup guide is not a place to hint that you
might need a queue API key.

## 6. Checking what is actually in effect

```bash
kwik doctor --json          # CLI, backend versions, devices, models, ffmpeg encoder, key fingerprint
kwik cloud status           # endpoint, fingerprint, last job; exit 3 when not signed in
guaardvark health           # the backend's own view: version, index, plugins, workers
guaardvark plugins list     # which plugins are enabled (comfyui, upscaling, lora_trainer, …)
```

`doctor` is the answer to "why is it behaving like that": it prints the *effective* endpoint, the
chosen device, the encoder it will use for `ogv`, and the key fingerprint — without ever printing a key.
