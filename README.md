# ai-tester-explorer

An AI-driven exploratory testing framework that turns any capable AI agent into a **Principal QA Engineer**. It combines a structured skill system with Playwright browser automation to run real, risk-proportional exploratory sessions against web applications — discovering bugs, writing professional bug reports, and learning from feedback over time.

> **Works with any AI agent that supports the Skills system** (Claude Code, and compatible runtimes). It is not tied to a specific model or provider.

---

## What it does

- Runs full **45-minute exploratory testing sessions** against any web application
- Navigates, interacts, and inspects the app using a real browser (Playwright)
- Risk-ranks features and allocates testing time proportionally (P0 → P3)
- Writes structured **bug reports** with business impact, reproduction steps, and screenshots
- Maintains a **knowledge base** of heuristics, techniques, and learned patterns that improve over time
- Supports **requirements gathering** to focus sessions on what actually matters
- Generates **session reports** and **coverage maps** after each run
- Optionally converts reproducible bugs into **Playwright regression specs**

---

## Usage

### Prerequisites

```bash
npm install
npx playwright install chromium
```

### Typical workflow

---

**1. Gather requirements** (OPTIONAL)

```
/qa-gather path/to/spec.md
/qa-gather https://jira.company.com/browse/PROJ-123
```

Produces a structured context file at `output/context/<feature>-context.md` with extracted acceptance criteria, risk areas, and derived test scenarios.

---

**2. Run an exploratory session**

```
# Blind exploration — no prior context
/qa-explore https://example.com

# With requirements context
/qa-explore https://example.com --context output/context/checkout-context.md

# Using a pre-configured target
/qa-explore --target saucedemo --context output/context/checkout-context.md

# With inline context (type requirements after the command)
/qa-explore https://example.com
> "New checkout flow — supports Visa and Mastercard, requires delivery address"
```

The session runs in phases (setup → auth → charter → discovery → testing → edge cases → reporting). After each phase, findings are written to disk. At the end, bug reports and a session report are generated automatically.

You can also invoke a session with a plain natural-language prompt directly to the AI agent, without using the slash command. This is useful for adding specific constraints, credentials, or scope inline:

```
Execute a testing exploratory session following qa-explore skill also using playwright-cli skill.

Test the audible.br website:
https://www.audible.com.br/

Use credentials inside .env file to login

Don't make any purchases, payments or changes to the account

Test inside the scope contained in "data/scope"

Look for issues that really impact the costumer

Work for 30 minutes on the exploratory session
```

---

**3. View the results**

```
# Summary of the latest session
/qa-explore-report --format summary

# Full report
/qa-explore-report

# Bugs only
/qa-explore-report --format bugs-only

# Specific session
/qa-explore-report 2026-06-10-saucedemo --format summary
```

Individual bug reports are at `output/sessions/<session>/bugs/BUG-NNN.md`. The session index is at `output/sessions/INDEX.md`.

---

**4. Quick check on a single page**

```
/qa-explore-quick https://example.com/checkout
/qa-explore-quick https://example.com/form --focus "validation"
```

---

**5. Capture feedback to improve future sessions**

```
/qa-explore-feedback
```

Reviews each bug (valid / false positive / severity), asks for missed bugs, and updates `data/knowledge/learned-patterns.md`. Patterns learned here are applied in future sessions.

---

### Knowledge base management

```bash
# List all entries
/qa-knowledge-list

# Filter by domain or type
/qa-knowledge-list --domain ecommerce
/qa-knowledge-list --type heuristic
/qa-knowledge-list --tag security

# Show a specific entry
/qa-knowledge-list --entry heuristic-sfdipot

# Add new knowledge (text, URL, or file)
/qa-knowledge-add
```

### Session housekeeping

```bash
/qa-explore-cleanup --list
/qa-explore-cleanup --archive 2026-06-10-saucedemo
/qa-explore-cleanup --delete-old 30
```


---

## How it works

### Skills

The framework is built as a collection of **skills** — structured instruction sets that an AI agent loads and executes. Each skill lives under `.claude/skills/` and defines a complete workflow with phases, rules, references, and output formats.

| Skill | Purpose |
|---|---|
| `/qa-explore` | Full exploratory session (~45 min). The core skill. |
| `/qa-explore-quick` | Fast focused check on a single page (~15 min). |
| `/qa-gather` | Analyze requirements and produce a context file for sessions. |
| `/qa-target-setup` | Interactive wizard to configure a new test target. |
| `/qa-explore-report` | Generate or reformat reports from existing sessions. |
| `/qa-explore-feedback` | Capture post-session feedback to improve future runs. |
| `/qa-explore-cleanup` | List, archive, or delete old sessions. |
| `/qa-knowledge-add` | Add heuristics, techniques, or patterns to the knowledge base. |
| `/qa-knowledge-list` | Browse and search the knowledge base. |

Skills are loaded by the AI agent at runtime. They are plain markdown files — no runtime compilation, no proprietary format.

### Browser automation: `playwright-cli`

All browser interactions happen through the `playwright-cli` skill, which wraps Playwright's CLI to give the AI agent a controlled interface for:

- Navigating pages and clicking elements
- Filling forms and submitting data
- Taking snapshots (page state + element refs)
- Capturing console errors and network requests
- Saving and restoring authentication state
- Recording video and traces for evidence

The AI uses element references from snapshots (e.g. `e15`) to interact with the page without relying on fragile CSS selectors.

**Playwright CLI documentation:** https://playwright.dev/docs/cli

### Knowledge base

Testing heuristics (SFDIPOT, FEW HICCUPPS, Test Tours, Boundary Value Analysis, etc.) are stored as YAML files under `data/knowledge/`. The AI loads relevant heuristics at the start of each session. Patterns learned from feedback are stored in `data/knowledge/learned-patterns.md` and applied in subsequent sessions.

### Output

Every session produces structured output under `output/sessions/<date>-<target>/`:

```
output/sessions/2026-06-10-saucedemo-001/
├── charter.md              # Session scope and risk ranking
├── phase-1-discovery.md    # Application mapping findings
├── phase-2-functional.md   # Feature testing findings
├── phase-3-edge-cases.md   # Edge case and boundary findings
├── session-report.md       # Full session report
├── coverage-map.md         # Feature coverage tracker
├── bugs/
│   ├── BUG-001.md
│   ├── BUG-002.md
│   └── ...
└── screenshots/
    └── bug-001-*.png
```

---

## Pre-configured targets

The `data/targets/` directory includes ready-to-use configs for popular testing practice sites:

| Target | Site |
|---|---|
| `saucedemo` | https://www.saucedemo.com — e-commerce practice site |
| `parabank` | ParaBank — banking application |
| `demoqa` | https://demoqa.com — QA practice site |
| `ultimateqa` | UltimateQA automation practice |
| `thinking-tester` | Thinking Tester practice site |
| `webdriveruniversity` | WebDriver University |
| `compendiumdev` | Alan Richardson's web testing practice |
| `gh-users-search` | GitHub API wrapper demo |
| `testers-ai` | Testing community site |

To add your own target, run `/qa-target-setup` or copy `data/targets/_default.yml`.

---

## Project structure

```
.claude/skills/          # Skill definitions (AI instruction sets)
data/
  knowledge/             # Heuristics, techniques, learned patterns
  domains/               # Domain-specific testing checklists
  targets/               # Target application configs
  templates/             # Bug report, charter, coverage templates
  security/              # Security and safety policies
output/
  sessions/              # Session results (bugs, reports, screenshots)
  bugs/                  # Aggregated bug list across sessions
  context/               # Requirements context from /qa-gather
tests/                   # Playwright regression specs
playwright.config.ts
package.json
```

---

## Compatibility

The skills in this repository follow a plain-markdown format with no runtime dependencies on a specific AI provider. They work with **any AI agent runtime that supports the Skills system** — Claude Code is the reference implementation, but the same skills can be loaded by other compatible agents.

The only hard dependency is **Playwright** for browser automation, which requires Node.js.

---

## Credits

This project is based on [qualiow-exploratory-testing-skills](https://github.com/willcoliveira/qualiow-exploratory-testing-skills) by [Will Oliveira](https://github.com/willcoliveira), which established the foundational skill architecture, session workflow, and testing philosophy used here. The main change was simplifying the skills architecture, alongside additions such as a knowledge base system, target management, and deeper Playwright integration.

---

## License

This project is licensed under the **MIT License** — see [LICENSE](LICENSE) for details.

The original work ([qualiow-exploratory-testing-skills](https://github.com/willcoliveira/qualiow-exploratory-testing-skills)) is also MIT licensed. Its copyright notice is preserved in [LICENSE](LICENSE) as required by the license terms.
