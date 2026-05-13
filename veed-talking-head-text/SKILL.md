---
version: 1.0.0
name: veed-talking-head-text
description: >
  Generate a talking head video from a static image and a text script
  using VEED's Fabric 1.0 Text API. VEED's AI voice generator automatically
  converts the script to speech and syncs it to the image. Use when:
  "generate a talking head video from a script", "make this image say this",
  "create a spokesperson video with this text", "turn this script into a
  talking video", "generate a video from text", "I want to type what the
  person says". NOT for: using your own audio file (use veed-talking-head),
  adding subtitles (use veed-subtitles), removing backgrounds
  (use veed-background-removal).
---

# VEED Talking Head — Text to Video (Fabric 1.0)

## What this skill does
Takes a static image and a text script, converts the script to speech
using VEED's AI voice generator, and produces a video where the person's
lips and facial movements sync to the generated audio. No audio file
needed — just type what you want the person to say.
Powered by VEED's Fabric 1.0 Text model on Fal.

## Before you start
The user needs:
- A Fal API key — get one free at https://fal.ai/dashboard/keys
- Set it as an environment variable: export FAL_KEY=your_key_here

## What to ask the user for
1. Image URL — a publicly accessible URL of the image to animate
   (must show a face clearly). Accepted formats: JPG, PNG, WebP, GIF, AVIF
2. Script / text — what do you want the person to say?
   Maximum duration: 5 minutes of speech
3. Resolution — 480p ($0.08/sec) or 720p ($0.15/sec).
   Default to 480p if not specified.

## API call

import fal_client

result = fal_client.run(
    "veed/fabric-1.0/text",
    arguments={
        "image_url": "<IMAGE_URL>",
        "text": "<SCRIPT_TEXT>",
        "resolution": "480p"  # or "720p"
    }
)
print(result["video"]["url"])

Or via curl:

curl -X POST \
  -H "Authorization: Key $FAL_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "image_url": "<IMAGE_URL>",
    "text": "<SCRIPT_TEXT>",
    "resolution": "480p"
  }' \
  "https://fal.run/veed/fabric-1.0/text"

## After the call
- Return the video.url from the response to the user
- Tell them the approximate cost:
  duration in seconds × $0.08 (480p) or × $0.15 (720p)
- 401 error: FAL_KEY is invalid or not set
- 422 error: image URL is not accessible or text is too long

## Pricing
- 480p: $0.08 per second of output video
- 720p: $0.15 per second of output video
- Maximum: 5 minutes (300 seconds) per generation
- Example: 30-second video at 480p = $2.40

Fal model page: https://fal.ai/models/veed/fabric-1.0/text
