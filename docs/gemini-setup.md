# Using web-quality-skills with Gemini CLI

This repository is also a [Gemini CLI extension](https://geminicli.com/docs/extensions/). The same `skills/` directory used by Claude Code is consumed by Gemini — no files are copied or duplicated.

## Install (one command)

```bash
gemini extensions install https://github.com/addyosmani/web-quality-skills
```

Make sure you have installed the [Gemini CLI](https://geminicli.com/).

Gemini clones the repo into `~/.gemini/extensions/web-quality-skills/`, registers the extension, and auto-discovers the bundled skills. Restart Gemini CLI if it's already running.

Local clones work too:

```bash
gemini extensions install /path/to/your/clone
```

## Usage

After install, the model activates a skill automatically when your prompt matches its description. All 6 skills under `skills/` are available:

- `web-quality-audit`
- `performance`
- `core-web-vitals`
- `accessibility`
- `seo`
- `best-practices`

Try prompts like _"audit my site for web quality issues"_, _"speed up this page"_, or _"WCAG audit"_ and Gemini will pick the right skill.

## Optional live browser measurements

The extension ships skills, not a bundled browser server. If Gemini already exposes [Chrome DevTools MCP](https://github.com/ChromeDevTools/chrome-devtools-mcp), the skills use performance traces, CrUX context, Lighthouse audits, rendered accessibility snapshots, console messages, and network requests automatically.

Without those tools, the same skills fall back to Lighthouse CLI, PageSpeed Insights/CrUX tools, manual browser checks, and static source inspection. Chrome DevTools MCP is an enhancement, not an installation requirement.

## How it works

- `gemini-extension.json` — extension manifest at the repo root. Gemini CLI requires this to recognize the directory as an extension.
- `skills/<name>/SKILL.md` — unchanged. Gemini CLI auto-discovers any `skills/` directory bundled with an extension. Frontmatter uses the same `name` + `description` format Claude Code expects, so one file serves both platforms.
- `.gemini/skills` — git-tracked symlink to `../skills` (mode `120000`). Lets Gemini load the same skills as **workspace skills** when you run Gemini CLI inside a clone of this repo, without going through `gemini extensions install`.

## Workspace mode (running from a clone)

If you've cloned the repo and want the skills active in just that working directory — without globally installing the extension — Gemini CLI auto-loads any skills it finds under `.gemini/skills/`. The symlink in this repo wires that up out of the box: launch Gemini CLI from the repo root and the 6 skills become available for that session.

## No symlinks (Windows or personal preference)

If symlinks don't work on your machine, replace the symlink with a real copy of the skills inside `.gemini/`. Run this from the repo root after cloning:

```bash
rm .gemini/skills
cp -R skills .gemini/skills
```

This is a **local-only** change — don't commit it. The upstream repo keeps the symlink so workspace mode stays a one-clone affair for everyone else.
