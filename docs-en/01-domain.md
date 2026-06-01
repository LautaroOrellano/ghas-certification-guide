> 🌐 **Available in:** **English 🇬🇧** | [Español 🇪🇸](../docs/01-dominio.md)
---

# DOMAIN 1: SECURITY FEATURES AND CAPABILITIES OF GHAS (15%)

<a id="d1-1"></a>
## 1.1 Compare GHAS features and its role in the security ecosystem

### What is GitHub Advanced Security (GHAS)?

**As of April 2025**, GHAS is divided into two independent products:

1. **GitHub Secret Protection** ($19/month per active committer)
    - Secret scanning (repository scanning)
    - Push protection (push protection)
    - Copilot secret scanning (AI detection)
    - Custom patterns (custom patterns)
    - Delegated bypass/dismissal
    - Security campaigns
    - Security overview

2. **GitHub Code Security** ($30/month per active committer)
    - Code scanning with CodeQL
    - Copilot Autofix
    - Security campaigns
    - Custom auto-triage rules for Dependabot
    - Dependency review
    - Security overview

### Core Components of GHAS:

#### a) **Code Scanning**
- Static analysis engine that identifies security vulnerabilities
- Uses CodeQL (GitHub's query language)
- Detects: SQL injection, XSS, authentication bypass, etc.
- Runs on GitHub Actions or external CI

#### b) **Secret Scanning**
- Detects credentials, tokens, API keys in the code
- Notifies service providers for revocation
- Includes push protection to block commits with secrets
- Supports custom patterns

#### c) **Dependabot**
- Scanning for vulnerable dependencies
- Automatic vulnerability alerts
- Automatic update pull requests
- Security and version updates

#### d) **Dependency Review**
- Review of vulnerable dependencies
- Blocks merges that introduce vulnerabilities
- Verifies licenses
- Analyzes the dependency graph

#### e) **Security Overview**
- Centralized security dashboard
- Vulnerability metrics and trends
- Organization/Enterprise-level view
- Identification of high-risk repositories

### Role in the Security Ecosystem:

GHAS integrates into **Shift Left Security** - moving security to the beginning of the SDLC:

```
Development → Commit → PR → Merge → Deploy → Production
    ↓         ↓      ↓      ↓        ↓         ↓
  CodeQL   Secret  Dep   Code    Runtime  Monitoring
           Scan   Review Scan    Security
```

**Links:**
- https://docs.github.com/en/get-started/learning-about-github/about-github-advanced-security
- https://github.com/security/advanced-security
- https://resources.github.com/evolving-github-advanced-security/

---
<a id="d1-2"></a>
## 1.2 Differentiate features for Open Source vs GHEC/GHES projects

### Public Repositories (Open Source) - FREE

**Automatic Features:**
- Secret scanning (secret scanning)
- Push protection (push protection)
- Dependency graph (dependency graph)
- Dependency alerts (vulnerable dependency alerts)
- Dependabot security updates (automatic updates)

**NOT Included:**
- Code scanning with CodeQL (requires manual configuration but is free)
- Security overview
- Custom patterns for secret scanning
- Advanced security overview features

### Private Repositories with GHAS (GHEC/GHES)

**Requires license for:**
- GitHub Secret Protection, or
- GitHub Code Security, or
- GitHub Enterprise (which included GHAS until April 2025)

**Additional Features:**

| Feature | Public | Private without GHAS | Private with GHAS |
|----------------|---------|------------------|------------------|
| Secret Scanning | ✅ | ❌ | ✅ |
| Push Protection | ✅ | ❌ | ✅ |
| Code Scanning | ✅ Manual | ❌ | ✅ |
| Dependabot Alerts | ✅ | ✅ | ✅ |
| Dependency Review | ❌ | ❌ | ✅ |
| Security Overview | ❌ | ❌ | ✅ |
| Custom Patterns | ❌ | ❌ | ✅ |
| Copilot Autofix | ❌ | ❌ | ✅ |
| Auto-triage Rules | ❌ | ❌ | ✅ |

### GHES (GitHub Enterprise Server) - Considerations

**Additional Requirements:**
- Runners with Docker for Dependabot
- Internet connectivity (or private registry)
- CodeQL CLI installed locally
- Minimum version: GHES 3.8+

**Features by version:**
- GHES 3.8: Dependabot without internet using private registry
- GHES 3.9+: Security configurations
- GHES 3.10+: Delegated bypass

**Links:**
- https://docs.github.com/en/enterprise-server@latest/admin/code-security
- https://github.com/advanced-security/advanced-security-material

---
<a id="d1-3"></a>
## 1.3 Features and benefits of the Security Overview

### What is Security Overview?

Centralized dashboard that provides visibility into the security posture at the:
- **Organization** level: All repositories
- **Enterprise** level: All organizations
- **Team** level: Team's repositories

### Key Features:

#### a) **Views and Metrics**

**1. Security Risk View**
- Repositories sorted by risk level
- Metrics: open alerts, severity, age
- Filters: language, team, archetype

**2. Security Coverage View**
- % of repositories with GHAS enabled
- Identification of security gaps
- Feature status per repository

**3. Alert Trends**
- Temporal graphs of alerts
- Remediation trends
- Comparisons by type

#### b) **Advanced Filters**

```yaml
Available filters:
  - Alert type: code-scanning, secret-scanning, dependabot
  - Severity: critical, high, medium, low
  - State: open, closed, dismissed, fixed
  - Repository: name, archetype, team
  - Language: java, javascript, python, etc.
  - Alert age: days since creation
```

#### c) **Management Capabilities**

- **Bulk actions**: close/dismiss multiple alerts
- **Security campaigns**: coordinate remediation
- **CSV exports**: reports for stakeholders
- **API access**: integration with SIEM/dashboards

### Benefits:

1. **Centralized visibility**: A single view of the entire security posture
2. **Prioritization**: Identify critical repositories
3. **Compliance**: Demonstrate compliance with policies
4. **Metrics**: Security KPIs (MTTR, coverage, trends)
5. **Accountability**: Assign ownership of remediation

### Use Cases:

**For Security Teams:**
- GHAS coverage audit
- Identification of vulnerability hotspots
- Remediation SLA tracking

**For Engineering Managers:**
- Compare security maturity across teams
- Plan security tech debt
- Justification for security investment

**For Compliance:**
- Status reports for audits
- Evidence of security controls
- Exception/waiver tracking

**Links:**
- https://docs.github.com/en/code-security/security-overview/about-security-overview
- https://docs.github.com/en/code-security/security-overview/assessing-adoption-code-security

---
<a id="d1-4"></a>
## 1.4 Differences between Secret Scanning and Code Scanning

### Secret Scanning

**Purpose**: Detect exposed credentials and secrets

**What does it detect?**
- API keys, tokens, certificates
- Passwords, connection strings
- Private SSH/PGP keys
- Cloud credentials (AWS, Azure, GCP)

**How it works:**
1. Scans the entire repository history
2. Uses regex patterns for matching
3. Validates with providers (validity check)
4. Notifies the service provider for revocation
5. Creates an alert in the Security tab

**Pattern Types:**
- **Default**: 200+ patterns from GitHub
- **Partner patterns**: Validated by providers
- **Custom patterns**: Defined by the organization

**Detection Example:**
```javascript
// This would trigger an alert:
const apiKey = "ghp_1234567890abcdefghijklmnopqrstuv";

// Detected pattern: GitHub Personal Access Token
// Validity: Active (verified with GitHub API)
// Action: Notify user, revoke token
```

**Push Protection:**
- Blocks commits with secrets in real-time
- Allows bypass with justification
- Configurable: block or warn

### Code Scanning

**Purpose**: Detect vulnerabilities and code errors

**What does it detect?**
- SQL injection, XSS, CSRF
- Path traversal, command injection
- Authentication/authorization flaws
- Resource leaks, race conditions
- CWE (Common Weakness Enumeration)

**How it works:**
1. CodeQL builds a database of the code
2. Runs queries to find patterns
3. Analyzes data flow and control flow
4. Generates SARIF results
5. Creates alerts with location and explanation

**Detection Example:**
```java
// This would trigger an alert:
String query = "SELECT * FROM users WHERE id = " + userId;
// ↑ CWE-89: SQL Injection

// CodeQL query detects:
// - userId comes from unsanitized input
// - It is concatenated directly into SQL
// - No prepared statement is used
// Severity: High
// Recommendation: Use PreparedStatement
```

### Direct Comparison

| Aspect | Secret Scanning | Code Scanning |
|---------|----------------|---------------|
| **Objective** | Exposed credentials | Code vulnerabilities |
| **Technology** | Pattern matching (regex) | Static analysis (CodeQL) |
| **Scope** | Entire history | Code on HEAD branch |
| **Speed** | Fast (minutes) | Slower (minutes to hours) |
| **False positives** | Low (with validation) | Medium (depends on queries) |
| **Remediation** | Revoke + rotate secret | Refactor vulnerable code |
| **Integration** | Automatic on push | GitHub Actions workflow |
| **CPU/Memory** | Low | High (compilation + analysis) |

### When to use each:

**Secret Scanning:**
- ✅ Detect credential exposure
- ✅ Secrets compliance
- ✅ Prevention of leaks in CI/CD
- ✅ Historical secrets audit

**Code Scanning:**
- ✅ Detect security bugs
- ✅ Automated code review
- ✅ SAST (Static Application Security Testing)
- ✅ Maintain code quality

**Links:**
- https://docs.github.com/en/code-security/secret-scanning
- https://docs.github.com/en/code-security/code-scanning

---
<a id="d1-5"></a>
## 1.5 Secure development lifecycle with GHAS

### Integration into the SDLC

```
┌─────────────┐
│   PLAN      │ → Security requirements
└──────┬──────┘
       │
┌──────▼──────┐
│   CODE      │ → CodeQL analysis (IDE extension)
└──────┬──────┘   Pre-commit hooks
       │
┌──────▼──────┐
│   COMMIT    │ → Secret scanning push protection
└──────┬──────┘   Block commits with secrets
       │
┌──────▼──────┐
│  PULL REQ   │ → Code scanning (PR check)
└──────┬──────┘   Dependency review (PR check)
       │           Block merge if vulnerabilities
┌──────▼──────┐
│   MERGE     │ → Alerts on main branch
└──────┬──────┘   Dependabot security updates
       │
┌──────▼──────┐
│   DEPLOY     │ → Security campaigns
└──────┬──────┘   SBOM generation
       │
┌──────▼──────┐
│ PRODUCTION  │ → Runtime monitoring (external)
└──────┬──────┘   Incident response
       │
┌──────▼──────┐
│  MONITOR    │ → Security overview
└─────────────┘   Metrics & trends
```

### Scenario A: Isolated Security (Traditional)

**Process:**
1. Complete development without checks
2. Manual code review
3. Merge to main without validation
4. Deploy to staging
5. **Pentest/security review** → Vulnerabilities discovered
6. Rollback or urgent hotfix
7. Cycle repeats

**Problems:**
- ❌ Vulnerabilities discovered late
- ❌ High cost of remediation
- ❌ Releases delayed
- ❌ Security as a bottleneck
- ❌ "Security vs Velocity" culture

**Typical Metrics:**
- Time to fix: 30-90 days
- Cost per vulnerability: $500-$5,000
- Accumulated security debt

### Scenario B: Integrated Security (GHAS)

**Process:**
1. Development with CodeQL in IDE
2. **Commit blocked** if secrets are present
3. **PR checks** block merge if:
   - Code scanning finds critical/high issues
   - Dependency review detects vulnerabilities
4. Merge only if all checks pass
5. Dependabot creates automatic PRs
6. Security overview monitors everything

**Benefits:**
- ✅ Shift left: bugs found early
- ✅ Automatic remediation: Dependabot PRs
- ✅ No delays: checks run in parallel
- ✅ Developer ownership: immediate context
- ✅ "Security enables velocity" culture

**Typical Metrics:**
- Time to fix: 1-7 days
- Cost per vulnerability: $50-$500
- 70% reduction in security debt

### Impact Comparison

| Metric | Without GHAS | With GHAS | Improvement |
|---------|----------|----------|--------|
| Vulnerabilities in production | 50/year | 10/year | -80% |
| Remediating time | 45 days | 5 days | -89% |
| Cost per vulnerability | $2,000 | $200 | -90% |
| Test coverage | Manual | Automatic | +100% |
| Developer satisfaction | Low | High | Better experience |

### Best Practices for Integration

**1. Plan Phase:**
- Define security requirements
- Threat modeling
- Align with compliance

**2. Code Phase:**
- CodeQL CLI local
- IDE extensions (VS Code, IntelliJ)
- Pre-commit hooks

**3. Commit Phase:**
- Enable push protection
- Educate on secrets management
- Use secret managers (Vault, etc.)

**4. PR Phase:**
- Required status checks
- Configure severity thresholds
- Code scanning + dependency review

**5. Merge Phase:**
- Protect main branch
- Require passing checks
- CODEOWNERS review

**6. Deploy Phase:**
- Generate SBOM
- Security campaigns for technical debt
- Automated rollback if critical alerts appear

**7. Monitor Phase:**
- Security overview dashboard
- Remediation SLAs
- Metrics for continuous improvement

**Links:**
- https://docs.github.com/en/code-security/getting-started/securing-your-repository
- https://docs.github.com/en/enterprise-cloud@latest/code-security/tutorials/adopting-github-advanced-security-at-scale

---
<a id="d1-6"></a>
## 1.6 Explain and utilize specific GHAS features

### a) Identification of vulnerable dependencies

1. **Manifest Analysis**
   - GitHub detects dependency files:
     ```
     package.json, package-lock.json (npm)
     Gemfile, Gemfile.lock (Ruby)
     pom.xml (Maven)
     build.gradle (Gradle)
     requirements.txt, Pipfile (Python)
     go.mod (Go)
     Cargo.toml (Rust)
     composer.json (PHP)
     packages.config, *.csproj (NuGet)
     ```

2. **Dependency Graph Construction**
   - Parses each manifest
   - Identifies direct and transitive (indirect) dependencies
   - Builds dependency tree
   - Updates on every commit

3. **Comparison with Databases**
   - **GitHub Advisory Database**: CVEs + community submissions
   - **NVD (National Vulnerability Database)**
   - **WhiteSource/Snyk advisories** (partner data)
   - Continuous updates (multiple times a day)

4. **Alert Generation**
   - Match: dependency + version → advisory
   - Severity calculation (CVSS score)
   - Notification according to configuration
   - Issue creation in Security tab

**Practical Example:**

```json
// package.json
{
  "dependencies": {
    "express": "4.16.0"  // ← Vulnerable version
  }
}
```

**Result:**
```
Dependabot Alert:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📦 express
🔴 Critical severity
🆔 CVE-2022-24999
📝 express <4.17.3: Prototype pollution vulnerability
🔧 Recommendation: Upgrade to 4.17.3+
✅ Auto-PR available
```

### b) How to act upon GHAS alerts

**Decision Workflow:**

```
Does a GHAS alert appear?
       │
       ├─→ [1] EVALUATE SEVERITY
       │    ├─ Critical/High → Priority 1 (resolve in 7 days)
       │    ├─ Medium → Priority 2 (resolve in 30 days)
       │    └─ Low → Priority 3 (resolve in 90 days)
       │
       ├─→ [2] VERIFY EXPLOITABILITY
       │    ├─ Is the affected code in use?
       │    ├─ Is it reachable by users?
       │    └─ Are there mitigations?
       │
       ├─→ [3] INVESTIGATE CONTEXT
       │    ├─ Read CVE description
       │    ├─ Review PoC (Proof of Concept)
       │    ├─ Consult vendor advisories
       │    └─ Check if public exploit exists
       │
       └─→ [4] DECIDE ACTION

ACTION A: FIX
  → Accept Dependabot PR
  → Update dependency manually
  → Refactor vulnerable code
  → Test + deploy

ACTION B: MITIGATE
  → Implement temporary workaround
  → Configure WAF rules
  → Disable affected feature
  → Monitor while waiting for patch

ACTION C: ACCEPT RISK
  → Dismiss alert with justification
  → Document decision
  → Set reminder to revisit
  → Notify security team

ACTION D: FALSE POSITIVE
  → Verify it is a false positive
  → Dismiss as "Won't fix" or "Used in tests"
  → Report to GitHub if it is a scanner bug
```

**Dismissal Options:**

```yaml
Valid reasons to dismiss:
  - won't_fix: Will not be corrected (business decision)
  - false_positive: Not actually vulnerable
  - used_in_tests: Only used in tests, not production
  - tolerable_risk: Risk accepted (documented)
```

### c) Implications of ignoring an alert

**Technical Risks:**
- 🔴 Exploitation in production
- 🔴 Data breach / compromise
- 🔴 Lateral movement by attackers
- 🔴 Supply chain attacks cascade

**Business Risks:**
- 💰 Compliance fines (GDPR, PCI-DSS)
- 💰 Incident response costs
- 💰 Reputational loss
- 💰 Lawsuits from affected users

**Operational Risks:**
- ⚠️ Tech debt accumulation
- ⚠️ Difficulty for future upgrades
- ⚠️ Growing complexity of remediation
- ⚠️ Team alert fatigue

**Best Practices:**
- ✅ **NEVER ignore without documented justification**
- ✅ Establish SLAs by severity
- ✅ Require approval to dismiss Critical/High alerts
- ✅ Audit trail for all decisions
- ✅ Revisit dismissed alerts periodically

### d) Role of the developer upon discovering an alert

**Responsibilities:**

**1. Immediate Triage**
```bash
# Upon seeing alert in PR
1. Read full description
2. Click "Show paths" to see data flow
3. Understand related CWE and CVE
4. Verify if it is a false positive
```

**2. Communication**

```yaml
- If critical/high:
    - Notify team lead IMMEDIATELY
    - Tag @security-team in PR
    - Create incident ticket
    
- If medium/low:
    - Comment in PR with remediation plan
    - Estimate effort
    - Schedule fix in next sprint
```

**3. Remediation**

```python
# For code scanning:
1. Read CWE documentation
2. Review fix examples
3. Apply fix following best practices
4. Add test verifying the correction
5. Re-run code scanning

# For secret scanning:
1. Revoke secret IMMEDIATELY
2. Rotate to new secret
3. Update in secret manager
4. Audit logs to check for exposure
5. Commit fix

# For Dependabot:
1. Review changelog of the new version
2. Check breaking changes
3. Update and run test suite
4. Merge Dependabot PR
```

**4. Documentation**

```markdown
## Security Fix: [CVE-2024-XXXX]

### Vulnerability
- Type: SQL Injection (CWE-89)
- Severity: High
- Affected: UserController.java:142

### Root Cause
Unsanitized user input concatenated in SQL query

### Fix Applied
Migrated to PreparedStatement with parameterized queries

### Testing
- Added test for SQL injection attempt
- Verified existing tests pass
- Manual security testing performed

### Prevention
- Added lint rule to catch similar patterns
- Updated coding guidelines
```

**5. Future Prevention**
- Add linting rules
- Update team guidelines
- Share learnings in team meeting
- Contribute queries to CodeQL if it is a new pattern

### e) Differences in access management by feature

**Secret Scanning:**

| Role | View Alerts | Dismiss Alerts | Configure | Bypass Push Protection |
|-----|-------------|----------------|------------|------------------------|
| Read | ❌ | ❌ | ❌ | ❌ |
| Triage | ❌ | ❌ | ❌ | ❌ |
| Write | ❌ | ❌ | ❌ | ✅ (with justification) |
| Maintain | ❌ | ❌ | ❌ | ✅ |
| Admin | ✅ | ✅ | ✅ | ✅ |
| Security Manager | ✅ | ✅ | ✅ | N/A |

**Code Scanning:**

| Role | View Alerts | Dismiss Alerts | Configure Workflow | View SARIF |
|-----|-------------|----------------|---------------------|-----------|
| Read | ✅ | ❌ | ❌ | ✅ |
| Triage | ✅ | ❌ | ❌ | ✅ |
| Write | ✅ | ✅ | ❌ | ✅ |
| Maintain | ✅ | ✅ | ✅ | ✅ |
| Admin | ✅ | ✅ | ✅ | ✅ |

**Dependabot:**

| Role | View Alerts | Dismiss Alerts | View PRs | Merge PRs | Configure |
|-----|-------------|----------------|---------|-----------|------------|
| Read | ✅ | ❌ | ✅ | ❌ | ❌ |
| Triage | ✅ | ❌ | ✅ | ❌ | ❌ |
| Write | ✅ | ✅ | ✅ | ✅ | ❌ |
| Maintain | ✅ | ✅ | ✅ | ✅ | ✅ |
| Admin | ✅ | ✅ | ✅ | ✅ | ✅ |

**Security Overview:**

| Role | Org View | Enterprise View | Export Data | Manage Campaigns |
|-----|-------------------|---------------|----------------|---------------------|
| Org member | ❌ | ❌ | ❌ | ❌ |
| Org owner | ✅ | ❌ | ✅ | ✅ |
| Security manager | ✅ | ❌ | ✅ | ✅ |
| Enterprise owner | ✅ | ✅ | ✅ | ✅ |

**Notification Configuration:**

```yaml
# .github/workflows/notify.yml
# Granular control of who receives what

secret_scanning:
  notifications:
    - email: security@company.com
      severity: [critical, high]
    - slack: #security-alerts
      severity: [critical, high, medium]
    - pagerduty: oncall-security
      severity: [critical]

code_scanning:
  notifications:
    - teams: ["@org/security", "@org/backend"]
      severity: [critical, high]
    - individuals: ["security-lead@company.com"]
      severity: [critical]

dependabot:
  notifications:
    - teams: ["@org/developers"]
      severity: [critical, high]
    - slack: #deps-updates
      severity: all
```

### f) Where to utilize Dependabot alerts in the SDLC

**1. Planning / Backlog Grooming**
- Review Dependabot alerts
- Prioritize by severity
- Estimate update effort
- Plan in sprints

**2. Development**
- Monitor new alerts daily
- Review Dependabot PRs
- Test compatibility of updates

**3. Code Review / PR**
- Dependency review action blocks PRs with vulnerabilities
- Approve Dependabot PRs after testing
- Merge fixes before features

**4. CI/CD Pipeline**
```yaml
# Integrate checks in pipeline
- name: Check Dependabot alerts
  run: |
    critical=$(gh api /repos/:owner/:repo/dependabot/alerts \
      --jq '.[] | select(.state=="open" and .security_advisory.severity=="critical") | .security_advisory.ghsa_id')
    if [ -n "$critical" ]; then
      echo "❌ Critical Dependabot alerts found!"
      exit 1
    fi
```

**5. Release Management**
- Verify 0 critical/high alerts before release
- Include security fixes in changelog
- Communicate updates to stakeholders

**6. Post-deployment**
- Monitor for new advisories
- Auto-merge low-risk Dependabot PRs
- Weekly review of open alerts

**7. Incident Response**
- If a public exploit emerges:
  - Dependabot alerts immediately
  - Emergency patch deploy
  - Postmortem and lessons learned

**Recommended Automation:**

```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "daily"
    # Auto-merge for security patches
    open-pull-requests-limit: 10
    labels:
      - "dependencies"
      - "security"
    reviewers:
      - "org/security-team"
    assignees:
      - "security-lead"
    
  # Grouping for multiple updates
  - package-ecosystem: "npm"
    directory: "/frontend"
    groups:
      development-dependencies:
        dependency-type: "development"
      production-dependencies:
        dependency-type: "production"
```

**Links:**
- https://docs.github.com/en/code-security/dependabot
- https://docs.github.com/en/code-security/supply-chain-security/understanding-your-software-supply-chain/about-the-dependency-graph
- https://docs.github.com/en/code-security/security-overview
