---
description: Execute ROBIN test cases against a running environment. Accepts a suite folder (e.g. suite1/), a single TC file, or a list of TC-IDs. Sequential (human-in-the-loop) by default; say "parallel" for an autonomous batch.
---

Use the **testcase-execution** skill to run ROBIN test cases described below.

Target / scope: $ARGUMENTS

**Resolving the scope from the arguments:**
- A bare suite name (`suite1/`, `suite2/`, or "run suite3") → `./testcases/suite<N>/`.
  Parse and run every TC in that folder.
- A single file path (`.md` or `.xlsx`) → parse and run only that file.
- A list of TC-IDs (e.g. `ROBIN-1234, ROBIN-1235`) with an optional scope folder → parse
  the folder (default `./testcases/`) and run only the named TCs.
- A URL in the arguments → treat it as the `TARGET_URL` override for this run (else use
  `ROBIN_URL` from `.env`, else ask the user).
- If a named suite folder doesn't exist yet, create it and seed it with a starter TC copied
  from `${CLAUDE_PLUGIN_ROOT}/testcases/suite1/`, tell the user to adapt it, then continue.

**Before running:**
- If the current project has no `testcases/` directory (or it has no TC files), scaffold
  a starting point first — run the same steps as `/init-execution` (copy the bundled
  samples from `${CLAUDE_PLUGIN_ROOT}/testcases/suite1/` and ensure `./executions/`
  exists), tell the user the samples are editable examples, then continue.
- If `testcases/` already has the user's own TCs, skip the scaffold and use theirs.

**Parsing gate (before any browser action):**
- Invoke the **testcase-parsing** skill on the resolved scope.
- Show the user the parse report (valid / skipped / duplicates) with reasons.
- Proceed only if there is at least one valid TC. In sequential mode, wait for the user's
  confirmation of the parsed set before opening the browser.

**Mode selection:**
- Default: **sequential** — pause at each checkpoint (understand → plan → execute per TC →
  report). One session named `default`.
- If the arguments include "parallel" / "fast" / "regression" / "autonomous", use
  **parallel** mode — one `tc-executor` subagent per TC in an isolated session, no
  per-checkpoint pauses. Read the skill's `references/playwright-cli.md` before the first
  browser action.

**Output:**
- Every run writes under `./executions/execu_<YYYY-MM-DD_HH-MM-SS>/` — see the skill's
  Execution output layout section. The final `bugs/bug-list.md` uses the Jira-ready format
  (`references/jira-bug-format.md`) so defects can be copy-pasted into Jira Create Issue.
