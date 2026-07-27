# Deploy a workload zone

## Outcome

You have a deployed SDAF workload zone connected to the existing control plane.

## Before you begin

Complete [Deploy the control plane](03-00-control-plane.md). Verify that the
control-plane state, Key Vault, service connection, and selected agent are
available.

## Inputs

- Workload-zone configuration name:
  `ENV-LOCA-VNET-INFRASTRUCTURE`.
- Workload environment name: `ENV-LOCA-VNET`.
- Deployer environment and region code.
- `WORKSPACES/LANDSCAPE/<workload-zone>/<workload-zone>.tfvars`.
- Decision for inheriting control-plane state settings.
- `SDAF-<workload-environment>` variable group.

## Prepare configuration

No verified full Azure DevOps equivalent exists for GitHub configuration
workflow `02`.

1. Copy a matching workload-zone example from
   [`Azure/SAP-automation-samples`](https://github.com/Azure/SAP-automation-samples/tree/main/Terraform/WORKSPACES/LANDSCAPE),
   export it from the SDAF configuration Web application where applicable, or
   create it through reviewed direct editing.
2. Store the file under `WORKSPACES/LANDSCAPE` using the exact configuration
   name that you will pass to pipeline `02`.
3. Review subscription, network, DNS, peering, Key Vault, state, sizing, and
   availability-zone values. The file must describe the approved environment.
4. Complete
   [Configure workload-zone artifacts](02-20-configure-workload-zone-artifacts.md)
   when the workload environment needs its Azure DevOps variable group or
   service connection. Verify the resulting `SDAF-<workload-environment>`
   group.
5. Commit and approve the configuration before queuing deployment.

## What the automation does

`pipelines/02-sap-workload-zone.yml` wraps
`deploy/pipelines/02-sap-workload-zone.yaml`. The core template first stores
deployment credentials in the deployer Key Vault. It then runs the workload-zone
deployment script with the prepared configuration. When `inherit_settings` is
`true`, the template derives state information from the control plane.

## Review before execution

Confirm that `workload_environment_parameter` and `workload_zone` identify the
same environment. Confirm the deployer environment and region point to the
control plane already deployed.

> [!WARNING]
> The credential-storage stage runs before the plan stage. Protect the
> associated variable group and Key Vault even when `test: true`.

## Run

1. Queue `02-sap-workload-zone.yml` with `test: true` and the reviewed names.
   `TEST_ONLY=true` reaches `installer.sh` or `installer_v2.sh`, which exits
   after Terraform plan without applying it.
2. Review the plan for replacements, deletion, subscription, state, network,
   DNS, and sizing changes.
3. Obtain approval and retain the reviewed run link.
4. Queue the same pipeline with `test: false` and unchanged identifiers.
5. Monitor **Save the Deployment Credentials** and **Deploy SAP workload zone**.

## Validate

1. Verify the workload-zone resource groups, network resources, and shared
   services in Azure.
2. Verify the workload-zone Terraform state exists in the approved storage
   account and is separate from control-plane state.
3. Verify connectivity from the selected agent and deployer to workload-zone
   resources.
4. Verify DNS resolution, routing, peering, and Key Vault access required by the
   planned SAP systems.

## If it fails

Do not recreate configuration with a different name. Preserve state and logs,
correct the failing dependency, and rerun with the same identifiers. If the
credential-storage stage failed, resolve that stage before attempting the
deployment stage.

## Next step

[Deploy SAP-system infrastructure](05-00-sap-system.md).
