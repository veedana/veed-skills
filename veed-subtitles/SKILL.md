---
version: 1.0.0
name: veed-subtitles
description: >
  Add styled, burned-in subtitles to any video using VEED's Subtitles API.
  Handles the full pipeline — transcribe, style, and render — in one call,
  returning a finished MP4 with captions baked in. Use when: "add subtitles
  to this video", "burn in captions", "caption this video", "add animated
  subtitles", "subtitle this in Spanish/French/etc", "style my subtitles".
  Accepts local file paths or public URLs, with 27 presets and optional
  SRT input.
  NOT for: generating standalone .srt files without rendering (this skill
  always returns a video with burned-in subtitles).
---

# VEED Subtitles

## What this skill does
Takes a video and burns styled subtitles into it, returning a finished MP4
with captions baked in. VEED auto-transcribes the audio (or uses an SRT
you provide), styles it with the preset of your choice, and renders the
output. Supports 125+ source languages and 27 visual presets.
Powered by VEED's Subtitles API on Fal.

## Before you start
The user needs:
- A Fal API key — get one free at https://fal.ai/dashboard/keys
- Set it as an environment variable: export FAL_KEY=your_key_here

## What to ask the user for

Collect the following before proceeding. Items 1, 2, and 5 are mandatory
questions to ask the user — items 3 and 4 only need to be raised if the
user hasn't already mentioned them.

1. Video — a local file path (e.g. /Users/ana/Desktop/video.mp4) or
   a public URL. Accepted formats: MP4, MOV, WEBM, M4V, GIF
   Tip on Mac: right-click file in Finder → hold Option → Copy as Pathname

2. Preset — which subtitle style they want. Two tiers:
   - Dynamic presets (2x multiplier) — richer, context-aware rendering
     that adapts to the input. Best for social / hook content:
     glass, whisper, glide2, fusion, glide, terminal, handwritten
   - Basic presets (1x multiplier) — fixed, lightweight styling with
     predictable output. Best for utility captioning or high-volume
     pipelines:
     simple, plain, beans, corpo, boo, shadeplay, casper, capri, lowkey,
     vinta, diego, ali, slay, kitty, hustle, karl, sprout, flex, mint,
     rizz, vegas
   If the user is unsure, suggest "glass" (dynamic) for social content
   or "simple" (basic) for plain utility captioning.

3. Language (optional) — source-audio language code (e.g. en-US, es-MX,
   ja-JP, fr-FR). Improves transcription accuracy. Leave blank to
   auto-detect.
   IMPORTANT: this is the source-audio language, not the output language.
   Subtitles render in the same language as the audio. The model supports
   125+ language codes — if the user names a language, map it to the
   closest BCP-47 code (e.g. "Spanish (Mexico)" → es-MX, "Brazilian
   Portuguese" → pt-BR).

4. Existing subtitles (optional) — if the user already has an SRT, ask:
   - srt_file_url — a public URL to a .srt file (or a local path,
     which you'll upload to Fal), OR
   - srt_content — raw SRT text pasted in
   Only one of these should be supplied. If neither is provided, the
   model auto-transcribes the audio.

5. Customization — ALWAYS ask this question explicitly. Do not skip it
   even though customization is optional in the API.
   Say to the user: "Do you want to customize the subtitle look, or use
   the preset defaults? You can override any of these:
   - Position — top, center, or bottom (default: preset's default)
   - Shadow intensity — none, min, mid, or max (improves readability
     over busy backgrounds)
   - Per-tier text styling — font, weight (100-900), and hex colour for
     each of the three word-importance tiers:
       - accessible: baseline styling applied to every word
       - highlighted: mid-rank words (key nouns, action verbs, salient
         adjectives) — presets typically bump size or weight here
       - viral: top-rank 'hook' words, only a handful per video (note:
         some presets don't use this tier, so overrides are a no-op
         on those)
   Any field you leave out keeps the preset's default. Say 'defaults'
   or 'skip' if you want to use the preset as-is."
   - If the user provides overrides → build a customization dict (see
     Step 2 below) and pass it in the API call
   - If the user says 'defaults', 'skip', 'no preference', or similar →
     set customization to None and do NOT pass it in the API call

## Step 1 — Handle video and SRT inputs

If the user provided a local video path, upload it to Fal first:

import fal_client
video_url = fal_client.upload_file("/path/to/video.mp4")

If already a public URL, use it directly — no upload needed.

If the user provided a local SRT file path, upload that too:

srt_file_url = fal_client.upload_file("/path/to/captions.srt")

## Step 2 — Build the customization dict (if provided)

Only build this if the user provided overrides. Any omitted field keeps
the preset's default.

customization = {
    "position": "bottom",
    "shadow": "mid",
    "text_customizations": {
        "accessible":  {"font": "Inter", "weight": 500, "color": "#FFFFFF"},
        "highlighted": {"font": "Inter", "weight": 700, "color": "#FFD500"},
        "viral":       {"font": "Inter", "weight": 900, "color": "#FF2E63"}
    }
}

Constraints:
- position: top, center, or bottom
- shadow: none, min, mid, or max
- font: must be a supported Google Font — see
  https://www.veed.io/api/v1/subtitle-renders/fonts for the canonical
  list. Unrecognized fonts return a 400.
- weight: 100-900. Values >= 700 render as bold.
- color: hex string (e.g. "#FFFFFF")
- The "viral" tier is a no-op on presets that don't use it.

## Step 3 — Show cost estimate before proceeding

Before making the API call, estimate and show the cost:
- Get video duration in minutes (use ffprobe or ask the user)
- Base rate = $0.10 per minute
- Resolution multiplier: 2x if video is above 1080p, else 1x
- Preset multiplier: 2x if preset is dynamic (glass, whisper, glide,
  glide2, fusion, terminal, handwritten), else 1x
- Estimated cost = duration_minutes x $0.10 x resolution_mult x preset_mult
- Minimum charge: 1 minute

Show the estimate to the user and ask them to confirm before proceeding.
Example: "This will cost approximately $0.40 for a 2-minute 1080p video
with the 'glass' preset (dynamic, 2x multiplier). Shall I proceed?"

## Step 4 — Build the arguments dict conditionally

Only include optional fields if the user actually provided them:

import fal_client

arguments = {
    "video_url": video_url,
    "preset": preset
}

if language is not None:
    arguments["language"] = language

if srt_file_url is not None:
    arguments["srt_file_url"] = srt_file_url
elif srt_content is not None:
    arguments["srt_content"] = srt_content

if customization is not None:
    arguments["customization"] = customization

## Step 5 — Render the subtitled video

Use fal_client.subscribe() instead of run() to handle long jobs and
show progress:

def on_queue_update(update):
    if hasattr(update, "logs"):
        for log in update.logs:
            print(log["message"])

result = fal_client.subscribe(
    "veed/subtitles",
    arguments=arguments,
    with_logs=True,
    on_queue_update=on_queue_update
)

print(result["video"]["url"])

## After the call
- Return the video.url to the user
- Warn the user that Fal URLs expire after ~24 hours — download the
  file locally if they want to keep it
- 401 error: FAL_KEY is invalid or not set
- 422 error: video not accessible, format not supported, or invalid
  SRT content
- 400 error: unrecognized font name in customization — check the
  Google Fonts list
- 429 error: rate limit hit — wait a moment and retry
- 500 error: model error on Fal's side — retry once before giving up

## Pricing
- Base rate: $0.10 per minute of input video
- Resolution multiplier: 2x for video above 1080p
- Preset multiplier: 2x for dynamic presets (glass, whisper, glide,
  glide2, fusion, terminal, handwritten)
- Multipliers compound (e.g. a 4K dynamic render = $0.40 per minute)
- Minimum charge: 1 minute
- Max duration: 2 hours at ≤1080p, 1 hour above 1080p
- Fal model page: https://fal.ai/models/veed/subtitles
