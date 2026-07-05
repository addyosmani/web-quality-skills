# Using web-quality-skills with Codex

This repository is also a [Codex plugin](https://developers.openai.com/codex/plugins/build). The Codex plugin packages the same skills under `codex/skills/` so they can be discovered reliably across platforms, including Windows.

## Install (one command)

```bash
codex plugin marketplace add addyosmani/web-quality-skills
```

> Requires Codex CLI v0.122 or later. On older releases the command was `codex marketplace add`. See the [Codex CLI docs](https://developers.openai.com/codex/cli).

Codex clones the repo into `~/.codex/plugins/web-quality-skills/`, registers the marketplace in `~/.codex/config.toml`, and makes the plugin available. Restart Codex if it's already running.

Local clones work too:

```bash
codex plugin marketplace add /path/to/your/clone
```

## Usage

After install, invoke a skill in Codex chat with `@` (e.g. `@performance`, `@accessibility`, `@core-web-vitals`) or just describe the task and let Codex pick the right skill. All 6 skills under `skills/` are available:

- `web-quality-audit`
- `performance`
- `core-web-vitals`
- `accessibility`
- `seo`
- `best-practices`

## How it works

- `codex/.codex-plugin/plugin.json` — Codex plugin manifest. Points `skills` at `./skills/`.
- `codex/skills` — real copied skill folders used by the Codex plugin. Keeping this as a real directory avoids Windows installs receiving a plain symlink text file instead of a traversable skills directory.
- `.agents/plugins/marketplace.json` — marketplace entry declaring the plugin at `./codex`. Codex requires plugins to live in a subdirectory of the marketplace root.
- `skills/<name>/SKILL.md` — unchanged. Codex and Claude Code share the same `name` + `description` frontmatter format, so one file serves both platforms.

## Maintainer note

When updating the root `skills/` directory, sync those changes into `codex/skills/` before publishing a Codex plugin release.
