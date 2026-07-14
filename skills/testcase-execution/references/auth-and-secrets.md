# Authentication & Secrets Handling

RobNTex needs to log into ROBIN to execute test cases whose preconditions require an
authenticated user state. This file is the **contract** for how the executor reads
credentials, uses them, and — most importantly — never leaks them.

## The three whitelisted app auth vars

Only these env vars are "app credentials" the executor is allowed to read for the purpose
of logging into ROBIN. Every other secret-looking env var (`*_KEY`, `*_TOKEN`, `*_SECRET`,
`PASSWORD*`, etc.) remains off-limits.

| Var             | Purpose                                                              |
|-----------------|----------------------------------------------------------------------|
| `ROBIN_URL`     | Root URL of the target ROBIN environment (Staging, UAT, …).          |
| `ROBIN_USER`    | Login username or email.                                             |
| `ROBIN_PASSWORD`| Login password.                                                      |

If any of the three is missing or empty when a TC requires authentication, the executor
STOPS and reports `BLOCKED — missing credentials` back to the orchestrator. The executor
does NOT prompt the user for credentials in-conversation and does NOT invent a placeholder.

## Login flow (preflight)

The executor runs this preflight ONCE per session, before the first step of the first TC
in the session — provided the TC's preconditions mention login state (e.g. "User is logged
in as X"). If the session is already authenticated from a previous action, skip.

1. Navigate to the ROBIN login page — see `robin-login.md` for the environment-specific
   path (once confirmed by the team). Default assumption: `$ROBIN_URL/login`.
2. `snapshot` to get element refs.
3. Type `$ROBIN_USER` into the username field. Use playwright-cli's `type` command with
   the value coming from the environment — do NOT echo the value in shell output.
4. Type `$ROBIN_PASSWORD` into the password field, same handling.
5. Click the submit / sign-in button.
6. Wait for the URL or DOM to indicate a signed-in state (dashboard visible / URL changes
   away from `/login`).
7. Take a `screenshot` of the LANDING page (not the login form) as evidence that login
   succeeded — save under `browser-sessions/<session>/screenshots/_login/landing.png`.
8. On failure: retry the login at most **2 more times**, spaced. After 3 total attempts,
   STOP with `BLOCKED — login failed` to avoid account lockout.

Once logged in, proceed with the TC's step 1. Do NOT include the login sequence as steps
in the TC — it's precondition handling, not TC content.

## The five inviolable rules

1. **Never echo credential values.** Not in shell output, not in logs, not in `report.md`,
   not in `bug-list.md`. If a reference is needed, use the variable name (`$ROBIN_USER`),
   never the value.
2. **Never screenshot the password field with the value visible in plain text.** If a
   "show password" toggle is on by default, turn it off before screenshotting, or skip
   the screenshot for that specific step.
3. **Never persist credentials to disk under `executions/`.** No `.env` copy, no dumped
   env vars, no debug prints.
4. **Redact credentials from any captured console / network log** before writing to
   `session/logs/`. If a request body contains the password, mask it as `***REDACTED***`.
5. **Never commit `.env`.** It's already in `.gitignore` — keep it there.

## What the executor CAN read

- The three `ROBIN_*` vars listed above (via shell `${ROBIN_USER}` interpolation or
  process env — either is fine as long as the value never appears in stdout/stderr).

## What the executor MUST NOT read

- `.env` values other than the three whitelisted vars above.
- Any file matching `*credentials*`, `*secret*`, `*.pem`, `*.key`.
- Real patient PHI from a DB or file — use disposable test data only, always.

## When to STOP and BLOCK

Do NOT try to work around these situations — stop and report:
- Any of the three `ROBIN_*` vars is missing or empty.
- Login fails 3 times (do not attempt password reset flows).
- The ROBIN URL is unreachable.
- The password appears to be expired (ROBIN prompts for a new password on first login).
