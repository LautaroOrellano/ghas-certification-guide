> 🌐 **Available in:** **English 🇬🇧** | [Español 🇪🇸](../docs/03-dominio.md)
---

# DOMAIN 3: CONFIGURE AND USE DEPENDABOT AND DEPENDENCY REVIEW (35%)

<a id="d3-1"></a>
## 3.1 Tools for managing vulnerabilities in dependencies

### Dependency Graph

**Definition:**
Visual and analytical representation of all dependencies of a project, including direct and transitive (indirect) dependencies.

**How is it generated?**

```
1. GitHub detects manifest files:
   ├─ package.json, package-lock.json (npm/Node.js)
   ├─ Gemfile, Gemfile.lock (Ruby)
   ├─ requirements.txt, poetry.lock (Python)
   ├─ pom.xml, build.gradle (Java/Maven/Gradle)
   ├─ go.mod, go.sum (Go)
   ├─ Cargo.toml, Cargo.lock (Rust)
   ├─ composer.json, composer.lock (PHP)
   └─ *.csproj, packages.config (NuGet/.NET)

2. Parser extracts dependencies:
   ├─ Direct dependencies (in manifest)
   └─ Transitive dependencies (lockfile)

3. Build dependency tree:
   my-app@1.0.0
     ├─ express@4.18.2
     │   ├─ body-parser@1.20.1
     │   ├─ cookie@0.5.0
     │   └─ debug@2.6.9
     │       └─ ms@2.0.0
     ├─ lodash@4.17.21
     └─ axios@1.4.0
         └─ follow-redirects@1.15.2

4. Submit to GitHub:
   - Via Dependency Submission API
   - Stored in repo metadata
   - Updated on every commit

5. Match against advisory database:
   - Check each dependency+version
   - Flag vulnerabilities
   - Calculate CVSS scores
```

**Visualizing the Dependency Graph:**

```
Repository → Insights → Dependency graph
  ├─ Dependencies tab
  │   ├─ Manifest files
  │   ├─ Package ecosystems
  │   └─ Dependency counts
  │
  ├─ Dependents tab
  │   └─ Repos that depend on this one
  │
  └─ Vulnerabilities tab (if GHAS)
      ├─ Known vulnerabilities
      ├─ Severity distribution
      └─ Remediation PRs
```

**Supported Ecosystems:**

| Ecosystem | Manifest Files | Lockfiles | Dependency Graph | Dependabot Alerts |
|-----------|----------------|-----------|------------------|-------------------|
| npm | package.json | package-lock.json | ✅ | ✅ |
| Yarn | package.json | yarn.lock | ✅ | ✅ |
| pnpm | package.json | pnpm-lock.yaml | ✅ | ✅ |
| RubyGems | Gemfile | Gemfile.lock | ✅ | ✅ |
| pip | requirements.txt | poetry.lock | ✅ | ✅ |
| Poetry | pyproject.toml | poetry.lock | ✅ | ✅ |
| Maven | pom.xml | - | ✅ | ✅ |
| Gradle | build.gradle | - | ✅ | ✅ |
| Go Modules | go.mod | go.sum | ✅ | ✅ |
| Cargo | Cargo.toml | Cargo.lock | ✅ | ✅ |
| NuGet | *.csproj | packages.lock.json | ✅ | ✅ |
| Composer | composer.json | composer.lock | ✅ | ✅ |
| Pub | pubspec.yaml | pubspec.lock | ✅ | ✅ |
| Hex | mix.exs | mix.lock | ✅ | ✅ |
| Swift | Package.swift | Package.resolved | ✅ | ✅ |
| CocoaPods | Podfile | Podfile.lock | ✅ | ✅ |

**Dependency Submission API:**

For ecosystems NOT natively supported or custom builds:

```javascript
// submit-dependencies.js
const { Octokit } = require("@octokit/rest");

const octokit = new Octokit({ auth: process.env.GITHUB_TOKEN });

// Build dependency snapshot
const snapshot = {
  version: 0,
  job: {
    correlator: "my-custom-build",
    id: "123"
  },
  sha: process.env.GITHUB_SHA,
  ref: process.env.GITHUB_REF,
  detector: {
    name: "custom-scanner",
    version: "1.0.0",
    url: "https://github.com/my-org/custom-scanner"
  },
  scanned: new Date().toISOString(),
  manifests: {
    "custom-manifest": {
      name: "custom-manifest",
      resolved: {
        "package-a": {
          package_url: "pkg:npm/package-a@1.0.0",
          dependencies: ["package-b"]
        },
        "package-b": {
          package_url: "pkg:npm/package-b@2.0.0"
        }
      }
    }
  }
};

// Submit to GitHub
await octokit.rest.dependency.submitSnapshot({
  owner: "my-org",
  repo: "my-repo",
  snapshot
});
```

### Software Bill of Materials (SBOM)

**Definition:**
Formal and complete list of all software components, libraries, and dependencies that make up an application.

**SPDX Format (used by GitHub):**

```json
{
  "SPDXID": "SPDXRef-DOCUMENT",
  "spdxVersion": "SPDX-2.3",
  "creationInfo": {
    "created": "2026-04-27T10:00:00Z",
    "creators": ["Tool: GitHub-1.0"]
  },
  "name": "my-app-sbom",
  "dataLicense": "CC0-1.0",
  "documentNamespace": "https://github.com/owner/repo/sbom/abc123",
  "packages": [
    {
      "SPDXID": "SPDXRef-Package-express",
      "name": "express",
      "versionInfo": "4.18.2",
      "filesAnalyzed": false,
      "downloadLocation": "https://registry.npmjs.org/express/-/express-4.18.2.tgz",
      "homepage": "http://expressjs.com/",
      "licenseConcluded": "MIT",
      "licenseDeclared": "MIT"
    },
    {
      "SPDXID": "SPDXRef-Package-lodash",
      "name": "lodash",
      "versionInfo": "4.17.21",
      "filesAnalyzed": false,
      "downloadLocation": "https://registry.npmjs.org/lodash/-/lodash-4.17.21.tgz",
      "homepage": "https://lodash.com/",
      "licenseConcluded": "MIT",
      "licenseDeclared": "MIT"
    }
  ],
  "relationships": [
    {
      "spdxElementId": "SPDXRef-DOCUMENT",
      "relatedSpdxElement": "SPDXRef-Package-express",
      "relationshipType": "DESCRIBES"
    },
    {
      "spdxElementId": "SPDXRef-Package-express",
      "relatedSpdxElement": "SPDXRef-Package-body-parser",
      "relationshipType": "DEPENDS_ON"
    }
  ]
}
```

**Generate SBOM from GitHub:**

```bash
# Via API
curl -H "Authorization: token $GITHUB_TOKEN" \
     -H "Accept: application/vnd.github+json" \
     https://api.github.com/repos/OWNER/REPO/dependency-graph/sbom \
     > sbom.json

# Via GitHub CLI
gh api /repos/OWNER/REPO/dependency-graph/sbom > sbom.json

# In GitHub Actions
- name: Generate SBOM
  run: |
    gh api repos/${{ github.repository }}/dependency-graph/sbom \
      --jq '.sbom' > sbom.spdx.json
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

- name: Upload SBOM artifact
  uses: actions/upload-artifact@v4
  with:
    name: sbom
    path: sbom.spdx.json
```

**SBOM Use Cases:**

```yaml
1. Compliance:
    - NIST guidelines require SBOM
    - Executive Order 14028 (US)
    - EU Cyber Resilience Act

2. Supply chain security:
    - Identify components with known vulnerabilities
    - License audit
    - Track updates

3. Incident response:
    - "Are we affected by Log4Shell?"
    - Query SBOM for "log4j" → Yes/No immediately
    - Exact version → Patch priority

4. Procurement:
    - Vendors must provide SBOM
    - Verify components before purchase
    - Automated due diligence
```

### Dependency Vulnerabilities

**Definition:**
Known security defect (CVE) in a dependency that can be exploited to compromise the application.

**Anatomy of a Vulnerability:**

```yaml
CVE-2022-24999:
  package: "express"
  vulnerable_versions: "< 4.17.3"
  severity: "High"
  cvss_score: 7.5
  cvss_vector: "CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:N/I:N/A:H"
  
  cwe: "CWE-400: Uncontrolled Resource Consumption"
  
  description: |
    qs before 6.10.3 allows attackers to cause a denial of service
    (memory exhaustion) by sending a crafted request.
  
  affected_functions:
    - "express.urlencoded"
    - "express.json"
  
  patch_versions: ">= 4.17.3"
  
  references:
    - "https://nvd.nist.gov/vuln/detail/CVE-2022-24999"
    - "https://github.com/advisories/GHSA-hrpp-h998-j3pp"
    - "https://github.com/ljharb/qs/pull/428"
  
  exploitability: "Proof of concept exists"
  
  impact: |
    Attackers can send specially crafted query strings to exhaust
    server memory, causing denial of service.
```

**Severity (CVSS Score):**

| Score | Rating | Priority | SLA |
|-------|--------|-----------|-----|
| 9.0-10.0 | Critical | P0 | 24 hours |
| 7.0-8.9 | High | P1 | 7 days |
| 4.0-6.9 | Medium | P2 | 30 days |
| 0.1-3.9 | Low | P3 | 90 days |

**GitHub Advisory Database:**

```
https://github.com/advisories

Sources:
  ├─ National Vulnerability Database (NVD)
  ├─ GitHub Security Advisories
  ├─ npm security advisories
  ├─ RubySec advisories
  ├─ Python Packaging Advisory Database
  ├─ Rust Security Advisory Database
  └─ Community contributions

Update frequency: Multiple times a day
```

### Dependabot Alerts

**What are they?**
Automated notifications when a dependency has a known vulnerability.

**How they work:**

```
1. Dependency Graph identifies dependencies
     ↓
2. GitHub Advisory Database gets a new CVE
     ↓
3. Match: lodash@4.17.15 → CVE-2021-23337
     ↓
4. Calculate severity: High (CVSS 7.2)
     ↓
5. Create alert in Security tab
     ↓
6. Notify:
     ├─ Email to repository admins
     ├─ Web notification
     ├─ Webhook (if configured)
     └─ Security Overview
```

**Alert Format:**

```
┌─────────────────────────────────────────────────────────────┐
│ 🔴 Prototype Pollution in lodash                            │
├─────────────────────────────────────────────────────────────┤
│ Package: lodash                                             │
│ Vulnerable: < 4.17.21                                       │
│ Patched: >= 4.17.21                                         │
│ Severity: High (CVSS 7.2)                                   │
│ CWE-1321: Improperly Controlled Modification                │
│                                                             │
│ Description:                                                │
│ Prototype pollution via setWith and set functions           │
│                                                             │
│ Remediation:                                                │
│ ├─ [✓] Dependabot security update available                 │
│ ├─ Update lodash to 4.17.21 or later                        │
│ └─ [View PR #123] [Dismiss alert]                           │
│                                                             │
│ References:                                                 │
│ • CVE-2021-23337                                            │
│ • GHSA-35jh-r3h4-6jhm                                       │
└─────────────────────────────────────────────────────────────┘
```

### Dependabot Security Updates

**What are they?**
Automated pull requests that update vulnerable dependencies to patched versions.

**Functioning:**

```
1. Dependabot alert created for lodash@4.17.15
     ↓
2. Dependabot checks if patched version exists
     ├─ lodash@4.17.21 exists
     └─ It is backward compatible (patch/minor)
     ↓
3. Create branch: dependabot/npm_and_yarn/lodash-4.17.21
     ↓
4. Update package.json and lockfile
     ↓
5. Run tests (if CI is configured)
     ↓
6. Open PR with details:
     ├─ Changelog
     ├─ Commits
     ├─ Compatibility score
     └─ Release notes
     ↓
7. Developer review + merge
     ↓
8. Alert auto-closed
```

**Dependabot PR Example:**

```markdown
## Bump lodash from 4.17.15 to 4.17.21

**Dependabot** will resolve any conflicts with this PR as long as you don't alter it yourself.

### Vulnerabilities fixed
🔴 **High severity** - CVE-2021-23337
Prototype Pollution in lodash

### Release notes
<details>
<summary>4.17.21</summary>

#### Fixed
- Prototype pollution via setWith and set

#### Changelog
See full changelog: https://github.com/lodash/lodash/releases/tag/4.17.21
</details>

### Commits
- [`f299b52`] Bump to v4.17.21
- [`c4847eb`] Fix prototype pollution
- See full diff: lodash/lodash@4.17.15...4.17.21

### Compatibility score
Dependabot will merge this PR once CI passes on it, as requested by @you.

**Note:** This PR was generated automatically by Dependabot.
```

### Dependency Review

**What is it?**
A feature that analyzes dependency changes in PRs and blocks the merge if vulnerabilities are introduced.

**Key Difference:**

```
Dependabot Alerts:
  - Scans existing dependencies
  - Reactive (alerts after merge)
  - Security tab

Dependency Review:
  - Scans changes in PR
  - Proactive (blocks before merge)
  - PR checks
```

**Functioning:**

```
Developer creates PR:
  package.json: lodash@4.17.15 → lodash@4.17.10 (downgrade!)
     ↓
Dependency Review Action runs:
     ↓
Compares:
  Base branch (main): lodash@4.17.15 (no vulnerabilities)
  PR branch: lodash@4.17.10 (CVE-2020-8203: HIGH)
     ↓
Result:
  ❌ Check failed: 1 high severity vulnerability introduced
     ↓
Blocks merge:
  - PR status: ❌ Dependency review — Changes introduce known vulnerabilities
  - Requires: Fix before merge
```

**Dependency Review Action Configuration:**

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
      - name: 'Checkout Repository'
        uses: actions/checkout@v4
      
      - name: 'Dependency Review'
        uses: actions/dependency-review-action@v4
        with:
          # Fail on moderate or higher only
          fail-on-severity: moderate
          
          # Allow specific licenses
          allow-licenses: MIT, Apache-2.0, BSD-3-Clause
          
          # Deny specific licenses
          deny-licenses: GPL-3.0, AGPL-3.0
          
          # Comment on PR with details
          comment-summary-in-pr: always
          
          # Check for malicious packages
          warn-on-openssf-scorecard-level: 3
```

### Alert Generation for Vulnerable Dependencies

**Complete Pipeline:**

```
┌──────────────────────────────────────────────────┐
│ 1. COMMIT pushed to repository                   │
└──────────────────┬───────────────────────────────┘
                   │
┌──────────────────▼───────────────────────────────┐
│ 2. Dependency Graph updated                      │
│    - Parse manifest files                        │
│    - Extract dependencies                        │
│    - Build dependency tree                       │
└──────────────────┬───────────────────────────────┘
                   │
┌──────────────────▼───────────────────────────────┐
│ 3. Match against GitHub Advisory Database        │
│    For each dependency:                          │
│      - Check package + version                   │
│      - Query advisories                          │
│      - Calculate CVSS score                      │
└──────────────────┬───────────────────────────────┘
                   │
               ┌────┴────┐
               │ Match?  │
               └────┬────┘
                   │
         ┌─────────┴─────────┐
        YES                  NO
         │                    │
┌────────▼────────┐    ┌─────▼──────┐
│ 4. Create Alert │    │  No Action │
│   - Generate    │    └────────────┘
│     alert       │
│   - Set severity│
│   - Add metadata│
└────────┬────────┘
         │
┌────────▼──────────────────────────────────────────┐
│ 5. Notify                                         │
│    ├─ Repository admins (email)                   │
│    ├─ Security managers                           │
│    ├─ Webhooks (if configured)                    │
│    └─ Integrations (Slack, PagerDuty, etc.)       │
└────────┬──────────────────────────────────────────┘
         │
┌────────▼──────────────────────────────────────────┐
│ 6. Dependabot evaluates security update           │
│    ├─ Is patch available?                         │
│    ├─ Is it backward compatible?                  │
│    ├─ Are there breaking changes?                 │
│    └─ Create PR? (if enabled)                     │
└───────────────────────────────────────────────────┘
```

### Difference between Dependabot and Dependency Review

**Complete Comparative Table:**

| Aspect | Dependabot Alerts | Dependabot Security Updates | Dependency Review |
|---------|------------------|----------------------------|-------------------|
| **When it acts** | After commit | After alert | During PR |
| **Objective** | Detect existing vulnerabilities | Automate fixes | Prevent new vulnerabilities |
| **Location** | Security tab | Pull requests tab | PR checks |
| **Action** | Create alert | Create fix PR | Block/approve merge |
| **Reactive/Proactive** | Reactive | Reactive | Proactive |
| **Requires GHAS** | No (public), Yes (private) | No (public), Yes (private) | Yes |
| **Blocks code** | No | No | Yes (configurable) |
| **Auto-remediation** | No | Yes (PR) | No |
| **Scope** | Entire repo | Vulnerable dependencies | Changes in PR |
| **Configuration** | Settings → Dependabot | Settings → Dependabot | GitHub Actions workflow |

**Ideal Combined Flow:**

```
┌─────────────────────────────────────────┐
│ Developer updates package.json          │
│ npm install lodash@4.17.10              │
└─────────┬───────────────────────────────┘
          │
┌─────────▼───────────────────────────────┐
│ git commit & push to feature branch     │
└─────────┬───────────────────────────────┘
          │
┌─────────▼───────────────────────────────┐
│ Opens PR to main                        │
└─────────┬───────────────────────────────┘
          │
┌─────────▼───────────────────────────────┐
│ Dependency Review Action runs           │
│ X Found: CVE-2020-8203 in lodash@4.17.10│
│ PR check FAILS                          │
└─────────┬───────────────────────────────┘
          │
┌─────────▼───────────────────────────────┐
│ Developer sees:                         │
│ "Cannot merge - vulnerabilities found"  │
│ Updates to lodash@4.17.21               │
└─────────┬───────────────────────────────┘
          │
┌─────────▼───────────────────────────────┐
│ Push update                             │
│ Dependency Review re-runs               │
│ ✅ No vulnerabilities                   │
│ PR check PASSES                         │
└─────────┬───────────────────────────────┘
          │
┌─────────▼───────────────────────────────┐
│ Merge to main                           │
└─────────┬───────────────────────────────┘
          │
┌─────────▼───────────────────────────────┐
│ Dependabot monitors main branch         │
│ (No alerts-all dependencies up to date) │
└─────────────────────────────────────────┘
          │
          ├─ [Future: New CVE discovered]
          │
┌─────────▼───────────────────────────────┐
│ Dependabot Alert created                │
│ Dependabot Security Update PR created   │
│ Team reviews & merges                   │
└─────────────────────────────────────────┘
```

**When to use each tool:**

```yaml
Dependabot Alerts:
  Use for:
    - ✅ Continuous monitoring of dependencies
    - ✅ Detecting vulnerabilities in main branch
    - ✅ Compliance reporting
    - ✅ Security overview metrics

Dependabot Security Updates:
  Use for:
    - ✅ Automating security updates
    - ✅ Reducing remediation time
    - ✅ Keeping dependencies current
    - ✅ Batch updates (via grouping)

Dependency Review:
  Use for:
    - ✅ Gating PRs with vulnerabilities
    - ✅ Preventing security regressions
    - ✅ License compliance
    - ✅ Enforcing security policies
    - ✅ Educating developers at PR time
```

**Links:**
- https://docs.github.com/en/code-security/supply-chain-security/understanding-your-software-supply-chain/about-the-dependency-graph
- https://docs.github.com/en/code-security/dependabot
- https://docs.github.com/en/code-security/supply-chain-security/understanding-your-software-supply-chain/about-dependency-review

---
<a id="d3-2"></a>
## 3.2 Default configuration for Dependabot alerts

### Public Repositories

**Automatic Configuration:**

```yaml
Dependabot Alerts: ✅ ENABLED by default
Dependency Graph: ✅ ENABLED by default
Dependabot Security Updates: ✅ ENABLED by default

Included features:
  - Automatic vulnerability alerts
  - Automatic security PRs
  - Email notifications
  - Visible Security tab
  - Public dependency graph
```

**Does not require:**
- ❌ GHAS License
- ❌ Manual configuration
- ❌ GitHub Actions minutes (Dependabot PRs are free)

### Private Repositories

**Without GHAS:**
```yaml
Dependency Graph: ✅ ENABLED by default
Dependabot Alerts: ✅ ENABLED by default (since 2022)
Dependabot Security Updates: ❌ DISABLED (requires enabling)
Dependency Review: ❌ NOT AVAILABLE (requires GHAS)
```

**With GHAS (GitHub Code Security):**
```yaml
Dependency Graph: ✅ ENABLED
Dependabot Alerts: ✅ ENABLED
Dependabot Security Updates: ✅ Can be enabled
Dependency Review: ✅ AVAILABLE
Custom Auto-triage Rules: ✅ AVAILABLE
```

### Comparative Table of Default Configurations

| Feature | Public | Private without GHAS | Private with GHAS |
|---------|---------|------------------|------------------|
| **Dependency Graph** | ✅ Auto | ✅ Auto | ✅ Auto |
| **Dependabot Alerts** | ✅ Auto | ✅ Auto | ✅ Auto |
| **Dependabot Security Updates** | ✅ Auto | Opt-in | Opt-in |
| **Dependabot Version Updates** | Opt-in | Opt-in | Opt-in |
| **Dependency Review** | ❌ | ❌ | ✅ Requires config |
| **Custom Auto-triage** | ❌ | ❌ | ✅ |
| **Security Overview** | ❌ | ❌ | ✅ |
| **Grouped Updates** | Opt-in | Opt-in | Opt-in |

### Verify Current Configuration

```bash
# Via GitHub CLI
gh api repos/:owner/:repo | jq '{
  dependency_graph: .has_dependency_graph,
  vulnerability_alerts: .vulnerability_alerts_enabled,
  automated_security_fixes: .automated_security_fixes_enabled
}'

# Via API
curl -H "Authorization: token $GITHUB_TOKEN" \
     https://api.github.com/repos/OWNER/REPO | \
     jq '.vulnerability_alerts_enabled, .automated_security_fixes_enabled'

# Via Web UI
Repository → Settings → Code security and analysis
  ├─ Dependency graph: [Enabled/Disabled]
  ├─ Dependabot alerts: [Enabled/Disabled]
  └─ Dependabot security updates: [Enabled/Disabled]
```

**Links:**
- https://docs.github.com/en/code-security/dependabot/dependabot-alerts/about-dependabot-alerts
- https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/enabling-features-for-your-repository/managing-security-and-analysis-settings-for-your-repository
- https://docs.github.com/en/code-security/getting-started/securing-your-repository

---
<a id="d3-3"></a>
## 3.3 Permissions and roles for Dependabot

### Permissions to ENABLE Dependabot Alerts

**At Repository Level:**

| Role | Enable Dependabot Alerts | Enable Security Updates | Configure dependabot.yml |
|-----|----------------------------|---------------------------|---------------------------|
| **Read** | ❌ | ❌ | ❌ |
| **Triage** | ❌ | ❌ | ❌ |
| **Write** | ❌ | ❌ | ✅ (via PR) |
| **Maintain** | ❌ | ❌ | ✅ |
| **Admin** | ✅ | ✅ | ✅ |

**At Organization Level:**

| Role | Enable for Org | Policies | Bulk Enable |
|-----|-------------------|----------|-------------|
| **Member** | ❌ | ❌ | ❌ |
| **Owner** | ✅ | ✅ | ✅ |
| **Security Manager** | ✅ | ✅ | ✅ |

### Permissions to VIEW Dependabot Alerts

**Important:** Dependabot alerts have different visibility rules than other security alerts.

| Role | View Dependabot Alerts | View Details | Dismiss Alerts | View PRs |
|-----|----------------------|--------------|----------------|---------|
| **Read** | ✅ | ✅ | ❌ | ✅ |
| **Triage** | ✅ | ✅ | ❌ | ✅ |
| **Write** | ✅ | ✅ | ✅ | ✅ |
| **Maintain** | ✅ | ✅ | ✅ | ✅ |
| **Admin** | ✅ | ✅ | ✅ | ✅ |
| **Security Manager** | ✅ | ✅ | ✅ | ✅ |

**Difference with Code Scanning:**
```yaml
Code Scanning:
  - Only Admin and Security Manager can view alerts

Dependabot:
  - All collaborators with Read+ can view alerts
  - Reason: Developers need to see dependencies to do their work
```

### Granular Access Configuration

**Grant access to a team:**

```bash
# Via GitHub CLI
gh api \
  --method PUT \
  -H "Accept: application/vnd.github+json" \
  /repos/OWNER/REPO/teams/TEAM_SLUG \
  -f permission='push'  # write access includes Dependabot alerts

# Via API
curl -X PUT \
  -H "Authorization: token $GITHUB_TOKEN" \
  -H "Accept: application/vnd.github+json" \
  https://api.github.com/repos/OWNER/REPO/teams/TEAM_SLUG \
  -d '{"permission":"push"}'
```

**Security Manager role (organization level):**

```bash
# Add security manager to org
gh api \
  --method PUT \
  -H "Accept: application/vnd.github+json" \
  /orgs/ORG/security-managers/teams/TEAM_SLUG

# List security managers
gh api /orgs/ORG/security-managers/teams
```

**Custom Notification Groups:**

```yaml
# No native configuration for custom groups
# Solution: Use webhooks + automation

# .github/workflows/dependabot-router.yml
name: Route Dependabot Alerts

on:
  dependabot_alert:
    types: [created, reopened]

jobs:
  route-alert:
    runs-on: ubuntu-latest
    steps:
      - name: Route based on package ecosystem
        uses: actions/github-script@v7
        with:
          script: |
            const alert = context.payload.alert;
            const ecosystem = alert.dependency.package.ecosystem;
            
            let team;
            if (ecosystem === 'npm') team = '@org/frontend-team';
            else if (ecosystem === 'pip') team = '@org/backend-team';
            else if (ecosystem === 'maven') team = '@org/java-team';
            
            // Create issue and assign
            await github.rest.issues.create({
              owner: context.repo.owner,
              repo: context.repo.name,
              title: `Dependabot: ${alert.dependency.package.name}`,
              body: `Security alert: ${alert.security_advisory.summary}`,
              assignees: [team],
              labels: ['security', 'dependencies']
            });
```

**Links:**
- https://docs.github.com/en/organizations/managing-user-access-to-your-organizations-repositories/repository-roles-for-an-organization
- https://docs.github.com/en/code-security/dependabot/dependabot-alerts/configuring-dependabot-alerts
- https://docs.github.com/en/organizations/managing-peoples-access-to-your-organization-with-roles/managing-security-managers-in-your-organization
- https://docs.github.com/en/code-security/dependabot/working-with-dependabot/configuring-access-to-private-registries-for-dependabot

---
<a id="d3-4"></a>
## 3.4 Enable Dependabot for private repositories

### Method 1: Via Web UI (Individual)

**Step-by-Step:**

```
1. Go to the repository
   └─ Settings tab

2. Navigate to Code security and analysis
   └─ Left sidebar

3. Enable Dependency graph (if not enabled)
   ├─ Click [Enable]
   └─ Wait 1-2 minutes for the initial analysis

4. Enable Dependabot alerts
   ├─ Click [Enable]
   └─ Confirm

5. [Optional] Enable Dependabot security updates
   ├─ Click [Enable]
   └─ This allows automatic PRs
```

### Method 2: Via GitHub CLI

```bash
# Enable everything at once
gh api \
  --method PATCH \
  -H "Accept: application/vnd.github+json" \
  /repos/OWNER/REPO \
  -f has_dependency_graph=true

# Enable vulnerability alerts
gh api \
  --method PUT \
  -H "Accept: application/vnd.github+json" \
  /repos/OWNER/REPO/vulnerability-alerts

# Enable security updates
gh api \
  --method PUT \
  -H "Accept: application/vnd.github+json" \
  /repos/OWNER/REPO/automated-security-fixes

# Verify
gh api repos/OWNER/REPO | jq '{
  dependency_graph: .has_dependency_graph,
  alerts: .vulnerability_alerts_enabled,
  security_updates: .automated_security_fixes_enabled
}'
```

### Method 3: Via API (Programmatic)

```python
import requests

GITHUB_TOKEN = "ghp_..."
ORG = "my-org"

headers = {
    "Authorization": f"token {GITHUB_TOKEN}",
    "Accept": "application/vnd.github+json"
}

# Get all private repos
repos_response = requests.get(
    f"https://api.github.com/orgs/{ORG}/repos",
    headers=headers,
    params={"type": "private", "per_page": 100}
)

for repo in repos_response.json():
    repo_name = repo["full_name"]
    
    print(f"Enabling Dependabot for {repo_name}...")
    
    # Enable vulnerability alerts
    alerts_response = requests.put(
        f"https://api.github.com/repos/{repo_name}/vulnerability-alerts",
        headers=headers
    )
    
    # Enable automated security fixes
    fixes_response = requests.put(
        f"https://api.github.com/repos/{repo_name}/automated-security-fixes",
        headers=headers
    )
    
    if alerts_response.status_code == 204 and fixes_response.status_code == 204:
        print(f"  ✅ {repo_name}: Dependabot enabled")
    else:
        print(f"  ❌ {repo_name}: Error - {alerts_response.status_code}")
```

### Method 4: Bulk Enablement (Organization level)

**Via UI:**

```
Organization Settings
  → Code security and analysis
  → Dependabot
      ├─ [Enable for all repositories]
      │   └─ Select:
      │       ├─ All repositories
      │       ├─ All private repositories
      │       └─ Selected repositories
      │
      └─ [✓] Automatically enable for new repositories
          ├─ New public repositories
          └─ New private repositories
```

**Bulk Enablement Script:**

```bash
#!/bin/bash
# enable-dependabot-all-repos.sh

ORG="my-org"
TOKEN="$GITHUB_TOKEN"

# Get all repos
repos=$(gh api --paginate "/orgs/$ORG/repos" --jq '.[].name')

echo "Found $(echo "$repos" | wc -l) repositories"
echo "Enabling Dependabot..."

for repo in $repos; do
    echo -n "Processing $repo... "
    
    # Enable vulnerability alerts
    gh api \
        --method PUT \
        --silent \
        "/repos/$ORG/$repo/vulnerability-alerts" 2>/dev/null
    
    # Enable automated security fixes
    gh api \
        --method PUT \
        --silent \
        "/repos/$ORG/$repo/automated-security-fixes" 2>/dev/null
    
    echo "✅"
done

echo "Done! Dependabot enabled for all repositories."
```

**Links:**
- https://docs.github.com/en/code-security/dependabot/dependabot-alerts/configuring-dependabot-alerts
- https://docs.github.com/en/rest/repos/repos#enable-vulnerability-alerts
- https://docs.github.com/en/rest/repos/repos#enable-automated-security-fixes
- https://docs.github.com/en/enterprise-cloud@latest/code-security/dependabot/dependabot-alerts/configuring-dependabot-alerts#managing-dependabot-alerts-for-your-organization

---
<a id="d3-5"></a>
## 3.5 Enable Dependabot for organizations

### Organization-level Policies

**Option 1: Enable for all**

```
Organization Settings
  → Code security and analysis
  → Dependabot alerts
      └─ [Enable for all repositories]
          ├─ Apply to: All repositories
          └─ [Confirm]
```

**Option 2: Default policy for new repos**

```
Organization Settings
  → Code security and analysis
  → Dependabot alerts
      └─ [✓] Automatically enable for new repositories
          ├─ Public repositories: [✓] Enabled
          └─ Private repositories: [✓] Enabled
```

**Option 3: Conditional Policies (GitHub Enterprise)**

```yaml
# Only available in GitHub Enterprise Cloud
# Via Enterprise Settings

Enterprise Settings
  → Policies
  → Advanced Security
      ├─ Dependabot alerts
      │   ├─ Enforced: All organizations must enable
      │   ├─ Not enforced: Organizations can choose
      │   └─ Disabled: Organizations cannot enable
      │
      └─ Dependabot security updates
          ├─ Enforced
          ├─ Not enforced
          └─ Disabled
```

### Organization-level Notification Configuration

```yaml
# Configure default recipients

Organization Settings
  → Code security
  → Dependabot alerts
      └─ Default notification settings
          ├─ Notify: security@company.com
          ├─ Slack: #security-alerts
          └─ Webhook: https://api.company.com/webhooks/dependabot
```

### Reporting and Compliance

```bash
# Script to generate coverage report

#!/bin/bash
ORG="my-org"

echo "Dependabot Coverage Report for $ORG"
echo "===================================="
echo ""

total_repos=0
enabled_repos=0
disabled_repos=0

# Get all repos
repos=$(gh api --paginate "/orgs/$ORG/repos" --jq '.[].name')
total_repos=$(echo "$repos" | wc -l)

echo "Total repositories: $total_repos"
echo ""
echo "Checking Dependabot status..."
echo ""

for repo in $repos; do
    status=$(gh api "/repos/$ORG/$repo" --jq '.vulnerability_alerts_enabled')
    
    if [ "$status" = "true" ]; then
        enabled_repos=$((enabled_repos + 1))
        echo "✅ $repo"
    else
        disabled_repos=$((disabled_repos + 1))
        echo "❌ $repo"
    fi
done

echo ""
echo "Summary:"
echo "--------"
echo "Enabled: $enabled_repos / $total_repos ($(( enabled_repos * 100 / total_repos ))%)"
echo "Disabled: $disabled_repos / $total_repos"
```

**Links:**
- https://docs.github.com/en/organizations/keeping-your-organization-secure/managing-security-settings-for-your-organization/managing-security-and-analysis-settings-for-your-organization
- https://docs.github.com/en/code-security/dependabot/dependabot-alerts/configuring-dependabot-alerts#managing-dependabot-alerts-for-your-organization
- https://docs.github.com/en/enterprise-cloud@latest/admin/policies/enforcing-policies-for-your-enterprise/enforcing-policies-for-code-security-and-analysis-for-your-enterprise

---
<a id="d3-6"></a>
## 3.6 Create Dependabot configuration file (dependabot.yml)

### Basic Structure

```yaml
# .github/dependabot.yml
version: 2
updates:
  # Configuration for npm
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
      day: "monday"
      time: "09:00"
      timezone: "America/New_York"
    
    open-pull-requests-limit: 10
    
    reviewers:
      - "octocat"
      - "org/team-name"
    
    assignees:
      - "security-lead"
    
    labels:
      - "dependencies"
      - "npm"
    
    commit-message:
      prefix: "chore"
      prefix-development: "build"
      include: "scope"
```

### Supported Ecosystems

```yaml
# All available ecosystems:

updates:
  - package-ecosystem: "npm"           # JavaScript (npm, yarn, pnpm)
  - package-ecosystem: "bundler"       # Ruby
  - package-ecosystem: "pip"           # Python
  - package-ecosystem: "maven"         # Java (Maven)
  - package-ecosystem: "gradle"        # Java/Kotlin (Gradle)
  - package-ecosystem: "cargo"         # Rust
  - package-ecosystem: "gomod"         # Go modules
  - package-ecosystem: "composer"      # PHP
  - package-ecosystem: "nuget"         # .NET
  - package-ecosystem: "docker"        # Docker
  - package-ecosystem: "terraform"     # Terraform
  - package-ecosystem: "github-actions" # GitHub Actions workflows
  - package-ecosystem: "pub"           # Dart/Flutter
  - package-ecosystem: "hex"           # Elixir
  - package-ecosystem: "swift"         # Swift Package Manager
```

### Grouping Updates

**Group updates to reduce PR noise:**

```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
    
    # Group by dependency type
    groups:
      # Group for all dev dependencies
      development-dependencies:
        dependency-type: "development"
        update-types:
          - "minor"
          - "patch"
      
      # Group for production dependencies (patches only)
      production-dependencies:
        dependency-type: "production"
        update-types:
          - "patch"
      
      # Group by name pattern
      react-related:
        patterns:
          - "react*"
          - "@types/react*"
      
      # Group for testing frameworks
      testing-frameworks:
        patterns:
          - "jest"
          - "@testing-library/*"
          - "vitest"
```

**Advanced Grouping Example:**

```yaml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/frontend"
    schedule:
      interval: "weekly"
    
    groups:
      # Monorepo pattern
      internal-packages:
        patterns:
          - "@company/*"
        update-types:
          - "minor"
          - "patch"
      
      # Major updates separately
      major-updates:
        update-types:
          - "major"
        # Do not group - one PR per major update
      
      # Framework updates
      angular:
        patterns:
          - "@angular/*"
          - "@angular-devkit/*"
          - "rxjs"
        update-types:
          - "minor"
          - "patch"
      
      # Tooling
      build-tools:
        patterns:
          - "webpack*"
          - "babel*"
          - "@babel/*"
          - "eslint*"
```

**Grouping Results:**

```
Without grouping:
  ├─ PR #1: Bump eslint from 8.0.0 to 8.0.1
  ├─ PR #2: Bump eslint-config-airbnb from 19.0.0 to 19.0.1
  ├─ PR #3: Bump eslint-plugin-import from 2.25.0 to 2.25.1
  └─ ... (10 more PRs for tooling)

With grouping:
  └─ PR #1: Bump build-tools group (13 dependencies)
      ├─ eslint: 8.0.0 → 8.0.1
      ├─ eslint-config-airbnb: 19.0.0 → 19.0.1
      └─ ... (11 more)
```

### Automerge Configuration

```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "daily"
    
    # Pull request branch name
    pull-request-branch-name:
      separator: "-"
```

**GitHub Actions workflow for auto-merge:**

```yaml
# .github/workflows/dependabot-auto-merge.yml
name: Dependabot Auto-merge

on: pull_request

permissions:
  contents: write
  pull-requests: write

jobs:
  auto-merge:
    runs-on: ubuntu-latest
    if: github.actor == 'dependabot[bot]'
    steps:
      - name: Dependabot metadata
        id: metadata
        uses: dependabot/fetch-metadata@v2
        with:
          github-token: "${{ secrets.GITHUB_TOKEN }}"
      
      - name: Auto-merge patch and minor updates
        if: |
          steps.metadata.outputs.update-type == 'version-update:semver-patch' ||
          steps.metadata.outputs.update-type == 'version-update:semver-minor'
        run: gh pr merge --auto --squash "$PR_URL"
        env:
          PR_URL: ${{ github.event.pull_request.html_url }}
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      
      - name: Approve PR
        if: |
          steps.metadata.outputs.update-type == 'version-update:semver-patch' ||
          steps.metadata.outputs.update-type == 'version-update:semver-minor'
        run: gh pr review --approve "$PR_URL"
        env:
          PR_URL: ${{ github.event.pull_request.html_url }}
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### Advanced Scheduling Configuration

```yaml
version: 2
updates:
  # Different schedules for different ecosystems
  
  # npm: Daily (high frequency of updates)
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "daily"
      time: "03:00"
      timezone: "UTC"
  
  # Docker: Weekly (less frequent)
  - package-ecosystem: "docker"
    directory: "/"
    schedule:
      interval: "weekly"
      day: "sunday"
      time: "04:00"
  
  # GitHub Actions: Monthly (very stable)
  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "monthly"
    
  # Terraform: Security updates only
  - package-ecosystem: "terraform"
    directory: "/infrastructure"
    schedule:
      interval: "weekly"
    open-pull-requests-limit: 0  # No version updates
    # Security updates only through Dependabot alerts
```

### Ignore Configurations

```yaml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
    
    # Ignore specific dependencies
    ignore:
      # Never update React (pinned version)
      - dependency-name: "react"
        update-types: ["version-update:semver-major"]
      
      # Ignore patches of lodash (too many updates)
      - dependency-name: "lodash"
        update-types: ["version-update:semver-patch"]
      
      # Ignore all major versions of Angular
      - dependency-name: "@angular/*"
        update-types: ["version-update:semver-major"]
      
      # Ignore specific versions that break
      - dependency-name: "webpack"
        versions: ["5.x"]
```

### Configuration for Monorepos

```yaml
version: 2
updates:
  # Frontend app
  - package-ecosystem: "npm"
    directory: "/apps/frontend"
    schedule:
      interval: "weekly"
    groups:
      frontend-deps:
        patterns: ["*"]
  
  # Backend app
  - package-ecosystem: "npm"
    directory: "/apps/backend"
    schedule:
      interval: "weekly"
    groups:
      backend-deps:
        patterns: ["*"]
  
  # Shared packages
  - package-ecosystem: "npm"
    directory: "/packages/shared"
    schedule:
      interval: "weekly"
  
  # Root workspace
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
    
  # Infrastructure
  - package-ecosystem: "terraform"
    directory: "/infrastructure"
    schedule:
      interval: "monthly"
```

### File Validation

```bash
# YAML linter check
yamllint .github/dependabot.yml

# Verify syntax with Python
python -c "import yaml; yaml.safe_load(open('.github/dependabot.yml'))"

# Verification after commit
# GitHub will validate the file automatically
# Errors will appear in:
# Repository → Insights → Dependency graph → Dependabot

# Validation workflow check
# .github/workflows/validate-dependabot.yml
name: Validate Dependabot Config

on:
  pull_request:
    paths:
      - '.github/dependabot.yml'

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Validate YAML syntax
        run: |
          python -c "import yaml; yaml.safe_load(open('.github/dependabot.yml'))"
      
      - name: Check required fields
        run: |
          python << 'EOF'
          import yaml
          
          with open('.github/dependabot.yml') as f:
              config = yaml.safe_load(f)
          
          assert config['version'] == 2, "Version must be 2"
          assert 'updates' in config, "Must have 'updates' section"
          
          for update in config['updates']:
              assert 'package-ecosystem' in update
              assert 'directory' in update
              assert 'schedule' in update
              assert 'interval' in update['schedule']
          
          print("✅ Dependabot config is valid")
          EOF
```

**Links:**
- https://docs.github.com/en/code-security/dependabot/dependabot-version-updates/configuration-options-for-the-dependabot.yml-file
- https://docs.github.com/en/code-security/dependabot/dependabot-version-updates/configuring-dependabot-version-updates
- https://docs.github.com/en/code-security/dependabot/dependabot-version-updates/about-dependabot-version-updates
- https://docs.github.com/en/code-security/dependabot/working-with-dependabot/keeping-your-actions-up-to-date-with-dependabot
- https://docs.github.com/en/code-security/dependabot/dependabot-version-updates/grouping-dependabot-updates

---
<a id="d3-7"></a>
## 3.7 Custom Auto-triage Rules for Dependabot

**Available with GitHub Code Security**

### What are Auto-triage Rules?

Automated rules to manage Dependabot alerts at scale:
- Auto-dismiss low-priority alerts
- Auto-snooze until a patch is available
- Auto-trigger security updates for specific criteria

### Create Auto-triage Rules

**Via UI:**

```
Repository/Organization Settings
  → Code security
  → Dependabot
  → Auto-triage rules
      → [New rule]
```

**Example 1: Dismiss low severity without patch**

```yaml
Rule name: Dismiss low severity without patch
Conditions:
  - Severity: Low
  - State: Open
  - Has patch: No

Action: Dismiss
Reason: "Low severity, will address when patch available"
```

**Example 2: Auto-approve dev dependency patches**

```yaml
Rule name: Auto-approve dev dependency patches
Conditions:
  - Dependency scope: Development
  - Update type: Patch (semver)
  - Severity: Any

Action: Create security update PR
Auto-merge: Yes (if CI passes)
```

**Via API:**

```bash
# Create auto-triage rule
gh api \
  --method POST \
  -H "Accept: application/vnd.github+json" \
  /repos/OWNER/REPO/dependabot/auto-triage-rules \
  -f name="Dismiss low without patch" \
  -f conditions='[{
    "type": "severity",
    "value": "low"
  }, {
    "type": "patch_available",
    "value": false
  }]' \
  -f action="dismiss" \
  -f dismissal_reason="Low severity without available patch"
```

### Common Use Cases

**1. Technical Debt Management:**

```yaml
Rule: Snooze medium severity for legacy code
Conditions:
  - Severity: Medium
  - File path: /legacy/**
Action: Snooze for 90 days
Reason: "Legacy code scheduled for deprecation"
```

**2. Prioritization by Ecosystem:**

```yaml
Rule: High priority for production npm packages
Conditions:
  - Package ecosystem: npm
  - Dependency scope: Production
  - Severity: High or Critical
Action: Create issue
Assign: @org/security-team
Labels: P0, security, npm
```

**3. Development vs Production:**

```yaml
# Rule 1: Auto-fix dev dependencies
Conditions:
  - Scope: Development
  - Severity: Any
Action: Create and merge PR

# Rule 2: Alert for production
Conditions:
  - Scope: Production
  - Severity: Medium+
Action: Create PR + notify team
```

**Links:**
- https://docs.github.com/en/code-security/dependabot/dependabot-auto-triage-rules/about-dependabot-auto-triage-rules
- https://docs.github.com/en/code-security/dependabot/dependabot-auto-triage-rules/customizing-auto-triage-rules-to-prioritize-dependabot-alerts
- https://github.blog/changelog/2024-01-24-dependabot-auto-triage-rules-and-custom-rules-public-beta/

---
<a id="d3-8"></a>
## 3.8 Dependency Review Workflow

### Configure Dependency Review Action

**Basic Configuration:**

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
      - name: 'Checkout Repository'
        uses: actions/checkout@v4
      
      - name: 'Dependency Review'
        uses: actions/dependency-review-action@v4
```

**Advanced Configuration with Thresholds:**

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
      - name: 'Checkout Repository'
        uses: actions/checkout@v4
      
      - name: 'Dependency Review'
        uses: actions/dependency-review-action@v4
        with:
          # Fail only on moderate or higher severity
          fail-on-severity: moderate
          
          # Fail on specific scopes
          fail-on-scopes: runtime, production
          
          # Allowed licenses
          allow-licenses: |
            MIT
            Apache-2.0
            BSD-2-Clause
            BSD-3-Clause
            ISC
            0BSD
          
          # Denied licenses
          deny-licenses: |
            GPL-3.0
            AGPL-3.0
            LGPL-3.0
          
          # Comment summary in PR
          comment-summary-in-pr: always
          
          # Fail on dependencies with low OpenSSF Scorecard level
          warn-on-openssf-scorecard-level: 3
          
          # Configure GitHub token
          github-token: ${{ secrets.GITHUB_TOKEN }}
          
          # Base and head for comparison
          base-ref: ${{ github.event.pull_request.base.sha }}
          head-ref: ${{ github.event.pull_request.head.sha }}
```

### License Configuration

**Define License Policies:**

```yaml
# .github/workflows/dependency-review.yml
- name: 'Dependency Review with License Check'
  uses: actions/dependency-review-action@v4
  with:
    # Allow only permissive open source licenses
    allow-licenses: |
      MIT
      Apache-2.0
      BSD-2-Clause
      BSD-3-Clause
      ISC
      Unlicense
    
    # Deny strict copyleft
    deny-licenses: |
      GPL-2.0
      GPL-3.0
      AGPL-3.0
      LGPL-3.0
    
    # Allow exceptions for specific dependencies
    allow-dependencies-licenses: |
      pkg:npm/some-gpl-package@1.0.0, GPL-3.0
    
    # License check action configuration
    license-check: true
    fail-on-severity: high
```

**Common Licenses Matrix:**

| License | Type | Allowed Corporate Use | Requires Attribution |
|----------|------|---------------------------|---------------------|
| MIT | Permissive | ✅ | ✅ |
| Apache-2.0 | Permissive | ✅ | ✅ |
| BSD-3-Clause | Permissive | ✅ | ✅ |
| ISC | Permissive | ✅ | ✅ |
| GPL-3.0 | Copyleft | ⚠️ | ✅ |
| AGPL-3.0 | Strong Copyleft | ❌ | ✅ |
| Unlicense | Public Domain | ✅ | ❌ |
| Proprietary | Proprietary | ⚠️ | Varies |

### Configure Custom Severity Thresholds

```yaml
# Different policies per project type

# Critical production app
- name: 'Dependency Review - Production'
  uses: actions/dependency-review-action@v4
  with:
    fail-on-severity: low  # Very strict
    fail-on-scopes: runtime
    
# Internal development tool
- name: 'Dependency Review - Dev Tool'
  uses: actions/dependency-review-action@v4
  with:
    fail-on-severity: high  # More relaxed
    fail-on-scopes: runtime, production
```

### Integration with Branch Protection

```yaml
# Configure in GitHub UI
Repository → Settings → Branches → Branch protection rules

Rules for 'main':
  ├─ [✓] Require status checks to pass before merging
  │   └─ Status checks that are required:
  │       ├─ [✓] dependency-review
  │       ├─ [✓] CI / test
  │       └─ [✓] Security / secret-scanning
  │
  ├─ [✓] Require branches to be up to date before merging
  └─ [✓] Do not allow bypassing the above settings
```

### Advanced Use Cases

**1. Different Policies by Directory:**

```yaml
# .github/workflows/dependency-review-frontend.yml
name: 'Dependency Review - Frontend'
on:
  pull_request:
    paths:
      - 'frontend/**'

jobs:
  review-frontend:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/dependency-review-action@v4
        with:
          config-file: '.github/dependency-review-frontend.yml'

# .github/dependency-review-frontend.yml
fail_on_severity: 'moderate'
allow_licenses:
  - 'MIT'
  - 'Apache-2.0'
```

**2. Monitor Without Blocking (Warning Mode):**

```yaml
- name: 'Dependency Review - Warn Only'
  uses: actions/dependency-review-action@v4
  with:
    fail-on-severity: critical  # Only critical blocks
    warn-only: true  # Comment but do not fail
    comment-summary-in-pr: always
```

**3. Custom Output and Reporting:**

```yaml
- name: 'Dependency Review'
  id: review
  uses: actions/dependency-review-action@v4
  with:
    fail-on-severity: high
  
- name: 'Upload results'
  if: always()
  uses: actions/upload-artifact@v4
  with:
    name: dependency-review-results
    path: |
      dependency-review-results.json
      dependency-review-summary.md
  
- name: 'Post to Slack'
  if: failure()
  uses: slackapi/slack-github-action@v1
  with:
    webhook-url: ${{ secrets.SLACK_WEBHOOK }}
    webhook: |
      {
        "text": "❌ Dependency Review failed on PR #${{ github.event.pull_request.number }}",
        "blocks": [
          {
            "type": "section",
            "text": {
              "type": "mrkdwn",
              "text": "New vulnerabilities detected in <${{ github.event.pull_request.html_url }}|PR #${{ github.event.pull_request.number }}>"
            }
          }
        ]
      }
```

**Links:**
- https://docs.github.com/en/code-security/supply-chain-security/understanding-your-software-supply-chain/about-dependency-review
- https://docs.github.com/en/code-security/supply-chain-security/understanding-your-software-supply-chain/configuring-dependency-review
- https://github.com/actions/dependency-review-action
- https://docs.github.com/en/code-security/supply-chain-security/understanding-your-software-supply-chain/dependency-review-enforcement

---
<a id="d3-9"></a>
## 3.9 Identify and correct vulnerable dependencies

### Identify Vulnerabilities

**From Dependabot Alert:**

```
Repository → Security tab → Dependabot alerts

Alert details:
  ├─ Package: lodash
  ├─ Version: 4.17.15
  ├─ Vulnerability: Prototype Pollution (CVE-2021-23337)
  ├─ Severity: High (7.2)
  ├─ Affected versions: < 4.17.21
  ├─ Patched versions: >= 4.17.21
  │
  ├─ Dependency path:
  │   my-app@1.0.0
  │     └─ express@4.16.0
  │         └─ lodash@4.17.15  ← Vulnerable
  │
  └─ Remediation:
      ├─ [View PR] Dependabot security update
      └─ [Dismiss] if false positive
```

**From Pull Request:**

```
PR checks:
  ❌ Dependency Review — Changes introduce known vulnerabilities
  
Details:
  ┌────────────────────────────────────────────────────┐
  │ 🔴 High severity vulnerabilities                   │
  ├────────────────────────────────────────────────────┤
  │ axios@0.21.0 → axios@0.21.4                        │
  │   CVE-2021-3749: Regular Expression Denial of      │
  │   Service in axios                                 │
  │                                                    │
  │ Recommendation: Update to axios@0.27.0 or later    │
  └────────────────────────────────────────────────────┘
```

### Resolve from Security Tab

**Remediation Flow:**

```
1. Go to Security → Dependabot alerts
   
2. Click on a specific alert
   
3. Review details:
   ├─ Read CVE description
   ├─ Understand the impact
   └─ Verify if the affected code is actually used
   
4. Remediation options:
   
   Option A: Accept Dependabot PR
     ├─ Click [Review security update]
     ├─ View PR with changes
     ├─ Verify tests pass
     ├─ [Merge pull request]
     └─ Alert auto-closes
   
   Option B: Update manually
     ├─ Click [Dismiss alert]
     ├─ Reason: "Fix is being prepared"
     ├─ Update package.json:
     │   "lodash": "4.17.15"  →  "lodash": "^4.17.21"
     ├─ npm update lodash
     ├─ git commit & push
     └─ Alert auto-closes
   
   Option C: Remove dependency
     ├─ If not necessary
     ├─ npm uninstall lodash
     ├─ Refactor code to stop using lodash
     ├─ git commit & push
     └─ Alert auto-closes
```

### Resolve from Pull Request

**When Dependency Review blocks the PR:**

```
Scenario: PR introduces axios@0.21.0 (vulnerable)

1. View the error in PR checks:
   ❌ Dependency Review — High severity vulnerability

2. Click on "Details" of the check

3. See report:
   Package: axios@0.21.0
   Vulnerability: CVE-2021-3749
   Fix: axios@0.27.0+

4. Update in the PR:
   # In your local branch
   npm update axios@^0.27.0
   git add package*.json
   git commit -m "fix: Update axios to resolve CVE-2021-3749"
   git push

5. Dependency Review re-runs:
   ✅ Dependency Review — No vulnerabilities detected

6. PR can now be merged
```

### Update Transitive Dependencies

**Problem:** The vulnerability is in an indirect (transitive) dependency.

```
Dependency tree:
  my-app
    └─ express@4.16.0
        └─ lodash@4.17.15  ← Vulnerable (transitive)
```

**Solution 1: Update the parent dependency**

```bash
# Update express to a version that uses a secure lodash version
npm update express
# or
npm install express@latest

# Verify dependency tree
npm list lodash
# my-app@1.0.0
# └─ express@4.18.2
#     └─ lodash@4.17.21  ✅ Fixed
```

**Solution 2: Resolution/override (npm 8.3+)**

```json
// package.json
{
  "overrides": {
    "lodash": "^4.17.21"
  }
}

// For yarn
{
  "resolutions": {
    "lodash": "^4.17.21"
  }
}

// For pnpm
{
  "pnpm": {
    "overrides": {
      "lodash": "^4.17.21"
    }
  }
}
```

**Solution 3: npm audit fix**

```bash
# Auto-fix vulnerabilities
npm audit fix

# Aggressive fix (can break compatibility)
npm audit fix --force

# View what will be fixed without applying
npm audit fix --dry-run

# Generate report
npm audit --json > audit-report.json
```

### Testing After Update

```yaml
# .github/workflows/test-after-update.yml
name: Test Dependabot PR

on:
  pull_request:
    branches: [main]

jobs:
  test:
    if: github.actor == 'dependabot[bot]'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run tests
        run: npm test
      
      - name: Run integration tests
        run: npm run test:integration
      
      - name: Check bundle size
        run: |
          npm run build
          npm run check-bundle-size
      
      - name: Auto-merge if tests pass
        if: success()
        run: gh pr merge --auto --squash "$PR_URL"
        env:
          PR_URL: ${{ github.event.pull_request.html_url }}
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### Merging Dependabot PRs

**Best practices:**

```yaml
✅ DO:
  - Review update changelog
  - Check for breaking changes
  - Run full test suite
  - Check bundle size impact
  - Verify deprecation warnings
  - Test in staging before production

❌ DON'T:
  - Auto-merge major versions without review
  - Ignore test failures
  - Skip security patches
  - Merge without verifying compatibility
```

**Merge Workflow:**

```bash
# 1. Checkout Dependabot PR
gh pr checkout 123

# 2. Verify changes
git diff main HEAD

# 3. Run tests locally
npm test

# 4. Build and verify
npm run build

# 5. If everything OK, merge
gh pr merge 123 --squash --delete-branch

# 6. Verify alert closes
gh api repos/:owner/:repo/dependabot/alerts | \
  jq '.[] | select(.state == "fixed")'
```

**Links:**
- https://docs.github.com/en/code-security/dependabot/working-with-dependabot/managing-pull-requests-for-dependency-updates
- https://docs.github.com/en/code-security/dependabot/dependabot-security-updates/about-dependabot-security-updates
- https://docs.github.com/en/code-security/dependabot/dependabot-security-updates/configuring-dependabot-security-updates
- https://docs.github.com/en/code-security/dependabot/working-with-dependabot/automating-dependabot-with-github-actions
- https://docs.npmjs.com/cli/v8/commands/npm-audit
- https://docs.github.com/en/code-security/supply-chain-security/understanding-your-software-supply-chain/troubleshooting-the-dependency-graph
