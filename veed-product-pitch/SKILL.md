---
version: 1.0.0
name: veed-product-pitch
description: >
  WORKFLOW SKILL — chains Nano Banana image generation + composition with
  VEED's Fabric talking head and Subtitles APIs to create a finished
  product spokesperson video in one flow. The user provides a product
  (uploaded image or text description), a spokesperson (uploaded image or
  text description), and what the spokesperson should say (audio or text).
  Optionally adds burned-in subtitles. Use when: "create a product pitch
  video", "make a spokesperson video for my product", "generate a talking
  head video promoting [product]", "create a UGC-style ad for my brand",
  "make a creator-style product video", "get a model to hold my product
  and talk about it", "create a product ad with someone talking",
  "make a testimonial-style video for my product".
  NOT for: generating a single product image (use Nano Banana directly),
  a talking head without a product (use veed-talking-head or
  veed-talking-head-text).
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
This workflow runs through the genmedia CLI, which handles model discovery,
file upload, execution, and downloads against Fal.

- Install it once — see https://github.com/fal-ai-community/genmedia-cli
- Configure your Fal API key (get one free at https://fal.ai/dashboard/keys):
      genmedia setup
  Or non-interactively (agents / CI):
      genmedia setup --non-interactive --api-key "$FAL_KEY"

All commands below assume `genmedia` is on your PATH.

Before every `genmedia run` in this workflow, confirm the endpoint's current
input fields with `genmedia schema <endpoint> --json` and use the exact field
names it lists — Fal schemas can change, and the flags shown in each step
reflect the expected schema rather than a guarantee.

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

4. Voice description — ONLY ask this if mode == "text". Say to the user:
   "How would you like the voice to sound? For best results, describe
   multiple attributes — gender, age, accent, tone, energy. For example:
   'Professional female voice, mid-30s, British accent, warm and
   confident' works much better than just 'British accent'. Or say
   'auto' to let VEED pick a voice based on the spokesperson image."
   - If the user provides a specific description (anything other than an
     opt-out phrase below) → store as voice_description
   - If the user says ANY of: 'auto', 'skip', 'no preference', 'you
     pick', 'doesn't matter', 'default', or any other phrase indicating
     no preference → set voice_description to None.
   CRITICAL: Do NOT store the literal string "auto" (or similar opt-out
   phrases) as voice_description. Fabric would interpret it as a voice
   description verbatim and produce a strange-sounding voice.
   If the user gave a very short description (1-3 words like "British
   accent"), gently prompt them to add more detail. Say: "That's a
   good start — to get the best voice match, could you add a bit more?
   E.g., gender, age range, and overall tone (calm / energetic /
   authoritative)?" Then update voice_description with their fuller
   answer.

5. Resolution — ask: "Which resolution do you want?
   - 480p — $0.08 per second of output video
   - 720p — $0.15 per second of output video"
   Default to 480p if not specified.

6. Subtitles — ask: "Do you want subtitles burned into the final video?"
   - If yes → ask which preset. Say to the user: "Which subtitle style?
     The dynamic presets (glass, whisper, glide2, fusion, glide, terminal,
     handwritten) are richer and best for social content. Basic presets
     (simple, plain, beans, corpo, boo, shadeplay, casper, capri, lowkey,
     vinta, diego, ali, slay, kitty, hustle, karl, sprout, flex, mint,
     rizz, vegas) are lightweight. Default suggestion: 'glass'."
     Store as subtitle_preset.
     Then ALWAYS ask a follow-up about customization. Say to the user:
     "Do you want to customize the subtitle look, or use the preset
     defaults? You can override any of these:
     - Position — top, center, or bottom
     - Shadow intensity — none, min, mid, or max (improves readability
       over busy backgrounds)
     - Per-tier text styling — font, weight (100-900), and hex colour
       for each of the three word-importance tiers: accessible (every
       word), highlighted (mid-rank words), viral (top-rank hook words)
     Any field you leave out keeps the preset's default. Say 'defaults'
     or 'skip' if you want to use the preset as-is."
     - If the user provides overrides → build a subtitle_customization
       JSON object (see Step 6 below) and store it
     - If the user says 'defaults', 'skip', 'no preference', or similar →
       set subtitle_customization to None
   - If no → set subtitle_preset to None and subtitle_customization to None

## Step 1 — Show cost estimate before proceeding

Estimate the total cost upfront and ask the user to confirm.

Cost components:
- Nano Banana product generation: $0.039 (only if user described the product)
- Nano Banana spokesperson generation: $0.039 (only if user described the spokesperson)
- Nano Banana composite (always): $0.039
- Fabric talking head:
    estimated_seconds = audio_duration OR (word_count / 150) x 60
    fabric_cost = estimated_seconds x price_per_second
    Where price_per_second = $0.08 (480p) or $0.15 (720p)
- Subtitles (only if subtitle_preset is not None):
    Minimum charge is 1 minute = $0.10. For dynamic presets (glass,
    whisper, glide2, fusion, glide, terminal, handwritten), apply 2x
    multiplier = $0.20 minimum.

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

Else (user described), generate one with Nano Banana:

    genmedia run fal-ai/gemini-25-flash-image \
      --prompt "<product_prompt>" \
      --num_images 1 \
      --json

Take the first image URL from the result as product_image_url.

## Step 3 — Get or generate the spokesperson image

If spokesperson_image_path is set (user uploaded), upload it and keep the URL:

    genmedia upload /path/to/spokesperson.jpg --json

Else (user described), generate one with Nano Banana:

    genmedia run fal-ai/gemini-25-flash-image \
      --prompt "<spokesperson_prompt>, clear face, looking at camera, professional photo, plain background" \
      --num_images 1 \
      --json

Take the first image URL from the result as spokesperson_image_url.

## Step 4 — Composite the spokesperson holding the product

Use Nano Banana's edit endpoint with both images and a fixed prompt.
Pass both URLs on the `--image_urls` flag as a JSON array:

    genmedia run fal-ai/gemini-25-flash-image/edit \
      --image_urls '["<spokesperson_image_url>", "<product_image_url>"]' \
      --prompt "Make the model hold this product in their hand. Keep the spokesperson's face clearly visible and looking at the camera. Natural pose, well-lit." \
      --json

Take the first image URL from the result as composite_image_url.

## Step 5 — Generate the talking head video

Branch based on mode:

If mode == "audio", upload the audio and run Fabric 1.0:

    genmedia upload /path/to/audio.mp3 --json

    genmedia run veed/fabric-1.0 \
      --image_url "<composite_image_url>" \
      --audio_url "<audio_url>" \
      --resolution 480p \
      --json

If mode == "text", run Fabric 1.0 Text (add `--voice_description` only if
voice_description is not None):

    genmedia run veed/fabric-1.0/text \
      --image_url "<composite_image_url>" \
      --text "<script_text>" \
      --resolution 480p \
      --json

Set `--resolution` to `720p` if the user chose it. Take `video.url` from
the result as video_url.

If subtitle_preset is None: return video_url to the user and stop here.

## Step 6 — Add subtitles (only if subtitle_preset is set)

Run the subtitles endpoint using video_url as the input. Add
`--customization` only if the user provided overrides:

    genmedia run veed/subtitles \
      --video_url "<video_url>" \
      --preset "<subtitle_preset>" \
      --json

If subtitle_customization is set, build the JSON object matching the values
the user gave and pass it as `--customization '<JSON>'`. Example:

    {
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
  https://www.veed.io/api/v1/subtitle-renders/fonts
- weight: 100-900
- color: hex string (e.g. "#FFFFFF")

Take `video.url` from the result as final_video_url.

Do not ask the user about language or SRT input — for this workflow,
auto-transcription is used. If the user needs language or SRT control,
suggest running veed-subtitles separately afterwards.

Return final_video_url to the user.

## After the call
- Return the final video URL to the user
- Warn the user that Fal URLs expire after ~24 hours — download the
  file locally if they want to keep it
- 401 / auth error: Fal key not configured — run `genmedia setup`
- 422 error: an input image/audio is not accessible, in a bad format,
  or the audio/script exceeds Fabric's 30-second cap
- 429 error: rate limit — wait a moment and retry
- 500 error: one of the underlying models had an issue. Identify which
  step failed (product gen / spokesperson gen / composite / talking head
  / subtitles) and retry that step only — don't restart the whole flow.

## Pricing
This skill chains multiple models. Per-call costs:
- Nano Banana (generation): $0.039 per image
- Nano Banana (edit/composite): $0.039 per call
- Fabric 1.0 (audio): $0.08/sec at 480p, $0.15/sec at 720p
- Fabric 1.0 text: $0.08/sec at 480p, $0.15/sec at 720p
- Subtitles: $0.10/min base, 2x for dynamic presets, 1-minute minimum

Typical end-to-end cost for a 10-second video at 720p with subtitles
where both images are user-uploaded: ~$1.70
Typical end-to-end cost for a 10-second video at 720p with subtitles
where both images are AI-generated: ~$1.82

Model pages:
- Nano Banana: https://fal.ai/models/fal-ai/gemini-25-flash-image
- Nano Banana edit: https://fal.ai/models/fal-ai/gemini-25-flash-image/edit
- Fabric audio: https://fal.ai/models/veed/fabric-1.0
- Fabric text: https://fal.ai/models/veed/fabric-1.0/text
- Subtitles: https://fal.ai/models/veed/subtitles
