---
name: test-script-healer
description: Debug and fix failing automation tests, in any test framework or language. Use when the user reports failing, flaky or broken tests, pastes a test error, log or report, or asks to "heal", "fix" or "debug" a test file or test case (e.g. "TC-1657 is failing", "fix the login spec").
---

# Test Script Healer

Find out why a test fails and fix the **script**. Never hide a real product bug.

## Rules

1. **Follow [the shared code standards](../_shared/code-standards.md)** for any code you write or change. Fix the shared page object, helper or mock once, rather than patching each test.
2. Never make a test pass by weakening it: don't delete an assertion, loosen an expected value to match wrong app output, skip or disable the test, or wrap steps in try/catch.
   - **Wording mismatches are the classic trap.** If the app's text, label or tooltip differs from the test case or design, that's an expected failure to report. Never change the expected text to match the app.
3. Don't add fixed sleeps or blanket retries as a fix (standards §4). Wait on a condition instead.
4. Keep the change minimal and within scope. Don't refactor unrelated code while healing.
5. **Never change protected files or core packages.** That means the files the Project profile lists as protected (runner config, annotations, reporter, env config, CI, docs), plus dependencies: no installing, upgrading or removing packages. Don't work around them either. If the only fix is there, stop healing that test and report it as **blocked**, with the reason.
6. **At most 3 fix attempts per test (or per failure group).** An attempt is one change followed by one re-run. If it still fails after 3, stop and report what you tried, what you ruled out, and your best guess at the cause.
7. **Don't change a test's preconditions without asking.** If a fix would change a precondition from the test-case doc (account type, role, permission, data state), ask the user first. Otherwise the test might still pass while checking something different.
8. **Follow [the product context procedure](../_shared/product-context.md):** look up existing notes before investigating (step 1), and record what you learned at the end (step 6). The product context folder, the failure reports folder and the verify evidence folder (step 5) are the only docs locations this skill may write to (rule 5 otherwise applies).

## Workflow

### 1. Triage (before running anything)

- Do step A of [the product context procedure](../_shared/product-context.md) first: read the index from the profile, then every context file relevant to the failing tests' screens, flows and APIs, including related topics. A known quirk, flaky area or mismatch there may already explain the failure.

- **Set aside known bugs first.** If a failing test has a bug tag (Bug tag on test row, e.g. `@PROJ-123`), and the failure matches that bug (the same screen, element or message as the failure report or context notes describe), mark it **known bug** and don't investigate it. If it fails in a *different* way than the bug describes, treat it as a new failure.
  - **Check the tagged bug is still open** (one small Jira read: `status` only). If it was closed but the test still fails, it isn't a known bug any more. It was fixed and came back, or closed without a fix. Treat it as a new failure, and say so in the report.
  - If Jira isn't available, treat it as a known bug, and mark it **status not checked** in the report.
- List the remaining failing tests. **Group them by likely shared cause:** the same error message, the same failing locator or helper, or the same stack location. Heal one representative test per group, then re-run the whole group. Fix the cause once (rule 1).
- For each test, gather the **existing evidence first**: the error the user pasted, the last run's artefacts (per the profile, e.g. an error-context file, screenshots, a trace). Compare it with the test-case doc (its expected result, gaps and version).

### 2. Reproduce (only if needed)

- Use the **Project profile** (already loaded with the agent instructions) for the run-one-test command, the target environment, the repeat and debug options, and the **run side effects**.
- **Before any run,** if the profile says runs report to a shared dashboard, turn that off for this run as the profile describes, or warn the user first.
- Re-run the test only if the existing evidence is missing, out of date, or not enough to find the cause. Run only that test (or group), with a minimal reporter, and keep only the error output.
- Turn on tracing or verbose logging only if the cause is still unclear after that.
- For suspected flakiness, repeat it several times with a single worker, using the profile's repeat option.

### 3. Work out the cause, then act on it

| Cause | Signs | What to do |
|---|---|---|
| Locator or selector drift | Element not found, or ambiguous match, after a UI change | Search the current frontend source for the element. Update the page-object locator, keeping the repo's locator priority. |
| Timing or race | Passes on retry; acts before the page or data settles; stale reads | Wait on the right condition (URL, network response, loader gone, element state), or poll inside a named helper. |
| Test isolation | Passes alone, fails in a full or parallel run | Find the shared state (account, data, record changed by another test). Give the test its own setup or data. Make tests run serially only if the data really must be shared. |
| Test data or environment | Missing permission, data changed or deleted, environment down, email or OTP not received | Fix the setup, seed data or mock (rule 7 if it changes a precondition). If it's truly environmental, report it and don't change the code. |
| Mock or contract drift | The app calls a new endpoint, or the response shape changed | Search the backend source or schema for the endpoint, and update the shared mock module. |
| Script logic bug | Wrong step order, or an expected value that doesn't match the test case | Fix the script so it matches the test-case doc. |
| Outdated test | The test-case doc changed since the test was written (new version, different expected result, or now TBC) | **Don't heal it.** Report it, and suggest the `test-script-generator` workflow to update it. If the case is now TBC, it becomes pending there. |
| Real app bug or known design mismatch | App behaviour or wording contradicts the test case, AC or design | **Don't change the assertion** (rule 2). Report it as an expected failure. Don't add a comment about it. |
| Out of scope | The test checks behaviour the profile's **Out of scope** row lists (e.g. keyboard-only behaviour) | **Don't heal it.** Report it so the user can remove it. Never raise it as a bug. |
| Needs a protected file or package | The only fix is in config, annotations, reporter, CI, env or dependencies | **Blocked** (rule 5). Report it, and don't change anything. |

When a cause needs the frontend or backend source (locator drift, mock or contract drift), follow [the source access procedure](../_shared/source-access.md): local clones first, otherwise ask the user which GitHub repo and branch to use. Never guess a new locator or response shape without checking the source.

State the cause and the evidence before editing.

### 4. Fix

- Apply the smallest fix at the right layer: page object, helper or mock first, then the test.
- Count each change plus re-run as one attempt (rule 6).

### 5. Confirm

- Re-run the healed test or group. For flaky tests, repeat it: every run must pass.
- **Check the related tests,** so a fix in shared code doesn't break other tests:
  - **Find them by search:** for every page-object method, locator, helper or mock you changed, search the tests for its name or import, and list the files that use it. Add the other tests in the same file.
  - **Know the baseline:** before blaming your change, check whether each related test was already failing, using the last run's results or artefacts. If there's no recent result, run the related test once on the code before your change.
  - **Run them** with a minimal reporter, and show only the summary and failures. If more than about 5 other files are affected, list them and ask before running.
  - **If your change breaks a related test that passed before,** the change is wrong. Revert it, or adjust it so both pass, and count that as an attempt (rule 6). Never leave a related test broken. Report tests that were already failing separately, without touching them.
- **Remove fixed bug tags.** If a test that now **passes** has a bug tag (Bug tag on test row, e.g. `@PROJ-123`), remove that tag. Change only the tag list, and list it in the final report, since it suggests the bug is fixed.
- **Capture verify evidence for each removed tag.** For every bug key removed above, capture fresh proof that the bug is fixed, and save it in the folder from the profile's **Verify evidence** row, named after the key (e.g. `docs/verify_evidence/PROJ-123/`). One folder per key, even if several tests carried it.
  - **Reach the same state as the bug.** Use the reproduce steps from the failure report or test-case doc. If the bug's own evidence folder exists (**Bug evidence** row), match its screen, viewport, scroll position and highlighted element, so the before and after images compare side by side.
  - **Capture it the way `bugs-reporter` does** ([its step 4](../bugs-reporter/SKILL.md#4-draft-one-ticket-per-new-group)): a throwaway capture script that reuses the test's setup helper and page objects, run with dashboard reporting off; a clean frame (no loaders or toasts); sensitive data masked; never the login or OTP screens. Delete the script in the same run, then check the repo's status to confirm nothing is left behind.
  - **Files:**
    - `1-fixed-context.png`: the screen with the previously faulty element highlighted, now showing the correct result
    - `2-fixed-closeup.png`: just that element
    - `3-recording.webm`: for behaviour bugs only; starts after login, covers only the reproduce steps, under about 30 seconds
    - `verification.md`: the bug key, case IDs, environment, date and time with time zone, app version (or "unknown"), and one line per file saying what it shows. No secrets or personal data.
  - **Check each image yourself:** the fix must be clearly visible, highlighted and uncovered. If not, re-capture. If the state can't be reached again, say so in the final report rather than saving the test run's raw artefacts.
  - Don't comment on, transition or close the bug in Jira. Verifying it there is the user's call.
- Run the repo's type-check and lint commands on the changed files.
- **Write the failure report for new app bugs.** If any failure was classified as a **real app bug or known design mismatch** (not a known bug), write or update a report in the folder from the profile's **Failure reports** row, following [the failure report format](../_shared/failure-report.md), with "Generated by: test-script-healer". Take the story key from the case's test-case doc, and copy each failure's evidence next to the report. This is the input for the `bugs-reporter` skill.

### 6. Record product context (required)

- Do step B of [the product context procedure](../_shared/product-context.md). Typical findings: the root cause and what really fixed it, timing or loading behaviour, environment or data quirks, contract changes, causes you ruled out, and app bugs or mismatches reported as expected failures.
- Record this even when you stopped after 3 attempts or the test is blocked: what you ruled out saves the next session the same work.
- Update the context file and its index row in the same change.

## Final report

Keep it short, one line per test:
- **Healed:** cause, evidence and fix (`file:line`).
- **Bug tags removed:** passing tests whose `@KEY` tag was removed (the bug looks fixed), with each key's verify evidence folder, or why it couldn't be captured.
- **Known bugs:** failures already covered by their bug tag. Not investigated. Mark any whose Jira status couldn't be checked.
- **Related tests already failing:** found while checking related tests, failing before your change. Not touched.
- **Expected failures:** new app bugs or app-vs-design mismatches, including wording. Not changed. Written to the failure report; suggest `bugs-reporter` as the next step.
- **Environment issues:** not changed.
- **Outdated tests:** handed to `test-script-generator`.
- **Blocked:** the fix needs a protected file or package (rule 5).
- **Stopped after 3 attempts:** what was tried, what was ruled out, and the best guess.
- **Needs your decision:** precondition changes (rule 7).
- **Context updated:** the context files created or updated (step 6).

End with a results summary in the same four groups as the generator: **passed · skipped (TBC) · expected failures · real failures**.
