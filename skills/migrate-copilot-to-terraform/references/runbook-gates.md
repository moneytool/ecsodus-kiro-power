# ecsodus checks and what they gate

| Command | Gates | Passes only if |
| --- | --- | --- |
| `ecsodus check --changeset <files> --manifest ecsodus-manifest.json --stack <stack>` | Executing a retain-patch change set | Only policy-only `Modify` entries with `Replacement: False` (plus a nested wrapper's `TemplateURL`). An empty change set is not a pass. |
| `ecsodus check --template-diff <current> <patched>` | `update-stack-set` for the StackSet | The patched template changes the retain policies and nothing else |
| `ecsodus verify-retain --app <app> --stack <stack> [--manifest ...]` | Any stack delete | Every resource in the stack, nested stacks included, has both Retain policies |
| `ecsodus verify-fresh --manifest ecsodus-manifest.json` | The import | Account, region and stacks still match the manifest |
| `ecsodus check <plan.json> --manifest ecsodus-manifest.json --phase import` | `terraform apply` of the import | The plan has only pure imports and no-ops, and every expected import is present |
| `ecsodus check --state <state.txt> --manifest ecsodus-manifest.json` | Moving on after the import | Every import is in Terraform state |
| `ecsodus check <plan.json> --manifest ecsodus-manifest.json --phase steady` | Teardown and later changes | The plan has zero changes (or only declared `--forgotten` removals) |

`plan.json` is `terraform show -json <planfile>`; `state.txt` is `terraform state list`. Inventories and manifests older than 24 hours are refused; `--allow-stale` is for offline review only, never before a real step.

Full reference: https://github.com/moneytool/ecsodus/blob/main/docs/guides/how-it-works.md
