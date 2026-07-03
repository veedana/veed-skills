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
Lip-syncs the face in an image to a provided audio file. Max 30 seconds of
output per generation.

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

Fetch the current rate rather than relying on memorised numbers:

    genmedia pricing veed/fabric-1.0 --json

Fabric is billed per second of output video, at a rate that depends on
resolution. Calculate and show the estimate:

    estimated_cost = audio_duration_seconds x price_per_second

Show the estimate to the user and ask them to confirm before proceeding.
Example (indicative rates $0.08/sec at 480p, $0.15/sec at 720p): "This will
cost approximately $1.50 for a 10-second video at 720p. Shall I proceed?"

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

Once completed, fetch the result and download the video locally in one step
(Fal URLs expire after ~24 hours):

    genmedia status veed/fabric-1.0 <request_id> \
      --download "./outputs/talking-head/{request_id}.{ext}" \
      --json

`--download` saves the file to the given path and still returns `video.url`.
Give the user both the local file path and the URL. If the session was
interrupted, resume here with the same `request_id`; do NOT re-run Step 3.

## After the call
- Return both the local file path and the video.url to the user
- The downloaded file is the durable copy — the Fal URL expires after ~24 hours
- 401 / auth error: Fal key not configured — run `genmedia setup`
- 422 error: image or audio not accessible, wrong format, or audio too long
- 429 error: rate limit — wait a moment and retry
- 500 error: model error on Fal's side — retry once before giving up

## Pricing
Run `genmedia pricing veed/fabric-1.0 --json` for the authoritative current
rate. Indicative rates at time of writing:
- 480p: $0.08 per second of output video
- 720p: $0.15 per second of output video
- Maximum: 30 seconds per generation
- Fal model page: https://fal.ai/models/veed/fabric-1.0
