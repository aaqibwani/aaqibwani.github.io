---
title: Soft and Hard Matching in Azure AD
nav_order: 9
---
# Soft and Hard Matching in Azure AD
When setting up Directory Synchronization from scratch with no existing users in the cloud, Azure AD Connect is quite straightforward. It synchronizes on-premises objects (and passwords, if you choose) to the cloud.

However, if you already have cloud-only user accounts that correspond to existing on-premises user objects, and Directory Synchronization hasn't been configured yet, the process is a bit different. 

This situation might arise if your organization previously migrated mailboxes to Office 365 using the cutover method or a third-party tool, or if users were provisioned for another Microsoft Online Service like CRM before attempting mailbox migration.

In such cases, you need to establish a matching mechanism between the on-premises accounts and the cloud ones so that Azure AD Connect recognizes them as the same user. There are two primary methods for this matching:

1. **Soft match (SMTP matching)**
2. **Hard match (by immutableID)**

### Soft match (SMTP matching)
First, you need to ensure that the UPN suffixes match between your on-premises and cloud accounts. This means that your users' sign-ins should be linked to the domain of their primary email address in both the local Active Directory (AD) and Azure AD.

When you open the properties of a user account object in AD, check the email address (the primary SMTP address for the user) and under the Account tab, the user logon name.
 The logon name for the user should be in the format username@domain.com rather than username@domain.local. 

If you do not have the option to drop down your suffix or the suffix is missing, you can add the suffix using Active Directory Domains & Trusts. Right-click Active Directory Domains and Trusts and select Properties. Under Alternative UPN Suffixes, enter your email domain name and click Add. Click OK.

In M365, you also need to make sure the sign-in name is the same as in on-premises AD.  So, the user should have username@domain.com in AAD as well, and not username@tenant.onmicrosoft.com.

Now, assuming the UPN and email addresses are matching, you can download & install Azure AD Connect. Upon running the first synchronization, SMTP matching should kick in and figure out that the on-premises accounts already have cloud counterparts existing.  When you login to the portal and view your active users again, you should see a field describing the synchronization status, and each account from the on-premises directory should read “Synced with Active Directory.”

### Hard match (by immutableID)

Sometimes soft matching fails because a pre-existing cloud account may have fields like `immutableID` already populated. In this case, you can "hard match" which involves taking the on-premises GUID, converting it to an `immutableID` for Azure AD, and writing this value directly into Azure AD. This ensures that Directory Synchronization explicitly recognizes the object.

However, before proceeding, ensure that the UPN suffixes match the primary email domain both on-premises and in the cloud. 

Once you've identified any accounts that failed to sync, you can run the following command for each affected account (make sure to fill in the variables appropriately):

1. Open Command Prompt in on-premises server, for the affected user run:

```
$guid =(Get-ADUser $ADUser).Objectguid

$immutableID=[system.convert]::ToBase64String($guid.tobytearray())
```

{: .note }
> The on-premises object values are GUIDs, whereas Microsoft Entra ID is a base64 encoded text string. So, you have to convert the GUID to Base64 string.


2. Install and connect to MS Online:

```
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned

Install-Module MSOnline

Connect-MsolService
```

3. Delete the duplicate object which is synced from AD but not matched to the cloud object (not the cloud only account)

4. Run the below for the affected user:

```
Set-MsolUser -UserPrincipalName user@domain.com -ImmutableId $immutableID
```

MS Graph

```
Get-MgUser -UserId "user@domain.com" -Property OnPremisesImmutableId, UserPrincipalName | Format-List UserPrincipalName, OnPremisesImmutableId

Update-MgUser -UserId "user@domain.com" -OnPremisesImmutableId $immutableID
```

### Disable Soft matching

To disable soft matching, use the [Update-MgDirectoryOnPremiseSynchronization](https://learn.microsoft.com/en-us/powershell/module/microsoft.graph.identity.directorymanagement/update-mgdirectoryonpremisesynchronization) Microsoft Graph PowerShell cmdlet:

```
Connect-MgGraph -Scopes "OnPremDirectorySynchronization.ReadWrite.All

$OnPremSync = Get-MgDirectoryOnPremiseSynchronization

$OnPremSync.Features.BlockSoftMatchEnabled = $true

Update-MgDirectoryOnPremiseSynchronization -OnPremisesDirectorySynchronizationId $OnPremSync.Id -Features $OnPremSync.Features
```

### Disable Hard matching

To disable hard matching, use the [Update-MgDirectoryOnPremiseSynchronization](https://learn.microsoft.com/en-us/powershell/module/microsoft.graph.identity.directorymanagement/update-mgdirectoryonpremisesynchronization) Microsoft Graph PowerShell cmdlet:

```
Connect-MgGraph -Scopes "OnPremDirectorySynchronization.ReadWrite.All

$OnPremSync = Get-MgDirectoryOnPremiseSynchronization

$OnPremSync.Features.BlockCloudObjectTakeoverThroughHardMatchEnabled = $true

Update-MgDirectoryOnPremiseSynchronization -OnPremisesDirectorySynchronizationId $OnPremSync.Id -Features $OnPremSync.Features
```

### Other objects than users

For mail-enabled groups and contacts, you can soft match based on proxyAddresses. Hard match isn't applicable since you can only update the sourceAnchor/immutableID (using PowerShell) on Users only. For groups that aren't mail-enabled, there's currently no support for soft match or hard match.

{: .note-title }
> Admin role considerations
> 
> To protect from untrusted on-premises users, Microsoft Entra ID won't match on-premises users with cloud users that have an admin role. This behavior is by default. To work around this, you can do the following steps:
>
> 1. Remove the directory roles from the cloud-only user object.
> 2. Hard-delete the quarantined object in the cloud.
> 3. Trigger a sync.
> 4. Optionally, add the directory roles back to the user object in cloud once the matching is done.
