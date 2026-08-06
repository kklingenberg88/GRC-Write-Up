# Project 02: Justifying a Statement of Applicability in ISO 27001

## Overview

The Statement of Applicability (SoA) is one of the most misunderstood artifacts in ISO 27001. This project requires you to build a **risk-driven SoA that explicitly excludes controls and defends those exclusions**.

Remember, as a GRC professional you may audit - and your career depends on transparency, radical honesty and ability to implement change while exercising sound judgement, insight and technical know how. Bonus if you can do it for both technical and non technical people as executives will often look to you as a SME.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      SOA: JUNIOR VS EXPERIENCED APPROACH                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   JUNIOR APPROACH                       EXPERIENCED APPROACH                │
│   ───────────────                       ────────────────────                │
│   Include everything "to be safe"       Include only what's justified       │
│   Mark controls as "implemented"        Honest implementation status        │
│   even when aspirational                                                    │
│   Avoid exclusions (feel risky)         Defend exclusions confidently       │
│   Generic justifications                Risk-driven justifications          │
│                                                                             │
│   Result: Audit debt, weak credibility  Result: Proportionate, defensible   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## The Scenario

**Company:** CloudNative Analytics Ltd
**Industry:** B2B SaaS - Data Analytics Platform
**Employees:** 85 (fully remote)
**Infrastructure:** 100% cloud (AWS), no physical data centers
**Customers:** 200+ enterprise clients requiring ISO 27001 certification
**Goal:** Achieve ISO 27001:2022 certification within 6 months

### Business Context

- No physical offices (remote-first since founding)
- All employees use company-managed laptops
- Customer data processed exclusively in AWS (eu-west-1)
- No mobile app development
- No cryptographic key generation (using AWS KMS)
- API-only product (no end-user facing application)
- No development outsourcing

### Risk Assessment Summary

The company has completed a risk assessment identifying these top risks:

| Risk ID | Risk Description | Likelihood | Impact | Risk Level |
|---------|-----------------|------------|--------|------------|
| R-001 | Unauthorized access to customer data | Medium | High | High |
| R-002 | Employee laptop theft/loss | Medium | Medium | Medium |
| R-003 | Cloud misconfiguration | High | High | Critical |
| R-004 | Insider threat | Low | High | Medium |
| R-005 | Third-party breach (AWS) | Low | Critical | High |
| R-006 | Business continuity failure | Low | High | Medium |

---

## What You Need to Do

### Phase 1: Understand Annex A Controls

ISO 27001:2022 contains 93 controls across 4 themes. Your job is NOT to implement all 93.
And ones that are not implemented but should be, end up being your POAMs. This is acceptable, and expected.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                     ISO 27001:2022 CONTROL STRUCTURE                        │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   Theme 5: Organizational (37 controls)                                     │
│   ├── Information security policies                                         │
│   ├── Roles and responsibilities                                            │
│   ├── Segregation of duties                                                 │
│   ├── Contact with authorities                                              │
│   └── ... and 33 more                                                       │
│                                                                             │
│   Theme 6: People (8 controls)                                              │
│   ├── Screening                                                             │
│   ├── Terms of employment                                                   │
│   ├── Awareness and training                                                │
│   └── ... and 5 more                                                        │
│                                                                             │
│   Theme 7: Physical (14 controls)                                           │
│   ├── Physical security perimeters        ◄── Likely exclusion candidate   │
│   ├── Physical entry                      ◄── Likely exclusion candidate   │
│   ├── Securing offices                    ◄── Likely exclusion candidate   │
│   └── ... and 11 more                                                       │
│                                                                             │
│   Theme 8: Technological (34 controls)                                      │
│   ├── User endpoint devices                                                 │
│   ├── Privileged access rights                                              │
│   ├── Information access restriction                                        │
│   └── ... and 31 more                                                       │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Phase 2: Identify Exclusion Candidates

**Task:** Review each control and identify those NOT applicable to CloudNative Analytics.

**Exclusion Decision Framework:**

```
                    ┌─────────────────────────┐
                    │   Is this control       │
                    │   relevant to our       │
                    │   context?              │
                    └───────────┬─────────────┘
                                │
                    ┌───────────┴───────────┐
                    │                       │
                    ▼                       ▼
            ┌───────────────┐       ┌───────────────┐
            │      YES      │       │      NO       │
            └───────┬───────┘       └───────┬───────┘
                    │                       │
                    ▼                       ▼
            ┌───────────────┐       ┌───────────────────────┐
            │   Include in  │       │   Document exclusion: │
            │   SoA with    │       │   • What's excluded   │
            │   implementation│     │   • Why excluded      │
            │   status      │       │   • Risk justification│
            └───────────────┘       │   • Alternative if any│
                                    └───────────────────────┘
```
```

### Phase 3: Build the Risk-Driven SoA

**Task:** For each of the 93 controls, document:

1. **Applicability** (Yes/No)
2. **Justification** (tied to risk assessment)
3. **Implementation Status** (if applicable)
4. **Evidence Reference** (where to find proof)


```
```
### Phase 4: Defend Your Exclusions

**Task:** For each exclusion, prepare an auditor defense document.

**Example Exclusion Defense:**
````

```
## Control 7.3: Securing Offices, Rooms and Facilities

### Exclusion Statement
This control is NOT APPLICABLE to CloudNative Analytics Ltd.

### Business Context
CloudNative Analytics operates as a fully remote organization since its founding
in 2021. The company:
- Has no physical office space
- Has no data centers (100% AWS)
- Has no server rooms or equipment storage
- Employees work from home locations

### Risk Assessment Linkage
Physical security risks are addressed through alternative controls:

| Physical Risk | Alternative Control | Reference |
|---------------|--------------------| ----------|
| Equipment theft | 8.1 User endpoint devices | Laptop encryption, MDM |
| Unauthorized access to data | 8.3 Information access restriction | AWS IAM, MFA |
| Document security | 5.10 Acceptable use | No printing policy |

### Evidence of Non-Applicability
- Company registration showing no physical address (registered agent only)
- AWS billing showing 100% cloud infrastructure
- HR policy confirming remote-first status
- Asset register showing no physical servers

### Auditor Anticipated Questions

Q: "What about employees' home offices?"
A: Home offices are not company facilities. Security of employee endpoint devices
   is addressed through control 8.1 and our MDM policy.

Q: "What about meeting spaces you might rent?"
A: Any temporary meeting space rental would not store company equipment or data.
   Data remains in AWS and on encrypted endpoints only.

### Conditions for Re-evaluation
This exclusion should be re-evaluated if:
- Company opens physical office space
- Company begins on-premises data processing
- Company acquires assets requiring physical storage
```

### Phase 5: Document Alternative Measures

**Task:** Where controls are excluded, show how risks are still addressed.
And don't exclude ones you think are outside of scope, produce evidence to prove they're outside of scope.
```
```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    ALTERNATIVE MEASURES MAPPING                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   EXCLUDED PHYSICAL CONTROLS          ALTERNATIVE CONTROLS                  │
│   ──────────────────────────          ────────────────────                  │
│                                                                             │
│   7.1 Physical perimeters    ───────▶ 8.1 Endpoint security                │
│   7.2 Physical entry         ───────▶ 5.15 Access control policy           │
│   7.3 Securing offices       ───────▶ 8.1 + 5.10 Acceptable use            │
│   7.4 Physical monitoring    ───────▶ 8.16 Monitoring activities (AWS)     │
│   7.5 Protection threats     ───────▶ AWS physical security (SOC 2)        │
│   7.6 Working in secure areas───────▶ N/A - no secure areas                │
│   7.7 Clear desk/screen      ───────▶ 8.1 Screen lock policy               │
│   7.8 Equipment siting       ───────▶ N/A - no on-prem equipment           │
│   7.9 Off-premises security  ───────▶ 8.1 Endpoint MDM                     │
│   7.10 Storage media         ───────▶ 8.10 + AWS encryption                │
│   7.11 Supporting utilities  ───────▶ AWS infrastructure                   │
│   7.12 Cabling security      ───────▶ N/A - no physical cabling            │
│   7.13 Equipment maintenance ───────▶ 8.1 Laptop refresh policy            │
│   7.14 Disposal/reuse        ───────▶ 8.10 + ITAM disposal process         │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Phase 6: Create Implementation Status Honesty

**Task:** Same as before, don't mark controls as "implemented" when they're aspirational.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    HONEST IMPLEMENTATION STATUS                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   STATUS          MEANING                 AUDITOR EXPECTATION               │
│   ──────          ───────                 ───────────────────               │
│   Implemented     Fully operational       Evidence of operation             │
│                   Evidence available      Metrics/logs available            │
│                                                                             │
│   Partially       Some elements in place  Remediation plan with dates       │
│   Implemented     Gaps documented         Evidence for implemented parts    │
│                                                                             │
│   Planned         Not yet started         Project plan with timeline        │
│                   Committed to implement  Resource allocation               │
│                                                                             │
│   Not Applicable  Excluded from scope     Documented justification          │
│                                           Alternative measures              │
│                                                                             │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Success Criteria

Your SoA demonstrates strong GRC judgment if:

| Criteria | Evidence |
|----------|----------|
| **Risk-driven inclusions** | Each included control links to specific risk(s) |
| **Defended exclusions** | Minimum 5 controls excluded with full justification |
| **Honest status** | No "implemented" status without evidence reference |
| **Alternative measures documented** | Exclusions show how risk is still addressed |
| **Anticipated auditor pushback** | Each exclusion includes likely questions |
| **Business context clear** | SoA reflects actual organizational reality |

### Red Flags (Weak Project)

- All 93 controls marked "applicable" with no exclusions
- Generic justifications like "required by ISO 27001"
- No linkage to risk assessment
- All controls marked "implemented" (unrealistic)
- Physical controls included for a cloud-only company

---

## Deliverables Checklist

```
□ Complete Statement of Applicability (all 93 controls addressed)
□ Minimum 5 detailed exclusion justifications
□ Alternative measures mapping document
□ Risk-to-control traceability matrix
□ Evidence reference index
□ Auditor challenge preparation (Q&A for exclusions)
□ 5-minute video walkthrough of your most controversial exclusion
```

### Key Exclusions

1. **7.1-7.14 (11 controls):** Physical security - No physical premises
2. **5.X:** [Control] - [Brief reason]
3. **8.X:** [Control] - [Brief reason]

### Risk Coverage

All identified risks from the risk assessment are addressed:
- R-001: Controls 5.x, 8.x, 8.y
- R-002: Controls 8.1, 8.10
- [Continue for all risks]
```

---

## Reflection Questions

After completing this project, answer:

1. Which exclusion did you find hardest to justify?
2. What control inclusion surprised you (thought you could exclude but couldn't)?
3. How would your SoA change if the company opened one physical office?
4. Where did the risk assessment drive a different decision than your instinct?
5. What's the cost of over-including controls (audit debt)?

---

ISO 27001 is not about maximal compliance. It's about proportionate, defensible security aligned to business reality.*


These templates were provided by Cloud Security Guy on YouTube. 
He is a multi-award-winning cybersecurity leader with 21+ years of experience.
