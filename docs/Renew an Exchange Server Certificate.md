---
title: How to renew an Exchange Server certificate
nav_order: 3
---

# How to renew an Exchange Server certificate

Renewing an Exchange certificate involves a few steps:

- Identify the Certificate using EAC or EMS
- Create CSR using PowerShell
- Submit to Certificate Authority (CA)
- Download the New Certificate
- Import the new certificate into Exchange
- Bind the certificate to Exchange services (IIS, SMTP, etc.)
- Export the certificate to be used on another Exchange Server(s)
- Re-Run HCW (for Hybrid) or manually update the Send/Receive connectors
- Remove the Old Certificate



### Identifying the certificate:

* In EMS, run:


```
Get-ExchangeCertificate | Where-Object {$_.IsSelfSigned -eq $false} | Format-List FriendlyName, CertificateDomains, Thumbprint, NotAfter
```

![image](https://github.com/user-attachments/assets/a00f3ba8-5dfc-4fdc-abb1-25b65eaa0733)


![image](https://github.com/user-attachments/assets/e915ad4a-53f4-4824-b82f-7d9951b3aebe)


### Generating the CSR:

* If you need to renew the certificate without any changes or modifications, run:


```
$certrequest = Get-ExchangeCertificate -Thumbprint <YourThumbprint> | New-ExchangeCertificate -GenerateRequest -PrivateKeyExportable:$true
[System.IO.File]::WriteAllBytes('\\MS01\C$\temp\CertRenewal.req', [System.Text.Encoding]::Unicode.GetBytes($certrequest))
```

![image](https://github.com/user-attachments/assets/4f4aa8db-aee4-41dc-9356-a68e61b4d444)


![image](https://github.com/user-attachments/assets/d23227fd-b0c3-4f89-873a-380418b34105)


![image](https://github.com/user-attachments/assets/0bf60957-7377-471d-8e34-0d564c9d9ff0)


* However, if you need to change any parameter example, add more domains, you can create a new CSR by running (Exchange 2016 and above):

```
$txtrequest = New-ExchangeCertificate -GenerateRequest -SubjectName "c=US,o=Woodgrove Bank,cn=mail.woodgrovebank.com" -DomainName autodiscover.woodgrovebank.com,mail.fabrikam.com,autodiscover.fabrikam.com

[System.IO.File]::WriteAllBytes('\\FileServer01\Data\woodgrovebank.req', [System.Text.Encoding]::Unicode.GetBytes($txtrequest))
```

### Submit the CSR and download the certificate:

* Open the CSR .req file and copy the contents and paste as below. The process may differ based on the CA (I'm using zeroSSL for a free 90 day certificate):


![image](https://github.com/user-attachments/assets/ee1682c0-076d-4f9d-bf51-9ff964d4b634)


![image](https://github.com/user-attachments/assets/2d8dd5e7-5418-4828-971e-242c81d8b82b)


### Import the certificate in Exchange


```
Import-ExchangeCertificate -FileData ([System.IO.File]::ReadAllBytes('\\MS01\C$\temp\certificat
e.crt')) -PrivateKeyExportable:$true -Password (ConvertTo-SecureString -String 'mypassword' -AsPlainText -Force)
```

![image](https://github.com/user-attachments/assets/ed4b2725-4708-4298-acf8-6bcc75cb22b3)


![image](https://github.com/user-attachments/assets/ca482d97-707c-447e-98e6-6420721196bc)


### Bind the certificate to the IIS and SMTP services

```
Enable-ExchangeCertificate -Thumbprint <NewThumbprint> -Services IIS,SMTP
```

![image](https://github.com/user-attachments/assets/ed1b7941-1ff1-415a-9d68-469831063b09)


![image](https://github.com/user-attachments/assets/f9ad5280-4c62-4876-b10f-a94ded5849c6)


![image](https://github.com/user-attachments/assets/f66d6a06-4589-4ec4-be20-4252b5f18eb7)


## Export the certificate

* On the Exchange Server where you imported the certificate and completed the CSR, run:

```
$bincert = Export-ExchangeCertificate -Thumbprint D337A0CE4C49B88F0CDC9A52AE447904C075ADC2 -BinaryEncoded -Password (Get-Credential).password

[System.IO.File]::WriteAllBytes('C:\temp\mycertificate.pfx', $bincert.FileData)
```

* Remember the password you put in the above command as it would be needed when you import the certificate on another server using:

```
Import-ExchangeCertificate -Server MS02 -FileData ([System.IO.File]::ReadAllBytes('\\FileServer01\Data\ExportedCert.pfx')) -Password (Get-Credential).password
```

### Update the Send/Receive Connectors


```
$cert = Get-ExchangeCertificate -Thumbprint D337A0CE4C49B88F0CDC9A52AE447904C075ADC2

$tlscertificatename = "<I>" + $($cert.Issuer) + "<S>" + $($cert.Subject)

Set-SendConnector "Outbound to Office 365 - ca3bd31d-e12a-4bb6-820f-666ca3a9bd24" -TlsCertificateName $tlscertificatename

Set-ReceiveConnector "Default Frontend MS01" -TlsCertificateName $tlscertificatename
```

![image](https://github.com/user-attachments/assets/fc55c14a-4a55-44ec-b382-9c3e43163dba)


* You may also update the Hybrid object using:

```
Set-HybridConfiguration -TlsCertificateName $tlscertificatename
Get-HybridConfiguration | fl *tls*
```

![image](https://github.com/user-attachments/assets/37db5c42-1d15-4802-a17b-2d45ae88cfac)


* Additionally, restart the transport service and perform iisreset for the new certificate to be used for SMTP and IIS


### Remove the old certificate(s)

```
Remove-ExchangeCertificate -Thumbprint EB287621198A3307EC3D5C177733A8491E86C9AD
```

![image](https://github.com/user-attachments/assets/90f46718-7a76-4106-84a4-c7ca04bf1ad1)


![image](https://github.com/user-attachments/assets/2931ae6f-e619-4541-8a9a-16489b870f14)


### Verify the new certificate 

* After doing an iisreset and restarting the browser, access ECP and verify the new certificate is getting reflected:


![image](https://github.com/user-attachments/assets/2e4e61a1-48aa-4463-bb82-9edf780cf519)

* Verify the IIS bindings. Open IIS manager (inetmgr). Go to <Server Name> > Sites > Default Web Site and ensure all https port 443 have the new certificate updated:

![image](https://github.com/user-attachments/assets/42afa2e1-49aa-4b48-a93f-fb12510ed42f)


* Then verify the new certificate is being used for SMTP by analyzing the protocol logs on the send connector. Protocol logs will be stored at %ExchangeInstallPath%\TransportRoles\Logs\Hub\ProtocolLog\SmtpSend. To confirm, run:

```
Get-TransportService | Select-Object Name, SendProtocolLogPath
```


* If protocol logging is disabled, enable it first:

```
Set-SendConnector -Identity "Outbound to Office 365 - ca3bd31d-e12a-4bb6-820f-666ca3a9bd24" -Pr
otocolLoggingLevel verbose
```

![image](https://github.com/user-attachments/assets/1e5e2d93-417b-4b46-a5d7-fb3df23951dd)

{: .important }
> Update the certificate on all the Exchange servers as well as on the Load Balancer
