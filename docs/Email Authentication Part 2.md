---
title: Email Authentication - Troubleshooting Email Authentication issues
nav_order: 8
---
# Troubleshooting Email Authentication issues

On a high level, email authentication issues can happen when:

* You are sending emails to external recipients and the external recipients email server rejects/quarantines it OR
* You are receiving emails from external recipients and your mail server rejects/quarantines it

In both the above cases, the first and the foremost thing with respect to Exchange Online is analyzing the Message trace. For the purpose of email authentication issues, you might not need extended message trace details to find the cause of the issue. 

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

