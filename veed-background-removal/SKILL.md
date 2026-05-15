---
version: 1.3.0
name: veed-background-removal
description: >
  Remove the background from any video using VEED's Background Removal API.
  Use when: "remove background from this video", "remove the background",
  "make the background transparent", "extract the subject", "green screen
  removal". Accepts local file paths or public URLs.
  NOT for: removing backgrounds from images (video only).
---

# VEED Background Removal

## What this skill does
Takes a video and removes the background, returning a clean video with
just the subject. Three modes available depending on use case.
Powered by VEED's Background Removal API on Fal.

## Before you start
The user needs:
- A Fal API key — get one free at https://fal.ai/dashboard/keys
- Set it as an environment variable: export FAL_KEY=your_key_here

## What to ask the user for

Collect ALL of the following before proceeding:

1. Video — a local file path (e.g. /Users/ana/Desktop/video.mp4) or
   a public URL. Accepted formats: MP4, MOV, WEBM, MKV, AVI
   Tip on Mac: right-click file in Finder → hold Option → Copy as Pathname

2. Mode — which fits their need:
   - Standard — best quality. Default if not specified.
   - Fast — quicker, good for high volume or testing.
   - Green screen — specialised for green screen footage.

3. Refine foreground edges:
   - ON — cleaner edges, higher cost ($0.0225 per 30 frames). Default.
   - OFF — faster, lower cost ($0.012 per 30 frames).

## Step 1 — Map user choices to variables

Based on the user's answers, set these variables:

# Mode → model slug
if mode == "Standard":
    model = "veed/video-background-removal"
elif mode == "Fast":
    model = "veed/video-background-removal/fast"
elif mode == "Green screen":
    model = "veed/video-background-removal/green-screen"

# Refine edges → boolean
refine = True if user chose ON else False

## Step 2 — Handle video input

If the user provided a local file path, upload it to Fal first:

import fal_client
video_url = fal_client.upload_file("/path/to/video.mp4")

If already a public URL, use it directly — no upload needed.

## Step 3 — Show cost estimte before proceeding

Before making the API call, estimate and show the cost:

- Get video duration in seconds (use ffprobe or ask the user)
- Assume 30fps unless the user specifies otherwise
- Estimated frames = duration_seconds x fps
- Estimated cost = (frames / 30) x $0.0225 (Refine ON) or x $0.012 (Refine OFF)

Show the estimate to the user and ask them to confirm before proceeding.
Example: "This will cost approximately $0.45 for a 10-second video at 30fps
with Refine ON. Shall I proceed?"

## Step 4 — Remove the background

Use fal_client.subscribe() instead of run() to handle long jobs and
show progress:

import fal_client

def on_queue_update(update):
    if hasattr(update, "logs"):
        for log in update.logs:
            print(log["message"])

result = fal_client.subscribe(
    model,
    arguments={
        "video_url": video_url,
        "refine_foreground_edges": refine
    },
    with_logs=True,
    on_queue_update=on_queue_update
)

print(result["video"]["url"])

## After the call
- Return the video.url to the user
- Warn the user that Fal URLs expire after ~24 hours — download the
  file locally if they want to keep it
- 401 error: FAL_KEY is invalid or not set
- 422 error: video not accessible or format not supported
- 429 error: rate limit hit — wait a moment and retry
- 500 error: model error on Fal's side — retry once before giving up

## Pricing
- Refine ON: $0.0225 per 30 frames
- Refine OFF: $0.012 per 30 frames

Model pages:
- Standard: https://fal.ai/models/veed/video-background-removal
- Fast: https://fal.ai/models/veed/video-background-removal/fast
- Green screen: https://fal.ai/models/veed/video-background-removal/green-screen
