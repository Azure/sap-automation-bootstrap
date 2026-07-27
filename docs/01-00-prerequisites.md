# Prepare Azure DevOps prerequisites

## Outcome

You have the access, identity, network, quota, software, and agent prerequisites
required to bootstrap an SDAF Azure DevOps project.

## Before you begin

Confirm the target Azure tenant, subscriptions, Azure DevOps organization, and
project naming. Complete architecture, network, DNS, quota, cost, and SAP
software planning before you create resources.

The automated setup path requires a local workstation with Windows PowerShell,
Azure CLI updated by using `az upgrade`, and access to the current
`SDAFUtilities` module. The utilities authenticate to Azure DevOps through
`AZURE_DEVOPS_EXT_PAT` when that environment variable is present; otherwise,
they can prompt for a personal access token (PAT).

Review the current
[Microsoft Learn prerequisites and setup methods](https://learn.microsoft.com/azure/sap/automation/configure-devops?tabs=linux)
before you begin.

## Inputs

Record these values in an approved location:

- Azure DevOps organization URL and project name.
- Tenant ID and control-plane subscription ID.
- Control-plane code and name.
- Authentication method: service principal or managed identity.
- Existing managed identity IDs or service-principal credentials, when used.
- Agent pool name.
- Azure regions, network address spaces, DNS ownership, and required private
  connectivity.
- SAP user credentials and selected BOM source for software acquisition.

## Required access

The bootstrap utilities create or update Azure DevOps projects, repositories,
pipelines, variable groups, service connections, agent pools, permissions, and
a wiki. The identity running them therefore needs corresponding Azure DevOps
administration rights.

When automatic Azure connection creation is selected, the utility assigns or
uses Azure roles that include Contributor, Storage Blob Data Owner, Key Vault
Administrator, Key Vault Secrets Officer, App Configuration Data Owner, and
Network Contributor. Review these assignments with your security team and
reduce scope where your approved deployment design permits.

## Agent requirements

1. Decide whether each pipeline stage will use Microsoft-hosted execution or an
   approved self-hosted pool. Record the selected pool name.
2. Confirm that the pool is authorized for every pipeline that uses it. The
   bootstrap utility can create a pool and set pipeline permissions.
3. Confirm that self-hosted agents can reach Azure Resource Manager, the target
   subscriptions, storage, Key Vault, Azure Repos, and required SAP endpoints.
4. Confirm that the agent workspace permits repository checkout and cleanup.
   Core templates clean the workspace and use the Post Build Cleanup extension.
5. Confirm that long-running installation jobs are allowed. The standard
   installation template sets an unlimited job timeout.

After these steps, the selected execution hosts meet the documented pipeline
requirements.

## Readiness check

1. Sign in to Azure CLI with the identity that will run bootstrap. Verify that
   the intended tenant and subscriptions are active.
2. Verify Azure DevOps CLI access to the target organization. The organization
   and project list commands must return without an authentication prompt or
   authorization error.
3. Verify that required resource providers, regional VM quota, IP address
   ranges, DNS zones, and private connectivity are approved.
4. Verify that SAP credentials can access the selected media and that credentials
   will be stored as secret variables rather than committed to `WORKSPACES`.
5. Estimate the cost of deployer, networking, storage, SAP infrastructure, and
   optional Web application resources. Record the approval.
6. Record the reviewed `sap-automation` and samples refs that the Azure DevOps
   repositories will consume.

## Review before execution

PATs, service-principal secrets, and SAP credentials are sensitive. Do not put
them in pipeline YAML, Terraform variable files, command history, or logs.
Prefer short-lived bootstrap credentials and rotate them after project setup.

## If it fails

Use [Troubleshoot Azure DevOps deployments](troubleshooting.md) for PAT,
permission, service-connection, agent, quota, networking, and DNS failures.

## Next step

[Bootstrap the Azure DevOps project](02-00-bootstrap.md).
