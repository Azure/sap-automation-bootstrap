# Troubleshoot Azure DevOps deployments

Use the first failing stage and task as the diagnostic boundary. Preserve the
run URL, configuration commit, selected parameters, agent name, and relevant
Azure correlation IDs.

## PAT or Azure DevOps authorization fails

1. Confirm `AZURE_DEVOPS_EXT_PAT` is present only in the current secure process
   environment, or provide it at the bootstrap prompt.
2. Verify the PAT is active and has the project, repository, pipeline, variable
   group, service-connection, and agent-pool permissions required by the
   selected bootstrap action.
3. Verify the project build service can read repository resources and contribute
   to the configuration repository.
4. Retry only the failed bootstrap or repository operation.

## A variable group is missing

1. Identify the group loaded by the core variable template:
   `SDAF-General`, `SDAF-<control-plane>`, or
   `SDAF-<workload-environment>`.
2. Verify the group exists and is authorized for the pipeline.
3. Verify required values such as `Deployment_Configuration_Path`,
   `AZURE_CONNECTION_NAME`, subscription, tenant, `POOL` or `AGENT`, and tool
   versions.
4. Verify secret values are marked secret and are not empty.

## A service connection fails

1. Compare `AZURE_CONNECTION_NAME` with the exact service-connection name.
2. Verify the connection targets the intended tenant and subscription.
3. Verify its service principal or federated identity exists and has the
   approved Azure roles.
4. Verify the connection is authorized for the failing pipeline.

## No agent can run the job

1. Inspect the resolved `POOL`, `AGENT`, or core `this_agent` value.
2. Verify the pool exists, contains an online compatible agent, and is
   authorized for the pipeline.
3. Verify network access from that agent to Azure, Azure Repos, Key Vault,
   storage, and SAP endpoints.
4. Verify the Post Build Cleanup extension is installed.

## Repository checkout or template resolution fails

1. Verify `pipelines/resources.yml` or
   `pipelines/resources_including_samples.yml` names repositories that exist in
   the Azure DevOps project.
2. Verify the selected ref exists and the build service can read it.
3. Verify the wrapper's `extends` path exists in that core ref.
4. For software download, verify `resources_including_samples.yml` resolves both
   `sap-automation` and `sap-samples`.
5. Do not run `07-sap-cal-installation.yml` when its referenced core template is
   absent.

## Configuration is not found

1. Verify `Deployment_Configuration_Path` points to `WORKSPACES`.
2. Verify the file is in `DEPLOYER`, `LIBRARY`, `LANDSCAPE`, or `SYSTEM` as
   appropriate.
3. Verify the folder and filename match the exact pipeline parameter.
4. Verify the queued branch contains the approved configuration commit.

## Terraform state or lock fails

1. Stop concurrent pipelines for the same state.
2. Verify the state storage account, container, key, subscription, and identity.
3. Confirm whether another active operation owns the lock.
4. Preserve state before any recovery action. Do not delete state to make a
   retry proceed.

## Quota, network, or DNS validation fails

1. Identify the Azure region, VM family, address space, route, or DNS name in
   the first error.
2. Compare it with the approved configuration and current subscription quota.
3. Correct the source configuration or approved Azure prerequisite.
4. Revalidate corrected workload-zone or SAP-system configuration with pipeline
   `02` or `03` and `test: true` before applying.

## Software download fails

1. Verify the selected BOM exists in the samples repository ref.
2. Verify `S-Username` and secret `S-Password` in `SDAF-General`.
3. Verify the agent can reach SAP and the SAP library storage.
4. Set `re_download: true` only when a deliberate fresh acquisition is required.

## Installation partially completes

1. Record the first failed Ansible play and host.
2. Correct the host, inventory, media, credential, or playbook input.
3. Disable completed installation booleans.
4. Rerun only the required stages and validate their services.

## Removal fails

1. Confirm child resources are being removed before parent resources.
2. Preserve configuration and state.
3. Retry Terraform removal after correcting the first failure.
4. Use the ARM fallback only after reviewing every cleanup boolean and accepting
   resource-group deletion and `WORKSPACES` artifact removal.

## Ownership

Open issues for wrapper YAML, `WORKSPACES`, or Azure DevOps repository wiring in
[`Azure/sap-automation-bootstrap`](https://github.com/Azure/sap-automation-bootstrap).
Open issues for core templates, deployment scripts, Terraform, or Ansible in
[`Azure/sap-automation`](https://github.com/Azure/sap-automation). Report
security vulnerabilities through [SECURITY.md](../SECURITY.md), not a public
issue.
