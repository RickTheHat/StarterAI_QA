# StarterAI QA

**Version 0.1.2** (2026-10-27). Maintainer: Richard J Graner III. Problems or ideas: open an issue on this repo.

A small team of AI agents that help a human QA tester plan, write, run, and review
Playwright tests on any project.

You talk to one agent, the **QA Director**. It coordinates the others and asks you for approval before anything is written or run.

These are [VS Code / GitHub Copilot custom agents](https://code.visualstudio.com/docs/copilot/customization/custom-agents)
(`.agent.md` files). They do nothing on their own; VS Code loads them from a fixed folder.

## Before you start

You need, in the project you are testing:

1. **VS Code with GitHub Copilot Chat** signed in.
2. **Node.js and Playwright already installed** in the project. The agents never
   install anything. If the project has no Playwright, run this in the project root:
   ```bash
   npm init playwright@latest
   ```
3. **A `tests/<type>/` folder** such as `tests/e2e/` or `tests/api/`. The agents only
   write tests inside an existing folder of that shape. If none exists, the Director
   will propose a path and you create it.
4. **The app running somewhere you can reach**, for example `http://localhost:3000`.
   Tell the Director that URL in your first message. The agents will not start the app.
5. **Recommended: a formatter (Prettier).** It keeps every new test formatted the same way.
   See [Set up formatting](#set-up-formatting-recommended) below. You set it up once; the agents
   never install or configure it.

If the app you are testing is not a Node.js project (for example .NET, Java, or Python),
keep the Playwright tests in a small separate Node.js repo pointed at the app's URL, and
install the kit into that repo instead.

## Install (one time per project)

Clone this repo once. Then, from the root of the project you are testing, copy only the runtime kit files. Do not copy this repository's own `.github/workflows`, which validate the kit rather than the project under test.

```bash
mkdir -p .github
cp -R /path/to/StarterAI_QA/.github/agents .github/
cp /path/to/StarterAI_QA/.github/BUG_REPORT_TEMPLATE.md .github/
```

On Windows, run these commands in PowerShell:

```powershell
New-Item -ItemType Directory -Force .github | Out-Null
Copy-Item -Recurse C:\path\to\StarterAI_QA\.github\agents .github\
Copy-Item C:\path\to\StarterAI_QA\.github\BUG_REPORT_TEMPLATE.md .github\
```

The agents live in `.github/agents`, where VS Code discovers them. These commands add or
overwrite only the kit's agents and bug-report template; they leave the project's other
`.github` files untouched.

Close and reopen the project in VS Code. Open Copilot Chat, open the agent picker (the
dropdown that says "Agent"), and choose **qa-director**. It is the only agent you can
pick; the rest are hidden and the Director calls them.

**Updating:** pull this repo and run the same copy commands again. They overwrite the
agents and bug-report template and touch nothing else. Check the version line at the top of this file.

**Do not edit the copied agent files** in your project. If something should change, open
an issue on this repo so every tester gets the same fix.

## Set up formatting (recommended)

Agents are told to match your project's formatting, but only a formatter can guarantee it.
Without one, a new test can come out indented differently from the rest of the file. With
Prettier in place, the Author and Reviewer follow its config, and the Runner can check it
before a run when you explicitly approve `npm run format:check`. Do this once per project,
yourself; the agents never install or configure it.

1. **Install Prettier** in the project root:
   ```bash
   npm install --save-dev prettier
   ```
2. **Create `.prettierrc.json`** in the project root. This is a sensible default; change it to
   match your team's style:
   ```json
   {
     "tabWidth": 2,
     "useTabs": false,
     "singleQuote": true,
     "semi": true,
     "trailingComma": "all",
     "printWidth": 120,
     "endOfLine": "auto"
   }
   ```
3. **Create `.editorconfig`** so VS Code uses the same indentation while you type:
   ```ini
   root = true

   [*]
   indent_style = space
   indent_size = 2
   insert_final_newline = true
   trim_trailing_whitespace = true
   charset = utf-8
   ```
4. **Create `.prettierignore`** so generated files are skipped:
   ```
   node_modules/
   test-results/
   playwright-report/
   blob-report/
   playwright/.cache/
   playwright/.auth/
   package-lock.json
   ```
5. **Add two scripts** to `package.json`. The script name `format:check` is required; the
   Runner looks for exactly that name:
   ```json
   "scripts": {
     "format": "prettier --write \"tests/**/*.{js,jsx,ts,tsx}\"",
     "format:check": "prettier --check \"tests/**/*.{js,jsx,ts,tsx}\""
   }
   ```
   Add other files to the globs if you want them covered, for example `playwright.config.ts`.

**Day to day**

- `npm run format:check` reports files that are not formatted. It changes nothing. The
  Director can include it as a separate command in the run approval; the Runner records the
  result in the results file only when you explicitly approve it.
- `npm run format` fixes the formatting. **You** run it; the agents never do. If the Runner
  reports a formatting failure, run it, then ask the Director to re-run.
- Turn on "Format on Save" in VS Code (set Prettier as the default formatter) so files are
  fixed as you edit.

If you skip this section, everything still works. The agents match the surrounding file
as best they can, and the Runner notes "format check not configured".

For always-on formatting checks in the project under test, include `npm run format:check`
in that project's CI. This repository's own workflow checks selected kit documentation and
agent files; it is not copied into projects that use the kit.

## First conversation

Tell the Director what you want tested, in plain words. Examples:

- "The app is at http://localhost:3000. Test the login page. Valid login lands on /dashboard, bad password shows an error."
- "There is a bug: adding an item to the cart twice shows quantity 1. Help me reproduce and write a regression test."
- "Review the tests in tests/e2e/checkout.spec.ts for gaps."
- "Write manual test cases for the password reset flow. No automation yet."
- "Here is the user story and acceptance criteria for search. What is missing or contradictory before I plan tests?"
- "Check the checkout page for accessibility. Here are the axe results and what I saw with keyboard only."

The Director will:

1. Look at the project and ask only the questions that change the plan.
2. Propose a test strategy and risks, then ask you to approve the plan and test edits.
3. If tests change, have the Reviewer check the changes against the approved criteria.
4. Present the exact Playwright command, any separate format-check command, run scope,
   environment, and report path for your separate run approval. Test reports use
   `tests/reports/<YYYY-MM-DD>-<scope>.md` unless your project already has a convention.
   You approve any required folder creation.
5. Run only the explicitly approved command(s), persist the result, and verify the report.
6. Report what passed, failed, what was not covered, and what it needs from you next.

Nothing is written or executed without your yes. Test-edit and test-run approval are separate;
silence is not approval.

## Run reports

Use the project's existing report convention when one is documented. Otherwise, store each
approved run summary in `tests/reports/<YYYY-MM-DD>-<scope>.md`. The Director must get your
approval for the exact path and for creating any missing folder. Include the exact approved
commands, runtime context, pass/fail/skipped counts, available artifact paths, issues, and
status; never include secret values. Commit reports with related test changes when they are
useful project evidence.

## Who does what

| Agent            | Can it change files? | Can it run commands?                                                                    | Job                                                                |
| ---------------- | -------------------- | --------------------------------------------------------------------------------------- | ------------------------------------------------------------------ |
| qa-director      | no                   | no                                                                                      | Talks to you, plans, delegates, reports                            |
| qa-explorer      | no                   | no                                                                                      | Maps how the feature works today and where risk is                 |
| qa-test-author   | tests only           | no                                                                                      | Writes or updates Playwright tests after approval                  |
| qa-runner        | results file only    | explicitly approved commands only (Playwright and, if approved, `npm run format:check`) | Runs the approved commands                                         |
| qa-debugger      | no                   | no                                                                                      | Diagnoses failures and flaky tests from the evidence               |
| qa-reviewer      | no                   | no                                                                                      | Reviews tests for coverage gaps and bad patterns                   |
| qa-accessibility | no                   | no                                                                                      | Keyboard, screen-reader, zoom, and contrast review; drafts defects |

## Rules the agents follow

- Playwright with Node.js only. No other test framework.
- Test files go under an existing `tests/<type>/` folder only.
- No production data, no destructive actions, no secrets in output.
- No arbitrary waits, retry-masking, or swallowed errors in tests.
- New tests match the project's formatting (`.prettierrc*`, `.editorconfig`, or the surrounding file). If the project has a `format:check` script, the Director may include it as a separate approved command; the Runner reports its result only when it is approved.
- No install, config, dependency, or application-code changes. Those are yours.

VS Code asks you to approve each terminal command the Runner wants to run. Keep that on.
Do not enable auto-approve for terminal commands.

## Filing a bug

Copy `.github/BUG_REPORT_TEMPLATE.md`, fill it in, and paste it to the Director if you want a
regression test written for it. Attach the Playwright trace or screenshot when you have one.

## If something goes wrong

- **The Director says it cannot find a `tests/` folder.** Create one (`mkdir -p tests/e2e`) and tell it.
- **The Director will not run a test.** It needs the exact command and a results file path approved in your own words. Say "Approved: run `npx playwright test tests/e2e/login.spec.ts` and write results to `tests/reports/login.md`".
- **It claims a pass with no evidence.** Ask for the results file path and open it. The Runner must write one for every run.
- **`npm install --save-dev prettier` fails with `UNABLE_TO_GET_ISSUER_CERT_LOCALLY`.** Your network inspects TLS traffic with a certificate Node does not trust. Try `NODE_OPTIONS=--use-system-ca` for that one command (PowerShell: `$env:NODE_OPTIONS='--use-system-ca'; npm install --save-dev prettier`), or ask IT for the corporate root certificate and point npm at it with `npm config set cafile <path>`. Do not turn off certificate checking.
- **The results file says the format check failed.** Run `npm run format`, review the change, then ask the Director to re-run.
