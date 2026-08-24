# Performance measurement workflow

Use this workflow when a runnable URL is available, the user asks for measured performance, or a change needs before/after verification.

## Keep the evidence types separate

| Evidence | What it represents | Best use |
|----------|--------------------|----------|
| CrUX field data | Aggregated experiences from eligible real Chrome users, normally a rolling 28-day window | Decide whether users have a Core Web Vitals problem |
| First-party RUM | Measurements collected and reported from the site's own user sessions | Segment and diagnose current production experience |
| DevTools performance trace | One observed browser session under stated local or emulated conditions | Find LCP, INP, CLS, network, and main-thread causes |
| Lighthouse lab run | A controlled synthetic navigation | Reproduce load problems and prevent regressions |
| Static code inspection | Potential issues inferred from source | Form hypotheses when no page can run |

A `PerformanceObserver` result injected into one browser page is a **single-session lab observation**, not field data. It becomes RUM only when measurements from actual users are reported and aggregated.

## Preferred low-friction route

When browser tooling can record a performance trace and run Lighthouse audits, prefer this route. With Chrome DevTools MCP:

1. Navigate to the exact route and state being audited. Record whether it is public, authenticated, local, or staging.
2. Record a reload trace with auto-stop for page-load performance (`performance_start_trace`). Current trace summaries can include both observed lab metrics and CrUX field metrics when CrUX has eligible data. Record whether field scope is URL or origin.
3. Analyze only the relevant failing or suspicious insights (`performance_analyze_insight`). Common examples are `LCPBreakdown`, `LCPDiscovery`, `DocumentLatency`, `RenderBlocking`, and `ThirdParties`.
4. Run the Lighthouse audit capability (`lighthouse_audit`) for Accessibility, SEO, Best Practices, and Agentic Browsing. It deliberately excludes performance; do not treat it as the performance path.
5. Re-run the same lab measurement after a fix. Field data will not reflect a new deployment immediately.

Use mobile conditions by default for a general public-site audit. Add desktop when the user asks for it, desktop traffic matters, or the product is desktop-oriented. Test authenticated and unauthenticated states separately when they render different pages.

When using emulation, set the viewport, network conditions, and CPU rate explicitly before the trace, confirm the reported conditions, and reset them before testing another profile.

Chrome DevTools MCP's Lighthouse navigation mode reloads the page. Use snapshot mode for the current state when a reload would lose an authenticated or user-created state. Do not run a navigation audit on an unsaved form or destructive workflow.

### Token-efficient tool use

* Start with one trace and one Lighthouse audit rather than broad DOM, network, console, and source dumps.
* Preserve large reports or traces to temporary files when the tool supports `filePath` or `outputDirPath`; summarize only actionable failures.
* Drill into the few insights tied to a poor field metric or a reproducible lab bottleneck.
* Filter and paginate network or console requests. Fetch individual request details only when they support a finding.
* Take a text snapshot before a screenshot unless visual inspection is necessary.

## Fallbacks when DevTools tools are unavailable

Use the first available option; do not block the audit on optional setup.

1. **Lighthouse CLI for lab data:** run the project's compatible Lighthouse version against the runnable URL. Keep JSON for comparison and avoid installing a permanent dependency unless the user wants one.
2. **PageSpeed Insights web UI for a public URL:** it provides a zero-setup view of Lighthouse lab diagnostics and available CrUX field data.
3. **CrUX Vis for history:** use it when trend data matters and the URL or origin is eligible.
4. **CrUX API or History API for automation:** both are free to use but require a Google Cloud API key. Do not make a key a prerequisite for an ordinary audit.
5. **Static inspection:** if nothing can run, label every performance finding as a hypothesis and provide the exact measurement needed to verify it.

The PageSpeed Insights API may be called without a key for occasional use, but a key is recommended for repeated automation. Google has announced that CrUX field data will be removed from that API, so new integrations should query the CrUX API directly.

## Reading CrUX correctly

* Prefer page-level data for the audited URL. If only origin data exists, label it as origin scope; it is context, not proof for that route.
* Compare the p75 value with the Core Web Vitals threshold and include the percentage of good experiences when available.
* Keep phone and desktop data separate. Do not combine form factors to answer a device-specific question.
* Treat missing CrUX data as **unavailable**, never as passing. Localhost, staging, new, private, and low-traffic pages commonly have no CrUX record.
* CrUX is aggregated and delayed. Use it to prioritize user outcomes, not to verify a change deployed minutes ago.

## Repeatable lab comparisons

Record these conditions with the result:

* final URL and page state
* browser and Lighthouse/tool version
* viewport or form factor
* CPU and network throttling
* cold or warm cache
* authentication, consent, and experiment state

For a decision based on a headline lab metric, run at least three equivalent navigations and report the median plus range. Do not compare a single local trace directly with the CrUX p75 or claim that the two should match.

Use metric values as the evidence. A Lighthouse score is a diagnostic summary whose weighting and implementation can change between versions.

## Reconciling lab and field

| Field | Lab | Interpretation |
|-------|-----|----------------|
| Poor | Poor | Reproducible user problem; trace and fix the dominant bottleneck |
| Poor | Good | Local run missed real-user conditions; segment first-party RUM or test representative devices, routes, cache states, and interactions |
| Good | Poor | The synthetic cold/throttled case is fragile, but do not claim users are currently failing |
| Unavailable | Any | Use lab data for diagnosis and recommend RUM if production impact matters |

## Compact audit output

Start reports with an evidence table:

| Signal | Scope and conditions | Baseline | After | Source |
|--------|----------------------|----------|-------|--------|
| LCP | URL, phone, p75/28 days | 3.1s | Pending field window | CrUX |
| LCP | URL, mobile lab, cold cache, median of 3 | 3.8s | 2.6s | DevTools trace |

Then separate:

1. measured failures
2. trace-backed causes
3. source-code hypotheses
4. fixes made or recommended
5. verification status and remaining uncertainty

## Sources

* [Chrome DevTools MCP tool reference](https://github.com/ChromeDevTools/chrome-devtools-mcp/blob/main/docs/tool-reference.md)
* [Chrome UX Report API](https://developer.chrome.com/docs/crux/api)
* [Getting started with measuring Web Vitals](https://web.dev/articles/vitals-measurement-getting-started)
* [PageSpeed Insights API](https://developers.google.com/speed/docs/insights/v5/get-started)
