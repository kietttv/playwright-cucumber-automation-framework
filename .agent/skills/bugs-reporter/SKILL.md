---
name: bugs-reporter
description: Raise Jira bug tickets for app failures found by automated tests. Reads a failure report (written by test-script-generator or test-script-healer) or a pasted generator or healer report, groups failures by root cause, checks Jira for existing bugs so nothing is duplicated, shows drafts for approval, then creates one bug per root cause assigned to the current user and linked to its original story (when that ticket and link type exist), saves its evidence locally, and adds the bug key as a tag on the failing tests. Framework-independent. Use when the user asks to "raise bugs", "log bugs", "create bug tickets" or "report the app failures" for a ticket, spec or failure report.
---

# Bugs Reporter

Turn app failures from automated test runs into Jira bug tickets: one per root cause, never duplicated, assigned to the user.

Repo-specific values come from these rows of the Project profile: **Bug tickets**, **Bug evidence**, **Bug tag on test**, **Failure reports**, **Run side effects** and **Ticket system**.

## Rules

1. **Only app failures become bugs:** cases where the app's behaviour or wording differs from the AC or design. Never raise bugs for script failures, flaky tests, environment problems, TBC gaps, or behaviour the profile's **Out of scope** row lists (mark those `not raised: out of scope`). TBC gaps are open questions for the PO, not bugs.
2. **One ticket per root cause, not per test.** Failures with the same cause (the same label, field, rule, endpoint or component) go in one ticket. Track which case IDs it covers in the failure report, not in the ticket (rule 10).
3. **Never duplicate an open bug.** Search Jira before drafting (step 3). If an **open** bug already covers the cause, don't create a new one: mark the failures `dup of <KEY>`. Closed bugs don't count: if a fixed bug comes back, create a new ticket.
4. **Nothing is written to Jira without the user's approval.** Show every draft first (step 5), and create only the ones the user approves. Never edit, comment on, transition, reopen, link or close existing tickets unless the user explicitly asks. The one exception: each new bug is linked to its original ticket (step 6), which the user approves along with the draft.
5. **Assign every new bug to the current user** (the account the Jira connector is authenticated as).
6. **Facts only.** Expected results come from the AC or design. Actual results come from the evidence, quoted exactly for text. Never guess at causes.
7. **No secrets, credentials, env URLs with tokens, or real personal data** in tickets. Describe accounts by type (e.g. "a user without the Reports permission"), never by email or password. Mask anything sensitive. Warn the user if the evidence contains real personal data (PDPA).
8. **Follow [the product context procedure](../_shared/product-context.md):** look up notes before starting (step 0), and record the raised bugs at the end (step 8).
9. **Every ticket has exactly these sections, in this order:** **Preconditions** (optional) → **Steps to Reproduce** → **Actual Result**, with a screenshot or recording → **Expected Result**. Nothing else in the description. The format is in [bug-template.md](bug-template.md).
10. **Everything is human-readable.** Write for a developer or PO who has never seen the test:
    - **Concise and unambiguous.** One fact per bullet, plain English, no "works correctly" or "is broken".
    - **The title is one plain sentence** in the title format from the **Bug tickets** row (at minimum `[<Area>] <what's wrong>`), naming the screen or element as users see it.
    - **The title prefix always names the area** where the bug lives, plus anything else the Bug tickets row asks for (e.g. a phase or release, `[<Area> - Phase 3]`). Leave extras out if the row doesn't mention them. Choose the area as [bug-template.md](bug-template.md) describes: the page, else the feature or flow, else the API or service.
    - **Steps** are numbered actions a person can follow by hand, from logging in, with UI names in bold.
    - **Actual and Expected** are observable facts, with exact on-screen text in backticks.
    - **No automation details anywhere in the ticket:** no test IDs, locators, code, stack traces, file paths or tool names.
11. **Priority follows the bug's impact, not the test case's priority.**
    - **High:** a core flow is blocked, or data is wrong or missing.
    - **Medium:** a feature behaves incorrectly, but there's a workaround or the flow still completes.
    - **Low:** a minor behaviour or UI problem.
    - **Lowest:** wording, label or cosmetic only.

    Use the case priority only as a hint, and give a one-line reason for each priority at the checkpoint. Map to the project's priority names (Bug tickets row).
12. **Keep Jira reads small.** Always request only the fields you need (e.g. `summary`, `status`, plus `description` only when comparing a likely match), with a small result limit. Bug descriptions embed images and can be huge.
13. **Evidence must make the problem obvious at a glance** to someone who has never seen the test: the right moment, the faulty element highlighted, nothing covering it, sensitive data masked, and a matching design image for comparison when there is one (step 4).

## Workflow

### 0. Read existing context

- Do step A of [the product context procedure](../_shared/product-context.md) for the story's area. Known mismatches or bugs recorded there may already have tickets.

### 1. Get the failures

- **Input**, in this order:
  - a report path the user gives
  - the newest report for the named ticket or spec in the failure reports folder
  - a generator or healer final report pasted in the chat
- The report follows [the failure report format](../_shared/failure-report.md). Take only the **App failures** table (rule 1), and skip rows whose **Bug** column is already filled.
- **Find the original ticket:** the story the failures were found in. Take it from the report's **Story** field, otherwise the ticket the user named. If neither gives one (e.g. a pasted report with no key), ask the user now, before step 3, so the key goes through the same check. If there isn't one, create the bugs without a link, and ask the user for the fix version instead (step 4).
- **Check the evidence is current.** If the report is **more than 2 days old**, or its app version differs from the one now deployed, tell the user and offer to re-run the affected tests first. Before any run, turn off reporting to shared dashboards as the Run side effects row describes. Drop failures that now pass.

### 2. Group by root cause

- Start from the report's suspected groups, then review them:
  - **Merge** failures that share a cause: the same wrong copy source, the same missing field from one API, the same component, or one rule broken on several screens.
  - **Split** a group whose failures have different causes.
- For each group, note the shared symptom, the affected case IDs, the screens it appears on, and its impact (rule 11).

### 3. Check Jira for open bugs

- Use the Atlassian (Jira) MCP tools. If they're unavailable or not authorised, say so: the user must connect the Atlassian connector, with **write** access for creating issues, in their connector or MCP settings. Offer to save the drafts to a file (step 5) for manual creation. Don't stop silently. Without Jira, the duplicate search and the link check below can't run, so every saved draft is marked **not checked for duplicates or links**.
- First check locally: the failure report's **Bug** column, and the known issues in the product context notes. A failure already mapped to an open ticket there doesn't need a new one.
- Then, for each group, search **open** bugs in the project (rule 3), keeping reads small (rule 12). Tickets don't contain test IDs (rule 10), so search by **symptom**:
  - the area part of the title prefix (e.g. `summary ~ "<Area>"`)
  - two or three distinctive keywords: the wrong or expected on-screen text, the element, the screen
  - Example JQL: `project = <KEY> AND issuetype = Bug AND statusCategory != Done AND summary ~ "<Area>" AND text ~ "<on-screen text>"`
- A clear match means the group is a **duplicate** of that ticket (rule 3). A possible match goes to the user to decide at step 5.
- **Check the link to the original ticket** (step 1) before drafting. Only link when both of these exist:
  - **The ticket:** fetch it by key (rule 12: `summary`, `status` and `fixVersions` only). Keep its fix versions for step 4.
  - **The link type:** the type from the **Bug tickets** row, otherwise `Relates` ("relates to"). Look it up with the connector's link-types tool and match its name exactly.

  If either is missing, don't link, and don't fall back to another link type. Show **not linked: <reason>** at the checkpoint (e.g. `not linked: no "Relates" link type`).

### 4. Draft one ticket per new group

- Use [bug-template.md](bug-template.md), following rules 9–11. The template is based on the team's existing bugs. Re-check it against two or three recent bugs (rule 12: `summary` and `description` only, `maxResults` 3) **only** if the user says the style has changed or drafts look off.
- **Steps** come from the test-case doc, rewritten as plain manual steps. They don't come from the automation code. If the bug appears on several screens, write the steps for the main one, and add an Actual Result bullet: `The same issue appears on: **<Screen>**, **<Screen>**.`
- **Capture clear evidence** (rule 13). Don't attach the test run's raw failure screenshot or full-test video. Those show the end state of the whole page, often with leftover toasts, and the video includes the login screens.
  - **Re-capture the bug state.** Write a small temporary capture script that reuses the test's setup helper and page objects to reach the exact state from the steps. Run it with dashboard reporting off (Run side effects row), and delete it afterwards. Never commit it.
    - If the test runner only finds tests in the repo's test folder, put the script there under an obvious throwaway name (e.g. `__evidence_capture__.spec.ts`), so nobody mistakes it for a real test.
    - Delete it in the same run, then check `git status` to confirm no capture files are left behind.
  - **Screenshots** (static bugs: wording, labels, missing or wrong elements, layout):
    - **Context:** the screen at the normal viewport, scrolled so the problem is in view, with the faulty element **highlighted** (e.g. a red outline drawn on it just before the screenshot).
    - **Close-up:** just the faulty element, with a little padding around it.
  - **Recording** (behaviour over time: navigation, filtering, sorting, timing): start **after login** (e.g. from a saved signed-in state), cover only the reproduce steps, and keep it short (aim for under 30 seconds).
  - **Expected:** if the profile lists a local design prototype and the same screen can be reached in it, capture the same screen and element at the same viewport, with the same highlight, so the two can be compared side by side.
  - **Clean frame:** wait for loaders to finish and toasts to disappear (or close them) before capturing. Nothing may cover the problem.
  - **Mask sensitive data** with the framework's masking option: emails, account numbers, wallet addresses, names that could be real. Never capture login or OTP screens.
  - **Check each file yourself.** Open every image and confirm the problem is clearly visible, highlighted and uncovered. If not, re-capture. Note any file you couldn't check (e.g. a video) at the checkpoint.
  - If the state can't be reached again (e.g. it needed data you can't recreate), fall back to the test run's artefacts, and say so at the checkpoint.
- Set the fields from the **Bug tickets** row: project, issue type, priority (rule 11), the story's fix version (from step 3; if there's no story, or it has no fix version, ask the user), and labels and other links only if the row lists any. The link to the original ticket is added only if it passed the check in step 3 (step 6).
- Before creating, check the project's issue-type metadata for required fields, and fill them in or ask the user.
- Find the current user's account ID through the connector's user-info tool, and set them as assignee (rule 5).

### 5. Review with the user (checkpoint)

- **Number the drafts** (1, 2, 3 …), and show a short table: number, draft title, priority with its reason, affected case IDs, status (**new** or **duplicate of KEY** or **possible match: KEY**), the original ticket it will be linked to (or **none**, or **not linked: <reason>** from step 3), and evidence: the files captured (e.g. `1-actual-context.png`, `2-actual-closeup.png`, `3-expected-design.png`), each marked **will attach** if the connector can upload files, otherwise **attach manually**.
- Show full drafts (all four sections) only when there are 3 or fewer, or when the user asks (e.g. "show 2").
- **Stop.** Accept short approvals, e.g. "approve all", "approve 1, 2, 4", "approve all except 3", or edits like "2: priority Low". Create only the approved drafts, with the user's edits applied. For a **possible match**, the user decides whether it's a duplicate or a new ticket.
- If Jira can't be written to, save the approved drafts to `<report name>_bug-drafts.md` next to the report, for manual creation. Head the file with a note that the drafts weren't checked for duplicates or links, so step 3 must be run (or the same searches done by hand) before anyone creates them.

### 6. Create and save evidence

- Create each approved ticket. If one fails to create, don't retry blindly: carry on with the rest, mark its rows `not raised: create failed (<error>)`, and list it in the final report. It gets no link, no evidence folder and no test tag.
- **Link it to the original ticket** with the connector's issue-link tool, using the ticket and link type confirmed in step 3. Skip this if step 3 found the ticket or the link type missing. If the link call fails, keep the bug, report the failure in the final report, and don't try another link type. Never add links to duplicates or other existing bugs (rule 4).
- **Save its evidence** in the folder from the **Bug evidence** row, named after the new key, e.g. `docs/bugs_evidence/PROJ-123/`:
  - `1-actual-context.png`: the screen with the faulty element highlighted
  - `2-actual-closeup.png`: just the faulty element
  - `3-expected-design.png`: the design at the same place, if captured
  - `4-recording.webm`: the short recording, for behaviour bugs

  The numbers keep them in reading order in Jira, and the folder shows which files go on which ticket. For drafts saved to a file (no Jira access), use `draft-<group>/` and rename it to the key once the ticket exists.
- Attach the files to the ticket if the connector supports attachments. Otherwise, the final report lists each ticket's evidence folder for manual attaching.
- Update the failure report's **Bug** column for every row: the new key, `dup of <KEY>`, or `not raised: <reason>` (e.g. "user declined", "now passes").

### 7. Tag the failing tests

- For every test whose row now has a key (newly created **or** `dup of <KEY>`), add the key as a **tag** on the test, in the format from the **Bug tag on test** row (e.g. `{ tag: ['@high', '@regression', '@PROJ-123'] }`).
  - A second bug means a second tag. Skip keys that are already tagged.
  - Change only the tag list: never the title, test ID, metadata, steps or assertions.
  - Find each test by its case ID (the report's Case ID column), not by its old title.
- Run the repo's type-check and lint on the changed test files.
- Keys are removed again by `test-script-generator` or `test-script-healer` when a tagged test passes.

### 8. Record product context (required)

- Do step B of [the product context procedure](../_shared/product-context.md): add each bug (key, one-line symptom, case IDs, date) under the area's known issues, so later sessions recognise these failures as already raised.
- Update the context file and its index row in the same change.

## Final report

- Tickets created: key, title, priority, the case IDs each covers (from the failure report), and the original ticket it's linked to (or **not linked**, with the reason).
- Duplicates: case IDs → existing key.
- Not raised, with the reason (declined, now passes, not an app failure, create failed).
- Evidence folders per ticket, marked **attached** or **attach manually**.
- Evidence to double-check: tickets that fell back to the test run's raw artefacts, and files you couldn't check yourself (e.g. recordings).
- Tests tagged with a bug key: case ID → key, per file.
- The failure report and context files updated.
