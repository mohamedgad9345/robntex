# Test Case Format (Markdown + Excel)

Every test case RobNTex executes — whether authored as Markdown or as a row in Excel —
must resolve to the same canonical **TC object** before execution. This file defines that
object and both surface formats.

## Canonical TC object (what the executor consumes)

```
{
  id:              string   # e.g. "ROBIN-1234" or "TC-CLAIM-042" — REQUIRED, unique per suite
  title:           string   # short action-oriented title — REQUIRED
  module:          string   # ROBIN module (Eligibility | Claims | Remittance | PA | Denials | Config)
  priority:        "Critical" | "High" | "Medium" | "Low"
  preconditions:   string[] # setup that must be true BEFORE step 1 (e.g. "User logged in as Coder")
  test_data:       object   # named values used in steps (e.g. { emiratesId: "784-...", cptCode: "99213" })
  steps:           Step[]   # numbered, ordered — see below
  stateful:        boolean  # if true, steps run in strict order in ONE session; if false, still ordered but not chain-dependent
  jira_ticket:     string?  # optional Jira key this TC traces to (e.g. "ROBIN-1234")
}

Step = {
  n:               integer  # 1-indexed
  action:          string   # what to do — imperative ("Enter Emirates ID in the search field")
  expected:        string   # what should happen after this action — REQUIRED per step
  test_data_ref:   string?  # optional pointer into test_data (e.g. "emiratesId")
}
```

**Rule:** every step MUST have its own `expected`. A TC without per-step expected results is
rejected by the parser — this is what lets the executor report "TC-X failed at step 3" precisely.

---

## Markdown format

One TC per file. Filename convention: `<TC-ID>-<slug>.md`
(e.g. `ROBIN-1234-eligibility-check-valid-emirates-id.md`).

```markdown
# TC ROBIN-1234 · Eligibility check with valid Emirates ID

- **Module:** Eligibility
- **Priority:** High
- **Stateful:** true
- **Jira:** ROBIN-1234

## Preconditions
- User is logged in to ROBIN as a Coder.
- Payer "Daman" is active in the payer master.

## Test data
- emiratesId: 784-1990-1234567-1
- payer: Daman
- serviceDate: 2026-07-10

## Steps

| # | Action | Expected |
|---|--------|----------|
| 1 | Navigate to Eligibility → New Enquiry. | Enquiry form is displayed with Payer and Emirates ID fields empty. |
| 2 | Select payer `Daman` from the dropdown. | Payer field shows "Daman"; Service Date defaults to today. |
| 3 | Enter Emirates ID `784-1990-1234567-1` and click "Check Eligibility". | Response returns within 30s; status shows "Eligible"; member details render. |
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
| `Module`      | ✅       | Eligibility / Claims / Remittance / PA / Denials / Config        |
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
| ROBIN-1234  | Eligibility – valid EID | Eligibility | High     | 1      | Navigate to Eligibility → New Enquiry | Enquiry form displayed            | User logged in as Coder    | payer=Daman; emiratesId=784-…| true     | ROBIN-1234  |
|             |                       |             |          | 2      | Select payer `Daman`                  | Payer field shows Daman           |                            |                              |          |             |
|             |                       |             |          | 3      | Enter Emirates ID and check           | Status "Eligible"; details render |                            |                              |          |             |

Only the **first row per TC-ID** needs to carry Title/Module/Priority/Preconditions/Test Data/
Stateful/Jira; the parser propagates them to the rest of the rows for that TC.

---

## Validation rules the parser MUST enforce

1. `TC-ID` unique per suite; if duplicated across files, fail loudly with both paths.
2. Every step has `Action` + `Expected`. Missing `Expected` → reject the TC.
3. `Priority` restricted to {Critical, High, Medium, Low}. Anything else → reject.
4. `Step #` starts at 1 and has no gaps within a TC.
5. `Module` value must be a known ROBIN module (see enum above). Unknown → warn, don't reject.
6. If `Jira` is present, format matches `^[A-Z][A-Z0-9]+-\d+$` (e.g. `ROBIN-1234`).
