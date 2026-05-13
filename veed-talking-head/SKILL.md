---
version: 1.2.0
name: veed-talking-head
description: >
  Generate a talking head video from a static image and audio using VEED's
  Fabric 1.0 API. Animates any face in an image to sync lip movements with
  the provided audio. Use when: "generate a talking head video", "animate
  this photo", "make this image speak", "create a spokesperson video",
  "lipsync this image to audio", "bring this photo to life".
  Accepts images dragged into chat, local file paths, or public URLs.
  NOT for: dubbing an existing video, adding subtitles, removing backgrounds.
---

# VEED Talking Head — Fabric 1.0

## What this skill does
Takes a static image and audio file, and produces a video where the person's
lips sync to the audio. Accepts images dragged into chat, local file paths,
or public URLs. Powered by VEED's Fabric 1.0 model on Fal.

## Before you start
The user needs:
- A Fal API key — get one free at https://fal.ai/dashboard/keys
- Set it as an environment variable: export FAL_KEY=your_key_here

## What to ask the user for
1. Image — drag into chat, provide a local file path, or a public URL.
2. Audio — drag into chat, provide a local file path, or a public URL.
   Accepted formats: MP3, OGG, WAV, M4A, AAC
3. Resolution — 480p ($0.08/sec) or 720p ($0.15/sec). Default: 480p.

## Step 1 — Handle image input
If the user dragged an image into the chat, save it as a temporary file
and upload it to Fal. Use this Python code:

import fal_client
import tempfile
import os

# Save the image from chat to a temp file
with tempfile.NamedTemporaryFile(suffix=".jpg", delete=False) as tmp:
    tmp.write(image_bytes)  # image_bytes from the dragged image
    tmp_path = tmp.name

# Upload to Fal storage
image_url = fal_client.upload_file(tmp_path)
os.unlink(tmp_path)  # clean up temp file

If the user provided a local file path, upload it directly:
image_url = fal_client.upload_file("<LOCAL_FILE_PATH>")

If already a public URL, use it directly — no upload needed.

## Step 2 — Handle audio input
Same approach as image above. If dragged into chat or local path:
audio_url = fal_client.upload_file("<LOCAL_AUDIO_PATH_OR_TEMP>")
If already a public URL, use it directly.

## Step 3 — Generate the video

import fal_client

result = fal_client.run(
    "veed/fabric-1.0",
    arguments={
        "image_url": image_url,
        "audio_url": audio_url,
        "resolution": "480p"
    }
)
print(result["video"]["url"])

## After the call
- Return the video.url to the user
- Cost: duration in seconds x $0.08 (480p) or x $0.15 (720p)
- 401 error: FAL_KEY is invalid or not set
- 422 error: file not accessible or wrong format

## Pricing
- 480p: $0.08 per second
- 720p: $0.15 per second
- Max duration: 5 minutes
- Fal model page: https://fal.ai/models/veed/fabric-1.0
