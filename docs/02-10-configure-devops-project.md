# Configure the Azure DevOps project and control-plane artifacts

Use the SDAF PowerShell utilities to create the Azure DevOps project and the
baseline artifacts required for control-plane deployment.

## Outcome

You have an Azure DevOps project with a configuration repository, pipelines,
variable groups, service connections, an agent pool, and permissions for the
SDAF control plane.

This procedure configures Azure DevOps. It doesn't deploy the control plane.

## Before you begin

Complete [Prepare Azure DevOps prerequisites](01-00-prerequisites.md). Run this
procedure from a local workstation with Windows PowerShell and the latest Azure
CLI.

You need:

- permission to create Azure resources and Azure DevOps project assets;
- the Azure DevOps organization URL;
- tenant and control-plane subscription IDs;
- approved control-plane and region codes;
- an agent-pool name; and
- SAP support credentials when software acquisition is in scope.

## Configure the script

1. Open Windows PowerShell.
2. Copy the following script into a local `.ps1` file.
3. Replace every placeholder value with the approved value for your
   environment.
4. Set `$repo` and `$branch` to the reviewed SDAF source. The script downloads
   `SDAFUtilities` from that ref.
5. Remove `-EnableWebApp` when the configuration Web application isn't in
   scope.

```powershell
# Azure DevOps configuration
$AzureDevOpsOrganizationUrl = "https://dev.azure.com/ORGANIZATIONNAME"

# Azure infrastructure configuration
$ControlPlaneCode = "MGMT"
$ControlPlaneRegionCode = "SECE"
$Location = "swedencentral"

$ControlPlaneName = "$ControlPlaneCode-$ControlPlaneRegionCode-DEP01"
$AzureDevOpsProjectName = "SDAF-" + $ControlPlaneCode + "-" + $ControlPlaneRegionCode

$ControlPlaneSubscriptionId = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
$TenantId = "yyyyyyyy-yyyy-yyyy-yyyy-yyyyyyyyyyyy"

# SAP support credentials
$Env:SUserName = Read-Host "Enter the SAP support user ID"
$SecureSPassword = Read-Host "Enter the SAP support user password" -AsSecureString
$Env:SPassword = (New-Object System.Net.NetworkCredential("", $SecureSPassword)).Password

# Managed identity and Azure DevOps agent configuration
$MSIResourceGroupName = "SDAF-MSIs"
$AgentPoolName = "SDAF-$ControlPlaneCode-$ControlPlaneRegionCode-POOL"

# SDAF source
$repo = "Azure/sap-automation"
$branch = "main"

Remove-Module SDAFUtilities -ErrorAction SilentlyContinue

$url = "https://raw.githubusercontent.com/$repo/refs/heads/$branch/deploy/scripts/pwsh/Output/SDAFUtilities/SDAFUtilities.psm1"

Write-Host "Downloading SDAFUtilities module from $url" -ForegroundColor Green

Invoke-WebRequest -Uri $url -OutFile "SDAFUtilities.psm1"
Unblock-File -Path ".\SDAFUtilities.psm1"
Import-Module ".\SDAFUtilities.psm1"

$ManagedServiceIdentity = New-SDAFUserAssignedIdentity `
    -ManagedIdentityName "$ControlPlaneName" `
    -ResourceGroupName $MSIResourceGroupName `
    -SubscriptionId $ControlPlaneSubscriptionId `
    -Location $Location `
    -Verbose

try {
    New-SDAFADOProject `
        -AdoOrganization $AzureDevOpsOrganizationUrl `
        -AdoProject $AzureDevOpsProjectName `
        -TenantId $TenantId `
        -ControlPlaneCode $ControlPlaneCode `
        -ControlPlaneSubscriptionId $ControlPlaneSubscriptionId `
        -ControlPlaneName $ControlPlaneName `
        -AuthenticationMethod "Managed Identity" `
        -AgentPoolName $AgentPoolName `
        -ManagedIdentityObjectId $ManagedServiceIdentity.PrincipalId `
        -CreateConnections `
        -EnableWebApp `
        -GitHubRepoName $repo `
        -BranchName $branch `
        -Verbose
}
finally {
    Remove-Item Env:SUserName -ErrorAction SilentlyContinue
    Remove-Item Env:SPassword -ErrorAction SilentlyContinue
}

Write-Output "Azure DevOps project '$AzureDevOpsProjectName' created successfully."
Write-Output "Managed identity ID: $($ManagedServiceIdentity.IdentityId)"
Write-Output "Agent pool name: $AgentPoolName"
```

> [!NOTE]
> The current utility reads the SAP password from `SPassword`. The script
> converts the securely entered value only for the utility call and removes both
> credential environment variables afterward.

## Run the script

1. Run `az upgrade`, and then sign in to the intended Azure tenant.
2. Run the `.ps1` file.
3. Complete each browser authentication prompt opened by the utilities.
4. Wait for `New-SDAFADOProject` to finish before closing PowerShell.
5. Record the project URL, managed identity ID, and agent-pool name.

## Validate

In Azure DevOps, confirm that:

1. The project exists with the expected name.
2. The configuration repository contains `pipelines` and `WORKSPACES`.
3. The deployment pipelines were created.
4. `SDAF-General` and the control-plane variable group exist.
5. The expected service connections exist and target the correct subscription.
6. The agent pool exists and pipeline permissions are configured.
7. The Build Service has the repository permissions required by the pipelines.
8. No SAP password, PAT, or Azure credential appears in source control or logs.

## If it fails

Don't rerun the script before checking which resources were created. Correct
the first failed permission, identity, repository, pipeline, or connection.
Then rerun the script with the same project and control-plane names.

## Next step

[Configure workload-zone artifacts](02-20-configure-workload-zone-artifacts.md).
