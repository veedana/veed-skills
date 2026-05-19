# VEED Skills

AI agent skills for video generation, lipsync, subtitles and background removal — powered by VEED's APIs on Fal.

Works with Claude Code, Cursor, and any AI agent that supports Markdown-based skills.

## What is this?

Each skill is a simple instruction file that teaches your AI agent how to call VEED's video APIs. Once installed, you can generate talking head videos, remove backgrounds, and add styled subtitles just by describing what you want — in plain language.

## Prerequisites

You need a Fal account and API key:

1. Sign up free at <https://fal.ai>
2. Get your API key at <https://fal.ai/dashboard/keys>
3. Set it in your terminal: `export FAL_KEY=your_key_here`

## Available Skills

### veed-talking-head

Generate a talking head video from a static image and an audio file. Animates any face to sync lip movements with the provided audio. Powered by VEED's Fabric 1.0 model.

- 480p: $0.08/second · 720p: $0.15/second
- Max duration: 30 seconds per generation

### veed-talking-head-text

Generate a talking head video from a static image and a text script. VEED's AI voice generator converts the script to speech automatically — no audio file needed. Powered by VEED's Fabric 1.0 Text model.

- 480p: $0.08/second · 720p: $0.15/second
- Max duration: 30 seconds per generation
- Optional voice description (e.g. "British accent", "Confident and warm")

### veed-background-removal

Remove the background from any video. Three modes: standard, fast, and green screen.

- Refine ON: $0.0225 per 30 frames
- Refine OFF: $0.012 per 30 frames

### veed-subtitles

Add styled, burned-in subtitles to any video. Handles the full pipeline — transcribe, style, and render — in one call. Choose from 27 visual presets, supply your own SRT or auto-transcribe in 125+ languages, and optionally override position, shadow, and per-tier text styling.

- Base rate: $0.10/minute of input video
- 2x multiplier for video above 1080p
- 2x multiplier for dynamic presets (`glass`, `whisper`, `glide2`, `fusion`, `glide`, `terminal`, `handwritten`)
- Minimum charge: 1 minute

## Installation

### Claude Code

```
npx skills add veedana/veed-skills
```

### Manual

```
git clone https://github.com/veedana/veed-skills
```

Copy the skill folder(s) you want into your agent's skills directory.

## Usage Examples

Once installed, just ask your agent:

> "Generate a talking head video from this image [url] with this audio [url]"

> "Make this photo say 'Welcome to our store' in a confident British accent"

> "Remove the background from this video [url]"

> "Add animated subtitles to this video [url] using the glass preset"

> "Caption this video in Spanish using the simple preset, with white text positioned at the bottom"

## Powered by VEED

VEED is the world's leading AI video platform.
Learn more at <https://www.veed.io/api>
