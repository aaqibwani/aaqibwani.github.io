---
title: Email Authentication: What is SPF, DKIM and DMARC and how to set it up in M365?
nav_order: 7
---
# Email Authentication Part 1 of 2: What is SPF, DKIM and DMARC and how to set it up in M365?

## What is Email Authentication and why we need it?
Email authentication is like a security check for emails. Just as you show an ID to prove who you are, email authentication helps prove that an email is really from the person or company it says it's from. When you get an email, you want to be sure it's from a trusted source, not a scammer pretending to be someone else. Email authentication helps with this by checking the email's "ID" using several methods. If the email passes the check, it's more likely to be genuine. If it fails, it might be a fake or spam email.

Without email authentication, it would be easy for scammers to send fake emails that look like they come from your bank, company, or friends. These fake emails could trick you into giving away personal information, clicking on harmful links, or downloading viruses. Authentication helps protect you from these threats by ensuring that emails are really from who they say they are.

SPF, DKIM, and DMARC are email authentication methods used to help prevent email spoofing and phishing attacks. Here’s a basic overview of each:

1. **SPF (Sender Policy Framework):**
   - **What It Is:** A protocol that allows domain owners to specify which mail servers and IPs in a TXT record in DNS that are permitted to send emails on behalf of their domain.
   - **How It Works:** When an email is received, the recipient's mail server checks the SPF record for the domain in the MailFrom address in the DNS to see if connecting IP is present. If it does, the email passes the SPF check; if not, it may be flagged as suspicious. In Exchange Online the results of the SPF check are stored in the **Authentication-Results** and **Received-SPF** header:

2. **DKIM (DomainKeys Identified Mail):**
   - **What It Is:** A method that adds a digital signature to emails, allowing the recipient to verify that the email was indeed sent by the domain it claims to be from and that it hasn't been altered in transit.
   - **How It Works:** The sender's mail server adds a DKIM signature to the email header. The recipient’s mail server uses the public key published in the DNS of the domain in the From address to verify the signature. If the signature matches, it confirms the email's authenticity and DKIM passes.

3. **DMARC (Domain-based Message Authentication, Reporting & Conformance):**
   - **What It Is:** A protocol that builds on SPF and DKIM to provide a way for domain owners to specify how their email should be handled if it fails authentication checks. DMARC also authenticates the emails by checking if (SPF passes AND the “From” address matches the smtp.MailFrom address) OR if (DKIM passes and the From address matches the signing domain header.d). If either of these is TRUE, then DMARC passes. It also provides a mechanism for receiving reports on email authentication results.
   - **How It Works:** Domain owners publish a DMARC policy in their DNS that tells receiving mail servers how to handle emails that fail SPF and/or DKIM checks (e.g., reject or quarantine them). DMARC also allows for reporting, so domain owners can receive feedback on authentication issues.

Together, SPF, DKIM, and DMARC help protect against email fraud and ensure that emails are legitimate and secure.


***


## How to setup SPF in Exchange Online

Before adding an SPF record, identify which mail servers are authorized to send emails for your domain. For Exchange Online, you generally need to include Microsoft’s SPF record. If you are in an Exchange hybrid deployment and have LOB on-premises applications, you will also need to add the public/egress IP of your organization to the SPF record. In addition, if you use bulk email services like SendGrid, then you'll need to add SendGrid IPs as well. 

SPF is a TXT record in the DNS that specifies the authorized mail servers, examples:

```
_v=spf1 include:spf.protection.outlook.com -all_

_v=spf1 ip4:203.0.113.45 include:spf.protection.outlook.com -all_

_v=spf1 include:spf.protection.outlook.com include:otheremailservice.com -all_
```

Here’s what each part means:

* **v=spf1** — This specifies the SPF version.

* **include:spf.protection.outlook.com** — This tells receiving servers to check Microsoft’s SPF record to see if the email is coming from an authorized source.

* **ip4:203.0.113.45:** This authorizes the IP address 203.0.113.45 to send emails on behalf of your domain. The ip4: prefix indicates it's an IPv4 address. If you have an IPv6 address, you would use ip6: instead.

* **-all —** This specifies that any server not listed in the SPF record should be treated as unauthorized.

Login to your DNS provider, add or modify the SPF record:

* **Name/Host:** Typically, this is @ or left blank to apply to the root domain.
* **Type:** TXT
* **Value/Data:** Enter the SPF record value, e.g., v=spf1 ip4:203.0.113.45 include:spf.protection.outlook.com -all.

Example:
![image](https://github.com/user-attachments/assets/b39abe86-758c-460e-ba46-fbd3266ff2b2)

Check if SPF is properly published:
![image](https://github.com/user-attachments/assets/14e63934-d9ad-4d4c-912c-02fdd6978457)


***


## How to setup DKIM in Exchange Online

* The first step to setup DKIM in Exchange Online is enabling the DKIM signing for the required domain(s). To do this go to the Defender portal https://www.security.microsoft.com > Email & Collaboration > Policies & Rules > Threat Policies > Email Authentication Settings > DKIM. Click on create DKIM keys:

![image](https://github.com/user-attachments/assets/046c6c2e-2fe6-43a8-9e6f-3a3a761995af)

* It will generate the CNAME records that we need to publish in the DNS. Copy the information and paste in Notepad to be used further:

![image](https://github.com/user-attachments/assets/275527a0-22e2-4815-ba0b-f267acf47954)

* Go to your DNS provider (GoDaddy in my case) and publish the 2 CNAME records as below:

![image](https://github.com/user-attachments/assets/a193aec6-a0e7-4862-b166-035eb24910e4)

![image](https://github.com/user-attachments/assets/226ee855-c7bb-45e2-afaf-115610f6c066)

* Verify if the CNAME records are successfully published using MXtoolbox.com. Put the query as domain:selector and go the DKIM check. Exmaple:

![image](https://github.com/user-attachments/assets/098ed892-3043-49fa-bef4-fa38b363bf5c)

* Once the CNAME records are propagated, go back to the Defender portal and enable DKIM signing for the domain. Microsoft will check if the CNAME records are properly published or not. If they are, you'll receive the below message:

![image](https://github.com/user-attachments/assets/0f72526c-49bb-4bb5-849f-2f3da291c026)


***


## How to setup DMARC in Exchange Online

* Before setting up DMARC, you should decide on the policy you want to apply. Think of it as the action the recipient server will take on emails from your domain if they fail DMARC:

* **None**: Monitor only; no specific action is taken on emails that fail DMARC checks.
* **Quarantine**: Mark emails that fail DMARC checks as spam or place them in a quarantine folder.
* **Reject**: Completely reject emails that fail DMARC checks, preventing them from being delivered.

Login to your DNS console, and add a new TXT Record:

* **Name/Host**: _dmarc.yourdomain.com (Replace yourdomain.com with your actual domain name.)
* **Type**: TXT
* **Value/Data**: The DMARC policy record. Here’s a basic example:
```
_v=DMARC1; p=none; rua=mailto:dmarc-reports@yourdomain.com_

_v=DMARC1; p=quarantine; rua=mailto:admin@idiotbox.in; ruf=admin@idiotbox.in; pct=100; sp=none_
```
![image](https://github.com/user-attachments/assets/17f3de38-2775-4e40-95f2-ea8442cb7dae)

* Check if DMARC is properly published:

![image](https://github.com/user-attachments/assets/eb310ddf-48c9-4140-8177-4af609d9a375)


Explanation:

* v=DMARC1: Specifies the DMARC version.
* p=none: Sets the policy to "none", meaning no specific action is taken on emails that fail DMARC checks (use p=quarantine or p=reject for stricter policies).
* rua=mailto:dmarc-reports@yourdomain.com: Specifies the email address where aggregate DMARC reports will be sent. Make sure to replace dmarc-reports@yourdomain.com with your actual reporting email address.

* If you want a more advanced setup, you can include additional tags:

```
_v=DMARC1; p=quarantine; rua=mailto:dmarc-reports@yourdomain.com; ruf=mailto:dmarc-forensic@yourdomain.com; pct=100; sp=none_
```

Explanation:

* p=quarantine: Sets the policy to quarantine emails that fail DMARC checks.
* ruf=mailto:dmarc-forensic@yourdomain.com: Specifies an email address for forensic (detailed) reports.
* pct=100: Applies the DMARC policy to 100% of emails. (You can set this to a lower percentage if you want to gradually implement DMARC.)
* sp=none: Sets a different policy for subdomains (optional).
