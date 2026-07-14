---
version: 1.0.0
name: veed-product-pitch
description: >
  WORKFLOW SKILL — chains image generation + composition, VEED Fabric talking
  head, and VEED Subtitles into a finished product spokesperson video in one
  flow: a person holding your product and talking about it, optionally with
  burned-in subtitles. Use when the user wants a product ad, UGC / creator-
  style video, or testimonial with a spokesperson: "make a product pitch
  video", "UGC ad for my brand", "get a model to hold my product and talk
  about it". NOT for: a single product image (use Nano Banana directly), or a
  talking head without a product (use veed-talking-head / veed-talking-head-text).
---

# VEED Product Pitch — Workflow Skill

## What this skill does
Generates a finished product spokesperson video in one flow:
1. Gets or generates a product image (Nano Banana)
2. Gets or generates a spokesperson image (Nano Banana)
3. Composites the spokesperson holding the product (Nano Banana edit)
4. Animates the composite with audio or text-to-speech (VEED Fabric 1.0)
5. Optionally burns in subtitles (VEED Subtitles)
Maximum output: 30 seconds of talking head video (Fabric model limit).

## Before you start
Setup (genmedia CLI + Fal key), the async execution pattern (schema → run
`--async` → poll → download → resume), uploading local inputs, and common
errors are shared across all VEED skills — see [../COMMON.md](../COMMON.md).
Every `genmedia run` below follows that pattern.

This workflow reuses the veed-talking-head, veed-talking-head-text, and
veed-subtitles skills' endpoints; those skills are the source of truth for
their full option sets (voice descriptions, the subtitle preset gallery,
customization). Defer to them for detail.

Responsible use (see [../COMMON.md](../COMMON.md)) applies. This skill scripts a
spokesperson selling a product, so also flag to the user (don't just proceed) if
the pitch makes health, financial, or legal claims as fact, or promotes a
product that appears fraudulent — and never fabricate claims about the product
yourself.

## What to ask the user for

Collect ALL of the following before showing the cost estimate.
Do not skip any question, even if the user gave you a one-line brief.

1. Product — ask: "What product is this video for? You can either upload
   an image of it, or describe it and I'll generate one for you."
   - If the user uploads → store as product_image_path
   - If the user describes → store the description as product_prompt
   - Accepted formats: JPG, PNG, WebP

2. Spokesperson — ask: "Who should the spokesperson be? You can upload
   a photo of someone (must show their face clearly), or describe the
   kind of person and I'll generate one."
   - If the user uploads → store as spokesperson_image_path
   - If the user describes → store as spokesperson_prompt
   - Accepted formats: JPG, PNG, WebP

3. What they should say — ask: "What should the spokesperson say? You
   can either upload an audio file of the script being read, or write
   the script as text. Max 30 seconds of speech per generation."
   - If the user uploads audio → store as audio_path, set mode = "audio"
   - If the user provides text → store as script_text, set mode = "text"
   - Accepted audio formats: MP3, WAV, M4A, AAC

4. Voice description — ONLY ask this if mode == "text". Ask the user how
   they want the voice to sound, encouraging multiple attributes (gender,
   age, accent, tone, energy) over a bare one-liner, or 'auto' to let VEED
   pick from the spokesperson image. See
   [references/voice-description.md](references/voice-description.md) for the
   exact phrasing and the short-description follow-up.
   - If the user provides a specific description → store as voice_description
   - If the user says ANY of: 'auto', 'skip', 'no preference', 'you pick',
     'doesn't matter', 'default', or any similar opt-out → set
     voice_description to None.
   CRITICAL: Do NOT store the literal string "auto" (or similar opt-out
   phrases) as voice_description. Fabric would interpret it as a voice
   description verbatim and produce a strange-sounding voice.

5. Resolution and aspect ratio —
   - Resolution: 480p ($0.08/sec) or 720p ($0.15/sec). Default 480p.
   - Aspect ratio — the orientation of the final video: 9:16 vertical for
     TikTok / Reels / Shorts (suggested for social / UGC), 1:1 square, or
     16:9 landscape. Default 9:16. Store as aspect_ratio.
   Fabric has no aspect setting of its own — the final video inherits the
   composite image's shape — so aspect_ratio is applied when generating the
   spokesperson and compositing (Steps 3-4), not at the talking-head step.

6. Subtitles — ask: "Do you want subtitles burned into the final video?"
   - If yes → ask which preset. Say to the user: "Which subtitle style?
     The dynamic presets (glass, whisper, glide2, fusion, glide, terminal,
     handwritten, backdrop, backdrop2) are richer and best for social content;
     there are also lightweight basic presets. Default suggestion: 'glass'." (See the
     veed-subtitles skill for the full preset gallery.)
     Store as subtitle_preset. Note the dynamic presets carry a 2x cost
     multiplier — this matters for the estimate in Step 1.
     Then ALWAYS ask a follow-up about customization: whether they want to
     override the subtitle look (position, shadow, per-word-tier font /
     weight / colour) or use the preset defaults. The veed-subtitles skill
     is the source of truth for the full option set and constraints.
     - If the user provides overrides → build a subtitle_customization
       JSON object (see Step 6 below) and store it
     - If the user says 'defaults', 'skip', 'no preference', or similar →
       set subtitle_customization to None
   - If no → set subtitle_preset to None and subtitle_customization to None

## Step 1 — Show cost estimate before proceeding

Estimate the total cost upfront and ask the user to confirm. Fetch current
rates for each endpoint this run will use rather than relying on memorised
numbers:

    genmedia pricing fal-ai/gemini-25-flash-image --json   # image gen / composite
    genmedia pricing veed/fabric-1.0 --json                # or veed/fabric-1.0/text
    genmedia pricing veed/subtitles --json                 # only if subtitles

The indicative rates below are for illustration — use the live rates in the
actual estimate.

Cost components:
- Nano Banana product generation: $0.039 (only if user described the product)
- Nano Banana spokesperson generation: $0.039 (only if user described the spokesperson)
- Nano Banana composite (always): $0.039
- Fabric talking head:
    estimated_seconds = audio_duration OR (word_count / 130) x 60
    fabric_cost = estimated_seconds x price_per_second
    Where price_per_second = $0.08 (480p) or $0.15 (720p)
- Subtitles (only if subtitle_preset is not None):
    Minimum charge is 1 minute = $0.10. For dynamic presets (glass,
    whisper, glide2, fusion, glide, terminal, handwritten, backdrop,
    backdrop2), apply 2x multiplier = $0.20 minimum.

Show the breakdown to the user. Example for a 10-second video at 720p
with the glass subtitle preset:
"Here's the estimated cost:
 - Product image generation: $0.039
 - Spokesperson generation: $0.039
 - Composite (holding product): $0.039
 - Talking head (10s @ 720p): $1.50
 - Subtitles (glass preset): $0.20
 - Total: ~$1.82
 Shall I proceed?"

Wait for explicit confirmation before continuing.

## Step 2 — Get or generate the product image

If product_image_path is set (user uploaded), upload it and keep the URL:

    genmedia upload /path/to/product.jpg --json

Else (user described), write a strong prompt first — see
[references/image-prompts.md](references/image-prompts.md), and show it to the
user before generating — then run Nano Banana:

    genmedia run fal-ai/gemini-25-flash-image \
      --prompt "<product_prompt>" \
      --num_images 1 \
      --json

Take the first image URL from the result as product_image_url.

## Step 3 — Get or generate the spokesperson image

If spokesperson_image_path is set (user uploaded), upload it and keep the URL:

    genmedia upload /path/to/spokesperson.jpg --json

Else (user described), write the prompt per
[references/image-prompts.md](references/image-prompts.md) — the spokesperson
section folds in the Fabric input rules (one clear face, framing, expression
matching the script's tone) since this image feeds the talking-head step — and
show it to the user before generating. Generate in the chosen aspect_ratio so
the person is framed for the final orientation:

    genmedia run fal-ai/gemini-25-flash-image \
      --prompt "<spokesperson_prompt>, clear face, looking at camera, professional photo, plain background" \
      --aspect_ratio "<aspect_ratio>" \
      --num_images 1 \
      --json

Take the first image URL from the result as spokesperson_image_url.

## Step 4 — Composite the spokesperson holding the product

Use Nano Banana's edit endpoint with both images and a fixed prompt.
Pass both URLs on the `--image_urls` flag as a JSON array:

    genmedia run fal-ai/gemini-25-flash-image/edit \
      --image_urls '["<spokesperson_image_url>", "<product_image_url>"]' \
      --prompt "Make the model hold this product in their hand. Keep the spokesperson's face clearly visible and looking at the camera. Natural pose, well-lit." \
      --aspect_ratio "<aspect_ratio>" \
      --json

Take the first image URL from the result as composite_image_url. This image's
shape sets the final video's orientation, so make sure aspect_ratio matches
what the user asked for. Note: Fabric snaps its output to a fixed set of aspect
buckets (dimensions divisible by 64), so the final video's ratio may shift
slightly from the requested aspect_ratio (e.g. 16:9 → ~1.86) — don't promise a
pixel-exact ratio.

## Step 5 — Generate the talking head video

The Nano Banana steps above are quick and run synchronously, but the talking
head is the long, expensive job — run it asynchronously so a dropped
connection doesn't lose it. Keep the `composite_image_url` from Step 4: if a
resume is needed you re-run only this step, not the paid image steps.

Branch based on mode.

If mode == "audio", upload the audio and run Fabric 1.0 with `--async`:

    genmedia upload /path/to/audio.mp3 --json

    genmedia run veed/fabric-1.0 \
      --image_url "<composite_image_url>" \
      --audio_url "<audio_url>" \
      --resolution 480p \
      --async \
      --json

If mode == "text", run Fabric 1.0 Text with `--async` (add
`--voice_description` only if voice_description is not None):

    genmedia run veed/fabric-1.0/text \
      --image_url "<composite_image_url>" \
      --text "<script_text>" \
      --resolution 480p \
      --async \
      --json

Use `720p` if the user chose it. Poll for the result per ../COMMON.md with
the endpoint you ran, and take `video.url` as video_url.

If subtitle_preset is None, this talking head is the final output — download
it to `./outputs/product-pitch/` (per ../COMMON.md) and return both the path
and URL, then stop. If subtitle_preset is set, do NOT download the
intermediate — keep video_url and continue to Step 6.

## Step 6 — Add subtitles (only if subtitle_preset is set)

Run the subtitles endpoint (async, per ../COMMON.md) using video_url as input.
Add `--customization` only if the user provided overrides:

    genmedia run veed/subtitles \
      --video_url "<video_url>" \
      --preset "<subtitle_preset>" \
      --async \
      --json

If subtitle_customization is set, build the JSON object per the veed-subtitles
skill (the source of truth for its shape and constraints) and pass it as
`--customization '<JSON>'`. Keep `video_url` — on a resume you re-run only the
subtitles step, not the paid talking-head step.

Download the final video to `./outputs/product-pitch/` (per ../COMMON.md) as
final_video_url, and give the user both the local path and the URL.

Do not ask the user about language, translation, SRT, or vocabulary — this
workflow uses plain auto-transcription. If the user needs any of that control,
suggest running veed-subtitles separately afterwards.

## After the call
- Return both the local file path and the final video URL to the user.
- Common errors (401 / 422 / 429 / 500) are in ../COMMON.md. Workflow-specific:
  on a 500, identify which step failed (product gen / spokesperson gen /
  composite / talking head / subtitles) and retry that step only — don't
  restart the whole flow. A 422 often means an input exceeds Fabric's 30s cap.

## Pricing
This skill chains multiple models — run `genmedia pricing <endpoint> --json`
per endpoint for authoritative current rates. Indicative per-call costs,
typical end-to-end totals, and model pages are in
[references/pricing.md](references/pricing.md).
