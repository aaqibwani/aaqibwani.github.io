# Enable and use Shared Channels in Teams

A Shared Channel in Microsoft Teams is a special type of channel that allows you to collaborate with people inside and outside your organization without needing to switch tenants or add them as guests.


## Prerequisites:

* Enable M365 Groups Sharing from M365 Admin Center on the tenant where the shared channel is to be created:

<img width="1200" height="500" alt="image" src="https://github.com/user-attachments/assets/26064845-5094-4be5-a801-7cea5d64f3e5" />


* Enabled Guest access from the Teams Admin portal where the shared channel is to be created:

<img width="1000" height="400" alt="image" src="https://github.com/user-attachments/assets/0f995744-a1aa-4147-a5e3-d4ca16aaef30" />


* Teams Policy applied to the user(s) should enable the creation of Shared channels. This could either be the Global policy or a custom policy:

<img width="1200" height="500" alt="image" src="https://github.com/user-attachments/assets/95bb7370-b847-4811-ab4d-41ebcd67fe24" />

* SharePoint organization level and site level sharing settings must allow guests. The domains you're sharing with must not be blocked.

<img width="1000" height="600" alt="image" src="https://github.com/user-attachments/assets/5897d5d6-a55d-47a0-970e-d339611c68ec" />


* Enable B2B Direct Connect from Entra ID. B2B Connect needs to be enabled in both the tenants. Inbound access settings need to be enabled on the tenant where the Shared channel will be created, and outbound access settings need to be enabled on the external tenant whose users will be added to the shared channel. B2B direct connect is disabled by default. To enable collaboration in shared channels with people from other organizations, you must:

1. Add an organization.
2. Configure inbound settings for the organization to allow users from the organization to be invited to your shared channels.
3. Configure outbound settings for the organization to allow your users to be invited to the other organization's shared channels.

### [Source & Target Tenant] Add an organization

* Sign in to the Microsoft Entra admin center using a Security administrator account.
* Select External Identities, and then select Cross-tenant access settings.
* Select Organizational settings.
* Select Add organization.
* On the Add organization pane, type the full domain name (or tenant ID) for the organization and press Enter.
* Select Add.
* The organization appears in the organizations list. At this point, all access settings for this organization are inherited from your default settings.

<img width="1800" height="600" alt="image" src="https://github.com/user-attachments/assets/5d030565-0690-4a46-b94e-a6acfc9624ba" />

<img width="1800" height="600" alt="image" src="https://github.com/user-attachments/assets/0b7ff00d-c588-4b94-b064-7b67293aa506" />


### [Source Tenant] Configure inbound settings for the organization to allow users from the organization to be invited to your shared channels

Follow this procedure for each organization where you want to invite external participants.

To configure inbound settings for an organization

* In the Microsoft Entra admin center, select External Identities, and then select Cross-tenant access settings.
* Select the inbound access link for the organization that you want to modify.
* On the B2B direct connect tab, choose Customize settings.
* On the External users and groups tab, choose Allow access and All external users and groups. (You can choose Select external users and groups if you want to limit access to specific users and groups, such as those who have signed a non-disclosure agreement.)
* On the Applications tab, choose Allow access and Select applications.
* Select Add Microsoft applications.
* Select the Office 365 application and then choose Select.
* Select Save and close the Inbound access settings blade.

<img width="1600" height="500" alt="image" src="https://github.com/user-attachments/assets/6c1faa85-5ae4-431d-a615-01025101c3f6" />

<img width="900" height="600" alt="image" src="https://github.com/user-attachments/assets/7ffd1deb-a467-4d4f-8b80-9fb8569c3669" />

<img width="900" height="600" alt="image" src="https://github.com/user-attachments/assets/f3fe01e3-3611-4036-8527-e149accf0faf" />

Additionally, select "Trust multi-factor authentication from Microsoft Entra tenants" if your Conditional Access policies require multifactor authentication (MFA). This setting allows your Conditional Access policies to trust MFA claims from external organizations:

<img width="1073" height="667" alt="image" src="https://github.com/user-attachments/assets/5c998f31-132b-4ec3-a95f-366c9488dc08" />


### [Target Tenant] Configure outbound settings for the organization to allow your users to be invited to the other organization's shared channels

Follow this procedure for each organization where you want your users to be able to participate in external shared channels.

To configure outbound settings for an organization

* In the Microsoft Entra admin center, select External Identities, and then select Cross-tenant access settings.
* Select the outbound access link for the organization that you want to modify.
* On the B2B direct connect tab, choose Customize settings.
* On the External users and groups tab, choose Allow access and set an Applies to of all users.
* On the External applications tab, choose Allow access and select external applications.
* Select Add Microsoft applications.
* Select the Office 365 application and then choose Select.
* Select Save, choose Yes to confirm, and close the Outbound access settings blade.

<img width="1200" height="500" alt="image" src="https://github.com/user-attachments/assets/0576faee-ad57-4e5a-9342-6068bfd0a424" />

<img width="900" height="600" alt="image" src="https://github.com/user-attachments/assets/8f6190ad-4017-4902-b46c-05c285a28a3c" />

<img width="900" height="600" alt="image" src="https://github.com/user-attachments/assets/e5ce19fe-f185-4bd2-bfef-27f8126bdbfb" />

## Create a Shared Channel

To create a Shared Channel, you need to be the Team owner. Follow the below steps:

* Under Teams, click on the 2 dots (...) next to the Team in which you want to create the Shared Channel, select Add Channel
* Provide a name and select the Channel Type as Shared Channel
* Invite the internal and external users from the domain added in B2B collaboration
* The Shared Channel will be created and will be visible for the external users under Teams without switching the Tenant

<img width="786" height="577" alt="image" src="https://github.com/user-attachments/assets/3de43e66-34a6-47bd-b71a-321beb3c0212" />

<img width="832" height="792" alt="image" src="https://github.com/user-attachments/assets/272d5a58-a550-485b-9e27-86a9bfe96850" />

<img width="760" height="557" alt="image" src="https://github.com/user-attachments/assets/1ae8f871-c736-4ab6-a170-c2b9fcf2f9ad" />

<img width="628" height="583" alt="image" src="https://github.com/user-attachments/assets/fe91c4e5-24d9-4f1d-ad34-26bd1cd1b135" />

