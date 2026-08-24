# AGENTS.md

This file provides guidance to AI coding agents (Claude Code, Cursor, Copilot, etc.) when working with code in this repository.

## Project overview

Web Quality Skills is a measurement-first collection of Agent Skills for Google Lighthouse, Chrome DevTools, Core Web Vitals, accessibility, SEO, best practices, and agentic browsing. Skills follow the [Agent Skills specification](https://agentskills.io/specification).

### Live audit routing

- Prefer capabilities over provider-specific tool names. Record a browser performance trace and analyze focused insights; with Chrome DevTools MCP, use `performance_start_trace` and `performance_analyze_insight`.
- Run live Lighthouse audits for Accessibility, SEO, Best Practices, and Agentic Browsing; with Chrome DevTools MCP, use `lighthouse_audit`. It intentionally excludes performance.
- Read `skills/performance/references/MEASUREMENT.md` for field/lab fallbacks, CrUX scope, and repeatable comparisons.
- Treat a single trace or `PerformanceObserver` result as a lab observation, not real-user data. When no page can run, label performance findings as source hypotheses.

## Directory structure

```
web-quality-skills/
├── README.md                    # Project documentation
├── AGENTS.md                    # This file
├── LICENSE                      # MIT License
└── skills/
    ├── web-quality-audit/       # Comprehensive audit skill
    │   ├── SKILL.md
    │   ├── scripts/
    │   └── references/
    ├── performance/             # Performance optimization
    │   ├── SKILL.md
    │   ├── scripts/
    │   └── references/
    ├── accessibility/           # Accessibility guidelines
    │   ├── SKILL.md
    │   ├── scripts/
    │   └── references/
    ├── seo/                     # SEO best practices
    │   ├── SKILL.md
    │   ├── scripts/
    │   └── references/
    ├── best-practices/          # General best practices
    │   ├── SKILL.md
    │   ├── scripts/
    │   └── references/
    └── core-web-vitals/         # CWV-specific optimization
        ├── SKILL.md
        ├── scripts/
        └── references/
```

## Skill format

### SKILL.md structure

```markdown
---
name: skill-name
description: One sentence describing when to use this skill. Include trigger phrases.
license: MIT
metadata:
  author: web-quality-skills
  version: "1.0"
---

# Skill title

Brief description of what the skill does.

## How it works

Numbered steps explaining the process.

## Guidelines

Categorized rules with clear formatting.

## Examples

Practical code examples with before/after.

## References

Links to additional documentation files.
```

### Naming conventions

- **Skill directory:** kebab-case (e.g., `core-web-vitals`, `best-practices`)
- **SKILL.md:** Always uppercase, exact filename
- **Scripts:** kebab-case.sh (e.g., `analyze-performance.sh`)
- **References:** UPPERCASE.md (e.g., `LCP.md`, `WCAG.md`)

### Content guidelines

1. **Keep SKILL.md under 500 lines** — use `references/` for detailed material
2. **Write specific descriptions** — include trigger phrases for skill activation
3. **Use progressive disclosure** — main instructions in SKILL.md, details in references
4. **Prefer scripts for automation** — script output doesn't consume context
5. **Include practical examples** — real code patterns, not just theory

## Writing guidelines

### For guidelines

Each guideline should follow this pattern:

```markdown
* **Guideline title.** Concise explanation of what to do. Include specific values or thresholds when applicable.
```

Example:
```markdown
* **Images have dimensions.** Set explicit `width` and `height` attributes on `<img>` to prevent layout shifts. Use CSS `aspect-ratio` as a fallback.
```

### For code examples

Always show the problem and solution:

```markdown
**Bad:**
```html
<img src="hero.jpg">
```

**Good:**
```html
<img src="hero.jpg" width="1200" height="600" alt="Hero image" loading="lazy">
```
```

### For thresholds

Use tables for clarity:

```markdown
| Metric | Good | Needs improvement | Poor |
|--------|------|-------------------|------|
| LCP | ≤ 2.5s | 2.5s – 4.0s | > 4.0s |
```

## Script requirements

Scripts in `scripts/` should:

1. Use `#!/bin/bash` shebang
2. Use `set -e` for fail-fast behavior
3. Write status messages to stderr: `echo "Message" >&2`
4. Write machine-readable output (JSON) to stdout
5. Include cleanup traps for temp files
6. Be self-contained or clearly document dependencies

Example:
```bash
#!/bin/bash
set -e

cleanup() {
  rm -f "$TEMP_FILE"
}
trap cleanup EXIT

TEMP_FILE=$(mktemp)

echo "Analyzing..." >&2
# ... analysis logic ...

echo '{"score": 85, "issues": []}'
```

## Reference material

Reference files in `references/` should:

1. Focus on a single topic
2. Be loadable independently
3. Include practical examples
4. Link back to authoritative sources

Keep individual reference files under 200 lines when possible.

## Testing skills

Before submitting a skill:

1. Verify YAML frontmatter is valid
2. Check that description includes activation triggers
3. Test that examples actually work
4. Ensure all referenced files exist
5. Validate against the [Agent Skills specification](https://agentskills.io/specification)

Run every skill validator with failure propagation:

```bash
for skill in skills/*/; do npx skills-ref validate "$skill" || exit 1; done
```

## Priority rules

When reviewing code, prioritize by impact:

1. **Critical** — Security vulnerabilities, complete failures
2. **High** — Core Web Vitals failures, major accessibility barriers
3. **Medium** — Performance opportunities, SEO improvements
4. **Low** — Code style, minor optimizations

## Framework agnosticism

Skills must work across frameworks. When providing examples:

- Show vanilla HTML/CSS/JS first
- Add framework-specific notes in separate sections
- Never require a specific framework
- Use standard web APIs when possible

## Updating skills

When updating existing skills:

1. Maintain backward compatibility in descriptions
2. Update version in frontmatter metadata
3. Document intentional behavior changes in the commit message and `CHANGELOG.md`
4. Keep existing trigger phrases working
