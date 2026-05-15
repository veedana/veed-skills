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
The user needs:
- A Fal API key — get one free at https://fal.ai/dashboard/keys
- Set it as an environment variable: export FAL_KEY=your_key_here

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

If the user provided a local file path for the image, upload it to Fal:

import fal_client
image_url = fal_client.upload_file(image_path)

If the user provided a local file path for the audio, upload it:

audio_url = fal_client.upload_file(audio_path)

If either is already a public URL, use it directly — no upload needed.

## Step 2 — Show cost estimate before proceeding

Before making the API call, calculate and show the estimated cost:

estimated_cost = audio_duration_seconds x price_per_second

Where price_per_second is $0.08 (480p) or $0.15 (720p).

Show the estimate to the user and ask them to confirm before proceeding.
Example: "This will cost approximately $1.50 for a 10-second video at 720p.
Shall I proceed?"

## Step 3 — Generate the video

Use fal_client.subscribe() to handle progress and avoid timeouts:

import fal_client

def on_queue_update(update):
    if hasattr(update, "logs"):
        for log in update.logs:
            print(log["message"])

result = fal_client.subscribe(
    "veed/fabric-1.0",
    arguments={
        "image_url": image_url,
        "audio_url": audio_url,
        "resolution": resolution
    },
    with_logs=True,
    on_queue_update=on_queue_update
)

print(result["video"]["url"])

## After the call
- Return the video.url to the user
- Warn the user that Fal URLs expire after ~24 hours — download locally
  if they want to keep it
- 401 error: FAL_KEY is invalid or not set
- 422 error: image or audio not accessible, wrong format, or audio too long
- 429 error: rate limit — wait a moment and retry
- 500 error: model error on Fal's side — retry once before giving up

## Pricing
- 480p: $0.08 per second of output video
- 720p: $0.15 per second of output video
- Maximum: 30 seconds per generation
- Fal model page: https://fal.ai/models/veed/fabric-1.0
