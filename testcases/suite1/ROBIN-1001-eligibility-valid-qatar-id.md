# TC ROBIN-1001 · Eligibility check with valid Qatar ID

- **Module:** Eligibility
- **Priority:** High
- **Stateful:** true
- **Jira:** ROBIN-1001

## Preconditions
- User is logged in to ROBIN as a Front-Desk operator.
- Payer `<PAYER-CODE-FROM-DB>` is active in the payer master.   <!-- fill from your real DB payer master -->
- The provider location is set for the operator's session.

## Test data
<!-- Replace these with real values from your test data set. TEST-QID-001 is a clearly-fake
     placeholder — do NOT use a real patient Qatar ID here. -->
- qatarId: TEST-QID-001
- payer: <PAYER-CODE-FROM-DB>
- serviceDate: today

## Steps
<!-- The steps below are a skeleton — fill in with the actual UI labels and expected texts
     as they appear in your ROBIN environment. -->

| # | Action | Expected |
|---|--------|----------|
| 1 | Navigate to **Eligibility → New Enquiry**. | The New Enquiry form is visible with Payer and Qatar ID fields empty. |
| 2 | Select payer `<PAYER-CODE-FROM-DB>` from the dropdown. | Payer-specific fields render. Service Date defaults to today. |
| 3 | Enter the Qatar ID `TEST-QID-001` and click **Check Eligibility**. | A response returns within 30s with status `Eligible` and coverage details rendered. |
| 4 | Click **Save Enquiry**. | Success toast appears; the enquiry is listed in the Enquiries grid with today's date and status `Eligible`. |

## Notes
- Do NOT submit any real claim after this enquiry — validation-only run.
- Any console error or failed network call counts as a defect even if the UI looks fine.
- Stateful chain: steps must run in order in one session; a failed step blocks the rest.
