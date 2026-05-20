# Install VEED Skills

Grab a Fal API key at <https://fal.ai/dashboard/keys>, then pick the install path that matches your agent.

The repo ships **four independent skills** you can install in any combination:

- `veed-talking-head` — image + audio → lip-synced video
- `veed-talking-head-text` — image + text → lip-synced video (with TTS)
- `veed-subtitles` — video → styled, burned-in captions
- `veed-background-removal` — video → clean subject

Skills are independent — they don't share state and don't depend on each other.

## Option 1 — `gh skill install` (works across 12+ agents)

If you have [GitHub CLI](https://cli.github.com) v2.90+, this is the most portable install. `gh skill` writes to the right directory for your agent automatically (Claude Code, Cursor, Codex, Gemini CLI, GitHub Copilot, Junie, Goose, OpenHands, Amp, Cline, OpenCode, Warp, and more):

```bash
gh skill install veedana/veed-skills veed-talking-head
gh skill install veedana/veed-skills veed-talking-head-text
gh skill install veedana/veed-skills veed-subtitles
gh skill install veedana/veed-skills veed-background-removal
```

Project scope (current repo only) is the default. For user scope (every project on this machine):

```bash
gh skill install veedana/veed-skills veed-talking-head --scope user
# ...etc per skill
```

Pin to a release tag for reproducibility:

```bash
gh skill install veedana/veed-skills veed-talking-head@v1.0.0 --pin
```

## Option 2 — Git clone

Clone into your agent's skills directory:

| Agent | Default install path |
|---|---|
| **Claude Code** | `~/.claude/skills/veed-skills` |
| **Cursor** | `~/.cursor/skills/veed-skills` |
| **Codex** | `~/.codex/skills/veed-skills` |
| **Other** | Whatever path your agent loads skills from. Ask the agent if unsure. |

```bash
git clone https://github.com/veedana/veed-skills.git ~/.claude/skills/veed-skills
```

After cloning, the four skills are auto-discovered at `veed-talking-head/SKILL.md`, `veed-talking-head-text/SKILL.md`, `veed-subtitles/SKILL.md`, and `veed-background-removal/SKILL.md`.

## Auth

Two paths. Both use the same `FAL_KEY` and bill the same per-model-run rates at Fal.

| Priority | Mode | Trigger | Best for |
|---|---|---|---|
| 1 | **Fal MCP** (Bearer auth) | `mcp__fal-ai__*` tools visible | Claude Code, Cursor, Codex, Windsurf |
| 2 | **Direct API** (`fal_client` / curl) | `FAL_KEY` env var set | Any agent that can shell out or run Python |

### Path 1 — Connect Fal MCP (preferred)

Fal runs an official remote MCP server at `https://mcp.fal.ai/mcp` (hosted on Vercel, free, stateless). When connected, your agent can call any Fal model — including all VEED skills — via the `mcp__fal-ai__run` tool. No Python or curl required.

**Claude Code:**

```bash
claude mcp add --transport http fal-ai https://mcp.fal.ai/mcp \
  --header "Authorization: Bearer $FAL_KEY"
```

**Cursor / Codex:** add to your MCP config (varies by host — see your agent's MCP setup docs and point it at `https://mcp.fal.ai/mcp` with the same Bearer header).

> **Caveat:** Fal MCP currently supports API-key Bearer auth only. OAuth is "coming soon" per Fal. Until OAuth ships, `claude.ai` Custom Connectors and Claude Desktop cannot reach Fal MCP — use Path 2 instead.

### Path 2 — Direct API (fallback)

Export your Fal key:

```bash
export FAL_KEY=your_key_here
```

For persistence across shells, add the export to `~/.zshrc`, `~/.bashrc`, or equivalent.

For Python-capable agents, the skills will use `fal_client`:

```bash
pip install fal-client
```

For non-Python agents, the skills shell out to curl against `https://fal.run/veed/<slug>`. No extra install needed beyond curl itself.

## First run

Paste this prompt to your agent (works for any install option above):

> Install the VEED Skills from <https://github.com/veedana/veed-skills.git> — clone it into your skills directory (find it via your config or ask the user if unsure). Then either: (a) connect Fal's remote MCP at <https://mcp.fal.ai/mcp> with `Authorization: Bearer $FAL_KEY` and skip the rest, OR (b) `export FAL_KEY=<key>` and `pip install fal-client`. Then use veed-talking-head-text to animate this photo: [paste image URL] saying "Hello from VEED."

## Troubleshooting

**The agent says `FAL_KEY is not set` but I exported it.** The export only applies to the shell that ran it. If the agent runs in a separate process (most do), the export needs to be in `~/.zshrc` / `~/.bashrc` and either re-sourced or used in a fresh shell.

**MCP tools listed but the skill is using curl/fal_client anyway.** Some skills fall back to the direct API if a specific MCP tool can't be resolved. Check that `mcp__fal-ai__run` is visible (not just `mcp__fal-ai__search`). If it isn't, your MCP config may be partial — re-run the `claude mcp add` command with the full header.

**`Authentication required` from Fal MCP.** The Bearer header isn't reaching the server. Verify `echo $FAL_KEY` is non-empty, then re-run `claude mcp add` so the header is captured with the resolved value (some shells expand `$FAL_KEY` only at the moment of the command).

**Skill loads but `/veed:<name>` doesn't autocomplete.** The plugin manifest at `.claude-plugin/plugin.json` (or your host's equivalent) wasn't picked up. Most agents need a restart after installing a new plugin. For `gh skill install`, this is usually automatic; for git clone, restart your agent.

## Upgrade

```bash
cd ~/.claude/skills/veed-skills && git pull origin main
```

Or, if you installed via `gh skill install`:

```bash
gh skill update veedana/veed-skills veed-talking-head
# ...per skill
```
