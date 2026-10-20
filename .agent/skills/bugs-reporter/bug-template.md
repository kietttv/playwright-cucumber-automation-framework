# Bug ticket template

Based on the team's existing bug tickets. If the project's recent bugs use a different style, match theirs (step 4 of the skill).

## Title

Use the title format from the **Bug tickets** row of the Project profile. At minimum it's `[<Area>] <what's wrong, in plain words>`.

- **The prefix always names the area:** where a reader would look for the bug. Pick the first that fits:
  1. **A page or screen**, if the bug shows on one page: the name the app uses in its navigation or page heading, e.g. `[Order Details]`. A drawer, modal or tab belongs to its page.
  2. **A feature or flow**, if the bug isn't tied to one page, or spans several, e.g. `[Sign in]`, `[Place order]`, `[Export]`, `[Notifications]`, `[Permissions]`.
  3. **An API or service**, for backend bugs with no UI, e.g. `[Orders API]`.
- **Naming:**
  - Use the name from the ticket, its epic or the design if they give one.
  - Reuse the exact spelling of earlier tickets for the same area (check the duplicate-search results).
  - If no name fits clearly, propose one at the checkpoint and let the user confirm.
- **Extra parts only if the project uses them.** If the Bug tickets row adds a phase, release or sprint, include it the way the row describes, e.g. `[Order Details - Phase 2]`. If the row doesn't mention one, the prefix is just the area.
- **The rest is one plain sentence** that someone who hasn't seen the test can understand. Name the screen or element as users see it.
- **Common shapes:**
  - `Incorrect <element> on <screen>`, e.g. `Incorrect search box placeholder on the Orders page`
  - `<Element> is not <expected behaviour> when <action>`, e.g. `Results list is not filtered after clicking a summary card`
  - `Missing <element> in <screen>`, e.g. `Missing History section in the Order details panel`
- **Avoid:** test IDs, locators, code terms, "test fails", and vague words like "broken" or "issue with".

## Description

Exactly these sections, in this order. Preconditions is optional.

```markdown
**Preconditions:**

* <only if needed: the account type or data the steps rely on, e.g. "Prepare a completed order visible to the user.">

**Steps to Reproduce:**

1. Log in to <app>. (Use the opening step from the Bug tickets row, if it gives one.)
2. Navigate to the **<Page>** page.
3. <one action per step, UI names in bold, e.g. "Click the **Action needed** summary card.">
4. <the last step is the action or observation that shows the bug, e.g. "Observe the placeholder text inside the search bar.">

**Actual Result:**

* <what happens, as an observable fact; exact on-screen text in backticks, e.g. "The search bar placeholder displays as `Search order, product` (missing `ticker`).">

<1-actual-context.png (screen, faulty element highlighted) · 2-actual-closeup.png · or 4-recording.webm for behaviour bugs>

**Expected Result:**

* <what should happen, using "should"; exact text in backticks; end with the source in brackets, e.g. "(matching design specs and AC)">

<3-expected-design.png: the same screen and element in the design, if available>
```

## Writing rules

- **Plain, human-readable English.** Short sentences. No jargon, test IDs, locators, stack traces, code or file paths in the ticket.
- **Steps** are numbered actions a person can follow by hand. Start from logging in, and name every page, button and field as it appears on screen, in bold. Use the test-case doc's steps, rewritten for a person, not the automation code.
- **Actual and Expected** are observable facts, not "works" or "doesn't work". Quote exact text in backticks. Put one fact per bullet.
- **Evidence** (captured fresh, as the skill's step 4 describes; never the test run's raw full-page screenshot):
  - **Actual Result** gets a context screenshot with the faulty element highlighted, plus a close-up of the element. For bugs about behaviour over time (navigation, filtering, sorting, timing), it gets a short recording that starts after login instead.
  - **Expected Result** gets the same screen and element from the design, at the same size and with the same highlight, when the design has it.
  - Nothing covers the problem (no toasts or loaders), and sensitive data is masked.
- **Accounts** are described by type ("a user without the Reports permission"), never by email or password.
