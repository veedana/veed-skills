---
version: 1.0.0
name: veed-talking-head
description: >
  Generate a talking head video from a static image and audio using VEED's
  Fabric 1.0 API. Animates any face in an image to sync lip movements with
  the provided audio. Use when: "generate a talking head video", "animate
  this photo", "make this image speak", "create a spokesperson video",
  "lipsync this image to audio", "bring this photo to life".
  NOT for: dubbing an existing video, adding subtitles, removing backgrounds.
---

# VEED Talking Head — Fabric 1.0

## What this skill does
Takes a static image of a person (or avatar) and an audio file, and produces
a video where the person's lips and facial movements sync to the audio.
Powered by VEED's Fabric 1.0 model on Fal.

## Before you start
The user needs:
- A Fal API key — get one free at https://fal.ai/dashboard/keys
- Set it as an environment variable: export FAL_KEY=your_key_here

## What to ask the user for
1. Image URL — a publicly accessible URL of the image to animate
   (must show a face clearly). Accepted formats: JPG, PNG, WebP, GIF, AVIF
2. Audio URL — a publicly accessible URL of the audio to sync to.
   Accepted formats: MP3, OGG, WAV, M4A, AAC
3. Resolution — 480p ($0.08/sec) or 720p ($0.15/sec).
   Default to 480p if not specified.

## API call

import fal_client

result = fal_client.run(
    "veed/fabric-1.0",
    arguments={
        "image_url": "<IMAGE_URL>",
        "audio_url": "<AUDIO_URL>",
        "resolution": "480p"
    }
)
print(result["video"]["url"])

Or via curl:

curl -X POST \
  -H "Authorization: Key $FAL_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "image_url": "<IMAGE_URL>",
    "audio_url": "<AUDIO_URL>",
    "resolution": "480p"
  }' \
  "https://fal.run/veed/fabric-1.0"

## After the call
- Return the video.url from the response to the user
- Tell them the approximate cost:
  duration in seconds x $0.08 (480p) or x $0.15 (720p)
- 401 error: FAL_KEY is invalid or not set
- 422 error: image or audio URL is not accessible

## Pricing
- 480p: $0.08 per second of output video
- 720p: $0.15 per second of output video
- Maximum: 5 minutes (300 seconds) per generation
- Fal model page: https://fal.ai/models/veed/fabric-1.0
