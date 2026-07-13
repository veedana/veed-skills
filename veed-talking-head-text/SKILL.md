---
version: 1.4.0
name: veed-talking-head-text
description: >
  Turn a static image + a text script into a lip-synced talking head video
  with VEED's Fabric 1.0 Text — VEED generates the speech, no audio file
  needed. Use when the user wants a spokesperson to say typed text: "make this
  photo say '...'", "talking head from a script", "spokesperson video from
  text". Image is a local path or URL. NOT for: using your own audio file
  (use veed-talking-head).
---

# VEED Talking Head — Text to Video (Fabric 1.0)

## What this skill does
Generates speech from a text script and lip-syncs it to the image — no audio
file needed. Max 30 seconds of output per generation.

## Before you start
Setup (genmedia CLI + Fal key), the async execution pattern, uploading local
inputs, and common errors are shared across all VEED skills — see
[../COMMON.md](../COMMON.md). This file covers only what's specific to Fabric
1.0 Text.

## What to ask the user for

IMPORTANT: Collect ALL four inputs before proceeding. Do not skip any.

1. Image — a local file path or public URL of the image to animate.
   Must show a face clearly. Accepted formats: JPG, PNG, WebP, GIF, AVIF
   Tip on Mac: right-click file in Finder → hold Option → Copy as Pathname

2. Script — what do you want the person to say?
   Keep it short — maximum 30 seconds of speech per generation.

3. Voice description — ALWAYS ask this question explicitly. Do not skip it.
   Say to the user: "How would you like the voice to sound? You can use
   simple descriptors like 'British accent' or 'Confident and warm', or
   something more detailed like 'Professional female voice, mid-30s, calm
   and authoritative tone'. If you have no preference, just say 'auto'
   and VEED will generate a voice based on the image."
   - If the user provides a description → store it as voice_description
   - If the user says 'auto', 'skip', 'no preference', or similar → do NOT
     pass a voice_description flag in the run command

4. Resolution — 480p or 720p (see Pricing). Default to 480p if not specified.

### For best results
Flag any of these to the user before spending on a run (advise, don't auto-fix):
- **One person only** — Fabric animates a single face; images with multiple people aren't supported well. Crop to the subject if needed.
- **Clear, well-lit face** — the face must be plainly visible for lip detection. Side angles are fine; avoid hats or heavy glasses that hide features.
- **Match the image to the script's tone** — the starting expression should fit how the script sounds; don't pair a big grin with a serious script.
- **High-res image** — aim for ≥2048×1152 px and ≤6MB.
- **English syncs best** — lip-sync is strongest in English; some languages (e.g. Hebrew) are noticeably weaker.

## Step 1 — Estimate cost and confirm

Fetch the rate (`genmedia pricing veed/fabric-1.0/text --json`). Fabric Text
is billed per second of output video by resolution; estimate the duration
from the script (roughly 130 words per minute):

    estimated_seconds = word_count / 130 x 60
    estimated_cost = estimated_seconds x price_per_second

Show the estimate and get confirmation. Example (indicative $0.08/sec at 480p,
$0.15/sec at 720p): "~20 words ≈ 9 seconds ≈ $0.72 at 720p. Proceed?"

## Step 2 — Generate the video

Upload the local image (see ../COMMON.md), then follow the async execution
pattern in ../COMMON.md (schema → run `--async` → poll → download). The
endpoint is `veed/fabric-1.0/text`. Include `--voice_description` ONLY if the
user gave one:

    genmedia run veed/fabric-1.0/text \
      --image_url "<image_url>" \
      --text "<script text>" \
      --resolution 480p \
      --async \
      --json

Add `--voice_description "<voice description>"` when provided, and use `720p`
if chosen. Download the result to `./outputs/talking-head-text/`, and return
both the local path and the URL.

## Errors
Common errors (401 / 422 / 429 / 500) are in ../COMMON.md. Here, a 422 usually
means the image is inaccessible, in a wrong format, or the script exceeds the
30-second cap.

## Pricing
Run `genmedia pricing veed/fabric-1.0/text --json` for the authoritative
current rate. Indicative: 480p $0.08/sec, 720p $0.15/sec; max 30s per
generation. Fal model page: https://fal.ai/models/veed/fabric-1.0/text
