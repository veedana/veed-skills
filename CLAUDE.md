# CLAUDE.md — VEED Skills

## What this is

VEED Skills is a set of skills that teach an AI agent to call VEED's video APIs (hosted on Fal). Five skills today, in two categories:

**Endpoint skills** — each calls a single VEED API, stateless and independent:

- **veed-talking-head** — image + audio → lip-synced presenter video (Fabric 1.0)
- **veed-talking-head-text** — image + text script → lip-synced video with AI-generated speech (Fabric 1.0 Text)
- **veed-subtitles** — video → styled, burned-in captions (30 presets, 125+ languages, optional translation)
- **veed-background-removal** — video → clean subject (3 modes)

**Workflow skills** — chain endpoint skills together in a single flow:

- **veed-product-pitch** — product image/description + spokesperson image/description + script/audio → finished product spokesperson video with optional subtitles (chains image generation + Fabric talking head + subtitles)

Endpoint skills are independent at runtime — they don't share state and don't depend on each other's execution. Workflow skills orchestrate endpoint skills but likewise don't persist state between invocations. All skills share one file, `COMMON.md` (setup, async execution pattern, common errors), so the repo installs as a whole rather than skill-by-skill.

## Architecture

```
veed-skills/
├── README.md                       # Public-facing description
├── INSTALL.md                      # Host-specific install paths
├── CLAUDE.md                       # This file. Conventions, transport, decisions.
├── COMMON.md                       # Shared: genmedia setup, async pattern, common errors
├── LICENSE                         # MIT
├── docs/                           # PRDs / ADRs / design notes (0001-, 0002-, ...)
├── .claude-plugin/
│   ├── plugin.json                 # Claude Code plugin manifest
│   └── marketplace.json            # Marketplace listing with 5 skills
├── .codex-plugin/plugin.json       # Codex manifest
├── .cursor-plugin/plugin.json      # Cursor manifest
├── veed-talking-head/SKILL.md      # Endpoint skill
├── veed-talking-head-text/SKILL.md # Endpoint skill
├── veed-subtitles/                 # Endpoint skill (+ references/: presets, customization)
├── veed-background-removal/SKILL.md # Endpoint skill
└── veed-product-pitch/             # Workflow skill (+ references/: voice-description, pricing)
```

There is **no root SKILL.md**. Endpoint skills are independent; the workflow skill (product-pitch) chains them but shares no state between invocations.

## Transport: genmedia CLI

Skills route every Fal call through the **genmedia CLI** (`fal-ai-community/genmedia-cli`) — the same execution layer the broader fal.ai community skills use. It handles model discovery, schema inspection, file upload, execution (sync or async), status polling, pricing, and downloads.

The commands skills rely on:

- `genmedia setup` — one-time auth (stores the Fal key, or reads `FAL_KEY`)
- `genmedia upload <path> --json` — push a local file to Fal's CDN, returns a URL
- `genmedia schema <endpoint> --json` — inspect a model's exact input fields before running
- `genmedia run <endpoint> --<field> <value> --json` — execute; add `--async` for a `request_id` to poll
- `genmedia status <endpoint> <request_id> --json` / `--download <template>` — poll or fetch an async job
- `genmedia pricing <endpoint> --json` — current price for a model

Endpoints are the real Fal slugs (`veed/fabric-1.0`, `veed/subtitles`, `veed/video-background-removal`, etc.). Skills should not hand-roll HTTP against `fal.run` / `api.fal.ai`, nor embed Python `fal_client` code — go through genmedia so discovery, schema, and downloads stay consistent.

## Frontmatter required fields

Every `SKILL.md` must have YAML frontmatter with at minimum:

```yaml
---
name: veed-<skill-slug>
version: 1.0.0
description: >
  One-paragraph description. Include "Use when: ..." triggers and a
  "NOT for: ..." clause to disambiguate from sibling skills.
---
```

The `description` field is what agents read to decide whether to invoke the skill. Triggers and anti-triggers matter — be specific.

## The 300-line rule (aspirational)

Each `SKILL.md` is injected into every prompt turn the agent spends in that skill. Token cost matters.

- **Today:** every SKILL.md sits well under 300 lines (largest is `veed-product-pitch` at ~250), helped by shared boilerplate in `COMMON.md` plus per-skill `references/` in `veed-subtitles` and `veed-product-pitch`.
- **When a SKILL.md grows past ~300 lines:** migrate procedural detail, gallery tables, and error matrices into a sibling `references/` folder loaded on demand (`See references/presets.md for the full preset list.`). Keep the SKILL.md as a thin decision tree + pointers.
- **What stays in SKILL.md regardless:** frontmatter, the decision tree for which mode/path to take, critical rules that apply every turn, short pointers to references.
- **The test:** if removing a section from `SKILL.md` would *not* break the agent's ability to decide what to do next, it belongs in `references/`. If it would, it stays.

## Bundle structure

The repo installs as a whole (git clone / the marketplace plugin), because all skills share `COMMON.md`. Rules:

- **Shared boilerplate lives once in `COMMON.md`** — genmedia setup, the async execution pattern (schema → run `--async` → poll → download → resume), uploading inputs, and the common error matrix. Each `SKILL.md` points to it via `../COMMON.md`; do not re-inline it.
- **`../COMMON.md` is the only permitted upward reference.** Within a skill, per-skill `references/*.md` are linked relatively and stay inside the skill folder.
- Every file linked from a `SKILL.md` exists; conversely every `references/` file is linked from its `SKILL.md` — no orphans.

Per-skill references today: `veed-subtitles/references/{presets,customization}.md`, `veed-product-pitch/references/{voice-description,pricing}.md`. No skill ships `scripts/` yet.

## Key decisions

Validated decisions that should not be revisited without new data:

1. **genmedia CLI is the transport, no veed CLI.** Building a veed-branded CLI (à la heygen-cli, higgsfield) is months of engineering. Instead, skills run on the community `genmedia` CLI, which already covers discovery, schema, upload, run, status, pricing, and downloads — and which the wider fal.ai community skills route to (its `model-routing` skill already defaults talking-head / UGC to `veed/fabric-1.0` and `veed/fabric-1.0/text`). We ride that ecosystem rather than reinventing it.
2. **No MCP / no inline `fal_client`.** Earlier scaffold used a Fal MCP → `fal_client` → curl ladder. Dropped: genmedia is one consistent path that works for any agent that can shell out, needs no per-host MCP config, and keeps schema/pricing/downloads uniform. `.mcp.json` was removed with this decision.
3. **Endpoints are real Fal slugs.** genmedia `run`/`schema`/`pricing` take the model slug directly (`veed/subtitles`, `veed/fabric-1.0`, …) — the same IDs documented on each model's Fal page.
4. **Inspect schema, don't guess fields.** Fal model schemas change. Skills should confirm exact input field names via `genmedia schema <endpoint> --json` rather than trusting a hard-coded argument list, and pull live cost via `genmedia pricing` rather than embedding prices that silently drift.
5. **No process overhead in Phase 1.** No CI, no release-please, no `CONTRIBUTING.md`, no `CODEOWNERS`, no `INSTALL_FOR_AGENTS.md`, no `setup` script. Add only when there's evidence one is needed.
6. **Brand assets ship in `assets/`.** `assets/icon.png` (515×512) is referenced as both `composerIcon` and `logo` in `.codex-plugin/plugin.json` and as `logo` in `.cursor-plugin/plugin.json`. Brand colour is `#96FF1A` (VEED green). A dedicated wordmark logo can replace the icon-for-logo reuse later if marketplace detail pages need it.
7. **Endpoint skills are independent — no shared state files.** Unlike heygen's `AVATAR-<NAME>.md` pattern that coordinates avatar→video skills, veed's endpoint skills (talking-head, talking-head-text, subtitles, background-removal) are stateless and independent. Workflow skills (product-pitch) chain endpoint skills together in a single invocation but don't persist state between runs — no cross-invocation coordination files. If a future workflow needs shared state across invocations, revisit.
8. **Shared boilerplate in `COMMON.md`, whole-repo install.** Setup, the async execution pattern, and the error matrix were duplicated verbatim across all five skills. Consolidated into `COMMON.md` (single source of truth) referenced via `../COMMON.md`. Trade-off, accepted deliberately: skills are no longer independently installable via `gh skill install <one>` — the repo installs as a whole. Chose DRY + one-place edits over per-skill install, since the skills already ship together as a plugin/marketplace bundle.

## When this file changes

Edit `CLAUDE.md` when:

- The transport changes (new execution path, deprecation of an old one)
- A "key decision" above is overturned by new data
- A new convention is added (e.g. a `scripts/` pattern, a per-skill state file)

Don't edit it for:

- Per-skill content changes (those go in the skill's own `SKILL.md`)
- Bug fixes or wording polish in skill files
