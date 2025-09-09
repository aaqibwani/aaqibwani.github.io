---
title: Email Authentication - Troubleshooting Email Authentication issues
nav_order: 8
---
# Email Authentication Part 2 of 2: Troubleshooting Email Authentication issues

Since the emails can be either outbound or inbound, there can be 2 scenarios:

* Outbound: You are sending emails to external recipients and the external recipients email server rejects, quarantines or sends the email to the SPAM/JUNK folder
* Inbound: You are receiving emails from external recipients and your mail server rejects, quarantines or sends the email to the SPAM/JUNK folder

In the first part of this article, we understood what SPF, DKIM and DMARC are and how the Anti-Spam agents on the mail servers determine if SPF, DKIM or DMARC PASS or FAIL. Let's now understand them with an example:

* I use M365 and have a domain idiotbox.in added in my M365 tenant
* The MX is pointed to M365
* I sent an email from my gmail account to my M365 account
* Since, gmail is managed by Google, the corresponding SPF, DKIM and DMARC records are also managed by Google
* I received the email in my Inbox and copied the messahe headers and pasted them in https://mha.azurewebsites.net/ to analyze, let's see what we find:

<img width="1827" height="435" alt="image" src="https://github.com/user-attachments/assets/24849a0e-af39-4f05-bef0-af2ab73718d1" />

* We see the the email was sent by Google mail server with IP address 2a00:1450:4864:20::630 to M365

<img width="1526" height="378" alt="image" src="https://github.com/user-attachments/assets/add3354b-ceca-432f-8b1d-d157e1719c31" />

* Here we see that the SCL value set by EOP was 1 (Not SPAM) and SPAM filtering verdict was NSPM (Not SPAM). And we also see the connecting IP correctly set as the IP of the original sender.

<img width="1882" height="281" alt="image" src="https://github.com/user-attachments/assets/dc9087a9-9e9a-4cf9-a285-0451014bc639" />

* Now if we check the Authentication-Results field, we see that SPF, DKIM and DMARC all passed, which is good. But we also need to understand why and how they passed.

### SPF:

SPF is used to verify if the mail server is authorized to send emails from the sender domain. SPF passes when the connecting IP is present in the SPF record of the mailfrom domain. Let's check the mailfrom (Return-Path NOT "FROM address") domain and the SPF record of gmail.com

Authentication-Results:	**spf=pass (sender IP is 2a00:1450:4864:20::630) smtp.mailfrom=gmail.com**; dkim=pass (signature was verified) header.d=gmail.com;dmarc=pass action=none header.from=gmail.com;compauth=pass reason=100

<img width="602" height="50" alt="image" src="https://github.com/user-attachments/assets/d7976802-74d2-4089-a775-74a6c5158c28" />

<img width="1423" height="191" alt="image" src="https://github.com/user-attachments/assets/c86d5639-5886-40e9-abae-ce3b2e2e3281" />


As we can see the mailfrom domain is gmail.com and we we do a SPF lookup on it using MXtoolbox, it has the IPv6 subnet 2a00:1450:4000::/36 mentioned which includes the 2a00:1450:4864:20::630 IPv6 address, _hence SPF passes_.

### DKIM:

DKIM is used to verifiy the integrity of the email to ensure that the email has not been tampered with. When the sender sends the email, a hash is calculated based on the email properties (From, Subject, Body, etc.) using a private key and when the recipient server receives the email, it also calculates the hash but based on the public key mentioned in the DKIM record. If the hash matches, DKIM passes.

Authentication-Results:	spf=pass (sender IP is 2a00:1450:4864:20::630) smtp.mailfrom=gmail.com; **dkim=pass (signature was verified) header.d=gmail.com**;dmarc=pass action=none header.from=gmail.com;compauth=pass reason=100

As we can see from Authentication Results header, DKIM passed and if we look in the DKIM Signature header, we can see the email was signed with selector 20230601. If we do a DKIM lookup using this selector, we will find the corresponsing DKIM record in the DNS:

<img width="1882" height="281" alt="image" src="https://github.com/user-attachments/assets/11ad80c1-124f-4cc1-b1a8-0b3fcab84a5c" />

<img width="1447" height="355" alt="image" src="https://github.com/user-attachments/assets/92f023c7-210b-4ca6-af38-8533cf61de48" />

Note:
There are common scenarios which cause DKIM to fail, example a email filtering gateway between the sender and the recipient that modifies the subject or the body of the email to either add "EXTERNAL" in the subject line or include a disclaimer in the email. However, in our case the email was sent directly from the sender to the recipient and there was no modification of any of the email properties, hence DKIM passes. 


### DMARC:

DMARC builds on SPF and DKIM. DMARC passes if either:

* SPF passes AND SPF alignment passes

     (OR)
  
* DKIM passes AND DKIM alignment passes

Authentication-Results:	spf=pass (sender IP is 2a00:1450:4864:20::630) smtp.mailfrom=gmail.com; dkim=pass (signature was verified) header.d=gmail.com;**dmarc=pass action=none** header.from=gmail.com;compauth=pass reason=100

As we can see from the Authentication Results, DMARC passed and action=none. Since both SPF/SPF alignment and DKIM/DKIM alignment passed, it was evident that DMARC will pass. We know how SPF and DKIM pass, let's also take a closer look at how SPF and DKIM alignment pass:

* SPF alignment:
The domain in the mailfrom address should match the domain in the from address:

 <img width="1882" height="281" alt="image" src="https://github.com/user-attachments/assets/f3cd66da-ec2a-46c5-9847-e0b1a7cd700e" />

<img width="901" height="223" alt="image" src="https://github.com/user-attachments/assets/fab772a1-6161-4bee-96af-cbcd4ce59e30" />

* DKIM alignment:
The domain in the header.d should match the from domain:

<img width="1882" height="153" alt="image" src="https://github.com/user-attachments/assets/e0a56e3f-c6f6-405d-aac4-33485f9cc189" />

<img width="901" height="223" alt="image" src="https://github.com/user-attachments/assets/56f4185d-1ecc-4ea7-8165-e028a9da9b95" />

The sender's DMARC record defines what to do with the emails sent from their domain that fail DMARC. This is defined by the <p> tag.

* If p=reject, it means that the sender wants the recipient to reject any emails that fail DMARC
* If p=quarantine, it means that the sender wants the recipient to quarantine any emails that fail DMARC
* If p=none, it means to do nothing

Note:
It is important to note that these verdicts in the sender's DMARC are only guidelines, the authority still lies with the recipient if they want to reject or quarantine the emails, example, the recipient can define the policy at their end to senf the emails to quarantine even though the sender has set the tag as p=reject. 

In both the above cases, the first and the foremost thing is analyzing the Message trace.  

Go to the Exchange Admin Center and under Mail Flow perform a message trace and check what happened to the email. Let's suppose the email was quarantined. We have an option to open the email in Explorer (Defender for O365) and look for the Detection technology involved (in case the email is quarantined because of other reasons than Email authentication failures). 

![Screenshot 2024-08-24 145946](https://github.com/user-attachments/assets/97df3f78-11af-43cc-b6e3-f3f51c202307)

## Analyzing Message Headers
Copy the message header from Explorer, open Message Header Analyzer https://mha.azurewebsites.net/ and click on Analyze Headers. For the purpose of this demonstration, we are assuming that the MX is pointed to M365 (without any immediate hops in between). Note the below points when analyzing message headers:

* The SPF domain is recorded in the Authentication-Results header as smtp.mailfrom, smtp.helo or smtp.ehlo and refers to the 5321.MailFrom domain or the domain provided as HELO/EHLO respectively.

* The DKIM domain is recorded in the Authentication-Results header as header.d and is specified when DKIM signing is applied to the message.

* The 5322.From address is recorded in the Authentication-Results header as header.from. This is the domain part of the From: address.


Using the above email as example, lets analyze the Message Headers:

![image](https://github.com/user-attachments/assets/e6b261f7-95a2-48cf-9fd4-71ece265ce14)

In the Summary tab, we see:

* **MessageID**, which is a unique identifier of this email. _Suppose if 2 emails with the same recipient(s), Subject and body were sent, you can differentiate them based on the MessageID._

* **From address**, which will be visible in the client (like Outlook) and on which DKIM checks will be done. 

In Received Headers tab, we see the different hops that the email took before reaching the server where the mailbox is hosted. The important information in this part is the 2nd row where we see Google server with egress IP 209.85.128.178 handing off the email to EOP. This IP from which EOP received the email is the Connecting IP (CIP).

* **Connecting IP (CIP)** is the IP on which SPF checks will be done. 


![image](https://github.com/user-attachments/assets/41cbfcb1-3ca9-4460-af97-6e8cea635467)

The Forefront Anti-Spam Report Header tab as you can see has more information, but the most important among there are:

* SCL (SPAM Confidence level): It's a score that Exchange assigns to each incoming email to determine the likelihood that the message is spam.

* The SCL score ranges from -1 to 9:
> -1: Trusted email (e.g., from an internal sender or safe sender).

> 0-1: Not likely to be spam.

> 2-4: Slightly suspicious, but not classified as spam.

> 5-6: Likely to be spam (sent to Junk Email folder).

> 7-9: Highly likely to be spam (blocked or quarantined).

* SFV (Spam filtering verdict): tells you why the email was marked as SPAM or not:

> SFV:BLK Filtering was skipped and the message was blocked because it was sent from an address in a user's Blocked Senders list.

> SFV:NSPM Spam filtering marked the message as nonspam and the message was sent to the intended recipients.

> SFV:SFE Filtering was skipped and the message was allowed because it was sent from an address in a user's Safe Senders list.

> SFV:SKA The message skipped spam filtering and was delivered to the Inbox because the sender was in the allowed senders list or allowed domains list in an anti-spam policy. 

> SFV:SKB The message was marked as spam because it matched a sender in the blocked senders list or blocked domains list in an anti-spam policy. 

> SFV:SKN The message was marked as nonspam before processing by spam filtering. For example, the message was marked as SCL -1 or Bypass spam filtering by a mail flow rule.

> SFV:SKQ The message was released from the quarantine and was sent to the intended recipients.

> SFV:SKS The message was marked as spam before processing by spam filtering. For example, the message was marked as SCL 5 to 9 by a mail flow rule.

> SFV:SPM The message was marked as spam by spam filtering.

> SRV:BULK The message was identified as bulk email by spam filtering and the bulk complaint level (BCL) threshold. When the MarkAsSpamBulkMail parameter is On (it's on by default), a bulk email message is marked as spam (SCL 6).

* IPV (IP Filter Verdict): Refers to the outcome of evaluating the reputation of the sender's IP address

> IPV:CAL The message skipped spam filtering because the source IP address was in the IP Allow List. For more information, see Configure connection filtering.

> IPV:NLI The IP address wasn't found on any IP reputation list.


* CAT (in Source Header field): The category of protection policy that's applied to the message:

> AMP: Anti-malware

> BULK: Bulk

> DIMP: Domain impersonation*

> FTBP: Anti-malware common attachments filter

> GIMP: Mailbox intelligence impersonation*

> HPHSH or HPHISH: High confidence phishing

> HSPM: High confidence spam

> INTOS: Intra-Organization phishing

> MALW: Malware

> OSPM: Outbound spam

> PHSH: Phishing

> SAP: Safe Attachments*

> SPM: Spam

> SPOOF: Spoofing

> UIMP: User impersonation*


The different values of the above parameters in the Forefront Anti-Spam Report Header are described here: https://learn.microsoft.com/en-us/defender-office-365/message-headers-eop-mdo 	

