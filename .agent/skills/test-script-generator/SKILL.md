---
name: test-script-generator
description: Generate automation test scripts from a test-case document, in any test framework or language. Use when the user asks to "automate", "script" or "generate tests/specs for" a ticket, test suite or list of test cases (e.g. "generate scripts for PROJ-101", "automate docs/testcases/xxx.md"). Builds empty annotated tests first, then page objects, then step-by-step implementations.
---

# Test Script Generator

Turn test cases into automation scripts that match the target repo's existing framework and patterns.

## Rules

1. **Follow [the shared code standards](../_shared/code-standards.md)**: reuse and focused helpers, one source of truth, separate layers, shared mocks, simplicity for QA engineers, waits and assertions, and comments.
2. Test titles, metadata and logs are concise, unambiguous and easy to understand.
   - One behaviour per test. Follow the repo's title style (e.g. `Verify <observable behaviour>`).
   - Metadata fields (scenario, expected result, …) are a short phrase or sentence. Never paste the whole case.
3. **The test ID is the case ID from the doc.** Test-case docs already use the repo's test-ID scheme (e.g. `TC-2135`), so use each case's ID as-is. Never allocate a different one and never keep a mapping. Only if a case has no ID: allocate the next free ID (profile command) and tell the user, so they can add it to the doc.
4. Never generate a test for a case that is already automated, or for behaviour an existing test already checks (see step 2 and [the coverage check](../_shared/coverage-check.md)).
5. Put tests in the file for their module or scenario: the existing file for that flow, or a new file named according to the repo's naming convention.
6. **Never script out-of-scope behaviour.** If a doc case tests something the profile's **Out of scope** row lists, don't create a test for it at all. Show it at the checkpoint as **out of scope, not scripted**.
7. **Never implement steps for a TBC case.** A case is TBC if the doc marks it TBC, blocked or partly blocked (e.g. ⏸️), or its expected result depends on an open gap.
   - Create its empty test (title, tags and metadata, as in step 3), but write **no steps**, page-object methods or mocks for it.
   - Mark it pending, with the gap as the reason, using the Project profile's **TBC / pending marker**. If the profile has none, use the framework's pending or skip mechanism (e.g. Playwright `test.fixme`, JUnit `@Disabled("TBC: G4 …")`), and suggest adding one to the profile. Otherwise an empty test would be reported as **passed**.
   - The marker must still let the required metadata reach the reporter, so the test shows up as skipped instead of disappearing.
   - Don't script it to the doc's working assumption.
   - If the doc allows two outcomes, the case is TBC. Don't write one test that passes either way.
   - **Exception: wording gaps follow the design.** If a case is TBC only because the **text** is unclear (copy, labels, tooltips, chip wording, empty-state messages), it isn't TBC. Write the expected text from the design source in the profile, implement the steps, and report any app mismatch as an expected failure.
     - If a stakeholder comment in the ticket disagrees with the design's wording, point it out in the report, because the user may prefer the comment.
     - Gaps about **behaviour** (what a count includes, filter logic, date boundaries, search rules) stay TBC; ask the user.
8. **Follow [the product context procedure](../_shared/product-context.md):** look up existing notes before exploring (step 1), and record what you learned at the end (step 7).

## Workflow

### 1. Learn the repo (every run; don't rely on memory)

- Do step A of [the product context procedure](../_shared/product-context.md) first: read the index from the profile, then every context file relevant to the screens, flows and APIs in the test-case doc, including related topics. Use them in steps 4–5 before searching the sources.

- Use the **Project profile** in the agent instructions (already loaded; don't re-read it or the README) for the stack, commands, test-ID scheme, metadata format, assertion and wait utilities, banned APIs, section markers, file naming, mock module location, test-case doc location, reference sources, run side effects and protected files.
  - If a field is missing, infer it from the repo (build files, config, existing tests), state the assumption, and suggest adding it to the profile.
- Look at one recent test file in the same area (newest in `git log`): its imports, section markers, setup helper, and one or two tests. Read only those parts, not the whole file. Copy the structure, not the implementation notes.

### 2. Parse the test cases and remove duplicates

- The source is the test-case doc location from the profile, or whatever the user provides. Extract each case's ID, title, priority, preconditions, steps and expected results.
- Find the cases that are already automated by **searching the tests for each case ID** (rule 3). A match means the case is automated.
- **Also check by behaviour,** not just by ID ([coverage check](../_shared/coverage-check.md)). For each remaining case, search the existing tests by its distinctive terms (exact expected text, page-object method with its argument), and for behaviour-only cases compare against the same-area test titles. Keep the search cheap, as the coverage check describes.
  - **Duplicate:** an existing test already makes the same check. Don't script it. Mark it **possible duplicate of <ID>** for the checkpoint.
  - **Overlap:** an existing test already makes part of the check. Script only the new part.
- Skip automated cases and list them in the final report. If a doc case changed since it was scripted (e.g. a new version in the doc's change log), **update** the existing test instead of adding one.
- Mark out-of-scope cases (rule 6). They get no test at all.
- Mark TBC cases (rule 7), noting the gap each one waits on.
- A case that was TBC and is now ready in the doc (its gap is resolved) gets its steps implemented, and its pending marker removed.
- **Batches:** if more than about 20 cases need work, split them into batches by section of the doc (e.g. A, B, C). Run steps 3–6 for one batch at a time, and finish each batch before starting the next.

### 3. Generate empty tests (checkpoint)

- Choose the target file (rule 5). A new file gets the section markers the profile requires: setup goes in the precondition section, tests in the test section.
- For each case in the batch, write a test with its case ID, title, tags and required metadata, and **no steps**. TBC cases also get the pending marker (rule 7).
- **Stop and show the user** the list: case ID, title, priority, target file, and whether it's ready, TBC (with the gap), a **possible duplicate of <ID>**, or **out of scope, not scripted**. The user decides each possible duplicate. Continue only after they confirm.

### 4. Design or update page objects, objects and mocks

- For each screen or API involved, **search** the frontend and backend sources listed in the profile for what you need: the component's roles, labels and text for locators, and the endpoint's response schema for mocks. Read only the matching parts, not whole files.
- **Where to search:** follow [the source access procedure](../_shared/source-access.md): local clones first, otherwise ask the user which GitHub repo and branch to use.
- Search the page-object, helper and mock folders for existing code, and extend it. Create a new page object only for a genuinely new page, component, drawer or modal. Follow the repo's page-object conventions.
- Put mock builders, fixture data and route installers in the shared mock module from the profile, never in the test file. Reuse an existing mock module for the same area.
- Put shared data models or builders where the repo keeps them.

### 5. Implement the steps

- Fill each **ready** test step by step, following the case, as the shared standards describe (one step block per doc step, no branching, no fixed sleeps). Leave TBC tests empty and pending (rule 7).
- Factor repeated setup (log in, navigate, install mocks) into one setup helper per file or area.
- When the design or AC and the current app disagree, assert the **design or AC**, and list the mismatch in the final report as an expected failure.
- Before moving on, re-read each test as a QA engineer would. If a test or helper takes more than a quick read to understand, simplify it.

### 6. Verify

- Run the repo's type-check and lint commands on the changed files. Both must be clean.
- **Before running tests,** check the profile's **run side effects**. If runs report to a shared dashboard, turn that off for this run as the profile describes. If the profile gives no way to do that, warn the user first.
- Run the new or changed tests against the environment named in the profile. Run the **new** tests a second time with the profile's repeat option (about 3 repeats), to catch flaky ones.
- Sort the results into four groups:
  - **Passed**
  - **Skipped (TBC)**
  - **Expected failures:** the app differs from the design or AC
  - **Real failures:** the script is wrong or unstable, including tests that pass only sometimes
- **Don't start healing on your own.** Report the real failures with their errors, and ask the user whether to run the `test-script-healer` workflow. Never weaken an assertion to make a test pass.
- **Remove fixed bug tags.** If a test that **passed** (not skipped) has a bug tag (Bug tag on test row, e.g. `@PROJ-123`), remove that tag. Change only the tag list, and list each removal in the final report, since it suggests the bug is fixed.
- **Write the failure report.** If there's at least one expected failure, write a report in the folder from the profile's **Failure reports** row, following [the failure report format](../_shared/failure-report.md). Copy each app failure's evidence next to it, and suggest grouping for failures that look like one root cause. This is the input for the `bugs-reporter` skill.

### 7. Record product context (required)

- Do step B of [the product context procedure](../_shared/product-context.md). Typical findings: how to reach a screen or state, which account or permission it needs, endpoints the screen calls and their response quirks, timing or loading behaviour, and app-vs-design mismatches found in step 6.
- Update the context file and its index row in the same change.

## Final report

- The batch or cases generated, with their case IDs and file.
- Cases skipped because they were already automated, or already covered by another test (case ID → existing test).
- Out-of-scope cases not scripted (rule 6).
- TBC cases left pending, each with the gap it waits on.
- Cases without an ID in the doc, and the ID allocated to each.
- Expected failures: cases where the app differs from the design or AC.
- Accounts, test data and mocks used, and any assumptions.
- Page objects, helpers and mock modules created or changed.
- Test results, in the four groups from step 6.
- Bug tags removed from passing tests: case ID → key.
- The failure report path, if one was written. Suggest `bugs-reporter` as the next step.
- Context files created or updated (step 7).
