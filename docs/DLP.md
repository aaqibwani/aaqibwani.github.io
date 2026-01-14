---
title: DLP in M365
nav_order: 18
---


## What is a DLP Policy?

In an era where a single misdirected email can lead to a multi-million-dollar compliance fine, **Data Loss Prevention (DLP)** becomes a necessity. 
A DLP policy is an automated "digital compliance officer." It monitors outgoing and incoming emails for sensitive information (like credit card numbers or medical IDs) and takes action based on the rules you define.

### The Anatomy of a Policy


Every policy consists of the following components:


Rule Component | Detail | Strategic Application
-- | -- | --
Conditions | Sensitive Information Types (SITs), Sensitivity Labels, Recipient Domains, Sender Scope | Identifying PII, PHI, or internal-only markings in outbound traffic.
Actions | Block Email, Encrypt Message, Forward for Approval, Add Disclaimer | Preventing unauthorized data egress while allowing for management oversight.
Exceptions | Sender is member of specific group, Recipient domain is trusted partner | Reducing false positives for established business-to-business workflows.
User Notifications | Policy Tips, Email Notifications to Sender | Providing real-time education to users at the moment of data entry.
Alerting | Incident Reports, Admin Alerts | Enabling SOC teams to respond to potential data breaches immediately.


The Exchange Online engine supports advanced mail-flow conditions that are unique to the email workload. These include evaluating the sender's IP address, checking for password-protected attachments, and identifying specific message headers. Furthermore, it also implements a "true file type" detection model, which evaluates the actual content (type/extension) of an attachment rather than relying on potentially misleading file extensions (like filename.exe.txt).


### Licensing Tier Feature Comparison



Feature Category | Microsoft 365 E5 / Purview Suite | Microsoft 365 E3 | Office 365 E5 | Office 365 E3 | Microsoft 365 Business Premium
-- | -- | -- | -- | -- | --
Basic Exchange Online DLP | Supported | Supported | Supported | Supported | Supported
Manual Sensitivity Labeling | Supported | Supported | Supported | Supported | Supported
Automatic Sensitivity Labeling | Supported | Not Supported | Supported | Not Supported | Not Supported
Exact Data Match (EDM) | Supported | Requires Add-on | Supported | Requires Add-on | Not Supported
Endpoint DLP Actions | Supported | Not Supported | Not Supported | Not Supported | Supported
OCR for Image Inspection | Supported | Requires Add-on | Supported | Requires Add-on | Not Supported
Adaptive Protection Scopes | Supported | Not Supported | Not Supported | Not Supported | Not Supported
Trainable Classifiers | Supported | Not Supported | Supported | Not Supported | Not Supported



### Technical Limits, Caveats, and Known Issues


Limit Description | Constraint Value | Administrative Impact
-- | -- | -- 
Max DLP Rules per Tenant | 600 | Requires strategic consolidation of rules to avoid hitting the ceiling.
Max Size of a DLP Policy | 100 KB | Restricts the number of complex regex patterns or large SIT lists in a single policy.
Max Text Scanned from File | 2,000,000 Characters | Extremely large attachments may result in partial scans and a "scanning incomplete" signal.
Max SITs per Rule (EXO/SPO) | 125 | Limits the breadth of detection types in a single rule object.
Max Regex Size (predicted) | 20 KB | High-complexity custom SITs may require optimization to fit within the memory limits.
Policy Name Length | 64 Characters | Demands a concise yet descriptive naming convention for organizational clarity.


Additionally,

* **S/MIME Conflicts**: Because S/MIME encrypts the email on the client device before it reaches the Exchange server, the service-side DLP engine cannot "see" inside the encrypted packet. Consequently, S/MIME effectively bypasses service-side DLP scanning unless the policy is configured to block all S/MIME traffic.
* **Purview Message Encryption (OME)**: This is the preferred method for DLP integration. The service can scan the plaintext message in the transport pipeline and then apply encryption as a DLP action, ensuring both protection and compliance.   
* **Password-Protected Files**: Traditional DLP engines cannot scan the content of password-protected ZIP or Office files. Administrators should consider creating a specific rule to "Block" or "Audit" any attachment where the property Attachment is password protected is true.   
* **Mail Flow Bifurcation**: When a single email is addressed to both internal and external recipients, the DLP engine may "bifurcate" the message. For example, the internal recipients might receive the email immediately, while the external copy is blocked or redirected based on policy actions. Exchange email won't be sent to recipients in the fork matching the rules. Use [NonBifurcatingAccessScope](https://learn.microsoft.com/en-us/powershell/module/exchange/set-dlpcompliancerule?view=exchange-ps#-nonbifurcatingaccessscope) to block all the recipients present in the original message. 


---
