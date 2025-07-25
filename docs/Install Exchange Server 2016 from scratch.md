---
title: Install Exchange Server 2016 from scratch
nav_order: 2
---

# Install Exchange Server 2016 from scratch

## Requirements:
At least 2 VMware or Hyper-V virtual machines. 3, if you want to add a client machine.

* A custom domain (optional) 

* 2 Windows 2012 R2 or Windows 2016 Datacenter edition servers. One to install the Domain Controller (DC) and the other to install Exchange Server. [Windows Server 2016 ISO](https://www.microsoft.com/en-us/evalcenter/download-windows-server-2016?msockid=1e825a2dad9464f405984ef5ac92651d)

* Exchange 2016 CU 23 [Exchange Server 2016 CU 23 ISO](https://www.microsoft.com/en-us/download/details.aspx?id=104132)

* Windows 10/11 for the client machine [Download Windows 11 ISO](https://www.microsoft.com/software-download/windows11?msockid=1e825a2dad9464f405984ef5ac92651d)


## Create the VMs:
For the DC, create a virtual machine with at least 8 GB of RAM, 2 CPU cores and 25 GB disk space. You can reduce the RAM to 4 GB once Windows Server is installed and promoted to DC if you do not have enough resources on the host machine.  

For the Exchange Server, create a virtual machine with at least 8 GB (16 GB preferably) of RAM, 2 CPU cores and 50 GB of disk space.  

## Setting up the Domain Controller 
For the purpose of this tutorial, we will be installing Windows 2016 Datacenter edition. Once Windows is installed, we need to add the Active Directory Domain Services role on the server. Follow the below steps in order:

* Name the Server: Run sysdm.cpl and in the Computer Name tab click on Change. Enter a relevant name for the VM, example DC or DC1. Click Ok and restart the machine.

![image](https://github.com/user-attachments/assets/d1004f4c-7660-4a4f-ba32-4c3c87d436be)


* Configure IP settings: Open cmd and run ipconfig to list the details of your network. Now simultaneously open Network Connections by running ncpa.cpl. Double-click your network adapter and go to Properties. Then double-click on the Internet Protocol Version 4 and select use the following IP address and put the existing IP, subnet mask and gateway you got from the ipconfig command as shown below. Then in the Preferred DNS server put the loopback IP 127.0.0.1. This is to ensure that the IP does not change when we restart the VM, since this VM will also be our DNS server. 


![image](https://github.com/user-attachments/assets/452fc595-6807-42a1-a97c-f3660dd1d095)

* Install ADDS (Active Directory Domain Services) Role: To install AD, go to Server Manager > Manage > Add Roles and Features. In the dialog box, click next, Role based or feature based option will be pre-selected > Click Next. In the next page the current server will be already selected, keep it as it and click Next. In the Select Server roles, tick the checkbox for Active Directory Domain Services. 

![image](https://github.com/user-attachments/assets/bc683017-4ff9-4a95-af9b-099c282b3ae7)

![image](https://github.com/user-attachments/assets/a7b0f983-3ece-4e06-86ec-5ffa1e5a10d8)

![image](https://github.com/user-attachments/assets/7460393a-1139-4249-952e-9818bb242632)

![image](https://github.com/user-attachments/assets/38d581dc-c35e-4603-bb4d-2ea3fb7400ad)

![image](https://github.com/user-attachments/assets/853e0ff0-4315-4263-acd6-cfb87cde1c51)

![image](https://github.com/user-attachments/assets/0f0a0dd0-7917-41db-8b1c-23e393a575c8)

![image](https://github.com/user-attachments/assets/8a55399f-ae9b-430c-bcd6-6100202df509)

![image](https://github.com/user-attachments/assets/0953c71b-e5b3-4f47-87d8-58b929eaab97)

![image](https://github.com/user-attachments/assets/a18321ab-8450-4733-a621-ead1978fecaf)

* Here you need to add your domain that you own OR you can also add a .local domain. If you already have ADDS setup, you can add the UPN suffix. 

![image](https://github.com/user-attachments/assets/8de8802c-0685-4112-b5da-60ff15cfc72e)

![image](https://github.com/user-attachments/assets/4568448a-2a4c-4198-9eab-42a1d3c53868)

![image](https://github.com/user-attachments/assets/9c2c7510-4369-4ede-b5e2-6b220befff4a)

![image](https://github.com/user-attachments/assets/c6f31b32-dbe8-47d3-992d-0e2853db461e)

![image](https://github.com/user-attachments/assets/f38cc62c-f2e3-42c8-92fb-c1311c3fe3a8)

![image](https://github.com/user-attachments/assets/49bcb210-db3d-40a0-9cbf-0dea01f17487)

Reboot the machine and login to the Domain:

![image](https://github.com/user-attachments/assets/d9567bbb-9182-46f6-ab09-c1d7b743a3de)

![image](https://github.com/user-attachments/assets/52abb887-3c81-4c0e-80b4-a9957ad8b51a)

![image](https://github.com/user-attachments/assets/712fe3ca-34b4-40e2-a0a4-3abb22a92839)

At this point we are done with installing Active Directory with domain as idiotbox.com. We can now create the users or groups if required. We can now proceed with the installation of Exchange Server. 

{: .note }

>In case you already have ADDS setup with a different domain or have a .local domain, you can add the UPN suffix using AD Domains & Trusts as below, the important thing is that one of these domains needs to be verified in M365 as well. 

![Screenshot 2024-11-01 145411](https://github.com/user-attachments/assets/d71bb1e2-15bd-46b8-a8de-addd377a58b2)

![image](https://github.com/user-attachments/assets/d4220d0e-49d2-4fdc-98ad-aa752cb57dde)


***

## Installing Exchange Server

Before we start installing Exchange on the VM, we'll need to do a couple other things:
* Change the DNS server to the DC we previously created:
![Screenshot 2024-10-10 185518](https://github.com/user-attachments/assets/9c134ba5-711e-4121-8114-2f871aabcaf2)

* Rename the VM (same steps we followed for the DC above), example we rename it to Exch1
* Connect the VM to the AD domain
![Screenshot 2024-10-10 185710](https://github.com/user-attachments/assets/d38cc5f2-ecc5-4a2e-8dd0-39d2f3167f16)


Now, we can proceed with the installation of the Exchange Server 2016 prerequisites for Windows Server 2016:
* Open PowerShell as admin and run the below cmd:
* `Install-WindowsFeature NET-Framework-45-Core, NET-Framework-45-ASPNET, NET-WCF-HTTP-Activation45, NET-WCF-Pipe-Activation45, NET-WCF-TCP-Activation45, NET-WCF-TCP-PortSharing45, Server-Media-Foundation, RPC-over-HTTP-proxy, RSAT-Clustering, RSAT-Clustering-CmdInterface, RSAT-Clustering-Mgmt, RSAT-Clustering-PowerShell, WAS-Process-Model, Web-Asp-Net45, Web-Basic-Auth, Web-Client-Auth, Web-Digest-Auth, Web-Dir-Browsing, Web-Dyn-Compression, Web-Http-Errors, Web-Http-Logging, Web-Http-Redirect, Web-Http-Tracing, Web-ISAPI-Ext, Web-ISAPI-Filter, Web-Lgcy-Mgmt-Console, Web-Metabase, Web-Mgmt-Console, Web-Mgmt-Service, Web-Net-Ext45, Web-Request-Monitor, Web-Server, Web-Stat-Compression, Web-Static-Content, Web-Windows-Auth, Web-WMI, Windows-Identity-Foundation, RSAT-ADDS`
* Install [.NET framework ](https://download.visualstudio.microsoft.com/download/pr/014120d7-d689-4305-befd-3cb711108212/0fd66638cde16859462a6243a4629a50/ndp48-x86-x64-allos-enu.exe)
* Install [December 13, 2016 (KB3206632) security update](https://support.microsoft.com/help/4004227) (You can only install this update if your Windows Server 2016 version is 14393.576 or earlier (circa December, 2016)).
* Install [Visual C++ Redistributable Package for Visual Studio 2012](https://www.microsoft.com/download/details.aspx?id=30679)
* Install [Visual C++ Redistributable Package for Visual Studio 2013](https://support.microsoft.com/help/4032938)
* Install [IIS URL Rewrite Module](https://www.iis.net/downloads/microsoft/url-rewrite) 
* Install [Microsoft Unified Communications Managed API 4.0, Core Runtime 64-bit](https://www.microsoft.com/download/details.aspx?id=34992) (If you get an error, reboot and try again)
![image](https://github.com/user-attachments/assets/5ffc5660-46ff-4f16-9533-fc25889b4bdc)

At this point of time, we'll restart the VM a couple of times to ensure all pending reboots are complete.

## Installing the Exchange 2016 Server:
After you restart the server, make sure to log back in using an AD admin account. If you have followed the previous steps, it will be default Administrator account we setup for our DC. Else, it needs to be an admin account with Domain Admin and Schema Admin privileges. 
![image](https://github.com/user-attachments/assets/cd0f98de-ba55-4bb9-88f4-2b60735ac9a7)

Mount the Exchange Server ISO, open command prompt as an admin and navigate to the root of the drive where the ISO was mounted, then run the below commands:
Example, if the ISO was mounted on D drive, run

`cd D:`

### Extend the AD Schema to include the Exchange attributes
`<Virtual DVD drive letter>:\Setup.exe /IAcceptExchangeServerLicenseTerms_DiagnosticDataON /PrepareSchema`

Example: `Setup.exe /IAcceptExchangeServerLicenseTerms_DiagnosticDataON /PrepareSchema`

![image](https://github.com/user-attachments/assets/784975af-f7e8-4b37-a411-a920c3cb2ca8)

### Preparing the AD to create containers, objects, and other items in Active Directory to store information
`<Virtual DVD drive letter>:\Setup.exe /IAcceptExchangeServerLicenseTerms_DiagnosticDataON /PrepareAD /OrganizationName:"<Organization name>"`

Example: `Setup.exe /IAcceptExchangeServerLicenseTerms_DiagnosticDataON /PrepareAD /OrganizationName:"Evil Corp"`

### Preparing the Domain to create additional containers and security groups, and set the permissions so Exchange can access them
`<Virtual DVD drive letter>:\Setup.exe /IAcceptExchangeServerLicenseTerms_DiagnosticDataON /PrepareAllDomains`

Example: `Setup.exe /IAcceptExchangeServerLicenseTerms_DiagnosticDataON /PrepareAllDomains`
![image](https://github.com/user-attachments/assets/1fae62be-bd45-416d-858e-2416f8d94526)

### Installing the Mailbox server Role
`Setup.exe /IAcceptExchangeServerLicenseTerms_DiagnosticDataON /Mode:Install /Roles:Mailbox /on:"Evil Corp"`

![image](https://github.com/user-attachments/assets/870aad30-cf83-4c9a-903e-fa49e69ba6e4)

### Verify Installation
Check if all the required Exchange services are running:

![image](https://github.com/user-attachments/assets/c2d51f65-fd4b-483c-8ce9-c43f50ef543f)

Check if you are able to access ECP and OWA

![image](https://github.com/user-attachments/assets/1ea3ed28-8420-4808-aae1-fd7b40f36b88)

Check if you are able to access Exchange Management Shell 

![image](https://github.com/user-attachments/assets/e586ed1c-fe4a-44d0-879c-b0c0f3c332c8)


### Post installation configurations:

* Renaming the Virtual Directories 


* Purchasing a 3rd party SAN certificate for the domain

You can either purchase a 3rd party SSL certificate or [Get and install a FREE Exchange SAN certificate](https://github.com/aaqibwani/M365/wiki/Get-and-install-a-FREE-Exchange-SAN-certificate-(Let's-Encrypt))

## Install and Configure AADConnect

* Download AADConnect (Microsoft Entra Connect) tool from [Microsoft Entra Connect](https://www.microsoft.com/en-us/download/details.aspx?id=47594)
* Install AADConnect. Use custom installation.
* On the User Sign-in page, we will select Password Hash Synchronization and enable Single Sign-on for the purpose of this tutorial. Read more about PHS and SSO:

![Screenshot 2024-10-12 015055](https://github.com/user-attachments/assets/0066fb33-03ab-4893-a81d-6306b5996a17)

* Enter the username of the AAD account with the following least privileges:

![Screenshot 2024-10-12 015214](https://github.com/user-attachments/assets/57d4b1f8-d889-48ea-8724-277205a9e7fd)

* Let AADConnect create a separate AD account that will be used for synchronization. Enter your Admin credentials to validate: 

![Screenshot 2024-10-12 020147](https://github.com/user-attachments/assets/c92237cf-afd1-4780-92e4-66cf7406c98e)

![Screenshot 2024-10-12 020312](https://github.com/user-attachments/assets/57793b49-1a32-4598-93be-e816adf5d87d)

* Since I have not added the idiotbox.com domain to my M365 tenant (as I do not own this), I will click on 'Continue without matching...'. But you can add your custom domain to M365.

![Screenshot 2024-10-12 020400](https://github.com/user-attachments/assets/64bb1f5f-717e-4a13-9bf5-b78cce6b3373)

* Select the OU's whose objects you want to sync to AAD. I have selected the Users and Computers default OU, however you can create a new OU and use it as well.

![Screenshot 2024-10-12 020436](https://github.com/user-attachments/assets/dc169851-5ae4-4651-ab00-500a75a24e88)

![Screenshot 2024-10-12 020450](https://github.com/user-attachments/assets/62aac2bd-1ac3-453d-bd54-d09d3d3e1ecb)

![Screenshot 2024-10-12 020504](https://github.com/user-attachments/assets/c147d233-facf-4396-8cab-fe93369e8f2c)

![Screenshot 2024-10-12 020522](https://github.com/user-attachments/assets/56d26070-3f5b-41e5-89aa-5a53c565dbd9)

![Screenshot 2024-10-12 020640](https://github.com/user-attachments/assets/d1461a91-2181-41d8-9d0c-41ff43de9884)

![Screenshot 2024-10-12 020658](https://github.com/user-attachments/assets/5517a981-5152-487e-bcfe-2d30fa1df4e7)

![Screenshot 2024-10-12 021318](https://github.com/user-attachments/assets/564ca551-886f-4dec-8dca-c11d05c4a459)

![Screenshot 2024-10-12 021912](https://github.com/user-attachments/assets/f87dd734-4b03-4d5e-a9c8-804ded470429)

![Screenshot 2024-10-12 022056](https://github.com/user-attachments/assets/b20e6d6d-cc8d-4c4e-b11d-88204db1147c)

![Screenshot 2024-10-12 022309](https://github.com/user-attachments/assets/164d580b-e47c-4d7e-890f-423f80bca875)


# Configuring Exchange Hybrid

* Download the Hybrid Configuration wizard from aka.ms/hybridwizard. Use Internet Explorer or Edge to download else you will get an error about different Security Zones
![Screenshot 2024-10-15 225124](https://github.com/user-attachments/assets/958dfc5a-1e7b-4dcf-9489-4bcd62ef4804)

![Screenshot 2024-10-15 225223](https://github.com/user-attachments/assets/6505fd1b-d1f9-4f4f-a39e-fbc315521a7b)

* If you get an error saying download is blocked, enable the downloads from:

![Screenshot 2024-10-15 230131](https://github.com/user-attachments/assets/cb880f52-bbb8-46f7-8426-80551f91b130)

* The HCW will open:
![Screenshot 2024-10-15 225337](https://github.com/user-attachments/assets/9d66cb2d-2f1f-4280-b944-3f4d43a0f5c4)

* Click on Next and select license this server. Your Exchange Server will be licensed:
![Screenshot 2024-10-15 232943](https://github.com/user-attachments/assets/64c95177-f6cf-48e4-a4e3-0269713aa3ce)

* Click Next and sign in to your AAD account with a Global Admin or Exchange Admin role.

![Screenshot 2024-11-01 153734](https://github.com/user-attachments/assets/489eadcf-c525-4a76-ad11-bd73c403b2b3)

* Click Next and select Full Hybrid

![Screenshot 2024-10-15 233035](https://github.com/user-attachments/assets/694b6956-b523-4808-8ee5-495ed9c79870)

* We can either go with Classic or Modern Hybrid depending upon if we procured a 3rd party SAN certificate and have the EWS virtual directory published in public DNS. For the purpose of this lab, we will go with Modern Hybrid. 

![Screenshot 2024-11-01 154022](https://github.com/user-attachments/assets/b3d4d4f3-7b1b-462f-801a-b8af55918601)

* Keep the default selection:

![Screenshot 2024-11-01 154044](https://github.com/user-attachments/assets/45230e34-ae33-405e-95ff-aea83e21ea2d)

* If you have not configured the Virtual Directories and populated the Internal/External URLs particularly the External URL for EWS, it will ask you to do that, enter the domain that has a public DNS:

![Screenshot 2024-11-01 154144](https://github.com/user-attachments/assets/9f255565-1731-449e-855f-236159717043)

* Enter the On-prem Admin credentials, this account will be used for hybrid mailbox migrations. It is also possible that the password expires in the near future. At that time you'll have to update the migration endpoint using PowerShell:

![Screenshot 2024-11-01 154216](https://github.com/user-attachments/assets/2e049043-1c2c-4e1b-b82e-5a082c7db65d)

* Now, the Hybrid Agent setup will happen:

![Screenshot 2024-11-01 154744](https://github.com/user-attachments/assets/d439c1ad-b04f-4893-b3dd-79aead74670f)

* On the next page, we will select CAS, as we do not intend to deploy an Edge Server:

![Screenshot 2024-11-01 154756](https://github.com/user-attachments/assets/7d2aae78-4509-4811-b54b-520050671d40)

* Next, you'll be asked to select the server to host the Receive and Send Connectors:

![Screenshot 2024-11-01 154811](https://github.com/user-attachments/assets/f2a94480-03b5-431b-a957-e20b837bc553)

![Screenshot 2024-11-01 154826](https://github.com/user-attachments/assets/f6b0921f-3507-4bb2-8459-1d623e3003f5)

* Next, you'll be asked to select the certificate. If you have installed a 3rd party certificate, you'll select that one.
* Next, enter the FQDN of your organization 
* The setup will complete or maybe have one or more issues. Depending upon the issues you encounter, you can fix them later by running the respective cmdlets in EMS:

![Screenshot 2024-11-01 163136](https://github.com/user-attachments/assets/78aa7564-725a-443c-b07e-83b5f3bd38fe)

I received an error because I did not install a new certificate on the server, however I can safely close it and later configure it manually. 

## Verify Exchange Hybrid Setup

* We can now try to provision a remote mailbox from Exchange on-premises and verify that the mailbox is provisioned in Exchange Online:
* Open EMS and run:

![Screenshot 2024-11-01 231731](https://github.com/user-attachments/assets/68a811fc-2238-481b-8369-0dbc68e09e40)

* Then run Delta Sync or wait for the next sync cycle
* Verify the mailbox is created in Exchange Online:

![image](https://github.com/user-attachments/assets/c5f0c77f-acd9-474a-961b-a255329b1147)

Our Exchange Hybrid Setup is complete!
   
