---
title: How to enable Sensitivity Labels for SharePoint, M365 groups, Teams and Office apps in M365
nav_order: 10
---
# Enable Sensitivity Labels for SharePoint, M365 groups, Teams and Office apps in M365

Enabling the use of Sensitivity labels for SharePoint sites, Teams, M365 groups and SharePoint/OneDrive (Office Online) is a 3-step process. 

## Enabling the use of Sensitivity labels for M365 groups in Entra ID:

1. Install the MS Graph Module and connect using Global Admin. Ensure you have at least one Entra ID P1 license:

```
Install-Module Microsoft.Graph -Scope CurrentUser

Install-Module Microsoft.Graph.Beta -Scope CurrentUser

Connect-MgGraph -Scopes "Directory.ReadWrite.All
```


2. Fetch the current group settings for the Microsoft Entra organization and display the current group settings:

```
$grpUnifiedSetting = Get-MgBetaDirectorySetting | Where-Object { $_.Values.Name -eq "EnableMIPLabels" }

$grpUnifiedSetting.Values
```

If the sensitivity label was enabled previously, you see EnableMIPLabels = True. In this case, you don't need to do anything. (_Proceed to the next section_)

3. If no group settings were created for this Microsoft Entra organization, you get an empty screen. In this case, you must change the default settings and create a new settings object using a settings template:

```
$TemplateId = (Get-MgBetaDirectorySettingTemplate | where { $_.DisplayName -eq "Group.Unified" }).Id

$Template = Get-MgBetaDirectorySettingTemplate | where -Property Id -Value $TemplateId -EQ
```

4. Create an object that contains values to be used to enable sensitivity labels.

```
$params = @{
   templateId = "$TemplateId"
   values = @(
      @{
         name = "EnableMIPLabels"
         value = "True"
      }
   )
}
```

5. Create the directory setting:

```
New-MgBetaDirectorySetting -BodyParameter $params

$Setting = Get-MgBetaDirectorySetting | where { $_.DisplayName -eq "Group.Unified"}

$Setting.Values
```

5. You should now see **EnableMIPLabels = True**. Update the value by using the below cmdlet:

```
Update-MgBetaDirectorySetting -DirectorySettingId $Setting.Id -BodyParameter $params
```

## Enable sensitivity labels for containers and synchronize labels

1. Install the Exchange Online module and connect to the Security and Compliance PowerShell:

```
Install-Module -Name ExchangeOnlineManagement -Scope CurrentUser

Set-ExecutionPolicy RemoteSigned

Connect-IPPSSession
```

2. Run the following command to ensure your sensitivity labels can be used with Microsoft 365 groups:

```
Execute-AzureAdLabelSync
```

Wait up to 24 hrs and you should be able to apply Sensitivity labels to a SharePoint Sites, OneDrive files, Teams meetings, M365 groups. 

Users however will not be able to apply Sensitivity labels in M365 online apps like Word Online, Excel Online, etc. 

## Enable sensitivity labels for files in SharePoint and OneDrive

1. Download the SharePoint Module: https://www.microsoft.com/en-us/download/details.aspx?id=35588

2. Import and connect to the SPO PowerShell:

```
Import-Module Microsoft.Online.SharePoint.PowerShell -DisableNameChecking

Connect-SPOService -url "https://<your tenant name>-admin.sharepoint.com"
```

3. Enable the use of Sensitivity labels:

```
Set-SPOTenant -EnableAIPIntegration $true
```

## Considerations:

* The organization should have an active Microsoft Entra ID P1 license.
* EnableMIPLabels should be set to True in the Microsoft Graph PowerShell module.
* The sensitivity labels should be published in the Microsoft Purview portal.
* Labels should be synchronized to Microsoft Entra ID using Execute-AzureAdLabelSync cmdlet. It can take up to 24 hours after synchronization for the label to be available.
* The Sensitivity label scope must be configured for Groups & Sites.
* The group type should be Microsoft 365 group.
* The user Should have sufficient privileges to assign sensitivity labels. The user must be the group owner or at least a Groups Administrator.
* The user must be within the scope of the Sensitivity label publishing policy.
* If the label you're looking for isn't in the list:
* The label might not be published in the Microsoft Purview portal or no longer be published.
* The label might be published, but it isn't available to the user who is signed in.
