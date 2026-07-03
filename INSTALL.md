# Install VEED Skills

Grab a Fal API key at <https://fal.ai/dashboard/keys>, then pick the install path that matches your agent.

The repo ships **five skills** plus a shared [`COMMON.md`](./COMMON.md) they
all reference:

- `veed-talking-head` — image + audio → lip-synced video
- `veed-talking-head-text` — image + text → lip-synced video (with TTS)
- `veed-subtitles` — video → styled, burned-in captions
- `veed-background-removal` — video → clean subject
- `veed-product-pitch` — product + spokesperson + script → finished product spokesperson video (workflow that chains the skills above)

Because the skills share `COMMON.md`, install the **whole repo** (not individual skills). Endpoint skills are otherwise independent — they don't share state and don't depend on each other; the workflow skill (product-pitch) chains them but doesn't persist state between invocations.

## Install — Git clone

Clone the whole repo into your agent's skills directory:

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

**Skill loads but `/veed:<name>` doesn't autocomplete.** The plugin manifest at `.claude-plugin/plugin.json` (or your host's equivalent) wasn't picked up. Most agents need a restart after installing a new plugin; restart your agent after cloning.

## Upgrade

```bash
cd ~/.claude/skills/veed-skills && git pull origin main
```
