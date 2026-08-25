# Project 05: Writing a Risk Acceptance for a Security Gap

## Overview

Every organization has risks it chooses to live with. The difference between strong and weak GRC is whether those decisions are **intentional and defensible**. This project requires you to write a risk acceptance that enables informed decision-making, not one that hides behind jargon or vague language.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    RISK ACCEPTANCE: FAILURE MODES                           │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   COMMON FAILURES                       WHAT SHOULD HAPPEN                  │
│   ───────────────                       ────────────────────                │
│                                                                             │
│   Drowning in technical jargon          Plain language explanation          │
│   "Credential stuffing attack           "Attackers may guess passwords      │
│   surface with lateral movement         and access other systems"           │
│   potential"                                                                │
│                                                                             │
│   Hiding behind vague language          Specific, evidenced statements      │
│   "Risk is low"                         "3 similar incidents in industry    │
│   "Controls are in place"               last year, each cost $2-5M"         │
│                                                                             │
│   Selling the decision                  Honest trade-off explanation        │
│   Downplays impact, inflates            States what could go wrong,         │
│   control effectiveness                 acknowledges uncertainty            │
│                                                                             │
│   Treating it as permission slip        Treating it as business decision    │
│   Security "approving" the risk         Business OWNING the risk            │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## The Scenario

**Company:** FinanceFlow Inc.
**Industry:** B2B Payment Processing
**Size:** 400 employees, $50M ARR
**Compliance:** SOC 2 Type II, PCI DSS Level 2

### The Security Gap

The security team has identified that the company's legacy payment reconciliation system:

1. Uses a shared service account with database admin privileges
2. The password has not been rotated in 18 months
3. 15 team members know the password
4. The system processes $2B in transactions annually
5. Replacing the system would require 12-18 months of development

**Technical Details:**
- System: PaymentReconciler v3.2 (internal application)
- Database: Oracle 12c with PCI cardholder data
- Authentication: Static username/password stored in config file
- Logging: Application logs exist, but no database query logging
- Network: System sits in PCI CDE

**Business Context:**
- Finance team relies on this system for daily operations
- No budget allocated for replacement in current fiscal year
- Previous attempt to modernize failed 2 years ago
- System owner is the CFO

---

## What You Need to Do

### Phase 1: Understand the Risk (Not Just the Vulnerability)

**Task:** Translate the technical finding into business risk.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    VULNERABILITY VS RISK                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   VULNERABILITY (Technical)             RISK (Business)                     │
│   ─────────────────────────             ───────────────                     │
│                                                                             │
│   Shared service account                Unauthorized access to $2B in       │
│                                         transaction data                    │
│                                                                             │
│   No password rotation                  If password is compromised,         │
│                                         attacker has 18+ months of access   │
│                                                                             │
│   15 people know password               Any of 15 people (or their          │
│                                         compromised accounts) could         │
│                                         access the database                 │
│                                                                             │
│   No query logging                      Breach may go undetected;           │
│                                         forensics impossible                │
│                                                                             │
│   DB admin privileges                   Attacker could modify/delete        │
│                                         financial records                   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Phase 2: Assess Likelihood and Impact

**Task:** Provide specific, evidence-based assessment.

**BAD Assessment (Vague):**
```
Likelihood: Low
Impact: High
Risk Rating: Medium

This is bad because:
- No evidence for "Low" likelihood
- No definition of "High" impact
- "Medium" is meaningless without context
```

**GOOD Assessment (Specific):**

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         LIKELIHOOD ASSESSMENT                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   FACTOR                                ASSESSMENT                          │
│   ──────                                ──────────                          │
│                                                                             │
│   Threat Actor Interest                 HIGH                                │
│   - Financial data is high-value target                                     │
│   - Payment processors frequently targeted                                  │
│   - 3 competitors breached in past 24 months                                │
│                                                                             │
│   Attack Complexity                     LOW (Easy to exploit)               │
│   - Password may be in source control                                       │
│   - 15 people know it (social engineering surface)                          │
│   - No MFA, no IP restrictions                                              │
│                                                                             │
│   Detection Capability                  LOW                                 │
│   - No query logging                                                        │
│   - Legitimate and malicious access look identical                          │
│   - Could exfiltrate data without triggering alerts                         │
│                                                                             │
│   Historical Incidents                  NONE KNOWN                          │
│   - No known compromise of this system                                      │
│   - However, we would not detect a sophisticated breach                     │
│                                                                             │
│   OVERALL LIKELIHOOD: MODERATE-HIGH                                         │
│   (Not certain, but conditions favor eventual compromise)                   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│                         IMPACT ASSESSMENT                                   │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   IMPACT CATEGORY           POTENTIAL CONSEQUENCE            ESTIMATED COST │
│   ───────────────           ──────────────────────           ───────────── │
│                                                                             │
│   Data Breach               Exposure of cardholder data      $5-15M        │
│                             (PCI penalties + notification)                  │
│                                                                             │
│   Financial Fraud           Manipulation of transaction      $1-10M        │
│                             records                          (Depends on   │
│                                                              detection)    │
│                                                                             │
│   Regulatory Action         PCI DSS non-compliance finding   $500K-5M      │
│                             SOC 2 qualified opinion          (Fines +      │
│                                                              remediation)  │
│                                                                             │
│   Reputation                Customer loss, media coverage    $2-20M        │
│                             (B2B customers leave)            (Hard to      │
│                                                              quantify)     │
│                                                                             │
│   Business Disruption       System taken offline for         $100K/day     │
│                             forensics                                      │
│                                                                             │
│   TOTAL POTENTIAL IMPACT: $8-50M (depending on severity)                   │
│   MOST LIKELY SCENARIO: $10-15M                                            │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Phase 3: Evaluate Compensating Controls

**Task:** Honestly assess what protection exists today.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    COMPENSATING CONTROLS ASSESSMENT                         │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   CLAIMED CONTROL           EFFECTIVENESS              HONEST ASSESSMENT    │
│   ───────────────           ─────────────              ─────────────────    │
│                                                                             │
│   "Network is                PARTIAL                   True, but:           │
│   segmented"                                           - 15 people have     │
│                                                          legitimate access  │
│                                                        - Lateral movement   │
│                                                          from their         │
│                                                          workstations       │
│                                                          possible           │
│                                                                             │
│   "Application              WEAK                       - App logs exist     │
│   logging enabled"                                     - DB queries not     │
│                                                          logged             │
│                                                        - Could copy entire  │
│                                                          DB without trace   │
│                                                                             │
│   "Background checks        MINIMAL                    - Doesn't prevent    │
│   on employees"                                          compromise         │
│                                                        - Doesn't prevent    │
│                                                          insider threat     │
│                                                        - Doesn't detect     │
│                                                          abuse              │
│                                                                             │
│   "Encrypted at rest"       IRRELEVANT                 - Legitimate user    │
│                                                          sees decrypted     │
│                                                          data anyway        │
│                                                        - Encryption doesn't │
│                                                          prevent authorized │
│                                                          access abuse       │
│                                                                             │
│   OVERALL: Compensating controls provide LIMITED protection                │
│   They reduce some risk but do not address the core vulnerability          │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Phase 4: Present Options to Business

**Task:** Give decision-makers real choices, not a single path.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         DECISION OPTIONS                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   OPTION A: FULL REMEDIATION                                                │
│   ─────────────────────────                                                 │
│   Replace system with modern architecture                                   │
│   - Individual service accounts with MFA                                    │
│   - Full audit logging                                                      │
│   - Automated password rotation                                             │
│                                                                             │
│   Cost: $800K-1.2M (development + migration)                               │
│   Timeline: 12-18 months                                                    │
│   Residual Risk: LOW                                                        │
│                                                                             │
│   OPTION B: PARTIAL REMEDIATION                                             │
│   ────────────────────────────                                              │
│   Keep system, add controls:                                                │
│   - Implement database activity monitoring                                  │
│   - Reduce password knowledge to 3 people                                   │
│   - Add IP allow-listing                                                    │
│   - Quarterly password rotation                                             │
│                                                                             │
│   Cost: $150K (tooling + configuration)                                     │
│   Timeline: 3 months                                                        │
│   Residual Risk: MEDIUM (still shared account, still no MFA)               │
│                                                                             │
│   OPTION C: ACCEPT RISK                                                     │
│   ───────────────────────                                                   │
│   Continue current state with documented acceptance                         │
│                                                                             │
│   Cost: $0 direct cost                                                      │
│   Potential Cost: $10-15M if breach occurs                                  │
│   Residual Risk: HIGH                                                       │
│                                                                             │
│   OPTION D: ACCEPT WITH MINIMAL IMPROVEMENTS                                │
│   ──────────────────────────────────────────                                │
│   Accept risk but implement:                                                │
│   - Password rotation (now and quarterly)                                   │
│   - Reduce to 5 people with access                                          │
│   - Add to next fiscal year budget for full fix                             │
│                                                                             │
│   Cost: $20K + staff time                                                   │
│   Timeline: 2 weeks                                                         │
│   Residual Risk: HIGH (but slightly reduced)                                │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Phase 5: Write the Risk Acceptance Document

**Task:** Create a formal document that a CFO would sign.

**Document Structure:**

```markdown
# RISK ACCEPTANCE DOCUMENT

## Risk ID: RA-2026-017
## System: PaymentReconciler v3.2
## Risk Owner: [CFO Name]
## Prepared By: [Your Name], GRC
## Date: [Date]

---

## 1. EXECUTIVE SUMMARY

This document requests formal acceptance of security risk associated with the
PaymentReconciler system's authentication architecture. The system uses a shared
service account with database administrator privileges, known to 15 employees,
with a password unchanged for 18 months.

**Recommendation:** Option D (Accept with Minimal Improvements) while allocating
budget for full remediation in FY2027.

---

## 2. RISK DESCRIPTION

### What Could Happen
An unauthorized person could gain access to the PaymentReconciler database and:
- View all transaction records ($2B annually)
- Modify financial data without detection
- Exfiltrate cardholder data subject to PCI DSS

### How It Could Happen
- One of 15 employees could have their credentials phished
- A former employee may still have knowledge of the password
- The password may exist in backups, source control, or documentation
- An insider could abuse their access

### Why We Might Not Know
- No database query logging exists
- Legitimate and malicious access appear identical
- A sophisticated attacker could operate undetected

---

## 3. RISK ASSESSMENT

| Factor | Assessment | Basis |
|--------|------------|-------|
| Likelihood | Moderate-High | 15 people know password; no rotation in 18 months; high-value target |
| Impact | $10-15M most likely | PCI fines, breach notification, reputation damage |
| Current Detection | Poor | No DB-level logging |
| Existing Controls | Limited | Network segmentation only |

---

## 4. OPTIONS CONSIDERED

[Insert options table from Phase 4]

---

## 5. SELECTED OPTION

**Option D: Accept with Minimal Improvements**

### Rationale
- Full remediation cannot be completed in current fiscal year
- Partial improvements meaningfully reduce likelihood
- Risk is documented and owned
- Budget allocated for FY2027 resolution

### Immediate Actions (Within 2 Weeks)
1. Rotate password immediately
2. Reduce access to 5 named individuals
3. Document all individuals with access
4. Establish quarterly rotation schedule

### FY2027 Commitment
- Allocate $1M for system replacement
- Begin requirements gathering Q1 FY2027

---

## 6. RESIDUAL RISK ACKNOWLEDGMENT

After implementing Option D, the following risks remain:

- Shared account still in use (no individual accountability)
- No MFA on database access
- No query-level audit trail
- 5 people still know the password

**If a breach occurs, we will likely NOT be able to:**
- Identify which individual's credentials were used
- Determine what data was accessed
- Prove to regulators that we had adequate controls

---

## 7. CONDITIONS AND REVIEW

This acceptance is valid under the following conditions:
- Password is rotated within 2 weeks and quarterly thereafter
- Access is limited to 5 named individuals
- Budget for remediation is included in FY2027 planning
- This acceptance is reviewed in 6 months

If any condition is not met, this acceptance is void and re-evaluation is required.

---

## 8. SIGNATURES

### Risk Owner (Business)
By signing, I acknowledge that I understand the risk described above and accept
responsibility for this risk on behalf of the organization.

Name: _______________________
Title: CFO
Date: _______________________
Signature: ___________________

### Security Review
By signing, I confirm that the risk has been accurately described and the
assessment is complete.

Name: _______________________
Title: CISO
Date: _______________________
Signature: ___________________

### Compliance Review
By signing, I confirm that relevant compliance implications have been considered.

Name: _______________________
Title: Head of Compliance
Date: _______________________
Signature: ___________________
```

---

## Success Criteria

Your risk acceptance demonstrates strong GRC judgment if:

| Criteria | Evidence |
|----------|----------|
| **Plain language** | Executive can understand without security background |
| **Specific, not vague** | Dollar amounts, timelines, named individuals |
| **Honest about controls** | Doesn't inflate effectiveness of compensating controls |
| **Options presented** | Business makes informed choice, not security dictating |
| **Residual risk clear** | Explicitly states what danger remains |
| **Conditions stated** | Acceptance has boundaries and review dates |

### Red Flags (Weak Project)

- Technical jargon without translation
- "Risk is low" without evidence
- "Controls are in place" without effectiveness assessment
- Only one option presented (no choice)
- No discussion of what happens if breach occurs
- Security "approving" instead of business "owning"

---

## Communication Comparison

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    BAD VS GOOD RISK COMMUNICATION                           │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   BAD (Technical Jargon)                                                    │
│   ──────────────────────                                                    │
│   "The PaymentReconciler system presents a credential stuffing attack       │
│   surface with potential for lateral movement into the CDE. The shared      │
│   service account architecture enables privilege escalation scenarios       │
│   and creates accountability gaps in audit trails."                         │
│                                                                             │
│   GOOD (Plain Language)                                                     │
│   ─────────────────────                                                     │
│   "15 people know the password to our payment database. The password        │
│   hasn't been changed in 18 months. If any of those people are phished,     │
│   or if a former employee shared the password, someone could access         │
│   $2B worth of transaction data. We would probably not detect it."          │
│                                                                             │
│   BAD (Vague)                                                               │
│   ───────────                                                               │
│   "Risk: Medium. Impact: High. Likelihood: Low. Controls are in place."     │
│                                                                             │
│   GOOD (Specific)                                                           │
│   ───────────────                                                           │
│   "If this system is breached, we estimate $10-15M in costs from PCI        │
│   fines, customer notification, and likely loss of two major clients.       │
│   Three similar companies were breached last year with comparable           │
│   impacts."                                                                 │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Deliverables Checklist

```
□ Risk description in plain language
□ Likelihood assessment with evidence
□ Impact assessment with dollar estimates
□ Compensating controls evaluation (honest)
□ Options analysis (minimum 3 options)
□ Formal risk acceptance document
□ Conditions and review schedule
□ 5-minute video presenting this to a hypothetical CFO
```

---

## Additional Scenarios (Practice)

Apply this framework to other common risk acceptance scenarios:

1. **Unpatched Legacy System** - Vendor no longer provides updates
2. **MFA Exception** - Executive refuses to use MFA
3. **Third-Party Risk** - Critical vendor fails security assessment
4. **Encryption Gap** - Database encryption "too expensive"
5. **Insider Threat** - No DLP on sensitive data

---

## Reflection Questions

After completing this project, answer:

1. What was hardest about writing in plain language?
2. How did you decide on the dollar impact estimates?
3. What would you do if the CFO refused all options and demanded acceptance?
4. How would you feel presenting this if the breach happened next month?
5. What's the difference between this document and a "cover your ass" exercise?

---

*Remember: Risk acceptance is a business decision, not a security permission slip. The goal is to present a clear, honest picture so the business can choose knowingly.*
