---
version: 1.2.0
name: veed-background-removal
description: >
  Remove the background from any video using VEED's Background Removal API.
  Use when: "remove background from this video", "remove the background",
  "make the background transparent", "extract the subject", "green screen
  removal". Accepts videos dragged into chat, local file paths, or URLs.
  NOT for: removing backgrounds from images (video only).
---

# VEED Background Removal

## What this skill does
Takes a video and removes the background. Accepts videos dragged into chat,
local file paths, or public URLs. Three modes available.
Powered by VEED's Background Removal API on Fal.

## Before you start
The user needs:
- A Fal API key — get one free at https://fal.ai/dashboard/keys
- Set it as an environment variable: export FAL_KEY=your_key_here

## What to ask the user for
1. Video — drag into chat, provide a local file path, or a public URL.
   Accepted formats: MP4, MOV, WEBM
2. Mode:
   - Standard (veed/video-background-removal) — best quality. Default.
   - Fast (veed/video-background-removal/fast) — quicker, high volume.
   - Green screen (veed/video-background-removal/green-screen) — for
     green screen footage.
3. Refine foreground edges — ON ($0.0225/30 frames) or OFF ($0.012/30 frames).
   Default to ON.

## Step 1 — Handle video input
If the user dragged a video into the chat, save it as a temp file
and upload it to Fal. Use this Python code:

import fal_client
import tempfile
import os

# Save dragged video to a temp file
with tempfile.NamedTemporaryFile(suffix=".mp4", delete=False) as tmp:
    tmp.write(video_bytes)  # video_bytes from the dragged video
    tmp_path = tmp.name

# Upload to Fal storage
video_url = fal_client.upload_file(tmp_path)
os.unlink(tmp_path)  # clean up temp file

If the user provided a local file path, upload directly:
video_url = fal_client.upload_file("<LOCAL_VIDEO_PATH>")

If already a public URL, use it directly — no upload needed.

## Step 2 — Remove the background

import fal_client

result = fal_client.run(
    "veed/video-background-removal",
    arguments={
        "video_url": video_url,
        "refine_foreground_edges": True
    }
)
print(result["video"]["url"])

## After the call
- Return the video.url to the user
- Cost: (total frames / 30) x $0.0225 (Refine ON) or x $0.012 (Refine OFF)
- Example: 10-second video at 30fps = 10 units x $0.0225 = $0.225
- 401 error: FAL_KEY is invalid or not set
- 422 error: video not accessible or format not supported

## Pricing
- Refine ON: $0.0225 per 30 frames
- Refine OFF: $0.012 per 30 frames
- Standard: https://fal.ai/models/veed/video-background-removal
- Fast: https://fal.ai/models/veed/video-background-removal/fast
- Green screen: https://fal.ai/models/veed/video-background-removal/green-screen
