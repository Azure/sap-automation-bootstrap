# Bootstrap the Azure DevOps project

## Outcome

You have an Azure DevOps project with the configuration repository, code-source
access, pipelines, cleanup task, agent pool, variable groups, service
connections, and permissions required by SDAF.

## Before you begin

Complete [Prepare Azure DevOps prerequisites](01-00-prerequisites.md). Use
[Configure Azure DevOps for SAP Deployment Automation](https://learn.microsoft.com/azure/sap/automation/configure-devops?tabs=linux)
as the supported baseline.

Choose one setup method:

- **Automated scripts** create the project and baseline artifacts by using
  `New-SDAFUserAssignedIdentity`, `New-SDAFADOProject`, and
  `New-SDAFADOWorkloadZone`.
- **Manual configuration** creates each Azure DevOps component through the
  portal and repository imports.

## Inputs

Record the Azure DevOps organization URL, project name, tenant ID,
control-plane and workload-zone subscription IDs, environment and region codes,
managed-identity resource group, agent-pool name, SAP support credentials, and
reviewed `sap-automation` branch.

## Automated script path

Run the current scripts from a local workstation with Windows PowerShell and
the latest Azure CLI.

1. [Configure the Azure DevOps project and control-plane artifacts](02-10-configure-devops-project.md).
   This page includes the complete `New-SDAFUserAssignedIdentity` and
   `New-SDAFADOProject` script.
2. [Configure workload-zone artifacts](02-20-configure-workload-zone-artifacts.md).
   This page includes the complete `Get-SDAFUserAssignedIdentity` and
   `New-SDAFADOWorkloadZone` script.
3. Return to this page and complete **Create optional control-plane samples**
   when you need starter deployer and SAP library configuration.

The scripts create Azure DevOps project artifacts, identities, variable groups,
and service connections. They don't deploy the control plane or workload zone.

### Create optional control-plane samples

1. Open the **Create Sample Deployer Configuration** pipeline, which uses
   `pipelines/22-sample-deployer-configuration.yml`.
2. Select the region and approved optional components.
3. Run the pipeline. It creates missing deployer and library examples and
   updates selected wrapper defaults on the queued branch.
4. Review every generated file and commit before merging it.

The sample pipeline creates control-plane examples only. It is not a complete
workload-zone or SAP-system configuration generator.

## Manual configuration path

Use this path when you need direct control over every Azure DevOps artifact.

### Create the project and configuration repository

1. Create a private Azure DevOps project and record its URL.
2. Import
   [`Azure/sap-automation-bootstrap`](https://github.com/Azure/sap-automation-bootstrap.git)
   into the repository that has the same name as the project.
3. If direct import is unavailable, create an empty Git repository, copy the
   bootstrap repository content into a local clone, commit it, and push it.
4. Verify `pipelines` and `WORKSPACES` exist in Azure Repos.

### Choose the code source

1. For Azure Repos-hosted code, import
   [`Azure/sap-automation`](https://github.com/Azure/sap-automation.git) as
   `sap-automation` and
   [`Azure/SAP-automation-samples`](https://github.com/Azure/SAP-automation-samples.git)
   as `sap-samples`.
2. For GitHub-hosted code, create a GitHub service connection and grant it to
   the required pipelines.
3. Verify `pipelines/resources.yml` resolves `sap-automation` and
   `pipelines/resources_including_samples.yml` resolves both `sap-automation`
   and `sap-samples`.

### Create the pipelines

1. Create an Azure Pipeline from each required YAML file under `pipelines`.
2. Create deployment pipelines for `01`, `02`, `03`, the selected `04`
   software-download wrapper, `05`, `10`, `11`, and `12`.
3. Create maintenance pipelines `20`, `21`, and optional sample pipeline `22`
   when your operating model uses them.
4. Verify each pipeline points to the customer configuration repository and the
   exact YAML path in [Pipeline reference](pipeline-reference.md).

### Install the cleanup task

1. Install
   [Post Build Cleanup](https://marketplace.visualstudio.com/items?itemName=mspremier.PostBuildCleanup)
   in the Azure DevOps organization.
2. Verify the core pipeline tasks can resolve `PostBuildCleanup@4`.

### Configure the agent

1. Create a self-hosted agent pool when the deployment design requires it, and
   grant the required pipelines access.
2. Create a PAT with the Learn-documented Agent Pools, Build, Code, and Variable
   Groups scopes. Store it securely.
3. Deploy the control plane before manually configuring its deployer VM as an
   Azure DevOps agent.
4. If automatic agent configuration did not complete, run
   `deploy/scripts/configure_deployer.sh`, reboot the deployer, and run
   `deploy/scripts/setup_ado.sh` as described in the Learn article.
5. Verify the agent is online in the selected pool.

### Configure variable groups

1. Create `SDAF-General` with `Deployment_Configuration_Path=WORKSPACES`, the
   reviewed branch and tool versions, `S-Username`, and secret `S-Password`.
2. Create `SDAF-<environment>` for each control-plane or workload environment.
3. Add the approved identity, subscription, tenant, service-connection, agent,
   state, Key Vault, and optional Web application values required by that
   environment.
4. Grant the required pipelines permission to use every group.

### Configure service connections and permissions

1. Create an Azure Resource Manager service connection for each target
   subscription and verify its credentials.
2. Grant the required pipelines access to each service connection.
3. Grant the project Build Service **Contribute** permission on the
   configuration repository because pipelines create or update files.
4. If the Web application is enabled, verify the Build Service also has the
   repository access required by the application.

## Validate

1. Open every pipeline and confirm its YAML path resolves.
2. Confirm both repository resource files resolve their declared repositories.
3. Confirm the cleanup task and selected agent are available.
4. Confirm variable groups and service connections are authorized.
5. Confirm no credential value appears in source control or pipeline logs.

## If it fails

Inventory the resources already created before rerunning an automated utility.
Correct the first failed permission, repository, connection, or agent
dependency. See [Troubleshoot Azure DevOps deployments](troubleshooting.md).

## Next step

[Deploy the control plane](03-00-control-plane.md).
