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
Auto-transcribes a video's audio (or uses an SRT you provide), styles it with
your chosen preset, and renders a finished MP4 with the captions baked in.

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

Collect the following before proceeding. Items 1, 2, and 5 are mandatory
questions to ask the user — items 3 and 4 only need to be raised if the
user hasn't already mentioned them.

1. Video — a local file path (e.g. /Users/ana/Desktop/video.mp4) or
   a public URL. Accepted formats: MP4, MOV, WEBM, M4V, GIF
   Tip on Mac: right-click file in Finder → hold Option → Copy as Pathname

2. Preset — which subtitle style they want. 27 presets in two tiers:
   dynamic (2x cost multiplier, richer, best for social) and basic (1x,
   lightweight). The dynamic presets are: glass, whisper, glide2, fusion,
   glide, terminal, handwritten. For the full gallery and guidance see
   [references/presets.md](references/presets.md). If the user is unsure,
   suggest "glass" (dynamic) for social or "simple" (basic) for utility.

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
   the preset defaults? You can override the position, shadow intensity,
   and per-word-tier text styling (font, weight, colour). Say 'defaults'
   or 'skip' to use the preset as-is." The full set of options and their
   meanings is in [references/customization.md](references/customization.md).
   - If the user provides overrides → build a customization JSON object
     (see Step 2 and references/customization.md) and pass it in the run
   - If the user says 'defaults', 'skip', 'no preference', or similar →
     do NOT pass a customization flag in the run command

## Step 1 — Handle video and SRT inputs

If the user gave a local video path, upload it to Fal's CDN first:

    genmedia upload /path/to/video.mp4 --json

Copy the returned "url". If already a public URL, use it directly — no
upload needed.

If the user gave a local SRT file path, upload that too:

    genmedia upload /path/to/captions.srt --json

## Step 2 — Build the customization object (if provided)

Only if the user provided overrides. Build the JSON object per
[references/customization.md](references/customization.md) — it holds the
full field list, the JSON shape, and the constraints (position, shadow,
supported fonts, weight range, colour format). Any omitted field keeps the
preset's default. You pass it on the `--customization` flag in Step 4.

## Step 3 — Show cost estimate before proceeding

Fetch the current base rate rather than relying on memorised numbers:

    genmedia pricing veed/subtitles --json

Subtitles are billed per minute of input video, with resolution and preset
multipliers. Estimate and show the cost:
- Get video duration in minutes (use ffprobe or ask the user)
- Base rate = per-minute rate from `genmedia pricing`
- Resolution multiplier: 2x if video is above 1080p, else 1x
- Preset multiplier: 2x if preset is dynamic (glass, whisper, glide,
  glide2, fusion, terminal, handwritten), else 1x
- Estimated cost = duration_minutes x base_rate x resolution_mult x preset_mult
- Minimum charge: 1 minute

Show the estimate to the user and ask them to confirm before proceeding.
Example (indicative base rate $0.10/min): "This will cost approximately $0.40
for a 2-minute 1080p video with the 'glass' preset (dynamic, 2x multiplier).
Shall I proceed?"

## Step 4 — Render the subtitled video

First confirm the endpoint's current input fields (Fal schemas can change):

    genmedia schema veed/subtitles --json

Then run the subtitles endpoint asynchronously, using the exact field names
from the schema. `--async` submits the job and returns immediately with a
`request_id`. Include the optional flags only when the user actually
provided them:

    genmedia run veed/subtitles \
      --video_url "<video_url>" \
      --preset glass \
      --async \
      --json

Add any of these only if provided:
- `--language "es-MX"` — source-audio language code
- `--srt_file_url "<url>"` OR `--srt_content "<raw SRT>"` — supply at most one
- `--customization '<JSON from Step 2>'` — the customization object

The flags above reflect the expected schema — if `genmedia schema` shows a
different name, follow it.

IMPORTANT — record the `request_id` and show it to the user. The render is
billed once submitted, so if the session is interrupted you can re-fetch the
result with `status` instead of paying to run it again.

## Step 5 — Poll for the result

Check the job with the recorded `request_id` until it reports completed:

    genmedia status veed/subtitles <request_id> --json

Once completed, fetch the result and download the video locally in one step
(Fal URLs expire after ~24 hours):

    genmedia status veed/subtitles <request_id> \
      --download "./outputs/subtitles/{request_id}.{ext}" \
      --json

`--download` saves the file to the given path and still returns `video.url`.
Give the user both the local file path and the URL. If the session was
interrupted, resume here with the same `request_id`; do NOT re-run Step 4.

## After the call
- Return both the local file path and the video.url to the user
- The downloaded file is the durable copy — the Fal URL expires after ~24 hours
- 401 / auth error: Fal key not configured — run `genmedia setup`
- 422 error: video not accessible, format not supported, or invalid
  SRT content
- 400 error: unrecognized font name in customization — check the
  Google Fonts list
- 429 error: rate limit hit — wait a moment and retry
- 500 error: model error on Fal's side — retry once before giving up

## Pricing
Run `genmedia pricing veed/subtitles --json` for the authoritative current
base rate. Indicative rates at time of writing:
- Base rate: $0.10 per minute of input video
- Resolution multiplier: 2x for video above 1080p
- Preset multiplier: 2x for dynamic presets (glass, whisper, glide,
  glide2, fusion, terminal, handwritten)
- Multipliers compound (e.g. a 4K dynamic render = $0.40 per minute)
- Minimum charge: 1 minute
- Max duration: 2 hours at ≤1080p, 1 hour above 1080p
- Fal model page: https://fal.ai/models/veed/subtitles
