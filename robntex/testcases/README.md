# testcases/ — your ROBIN test cases

This is where you keep the test cases RobNTex executes. Nothing here is application code —
each file is a structured description of what to test and what to expect.

## How test cases are organized

- **One suite = one folder** — e.g. `testcases/suite1/`. A suite is what RobNTex runs
  together in a single command.
- **Two input formats, same result:**
  - **Markdown** — one TC per file, filename `<TC-ID>-<slug>.md`.
  - **Excel** — one workbook can hold many TCs; each sheet = a sub-suite, each row = a step.
- In parallel mode RobNTex dispatches **one `tc-executor` subagent per TC** (not per file),
  so a workbook with 20 TCs = 20 isolated browser sessions.

## Required fields (both formats)

- `TC-ID` — unique per suite. Free form (`TC-CLAIM-001`) or Jira key (`ROBIN-1234`).
- `Title` — short, action-oriented.
- `Module` — Eligibility / Claims / Remittance / PA / Denials / Config.
- `Priority` — Critical / High / Medium / Low.
- `Preconditions` — what must be true before step 1.
- `Steps` — numbered, ordered; **every step has its own `Action` + `Expected`**.

See `${CLAUDE_PLUGIN_ROOT}/skills/testcase-execution/references/tc-format.md` for the
full contract (canonical TC object + both surface formats + validation rules).

## Rules the agent already follows

- Never uses real personal data — masks Emirates IDs and uses disposable emails
  (`qa.tester@example.com`).
- Never submits real claims, never completes real enrollment or payment. Validation-only.
- Never reads or prints secrets, never modifies application source.
- Captures a screenshot per step (PASS and FAIL); console errors and failed network calls
  count as defects even when the UI looks fine.
- Never invents steps — executes the TC exactly as written. Ambiguous steps stop the TC
  and get flagged.

## Running

- Sequential (human-in-the-loop, default): `/execute-tc suite1/`
- Single TC: `/execute-tc suite1/ROBIN-1001-eligibility-valid-emirates-id.md`
- Filter by IDs: `/execute-tc suite1/ ROBIN-1001, ROBIN-1002`
- Parallel (autonomous): add "parallel" — `/execute-tc suite1/ parallel`
- Override target: append the URL — `/execute-tc suite1/ https://robin-uat.example.com`
