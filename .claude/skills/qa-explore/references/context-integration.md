# Context Integration

How the `--context` parameter changes the session.

## Mode 1: Context File (`--context <file>`)

Read the file. It may be a `tool-qa-workflow` output (structured markdown with Jira data, Confluence specs, GitLab MR diffs, Figma design notes), a known-bugs spreadsheet, or any document describing what to test.

Extract and use:

| Context Element | How It Changes the Session |
|----------------|---------------------------|
| **Acceptance criteria** | Add as explicit checkpoints -- verify each AC during testing |
| **What changed** (MR diff, commits) | Focus testing on changed areas -- highest regression risk |
| **Design specs** (Figma, mockups) | Compare live app against intended design -- spot deviations |
| **User stories / requirements** | Test against INTENT, not just what's visible |
| **Known risks** (from risk analysis) | Prioritize testing on flagged risk areas |
| **API contracts** (OpenAPI, endpoints) | Verify UI matches API behavior |
| **Previous test results** | Don't re-test what passed -- focus on gaps |
| **Known bugs list** (XLSX/CSV/MD) | Derive adjacent hypotheses — see below |

### Known-Bugs Spreadsheet (XLSX / CSV / Markdown table)

When the context file is a bug list (any format — XLSX, CSV, markdown table, plain text), treat it as a **hypothesis generator**, not a re-test list.

**Reading the file:** Claude can read XLSX files natively. Pass the path directly:
```
/qa-explore https://www.audible.com.br/ --context path/to/known-issues.xlsx
```

**What to extract from each row:**
- Bug ID / title
- Affected URL or page
- Bug type (localization, JS error, broken link, layout, data mismatch, etc.)
- Component or area name (JS asset, UI widget, page section)
- Status (open / fixed / wontfix) — skip wontfix entries

**Generating adjacent hypotheses — the core value:**

For each bug, ask: *"If this broke here, what else nearby could be broken?"*

| Bug type | Adjacent hypothesis to test |
|----------|----------------------------|
| `localization` | Check `<title>`, `<h1>`, visible strings on every other in-scope page for wrong-language content |
| `javascript_error` | Does the same JS asset/component load on other pages? Run console check there too |
| `broken_link` | Spot-check other links in the same navigation area or section |
| `layout` | Check same layout at different viewports and browsers |
| `data_mismatch` | Find every place the same data point appears and compare for consistency |
| `missing_feature` | Look for the same gap on similar pages (e.g., missing button on page A → check page B) |

**Output — add to the charter as a "Known-Bug Hypotheses" section:**
```markdown
## Known-Bug Hypotheses (from context file)
| Source Bug | Hypothesis | Where to test | Priority |
|------------|-----------|---------------|----------|
| BUG-001 (Spanish title on /library) | Other pages may also have wrong-language <title> | /library/wishlist, PDPs, /newreleases, /charts | P1 |
| BUG-002 (JS TypeError in AudibleLibraryAssets on /wishlist) | Same asset loads on /library/titles — may have same error there | /library/titles | P1 |
```

**Do NOT spend time re-verifying the exact known bug unless:** (a) its status is "fixed" — confirm it's actually fixed, or (b) the hypothesis test naturally re-enters the same page.

## Mode 2: Inline Context

Extract from the user's message any descriptions of: what the feature does, acceptance criteria, what changed, what the design should look like, known risks, user personas.

## Mode 3: Blind Exploration (no context)

That's fine -- run in blind exploration mode. The agent discovers everything on its own. **Context makes exploration sharper but NEVER blocks it.** A QA walking up to a random app with zero documentation should still get a full, valuable session. Context is a boost, not a gate.

## Charter Annotation

When context is loaded, note in the charter:
```
CONTEXT SOURCE: [filename or "inline" or "none -- blind exploration"]
ACCEPTANCE CRITERIA: [list extracted ACs or "none provided -- discovering from app"]
CHANGE SCOPE: [what changed or "unknown -- full exploration"]
```

### Context-Driven Adjustments

**If context was provided**, during the charter phase also check:
- Do the acceptance criteria match what you see in the app?
- Does the UI match the design specs?
- Are the features described in the requirements actually present?
- What does the context say the RISKS are? Do you agree after seeing the app?
