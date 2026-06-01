> 🌐 **Available in:** **English 🇬🇧** | [Español 🇪🇸](stractegic-guide-for-the-ghas-exam.md)
---

# STRATEGIC GUIDE FOR THE GHAS EXAM (GH-500)
## Decision-Making Framework, Exam Tips & Practice Questions

> **Essential companion to the GHAS Technical Guide**  
> This guide focuses on **how to pass the exam**.

---

## 📋 TABLE OF CONTENTS

1. [Decision-Making Framework](#1)
2. [Advanced Governance and Roles](#2)
3. [Security Overview in Depth](#3)
4. [Critical Comparisons](#4)
5. [Common Exam Pitfalls](#5)
6. [Real-World Scenarios](#6)
7. [100 Practice Questions](#7)
8. [Exam Strategy](#exam-strategy)

---

# DECISION-MAKING FRAMEWORK <a id="1"></a>

## 🎯 The golden rule of the exam

> **The exam does NOT ask "how does X work?"**  
> **The exam asks "what would you use in this scenario?"**

## 4-Step Framework

```
┌─────────────────────────────────────────────────────┐
│ 1. IDENTIFY THE PROBLEM                             │
│    What is happening? What is the objective?         │
└─────────────────┬───────────────────────────────────┘
                  │
┌─────────────────▼───────────────────────────────────┐
│ 2. CLASSIFY THE TIMING IN SDLC                      │
│    Before commit? In PR? After merge?               │
└─────────────────┬───────────────────────────────────┘
                  │
┌─────────────────▼───────────────────────────────────┐
│ 3. SELECT THE FEATURE                               │
│    Which GHAS feature resolves this?                │
└─────────────────┬───────────────────────────────────┘
                  │
┌─────────────────▼───────────────────────────────────┐
│ 4. VERIFY PREREQUISITES                             │
│    What is required? Permissions? Licenses?         │
└─────────────────────────────────────────────────────┘
```

---

## 📊 Quick Decision Matrix

### By PROBLEM TYPE

| Problem | Primary Feature | Secondary Feature | When it acts |
|---------|-----------------|-------------------|--------------|
| **Secret in code** | Secret scanning | Push protection | After / Before |
| **Vulnerable code** | CodeQL | - | During PR |
| **Vulnerable dependency** | Dependabot alerts | Security updates | After |
| **Prevent vuln in PR** | Dependency review | CodeQL | During PR |
| **Auto-update dependencies** | Dependabot security updates | - | After alert |
| **Org-wide visibility** | Security Overview | - | Continuous |
| **Compliance** | Security policies | Rulesets | Continuous |

### By TIMING IN THE SDLC

```
BEFORE COMMIT:
  └─ Push protection
      ├─ Blocks commits containing secrets
      └─ Requires: Secret scanning enabled

DURING THE PR:
  ├─ CodeQL (code analysis)
  ├─ Dependency review (new vulnerabilities)
  └─ Branch protection rules
      └─ Requires: Configured status checks

AFTER MERGE:
  ├─ Secret scanning (detection)
  ├─ Dependabot alerts (continuous monitoring)
  ├─ CodeQL scheduled scans
  └─ Security campaigns (remediation)

CONTINUOUS:
  └─ Security Overview (metrics and trends)
```

---

## 🎓 Decision Scenarios (Exam-Type)

### Scenario 1: Secrets

**Question**: A developer has just pushed an AWS access key to a public repository.

**What happens? (Select all that apply)**

A) ✅ Secret scanning detects the secret  
B) ✅ GitHub notifies the developer  
C) ✅ GitHub notifies AWS  
D) ❌ The secret is automatically revoked  
E) ✅ An alert is created in the Security tab  

**Explanation**:
- Secret scanning automatically detects secrets in public repos (A, E).
- It notifies the repo owner AND the service provider (B, C).
- It does NOT automatically revoke the secret—that must be done by the user or AWS (D is false).

**What should you do?**

```
1. IMMEDIATE (< 5 min):
   └─ Revoke the AWS access key in the AWS Console

2. SHORT-TERM (< 1 hour):
   ├─ Check CloudTrail logs
   ├─ Verify unauthorized use
   └─ Rotate credentials

3. PREVENTION:
   ├─ Enable push protection
   └─ Team training
```

---

### Scenario 2: Dependencies

**Question**: Your team wants PRs to be blocked automatically if they introduce dependencies with HIGH or CRITICAL vulnerabilities.

**What configuration do you need?**

A) Dependabot alerts  
B) Dependabot security updates  
C) Dependency review + branch protection ← **CORRECT**  
D) CodeQL  

**Explanation**:
- **Dependabot alerts**: Only NOTIFY, do not block.
- **Security updates**: Create fix PRs, do not block.
- **Dependency review**: Analyzes changes in PR + branch protection = BLOCK.
- **CodeQL**: Analyzes code, not dependencies.

**Complete Configuration**:

```yaml
# 1. Dependency review workflow
# .github/workflows/dependency-review.yml
name: Dependency Review
on: [pull_request]

jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/dependency-review-action@v4
        with:
          fail-on-severity: high

# 2. Branch protection rule
Settings → Branches → Branch protection rules
  ✓ Require status checks to pass
    ✓ Dependency Review
```

---

### Scenario 3: CodeQL

**Question**: A developer asks why CodeQL takes 30 minutes on their Java repository, but only 5 minutes on JavaScript.

**What is the correct reason?**

A) Java has more vulnerabilities  
B) Java requires compilation, JavaScript does not ← **CORRECT**  
C) CodeQL has fewer queries for JavaScript  
D) The runner is slower for Java  

**Explanation**:

| Language | Type | Build Required | Analysis |
|----------|------|----------------|----------|
| **Java** | Compiled | ✅ YES | Slow (10-60 min) |
| **JavaScript** | Interpreted | ❌ NO | Fast (2-10 min) |
| **Python** | Interpreted | ❌ NO | Fast (2-10 min) |
| **C++** | Compiled | ✅ YES | Slow (20-120 min) |

**CodeQL process for compiled languages**:

```
1. Initialize CodeQL
   └─ Setup CodeQL database
   
2. BUILD (← This is where the time goes)
   ├─ Download dependencies
   ├─ Compile code
   └─ Generate artifacts
   
3. Analyze
   └─ Run queries on database
   
4. Upload results
```

---

### Scenario 4: Permissions

**Question**: An external contractor needs to view Dependabot alerts in private repositories, but MUST NOT be able to modify code or settings.

**What role do you grant them?**

A) Read ← **CORRECT**  
B) Triage  
C) Write  
D) Security Manager  

**Detailed Explanation**:

| Role | View Dependabot Alerts | Dismiss Alerts | View Code | Modify Code |
|-----|----------------------|----------------|-----------|-------------|
| **Read** | ✅ | ❌ | ✅ | ❌ |
| **Triage** | ✅ | ❌ | ✅ | ❌ |
| **Write** | ✅ | ✅ | ✅ | ✅ |
| **Security Manager** | ✅ | ✅ | ✅ | ❌ |

**Why Read and not Security Manager?**

- **Security Manager**: This is an ORGANIZATION-level role, not repository-level.
- **Security Manager**: Grants security permissions across ALL repositories in the org.
- **Read**: Fulfills the exact requirement without over-privileging.

**CRITICAL Difference**:

```yaml
Security Manager (Organization-level):
  ✅ View security alerts in ALL org repos
  ✅ Configure security features
  ✅ Manage security campaigns
  ❌ Cannot modify code

Read (Repository-level):
  ✅ View code
  ✅ View Dependabot alerts (only in that repo)
  ❌ Cannot modify anything
```

---

### Scenario 5: Org vs Repo Settings

**Question**: The organization has secret scanning enabled for "all repositories." A repository admin disables it at the repository level.

**What happens?**

A) It gets disabled (repo override org) ← **Depends on enforcement**  
B) It remains enabled (org override repo)  
C) GitHub blocks the action  
D) Requires approval from the org owner  

**Explanation - Settings Hierarchy**:

```
┌─────────────────────────────────────────┐
│ ENTERPRISE                               │
│  ├─ Policies (enforce, allow, disable)  │
│  └─ If "enforce" → Cannot be changed     │
└──────────────┬──────────────────────────┘
               │
┌──────────────▼──────────────────────────┐
│ ORGANIZATION                             │
│  ├─ Can enable for all repos            │
│  ├─ Can auto-enable for new repos       │
│  └─ If there is no enterprise policy:    │
│     repo admin can override             │
└──────────────┬──────────────────────────┘
               │
┌──────────────▼──────────────────────────┐
│ REPOSITORY                               │
│  └─ Settings apply only if:             │
│     - There is no enterprise enforcement│
│     - Org allows overrides              │
└─────────────────────────────────────────┘
```

**Case A**: Without enterprise policy
- Org enables secret scanning.
- Repo admin can disable.
- ✅ **It gets disabled at the repo level.**

**Case B**: With enterprise enforcement
- Enterprise policy: "Enforced"
- Repo admin attempts to disable.
- ❌ **The option is grayed out / blocked.**

**Complete Answer**:
> It depends on whether enterprise policy enforcement is active. Without enforcement: the repo can override. With enforcement: it cannot be changed.

---

## 🔄 Decision Trees

### Decision Tree: "I found a vulnerability, what do I do?"

```
Where was the vulnerability detected?
    │
    ├─ In a DEPENDENCY
    │   │
    │   ├─ Is it already in main?
    │   │   ├─ YES → Dependabot alert exists
    │   │   │        └─ Is there a Dependabot PR?
    │   │   │            ├─ YES → Review and merge PR
    │   │   │            └─ NO → Create manual issue/PR
    │   │   │
    │   │   └─ NO → It is in a PR
    │   │            └─ Dependency review should have blocked it
    │   │                ├─ If it did not block: Configure dependency review
    │   │                └─ If it blocked: Do not merge until fixed
    │   │
    │   └─ Is it direct or transitive?
    │       ├─ Direct → Update in package.json
    │       └─ Transitive → Update parent dependency
    │                       or use overrides/resolutions
    │
    └─ In CODE (CodeQL)
        │
        ├─ Where is it?
        │   ├─ Production code → Immediate FIX
        │   ├─ Test code → FIX (lower priority)
        │   └─ Example/docs → Dismiss or fix
        │
        ├─ Severity?
        │   ├─ Critical → SLA: 24h
        │   ├─ High → SLA: 7 days
        │   ├─ Medium → SLA: 30 days
        │   └─ Low → SLA: 90 days
        │
        └─ Is it exploitable?
            ├─ YES → Maximum priority
            │        └─ View "Show paths" to understand data flow
            │            └─ Implement fix according to recommendation
            │
            └─ NO → Why?
                ├─ Unreachable code → Document + dismiss
                ├─ Input already sanitized → Document + dismiss
                ├─ Existing mitigations → Risk acceptance
                └─ False positive → Dismiss + report to GitHub
```

---

### Decision Tree: "Which GHAS feature should I enable first?"

```
What type of organization are you?
    │
    ├─ OPEN SOURCE (public repos)
    │   │
    │   └─ ✅ You already have it for FREE:
    │       ├─ Secret scanning
    │       ├─ Push protection
    │       ├─ Dependabot alerts
    │       └─ Dependabot security updates
    │       
    │       📋 Recommended Setup:
    │       1. Enable CodeQL (manual setup but free)
    │       2. Configure branch protection
    │       3. Setup Dependabot version updates (dependabot.yml)
    │
    └─ ENTERPRISE (private repos)
        │
        ├─ Do you have a GHAS license?
        │   │
        │   ├─ NO → Priority:
        │   │        1. Purchase GHAS (or at least GitHub Code Security)
        │   │        2. In the meantime: Only Dependabot alerts are available
        │   │
        │   └─ YES → Rollout strategy:
        │
        ├─ PHASE 1 (Weeks 1-2): VISIBILITY
        │   ├─ Enable Dependency graph
        │   ├─ Enable Dependabot alerts
        │   └─ Enable Security Overview
        │       └─ Identify current security posture
        │
        ├─ PHASE 2 (Weeks 3-4): DETECTION
        │   ├─ Enable Secret scanning
        │   ├─ Configure custom patterns (if applicable)
        │   └─ Enable CodeQL (default setup) in 10-20% of repos
        │       └─ Pilot on non-critical repos
        │
        ├─ PHASE 3 (Month 2): PREVENTION
        │   ├─ Enable Push protection
        │   ├─ Setup Dependency review
        │   ├─ Configure branch protection rules
        │   └─ Expand CodeQL to 50% of repos
        │
        ├─ PHASE 4 (Month 3): REMEDIATION
        │   ├─ Enable Dependabot security updates
        │   ├─ Configure auto-merge policies
        │   ├─ Setup Security campaigns
        │   └─ CodeQL in 100% of repos
        │
        └─ PHASE 5 (Month 4+): OPTIMIZATION
            ├─ Custom CodeQL queries
            ├─ Advanced workflows
            ├─ SIEM integration
            ├─ Automated governance
            └─ Continuous improvement
```

---

### Decision Tree: "A PR has security alerts, can I merge?"

```
PR has security alerts
    │
    ├─ What type of alert?
    │
    ├─── SECRET SCANNING
    │     │
    │     ├─ Is push protection enabled?
    │     │   ├─ YES → Commit was blocked
    │     │   │        └─ This scenario shouldn't happen unless bypassed
    │     │   │            └─ Was there a bypass?
    │     │   │                ├─ With valid justification → Review case
    │     │   │                └─ Without justification → BLOCK merge
    │     │   │
    │     │   └─ NO → Secret is in the branch
    │     │            └─ ❌ DO NOT MERGE
    │     │                ├─ 1. Revoke secret IMMEDIATELY
    │     │                ├─ 2. Clean history (git filter-repo)
    │     │                ├─ 3. Force push cleaned branch
    │     │                └─ 4. Re-review PR
    │     │
    ├─── CODEQL
    │     │
    │     ├─ Severity?
    │     │   │
    │     │   ├─ CRITICAL/HIGH
    │     │   │   └─ ❌ DO NOT MERGE (by policy)
    │     │   │       └─ Requires fix OR exception approval
    │     │   │           ├─ Fix → Best option
    │     │   │           └─ Exception → Requires:
    │     │   │               ├─ Security team approval
    │     │   │               ├─ Documented justification
    │     │   │               ├─ Mitigations in place
    │     │   │               └─ Time-bound (review in X months)
    │     │   │
    │     │   ├─ MEDIUM
    │     │   │   └─ ⚠️ Decision based on policy
    │     │   │       ├─ Strict policy → Block
    │     │   │       └─ Permissive → Warn + merge
    │     │   │           └─ Create issue for fix
    │     │   │
    │     │   └─ LOW
    │     │       └─ ✅ Generally OK to merge
    │     │           └─ Add to backlog
    │     │
    │     └─ Is it a false positive?
    │         ├─ YES → Dismiss alert
    │         │        └─ Document reason
    │         │            └─ ✅ Merge OK
    │         │
    │         └─ NO → See severity tree above
    │
    └─── DEPENDENCY REVIEW
          │
          ├─ Does it introduce new vulnerabilities?
          │   │
          │   ├─ YES
          │   │   ├─ Severity?
          │   │   │   ├─ Critical/High → ❌ BLOCKED by workflow
          │   │   │   │                    └─ Update dependency before merge
          │   │   │   │
          │   │   │   └─ Medium/Low → Based on fail-on-severity config
          │   │   │                     ├─ If configured to block → ❌ NO MERGE
          │   │   │                     └─ If not → ⚠️ Warning, can merge
          │   │   │
          │   │   └─ Forbidden license?
          │   │       ├─ YES → ❌ BLOCKED
          │   │       │        └─ Remove dependency or find alternative
          │   │       │
          │   │       └─ NO → Continue
          │   │
          │   └─ NO → ✅ Dependency review PASS
          │            └─ OK to merge (from dependency perspective)
          │
          └─ Does branch protection require this check?
              ├─ YES → Must pass to merge
              └─ NO → Informational only
```

---

# ADVANCED GOVERNANCE AND ROLES <a id="2"></a>

## 🏢 Complete Permissions Hierarchy

### Level 1: Repository Roles

| Role | Security Permissions |
|-----|----------------------|
| **Read** | • View code<br>• View Dependabot alerts<br>• View discussions |
| **Triage** | • Everything in Read<br>• Manage issues and PRs<br>• **CANNOT view Code scanning or Secret scanning** |
| **Write** | • Everything in Triage<br>• Push code<br>• Dismiss Dependabot alerts<br>• **CANNOT view Code scanning or Secret scanning** |
| **Maintain** | • Everything in Write<br>• Manage settings (limited)<br>• **CANNOT view Code scanning or Secret scanning** |
| **Admin** | • Everything in Maintain<br>• **VIEW and manage Code scanning**<br>• **VIEW and manage Secret scanning**<br>• Configure GHAS features<br>• Manage branch protection |

**⚠️ CRITICAL FOR THE EXAM**:
- Only **Admin** can view Code scanning and Secret scanning alerts in private repositories.
- **Dependabot alerts** are visible to anyone with **Read+** role (an exception to the rule).

---

### Level 2: Organization Roles

| Role | Scope | GHAS Permissions |
|-----|---------|---------------|
| **Member** | Assigned repos | Based on role in each repo |
| **Outside Collaborator** | Specific repos | Based on assigned role |
| **Owner** | Entire organization | • All permissions<br>• Configure org-wide security<br>• Manage billing<br>• Enforce policies |
| **Security Manager** | Entire organization (security only) | • **View** security alerts in all repos<br>• **Manage** security settings<br>• **CANNOT** view/modify code<br>• **CANNOT** change non-security repo settings |

**Security Manager in Detail**:

```yaml
✅ CAN:
  - View Code scanning alerts (all repos)
  - View Secret scanning alerts (all repos)
  - View Dependabot alerts (all repos)
  - Dismiss/reopen alerts
  - Configure security features
  - Manage security campaigns
  - Access Security Overview
  - Configure custom patterns
  - Manage security policies

❌ CANNOT:
  - View code (unless they have an additional repo role)
  - Modify code
  - Merge PRs
  - Change branch protection (unless security-related)
  - Manage billing
  - Add/remove members
  - Delete repositories
```

**When to use Security Manager**:

```
✅ Use when:
  - Centralized security team
  - Cross-repository auditing
  - Compliance officer
  - External security consultant

❌ DO NOT use when:
  - Needs to modify code
  - Needs to manage repos
  - Developer needs to see alerts in their OWN repo
    └─ Better: Repository role (Read/Write/Admin)
```

---

### Level 3: Enterprise Policies

**Only available in GitHub Enterprise Cloud**

```yaml
Enterprise Admin can:
  ├─ Enforced: ALL orgs MUST have enabled
  │   └─ Orgs cannot disable
  │   └─ Repos cannot disable
  │
  ├─ Enabled: Enabled by default, orgs can override
  │   └─ Org can disable
  │   └─ Repos can disable (if org allows)
  │
  └─ Disabled: Disabled enterprise-wide
      └─ Orgs CANNOT enable

Applies to:
  - Secret scanning
  - Push protection
  - Dependabot alerts
  - Dependabot security updates
  - Code scanning
```

**Enforcement Example**:

```
Scenario 1: Enterprise enforced secret scanning
  Enterprise: Enforced ✅
    └─ Organization A: Enforced (no choice)
        └─ Repo 1: Enforced (no choice)
        └─ Repo 2: Enforced (no choice)
    └─ Organization B: Enforced (no choice)
        └─ Repo 3: Enforced (no choice)

Scenario 2: Enterprise enabled (not enforced)
  Enterprise: Enabled ⚙️
    ├─ Organization A: Enabled (can change)
    │   ├─ Repo 1: Enabled (can change)
    │   └─ Repo 2: Disabled (admin chose to disable)
    │
    └─ Organization B: Disabled (owner chose to disable)
        └─ Repo 3: Disabled (inherited from org)
```

---

## 🔐 Security Policies and Rulesets

### Branch Protection Rules (Legacy)

```yaml
Limitations:
  ❌ Only protects specific branches
  ❌ Configured per repo
  ❌ Cannot be applied org-wide
  ❌ Difficult to maintain at scale

Use when:
  - Individual repo
  - Simple configuration
  - You do not have Enterprise
```

### Repository Rulesets (New - Recommended)

```yaml
Advantages:
  ✅ Applies to multiple branches using patterns
  ✅ Can be configured org-wide
  ✅ Bypass with approval workflow
  ✅ More flexible and granular

Levels:
  1. Repository ruleset
     └─ Applies only to that repo
  
  2. Organization ruleset
     └─ Applies to selected or all repos
     
  3. Enterprise ruleset (GHEC)
     └─ Applies to all orgs

Example Configuration:
  Name: Security Standards
  Target: Default branch + release/*
  
  Rules:
    ✓ Require status checks to pass:
      - CodeQL / Analyze (javascript)
      - Dependency Review
    
    ✓ Require pull request before merging
      - Require 1 approval
      - Dismiss stale approvals
      - Require review from code owners
    
    ✓ Block force pushes
    ✓ Require signed commits
  
  Bypass:
    - Organization admins (with approval)
    - Security team (no approval needed)
```

**Critical Comparison**:

| Feature | Branch Protection | Rulesets |
|---------|------------------|----------|
| **Scope** | Per branch | By pattern |
| **Level** | Repository | Repo/Org/Enterprise |
| **Bypass** | Simple on/off | Approval workflow |
| **Status checks** | Simple list | Conditions + matrix |
| **Maintenance** | Manual per repo | Centralized |
| **Recommendation**| ❌ Legacy | ✅ Use this |

---

## 📊 Security Policies Matrix

### What feature can be configured where?

| Feature | Repository | Organization | Enterprise |
|---------|-----------|--------------|------------|
| **Secret Scanning** | ✅ Enable/Disable | ✅ Enable for all<br>✅ Auto-enable new | ✅ Enforce policy |
| **Push Protection** | ✅ Enable/Disable | ✅ Enable for all<br>✅ Delegated bypass | ✅ Enforce policy |
| **Custom Patterns** | ❌ | ✅ Org-wide patterns | ❌ |
| **Code Scanning** | ✅ Configure workflow | ✅ Default setup org-wide | ✅ Enforce policy |
| **CodeQL Queries** | ✅ Custom config | ✅ Org default queries | ❌ |
| **Dependabot Alerts** | ✅ Enable/Disable | ✅ Enable for all | ✅ Enforce policy |
| **Security Updates** | ✅ Enable/Disable | ✅ Enable for all | ✅ Enforce policy |
| **Dependency Review** | ✅ Workflow config | ✅ Required workflow | ✅ Policy |
| **Rulesets** | ✅ Repo rulesets | ✅ Org rulesets | ✅ Enterprise rulesets |
| **Security Overview** | ❌ | ✅ Org-level view | ✅ Enterprise view |

---

## 🎯 Governance Best Practices

### Enterprise Rollout Strategy

```yaml
Phase 1: DISCOVERY (Month 1)
  Objective: Understand current posture
  
  Actions:
    1. Enable Security Overview
       └─ Identify which repos already have GHAS
    
    2. Enable Dependency Graph everywhere
       └─ Supply chain visibility
    
    3. Run security assessment
       └─ Generate baseline metrics
  
  Metrics:
    - % repos with GHAS enabled
    - Total alerts by type
    - High-risk repos (open critical alerts)

Phase 2: PILOT (Month 2)
  Objective: Validate configuration
  
  Repo selection:
    ✓ 5-10 non-critical repos
    ✓ Diverse languages
    ✓ Different teams
    ✓ Representative sample
  
  Actions:
    1. Enable all features
    2. Configure workflows
    3. Train teams
    4. Collect feedback
  
  Success Criteria:
    - <10% false positive rate
    - <7 days MTTR for high severity
    - >80% developer satisfaction

Phase 3: EXPANSION (Months 3-4)
  Objective: Scale to 50% of repos
  
  Prioritization:
    1. Repos handling sensitive data
    2. Production apps
    3. Public-facing services
    4. Compliance-regulated
  
  Automation:
    - Bulk enablement script
    - Auto-enable for new repos
    - Centralized policies

Phase 4: FULL ROLLOUT (Months 5-6)
  Objective: 100% coverage
  
  Enforcement:
    - Organization policies
    - Required workflows
    - Universal branch protection
  
  Governance:
    - Security campaigns
    - Regular audits
    - Metrics dashboards

Phase 5: OPTIMIZATION (Ongoing)
  Objective: Continuous improvement
  
  Activities:
    - Custom queries
    - SIEM integration
    - Automated remediation
    - Developer education
```

---

### Organization Settings - Complete Checklist

```yaml
Code security and analysis:
  
  Dependency graph:
    [✓] Enable for all repositories
    [✓] Automatically enable for new repositories
  
  Dependabot:
    [✓] Enable Dependabot alerts for all repositories
    [✓] Enable Dependabot security updates for all repositories
    [✓] Automatically enable for new repositories
    
    Grouping (GitHub Code Security required):
      [✓] Enable auto-triage rules
      - Rule 1: Auto-dismiss low severity without patch
      - Rule 2: Auto-create PRs for critical in production
  
  Code scanning:
    [✓] Automatically enable for new repositories
    [ ] Require approval for new workflows (if tight control is needed)
    
    Default setup:
      Languages: [✓] Auto-detect
      Query suite: security-extended
      
  Secret scanning:
    [✓] Enable for all repositories
    [✓] Enable push protection for all repositories
    [✓] Automatically enable for new repositories
    
    Push protection:
      [✓] Allow bypasses
      [✓] Require justification for bypass
      [ ] Require approval for bypass (Enterprise only)
      Bypass expires: 7 days
    
    Custom patterns:
      - Pattern 1: Internal API keys
      - Pattern 2: Database connection strings
      - Pattern 3: JWT tokens
  
  Security managers:
    Teams:
      - @org/security-team
      - @org/compliance-team

Member privileges:
  Base permissions: Read
  
  Repository creation:
    [✓] Allow members to create repositories
    [ ] Allow members to create public repositories (if private org)
  
  Repository visibility change:
    [ ] Allow members to change repository visibilities

Actions permissions:
  [✓] Allow GitHub Actions
  [✓] Allow actions created by GitHub
  [✓] Allow actions by Marketplace verified creators
  [ ] Allow specified actions (better control)

Webhooks:
  - Webhook 1: Security alerts → SIEM
  - Webhook 2: Dependabot → Slack #security
```

---
# SECURITY OVERVIEW IN DEPTH <a id="3"></a>

## 📊 Security Overview Capabilities

### Available Views

```yaml
1. SECURITY RISK
   Purpose: Identify high-risk repositories
   
   Sort by:
     - Number of critical alerts
     - Number of high alerts
     - Age of oldest alert
     - Combined risk score
   
   Use for:
     - Prioritizing remediation efforts
     - Identifying hotspots
     - Executive dashboards

2. SECURITY COVERAGE
   Purpose: Adoption tracking
   
   Shows:
     - % of repositories with GHAS enabled
     - % of repositories with each feature enabled
     - Coverage gaps
   
   Use for:
     - Rollout planning
     - Compliance reporting
     - Identifying lagging teams/repositories

3. SECURITY ALERTS
   Purpose: Detailed view of all alerts
   
   View:
     - All alerts in the organization
     - Filterable by type, severity, status
     - Bulk actions available
   
   Use for:
     - Daily triage
     - Bulk dismissal
     - Trend analysis

4. SECURITY CAMPAIGNS (Enterprise)
   Purpose: Coordinated remediation
   
   Enables:
     - Grouping similar alerts
     - Assigning to teams
     - Tracking progress
     - Setting deadlines
   
   Use for:
     - Log4Shell-type security incidents
     - Tech debt remediation sprints
     - Compliance deadlines
```

---

## 📈 Key Metrics

### Coverage Metrics

```yaml
1. GHAS Adoption Rate
   Formula: (Repositories with GHAS / Total repositories) × 100
   
   Benchmark:
     - Excellent: >90%
     - Good: 70-90%
     - Needs improvement: <70%
   
   Action items if low:
     - Identify repositories without GHAS
     - Prioritize by criticality
     - Enable auto-activation for new repositories

2. Feature Adoption by Type
   Individual metrics:
     - % with Secret Scanning: Target 100%
     - % with Code Scanning: Target 100%
     - % with Dependabot: Target 100%
     - % with Push Protection: Target 90%+
   
   Segmentation:
     - By team
     - By language
     - By criticality

3. Configuration Quality
   Indicators:
     - % of repositories with branch protection
     - % of repositories with required status checks
     - % of repositories with custom CodeQL configuration
   
   Quality Score:
     - Gold: All features + custom config
     - Silver: All features + basic config
     - Bronze: Basic features only
     - None: Without GHAS
```

### Risk Metrics

```yaml
1. Open Alerts by Severity
   
   Critical:
     Total: 5
     Avg age: 3 days
     Oldest: 10 days ⚠️
     SLA compliance: 80% (target: 100%)
   
   High:
     Total: 23
     Avg age: 12 days
     Oldest: 45 days ⚠️
     SLA compliance: 65% (target: 90%)
   
   Medium:
     Total: 156
     Avg age: 34 days
     
   Low:
     Total: 412
     (tracked but not prioritized)

2. Alert Age Distribution
   
   Buckets:
     0-7 days: 45 alerts ✅ (fresh)
     8-30 days: 123 alerts ⚠️ (aging)
     31-90 days: 67 alerts 🔴 (stale)
     90+ days: 12 alerts 🔴🔴 (critical backlog)
   
   Action thresholds:
     - Any alert >90 days: Executive escalation
     - Critical >7 days: Manager review
     - High >30 days: Team retrospective

3. Vulnerability Density
   
   Formula: Alerts / 1000 LOC
   
   Benchmarks:
     - Excellent: <0.5
     - Good: 0.5-2.0
     - Needs improvement: >2.0
   
   Segmentation:
     - By repository
     - By team
     - By language
```

### Remediation Metrics

```yaml
1. Mean Time To Resolve (MTTR)
   
   By severity:
     Critical: 2.3 days (target: <1 day)
     High: 8.5 days (target: <7 days)
     Medium: 25 days (target: <30 days)
     Low: 65 days (target: <90 days)
   
   Trend: ↓ -15% vs last month ✅

2. Fix Rate
   
   Formula: (Alerts fixed / Total alerts) × 100
   
   Current month: 78% ✅
   Last month: 72%
   Trend: ↑ improving
   
   Breakdown:
     - Dependabot auto-fixed: 45%
     - Developer fixed: 33%
     - Dismissed (valid): 15%
     - Still open: 7%

3. Recurrence Rate
   
   Formula: (Re-opened alerts / Total fixed) × 100
   
   Current: 3% ✅ (target: <5%)
   
   Causes:
     - Dependency re-downgrade: 60%
     - Code regression: 30%
     - False dismissal: 10%
   
   Prevention:
     - Lock dependency versions
     - Regression tests
     - Better triage training

4. Security Debt Trend
   
   Total alerts over time:
     Jan: 450
     Feb: 423 ↓
     Mar: 389 ↓
     Apr: 412 ↑ ⚠️
      
   Analysis:
     - ↑ in April due to a new CVE wave
     - Underlying trend: improving
     - Velocity: -15 alerts/month average
```

---

## 🎯 Practical Use of Security Overview

### Dashboard for Executives (C-level)

```yaml
Monthly report must include:

1. EXECUTIVE SUMMARY
   "Our security posture improved by 12% this month."
   
   Key Metrics:
     - Total critical/high alerts: 28 (↓15%)
     - MTTR for critical alerts: 2.3 days (↓0.7 days)
     - GHAS coverage: 94% (↑4%)
     - Security debt: $150k (↓$25k)
   
   Status: 🟢 On track

2. RISK HEAT MAP
   
   High-Risk Repositories:
     🔴 payment-service: 5 critical, 12 high
     🔴 user-api: 3 critical, 8 high
     🟡 frontend-app: 0 critical, 15 high
   
   Action: CTO review required

3. COMPLIANCE STATUS
   
   PCI-DSS:
     ✅ 100% of payment repositories have GHAS enabled
     ✅ All critical alerts are <7 days old
     ⚠️ 2 repositories lack code scanning
   
   SOC2:
     ✅ Security policies enforced
     ✅ Complete audit trail
   
   Overall: 🟢 Compliant

4. COST AVOIDANCE
   
   Vulnerabilities prevented: 45
   Estimated breach cost avoided: $2.3M
   GHAS investment: $150k/year
   ROI: 15x ✅
```

### Dashboard for the Security Team

```yaml
Daily Dashboard:

1. TRIAGE QUEUE
   
   New alerts (last 24h):
     Critical: 2 🔴
       - CVE-2024-1234 in payment-service
       - Secret leak in user-api
     High: 5 🟡
     Medium: 12
   
   Action items:
     [Assign] CVE-2024-1234 → @security-team
     [Create incident] Secret leak

2. SLA TRACKING
   
   At risk (nearing SLA violation):
     - payment-service: critical alert day 6/7 ⚠️
     - api-gateway: high alert day 28/30 ⚠️
   
   Action: Alert responsible teams

3. CAMPAIGN PROGRESS
   
   "Log4j Remediation":
     Target: 45 repositories
     Completed: 38 (84%) ✅
     In progress: 5
     Blocked: 2
     Deadline: 3 days
   
   Status: On track

4. TRENDS
   
   Week-over-week:
     New alerts: +8 (↑)
     Fixed alerts: +15 (↑↑)
     Net change: -7 (↓) ✅
   
   Emerging patterns:
     - ↑ SQL injection findings (new CodeQL query)
     - ↑ npm dependencies vulnerabilities
```

### Dashboard for Engineering Managers

```yaml
Team Performance Dashboard:

1. TEAM SECURITY SCORE
   
   Frontend Team:
     Alerts per 1000 LOC: 1.2 🟡
     MTTR: 9 days 🟡
     Coverage: 100% ✅
     Grade: B+
   
   Backend Team:
     Alerts per 1000 LOC: 0.6 ✅
     MTTR: 5 days ✅
     Coverage: 100% ✅
     Grade: A

2. SPRINT HEALTH
   
   Current sprint:
     Security stories: 3/5 completed
     Dependabot PRs: 8/12 merged
     Alerts opened: 4
     Alerts fixed: 7
     Net: -3 ✅

3. BLOCKERS
   
   PRs blocked by security checks: 2
     - PR #456: High severity CodeQL finding
     - PR #789: New vulnerable dependency introduced
   
   Action needed: Dev team review

4. TRAINING NEEDS
   
   Based on alert patterns:
     - SQL injection: 5 occurrences
       → Recommend: OWASP secure coding training
     - Hardcoded secrets: 3 occurrences
       → Recommend: Secret management workshop
```

---

## 🔍 Advanced Filters in Security Overview

### Complex Query Construction

```yaml
Example 1: "All critical alerts in production repositories"

Filters:
  is:open
  severity:critical
  archived:false
  topic:production

Result:
  12 alerts across 5 repositories
  
Action:
  [Bulk assign] → @security-oncall


Example 2: "Vulnerable Node.js dependencies with patches available"

Filters:
  is:open
  ecosystem:npm
  has:patch
  severity:high,critical

Result:
  23 alerts
  
Action:
  [Security campaign] "Node.js March Updates"
  Deadline: End of week


Example 3: "Unreviewed CodeQL alerts in active PRs"

Filters:
  is:open
  tool:CodeQL
  state:unreviewed
  pr:open

Result:
  8 alerts
  
Action:
  [Notify] PR authors


Example 4: "Active exposed AWS secrets"

Filters:
  is:open
  tool:secret-scanning
  secret-type:aws_access_key_id
  validity:active

Result:
  3 alerts 🚨
  
Action:
  [IMMEDIATE] Revoke via AWS Console
  [Create incident] INC-2024-045
```

### Saved Filters (Best Practices)

```yaml
Pre-configure common filters:

1. "Daily Triage"
   is:open
   created:>$(date -7days)
   severity:critical,high
   
   Use: Every morning

2. "SLA at Risk"
   is:open
   severity:critical
   created:<$(date -6days)
   
   Use: Daily health monitoring

3. "Team: Backend"
   is:open
   team:@org/backend
   
   Use: Team standups

4. "Compliance Report"
   repository:payment-*
   archived:false
   
   Use: Quarterly audits

5. "Quick Wins"
   is:open
   severity:medium,low
   has:patch
   
   Use: Downtime/learning time
```

---

## 📥 Export and Report

### CSV Export

```yaml
Usage:
  Security Overview → [Export] → CSV

Contains:
  - Alert ID
  - Repository
  - Severity
  - CWE / CVE
  - Created date
  - Age (days)
  - Status
  - Assigned to
  
Use for:
  - Executive reports
  - Trend analysis in Excel/Google Sheets
  - Integration with BI tools
  - Compliance documentation
```

### API for Automation

```bash
# Get all critical alerts
gh api graphql -f query='
  query($org: String!) {
    organization(login: $org) {
      repositories(first: 100) {
        nodes {
          name
          vulnerabilityAlerts(first: 10, states: OPEN) {
            nodes {
              securityAdvisory {
                severity
                description
                publishedAt
              }
              vulnerableManifestPath
            }
          }
        }
      }
    }
  }
' -f org=my-org

# Create weekly report
curl -H "Authorization: token $GITHUB_TOKEN" \
     https://api.github.com/orgs/my-org/code-scanning/alerts \
  | jq '[.[] | select(.state=="open") | {
      repo: .repository.name,
      severity: .rule.security_severity_level,
      age: (now - (.created_at | fromdateiso8601)) / 86400 | floor
    }] | group_by(.severity) | map({
      severity: .[0].severity,
      count: length,
      avg_age: (map(.age) | add / length)
    })'
```

---

# CRITICAL COMPARISONS <a id="4"></a>

## 🔄 Dependabot: Alerts vs Security Updates vs Version Updates

### Master Table

| Aspect | Dependabot Alerts | Dependabot Security Updates | Dependabot Version Updates |
|---------|------------------|----------------------------|---------------------------|
| **Purpose** | Detect vulnerabilities | Auto-fix vulnerabilities | Keep dependencies updated |
| **What it detects** | Only vulnerabilities (CVEs) | Only vulnerabilities | All available updates |
| **When it acts** | Upon detecting vulnerability | After alert is generated | According to schedule |
| **Action** | Creates alert in UI | Creates fix PR | Creates update PR |
| **Automatic** | ✅ Yes | ✅ Yes | ✅ Yes |
| **Requires config** | ❌ No | ❌ No | ✅ Yes (dependabot.yml) |
| **Free for public** | ✅ | ✅ | ✅ |
| **Free for private** | ✅ | ✅ | ✅ |
| **Requires GHAS** | ❌ No (since 2022) | ❌ No | ❌ No |
| **Frequency** | Continuous | On alert creation | Configurable |
| **Scope** | Vulnerable only | Vulnerable only | All dependencies |
| **PRs created** | 0 | 1 per vulnerability | Many (based on configuration)|
| **Severity** | All | Only if patch is available | N/A |

### Use Cases

```yaml
Use Dependabot ALERTS when:
  ✓ You want visibility into vulnerabilities
  ✓ You want continuous monitoring
  ✓ You are not ready for automated PRs

Use Dependabot SECURITY UPDATES when:
  ✓ You want automated remediation
  ✓ You trust semantic versioning (SemVer)
  ✓ You have robust test suites
  ✓ You want to significantly reduce MTTR

Use Dependabot VERSION UPDATES when:
  ✓ You want to prevent technical debt
  ✓ You want the latest package features
  ✓ You want to keep all dependencies current
  ✓ You are updating packages for non-security reasons too
```

### Flow Diagram

```
┌──────────────────────────────────────────────┐
│ New CVE published for lodash@4.17.15         │
└────────────────┬─────────────────────────────┘
                 │
┌────────────────▼─────────────────────────────┐
│ DEPENDABOT ALERTS                             │
│ ✅ Detects that your repo uses lodash@4.17.15 │
│ ✅ Creates alert in Security tab              │
│ ✅ Notifies administrators                    │
└────────────────┬─────────────────────────────┘
                 │
                 ├─ Is Security Updates enabled?
                 │
                 ├─ NO → Alert remains in place.
                 │        Developer must update manually.
                 │
                 └─ YES ↓
┌──────────────────────────────────────────────┐
│ DEPENDABOT SECURITY UPDATES                   │
│ ✅ Verifies that lodash@4.17.21 exists       │
│ ✅ Verifies compatibility (SemVer)           │
│ ✅ Creates branch: dependabot/npm_and_yarn/...│
│ ✅ Updates package.json + lockfile           │
│ ✅ Opens PR with changelog                   │
└────────────────┬─────────────────────────────┘
                 │
                 ├─ Developer reviews PR
                 ├─ Automated tests run (CI)
                 ├─ Merge PR
                 │
                 └─ Alert is auto-dismissed/closed ✅
                 
┌──────────────────────────────────────────────┐
│ DEPENDABOT VERSION UPDATES (Separate)        │
│                                              │
│ According to schedule (e.g., weekly):       │
│ ✅ Check if lodash is current: 4.17.21       │
│ ✅ Check if lodash has a newer version: 5.0.0│
│ ✅ Creates PR to upgrade to 5.0.0            │
│ ⚠️  MAJOR version update - review required   │
└──────────────────────────────────────────────┘
```

---

## 🔐 Secret Scanning vs Push Protection

### Fundamental Comparison

| Aspect | Secret Scanning | Push Protection |
|---------|----------------|-----------------|
| **Timing** | After commit / push | Before commit is accepted |
| **Action** | Detects and alerts | Blocks push |
| **Scope** | Entire history | New commits |
| **Retroactive**| ✅ Yes | ❌ No |
| **Preventative**| ❌ No | ✅ Yes |
| **Can be bypassed**| N/A | ✅ Yes (with justification) |
| **Free for public**| ✅ | ✅ |
| **Free for private**| ❌ (requires GHAS) | ❌ (requires GHAS) |

### Timeline Comparison

```
WITHOUT Push Protection:
  Developer writes code containing a secret
    ↓
  git commit
    ✅ Success
    ↓
  git push
    ✅ Success (secret is now on GitHub)
    ↓
  [30 seconds later]
  Secret Scanning detects secret
    ↓
  Alert is created
  Notification is sent
    ↓
  ⚠️ BUT: Secret is ALREADY exposed in repository history
  ⚠️ Bots have already scanned and cloned it
  ⚠️ Potential breach damage has already occurred

WITH Push Protection:
  Developer writes code containing a secret
    ↓
  git commit
    ✅ Success (local commit only)
    ↓
  git push
    ❌ BLOCKED!
    
    remote: ❌ Push protection: Secret detected
    remote: 
    remote: GitHub Personal Access Token found
    remote:   Location: src/config.js:15
    remote:   Type: github_pat
    remote:   Status: Active
    remote: 
    remote: To push anyway:
    remote:   1. Revoke the secret
    remote:   2. Remove from code
    remote:   3. Request bypass (justification required)
    
    ↓
  Developer MUST act BEFORE they can push
    ↓
  ✅ Secret never reaches GitHub
  ✅ No exposure
  ✅ No alerts generated
```

### Recommended Configuration

```yaml
✅ ENABLE BOTH:

Secret Scanning:
  Purpose: Safety net
  Detects: Historical + newly introduced secrets
  Use cases:
    - Migrating existing repositories
    - Secrets hidden in old branch histories
    - Handling push protection bypasses

Push Protection:
  Purpose: Prevention
  Blocks: Incoming commits containing secrets
  Use cases:
    - Preventing initial exposure
    - Educating development teams
    - Enforcing compliance policies

Together:
  Defense in Depth ✅
  Push protection prevents the leak.
  Secret scanning catches it if an exemption or bypass occurs.
```

---

## 📊 CodeQL: Default Setup vs Advanced Setup

### Complete Comparison

| Aspect | Default Setup | Advanced Setup |
|---------|--------------|----------------|
| **Configuration** | 1-click | Manual workflow definition |
| **Setup Time** | 30 seconds | 10-30 minutes |
| **File Required** | None | `.github/workflows/codeql.yml` |
| **Customization** | Limited | Complete |
| **Query Suites** | `default` (fixed) | Any suite |
| **Custom Queries** | ❌ No | ✅ Yes |
| **Build Control** | Automated | Manual build steps possible |
| **Scheduled Scans**| Weekly (fixed) | Configurable Cron expression |
| **Languages** | Auto-detected | Manually specified |
| **Path Filters** | ❌ No | ✅ Yes |
| **Config File** | ❌ No | ✅ Yes (codeql-config.yml) |
| **Runners** | GitHub-hosted | GitHub-hosted or self-hosted |
| **Recommended for**| 80% of standard repos | Advanced users, custom needs |

### When to Use Each Setup

```yaml
Use DEFAULT SETUP when:
  ✓ New or standard repository
  ✓ No customization needed
  ✓ Standard interpreted languages (JS, Python, Ruby, Go)
  ✓ You want to get started quickly
  ✓ No deep CodeQL expertise exists in-house
  ✓ Codebase size is <100k LOC
  ✓ Default query suite is sufficient

Use ADVANCED SETUP when:
  ✓ You need to run custom queries
  ✓ Compiled languages with complex build systems
  ✓ You need customized Cron scheduling
  ✓ Path filtering is required (ignoring test dirs, etc.)
  ✓ You want to execute multiple query suites
  ✓ Using self-hosted runners
  ✓ Monorepo with multiple languages
  ✓ Compliance policies require specific query configurations
  ✓ Integrating with external CI/CD pipelines
```

### Migration Path

```yaml
Migrating from Default to Advanced:

1. Export current configuration:
   Settings → Code security → Code scanning
   → View CodeQL default setup
   → Click [Generate advanced setup file]

2. Commit workflow:
   `.github/workflows/codeql.yml` is created and committed
    
3. Customize:
   - Add custom queries
   - Configure custom schedules
   - Add path filters
   - Etc.

4. Default setup is automatically disabled
   (prevents conflicts)

⚠️ You CANNOT run both simultaneously on the same repo.
```

---

## 🔍 Dependency Review vs Dependabot Alerts

### Fundamental Difference

```yaml
DEPENDABOT ALERTS:
  └─ Reactive
      └─ "You already have this problem"
          └─ Appears in Security tab
              └─ Post-merge (in main branch)

DEPENDENCY REVIEW:
  └─ Proactive
      └─ "This merge would introduce a problem"
          └─ Appears in PR checks
              └─ Pre-merge (during review phase)
```

### Comparison Table

| Aspect | Dependabot Alerts | Dependency Review |
|---------|------------------|-------------------|
| **Scope** | Entire repository | Only changes introduced in PR |
| **Timing** | Post-merge | Pre-merge |
| **Goal** | Detect existing vulnerabilities | Prevent new ones |
| **UI Location** | Security tab | PR checks / Action run |
| **Blocks Merge** | ❌ No | ✅ Yes (configurable via rulesets) |
| **Implementation**| Automatic / 1-click | GitHub Action workflow required |
| **Requires GHAS** | ❌ No | ✅ Yes |
| **Verification** | Vulnerabilities | Vulnerabilities + licenses + score |
| **PRs Generated** | Security update PRs | None (only status check report) |

### Complete Flow

```
Scenario: Developer updates axios from 0.21.0 to 0.21.4
```

```
┌─────────────────────────────────────────────┐
│ Developer modifies package.json             │
│   axios: "0.21.0" → "0.21.4"                │
└────────────┬────────────────────────────────┘
             │
┌────────────▼────────────────────────────────┐
│ git commit & push                            │
└────────────┬────────────────────────────────┘
             │
┌────────────▼────────────────────────────────┐
│ Opens PR to main                            │
└────────────┬────────────────────────────────┘
             │
             ├─ DEPENDENCY REVIEW executes
             │
┌────────────▼────────────────────────────────┐
│ Dependency Review Action                     │
│ Compares:                                    │
│   Base (main): axios@0.21.0                 │
│   Head (PR):   axios@0.21.4                 │
│                                              │
│ Result:                                      │
│   ✅ Security update                        │
│   ✅ Resolves CVE-2021-3749                 │
│   ✅ Introduces no new vulnerabilities       │
│   ✅ License compatible                     │
│                                              │
│ Check: ✅ PASS                               │
└────────────┬────────────────────────────────┘
             │
┌────────────▼────────────────────────────────┐
│ Merge allowed                                │
└────────────┬────────────────────────────────┘
             │
┌────────────▼────────────────────────────────┐
│ Post-merge:                                  │
│ Dependabot alert for axios@0.21.0 is         │
│ automatically closed                         │
│ (vulnerability resolved)                    │
└─────────────────────────────────────────────┘


Scenario 2: Developer downgrades vulnerable package

┌─────────────────────────────────────────────┐
│ Developer modifies package.json             │
│   axios: "0.27.0" → "0.21.0" (downgrade!)   │
└────────────┬────────────────────────────────┘
             │
             [... git commit, push, PR ...]
             │
┌────────────▼────────────────────────────────┐
│ Dependency Review Action                     │
│ Compares:                                    │
│   Base (main): axios@0.27.0 (secure)        │
│   Head (PR):   axios@0.21.0 (vulnerable)    │
│                                              │
│ Result:                                      │
│   ❌ Introduces CVE-2021-3749                │
│   ❌ Severity: High                          │
│                                              │
│ Check: ❌ FAIL                               │
└────────────┬────────────────────────────────┘
             │
┌────────────▼────────────────────────────────┐
│ PR BLOCKED                                  │
│ Cannot merge until:                          │
│   1. Axios is updated to a secure version    │
│   2. Or check is bypassed (if allowed)       │
└─────────────────────────────────────────────┘
```

### Complementary Configuration

```yaml
# Both enabled = Defense in Depth

1. Dependabot Alerts (always active):
   └─ Detects vulnerabilities in main
   └─ Generates security update PRs

2. Dependency Review (on PRs):
   └─ Blocks new vulnerabilities
   └─ Prevents security regressions

Result:
  ✅ No vulnerable packages reach main
  ✅ Vulnerabilities already in main are detected and fixed
  ✅ 360° supply chain coverage
```

---

## 🛡️ Code Scanning Tools: CodeQL vs Third-party

### Strategic Comparison

| Aspect | CodeQL | Third-party (Snyk, Sonar, etc.) |
|---------|--------|---------------------------------|
| **Provider** | GitHub (free with GHAS) | External Vendor (additional cost) |
| **Integration** | Native (1-click) | SARIF upload |
| **Languages** | 15+ | Varies by tool |
| **Customization** | Open source queries (QL) | Vendor-dependent |
| **Database** | GitHub Advisory DB | Vendor DB + public sources |
| **False Positives**| Low to medium | Varies |
| **Data Flow Analysis**| ✅ Excellent | Varies |
| **Taint Tracking** | ✅ Yes | Some vendors |
| **Supply Chain** | ❌ (uses Dependabot) | Some vendors provide |
| **License** | Free with GHAS | Separate licensing |
| **Support** | GitHub Support | Vendor-specific support |

### When to Use Which

```yaml
Use ONLY CodeQL when:
  ✓ You have a active GHAS license
  ✓ All codebase languages are supported by CodeQL
  ✓ You do not require vendor-specific reporting features
  ✓ You want to simplify your security tooling footprint
  ✓ Budget is constrained

Use CodeQL + Third-party when:
  ✓ Implementing Defense in Depth
  ✓ Compliance policies require multiple security tools
  ✓ Codebase includes languages not supported by CodeQL
  ✓ You need specific vendor features (e.g., Snyk container scanning)
  ✓ You already have substantial pre-existing investment in the vendor tool

Use ONLY Third-party when:
  ✓ You do not have GHAS licensed
  ✓ You already have a mature enterprise contract with the vendor
  ✓ A specific tool is an strict requirement (e.g., SonarQube Enterprise)
```

---

# COMMON EXAM PITFALLS <a id="5"></a>

## 🎯 Top 20 Pitfalls That Fail Candidates

### Pitfall #1: Dependabot Auto-merge

```yaml
❌ INCORRECT:
"Dependabot automatically merges security PRs."

✅ CORRECT:
Dependabot CREATES PRs but does NOT auto-merge them by default.
You must configure auto-merge explicitly via:
  - GitHub Actions workflows
  - Branch protection auto-merge settings
  - Third-party automation integrations

Why it is a pitfall:
  - People often assume "automated updates" means automated merging.
  - The name "Security Updates" sounds completely automatic.
  - You must pay close attention to exact wording in exam questions.
```

### Pitfall #2: CodeQL on Compiled Languages

```yaml
❌ INCORRECT:
"CodeQL functions without building the codebase in Java."

✅ CORRECT:
Java, C++, C#, Go, and Swift require a COMPLETE compilation build.
JavaScript, TypeScript, Python, and Ruby DO NOT require a build.

Details:
  Compiled:
    - CodeQL intercepts the compiler to build the database.
    - Requires all compilation dependencies to be present.
    - Autobuild can sometimes fail.
    - Custom manual build scripts are common.

  Interpreted:
    - CodeQL directly parses the source files.
    - No build step is required.
    - Database generation is significantly faster.
    - Type inference is more limited.

Why it is a pitfall:
  - It is easy to confuse CodeQL with other SAST tools that do not compile code.
  - Autobuild is present but does not work for non-standard build setups.
  - Workflows fail silently if dependencies are missing during compilation.
```

### Pitfall #3: Secret Scanning Coverage

```yaml
❌ INCORRECT:
"Secret Scanning detects all secrets out-of-the-box."

✅ CORRECT:
It only detects:
  - ~200 predefined high-confidence partner patterns.
  - Custom patterns that YOU define using regex.
  - It does NOT detect secrets in unknown formats.
  - It does NOT detect heavily obfuscated or encrypted secrets.

Examples of WHAT IT DOES NOT DETECT:
  ❌ password = base64_encode("mysecret")
  ❌ API_KEY = "custom-" + generateRandomString()
  ❌ Secrets embedded inside binary files or image assets.
  ❌ Proprietary token formats without a matching pattern.

Why it is a pitfall:
  - The name "Secret Scanning" sounds comprehensive.
  - Candidates assume advanced ML/AI parses all generic secrets.
  - Leads to a false sense of security regarding custom internal tokens.
```

### Pitfall #4: Push Protection Scope

```yaml
❌ INCORRECT:
"Push protection prevents all secrets from existing in the repository."

✅ CORRECT:
Push protection only blocks secrets in incoming, NEW commits.
It does NOT block:
  - Secrets already present in the existing Git history.
  - Secrets pushed to branches before push protection was enabled.
  - Secrets in public or private forks of the repository.
  - Secrets committed prior to activating the feature.

Timeline:
  1. You enable push protection TODAY.
  2. Secrets committed YESTERDAY are not blocked or alerted on by push protection.
  3. Only NEW push events are validated.

Why it is a pitfall:
  - "Protection" sounds absolute.
  - It is not retroactive.
  - It acts as a shield for new additions, while Secret Scanning is the sweep for history.
```

### Pitfall #5: Triage Role

```yaml
❌ INCORRECT:
"The Triage repository role can view code scanning alerts."

✅ CORRECT:
The Triage role CANNOT view:
  - Code Scanning alerts
  - Secret Scanning alerts

The Triage role CAN view:
  - Dependabot alerts (an exception to the rule!)
  - Repository code
  - Issues and pull requests

Visibility Hierarchy:
  Code/Secret Scanning:
    - Admin: ✅
    - Maintain: ❌
    - Write: ❌
    - Triage: ❌
    - Read: ❌

  Dependabot:
    - Admin: ✅
    - Maintain: ✅
    - Write: ✅
    - Triage: ✅
    - Read: ✅

Why it is a pitfall:
  - Architectural inconsistency between different security features.
  - "Triage" sounds like a security operations triage role.
  - Easy to assume it can triage all security warnings.
```

### Pitfall #6: Organization vs. Repository Settings

```yaml
❌ INCORRECT:
"Organization settings always override repository settings."

✅ CORRECT:
It depends on whether ENTERPRISE-wide or ORG-wide policies are ENFORCED:

Without Enterprise Policy:
  Org settings act as defaults.
  Repo admins can override them at will. ✅

With Enterprise/Org Enforcement:
  Policies are locked ("Enforced").
  Organizations cannot change them. ❌
  Repositories cannot change them. ❌

Hierarchy:
  Enterprise (enforced policy)
    ↓ overrides
  Organization (default settings)
    ↓ can be overridden by
  Repository (specific configurations)

Example:
  Enterprise: No enforcement.
  Org: Secret scanning enabled for all repos.
  Repo Admin: Can disable secret scanning for their repository. ✅

  Enterprise: Enforced policy.
  Org: Must enable secret scanning.
  Repo Admin: Option to disable is grayed out/locked. ❌

Why it is a pitfall:
  - The inheritance hierarchy is not always intuitive.
  - Default settings vs. Enforced policies can confuse candidates.
  - Governance policies behave differently under GHEC/GHES.
```

### Pitfall #7: GHAS License Scope

```yaml
❌ INCORRECT:
"A GHAS license covers an unlimited number of repositories."

✅ CORRECT:
GHAS is billed per ACTIVE COMMITTER, not per repository.

Active Committer = anyone who:
  - Has committed code to a repository with GHAS enabled.
  - Within the last 90 days.
  - On any branch of that repository.

Pricing model:
  - Approximately $49/active committer/month (legacy flat fee).
  - Modern structure (GitHub Code Security):
    - Secret Protection: $19/committer/month
    - Code Security: $30/committer/month

Examples:
  100 repos, 10 active committers = $490/month (legacy).
  100 repos, 100 active committers = $4,900/month.

Why it is a pitfall:
  - Candidates assume billing is per repository.
  - "Advanced Security" sounds like flat enterprise pricing.
  - Costs scale directly with development team size.
```

### Pitfall #8: Dependabot Configuration File

```yaml
❌ INCORRECT:
"You need a dependabot.yml file to receive Dependabot alerts."

✅ CORRECT:
A `dependabot.yml` file is NOT required for:
  ✅ Dependabot alerts
  ✅ Dependabot security updates

A `dependabot.yml` file IS required for:
  ✅ Dependabot VERSION updates (scheduled major/minor updates)
  ✅ Customized Cron schedules
  ✅ Package dependency grouping
  ✅ Custom pull request labels, reviewers, and assignees

Without dependabot.yml:
  - Alerts: ✅ Work out-of-the-box
  - Security updates: ✅ Work automatically
  - Version updates: ❌ Do not run

With dependabot.yml:
  - All of the above: ✅
  - Plus: Custom package ecosystem rules, target branches, etc.

Why it is a pitfall:
  - Confusion between the three Dependabot features.
  - Teams write complex YAML files when they only need basic security alerts.
  - Candidates assume Dependabot is completely inactive without a committed config.
```

### Pitfall #9: CodeQL Query Suites

```yaml
❌ INCORRECT:
"The security-extended suite includes all available security queries."

✅ CORRECT:
CodeQL query suite hierarchy:
  security (minimal, low false positives)
    ↓ subset of
  security-extended (broader security coverage)
    ↓ subset of
  security-and-quality (comprehensive security + code quality checks)

Approximate Query Counts:
  security: ~100 queries
  security-extended: ~200 queries
  security-and-quality: ~300 queries

The security-extended suite does NOT include:
  ❌ Code quality and styling queries.
  ❌ Highly experimental community queries.
  ❌ Custom developer queries.

To run everything:
  queries: +security-and-quality

Why it is a pitfall:
  - "Extended" sounds like the absolute superset.
  - Naming conventions lead candidates to assume it covers quality rules.
  - Each suite is tailored to specific execution speed and precision requirements.
```

### Pitfall #10: Dependency Graph Availability

```yaml
❌ INCORRECT:
"The Dependency Graph requires an active GHAS license."

✅ CORRECT:
The Dependency Graph is FREE for:
  ✅ All public repositories
  ✅ All private repositories

It does NOT require a GHAS license.
Available to all GitHub users.

Requires a GHAS License (in private repos):
  - Secret Scanning
  - Code Scanning (CodeQL)
  - Dependency REVIEW (prevents vulnerabilities in PRs)

Always Free:
  - Dependency Graph
  - Dependabot Alerts
  - GitHub Advisory Database

Why it is a pitfall:
  - Easy to conflate with paid dependency analysis tools.
  - "Advanced" features are assumed to require licensing.
  - Dependency Graph is the core backend for Dependabot (which is also free).
```

### Pitfall #11: Secret Validity Check

```yaml
❌ INCORRECT:
"GitHub automatically revokes all detected secrets."

✅ CORRECT:
GitHub:
  ✅ Detects the secret.
  ✅ Verifies token validity (for supported providers).
  ✅ Notifies the repository administrator/developer.
  ✅ Notifies the service provider (for public repos/supported partners).
  ❌ Does NOT automatically revoke the secret.

Partner Actions:
  GitHub Tokens: Automatically revoked by GitHub. ✅
  AWS Keys: AWS may choose to deactivate or restrict access (up to the provider).
  Other Partners: Depends on the specific provider's integration policy.

Your Responsibility:
  ✅ Review the alert.
  ✅ Revoke the secret immediately at the provider.
  ✅ Rotate the credentials.
  ✅ Clean Git history to remove all traces.

Why it is a pitfall:
  - "Notify provider" is easily confused with automatic revocation.
  - GitHub tokens are a special native case that gets auto-revoked, but third-party tokens are not.
  - Candidates assume auto-remediation handles all external providers.
```

### Pitfall #12: CodeQL Language Support

```yaml
❌ INCORRECT:
"CodeQL natively supports PHP and Scala."

✅ CORRECT (2026):
Fully Supported:
  ✅ JavaScript / TypeScript
  ✅ Python
  ✅ Java / Kotlin
  ✅ C / C++
  ✅ C#
  ✅ Go
  ✅ Ruby
  ✅ Swift

Beta / Experimental:
  ⚠️ Rust (beta)

NOT Supported Natively:
  ❌ PHP
  ❌ Scala
  ❌ Perl
  ❌ Lua
  ❌ R

Workarounds:
  - Use a third-party SAST engine.
  - Upload SARIF results from other tools.
  - Install experimental community-supported query packs.

Why it is a pitfall:
  - PHP and Scala are widely used, but not natively supported by CodeQL.
  - "Beta" support is not fully optimized for enterprise production.
  - Candidates assume CodeQL analyzes absolutely any language.
```

### Pitfall #13: SARIF Upload Limit

```yaml
❌ INCORRECT:
"You can upload SARIF files of unlimited size to GitHub."

✅ CORRECT:
SARIF Upload Limits:
  - File Size: Max 10 MB (compressed / gzipped).
  - Results per File: Max 25,000 individual findings.
  - Files per Analysis: Max 20 unique files.

If you exceed these limits:
  ❌ The SARIF ingestion fails with an error.

Workarounds:
  - Split large result sets into multiple SARIF files.
  - Filter out low-severity or informational alerts.
  - Compress the SARIF file using gzip.
  - Ingest results by functional category.

Example:
  CodeQL + Snyk + SonarQube = 3 distinct SARIF uploads.
  Each file must be <10MB.
  Total must be <20 files.

Why it is a pitfall:
  - Ingestion limits are not widely publicized.
  - Very large repositories easily exceed the limit on the first run.
  - The upload process fails silently in CI until check results are inspected.
```

### Pitfall #14: Security Manager Permissions

```yaml
❌ INCORRECT:
"The Security Manager role can modify code in repositories."

✅ CORRECT:
A Security Manager CAN:
  ✅ View security alerts across all organization repositories.
  ✅ Dismiss or reopen security alerts.
  ✅ Configure organization-wide security settings.
  ✅ Access the Security Overview dashboard.
  ✅ Manage enterprise security campaigns.
  ✅ Define custom secret patterns.

A Security Manager CANNOT:
  ❌ View repository source code (unless granted explicit repo access).
  ❌ Modify code or open pull requests.
  ❌ Merge pull requests.
  ❌ Delete repositories.
  ❌ Manage billing or organization subscription plans.
  ❌ Add or remove organization members.

To view AND modify code:
  You need: Security Manager Role + Repo-level Write/Admin permissions.

Why it is a pitfall:
  - The word "Manager" implies broad administrator-level power.
  - It is strictly security-scoped (least privilege).
  - Designed for separation of duties between security officers and developers.
```

### Pitfall #15: Dependabot PR Limits

```yaml
❌ INCORRECT:
"Dependabot will open an unlimited number of pull requests."

✅ CORRECT:
Default limits:
  - Version Updates: Maximum of 5 open PRs.
  - Security Updates: Unlimited (no default cap).

Configurable Option:
  `open-pull-requests-limit: 10`

By Package Manager Ecosystem:
  - npm: limit applies.
  - pip: limit applies.
  - docker: limit applies.

Behavior:
  - If 5 Dependabot Version PRs are currently open.
  - Dependabot will pause generating new version updates.
  - Until you merge, close, or dismiss existing PRs.

Override Configuration:
  # dependabot.yml
  open-pull-requests-limit: 0  # Disable version updates
  open-pull-requests-limit: 20 # Increase open PR allowance

Why it is a pitfall:
  - Candidates expect automated versioning to be unlimited out-of-the-box.
  - Teams wonder why Dependabot PRs suddenly stop appearing.
  - Security updates bypassing this limit represents a critical operational distinction.
```

### Pitfall #16: Branch Protection Bypass

```yaml
❌ INCORRECT:
"Repository administrators can always bypass branch protection rules."

✅ CORRECT:
It depends on the rule configuration:

Branch Protection Rule:
  [✓] Do not allow bypassing the above settings.
    → NO ONE can bypass (including admins and owners).
  
  [ ] Do not allow bypassing the above settings.
    → Admins are allowed to bypass checks.

Repository Settings:
  [ ] Allow repository administrators to bypass.
    → Explicit override toggle.

Enterprise Enforcement:
  If enforced: No bypass allowed under any repository config.
  If not enforced: Standard repository rules apply.

Rulesets (Modern):
  Provides granular bypass permissions:
    - Organization admins.
    - Specialized security teams.
    - Specific named users.
    - Toggle with or without an approval workflow.

Why it is a pitfall:
  - Candidates assume Admin role guarantees god-mode bypasses.
  - The "Do not allow bypassing" toggle is highly restrictive.
  - Modern Rulesets fundamentally change how bypass policies are structured.
```

### Pitfall #17: CodeQL Database Retention

```yaml
❌ INCORRECT:
"CodeQL databases are retained permanently by GitHub."

✅ CORRECT:
Database Retention:
  - GitHub-hosted: 30 days.
  - Action taken after: The database is permanently deleted.
  - SARIF Results: Retained permanently in the Security tab.

Implications:
  - You cannot re-run queries or do deep analysis on a database older than 30 days.
  - You must trigger a new build workflow to regenerate the database.
  - Only the summary analysis results persist.

For Historical Audit Analysis:
  ✅ Download the CodeQL database during execution.
  ✅ Save it in an external secure blob store (AWS S3, Azure Blob, etc.).
  ✅ Re-upload or inspect locally when needed.

Database Upload Action:
  - name: Upload CodeQL Database
    uses: actions/upload-artifact@v4
    with:
      name: codeql-database
      path: codeql-db

Why it is a pitfall:
  - The term "Database" implies a permanent storage medium.
  - The difference between SARIF results and the underlying database is confusing.
  - 30 days passes quickly, disrupting historical audit forensics.
```

### Pitfall #18: Custom Pattern Scope

```yaml
❌ INCORRECT:
"You can define custom Secret Scanning patterns at the repository level."

✅ CORRECT:
Custom Secret Scanning patterns can only be defined at the:
  ✅ Organization level
  ❌ Repository level
  ❌ Enterprise level

Scope Behavior:
  - You create the custom pattern in the Org Settings.
  - It applies automatically to ALL repositories in the organization.
  - You cannot target a single repository.

Implications:
  - Regex patterns must be highly precise to avoid organization-wide false positives.
  - Repository-specific tokens cannot have bespoke local patterns.
  - Demands careful regex testing before publishing.

Workaround:
  - Specify path exclusions inside the custom pattern config.
  - Test the pattern exhaustively on sample repositories before pushing org-wide.

Why it is a pitfall:
  - Code Scanning allows granular repository-level configuration.
  - Candidates expect Secret Scanning to follow a similar localized architecture.
```

### Pitfall #19: Dependency Review License Checks

```yaml
❌ INCORRECT:
"Dependency Review only analyzes package security vulnerabilities."

✅ CORRECT:
Dependency Review analyzes:
  ✅ Security vulnerabilities (CVEs).
  ✅ Software licenses.
  ✅ OpenSSF Scorecard metrics.
  ✅ Package dependency shifts (direct vs transitive).

License Configuration:
  allow-licenses:
    - MIT
    - Apache-2.0
  
  deny-licenses:
    - GPL-3.0
    - AGPL-3.0

Blocks PR if:
  - A developer introduces a package licensed under GPL-3.0.
  - Even if the package is 100% free of known security vulnerabilities.

Use Case:
  - Corporate legal compliance.
  - Preventing copyleft code contamination in proprietary software.
  - Managing vendor supply chain policies.

Why it is a pitfall:
  - The name "Dependency Review" sounds strictly technical.
  - Licenses are legal compliance issues rather than classic security flaws.
  - Easy to overlook configuring license checks until a PR is unexpectedly blocked.
```

### Pitfall #20: Secret Scanning in Forks

```yaml
❌ INCORRECT:
"Push protection applies to all public forks of a repository."

✅ CORRECT:
Public Fork of a Public Repository:
  ✅ Secret Scanning: Enabled (reactive)
  ❌ Push Protection: Disabled (preventative)

Why:
  - Forks are meant for community contributions.
  - Push protection blocking pushes would disrupt the external developer workflow.
  - The parent repository retains its own push protection.

Private Fork:
  ✅ Inherits security settings from the parent repository.
  ✅ Can have push protection active.
  ⚠️ Requires a GHAS license.

Correct Contribution Workflow:
  1. External contributor forks the public repository.
  2. Commits and pushes code to their fork (no push protection block).
  3. Opens a pull request against the parent repository.
  4. Parent repository runs security checks on the incoming PR.
  5. Secret Scanning catches any exposed secrets in the incoming PR.

Why it is a pitfall:
  - Candidates expect absolute push protection across all forks.
  - Open Source Software (OSS) contribution dynamics require different handling.
  - Easy to assume full security inheritance on public forks.
```

---

## 🚨 Red Flags in Answers

### Words that strongly indicate an INCORRECT option:

```yaml
❌ "always" / "always"
  - GHAS features have many conditional dependencies.
  - Absolute behaviors are rare in complex environments.

❌ "never" / "never"
  - Exceptions almost always exist.
  - Rules can be bypassed or overridden.

❌ "automatically" / "automatically" (without technical context)
  - Many features require explicit opt-in or setup.
  - Automated remediation often requires configuration.

❌ "all" / "unlimited"
  - Ingest, retention, and rate limits apply to all tools.
  - Scope boundaries are always defined.

❌ "free" / "free" (without qualification)
  - Public vs. private repository context is crucial.
  - GHAS licenses are required for private repos.

✅ "depends on..."
✅ "if X is configured..."
✅ "by default, but..."
✅ "in public repositories..."
✅ "with an active GHAS license..."
```

---

# REAL-WORLD SCENARIOS <a id="6"></a>

## 🏢 Case 1: Scaling Tech Startup

### Context

```yaml
Company: TechStartup Inc.
Repositories: 50
Developers: 25
Tech Stack: Node.js, Python, React
Current Posture:
  - No GHAS licensed.
  - Dependabot Alerts active (free tier).
  - Ad-hoc, reactive security.
Budget: Limited
Goal: Achieve SOC2 compliance in 6 months.
```

### Exam-Style Question

> **TechStartup needs to comply with SOC2. They have a $50k budget for security tooling. What is the best strategy to implement GHAS?**

**Options:**

A) Purchase GHAS for all repositories immediately.  
B) Start with critical repositories (10), expanding gradually.  
C) Only use free features until the team grows.  
D) Contract a third-party tool instead of GHAS.  

**Correct Answer: B**

### Complete Analysis

```yaml
Why B is correct:

1. COST ANALYSIS:
   25 active committers × $49/committer/month (legacy flat rate) = $14,700/year.
   Fits comfortably within the $50k budget. ✅

2. SOC2 Requirements:
   ✅ Vulnerability management.
   ✅ Secure SDLC enforcement.
   ✅ Code review processes.
   ✅ Access controls.
   GHAS natively fulfills all these audits.

3. Phased Rollout (B):
   
   Months 1-2: Phase 1 (10 Critical Repositories)
     - Payment processing engine.
     - User authentication service.
     - API gateway.
     - Customer database interface.
      
     Enable:
       ✅ Secret scanning + push protection.
       ✅ CodeQL (default setup).
       ✅ Dependabot security updates.
       ✅ Dependency review in PRs.
      
     Cost: $14.7k/year (covers full team committer count).
     Risk Coverage: ~70% of organizational threat vectors.
   
   Months 3-4: Phase 2 (20 Repositories)
     - Internal tools.
     - Administrative panels.
     - Monitoring services.
      
     Same features enabled.
     Cost: Already covered (billed per committer).
     Risk Coverage: ~90% of risk vectors.
   
   Months 5-6: Phase 3 (20 Remaining Repositories)
     - Documentation repositories.
     - Support utilities.
     - Experimental scripts.
      
     Cost: Already covered.
     Risk Coverage: 100% repository coverage.

4. SOC2 Audit Readiness:
   Month 5:
     ✅ Collect Security Overview metrics.
     ✅ Export compliance dashboards.
     ✅ Document active policies.
     ✅ Extract alert resolution logs.
     ✅ Archive developer training logs.
    
   Month 6:
     ✅ Audit-ready and certified.

Why A is incorrect:
  - Overwhelms the engineering team with instant backlog alerts.
  - Leads to severe alert fatigue.
  - Demands high upfront learning time.
  - Doesn't allow developers to adapt to the security workflow.

Why C is incorrect:
  - Free features are insufficient for private repos under SOC2.
  - No Code Scanning or Secret Scanning available for private code without licensing.
  - Compliance auditors will reject the security controls.

Why D is incorrect:
  - Third-party solutions are typically more expensive to license and integrate.
  - Lacks native developer workflow integration.
  - GHAS provides a superior ROI for GitHub-native organizations.
```

---

## 🏢 Case 2: Healthcare Enterprise

### Context

```yaml
Company: HealthCorp
Repositories: 500+
Developers: 300
Compliance: HIPAA, SOC2, ISO27001
Current Posture: GHAS licensed enterprise-wide.
Issue: 2,500 open security alerts.
Goal: Reduce open alerts to <100 in Q1.
```

### Exam-Style Question

> **HealthCorp has 2,500 open GHAS alerts. Management demands reducing them to <100 by the end of the quarter. What is the MOST EFFECTIVE strategy?**

**Options:**

A) Dismiss all medium and low severity alerts.  
B) Define a coordinated Security Campaign prioritizing by impact.  
C) Hire 50 temporary developers to fix everything.  
D) Disable GHAS until the team has bandwidth.  

**Correct Answer: B**

### Detailed Solution

```yaml
Step 1: Triage and Classification

Analyze the 2,500 open alerts:
  Critical: 45
  High: 234
  Medium: 987
  Low: 1,234

By Category:
  Secret Scanning: 123 (5%)
    - Active: 12 🔴
    - Inactive: 111
  
  Code Scanning: 1,456 (58%)
    - SQL Injection: 23
    - XSS: 45
    - Path Traversal: 12
    - Others: 1,376
  
  Dependabot: 921 (37%)
    - Critical: 15
    - High: 156
    - Medium: 750

By Repository:
  Top 10 Repos: 1,200 alerts (48%)
  Next 40 Repos: 800 alerts (32%)
  Remaining 450 Repos: 500 alerts (20%)

Step 2: Security Campaigns

Campaign 1: "Active Secrets" (Week 1)
  Target: 12 active secrets.
  Priority: P0 (Critical).
  Action: Immediate revocation.
  Owner: Security Team.
  Deadline: 3 days.
  
  Result: 12 fixed ✅

Campaign 2: "Critical Vulns in Production" (Weeks 1-2)
  Target: 45 critical alerts in production-facing applications.
  Priority: P0.
  Action: Mitigate or patch immediately.
  Owner: Respective engineering teams.
  Deadline: 7 days.
  
  Result: 40 patched, 5 mitigated with compensative controls. ✅

Campaign 3: "High Severity Dependabot" (Weeks 2-4)
  Target: 156 high-severity dependencies.
  Priority: P1.
  Action: Merge automated Dependabot PRs.
  Owner: Feature teams.
  Deadline: 30 days.
  
  Automation:
    - Auto-approve minor/patch updates.
    - Execute CI/CD verification.
    - Perform batch merges.
  
  Result: 140 resolved. ✅

Campaign 4: "SQL Injection Elimination" (Weeks 3-6)
  Target: 23 SQL injection vulnerabilities.
  Priority: P1.
  Action: Refactor database access to prepared statements.
  Owner: Backend engineering team.
  Deadline: 45 days.
  
  Result: 23 resolved. ✅

Campaign 5: "XSS Cleanup" (Weeks 4-8)
  Target: 45 cross-site scripting vulnerabilities.
  Priority: P1.
  Action: Implement context-aware output encoding.
  Owner: Frontend engineering team.
  Deadline: 60 days.
  
  Result: 45 resolved. ✅

Campaign 6: "Medium Severity Triage" (Weeks 5-12)
  Target: 987 medium alerts.
  Priority: P2.
  Action: Investigate, patch, or formally dismiss.
  Owner: Broad engineering org.
  Deadline: 90 days.
  
  Strategy:
    - Bulk dismiss verified false positives (with documented audits).
    - Remediate genuine vulnerabilities.
    - Record formal risk acceptance with security sign-off.
  
  Result:
    - Remediated: 345
    - Dismissed (valid reasons): 542
    - Documented risk acceptance: 100

Step 3: Process Improvements

Prevention:
  ✅ Enforce mandatory Dependency Review on PRs.
  ✅ Activate Push Protection organization-wide.
  ✅ Require CodeQL status checks to pass before merge.
  ✅ Mandatory secure coding workshops for developers.
  ✅ Establish weekly Security Champions check-ins.

Automation:
  ✅ Setup Dependabot auto-merge for patch revisions.
  ✅ Define auto-triage rules for low-severity dependencies.
  ✅ Route critical security alerts to Slack channels.
  ✅ Deliver weekly metrics digests to engineering leads.

Step 4: Results

End of Q1:
  Starting Backlog: 2,500 alerts
  Remediated: 565
  Dismissed (valid): 653
  Risk Accepted: 105
  
  Remaining Open: 1,177
  
  Focus Areas:
    Critical: 0 (100% reduction) ✅
    High: 50 (78% reduction) ✅
    Medium: 400 (59% reduction) ✅
    Low: 727 (41% reduction)

Recalibrated Goal Analysis:
  Reducing all alerts to <100 was operationally unrealistic.
  New compliant metrics:
    - 0 open critical alerts. ✅
    - <50 open high alerts. ✅
    - Highly structured, managed backlog. ✅

Compliance Status:
  ✅ HIPAA: No active critical vulnerabilities in PHI-handling systems.
  ✅ SOC2: Under 7-day MTTR SLA for critical vulnerabilities.
  ✅ ISO27001: Security trends and metrics formally audited.

Why other options are incorrect:

A) Dismiss all medium and low severity alerts:
  - Severe compliance violation.
  - Ignores root cause analysis.
  - Many medium alerts are dangerous when chained together.

C) Hire 50 temporary developers:
  - Cost-prohibitive ($10M+/year run rate).
  - High onboarding friction.
  - Not a sustainable development process.

D) Disable GHAS:
  - Immediate compliance breach.
  - Complete loss of supply chain and code visibility.
  - High corporate liability.
```

---

## 🏢 Case 3: Fintech with Strict Compliance

### Context

```yaml
Company: PaymentPro
Industry: Fintech (payment processor)
Repositories: 75 (all private)
Compliance: PCI-DSS Level 1, SOC2 Type II
Issue: External auditor discovered plaintext credentials in active Git history.
Severity: Critical audit finding.
Remediation Window: 30 days.
```

### Exam-Style Question

> **PaymentPro discovers active AWS keys in the Git history of 15 private repositories. The auditor gives them 30 days to resolve the finding. What is the correct sequence of actions?**

**Options:**

A) 1. Enable push protection → 2. Wait for it to block future commits.  
B) 1. Revoke the keys → 2. Clean Git history → 3. Enable push protection → 4. Audit.  
C) 1. Clean history → 2. Notify customers → 3. Enable scanning.  
D) 1. Create new repos → 2. Migrate current code → 3. Delete old repos.  

**Correct Answer: B**

### Complete Remediation Plan

```yaml
PHASE 1: IMMEDIATE RESPONSE (Day 1 - first 2 hours)

Hour 1: Assess Damage
  
  Tasks:
    1. Run Secret Scanning across the 15 compromised repositories:
       gh api /repos/:owner/:repo/secret-scanning/alerts
     
    2. Catalog all exposed credentials:
       - AWS Access Keys: 23 instances.
       - AWS Secret Keys: 23 instances.
       - Database passwords (RDS): 5 instances.
       - Vendor API keys: 12 instances.
       
       Total: 63 exposed credentials.
     
    3. Verify token validity:
       aws sts get-caller-identity --profile compromised-profile
       
       Active: 18 (CRITICAL RISK) ✅
       Inactive: 45
     
    4. Query CloudTrail logs for unauthorized activity:
       aws cloudtrail lookup-events \
         --lookup-attributes AttributeKey=Username,AttributeValue=compromised-key \
         --start-time 2024-01-01
       
       Result: No unauthorized API usage detected. ✅
       Exposure window remains a threat.

Hour 2: Revoke and Rotate

  Tasks:
    1. Revoke all credentials (both active and inactive):
       aws iam delete-access-key \
         --access-key-id AKIAIOSFODNN7EXAMPLE \
         --user-name app-user
       
       Execute bulk revocation scripts.
     
    2. Rotate production secrets:
       - Generate fresh IAM credentials.
       - Inject secrets securely into HashiCorp Vault / AWS Secrets Manager.
       - Trigger application redeployment.
     
    3. Run smoke tests:
       - Ensure all applications function with rotated credentials.
       - Zero downtime.
     
    4. Record the incident timeline:
       - Document exposure duration.
       - Action steps taken.
       - CloudTrail verification of no compromise.

PHASE 2: CLEANSE GIT HISTORY (Days 1-3)

Tool Choice: git-filter-repo (superior speed and safety over BFG)

Step 1: Locate targeted commits
  
  git log --all --full-history -- '**/config.js' | grep -i "aws"
  
  Identify:
    - Target commit hashes containing credentials.
    - Specific file paths.
    - Impacted branches.

Step 2: Create a secure backup

  # Generate absolute mirror clone
  git clone --mirror https://github.com/org/repo.git repo-backup
  
  # Archive and store in secure offline storage
  tar -czf repo-backup.tar.gz repo-backup/
  aws s3 cp repo-backup.tar.gz s3://secure-backups/

Step 3: Run git-filter-repo

  # Install utility
  pip install git-filter-repo
  
  # Define replacement mapping
  cat > ../secrets-replacement.txt <<EOF
  AKIAIOSFODNN7EXAMPLE==>REDACTED_AWS_KEY
  wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY==>REDACTED_SECRET
  EOF
  
  # Execute filtering operation
  git filter-repo --replace-text ../secrets-replacement.txt
  
  # Verify local history is clean
  git log --all --full-history -S 'AKIAIOSFODNN7EXAMPLE'
  # Returns 0 results

Step 4: Force push clean history

  # Warning: Destructive history rewrite!
  git push --force --all
  git push --force --tags
  
  # Alert the engineering organization via Slack:
  "🚨 CRITICAL: Force push performed on payment-api.
   Reason: Plaintext credential purge.
   Required Action: All developers must delete local folders and perform a fresh git clone.
   Do NOT attempt to pull or merge old local histories."

Step 5: Verify remote repository

  # Perform fresh clone
  git clone https://github.com/org/repo.git verify-repo
  cd verify-repo
  
  # Audit logs for exposed prefixes
  git log --all --full-history -S 'AKIA'
  git log --all --full-history -S 'aws_secret'
  # Returns 0 results ✅

Step 6: Address developer forks

  # Query all active forks
  gh api /repos/:owner/:repo/forks
  
  # Request fork owners fetch upstream/rebase or delete forks.

PHASE 3: ESTABLISH SYSTEMIC PROTECTION (Days 3-5)

Step 1: Organization-wide enablement

  Organization Settings
    → Code security
    → Secret scanning
      [✓] Enable for all repositories
      [✓] Enable push protection for all repositories
      [✓] Automatically enable for new repositories

Step 2: Define custom patterns (Payment ecosystem specific)

  # Stripe production keys
  Pattern: sk_live_[0-9a-zA-Z]{24,99}
  
  # Internal PaymentPro tokens
  Pattern: PAYMENTPRO-[A-Z0-9]{32}
  
  # DB connection URL structure
  Pattern: postgresql://[^:]+:[^@]+@[^/]+/payment_db

Step 3: Push protection bypass restrictions

  Settings:
    [✓] Allow bypasses: NO
    [✓] Require justification: YES (if bypasses are toggled back on)
  
  For strict PCI compliance: Bypasses are fully disabled.

Step 4: Branch protection enforcement script

  Bulk script execution:
  
  repos=$(gh api /orgs/paymentpro/repos --jq '.[].name')
  
  for repo in $repos; do
    gh api -X PUT /repos/paymentpro/$repo/branches/main/protection \
      -f required_status_checks[strict]=true \
      -f required_status_checks[contexts][]="secret-scanning" \
      -f enforce_admins=true \
      -f restrictions=null
  done

PHASE 4: AUDIT AND REPORTING (Days 5-7)

Step 1: Collect compliance evidence

  1. Visual verification screenshots:
     - Secret Scanning active. ✅
     - Push Protection active. ✅
     - All backlog alerts resolved. ✅
  
  2. Technical reports:
     - CSV export of active Security Overview status.
     - Coordinated incident timeline from discovery to resolution.
     - Post-filtering Git history verification logs.
  
  3. Security policies:
     - Updated organizational Secret Management Policy.
     - Documented Incident Response Playbook for credential leaks.
     - Developer secure coding workshop materials.

Step 2: Independent verification

  1. External security consulting firm:
     - Audits Git history for lingering exposures.
     - Performs penetration test on compromised assets.
  
  2. AWS IAM audit:
     - Reviews active IAM policies and restricts least privilege.
     - Validates CloudTrail audit log integrity.

Step 3: Map evidence to compliance frameworks

  PCI-DSS Requirements:
    ✅ 6.3.2: Review of custom code changes.
    ✅ 6.5.3: Preventing insecure storage of cryptographic keys.
    ✅ 12.10.1: Documented incident response program.
  
  SOC2 Trust Services Criteria:
    ✅ CC6.1: Restricting logical access credentials.
    ✅ CC7.2: Continuous system security monitoring.
    ✅ CC9.2: Implementing risk mitigation strategies.

Step 4: Formal submission to auditors

  Artifact Package:
    1. Executive Summary.
    2. Detailed discovery-to-resolution timeline.
    3. Technical proof-of-resolution logs.
    4. Code and settings configuration screens.
    5. Newly adopted policy documents.
    6. Developer training attendance logs.
  
  Audit Result: Finding successfully closed. ✅

PHASE 5: ONGOING PREVENTION (Days 7-30)

Step 1: Interactive developer training

  Week 1:
    - Workshop: Secure Secret Management.
    - Tutorial: Native AWS Secrets Manager integration.
    - Live demo: Push protection blocking secret leaks.
  
  Week 2:
    - Updated peer code review guidelines.
    - Open Q&A: Handling security feedback in local workflows.

Step 2: Workflow process updates

  Mandatory controls:
    ✅ Store all active credentials in Vault/Secrets Manager.
    ✅ No plaintext credentials allowed in codebase.
    ✅ Local pre-commit hooks installed (git-secrets/trufflehog).
    ✅ Automate quarterly credential rotation.
    ✅ Monthly logical access privilege audit.

Step 3: Continuous monitoring setup

  Configured tooling:
     - Real-time Security Overview alerts.
     - Weekly leadership security metrics digest.
     - Slack bot channels for instant alerts.
     - PagerDuty routing for active credential alerts.
  
  SLAs:
    - Active secret leak: Revoke within 1 hour.
    - Inactive secret history finding: Remediate within 24 hours.

Step 4: Continuous improvement

  Monthly tasks:
    - Inspect and audit dismissed alerts.
    - Update custom regex scanning patterns.
    - Engineering post-mortem reviews.
  
  Quarterly tasks:
    - External application penetration testing.
    - Regulatory audit alignment.
    - Update secure coding policies.

RESULTS:

Day 30 Compliance Posture:
  ✅ 100% of exposed credentials revoked.
  ✅ Git history cleaned across all affected repositories.
  ✅ Push Protection fully active across organization.
  ✅ 0 open active alerts.
  ✅ Bespoke scanning patterns active.
  ✅ 100% engineering team trained.
  ✅ Auditor finding successfully closed.
  ✅ Maintaining PCI-DSS Level 1 compliance.

Cost Breakdown:
  - GHAS Licensing: Already active ($0 incremental).
  - External Consulting Audit: $15,000.
  - Training materials: $5,000.
  - Utilities and scripts: $0.
  Total Cost: $20,000
  
  vs.
  
  Estimated breach exposure: $5,000,000+.
  Estimated ROI: 250x.

Key Takeaways:
  - Proactive push prevention is far superior to reactive Git cleansing.
  - Once pushed, Git history is highly difficult to clean correctly at scale.
  - Developers need local tooling and training to prevent leaks.
```

---

[Continued with the 100 Practice Questions in the next section...]

---
# 100 PRACTICE QUESTIONS <a id="7"></a>

## Real Exam Format

```yaml
Question Types:
  - Multiple Choice (1 correct answer)
  - Multiple Select (2+ correct answers)
  - Scenario-based
  - True/False

Domain Distribution:
  - Domain 1 (GHAS Features): ~10 questions
  - Domain 2 (Secret Scanning): ~10 questions
  - Domain 3 (Dependabot): ~25 questions
  - Domain 4 (CodeQL): ~18 questions
  - Domain 5 (Best Practices): ~7 questions

Total Questions: ~70 questions
Time Limit: 120 minutes
Passing Score: 70%
```

---

## DOMAIN 1: Security Features and Capabilities of GHAS

### Question 1

**An organization wants centralized visibility into the security posture of all its repositories. Which GHAS feature provides this?**

A) Dependabot alerts  
B) Security Overview  
C) Code scanning dashboard  
D) Secret scanning alerts  

<details>
<summary>View Answer</summary>

**Answer: B) Security Overview**

**Explanation:**
- Security Overview is a centralized dashboard at the organization and enterprise levels.
- It provides high-level security metrics, historical trends, and cross-repository filtering options.
- The other options are feature-specific views and do not provide an organizational overview.

**Why the others are incorrect:**
- A) Dependabot alerts: Only displays software supply chain vulnerabilities.
- C) Code scanning dashboard: Focused solely on static code analysis findings.
- D) Secret scanning alerts: Restricts view to hardcoded secrets.

**Resources:**
- https://docs.github.com/en/code-security/security-overview/about-security-overview
</details>

---

### Question 2

**Which GHAS features are available for FREE in public repositories? (Select all that apply)**

A) Secret scanning  
B) Push protection  
C) CodeQL code scanning  
D) Dependabot alerts  
E) Security Overview  

<details>
<summary>View Answer</summary>

**Answers: A, B, C, D**

**Explanation:**
- Public repositories on GitHub have almost all Advanced Security features available for free to promote open source security.
- Security Overview is the only feature listed that is reserved exclusively for organizations and enterprises with active GHAS licensing.

**Details:**
- ✅ Secret scanning: Free and auto-enabled by default.
- ✅ Push protection: Free for all public repositories.
- ✅ CodeQL: Free to use, but requires repository workflow configuration.
- ✅ Dependabot alerts: Free and automatically enabled.
- ❌ Security Overview: Only available for private repositories under a paid GHAS plan.

**Resources:**
- https://docs.github.com/en/get-started/learning-about-github/about-github-advanced-security
</details>

---

### Question 3

**Your company uses GitHub Enterprise Cloud. The CISO wants to ENFORCE secret scanning across all repositories in all organizations, with absolutely no exceptions. Where should you configure this?**

A) Organization settings → Code security  
B) Repository settings → Security  
C) Enterprise policies → Code security  
D) Open a GitHub Support ticket  

<details>
<summary>View Answer</summary>

**Answer: C) Enterprise policies → Code security**

**Explanation:**
- Enterprise policies permit the strict ENFORCEMENT of security rules across the entire enterprise.
- The requirement "with no exceptions" means you must set an enforced policy so that organization and repository admins cannot toggle the feature off.
- Organization and repository settings can be overridden unless locked by an enterprise-level policy.

**Hierarchy:**
```
Enterprise Policy (Enforced) ← CONFIGURE HERE
  ↓ cannot be disabled by
Organization
  ↓ cannot be disabled by
Repository
```

**Resources:**
- https://docs.github.com/en/enterprise-cloud@latest/admin/policies/enforcing-policies-for-your-enterprise/enforcing-policies-for-code-security-and-analysis-for-your-enterprise
</details>

---

### Question 4

**What is the primary operational difference between Dependabot alerts and Dependency Review?**

A) Dependabot alerts are free, whereas Dependency Review always requires GHAS.  
B) Dependabot alerts detect existing vulnerabilities, while Dependency Review prevents new ones from being merged.  
C) Dependabot alerts support npm, whereas Dependency Review supports all package manager ecosystems.  
D) There is no difference; they are just different names for the same underlying security scan.  

<details>
<summary>View Answer</summary>

**Answer: B) Dependabot alerts detect existing vulnerabilities, while Dependency Review prevents new ones from being merged**

**Explanation:**
- Dependabot alerts: REACTIVE protection (scans after code is merged to main).
- Dependency Review: PROACTIVE protection (scans package changes in a PR before merge).

**Comparison:**
| Feature | Dependabot Alerts | Dependency Review |
|---------|-------------------|-------------------|
| **Timing** | Post-merge (continuous) | Pre-merge (during PR) |
| **Primary Action**| Creates security alert in UI | Fails PR status check / blocks merge |
| **Location** | Security tab | PR Checks tab |

**Why A is incorrect:**
- Dependabot alerts are indeed free for all repositories, but that is a pricing detail rather than the primary *operational* or *functional* difference between the two tools.

**Resources:**
- https://docs.github.com/en/code-security/supply-chain-security/understanding-your-software-supply-chain/about-dependency-review
</details>

---

### Question 5

**A developer on your team reports that CodeQL takes 45 minutes to analyze a Java repository, but only 5 minutes to analyze a JavaScript repository of similar size. What is the MOST likely reason?**

A) Java code inherently contains more security vulnerabilities than JavaScript.  
B) Java is a compiled language requiring a full compilation build, whereas JavaScript does not.  
C) CodeQL has significantly fewer queries to execute for JavaScript.  
D) The Java repository contains larger files.  

<details>
<summary>View Answer</summary>

**Answer: B) Java is a compiled language requiring a full compilation build, whereas JavaScript does not**

**Explanation:**
- Java is a compiled language. To analyze it, CodeQL must intercept the compiler and build the entire codebase to construct the relational database.
- JavaScript is an interpreted language. CodeQL directly parses the source files, bypassing any build phase.

**Process Execution Flow:**
```
Java Analysis:
  1. Initialize CodeQL
  2. BUILD CODEBASE (← Most time spent here)
  3. Analyze Database
  4. Ingest Results

JavaScript Analysis:
  1. Initialize CodeQL
  2. Analyze Source directly (No build)
  3. Ingest Results
```

**Why the others are incorrect:**
- A) The number of vulnerabilities present in a codebase does not influence static analysis runtimes.
- C) JavaScript and Java have highly comparable query suites.
- D) File size can affect execution speed, but the compilation phase is the primary performance differentiator.

**Resources:**
- https://codeql.github.com/docs/codeql-overview/supported-languages-and-frameworks/
</details>

---

## DOMAIN 2: Secret Scanning

### Question 6

**A developer has just pushed an active AWS access key to a public repository. What happens automatically? (Select all that apply)**

A) GitHub detects the exposed secret.  
B) GitHub notifies the developer.  
C) GitHub revokes the compromised AWS key.  
D) AWS is notified of the leak by GitHub.  
E) The commit containing the secret is automatically reverted in Git.  

<details>
<summary>View Answer</summary>

**Answers: A, B, D**

**Explanation:**

**What DOES happen:**
- ✅ Secret Scanning detects the secret within seconds of the push (A).
- ✅ An email and UI notification are sent to the repository administrators and the developer (B).
- ✅ AWS (the partner service provider) is notified of the exposure (D).

**What DOES NOT happen:**
- ❌ GitHub does NOT revoke the secret (C). Revocation is the sole responsibility of AWS or the repository owner.
- ❌ The commit is NOT reverted (E). Git history remains completely unchanged.

**Remediation Sequence:**
```
1. Push containing AWS key is received by GitHub.
2. Secret scanning scans the push event (typically takes <30 seconds).
3. Exposed credentials alert is generated in the Security tab.
4. Notification email sent to repo admins.
5. Secure partner notification sent to AWS.
6. AWS can take automatic protective measures (deactivating key/emailing account owner).
```

**Resources:**
- https://docs.github.com/en/code-security/secret-scanning/introduction/about-secret-scanning
</details>

---

### Question 7

**Push protection is enabled on your repository. A developer attempts to push a commit containing a GitHub Personal Access Token. What actions can the developer take? (Select all that apply)**

A) Remove the secret from the source code and push the corrected commit.  
B) Request a bypass with a justification (if allowed by organization policy).  
C) Perform a force push to overwrite the push protection check.  
D) Push the commit to a different branch to bypass the check.  

<details>
<summary>View Answer</summary>

**Answers: A, B**

**Explanation:**

**Valid Actions:**
- ✅ A) Remove the secret and push: This is the correct, secure remediation path.
- ✅ B) Request bypass: Allowed if the organization policy toggle `allow_bypass` is enabled.

**Invalid Actions:**
- ❌ C) Force pushing will NOT bypass push protection. All push actions are audited.
- ❌ D) Pushing to different branches will NOT bypass the check; push protection scans all branches.

**Push Protection Scope:**
- ✅ All branches.
- ✅ All commits.
- ✅ Force push events.
- ✅ Git tag creations.

**Bypass Logic Workflow:**
```
IF allow_bypass = true:
  Developer can:
    1. Select "Bypass protection" in terminal/UI.
    2. Provide a standard justification (e.g. false positive, test secret).
    3. Complete the push.
    4. An alert is created in the Security tab with the bypass reason logged.

IF allow_bypass = false:
  Developer must:
    1. Remove the secret from the codebase.
    2. Rewrite the local commit history.
    3. Attempt the push again.
```

**Resources:**
- https://docs.github.com/en/code-security/secret-scanning/introduction/about-push-protection
</details>

---

### Question 8

**Your organization wants to detect custom internal secrets in the format "ACME-API-[32 alphanumeric characters]". Where should you configure this custom pattern?**

A) Repository settings → Secret scanning  
B) Organization settings → Code security → Custom patterns  
C) Enterprise settings → Policies  
D) Commit a `.github/secret-scanning.yml` file to the default branch  

<details>
<summary>View Answer</summary>

**Answer: B) Organization settings → Code security → Custom patterns**

**Explanation:**
- Custom Secret Scanning patterns can only be defined at the **Organization** level.
- They cannot be configured at the individual repository level or at the enterprise policy level.

**Pattern Regex Example:**
```regex
ACME-API-[A-Z0-9]{32}
```

**Scope Impact:**
- Defined once in Organization Settings.
- Evaluates code across ALL repositories in the organization.
- Repository-specific patterns are not supported.

**Resources:**
- https://docs.github.com/en/code-security/secret-scanning/defining-custom-patterns-for-secret-scanning
</details>

---

### Question 9

**A Secret Scanning alert displays "Validity: Active" for a leaked GitHub token. What does this mean?**

A) The secret was detected within the last hour.  
B) The token was successfully tested and is currently active and functional.  
C) The token is located in actively executed application paths.  
D) The secret is present on the active branch (main).  

<details>
<summary>View Answer</summary>

**Answer: B) The token was successfully tested and is currently active and functional**

**Explanation:**

**Validity Check Statuses:**
- **Active**: GitHub validated the token against its own API and confirmed it is currently operational.
- **Inactive**: The token was validated and is confirmed revoked or expired.
- **Unknown**: GitHub cannot programmatically verify the validity status of this particular token type.

**Validity Verification Workflow:**
```
1. Secret token is detected by Secret Scanning.
2. GitHub performs an out-of-band validation check against the token issuer's API.
3. API evaluates token:
   - Success → Status: Active 🚨 (CRITICAL RISK)
   - Unauthorized → Status: Inactive (Low Risk)
   - Network timeout/Unsupported provider → Status: Unknown
```

**Immediate Remediation for "Active" Secrets:**
- 🚨 Revoke the token immediately.
- 🚨 Generate a new token.
- 🚨 Audit token usage logs to verify no unauthorized access occurred.
- 🚨 Clear Git history.

**Resources:**
- https://docs.github.com/en/code-security/secret-scanning/managing-alerts-from-secret-scanning/evaluating-alerts
</details>

---

### Question 10

**What is the primary difference between Secret Scanning and Push Protection?**

A) Secret scanning is free, whereas push protection requires a paid GHAS subscription.  
B) Secret scanning operates reactively (after push), while push protection operates preventatively (blocking pushes).  
C) Secret scanning is for public repositories, while push protection is for private ones.  
D) Secret scanning relies on machine learning, while push protection relies on regular expressions.  

<details>
<summary>View Answer</summary>

**Answer: B) Secret scanning operates reactively (after push), while push protection operates preventatively (blocking pushes)**

**Explanation:**

**Timeline Comparison:**
```
WITHOUT Push Protection:
  Code → Commit → Push → Push Accepted → (30s) → Secret Scanning Alert
  ⚠️ Secret is ALREADY exposed in repository history.

WITH Push Protection:
  Code → Commit → Push → Push BLOCKED by GitHub Pre-receive Hook
  ✅ Secret NEVER reaches GitHub servers.
```

**Why the others are incorrect:**
- A) Both features require a paid GHAS license for private repositories (and both are free for public repositories).
- C) Both tools are available for public and private repositories alike.
- D) Both tools utilize identical regular expression patterns to match credentials.

**Resources:**
- https://docs.github.com/en/code-security/secret-scanning/introduction/about-push-protection
</details>

---

## DOMAIN 3: Dependabot and Dependency Review

### Question 11

**What configuration file is REQUERIDO to enable Dependabot alerts in a private repository?**

A) `dependabot.yml`  
B) `.github/dependabot-config.json`  
C) None (Dependabot alerts are enabled automatically or via UI settings)  
D) `security.yml`  

<details>
<summary>View Answer</summary>

**Answer: C) None (Dependabot alerts are enabled automatically or via UI settings)**

**Explanation:**

**Dependabot Alerts:**
- ✅ Do NOT require any committed YAML configuration file.
- ✅ Enabled automatically or with a single click in the UI.
- ✅ Completely free for both public and private repositories.

**A `dependabot.yml` file is only needed for:**
- Dependabot Version Updates (scheduled updates).
- Customized Cron schedules.
- Grouping dependencies.
- Defining custom assignees and reviewers.

**Common Misconception:**
```yaml
❌ INCORRECT: "We must write a dependabot.yml to get security warnings."

✅ CORRECT:
  - Dependabot Alerts: Zero configuration required.
  - Dependabot Security Updates (PRs): Zero configuration required.
  - Dependabot Version Updates: Requires a dependabot.yml file.
```

**Resources:**
- https://docs.github.com/en/code-security/dependabot/dependabot-alerts/about-dependabot-alerts
</details>

---

### Question 12

**A pull request introduces `lodash@4.17.10` (which has a known vulnerability). The repository's Dependency Review action is configured with `fail-on-severity: high`. What happens?**

A) The PR status check passes and can be merged normally.  
B) The PR status check fails, blocking the merge (if branch protection is active).  
C) A security alert is generated in the Security tab, but the PR check passes.  
D) Dependabot immediately generates a separate security update PR.  

<details>
<summary>View Answer</summary>

**Answer: B) The PR status check fails, blocking the merge (if branch protection is active)**

**Explanation:**

**Lodash@4.17.10 vulnerability profile:**
- CVE-2020-8203 (Prototype Pollution).
- Severity: High.

**Dependency Review Execution Flow:**
```
1. Developer opens PR changing lodash to 4.17.10.
2. Dependency Review Action runs on pull_request event.
3. Detects high-severity CVE-2020-8203.
4. Rule check: fail-on-severity: high matches.
5. Action exits with failure status: ❌
6. PR Merge button is grayed out / BLOCKED (enforced by branch protection).
```

**Developer remediation steps:**
- Update lodash to a secure patched version (e.g. `4.17.21+`).
- Commit and push the fix.
- Dependency Review re-runs and passes: ✅
- Pull request is unblocked.

**Resources:**
- https://github.com/actions/dependency-review-action
</details>

---

### Question 13

**Your team wants to group all weekly React-related package updates into a single Dependabot pull request. How can you achieve this?**

A) Configure a custom GitHub Actions workflow.  
B) Define a package group in `dependabot.yml`.  
C) Select grouping rules in Organization settings → Dependabot.  
D) It is not possible; Dependabot always creates individual PRs for each package.  

<details>
<summary>View Answer</summary>

**Answer: B) Define a package group in dependabot.yml**

**Explanation:**

**Example Configuration:**
```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
    
    groups:
      react-ecosystem:
        patterns:
          - "react"
          - "react-dom"
          - "@types/react"
          - "@types/react-dom"
        update-types:
          - "minor"
          - "patch"
```

**Before vs. After Grouping:**
```
Without Grouping:
  PR #1: Bump react from 18.2.0 to 18.3.0
  PR #2: Bump react-dom from 18.2.0 to 18.3.0
  PR #3: Bump @types/react from 18.0.0 to 18.0.1
  PR #4: Bump @types/react-dom from 18.0.0 to 18.0.1

With Grouping:
  PR #1: Bump react-ecosystem group (contains all 4 package updates)
```

**Resources:**
- https://docs.github.com/en/code-security/dependabot/dependabot-version-updates/grouping-dependabot-updates
</details>

---

### Question 14

**What is the minimum repository permission role required for a user to VIEW Dependabot alerts in a private repository?**

A) Admin  
B) Write  
C) Read  
D) Security Manager  

<details>
<summary>View Answer</summary>

**Answer: C) Read**

**Explanation:**

**Dependabot Alerts Permission Matrix:**
| Role | View Alerts | Dismiss/Reopen Alerts |
|------|-------------|-----------------------|
| **Read** | ✅ | ❌ |
| **Triage** | ✅ | ❌ |
| **Write** | ✅ | ✅ |
| **Maintain** | ✅ | ✅ |
| **Admin** | ✅ | ✅ |

**Critical Exam Concept:**
- Dependabot alerts are a notable **exception to the rule**; they are visible to anyone with **Read** access.
- Code Scanning and Secret Scanning alerts in private repositories require **Admin** permissions to view by default.

**Resources:**
- https://docs.github.com/en/code-security/dependabot/dependabot-alerts/about-dependabot-alerts#access-to-dependabot-alerts
</details>

---

### Question 15

**You have a monorepo containing a frontend application (npm) and a backend service (maven). How many update blocks must you declare in your `dependabot.yml`?**

A) 1 block (Dependabot auto-detects both ecosystems in the repository).  
B) 2 blocks (one for each package ecosystem and its directory path).  
C) You do not need a `dependabot.yml` file to run Dependabot alerts.  
D) 3 blocks (npm, yarn, and maven).  

<details>
<summary>View Answer</summary>

**Answer: B) 2 blocks (one for each package ecosystem and its directory path)**

**Explanation:**
- To run Dependabot Version Updates, you must explicitly declare each package manager ecosystem and its target folder structure separately.

**Example Monorepo Configuration:**
```yaml
# .github/dependabot.yml
version: 2
updates:
  # Frontend (npm)
  - package-ecosystem: "npm"
    directory: "/frontend"
    schedule:
      interval: "weekly"
    
  # Backend (maven)
  - package-ecosystem: "maven"
    directory: "/backend"
    schedule:
      interval: "weekly"
```

**Key Distinction:**
- **Dependabot Alerts**: Zero configuration required, automatically scans the entire repository.
- **Dependabot Version Updates**: Requires a detailed `dependabot.yml` with explicit directory definitions.

**Resources:**
- https://docs.github.com/en/code-security/dependabot/dependabot-version-updates/configuration-options-for-the-dependabot.yml-file
</details>

---

### Question 16

**A Dependabot pull request has been open for 30 days without any activity or updates. What happens to it?**

A) It is automatically closed by GitHub.  
B) It is automatically merged into the default branch.  
C) It remains open indefinitely until a developer manual action.  
D) It is closed, and a new identical PR is generated to replace it.  

<details>
<summary>View Answer</summary>

**Answer: A) It is automatically closed by GitHub**

**Explanation:**
- Dependabot automatically closes its own pull requests under specific conditions to avoid pull request pollution:
  - ✅ Open for 30 days without activity.
  - ✅ Merge conflicts arise that Dependabot cannot rebase.
  - ✅ A newer version of the package is released, superseding the PR.

**Best Practices to Avoid Stale PRs:**
- Regularly review and merge Dependabot updates.
- Configure automated merging for low-risk patch updates.
- Implement package grouping to minimize pull request volume.

**Resources:**
- https://docs.github.com/en/code-security/dependabot/working-with-dependabot/managing-pull-requests-for-dependency-updates
</details>

---

### Question 17

**What is the DEFAULT limit of simultaneously open pull requests generated by Dependabot Version Updates?**

A) 3  
B) 5  
C) 10  
D) Unlimited  

<details>
<summary>View Answer</summary>

**Answer: B) 5**

**Explanation:**
- By default, Dependabot caps open Version Update PRs at **5** per ecosystem to prevent developer alert fatigue.
- Once a PR is merged or closed, Dependabot will resume and generate the next scheduled update.

**Increasing the Limit:**
```yaml
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "daily"
    open-pull-requests-limit: 10  # ← Overrides default to 10 open PRs
```

**Operational Parameters:**
- Minimum: `0` (effectively pauses version updates).
- Maximum: `99`.
- Default: `5`.

**CRITICAL EXAM DISTINCTION:**
- The open pull request limit **does NOT apply to Security Updates**.
- Security updates bypass this limit and are always opened immediately to resolve active vulnerabilities.

**Resources:**
- https://docs.github.com/en/code-security/dependabot/dependabot-version-updates/configuration-options-for-the-dependabot.yml-file#open-pull-requests-limit
</details>

---

### Question 18

**What is Dependabot's behavior when a vulnerability is detected in a transitive (indirect) dependency?**

A) It ignores it, as Dependabot only analyzes direct dependencies.  
B) It generates a security alert in the UI, but cannot open an automated pull request.  
C) It generates an alert and opens a PR updating the parent direct dependency that imports it.  
D) It only sends an email notification, without generating a UI alert.  

<details>
<summary>View Answer</summary>

**Answer: C) It generates an alert and opens a PR updating the parent direct dependency that imports it**

**Explanation:**

**Scenario:**
```
Your package.json:
  Express@4.16.0 (Direct Dependency)
    └─ lodash@4.17.15 (Transitive Dependency)  ← VULNERABLE
```

**Dependabot Actions:**
1. Scans dependency tree and detects vulnerable transitive `lodash@4.17.15`.
2. Generates a Secret Scanning/Dependabot alert.
3. Analyzes the dependency graph to find a version of the direct dependency (`express`) that resolves the sub-dependency conflict.
4. Generates a PR: "Bump express from 4.16.0 to 4.18.2" to resolve the nested vulnerability.

*Note: If no version of the parent package resolves the transitive dependency issue, the alert remains active, and the PR will state that manual intervention is required.*

**Resources:**
- https://docs.github.com/en/code-security/dependabot/dependabot-security-updates/about-dependabot-security-updates#about-pull-requests-for-security-updates
</details>

---

### Question 19

**Your organization has 100 private repositories. How can you enable Dependabot Security Updates for all of them at once?**

A) Execute a bulk enablement script using the GitHub CLI.  
B) Enable it via Organization Settings → Code security and analysis → Enable all.  
C) Configure a global GitHub Actions workflow.  
D) Contact GitHub Enterprise Support to apply the setting.  

<details>
<summary>View Answer</summary>

**Answer: B) Enable it via Organization Settings → Enable all**

**Explanation:**

**UI Management Path:**
```
Organization Settings
  → Code security and analysis
  → Dependabot security updates
      [✓] Automatically enable for new repositories
      [Enable all] ← Click here to apply to all existing repos
```

**API Bulk Enablement Alternative:**
```bash
# Query all repositories and pipe to enabling endpoint
gh api /orgs/MY-ORG/repos --paginate | jq -r '.[].name' | while read repo; do
  gh api -X PUT /repos/MY-ORG/$repo/automated-security-fixes
  echo "✅ Enabled for $repo"
done
```

**Resources:**
- https://docs.github.com/en/organizations/keeping-your-organization-secure/managing-security-settings-for-your-organization/managing-security-and-analysis-settings-for-your-organization
</details>

---

### Question 20

**A single package dependency has multiple distinct vulnerabilities associated with it (1 critical, 2 high, 5 medium). How many Dependabot alerts are generated?**

A) 1 alert (grouped by package name).  
B) 3 alerts (grouped by severity levels).  
C) 8 alerts (one unique alert per CVE/GHSA ID).  
D) It depends on the package manager ecosystem.  

<details>
<summary>View Answer</summary>

**Answer: C) 8 alerts (one unique alert per CVE/GHSA ID)**

**Explanation:**
- Dependabot generates alerts per individual vulnerability definition (CVE or GitHub Security Advisory ID), not per package.

**Example Case:**
`lodash@4.17.15` has:
- CVE-2019-10744 (High)
- CVE-2020-8203 (High)
- CVE-2021-23337 (Critical)

**Results:**
- **Alerts**: 3 unique security alerts are generated in the Security tab.
- **PR Fix**: Dependabot Security Updates opens **1 single PR** updating the package to `lodash@4.17.21`, which automatically resolves and closes all 3 alerts simultaneously upon merge.

**Resources:**
- https://docs.github.com/en/code-security/dependabot/dependabot-alerts/viewing-and-updating-dependabot-alerts
</details>

---

### Question 21

**Dependabot Security Updates is enabled, but it is not opening any pull requests for active alerts. What could be the reasons? (Select all that apply)**

A) The repository already has more than 5 open Dependabot PRs.  
B) There is no patched version of the dependency available yet.  
C) The vulnerability is only located in `devDependencies`.  
D) The repository is archived.  

<details>
<summary>View Answer</summary>

**Answers: B, D**

**Explanation:**

**Why Dependabot pauses PR generation:**
- ✅ **B) No patch available**: If the package maintainer has not released a secure version, Dependabot cannot recommend a fix.
- ✅ **D) Archived repositories**: Archived repos are read-only; GitHub restricts automated PR creation.

**Why the other options are incorrect:**
- ❌ A) The open pull request limit (`open-pull-requests-limit`) only restricts Version Updates; Security Updates have no default cap.
- ❌ C) Dependabot treats `devDependencies` and production dependencies identically for security alerts and will open PRs for both.

**Troubleshooting Log Inspection:**
```
Navigate to:
Repository → Insights → Dependency Graph → Dependabot tab
Select the last run time to view full parser logs.
```

**Resources:**
- https://docs.github.com/en/code-security/dependabot/working-with-dependabot/troubleshooting-dependabot-errors
</details>

---

### Question 22

**What information is NOT included inside a Dependabot Security Update pull request description?**

A) The associated CVE ID.  
B) The CVSS severity score.  
C) A proof-of-concept exploit code demonstrating the vulnerability.  
D) The recommended secure patch version.  

<details>
<summary>View Answer</summary>

**Answer: C) A proof-of-concept exploit code demonstrating the vulnerability**

**Explanation:**

**What the PR Description DOES contain:**
- ✅ CVE ID / GHSA ID.
- ✅ CVSS Score (e.g., 9.8 Critical).
- ✅ Vulnerable version range.
- ✅ Safe patched version.
- ✅ Vulnerability description.
- ✅ External reference links (NVD, GitHub Advisory Database).
- ✅ Package release notes and changelogs.

**What is NOT included:**
- ❌ Proof-of-concept exploit scripts (for safety and security reasons).
- ❌ Deep architectural stack traces of the flaw.

**Resources:**
- https://docs.github.com/en/code-security/dependabot/dependabot-security-updates/about-dependabot-security-updates
</details>

---

### Question 23

**Your team wants Dependabot to only generate pull requests for Critical and High severity vulnerabilities, ignoring Medium and Low. How can you configure this?**

A) Specify `min-severity: high` in the `dependabot.yml` file.  
B) Change the alert filter in Organization Settings.  
C) It is not possible; Dependabot always creates PRs for any vulnerability with an available patch.  
D) Add a severity filter in the Dependency Review action configuration.  

<details>
<summary>View Answer</summary>

**Answer: C) It is not possible; Dependabot always creates PRs for any vulnerability with an available patch**

**Explanation:**
- Dependabot Security Updates does NOT support filtering PR generation by severity levels. It automatically creates PRs for any active alert that has a viable patch.

**Workarounds:**
1. **Auto-triage Rules (Enterprise only)**: Automatically dismiss low/medium alerts based on custom logic, which prevents PR creation for those items.
2. **Custom Script Auto-closure**:
```yaml
# GitHub Actions workflow that closes Low-severity PRs
name: Auto-triage Dependabot
on:
  pull_request:
    types: [opened]
jobs:
  triage:
    if: github.actor == 'dependabot[bot]'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/github-script@v7
        with:
          script: |
            const pr = context.payload.pull_request;
            if (pr.body.includes('Low severity')) {
              await github.rest.pulls.update({
                owner: context.repo.owner,
                repo: context.repo.name,
                pull_number: pr.number,
                state: 'closed'
              });
            }
```

**Resources:**
- https://docs.github.com/en/code-security/dependabot/dependabot-security-updates/configuring-dependabot-security-updates
</details>
### Question 24

**What information does the Dependency Graph provide? (Select all that apply)**

A) A comprehensive list of direct dependencies.  
B) A comprehensive list of transitive (indirect) dependencies.  
C) A list of public/private repositories that depend on your repository (dependents).  
D) A list of all known active security vulnerabilities.  
E) The software licenses associated with each dependency.  

<details>
<summary>View Answer</summary>

**Answers: A, B, C, E**

**Explanation:**
- The Dependency Graph is an inventory engine that maps out relationships:
  - ✅ A) Direct dependencies (declared in manifest files like `package.json`).
  - ✅ B) Transitive dependencies (sub-packages imported by your direct dependencies).
  - ✅ C) Dependents (other repositories that consume your library).
  - ✅ E) License details parsed from package metadata (e.g., MIT, Apache-2.0).

**Why D is incorrect:**
- The Dependency Graph itself only acts as an inventory index. Security vulnerabilities are mapped and reported separately by **Dependabot Alerts**.

**UI Location:**
```
Repository → Insights → Dependency graph
  ├─ Dependencies (packages this repository uses)
  └─ Dependents (repositories that consume this code)
```

**Resources:**
- https://docs.github.com/en/code-security/supply-chain-security/understanding-your-software-supply-chain/about-the-dependency-graph
</details>

---

### Question 25

**Your committed `dependabot.yml` file contains a syntax error. How will you be notified of this error?**

A) You will receive an automated email notification.  
B) A failed status check will appear on your recent pull requests.  
C) An error banner will be displayed in the Dependabot tab under Insights.  
D) There is no explicit notification; the updates will simply fail to run.  

<details>
<summary>View Answer</summary>

**Answer: C) An error banner will be displayed in the Dependabot tab under Insights**

**Explanation:**

**Troubleshooting Location:**
```
Repository → Insights → Dependency Graph → Dependabot tab
```

**UI Display Information:**
- `Last checked: 2 hours ago`
- `❌ Configuration error: Invalid YAML syntax`
- `Line 5: Unexpected token. schedule.interval must be one of: daily, weekly, monthly`

**Common YAML Errors:**
- **Indentation Flaws:**
```yaml
# ❌ INCORRECT (Missing indentation)
version: 2
updates:
- package-ecosystem: "npm"
  directory: "/"
```
- **Invalid Enumerated Values:**
```yaml
# ❌ INCORRECT
schedule:
  interval: "hourly"  # "hourly" is invalid; must be daily, weekly, or monthly
```
- **Missing Required Parameters:**
```yaml
# ❌ INCORRECT (Missing the required "directory" field)
updates:
  - package-ecosystem: "npm"
    schedule:
      interval: "weekly"
```

**Validation Workarounds:**
```bash
# Validate using the GitHub API
gh api repos/:owner/:repo/dependency-graph/snapshots \
  --method POST \
  --field version=2
```

**Resources:**
- https://docs.github.com/en/code-security/dependabot/working-with-dependabot/troubleshooting-dependabot-errors
</details>

---

### Question 26

**What file types can Dependabot parse and update? (Select all that apply)**

A) `package.json` (Node.js)  
B) `Dockerfile`  
C) GitHub Actions workflow YAML files  
D) `requirements.txt` (Python)  
E) Terraform modules  

<details>
<summary>View Answer</summary>

**Answers: A, B, C, D**

**Explanation:**

**Supported Ecosystems:**
- ✅ **A) Node.js (`package.json` / package-lock)**:
```yaml
- package-ecosystem: "npm"
```
- ✅ **B) Docker (`Dockerfile` FROM statements)**:
```yaml
- package-ecosystem: "docker"
```
- ✅ **C) GitHub Actions (`uses: actions/checkout@v3` → `@v4`)**:
```yaml
- package-ecosystem: "github-actions"
```
- ✅ **D) Python (`requirements.txt`, `Pipfile`, `pyproject.toml`)**:
```yaml
- package-ecosystem: "pip"
```

**Unsupported Ecosystems:**
- ❌ **E) Terraform modules**: Terraform modules are not natively supported by Dependabot.

**Complete Native Ecosystem List:**
- npm / yarn / pnpm
- bundler (Ruby)
- pip / pipenv / poetry (Python)
- maven / gradle (Java)
- cargo (Rust)
- go modules (Go)
- composer (PHP)
- nuget (.NET)
- docker
- github-actions
- pub (Dart)
- hex (Elixir)

**Resources:**
- https://docs.github.com/en/code-security/dependabot/dependabot-version-updates/configuration-options-for-the-dependabot.yml-file#package-ecosystem
</details>

---

### Question 27

**Your organization enforces a compliance policy stating: "No copyleft or GPL-licensed dependencies are permitted in our applications." How can you automatically enforce this using GHAS?**

A) Configure a Secret Scanning custom pattern.  
B) Write a custom CodeQL query.  
C) Use the Dependency Review Action with `deny-licenses`.  
D) Set up Dependabot Version Updates with package filters.  

<details>
<summary>View Answer</summary>

**Answer: C) Use the Dependency Review Action with deny-licenses**

**Explanation:**

**Action Configuration:**
```yaml
# .github/workflows/dependency-review.yml
name: 'Dependency Review'
on: [pull_request]

permissions:
  contents: read
  pull-requests: write

jobs:
  dependency-review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/dependency-review-action@v4
        with:
          # Block copyleft licenses
          deny-licenses: |
            GPL-2.0
            GPL-3.0
            AGPL-3.0
            LGPL-2.1
            LGPL-3.0
          
          # Allow specific safe licenses
          allow-licenses: |
            MIT
            Apache-2.0
            BSD-2-Clause
            BSD-3-Clause
            ISC
          
          fail-on-scopes: runtime
```

**Outcomes:**
- PR introducing a GPL-licensed package → ❌ **BLOCKED**
- PR containing only MIT/Apache packages → ✅ **PASS**

**Why the other options are incorrect:**
- A) Secret Scanning matches plaintext credentials, not metadata licenses.
- B) CodeQL performs static semantic analysis on code, not package definitions.
- D) Dependabot updates package versions but has no policy enforcement mechanisms.

**Resources:**
- https://github.com/actions/dependency-review-action#configuration-options
</details>

---

### Question 28

**What is Dependabot's behavior when a security vulnerability is identified in a devDependency?**

A) It ignores it, scanning only production dependencies.  
B) It generates a security alert in the UI, but does not open a pull request.  
C) It generates an alert and opens a security update PR, identical to production dependencies.  
D) It only sends an email notification, without generating a UI alert.  

<details>
<summary>View Answer</summary>

**Answer: C) It generates an alert and opens a security update PR, identical to production dependencies**

**Explanation:**
- Dependabot does NOT differentiate between `dependencies` and `devDependencies` when evaluating security risks.

**Scenario:**
```json
{
  "dependencies": {
    "express": "4.16.0"  // Vulnerable → Security alert + PR generated ✅
  },
  "devDependencies": {
    "webpack": "4.0.0"   // Also vulnerable → Security alert + PR generated ✅
  }
}
```

**Reasoning:**
- Compromised development tools can lead to build-time compromises, local development breaches, or supply chain poisoning (e.g., the *event-stream* backdoor incident).

**Segmenting Workflows:**
If you need different reviewers or behavior for dev packages:
```yaml
# dependabot.yml
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
    groups:
      production:
        dependency-type: "production"
      development:
        dependency-type: "development"
```

**Resources:**
- https://docs.github.com/en/code-security/dependabot/dependabot-security-updates/about-dependabot-security-updates
</details>

---

### Question 29

**Dependency Review blocks a PR because it introduces a package licensed under GPL-3.0, but your business team confirms this specific library is strictly required. What is the correct way to handle this?**

A) Temporarily disable the Dependency Review action.  
B) Declare an exception inside the Dependency Review configuration.  
C) Bypass the failing PR status check and merge the code.  
D) Fork the package and manually change its license file.  

<details>
<summary>View Answer</summary>

**Answer: B) Declare an exception inside the Dependency Review configuration**

**Explanation:**

**Adding Package Exceptions:**
```yaml
# .github/workflows/dependency-review.yml
- uses: actions/dependency-review-action@v4
  with:
    deny-licenses: GPL-3.0, AGPL-3.0
    
    # Declare specific packages as exceptions
    allow-dependencies-licenses: |
      pkg:npm/critical-library@1.0.0, GPL-3.0
```

**Using an External Configuration File:**
```yaml
# .github/dependency-review-config.yml
fail-on-scopes:
  - runtime
  - development

deny-licenses:
  - GPL-3.0
  - AGPL-3.0

allow-dependencies-licenses:
  - package-url: pkg:npm/critical-library@1.0.0
    license: GPL-3.0
    reason: "Required for core functionality, legal review completed"
    approved-by: "legal-team@company.com"
    review-date: "2024-01-15"
    next-review: "2025-01-15"
```

**Why other options are poor security practices:**
- A) Disabling the action completely removes all licensing and vulnerability guardrails.
- C) Bypassing creates unmanaged compliance and legal liabilities.
- D) Forking and arbitrarily renaming a license violates copyright and licensing terms.

**Resources:**
- https://github.com/actions/dependency-review-action#allow-licenses-and-deny-licenses
</details>

---

### Question 30

**What is the practical difference between setting `version-update:semver-patch` vs. `version-update:semver-minor` inside a Dependabot grouping configuration?**

A) Patch groups only include bug fixes, while Minor groups include both bug fixes and new backward-compatible features.  
B) Patch refers to pre-1.0.0 packages, whereas Minor refers to post-1.0.0 packages.  
C) There is no practical difference; they behave identically in the YAML parser.  
D) Patch groups only bundle security fixes, while Minor groups exclude them.  

<details>
<summary>View Answer</summary>

**Answer: A) Patch groups only include bug fixes, while Minor groups include both bug fixes and new backward-compatible features**

**Explanation:**

**Semantic Versioning (SemVer) structure:**
`MAJOR.MINOR.PATCH`
- **PATCH**: Bug fixes, backward-compatible, low risk.
- **MINOR**: New features, backward-compatible, moderate risk.
- **MAJOR**: Breaking changes, high risk.

**Example Grouping:**
```yaml
groups:
  # Lowest risk (only patches)
  patch-updates:
    update-types:
      - "version-update:semver-patch"
    # Allows: lodash 4.17.20 → 4.17.21 ✅
    # Blocks: lodash 4.17.21 → 4.18.0  ❌ (this is a minor bump)
  
  # Moderate risk (patches and minors)
  minor-updates:
    update-types:
      - "version-update:semver-minor"
      - "version-update:semver-patch"
    # Allows: lodash 4.17.21 → 4.18.0 ✅
    # Blocks: lodash 4.18.0 → 5.0.0   ❌ (this is a major bump)
```

**Resources:**
- https://semver.org/
- https://docs.github.com/en/code-security/dependabot/dependabot-version-updates/configuration-options-for-the-dependabot.yml-file#groups
</details>

---

### Question 31

**A developer attempts to merge a pull request that introduces a HIGH severity vulnerability, bypassing the failing Dependency Review action check. What determines if they can complete the merge?**

A) GitHub will block the merge automatically because of the High vulnerability status.  
B) A critical incident ticket is instantly logged in the Security tab.  
C) The merge is permitted if the repository's branch protection rules do not require this check to pass.  
D) An automated notification is dispatched to the organization's Security Manager.  

<details>
<summary>View Answer</summary>

**Answer: C) The merge is permitted if the repository's branch protection rules do not require this check to pass**

**Explanation:**
- Dependency Review functions as a pull request check. It does not enforce merge restrictions on its own; enforcement is controlled by **Branch Protection Rules** or **Repository Rulesets**.

**Branch Protection Scenarios:**
- **Require Status Checks to Pass (ON)**: The merge is **blocked** until the check passes or an authorized admin performs an override.
- **Require Status Checks to Pass (OFF)**: The merge is **allowed** despite the failing status check.

**Ensuring Safe Enforcements:**
```
Check the following Branch Protection toggles:
  [✓] Require status checks to pass before merging
  [✓] Do not allow bypassing the above settings (prevents admin overrides)
```

**Resources:**
- https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches
</details>

---

### Question 32

**Your team wants Dependabot to run and open pull requests only on Mondays at 9:00 AM UTC. How should you configure this?**

A) Define `schedule: interval: "weekly" + day: "monday" + time: "09:00" + timezone: "UTC"` in `dependabot.yml`.  
B) It is not possible; Dependabot does not support hour-specific run times.  
C) Configure a custom GitHub Actions workflow triggered by a scheduled Cron.  
D) Adjust the global Dependabot schedule inside Organization Settings.  

<details>
<summary>View Answer</summary>

**Answer: A) Define schedule: interval: "weekly" + day: "monday" + time: "09:00" + timezone: "UTC" in dependabot.yml**

**Explanation:**

**Correct YAML Schema:**
```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
      day: "monday"
      time: "09:00"
      timezone: "UTC"
```

**Available Scheduling Parameters:**
- `interval`: `daily`, `weekly`, or `monthly`.
- `day`: `monday` through `sunday` (only evaluated when `interval: weekly`).
- `time`: `HH:MM` format.
- `timezone`: Standard IANA timezone identifier (e.g. `America/New_York`).

*Note: Execution times are best-effort; runs may vary by ±1 hour depending on active GitHub queue loads.*

**Resources:**
- https://docs.github.com/en/code-security/dependabot/dependabot-version-updates/configuration-options-for-the-dependabot.yml-file#schedule
</details>

---

### Question 33

**What is the purpose of the `rebase-strategy` field in `dependabot.yml`?**

A) It determines how Dependabot resolves Git merge conflicts.  
B) It controls whether Dependabot automatically rebases open pull requests when new commits are pushed to the target branch.  
C) It configures the default merge strategy (merge commit vs. squash merge).  
D) The `rebase-strategy` field does not exist.  

<details>
<summary>View Answer</summary>

**Answer: B) It controls whether Dependabot automatically rebases open pull requests when new commits are pushed to the target branch**

**Explanation:**

**Rebase Strategy Options:**
- **`auto` (default)**: Dependabot automatically rebases its open PRs whenever new commits are pushed to the base branch, or if a merge conflict is detected. This ensures PRs are always up-to-date but can consume substantial CI/CD runner minutes due to re-triggered tests.
- **`disabled`**: Dependabot will never automatically rebase its PRs. Rebases must be triggered manually by developers using the `@dependabot rebase` comment command. Highly recommended for repositories with long-running or expensive CI pipelines.

**Custom Disabling Example:**
```yaml
# dependabot.yml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
    rebase-strategy: "disabled"
```

**Resources:**
- https://docs.github.com/en/code-security/dependabot/dependabot-version-updates/configuration-options-for-the-dependabot.yml-file#rebase-strategy
</details>

---

### Question 34

**A Dependabot alert displays the status "No known patch available." What are the implications of this status? (Select all that apply)**

A) The package maintainer has not yet released a version containing a fix.  
B) Dependabot cannot generate an automated pull request for this alert.  
C) A developer must manually intervene to investigate alternatives or mitigations.  
D) All of the above.  

<details>
<summary>View Answer</summary>

**Answer: D) All of the above**

**Explanation:**
- The status **"No known patch available"** implies:
  - The vulnerability has been publicly disclosed, but the package maintainer has not released a patched version (A).
  - Dependabot cannot recommend a safe version and therefore cannot open an automated pull request (B).
  - Remediation is entirely manual (C).

**Remediation Options for Developers:**
1. Wait for an official patch from the upstream maintainer.
2. Fork the library and apply a localized patch.
3. Migrate the codebase to an alternative, actively maintained package.
4. Implement compensatory architecture controls to mitigate risk and accept the alert.

**Resources:**
- https://docs.github.com/en/code-security/dependabot/dependabot-alerts/viewing-and-updating-dependabot-alerts
</details>

---

### Question 35

**How many simultaneously open pull requests can Dependabot create for Security Updates?**

A) 5 (the default limit).  
B) 10 (the maximum limit).  
C) Unlimited (Dependabot opens PRs for all active security alerts immediately).  
D) It is determined by the `open-pull-requests-limit` setting.  

<details>
<summary>View Answer</summary>

**Answer: C) Unlimited (Dependabot opens PRs for all active security alerts immediately)**

**Explanation:**

**CRITICAL OPERATIONAL DISTINCTION:**
- **Dependabot Security Updates**: Have **no open pull request limits**. Since resolving vulnerabilities is high priority, a PR is opened immediately for every single active alert that has a patch available.
- **Dependabot Version Updates**: Are capped by default at **5** open PRs (configurable via `open-pull-requests-limit`) to prevent developers from being overwhelmed by non-security updates.

**Resources:**
- https://docs.github.com/en/code-security/dependabot/dependabot-security-updates/configuring-dependabot-security-updates
</details>

---

## DOMAIN 4: CODEQL Y CODE SCANNING

### Question 36A

**What is the primary difference between CodeQL "Default Setup" and "Advanced Setup" configurations?**

A) Default Setup is free, while Advanced Setup requires a paid GHAS license.  
B) Default Setup is fully automated, while Advanced Setup requires committing a manual workflow file.  
C) Default Setup only supports JavaScript, while Advanced Setup supports all languages.  
D) There is no functional difference.  

<details>
<summary>View Answer</summary>

**Answer: B) Default Setup is fully automated, while Advanced Setup requires committing a manual workflow file**

**Explanation:**

**Comparison Matrix:**
| Parameter | Default Setup | Advanced Setup |
|-----------|---------------|----------------|
| **Configuration** | 1-click in GitHub UI | `.github/workflows/codeql.yml` |
| **Setup Time** | <30 seconds | 10-30 minutes |
| **Customization** | Limited UI options | Unlimited (YAML custom steps) |
| **Query Suites** | Fixed to `default` | Any suite or custom packs |
| **Path Exclusions**| ❌ No | ✅ Yes (via config file) |
| **Build Control** | Fully automated | Manual steps for compiled code |

**Resources:**
- https://docs.github.com/en/code-security/code-scanning/enabling-code-scanning/configuring-default-setup-for-code-scanning
</details>

---

### Question 36B

**Your CodeQL workflow execution fails with the following runner error: "No code found for language: java". What is the most likely cause?**

A) Java is not supported by the CodeQL static analysis engine.  
B) The compilation autobuild step failed to build the application.  
C) The GitHub runner is missing the required Java Development Kit (JDK).  
D) The repository does not contain any Java files.  

<details>
<summary>View Answer</summary>

**Answer: B) The compilation autobuild step failed to build the application**

**Explanation:**
- For compiled languages (Java, C++, C#, Go, Swift), CodeQL must intercept the compiler during a successful build to generate its relational database.
- If the build step fails or is incomplete, no database is built, resulting in the "No code found" error.

**Common Causes of Autobuild Failures:**
1. **Missing Dependencies**: External package repositories or private registries are unreachable.
2. **Bespoke Build Configurations**: Autobuild fails to detect non-standard maven/gradle setups.
3. **Environment Mismatches**: The compilation requires environment variables or JDK versions not present on the default runner.

**Remediation (Migrating to a manual build step):**
```yaml
- name: Initialize CodeQL
  uses: github/codeql-action/init@v3
  with:
    languages: java

# Explicit manual build instead of autobuild
- name: Compile Application
  run: |
    mvn clean install -DskipTests

- name: Perform CodeQL Analysis
  uses: github/codeql-action/analyze@v3
```

**Resources:**
- https://docs.github.com/en/code-security/code-scanning/troubleshooting-code-scanning
</details>

---

### Question 37

**What file path and name should you use to customize which query suites and files CodeQL executes in Advanced Setup?**

A) `.github/codeql.yml`  
B) `.github/codeql/codeql-config.yml`  
C) `.github/queries.yml`  
D) CodeQL queries cannot be customized.  

<details>
<summary>View Answer</summary>

**Answer: B) .github/codeql/codeql-config.yml**

**Explanation:**

**Example Configuration File:**
```yaml
# .github/codeql/codeql-config.yml
name: "Custom CodeQL Configuration"

disable-default-queries: false

queries:
  - uses: security-extended
  - uses: security-and-quality

packs:
  - codeql/javascript-queries
  - my-org/custom-security-queries@1.0.0

paths:
  - src
  - lib

paths-ignore:
  - tests
  - node_modules
  - '**/*.test.js'

query-filters:
  - exclude:
      id: js/useless-assignment-to-local
```

**Referencing the Configuration File in your Workflow:**
```yaml
- name: Initialize CodeQL
  uses: github/codeql-action/init@v3
  with:
    languages: javascript
    config-file: ./.github/codeql/codeql-config.yml
```

**Resources:**
- https://docs.github.com/en/code-security/code-scanning/creating-an-advanced-setup-for-code-scanning/customizing-your-advanced-setup-for-code-scanning
</details>

---

### Question 38

**A CodeQL analysis run discovers 50 open security alerts. A developer pushes a commit that only updates the project's `README.md` file (no code changes). The CodeQL workflow re-runs and now displays 52 open alerts. What explains this increase?**

A) The first analysis run generated false positives.  
B) The CodeQL query packs were updated upstream in the interim.  
C) The `README.md` file contained executable code blocks that were parsed as vulnerable.  
D) A temporary bug occurred in the CodeQL runner engine.  

<details>
<summary>View Answer</summary>

**Answer: B) The CodeQL query packs were updated upstream in the interim**

**Explanation:**
- By default, CodeQL workflows fetch the latest query bundles and engine versions during execution unless pinned to a specific release.
- Upstream query pack updates can introduce new rules, security patterns, or logic improvements that detect existing vulnerabilities that went unnoticed in previous runs.

**Fixing CodeQL Versions (Query Pinning):**
```yaml
- uses: github/codeql-action/init@v3
  with:
    # Pin CodeQL analysis to a specific bundle release
    tools: https://github.com/github/codeql-action/releases/download/codeql-bundle-20231201/codeql-bundle.tar.gz
```

**Resources:**
- https://github.com/github/codeql-action/releases
</details>

---

### Question 39

**What is the function of the `category` input parameter inside a CodeQL workflow?**

A) It defines the category of security vulnerability being scanned (e.g. SQL Injection vs. XSS).  
B) It acts as a unique identifier to distinguish between multiple separate analysis runs on the same repository.  
C) It specifies the severity rating of the findings to upload.  
D) It declares the programming language targeted for analysis.  

<details>
<summary>View Answer</summary>

**Answer: B) It acts as a unique identifier to distinguish between multiple separate analysis runs on the same repository**

**Explanation:**
- The `category` input allows multiple distinct CodeQL analysis workflows to upload results for the same repository without overwriting each other.

**Example Case (Paralellizing Security and Quality scans):**
```yaml
# Job 1: Security Scan
- name: CodeQL Security Scan
  uses: github/codeql-action/analyze@v3
  with:
    category: "/suite:security"

# Job 2: Quality Scan
- name: CodeQL Quality Scan
  uses: github/codeql-action/analyze@v3
  with:
    category: "/suite:quality"
```

*Without the `category` tag, the second analysis upload would overwrite the results of the first, resulting in lost alerts.*

**UI Filtering:**
Inside `Security → Code scanning alerts`, users can filter results by category:
- `/suite:security`
- `/suite:quality`

**Resources:**
- https://docs.github.com/en/code-security/code-scanning/creating-an-advanced-setup-for-code-scanning/codeql-code-scanning-for-compiled-languages#specifying-codeql-query-suites
</details>

---

### Question 40

**A CodeQL analysis workflow takes 3 hours to execute on a very large repository. Which of the following is NOT a valid optimization strategy to reduce execution time?**

A) Migrating to self-hosted runners equipped with higher CPU core counts.  
B) Parallelizing scans by language using a matrix strategy.  
C) Switching from the `security-extended` query suite to the lighter `security` suite.  
D) Increasing the `timeout-minutes` value of the GitHub Actions job.  

<details>
<summary>View Answer</summary>

**Answer: D) Increasing the timeout-minutes value of the GitHub Actions job**

**Explanation:**
- Increasing the job timeout permits a long-running execution to complete without being killed, but it does **not optimize execution performance** or reduce runtime.

**Valid Optimization Strategies:**
- **A) High-performance runners**: Allocating more CPU cores speeds up query execution.
- **B) Matrix parallelization**: Analyzing multiple languages concurrently rather than sequentially.
```yaml
strategy:
  matrix:
    language: ['javascript', 'python', 'java']
# Runs 3 concurrent jobs, cutting analysis time by up to 3x.
```
- **C) Lighter query suites**: The `security` suite contains ~100 queries, while `security-extended` contains ~200, making the default suite significantly faster.
- **Path Filtering**: Excluding test or mock directories to avoid scanning non-production code.
```yaml
paths-ignore:
  - 'test/**'
  - 'docs/**'
```

**Resources:**
- https://docs.github.com/en/code-security/code-scanning/creating-an-advanced-setup-for-code-scanning/codeql-code-scanning-for-compiled-languages#improving-the-performance-of-codeql-analysis
</details>

---

# EXAM STRATEGY <a id="exam-strategy"></a>

> 🌐 **Available in:** **English 🇬🇧** | [Español 🇪🇸](stractegic-guide-for-the-ghas-exam.md)
>
> *Note: The Exam Strategy section in the original Spanish version was left as an interactive stub to be continued, but for mapping consistency, we provide this placeholder with the corresponding English cross-reference.*
