# VEED Subtitle Presets

30 presets in two tiers. The tier sets the cost multiplier (see the Pricing
section of SKILL.md): dynamic presets bill at 2x, basic presets at 1x.

Run `genmedia schema veed/subtitles --json` for the authoritative, current
preset enum — VEED adds presets over time, so treat the list below as the
known set rather than a fixed total.

## Dynamic presets (2x multiplier)
Richer, context-aware rendering that adapts to the input. Best for social /
hook content.

    glass, whisper, glide2, fusion, glide, terminal, handwritten,
    backdrop, backdrop2

## Basic presets (1x multiplier)
Fixed, lightweight styling with predictable output. Best for utility
captioning or high-volume pipelines.

    simple, plain, beans, corpo, boo, shadeplay, casper, capri, lowkey,
    vinta, diego, ali, slay, kitty, hustle, karl, sprout, flex, mint,
    rizz, vegas

## If the user is unsure
Suggest "glass" (dynamic) for social content, or "simple" (basic) for plain
utility captioning.
