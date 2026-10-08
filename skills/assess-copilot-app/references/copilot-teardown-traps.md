# Why deleting Copilot stacks is dangerous

AWS ended support for the Copilot CLI on 2026-06-12 and archived its repository on 2026-06-22. Copilot apps keep running as CloudFormation stacks. Leaving Copilot means deleting those stacks without deleting what they manage. Four things make a naive delete destructive.

## 1. Custom-resource Delete handlers

Copilot deploys Lambda-backed custom resources (13 in total; 9 have destructive Delete handlers). When their stack is deleted, they can:

- delete the ACM certificate and its DNS validation records,
- delete every Route 53 alias record for the app's custom domains,
- delete the NS delegation record in the app's hosted zone,
- empty the ELB access-logs bucket, every object version included.

Importing those resources into Terraform first does not help: the Lambda still runs on delete.

## 2. The env-controller

Each service stack has an `EnvControllerAction` custom resource. When the last service that needs a shared feature is deleted, it updates the environment stack and removes that feature: the shared ALB, the NAT gateways, or the EFS file system.

## 3. Unprotected addons

Aurora/RDS, DynamoDB and S3 addons live in a nested stack with no `DeletionPolicy`. Deleting the parent stack deletes the database.

## 4. `--retain-resources` does not help

`aws cloudformation delete-stack --retain-resources` only works on stacks already in `DELETE_FAILED`.

## The safe pattern

1. Put `DeletionPolicy: Retain` and `UpdateReplacePolicy: Retain` on every resource in every stack (nested stacks and the StackSet included), through policy-only change sets. Retain on a `Custom::*` resource also stops CloudFormation from invoking its Delete handler; this was verified on real AWS.
2. Import every resource into Terraform with its exact deployed values, so the first plan is import-only.
3. Delete the Copilot stacks. CloudFormation forgets the resources; nothing is deleted.

Sources: the ecsodus knowledge base (https://github.com/moneytool/ecsodus/tree/main/docs/knowledge) and its AWS end-to-end report (https://github.com/moneytool/ecsodus/blob/main/docs/e2e/2026-09-30-aws-e2e.md).
