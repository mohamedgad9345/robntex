---
name: testcase-parsing
description: Parse ROBIN test cases from Markdown files or Excel workbooks into a canonical TC object array. Use whenever the testcase-execution orchestrator needs to load TCs from disk before dispatching them. Handles both formats, validates every TC, and rejects invalid ones with clear reasons.
---

# Test Case Parsing

## Role
You convert ROBIN test cases on disk (Markdown files or Excel workbooks) into an array of
**canonical TC objects** that the executor can consume. You do NOT execute anything — you
only parse and validate. If a TC is malformed, reject it with a clear message and continue
with the rest.

## Read before you act
The formats you must produce and consume are defined here:
- **`${CLAUDE_PLUGIN_ROOT}/skills/testcase-execution/references/tc-format.md`** — the
  canonical TC object shape and both surface formats (MD + Excel). This is the contract.
- **`${CLAUDE_PLUGIN_ROOT}/skills/testcase-parsing/references/excel-parsing.md`** — the
  `openpyxl` reading pattern, column-name mapping, and gotchas. Read before parsing any
  `.xlsx`.

## Input resolution

The orchestrator hands you one of:
1. A **folder path** (a suite) — parse every `.md` and `.xlsx` inside recursively.
2. A **single file path** — parse only that file.
3. A **list of TC-IDs** with a scope folder — parse the folder, then filter to just those IDs.

## Parsing flow

For each source file, in order:

1. **Detect format** by extension (`.md` → Markdown; `.xlsx` → Excel). Reject `.xls`, `.csv`,
   `.docx` with a clear message telling the user to convert.
2. **Parse to raw records** using the format's rules (see `tc-format.md`).
3. **Normalize** every record into the canonical TC object:
   ```
   { id, title, module, priority, preconditions[], test_data{}, steps[], stateful, jira_ticket? }
   ```
4. **Validate** every TC individually. Rejection criteria (each produces a `SKIPPED` entry
   with a reason — never crash the run):
   - Missing `id`, `title`, `module`, or `priority`.
   - `priority` not in `{Critical, High, Medium, Low}`.
   - `steps` empty or any step missing `action` or `expected`.
   - `Step #` values not contiguous starting at 1.
   - `jira_ticket` present but not matching `^[A-Z][A-Z0-9]+-\d+$`.
5. **De-duplicate** by `id`. If the same `id` appears in two files, reject both and report
   which paths conflicted — do NOT silently pick one.

## Output

Return two things to the orchestrator:

**Parsed TCs** — an array of valid canonical TC objects, ready to dispatch.

**Parse report** — a short summary:
```
Parsed <N> TCs from <M> file(s).
Valid: <valid_count>
Skipped: <skipped_count>
  - <TC-ID or filename>: <reason>
  - ...
Duplicates: <dup_count>
  - <TC-ID>: <path A>, <path B>
```

The orchestrator surfaces this report to the user BEFORE any browser action, so the user
sees exactly what will (and won't) be executed.

## Excel specifics

For `.xlsx` files:
- Use `openpyxl` with `data_only=True, read_only=True` (see the reference file).
- Match column names case-insensitively, by name — never by column index.
- Group rows by `TC-ID`; take header fields (Title/Module/Priority/Preconditions/Test Data/
  Stateful/Jira) from the **first non-empty value** in the group.
- Sort each group's rows by `Step #` before emitting the TC.
- One sheet = one suite (sheet name becomes the suite label in the report).

## Markdown specifics

For `.md` files:
- One TC per file. If a file has multiple `#` H1 headings, treat it as malformed.
- The H1 must contain the TC-ID; parse it out (`# TC ROBIN-1234 · ...` or
  `# ROBIN-1234 - ...`).
- Meta bullets under the H1 (`- **Module:** ...`, `- **Priority:** ...`, etc.) map to
  normalized fields.
- The `## Steps` section MUST be a table with `#`, `Action`, `Expected` columns.

## Rules

- Never modify the source file. Parsing is read-only.
- Never invent missing fields. Reject and report; the orchestrator decides whether to ask
  the user or proceed with the valid subset.
- Never print secrets found in `Test Data` (real Qatar IDs, tokens). If a value looks
  like real PHI, warn the user and mask it in reports.
- If `openpyxl` is missing, run `pip install openpyxl --break-system-packages` after
  telling the user; don't just fail.
