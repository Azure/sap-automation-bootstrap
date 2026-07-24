# Configure workload-zone artifacts

Use the SDAF PowerShell utilities to create the variable group and service
connection required for a new workload zone.

## Outcome

Your Azure DevOps project contains an environment-specific variable group and
service connection for the workload-zone subscription.

This procedure configures Azure DevOps artifacts. It doesn't create the
workload-zone Terraform configuration or deploy Azure infrastructure.

## Before you begin

Complete
[Configure the Azure DevOps project and control-plane artifacts](02-10-configure-devops-project.md).
Confirm that the control-plane managed identity and Azure DevOps project exist.

You need:

- the Azure DevOps organization URL and project name;
- tenant, control-plane subscription, and workload-zone subscription IDs;
- the control-plane managed identity name and resource group;
- approved control-plane, workload-zone, and region codes; and
- permission to create variable groups and service connections.

## Configure the script

1. Open Windows PowerShell.
2. Copy the following script into a local `.ps1` file.
3. Replace every placeholder value with the values used for the existing
   control plane and the new workload zone.
4. Set `$repo` and `$branch` to the same reviewed SDAF source used to configure
   the project.

```powershell
# Azure DevOps configuration
$AzureDevOpsOrganizationUrl = "https://dev.azure.com/ORGANIZATIONNAME"

# Azure infrastructure configuration
$ControlPlaneCode = "MGMT"
$ControlPlaneRegionCode = "SECE"
$Location = "swedencentral"

$ControlPlaneName = "$ControlPlaneCode-$ControlPlaneRegionCode-DEP01"
$ManagedIdentityName = "$ControlPlaneName"
$MSIResourceGroupName = "SDAF-MSIs"

$AzureDevOpsProjectName = "SDAF-" + $ControlPlaneCode + "-" + $ControlPlaneRegionCode

$ControlPlaneSubscriptionId = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
$WorkloadSubscriptionId = "zzzzzzzz-zzzz-zzzz-zzzz-zzzzzzzzzzzz"
$TenantId = "yyyyyyyy-yyyy-yyyy-yyyy-yyyyyyyyyyyy"

$WorkloadCode = "TEST"
$WorkloadRegionCode = "SECE"
$WorkloadZoneCode = "$WorkloadCode-$WorkloadRegionCode-SAP01"

# SDAF source
$repo = "Azure/sap-automation"
$branch = "main"

Remove-Module SDAFUtilities -ErrorAction SilentlyContinue

$url = "https://raw.githubusercontent.com/$repo/refs/heads/$branch/deploy/scripts/pwsh/Output/SDAFUtilities/SDAFUtilities.psm1"

Write-Host "Downloading SDAFUtilities module from $url" -ForegroundColor Green

Invoke-WebRequest -Uri $url -OutFile "SDAFUtilities.psm1"
Unblock-File -Path ".\SDAFUtilities.psm1"
Import-Module ".\SDAFUtilities.psm1"

$ManagedServiceIdentity = Get-SDAFUserAssignedIdentity `
    -ManagedIdentityName $ManagedIdentityName `
    -ResourceGroupName $MSIResourceGroupName `
    -SubscriptionId $ControlPlaneSubscriptionId `
    -Verbose

Write-Output "Managed identity ID: $($ManagedServiceIdentity.IdentityId)"

New-SDAFADOWorkloadZone `
    -AdoOrganization $AzureDevOpsOrganizationUrl `
    -AdoProject $AzureDevOpsProjectName `
    -TenantId $TenantId `
    -ControlPlaneCode $ControlPlaneCode `
    -WorkloadZoneCode $WorkloadZoneCode `
    -WorkloadZoneSubscriptionId $WorkloadSubscriptionId `
    -AuthenticationMethod "Managed Identity" `
    -ManagedIdentityObjectId $ManagedServiceIdentity.PrincipalId `
    -ManagedIdentityId $ManagedServiceIdentity.IdentityId `
    -ControlPlaneSubscriptionId $ControlPlaneSubscriptionId `
    -CreateConnections `
    -Verbose
```

## Run the script

1. Sign in to the Azure tenant that contains the control-plane managed identity.
2. Run the `.ps1` file.
3. Complete each browser authentication prompt opened by the utilities.
4. Confirm that the returned managed identity is the control-plane identity.
5. Wait for `New-SDAFADOWorkloadZone` to finish.

## Validate

In Azure DevOps, confirm that:

1. `SDAF-<workload-environment>` exists in **Pipelines** > **Library**.
2. The variable group contains the intended workload-zone subscription and
   identity values.
3. Sensitive variables are marked as secret.
4. The workload-zone service connection exists and targets the intended
   subscription.
5. The required pipelines are authorized to use the variable group and service
   connection.

The script doesn't create
`WORKSPACES/LANDSCAPE/<workload-zone>/<workload-zone>.tfvars`. Prepare that
file before you run the workload-zone deployment pipeline.

## If it fails

Confirm the project name, managed identity, subscription IDs, and Azure DevOps
permissions before rerunning the script. Reuse the same workload-zone code so
you don't create duplicate environment artifacts.

## Next step

Return to [Bootstrap the Azure DevOps project](02-00-bootstrap.md), or continue
to [Deploy the control plane](03-00-control-plane.md) when project setup is
complete.
