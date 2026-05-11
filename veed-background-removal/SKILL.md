---
version: 1.0.0
name: veed-background-removal
description: >
  Remove the background from any video using VEED's Background Removal API
  on Fal. Returns a clean video with the background removed. Use when:
  "remove background from this video", "remove the background", "make the
  background transparent", "extract the subject from this video", "green
  screen removal", "remove green screen". Three modes available: standard,
  fast, and green screen.
  NOT for: removing backgrounds from images (video only), generating
  talking head videos (use veed-talking-head), adding subtitles
  (use veed-subtitles).
---

# VEED Background Removal

## What this skill does
Takes a video and removes the background, returning a clean video with
just the subject. Three modes available depending on speed and use case.
Powered by VEED's Background Removal API on Fal.

## Before you start
The user needs:
- A Fal API key — get one free at https://fal.ai/dashboard/keys
- Set it as an environment variable: export FAL_KEY=your_key_here

## What to ask the user for
1. Video URL — a publicly accessible URL of the video.
   Accepted formats: MP4, MOV, WEBM
2. Mode — which fits their need:
   - Standard (veed/video-background-removal) — best quality. Default.
   - Fast (veed/video-background-removal/fast) — quicker, slightly lower
     quality. Good for high volume or testing.
   - Green screen (veed/video-background-removal/green-screen) —
     specialized for green screen footage.
3. Refine foreground edges — ON or OFF.
   ON = cleaner edges ($0.0225 per 30 frames)
   OFF = faster, lower cost ($0.012 per 30 frames)
   Default to ON.

## API call

import fal_client

result = fal_client.run(
    "veed/video-background-removal",
    arguments={
        "video_url": "<VIDEO_URL>",
        "refine_foreground_edges": True
    }
)
print(result["video"]["url"])

Or via curl:

curl -X POST \
  -H "Authorization: Key $FAL_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "video_url": "<VIDEO_URL>",
    "refine_foreground_edges": true
  }' \
  "https://fal.run/veed/video-background-removal"

## After the call
- Return the video.url from the response to the user
- Approximate cost: (total frames / 30) x $0.0225 (Refine ON)
  or x $0.012 (Refine OFF)
  Example: 10-second video at 30fps = 300 frames = 10 units x $0.0225 = $0.225
- 401 error: FAL_KEY is invalid or not set
- 422 error: video URL is not accessible or format not supported

## Pricing
- Refine foreground edges ON: $0.0225 per 30 frames
- Refine foreground edges OFF: $0.012 per 30 frames

Fal model pages:
- Standard: https://fal.ai/models/veed/video-background-removal
- Fast: https://fal.ai/models/veed/video-background-removal/fast
- Green screen: https://fal.ai/models/veed/video-background-removal/green-screen
