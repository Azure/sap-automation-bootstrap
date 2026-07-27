# Deploy SAP-system infrastructure

## Outcome

You have deployed the Azure infrastructure for one SAP system in an existing
workload zone.

## Before you begin

Complete [Deploy a workload zone](04-00-workload-zone.md). Verify the
workload-zone network, DNS, Key Vault, Terraform state, and variable group.

## Inputs

- SAP-system configuration name: `ENV-LOCA-VNET-SID`.
- Workload environment name: `ENV-LOCA-VNET`.
- `WORKSPACES/SYSTEM/<sap-system>/<sap-system>.tfvars`.
- `SDAF-<workload-environment>` variable group.
- Approved SAP topology, database platform, VM sizing, storage, availability,
  and network design.

## Prepare configuration

No verified full Azure DevOps equivalent exists for GitHub configuration
workflow `04`.

1. Copy a matching SAP-system example from
   [`Azure/SAP-automation-samples`](https://github.com/Azure/SAP-automation-samples/tree/main/Terraform/WORKSPACES/SYSTEM),
   export it from the SDAF configuration Web application where applicable, or
   create it through reviewed direct editing.
2. Store the file under `WORKSPACES/SYSTEM` using the exact configuration name
   that you will pass to pipeline `03`.
3. Review the system identifier, database platform, host counts, VM sizes,
   storage, zones, subnets, load balancers, and high-availability settings.
4. Confirm the design fits the deployed workload-zone address space and the
   available regional quota.
5. Commit and approve the configuration before queuing deployment.

## What the automation does

`pipelines/03-sap-system-deployment.yml` wraps
`deploy/pipelines/03-sap-system-deployment.yaml`. The core template checks out
the core and configuration repositories, installs the configured Terraform
version, and runs the SAP-system deployment script. It uses the workload-zone
variable group and remote state information.

## Review before execution

Topology changes can create, resize, or replace infrastructure. Review the
configuration for replacement or deletion risk. Confirm that the environment
parameter is the workload environment, not the full SAP-system name.

> [!WARNING]
> A plan can expose replacement or deletion risk. Do not run the apply until
> the plan has been reviewed and approved.

## Run

1. Queue `03-sap-system-deployment.yml` with `test: true`, the reviewed
   SAP-system name, and the workload environment. `TEST_ONLY=true` reaches
   `installer.sh` or `installer_v2.sh`, which exits after Terraform plan.
2. Review the plan for replacements, deletion, topology, subscription, state,
   and sizing changes.
3. Obtain approval and retain the reviewed run link.
4. Queue the same pipeline with `test: false` and unchanged identifiers.
5. Monitor **Deploy SAP infrastructure** until the Terraform operation and
   pipeline complete.

## Validate

1. Verify the expected SAP-system resource groups, virtual machines, disks,
   load balancers, and network interfaces in Azure.
2. Verify the SAP-system Terraform state exists in the approved state storage
   account.
3. Verify host reachability from the selected deployer agent.
4. Verify the installation pipeline can resolve its inventory inputs and
   required secrets without exposing secret values.
5. Verify the deployed topology matches the reviewed design.

## If it fails

Preserve the Terraform state and run logs. Correct the first failing resource or
permission, then rerun with the same system and environment names. Do not delete
the state or use the ARM fallback pipeline to recover a normal deployment.

## Next step

[Acquire SAP software and run installation](06-00-software-and-installation.md).
