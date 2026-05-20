# 0001 — PRD: Multi-host scaffold for veed-skills (Phase 1)

- **Status:** Proposed
- **Date:** 2026-05-20
- **Branch:** `t/scaffold`
- **Author:** Timur (with Ana)

## Problem Statement

veed-skills today is 4 `SKILL.md` files and a thin README. For users to *find* and *install* it from their AI agent of choice (Claude Code, Codex, Cursor), the repo needs the standard discovery scaffold those hosts expect: plugin manifests, an MCP transport pointer, a proper LICENSE, conventions docs, and host-specific install paths.

Without this layer, even users on a Veed-favourable agent can't `/plugin` their way to our skills, and our discovery surface is limited to whoever happens to find the GitHub repo. Competitors (heygen, higgsfield) ship rich distribution stacks; popular reference repos (mattpocock/skills) ship a lean version of the same idea. veed-skills currently ships neither.

## Solution

Add a **Phase 1 distribution scaffold** to `veed-skills` on branch `t/scaffold`. Optimise for *discovery surface across Claude Code + Codex + Cursor*, not for process overhead (CI, release automation, governance). 10 net-new files, no modifications to existing `SKILL.md` content.

Auth/transport: lead with **Fal's official remote MCP** (`https://mcp.fal.ai/mcp` — free, hosted on Vercel, API-key Bearer auth) as the preferred path; fall back to `curl` / `fal_client` with `FAL_KEY` for hosts that can't yet do remote MCP. Document this mode-detection ladder in `CLAUDE.md` so it survives later edits.

Pitch: **"VEED Skills — Agentic AI Video Creation"** with subtitle *"Drop-in skills that teach your agent to make video with VEED."*

## Implementation Decisions

### Repo essentials

1. **`LICENSE`** — MIT, `Copyright (c) 2026 VEED LIMITED`
2. **`.gitignore`** — `.DS_Store`, `.env`, `__pycache__/`, common OS / editor junk
3. **`README.md`** — full rewrite. Sections: hero (pitch + subtitle), per-skill table (4 rows: talking-head, talking-head-text, subtitles, background-removal), auth (MCP primary + curl fallback with priority table), install matrix (Claude Code / Codex / Cursor), "things to try" prompt examples, links
4. **`CLAUDE.md`** — ~100 lines. Architecture (4 independent skills, no shared state), **mode-detection ladder** (`mcp__fal-ai__*` tools visible → MCP; else `FAL_KEY` set → curl / `fal_client`), frontmatter required fields (`name`, `description`, `version`), 300-line rule documented as aspirational (skills that grow past it migrate content to `references/`), key-decisions log
5. **`INSTALL.md`** — host-specific install paths: `gh skill install veedana/veed-skills <skill>` (per-skill × 4), manual git clone with default skills directories per host, MCP setup snippet, curl / `FAL_KEY` fallback. Companion `INSTALL_FOR_AGENTS.md` deferred to Phase 2

### Distribution manifests

6. **`.claude-plugin/plugin.json`** — `name: "veed"`, `version: "1.0.0"`, MIT, repo `https://github.com/veedana/veed-skills`, keywords `[veed, video, talking-head, lipsync, subtitles, background-removal, ai-video, fal, agentic-video]`
7. **`.claude-plugin/marketplace.json`** — one plugin entry with `skills` array of 4: `talking-head` → `/veed:talking-head`, `talking-head-text` → `/veed:talking-head-text`, `subtitles` → `/veed:subtitles`, `background-removal` → `/veed:background-removal`
8. **`.codex-plugin/plugin.json`** — mirrors heygen's structure with veed values. `interface` block has `displayName`, `shortDescription`, `longDescription`, `developerName: "VEED"`, `category: "Design"`, `capabilities: ["Read", "Write"]`, `websiteURL: "https://veed.io"`, `defaultPrompt` (4 example prompts). **Omit** `composerIcon`, `logo`, `brandColor` — re-added in Phase 2 polish PR alongside `assets/`
9. **`.cursor-plugin/plugin.json`** — `$schema: "https://cursor.com/schemas/cursor-plugin/plugin.json"`, `name: "veed"`, `displayName: "VEED"`, `publisher: "VEED"`, `category: "developer-tools"`, `skills: ["./veed-talking-head/", "./veed-talking-head-text/", "./veed-subtitles/", "./veed-background-removal/"]`. **Omit** `logo` field

### Transport

10. **`.mcp.json`** — single entry pointing at the official Fal remote MCP:

```json
{
  "mcpServers": {
    "fal-ai": {
      "type": "http",
      "url": "https://mcp.fal.ai/mcp",
      "headers": { "Authorization": "Bearer ${FAL_KEY}" }
    }
  }
}
```

### Constants

- Version: `1.0.0` across all manifests and existing skill frontmatter
- Repo org: `veedana/veed-skills` for now; transfer to `veedstudio` later relies on GitHub's automatic org-transfer redirects (one-time `sed` for hardcoded URLs)

### Commit order on `t/scaffold`

1. `docs/0001-prd-scaffold-phase-1.md` (this file)
2. `LICENSE`, `.gitignore`
3. `README.md` rewrite
4. `CLAUDE.md` + `INSTALL.md`
5. `.claude-plugin/*` + `.codex-plugin/plugin.json` + `.cursor-plugin/plugin.json` + `.mcp.json`

## Testing Decisions

No automated tests — Phase 1 ships zero CI infrastructure. Validation is **manual smoke testing across the three agent hosts**:

1. **Claude Code direct install:** `gh skill install veedana/veed-skills veed-talking-head` on a fresh project. Verify `SKILL.md` loads, frontmatter parses, `/veed:talking-head` slash command appears.
2. **Claude Code MCP:** `claude mcp add --transport http fal-ai https://mcp.fal.ai/mcp --header "Authorization: Bearer $FAL_KEY"`. Verify `mcp__fal-ai__*` tools appear in toolset.
3. **Codex:** install via Codex's plugin flow. Verify plugin appears with correct `displayName` + `shortDescription`.
4. **Cursor:** install via Cursor's plugin marketplace flow. Verify plugin loads and `skills` array resolves.
5. **Manual link / frontmatter audit:** read every modified markdown file end-to-end before opening PR. No broken relative links, no orphan references.

Quality bar: PR is mergeable when (a) all 10 files exist with the documented schemas, (b) at least one of the three host installs has been smoke-tested manually, (c) README reads clean to a fresh reader.

## Out of Scope (Phase 2+)

Deferred to follow-up branches, in rough priority order:

- `CONTRIBUTING.md` — solo-contributor today; revisit on first external PR
- `.github/workflows/validate-skills.yml` — no load-bearing conventions yet
- `.github/` issue + PR templates — zero issue/PR volume today
- `VERSION` + `CHANGELOG.md` + release-please — no release ceremony yet
- ClawHub publish workflow + `.clawhubignore` — only if veed targets ClawHub as a channel
- `.app.json` (ChatGPT app linkage) — veed has no ChatGPT app yet
- `assets/icon.png` + `assets/logo.png` + re-adding `composerIcon` / `logo` / `brandColor` — Phase 2 polish PR
- `setup` shell script — `claude mcp add` + `export FAL_KEY` are one-liners
- `CODEOWNERS` — add when there's >1 maintainer
- `references/` refactor for `veed-subtitles` — only when SKILL.md outgrows ~12 KB
- `evals/` autoresearch loop (Notion-tracked round-by-round) — separate scoped project
- `platforms/<host>/` per-host SKILL variants — premature
- veed CLI (Go binary à la heygen-cli, npm à la higgsfield) — months of engineering, separate brainstorm
- Multi-skill shared state files (heygen's `AVATAR-*.md` pattern) — veed skills are independent today

## Further Notes

- **Fal MCP OAuth gap.** Fal's hosted MCP currently supports only API-key Bearer auth. Until Fal ships OAuth, `claude.ai` Custom Connectors and Claude Desktop (which require OAuth) can't reach it. Claude Code / Codex / Cursor are unaffected. Documented in `CLAUDE.md` key-decisions log.
- **Generic vs typed MCP surface.** Fal's MCP exposes 9 generic tools (`search`, `run`, `chain`) over its whole catalog, not per-model named tools. Skills' calls look like `mcp__fal-ai__run` with `model: "veed/subtitles"` and `arguments: {...}`. Different from heygen's typed-tool pattern.
- **Org transition `veedana` → `veedstudio`.** GitHub auto-301s old org URLs. Affected files at transition: `README.md`, `INSTALL.md`, all four plugin manifests, `marketplace.json`. One `sed` pass + repo transfer.
- **Branch already created.** `t/scaffold` exists locally.
- **Submitter (Timur) is read-only on the repo.** PRD lives as `docs/0001-prd-scaffold-phase-1.md` in-tree; if a GitHub issue is wanted in addition, Ana opens it manually.
