---
description: Initialize RobNTex in the current project — scaffold a sample test-case suite and the executions output folder.
---

Initialize **RobNTex** in the current project so the user has a working starting point for
executing ROBIN test cases. Do these steps, then report what was created:

1. **Sample TCs** — if `./testcases/` has no test-case files, copy the bundled samples from
   `${CLAUDE_PLUGIN_ROOT}/testcases/suite1/` into `./testcases/suite1/` (creating
   `./testcases/` if needed). The samples include a Markdown TC and a small Excel workbook
   so the user sees both input formats.
   If `./testcases/` already has the user's own TCs, do NOT overwrite them — leave them as
   they are and say so.

2. **Executions folder** — ensure `./executions/` exists (this is where each run's
   `report.md`, screenshots, and `bug-list.md` land). Do NOT create a timestamped
   `execu_<...>/` folder now; that happens when a test actually runs.

3. **Permissions reminder** — remind the user to copy the `permissions` block from
   `${CLAUDE_PLUGIN_ROOT}/settings.example.json` into their project's
   `.claude/settings.json` if they haven't already (plugin manifests can't ship permission
   rules). Point out that RobNTex needs the same `playwright-cli` permissions AgenTeX uses,
   plus `openpyxl`-related Python execution for Excel parsing.

4. **Preflights** — mention the two prerequisites and offer to run them (do not install
   without confirmation):
   - `@playwright/cli` — `npm install -D @playwright/cli && npx playwright-cli install-browser chromium`
   - `openpyxl` — `pip install openpyxl` (only needed if the user has Excel TCs).

5. **Environment hint** — if `.env` is absent, mention that RobNTex reads `ROBIN_URL` from
   `.env` as the default target (users can override per run). Point to
   `${CLAUDE_PLUGIN_ROOT}/.env.example` as a template. Do NOT create `.env` automatically.

Finish by telling the user to edit the sample TCs in `./testcases/suite1/` to match their
ROBIN environment (payer codes, member IDs, module URLs), then run
`/execute-tc <suite or TC-ID>`.
