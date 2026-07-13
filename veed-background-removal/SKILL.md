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
Removes the background from a video, returning just the subject — as a
transparent WebM (default) or an MP4 on black. Three modes: standard, fast,
and green screen.

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

2. Mode — which fits their need (sets the endpoint AND which options apply):
   - Standard — best quality. Default. → `veed/video-background-removal`
   - Fast — quicker, for high volume or testing. → `.../fast`
   - Green screen — for footage shot on a green/blue screen. → `.../green-screen`

3. Output codec — `output_codec`, applies to all modes:
   - `vp9` (default) — WebM with a transparent alpha channel. Use when the
     subject will be composited onto a new background.
   - `h264` — MP4, no transparency (subject on black). Use for direct playback.

4. Mode-specific options:
   - Standard / Fast:
     - `refine_foreground_edges` (default `true`) — cleaner edges, higher cost;
       `false` is faster and cheaper.
     - `subject_is_person` (default `true`) — set `false` if the subject isn't
       a person (product, animal, object).
   - Green screen (note: these two do NOT apply — the endpoint has neither):
     - `spill_suppression_strength` (0–1, default `0.8`) — how aggressively to
       remove green/blue spill on the subject's edges. Raise toward 1 for heavy
       spill; lower if edges look eroded.

### For best results
Flag any of these to the user before spending on a run (advise, don't auto-fix):
- **Keep one stable subject** — the model works frame-by-frame, so a consistent subject removes cleanly; big focus or subject changes mid-clip cause flicker.
- **Contrast helps, but plain isn't required** — it handles natural, busy backgrounds; the more the subject stands out, the sharper the edges.
- **Hair and fine detail are the hard cases** — keep `refine_foreground_edges` on when they matter.
- **Use unprocessed source** — footage already run through virtual cameras or filters removes worse; use the original.
- **Green-screen mode is only for actual green/blue-screen footage** — otherwise use standard or fast.

## Step 1 — Estimate cost and confirm

Fetch the rate for the chosen endpoint variant
(`genmedia pricing <endpoint> --json`). Billed per 30 frames. Standard and Fast
have different per-30-frame rates, each varying with `refine_foreground_edges`;
green-screen has no refine setting, so price it from its own `genmedia pricing`.

- Get video duration in seconds (use ffprobe or ask the user)
- Assume 30fps unless the user specifies otherwise
- Estimated cost = (duration_seconds x fps / 30) x price_per_30_frames

Show the estimate and get confirmation. Example (indicative — Standard, refine
ON at $0.0225/30f): "~$0.45 for a 10-second 30fps clip, Standard with refine
ON. Proceed?"

## Step 2 — Remove the background

Upload the local video (see ../COMMON.md), then follow the async execution
pattern in ../COMMON.md (schema → run `--async` → poll → download). The flags
differ by mode — pass only the ones that apply.

Standard / Fast:

    genmedia run veed/video-background-removal \
      --video_url "<video_url>" \
      --refine_foreground_edges true \
      --subject_is_person true \
      --output_codec vp9 \
      --async \
      --json

Green screen (no `refine_foreground_edges` / `subject_is_person`):

    genmedia run veed/video-background-removal/green-screen \
      --video_url "<video_url>" \
      --spill_suppression_strength 0.8 \
      --output_codec vp9 \
      --async \
      --json

Adjust per the user's choices (`--output_codec h264`,
`--refine_foreground_edges false`, `--subject_is_person false`,
`--spill_suppression_strength <0-1>`), swap in the fast endpoint if chosen,
then download the result to `./outputs/background-removal/` and return both
the path and the URL.

## Errors
Common errors (401 / 422 / 429 / 500) are in ../COMMON.md. Here, a 422 usually
means the video is inaccessible or in an unsupported format.

## Pricing
Run `genmedia pricing veed/video-background-removal --json` (or the fast /
green-screen variant) for the authoritative current rate. Standard and Fast
have different rates; indicative per 30 frames:
- Standard: refine ON $0.0225 / OFF $0.015
- Fast: refine ON $0.012 / OFF $0.008
- Green screen: ~$0.012–0.015 (no refine setting)

Model pages:
- Standard: https://fal.ai/models/veed/video-background-removal
- Fast: https://fal.ai/models/veed/video-background-removal/fast
- Green screen: https://fal.ai/models/veed/video-background-removal/green-screen
