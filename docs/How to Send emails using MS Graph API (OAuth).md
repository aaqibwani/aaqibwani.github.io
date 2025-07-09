---
title: How to Send emails using MS Graph API (OAuth)
nav_order: 6
---

# How to Send emails using MS Graph API (OAuth)
Sending emails using the Microsoft Graph API is a modern, secure, and scalable approach that stands in stark contrast to the legacy SMTP client authentication model. One of the major advantages of using the Graph API over traditional SMTP client AUTH is its support for modern authentication protocols such as OAuth 2.0. With OAuth, you can implement both delegated and application-level permissions, offering a granular level of control over what actions an application can perform on behalf of a user or service. 

# Using application-level permission to send email using Graph API

## Create an application registration in Entra ID 

* Go to Entra ID > [App Registrations](https://portal.azure.com/#view/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/~/RegisteredApps ) (using a Global Administrator or an Application Administrator)
* Select New registration. Provide a descriptive name (e.g., "GraphEmailSender"). 
* For a single-tenant solution, select Accounts in this organizational directory only.
* Redirect URI: You may leave it blank 
* Click on Register.

![image](https://github.com/user-attachments/assets/13c0ad0b-b299-49dd-9eb6-211d65c08815)

## Add Microsoft Graph API Permissions

* In the menu for your app, click on API permissions.
* Click the Add a permission button.
* Under the Microsoft APIs section, select Microsoft Graph.
* Choose Application permissions (this enables app-only/authentication).
* Scroll down and check Mail.Send 
* Click Add permissions button to add the selected permission.

![image](https://github.com/user-attachments/assets/c0585d48-eb3a-4ff7-b915-43409c2e0c6f)

![image](https://github.com/user-attachments/assets/7ddecbad-96a9-475d-af9e-bab98b7796be)

![image](https://github.com/user-attachments/assets/3daf8862-a611-4710-8a40-51542ef931d2)

## Grant Admin Consent to the Application

Since you’re using application permissions, you must grant tenant-wide consent:

* In the API permissions blade, click Grant admin consent.
* Confirm the consent when prompted.

![image](https://github.com/user-attachments/assets/83e6defd-afa4-48d0-8bc8-0b7f33b594dc)

![image](https://github.com/user-attachments/assets/54d64e69-26e2-4a9a-bb16-55177d9feb75)

![image](https://github.com/user-attachments/assets/c17d316b-8bf0-4897-8f62-cf12685eea14)


## Configure Client Credentials

* In the app’s left-hand menu, click on Certificates & secrets.
* Under Client secrets, click New client secret.
* Provide a description (e.g., "GraphAPISecret") and choose an appropriate expiration period.
* Click Add.
* The Value will be displayed. IMPORTANT: Copy this value immediately as it will be hidden as soon as you move away from the page. Value is the one highlighted in the screenshot. Do not confuse it with the Secret ID. 

![image](https://github.com/user-attachments/assets/6ba2dc2b-503b-4f51-9100-c3c50a2ded22)

![image](https://github.com/user-attachments/assets/6e2cd7cd-31ae-4dd6-b5e8-e8f8ae91440d)

## Test the mail flow

Use the below test script to verify if you're able to send emails using Graph API:

```
param(
    [Parameter(Mandatory = $true)]
    [string]$TenantId,

    [Parameter(Mandatory = $true)]
    [string]$ClientId,

    [Parameter(Mandatory = $true)]
    [string]$ClientSecret,

    [Parameter(Mandatory = $true)]
    [string]$SenderEmail,

    [Parameter(Mandatory = $true)]
    [string]$RecipientEmail
)

# 1. Get an OAuth token using client credentials
$tokenEndpoint = "https://login.microsoftonline.com/$TenantId/oauth2/v2.0/token"
$body = @{
    client_id     = $ClientId
    client_secret = $ClientSecret
    scope         = "https://graph.microsoft.com/.default"
    grant_type    = "client_credentials"
}

Write-Host "Requesting OAuth token from Azure AD..."
try {
    $tokenResponse = Invoke-RestMethod -Uri $tokenEndpoint -Method Post -Body $body -ContentType "application/x-www-form-urlencoded"
    $accessToken = $tokenResponse.access_token
    Write-Host "✔ Access token acquired." -ForegroundColor Green
} catch {
    Write-Error "Failed to obtain access token: $_"
    exit 1
}

# 2. Create the email message as a JSON payload for Microsoft Graph
$message = @{
    message = @{
        subject = "Test Email from Graph API Using Application Permissions"
        body = @{
            contentType = "Text"
            content     = "Hello, this email was sent by an application using Microsoft Graph API and app-only authentication."
        }
        toRecipients = @(
            @{
                emailAddress = @{
                    address = $RecipientEmail
                }
            }
        )
    }
    saveToSentItems = "false"
} | ConvertTo-Json -Depth 4

# 3. Send the email via Microsoft Graph API
# Use the /users/{senderMail}/sendMail endpoint.
$graphEndpoint = "https://graph.microsoft.com/v1.0/users/$SenderEmail/sendMail"
$headers = @{
    Authorization = "Bearer $accessToken"
    "Content-Type" = "application/json"
}

Write-Host "Sending email via Microsoft Graph..." -ForegroundColor Cyan
try {
    Invoke-RestMethod -Uri $graphEndpoint -Method Post -Headers $headers -Body $message
    Write-Host "✔ Email sent successfully!" -ForegroundColor Green
} catch {
    Write-Error "Failed to send email: $_"
}
```

* Save the above file as graphemail.ps1
* Open Command Prompt and navigate to the directory where the script is saved
* Run the below command with the details from your tenant

```
.\graphemail.ps1 -TenantId <> -ClientId <> -ClientSecret <> -SenderEmail adele@idiotbox.onmicrosoft.com -RecipientEmail aaqib@idiotbox.in -Verbose
```
Tenant ID: Your unique tenant identifier.
ClientId: The Application (client ID). 
ClientSecret: The value we copied when creating the client secret (not the Secret ID).

![image](https://github.com/user-attachments/assets/c2755f1f-80af-4bd5-9df6-a41a658151d3)

![image](https://github.com/user-attachments/assets/eeaa3538-9cc7-483c-8211-2a2b6eee0486)

![image](https://github.com/user-attachments/assets/d9fd4ddf-0fbf-4ee2-a2dd-cda80784e907)

## Create an Application Access Policy in Exchange Online

An Application Access Policy in Exchange Online is used to restrict what mailboxes an app can access when it's granted application permissions (i.e., it runs without a signed-in user). Without this policy, an app with permissions like Mail.Read could potentially access every mailbox in your tenant or Mail.Send could potentially Send As any mailbox which is a big no-no for security and compliance.

Hence, to limit the app’s ability to send email from only specific mailboxes, you can create an application access policy in Exchange Online:

* Connect to EXO PowerShell

```
Connect-ExchangeOnline
```

* Create the Application Policy

```
New-ApplicationAccessPolicy -AppId "your-app-client-id" -PolicyScopeGroupId "group@yourdomain.com" -AccessRight RestrictAccess -Description "Restrict app to specific mailboxes"
```
{: .note }

>Use a user mailbox or a mail-enabled security group in PolicyScopeGroupId. For Shared mailboxes, add the Shared mailbox to a mail-enabled security group and use the group in PolicyScopeGroupId.

* Test the Application Policy

Once the application policy is created, verify that access is granted only to the scoped users.

```
Test-ApplicationAccessPolicy -AppId <APP ID or Client ID> -Identity <email address>
```

![Screenshot 2025-06-18 235126](https://github.com/user-attachments/assets/f26be77b-8f3c-44a4-9e64-7c58e7507d5e)

![image](https://github.com/user-attachments/assets/d5811be4-b6ca-4a26-8f11-3a4599ddbd2c)
