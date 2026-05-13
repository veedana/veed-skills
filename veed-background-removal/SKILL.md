---
version: 1.1.0
name: veed-background-removal
description: >
  Remove the background from any video using VEED's Background Removal API
  on Fal. Use when: "remove background from this video", "remove the
  background", "make the background transparent", "extract the subject",
  "green screen removal". Accepts both URLs and local file paths.
  NOT for: removing backgrounds from images (video only).
---

# VEED Background Removal

## What this skill does
Takes a video and removes the background, returning a clean video with
just the subject. Accepts both public URLs and local files from the
user's computer. Three modes available.
Powered by VEED's Background Removal API on Fal.

## Before you start
The user needs:
- A Fal API key — get one free at https://fal.ai/dashboard/keys
- Set it as an environment variable: export FAL_KEY=your_key_here

## What to ask the user for
1. Video — a local file path (e.g. /Users/ana/Desktop/video.mp4) or
   a public URL. Accepted formats: MP4, MOV, WEBM
2. Mode:
   - Standard (veed/video-background-removal) — best quality. Default.
   - Fast (veed/video-background-removal/fast) — quicker, high volume.
   - Green screen (veed/video-background-removal/green-screen) —
     for green screen footage.
3. Refine foreground edges — ON ($0.0225/30 frames) or OFF ($0.012/30 frames).
   Default to ON.

## Step 1 — Upload local video to Fal (if needed)
If the user provides a local file path instead of a URL, upload it first.

import fal_client

video_url = fal_client.upload_file("<LOCAL_VIDEO_PATH>")

If already a URL, skip this step and use the URL directly.

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
- Return the video.url from the response to the user
- Approximate cost: (total frames / 30) x $0.0225 (Refine ON)
  or x $0.012 (Refine OFF)
  Example: 10-second video at 30fps = 300 frames = 10 units x $0.0225 = $0.225
- 401 error: FAL_KEY is invalid or not set
- 422 error: video not accessible or format not supported

## Pricing
- Refine ON: $0.0225 per 30 frames
- Refine OFF: $0.012 per 30 frames
- Standard: https://fal.ai/models/veed/video-background-removal
- Fast: https://fal.ai/models/veed/video-background-removal/fast
- Green screen: https://fal.ai/models/veed/video-background-removal/green-screen
