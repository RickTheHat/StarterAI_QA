# StarterAI QA

**Version 0.1.0** (2026-10-26). Maintainer: Richard J Graner III. Problems or ideas: open an issue on this repo.

A small team of AI agents that help a human QA tester plan, write, run, and review
Playwright tests on any project.

You talk to one agent, the **QA Director**. It
coordinates the others and asks you for approval before anything is written or run.

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

## Install (one time per project)

Clone this repo once. Then, from the root of the project you are testing, copy the kit in:

```bash
cp -R /path/to/StarterAI_QA/.github .
```

That is the whole install. Everything in this kit lives inside one hidden folder named
`.github`, because that is the only place VS Code looks for custom agents. The folder is
hidden in Finder, which is why you copy it with the command rather than by dragging.
If your project already has a `.github` folder, the copy adds to it and changes nothing else.

Close and reopen the project in VS Code. Open Copilot Chat, open the agent picker (the
dropdown that says "Agent"), and choose **qa-director**. It is the only agent you can
pick; the rest are hidden and the Director calls them.

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
2. Propose a test strategy and risks. **You approve it.**
3. Have tests written under `tests/<type>/`. **You approve the exact command to run**
   and the path of a Markdown results file (for example `tests/reports/2026-10-26-login.md`).
4. Run the approved command and write the results file.
5. Report what passed, failed, what was not covered, and what it needs from you next.

Nothing is written or executed without your yes. Silence is not approval.

## Who does what

| Agent | Can it change files? | Can it run commands? | Job |
|---|---|---|---|
| qa-director | no | no | Talks to you, plans, delegates, reports |
| qa-explorer | no | no | Maps how the feature works today and where risk is |
| qa-test-author | tests only | no | Writes or updates Playwright tests after approval |
| qa-runner | results file only | approved command only | Runs the exact approved Playwright command |
| qa-debugger | no | no | Diagnoses failures and flaky tests from the evidence |
| qa-reviewer | no | no | Reviews tests for coverage gaps and bad patterns |
| qa-accessibility | no | no | Keyboard, screen-reader, zoom, and contrast review; drafts defects |

## Rules the agents follow

- Playwright with Node.js only. No other test framework.
- Test files go under an existing `tests/<type>/` folder only.
- No production data, no destructive actions, no secrets in output.
- No arbitrary waits, retry-masking, or swallowed errors in tests.
- No install, config, dependency, or application-code changes. Those are yours.

## Filing a bug

Copy `.github/BUG_REPORT_TEMPLATE.md`, fill it in, and paste it to the Director if you want a
regression test written for it. Attach the Playwright trace or screenshot when you have one.

## If something goes wrong

- **The Director says it cannot find a `tests/` folder.** Create one (`mkdir -p tests/e2e`) and tell it.
- **The Director will not run a test.** It needs the exact command and a results file path approved in your own words. Say "Approved: run `npx playwright test tests/e2e/login.spec.ts` and write results to `tests/reports/login.md`".
- **It claims a pass with no evidence.** Ask for the results file path and open it. The Runner must write one for every run.
