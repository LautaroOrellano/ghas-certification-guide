> 🌐 **Available in:** **English 🇬🇧** | [Español 🇪🇸](../docs/02-dominio.md)
---

# DOMAIN 2: CONFIGURE AND USE SECRET SCANNING (15%)

<a id="d2-1"></a>
## 2.1 Describe secret scanning

### What is Secret Scanning?

**Definition**: A GHAS feature that detects credentials, tokens, and other secrets that have been accidentally committed to repositories.

**Internal Functioning:**

1. **Initial Scan**: Upon enablement, scans the entire history of the repository.
2. **Continuous Scan**: Every new push is scanned automatically.
3. **Pattern Matching**: Uses regex patterns to identify secrets.
4. **Validation**: Verifies with the service provider whether the secret is valid.
5. **Alerts**: Notifies the user and the service provider.

### Types of Secrets Detected

**Main Categories:**

```yaml
Authentication Tokens:
  - GitHub Personal Access Tokens (PAT)
  - OAuth tokens
  - JWT tokens
  - Session tokens

API Keys:
  - AWS Access Keys
  - Google Cloud API keys
  - Azure Storage keys
  - Stripe API keys
  - Twilio Auth tokens
  - +200 providers

Certificates and Keys:
  - Private SSH keys
  - PGP private keys
  - TLS/SSL certificates
  - Code signing certificates

Database Credentials:
  - MongoDB connection strings
  - PostgreSQL passwords
  - MySQL credentials
  - Redis authentication

Cloud Credentials:
  - AWS IAM credentials
  - Azure service principals
  - GCP service account keys
  - Docker Hub tokens

Passwords:
  - Generic passwords (with AI detection)
  - LDAP credentials
  - FTP passwords
```

### Partner Patterns vs Custom Patterns

**Partner Patterns (GitHub + Providers):**
- 200+ predefined patterns
- Automatic validation with providers
- Automatic revocation possible
- Maintained by GitHub
- **Examples**: AWS, Stripe, Slack, Azure

**Custom Patterns (User-defined):**
- Specific organization patterns
- Custom regexes
- No automatic validation
- Manual maintenance
- **Examples**: Internal API keys, proprietary tokens

### Validity Checks

**What are they?**

When secret scanning detects a secret, it attempts to verify if it is still valid:

```
Secret detected → Pattern match
        ↓
Does provider support validation?
        ├─ YES → Call provider's API
        │        ├─ Active ✅ → CRITICAL alert
        │        ├─ Inactive ❌ → Low priority
        │        └─ Unknown ⚠️ → Medium priority
        │
        └─ NO → Create alert without validation
```

**Validity States:**

| State | Meaning | Priority | Action |
|--------|-------------|-----------|--------|
| **Active** | Valid and active secret | 🔴 Critical | Revoke IMMEDIATELY |
| **Inactive** | Revoked/expired secret | 🟢 Low | Clean code |
| **Unknown** | Could not be verified | 🟡 Medium | Investigate manually |
| **No check** | Provider doesn't support validation | 🟡 Medium | Assume active |

**Providers with Validity Checks:**

- ✅ GitHub (tokens)
- ✅ AWS (IAM keys)
- ✅ Google Cloud
- ✅ Azure
- ✅ Stripe
- ✅ Slack
- ✅ Twilio
- ✅ Dropbox
- ❌ Many custom patterns

**Practical Example:**

```python
# secrets.py - INCORRECT ❌
GITHUB_TOKEN = "ghp_AbCd1234567890EfGhIjKlMnOpQrStUv"
AWS_KEY = "AKIA2345678901234567"

# Secret scanning detects:
# 1. GitHub token
#    → Validates with GitHub API
#    → State: Active ✅
#    → Alert: CRITICAL
#    → Action: Token auto-revoked by GitHub
#
# 2. AWS key
#    → Validates with AWS
#    → State: Active ✅
#    → Alert: CRITICAL
#    → Action: Notify AWS account owner
```

### Secret Scanning Architecture

```
┌──────────────────────────────────────────┐
│         GitHub Repository                │
│  ┌────────────────────────────────────┐  │
│  │  Git History                       │  │
│  │  ├─ commit 1                       │  │
│  │  ├─ commit 2                       │  │
│  │  └─ commit N                       │  │
│  └────────────────────────────────────┘  │
└──────────────┬───────────────────────────┘
               │
               ▼
┌──────────────────────────────────────────┐
│       Secret Scanning Engine             │
│  ┌────────────────────────────────────┐  │
│  │  Pattern Library                   │  │
│  │  ├─ Partner patterns (200+)        │  │
│  │  ├─ Custom patterns (user-defined) │  │
│  │  └─ AI-powered detection           │  │
│  └────────────────────────────────────┘  │
│               │                          │
│               ▼                          │
│  ┌────────────────────────────────────┐  │
│  │  Validity Checker                  │  │
│  │  ├─ Call provider APIs             │  │
│  │  ├─ Cache results                  │  │
│  │  └─ Update alert severity          │  │
│  └────────────────────────────────────┘  │
└──────────────┬───────────────────────────┘
               │
               ├──────────────┬────────────────┐
               ▼              ▼                ▼
       ┌─────────────┐  ┌──────────┐  ┌────────────┐
       │GitHub Alert │  │ Provider │  │  Webhook   │
       │   (UI)      │  │Notificat.│  │  (SIEM)    │
       └─────────────┘  └──────────┘  └────────────┘
```

**Links:**
- https://docs.github.com/en/code-security/secret-scanning/introduction/about-secret-scanning
- https://docs.github.com/en/code-security/secret-scanning/introduction/supported-secret-scanning-patterns

---
<a id="d2-2"></a>
## 2.2 Describe Push Protection

### What is Push Protection?

**Definition**: A feature that **blocks in real-time** any push containing secrets, preventing them from reaching the repository.

**Difference with Traditional Secret Scanning:**

```
Secret Scanning (Traditional):
  Developer commit → Push → Repository → Scan → Alert
  ❌ Secret is ALREADY in history

Push Protection:
  Developer commit → Push BLOCKED → Notification → Fix → Push again
  ✅ Secret NEVER reaches the repository
```

### How Push Protection Works

```
┌─────────────────┐
│ Developer       │
│ git push origin │
└────────┬────────┘
         │
         ▼
┌─────────────────────────────┐
│  Pre-receive Hook           │
│  ├─ Scan commits for secrets│
│  ├─ Run pattern matching    │
│  └─ Check validity          │
└────────┬────────────────────┘
         │
     ┌────┴────┐
     │ Secret? │
     └────┬────┘
         │
     ┌────┴─────┐
    YES        NO
     │          │
     ▼          ▼
┌─────────┐  ┌──────────┐
│ BLOCK   │  │ ALLOW    │
│ Push    │  │ Push     │
└────┬────┘  └──────────┘
     │
     ▼
┌──────────────────────────────┐
│ Display to developer:        │
│ ❌ Push blocked!             │
│ 📍 Secret found in:          │
│    file.py:42                │
│ 🔐 Type: GitHub PAT          │
│ ⚠️ Validity: Active          │
│                              │
│ Options:                     │
│ 1. Remove secret & push again│
│ 2. Request bypass (if allowed│
└──────────────────────────────┘
```

### Configuring Push Protection

**Enablement Levels:**

```yaml
Repository:
  Settings → Code security → Secret scanning
  ├─ Enable secret scanning ✅
  └─ Enable push protection ✅

Organization:
  Settings → Code security → Secret scanning
  ├─ Enable for all repositories
  ├─ Enable for new repositories
  └─ Enable push protection

Enterprise:
  Settings → Policies → Advanced Security
  └─ Push protection policy for all orgs
```

**Bypass Options:**

```yaml
Bypass settings:
  - allow_bypass: true/false
  - require_reason: true/false
  - require_approval: true/false (Enterprise only)
  - bypass_expires: 7days/30days/never
  
Delegated bypass (Enterprise):
  - designated_reviewers:
      - security-team
      - @octocat
  - approval_required: true
  - auto_dismiss_after: 7days
```

### Bypass Workflow

**When a developer needs a bypass:**

```
Developer encounters blocked push
        ↓
Click "Bypass protection"
        ↓
Provide justification:
  - "Testing vulnerability fix"
  - "False positive - not real secret"
  - "Legacy code - will fix in separate PR"
        ↓
    ┌───┴────┐
    │ Policy │
    └───┬────┘
         │
   ┌─────┴─────┐
Auto-approve  Require approval
   │              │
   ▼              ▼
Push allowed   Pending review
                   │
              ┌────┴─────┐
           Approved   Denied
              │          │
              ▼          ▼
         Push allowed  Push blocked
```

**Audit Trail:**

Every bypass is recorded:

```json
{
  "event": "secret_scanning.push_protection_bypass",
  "actor": "developer@company.com",
  "repository": "company/api",
  "commit_sha": "abc123...",
  "secret_type": "github_pat",
  "bypass_reason": "False positive - test token",
  "approved_by": "security@company.com",
  "timestamp": "2026-04-27T10:30:00Z"
}
```

### Developer Experience

**Without Push Protection:**
```bash
$ git push origin main
Enumerating objects: 5, done.
Counting objects: 100% (5/5), done.
Writing objects: 100% (3/3), 289 bytes | 289.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0)
To github.com:company/api.git
   abc123..def456  main -> main

# ⚠️ Secret is in the repository
# ⚠️ Alert appears 30 seconds later
```

**With Push Protection:**
```bash
$ git push origin main
Enumerating objects: 5, done.
Counting objects: 100% (5/5), done.
Writing objects: 100% (3/3), 289 bytes | 289.00 KiB/s, done.
remote: 
remote: ❌ PUSH REJECTED due to secret found
remote: 
remote: Secret scanning found the following secret(s):
remote: 
remote:   Locations:
remote:     src/config.js:15
remote: 
remote:   Type: GitHub Personal Access Token
remote:   Status: Active ✅
remote: 
remote: ⚠️  This token is currently active and can be used.
remote:    You should revoke it immediately.
remote: 
remote: To push anyway, remove the secret or:
remote:   • Request bypass (requires justification)
remote:   • Contact your administrator
remote: 
remote: Learn more:
remote:   https://docs.github.com/secret-scanning
remote: 
To github.com:company/api.git
 ! [remote rejected] main -> main (push declined due to secret)
error: failed to push some refs to 'github.com:company/api.git'

# ✅ Secret is NOT in the repository
# ✅ Developer can act BEFORE exposure
```

### Use Cases and Best Practices

**Scenarios where push protection is critical:**

1. **Cloud Services (AWS, Azure, GCP)**
   - Keys have access to expensive resources.
   - Exploitation can result in cryptomining.
   - Bills of $10k+ in 24 hours.

2. **Payment Processors (Stripe, PayPal)**
   - Access to financial transactions.
   - PCI-DSS compliance requirements.
   - Legal and reputational risk.

3. **Database Credentials**
   - Access to customer PII.
   - GDPR compliance.
   - Data breach notifications.

4. **Third-party APIs**
   - Quota exhaustion.
   - Account suspension.
   - Service disruption.

**Best Practices:**

```yaml
✅ DO:
  - Enable push protection in all repositories
  - Require bypass justification
  - Set up delegated bypass for sensitive repositories
  - Educate developers on secrets management
  - Use secret managers (Vault, AWS Secrets Manager)
  - Rotate secrets regularly
  - Monitor bypass patterns

❌ DON'T:
  - Allow bypasses without approval for production repositories
  - Ignore push protection alerts
  - Hardcode secrets "temporarily"
  - Use comments as an excuse for bypass
  - Disable push protection for "convenience"
```

**Integration with Secret Managers:**

```javascript
// ❌ INCORRECT - Hardcoded
const apiKey = "sk_live_1234567890";

// ✅ CORRECT - Secret manager
const apiKey = await secretManager.getSecret('stripe_api_key');

// ✅ CORRECT - Environment variables
const apiKey = process.env.STRIPE_API_KEY;

// ✅ CORRECT - GitHub Secrets (Actions)
// In workflow:
# ${{ secrets.STRIPE_API_KEY }}
```

### Limitations of Push Protection

**It does not protect against:**
- ❌ Secrets in binary files (images, PDFs)
- ❌ Secrets obfuscated intentionally
- ❌ Secrets in unmonitored private repositories
- ❌ Secrets shared verbally or via email
- ❌ Secrets in wikis, issues, discussions

**Necessary Workarounds:**
- Local pre-commit hooks
- IDE plugins (VS Code extension)
- Git hooks on workstation
- Security awareness training

**Links:**
- https://docs.github.com/en/code-security/secret-scanning/introduction/about-push-protection
- https://docs.github.com/en/code-security/secret-scanning/using-advanced-secret-scanning-and-push-protection-features/delegated-bypass-for-push-protection

---
<a id="d2-3"></a>
## 2.3 Secret Scanning availability by repository type

### Public Repositories

**Secret scanning: ✅ FREE (Enabled by default)**

Included Features:
- ✅ Automatic scanning of the entire history
- ✅ 200+ partner patterns
- ✅ Validity checks with providers
- ✅ Notifications to service providers
- ✅ Automatic revocation (some providers)
- ✅ Push protection (NEW since 2025)
- ❌ Custom patterns (not available)
- ❌ Security overview (not available)
- ❌ Delegated bypass (not available)

**Reason**: Protect the open source ecosystem and prevent leaked credentials.

### Private Repositories WITHOUT GHAS

**Secret scanning: ❌ NOT AVAILABLE**

To enable, you need:
- **GitHub Secret Protection** ($19/month per active committer), or
- **GitHub Enterprise** (legacy bundle)

Without license:
- ❌ No secrets scanning
- ❌ No push protection
- ❌ No alerts
- ⚠️ Risk: secrets can be exposed without detection

### Private Repositories WITH GHAS (GitHub Secret Protection)

**Secret scanning: ✅ COMPLETE**

Additional Features:
- ✅ Private repository scanning
- ✅ Push protection
- ✅ Custom patterns (organization-level)
- ✅ Delegated bypass workflows
- ✅ Delegated alert dismissal
- ✅ Security overview
- ✅ Security campaigns
- ✅ Copilot secret scanning (AI-powered)
- ✅ Advanced analytics

**Complete Comparison:**

| Feature | Public | Private without GHAS | Private with Secret Protection |
|---------|---------|------------------|-------------------------------|
| Repository scanning | ✅ | ❌ | ✅ |
| Partner patterns | ✅ (200+) | ❌ | ✅ (200+) |
| Push protection | ✅ | ❌ | ✅ |
| Validity checks | ✅ | ❌ | ✅ |
| Provider notifications | ✅ | ❌ | ✅ |
| Custom patterns | ❌ | ❌ | ✅ |
| Delegated bypass | ❌ | ❌ | ✅ |
| Copilot scanning | ❌ | ❌ | ✅ |
| Security overview | ❌ | ❌ | ✅ |
| Security campaigns | ❌ | ❌ | ✅ |
| Alert dismissal workflow | Basic | ❌ | Advanced |
| Audit logs | Basic | ❌ | Complete |
| API access | Basic | ❌ | Full |
| Webhooks | Basic | ❌ | Full |

### Enablement by Level

**At Repository Level:**
```yaml
Settings → Code security and analysis
  └─ Secret scanning
      ├─ [✓] Secret scanning (Free for public, GHAS for private)
      └─ [✓] Push protection (Free for public, GHAS for private)
```

**At Organization Level:**
```yaml
Settings → Code security and analysis
  ├─ Enable for all existing repositories
  ├─ Enable for new repositories
  └─ Configure default settings
      ├─ Push protection: Enabled
      ├─ Bypass allowed: Require justification
      └─ Custom patterns: [Add patterns]
```

**At Enterprise Level:**
```yaml
Policies → Advanced Security
  ├─ Enforce for all organizations
  ├─ Allow organizations to override
  └─ Billing (per active committer)
```

### Special Cases
**Forked Repositories:**
- Public fork of public: ✅ Secret scanning enabled
- Private fork of private: ⚠️ Depends on parent's GHAS license
- Private fork of public: ❌ Requires GHAS license

**Archived Repositories:**
- ✅ Secret scanning remains active
- ❌ No new alerts generated (no new commits)
- ℹ️ Existing alerts remain visible

**Template Repositories:**
- Settings are copied to repositories created from templates.
- GHAS is required in each repository, it is not inherited.

**Mirrored Repositories:**
- ✅ Secret scanning works on mirrors.
- ⚠️ Alerts are created in the mirrored repository.
- ℹ️ Push protection applies to the mirror, not the origin.

**Links:**
- https://docs.github.com/en/get-started/learning-about-github/about-github-advanced-security#about-advanced-security-features

---
<a id="d2-4"></a>
## 2.4 Enable Secret Scanning for private repositories

### Prerequisites

**Required Licenses:**
- ✅ GitHub Secret Protection ($19/active committer/month), or
- ✅ GitHub Enterprise (legacy bundle)

**Required Permissions:**
- Repository: Admin role
- Organization: Owner or Security manager
- Enterprise: Enterprise owner

**Eligibility Verification:**

```bash
# Via GitHub CLI
gh api /repos/:owner/:repo/vulnerability-alerts

# Response:
{
  "enabled": false,
  "reason": "Advanced Security not enabled"
}

# Check billing
gh api /orgs/:org/settings/billing/advanced-security

# Response:
{
  "total_seats_purchased": 50,
  "total_seats_used": 32,
  "total_seats_available": 18
}
```

### Step-by-Step Enablement

#### Method 1: Via Web UI (Individual Repository)

**Step 1**: Navigate to Settings
```
Repository → Settings tab
```

**Step 2**: Go to Security
```
Sidebar → Code security and analysis
```

**Step 3**: Enable Advanced Security
```
[ ] GitHub Advanced Security
    └─ [Enable] ← Click here first
```

**Step 4**: Enable Secret Scanning
```
[✓] GitHub Advanced Security (now enabled)
    ├─ [ ] Secret scanning
    │   └─ [Enable] ← Click to enable
    └─ [ ] Push protection
        └─ [Enable] ← Optional but recommended
```

**Step 5**: Wait for initial scan
```
⏳ Scanning repository history...
   Commits scanned: 1,234 / 5,678
   ETA: 2 minutes

✅ Initial scan complete
   0 secrets found
```

#### Method 2: Via API (Programmatic)

**Enable GHAS:**
```bash
curl -X PATCH \
  https://api.github.com/repos/OWNER/REPO \
  -H "Authorization: token $GITHUB_TOKEN" \
  -H "Accept: application/vnd.github+json" \
  -d '{
    "security_and_analysis": {
      "advanced_security": {
        "status": "enabled"
      },
      "secret_scanning": {
        "status": "enabled"
      },
      "secret_scanning_push_protection": {
        "status": "enabled"
      }
    }
  }'
```

**Verify Status:**
```bash
curl \
  https://api.github.com/repos/OWNER/REPO \
  -H "Authorization: token $GITHUB_TOKEN" | \
  jq '.security_and_analysis'

# Response:
{
  "advanced_security": {
    "status": "enabled"
  },
  "secret_scanning": {
    "status": "enabled"
  },
  "secret_scanning_push_protection": {
    "status": "enabled"
  }
}
```

#### Method 3: Via GitHub CLI

```bash
# Enable GHAS + Secret Scanning
gh api -X PATCH /repos/:owner/:repo \
  -f security_and_analysis[advanced_security][status]=enabled \
  -f security_and_analysis[secret_scanning][status]=enabled \
  -f security_and_analysis[secret_scanning_push_protection][status]=enabled

# Verify
gh api /repos/:owner/:repo | jq '.security_and_analysis'
```

#### Method 4: Bulk Enablement (Organization)

**Via UI:**
```
Organization Settings
  → Code security and analysis
  → Configure security and analysis features
      ├─ [Enable all] ← Enable for all repositories
      └─ [✓] Automatically enable for new repositories
```

**Via Script (Python):**
```python
import requests

ORG = "my-org"
TOKEN = "ghp_..."
HEADERS = {
    "Authorization": f"token {TOKEN}",
    "Accept": "application/vnd.github+json"
}

# Get all repos
repos = requests.get(
    f"https://api.github.com/orgs/{ORG}/repos",
    headers=HEADERS,
    params={"per_page": 100}
).json()

for repo in repos:
    repo_name = repo["full_name"]
    
    # Enable GHAS + Secret Scanning
    response = requests.patch(
        f"https://api.github.com/repos/{repo_name}",
        headers=HEADERS,
        json={
            "security_and_analysis": {
                "advanced_security": {"status": "enabled"},
                "secret_scanning": {"status": "enabled"},
                "secret_scanning_push_protection": {"status": "enabled"}
            }
        }
    )
    
    if response.status_code == 200:
        print(f"✅ {repo_name}: Secret scanning enabled")
    else:
        print(f"❌ {repo_name}: {response.json()['message']}")
```

### Post-Enablement Configuration

**1. Configure Notifications:**

```
Settings → Notifications
  └─ Security alerts
      ├─ [✓] Email notifications
      │   └─ security@company.com
      ├─ [✓] Web notifications
      └─ [✓] Slack integration
          └─ #security-alerts
```

**2. Configure Custom Patterns (Optional):**

```
Settings → Code security → Secret scanning
  → Custom patterns
      → [New pattern]
          ├─ Name: Internal API Key
          ├─ Secret format: INT-[A-Z0-9]{32}
          ├─ Test string: INT-ABC123...
          └─ [Save pattern]
```

**3. Configure Bypass Policies:**

```
Organization Settings → Code security
  → Secret scanning
      → Push protection
          ├─ [✓] Allow bypasses
          ├─ [✓] Require bypass reason
          ├─ [ ] Require approval (Enterprise only)
          └─ Bypass expires: [7 days ▼]
```

**4. Configure CODEOWNERS for Alerts:**

```bash
# .github/CODEOWNERS
# Security team owns all secret scanning alerts

* @org/developers
/.github/workflows/* @org/devops
**/secrets.yml @org/security
**/config*.* @org/security
```

### Common Troubleshooting

**Problem 1**: "Advanced Security not available"
```
Cause: No GHAS licenses available
Solution: Buy more seats or release unused seats

# Check usage:
gh api /orgs/:org/settings/billing/advanced-security
```

**Problem 2**: "Secret scanning failed to start"
```
Cause: Repository is a fork without GHAS enabled on the parent
Solution: Enable GHAS on the parent repository or detach the fork

# Detach fork:
# Settings → Danger Zone → "Detach fork"
```

**Problem 3**: "No scan results after 1 hour"
```
Cause: Very large repository or many commits
Solution: Wait longer or contact GitHub Support

# Monitor status:
gh api /repos/:owner/:repo/code-scanning/analyses | \
  jq '.[0].created_at'
```

**Problem 4**: "Push protection not working"
```
Cause: Feature not enabled or bypass configured
Solution:
# Verify:
gh api /repos/:owner/:repo | \
  jq '.security_and_analysis.secret_scanning_push_protection.status'

# Should be: "enabled"
```

### Best Practices

```yaml
✅ Rollout Strategy:
  1. Pilot with 5-10 non-critical repositories
  2. Evaluate false positives
  3. Adjust custom patterns
  4. Expand to 50% of repositories
  5. Enable push protection
  6. Full rollout to 100%

✅ Team Enablement:
  - Training session on secrets management
  - Document workflows
  - Runbooks for common scenarios
  - Regular retros of alerts

✅ Monitoring:
  - Weekly reports of:
      - New alerts
      - Resolved alerts
      - Bypass requests
      - False positive rate
  - Dashboard with key metrics
  - Alerting for critical findings

❌ Avoid:
  - Enabling in production without testing
  - Not configuring notifications
  - Ignoring alerts due to "alert fatigue"
  - Failing to train the team
  - Disabling push protection "temporarily"
```

**Links:**
- https://docs.github.com/en/code-security/secret-scanning/configuring-secret-scanning-for-your-repositories
- https://docs.github.com/en/code-security/secret-scanning/introduction/about-push-protection

---
<a id="d2-5"></a>
## 2.5 Appropriate responses to Secret Scanning alerts

### Decision Workflow

```
Secret scanning alert appears
        ↓
[1] EVALUATE VALIDITY
        ├─ Active → 🔴 CRITICAL
        ├─ Inactive → 🟡 MEDIUM
        └─ Unknown → 🟠 HIGH
        ↓
[2] VERIFY EXPOSURE
        ├─ How long exposed?
        ├─ Public or private repository?
        ├─ Who has access?
        └─ Was the secret used?
        ↓
[3] EVALUATE IMPACT
        ├─ What resources does it protect?
        ├─ What is the blast radius?
        ├─ Are sensitive data accessible?
        └─ Compliance implications?
        ↓
[4] DECIDE ACTION
```

### Decision Matrix

| Validity | Repo Type | Time Exposed | Action |
|---------|--------------|-----------------|--------|
| **Active** | Public | Any | 🚨 EMERGENCY |
| **Active** | Private | >24h | 🔴 URGENT |
| **Active** | Private | <24h | 🟠 HIGH |
| **Inactive** | Any | Any | 🟡 MEDIUM |
| **Unknown** | Public | >7d | 🔴 URGENT |
| **Unknown** | Private | Any | 🟠 HIGH |

### Actions by Secret Type

#### A) GitHub Personal Access Token

**If Active:**

```bash
# 1. REVOKE IMMEDIATELY
https://github.com/settings/tokens
   → Locate token
   → [Delete] or [Revoke]

# 2. VERIFY USE
gh api /user/events | jq '.[] | select(.created_at > "2026-04-26")'
# Review:
# - Access IPs
# - Repositories accessed
# - Actions performed

# 3. ROTATE
# Create new token with minimum scopes required
gh auth login --scopes repo,read:org

# 4. UPDATE DEPENDENCIES
# CI/CD pipelines
# GitHub Actions secrets
# Applications using the token

# 5. CLEAN CODE
git filter-repo --invert-paths --path config.js
git push --force

# 6. DOCUMENT INCIDENT
# Create postmortem:
# - Exposure timeline
# - Compromise scope
# - Actions taken
# - Prevention measures
```

**If Inactive:**

```bash
# 1. VERIFY REVOCATION
# Confirm the token no longer works

# 2. CLEAN CODE
# Remove references to the token
git rm config/secrets.js
git commit -m "Remove revoked GitHub token"

# 3. DISMISS ALERT
# In GitHub UI:
# Reason: "Won't fix - token already revoked"
# Comment: "Token was revoked on 2026-04-20"
```

#### B) AWS Access Key

**If Active:**

```bash
# 1. REVOKE IMMEDIATELY
aws iam delete-access-key \
  --access-key-id AKIA... \
  --user-name compromised-user

# 2. AUDIT AWS CloudTrail
aws cloudtrail lookup-events \
  --lookup-attributes AttributeKey=Username,AttributeValue=compromised-user \
  --start-time 2026-04-01 \
  --end-time 2026-04-27 \
  > cloudtrail-audit.json

# Look for:
# - EC2 instances launched
# - S3 buckets accessed
# - IAM changes
# - Unusual regions/IPs

# 3. VERIFY UNAUTHORIZED RESOURCES
# EC2 cryptomining instances
aws ec2 describe-instances --filters "Name=key-name,Values=*"

# Exposed S3 buckets
aws s3api list-buckets

# Suspicious Lambda functions
aws lambda list-functions

# 4. ROTATE CREDENTIALS
aws iam create-access-key --user-name production-app

# 5. APPLY LEAST PRIVILEGE
aws iam attach-user-policy \
  --user-name production-app \
  --policy-arn arn:aws:iam::aws:policy/ReadOnlyAccess

# 6. ENABLE MFA
aws iam enable-mfa-device \
  --user-name production-app \
  --serial-number arn:aws:iam::123456:mfa/app \
  --authentication-code1 123456 \
  --authentication-code2 789012

# 7. CONFIGURE ALERTS
# CloudWatch alarm for unauthorized API calls
aws cloudwatch put-metric-alarm \
  --alarm-name UnauthorizedAPICalls \
  --alarm-actions arn:aws:sns:us-east-1:123456:security-alerts

# 8. BILLING REVIEW
# Check for unexpected charges
aws ce get-cost-and-usage \
  --time-period Start=2026-04-01,End=2026-04-27 \
  --granularity DAILY \
  --metrics UnblendedCost
```

**Real Case - Cryptomining:**

```bash
# Scenario: AWS key leaked, used for cryptomining

# 1. Detected:
#    - CloudWatch: EC2 CPU 100% in us-west-2
#    - 50 c5.24xlarge instances (!!!!)
#    - Cost: $120/hour = $86,400/month

# 2. Response:
# Terminate all suspicious instances
aws ec2 terminate-instances \
  --instance-ids $(aws ec2 describe-instances \
    --filters "Name=instance-state-name,Values=running" \
    --query "Reservations[*].Instances[?LaunchTime>'2026-04-26'][].InstanceId" \
    --output text)

# 3. Revoke key
aws iam delete-access-key --access-key-id AKIA...

# 4. Contact AWS Support
# Request billing adjustment (goodwill credit)

# 5. Enable AWS GuardDuty
aws guardduty create-detector --enable

# 6. Setup billing alerts
aws budgets create-budget \
  --budget file://budget.json \
  --notifications-with-subscribers file://alerts.json
```

#### C) Database Connection String

**If Active:**

```sql
-- 1. VERIFY ACTIVE CONNECTIONS
SELECT pid, usename, application_name, client_addr, backend_start
FROM pg_stat_activity
WHERE usename = 'compromised_user'
ORDER BY backend_start DESC;

-- 2. KILL SUSPICIOUS SESSIONS
SELECT pg_terminate_backend(pid)
FROM pg_stat_activity
WHERE usename = 'compromised_user'
  AND client_addr NOT IN ('10.0.1.5', '10.0.1.6');

-- 3. CHANGE PASSWORD
ALTER USER compromised_user WITH PASSWORD 'new_secure_password_12345!';

-- 4. REVIEW AUDIT LOGS
SELECT * FROM pg_stat_statements
WHERE userid = (SELECT oid FROM pg_user WHERE usename = 'compromised_user')
ORDER BY calls DESC
LIMIT 100;

-- 5. CHECK FOR DATA EXFILTRATION
-- Queries with large resultsets
SELECT query, calls, total_time, rows
FROM pg_stat_statements
WHERE rows > 10000
ORDER BY total_time DESC;

-- 6. ROTATE CREDENTIALS IN APPLICATIONS
-- Update connection strings in:
# - Kubernetes secrets
# - AWS Secrets Manager
# - Environment variables
# - Application configs

-- 7. RESTRICT ACCESS
-- Limit by IP
-- pg_hba.conf:
host    dbname    username    10.0.1.0/24    md5
host    dbname    username    0.0.0.0/0      reject

-- 8. ENABLE SSL/TLS REQUIRED
ALTER USER compromised_user SET ssl TO on;

-- 9. AUDIT SCHEMA CHANGES
SELECT schemaname, tablename, usename, query_start
FROM pg_stat_activity
WHERE query LIKE '%ALTER%' OR query LIKE '%DROP%';
```

#### D) API Keys (Stripe, Twilio, etc.)

**If Active:**

```javascript
// 1. REVOKE KEY
// Stripe Dashboard:
// Developers → API keys → [Reveal] → [Roll key]

// 2. REVIEW API CALLS
const stripe = require('stripe')('sk_live_...');

const charges = await stripe.charges.list({
  created: {
    gte: Math.floor(Date.parse('2026-04-26') / 1000)
  },
  limit: 100
});

// Look for:
// - Unauthorized charges
// - Refunds needed
// - Unusual patterns

// 3. ALERT CUSTOMERS (if needed)
// If customer data was accessed:
// - Email notification
// - Credit monitoring offer
// - Incident report

// 4. UPDATE APPLICATIONS
// Kubernetes:
kubectl create secret generic stripe-secret \
  --from-literal=api-key=sk_live_NEW_KEY \
  --dry-run=client -o yaml | kubectl apply -f -

// Restart pods to pick up new secret
kubectl rollout restart deployment/payment-service

// 5. IMPLEMENT KEY ROTATION
// Set up automatic rotation
// AWS Secrets Manager with Lambda:
exports.handler = async (event) => {
  const newKey = await rotateStripeKey();
  await updateSecret('stripe-api-key', newKey);
  await notifySlack('#security', 'Stripe key rotated');
};

// 6. ENABLE WEBHOOK SIGNATURE VERIFICATION
// stripe-webhook.js
const sig = request.headers['stripe-signature'];
let event;
try {
  event = stripe.webhooks.constructEvent(
    request.body,
    sig,
    process.env.STRIPE_WEBHOOK_SECRET
  );
} catch (err) {
  return response.status(400).send(`Webhook Error: ${err.message}`);
}

// 7. SETUP MONITORING
// CloudWatch/Datadog alerts for:
// - Unusual charge volumes
// - Failed charges spike
// - API errors increase
// - Charges from new countries
```

### False Positives

**How to identify:**

```yaml
False positive indicators:
  - Secret in a test file: *_test.py, *_spec.js
  - Secret in an explanatory comment
  - Secret in documentation (README, docs/)
  - Secret is an example/placeholder: "your-api-key-here"
  - Secret is a hash/checksum, not a credential
  - Pattern match but not a real secret
```

**How to handle:**

```bash
# 1. VERIFY it is a false positive
# DO NOT ASSUME without verification

# 2. DISMISS ALERT
# GitHub UI → Alert → Dismiss
# Reason: "False positive"
# Comment: "This is a test fixture, not a real API key. File: tests/fixtures/sample.json"

# 3. PREVENT RECURRENCE
# Option A: Exclude path from scanning
# .github/secret_scanning.yml
paths-ignore:
  - 'tests/**'
  - 'test/**'
  - '__tests__/**'
  - '**/*.test.js'
  - '**/*.spec.ts'
  - 'docs/**'
  - '**/*.md'

# Option B: Comment in code
# Some scanners respect:
# secretlint-disable-next-line
API_KEY = "example_key_12345"

# Option C: Use placeholder values
# Good:
API_KEY = "sk_test_YOUR_KEY_HERE"
# Bad:
API_KEY = "sk_test_4eC39HqLyjWDarjtT1zdp7dc"  # ← Looks real!
```

### Automated Workflow with GitHub Actions

```yaml
# .github/workflows/secret-alert-handler.yml
name: Secret Scanning Alert Handler

on:
  secret_scanning_alert:
    types: [created]

jobs:
  handle-alert:
    runs-on: ubuntu-latest
    steps:
      - name: Get alert details
        id: alert
        uses: actions/github-script@v7
        with:
          script: |
            const alert = context.payload.alert;
            return {
              secret_type: alert.secret_type,
              validity: alert.validity,
              created_at: alert.created_at
            };

      - name: Post to Slack
        if: steps.alert.outputs.validity == 'active'
        uses: slackapi/slack-github-action@v1
        with:
          channel-id: 'C01234567'
          slack-message: |
            🚨 CRITICAL: Active secret detected!
            Type: ${{ steps.alert.outputs.secret_type }}
            Repo: ${{ github.repository }}
            Action required: Revoke immediately
        env:
          SLACK_BOT_TOKEN: ${{ secrets.SLACK_BOT_TOKEN }}

      - name: Create PagerDuty incident
        if: |
          steps.alert.outputs.validity == 'active' &&
          contains(steps.alert.outputs.secret_type, 'aws')
        uses: peter-murray/pagerduty-incident-action@v1
        with:
          pagerduty-token: ${{ secrets.PAGERDUTY_TOKEN }}
          incident-title: "AWS credentials leaked in ${{ github.repository }}"
          incident-urgency: "high"

      - name: Auto-dismiss if test file
        if: contains(github.event.alert.locations[0].path, 'test')
        uses: actions/github-script@v7
        with:
          script: |
            await github.rest.secretScanning.updateAlert({
              owner: context.repo.owner,
              repo: context.repo.name,
              alert_number: context.payload.alert.number,
              state: 'resolved',
              resolution: 'false_positive',
              resolution_comment: 'Auto-dismissed: found in test file'
            });
```

### Recommended SLAs

| Severity | Validity | Repo Type | SLA for Resolution |
|-----------|---------|-----------|---------------------|
| Critical | Active | Public | 1 hour |
| Critical | Active | Private | 4 hours |
| High | Active | Any | 24 hours |
| High | Unknown | Public | 24 hours |
| Medium | Inactive | Any | 7 days |
| Medium | Unknown | Private | 7 days |
| Low | Inactive | Private | 30 days |

### Response Checklist

```markdown
## Secret Scanning Alert Response Checklist

### Immediate Actions (0-1 hour)
- [ ] Verify alert authenticity (not false positive)
- [ ] Check validity status (active/inactive/unknown)
- [ ] Assess repository exposure (public/private/internal)
- [ ] Identify secret type and associated resources
- [ ] Escalate to security team if critical

### Short-term Actions (1-24 hours)
- [ ] Revoke/rotate compromised secret
- [ ] Audit logs for unauthorized access
- [ ] Identify all services using the secret
- [ ] Update secret in all locations
- [ ] Test applications after rotation
- [ ] Notify stakeholders if needed

### Medium-term Actions (1-7 days)
- [ ] Review security posture of affected resources
- [ ] Implement additional monitoring
- [ ] Clean secret from git history
- [ ] Update documentation and runbooks
- [ ] Conduct team training if needed

### Long-term Actions (ongoing)
- [ ] Implement secret management solution (Vault, etc.)
- [ ] Set up automated secret rotation
- [ ] Enable push protection if not enabled
- [ ] Review and update custom patterns
- [ ] Quarterly audit of all secrets
- [ ] Document lessons learned
```

**Links:**
- https://docs.github.com/en/code-security/secret-scanning/managing-alerts-from-secret-scanning
- https://docs.github.com/en/code-security/secret-scanning/secret-scanning-partnership-program

---
<a id="d2-6"></a>
## 2.6 Customize the behavior of Secret Scanning

### Configure Alert Recipients

**At Repository Level:**

```
Settings → Code security and analysis
  → Secret scanning
    → Alert notifications
        ├─ [✓] Email notifications
        │   └─ Recipients:
        │       ├─ Repository administrators (default)
        │       ├─ Security managers
        │       └─ Custom: security@company.com
        ├─ [✓] Web notifications
        └─ [✓] Integrations
            ├─ Slack: #security-alerts
            ├─ PagerDuty: Security-Oncall
            └─ Webhook: https://api.company.com/security/webhooks
```

**Advanced Granular Access Control:**

```yaml
# Via GitHub API
# Grant read access to non-admin team

PUT /repos/:owner/:repo/teams/:team_slug
{
  "permission": "pull"  # read access
}

# Grant secret scanning access specifically
PUT /repos/:owner/:repo/teams/:team_slug/security-managers
{
  "team_id": 12345
}

# Via GitHub CLI
gh api \
  --method PUT \
  -H "Accept: application/vnd.github+json" \
  /repos/OWNER/REPO/collaborators/USERNAME \
  -f permission='maintain'  # Can view and dismiss alerts
```

**Detailed Roles and Permissions:**

| Action | Read | Triage | Write | Maintain | Admin | Security Manager |
|--------|------|--------|-------|----------|-------|------------------|
| View alerts | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ |
| Comment on alerts | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ |
| Dismiss alerts | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ |
| Reopen alerts | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ |
| Configure scanning | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ |
| View audit log | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ |

**Advanced Notification Routing:**

```javascript
// GitHub App / Webhook handler
// Route alerts based on secret type

const routingRules = {
  github_pat: {
    notify: ['#eng-leads', 'security@company.com'],
    severity: 'high',
    oncall: false
  },
  aws_access_key: {
    notify: ['#cloud-team', '#security-oncall'],
    severity: 'critical',
    oncall: true,
    pagerduty: 'cloud-security'
  },
  stripe_api_key: {
    notify: ['#payments-team', 'cfo@company.com'],
    severity: 'critical',
    oncall: true
  },
  generic_password: {
    notify: ['#security-alerts'],
    severity: 'medium',
    oncall: false
  }
};

app.post('/webhooks/secret-scanning', async (req, res) => {
  const alert = req.body.alert;
  const routing = routingRules[alert.secret_type] || routingRules.generic_password;
  
  // Send to Slack
  for (const channel of routing.notify) {
    await slack.post(channel, {
      text: `🔐 Secret detected: ${alert.secret_type}`,
      severity: routing.severity,
      repo: alert.repository.full_name,
      validity: alert.validity
    });
  }
  
  // Page oncall if critical
  if (routing.oncall && alert.validity === 'active') {
    await pagerduty.createIncident({
      title: `Active ${alert.secret_type} leaked`,
      service: routing.pagerduty,
      urgency: 'high'
    });
  }
  
  res.sendStatus(200);
});
```

### Exclude Files from Scanning

**Option 1: .gitignore (Not Sufficient)**

```bash
# ⚠️ .gitignore DOES NOT affect secret scanning
# Secret scanning scans the ENTIRE history, including ignored files

# .gitignore
secrets.json
.env
```

**Option 2: Path Exclusions (Repository level)**

```yaml
# .github/secret_scanning.yml
# Exclude specific paths

paths-ignore:
  - 'tests/**'
  - 'test/**'
  - '__tests__/**'
  - '**/*.test.js'
  - '**/*.spec.ts'
  - 'docs/**'
  - '**/*.md'
  - 'examples/**'
  - 'node_modules/**'  # Usually auto-ignored
  - 'vendor/**'
  - '.github/workflows/**'  # If using sample secrets in workflows
```

**Option 3: File-level Exclusions (Custom patterns)**

```yaml
# Exclude specific files by name
# Settings → Secret scanning → Custom patterns

# Pattern name: Internal test fixture
# Pattern:  TEST_API_KEY_[A-Z0-9]{32}
# Exclude paths:
#   - tests/fixtures/sample.json
#   - tests/integration/*.test.js
```

**Option 4: Inline Comments (Limited support)**

```javascript
// Some patterns can be suppressed with comments
// NOT ALL SCANNERS support this

// gitleaks:ignore
const API_KEY = "test_key_12345";

// secret-scanner-ignore
const PASSWORD = "example_password";

// ⚠️ This DOES NOT work for native GitHub secret scanning
// Only works for some third-party tools
```

**Option 5: Organization-level Exclusions**

```yaml
# Organization Settings → Secret scanning
# → Path exclusions (applies to all repos)

Global exclusions:
  - '**/test/**'
  - '**/tests/**'
  - '**/__tests__/**'
  - '**/*.test.*'
  - '**/*.spec.*'
  - '**/examples/**'
  - '**/docs/**'
```

**Best Practices for Exclusions:**

```yaml
✅ EXCLUDE:
  - Test fixtures with fake data
  - Documentation with examples
  - Vendored dependencies (already scanned)
  - Generated files (build artifacts)

❌ DO NOT EXCLUDE:
  - Production code
  - Configuration files
  - Deployment scripts
  - Infrastructure as Code
  - CI/CD workflows (unless only examples)

⚠️ CAUTION:
  - Over-excluding creates blind spots
  - Review exclusions quarterly
  - Document WHY each exclusion exists
```

**Verify Exclusions Effectiveness:**

```bash
# Test if exclusion works
# 1. Commit a fake secret in excluded path
echo "test_key_12345" > tests/fixtures/fake-secret.json
git add tests/fixtures/fake-secret.json
git commit -m "test: Add test fixture"
git push

# 2. Verify NO alert is generated
gh api /repos/:owner/:repo/secret-scanning/alerts | \
  jq '.[] | select(.locations[].path | contains("tests/fixtures"))'

# 3. If alert appears = exclusion failed
# 4. If NO alert appears = exclusion effective ✅
```

### Enable Custom Patterns

**Why custom patterns?**

GitHub patterns cover 200+ common secret types, but DO NOT cover:
- ❌ Internal API keys of proprietary systems
- ❌ Legacy authentication schemes
- ❌ Custom encryption keys
- ❌ Organization-specific token formats

**Requirements:**

- ✅ GitHub Advanced Security (Secret Protection)
- ✅ Organization owner or Security manager role
- ✅ Knowledge of regex patterns

**Custom Pattern Creation Step-by-Step:**

**Step 1**: Identify the pattern

```regex
# Example: Internal API key
# Format: ACME-[PROJECT]-[A-Z0-9]{32}
# Examples:
#   ACME-WEB-ABC123DEF456GHI789JKL012MNO345PQ
#   ACME-API-XYZ987UVW654TSR321QPO098NML765KJ

# Regex:
ACME-[A-Z]{3,10}-[A-Z0-9]{32}
```

**Step 2**: Test the pattern

```
GitHub UI → Organization Settings
  → Code security and analysis
    → Secret scanning
      → Custom patterns
        → [New pattern]

Pattern details:
  ├─ Name: ACME Internal API Key
  ├─ Secret format (regex):
  │   ACME-[A-Z]{3,10}-[A-Z0-9]{32}
  │
  ├─ Test strings:
  │   ✅ ACME-WEB-ABC123DEF456GHI789JKL012MNO345PQ
  │   ✅ ACME-MOBILE-XYZ987UVW654TSR321QPO098NML76
  │   ❌ ACME-WEB-SHORT
  │   ❌ acme-web-lowercase
  │
  └─ [Publish pattern]
```

**Step 3**: Advanced regex features

```regex
# Before secret (context):
^[\s]*(?:api[_-]?key|token)[\s]*[=:]["']?

# Secret pattern:
ACME-[A-Z]{3,10}-[A-Z0-9]{32}

# After secret (context):
["']?[\s]*$

# Full pattern:
(?:api[_-]?key|token)[\s]*[=:]["']?ACME-[A-Z]{3,10}-[A-Z0-9]{32}["']?

# Matches:
# ✅ API_KEY="ACME-WEB-ABC..."
# ✅ token: 'ACME-API-XYZ...'
# ✅ apiKey = "ACME-MOBILE-..."
# ❌ Just "ACME-WEB-..." by itself (no context)
```

**Step 4**: Dry run mode

```yaml
# Start with "alert only" mode
# Don't enable push protection yet

Pattern settings:
  ├─ Push protection: [ ] Disabled (first)
  ├─ Alert on matches: [✓] Enabled
  └─ Test period: 2 weeks

After 2 weeks:
  ├─ Review all alerts
  ├─ Calculate false positive rate
  ├─ If FP < 10%: Enable push protection
  └─ If FP > 10%: Refine pattern
```

**Useful Custom Patterns Examples:**

**1. Database connection strings:**
```regex
# Pattern:
(postgres|mysql|mongodb)://[a-zA-Z0-9_-]+:[^@\s]+@[^/\s]+

# Matches:
# postgres://user:password@host:5432/db
# mysql://admin:secret@192.168.1.1:3306/mydb
# mongodb://dbuser:dbpass@mongo.example.com:27017/prod
```

**2. JWT tokens:**
```regex
# Pattern:
eyJ[A-Za-z0-9_-]{10,}\.eyJ[A-Za-z0-9_-]{10,}\.[A-Za-z0-9_-]{10,}

# Matches:
# eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIx...
```

**3. Private keys:**
```regex
# Pattern:
-----BEGIN (RSA |EC )?PRIVATE KEY-----[^-]*-----END (RSA |EC )?PRIVATE KEY-----

# Matches:
# -----BEGIN RSA PRIVATE KEY-----
# MIIEpAIBAAKCAQEA...
# -----END RSA PRIVATE KEY-----
```

**4. API keys with checksums:**
```regex
# Pattern:
sk_[a-z]{4}_[a-zA-Z0-9]{24,99}

# Matches Stripe keys:
# sk_live_51H7...(varies)
# sk_test_51H7...(varies)
```

**Pattern Testing Workflow:**

```python
# test_custom_patterns.py
import re

def test_pattern(pattern, test_cases):
    """Test custom pattern against known cases"""
    compiled = re.compile(pattern)
    
    results = {
        'true_positives': [],
        'false_negatives': [],
        'false_positives': []
    }
    
    for case in test_cases:
        match = compiled.search(case['text'])
        
        if case['should_match'] and match:
            results['true_positives'].append(case)
        elif case['should_match'] and not match:
            results['false_negatives'].append(case)
        elif not case['should_match'] and match:
            results['false_positives'].append(case)
    
    return results

# Test cases
ACME_PATTERN = r'ACME-[A-Z]{3,10}-[A-Z0-9]{32}'

test_cases = [
    {
        'text': 'API_KEY="ACME-WEB-ABC123DEF456GHI789JKL012MNO345PQ"',
        'should_match': True,
        'note': 'Valid ACME API key'
    },
    {
        'text': 'const token = "ACME-MOBILE-XYZ987UVW654TSR321QPO098";',
        'should_match': True,
        'note': 'Mobile app token'
    },
    {
        'text': 'ACME-WEB-SHORT',
        'should_match': False,
        'note': 'Too short'
    },
    {
        'text': '<!-- Example: ACME-PROJECT-ABC123DEF456GHI789JKL012MNO345PQ -->',
        'should_match': False,
        'note': 'In comment, likely example'
    }
]

results = test_pattern(ACME_PATTERN, test_cases)

print(f"True Positives: {len(results['true_positives'])}")
print(f"False Negatives: {len(results['false_negatives'])}")
print(f"False Positives: {len(results['false_positives'])}")

# Ideal: TP high, FN low, FP very low
# Acceptable: FP rate < 10%
```

**Managing Custom Patterns at Scale:**

```yaml
# Terraform configuration for custom patterns
# Infrastructure as Code

resource "github_organization_secret_scanning_pattern" "acme_api_key" {
  pattern = "ACME-[A-Z]{3,10}-[A-Z0-9]{32}"
  name    = "ACME Internal API Key"
  
  # Optional: before/after secret for context
  before_secret = "^[\\s]*(?:api[_-]?key|token)[\\s]*[=:]"
  after_secret  = "[\\s]*$"
  
  # Dry run first
  enabled = true
  push_protection_enabled = false  # Enable after testing
}

resource "github_organization_secret_scanning_pattern" "internal_jwt" {
  pattern = "eyJ[A-Za-z0-9_-]{10,}\\.eyJ[A-Za-z0-9_-]{10,}\\.[A-Za-z0-9_-]{10,}"
  name    = "Internal JWT Token"
  enabled = true
}

# Apply:
# terraform apply
# Review:
# terraform plan
```

**Monitoring Custom Patterns:**

```bash
# Get all alerts for custom patterns
gh api /orgs/:org/secret-scanning/alerts \
  --jq '.[] | select(.secret_type_display_name | contains("ACME")) | {
    number,
    secret_type: .secret_type_display_name,
    state,
    resolution,
    created_at,
    repository: .repository.full_name
  }'

# Metrics to track:
# - Total alerts generated
# - False positive rate
# - Time to resolution
# - Bypass requests
# - Developer feedback
```

**Best Practices:**

```yaml
✅ DO:
  - Start specific, broaden gradually
  - Test thoroughly before enabling push protection
  - Document pattern purpose and examples
  - Review quarterly for effectiveness
  - Engage developers in pattern design
  - Monitor false positive rates

❌ DON'T:
  - Create overly broad patterns
  - Enable push protection without testing
  - Ignore developer feedback on false positives
  - Set and forget
  - Copy patterns from internet without testing
```

**Links:**
- https://docs.github.com/en/code-security/secret-scanning/defining-custom-patterns-for-secret-scanning
- https://docs.github.com/en/code-security/secret-scanning/managing-alerts-from-secret-scanning/viewing-alerts
