# PCI DSS Scope Determination Document

## Document Information
| Field | Value |
|-------|-------|
| Organization | [Company Name] |
| Assessment Date | [Date] |
| Assessor | [Your Name] |
| Version | 1.0 |

---

## 1. Executive Summary

### Scope Statement
[One paragraph describing the boundaries of the CDE]

### Key Findings
- [ ] Finding 1
- [ ] Finding 2
- [ ] Finding 3

### Overall Risk Rating
[ ] Low | [ ] Medium | [ ] High | [ ] Critical

---

## 2. Cardholder Data Environment Definition

### 2.1 Systems That STORE Cardholder Data

| System Name | Data Types Stored | Retention Period | Justification |
|-------------|------------------|------------------|---------------|
|Cardholder Database Server |PAN, transaction/payment related data |Unknown |Primary location for cardholder data used during payment processing. |
|Backup System |Bakcup copies of payment releated databases and stored CHD |Unknown |May store backup copies of systems containing CHD for recovery and retention purposes |

### 2.2 Systems That PROCESS Cardholder Data

| System Name | Processing Function | Data Elements | Justification |
|-------------|---------------------|---------------|---------------|
|Payment Application Server |Processes payment transaction |PAN, payment transation data |Receives and processes customer payment information during transaction handling. |
|Tokenization Service |Replaces PAN with tokens |PAN/tokenized payment data |Processes CHD to generate tokens that reduce exposure of cardholder data. |
|HSM Appliance | Performs encryption/cryptographic operations | Encrypted CHD,Cryptographic keys | Provides encryption and cryptographic services used to protect CHD during processing and storage |

### 2.3 Systems That TRANSMIT Cardholder Data

| System Name | Transmission Path | Encryption | Justification |
|-------------|-------------------|------------|---------------|
|Payment App Server | Payment systems <-> Tokenization/HSM/Database | Internal encryption machanisms likely used | Transmits CHD between payment-processing components, tokenization services, and storage systems. |
|Web Server |Customer -> Web Server -> Payment Application Server |Likely TLS/HTTPS |Recieves customer payment information and transmits CHD into the CDE for transaction processing. |
|Load Balancer |Web traffic distribution to payment systems |Likely TLS/HTTPS |Distributes incoming payment-related traffic to systems involved in processing CHD. |

---

## 3. Connected Systems Assessment

### 3.1 Direct Connections to CDE

| System | Connection Type | Business Purpose | In Scope? | Justification |
|--------|-----------------|------------------|-----------|---------------|
| | | | Yes/No | |
| | | | Yes/No | |

### 3.2 Indirect Connections (Via Intermediary)

| System | Connection Path | Business Purpose | In Scope? | Justification |
|--------|-----------------|------------------|-----------|---------------|
| | | | Yes/No | |
| | | | Yes/No | |

---

## 4. Security-Impacting Systems

### 4.1 Systems That Could Impact CDE Security

| System | Security Function | Impact if Compromised | In Scope? | Justification |
|--------|-------------------|----------------------|-----------|---------------|
| | | | Yes/No | |
| | | | Yes/No | |

---

## 5. Trust Relationship Analysis

### 5.1 Authentication Dependencies

| CDE System | Authentication Source | Risk if Compromised | Mitigation |
|------------|----------------------|---------------------|------------|
| | | | |
| | | | |

### 5.2 Shared Services

| Service | Used By CDE | Used By Non-CDE | Segmentation Effective? |
|---------|-------------|-----------------|------------------------|
| DNS | | | Yes/No |
| NTP | | | Yes/No |
| AD | | | Yes/No |
| Backup | | | Yes/No |
| Logging | | | Yes/No |
| Patching | | | Yes/No |

---

## 6. Segmentation Assessment

### 6.1 Segmentation Controls

| Control | Description | Effectiveness | Evidence |
|---------|-------------|---------------|----------|
| Firewall rules | | Strong/Weak/None | |
| VLAN separation | | Strong/Weak/None | |
| Access controls | | Strong/Weak/None | |
| Monitoring | | Strong/Weak/None | |

### 6.2 Segmentation Gaps

| Gap ID | Description | Risk | Remediation |
|--------|-------------|------|-------------|
| GAP-001 | | High/Med/Low | |
| GAP-002 | | High/Med/Low | |
| GAP-003 | | High/Med/Low | |

---

## 7. QSA Challenge Preparation

### 7.1 Anticipated Questions

| Area | Likely Question | Your Response | Evidence |
|------|-----------------|---------------|----------|
| Shared Services | | | |
| Admin Access | | | |
| Logging | | | |
| Backups | | | |

---

## 8. Recommendations

### 8.1 Immediate Actions (0-30 days)
1. [ ]
2. [ ]
3. [ ]

### 8.2 Short-Term Actions (30-90 days)
1. [ ]
2. [ ]
3. [ ]

### 8.3 Long-Term Actions (90+ days)
1. [ ]
2. [ ]
3. [ ]

---

## 9. Residual Risk Statement

After implementing recommendations, the following risks remain:

| Risk | Likelihood | Impact | Acceptance Required? |
|------|------------|--------|---------------------|
| | | | Yes/No |
| | | | Yes/No |

---

## Appendices

### A. Network Diagram
[Insert or reference diagram]

### B. Data Flow Diagram
[Insert or reference diagram]

### C. Evidence Index
| Evidence ID | Description | Location |
|-------------|-------------|----------|
| | | |

---

## Approval

| Role | Name | Date | Signature |
|------|------|------|-----------|
| Assessor | | | |
| Reviewer | | | |
| Approver | | | |
