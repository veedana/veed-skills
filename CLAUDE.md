# CLAUDE.md — VEED Skills

## What this is

VEED Skills is a set of self-contained skills that teach an AI agent to call VEED's video APIs (hosted on Fal). Five skills today, in two categories:

**Endpoint skills** — each calls a single VEED API, stateless and independent:

- **veed-talking-head** — image + audio → lip-synced presenter video (Fabric 1.0)
- **veed-talking-head-text** — image + text script → lip-synced video with AI-generated speech (Fabric 1.0 Text)
- **veed-subtitles** — video → styled, burned-in captions (27 presets, 125+ languages)
- **veed-background-removal** — video → clean subject (3 modes)

**Workflow skills** — chain endpoint skills together in a single flow:

- **veed-product-pitch** — product image/description + spokesperson image/description + script/audio → finished product spokesperson video with optional subtitles (chains image generation + Fabric talking head + subtitles)

Endpoint skills are independent — they don't share state and don't depend on each other. Workflow skills orchestrate endpoint skills but likewise don't persist state between invocations. An agent can install any subset.

## Architecture

```
veed-skills/
├── README.md                       # Public-facing description
├── INSTALL.md                      # Host-specific install paths
├── CLAUDE.md                       # This file. Conventions, mode-detection ladder, decisions.
├── LICENSE                         # MIT
├── docs/                           # PRDs / ADRs / design notes (0001-, 0002-, ...)
├── .mcp.json                       # Fal MCP server config (Bearer FAL_KEY)
├── .claude-plugin/
│   ├── plugin.json                 # Claude Code plugin manifest
│   └── marketplace.json            # Marketplace listing with 5 skills
├── .codex-plugin/plugin.json       # Codex manifest
├── .cursor-plugin/plugin.json      # Cursor manifest
├── veed-talking-head/SKILL.md      # Endpoint skill
├── veed-talking-head-text/SKILL.md # Endpoint skill
├── veed-subtitles/SKILL.md         # Endpoint skill
├── veed-background-removal/SKILL.md # Endpoint skill
└── veed-product-pitch/SKILL.md     # Workflow skill (chains endpoint skills)
```

There is **no root SKILL.md**. Endpoint skills are independent; the workflow skill (product-pitch) chains them but shares no state between invocations.

## Mode-detection ladder (transport)

Skills route the actual Fal API call through one of two transports. **Detect at runtime, in this order**:

1. **Fal MCP** — if tools matching `mcp__fal-ai__*` are visible in the toolset (the agent has connected the Fal remote MCP via `claude mcp add ... https://mcp.fal.ai/mcp ...`), prefer it. Call `mcp__fal-ai__run` with `model: "veed/<slug>"` and the appropriate `arguments`.
2. **Direct API** (`fal_client` / curl) — if MCP is unavailable but `FAL_KEY` is set in the environment, fall back to `fal_client.subscribe("veed/<slug>", arguments={...})` for Python-capable agents, or curl against `https://fal.run/veed/<slug>` with `Authorization: Key $FAL_KEY` for everyone else.

Skills should never call `api.fal.ai` directly with a hand-rolled HTTP client — go through `fal_client` or the documented `fal.run` REST endpoints.

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

- **Today:** every veed skill is well under 300 lines (largest is `veed-subtitles` at ~200 lines). No action needed.
- **When a SKILL.md grows past ~300 lines:** migrate procedural detail, gallery tables, and error matrices into a sibling `references/` folder loaded on demand (`See references/preset-gallery.md for the full preset list.`). Keep the SKILL.md as a thin decision tree + pointers.
- **What stays in SKILL.md regardless:** frontmatter, the decision tree for which mode/path to take, critical rules that apply every turn, short pointers to references.
- **The test:** if removing a section from `SKILL.md` would *not* break the agent's ability to decide what to do next, it belongs in `references/`. If it would, it stays.

## Self-contained bundles

Each skill must be installable on its own via `gh skill install veedana/veed-skills <skill-name>`. That means:

- No `../` references in `SKILL.md` or anything under `references/` / `scripts/`. Refer only inside the skill's own folder.
- Every file mentioned in `SKILL.md` exists in the skill's bundle.
- Conversely, every file in `references/` and `scripts/` is mentioned (linked) from `SKILL.md` — no orphans.

(Today no skill ships `references/` or `scripts/`. This is enforced lazily; document the rule now so the convention holds when the first one is added.)

## Key decisions

Validated decisions that should not be revisited without new data:

1. **Curl/`fal_client`-first, no veed CLI.** Building a veed-branded CLI (à la heygen-cli, higgsfield) is months of engineering. Defer indefinitely; the scaffold leaves seams (mode-detection ladder, `.mcp.json`) for a future CLI to slot in without rewrites.
2. **Fal MCP is the preferred transport.** Official, free, hosted on Vercel at `https://mcp.fal.ai/mcp`. Bearer auth using `FAL_KEY`. We pay nothing; users pay only for model runs at the same rates as direct API calls.
3. **Fal MCP exposes a generic surface, not typed per-model tools.** 9 tools (`search`, `run`, `chain`) over the whole Fal catalog. Skills' calls look like `mcp__fal-ai__run` with `model: "veed/subtitles"`, not `mcp__fal-ai__veed_subtitles`.
4. **Fal MCP OAuth is not yet live.** Until Fal ships OAuth, `claude.ai` Custom Connectors and Claude Desktop (which require OAuth) cannot use it. Claude Code, Cursor, Codex, Windsurf work today via the Bearer-key path.
5. **No process overhead in Phase 1.** No CI, no release-please, no `CONTRIBUTING.md`, no `CODEOWNERS`, no `INSTALL_FOR_AGENTS.md`, no `setup` script. Add only when there's evidence one is needed.
6. **Brand assets ship in `assets/`.** `assets/icon.png` (515×512) is referenced as both `composerIcon` and `logo` in `.codex-plugin/plugin.json` and as `logo` in `.cursor-plugin/plugin.json`. Brand colour is `#96FF1A` (VEED green). A dedicated wordmark logo can replace the icon-for-logo reuse later if marketplace detail pages need it.
7. **Endpoint skills are independent — no shared state files.** Unlike heygen's `AVATAR-<NAME>.md` pattern that coordinates avatar→video skills, veed's endpoint skills (talking-head, talking-head-text, subtitles, background-removal) are stateless and independent. Workflow skills (product-pitch) chain endpoint skills together in a single invocation but don't persist state between runs — no cross-invocation coordination files. If a future workflow needs shared state across invocations, revisit.

## When this file changes

Edit `CLAUDE.md` when:

- The mode-detection ladder changes (new transport, deprecation of an old one)
- A "key decision" above is overturned by new data
- A new convention is added (e.g. a `scripts/` pattern, a per-skill state file)

Don't edit it for:

- Per-skill content changes (those go in the skill's own `SKILL.md`)
- Bug fixes or wording polish in skill files
