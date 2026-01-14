---
title: How to implement a DLP policy in Exchange Online
parent: DLP
nav_order: 1
---

## Create a DLP policy for Exchange Online

* To get started, you’ll need **Compliance Administrator** or **Global Admin** permissions.

1. Log in to the **[Microsoft Purview compliance portal](https://purview.microsoft.com/)**.
2. In the left-hand navigation pane, select **Solution** > **Data loss prevention** > **Policies**.
3. Click **+ Create policy** and Select **Enterprise applications & devices**

<img width="1000" height="600" alt="image" src="https://github.com/user-attachments/assets/68fc088b-12a5-442e-a9ea-721885594930" />

<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/5d3d7ac2-5aae-4255-8bd4-728a9f3003f1" />

* Microsoft provides dozens of pre-configured templates (GDPR, HIPAA, PCI-DSS) that automatically look for the right data types. You can either start with a template or use custom. 

4. Select Custom > Custom Policy, then click Next
5. Provide a relevant name as per your organizations naming standards and click Next. Optionally, you can add a description as well to document the use of the policy. 
6. If you want to scope the policy to specific Admin Units, select the Admin Unit or Skip to scope for Full Directory 
7. On the **Locations** page, you’ll see several toggles (SharePoint, OneDrive, Teams). * Turn **On** the status for **Exchange email**. You can choose to include or exclude specific distribution groups or users if you want to pilot the policy with a small group first.

<img width="900" height="700" alt="image" src="https://github.com/user-attachments/assets/5bad91c0-9793-4c10-accf-cbde8fa72432" />

8. On the **advanced DLP rules** page, click **+Create Rule** and provide a name for the rule. A policy can contain several rules.

We will create 3 rules in this policy:

* To notify the user with a Policy tip and email notification when he sends an INTERNAL email containing 1 Sensitive Info Type (SIT) as defined in the rule but perform no other action.
* To notify the user with the tip and email AND also **Encrypt** the email when he sends an INTERNAL email containing more than 1 Sensitive Info Type (SIT) as defined in the rule.
* To notify the user with the tip and email AND also **Block** the email when he sends an EXTERNAL email containing 1 or more Sensitive Info Type (SIT) as defined in the rule.

## Rule 1: Block any email with Sensitive Info - for External users

1. Provide a name and description for the rule. Click "Add condition" and select **Content contains**. Choose **Sensitive info types** and select the data you want to protect (e.g., Credit Card Number).
1. For each SIT select "Medium" or "High" Confidence and set the Instance count 1 to Any. 
2. Click "Add condition" again and select **Content is shared from Microsoft 365**. Choose **with people outside my organization**
3. Click **Add Action** > **Restrict access or encrypt the content in Microsoft 365 locations** > **Block users from receiving email, or accessing ...** > **Block everyone**
4. Under **User notifications** check both "Email notifications" and "Policy tips" or as required. Customize the Email notifications and Policy tips if required. 
5. Under **Incident Reports** select **High** severity level for alerts and reports and enable **Send an alert to admins when a rule match occurs.**
6. Under additional options, you can optionally select _stop processing additional DLP rules_
6. Set the Rule priority to 0 > Save


## Rule 2: Notify user and Encrypt emails containing high number of Sensitive Info - for Internal users

1. Provide a name and description name for the rule. Click "Add condition" and select **Content contains**. Choose **Sensitive info types** and select the data you want to protect (e.g., Credit Card Number).
1. For each SIT select "Medium" or "High" Confidence and set the Instance count 2 to Any. 
2. Click "Add condition" again and select **Content is shared from Microsoft 365**. Choose **Only with people inside my organization**
3. Click **Add Action** > **Restrict access or encrypt the content in Microsoft 365 locations** > **Encrypt Email messages >> Select a built in or custom template example **Encrypt**
4. Under **User notifications** check both "Email notifications" and "Policy tips" or as required. Customize the Email notifications and Policy tips if required. 
5. Under **Incident Reports** select Low to generate 
6. Set the Rule priority to 1 > Save

## Rule 3: Notify user Only containing Sensitive Info - for Internal users

1. Provide a name and description name for the rule. Click "Add condition" and select **Content contains**. Choose **Sensitive info types** and select the data you want to protect (e.g., Credit Card Number).
1. For each SIT select "Medium" or "High" Confidence and set the Instance count 1 to 1. 
2. Click "Add condition" again and select **Content is shared from Microsoft 365**. Choose **Only with people inside my organization**
3. We are not performing any action on these conditions 
4. Under **User notifications** check both "Email notifications" and "Policy tips" or as required. Customize the Email notifications and Policy tips if required. 
5. Set the Rule priority to 2 > Save


Note: **Policy Tips**: These are small banners that appear at the top of an Outlook email *while the user is typing*, alerting them to the sensitive content before they even hit "Send."

Below is how the rules will look like once you're done:

<img width="1900" height="800" alt="image" src="https://github.com/user-attachments/assets/d0990a49-ab57-46bc-8060-e071257bcb83" />


## Behavior

Now, let us check the behavior when the conditions of each of the rule matches:

### Rule 1: Block any email with Sensitive Info - for External users

<img width="700" height="500" alt="image" src="https://github.com/user-attachments/assets/43d766c2-50f3-4da9-9567-6f692b1d5f2c" />
<img width="600" height="200" alt="Screenshot 2026-01-11 224104" src="https://github.com/user-attachments/assets/f98a1ed7-7c68-4913-b28e-bc82e14b2a01" />

If you still manage to send the email, you will receive the notification as below:

<img width="700" height="500" alt="image" src="https://github.com/user-attachments/assets/12f44d3e-6e2a-4f3c-8743-c48d22ad843d" />

and since the rule is configured to generate a **High** severity alert and send email to admin, the admin will receive the below email, and an alert will be generated in the Purview and Defender Portal (see Alerts & Monitoring section)

<img width="1000" height="700" alt="image" src="https://github.com/user-attachments/assets/9c4c1b13-ef2e-4128-9377-fb600cc319d2" />


### Rule 2: Notify user and Encrypt emails containing high number of Sensitive Info - for Internal users

<img width="700" height="500" alt="Screenshot 2026-01-11 223841" src="https://github.com/user-attachments/assets/70ef9df8-734a-4740-b2ad-b06b016ae7c9" />
<img width="700" height="300" alt="image" src="https://github.com/user-attachments/assets/ad640baa-b3bb-42a7-99e5-a0c9704e06e9" />
<img width="700" height="350" alt="image" src="https://github.com/user-attachments/assets/a42f127a-5e56-478e-802e-bb559856f82c" />


### Rule 3: Notify user Only containing Sensitive Info - for Internal users

<img width="700" height="400" alt="Screenshot 2026-01-13 022532" src="https://github.com/user-attachments/assets/f51c1f9e-a8bd-4cf1-a0c9-45c5c2aac506" />
<img width="700" height="350" alt="Screenshot 2026-01-13 022742" src="https://github.com/user-attachments/assets/351fded2-74a5-4067-96f8-a969cf812ad1" />
<img width="700" height="300" alt="Screenshot 2026-01-13 022857" src="https://github.com/user-attachments/assets/d388262b-7e40-4ad2-b070-be0c1faef063" />


## Alerts & Monitoring 

### Activity Explorer

Activity Explorer is a granular audit log that tracks how sensitive data is handled across the organization. It allows you to investigate specific policy matches, validate "Simulation" rules before they go live, and identify high-risk user behaviors across email, cloud storage, and physical devices.

* Below is the DLP rule matched activity captured in the **Activity Explorer** for the internal email that was sent containing sensitive info.
* We can see the type of SIT that was found in the email
* The **Policy** and **Rule** that matched for the activity
* The rule action that took place and other information like Sender, Subject, etc. 

<img width="1800" height="700" alt="image" src="https://github.com/user-attachments/assets/09cf5f54-8e32-48af-b410-7bc42212f001" />

### Alerts 

While Activity Explorer is for deep forensic research, the Alerts option is your real-time "alarm system." It is designed to notify you immediately when a specific rule is broken so you can take quick action.

* Below is the alert that was generated when a user tried to send an email with sensitive info to an external recipient. 
* Similar to Activity explorer, we can see the type of SIT that was found in the email, the **Policy** and **Rule** that matched for the activity in addition to other information like Sender, Subject, etc. 
* However, it also enables you to triage events, assign owners for investigation, and track the status (Active, Investigating, or Resolved) of a data leak in real time.

<img width="1800" height="700" alt="image" src="https://github.com/user-attachments/assets/73ab7624-530b-417c-bf95-81401ea4d612" />


* To See the "Context" (The actual SIT info), in Activity Explorer and Alerts, you must have the Data Classification Content Viewer role. Without this, the actual sensitive digits or text will be masked or unavailable for preview in Activity Explorer and Alerts.

<img width="400" height="300" alt="image" src="https://github.com/user-attachments/assets/fe733a37-645c-48b4-b02b-ff1fb043a4c5" />

* The **Data Classification Content Viewer role** is part of **Content Explorer Content Viewer** and **Content Explorer List Viewer** built-in roles. You can add the user to either of the Role groups or create a custom role group.

<img width="400" height="300" alt="image" src="https://github.com/user-attachments/assets/33c51dd4-997a-4cd0-a881-3cb6475a89a6" />

* After the user is added, we are able to see the sensitive information.

<img width="400" height="300" alt="image" src="https://github.com/user-attachments/assets/78b2d6cc-13fd-4939-b57b-ba170087917b" />

<img width="400" height="300" alt="image" src="https://github.com/user-attachments/assets/512b9090-9a3e-481a-81b2-bda6f70030d5" />

Note: Access to sensitive data should follow the Principle of Least Privilege. When required it should be provided to Privacy & Compliance Officers, Tier 2 or Tier 3 SOC analysts or Legal Counsel only.

---

## Best Practices 

* One of the biggest mistakes admins make is turning on a policy "Hot" on day one. This can lead to hundreds of blocked legitimate emails (false positives). **Always choose "Run the policy in simulation mode" first.** > **Why?** This allows you to see what would have been blocked in the **Activity Explorer** without actually stopping any mail flow. Run this for 1–2 weeks to fine-tune your rules.
* **Use "Block with Override":** Instead of a hard block, allow users to provide a business justification to bypass the policy. This empowers users and reduces IT support tickets.
* **Watch the "High Volume" Threshold:** Create two rules within one policy: one for "Low Volume" (1-9 items) that just warns the user, and one for "High Volume" (10+ items) that strictly blocks the email.
* **Monitor Activity Explorer:** Regularly check the **DLP Activity Explorer** to identify "top offenders" or common false positives that require rule adjustments.
* Implementing DLP is an iterative process. Start small, monitor the results, and gradually increase your enforcement.
