# Azure DevOps pipeline reference

This page identifies wrapper ownership and implementation ownership. A wrapper
in this repository supplies customer-facing parameters and repository wiring.
A core template in `Azure/sap-automation` implements deployment behavior.

All pipeline files in this repository use `trigger: none`.

## Repository resources

| File | Repositories | Agent paths |
| --- | --- | --- |
| `pipelines/resources.yml` | `sap-automation` | Core `/sap-automation`; self `/config` |
| `pipelines/resources_including_samples.yml` | `sap-automation`, `sap-samples` | Core `/sap-automation`; samples `/samples`; self `/config` |

Both resource files currently use `ref: main`. The two software-download
wrappers correctly use `resources_including_samples.yml`.

## Deployment wrappers

| Wrapper | Core template | Key parameters | Result and limitations |
| --- | --- | --- | --- |
| `01-deploy-control-plane.yml` | `deploy/pipelines/01-deploy-control-plane.yaml` | `deployer`, `library`, `environment`, Web application options, `use_deployer`, `reset`, `test` | Deploys control-plane resources and optional Web application. Loads `SDAF-<environment>`. `reset` can require multiple runs. The core template does not pass `test` to deployment scripts. |
| `02-sap-workload-zone.yml` | `deploy/pipelines/02-sap-workload-zone.yaml` | Workload-zone and environment names, deployer environment and region, `inherit_settings`, `test` | Stores credentials in Key Vault and deploys the workload zone. `test: true` reaches the v1 or v2 installer and exits after Terraform plan. No stage-specific configuration generator is included. |
| `03-sap-system-deployment.yml` | `deploy/pipelines/03-sap-system-deployment.yaml` | `sap_system`, `environment`, `test` | Deploys SAP-system infrastructure from prepared `SYSTEM` configuration. `test: true` reaches the v1 or v2 installer and exits after Terraform plan. No stage-specific configuration generator is included. |
| `04-sap-software-download.yml` | `deploy/pipelines/04-sap-software-download.yaml` | Combined BOM, override, environment, region, extra parameters, `re_download` | Downloads a selected combined BOM. Public selection criteria versus `_v2` are unresolved. |
| `04-sap-software-download_v2.yml` | `deploy/pipelines/04-sap-software-download.yaml` | Application, database, and kernel BOMs, platform, combined name, environment, extra parameters, `re_download` | Constructs a combined BOM and derives region from the environment. Public selection criteria versus the other wrapper are unresolved. |
| `05-DB-and-SAP-installation.yml` | `deploy/pipelines/05-DB-and-SAP-installation.yaml` | System, environment, BOM, stage booleans, extra parameters | Runs parameter validation and selected Ansible installation stages. The core job has no timeout. |
| `07-sap-cal-installation.yml` | `deploy/pipelines/07-sap-cal-installation.yaml` | System, environment, CAL product, OS and CAL options, ACSS values | The referenced core template was absent from the validated core checkout. Do not run until the selected core ref supplies a validated template. |

The deployment templates change Azure resources, Terraform state, Key Vault
content, SAP library content, or host configuration. They do not publish a
general plan artifact through these wrappers. Pipeline `01` does not pass its
`test` parameter to deployment scripts. Pipelines `02` and `03` provide
plan-only runs through `TEST_ONLY`. Use run logs, Terraform output, Azure
resources, state, and service checks as validation evidence.

## Standalone configuration and update pipelines

| Pipeline | Key parameters | Behavior |
| --- | --- | --- |
| `22-sample-deployer-configuration.yml` | Control-plane and workload names, region, optional services, deployer count, identity | Creates missing deployer and library examples, commits them, rewrites selected wrapper defaults, and pushes each change. It does not generate complete workload-zone or SAP-system configuration. |
| `20-update-repositories.yml` | Core source URL, sample source URL, branch, tag, `force` | Pulls GitHub content into the Azure Repos core and sample repositories. `force: true` force-pushes. |
| `21-update-pipelines.yml` | Bootstrap source URL, branch, `force` | Copies an enumerated set of wrapper files from source `main`, commits, and pushes. It omits `_v2` and pipeline `07`. `force: true` force-pushes. |

## Removal wrappers

| Wrapper | Core template | Destructive behavior |
| --- | --- | --- |
| `10-remover-terraform.yml` | `deploy/pipelines/10-remover-terraform.yaml` | Runs Terraform-based SAP-system and workload-zone removal according to `cleanup_sap` and `cleanup_zone`. The wrapper defaults to SAP-system cleanup enabled and workload-zone cleanup disabled. |
| `11-remover-arm-fallback.yml` | `deploy/pipelines/11-remover-arm-fallback.yaml` | Deletes selected resource groups through Azure Resource Manager and removes corresponding `WORKSPACES` deployment artifacts. All three cleanup scopes default to enabled in the wrapper. |
| `12-remove-control-plane.yml` | `deploy/pipelines/12-remove-control-plane.yaml` | Removes the control plane in a deployer-agent stage and a finalization stage. It loads the control-plane variable group and uses `AZURE_CONNECTION_NAME`. |

Use removal in reverse dependency order. Prefer Terraform removal. Treat the ARM
pipeline as a last-resort fallback, not a normal retry.

## Variable-group dependencies

- Infrastructure templates load `SDAF-General`.
- Control-plane templates load `SDAF-<control-plane>`.
- Workload-zone, SAP-system, software, installation, and workload removal
  templates load `SDAF-<workload-environment>` as applicable.
- `SDAF-General` supplies shared values including the configuration path, tool
  versions, and SAP credentials.
- Environment groups supply Azure identity, subscription, connection, agent,
  state, and Key Vault values.

See [Troubleshoot Azure DevOps deployments](troubleshooting.md) when a
dependency cannot be resolved.
