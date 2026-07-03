---
version: 1.3.0
name: veed-talking-head
description: >
  Generate a talking head video from a static image and audio using VEED's
  Fabric 1.0 API. Animates any face in an image to sync lip movements with
  the provided audio. Use when: "generate a talking head video", "animate
  this photo", "make this image speak", "create a spokesperson video",
  "lipsync this image to audio", "bring this photo to life".
  Accepts local file paths or public URLs for image and audio.
  NOT for: dubbing an existing video, adding subtitles, removing backgrounds.
---

# VEED Talking Head — Fabric 1.0

## What this skill does
Takes a static image and audio file and produces a video where the person's
lips sync to the audio. Maximum duration: 30 seconds per generation.
Powered by VEED's Fabric 1.0 model on Fal.

## Before you start
These skills run through the genmedia CLI, which handles model discovery,
file upload, execution, and downloads against Fal.

- Install it once — see https://github.com/fal-ai-community/genmedia-cli
- Configure your Fal API key (get one free at https://fal.ai/dashboard/keys):
      genmedia setup
  Or non-interactively (agents / CI):
      genmedia setup --non-interactive --api-key "$FAL_KEY"

All commands below assume `genmedia` is on your PATH.

## What to ask the user for

Collect ALL of the following before proceeding:

1. Image — a local file path or public URL of the image to animate.
   Must show a face clearly. Accepted formats: JPG, PNG, WebP, GIF, AVIF
   Tip on Mac: right-click file in Finder → hold Option → Copy as Pathname

2. Audio — a local file path or public URL of the audio to sync to.
   Accepted formats: MP3, WAV, M4A, AAC
   Maximum audio duration: 30 seconds per generation

3. Resolution — ask which the user prefers:
   - 480p — $0.08 per second of output video
   - 720p — $0.15 per second of output video
   Default to 480p if not specified.

## Step 1 — Handle file inputs

If the user gave a local file path for the image, upload it to Fal's CDN:

    genmedia upload /path/to/image.jpg --json

Copy the returned "url". Do the same for the audio if it's a local file:

    genmedia upload /path/to/audio.mp3 --json

If either input is already a public URL, use it directly — no upload needed.

## Step 2 — Show cost estimate before proceeding

Before making the API call, calculate and show the estimated cost:

estimated_cost = audio_duration_seconds x price_per_second

Where price_per_second is $0.08 (480p) or $0.15 (720p).

Show the estimate to the user and ask them to confirm before proceeding.
Example: "This will cost approximately $1.50 for a 10-second video at 720p.
Shall I proceed?"

## Step 3 — Generate the video

First confirm the endpoint's current input fields (Fal schemas can change):

    genmedia schema veed/fabric-1.0 --json

Then run Fabric 1.0 asynchronously, using the exact field names from the
schema. `--async` submits the job and returns immediately with a `request_id`:

    genmedia run veed/fabric-1.0 \
      --image_url "<image_url>" \
      --audio_url "<audio_url>" \
      --resolution 480p \
      --async \
      --json

Set `--resolution` to `720p` if the user chose it. The flags above reflect
the expected schema — if `genmedia schema` shows a different name, follow it.

IMPORTANT — record the `request_id` and show it to the user. The run is
billed once submitted, so if the session is interrupted you can re-fetch the
result with `status` instead of paying to run it again.

## Step 4 — Poll for the result

Check the job with the recorded `request_id` until it reports completed:

    genmedia status veed/fabric-1.0 <request_id> --json

The completed result JSON contains `video.url` — return it to the user. If
the session was interrupted, resume here with the same `request_id`; do NOT
re-run Step 3.

## After the call
- Return the video.url to the user
- Warn the user that Fal URLs expire after ~24 hours — download locally
  if they want to keep it
- 401 / auth error: Fal key not configured — run `genmedia setup`
- 422 error: image or audio not accessible, wrong format, or audio too long
- 429 error: rate limit — wait a moment and retry
- 500 error: model error on Fal's side — retry once before giving up

## Pricing
- 480p: $0.08 per second of output video
- 720p: $0.15 per second of output video
- Maximum: 30 seconds per generation
- Fal model page: https://fal.ai/models/veed/fabric-1.0
