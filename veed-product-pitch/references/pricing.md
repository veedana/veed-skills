# Product Pitch — Pricing Reference

This workflow chains multiple models. Run `genmedia pricing <endpoint> --json`
for each endpoint used for the authoritative current rate. The figures below
are indicative, at time of writing.

## Per-call costs
- Nano Banana (generation): $0.039 per image
- Nano Banana (edit/composite): $0.039 per call
- Fabric 1.0 (audio): $0.08/sec at 480p, $0.15/sec at 720p
- Fabric 1.0 text: $0.08/sec at 480p, $0.15/sec at 720p
- Subtitles: $0.10/min base, 2x for dynamic presets, 1-minute minimum

## Typical end-to-end (10-second video at 720p with subtitles)
- Both images user-uploaded: ~$1.70
- Both images AI-generated: ~$1.82

## Model pages
- Nano Banana: https://fal.ai/models/fal-ai/gemini-25-flash-image
- Nano Banana edit: https://fal.ai/models/fal-ai/gemini-25-flash-image/edit
- Fabric audio: https://fal.ai/models/veed/fabric-1.0
- Fabric text: https://fal.ai/models/veed/fabric-1.0/text
- Subtitles: https://fal.ai/models/veed/subtitles
