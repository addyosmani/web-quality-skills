# Web Quality Skills

An (unofficial) measurement-first collection of [Agent Skills](https://agentskills.io/) for optimizing web projects with [Google Lighthouse](https://developer.chrome.com/docs/lighthouse/overview/), Chrome DevTools for agents, Core Web Vitals, WCAG, and search guidance.

**Stack-agnostic.** Works with any framework: React, Vue, Angular, Svelte, Next.js, Nuxt, Astro, plain HTML, and more.

## Why web quality skills?

While interface guidelines tell you *what* to build, Web Quality Skills tell you *how* to build it performantly, accessibly, and optimally for search engines. These skills combine:

- **Live Lighthouse audits for agents** across Accessibility, SEO, Best Practices, and Agentic Browsing when Chrome DevTools MCP is available
- **Performance traces and Insights** for diagnosing LCP, INP, CLS, network, and main-thread bottlenecks
- **Field plus lab measurement** using CrUX for real-user context and controlled browser runs for repeatable diagnosis
- **Core Web Vitals** optimization patterns for LCP, INP, and CLS
- **WCAG 2.2** accessibility standards
- **Modern SEO** requirements

## Available skills

| Skill | Description | Use when |
|-------|-------------|----------|
| **[web-quality-audit](#web-quality-audit)** | Evidence-led review across Lighthouse and source categories | "Audit my site", "Review this for quality", "Check web quality" |
| **[performance](#performance)** | Field/lab measurement, traces, loading and runtime fixes | "Optimize performance", "Speed up my site", "Fix slow loading" |
| **[core-web-vitals](#core-web-vitals)** | Measured LCP, INP, and CLS diagnosis and optimization | "Improve Core Web Vitals", "Fix LCP", "Reduce CLS" |
| **[accessibility](#accessibility)** | WCAG compliance, screen reader support, keyboard navigation | "Improve accessibility", "WCAG audit", "a11y review" |
| **[seo](#seo)** | Search engine optimization, crawlability, structured data | "Optimize for SEO", "Improve search ranking", "Fix meta tags" |
| **[best-practices](#best-practices)** | Security, modern APIs, code quality patterns | "Apply best practices", "Security audit", "Code quality review" |

## Measurement model

The performance skills keep four evidence types explicit:

| Evidence | Answers |
|----------|---------|
| CrUX field data | How eligible real Chrome users experienced the URL or origin over the recent aggregation window |
| First-party RUM | How the site's own users experience routes, releases, devices, and interactions |
| DevTools trace or Lighthouse lab run | What happened in one controlled browser session and why |
| Static source inspection | What might be a problem when the page cannot run |

Field data determines user impact. Lab traces reproduce and diagnose it. A `PerformanceObserver` snippet run in one browser is a lab observation, not real-user data by itself.

## Quick start

### Installation

add-skill is a powerful CLI tool that lets you install agent skills onto your coding agents from git repositories. Whether you're using OpenCode, Claude Code, Codex, or Cursor, the add-skill tool makes it simple to extend your agent's capabilities with specialized instruction sets. Use add-skill to automate release notes, create pull requests, integrate with external tools, and more. Simply run npx add-skill to get started.

```bash
npx skills add addyosmani/web-quality-skills
```

or

```
npx add-skill addyosmani/web-quality-skills
```

Or manually:

```bash
cp -r skills/* ~/.claude/skills/
```

#### Claude Code (plugin)

Install as a versioned, namespaced plugin from inside Claude Code:

```text
/plugin marketplace add addyosmani/web-quality-skills
/plugin install web-quality-skills@addy-web-quality-skills
```

Skills are then namespaced (e.g. `/web-quality-skills:performance`) and update with `/plugin update`. The plugin reads the same `skills/` directory as the manual copy above — no duplication.

#### Codex

Install directly via the Codex plugin marketplace (Codex CLI v0.122+):

```bash
codex plugin marketplace add addyosmani/web-quality-skills
```

Once installed, invoke skills in chat using `@` (e.g. `@performance`, `@accessibility`). See [docs/codex-setup.md](docs/codex-setup.md) for local installation and troubleshooting.

#### Gemini CLI

Install directly via Gemini CLI extensions:

```bash
gemini extensions install https://github.com/addyosmani/web-quality-skills
```

Skills are auto-discovered by Gemini and activate when prompts match their description. See [docs/gemini-setup.md](docs/gemini-setup.md) for workspace mode and troubleshooting.

#### claude.ai

Add skills to your project knowledge or paste the SKILL.md contents into your conversation.

### Usage

Skills activate automatically when your request matches their description. Examples:

```
Audit this page for web quality issues
```

```
Optimize performance and fix Core Web Vitals
```

```
Review accessibility and suggest improvements
```

```
Make this SEO-ready
```

### Optional Chrome DevTools integration

When the agent exposes equivalent live-browser capabilities, the skills prefer them. With [Chrome DevTools MCP](https://github.com/ChromeDevTools/chrome-devtools-mcp), the current tool names are:

- `performance_start_trace` and focused Performance Insights for performance work, including CrUX context when eligible field data exists
- `lighthouse_audit` for Accessibility, SEO, Best Practices, and Agentic Browsing
- rendered accessibility snapshots, console messages, and network requests for issue-level diagnosis

This integration is optional. Without it, the skills fall back to Lighthouse CLI, PageSpeed Insights/CrUX tools, manual browser checks, and static inspection rather than stopping the task.

## Skill details

### web-quality-audit

The comprehensive skill that orchestrates all other skills. It starts with live evidence when a page is runnable and keeps measured failures separate from source-code hypotheses.

**Trigger phrases:** "audit my site", "quality review", "lighthouse audit", "check web quality"

**What it checks:**
- All Core Web Vitals metrics
- Performance Insights, network delivery, and runtime bottlenecks
- Automated and manual accessibility signals
- Crawl, index, metadata, and structured-data requirements
- Security, compatibility, and browser best-practice patterns
- Agentic Browsing signals such as agent-facing semantics and optional WebMCP/`llms.txt` checks

### performance

Deep-dive into loading and runtime performance optimization, using CrUX or first-party RUM to prioritize user impact and DevTools traces to diagnose causes.

**Trigger phrases:** "speed up", "optimize performance", "reduce load time", "fix slow"

**Key optimizations:**
- Critical rendering path
- JavaScript bundling and code splitting
- Image optimization (formats, sizing, lazy loading)
- Font loading strategies
- Caching and preloading
- Server response optimization

### core-web-vitals

Specialized skill for the three Core Web Vitals that describe loading, responsiveness, and visual stability. Google uses Core Web Vitals in its page-experience systems, but a passing score does not promise a ranking change.

**Trigger phrases:** "Core Web Vitals", "LCP", "INP", "CLS", "page experience"

**Metrics covered:**
- **LCP** (Largest Contentful Paint) < 2.5s
- **INP** (Interaction to Next Paint) < 200ms
- **CLS** (Cumulative Layout Shift) < 0.1

### accessibility

Comprehensive accessibility audit following WCAG 2.2 guidelines.

**Trigger phrases:** "accessibility", "a11y", "WCAG", "screen reader", "keyboard navigation"

**Categories:**
- Perceivable (text alternatives, captions, contrast)
- Operable (keyboard, timing, seizures, navigation)
- Understandable (readable, predictable, input assistance)
- Robust (compatible with assistive technologies)

### seo

Search engine optimization for better visibility and ranking.

**Trigger phrases:** "SEO", "search optimization", "meta tags", "structured data", "sitemap"

**What it covers:**
- Technical SEO (crawlability, indexability)
- On-page SEO (meta tags, headings, content structure)
- Structured data (JSON-LD, schema.org)
- Mobile-friendliness
- Performance signals

### best-practices

Modern web development standards and security practices.

**Trigger phrases:** "best practices", "security audit", "modern standards", "code quality"

**Areas covered:**
- HTTPS and security headers
- Modern JavaScript APIs
- Browser compatibility
- Error handling
- Console cleanliness

## Thresholds reference

### Core Web Vitals

| Metric | Good | Needs improvement | Poor |
|--------|------|-------------------|------|
| LCP | ≤ 2.5s | 2.5s – 4.0s | > 4.0s |
| INP | ≤ 200ms | 200ms – 500ms | > 500ms |
| CLS | ≤ 0.1 | 0.1 – 0.25 | > 0.25 |

Field Core Web Vitals are assessed at the 75th percentile; these thresholds are not a claim about a single lab run.

### Starting performance budget recommendations

These are initial guardrails, not universal thresholds. Set budgets from the product's target devices, networks, page types, and user journeys.

| Resource type | Budget |
|---------------|--------|
| Total page weight | < 1.5 MB |
| JavaScript | < 300 KB (compressed) |
| CSS | < 100 KB (compressed) |
| Images | < 500 KB total above-fold |
| Fonts | < 100 KB |
| Third-party | < 200 KB |

### Example automated audit guardrails

Scores help catch regressions but do not prove accessibility, security, SEO ranking, or user experience. Preserve project-specific targets and compare runs made with the same tool version and conditions.

| Category | Example guardrail |
|----------|-------------------|
| Performance | ≥ 90 |
| Accessibility | Aim for 100 automated coverage; complete manual checks |
| Best Practices | ≥ 95 |
| SEO | ≥ 95 |

## Framework-specific notes

These skills are framework-agnostic, but some common patterns:

**React/Next.js:** Use `next/image`, `React.lazy()`, `Suspense`, `useCallback`/`useMemo` for INP  
**Vue/Nuxt:** Use `nuxt/image`, async components, `v-once`, computed properties  
**Svelte/SvelteKit:** Use `{#await}`, `svelte:image`, reactive statements  
**Astro:** Use `<Image>`, partial hydration, view transitions  
**Static HTML:** Use native lazy loading, `<picture>`, preconnect hints

## Contributing

Contributions welcome! Please follow the [Agent Skills specification](https://agentskills.io/specification).

1. Fork the repository
2. Create your skill in `skills/{skill-name}/SKILL.md`
3. Keep SKILL.md under 500 lines (use `references/` for details)
4. Include practical examples and patterns
5. Submit a pull request

## Resources

- [Google Lighthouse Documentation](https://developer.chrome.com/docs/lighthouse/)
- [Lighthouse audits with AI agents](https://developer.chrome.com/docs/devtools/agents/use-cases/lighthouse-audit)
- [Chrome DevTools MCP](https://github.com/ChromeDevTools/chrome-devtools-mcp)
- [Chrome UX Report](https://developer.chrome.com/docs/crux/)
- [web.dev Learn Performance](https://web.dev/learn/performance/)
- [Core Web Vitals](https://web.dev/articles/vitals)
- [WCAG 2.2 Guidelines](https://www.w3.org/WAI/WCAG22/quickref/)
- [Agent Skills Specification](https://agentskills.io/specification)

## License

MIT License - see [LICENSE](LICENSE) for details.

---

Built with insights from the Chrome DevTools team, web performance experts, and accessibility advocates to help developers create high-quality web experiences.
