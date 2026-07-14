# TC ROBIN-1003 · Create patient — happy path (Qatar)

- **Module:** Patient Management       <!-- tentative; add to MODULES.md once confirmed -->
- **Priority:** High
- **Stateful:** true
- **Jira:** ROBIN-1003

## Preconditions
- ROBIN Staging is reachable at `$ROBIN_URL`.
- `$ROBIN_USER` and `$ROBIN_PASSWORD` are set in `.env`.
- **User will be logged in by the executor's preflight** (see
  `skills/testcase-execution/references/auth-and-secrets.md`) — login steps are NOT part
  of this TC.
- Payer `<PAYER-CODE-FROM-DB>` is active in the payer master.
- No patient exists with QID `TEST-QID-CREATE-<RUN_TS>` (the timestamp suffix avoids
  collisions across runs).

## Test data
<!-- These are placeholders. Replace UI-visible values with the exact wording your ROBIN
     environment uses (e.g. gender option label, nationality option label). -->
- qatarId: TEST-QID-CREATE-{RUN_TS}    <!-- executor substitutes {RUN_TS} with the run timestamp -->
- firstName: Ahmed
- lastName: TestPatient
- dob: 1990-01-15
- gender: Male
- nationality: Qatari
- phone: +974-XXXXXXXX
- email: qa.tester@example.com
- payer: <PAYER-CODE-FROM-DB>
- membershipNumber: MEM-TEST-001

## Steps
<!-- Skeleton. Adjust the menu path, button labels, and field names to what ROBIN actually
     shows. The executor will honour whatever wording you put in "Action" and match against
     what you put in "Expected". -->

| # | Action | Expected |
|---|--------|----------|
| 1 | Navigate to **Definitions → Patients → +Add Patient** (adjust to the actual menu path in ROBIN). | The New Patient form is displayed with the required-field indicators visible on Qatar ID, First Name, Last Name, DOB, and Payer. |
| 2 | In the Personal Info section, fill: Qatar ID `TEST-QID-CREATE-{RUN_TS}`, First Name `Ahmed`, Last Name `TestPatient`, DOB `1990-01-15`, Gender `Male`, Nationality `Qatari`. | Fields accept the values. QID field passes format validation (no inline error). DOB field shows the parsed date. |
| 3 | In the Contact section, fill: Phone `+974-XXXXXXXX`, Email `qa.tester@example.com`. | Phone field accepts the +974 format without error. Email field passes format validation. |
| 4 | In the Insurance section, select payer `<PAYER-CODE-FROM-DB>` from the dropdown and enter Membership Number `MEM-TEST-001`. | Payer field shows the selected value; Membership Number field is enabled and accepts the value. Effective/Expiry Date fields default to reasonable values (today / today+1y) or become mandatory as per the payer's setup. |
| 5 | Click **Save** (or **Create Patient** — whichever label your build uses). | Success toast appears (text like "Patient created successfully"); the page navigates to the patient detail view OR returns to the patient list with the new record visible. |
| 6 | Navigate to **Patient Management → Search** and search by Qatar ID `TEST-QID-CREATE-{RUN_TS}`. | Exactly one result is returned. The patient's displayed data (name, DOB, payer, membership number) matches what was entered in steps 2–4. |

## Notes
- **Dynamic QID.** The `{RUN_TS}` token in the Qatar ID lets multiple runs execute without
  patient-uniqueness collisions. The executor MUST substitute `{RUN_TS}` with the run
  timestamp (`YYYYMMDD-HHMMSS`) before typing it into the form, and log the resolved value
  once in `report.md` so the tester can look up the record later.
- **Stateful chain.** Steps must run in order in one session. If step 5 fails, step 6 is
  meaningless — mark it SKIPPED and record the failure at step 5.
- **No cleanup in this TC.** The created patient is left in the environment. A separate
  cleanup TC or a scheduled DB job should handle removal — do NOT delete records here.
- **No PHI.** All values are clearly-fake test data. The QID pattern
  `TEST-QID-CREATE-<timestamp>` is intentionally not a valid Qatar ID format.
- Any console error or failed network call during any step counts as a defect even if the
  UI outcome matches Expected.
