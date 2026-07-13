# TC ROBIN-1002 · Eligibility check rejects malformed Qatar ID

- **Module:** Eligibility
- **Priority:** Medium
- **Stateful:** false
- **Jira:** ROBIN-1002

## Preconditions
- User is logged in to ROBIN as a Front-Desk operator.
- Payer `<PAYER-CODE-FROM-DB>` is active in the payer master.

## Test data
- badQatarId: 123-abc      <!-- deliberately malformed -->
- payer: <PAYER-CODE-FROM-DB>

## Steps
<!-- Skeleton — adjust UI labels and expected error texts to what ROBIN actually shows. -->

| # | Action | Expected |
|---|--------|----------|
| 1 | Navigate to **Eligibility → New Enquiry**. | The New Enquiry form is visible. |
| 2 | Select the payer, enter Qatar ID `123-abc`, and click **Check Eligibility**. | Inline validation error is shown on the Qatar ID field (e.g. "Invalid Qatar ID format"). No network call is issued to the payer. |
| 3 | Clear the field and enter an empty value; click **Check Eligibility** again. | Inline "required" error appears; the form does not submit. |

## Notes
- Negative case: no successful eligibility response is expected in any step.
- Verify via network log that step 2 does NOT trigger a call to the payer endpoint.
