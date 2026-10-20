---
name: test-script-review
description: Review automation test code for quality, code debt and exposed secrets or sensitive data, then apply the fixes, in any test framework or language. Use when the user asks to "review", "clean up" or "audit" tests, a test file, a branch or recent changes, or asks to check for leaked credentials or personal data before a commit or push.
---

# Test Script Review

Review changed test code, report findings, then fix them.

## Rules

1. **Enforce [the shared code standards](../_shared/code-standards.md)**: reuse and focused helpers, one source of truth, separate layers, shared mocks, simplicity for QA engineers, waits and assertions, and comments.
   - Fixes merge duplicates into shared helpers instead of adding code, and replace over-engineered code with the simpler version.
   - Don't propose "more advanced" rewrites of code that's already clear.
   - When removing comments (standards §5), strip implementation notes from inside section markers and keep the bare marker. If an old marker name has been replaced (per the profile), rename it.
2. **Never modify** these files; report findings in them instead:
   - **Core framework setup:** runner and test config, build and dependency files, lockfiles, compiler and lint config.
   - **Custom annotation or metadata code, and reporter setup.**
   - **CI and pipeline configuration.**
   - **Environment config and env files.**
   - **Development repository documentation:** reference folders such as `docs/`, plus `README`, `AGENTS.md` and `CLAUDE.md`.
   - **Anything else** listed as protected in the Project profile of `AGENTS.md` (or `CLAUDE.md`).
   - The one exception is the product context folder from the profile, which this skill updates in workflow step 5 (rule 8).
3. **Never change a test's meaning.** Don't touch test IDs, priorities, expected results, or which behaviour an assertion checks. Cleanup only.
   - A test that **passes either way** (it branches to accept two different outcomes) can't be fixed without choosing an expected result. Report it as **needs user action**, since it's really an open question (a TBC gap). Don't rewrite it or pick a branch.
4. Never print a secret you find. Refer to it by file and line, and mask the value (`abc***`).
5. **Review only the changed code.** Check its links to old code with targeted searches, never by reading old files in full (see "Related code" and checklist D). Fix old code only when a fix to the new code requires it, for example merging a duplicate or updating callers of a changed method. Don't review or report on old code otherwise.
6. **Small fixes are applied automatically. Structural fixes need confirmation.**
   - **Auto-apply:** comments, unused code, naming, waits and sleeps, simplifying one method, section markers, TBC marker form.
   - **Confirm first:** merging page objects or methods, moving mocks into a shared module, changing a shared helper's behaviour or signature, and any change to old code. List them in the report, and apply only the ones the user approves.
7. **Don't debug.** If a test fails after your fixes, revert the fix that caused it, and report it. Diagnosing failures is `test-script-healer`'s job.
8. **Follow [the product context procedure](../_shared/product-context.md):** look up existing notes before reviewing (workflow step 1), and record what you learned at the end (step 5). Notes explain *why* code is the way it is (e.g. a wait that covers a known quirk), so check them before flagging something as wrong.

## Scope

Review what the user names: a file, a folder, a commit, or a branch. If they don't say, review **only the uncommitted changes** (`git diff` plus new untracked files). Review a whole branch only when asked. If the scope is large (more than about 1,000 changed lines), say so and ask whether to narrow it.

**Read the diff, not whole files.** Review the changed hunks with a little surrounding context (e.g. `git diff -U10`). Open a larger part of a file only when a finding needs it, and read just that part. Read new files in full. For conventions, use the Project profile already loaded with the agent instructions; don't read the README.

**Related code: search, don't read.** Check links to old code with targeted searches (grep for a name, selector, string or signature), and read only the matching lines:
- **Duplicates:** for each new method, locator, constant or mock, search old code for the same selector, string or a similar name.
- **Callers:** for each changed or removed shared method, locator or signature, find its callers and read only the call lines.
- Don't read whole dependencies, and don't review sibling files for the same screen.

## Checklist

**A. Sensitive data (always check first)**
- **Hard-coded secrets:** passwords, OTPs, API keys, tokens or JWTs (`eyJ…`), private keys, connection strings, cloud or vendor keys. Credentials must come from the repo's config or secret mechanism.
- **Real personal data:** emails outside the company domain, phone numbers, national ID numbers (e.g. NRIC/FIN), bank account numbers, real customer wallet addresses. Test data must be synthetic. Under privacy rules such as PDPA, **warn the user** and ask before continuing.
- **Secrets leaking through logs:** printing tokens, passwords, full API responses or env values.
- **Committed artefacts:** among the new and changed files in scope, any env file other than the example template, traces, screenshots or reports.

**B. Test correctness** (from the Project profile)
- **Out of scope:** a new test for behaviour the profile's **Out of scope** row lists (e.g. keyboard-only behaviour). Report it as **needs user action**, and don't delete it yourself.
- **False passes:** a test with no steps and no pending marker. The framework reports it as passed.
- **TBC marker form:** pending tests use exactly the profile's TBC / pending marker, one that still lets the required metadata reach the reporter (e.g. metadata first, then the in-body pending call, never a form that skips the body).
- **Passes either way:** branching that accepts two outcomes. Needs user action (rule 3).
- **Case ID ↔ test-case doc:** each new test's ID exists in the test-case docs, and its title matches that case. Skip this for suites that still use an older doc-ID scheme.
- **Metadata:** required metadata is present and consistent (e.g. tags match priority; extra bug tags from the Bug tag on test row are allowed). Test IDs are unique and never reused.
- **Section markers:** new test files have the section markers the profile requires.
- **Utilities and config:** the repo's assertion and wait utilities are used, and banned APIs are absent. Settings come from the config layer, never from env variables directly in tests. File naming and imports follow the repo.

**C. Code quality and flakiness**
- **Any violation of [the shared code standards](../_shared/code-standards.md).**
- Dead code: unused imports, variables, methods, classes and files.
- Brittle locators (positional XPath, generated classes) where a semantic locator works.
- Weak typing (`any`, raw `Object`), missing `await` or async handling.
- Race conditions: acting before load or settle, reading lists before refresh, point-in-time visibility checks used as assertions.
- Order-dependent tests, or mutable data shared between parallel workers.

**D. New vs old code** (search-based only; see Scope → Related code)
- **Duplication or missed reuse:** a new locator, helper, constant or mock that already exists in old code. Reuse the old one.
- **Duplicate page objects or methods:** a new page object for a screen, drawer or modal that already has one, or a new page-object method that does what an existing method does. Search the page-object folders by screen name, route and main locators, then merge (rule 6: confirm first).
- **Mocks in the wrong place:** mock builders or fixture data inside a test file instead of the shared mock module (rule 6: confirm first).
- **Breaking changes:** a changed or removed shared method, locator, signature or default value that old callers still use. Update those callers, or keep compatibility.
- **Duplicate coverage:** the same case automated twice, or a new test that makes the same check as an existing one under a different ID. Search by test ID, and by behaviour as [the coverage check](../_shared/coverage-check.md) describes: distinctive terms (exact expected text, page-object method) or, for behaviour-only tests, the same-area test titles. Report it as **needs user action**, naming both tests. Never delete a test yourself, since that changes coverage.

## Workflow

1. **Collect.**
   - Do step A of [the product context procedure](../_shared/product-context.md): read the index from the profile, then the context files for the screens and flows the diff touches, including related topics.
   - Get the diff (Scope).
   - Run lint on the changed files only, and the type-check. Keep only the errors from their output.
   - Run the mechanical checks as searches over the changed lines, not by reading: secrets and PII patterns, banned APIs, fixed sleeps, direct env reads, missing metadata, duplicate test IDs, TBC marker form, case IDs missing from the docs, missing section markers.
   - **Duplicate coverage works the other way round:** take the distinctive terms **from** the changed tests, and search the **existing** tests outside the diff for them (checklist D).
   - Then read the diff hunks for the judgement checks (false passes, branching, quality, flakiness, comments, simplicity).
2. **Report.** List findings ranked by severity. Use one line each: `file:line`, the problem, the fix. Mark each as **auto** or **needs confirmation** (rule 6). Don't paste code blocks, except where a fix isn't obvious from one line.
   - **Critical:** secrets or PII. Stop and confirm with the user before going on, because leaked secrets may need rotating and history rewriting, not just a code edit.
   - **High:** false passes, tests that pass either way, bugs, flakiness.
   - **Medium:** convention violations.
   - **Low:** cleanup.
3. **Apply.** Apply the auto fixes, then the confirmed structural fixes. Never touch protected files (rule 2). Touch old code only as rule 5 allows.
4. **Verify.**
   - Run lint and type-check on the changed files.
   - **Before running tests,** check the profile's **run side effects**. If runs report to a shared dashboard, turn that off for this run as the profile describes, or warn the user first.
   - Run the touched tests with a minimal reporter. Show only the summary and any failures, not full logs.
   - Run old tests only if you changed a shared method, locator or helper they call. If that's more than about 5 test files, list them and ask before running.
   - If a test fails after your fixes, revert the causing fix and report it (rule 7).
5. **Record product context (required).** Do step B of [the product context procedure](../_shared/product-context.md). Typical findings: product facts that were only in the implementation-note comments you removed (standards §5), reasons behind non-obvious waits or data choices, and duplicate helpers or page objects that cover the same screen. Update the context file and its index row in the same change.

## Final report

Keep it short:
- Findings, each marked **fixed**, **awaiting confirmation**, **reverted** (broke a test), **skipped** (with the reason) or **needs user action** (e.g. passes either way, or a leaked key to rotate).
- Any old code changed, and why.
- Information from removed comments that's still needed (standards §5).
- Files changed.
- Context files created or updated (step 5).
- Verification summary in the same four groups as the other skills: **passed · skipped (TBC) · expected failures · real failures**.
