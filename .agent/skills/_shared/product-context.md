# Product context notes

All five skills (`test-cases-generator`, `test-script-generator`, `test-script-healer`, `test-script-review`, `bugs-reporter`) follow this file, so that product knowledge builds up across sessions instead of being rediscovered every time. It's framework-agnostic: the notes folder and index path come from the **Product context** row of the Project profile in `AGENTS.md` (or `CLAUDE.md`). If the profile has no such row, skip both steps and suggest adding one.

## What goes in a context file

Knowledge that **isn't obvious from the code or the docs**, and that a future session would otherwise have to rediscover:

- Flows and navigation paths: how to reach a screen or state, which role or data it needs.
- Business rules and their reasoning: decisions from the PO, design or ticket comments.
- Quirks and gotchas: timing, caching, environment behaviour, flaky areas and what really fixed them.
- Edge cases, and dependencies between features or modules.
- Known gaps and app-vs-design mismatches, with the ticket or gap ID and the date found.
- Useful test data shapes and account types (described, never real values).

Never write secrets, credentials, env values or real personal data (PDPA). Don't copy what the repo already records (code structure, conventions in `AGENTS.md`, test-case docs, git history): link to it instead.

## Step A: look up (before exploring)

1. Read the index in full. It's one line per file, so it's cheap. If you already read it earlier in this session (the agent instructions ask for it at session start) and haven't changed it since, don't read it again.
2. Pick every row whose topic, summary **or Related column** touches the task, including topics filed under a different feature or module.
3. Read those context files before searching the code, designs or app. Treat them as ground truth, and extend them rather than rediscovering the same behaviour.
4. If something you find contradicts a note, trust the fresh evidence, and correct the note in step B.

## Step B: record (required at the end of the workflow)

Do this before the final report, every run, even when the task feels done. Skip it only if you truly learned nothing new, and then say so in the report.

1. Write down what you learned: flows, quirks, edge cases, business rules, anything surprising.
2. **Update the existing file for the topic** instead of creating a duplicate. Create `<topic>-context.md` (lowercase, hyphenated, e.g. `orders-context.md`) only for a genuinely new topic.
3. Keep entries short and factual: one bullet per fact, under a heading per sub-area. Add the date (`YYYY-MM-DD`) and source (ticket, case ID, gap ID or file) to facts that can go stale. Remove or correct facts that turned out to be wrong.
4. **In the same change, update that file's row in the index:** topic, one-line summary, and Related topics. Add a row for a new file. A context file without an up-to-date index row is invisible to step A.
5. List the context files you created or updated in the final report.
