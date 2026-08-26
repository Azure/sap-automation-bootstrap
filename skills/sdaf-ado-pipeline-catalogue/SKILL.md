---
name: sdaf-ado-pipeline-catalogue
description: |
  Reference catalogue of the 13 Azure DevOps pipeline wrappers that ship in
  sap-automation-bootstrap/pipelines: what each wrapper does, its documented
  parameters and preconditions, which variable groups it reads
  (SDAF-General, SDAF-<control-plane>, SDAF-<workload-environment>), and the
  documented 20 / 21 / 22 self-update ("level-up") path. Grounded in
  docs/pipeline-reference.md and docs/07-00-operations.md. Use when an
  operator says: "which pipeline should I run for X?", "what does pipeline
  01/02/03/04/05/07/10/11/12/20/21/22 do?", "what parameters does the
  workload-zone pipeline take?", "what variable group does 05 read?",
  "explain 20 vs 21 vs 22", "how do I level up SDAF pipelines?". NOT for
  standing the ADO project up in the first place (see
  sdaf-ado-project-bootstrap), for running a deployment, or for GitHub
  Actions workflows.
license: MIT
---

# SDAF — Azure DevOps pipeline catalogue

Context primer for the 13 pipeline wrappers shipped in
[`pipelines/`](../../pipelines/). Answers "what does this pipeline do, what
inputs does it take, what does it read, and when should I use it?" using only
what `docs/pipeline-reference.md` and `docs/07-00-operations.md` document.

Two repository-resource files back the wrappers:
[`pipelines/resources.yml`](../../pipelines/resources.yml) declares
`sap-automation`; [`pipelines/resources_including_samples.yml`](../../pipelines/resources_including_samples.yml)
declares `sap-automation` and `sap-samples`. Both currently use `ref: main`.
All pipeline files use `trigger: none`
([`docs/pipeline-reference.md`](../../docs/pipeline-reference.md)).

## When to invoke

- "What does pipeline `03` do?" / "when do I run `10` vs `11` vs `12`?"
- "Which variable group does `05` read?" / "what does `SDAF-General` supply?"
- "What are the parameters for the workload-zone pipeline?"
- "How do I level up my ADO pipelines from upstream?"
- "Explain `20` vs `21` vs `22`."

Do **not** invoke for:

- Creating the Azure DevOps project, agent pool, MSI, service connections, or
  variable groups (`sdaf-ado-project-bootstrap`).
- Actually running a deployment — this skill primes context; it does not
  drive commands.
- GitHub Actions workflows.

## Deployment wrappers

Each wrapper lives in
[`pipelines/`](../../pipelines/) and `extends` a core template in
`Azure/sap-automation`. See
[`docs/pipeline-reference.md § Deployment wrappers`](../../docs/pipeline-reference.md).

| Wrapper | Purpose (documented) | Reads |
| --- | --- | --- |
| `01-deploy-control-plane.yml` | Deploys control-plane resources and optional Web App. Key params: `deployer`, `library`, `environment`, `use_deployer`, `reset`, `test`. **`test` is NOT forwarded to deployment scripts.** `reset` may need multiple runs. | `SDAF-<control-plane>` |
| `02-sap-workload-zone.yml` | Stores credentials in Key Vault and deploys the workload zone. `test: true` reaches the installer and exits after `terraform plan`. Params include `inherit_settings`. | `SDAF-<workload-environment>` |
| `03-sap-system-deployment.yml` | Deploys SAP-system infrastructure from a prepared `SYSTEM` configuration. Params: `sap_system`, `environment`, `test` (plan-only via `TEST_ONLY`). | `SDAF-<workload-environment>` |
| `04-sap-software-download.yml` | Downloads a selected combined BOM. Uses `resources_including_samples.yml`. Params: combined BOM, override, environment, region, extras, `re_download`. | `SDAF-<workload-environment>` (+ shared values from `SDAF-General`) |
| `04-sap-software-download_v2.yml` | Documented alongside `04` in `docs/pipeline-reference.md § Deployment wrappers`: constructs a combined BOM from application, database, and kernel BOMs and derives region from environment. Uses `resources_including_samples.yml`. Public selection criteria versus `04` are **unresolved in docs**. | `SDAF-<workload-environment>` (+ shared values from `SDAF-General`) |
| `05-DB-and-SAP-installation.yml` | Runs parameter validation and selected Ansible install stages. Params: system, environment, BOM, stage booleans, extras. The core job has no timeout. | `SDAF-<workload-environment>` |
| `07-sap-cal-installation.yml` | SAP CAL install wrapper. **The referenced core template was absent from the validated core checkout**; do not run until the selected core `ref` supplies a validated template ([`docs/pipeline-reference.md`](../../docs/pipeline-reference.md)). | `SDAF-<workload-environment>` |

**Validation evidence:** deployment templates change Azure resources,
Terraform state, Key Vault content, SAP library content, or host
configuration. They do not publish a general plan artefact through these
wrappers. Use run logs, Terraform output, Azure resources, state, and
service checks. Pipeline `01`'s `test` parameter does **not** drive a dry run.
`02` and `03` provide plan-only through `TEST_ONLY`.

## Removal wrappers

Reverse-dependency order. Prefer Terraform removal; ARM fallback is
last-resort, not a normal retry ([`docs/pipeline-reference.md § Removal
wrappers`](../../docs/pipeline-reference.md)).

| Wrapper | Documented behaviour |
| --- | --- |
| `10-remover-terraform.yml` | Terraform-based removal of the SAP system and workload zone per `cleanup_sap` / `cleanup_zone`. Wrapper defaults: SAP-system cleanup enabled, workload-zone cleanup disabled. |
| `11-remover-arm-fallback.yml` | Deletes selected resource groups via ARM and removes corresponding `WORKSPACES` artefacts. All three cleanup scopes default to enabled in the wrapper. Last resort. |
| `12-remove-control-plane.yml` | Removes the control plane in a deployer-agent stage and a finalization stage. Loads the control-plane variable group; uses `AZURE_CONNECTION_NAME`. |

## Standalone configuration and update pipelines — the 20 / 21 / 22 level-up path

Documented in [`docs/pipeline-reference.md § Standalone configuration and
update pipelines`](../../docs/pipeline-reference.md) and
[`docs/07-00-operations.md`](../../docs/07-00-operations.md).

| Pipeline | Behaviour |
| --- | --- |
| `20-update-repositories.yml` | Pulls GitHub content into the Azure Repos core and sample repositories. Params: core source URL, sample source URL, branch, tag, `force`. `force: true` force-pushes. |
| `21-update-pipelines.yml` | Copies an enumerated set of wrapper files from source `main` into the bootstrap repository, commits, pushes. **Omits `_v2` and `07`.** `force: true` force-pushes. |
| `22-sample-deployer-configuration.yml` | Creates missing deployer and library examples, commits them, rewrites selected wrapper defaults, pushes each change. Does **not** generate complete workload-zone or SAP-system configuration. Params include control-plane and workload names, region, optional services, deployer count, identity. |

**Level-up sequence.** `docs/07-00-operations.md` documents these pipelines
in the order `20` (refresh source refs into Azure Repos) → `21` (refresh
wrapper files in this repository) → `22` (regenerate optional deployer /
library samples). Treat that as the documented order, not a mandate the
docs enforce. Keep `force=false` unless a rollback is being executed. Re-run
dependent deploy stages only after reviewing the new configuration and
preserving state.

## Variable-group dependencies

From [`docs/pipeline-reference.md § Variable-group dependencies`](../../docs/pipeline-reference.md):

- Infrastructure templates load `SDAF-General`.
- Control-plane templates load `SDAF-<control-plane>`.
- Workload-zone, SAP-system, software, installation, and workload removal
  templates load `SDAF-<workload-environment>` as applicable.
- `SDAF-General` supplies shared values including
  `Deployment_Configuration_Path`, tool versions, and SAP credentials
  (`S-Username` / `S-Password`).
- Environment groups supply Azure identity, subscription, connection, agent,
  state, and Key Vault values.

An exhaustive per-variable catalogue for each environment group is **NOT
documented** in this repository. When an operator asks for the full field
list, state that gap and point at the two "Configure" pages that create the
groups.

## Preconditions common to every pipeline

Documented in [`docs/02-00-bootstrap.md`](../../docs/02-00-bootstrap.md) and
[`docs/02-10-configure-devops-project.md`](../../docs/02-10-configure-devops-project.md):

1. The pipeline YAML path is created as a pipeline definition in the project.
2. Repository resources in `resources.yml` /
   `resources_including_samples.yml` resolve their declared repositories.
3. The Post Build Cleanup extension (`PostBuildCleanup@4`) is installed.
4. The referenced agent pool exists, has an online compatible agent, and is
   authorized for the pipeline.
5. Every referenced variable group and service connection is authorized for
   the pipeline.

## See also

- `sdaf-ado-project-bootstrap` — creates the project, pipelines, agent pool,
  MSI, service connections, variable groups, and Web App that this catalogue
  describes.
- [`docs/troubleshooting.md`](../../docs/troubleshooting.md) — precondition
  failures (variable group missing, service connection fails, no agent,
  repository checkout fails, configuration not found).
- [`docs/07-00-operations.md`](../../docs/07-00-operations.md) — operator
  context for level-up and removal ordering.