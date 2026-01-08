---
title: Use Privileged Identity Management (PIM) for Groups
nav_order: 17
---

Managing password reset requests is one of the biggest burdens on IT helpdesks. **Self-Service Password Reset (SSPR)** allows users to reset or unlock their own passwords securely without calling IT.

When SSPR is not enabled:

* User clicks on **Forgot Password**
* User is prompted with verification 
* An error is displayed that the user cannot reset his password and needs to contact helpdesk/administrator
* User contacts helpdesk
* Helpdesk verifies the user and resets the passwords either in Entra (Cloud Only) or on-prem AD (Hybrid) and sets the option require the user to change the password after signing in)

When SSPR is enabled:

* User goes to the password reset portal (or clicks Forgot password).
* User verifies identity using configured methods (MFA, phone, authenticator, etc.)
* User resets password
* Password is updated in Entra ID (Cloud only), and password is written back to on-prem Active Directory (in case of hybrid and when password writeback is enabled).


## Step 1: Prerequisites for Enabling SSPR

### 1. Licensing Requirements

| Scenario                         | License Needed                  |
| -------------------------------- | ------------------------------- |
| Cloud-only users                 | **Microsoft Entra ID Free**     |
| Hybrid users (on-prem AD synced) | **Microsoft Entra ID P1 or P2** |
| Writeback to on-prem AD          | **Entra ID P1 or P2**           |

{: .note }
> **Password writeback** is required if users have accounts in **on-premises Active Directory**.


### 2. User Authentication Data

Before users can reset passwords, they must register:

* Mobile phone number
* Alternate email
* Microsoft Authenticator app

{: .note }
> You can **force registration** (recommended).


## Step 2: Enable SSPR


1. Go to **[https://entra.microsoft.com](https://entra.microsoft.com)**
2. Sign in using an account with at least Authentication Policy Administrator role 
3. Protection > Password reset > Properties
4. Enable SSPR and choose a Pilot group to use SSPR:

<img width="1000" height="500" alt="image" src="https://github.com/user-attachments/assets/266dcd72-ae0a-4d9c-afc9-6750f6779f71" />


{: .note }
> **Best Practice:** Start with a **pilot group**


## Step 3: Configure Authentication Methods

When users need to unlock their account or reset their password, they're prompted for an authentication method to verify themselves. You can choose which authentication methods to allow in the authentication methods policy. If the user has updated the authentication method in his profile and if the authentication method is supported for SSPR, then he can use that method to reset his password.

* Password Reset > Authentication Methods
* Set the number to 2

<img width="900" height="500" alt="image" src="https://github.com/user-attachments/assets/342e8531-639c-4a50-8daf-7c56cb844c7a" />


* To select which methods to use for authentication go to **Authentication Methods** > Policies


<img width="1000" height="500" alt="image" src="https://github.com/user-attachments/assets/9784bbc2-2c35-49fb-b69e-ae05064f44ba" />


As you can see, I have **Authenticator**, **SMS** and **Email OTP** enabled for all users, hence the user if enabled for SSPR will be able to use these methods to reset his password. 

{: .note }
> Avoid security questions unless absolutely required. Security questions for now can only be enabled for SSPR from Password Reset > Authentication Methods and not from the converged authentication methods policy. 
A strong, two-method authentication policy is always applied to accounts with an administrator role, regardless of your configuration for other users.
The security-question method isn't available to accounts associated with an administrator role.

* _Behavior when SSPR is successfully enabled:_

<img width="500" height="400" alt="Screenshot 2026-01-06 154249" src="https://github.com/user-attachments/assets/57f1f882-5219-4cbf-8895-60b54cd33d23" />
<img width="500" height="400" alt="Screenshot 2026-01-06 154314" src="https://github.com/user-attachments/assets/47c37e50-932e-47e8-aa30-3f317d7c9fc7" />
<img width="500" height="400" alt="Screenshot 2026-01-06 154338" src="https://github.com/user-attachments/assets/4876b221-dbd9-497a-a20d-816590f59f58" />
<img width="500" height="400" alt="Screenshot 2026-01-06 154400" src="https://github.com/user-attachments/assets/6595adfd-48a1-4c7f-96e5-349881bb1f26" />
<img width="500" height="400" alt="Screenshot 2026-01-06 154423" src="https://github.com/user-attachments/assets/34ab16f5-9d27-485a-a454-60be65e40f60" />
<img width="500" height="400" alt="Screenshot 2026-01-06 154432" src="https://github.com/user-attachments/assets/ddec1036-3dd6-4654-b5e3-41c80ae94ae1" />


{: .important }
> If you have setup the number of required methods for SSPR to 2 and the user has not registered for another authentication method, he will be unable to reset his password (see below). This can be avoided by requiring the users to register when signing in using the **Registration** policy under Password reset or an admin can manually add authentication information for the user.

* _Behavior when user does not have the required authentication methods registered:_

<img width="600" height="400" alt="image" src="https://github.com/user-attachments/assets/a3ba77d0-cbfd-4b80-ae36-720dfe0f91bf" />


## Step 4: Configure Registration Settings

Force users to register authentication info.

* Navigate to Password Reset > Registration
* Enable **Require registration at sign-in**
* Set the value for **Number of days before users are asked to re-confirm their authentication information**

This designates the time period before registered users are prompted to re-confirm their existing authentication information is still valid, up to a maximum of 730 days. If set to 0 days, registered users will never be prompted to re-confirm their existing authentication information **180 days**

<img width="900" height="600" alt="image" src="https://github.com/user-attachments/assets/10bf3dc3-8764-423e-9733-f3d3dcc0dee8" />

---

## Step 5: Enable Notifications

To keep users informed about account activity, you can set up Microsoft Entra ID to send email notifications when an SSPR event happens. This helps detect **unauthorized activity**.

* Go to Password Reset > Notifications
* Turn ON --> **Notify users on password resets?** _to let users receive an email to their primary and alternate email addresses notifying them when their own password has been reset via the Self-Service Password Reset portal_
* Turn ON --> **Notify all admins when other admins reset their password?** _to let global administrators receive an email to their primary email address when other administrators reset their own passwords via the Self-Service Password Reset Portal_

<img width="900" height="600" alt="image" src="https://github.com/user-attachments/assets/0a3ffdad-bbd5-468d-9dab-bff82ab9e68c" />

<img width="700" height="500" alt="image" src="https://github.com/user-attachments/assets/0a7443ad-865f-441d-9f94-3ae1ab1eb2a9" />

---

## Step 6: Customize User Experience

If users need more help with the SSPR process, you can customize the "Contact your administrator" link. The user can select this link in the SSPR registration process and when they unlock their account or resets their password. 

* Go to **Password Reset** > **Customization**
* Turn On **Customize helpdesk link**
* Provide a link to an article or form or an email address

<img width="900" height="500" alt="image" src="https://github.com/user-attachments/assets/119dc8e5-dcc0-4a72-9256-1f02e3031dce" />


---

## Step 7: Enable Password Writeback for SSPR (Hybrid Only)

If users exist in **on-prem AD**, you will need to enable password writeback to keep the passwords in sync. Depending upon whether you are using Entra Cloud Sync or Entra Connect, you will follow either of the below steps:

### Option 1: When Using Entra Connect Sync

To enable SSPR writeback, first enable the writeback option in Microsoft Entra Connect. From your Microsoft Entra Connect server, complete the following steps:

* Login to the server where Entra Connect is installed and run the Entra Connect configuration wizard.
* Select Configure > Customize synchronization options, and then select Next.
* Authenticate using atleast a Hybrid Administrator credential 
* On the Connect directories and Domain/OU filtering pages, select Next.
* On the Optional features page, select the box next to Password writeback and select Next.
* On the Directory extensions page, select Next.
* On the Ready to configure page, select Configure and wait for the process to finish > Exit.

Now you will also need to enable password writeback for SSPR from the Entra portal

* Go to Password reset, then choose On-premises integration.
* Check the option for Enable password write back for synced users.
* (optional) If Microsoft Entra Connect provisioning agents are detected, you can additionally check the option for Write back passwords with Microsoft Entra Connect cloud sync.
* (optional) Check the option for Allow users to unlock accounts without resetting their password to Yes

<img width="800" height="600" alt="image" src="https://github.com/user-attachments/assets/526d05af-57a8-4248-b7a9-05035f8102b9" />

<img width="800" height="600" alt="image" src="https://github.com/user-attachments/assets/c1f04074-5d44-4b2c-acc6-4280362865be" />

<img width="1000" height="800" alt="image" src="https://github.com/user-attachments/assets/9b7c6b80-bd99-40f8-af77-38d0c3e676a2" />

### Option 2: Using Entra Cloud Sync

* Go to Password reset, then choose On-premises integration.
* Check the option for Enable password write back for synced users.
* Check the option for Write back passwords with Microsoft Entra Connect cloud sync.
* (optional) Check the option for Allow users to unlock accounts without resetting their password to Yes

<img width="1000" height="700" alt="image" src="https://github.com/user-attachments/assets/8270b71b-38b4-42a7-9bb2-2ef4dcdd6d5f" />


---

## Security Considerations

* Prefer **Authenticator app**
* Require **2 methods**
* Avoid email-only setups
* SSPR works best when paired with **Multi-Factor Authentication (MFA)**.
* Block risky locations using Conditional Access
* Require MFA for password reset using Conditional Access
* Restrict legacy authentication using Conditional Access
* Monitor Audit Logs → Password reset events
* Monitor Sign-in logs → Suspicious activity

---

## Common Mistakes to Avoid

❌ Enabling SSPR without user training

❌ Allowing only 1 authentication method

❌ Forgetting password writeback

❌ Enabling for all users without pilot testing

❌ Using weak methods like security questions only

---

## Best Practices Summary

✔ Start with a pilot group

✔ Require 2 authentication methods

✔ Prefer Authenticator app

✔ Enable notifications

✔ Pair with MFA and Conditional Access

✔ Educate users with screenshots or short guides


