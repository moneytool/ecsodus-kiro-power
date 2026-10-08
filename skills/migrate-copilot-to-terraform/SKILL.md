---
name: migrate-copilot-to-terraform
description: Generate Terraform, retain patches and a gated runbook for an AWS Copilot app, then guide the user through the runbook one checked step at a time. Use after assess-copilot-app, when the user wants to move a Copilot app to Terraform.
---

# Migrate a Copilot app to Terraform (adopt in place)

ecsodus generates files and checks results; it never applies Terraform, deletes stacks or moves traffic. The user runs every mutating step. Follow these rules throughout:

- **Never run a command that changes AWS** (`aws cloudformation execute-change-set`, `update-stack`, `update-stack-set`, `delete-stack`, `terraform apply`, or any `copilot` deploy or delete command) unless the user explicitly approves that specific step in this conversation.
- **Never skip a check.** Every mutating step in `RUNBOOK.md` is preceded by an `ecsodus check` or `ecsodus verify-*` command. If one fails, stop and explain the failure; do not work around it.
- Recommend a non-production copy first when the report says the app uses something not yet verified on real AWS (custom domains and ACM certificates, Aurora addons, private placement with NAT, partial migrations).

## Steps

1. Make sure `inventory.json` exists and is less than 24 hours old. If not, run the `assess-copilot-app` skill again.
2. Generate the hand-off:

    ```bash
    uvx ecsodus generate inventory.json --out infra/
    ```

    By default patched templates go to the Copilot artifact bucket; use `--patch-bucket <bucket>` to choose another. The output is flat Terraform with `import` blocks, retain patches per stack, `ecsodus-manifest.json`, `REPORT.md` and `RUNBOOK.md`.
3. Tell the user to review the generated `*.tf` files before committing them: they contain task-definition environment values copied from the stacks.
4. Walk through `infra/RUNBOOK.md` section by section, running blocks from the `infra` directory. The sections are:
    1. **Freeze**: stop all Copilot deploys and pipelines for the app.
    2. **Protect**: deletion protection and backups for stateful resources.
    3. **Retain patches**: create change sets, gate each with `ecsodus check --changeset ... --manifest ecsodus-manifest.json --stack <stack>`, execute only after approval, then `ecsodus verify-retain`.
    4. **Import into Terraform**: `ecsodus verify-fresh`, then `terraform plan` / `terraform show -json`, gated by `ecsodus check plan-import.json --manifest ecsodus-manifest.json --phase import` (it rejects any create, update, delete or replace). Apply only after approval, then the steady-phase check.
    5. **Teardown of Copilot stacks**: `ecsodus verify-retain` immediately before each stack delete, in the order the runbook gives.
    6. **Verify**: a final `--phase steady` check must show no changes.
    7. **Never, after migrating**: read the list to the user.
5. Before each mutating block, show the user the exact commands and the check that guards them, and wait for an explicit "yes".
6. After the runbook, point out the deploy-tool note in step 4 of `RUNBOOK.md`: image rollouts move to the user's deploy tool, and Terraform ignores `task_definition` and `desired_count` on services.

See `references/runbook-gates.md` for what each check accepts and rejects.

## If something is blocked

Blocked workloads stay on Copilot and the report says why. Suggest the user open an issue or post in the ecsodus Discussions with the blocked reason (https://github.com/moneytool/ecsodus/discussions), without account IDs or secrets.
