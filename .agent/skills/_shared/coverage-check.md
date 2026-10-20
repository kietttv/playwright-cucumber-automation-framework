# Coverage check: don't test the same thing twice

`test-cases-generator`, `test-script-generator` and `test-script-review` follow this file, so the same functionality never gets several test cases or tests. Five tests covering one behaviour waste run time and maintenance, and add noise. Test-case doc and test locations come from the Project profile.

## What counts as the same functionality

Two cases or tests cover the same functionality when they have the same:

1. **Screen or state:** the same page, panel, modal or record state (e.g. "the Orders list with no filters").
2. **Action:** the same user action, or none, for a display check (e.g. "type in the search box").
3. **Check:** the same element and the same expected result (e.g. "the placeholder reads `Search order, product`").

**Different data is a different case only if it changes the expected result,** e.g. a boundary, an empty or error state, a different role or permission, or a different product type that shows something different. Five searches that each filter the list the same way are **one** behaviour with a data list, not five cases.

## How to check (search, don't read)

1. Build a short **fingerprint** for each new case or test: the screen, the action, the element, and the key expected text (exact on-screen copy, if any).
2. **Search** for the fingerprint's **distinctive terms only** (Keep it cheap, rule 1), not by case ID or title:
   - in the existing **tests**: the exact expected text, and the page-object method with its argument (e.g. `getMetricValue('Current NAV')`)
   - in the other **test-case docs**: the exact expected text

   An element name only counts as part of an exact string or a method call, never as a search term on its own.
3. **Cases without distinctive text** (pure behaviour, e.g. "the list filters to the matching order", "sorting puts the newest first", "the panel closes on Escape"): once per batch, list only the **test titles** in the same-area test files (one short line per test), and the calls to the page-object action the case would use (e.g. `ordersPage.search(`). Compare the case against those.
4. Read only the matching tests or cases, and compare their fingerprints.

## Keep it cheap

Searches are fast, but their output costs tokens. A generic word can return hundreds of long lines.

1. **Only distinctive terms:** exact on-screen text (e.g. `Current NAV`), or a page-object method with its argument (e.g. `getMetricValue('Current NAV')`). Never search single generic words like "search", "Orders", "button", "list" or "table".
2. **Same area first:** search the test files and suite docs for the same screen or module (by file name or folder). Search the whole repo only when the screen is shared, e.g. navigation or login.
3. **One search per batch, not per case:** combine all the batch's terms into one search, and collect only which term matched which file and line.
4. **Small output:** list the matching files first. Then show only the matched lines, cut to a short width (e.g. 150 characters). Never print whole long lines or whole files.
5. **Cap:** if a term matches more than about 20 lines, it isn't distinctive. Refine it (add more of the exact text) or drop it, and don't read those results.
6. **Zero hits on distinctive text, and no matching title or action call in the same area, means distinct.** Don't read anything more for that case.

## Outcomes

| Match | Meaning | What to do |
|---|---|---|
| **Duplicate** | Same screen, action and check | Don't add it. Point to the existing case or test. |
| **Overlap** | The new one repeats a check another already makes, but also checks something new | Keep only the new part. Drop the repeated check. |
| **Distinct** | A different screen, action or check, or data that changes the result | Keep it. |

When unsure, treat it as a **possible duplicate** and ask the user. Never delete an existing test or case on your own: removing one changes coverage, so it's the user's decision.
