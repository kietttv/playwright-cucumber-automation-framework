---
name: test-cases-generator
description: Generate manual test cases (a test-suite document) from Jira tickets, based only on the ticket's acceptance criteria and designs (dev repos are used only to make the steps accurate). Framework-independent. Use when the user asks to "write test cases", "create a test suite" or "generate cases" for a Jira ticket (e.g. "PROJ-101"), or asks which of their assigned tickets need test cases.
---

# Test Cases Generator

Read Jira tickets and turn their acceptance criteria (AC) and designs into a test-suite document.

## Rules

1. Test cases are concise, unambiguous and easy to understand.
   - One behaviour per case. Steps are numbered actions, and expected results are observable facts. Never write "works correctly".
   - Quote exact UI copy or API values **as they appear in the AC or design**. Name the exact account type or data each case needs.
2. **Build test cases from the AC and design only.**
   - The *what*: which cases exist, their scenarios, expected results, copy, priorities and coverage. These come only from the ticket (description, AC, decision comments, attachments) and the design sources.
   - Never derive a case, expected result, or piece of copy from the development repos: the frontend and backend source code, their text or i18n files, schemas, tests or mocks.
   - The dev repos may be used **only for the *how***: how to reach a screen or state, which navigation path, fields or inputs to use, or which endpoint and request to call. See step 3.
   - The test verifies the code against the spec. Building the spec from the code would make every case pass by definition.
3. Every AC is covered by at least one case, and every case traces back to an AC, a design, or a clearly labelled edge case.
   - **Out of scope:** never write cases for behaviour the profile's **Out of scope** row lists, even if an AC mentions it. List those ACs in the report as out of scope.
   - **No duplicate coverage.** Follow [the coverage check](../_shared/coverage-check.md). Never write a case for behaviour that another case already covers, in this suite, another suite or the existing tests. Merge cases that differ only by data, unless the data changes the result.
4. Never invent behaviour. When the AC or design is silent or contradictory, log a **gap** with a working assumption. Don't guess silently.
   - If you notice while looking up steps that the code does something the AC or design doesn't mention, or does it differently, **don't add or change a case because of it**. Log it as a gap to confirm with the PO, and keep the case written to the AC or design.
5. Jira is read-only. Never comment on, transition or edit a ticket unless the user explicitly asks.
6. If a ticket contains real customer data (non-company emails, account numbers, national ID numbers, anything under privacy rules such as PDPA), warn the user and ask before using it. Test cases use synthetic data.
7. **Follow [the product context procedure](../_shared/product-context.md):** look up existing notes before exploring (step 0), and record what you learned at the end (step 6). Context notes help with the *how* (step 3) and with spotting gaps; they never replace the AC or design as the source of expected results (rule 2).

## Workflow

### 0. Read existing product context

- Do step A of [the product context procedure](../_shared/product-context.md): read the index from the profile, then every context file relevant to the ticket's area, including related topics.

### 1. Get the tickets

- Use the Atlassian (Jira) MCP tools. If they're unavailable or not authorised, say so and tell the user to connect the Atlassian connector in their AI tool's connector or MCP settings. Then offer to work from a ticket key plus pasted content. Don't stop silently.
- Only tickets matching the profile's **Ticket intake** row (issue type and status) are in scope. If the profile has no such row, ask the user which issue type and status to use, and suggest adding the row.
  - If the user doesn't name a ticket, search with the intake JQL. Show a short list (key, summary, fix version) and let the user choose. If the search finds nothing, say so. Don't widen the filter unless the user asks.
  - If the user names a ticket, fetch it. If it doesn't match the intake type and status, tell the user its actual type and status, and ask before continuing.
- From each ticket, collect:
  - The description and AC.
  - The epic and fix version.
  - The reporter or product owner.
  - Linked tickets (they define what's out of scope).
  - Comments with decisions.
  - Attached designs (images, Figma links).
- **Only use attachments you could actually open.** If an attachment or link can't be opened or viewed (unsupported format, no access, the tool returns no content), list it in the report as **not reviewed**, and never guess what it shows. Ask the user to describe it, or to save it into the repo's design folder, if a case depends on it.

### 2. Gather the design

- Use the **Project profile** (already loaded with the agent instructions) for where designs and specs live in this repo. Designs are design-team output, such as prototypes, screen specs and rulebooks. Dev repos are not designs.
- Look in order: ticket attachments and links, then the repo's design or spec folder (including its changelog), then any naming or formatting rulebook.
- If the design is newer than the AC, the design wins for UI detail. Record every conflict as a gap.
- For API or backend tickets without a UI design, write the expected results from the AC. Use an API contract as a source of expected results only if it's attached or linked in the ticket. If the AC doesn't define a response, log a gap. Don't fill it in from the backend code.

### 3. Look up how to perform the steps (optional, dev repos)

Use this only once the case list and expected results are already fixed from steps 1–2.
- **Allowed:** read the frontend and backend sources, following [the source access procedure](../_shared/source-access.md) (local clones first, otherwise ask which GitHub repo and branch), to make the *steps* accurate and runnable. For example, the navigation path to a screen, the order of actions, which role or data puts the app into the required state, required form fields, or the endpoint and request to call.
- **Not allowed:** changing the case list or any expected result because of what the code does (rule 2). Anything the code shows beyond or against the AC or design is logged as a gap (rule 4).

### 4. Check for an existing suite

- Find the test-case doc location in the profile (default `docs/testcases/`). Look for an existing `<KEY>_*.md` there.
- If one exists, **update it**: bump the version, add a change-log entry, and never renumber existing cases. Mark removed cases with strikethrough.
- **Check coverage elsewhere** ([coverage check](../_shared/coverage-check.md)). For each planned case, search the other suite docs and the existing tests by its distinctive terms (exact expected text, page-object method), and for behaviour-only cases compare against the same-area test titles, as the coverage check describes.
  - **Duplicate:** don't write the case. List it in the suite's **Already covered** section, with the existing case or test ID.
  - **Overlap:** keep only the new part of the case.
  - **Within this suite:** merge cases that only differ by data into one case with a data list, unless the data changes the expected result.

### 5. Write the suite

- Use the template in [template.md](template.md). If the repo already has suite documents, match their style over the template.
- **Case IDs use the repo's test-ID scheme** from the profile (e.g. `TC-2135`), so automation uses the same ID with no mapping.
  - Allocate IDs sequentially from the **next free ID across both the tests and the existing suite docs**, using the profile's command. Suite docs reserve IDs before they're automated.
  - When updating an existing suite, new cases get the next free IDs, and existing cases keep theirs. Never renumber.
- Title format: `Verify_<Area>_<Behaviour>`.
- Priority:
  - 🔴 High: core flow or data integrity.
  - 🟡 Medium: validation, permissions, edge cases.
  - 🟢 Low: cosmetic or copy.
- Cover:
  - Each AC's positive path.
  - Negative and validation cases.
  - Role and permission variants.
  - Empty, loading and error states.
  - Boundaries.
  - Navigation.
  - Copy, tooltips and messages.
  - For APIs: status codes, schema and auth.
- Save to `<test-case doc location>/<KEY>_<Short_Title>_Test_Suite.md`. This and the product context folder (step 6) are the only reference-docs locations this skill may write to.

### 6. Record product context (required)

- Do step B of [the product context procedure](../_shared/product-context.md). Typical findings: business rules and decisions from the ticket and its comments, navigation paths and states found in step 3, design-vs-AC conflicts, and open gaps with their IDs.
- Update the context file and its index row in the same change.

### 7. Report

Give:
- The file path.
- Case totals: ready, partly blocked, fully blocked.
- The AC coverage summary.
- The open gaps, grouped by owner.
- Attachments or links that couldn't be opened (not reviewed).
- Behaviour already covered elsewhere (not written again), and cases merged within the suite.
- The context files created or updated.

Suggest `test-script-generator` as the next step.
