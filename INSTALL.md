# Install VEED Skills

Grab a Fal API key at <https://fal.ai/dashboard/keys>, then pick the install path that matches your agent.

The repo ships **five skills** you can install in any combination:

- `veed-talking-head` — image + audio → lip-synced video
- `veed-talking-head-text` — image + text → lip-synced video (with TTS)
- `veed-subtitles` — video → styled, burned-in captions
- `veed-background-removal` — video → clean subject
- `veed-product-pitch` — product + spokesperson + script → finished product spokesperson video (workflow that chains the skills above)

Endpoint skills are independent — they don't share state and don't depend on each other. The workflow skill (product-pitch) chains endpoint skills but doesn't persist state between invocations.

## Option 1 — `gh skill install` (works across 12+ agents)

If you have [GitHub CLI](https://cli.github.com) v2.90+, this is the most portable install. `gh skill` writes to the right directory for your agent automatically (Claude Code, Cursor, Codex, Gemini CLI, GitHub Copilot, Junie, Goose, OpenHands, Amp, Cline, OpenCode, Warp, and more):

```bash
gh skill install veedana/veed-skills veed-talking-head
gh skill install veedana/veed-skills veed-talking-head-text
gh skill install veedana/veed-skills veed-subtitles
gh skill install veedana/veed-skills veed-background-removal
gh skill install veedana/veed-skills veed-product-pitch
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

After cloning, all five skills are auto-discovered at `veed-talking-head/SKILL.md`, `veed-talking-head-text/SKILL.md`, `veed-subtitles/SKILL.md`, `veed-background-removal/SKILL.md`, and `veed-product-pitch/SKILL.md`.

## Auth

The skills run through the [genmedia CLI](https://github.com/fal-ai-community/genmedia-cli), which handles model discovery, file upload, execution, and downloads against Fal. Install it, then configure your Fal API key (from <https://fal.ai/dashboard/keys>):

```bash
genmedia setup
```

For agents / CI without a TTY, configure non-interactively:

```bash
export FAL_KEY=your_key_here
genmedia setup --non-interactive --api-key "$FAL_KEY"
```

`genmedia setup` stores the key locally (or reads `FAL_KEY` from the environment). You're billed per model run at standard Fal rates — the CLI adds no markup.

## First run

Paste this prompt to your agent (works for any install option above):

> Install the VEED Skills from <https://github.com/veedana/veed-skills.git> — clone it into your skills directory (find it via your config or ask the user if unsure). Make sure the genmedia CLI is installed and `genmedia setup` has been run with a Fal key. Then use veed-talking-head-text to animate this photo: [paste image URL] saying "Hello from VEED."

## Troubleshooting

**The agent says the Fal key isn't set.** Run `genmedia setup` (or `genmedia setup --non-interactive --api-key "$FAL_KEY"`). If you rely on the `FAL_KEY` environment variable, note that an `export` only applies to the shell that ran it — add it to `~/.zshrc` / `~/.bashrc` for persistence across shells.

**`genmedia: command not found`.** The CLI isn't installed or isn't on your PATH. See <https://github.com/fal-ai-community/genmedia-cli> for install instructions, then re-run `genmedia setup`.

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
