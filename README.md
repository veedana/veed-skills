# VEED Skills

![VEED Skills](./assets/veed-cli-banner.png)

### Agentic AI Video Creation

Drop-in skills that teach your AI agent to make video with VEED — talking heads, product pitch videos, styled subtitles, and background removal — powered by VEED's APIs on Fal.

Works with Claude Code, Codex, Cursor, and any AI agent that supports Markdown-based skills.

```
"Make this photo say 'Welcome to our store' in a confident British accent.
Then remove the background and add animated subtitles in the glass preset."
→ generates the talking-head video → strips the background → burns captions → returns three URLs
```

## What's included

Five skills — four endpoint skills and one workflow skill — that work standalone or chain together:

| Skill | What it does | Invoke |
|---|---|---|
| **veed-talking-head** | Static image + audio file → lip-synced presenter video. Powered by VEED's Fabric 1.0. | `/veed:talking-head` |
| **veed-talking-head-text** | Static image + text script → lip-synced video. AI voice generation built in (optional voice description). | `/veed:talking-head-text` |
| **veed-subtitles** | Any video → styled, burned-in subtitles. 27 visual presets, auto-transcription in 125+ languages, full positioning control. | `/veed:subtitles` |
| **veed-background-removal** | Any video → clean subject. Three modes: standard, fast, green screen. Optional foreground refinement. | `/veed:background-removal` |
| **veed-product-pitch** | Product image/description + spokesperson image/description + script/audio → finished product spokesperson video with optional subtitles. Chains image generation, talking head, and subtitles in one flow. | `/veed:product-pitch` |

## Install

The repo ships five skills you can install independently. Pick whichever subset you need.

### Option 1 — `gh skill install` (Claude Code, Cursor, Codex, Gemini CLI, Copilot, and more)

```bash
gh skill install veedana/veed-skills veed-talking-head
gh skill install veedana/veed-skills veed-talking-head-text
gh skill install veedana/veed-skills veed-subtitles
gh skill install veedana/veed-skills veed-background-removal
gh skill install veedana/veed-skills veed-product-pitch
```

Requires GitHub CLI v2.90+. The CLI writes to the right directory for your agent automatically. See [INSTALL.md](./INSTALL.md) for per-host paths and full options.

### Option 2 — Git clone

```bash
git clone https://github.com/veedana/veed-skills.git ~/.claude/skills/veed-skills
```

After cloning, all five skills are auto-discovered at `veed-talking-head/SKILL.md`, `veed-talking-head-text/SKILL.md`, `veed-subtitles/SKILL.md`, `veed-background-removal/SKILL.md`, and `veed-product-pitch/SKILL.md`.

## Authentication

Two paths, in order of preference: **MCP** (preferred) → **direct API** with `FAL_KEY`.

| Priority | Mode | Trigger | Best for |
|---|---|---|---|
| 1 | **Fal MCP** (Bearer auth) | `mcp__fal-ai__*` tools visible in toolset | Claude Code, Cursor, Codex |
| 2 | **Direct API** (`fal_client` / curl) | `FAL_KEY` env var set | Any agent that can shell out or run Python |

Both use the same `FAL_KEY` from <https://fal.ai/dashboard/keys>. You're billed per model run at standard Fal rates either way — the MCP server itself is free.

### Connect Fal MCP (Claude Code example)

```bash
claude mcp add --transport http fal-ai https://mcp.fal.ai/mcp \
  --header "Authorization: Bearer $FAL_KEY"
```

> **Caveat:** Fal's MCP currently supports API-key Bearer auth only. OAuth is "coming soon" per Fal — until it ships, `claude.ai` Custom Connectors and Claude Desktop (which require OAuth) can't reach it. Claude Code, Cursor, Codex, Windsurf are unaffected.

### Direct API fallback

```bash
export FAL_KEY=your_key_here
```

The skills detect `FAL_KEY` and use `fal_client.subscribe(...)` or curl against `fal.run/veed/*` endpoints.

## Things to try

Once installed, try these prompts:

| Prompt | What happens |
|---|---|
| "Use veed-talking-head with this image and this audio file" | Animates the face, lip-syncs to the audio, returns the video URL |
| "Use veed-talking-head-text to make this photo say 'Our biggest launch yet' in a confident, warm voice" | Generates speech, animates, returns the video URL |
| "Use veed-subtitles to add animated captions to this video using the glass preset" | Auto-transcribes, styles, burns in captions, returns the video URL |
| "Use veed-background-removal on this clip in green-screen mode" | Removes the background, returns a clean subject video |
| "Take this product clip, remove the background, and add subtitles in the whisper preset" | Chains background-removal → subtitles, returns both URLs |
| "Create a product pitch video: here's my product [image], use a friendly female spokesperson, and have her say 'Check out our new launch'" | Generates a product composite, animates the spokesperson, optionally adds subtitles |

## Pricing

Per-skill costs (charged by Fal — these skills add no markup):

| Skill | Cost |
|---|---|
| veed-talking-head / veed-talking-head-text | $0.08/sec (480p) · $0.15/sec (720p) · max 30s per generation |
| veed-subtitles | $0.10/min input video · 2× for >1080p · 2× for dynamic presets · 1-min minimum |
| veed-background-removal | $0.0225 per 30 frames (refine ON) · $0.012 per 30 frames (refine OFF) |
| veed-product-pitch | Composite of image generation + talking-head + optional subtitles costs (see individual skills above) |

## Powered by VEED

VEED is the world's leading AI video platform. Learn more at <https://veed.io/api>.

## License

[MIT](./LICENSE) — Copyright (c) 2026 VEED LIMITED.
