---
title: Export a list of all Enterprise Applications with their permissions from Entra ID
nav_order: 19
---

# Export a list of all Enterprise Applications with their permissions from Entra ID

Exporting a list of enterprise applications and their permissions helps organizations clearly see **which apps can access their data and how**. Many integrations run quietly in the background with powerful access, so having this inventory makes it easier to spot security risks, clean up old or unused apps, and make sure permissions follow the principle of least privilege. 

It’s especially useful for catching **high‑risk permissions**—such as **Directory.ReadWrite.All**, **User.ReadWrite.All**, **Group.ReadWrite.All**, or **Application.ReadWrite.All**, which allow tenant‑wide changes without a user, and data‑sensitive access like **Mail.Read**, **Mail.ReadWrite**, **Files.Read.All**, or **Files.ReadWrite.All**, which expose emails and files across the organization. In practice, this export supports security reviews, admin‑consent decisions, audits, and incident response by turning app access into something visible, understandable, and actionable instead of an unseen risk.

# Steps

* Install the Graph PowerShell Module
```
Install-Module Microsoft.Graph -Scope CurrentUser
```

* Save the below script in .ps1 example GetEnterpriseAppPermissions_Aggregated.ps1

```
# Optional workaround if you have WAM auth issues:
# $env:MSAL_USE_WAM = "false"

Import-Module Microsoft.Graph
Connect-MgGraph -Scopes "Application.Read.All","Directory.Read.All"

# Increase capacity (Windows PowerShell 5.1 sessions hit 4096 easily with Graph)
$MaximumFunctionCount = 32768
$MaximumVariableCount = 32768

# Pull all enterprise apps (service principals)
$servicePrincipals = Get-MgServicePrincipal -All -Property "id,displayName,appId"

# Cache resource SPs (APIs) so we can translate appRoleIds -> role values and get resource names
$resourceCache = @{}

function Get-ResourceSp {
    param([string]$resourceId)

    if (-not $resourceCache.ContainsKey($resourceId)) {
        try {
            $resourceCache[$resourceId] = Get-MgServicePrincipal -ServicePrincipalId $resourceId -Property "id,displayName,appId,appRoles"
        } catch {
            $resourceCache[$resourceId] = $null
        }
    }
    return $resourceCache[$resourceId]
}

# Aggregation store:
# Key = "<clientSpId>|<PermissionType>|<ResourceDisplayName>|<ResourceAppId>"
# Value = HashSet of permission strings (dedup)
$permMap = @{}

function Add-Permission {
    param(
        [string]$clientSpId,
        [string]$permType,
        [string]$resourceDisplayName,
        [string]$resourceAppId,
        [string]$permValue
    )

    if ([string]::IsNullOrWhiteSpace($permValue)) { return }  # valid syntax [2](https://bing.com/search?q=PowerShell+%5bstring%5d%3a%3aIsNullOrWhiteSpace+syntax)

    if ([string]::IsNullOrWhiteSpace($resourceDisplayName)) { $resourceDisplayName = "<unknown resource>" }
    if ($null -eq $resourceAppId) { $resourceAppId = "" }

    $key = "$clientSpId|$permType|$resourceDisplayName|$resourceAppId"

    if (-not $permMap.ContainsKey($key)) {
        $permMap[$key] = New-Object 'System.Collections.Generic.HashSet[string]'
    }
    [void]$permMap[$key].Add($permValue)
}

# Build lookup for SP details by id
$spById = @{}
foreach ($sp in $servicePrincipals) { $spById[$sp.Id] = $sp }

foreach ($sp in $servicePrincipals) {

    # ---------------------------
    # Delegated permissions (OAuth2PermissionGrants)
    # ---------------------------
    $delegated = @()
    try {
        $delegated = Get-MgServicePrincipalOauth2PermissionGrant -ServicePrincipalId $sp.Id -All
    } catch {
        $delegated = @()
    }

    foreach ($grant in $delegated) {
        $resourceId = $grant.ResourceId
        $resourceSp = Get-ResourceSp -resourceId $resourceId

        $resourceName = $resourceId
        $resourceAppId = ""

        if ($null -ne $resourceSp) {
            $resourceName = $resourceSp.DisplayName
            $resourceAppId = $resourceSp.AppId
        }

        # grant.Scoped list of delegated scopes [3](https://learn.microsoft.com/en-us/graph/api/resources/oauth2permissiongrant?view=graph-rest-1.0)
        if (-not [string]::IsNullOrWhiteSpace($grant.Scope)) {
            $scopes = $grant.Scope -split "\s+"
            foreach ($s in $scopes) {
                Add-Permission -clientSpId $sp.Id `
                              -permType "Delegated" `
                              -resourceDisplayName $resourceName `
                              -resourceAppId $resourceAppId `
                              -permValue $s
            }
        }
    }

    # ---------------------------
    # Application permissions (App role assignments)
    # ---------------------------
    $appRoles = @()
    try {
        $appRoles = Get-MgServicePrincipalAppRoleAssignment -ServicePrincipalId $sp.Id -All
    } catch {
        $appRoles = @()
    }

    foreach ($ra in $appRoles) {
        $resourceId = $ra.ResourceId
        $resourceSp = Get-ResourceSp -resourceId $resourceId

        $resourceName = $resourceId
        $resourceAppId = ""

        if ($null -ne $resourceSp) {
            $resourceName = $resourceSp.DisplayName
            $resourceAppId = $resourceSp.AppId
        }

        # Translate AppRoleId -> role.Value (e.g. User.Readre possible [4](https://stackoverflow.com/questions/78909547/how-to-query-oauth2-permission-grants-for-service-principals-in-entra-id-using-m)
        $perm = $ra.AppRoleId
        if ($null -ne $resourceSp -and $null -ne $resourceSp.AppRoles) {
            $role = $resourceSp.AppRoles | Where-Object { $_.Id -eq $ra.AppRoleId }
            if ($null -ne $role -and -not [string]::IsNullOrWhiteSpace($role.Value)) {
                $perm = $role.Value
            }
        }

        Add-Permission -clientSpId $sp.Id `
                      -permType "Application" `
                      -resourceDisplayName $resourceName `
                      -resourceAppId $resourceAppId `
                      -permValue $perm
    }
}

# Emit rows: one row per (Enterprise App × PermissionType × Resource)
$rows = foreach ($k in $permMap.Keys) {
    $parts = $k -split "\|", 4
    $clientSpId = $parts[0]
    $permType   = $parts[1]
    $resName    = $parts[2]
    $resAppId   = $parts[3]

    $clientSp = $spById[$clientSpId]
    if ($null -eq $clientSp) { continue }

    #$permList = $permMap[$k].ToArray() | Sort-Object
    
$val = $permMap[$k]

# If it's a collection (HashSet etc.), enumerate it; if it's a string, wrap it into an array
if ($val -is [System.Collections.IEnumerable] -and -not ($val -is [string])) {
    $permList = @($val) | Sort-Object
} else {
    $permList = @($val)
}

    $joined = $permList -join ", "

    [pscustomobject]@{
        EnterpriseAppDisplayName = $clientSp.DisplayName
        EnterpriseAppAppId       = $clientSp.AppId
        PermissionType           = $permType
        ResourceDisplayName      = $resName
        ResourceAppId            = $resAppId
        Permissions              = $joined
    }
}

# Export
$timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$outFile = ".\EnterpriseApps_Permissions_ByResource_$timestamp.csv"

$rows |
    Sort-Object EnterpriseAppDisplayName, PermissionType, ResourceDisplayName |
    Export-Csv -NoTypeInformation -Encoding UTF8 -Path $outFile

Write-Host "Export complete:" $outFile
```

* Run the script

```
.\GetEnterpriseAppPermissions_Aggregated.ps1
```

# Output:

<img width="1800" height="500" alt="image" src="https://github.com/user-attachments/assets/c9947e17-4d83-47f5-a18a-b82f1ac81f12" />

