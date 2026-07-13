# Test Case Format (Markdown + Excel)

Every test case RobNTex executes — whether authored as Markdown or as a row in Excel —
must resolve to the same canonical **TC object** before execution. This file defines that
object and both surface formats.

## Canonical TC object (what the executor consumes)

```
{
  id:              string   # e.g. "ROBIN-1234" or "TC-CLAIM-042" — REQUIRED, unique per suite
  title:           string   # short action-oriented title — REQUIRED
  module:          string   # ROBIN module — free-form, but see ${CLAUDE_PLUGIN_ROOT}/MODULES.md
                            # for the running list of modules the team has agreed on.
                            # New modules get added to that file as we encounter them.
  priority:        "Critical" | "High" | "Medium" | "Low"
  preconditions:   string[] # setup that must be true BEFORE step 1 (e.g. "User logged in as Coder")
  test_data:       object   # named values used in steps (e.g. { qatarId: "TEST-QID-001", cptCode: "99213" })
  steps:           Step[]   # numbered, ordered — see below
  stateful:        boolean  # if true, steps run in strict order in ONE session; if false, still ordered but not chain-dependent
  jira_ticket:     string?  # optional Jira key this TC traces to (e.g. "ROBIN-1234")
}

Step = {
  n:               integer  # 1-indexed
  action:          string   # what to do — imperative ("Enter Qatar ID in the member field")
  expected:        string   # what should happen after this action — REQUIRED per step
  test_data_ref:   string?  # optional pointer into test_data (e.g. "qatarId")
}
```

**Rule:** every step MUST have its own `expected`. A TC without per-step expected results is
rejected by the parser — this is what lets the executor report "TC-X failed at step 3" precisely.

---

## Markdown format

One TC per file. Filename convention: `<TC-ID>-<slug>.md`
(e.g. `ROBIN-1234-eligibility-check-valid-qatar-id.md`).

```markdown
# TC ROBIN-1234 · Eligibility check with valid Qatar ID

- **Module:** Eligibility
- **Priority:** High
- **Stateful:** true
- **Jira:** ROBIN-1234

## Preconditions
- User is logged in to ROBIN as a Coder.
- Payer `<PAYER-CODE-FROM-DB>` is active in the payer master.  <!-- fill from your DB -->

## Test data
- qatarId: TEST-QID-001                                <!-- clearly-fake placeholder — replace with a real test QID from your test data -->
- payer: <PAYER-CODE-FROM-DB>
- serviceDate: 2026-07-10

## Steps

| # | Action | Expected |
|---|--------|----------|
| 1 | Navigate to Eligibility → New Enquiry. | Enquiry form is displayed with Payer and Qatar ID fields empty. |
| 2 | Select payer `<PAYER-CODE-FROM-DB>` from the dropdown. | Payer field shows the selected payer; Service Date defaults to today. |
| 3 | Enter Qatar ID `TEST-QID-001` and click "Check Eligibility". | Response returns within 30s; status shows "Eligible"; member details render. |
| 4 | Click "Save Enquiry". | Success toast appears; enquiry is listed in the Enquiries grid with today's date. |

## Notes
- Do not submit any real claim after this enquiry.
- Flag any console error or failed network call as a defect even if the UI looks correct.
```

**Required sections:** the H1 title (with TC-ID), the meta bullets (Module, Priority, Stateful),
Preconditions, and the Steps table with `#`, `Action`, `Expected` columns. Test data and Notes
are optional but recommended.

---

## Excel format

One **sheet** = one **suite**. One **row** = one **step** of a TC. Multiple TCs per sheet are
grouped by `TC-ID` (rows sharing the same `TC-ID` belong to the same TC and run in order).

**Required columns** (header row = row 1, case-insensitive match):

| Column        | Required | Notes                                                            |
|---------------|:--------:|------------------------------------------------------------------|
| `TC-ID`       | ✅       | Same value across all rows of one TC                             |
| `Title`       | ✅       | Repeat on each row of the TC (parser reads it from the first row) |
| `Module`      | ✅       | Free-form; see `MODULES.md` for the team's running list.         |
| `Priority`    | ✅       | Critical / High / Medium / Low                                   |
| `Step #`      | ✅       | 1-indexed; determines execution order                            |
| `Action`      | ✅       | Imperative                                                       |
| `Expected`    | ✅       | Per-step expected result — required on every row                 |
| `Preconditions` | ⚪    | Fill on row 1 of the TC only (parser reads it from the first row) |
| `Test Data`   | ⚪       | JSON blob OR `key=value; key=value` on row 1 of the TC           |
| `Stateful`    | ⚪       | `true`/`false` on row 1 (defaults to `true` if omitted)          |
| `Jira`        | ⚪       | Jira key (e.g. `ROBIN-1234`)                                     |

**Empty rows separate TCs is NOT reliable** — always group by `TC-ID`. The parser sorts rows of
one TC by `Step #`, so row order in the sheet does not matter.

### Example row layout (one TC across three rows)

| TC-ID       | Title                 | Module      | Priority | Step # | Action                                | Expected                          | Preconditions              | Test Data                    | Stateful | Jira        |
|-------------|-----------------------|-------------|----------|--------|---------------------------------------|-----------------------------------|----------------------------|------------------------------|----------|-------------|
| ROBIN-1234  | Eligibility – valid QID | Eligibility | High     | 1      | Navigate to Eligibility → New Enquiry | Enquiry form displayed            | User logged in as Coder    | payer=<PAYER-CODE>; qatarId=TEST-QID-001 | true     | ROBIN-1234  |
|             |                       |             |          | 2      | Select payer from dropdown            | Payer field shows selected value  |                            |                              |          |             |
|             |                       |             |          | 3      | Enter Qatar ID and check              | Status "Eligible"; details render |                            |                              |          |             |

Only the **first row per TC-ID** needs to carry Title/Module/Priority/Preconditions/Test Data/
Stateful/Jira; the parser propagates them to the rest of the rows for that TC.

---

## Validation rules the parser MUST enforce

1. `TC-ID` unique per suite; if duplicated across files, fail loudly with both paths.
2. Every step has `Action` + `Expected`. Missing `Expected` → reject the TC.
3. `Priority` restricted to {Critical, High, Medium, Low}. Anything else → reject.
4. `Step #` starts at 1 and has no gaps within a TC.
5. `Module` — free-form string; if the value is not in `MODULES.md`, warn but do not
   reject (new modules get added to that file as the team encounters them).
6. If `Jira` is present, format matches `^[A-Z][A-Z0-9]+-\d+$` (e.g. `ROBIN-1234`).
