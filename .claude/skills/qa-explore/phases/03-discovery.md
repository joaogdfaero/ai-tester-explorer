# Phase 3: Discovery & Mapping (~10% of time)

**Goal:** Map the app structure and refine your risk ranking.

## Site Mapping

```bash
playwright-cli goto <start_url>
playwright-cli snapshot
playwright-cli console error
playwright-cli network
```

For each page:
1. `playwright-cli snapshot` -- note structure, forms, interactive elements
2. Click through navigation to discover all sections
3. Log: `[DISCOVERED] <url> -- <description> -- Risk: <P0/P1/P2/P3>`
4. Check `playwright-cli console error` on each page

## Scope Enforcement

Stay within the target domain. Do not follow links to external sites unless they are part of the application's workflow (e.g., OAuth redirects, payment gateways).

<!-- ## What's MISSING Check

**Critical during discovery -- look for what's MISSING:**
- Read the domain completeness checklist from `data/domains/<domain>.md` (or the corresponding `.yml`)
- Compare what the app HAS vs what it SHOULD have
- E-commerce missing payment method? Log it now.
- Banking app missing transaction history? Flag it immediately.
- No error messages on empty states? Note it. -->

## Console & Network Checks

### Load the known-errors baseline first

Before checking any page, read the target config (`data/targets/<target-id>.yml`) and extract the `known_errors` section if present. This is your noise filter — errors matching these patterns are **expected** and should be logged under a "Baseline / suppressed" bucket, not treated as findings.

If the target has no `known_errors` section (e.g. first session on a new target), treat every error as new.

### Per-page checks

On every page during discovery:
- `playwright-cli console error` — collect all JavaScript errors
- `playwright-cli network` — collect all failed requests (4xx, 5xx, status 0)
- Watch for mixed content warnings (HTTP resources on HTTPS pages)

### Classify each error immediately

For each error or failed request, check it against the `known_errors` baseline:

| Match | Action |
|-------|--------|
| Matches a `known_errors` entry | Log under **"Baseline errors (suppressed)"** — one line, no investigation needed |
| Does NOT match any entry | Log under **"NEW errors"** — flag for investigation in Phase 4 |

Log format:
```
[BASELINE] <error message> — <reason from config>
[NEW ERROR] <error message> — investigate in Phase 4
```

This means a page with 10 CORS analytics errors and 1 genuine JS TypeError should produce:
- 10 lines under Baseline (collapsed/summarized is fine)
- 1 line under NEW ERROR — prominently flagged

### Updating the baseline

If during Phase 4 testing you confirm a NEW ERROR has no customer-facing impact (e.g. it fires on every page, analytics only), add it to the target's `known_errors` in `data/targets/<target-id>.yml` so future sessions don't noise-investigate it again.

Save baseline + new errors summary to session context to check if new errors affect the customer during Phase 4 testing.

### API-level observation helpers (optional, Playwright 1.56+)

If the session has access to the `@playwright/test` peer dep AND is running inside a Playwright test context (rare for qa-explore, common for bug-reproduction follow-ups), the following 1.56 retrieval methods can be used alongside the CLI commands above:

- `await page.consoleMessages()` — returns all console messages captured so far on the page. Useful when you need the full history rather than the most-recent tail.
- `await page.pageErrors()` — returns runtime page errors (uncaught exceptions). More reliable than filtering `console error` output.
- `await page.requests()` — returns all network requests issued by the page, with status, resource type, and timing. Use this for a single network snapshot instead of parsing `playwright-cli network` output twice.

These are OPTIONAL and only apply when exploratory testing is done from a Playwright test harness. The bash `playwright-cli` commands above remain the primary path for the skill.

### Service Workers (Playwright 1.56+)

Playwright 1.56 added Service Worker network routing. If `playwright-cli network` shows requests you cannot account for on the main document, assume a Service Worker is intercepting them and flag this as a high-risk area worth re-testing with the route filter.

## After Discovery

1. Write findings to `phase-1-discovery.md`
2. **Update your risk ranking** if discovery changed your understanding
3. Carry forward: site map summary, risk ranking, list of what's missing

**Log your reasoning:** "I expected a payment method selector in checkout because this is e-commerce, but it's missing. I'll flag this as a medium bug."

**Update progress:** Set discovery phase complete in progress.json with pages_explored count. Append to session-log.md:
`[<timestamp>] [PHASE] Discovery complete — <N> pages mapped, <N> console errors, <N> missing checklist items, <N> bugs so far`
