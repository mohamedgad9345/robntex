# RobNTex — agentic ROBIN test-case execution

**RobNTex** (ROBIN eXecution) takes the manual grind out of **executing test cases** against
the ROBIN Revenue Cycle Management platform. You keep authoring TCs the way you already do
(Markdown or Excel); RobNTex reads them, drives the ROBIN portal in a real browser, captures
**per-step evidence**, and produces a **Jira-ready defect report** linked back to your TC-IDs.

Sequential (human-in-the-loop) by default; parallel autonomous mode for regression sweeps.
The agent **never modifies your application code** and **never invents steps** — it runs the
TC exactly as written.

### What works today (Level 1)

| Capability | Status |
|-----------|--------|
| **Markdown TCs** — one TC per `.md` file | ✅ |
| **Excel TCs** — one workbook, many TCs across sheets | ✅ |
| **Browser execution** via `@playwright/cli` on the ROBIN portal | ✅ |
| **Per-step evidence** — screenshot + PASS/FAIL for every step | ✅ |
| **Jira-ready `bug-list.md`** — copy-paste into Jira Create Issue | ✅ |
| **Automatic Jira ticket creation** via Atlassian MCP | 🚧 Level 2 — planned |
| **Bidirectional Jira** (read TCs from Xray/Zephyr, write results back) | 🚧 Level 3 — planned |

## Quick start

```
/plugin marketplace add MhdGad/mhdgad-plugins
/plugin install robntex@mhdgad-plugins
```

Then, in the project you want to test:
1. `/init-execution` — scaffolds `testcases/suite1/` with sample MD + Excel TCs and the
   `executions/` output folder.
2. Edit the sample TCs (module URLs, payer codes, disposable member IDs) to match your
   ROBIN environment.
3. `/execute-tc suite1/` — runs the suite. Add `parallel` for an autonomous batch.

See [One-time setup](#one-time-setup) for the Playwright + permissions steps.

## What's inside

| Component | File | Purpose |
|-----------|------|---------|
| Skill | `skills/testcase-execution/SKILL.md` | The orchestrator — modes, output layout, per-step evidence rules |
| Skill | `skills/testcase-parsing/SKILL.md` | Parses MD or Excel into canonical TC objects |
| Agent | `agents/tc-executor.md` | Subagent that runs one TC in an isolated browser session |
| Reference | `skills/testcase-execution/references/tc-format.md` | The canonical TC object + both input formats + validation rules |
| Reference | `skills/testcase-execution/references/playwright-cli.md` | The browser driver — setup & gotchas |
| Reference | `skills/testcase-execution/references/jira-bug-format.md` | The Jira-ready bug block layout |
| Reference | `skills/testcase-parsing/references/excel-parsing.md` | `openpyxl` reading pattern, grouping by TC-ID, gotchas |
| Command | `commands/init-execution.md` | `/init-execution` — scaffold sample TCs + `executions/` |
| Command | `commands/execute-tc.md` | `/execute-tc <suite \| file \| TC-IDs>` — run |
| Permissions | `settings.example.json` | Recommended permission rules for the project |
| Example TCs | `testcases/suite1/` | Ready-to-adapt Markdown + Excel samples |
| Config | `.env.example` | Optional `ROBIN_URL`, `ROBIN_ENV` |
| Output | `executions/` | Where each run's report, screenshots & bugs land (auto-created) |

## One-time setup

In the project you want to test:

1. **Playwright CLI** (the agent will offer to do this, or run it yourself):
   ```
   npm install -D @playwright/cli
   npx playwright-cli install-browser chromium
   ```
2. **openpyxl** (only if you have Excel TCs):
   ```
   pip install openpyxl
   ```
3. **Permissions** — plugin manifests can't ship permission rules, so copy the
   `permissions` block from [`settings.example.json`](./settings.example.json) into your
   project's `.claude/settings.json` (merge with anything already there). This pre-approves
   the safe `playwright-cli` + `openpyxl` commands and denies secret reads / destructive
   actions.
4. **Environment (optional)** — copy [`.env.example`](./.env.example) to `.env` and fill in
   `ROBIN_URL` / `ROBIN_ENV` if you want a default target instead of passing the URL each run.

## Writing your TCs

Start from the examples in [`testcases/suite1/`](./testcases/suite1/):
- Two Markdown TCs (`ROBIN-1001-...md`, `ROBIN-1002-...md`) covering positive and negative
  eligibility cases.
- One Excel workbook (`claims-remittance-suite.xlsx`) with 3 TCs across a `Claims` sheet
  and a `Remittance` sheet.

Both formats normalize to the same canonical TC object — the executor doesn't care which
you use. Full spec:
[`skills/testcase-execution/references/tc-format.md`](./skills/testcase-execution/references/tc-format.md).

### The one rule that matters most

**Every step needs its own `Expected` result.** That's what lets the executor report
"TC-ROBIN-1234 failed at step 3" precisely, and what lets the defect list link back to a
single failing step instead of a whole scenario.

## Modes

- **Sequential (default, human-in-the-loop):**
  > `/execute-tc suite1/` — the agent restates scope, shows the parsed TC list (with skips
  > and reasons), waits for approval, then runs TCs one at a time, pausing between TCs.

- **Parallel (autonomous):** one `tc-executor` subagent per TC, each in its own
  `-s=<session>`:
  > `/execute-tc suite1/ parallel` — expect ~6–8 browser sessions concurrent, rest queue.

- **Single TC or filtered list:**
  > `/execute-tc suite1/ROBIN-1001-eligibility-valid-emirates-id.md`
  > `/execute-tc suite1/ ROBIN-1001, ROBIN-2002`

- **Override target URL inline:**
  > `/execute-tc suite1/ https://robin-uat.your-company.example`

## Output

Every run writes everything under one timestamped folder:

```
executions/execu_<YYYY-MM-DD_HH-MM-SS>/
├── report.md
├── testcases/                          # copy of the TCs that ran
├── browser-sessions/<session>/
│   ├── logs/<TC-ID>-console.log
│   └── screenshots/<TC-ID>/step-N.png
└── bugs/
    ├── bug-list.md                     # Jira-ready blocks — one per defect
    └── screenshots/                    # copies of bug-evidence shots
```

Every defect in `bug-list.md` is a self-contained Jira Create Issue payload:
- `###` heading → **Summary**
- Bullet meta (Priority / Environment / Module) → Jira fields
- Steps / Expected / Actual → **Description**
- Evidence paths → attachments (drag-drop)

## Roadmap

- **Level 2 — Jira integration.** A `jira-integration` skill using the Atlassian MCP server
  to create tickets directly from `bug-list.md` blocks.
- **Level 3 — Bidirectional.** Read TCs from Xray/Zephyr; write results back on the same
  tickets.
- **Broader targets.** API and DB verification alongside the browser flow.

## License

MIT — see [LICENSE](./LICENSE).
