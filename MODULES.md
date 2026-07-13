# ROBIN Modules — RobNTex reference

This file is the **team's living catalog** of the ROBIN modules RobNTex knows about. It's
consulted (not enforced) by the parser and by the Jira-ready bug format when populating the
`Module` field on a TC or defect.

## How this file works

- **Free-form on entry:** a TC's `Module` field can hold any string. If the string isn't
  listed here yet, the parser emits a **warning** (not a rejection) and the run continues.
- **We add modules as we encounter them.** When Mohamed or the team confirms a new module
  is part of ROBIN, add a row below with a short description of what the module covers.
  That way anyone using RobNTex later has a shared vocabulary.
- **Case-insensitive matching.** `Eligibility`, `eligibility`, and `ELIGIBILITY` are the
  same module.

## Known modules

| Module        | Confirmed | Description | Notes |
|---------------|:---------:|-------------|-------|
| _None yet — Mohamed will add them as we work._ | | | |

<!--
Template row to copy when adding a new module:

| <ModuleName>  | ✅ / 🤔    | <one-line description> | <any gotchas, related modules, or Jira component mapping> |

The "Confirmed" column:
  ✅ = Mohamed / team explicitly confirmed this module exists in ROBIN.
  🤔 = Mentioned in a TC or conversation but not yet confirmed; treat as tentative.
-->

## Not a module

Things RobNTex should NOT treat as modules (put them under `Notes` on a TC instead):

- Test environments (`UAT`, `Staging`, `Prod`) — those go in the `Environment` field.
- Payer codes, provider codes, or member IDs — those go in `Test Data`.
- Jira project keys — those go in `Jira` or `linked_issue`.
