---
name: sdaf-ado-project-bootstrap
description: |
  Set up SDAF on Azure DevOps end-to-end: create the Azure DevOps project,
  configuration repository, pipelines, cleanup task, agent pool, control-plane
  managed identity, service connections, variable groups, optional Web App,
  and onboard each SDAF workload zone. Drives the documented
  New-SDAFUserAssignedIdentity + New-SDAFADOProject + New-SDAFADOWorkloadZone
  path from docs/02-00-bootstrap.md, docs/02-10-configure-devops-project.md,
  and docs/02-20-configure-workload-zone-artifacts.md. Use when an operator
  says: "set up SDAF on Azure DevOps", "bootstrap the ADO project", "onboard
  a workload zone", "create a new SDAF DevOps project", "add SDAF-<env>
  variable group and service connection". NOT for what an individual pipeline
  does or when to run it (see sdaf-ado-pipeline-catalogue), and not for the
  GitHub Actions surface.
license: MIT
---

# SDAF — Azure DevOps project bootstrap and workload-zone onboarding

Drives the two documented PowerShell utilities that create the SDAF Azure
DevOps footprint: `New-SDAFADOProject` (project + control-plane artifacts) and
`New-SDAFADOWorkloadZone` (variable group + service connection for each extra
workload environment). Same script family, same doc pages, one operator sitting. Documented in
[`docs/02-00-bootstrap.md`](../../docs/02-00-bootstrap.md),
[`docs/02-10-configure-devops-project.md`](../../docs/02-10-configure-devops-project.md),
and [`docs/02-20-configure-workload-zone-artifacts.md`](../../docs/02-20-configure-workload-zone-artifacts.md).

## When to invoke

Trigger utterances: "set up SDAF on Azure DevOps", "bootstrap the Azure
DevOps project", "create the SDAF-MGMT-SECE project", "add / onboard the
TEST-SECE-SAP01 workload zone", "create the SDAF-<env> variable group and
workload service connection".

Do **not** invoke for: "what does pipeline `01` do?" or "which variable group
does `05` read?" (→ `sdaf-ado-pipeline-catalogue`); anything on the GitHub
Actions surface; or actually deploying the control plane, workload zone, or
SAP system (bootstrap creates artefacts; deployment is a separate hub-plugin
skill).

## Recipe

Both journeys run from Windows PowerShell with the latest Azure CLI, download
`SDAFUtilities.psm1` from the reviewed `sap-automation` ref, sign in, run the
utility, capture returned values.

### A. First-time project bootstrap

1. Confirm inputs listed at the top of
   `docs/02-10-configure-devops-project.md` (see also
   [`docs/01-00-prerequisites.md`](../../docs/01-00-prerequisites.md)):
   Azure DevOps organization URL, tenant ID, control-plane subscription ID,
   control-plane and region codes, agent-pool name, SAP support credentials,
   reviewed `sap-automation` branch, `SDAF-MSIs` resource group.
2. Copy the script in `docs/02-10-configure-devops-project.md § Configure the script` into a local `.ps1`. Replace every placeholder. Drop `-EnableWebApp` when the configuration Web App is not in scope.
3. `az upgrade`, sign in to the intended Azure tenant, then run the `.ps1`;
   complete each browser prompt the utilities open.
4. Wait for `New-SDAFADOProject` to finish. Record the project URL, the
   managed-identity ID from `New-SDAFUserAssignedIdentity`, and the
   agent-pool name.

### B. Onboarding an additional workload zone

Prerequisite: the project and control-plane MSI from A already exist.

1. Copy the script in `docs/02-20-configure-workload-zone-artifacts.md § Configure the script`. Reuse `$ControlPlaneCode`, `$ControlPlaneRegionCode`, `$ManagedIdentityName`, `$MSIResourceGroupName` from A; set `$WorkloadCode`, `$WorkloadRegionCode`, and `$WorkloadSubscriptionId`.
2. Sign in to the tenant that holds the control-plane MSI. Run the `.ps1`
   and confirm `Get-SDAFUserAssignedIdentity` returns the control-plane
   identity before `New-SDAFADOWorkloadZone` runs.

### C. Validate concretely after either journey

Walk the "Validate" section of the matching configure page
([A: `docs/02-10-configure-devops-project.md`](../../docs/02-10-configure-devops-project.md);
[B: `docs/02-20-configure-workload-zone-artifacts.md`](../../docs/02-20-configure-workload-zone-artifacts.md)):

1. Project exists with the expected name; configuration repository contains
   `pipelines` and `WORKSPACES`
   (`docs/02-00-bootstrap.md § Create the project and configuration repository`).
2. Deployment pipelines were created — canonical inventory and variable-group
   mapping is `sdaf-ado-pipeline-catalogue § Variable-group dependencies`;
   do not restate it here.
3. Sensitive values (`S-Password`, PAT, Azure credentials) are marked secret.
4. Every service connection targets the correct subscription and tenant.
5. Agent pool exists and required pipelines are authorized on it.
6. Build Service has the repository permissions the pipelines need
   (`docs/02-00-bootstrap.md § Configure service connections and permissions`).
7. No SAP password, PAT, or Azure credential appears in source control or
   pipeline logs.


## Hard rules

1. **Documented utilities only.** Drive `New-SDAFUserAssignedIdentity`,
   `New-SDAFADOProject`, `Get-SDAFUserAssignedIdentity`, and
   `New-SDAFADOWorkloadZone` as documented. `setup_ado.sh` and
   `configure_deployer.sh` are named in
   `docs/02-00-bootstrap.md § Configure the agent` as Learn-article steps —
   drive them only via that path; their internals are not documented here.
   Other ADO-setup scripts (`create_devops_artifacts.sh`,
   `New-SDAFDevopsProject.ps1`, `New-SDAFDevopsWorkloadZone.ps1`,
   `setup_devops.ps1`, `Upgrade-*`) are not documented here at all — do not
   reconstruct their behaviour.
2. **Never invent parameters, roles, or variable-group values.** The
   per-variable catalogue for `SDAF-<environment>` is not exhaustively
   documented; state the gap, do not guess.
3. **Credential hygiene.** Read SAP support credentials with
   `Read-Host -AsSecureString`, do not echo, and remove `Env:SUserName` /
   `Env:SPassword` in a `finally` block exactly as
   `docs/02-10-configure-devops-project.md § Configure the script` shows.
4. **Do not rerun a partial script blind.** Inventory what was created,
   correct the first failed dependency, then rerun with the same project and
   control-plane names (`docs/02-00-bootstrap.md § If it fails`).
5. **Reuse the workload-zone code.** Same `$WorkloadZoneCode` updates in
   place; a new code creates duplicate artefacts.
6. **`WORKSPACES/LANDSCAPE/<zone>/<zone>.tfvars` is not created here** — it
   is a prerequisite of the `02` deployment pipeline.

## What this skill does NOT do

Choose between automated and manual paths (`docs/02-00-bootstrap.md § Manual
configuration path` is a fallback the operator invokes directly); deploy the
control plane, workload zone, or any SAP system; explain individual pipeline
parameters or run order (see `sdaf-ado-pipeline-catalogue`); cover the
GitHub Actions bootstrap (`sap-automation-gh-bootstrap`); register the
deployer VM as the self-hosted agent (that happens during control-plane
deployment — `docs/02-00-bootstrap.md § Configure the agent` points 3–4); or
author / debug `SDAFUtilities.psm1` itself.

## See also

`sdaf-ado-pipeline-catalogue` — what each of the 13 pipeline wrappers does,
the variable groups they read, and the `20`/`21`/`22` level-up path.
[`docs/troubleshooting.md`](../../docs/troubleshooting.md) — first-triage
for PAT/authorization, variable groups, service connections, and agents.