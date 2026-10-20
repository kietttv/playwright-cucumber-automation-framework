# Shared code standards for test automation

`test-script-generator`, `test-script-healer` and `test-script-review` all follow this file. It's framework-agnostic: repo-specific values (utilities, markers, folders) come from the **Project profile** in `AGENTS.md` (or `CLAUDE.md`).

## 1. Reuse, and focused helpers

- **Reuse before you write.** Extend an existing page object, helper, util or mock before creating a new one.
- **Use one parameterised method instead of near-duplicates:** `getLinkState(label)`, not one method per link.
- **Keep each helper focused.** A parameter must be a real variation, like a label, a status or a role. If a helper needs more than about 3 parameters, or on/off flags that change its behaviour, split it into focused methods.
- **Put a helper at the lowest level that serves all its callers.** Keep it in the test file while only that file uses it. Move it to the page object or a shared helper when a second file needs it.

## 2. One source of truth, separate layers

- **Locators** live only in page objects. Tests never build selectors.
- **Expected text, URLs and test data** are named constants or data builders, defined once. Tests don't repeat string literals.
- **Mocks and fixtures** (mock response builders, fixture data, route installers) live in the shared mock module named in the profile. They never go in the test file. The test file holds only its expected-text constants, its setup helper and its tests.
- **Layers:**
  - A test states *what* it checks, by calling intent-level page-object methods (`ordersPage.search(id)`).
  - A page object knows *how* to interact with its UI, and never asserts.
  - Shared helpers and utils know nothing about any single screen.
- **Independent tests:** each test sets up its own state and data, and never relies on another test's side effects or on run order.

## 3. Simple enough for QA engineers

The code is maintained by QA engineers, not full-time developers. Easy to understand beats elegant.

- **A test reads like its test case.** Each step in the doc becomes one step block, with the doc's wording as its title and in the same order, containing a few plain calls.
- **No branching in a test body** (`if`/`else`, `switch`, ternaries that choose what to do or check, `try/catch`). A test follows one fixed path and checks one expected result. If cases differ by path or data, write separate tests.
- **Loops only for repeating the same check over a fixed list** (e.g. `for (const card of ALL_CARDS) verify…`), with a one-line body. Anything more complex goes in a clearly named helper.
- **Plain, explicit code:** descriptive names, one action or check per line, small functions, early returns instead of nested ternaries.
- **Nothing "advanced" the repo doesn't already use:**
  - chaining `map`/`filter`/`reduce` instead of a readable loop
  - heavy generics or type tricks
  - clever one-liners, dense regex or metaprogramming
  - extra abstraction layers or design patterns (factories, strategies, decorators, mixins)
- **Reuse doesn't justify complexity.** A shared helper must be as easy to read as the code it replaces.

## 4. Waits and assertions

- Use the repo's assertion and wait utilities from the profile. Never use its banned APIs.
- **No fixed sleeps** (`waitForTimeout`, `Thread.sleep`, `time.sleep`, …) and no blanket retries. Wait on a condition instead: a URL, a network response, a loader disappearing, or an element's state.
- Assertion messages say what should be true: `Holding link should be disabled for an in-progress order`.

## 5. Comments

- **Keep:**
  - The section markers the profile requires (e.g. a precondition section and a test section). Keep each marker bare: no text other than the marker itself.
  - Comments that explain **how a method or line of code works**, where that isn't obvious (a workaround, a timing reason, an algorithm, why an unusual API is used).
  - Tool directives, such as lint or type-check suppressions with their reason, and license headers.
- **Never write, and remove when found:**
  - **Implementation notes:** source-doc references, account choices, test-data or mock descriptions, behaviour notes, known mismatches, TBC lists. This applies anywhere, including inside a section-marker comment.
  - Comments on constants, fixtures, data or page objects saying where something comes from, which component it models, or what the app does.
  - Comments that restate the code, step narration (use the step construct), stale comments, change history, author notes, decorative banners that aren't required markers, and commented-out code.
- **Report, don't lose:** information that would have gone in such a comment goes into the skill's final report to the user.
