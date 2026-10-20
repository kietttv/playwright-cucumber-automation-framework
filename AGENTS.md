# Agent instructions

QA work in this repo goes through the five skills in `.cursor/skills/` (the same files as `.agent/skills/`). Read the matching `SKILL.md` and follow it. Shared rules are in `.agent/skills/_shared/`.

| Skill | Use when |
|---|---|
| `test-cases-generator` | Write or update a test-case doc from a Jira story |
| `test-script-generator` | Turn a test-case doc into automated tests |
| `test-script-healer` | A spec or scenario is failing, flaky, or broken |
| `test-script-review` | Review test changes before a commit or push |
| `bugs-reporter` | Turn app failures in a failure report into Jira bugs |

At the start of a skill run, read `docs/context/INDEX.md` when it exists, then the context files that match the task.

Nothing is committed or pushed unless the user asks.

## Project profile

| Row | Value |
|---|---|
| Stack | JavaScript (CommonJS, type-checked with JSDoc). Playwright Test and Cucumber share `config/index.js`, `pages/`, and `utils/apiClient.js`. Reports go to Allure. Node.js 22.8 or newer. |
| Target environment | Demo app when `BASE_URL` is empty (Playwright starts it). Otherwise the app at `BASE_URL` from the environment or `.env`. Browser is `TEST_BROWSER` (default `chromium`). |
| Run all / one file / one test | All: `npm test`. One Playwright file: `npx playwright test tests/ui/login.spec.js`. One Playwright test: `npx playwright test tests/ui/login.spec.js -g "logs in"`. Cucumber does not filter by feature path. One scenario: `npx cucumber-js --name "Log in with valid credentials"`. By tag: `npx cucumber-js --tags "@Smoke"`. |
| Repeat / debug | Playwright: `--repeat-each=3 --retries=0 --workers=1`. Debug: `npx playwright test <file> -g "<title>" --debug`, or `HEADLESS=false`. Cucumber: re-run that `--name`. Traces land in `traces/` when `TRACE` is on; open with `npx playwright show-trace <path>`. |
| Type-check / lint | `npm run typecheck`. `npm run lint` (ESLint, Prettier, Gherkin). `npm run check` confirms every Cucumber step is defined once and every spec loads. |
| Test location and naming | One behaviour lives in one place. New product behaviour goes in `features/<ui\|api\|db>/<area>.feature` plus `step_definitions/<area>.steps.js`, unless that flow already exists only as a Playwright spec or the user asks for a spec. Specs: `tests/ui/`, `tests/api/`, `tests/visual/`, file `*.spec.js`. Import `{ test, expect }` from `tests/fixtures.js`. Steps import `Given` / `When` / `Then` from `utils/steps.js`. |
| Test ID scheme | `TC-<number>`. Next free ID is one higher than the highest `TC-` number in `features/`, `tests/`, and `docs/testcases/`. If none exist, start at `TC-1`. Existing examples have no `TC-` id; leave them. Put the id at the start of the scenario name or Playwright test title. Do not renumber. |
| Required metadata | Gherkin: an area tag `@ui`, `@api`, or `@db`; `@Smoke` or `@Regression` when the case says so; `@jira:KEY` when a ticket exists. Every scenario needs a `Then`. Tags must stay in the allowlist in `utils/lintGherkin.js`. Playwright: case id in the title; `@smoke` in the title when it is a smoke test (lowercase, as in the existing specs). Browser specs start logged in; use `test.use({ storageState: LOGGED_OUT })` for a visitor. |
| TBC / pending marker | Do not use `test.fixme()`. Playwright: `test.skip(true, 'TBC: <gap>')` on a test whose title still has the case id; leave the body empty. Cucumber: keep the scenario and end with a Then whose body is `return 'pending';` and whose text includes the gap. Do not add a `@tbc` tag. `@quarantine` is only for a flaky test that still runs, and it needs a real `@jira:` ticket. |
| Assertions / waits | Web-first `expect` (`toHaveText`, `toBeVisible`, and so on). Specs take `expect` from `tests/fixtures.js`. Steps use `expect` from `@playwright/test`. Rely on auto-waiting. Page objects never assert. |
| Banned | `waitForTimeout`, `networkidle`, `test.fixme()`, `test.only`, `{ force: true }`, hard-coded sleeps, credentials or secrets in source, XPath, and locators built in a test. No branching in a test body. |
| Page objects / helpers | `pages/`, classes extend `BasePage`. Locators prefer `getByRole` and `getByLabel`. Actions live on the page object. Register a new page in `tests/fixtures.js` and `utils/world.js`. |
| Shared mock module | None. Do not add `page.route` mocks. Drive the demo app or `BASE_URL`. Specs create data with `createItem` / `trackItem`. Steps use `this.cleanUpItem()` or `this.addCleanup()`. `@api` scenarios use `this.api()` or `this.authedApi()` and do not open a browser. |
| Comments | No section markers. JSDoc on exported helpers and page classes, matching the existing files. No comments that restate a step or record where a string came from. |
| Required comments | None. |
| Test-case docs | `docs/testcases/<KEY>_*.md` |
| Frontend / backend / design sources | Bundled app: `demo-app/` (UI and `demo-app/server.js`). For another product, ask which GitHub repo and branch to read. Designs: `docs/app/design/` when that folder exists, otherwise ticket attachments only. |
| Product context | `docs/context/INDEX.md` |
| Failure reports | `docs/failure-reports/` |
| Bug evidence | `docs/bugs_evidence/<BUG-KEY>/` |
| Ticket system | Jira. `JIRA_BASE_URL` turns `@jira:KEY` into Allure links. This repo's `.mcp.json` only configures the Playwright test server. Without an Atlassian connector, work from pasted ticket content and save approved bug drafts to a file. |
| Ticket intake | No default project. Ask for the Jira project key, issue type, and status before searching. Do not guess a project. |
| Bug tickets | Title `[<Area>] <what's wrong>`. Issue type Bug. Link type Relates. Assign to the current user. Priorities: High, Medium, Low, Lowest. Map those names if the Jira project uses different ones. |
| Bug tag on test | Gherkin: `@jira:<BUG-KEY>` on the scenario. Playwright: `@jira:<BUG-KEY>` in the test title. Add `@quarantine` only when the test must not block merges. |
| Out of scope | Keyboard-only behaviour unless the acceptance criteria require it. Framework internals (`unit/`, CI, Docker, lint). Do not change `demo-app/` to make a test pass. Visual baselines only when the user asks (`npm run test:visual -- --update`). |
| Run side effects | Runs stay local. Allure writes `allure-results/` and does not post to a shared dashboard. No extra flag is required. `@db` scenarios run only when `DB_HOST` is set. |
| Protected | `playwright.config.js`, `cucumber.js`, `config/`, `eslint.config.js`, `package.json`, `package-lock.json`, `Dockerfile`, `docker-compose.yml`, `.github/`, `.env`, `.env.example`, `utils/lintGherkin.js`, `demo-app/`, `README.md`, `docs/GUIDE.md`, `CHANGELOG.md`, `AGENTS.md`. Skills may write `docs/testcases/`, `docs/failure-reports/`, `docs/bugs_evidence/`, and `docs/context/`. Do not install, upgrade, or remove packages. |
| Compliance | No secrets, credentials, or real personal data in tests, docs, tickets, or commits. Credentials come from `APP_USERNAME` and `APP_PASSWORD`. Test data is synthetic. |
