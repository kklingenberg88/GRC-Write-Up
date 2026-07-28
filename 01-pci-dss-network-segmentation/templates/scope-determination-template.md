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
| | | | |
| | | | |

### 2.2 Systems That PROCESS Cardholder Data

| System Name | Processing Function | Data Elements | Justification |
|-------------|---------------------|---------------|---------------|
| | | | |
| | | | |

### 2.3 Systems That TRANSMIT Cardholder Data

| System Name | Transmission Path | Encryption | Justification |
|-------------|-------------------|------------|---------------|
| | | | |
| | | | |

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
