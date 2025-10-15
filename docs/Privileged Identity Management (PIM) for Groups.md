---
title: Use Privileged Identity Management (PIM) for Groups
nav_order: 16
---

# Use Privileged Identity Management (PIM) for Groups

Privileged Identity Management (PIM) for Groups is a feature in Microsoft Entra ID that allows organizations to manage, control, and monitor access to Microsoft 365 Groups and security groups with elevated privileges. It extends the capabilities of PIM beyond roles to group memberships, enabling just-in-time access and governance for sensitive group-based access.

## Key Benefits of PIM for Groups

* Just-in-Time Access: Users can be assigned eligible membership to a group and activate it only when needed, reducing standing access.
* Approval Workflow: Group membership activation can require approval from designated approvers, adding a layer of control.
* Time-Bound Access: Access can be limited to a specific duration, automatically removing users after the time expires.
* Audit and Alerts: All activations and changes are logged, and alerts can be configured for suspicious or unexpected activity.
* Access Reviews: Periodic reviews can be scheduled to ensure only the right users retain access.
* Integration with Role Assignments: Useful when group membership controls access to resources like SharePoint sites, Teams, or role-based access in Azure.


## Limitations of PIM for Groups

* Licensing Requirements: Requires Microsoft Entra ID P2 
* Limited to Cloud Groups: Only works with cloud-managed groups on-premises synced
* No Nested Group Support: PIM doesn’t support activation for nested group memberships

## Implement PIM for Groups

Here let's take an example of an Exchange Engineer who needs the Exchange Admin role and also needs the ability to Hard/Soft delete emails using Defender. As such, the Exchange engineer needs to have:

* The Exchange Admin Entra role (Using PIM for Entra)
* The Search & Purge role from Defender (Using PIM for Groups)

### Create a group 

1. Go to Entra ID > Groups > New Group > Security or Microsoft 365
2. Enable "Microsoft Entra roles can be assigned to the group"
3. Create the group. Don't add any roles, members or owners.
4. Open the Group > Select "Privileged Identity Management" and choose "Enable PIM for this group"

<img width="900" height="600" alt="image" src="https://github.com/user-attachments/assets/fbb4af98-5e3b-40c7-9d88-91359bbde00b" />

<img width="1000" height="600" alt="image" src="https://github.com/user-attachments/assets/42e750e4-94d4-4148-8182-c8c39359294d" />

### Assign the Exchange Entra Role

1. Assign the Exchange Administrator PIM role to the user either as Eligible or Permanent. 
2. Go to the User > Assigned Roles > Add Assignments > Select the Role "Exchange Administrator" 
3. Set the assignment type and select Assign

<img width="1200" height="550" alt="image" src="https://github.com/user-attachments/assets/a435467f-90d0-4397-9e58-c16c58eb0acb" />

<img width="600" height="700" alt="image" src="https://github.com/user-attachments/assets/7065e381-bd0f-4a1d-bd06-aaefc23424b6" />

<img width="500" height="700" alt="image" src="https://github.com/user-attachments/assets/ff34d1d9-88fd-47f2-b6d0-be738a41e946" />

{: .important }
> You should not make active assignment of a group to a role and assign users to be eligible to group membership, as it may take significant time to have all permissions of the role activated and ready to use. To avoid activation delays, use [PIM for Microsoft Entra roles](https://learn.microsoft.com/en-us/entra/id-governance/privileged-identity-management/pim-how-to-add-role-to-user) instead of PIM for Groups to provide just-in-time access to SharePoint, Exchange, or Microsoft Purview compliance portal. For more information, see [Error when accessing SharePoint or OneDrive after role activation in PIM](https://learn.microsoft.com/en-us/sharepoint/troubleshoot/administration/access-denied-to-pim-user-accounts).


### Create and assign the "Search & Purge" Defender role

1. Go to Defender > Permissions > Microsoft Defender XDR (Roles)
2. Create a new role
3. Under Permissions > Security Operations > enable "Email & Collaboration advanced actions (manage)"
4. Under Assignments > Add Assignment. Under Employees > select the "Exchange L3" group created in Entra
5. Click Next and Submit

{: .note }
> Ensure that the "Defender for Office 365" workload is activated as part of Unified RBAC. 


<img width="900" height="700" alt="image" src="https://github.com/user-attachments/assets/79caa5d3-add2-4cd0-af6f-967c41d6f285" />

<img width="1200" height="700" alt="image" src="https://github.com/user-attachments/assets/129954c3-1b99-4072-92fe-3105181b2f95" />

<img width="900" height="550" alt="image" src="https://github.com/user-attachments/assets/d89b5e55-7e08-478f-bec6-9f5db760a8f1" />

<img width="500" height="600" alt="image" src="https://github.com/user-attachments/assets/ba3178d2-8d91-4511-88ff-6b7554d19311" />


### Assign Eligible membership to the Entra Role

1. Open the PIM Group > Privileged Identity Management > Eligible Assignments > Add assignment 
2. Select the role as "Member" > and add the Exchange engineer as the member
3. Set the assignment as Eligible and Save
4. To configure activation duration or approval workflow, go to "Privileged Identity Management" > Settings > Member. For this tutorial, we are going with the defaults. 


<img width="1000" height="600" alt="image" src="https://github.com/user-attachments/assets/d1ce8152-ff1e-49db-b857-540526469048" />

<img width="1200" height="700" alt="image" src="https://github.com/user-attachments/assets/90667964-fa58-4096-b988-2b163d590b9f" />

<img width="550" height="400" alt="image" src="https://github.com/user-attachments/assets/1ebb0ec5-06af-47e6-afaa-8ad1996177e4" />


### Activating the PIM role as a User

1. Go to portal.azure.com > Privileged Identity Management
2. Under Tasks > My Roles
3. Under Activate > Groups
4. You will see the Group listed > click Activate 
5. Provide a justification and click Activate

<img width="1600" height="500" alt="image" src="https://github.com/user-attachments/assets/1a379d25-632e-4497-9ce4-97ed5dbf8c0e" />

<img width="500" height="700" alt="image" src="https://github.com/user-attachments/assets/4b4f4688-fc7e-4f29-b680-8ed97de4c6c7" />


### Validation


* Behavior with Exchange Admin role but without PIM for Group:

<img width="1600" height="600" alt="Screenshot 2025-10-14 172540" src="https://github.com/user-attachments/assets/44a4c5ed-57db-4387-8f74-652055fb7a4c" />


* Behavior with Exchange Admin role AND with PIM for Group:

<img width="1600" height="600" alt="Screenshot 2025-10-15 180844" src="https://github.com/user-attachments/assets/18b106cc-3e6b-4ecf-8991-702f1b125235" />

