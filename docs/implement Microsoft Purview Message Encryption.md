---
title: Implement Microsoft Purview Message Encryption and Microsoft Purview Advanced Message Encryption in Exchange Online
nav_order: 11
---
# How to implement Microsoft Purview Message Encryption and Microsoft Purview Advanced Message Encryption in Exchange Online

Microsoft Purview Message Encryption is an intuitive service that enables email users to send encrypted messages to people inside and outside their organization. Selected recipients can easily view their encrypted messages and return encrypted replies. This is done regardless of the destination email service, whether it's Microsoft 365, Outlook.com, Yahoo, Gmail, or another service.

Microsoft Purview Advanced Message Encryption provides organizations with greater control. It can be used to create multiple templates for encrypted emails originating from within the organization. These templates can be used to control parts of the end-user experience.

# Configure Microsoft Purview Message Encryption

*  Check if Azure RMS is enabled using AIP PowerShell:

```
Install-Module -Name AIPService -Scope CurrentUser
Connect-AipService
Get-AipService
```

* If Azure RMS is not enabled for some reason for you tenant, enable it by running:

```
Enable-AipService
```

![image](https://github.com/user-attachments/assets/846222bc-bf8f-4348-801f-feb5b716990a)


* Check if Purview Encryption is enabled using EXO PowerShell:

```
Get-IRMConfiguration
```

![image](https://github.com/user-attachments/assets/cf5810b8-e146-4d7b-abfe-2db1de456b87)


* If it is disabled, enable it by running:

```
Set-IRMConfiguration -AzureRMSLicensingEnabled $true
```

* Test IRM configuration by running:

```
Test-IRMConfiguration [-Sender <email address> -Recipient <email address>]
```

![image](https://github.com/user-attachments/assets/12e6a392-f2b9-4dfe-a107-71e869d11dd7)

* If the test FAILS, disable with an error message Failed to acquire RMS templates, run:

```
$RMSConfig = Get-AipServiceConfiguration
$LicenseUri = $RMSConfig.LicensingIntranetDistributionPointUrl
Set-IRMConfiguration -LicensingLocation $LicenseUri
Set-IRMConfiguration -InternalLicensingEnabled $true
```

## User Experience

One of the key strengths of Purview Message Encryption is that it works across many platforms, including Outlook for desktop, mobile, and web. External recipients using Gmail, Yahoo, or other services can access encrypted messages through a secure web portal. They don't need to install special apps or sign in with a Microsoft 365 account.

The reading experience varies depending on the client. Users in Outlook often see the message natively, while other recipients receive a wrapper message with a link to the secure portal. 

You can customize the look and feel of encrypted messages by modifying the branding template used in the encrypted message portal. This helps ensure that external recipients know the message is legitimate and tied to your organization.

If you plan to use Advanced Message Encryption features, such as message expiration or revocation, custom branding is required. This branding is what allows the portal wrapper to be applied consistently and trigger the desired behavior.

## Manually applying Encryption using the default Encrypt-Only and Do Not Forward templates

* Encrypt-only option. This option enables organizations to **encrypt data without other restrictions**. The recipients have all usage rights except Save As, Export and Full Control. This combination of usage rights means the recipients have no restrictions except that they can't remove the protection.

* Do Not Forward option. When the system applies this option to an email message, the email is encrypted. This process forces the recipients to authenticate. **Recipients can't forward the message, print it, or copy from it**.

***


### Encrypt

![Screenshot 2025-06-02 214629](https://github.com/user-attachments/assets/6f681e04-ee68-4feb-9398-6170cfdc4eb9)

![Screenshot 2025-06-02 215045](https://github.com/user-attachments/assets/c32e4015-cb64-4ee9-bc0b-410701670aa4)

![Screenshot 2025-06-02 215201](https://github.com/user-attachments/assets/4d471147-6dd8-4d3c-8f51-adb4ecf10423)

***


### Do Not Forward

![image](https://github.com/user-attachments/assets/d238d1b0-fa29-4ec4-9d79-3d1744d6a398)

![Screenshot 2025-06-03 001847](https://github.com/user-attachments/assets/d55115a3-3838-42a9-a3b7-ce843a2ae53a)

![Screenshot 2025-06-03 002300](https://github.com/user-attachments/assets/f41e07db-b07c-48a8-b772-9ca5e4d13810)

![Screenshot 2025-06-03 002429](https://github.com/user-attachments/assets/2d0a767b-f8ef-412d-b148-b933c80a6a77)

![Screenshot 2025-06-03 002041](https://github.com/user-attachments/assets/669a2b91-1062-406c-9206-ea82ff0f3218)

![image](https://github.com/user-attachments/assets/73b4f994-919a-48cc-bc71-ded7f12cf7d8)

![image](https://github.com/user-attachments/assets/5eb4a590-e1d5-45e5-8557-793e024d0896)

![Screenshot 2025-06-03 010334](https://github.com/user-attachments/assets/0c4447e6-643f-4140-bf42-d58472585a8d)


* The attached document in both the case does not inherit the access rights and can be printed:

![image](https://github.com/user-attachments/assets/376127bc-d772-42a5-8237-231fba673bdf)

## Define mail flow rules to automatically encrypt email messages

You can create mail flow rules to protect email messages you send and receive. For example, you can:

* Set up rules to encrypt outgoing email messages.
* Remove encryption from encrypted messages coming from inside your organization.
* Remove encryption from replies to encrypted messages sent from your organization.
* You can't encrypt inbound mail from senders outside your organization.

### Set up rules to encrypt outgoing email messages:

* Go to EAC > Mail Flow > Rules > Add a new rule > Create a new rule
* Define the conditions when to apply encryption. Example, I have set the condition to encrypt the emails sent from the members of the 'Finance' group
* Under '**Do the Following**' select '**Modify the Message Security**' and then select '**Apply Office 365 Message Encrption and rights protection**'
* Then select the RMS template you want to apply. In this case, I will be applying the 'Encrypt' template
* Click Next and Create. The rule will be disbaled by default, make sure to Enable it. 

![image](https://github.com/user-attachments/assets/f7136451-1ec1-42e9-843d-d17c57e15ff5)

![image](https://github.com/user-attachments/assets/32ae44df-2b29-4c36-beba-542a1cfce41c)

### Validating the mail flow rule

* Sending the email without encryption to Gmail:

![image](https://github.com/user-attachments/assets/69b1b7cb-eb06-4ce5-a16e-0446942e5296)

![image](https://github.com/user-attachments/assets/2ab6b1d1-0c28-4894-ae58-c0ebdd636ae9)

* Performing the message trace to verify the rule was applied:

![image](https://github.com/user-attachments/assets/ca151bea-25ab-416a-a06b-a0488b668b90)

* And as expected, received the email as encypted in Gmail:

![image](https://github.com/user-attachments/assets/691bcf90-a4f6-4590-84a2-d28d888d5e00)

## Cutomizing the default branding template:

You can apply your company branding to customize the look of your organization's email messages and the encryption portal. To do so, you must first apply Global Administrator permissions to your Microsoft 365 account before you can get started. Once you have these permissions, you can customize the follwing parts of encrypted email messages:

* Introductory text.
* Disclaimer text.
* URL for Your organization's privacy statement.
* Text in the message encryption portal.
* Logo that appears in the email message and encryption portal, or whether to use a logo at all.
* Background color in the email message and encryption portal.

![image](https://github.com/user-attachments/assets/b32d503c-ea39-4393-b089-6ca0ab237139)

* Let us cutomize our branding template:

```
Set-OMEConfiguration -Identity "OME Configuration" -DisclaimerText "This message is the property of IdiotBox Ltd." -PortalText "IdiotBox Secure Portal" -BackgroundColor "#096B18" -Image ([System.IO.File]::ReadAllBytes('C:\Users\aaqib\OneDrive\Pictures\download.png')) -IntroductionText "sent you a secure message." -ReadButtonText "Click to Read" -EmailText "Encrypted message from IdiotBox secure messaging system" -PrivacyStatementURL "https://idiotbox.in/privacystatement.html" 
```

![image](https://github.com/user-attachments/assets/fc619192-c5fd-44e6-8815-a604c795bd32)

![image](https://github.com/user-attachments/assets/74510d69-c22a-44fe-942f-26ec6373399c)

![Untitled](https://github.com/user-attachments/assets/af6c9d17-48dc-4ce6-8a96-89e18a8f3213)


# Configure Microsoft Purview Advanced Message Encryption

Microsoft Purview Advanced Message Encryption provides additional protection and control for email messages sent to external recipients. The key features in Microsoft Purview Advanced Message Encryption include:

* Create multiple branding templates.
* Revoke encrypted email.
* Set an expiration date for encrypted email.
* Monitor encrypted message activity

The following subscriptions include Microsoft Purview Advanced Message Encryption:

* Microsoft 365 Enterprise E5
* Office 365 E5
* Microsoft 365 E5 (Nonprofit Staff Pricing)
* Office 365 Enterprise E5 (Nonprofit Staff Pricing)
* Office 365 Education A5

If your organization has a subscription that doesn't include Microsoft Purview Advanced Message Encryption, you can purchase it with one of the following add-on's:

* Microsoft 365 E5 Compliance SKU add-on for Microsoft 365 E3
* Microsoft 365 E3 (Nonprofit Staff Pricing)
* Office 365 Advanced Compliance SKU add-on for Microsoft 365 E3, Microsoft 365 E3 (Nonprofit Staff Pricing), Office 365 SKUs,
* Microsoft 365 E5/A5 Information Protection and Governance SKU add-on for Microsoft 365 A3/E3

## Create multiple branding templates

Microsoft Purview Advanced Message Encryption doesn't limit you to a single branding template. Instead, you can create and use multiple branding templates.

Let's create a new OME template that we will use for Finance users, disable the social sign in and set the expiry as 7 days. 

```
New-OMEConfiguration -Identity "Finance Template" -DisclaimerText "This message is the property of IdiotBox Ltd. Finanace Department" -PortalText "IdiotBox Finance" -BackgroundColor "#096B18" -Image ([System.IO.File]::ReadAllBytes('C:\Users\aaqib\OneDrive\Pictures\download.png')) -IntroductionText "sent you a secure message." -ReadButtonText "Click to Read" -EmailText "Encrypted message from IdiotBox financing secure messaging system" -PrivacyStatementURL "https://idiotbox.in/privacystatement.html" -SocialIdSignIn $false -ExternalMailExpiryInDays 7 
```

![image](https://github.com/user-attachments/assets/1715391a-2eb4-433e-98c0-8b4f3bfca4ab)

### Set up rule to apply the custom branding template:

I will modify the rule created in the previous step and add the new custom branding template 'Finance template':

IMPORTANT: To use the custom branding template, it is required to use the condition "Sender is in Organization" in addition to other conditions as required. To ensure that the rule encrypts the email either have another rule with higher priority or add the condition like below:

The additions I made to the above rule are highlighted:

![image](https://github.com/user-attachments/assets/c5e15e68-0b32-4e34-a944-a6a537247e5c)

Now when I send the email from a user who is part of the Finance group, this is what the receipint will see. Notice the Expiry and changes the the branding:

![image](https://github.com/user-attachments/assets/1851d9ab-f955-473c-b727-5d48dd02a010)

And since I disabled social sign in on this template, I only get an option to authenticate using an OTP:

![image](https://github.com/user-attachments/assets/e8374200-f892-42aa-9ca7-b64c7822fa36)

![image](https://github.com/user-attachments/assets/0a9d7c9b-f98d-44ce-ab4c-c0233cf0fc99)

![image](https://github.com/user-attachments/assets/f746514e-747e-4300-ba9b-2930d72bf1cd)

### Revoke the encrypted email:

You can only revoke messages that users receive through the message encryption portal. In other words, email that has a custom branding template applied.

Let's assume a message was encrypted using Microsoft Purview Advanced Message Encryption. Either a Microsoft 365 administrator or the sender of the message can revoke the message under certain conditions:

* Microsoft 365 administrators can revoke messages using PowerShell.
* The sender can revoke a message if they sent it directly from Outlook on the web.
* Administrators and message senders can revoke encrypted emails if the recipient received a link-based, branded encrypted email. If the recipient received a native inline experience in a supported Outlook client, then they can't revoke the message.

Connect to EXO PowerShell and run:

```
Get-OMEMessageStatus -MessageId "<message id>" | ft -a  Subject, IsRevocable
```

If the IsRevocable field is True, you can revoke the access to the encrypted email:

```
Set-OMEMessageRevocation -Revoke $true -MessageId "<messageId>"
Get-OMEMessageStatus -MessageId "<messageId>" | ft -a  Subject, Revoked
```

![image](https://github.com/user-attachments/assets/2d8b530b-ee80-410e-a745-ff30058acc7a)

![image](https://github.com/user-attachments/assets/b8bcc4f8-4820-4764-8756-f7c606d624e6)


