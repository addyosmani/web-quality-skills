# Web Quality Skills

An (unofficial) comprehensive collection of [Agent Skills](https://agentskills.io/) for optimizing web projects based on [Google Lighthouse](https://developer.chrome.com/docs/lighthouse/overview/) guidelines and Core Web Vitals best practices.

**Stack-agnostic.** Works with any framework: React, Vue, Angular, Svelte, Next.js, Nuxt, Astro, plain HTML, and more.

## Why web quality skills?

While interface guidelines tell you *what* to build, Web Quality Skills tell you *how* to build it performantly, accessibly, and optimally for search engines. These skills encode the collective wisdom from:

- **150+ Lighthouse audits** across Performance, Accessibility, SEO, and Best Practices
- **Core Web Vitals** optimization patterns (LCP, INP, CLS)
- **Real-world performance engineering** experience
- **WCAG 2.1** accessibility standards
- **Modern SEO** requirements

## Available skills

| Skill | Description | Use when |
|-------|-------------|----------|
| **[web-quality-audit](#web-quality-audit)** | Comprehensive quality review across all categories | "Audit my site", "Review this for quality", "Check web quality" |
| **[performance](#performance)** | Loading speed, runtime efficiency, resource optimization | "Optimize performance", "Speed up my site", "Fix slow loading" |
| **[core-web-vitals](#core-web-vitals)** | LCP, INP, CLS specific optimizations | "Improve Core Web Vitals", "Fix LCP", "Reduce CLS" |
| **[accessibility](#accessibility)** | WCAG compliance, screen reader support, keyboard navigation | "Improve accessibility", "WCAG audit", "a11y review" |
| **[seo](#seo)** | Search engine optimization, crawlability, structured data | "Optimize for SEO", "Improve search ranking", "Fix meta tags" |
| **[best-practices](#best-practices)** | Security, modern APIs, code quality patterns | "Apply best practices", "Security audit", "Code quality review" |

## Quick start

### Installation

add-skill is a powerful CLI tool that lets you install agent skills onto your coding agents from git repositories. Whether you're using OpenCode, Claude Code, Codex, or Cursor, the add-skill tool makes it simple to extend your agent's capabilities with specialized instruction sets. Use add-skill to automate release notes, create pull requests, integrate with external tools, and more. Simply run npx add-skill to get started.

```bash
npx skills add addyosmani/web-quality-skills
```

or use skillkit to install skills across multiple AI agents:

```bash
# Install all skills
npx skillkit install addyosmani/web-quality-skills

# Install specific skills
npx skillkit install addyosmani/web-quality-skills --skill accessibility

# List available skills without installing
npx skillkit install addyosmani/web-quality-skills --list
```

Once installed, translate skills to any supported agent:

```bash
npx skillkit agent translate --from claude-code --to cursor
```

or

```
npx add-skill addyosmani/web-quality-skills
```

Or manually:

```bash
cp -r skills/* ~/.claude/skills/
```

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

## Skill details

### web-quality-audit

The comprehensive skill that orchestrates all other skills. Use this for full-site audits or when you're unsure which specific area needs attention.

**Trigger phrases:** "audit my site", "quality review", "lighthouse audit", "check web quality"

**What it checks:**
- All Core Web Vitals metrics
- 50+ performance patterns
- 40+ accessibility rules
- 30+ SEO requirements
- 20+ security/best practice patterns

### performance

Deep-dive into loading and runtime performance optimization.

**Trigger phrases:** "speed up", "optimize performance", "reduce load time", "fix slow"

**Key optimizations:**
- Critical rendering path
- JavaScript bundling and code splitting
- Image optimization (formats, sizing, lazy loading)
- Font loading strategies
- Caching and preloading
- Server response optimization

### core-web-vitals

Specialized skill for the three Core Web Vitals that affect Google Search ranking.

**Trigger phrases:** "Core Web Vitals", "LCP", "INP", "CLS", "page experience"

**Metrics covered:**
- **LCP** (Largest Contentful Paint) < 2.5s
- **INP** (Interaction to Next Paint) < 200ms
- **CLS** (Cumulative Layout Shift) < 0.1

### accessibility

Comprehensive accessibility audit following WCAG 2.1 guidelines.

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

### Performance budget recommendations

| Resource type | Budget |
|---------------|--------|
| Total page weight | < 1.5 MB |
| JavaScript | < 300 KB (compressed) |
| CSS | < 100 KB (compressed) |
| Images | < 500 KB total above-fold |
| Fonts | < 100 KB |
| Third-party | < 200 KB |

### Lighthouse score targets

| Category | Target score |
|----------|--------------|
| Performance | ≥ 90 |
| Accessibility | 100 |
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
- [web.dev Learn Performance](https://web.dev/learn/performance/)
- [Core Web Vitals](https://web.dev/articles/vitals)
- [WCAG 2.1 Guidelines](https://www.w3.org/WAI/WCAG21/quickref/)
- [Agent Skills Specification](https://agentskills.io/specification)

## License

MIT License - see [LICENSE](LICENSE) for details.

---

Built with insights from the Chrome DevTools team, web performance experts, and accessibility advocates to help developers create high-quality web experiences.
