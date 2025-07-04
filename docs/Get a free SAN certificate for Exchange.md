---
title: How to get a free SAN certificate for Exchange
nav_order: 4
---
# How to get a free SAN certificate for Exchange


* Identify the domains for which you need the certificate example: mail.domain.com, autodiscover.domain.com, etc.
* Go to https://punchsalad.com/ssl-certificate-generator/
* Enter the domains and email address and select the authentication method as DNS:

![image](https://github.com/user-attachments/assets/ddb078f9-179c-47b9-81d9-7bb9eb82bc0e)

* Click "Create Free SSL.."
* In the next page, you'll get the TXT records that you need to publish in your DNS to verify ownership.

![image](https://github.com/user-attachments/assets/a07e1fdf-9452-48a8-8d47-bd873be666bc)

* Copy the provided host and value. Login to you DNS provider and create the TXT records in your DNS like below:

![image](https://github.com/user-attachments/assets/1d094678-35bc-434b-a346-31a5f0884df1)

* It can take some time for DNS propagation, so be patient and periodically click on 'Check DNS'. Once verification is successful, you'll get the CRT bundle and the private key like below:

![image](https://github.com/user-attachments/assets/c1d71a44-6e2a-4fb6-bc8d-40d63d8c22bf)

![image](https://github.com/user-attachments/assets/0c797965-58af-4468-ab5a-c40957dcca6b)

* Download both the files
* Convert both the files to a PFX file: https://github.com/aaqibwani/M365/wiki/Create-a-PFX-certificate-from-a-CRT-and-private-key-using-Windows
* Save the file on your Exchange server and then run the below cmd in EMS: 

`Import-ExchangeCertificate -Server <server_name> -FileData ([System.IO.File]::ReadAllBytes('\\<server_name>\c$\<path>\mail.pfx')) -Password (ConvertTo-SecureString -String '<password>' -AsPlainText -Force)`

![image](https://github.com/user-attachments/assets/d813e7af-f7f8-41c9-aa99-c98809bf3c3f)

* Now if you go to EAC, you can see the certificate added to the Server. However, the certificate has not yet been enabled. We'll need to enable it for IIS (EAC,OWA) and SMTP OR IMAP and POP if required. 

![image](https://github.com/user-attachments/assets/6b3e7d8e-c05d-4bbd-8bff-4469c8b13d21)

* Enable the certificate for IIS and SMTP services using the below cmd:

`Enable-ExchangeCertificate -Thumbprint 8470DC4875E15EC0838013BCA14172FCEF9B0501 -Services SMTP,IIS`

![Screenshot 2024-09-08 022528](https://github.com/user-attachments/assets/98556073-a894-44f8-bbb8-b28443799f3d)

* You have successfully installed and enabled the certificate on the Exchange Server. You can now export and import the same certificate on other Exchange Servers as well. 
