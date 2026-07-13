---
version: 1.3.0
name: veed-talking-head
description: >
  Lip-sync a face in a static image to an audio file with VEED's Fabric 1.0 —
  a talking head / spokesperson video. Use when the user wants to animate a
  photo to speak from an audio track: "make this image talk", "lipsync this
  photo to audio", "create a spokesperson video". Image + audio, local paths
  or URLs. NOT for: generating speech from text (use veed-talking-head-text),
  dubbing an existing video, subtitles, or background removal.
---

# VEED Talking Head — Fabric 1.0

## What this skill does
Lip-syncs the face in an image to a provided audio file. Max 30 seconds of
output per generation.

## Before you start
Setup (genmedia CLI + Fal key), the async execution pattern, uploading local
inputs, and common errors are shared across all VEED skills — see
[../COMMON.md](../COMMON.md). This file covers only what's specific to Fabric
1.0 talking heads.

## What to ask the user for

Collect ALL of the following before proceeding:

1. Image — a local file path or public URL of the image to animate.
   Must show a face clearly. Accepted formats: JPG, PNG, WebP, GIF, AVIF
   Tip on Mac: right-click file in Finder → hold Option → Copy as Pathname

2. Audio — a local file path or public URL of the audio to sync to.
   Accepted formats: MP3, WAV, M4A, AAC
   Maximum audio duration: 30 seconds per generation

3. Resolution — 480p or 720p (see Pricing). Default to 480p if not specified.

### For best results
Flag any of these to the user before spending on a run (advise, don't auto-fix):
- **One person only** — Fabric animates a single face; images with multiple people aren't supported well. Crop to the subject if needed.
- **Clear, well-lit face** — the face must be plainly visible for lip detection. Side angles are fine; avoid hats or heavy glasses that hide features.
- **Match the expression to the audio** — the starting frame's emotion should fit the tone of the speech; a big smile over somber audio looks uncanny.
- **High-res image** — aim for ≥2048×1152 px and ≤6MB.
- **English syncs best** — lip-sync is strongest in English; some languages (e.g. Hebrew) are noticeably weaker.

## Step 1 — Estimate cost and confirm

Fabric is billed per second of output video, at a rate that depends on
resolution. Fetch the rate (`genmedia pricing veed/fabric-1.0 --json`), then:

    estimated_cost = audio_duration_seconds x price_per_second

Show the estimate and get confirmation before running. Example (indicative
$0.08/sec at 480p, $0.15/sec at 720p): "~$1.50 for a 10-second 720p video.
Proceed?"

## Step 2 — Generate the video

Upload any local image/audio (see ../COMMON.md), then follow the async
execution pattern in ../COMMON.md (schema → run `--async` → poll → download).
The endpoint is `veed/fabric-1.0`:

    genmedia run veed/fabric-1.0 \
      --image_url "<image_url>" \
      --audio_url "<audio_url>" \
      --resolution 480p \
      --async \
      --json

Use `720p` if the user chose it. Download the result to
`./outputs/talking-head/`, and return both the local path and the URL.

## Errors
Common errors (401 / 422 / 429 / 500) are in ../COMMON.md. Here, a 422 usually
means the image or audio is inaccessible, in a wrong format, or the audio
exceeds the 30-second cap.

## Pricing
Run `genmedia pricing veed/fabric-1.0 --json` for the authoritative current
rate. Indicative: 480p $0.08/sec, 720p $0.15/sec; max 30s per generation.
Fal model page: https://fal.ai/models/veed/fabric-1.0
