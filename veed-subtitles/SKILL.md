---
version: 1.0.0
name: veed-subtitles
description: >
  Burn styled, auto-transcribed subtitles into any video with VEED's Subtitles
  API — 30 styled presets, 125+ source languages, optional translation and SRT
  input. Use when the
  user wants captions rendered onto a video: "add subtitles to this video",
  "burn in captions", "caption this in Spanish". NOT for: producing a
  standalone .srt without rendering (this always returns a video with the
  captions baked in).
---

# VEED Subtitles

## What this skill does
Auto-transcribes a video's audio (or uses an SRT you provide), optionally
translates it, styles it with your chosen preset, and renders a finished MP4
with the captions baked in.

## Before you start
Setup (genmedia CLI + Fal key), the async execution pattern, uploading local
inputs, and common errors are shared across all VEED skills — see
[../COMMON.md](../COMMON.md). This file covers only what's specific to
subtitles.

## What to ask the user for

Collect the following before proceeding. Items 1, 2, and 7 are mandatory
questions to ask the user — items 3–6 only need to be raised if the user
hasn't mentioned them or the use-case calls for it.

1. Video — a local file path (e.g. /Users/ana/Desktop/video.mp4) or
   a public URL. Accepted formats: MP4, MOV, WEBM, M4V, GIF
   Tip on Mac: right-click file in Finder → hold Option → Copy as Pathname

2. Preset — which subtitle style they want. Presets come in two tiers:
   dynamic (2x cost multiplier, richer, best for social) and basic (1x,
   lightweight). The dynamic presets are: glass, whisper, glide2, fusion,
   glide, terminal, handwritten, backdrop, backdrop2. For the full gallery and
   the authoritative current list see [references/presets.md](references/presets.md).
   If the
   user is unsure, suggest "glass" (dynamic) for social or "simple" (basic)
   for utility.

3. Language (optional) — the source-audio language code, to improve
   transcription accuracy (e.g. en-US, es-MX, ja-JP, fr-FR). Leave blank to
   auto-detect. The API accepts a fixed set of ~160 BCP-47 codes; if the user
   names a language, map it to the closest (e.g. "Spanish (Mexico)" → es-MX,
   "Brazilian Portuguese" → pt-BR).

4. Translate output (optional) — by default subtitles are rendered in the
   spoken language. To render them in a DIFFERENT language, set
   translation_language to the target BCP-47 code. (If you also supply an SRT,
   set language to its source language for a reliable translation.)

5. Existing subtitles (optional) — if the user already has an SRT, ask:
   - srt_file_url — a public URL to a .srt file (or a local path,
     which you'll upload to Fal), OR
   - srt_content — raw SRT text pasted in
   Only one of these should be supplied. If neither is provided, the
   model auto-transcribes the audio.

6. Custom vocabulary (optional) — brand names, product names, or jargon the
   transcriber commonly mis-spells (e.g. "VEED" heard as "vid"). Collect as
   {word, replaces} entries — the correct term plus the misspellings to swap
   out. Up to 100 entries; ignored when an SRT is supplied.

7. Customization — ALWAYS ask this question explicitly. Do not skip it
   even though customization is optional in the API.
   Say to the user: "Do you want to customize the subtitle look, or use
   the preset defaults? You can override the position, shadow intensity,
   and per-word-tier text styling (font, weight, colour). Say 'defaults'
   or 'skip' to use the preset as-is." The full set of options and their
   meanings is in [references/customization.md](references/customization.md).
   - If the user provides overrides → build a customization JSON object
     (see Step 1 and references/customization.md) and pass it in the run
   - If the user says 'defaults', 'skip', 'no preference', or similar →
     do NOT pass a customization flag in the run command

### For best results
- **Pass the source language when you know it** — improves transcription accuracy over auto-detect (`--language`).
- **Add brand/product terms to `vocabulary`** — stops the transcriber mishearing names ("VEED" → "vid"); up to 100 entries.
- **Legibility is automatic** — the renderer places captions in safe zones for 9:16 / 1:1 / 16:9, so no manual positioning is needed for readability.

## Step 1 — Build the customization object (if provided)

Only if the user provided overrides. Build the JSON object per
[references/customization.md](references/customization.md) — it holds the full
field list, the JSON shape, and the constraints (position, shadow, supported
fonts, weight range, colour format). Any omitted field keeps the preset's
default. You pass it on the `--customization` flag in Step 3.

## Step 2 — Estimate cost and confirm

Fetch the base rate (`genmedia pricing veed/subtitles --json`). Billed per
minute of input video, with multipliers:
- Base rate = per-minute rate from `genmedia pricing`
- Resolution multiplier: 2x if video is above 1080p, else 1x
- Preset multiplier: 2x if the preset is dynamic, else 1x
- Cost = duration_minutes x base_rate x resolution_mult x preset_mult
- Minimum charge: 1 minute

Show the estimate and get confirmation. Example (indicative $0.10/min base):
"~$0.40 for a 2-minute 1080p video with 'glass' (dynamic, 2x). Proceed?"

## Step 3 — Render the subtitled video

Upload the local video, and any local SRT file, per ../COMMON.md. Then follow
the async execution pattern in ../COMMON.md (schema → run `--async` → poll →
download). The endpoint is `veed/subtitles`; include the optional flags only
when the user provided them:

    genmedia run veed/subtitles \
      --video_url "<video_url>" \
      --preset glass \
      --async \
      --json

Optional flags:
- `--language "es-MX"` — source-audio language code
- `--translation_language "<code>"` — render subtitles in a different language
- `--srt_file_url "<url>"` OR `--srt_content "<raw SRT>"` — supply at most one
- `--vocabulary '[{"word":"VEED","replaces":["vid","veet"]}]'` — brand/jargon fixes
- `--customization '<JSON from Step 1>'` — the customization object

Download the result to `./outputs/subtitles/`, and return both the local path
and the URL.

## Errors
Common errors (401 / 422 / 429 / 500) are in ../COMMON.md. Subtitles-specific:
- **400** — unrecognized font name in customization; check the Google Fonts list
- **422** — here also covers invalid SRT content

## Pricing
Run `genmedia pricing veed/subtitles --json` for the authoritative current
base rate. Indicative rates at time of writing:
- Base rate: $0.10 per minute of input video
- Resolution multiplier: 2x for video above 1080p
- Preset multiplier: 2x for dynamic presets (glass, whisper, glide, glide2,
  fusion, terminal, handwritten, backdrop, backdrop2)
- Multipliers compound (e.g. a 4K dynamic render = $0.40 per minute)
- Minimum charge: 1 minute
- Max duration: 2 hours at ≤1080p, 1 hour above 1080p
- Fal model page: https://fal.ai/models/veed/subtitles
