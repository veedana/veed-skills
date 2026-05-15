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
The user needs:
- A Fal API key — get one free at https://fal.ai/dashboard/keys
- Set it as an environment variable: export FAL_KEY=your_key_here

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
   - If the user says 'auto', 'skip', 'no preference', or similar → set
     voice_description to None and do NOT pass it in the API call

4. Resolution — ask which the user prefers:
   - 480p — $0.08 per second of output video
   - 720p — $0.15 per second of output video
   Default to 480p if not specified.

## Step 1 — Handle image input

If the user provided a local file path, upload it to Fal:

import fal_client
image_url = fal_client.upload_file(image_path)

If already a public URL, use it directly — no upload needed.

## Step 2 — Show cost estimate before proceeding

Estimate the output duration from the script length
(roughly 130 words per minute as a baseline).

estimated_seconds = word_count / 130 x 60
estimated_cost = estimated_seconds x price_per_second

Where price_per_second is $0.08 (480p) or $0.15 (720p).

Show the estimate and ask the user to confirm before proceeding.
Example: "This script (~20 words) will take roughly 9 seconds and cost
approximately $0.72 at 720p. Shall I proceed?"

## Step 3 — Generate the video

Build the arguments dict conditionally — only include voice_description
if the user provided one (not None):

import fal_client

arguments = {
    "image_url": image_url,
    "text": script_text,
    "resolution": resolution
}

if voice_description is not None:
    arguments["voice_description"] = voice_description

def on_queue_update(update):
    if hasattr(update, "logs"):
        for log in update.logs:
            print(log["message"])

result = fal_client.subscribe(
    "veed/fabric-1.0/text",
    arguments=arguments,
    with_logs=True,
    on_queue_update=on_queue_update
)

print(result["video"]["url"])

## After the call
- Return the video.url to the user
- Warn the user that Fal URLs expire after ~24 hours — download locally
  if they want to keep it
- 401 error: FAL_KEY is invalid or not set
- 422 error: image not accessible, wrong format, or script too long
- 429 error: rate limit — wait a moment and retry
- 500 error: model error on Fal's side — retry once before giving up

## Pricing
- 480p: $0.08 per second of output video
- 720p: $0.15 per second of output video
- Maximum: 30 seconds per generation
- Fal model page: https://fal.ai/models/veed/fabric-1.0/text
