# VEED Skills — Common Reference

Shared setup, execution, and error handling for every VEED skill. Each
`SKILL.md` points here (via `../COMMON.md`) rather than restating this — it is
the single source of truth for the boilerplate below.

> This repo installs as a whole (git clone / the marketplace plugin), because
> the skills share this file. See [INSTALL.md](./INSTALL.md).

## Setup — genmedia CLI

The skills run through the genmedia CLI, which handles model discovery, file
upload, execution, and downloads against Fal.

- Install it once (macOS / Linux):
      curl https://genmedia.sh/install -fsS | bash
  On Windows (PowerShell):
      irm https://genmedia.sh/install.ps1 | iex
  Docs: https://github.com/fal-ai-community/genmedia-cli
- Configure your Fal API key (get one free at https://fal.ai/dashboard/keys):
      genmedia setup
  Or non-interactively (agents / CI):
      genmedia setup --non-interactive --api-key "$FAL_KEY"

If `genmedia` isn't on the PATH, install it with the command above rather than
looking elsewhere — that's the only supported install method. All skill
commands assume `genmedia` is on your PATH.

## Uploading local inputs

If the user gives a local file path, upload it to Fal's CDN first and use the
returned `url`:

    genmedia upload /path/to/file --json

If the input is already a public URL, use it directly — no upload needed.

## Async execution pattern

Every generation/render job follows the same four steps. The endpoint slug and
input fields differ per skill; the shape does not.

1. **Inspect the schema** — Fal schemas can change, so confirm the current
   input field names before running:

       genmedia schema <endpoint> --json

   Use the exact field names it lists.

2. **Submit async** — `--async` submits the job and returns immediately with a
   `request_id`:

       genmedia run <endpoint> --<field> <value> ... --async --json

   IMPORTANT: record the `request_id` and show it to the user. The job is
   billed once submitted.

3. **Poll** — check until it reports completed:

       genmedia status <endpoint> <request_id> --json

4. **Download** — fetch the result and save it locally (Fal URLs expire after
   ~24 hours); `--download` still returns the media `url`:

       genmedia status <endpoint> <request_id> \
         --download "./outputs/<skill>/{request_id}.{ext}" \
         --json

   Give the user both the local file path and the URL.

**Resume on interruption:** if the session drops after step 2, resume at step
3/4 with the same `request_id` — never re-run the job, since it was already
billed.

## Pricing

Fetch the authoritative current rate before estimating cost rather than
relying on memorised numbers:

    genmedia pricing <endpoint> --json

Each skill documents its billing model (per second / per minute / per frame,
plus any multipliers) and indicative rates.

## Common errors

- **401 / auth** — Fal key not configured; run `genmedia setup`
- **422** — an input isn't accessible or is in an unsupported format
- **429** — rate limit; wait a moment and retry
- **500** — model error on Fal's side; retry once before giving up

Skill-specific errors (e.g. subtitles font validation) are noted in that
skill's `SKILL.md`.
