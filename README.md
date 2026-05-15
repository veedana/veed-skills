# VEED Skills

AI agent skills for video generation, lipsync, subtitles and background
removal — powered by VEED's APIs on Fal.

Works with Claude Code desktop, Claude Code CLI, Cursor, and any AI agent
that supports Markdown-based skills.

## What is this?

Each skill is a simple instruction file that teaches your AI agent how to
call VEED's video APIs. Once installed, you can generate talking head
videos, remove backgrounds and add styled subtitles just by describing
what you want — in plain language. No coding required.

## How is this different from VEED's MCP?

VEED also has an MCP server that connects directly to Claude. The key
differences:

| | VEED MCP | VEED Skills (this repo) |
|---|---|---|
| Needs VEED account | Yes | No |
| Needs Fal account | No | Yes |
| Custom images | No (20 presets) | Yes — any image |
| Maintained by | VEED engineering | GitHub file |
| Target audience | Anyone | Developers |

Use the MCP for a polished out-of-the-box experience. Use these skills
for custom images and full API flexibility.

## Prerequisites

You need a Fal account and API key:
1. Sign up free at https://fal.ai
2. Get your API key at https://fal.ai/dashboard/keys
3. Set it in your terminal: export FAL_KEY=your_key_here

## Getting your image or video file path

The skills require a local file path or public URL for images and videos.

Quickest way on Mac:
1. Find your file in Finder
2. Right-click it
3. Hold the Option key
4. Click "Copy as Pathname"
5. Paste it into Claude

If your photo is in Apple Photos:
1. Open Photos
2. Right-click the photo
3. Export → Export 1 Photo
4. Save to Desktop
5. Follow the steps above

## Available Skills

### veed-talking-head
Generate a talking head video from a static image and audio file.
Powered by VEED's Fabric 1.0 model.
- 480p: $0.08/second · 720p: $0.15/second
- Max duration: 30 seconds per generation
- Claude will ask for: image path, audio path, resolution
- Claude will show cost estimate before generating

### veed-talking-head-text
Generate a talking head video from a static image and a text script.
VEED's AI voice generator handles the speech — no audio file needed.
- 480p: $0.08/second · 720p: $0.15/second
- Max duration: 30 seconds per generation
- Claude will ask for: image path, script, voice description, resolution
- Voice description examples: "British accent", "Confident female voice,
  mid-30s, warm and professional tone". Say "auto" to let VEED generate
  a voice from the image.
- Claude will show cost estimate before generating

### veed-background-removal
Remove the background from any video. Three modes available.
- Refine ON: $0.0225 per 30 frames · Refine OFF: $0.012 per 30 frames
- Claude will ask for: video path, mode (Standard/Fast/Green screen),
  refine edges ON or OFF
- Claude will show cost estimate before generating

### veed-subtitles
Coming soon — add styled static or animated captions to any video.

## Installation

### Claude Code desktop or CLI
npx skills add veedana/veed-skills

### Manual
Copy the skill folder(s) you want into your agent's skills directory:
- Personal (all projects): ~/.claude/skills/
- Project only: .claude/skills/

## Usage examples

Once installed, just describe what you want:

"Generate a talking head video from this image /Users/me/Desktop/photo.jpg
saying Hello world"

"Make a talking head from this photo with a British accent:
/Users/me/Desktop/photo.jpg"

"Remove the background from this video /Users/me/Desktop/video.mp4"

## Notes

- Fal video URLs expire after ~24 hours. Download your video locally
  if you want to keep it.
- All three skills use fal_client.subscribe() for reliable handling of
  longer jobs and real-time progress reporting.
- Cost is estimated and shown to you before every generation — you always
  confirm before anything is charged.

## Powered by VEED

VEED is the world's leading AI video platform.
Learn more at https://www.veed.io/api
