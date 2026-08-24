# Using web-quality-skills with Codex

This repository is also a [Codex plugin](https://developers.openai.com/codex/plugins/build). The same `skills/` directory used by Claude Code is consumed by Codex — no files are copied or duplicated.

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

## Optional live browser measurements

The plugin ships skills, not a bundled browser server. If your Codex environment already exposes [Chrome DevTools MCP](https://github.com/ChromeDevTools/chrome-devtools-mcp), the skills use performance traces, CrUX context, Lighthouse audits, rendered accessibility snapshots, console messages, and network requests automatically.

Without those tools, the same skills fall back to Lighthouse CLI, PageSpeed Insights/CrUX tools, manual browser checks, and static source inspection. Chrome DevTools MCP is an enhancement, not an installation requirement.

## How it works

- `codex/.codex-plugin/plugin.json` — Codex plugin manifest. Points `skills` at `./skills/`.
- `codex/skills` — git-tracked symlink to `../skills` (9 bytes, mode `120000`). Keeps the plugin directory self-contained without duplicating skill content. macOS and Linux handle this natively.
- `.agents/plugins/marketplace.json` — marketplace entry declaring the plugin at `./codex`. Codex requires plugins to live in a subdirectory of the marketplace root.
- `skills/<name>/SKILL.md` — unchanged. Codex and Claude Code share the same `name` + `description` frontmatter format, so one file serves both platforms.

## No symlinks (Windows or personal preference)

If symlinks don't work on your machine, replace the symlink with a real copy of the skills inside the plugin directory. Run this from the repo root after cloning:

```bash
rm codex/skills
cp -R skills codex/skills
```

This is a **local-only** change — don't commit it. The upstream repo keeps the symlink so `codex marketplace add addyosmani/web-quality-skills` stays a one-liner for everyone else.
