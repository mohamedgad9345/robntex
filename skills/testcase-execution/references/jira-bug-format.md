# Jira-Ready Bug Format (Level 1 — copy-paste output)

RobNTex writes every defect in the `bug-list.md` in a format that maps 1:1 to a Jira "Create
Issue" form. In Level 1 there is **no automatic Jira integration** — the user copies the block
into Jira. This file defines the exact structure so the copy-paste is friction-free.

## One bug block

```markdown
### BUG-<nnn> · <concise action-oriented title>

- **Linked TC:** <TC-ID> (failed at step <k>)
- **Jira Project:** <e.g. ROBIN>          <!-- from the TC's `Jira` field prefix if present -->
- **Priority:** <Critical | High | Medium | Low>
- **Environment:** <UAT | Prod | Dev | Staging>
- **Module:** <see MODULES.md for the team's list; free-form if new>          <!-- becomes Jira Component -->

**Steps to reproduce**
1. …
2. …
3. …

**Expected result**
…

**Actual result**
…

**Evidence**
- `bugs/screenshots/<TC-ID>-step-<k>.png`
- `browser-sessions/<session>/logs/<TC-ID>-console.log` (if a console/network error)

**Notes**
- Any relevant browser console errors, failed network calls, or timing observations.
```

## Rules

1. **One block per defect.** If the same underlying issue breaks 5 TCs, write ONE block and list
   all 5 TC-IDs under `Linked TC`, comma-separated, with the step each one failed at.
2. **Title style** — action-oriented, present tense, no filler:
   - ✅ "Eligibility check returns 500 for valid Qatar ID"
   - ❌ "There is a bug in eligibility that sometimes makes the check fail"
3. **Priority mapping** (inherit from the TC unless the defect is worse than the TC's severity):
   - Critical: blocks the whole flow, corrupts data, or breaks a regulator/payer submission
     from the target country (edit this line per your deployment — e.g. the national health
     insurance authority or the payer's submission gateway).
   - High: fails an acceptance criterion of a High-priority TC.
   - Medium: cosmetic/UX issues that don't block the flow.
   - Low: nice-to-have, minor wording, non-blocking.
4. **Evidence paths must resolve** relative to the execution folder root — never absolute paths.
5. **Never include secrets or real patient data** in the block — no tokens, no real Qatar
   IDs / national IDs, no real patient names. Use clearly-fake placeholders in test data
   (e.g. `TEST-QID-001`); if a real ID accidentally leaks in, mask it before the block is
   written.

## Copy-paste target in Jira

The block is designed so:
- The `###` heading becomes the **Summary**.
- The bullet meta lines map to **Priority**, **Environment**, **Component** (Module) fields.
- The three text sections (Steps / Expected / Actual) go into the **Description** field.
- The Evidence file paths are attached separately (drag-and-drop into the Jira attachments).

In Level 2 (future), the `jira-integration` skill will consume this same block and POST it to
Jira via the Atlassian MCP server, so the format is forward-compatible.

## `bug-list.md` file structure

```markdown
# Defect list — execu_<timestamp>

**Run summary:** <n> pass / <m> fail / <k> defects
**Suite:** <path>
**Target:** <url>

---

### BUG-001 · …
…

### BUG-002 · …
…
```
