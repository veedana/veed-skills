---
version: 1.3.0
name: veed-background-removal
description: >
  Remove the background from a video with VEED's Background Removal API —
  standard, fast, or green-screen mode. Use when the user wants to isolate the
  subject or drop a video's background: "remove the background", "extract the
  subject", "green screen removal". Accepts local paths or URLs.
  NOT for: images (video only).
---

# VEED Background Removal

## What this skill does
Removes the background from a video, returning just the subject. Three modes:
standard, fast, and green screen.

## Before you start
Setup (genmedia CLI + Fal key), the async execution pattern, uploading local
inputs, and common errors are shared across all VEED skills — see
[../COMMON.md](../COMMON.md). This file covers only what's specific to
background removal.

## What to ask the user for

Collect ALL of the following before proceeding:

1. Video — a local file path (e.g. /Users/ana/Desktop/video.mp4) or
   a public URL. Accepted formats: MP4, MOV, WEBM, MKV, AVI
   Tip on Mac: right-click file in Finder → hold Option → Copy as Pathname

2. Mode — which fits their need (sets the endpoint):
   - Standard — best quality. Default. → `veed/video-background-removal`
   - Fast — quicker, for high volume or testing. → `.../fast`
   - Green screen — for green screen footage. → `.../green-screen`

3. Refine foreground edges — `refine_foreground_edges`:
   - ON (`true`) — cleaner edges, higher cost. Default.
   - OFF (`false`) — faster, lower cost.

## Step 1 — Estimate cost and confirm

Fetch the rate for the chosen endpoint variant
(`genmedia pricing veed/video-background-removal --json`). Billed per 30
frames, at a rate that depends on the refine setting:

- Get video duration in seconds (use ffprobe or ask the user)
- Assume 30fps unless the user specifies otherwise
- Estimated cost = (duration_seconds x fps / 30) x price_per_30_frames

Show the estimate and get confirmation. Example (indicative $0.0225/30f Refine
ON, $0.012 OFF): "~$0.45 for a 10-second 30fps video, Refine ON. Proceed?"

## Step 2 — Remove the background

Upload the local video (see ../COMMON.md), then follow the async execution
pattern in ../COMMON.md (schema → run `--async` → poll → download). Use the
endpoint variant from the user's mode choice:

    genmedia run veed/video-background-removal \
      --video_url "<video_url>" \
      --refine_foreground_edges true \
      --async \
      --json

Set `--refine_foreground_edges` to `false` if the user chose OFF. Download the
result to `./outputs/background-removal/`, and return both the path and the URL.

## Errors
Common errors (401 / 422 / 429 / 500) are in ../COMMON.md. Here, a 422 usually
means the video is inaccessible or in an unsupported format.

## Pricing
Run `genmedia pricing veed/video-background-removal --json` (or the fast /
green-screen variant) for the authoritative current rate. Indicative: Refine
ON $0.0225 per 30 frames, Refine OFF $0.012 per 30 frames.

Model pages:
- Standard: https://fal.ai/models/veed/video-background-removal
- Fast: https://fal.ai/models/veed/video-background-removal/fast
- Green screen: https://fal.ai/models/veed/video-background-removal/green-screen
