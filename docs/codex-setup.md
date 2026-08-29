# Using web-quality-skills with Codex

This repository is also a [Codex plugin](https://developers.openai.com/plugins/build/plugins). The same root-level `skills/` directory used by Claude Code is packaged directly for Codex — no files are copied or duplicated.

## Install

```bash
codex plugin marketplace add addyosmani/web-quality-skills
codex plugin add web-quality-skills@web-quality-skills
```

> Requires Codex CLI v0.131 or later. See the
> [Codex CLI docs](https://developers.openai.com/codex/cli).

The first command registers the Git marketplace. The second installs and enables the plugin from that marketplace. Start a new Codex session after installation so the bundled skills are discovered.

Local clones work too:

```bash
codex plugin marketplace add /path/to/your/clone
codex plugin add web-quality-skills@web-quality-skills
```

## Usage

After installation, invoke a namespaced skill with `$` (for example,
`$web-quality-skills:performance`) or use `/skills` to select one. You can also
describe the task and let Codex choose the appropriate skill. All 6 skills under
`skills/` are available:

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

- `.codex-plugin/plugin.json` — Codex plugin manifest at the repository root. Points `skills` at `./skills/`.
- `.agents/plugins/marketplace.json` — marketplace entry declaring the repository root (`./`) as the plugin source.
- `skills/<name>/SKILL.md` — shared unchanged by Codex and Claude Code. Both use the same `name` + `description` frontmatter format.

All manifest paths resolve inside the plugin root, so Codex can materialize the complete plugin without following cross-root symlinks.
