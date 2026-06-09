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
The user needs:
- A Fal API key — get one free at https://fal.ai/dashboard/keys
- Set it as an environment variable: export FAL_KEY=your_key_here

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
   "How would you like the voice to sound? You can use descriptors like
   'British accent' or 'Confident and warm', or say 'auto' to let VEED
   pick a voice based on the spokesperson image."
   - If the user provides a description → store as voice_description
   - If the user says 'auto' or similar → set voice_description to None

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
   - If no → set subtitle_preset to None

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

If product_image_path is set (user uploaded):
    product_image_url = fal_client.upload_file(product_image_path)

Else (user described):
    import fal_client
    result = fal_client.subscribe(
        "fal-ai/gemini-25-flash-image",
        arguments={
            "prompt": product_prompt,
            "num_images": 1
        },
        with_logs=True
    )
    product_image_url = result["images"][0]["url"]

## Step 3 — Get or generate the spokesperson image

If spokesperson_image_path is set (user uploaded):
    spokesperson_image_url = fal_client.upload_file(spokesperson_image_path)

Else (user described):
    result = fal_client.subscribe(
        "fal-ai/gemini-25-flash-image",
        arguments={
            "prompt": spokesperson_prompt + ", clear face, looking at camera, "
                      "professional photo, plain background",
            "num_images": 1
        },
        with_logs=True
    )
    spokesperson_image_url = result["images"][0]["url"]

## Step 4 — Composite the spokesperson holding the product

Use Nano Banana's edit endpoint with both images and a fixed prompt:

result = fal_client.subscribe(
    "fal-ai/gemini-25-flash-image/edit",
    arguments={
        "image_urls": [spokesperson_image_url, product_image_url],
        "prompt": "Make the model hold this product in their hand. "
                  "Keep the spokesperson's face clearly visible and "
                  "looking at the camera. Natural pose, well-lit."
    },
    with_logs=True
)
composite_image_url = result["images"][0]["url"]

## Step 5 — Generate the talking head video

Branch based on mode:

If mode == "audio":
    audio_url = fal_client.upload_file(audio_path)
    result = fal_client.subscribe(
        "veed/fabric-1.0",
        arguments={
            "image_url": composite_image_url,
            "audio_url": audio_url,
            "resolution": resolution
        },
        with_logs=True
    )

If mode == "text":
    arguments = {
        "image_url": composite_image_url,
        "text": script_text,
        "resolution": resolution
    }
    if voice_description is not None:
        arguments["voice_description"] = voice_description
    result = fal_client.subscribe(
        "veed/fabric-1.0/text",
        arguments=arguments,
        with_logs=True
    )

video_url = result["video"]["url"]

If subtitle_preset is None: return video_url to the user and stop here.

## Step 6 — Add subtitles (only if subtitle_preset is set)

Follow the veed-subtitles skill workflow using video_url as the input.
Pass:
- video_url: the talking head video URL from Step 5
- preset: subtitle_preset

Do not ask the user about language, customization, or SRT input — for
this workflow, use auto-transcription and preset defaults. If the user
wants more control, suggest running veed-subtitles separately afterwards.

result = fal_client.subscribe(
    "veed/subtitles",
    arguments={
        "video_url": video_url,
        "preset": subtitle_preset
    },
    with_logs=True
)
final_video_url = result["video"]["url"]

Return final_video_url to the user.

## After the call
- Return the final video URL to the user
- Warn the user that Fal URLs expire after ~24 hours — download the
  file locally if they want to keep it
- 401 error: FAL_KEY is invalid or not set
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
