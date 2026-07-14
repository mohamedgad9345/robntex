# TC ROBIN-1002 · Eligibility check rejects malformed Emirates ID

- **Module:** Eligibility
- **Priority:** Medium
- **Stateful:** false
- **Jira:** ROBIN-1002

## Preconditions
- User is logged in to ROBIN as a Front-Desk operator.
- Payer `DAMAN-ENH` is active in the payer master.

## Test data
- badEmiratesId: 123-abc
- payer: DAMAN-ENH

## Steps

| # | Action | Expected |
|---|--------|----------|
| 1 | Navigate to **Eligibility → New Enquiry**. | The New Enquiry form is visible. |
| 2 | Select payer `DAMAN-ENH`, enter Emirates ID `123-abc`, and click **Check Eligibility**. | Inline validation error is shown on the Emirates ID field: "Invalid Emirates ID format". No network call is issued to the payer. |
| 3 | Clear the field and enter an empty value; click **Check Eligibility** again. | Inline "required" error appears; the form does not submit. |

## Notes
- Negative case: no successful eligibility response is expected in any step.
- Verify via network log that step 2 does NOT trigger a call to the payer endpoint.
