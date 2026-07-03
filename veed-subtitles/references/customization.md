# VEED Subtitle Customization

Optional. Any field you leave out keeps the preset's default. This is the
single source of truth for subtitle customization — the SKILL.md points here
rather than restating it.

## What the user can override
- **Position** — top, center, or bottom (default: preset's default)
- **Shadow intensity** — none, min, mid, or max (improves readability over
  busy backgrounds)
- **Per-tier text styling** — font, weight (100-900), and hex colour for each
  of the three word-importance tiers:
  - **accessible**: baseline styling applied to every word
  - **highlighted**: mid-rank words (key nouns, action verbs, salient
    adjectives) — presets typically bump size or weight here
  - **viral**: top-rank 'hook' words, only a handful per video (some presets
    don't use this tier, so overrides are a no-op on those)

## JSON shape
Pass as a JSON string on the `--customization` flag. Include only the fields
the user wants to change:

    {
      "position": "bottom",
      "shadow": "mid",
      "text_customizations": {
        "accessible":  {"font": "Inter", "weight": 500, "color": "#FFFFFF"},
        "highlighted": {"font": "Inter", "weight": 700, "color": "#FFD500"},
        "viral":       {"font": "Inter", "weight": 900, "color": "#FF2E63"}
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
- The "viral" tier is a no-op on presets that don't use it.
