# ecsodus Kiro power: migrate AWS Copilot apps to Terraform

A [Kiro power](https://kiro.dev/docs/powers/) that teaches Kiro's agent to move AWS Copilot CLI apps to Terraform **in place**, without deleting production, using the open-source [ecsodus](https://github.com/moneytool/ecsodus) CLI.

AWS ended support for the Copilot CLI on 2026-06-12. Copilot apps keep running as CloudFormation stacks, but deleting those stacks naively can delete certificates, DNS records, the shared load balancer, EFS and addon databases. This power makes the agent assess the app read-only, explain what a delete would destroy, and walk the user through a runbook where every mutating step is gated by a machine check.

## What it adds

| Skill | What the agent does |
| --- | --- |
| `assess-copilot-app` | Runs `ecsodus inventory` and `ecsodus report` (read-only), summarises readiness, blocked workloads and what each stack delete would destroy |
| `migrate-copilot-to-terraform` | Runs `ecsodus generate`, then guides the user through `RUNBOOK.md` one checked step at a time, never running a mutating command without explicit approval |

Activates on phrases such as "AWS Copilot", "Copilot CLI", "migrate Copilot to Terraform" and "Copilot end of support".

## Requirements

- [uv](https://docs.astral.sh/uv/) (for `uvx ecsodus`) or pipx, and Python 3.11+
- AWS credentials with read access to the Copilot app's account (assessment is read-only)
- Terraform, for the migration steps

No MCP server and no API keys are needed: the power uses the ecsodus CLI from [PyPI](https://pypi.org/project/ecsodus/).

## Install

In the Kiro IDE: open the Powers panel, select **Add Custom Power**, choose **Import power from GitHub**, enter `https://github.com/moneytool/ecsodus-kiro-power`, and click **Install**.

Then ask Kiro something like: "Assess my AWS Copilot app `myapp` in us-east-1 for migration to Terraform."

## Status

ecsodus is alpha (v0.1). It supports Load Balanced Web Services and Backend Services, their environment, and Aurora/RDS, DynamoDB and S3 addons; other workload types are detected and reported as blocked. It passed a real AWS end-to-end run on 2026-09-30 ([report](https://github.com/moneytool/ecsodus/blob/main/docs/e2e/2026-09-30-aws-e2e.md)). See the [verified scope](https://github.com/moneytool/ecsodus/blob/main/docs/STATUS.md) before using it on production.

## Tested

Tested in Kiro IDE 1.2.37 on 2026-10-07: the power installs from GitHub, activates on Copilot deletion requests, refuses an unsafe `copilot app delete`, and explains the teardown traps correctly. Full results, failures found along the way, and screenshots: [TESTING.md](TESTING.md).

## Privacy

This power collects no data. ecsodus runs locally on your machine, talks only to your AWS account with your credentials, and sends nothing to the author. See [PRIVACY.md](PRIVACY.md).

## Support

- Bugs and feature requests: [ecsodus issues](https://github.com/moneytool/ecsodus/issues)
- Questions: [ecsodus Discussions](https://github.com/moneytool/ecsodus/discussions)
- Security reports: see [SECURITY.md](https://github.com/moneytool/ecsodus/blob/main/SECURITY.md)

## License

Apache-2.0
