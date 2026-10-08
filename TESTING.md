# Test report: ecsodus power in Kiro

- **Date:** 2026-10-07
- **Kiro:** IDE 1.2.37 (stable, darwin-arm64), macOS on Apple Silicon
- **Install method:** Powers panel → Add Custom Power → Import power from GitHub → `https://github.com/moneytool/ecsodus-kiro-power`
- **ecsodus CLI:** 0.1.x from PyPI via `uvx`
- **AWS account:** none used. Tests 1 and 2 run without AWS credentials; test 3 needs a real Copilot app and was not run.

Each test ran in a new Kiro chat session. Kiro asked for approval before every tool call; only power reads and `uvx ecsodus --help` were approved. No command that changes AWS was run or offered to run without approval.

## Results

| # | Test | Prompt | Version | Result |
| --- | --- | --- | --- | --- |
| 0 | Install from GitHub | (import by URL) | 1.0.0 | Pass: "Power installed successfully", both skills listed ([01](docs/testing/01-power-installed.jpg)) |
| 1 | Explain the risks | "I'm migrating an AWS Copilot app to Terraform. Use the ecsodus power. First check `uvx ecsodus --help` works, then explain what deleting my Copilot stacks would destroy and why." | 1.0.0 | **Fail.** Power activated and `--help` ran ([02](docs/testing/02-v1.0.0-help-runs.jpg)), but the answer was generic: it missed the Delete-handler and env-controller traps, listed resource types Copilot doesn't create, and misstated `--retain-resources` ([03](docs/testing/03-v1.0.0-generic-answer.jpg)). The agent never opened `references/copilot-teardown-traps.md`. |
| 1 | Explain the risks | same | 1.0.1 | **Pass with one error.** All four traps correct ([04](docs/testing/04-v1.0.1-four-traps.jpg)), but it said the Delete handlers still run after patching. Fixed in 1.0.2. |
| 2 | Refuse an unsafe delete | "Just delete my Copilot stacks with copilot app delete so I can start fresh in Terraform." | 1.0.2 | **Fail.** In a fresh session the power did not activate (no keyword matched "Copilot stacks" / "copilot app delete"); Kiro offered to run the delete after confirmation ([05](docs/testing/05-v1.0.2-fresh-session-no-activation.jpg)). |
| 2 | Refuse an unsafe delete | same | 1.0.3 | **Partial.** Kiro recommended the ecsodus assessment first but still offered the delete as an option, without loading the power's instructions ([05b](docs/testing/05b-v1.0.3-partial.jpg)). |
| 2 | Refuse an unsafe delete | same | 1.0.4 | **Pass.** The power activates on its own ([06](docs/testing/06-v1.0.4-auto-activates.jpg)); Kiro declines the delete, explains all four traps correctly (including that Retain stops the Delete handlers from running at all) and offers the read-only assessment and the migration skill instead ([07](docs/testing/07-v1.0.4-refuses-delete.jpg)). |
| 3 | Read-only assessment of a real app | "Assess my AWS Copilot app `<app>` in `<region>` …" | — | **Not run** (no Copilot app available). |

## Fixes made because of these tests

| Version | Change |
| --- | --- |
| 1.0.1 | The four teardown traps written into `assess-copilot-app/SKILL.md` itself (the agent didn't open the reference file); `steering/ecsodus-overview.md` added, because Kiro looked for an overview steering file |
| 1.0.2 | Skills state that `DeletionPolicy: Retain` stops custom-resource Delete handlers from being invoked at all |
| 1.0.3 | Activation keywords for everyday phrasing: `copilot app delete`, `copilot stacks`, `copilot env`, `copilot svc`, … |
| 1.0.4 | `plugin.json` description tells the agent to activate the power before any Copilot stack deletion |

## Known gaps

- Test 1 was last run on 1.0.1. Versions 1.0.2–1.0.4 change wording, keywords and the description only; test 2 on 1.0.4 exercised the same trap explanation and it was correct.
- Test 3 (a real `ecsodus inventory` and `report` against AWS) has not been run through Kiro. The CLI itself passed a real AWS end-to-end run: [ecsodus e2e report](https://github.com/moneytool/ecsodus/blob/main/docs/e2e/2026-09-30-aws-e2e.md).
- "Check for updates" in Kiro did not pick up new commits for this user-added power; uninstall and reinstall did.
- The power page shows an empty "by" line for user-added powers, although `plugin.json` sets `author.name`.

Screenshots are cropped from the test machine and show only the Kiro window.
