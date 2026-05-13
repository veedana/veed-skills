---
version: 1.2.0
name: veed-talking-head-text
description: >
  Generate a talking head video from a static image and a text script
  using VEED's Fabric 1.0 Text API. VEED's AI voice generator converts
  the script to speech automatically. Use when: "generate a talking head
  video from a script", "make this image say this", "create a spokesperson
  video with this text", "I want to type what the person says".
  Accepts images dragged into chat, local file paths, or public URLs.
  NOT for: using your own audio file (use veed-talking-head).
---

# VEED Talking Head — Text to Video (Fabric 1.0)

## What this skill does
Takes a static image and a text script. VEED's AI voice generator converts
the text to speech and syncs it to the image. No audio file needed.
Accepts images dragged into chat, local file paths, or public URLs.
Powered by VEED's Fabric 1.0 Text model on Fal.

## Before you start
The user needs:
- A Fal API key — get one free at https://fal.ai/dashboard/keys
- Set it as an environment variable: export FAL_KEY=your_key_here

## What to ask the user for
1. Image — drag into chat, provide a local file path, or a public URL.
2. Script / text — what do you want the person to say?
3. Resolution — 480p ($0.08/sec) or 720p ($0.15/sec). Default: 480p.
4. Voice description (optional) — any voice characteristics you want.
   Simple: "British accent", "Confident", "Friendly"
   Detailed: "Confident female voice, mid-30s, warm and professional tone"
   If not specified, VEED auto-generates a voice from the image.

## Step 1 — Handle image input
If the user dragged an image into the chat, save it as a temporary file
and upload it to Fal. Use this Python code:

import fal_client
import tempfile
import os

# Save dragged image to a temp file
with tempfile.NamedTemporaryFile(suffix=".jpg", delete=False) as tmp:
    tmp.write(image_bytes)  # image_bytes from the dragged image
    tmp_path = tmp.name

# Upload to Fal storage
image_url = fal_client.upload_file(tmp_path)
os.unlink(tmp_path)  # clean up temp file

If the user provided a local file path, upload directly:
image_url = fal_client.upload_file("<LOCAL_FILE_PATH>")

If already a public URL, use it directly — no upload needed.

## Step 2 — Generate the video

import fal_client

result = fal_client.run(
    "veed/fabric-1.0/text",
    arguments={
        "image_url": image_url,
        "text": "<SCRIPT_TEXT>",
        "resolution": "480p",
        "voice_description": "<VOICE_DESCRIPTION>" 
    }
)
print(result["video"]["url"])

## After the call
- Return the video.url to the user
- Cost: duration in seconds x $0.08 (480p) or x $0.15 (720p)
- 401 error: FAL_KEY is invalid or not set
- 422 error: image not accessible or text too long

## Pricing
- 480p: $0.08 per second
- 720p: $0.15 per second
- Max duration: 5 minutes
- Fal model page: https://fal.ai/models/veed/fabric-1.0/text
