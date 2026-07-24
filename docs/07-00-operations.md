# Operate and remove SDAF deployments

## Outcome

You can update repository content, rotate credentials, diagnose failed runs,
and remove SDAF resources in dependency order.

## Before you begin

Back up configuration and confirm access to remote Terraform state. Review
every source ref before an update and every resource name before removal.

## Update repositories

`20-update-repositories.yml` is a standalone pipeline. It pulls core
`sap-automation` content and sample content from the configured GitHub
repositories into Azure Repos and pushes the result.

1. Record the current Azure Repos commit IDs and create a recovery branch or
   tag.
2. Queue `20-update-repositories.yml` with the reviewed source repository,
   sample repository, branch, and tag values.
3. Keep `force: false` for normal updates. The pipeline performs a Git pull and
   push for each repository.
4. Review merge conflicts and the resulting commits before running deployment
   pipelines.

> [!WARNING]
> `force: true` performs a force push. Use it only through an approved recovery
> process because it can discard Azure Repos history.

## Update wrapper pipelines

`21-update-pipelines.yml` is a standalone pipeline. It checks selected wrapper
files out of the configured bootstrap source repository and pushes them to the
target branch.

1. Back up customer changes under `pipelines`.
2. Review the source repository's `main` versions of the files enumerated in
   pipeline `21`.
3. Queue the pipeline with `force: false`.
4. Review the resulting commit before using an updated wrapper.

Pipeline `21` does not enumerate every current wrapper. It does not copy
`04-sap-software-download_v2.yml` or `07-sap-cal-installation.yml`. Review those
files separately during updates.

> [!WARNING]
> `force: true` force-pushes the selected branch and can discard target history.

## Regenerate control-plane examples

Pipeline `22-sample-deployer-configuration.yml` commits generated files and
wrapper-default replacements directly to the queued branch.

1. Create a review branch before queuing pipeline `22`.
2. Record whether the deployer and library target files already exist. The
   pipeline preserves existing files but still rewrites selected wrapper
   defaults.
3. Queue the pipeline with approved values.
4. Review every generated commit before merging.

## Rotate credentials

1. Update the Azure service connection or managed identity through the approved
   Azure DevOps and Azure process.
2. Update secret variables through Azure DevOps Library without printing their
   values.
3. Verify the affected `SDAF-<environment>` group still points to the intended
   tenant, subscription, connection, and agent.
4. Validate the rotated identity with an approved access check. For workload
   zones or SAP systems, use pipeline `02` or `03` with `test: true` to verify
   Terraform planning before an apply.

## Remove resources

Remove dependents before shared infrastructure: SAP systems, workload zones,
then the control plane.

1. Back up configuration, state, and required data.
2. Queue `10-remover-terraform.yml` for the SAP system with
   `cleanup_sap: true` and `cleanup_zone: false`.
3. Verify the SAP-system resources are removed and state reflects the completed
   Terraform destroy.
4. Queue pipeline `10` for the workload zone only after all dependent SAP
   systems are removed.
5. Verify the workload-zone resources are removed and control-plane resources
   remain.
6. Queue `12-remove-control-plane.yml` only after every dependent workload zone
   is removed.
7. Verify both control-plane removal stages complete and the intended state and
   shared resources are removed.

> [!WARNING]
> `11-remover-arm-fallback.yml` deletes resource groups through Azure Resource
> Manager and removes deployment artifacts from `WORKSPACES`. Its wrapper
> defaults enable SAP-system, workload-zone, and control-plane cleanup. Use it
> only as a reviewed last resort after Terraform removal fails.

## If it fails

Stop at the first failed dependency. Do not continue to remove a parent layer
while child resources or state remain. Preserve logs and state, then use
[Troubleshoot Azure DevOps deployments](troubleshooting.md).

## Next step

Use [Pipeline reference](pipeline-reference.md) for wrapper details or return to
the [central SDAF hub](https://github.com/Azure/sap-automation).
