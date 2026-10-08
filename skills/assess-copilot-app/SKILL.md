---
name: assess-copilot-app
description: Assess an AWS Copilot CLI app for migration to Terraform, read-only. Use when someone asks whether they can move off Copilot, what deleting Copilot stacks would destroy, or how ready an app is to migrate.
---

# Assess a Copilot app (read-only)

ecsodus only calls `Describe`, `List`, `Get` and `Lookup` AWS APIs, and never reads secret values. Nothing in this skill changes AWS.

1. Confirm the context with the user:
    - the Copilot app name (`copilot app ls` lists them if the Copilot CLI is still installed),
    - the AWS profile and region the app lives in,
    - whether every environment should be assessed, or only some (`--env`, repeatable).
2. Check the tool runs: `uvx ecsodus --help`. If `uvx` is missing, use `pipx run ecsodus --help`, or ask the user to install uv (https://docs.astral.sh/uv/). ecsodus needs Python 3.11 or newer.
3. Take the inventory:

    ```bash
    uvx ecsodus inventory --app <app> --profile <profile> --region <region> -o inventory.json
    ```

    Add `--env <name>` to limit environments, and `--keep-on-copilot <env>/<workload>` for any workload the user wants to leave on Copilot.
4. Write the report:

    ```bash
    uvx ecsodus report inventory.json -o REPORT.md
    ```

5. Read `REPORT.md` and summarise it for the user:
    - the verdict, and how many stacks hand off versus stay on Copilot (and why),
    - for each stack, what a plain stack delete **without** retain patches would destroy,
    - every workload marked blocked, with the reason the report gives,
    - the zero-risk baseline the report starts with (keep the CloudFormation).
6. Explain the teardown traps in plain words. Use exactly these four facts (details in `references/copilot-teardown-traps.md`), and do not add resource types or claims that aren't in them or in `REPORT.md`:
    1. **Custom-resource Delete handlers.** Copilot deploys Lambda-backed custom resources; when their stack is deleted they delete the ACM certificate and its DNS validation records, every Route 53 alias record for the app's custom domains, and the NS delegation record, and they empty the ELB access-logs bucket. Importing those resources into Terraform first does not stop this.
    2. **The env-controller.** Each service stack has an `EnvControllerAction` custom resource. Deleting the last service that needs a shared feature makes it update the environment stack and remove the shared ALB, the NAT gateways or the EFS file system.
    3. **Unprotected addons.** Aurora/RDS, DynamoDB and S3 addons live in a nested stack with no `DeletionPolicy`, so deleting the parent stack deletes the data.
    4. **`aws cloudformation delete-stack --retain-resources` doesn't help.** It only works on stacks already in `DELETE_FAILED`.

    The safe pattern is: retain-patch every resource in every stack (policy-only change sets), import into Terraform, then delete the stacks.

    Never suggest deleting a Copilot stack, running `copilot app delete`, `copilot env delete` or `copilot svc delete`, or using `--retain-resources` as a shortcut. If the user asks for any of these, say no, explain the traps above, and offer the safe pattern instead.
7. If the user wants to proceed, hand over to the `migrate-copilot-to-terraform` skill.

`inventory.json` and `REPORT.md` contain account IDs, resource names and task-definition environment values. Tell the user to keep them out of public repositories.
