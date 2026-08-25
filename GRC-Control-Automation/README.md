# Project 04: Automating a GRC Control

## Overview

One of the biggest shifts happening in GRC is the move toward **GRC Engineering**. Traditional GRC relies on manual processes that technically exist but fail silently. This project requires you to redesign a common control to be automated and continuous.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    TRADITIONAL VS ENGINEERED GRC                            │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   TRADITIONAL GRC                       GRC ENGINEERING                     │
│   ──────────────                        ───────────────                     │
│   Quarterly reviews                     Continuous monitoring               │
│   Spreadsheet evidence                  Automated evidence collection       │
│   Screenshots in tickets                API-driven validation               │
│   Email confirmations                   Real-time alerting                  │
│   Manual sampling                       Complete coverage                   │
│   Point-in-time compliance              Continuous compliance               │
│                                                                             │
│   Failure Mode: Silent                  Failure Mode: Visible               │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## The Scenario

**Company:** DataStream Technologies
**Industry:** Financial Services SaaS
**Infrastructure:** AWS (multi-account), Kubernetes
**Compliance:** SOC 2 Type II, PCI DSS
**Team Size:** 120 engineers across 8 teams
**Current Problem:** Access review failures in last two audits

### The Control to Automate

**Control:** Privileged Access Review
**Framework Reference:**
- SOC 2: CC6.1, CC6.2, CC6.3
- PCI DSS: Requirement 7.2, 8.1.4

**Current Manual Process:**

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    CURRENT QUARTERLY ACCESS REVIEW                          │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   Week 1                                                                    │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │  GRC analyst exports user list from AWS IAM (3 hours)               │   │
│   │  GRC analyst exports user list from each application (8 hours)      │   │
│   │  GRC analyst merges into master spreadsheet (4 hours)               │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│   Week 2-3                                                                  │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │  Spreadsheet sent to each manager via email                         │   │
│   │  Managers review and respond (if they respond)                      │   │
│   │  GRC analyst chases non-responders                                  │   │
│   │  Responses compiled into evidence document                          │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│   Week 4                                                                    │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │  Remediation tickets created for terminated users                   │   │
│   │  Tickets sit in backlog                                             │   │
│   │  Some get resolved, some don't                                      │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│   ACTUAL PROBLEM: By week 4, the data from week 1 is stale               │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Current Failure Modes

| Failure | Impact | Detection |
|---------|--------|-----------|
| Terminated employee not removed | Unauthorized access for weeks | Only at next review |
| Manager doesn't respond | Access assumed approved | Not detected |
| Spreadsheet data stale | Decisions made on old data | Not detected |
| Remediation ticket ignored | Access persists | Maybe at next review |
| New privileged user added | Not reviewed for 90 days | Next quarterly review |

---

## What You Need to Do

### Phase 1: Identify Control Intent

**Task:** Define what the control is actually trying to achieve.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         CONTROL INTENT ANALYSIS                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   STATED CONTROL:                                                           │
│   "Privileged access shall be reviewed quarterly to ensure appropriateness" │
│                                                                             │
│   ACTUAL INTENT:                                                            │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │  1. Only authorized users have privileged access                    │   │
│   │  2. Privileged access is appropriate to job function                │   │
│   │  3. Terminated users are promptly removed                           │   │
│   │  4. Role changes result in access changes                           │   │
│   │  5. Unnecessary privileges are removed                              │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│   KEY INSIGHT: "Quarterly" is the compliance minimum, not the security     │
│   objective. The objective is CONTINUOUS appropriateness.                   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Phase 2: Map Manual Steps and Failure Points

**Task:** Document every manual step and where it can fail.

**Manual Process Breakdown:**

| Step | Action | Owner | Failure Points |
|------|--------|-------|----------------|
| 1 | Export AWS IAM users | GRC Analyst | Manual, incomplete, stale immediately |
| 2 | Export application users | GRC Analyst | Different formats, missing systems |
| 3 | Merge into spreadsheet | GRC Analyst | Human error, version control |
| 4 | Send to managers | GRC Analyst | Wrong manager, outdated org chart |
| 5 | Manager review | Manager | Non-response, rubber-stamping |
| 6 | Compile responses | GRC Analyst | Lost emails, partial responses |
| 7 | Create remediation tickets | GRC Analyst | Manual, delayed |
| 8 | Execute remediation | IT Admin | Backlog, incomplete |

### Phase 3: Design Automated Solution

**Task:** Create an architecture for continuous, automated access review.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    AUTOMATED ACCESS REVIEW ARCHITECTURE                     │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   DATA SOURCES                                                              │
│   ────────────                                                              │
│   ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐    │
│   │  AWS     │  │  Okta    │  │  GitHub  │  │  K8s     │  │  DB      │    │
│   │  IAM     │  │  (IdP)   │  │  (Code)  │  │  RBAC    │  │  Access  │    │
│   └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘    │
│        │             │             │             │             │           │
│        └─────────────┴─────────────┴─────────────┴─────────────┘           │
│                                    │                                        │
│                                    ▼                                        │
│                    ┌───────────────────────────────┐                        │
│                    │     ACCESS DATA AGGREGATOR    │                        │
│                    │     (Lambda / Scheduled Job)  │                        │
│                    └───────────────┬───────────────┘                        │
│                                    │                                        │
│                                    ▼                                        │
│                    ┌───────────────────────────────┐                        │
│                    │     IDENTITY CORRELATION      │                        │
│                    │     (Match users across       │                        │
│                    │      systems)                 │                        │
│                    └───────────────┬───────────────┘                        │
│                                    │                                        │
│        ┌───────────────────────────┼───────────────────────────┐           │
│        │                           │                           │           │
│        ▼                           ▼                           ▼           │
│   ┌─────────────┐          ┌─────────────────┐         ┌─────────────┐     │
│   │  HR SYSTEM  │          │  ACCESS         │         │  POLICY     │     │
│   │  COMPARISON │          │  INVENTORY      │         │  ENGINE     │     │
│   │             │          │  (Database)     │         │             │     │
│   │ Active vs   │          │                 │         │ Role-based  │     │
│   │ Access      │          │ Who has what    │         │ rules       │     │
│   └──────┬──────┘          └─────────────────┘         └──────┬──────┘     │
│          │                                                    │            │
│          └────────────────────────┬───────────────────────────┘            │
│                                   │                                        │
│                                   ▼                                        │
│                    ┌───────────────────────────────┐                        │
│                    │      ANOMALY DETECTION        │                        │
│                    └───────────────┬───────────────┘                        │
│                                    │                                        │
│          ┌─────────────────────────┼─────────────────────────┐             │
│          │                         │                         │             │
│          ▼                         ▼                         ▼             │
│   ┌─────────────┐          ┌─────────────────┐        ┌─────────────┐      │
│   │  TERMINATED │          │  OVER-          │        │  NEW        │      │
│   │  USER ALERT │          │  PRIVILEGED     │        │  PRIVILEGED │      │
│   │             │          │  ALERT          │        │  ALERT      │      │
│   │ Immediate   │          │                 │        │             │      │
│   │ Slack +     │          │ Manager review  │        │ Auto-review │      │
│   │ Jira ticket │          │ triggered       │        │ workflow    │      │
│   └─────────────┘          └─────────────────┘        └─────────────┘      │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Phase 4: Define Detection Rules

**Task:** Create specific rules that trigger alerts or actions.

**Rule Examples:**

```yaml
# Rule 1: Terminated User Detection
rule: terminated_user_with_access
description: User in HR system marked as terminated but still has active access
trigger: daily_scan
condition:
  - hr_status == "terminated"
  - termination_date < today
  - active_access_count > 0
action:
  - severity: critical
  - auto_create_jira: true
  - slack_alert: "#security-ops"
  - auto_disable_after: 24h  # if not manually resolved
sla: 24_hours

# Rule 2: Orphaned Privileged Account
rule: orphaned_privileged_account
description: Account with admin rights not linked to active employee
trigger: daily_scan
condition:
  - privilege_level == "admin"
  - hr_employee_match == null
action:
  - severity: high
  - auto_create_jira: true
  - require_justification: true
sla: 72_hours

# Rule 3: New Privileged Access Granted
rule: new_privileged_access
description: Admin access granted to user
trigger: real_time (event-driven)
condition:
  - access_change_type == "grant"
  - privilege_level == "admin"
action:
  - severity: medium
  - notify_manager: true
  - require_approval_within: 48h
  - auto_revoke_if_not_approved: true
sla: 48_hours

# Rule 4: Access Without Recent Activity
rule: dormant_privileged_access
description: Admin access not used in 90 days
trigger: weekly_scan
condition:
  - privilege_level == "admin"
  - last_access_date < (today - 90_days)
action:
  - severity: low
  - manager_notification: true
  - recommend_revocation: true
sla: 30_days
```

### Phase 5: Design Evidence Generation

**Task:** Define what evidence the system produces for auditors.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    AUTOMATED EVIDENCE GENERATION                            │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   EVIDENCE TYPE              FORMAT              GENERATION                 │
│   ─────────────              ──────              ──────────                 │
│                                                                             │
│   Access Inventory           JSON/CSV            Daily snapshot             │
│   - All users with access                        Stored in S3               │
│   - Permission levels                            Versioned                  │
│   - Last access date                                                        │
│                                                                             │
│   Termination Report         PDF                 On-demand                  │
│   - Users terminated                             Auto-generated             │
│   - Access removal date                                                     │
│   - Time to removal                                                         │
│                                                                             │
│   Alert History              Dashboard           Real-time                  │
│   - All triggered alerts                         Immutable log              │
│   - Resolution status                                                       │
│   - Resolution time                                                         │
│                                                                             │
│   Approval Workflow          Audit trail         Event-driven               │
│   - Who approved what                            Cryptographically          │
│   - When                                         signed                     │
│   - Justification                                                           │
│                                                                             │
│   Exception Report           Spreadsheet         Weekly                     │
│   - Open exceptions                              Includes aging             │
│   - Justifications                                                          │
│   - Risk acceptance                                                         │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Phase 6: Document What Remains Manual

**Task:** Be explicit about what is NOT automated and why.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    AUTOMATED VS REMAINING MANUAL                            │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   AUTOMATED                             REMAINS MANUAL (WITH REASON)        │
│   ─────────                             ────────────────────────────        │
│                                                                             │
│   ✓ Data collection from systems        ✗ Business justification review    │
│                                           (Requires human judgment)         │
│                                                                             │
│   ✓ Terminated user detection           ✗ Exception approval               │
│                                           (Requires accountability)         │
│                                                                             │
│   ✓ Alert generation                    ✗ Complex role changes             │
│                                           (Context needed)                  │
│                                                                             │
│   ✓ Ticket creation                     ✗ Policy updates                   │
│                                           (Governance decision)             │
│                                                                             │
│   ✓ Evidence packaging                  ✗ Auditor Q&A                      │
│                                           (Cannot be automated)             │
│                                                                             │
│   ✓ Compliance reporting                                                    │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Phase 7: Identify Residual Risk

**Task:** Document what risks remain even with automation.

**Residual Risk Assessment:**

| Risk | Description | Mitigation | Residual Level |
|------|-------------|------------|----------------|
| Data source incomplete | System not integrated | Quarterly manual review of integrations | Low |
| Identity correlation failure | Same person, different accounts | Manual reconciliation for unknowns | Medium |
| Automation failure | Job doesn't run | Monitoring + alerting on automation | Low |
| False positives | Alert fatigue | Tuning + feedback loop | Medium |
| Policy drift | Rules don't match current risk | Quarterly policy review | Low |

---

## Implementation Approach

**Suggested Technology Stack:**

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    EXAMPLE TECHNOLOGY STACK                                 │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   Component                 Options                                         │
│   ─────────                 ───────                                         │
│                                                                             │
│   Data Collection           AWS Lambda + EventBridge                        │
│                             OR Temporal workflows                           │
│                             OR Apache Airflow                               │
│                                                                             │
│   Data Storage              PostgreSQL / DynamoDB                           │
│                             S3 for evidence archives                        │
│                                                                             │
│   Policy Engine             Open Policy Agent (OPA)                         │
│                             OR custom rules engine                          │
│                                                                             │
│   Alerting                  PagerDuty / Slack / Jira                        │
│                                                                             │
│   Dashboard                 Grafana / Metabase / Custom                     │
│                                                                             │
│   Evidence Generation       Python + Jinja2 templates                       │
│                             PDF generation via WeasyPrint                   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Success Criteria

Your project demonstrates GRC engineering capability if:

| Criteria | Evidence |
|----------|----------|
| **Control intent understood** | You went beyond "quarterly review" to actual security objectives |
| **Failure points identified** | Specific ways the manual process fails |
| **Automation is meaningful** | Addresses real failure modes, not just digitizing spreadsheets |
| **Manual elements justified** | Clear reason for what remains manual |
| **Evidence design included** | Auditors can understand the output |
| **Residual risk documented** | Honest about what's still not covered |

### Red Flags (Weak Project)

- Just replacing spreadsheets with a database (no new capability)
- No integration architecture
- Ignoring the "what remains manual" question
- No residual risk analysis
- No evidence generation consideration
- Generic "we'll use automation" without specifics

---

## Deliverables Checklist

```
□ Control intent analysis document
□ Current state failure mode analysis
□ Automated solution architecture diagram
□ Detection rules specification (minimum 5 rules)
□ Evidence generation design
□ Manual vs automated comparison table
□ Residual risk assessment
□ Implementation approach recommendation
□ 5-minute video demonstrating the automation flow
```

---

## Alternative Controls to Automate

If you want additional practice, apply this framework to:

1. **Configuration Compliance** - Drift detection for cloud resources
2. **Vulnerability Management** - SLA tracking for remediation
3. **Change Management** - Unauthorized change detection
4. **Data Classification** - Automated sensitive data discovery
5. **Third-Party Risk** - Vendor security posture monitoring

---

## Reflection Questions

After completing this project, answer:

1. What was the biggest gap between "compliance" and "security" in the manual process?
2. Which automation would have the highest impact with lowest effort?
3. Where did you have to accept residual risk? Why?
4. How would you explain this automation to a non-technical auditor?
5. What would you monitor to know if the automation is working?

---

*Remember: The power is not in the tooling. It's in explaining what has been automated, what remains manual, what evidence auditors will see, and what residual risk still exists.*
