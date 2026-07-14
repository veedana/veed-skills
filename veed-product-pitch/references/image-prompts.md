# Writing Image Prompts (product-pitch)

Only relevant on the *describe* branch — when the user describes the product or
spokesperson instead of uploading an image. A bare description ("a water
bottle") produces a weak image; expand it across the dimensions below, then
show the user the expanded prompt and let them adjust before generating (each
Nano Banana call is billed, ~$0.039).

Nano Banana (`fal-ai/gemini-25-flash-image`) responds well to natural,
descriptive language — write full phrases, not keyword lists, and describe what
you *want* (there is no negative-prompt field).

## Dimensions to specify
- **Subject** — the concrete thing, with defining details (material, colour, size, brand cues).
- **Framing / shot** — hero product shot, head-and-shoulders portrait, etc.; compose for the chosen `aspect_ratio` (vertical for 9:16).
- **Lighting** — e.g. soft studio lighting, natural window light.
- **Background** — clean/seamless for products; plain, uncluttered for people.
- **Style** — photorealistic product photography, professional headshot, etc.

## Product prompt
Expand the user's product description across the dimensions above.

- User says: "a water bottle"
- Expanded: "A matte-black stainless-steel insulated water bottle, upright and
  centered, on a clean light-grey seamless studio background, soft even studio
  lighting, subtle reflection, photorealistic product photography, sharp focus."

## Spokesperson prompt
The spokesperson image is fed into Fabric, so its prompt must also satisfy the
talking-head input rules (see veed-talking-head's "For best results"): **one
clearly-visible, well-lit face, head-and-shoulders framing, expression matching
the script's tone, plain background**. This image sets the final video's quality.

- User says: "a friendly young woman"
- Expanded: "A friendly woman in her late 20s, warm natural smile, looking
  directly at camera, head-and-shoulders framing, soft daylight, plain light
  background, photorealistic professional portrait, sharp focus on the face."

Append the fixed qualifiers the skill already adds ("clear face, looking at
camera, professional photo, plain background") and generate in the chosen
`aspect_ratio`.

## A note on aspect ratio
Fabric snaps its output to a fixed set of aspect buckets (dimensions divisible
by 64), so exact 9:16 / 1:1 / 16:9 may shift slightly (e.g. 16:9 → ~1.86).
Generate the image in the requested `aspect_ratio` anyway — it gets the framing
close — but don't promise the final video will match the ratio to the pixel.

## Show before generating
Show the user the expanded prompt(s) and invite tweaks before running. Only
generate once they're happy — each Nano Banana call is billed (~$0.039).
