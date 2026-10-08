# Privacy policy

Last updated: 2026-10-07

The ecsodus Kiro power and the ecsodus CLI it uses do not collect, store or transmit any personal data or usage data to the author or any third party.

- The power consists of instructions (skills) that run inside your Kiro installation.
- The ecsodus CLI runs locally. It calls only your AWS account's read-only APIs (`Describe`, `List`, `Get`, `Lookup`) using the AWS credentials you provide, and never reads secret values.
- Files ecsodus writes (`inventory.json`, `REPORT.md`, Terraform files, `RUNBOOK.md`) stay on your machine. They can contain account IDs, resource names and task-definition environment values, so keep them out of public repositories.
- The CLI is downloaded from PyPI by `uvx` or `pipx`; PyPI's own privacy policy applies to that download.

Kiro's handling of your conversations is governed by Kiro's privacy notice, not by this power.

Questions: open an issue at https://github.com/moneytool/ecsodus/issues.
