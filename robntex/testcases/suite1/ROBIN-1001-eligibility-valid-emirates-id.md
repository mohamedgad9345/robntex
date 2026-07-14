# TC ROBIN-1001 · Eligibility check with valid Emirates ID

- **Module:** Eligibility
- **Priority:** High
- **Stateful:** true
- **Jira:** ROBIN-1001

## Preconditions
- User is logged in to ROBIN as a Front-Desk operator.
- Payer `DAMAN-ENH` is active in the payer master.
- The provider location is set for the operator's session.

## Test data
- emiratesId: 784-XXXX-XXXXXXX-X       <!-- disposable / masked; DO NOT use real PHI -->
- payer: DAMAN-ENH
- serviceDate: today

## Steps

| # | Action | Expected |
|---|--------|----------|
| 1 | Navigate to **Eligibility → New Enquiry**. | The New Enquiry form is visible with Payer and Emirates ID fields empty. |
| 2 | Select payer `DAMAN-ENH` from the dropdown. | Payer-specific fields (Service Type, Provider Location) render. Service Date defaults to today. |
| 3 | Enter the Emirates ID `784-XXXX-XXXXXXX-X` and click **Check Eligibility**. | A response returns within 30s with status `Eligible` and coverage details rendered. |
| 4 | Click **Save Enquiry**. | Success toast appears; the enquiry is listed in the Enquiries grid with today's date and status `Eligible`. |

## Notes
- Do NOT submit any real claim after this enquiry — this is a validation-only run.
- Any console error or failed network call counts as a defect even if the UI looks fine.
- Stateful chain: steps must run in this order in one session; do not retry a failed step.
