# Deploy the control plane

## Outcome

You have deployed the SDAF deployer and SAP library. If selected, you also have
the configuration Web application infrastructure and software.

## Before you begin

Complete [Bootstrap the Azure DevOps project](02-00-bootstrap.md). Confirm that
`SDAF-General` and `SDAF-<control-plane>` contain valid identity, subscription,
agent, and version values.

## Inputs

- Deployer configuration name:
  `ENV-LOCA-VNET-INFRASTRUCTURE`.
- Library configuration name: `ENV-LOCA-SAP_LIBRARY`.
- Control-plane environment name, such as `MGMT-WEEU-DEP00`.
- `WORKSPACES/DEPLOYER/<deployer>/<deployer>.tfvars`.
- `WORKSPACES/LIBRARY/<library>/<library>.tfvars`.
- Decisions for the Web application, self-hosted execution, reset, and test
  parameters.

## Prepare configuration

Pipeline `22-sample-deployer-configuration.yml` can create deployer and library
examples when the target files do not exist. It also commits the files and
rewrites defaults in selected pipeline wrappers.

1. Create a review branch in the customer configuration repository.
2. Queue `22-sample-deployer-configuration.yml` with the approved environment,
   region, network, and optional-service choices. The pipeline creates or
   preserves the control-plane files and pushes configuration-repository
   commits.
3. Review every generated Terraform value. Replace sample address spaces, DNS
   labels, resource choices, identity settings, and subscription values with
   approved production values.
4. Review the wrapper-default changes committed by pipeline `22`. Confirm that
   they match the intended deployer, library, workload-zone, and region names.
5. Merge the reviewed configuration through your normal branch policy.

## What the automation does

`pipelines/01-deploy-control-plane.yml` is a wrapper. It loads variable group
`SDAF-<environment>` and extends
`deploy/pipelines/01-deploy-control-plane.yaml` from `sap-automation`.

The core template prepares the execution agent, stores deployment credentials
in Key Vault, deploys the control plane, and optionally builds and deploys the
configuration Web application. Repository checkouts map the core repository to
`/sap-automation` and this repository to `/config`.

## Review before execution

The default wrapper enables both Web application options. Set them to `false`
when they are not approved. The `reset` option forces reinstallation and the
pipeline text warns that it can require multiple runs; do not use it as a
general retry option.

> [!WARNING]
> The wrapper labels `test` as a no-change option, but the validated core
> template does not pass that parameter to the deployment scripts. Do not use
> `test: true` as a plan or safety gate. Treat every run as state-changing.

## Run

1. Review the approved configuration commit, subscription, service connection,
   selected agent, and optional features. Record the approval evidence.
2. Queue `01-deploy-control-plane.yml` with the exact reviewed deployer,
   library, and environment names. The pipeline begins state-changing
   preparation and deployment.
3. Monitor **Prepare the self hosted agent**, **Save the Deployment
   Credentials**, and **Deploy the control plane**. If enabled, also monitor
   **Deploy SAP configuration Web App**.

## Configure the optional Web application

The Learn-supported journey can provision Web application infrastructure
during control-plane deployment and deploy the application software from the
same core template.

1. Create or verify the Microsoft Entra app registration and store its client
   secret securely.
2. Set the approved app registration and Web application values in the
   control-plane variable group.
3. Enable both Web application parameters when queuing pipeline `01`.
4. After deployment, complete the application registration redirect URI and
   subscription Reader role steps in the
   [Learn article](https://learn.microsoft.com/azure/sap/automation/configure-devops?tabs=linux#deploy-the-control-plane-web-application).
5. Verify the Web application opens and can resolve the Azure DevOps project,
   repository, pipelines, and variable group.

## Validate

1. Verify the deployer and SAP library resource groups and expected resources in
   Azure.
2. Verify the remote Terraform state keys for the deployer and library are
   present in the approved state storage account.
3. Verify the deployer Key Vault and storage access through the pipeline
   identity.
4. Verify the selected agent pool is available for later workload-zone and
   installation stages.
5. If enabled, open the configuration Web application and verify that its
   project, repository, pipeline, and variable-group settings are correct.

## If it fails

Preserve the configuration commit, state, and failed run logs. Correct the
first failed stage, then rerun with the same identifiers. Do not enable `reset`
unless release guidance requires reinstallation and you have reviewed its
effects.

## Next step

[Deploy a workload zone](04-00-workload-zone.md).
