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
These skills run through the genmedia CLI, which handles model discovery,
file upload, execution, and downloads against Fal.

- Install it once — see https://github.com/fal-ai-community/genmedia-cli
- Configure your Fal API key (get one free at https://fal.ai/dashboard/keys):
      genmedia setup
  Or non-interactively (agents / CI):
      genmedia setup --non-interactive --api-key "$FAL_KEY"

All commands below assume `genmedia` is on your PATH.

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

Based on the user's answers, pick the endpoint and refine flag:

- Standard     → endpoint `veed/video-background-removal`
- Fast         → endpoint `veed/video-background-removal/fast`
- Green screen → endpoint `veed/video-background-removal/green-screen`

Refine edges → `refine_foreground_edges` is `true` (ON) or `false` (OFF).

## Step 2 — Handle video input

If the user gave a local file path, upload it to Fal's CDN first:

    genmedia upload /path/to/video.mp4 --json

Copy the returned "url". If the input is already a public URL, use it
directly — no upload needed.

## Step 3 — Show cost estimate before proceeding

Before making the API call, estimate and show the cost:

- Get video duration in seconds (use ffprobe or ask the user)
- Assume 30fps unless the user specifies otherwise
- Estimated frames = duration_seconds x fps
- Estimated cost = (frames / 30) x $0.0225 (Refine ON) or x $0.012 (Refine OFF)

Show the estimate to the user and ask them to confirm before proceeding.
Example: "This will cost approximately $0.45 for a 10-second video at 30fps
with Refine ON. Shall I proceed?"

## Step 4 — Remove the background

First confirm the endpoint's current input fields (Fal schemas can change):

    genmedia schema veed/video-background-removal --json

Then run the chosen endpoint asynchronously, using the exact field names
from the schema. `--async` submits the job and returns immediately with a
`request_id`:

    genmedia run veed/video-background-removal \
      --video_url "<video_url>" \
      --refine_foreground_edges true \
      --async \
      --json

Swap the endpoint for the fast or green-screen variant per Step 1, and set
`--refine_foreground_edges` to `false` if the user chose OFF. The flags above
reflect the expected schema — if `genmedia schema` shows a different name,
follow it.

IMPORTANT — record the `request_id` and show it to the user. The run is
billed once submitted, so if the session is interrupted you can re-fetch the
result with `status` instead of paying to run it again.

## Step 5 — Poll for the result

Check the job with the recorded `request_id` until it reports completed
(use the same endpoint variant you ran):

    genmedia status veed/video-background-removal <request_id> --json

The completed result JSON contains `video.url` — return it to the user. If
the session was interrupted, resume here with the same `request_id`; do NOT
re-run Step 4.

The result JSON contains `video.url` — return it to the user.

## After the call
- Return the video.url to the user
- Warn the user that Fal URLs expire after ~24 hours — download the
  file locally if they want to keep it
- 401 / auth error: Fal key not configured — run `genmedia setup`
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
