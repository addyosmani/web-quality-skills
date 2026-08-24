# CLAUDE.md

This file provides guidance to Claude Code when working with this repository.

## Project summary

Web Quality Skills is a measurement-first collection of Agent Skills for Google Lighthouse, Chrome DevTools, Core Web Vitals, accessibility, SEO, best practices, and agentic browsing. The skills are framework-agnostic and follow the [Agent Skills specification](https://agentskills.io/specification).

## Quick reference

### Available skills

| Skill | Location | Triggers |
|-------|----------|----------|
| web-quality-audit | `skills/web-quality-audit/` | "audit", "quality review", "lighthouse" |
| performance | `skills/performance/` | "speed up", "optimize", "load time" |
| core-web-vitals | `skills/core-web-vitals/` | "LCP", "INP", "CLS", "Core Web Vitals" |
| accessibility | `skills/accessibility/` | "a11y", "WCAG", "accessible" |
| seo | `skills/seo/` | "SEO", "meta tags", "search" |
| best-practices | `skills/best-practices/` | "security", "best practices", "modern" |

### Key thresholds

**Core Web Vitals (Good):**
- LCP ≤ 2.5s
- INP ≤ 200ms  
- CLS ≤ 0.1

Apply these thresholds to field data at the 75th percentile. A single trace or `PerformanceObserver` result is a lab observation, not real-user data.

**Performance Budgets:**
- Total: < 1.5 MB
- JS: < 300 KB
- CSS: < 100 KB

**Lighthouse Targets:**
- Performance: ≥ 90
- Accessibility: aim for 100 automated coverage, then complete manual checks
- Best Practices: ≥ 95
- SEO: ≥ 95

Scores are regression guardrails, not proof of accessibility, security, rankings, or user experience.

### Live audit routing

- Prefer capabilities over provider-specific tool names. Record a browser performance trace and analyze focused insights; with Chrome DevTools MCP, use `performance_start_trace` and `performance_analyze_insight`. Current traces may include CrUX field context.
- Run live Lighthouse audits for Accessibility, SEO, Best Practices, and Agentic Browsing; with Chrome DevTools MCP, use `lighthouse_audit`. It intentionally excludes performance.
- Read `skills/performance/references/MEASUREMENT.md` for field/lab fallbacks and repeatable comparisons.
- When no page can run, label performance findings as source hypotheses and provide a verification workflow.

## Common tasks

### Adding a new skill

1. Create directory: `skills/{skill-name}/`
2. Create `SKILL.md` with YAML frontmatter
3. Add optional `scripts/`, `references/`, `assets/`
4. Update `README.md` skills table

### Updating guidelines

1. Edit relevant `SKILL.md` file
2. Keep under 500 lines
3. Move detailed content to `references/`
4. Update version in frontmatter

### Running tests

```bash
# Validate skill format
for skill in skills/*/; do npx skills-ref validate "$skill" || exit 1; done

# Lint markdown
npx markdownlint skills/**/*.md
```

## Code style

- Use kebab-case for directories and files
- YAML frontmatter required on all SKILL.md files
- Markdown follows standard formatting
- Code examples should show ❌ bad and ✅ good patterns
- Include specific values/thresholds where applicable

## Dependencies

This project has no runtime dependencies. Skills are pure markdown with optional shell scripts.

For development:
- Node.js 18+ (for validation tools)
- skills-ref (optional, for validation)
