# QA agent skills: setup and usage

Five AI-agent skills that cover the QA pipeline, from Jira ticket to bug report. They're framework-agnostic: everything specific to a project comes from the **Project profile** table in that repo's `AGENTS.md`.

```
Jira story ──► test-cases-generator ──► test-case doc
                                            │
                                            ▼
                               test-script-generator ──► tests + failure report
                                            │                       │
                failing tests ──► test-script-healer ──► failure report
                                                                    │
                                            bugs-reporter ◄─────────┘ ──► Jira bugs + bug tags on tests

before commit/push ──► test-script-review
```

| Skill | What it does | You'll be asked to approve |
|---|---|---|
| `test-cases-generator` | Reads a Jira story's AC and designs, and writes a test-case doc | Nothing (review the doc afterwards) |
| `test-script-generator` | Turns a test-case doc into automated tests, page objects and mocks, then runs them | The list of empty tests, before any code is written |
| `test-script-healer` | Finds out why tests fail and fixes the script. Real app bugs are reported, never hidden. | Changes to a test's preconditions |
| `test-script-review` | Reviews changed code for quality, secrets and personal data, then fixes it | Structural refactors (merging page objects, moving mocks, changing old code) |
| `bugs-reporter` | Turns app failures into Jira bugs (one per root cause, no duplicates), assigned to you | Every ticket draft, before anything is created in Jira |

Shared rules live in `skills/_shared/`: code standards, the coverage check (no duplicate tests), the failure-report format, product-context notes and source access.

---

## 1. Prerequisites

| Need | Used by | How to check or set up |
|---|---|---|
| An AI coding agent | all | Claude Code (CLI, VS Code or desktop) is recommended, because it discovers the skills automatically. Other agents work too (see section 3). |
| The repo's test toolchain | the generator, healer and review | Whatever the repo's README says (here: `npm install`, `npx playwright install --with-deps chromium`, and a filled-in `.env.qa`) |
| **Jira connector, read access** | `test-cases-generator`, `bugs-reporter` (duplicate check) | In claude.ai → Settings → Connectors, connect **Atlassian**. Without it, the skills work from pasted ticket content. |
| **Jira connector, write access** | `bugs-reporter` (creating tickets) | Same connector, authorised with permission to create issues. Without it, approved drafts are saved to a file for you to create by hand. |
| GitHub access (optional) | the generator, healer, cases generator | Only needed if you haven't cloned the frontend and backend repos locally. Either the GitHub MCP connector, or the `gh` CLI logged in (`gh auth status`). |

## 2. Set up in this repo

1. **Windows only: enable symlinks before cloning.** `CLAUDE.md`, `.claude/skills`, and `.cursor/skills` are symlinks (`AGENTS.md`, and `.agent/skills` twice).
   - Turn on **Settings → System → For developers → Developer Mode**.
   - Run `git config --global core.symlinks true`.
   - In an existing clone, run `git checkout -- CLAUDE.md .claude/skills .cursor/skills` to re-create the links.
2. **Check the links.** `CLAUDE.md` should open as the full agent instructions. `.claude/skills/` and `.cursor/skills/` should each list the five skill folders. If a link is a one-line text file, redo step 1.
3. **Reference material (optional, local only; all gitignored):**
   - `docs/app/frontend/`, `docs/app/backend/`: clone the dev repos here for locators and API schemas. Skip this, and the skills will ask which GitHub repo and branch to read instead.
   - `docs/app/design/`: the design prototype and specs, from the team.
   - `docs/testcases/`: test-case docs, from the team or from `test-cases-generator`.
   - `docs/context/INDEX.md`: product notes the skills build up. If it doesn't exist yet, create it with a `# Product context index` heading and this table header: `| Topic | File | Summary | Related |` (the skills add the rows)
4. **Start a new agent session** in the repo root. Skills are loaded when a session starts, so restart after pulling skill changes.

## 3. Run a skill

**Claude Code:** ask in plain language, and the matching skill runs. You can also call one directly by name, e.g. `/test-script-generator`.

**Cursor:** the same five skills are project skills via `.cursor/skills`. Ask in plain language, or by name (`/test-script-healer`). Cursor also loads `AGENTS.md` (the Project profile) in every session. Start a new session after the link is created or the skills change.

**Other agents** (Codex, Copilot, and others): they read `AGENTS.md` automatically. Name the skill file in your request, e.g. *"Follow `.agent/skills/test-script-healer/SKILL.md` to fix GW-1657."*

Example requests:

| Skill | Say something like |
|---|---|
| `test-cases-generator` | "Which of my tickets need test cases?" · "Write test cases for SIG-5401" |
| `test-script-generator` | "Automate docs/testcases/SIG-5401_Issuances_Layout_Cards_Search_Test_Suite.md" · "Generate scripts for SIG-5401" |
| `test-script-healer` | "GW-1657 is failing" · "Fix the failing tests in ordersfilterandsearch.spec.ts" (paste the error if you have it) |
| `test-script-review` | "Review my changes before I push" · "Review tests/v6.7/phase3/issuancedetailsoverviewtab.spec.ts" |
| `bugs-reporter` | "Raise bugs from the latest failure report for SIG-5401" · "Report the app failures" |

**What to expect:**
- **Checkpoints:** the skill stops and waits for your answer at the points in the first table. Reply in short form, e.g. "go ahead", "approve 1, 2, 4", or "approve all except 3".
- **Final report:** every skill ends with a short summary of what it did, what it skipped, and what needs your attention. Test results always come in four groups: passed · skipped (TBC) · expected failures · real failures.
- **Files it may create:**
  - test-case docs (`docs/testcases/`)
  - failure reports (`docs/failure-reports/`)
  - bug evidence (`docs/bugs_evidence/<BUG-KEY>/`): screenshots and recordings to attach to each Jira ticket
  - product notes (`docs/context/`)

  All of these are local and gitignored.
- **Nothing is committed or pushed** unless you ask.

## 4. Use the skills in another project

1. Copy the `.agent/` folder into the new repo.
2. Create the symlinks (from the repo root):
   - macOS / Linux: `ln -s AGENTS.md CLAUDE.md && mkdir -p .claude .cursor && ln -s ../.agent/skills .claude/skills && ln -s ../.agent/skills .cursor/skills`
   - Windows (Developer Mode on): `cmd /c mklink CLAUDE.md AGENTS.md`, `cmd /c mklink /D .claude\skills ..\.agent\skills`, and `cmd /c mklink /D .cursor\skills ..\.agent\skills`, then `git config core.symlinks true`
3. Write that repo's `AGENTS.md` with a **Project profile** table. Use this repo's `AGENTS.md` as the example. The skills read these rows:
   - **Stack and commands:** Stack, Target environment, Run all / one file / one test, Repeat / debug, Type-check / lint
   - **Test conventions:** Test location and naming, Test ID scheme, Required metadata, TBC / pending marker, Assertions / waits, Banned, Page objects / helpers, Shared mock module, Comments, Required comments
   - **Docs and sources:** Test-case docs, Frontend / backend / design sources, Product context, Failure reports, Bug evidence
   - **Tickets:** Ticket system, Ticket intake, Bug tickets, Bug tag on test
   - **Scope:** Out of scope (behaviour the team doesn't test, e.g. keyboard accessibility)
   - **Safety:** Run side effects, Protected, Compliance

   If a row is missing, the skill infers what it can, tells you its assumption, and suggests adding the row.
4. Add the local folders to `.gitignore`: `docs/testcases/`, `docs/context/`, `docs/failure-reports/`, `docs/bugs_evidence/`, and any cloned dev repos.

## 5. Troubleshooting

| Problem | Fix |
|---|---|
| Skills don't appear, or `/skill-name` isn't found | Start a new session in the repo root. On Windows, check that `.claude/skills` is a real symlink, not a text file (section 2, step 1). |
| "Jira tools unavailable" or "not authorised" | Connect the Atlassian connector (section 1). Until then, paste the ticket content into the chat. |
| `bugs-reporter` saves drafts instead of creating tickets | The connector only has read access. Re-authorise it with permission to create issues. |
| The skill asks for a GitHub repo and branch | Expected when the dev repos aren't cloned locally. Answer with `owner/name` and the release branch, or clone them into `docs/app/`. |
| `gh` errors | Run `gh auth login`, then check with `gh auth status`. |
| Worried about test runs posting to the QA Platform | The skills turn that off for their own verification runs (Run side effects row). Your normal runs are unaffected. |
