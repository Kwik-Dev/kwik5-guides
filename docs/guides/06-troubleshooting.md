---
title: Troubleshooting
description: What each exit code and error body means, and the one thing to do next
order: 6
audience: everyone
status: available (codes are the CLI contract)
updated: 2026-11-13
---

# Troubleshooting

The rule behind all of this: **a failure is a decision with one next action**, not a stack trace.

## `kwik` exit codes

| exit | means | next |
| --- | --- | --- |
| `0` | ok | — |
| `1` | usage, or an environment prerequisite is missing | read the message; it names the command |
| `2` | bad key or bad argument | fix the argument, or `kwik cloud login` |
| `3` | not signed in | `kwik cloud login` |
| `4` | network — the endpoint is unreachable | retry; check `kwik cloud status` for the endpoint |
| `5` | the worker refused the scene | edit the scene (usually an unknown `@tag`) |
| `6` | unknown job id | check the id; jobs are scoped to your customer |
| `7` | the job failed — **credits were returned** | retry, or skip the page |

`status` with no key exits `3`, which is why it is safe in a script.

## Cloud error bodies

| status | `error` | what it means |
| --- | --- | --- |
| `400` | `bad_request` | missing `Idempotency-Key`, unknown tool, bad `units` |
| `400` | `duration_cap` | over 15 s per clip or 60 s per job — the body names both numbers |
| `400` | `asset_expired` | the upload you referenced is older than 24 h |
| `401` | `unauthorized` | missing or invalid Polar key |
| `402` | `insufficient_credits` | read `credits_available`, `estimated_cost`, `top_up_url`; do not loop |
| `404` | `not_found` | unknown job, or not yours |
| `409` | `idempotency_conflict` | same key, different body — use a new key for different work |
| `503` | `runner_unavailable` | could not enqueue; **the hold was released**; retry after `retry_after` |
| `503` | `tool_unavailable` | that tool has no worker yet; also refunded |

Over MCP, every one of these arrives as a **tool result** with `isError: true` and HTTP 200 — read the
payload, not the status code.

## Local failures and what they actually mean

| message | cause | fix |
| --- | --- | --- |
| `no CUDA device: kwik-render needs a GPU attached to its RunPod endpoint` | the **deployed** profile with no GPU | that is correct behaviour in the cloud. Locally use `KWIK_DEVICE=mps`, or the local backend |
| `KWIK_DEVICE=mps but this torch build has no usable MPS backend` | Apple Silicon without MPS in torch | update torch, or install the local backend |
| `KWIK_DEVICE=cpu … KWIK_ALLOW_CPU=1` | a CPU render was requested without the second opt-in | a 40-image book on CPU is a day; install a backend or use the cloud |
| `no upload destination: set R2_* env or send upload.url` | the worker has nowhere to put the PNG | set the upload target; this fails **before** GPU time is spent, on purpose |
| `diffusers X has no ZImagePipeline; Z-Image needs diffusers >= 0.38` | an old diffusers in the image | use the pinned image |
| `ffmpeg` produced no `.ogv`, or the player shows nothing | no libtheora in the build | install `ffmpeg-full`, or point `FFMPEG_THEORA` at a build that has it |
| the app shows an old image but the tree looks right | the install step was skipped | the md5 asset gate exists for this: `kwik asset check` |
| a page shows a second copy of the scene at the edges | a bleed was mirrored instead of cropped | crop from a larger source; assert zero mirrored columns |
| a caption is cut top and bottom | the type is larger than its 1600×240 plate | raise the plate or shrink the type — then look at the frame |
| narration is silent although the YAML declares it | the file was never generated | `kwik` refuses this now: a declared file must exist, or be marked `ungenerated: true` |
| `Cannot connect to the Docker daemon` | the compose path needs Docker; on macOS it also cannot use the GPU | use the native install; Docker GPU works on Linux/WSL2 only |

## When something looks impossible

Three things have cost real afternoons in this project, all of them environmental:

1. **An orphaned process fabricates failures.** A forgotten dev server reconciled a shared database
   and failed jobs another harness had just submitted; orphaned `workerd` children hold no port and
   survive a port-based teardown. If a result makes no sense, look for the process before the code.
2. **A stale log is not evidence.** Read the newest one and check what actually failed — an old log
   with passing assertions and a teardown error reads as a failure, and vice versa.
3. **"Keyed but black" is a decoder, not a keyer.** VP9 alpha is only decoded by `-c:v libvpx-vp9`;
   `ffprobe` will say `yuv420p` and a decoded frame looks fully opaque on the wrong decoder.

## Getting help with something reproducible

Include the **version receipt**, not a screenshot:

```bash
kwik doctor --json
```

It reports the backend version, the ComfyUI commit, torch, diffusers, the model ids, the ffmpeg
encoder, and the cloud endpoint with a key *fingerprint* — never the key. With that, "it does not
work" becomes a testable claim; without it, every answer is a guess.
