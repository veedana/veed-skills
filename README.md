# VEED Skills

AI agent skills for video generation, lipsync, subtitles and background 
removal — powered by VEED's APIs on Fal.

Works with Claude Code, Cursor, and any AI agent that supports 
Markdown-based skills.

## What is this?

Each skill is a simple instruction file that teaches your AI agent how to 
call VEED's video APIs. Once installed, you can generate talking head 
videos, remove backgrounds and add styled subtitles just by describing 
what you want — in plain language.

## Prerequisites

You need a Fal account and API key:
1. Sign up free at https://fal.ai
2. Get your API key at https://fal.ai/dashboard/keys
3. Set it in your terminal: export FAL_KEY=your_key_here

## Available Skills

### veed-talking-head
Generate a talking head video from a static image and audio file.
Powered by VEED's Fabric 1.0 model.
- 480p: $0.08/second · 720p: $0.15/second
- Max duration: 5 minutes

### veed-background-removal
Remove the background from any video. Three modes: standard, fast, 
and green screen.
- Refine ON: $0.0225 per 30 frames
- Refine OFF: $0.012 per 30 frames

### veed-subtitles
Coming soon — add styled static or animated captions to any video.

## Installation

### Claude Code
npx skills add veedana/veed-skills

### Manual
git clone https://github.com/veedana/veed-skills
Copy the skill folder(s) you want into your agent's skills directory.

## Usage Examples

Once installed, just ask your agent:

"Generate a talking head video from this image [url] with this audio [url]"

"Remove the background from this video [url]"

"Add animated subtitles to this video [url]"

## Powered by VEED

VEED is the world's leading AI video platform.
Learn more at https://www.veed.io/api
