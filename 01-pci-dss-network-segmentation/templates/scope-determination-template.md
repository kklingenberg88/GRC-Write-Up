# PCI DSS Scope Determination Document

## Document Information
| Field | Value |
|-------|-------|
| Organization | Emerald Retail |
| Assessment Date | 7/28/2026 |
| Assessor | Kevin Klingenberg |
| Version | 1.0 |

---

## 1. Executive Summary

### Current Network Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           INTERNET                                          │
└───────────────────────────────┬─────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                         PERIMETER FIREWALL                                  │
│                         (Palo Alto PA-3400)                                 │
└───────────────────────────────┬─────────────────────────────────────────────┘
                                │
                ┌───────────────┴───────────────┐
                │                               │
                ▼                               ▼
┌───────────────────────────┐   ┌───────────────────────────────────────────┐
│        DMZ ZONE           │   │              CORPORATE ZONE               │
│      (10.0.1.0/24)        │   │             (10.2.1.0/16)                 │
├───────────────────────────┤   ├───────────────────────────────────────────┤
│                           │   │                                           │
│  ┌─────────────────────┐  │   │  ┌─────────────────┐  ┌────────────────┐  │
│  │   Web Servers (3)   │  │   │  │  AD Domain      │  │  File Servers  │  │
│  │   10.0.1.10-12      │  │   │  │  Controllers    │  │  10.2.1.20-22  │  │
│  └─────────────────────┘  │   │  │  10.2.1.10-11   │  └────────────────┘  │
│                           │   │  └─────────────────┘                      │
│  ┌─────────────────────┐  │   │                                           │
│  │   Load Balancer     │  │   │  ┌─────────────────┐  ┌────────────────┐  │
│  │   10.0.1.5          │  │   │  │  Jump Host      │  │  Workstations  │  │
│  └─────────────────────┘  │   │  │  10.2.1.50      │  │  10.2.2.0/24   │  │
│                           │   │  └─────────────────┘  └────────────────┘  │
└───────────────────────────┘   └───────────────────────────────────────────┘
                │
                │ (Claimed Segmentation Boundary)
                ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                    CARDHOLDER DATA ENVIRONMENT (CDE)                        │
│                           (10.3.0.0/24)                                     │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────────────┐  ┌─────────────────────┐  ┌─────────────────────┐  │
│  │   Payment App       │  │   Card Database     │  │   HSM Appliance     │  │
│  │   Server            │  │   Server            │  │                     │  │
│  │   10.3.0.10         │  │   10.3.0.20         │  │   10.3.0.30         │  │
│  └─────────────────────┘  └─────────────────────┘  └─────────────────────┘  │
│                                                                             │
│  ┌─────────────────────┐  ┌─────────────────────┐                           │
│  │   Tokenization      │  │   Log Collector     │                           │
│  │   Service           │  │  (Splunk Forwarder) │                           │
│  │   10.3.0.40         │  │   10.3.0.50         │                           │
│  └─────────────────────┘  └─────────────────────┘                           │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
                                        │
                                        ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                         MANAGEMENT ZONE                                     │
│                          (10.4.0.0/24)                                      │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────────────┐  ┌─────────────────────┐  ┌─────────────────────┐  │
│  │   SIEM (Splunk)     │  │   Backup Server     │  │   Patch Management  │  │
│  │   10.4.0.10         │  │   10.4.0.20         │  │   (WSUS)            │  │
│  │                     │  │                     │  │   10.4.0.30         │  │
│  └─────────────────────┘  └─────────────────────┘  └─────────────────────┘  │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```
### Scope Statement
This assessment encompasses the Emerald Retail's payment processing environment. This encompasses all associated infrastructure utilized to store, process, or transmit cardholder data. The scope includes all system components within the client environment. Systems that may be out-of-scope are separated via verified and tested network segmentation controls ensuring no direct or indirect communication paths exist to non-compliant segments. Further analysis will show us our flaws and potential areas of movement to analyze potential risk and losses.

GRC's responsibility: What needs to be asked?
- Which systems store, process, or transmit cardholder data?
- Which systems are "connected to" systems that handle CHD?
- What about systems that could impact the security of the CDE?
- Where does cardholder data actually flow?

**Deliverable:** A data flow diagram showing actual CHD movement
Note: Mark with a legend when presenting to avoid questions such as:
What stores/processes/transmits the information vs Transmits?
Does this data move one-way or is it transmitted in two directions?

```
EXAMPLE DATA FLOW ANALYSIS:

Customer Browser
       │
       ▼ (HTTPS - PAN entered)
┌──────────────┐
│  Web Server  │ ─────────────────────────────────────────┐
└──────────────┘                                          │
       │                                                  │
       ▼ (API Call - PAN in transit)                      │
┌──────────────┐                                          │
│ Payment App  │                                          │
└──────────────┘                                          │
       │                                                  │
       ├──────────────────────┐                           │
       │                      │                           │
       ▼                      ▼                           ▼
┌──────────────┐    ┌──────────────┐            ┌──────────────┐
│     HSM      │    │   Card DB    │            │  Log Server  │
│ (Encryption) │    │  (Storage)   │            │  (Contains   │
└──────────────┘    └──────────────┘            │   PAN logs?) │
                                                └──────────────┘

QUESTION: Is the log server in scope? What data does it receive?
```

**Areas to Investigate:**

| Connection Type   |   Question |   Risk Level   |
|-------------------|------------|----------------|
| Authentication | Does CDE use corporate AD? | HIGH |
| DNS | Where do CDE systems resolve DNS? | MEDIUM |
| Time Sync | NTP source for CDE systems? | LOW |
| Patching | How are CDE systems patched? | HIGH |
| Backup | Where do backups go? Do they contain CHD? | HIGH |
| Logging | Where do logs flow? What's in them? | MEDIUM |
| Administration | Who has admin access? From where? | HIGH |

**Deliverable(s):** Trust relationship matrix with risk assessment
Note: I usually go one step further with a little more information on findings, and usually include remediation where I can.
Giving the client insight to vulnerabilities with a solid "Why" always gives the proper push. Huge bonus if you can list an attack vector.


### Example Key Findings
- [X] CWE-327: Use of a Broken or Risky Cryptographic Algorithm (MD-5 hashing found)
  Risk: High
  Score: 7.5
  Business Impact: Data confidentiality and integrity if attackers decrypt weak ciphers or break obsolete hashing functions
  Note: This increases to High/Critical if used for session tokens, password hashing, or protecting high-value regulated data
  Remediation: Use a more secure hashing algorithm, MD-5 is known to be compromised
  
- [X] CWE-284: Improper Access Control (Priority 1) 
  Risk: Critical
  Score: 9.8
  Business Impact:  Allows actors to read, modify, or delete sensitive resources outside their permission scope, bypassing security boundaries
  Note: This can permit unprivileged remote code execution, data theft, or complete system takeover
  Remediation: Immediately review permissions and implement least privilege, separation of duties and zero trust models if possible
  
- [X] CWE-311: Missing Encryption of Sensitive Data
  Risk: High
  Score 7.5
  Business Impact: Exposes unencrypted sensitive information (like passwords, PII, or financial records) to network sniffing or local storage theft
  Note: If data requires internal network access to intercept, but escalates to High or Critical if plain-text credentials or session secrets are exposed publicly
  Remediation: Implement at rest and in-transit encryption. 

### Overall Risk Rating
[ ] Low | [ ] Medium | [X] High | [X] Critical

---

## 2. Cardholder Data Environment Definition

| System               | Stores CHD/PAN | Processes CHD/PAN | Transmits CHD/PAN | Justification                                                                                                                                                          |
| -------------------- | :------------: | :---------------: | :---------------: | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Customer Browser     |       No*      |        Yes        |        Yes        | User enters PAN and submits it via HTTPS. Browser is not part of the CDE.                                                                                              |
| Load Balancer        |       No       |         No        |        Yes        | Forwards encrypted traffic to web servers.                                                                                                                             |
| Web Servers          |       No       |      Limited      |        Yes        | Terminates or proxies HTTPS and forwards requests to the Payment Application. Whether it "processes" depends on application architecture, but it definitely transmits. |
| Payment Application  |       No       |      **Yes**      |      **Yes**      | Performs payment logic and communicates with HSM, database, and tokenization service.                                                                                  |
| HSM Appliance        |       No       |      **Yes**      |        Yes        | Performs cryptographic operations on PAN; does not permanently store cardholder data.                                                                                  |
| Card Database        |     **Yes**    |        Yes        |        Yes        | Primary storage location for PAN and queried by the payment application.                                                                                               |
| Tokenization Service |       No       |      **Yes**      |        Yes        | Receives PAN, generates tokens, returns tokenized value.                                                                                                               |
| Log Collector        |       No†      |         No        |      Possible     | Should not contain PAN. If improperly configured, PAN could appear in logs.                                                                                            |
| SIEM                 |       No†      |         No        |         No        | Receives security logs only. PAN should never reach the SIEM.                                                                                                          |
| Backup Server        |     **Yes**    |         No        |        Yes        | Stores encrypted backups containing cardholder data.                                                                                                                   |
| WSUS                 |       No       |         No        |         No        | Administrative only.                                                                                                                                                   |
| Domain Controllers   |       No       |         No        |         No        | Authentication services only.                                                                                                                                          |
| Jump Host            |       No       |         No        |         No        | Administrative access only.                                                                                                                                            |
| File Servers         |       No       |         No        |         No        | No documented interaction with cardholder data.                                                                                                                        |
| Workstations         |       No       |         No        |         No        | Administrative endpoints only unless otherwise defined.                                                                                                                |


---
### Common Questions: 
Is the log server in scope? 
What data does it receive?
Is data transmitted encrypted?


## 3. Connected Systems Assessment

## CDE Communication Matrix

| System | Communication with CDE | Purpose | PCI Scope |
|----------|------------------------|---------|-----------|
| **Payment Application** | Direct | Payment processing and business logic | In Scope (CDE) |
| **Card Database** | Direct | Stores cardholder data (PAN) | In Scope (CDE) |
| **HSM Appliance** | Direct | Cryptographic operations and key management | In Scope (CDE) |
| **Tokenization Service** | Direct | Token generation and PAN protection | In Scope (CDE) |
| **Web Servers** | Direct | Forward payment requests to the Payment Application | Connected-to-CDE |
| **Log Collector** | Direct | Collect security logs from CDE systems | Connected-to-CDE |
| **Backup Server** | Direct | Backup and recovery of CDE systems | Connected-to-CDE |
| **WSUS / Patch Management** | Direct | Patch deployment to CDE systems | Connected-to-CDE |
| **Jump Host** | Direct | Administrative access into the CDE | Connected-to-CDE |
| **Active Directory Domain Controllers** | Direct | Authentication and authorization services | Connected-to-CDE |
| **Customer Browser** | Indirect | Connects only to the Web Server | Out of Scope |
| **Load Balancer** | Indirect | Routes traffic to Web Servers | Connected Infrastructure |
| **SIEM (Splunk)** | Indirect | Receives forwarded security logs | Connected Infrastructure |
| **File Servers** | None Documented | No documented communication path | Out of Scope |
| **Corporate Workstations** | None Documented | Administration occurs through the Jump Host | Out of Scope |


### Communication Summary

| Classification | Systems |
|----------------|---------|
| **CDE Systems** | Payment Application, Card Database, HSM Appliance, Tokenization Service |
| **Direct Communication with CDE** | Web Servers, Log Collector, Backup Server, WSUS, Jump Host, Active Directory Domain Controllers |
| **Indirect Communication with CDE** | Customer Browser, Load Balancer, SIEM |
| **No Documented Communication** | File Servers, Corporate Workstations |

---

## 4. Security-Impacting Systems

### 4.1 Systems That Could Impact CDE Security

| Source Zone         | Source System        | Destination System     | Trust Relationship     | Data Classification              | CDE Impact                          | Security Controls                        |
| ------------------- | -------------------- | ---------------------- | ---------------------- | -------------------------------- | ----------------------------------- | ---------------------------------------- |
| Internet            | Customer Browser     | Web Servers            | HTTPS (443)            | Cardholder Data (PAN in transit) | **Direct**                          | TLS, WAF, Palo Alto Policy               |
| DMZ                 | Web Servers          | Payment Application    | HTTPS/API              | PAN                              | **Direct**                          | Firewall ACL, Application Authentication |
| Payment Application | HSM                  | Cryptographic Services | Encryption Keys        | **Critical CDE**                 | HSM Access Controls, Key Management |                                          |
| Payment Application | Card Database        | SQL/TLS                | PAN Storage            | **Critical CDE**                 | Database Encryption, RBAC           |                                          |
| Payment Application | Tokenization Service | HTTPS/API              | Token Requests         | **Direct**                       | Service Authentication              |                                          |
| Payment Application | Log Collector        | Syslog/TLS             | Application Logs       | **Potential CDE**                | Log Sanitization, TLS               |                                          |
| Log Collector       | SIEM                 | Forwarded Logs         | Security Events        | Potential PAN Exposure           | Log Filtering                       |                                          |
| Management Zone     | WSUS                 | Payment Application    | Windows Updates        | Administrative                   | **Administrative Trust into CDE**   | Firewall Rules, Patch Approval           |
| Management Zone     | Backup Server        | Card Database          | Backup Operations      | Encrypted PAN                    | **Direct CDE**                      | Encrypted Backup, Access Control         |
| Corporate Zone      | Jump Host            | Payment Application    | Administrative RDP/SSH | Administrative                   | **High Trust**                      | MFA, PAM, Session Logging                |
| Domain Controllers  | Payment Application  | Kerberos/LDAP          | Authentication         | Administrative                   | **High Trust**                      | Tiered Administration                    |
| Payment Application | Splunk Forwarder     | Log Forwarding         | Security Events        | Potential PAN                    | Log Filtering                       |                                          |



---

## 5. Trust Relationship Analysis

### 5.1 Authentication Dependencies

| Trust Boundary   | Crossing Systems                         | Business Purpose    | Risk     | Notes                              |
| ---------------- | ---------------------------------------- | ------------------- | -------- | ---------------------------------- |
| Internet → DMZ   | Browser → Web Server                     | Public Access       | High     | Internet-facing entry point        |
| DMZ → CDE        | Web Server → Payment Application         | Payment Processing  | Critical | Primary CDE ingress                |
| Corporate → CDE  | Jump Host → Payment Application          | Administration      | Critical | Requires MFA and restricted access |
| Corporate → CDE  | Domain Controllers → Payment Application | Authentication      | High     | Expands PCI scope if unrestricted  |
| Management → CDE | WSUS → Payment Application               | Patch Management    | High     | Administrative trust relationship  |
| Management → CDE | Backup Server → Card Database            | Backup Operations   | Critical | Handles stored PAN                 |
| CDE → Management | Log Collector → SIEM                     | Security Monitoring | Medium   | Ensure PAN redaction               |


### 5.2 Shared Services

Shared services provide centralized enterprise functionality while supporting systems across multiple security zones. These systems do not directly participate in payment processing but are required to maintain, monitor, and administer the Cardholder Data Environment (CDE). As such, they are considered Connected-to-CDE systems and must be appropriately secured to prevent unauthorized access into the CDE.

### Shared Services Included

| Service | Purpose | Security Considerations |
|----------|---------|-------------------------|
| Active Directory | Centralized authentication and authorization | Tiered administration, least privilege, MFA |
| DNS | Internal name resolution | Restrict zone transfers, monitor DNS queries |
| NTP | Time synchronization | Secure time source for audit log integrity |
| WSUS / Patch Management | Operating system and application patching | Controlled deployment and change management |
| SIEM (Splunk) | Centralized security monitoring | Log integrity, access controls, retention |
| Backup Server | Backup and disaster recovery | Encryption, offline copies, access restrictions |
| Jump Host | Administrative access into secured environments | MFA, session recording, privileged access management |
| File Services | Shared documentation and software repository | Role-based access control and auditing |

---

## 6. Segmentation Assessment

### 6.1 Segmentation Controls

| Control | Description | Effectiveness | Evidence |
|---------|-------------|---------------|----------|
| Firewall Rules | Restrict traffic between Internet, DMZ, Corporate, Management, and CDE zones to only required ports, protocols, and services. | Strong | Palo Alto firewall policies, rule review, configuration backup |
| VLAN Separation | Separate network zones using dedicated VLANs and routed interfaces to prevent unrestricted lateral movement. | Strong | Network diagrams, switch configuration, VLAN assignments |
| Access Controls | Restrict administrative and application access using least privilege, RBAC, MFA, and Jump Host administration. | Strong | Active Directory groups, MFA configuration, access reviews |
| Monitoring | Monitor network traffic, authentication events, firewall logs, and administrative activity through centralized logging. | Strong | SIEM dashboards, firewall logs, Windows Event Logs, audit reports |

### 6.2 Segmentation Gaps

| Finding | Risk | Recommendation |
|---------|------|----------------|
| Active Directory directly authenticates CDE systems | Compromise of AD may impact the entire CDE. | Implement Tier-0 administration and privileged access controls. |
| WSUS communicates directly with CDE assets | Patch infrastructure becomes part of PCI scope. | Restrict communication to approved update windows and firewall rules. |
| Backup Server accesses Cardholder Data | Backup compromise may expose encrypted PAN. | Encrypt backups, restrict administrative access, perform restore testing. |
| Jump Host provides administrative access into CDE | Administrative compromise provides direct access to sensitive systems. | Require MFA, session recording, and privileged access management. |
| Log forwarding from CDE | Sensitive information may be inadvertently forwarded. | Verify PAN redaction and log filtering before forwarding. |
| Shared authentication infrastructure | Authentication services become security dependencies of the CDE. | Separate privileged accounts and monitor authentication events. |
| Firewall rule expansion over time | Excessive allow rules weaken segmentation. | Conduct periodic firewall rule recertification and cleanup. |
| Management network trust | Administrative systems may become attack paths. | Separate management traffic and enforce least privilege. |

---

## 7. QSA Challenge Preparation

| Question | Purpose |
|----------|---------|
| What systems store, process, or transmit cardholder data? | Define PCI scope. |
| What systems can directly communicate with the CDE? | Validate segmentation boundaries. |
| How is administrative access into the CDE controlled? | Review privileged access controls. |
| What firewall rules permit traffic into and out of the CDE? | Verify least-privilege network access. |
| How is cardholder data encrypted in transit and at rest? | Validate cryptographic controls. |
| How are security patches deployed and verified? | Assess vulnerability management practices. |
| How are vulnerabilities identified, prioritized, and remediated? | Review the vulnerability management lifecycle. |
| How are audit logs collected, protected, and reviewed? | Verify logging and monitoring controls. |
| What evidence demonstrates that segmentation is effective? | Confirm systems outside the CDE cannot access protected assets. |
| How are POA&Ms, exceptions, and remediation activities tracked? | Evaluate governance and continuous compliance. |

---

## 8. Recommendations

### 8.1 Immediate Actions (0-30 days)
These would be reserved for your critical findings, you never want those to go past this timeline.

### 8.2 Short-Term Actions (30-90 days)
This is reserved for high priority, and sometimes urgent Medium severity findings.

### 8.3 Long-Term Actions (90+ days)
Usually you will find most Medium and Low severity findings here. You should know and be aware of their existence, even if they're not top priority.

---

## 9. Residual Risk Statement

After implementing recommendations, the following risks remain:
You can use a table like this to address residual risk - not for critical findings, but risk that remains in light of remediation

| Risk | Likelihood | Impact | Acceptance Required? |
|------|------------|--------|---------------------|


---

## Approval

Lastly we come to process tracking. This is used as a paper trail when auditing.

| Role | Name | Date | Signature |
|------|------|------|-----------|
| Assessor | | | |
| Reviewer | | | |
| Approver | | | |
