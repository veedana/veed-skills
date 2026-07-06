# VEED Subtitle Customization

Optional. Any field you leave out keeps the preset's default. This is the
single source of truth for subtitle customization — the SKILL.md points here
rather than restating it.

## What the user can override
- **Position** — top, center, or bottom (default: preset's default)
- **Shadow intensity** — none, min, mid, or max (improves readability over
  busy backgrounds)
- **Per-tier text styling** — font, weight (100-900), and hex colour for each
  of the two word-importance tiers (these are the exact keys the API accepts
  under `text_customizations`):
  - **baseline**: styling applied to every word by default, before any
    per-word emphasis
  - **highlighted**: mid-rank words automatically marked as noteworthy (key
    nouns, action verbs, salient adjectives) — presets typically bump size or
    weight here

## JSON shape
Pass as a JSON string on the `--customization` flag. Include only the fields
the user wants to change:

    {
      "position": "bottom",
      "shadow": "mid",
      "text_customizations": {
        "baseline":    {"font": "Inter", "weight": 500, "color": "#FFFFFF"},
        "highlighted": {"font": "Inter", "weight": 700, "color": "#FFD500"}
      }
    }

## Constraints
- position: top, center, or bottom
- shadow: none, min, mid, or max
- font: must be a supported Google Font — see
  https://www.veed.io/api/v1/subtitle-renders/fonts for the canonical list.
  Unrecognized fonts return a 400.
- weight: 100-900. Values >= 700 render as bold.
- color: hex string (e.g. "#FFFFFF")
- `text_customizations` accepts only the `baseline` and `highlighted` keys —
  any other tier name is ignored.
