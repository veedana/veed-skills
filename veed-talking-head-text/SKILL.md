---
version: 1.4.0
name: veed-talking-head-text
description: >
  Generate a talking head video from a static image and a text script
  using VEED's Fabric 1.0 Text API. VEED's AI voice generator converts
  the script to speech automatically. Use when: "generate a talking head
  video from a script", "make this image say this", "create a spokesperson
  video with this text", "I want to type what the person says".
  Accepts local file paths or public URLs for the image.
  NOT for: using your own audio file (use veed-talking-head).
---

# VEED Talking Head — Text to Video (Fabric 1.0)

## What this skill does
Takes a static image and a text script. VEED's AI voice generator converts
the text to speech and syncs it to the image. No audio file needed.
Maximum generation: 30 seconds of output video.
Powered by VEED's Fabric 1.0 Text model on Fal.

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

IMPORTANT: Collect ALL four inputs before proceeding. Do not skip any.

1. Image — a local file path or public URL of the image to animate.
   Must show a face clearly. Accepted formats: JPG, PNG, WebP, GIF, AVIF
   Tip on Mac: right-click file in Finder → hold Option → Copy as Pathname

2. Script — what do you want the person to say?
   Keep it short — maximum 30 seconds of speech per generation.

3. Voice description — ALWAYS ask this question explicitly. Do not skip it.
   Say to the user: "How would you like the voice to sound? You can use
   simple descriptors like 'British accent' or 'Confident and warm', or
   something more detailed like 'Professional female voice, mid-30s, calm
   and authoritative tone'. If you have no preference, just say 'auto'
   and VEED will generate a voice based on the image."
   - If the user provides a description → store it as voice_description
   - If the user says 'auto', 'skip', 'no preference', or similar → do NOT
     pass a voice_description flag in the run command

4. Resolution — ask which the user prefers:
   - 480p — $0.08 per second of output video
   - 720p — $0.15 per second of output video
   Default to 480p if not specified.

## Step 1 — Handle image input

If the user gave a local file path, upload it to Fal's CDN first:

    genmedia upload /path/to/image.jpg --json

Copy the returned "url". If already a public URL, use it directly — no
upload needed.

## Step 2 — Show cost estimate before proceeding

Fetch the current rate rather than relying on memorised numbers:

    genmedia pricing veed/fabric-1.0/text --json

Estimate the output duration from the script length (roughly 130 words per
minute as a baseline), then apply the per-second rate:

    estimated_seconds = word_count / 130 x 60
    estimated_cost = estimated_seconds x price_per_second

Show the estimate and ask the user to confirm before proceeding.
Example (indicative rates $0.08/sec at 480p, $0.15/sec at 720p): "This script
(~20 words) will take roughly 9 seconds and cost approximately $0.72 at 720p.
Shall I proceed?"

## Step 3 — Generate the video

First confirm the endpoint's current input fields (Fal schemas can change):

    genmedia schema veed/fabric-1.0/text --json

Then run Fabric 1.0 Text asynchronously, using the exact field names from
the schema. `--async` submits the job and returns immediately with a
`request_id`. Include `--voice_description` ONLY if the user gave one:

    genmedia run veed/fabric-1.0/text \
      --image_url "<image_url>" \
      --text "<script text>" \
      --resolution 480p \
      --async \
      --json

If the user provided a voice description, add:

      --voice_description "<voice description>"

Set `--resolution` to `720p` if the user chose it. The flags above reflect
the expected schema — if `genmedia schema` shows a different name, follow it.

IMPORTANT — record the `request_id` and show it to the user. The run is
billed once submitted, so if the session is interrupted you can re-fetch the
result with `status` instead of paying to run it again.

## Step 4 — Poll for the result

Check the job with the recorded `request_id` until it reports completed:

    genmedia status veed/fabric-1.0/text <request_id> --json

Once completed, fetch the result and download the video locally in one step
(Fal URLs expire after ~24 hours):

    genmedia status veed/fabric-1.0/text <request_id> \
      --download "./outputs/talking-head-text/{request_id}.{ext}" \
      --json

`--download` saves the file to the given path and still returns `video.url`.
Give the user both the local file path and the URL. If the session was
interrupted, resume here with the same `request_id`; do NOT re-run Step 3.

## After the call
- Return both the local file path and the video.url to the user
- The downloaded file is the durable copy — the Fal URL expires after ~24 hours
- 401 / auth error: Fal key not configured — run `genmedia setup`
- 422 error: image not accessible, wrong format, or script too long
- 429 error: rate limit — wait a moment and retry
- 500 error: model error on Fal's side — retry once before giving up

## Pricing
Run `genmedia pricing veed/fabric-1.0/text --json` for the authoritative
current rate. Indicative rates at time of writing:
- 480p: $0.08 per second of output video
- 720p: $0.15 per second of output video
- Maximum: 30 seconds per generation
- Fal model page: https://fal.ai/models/veed/fabric-1.0/text
