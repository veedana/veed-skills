---
version: 1.1.0
name: veed-talking-head-text
description: >
  Generate a talking head video from a static image and a text script
  using VEED's Fabric 1.0 Text API. VEED's AI voice generator automatically
  converts the script to speech and syncs it to the image. Use when:
  "generate a talking head video from a script", "make this image say this",
  "create a spokesperson video with this text", "turn this script into a
  talking video", "I want to type what the person says".
  Accepts both URLs and local file paths for the image.
  NOT for: using your own audio file (use veed-talking-head).
---

# VEED Talking Head — Text to Video (Fabric 1.0)

## What this skill does
Takes a static image and a text script, converts the script to speech
using VEED's AI voice generator, and produces a video where the person's
lips and facial movements sync to the generated audio.
Accepts both public URLs and local files from the user's computer.
Powered by VEED's Fabric 1.0 Text model on Fal.

## Before you start
The user needs:
- A Fal API key — get one free at https://fal.ai/dashboard/keys
- Set it as an environment variable: export FAL_KEY=your_key_here

## What to ask the user for
1. Image — a local file path (e.g. /Users/ana/Desktop/photo.jpg) or a
   public URL. Must show a face clearly.
   Accepted formats: JPG, PNG, WebP, GIF, AVIF
2. Script / text — what do you want the person to say?
   Maximum duration: 5 minutes of speech
3. Resolution — 480p ($0.08/sec) or 720p ($0.15/sec).
   Default to 480p if not specified.

## Step 1 — Upload local image to Fal (if needed)
If the user provides a local file path instead of a URL, upload it first.

import fal_client

image_url = fal_client.upload_file("<LOCAL_IMAGE_PATH>")

If already a URL, skip this step and use the URL directly.

## Step 2 — Generate the video

import fal_client

result = fal_client.run(
    "veed/fabric-1.0/text",
    arguments={
        "image_url": image_url,
        "text": "<SCRIPT_TEXT>",
        "resolution": "480p"  # or "720p"
    }
)
print(result["video"]["url"])

## After the call
- Return the video.url from the response to the user
- Tell them the approximate cost:
  duration in seconds x $0.08 (480p) or x $0.15 (720p)
- 401 error: FAL_KEY is invalid or not set
- 422 error: image not accessible or text too long

## Pricing
- 480p: $0.08 per second of output video
- 720p: $0.15 per second of output video
- Maximum: 5 minutes (300 seconds) per generation
- Fal model page: https://fal.ai/models/veed/fabric-1.0/text
