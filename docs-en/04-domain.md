> 🌐 **Available in:** **English 🇬🇧** | [Español 🇪🇸](../docs/04-dominio.md)
---

# DOMAIN 4: CONFIGURE AND USE CODE ANALYSIS WITH CODEQL (25%) {#domain4}

<a id="d4-1"></a>
## 4.1 Third-party scanning tools

### Enable Code Scanning for Third-Party Tools

**GitHub supports SARIF-compatible tools:**

```yaml
Compatible tools:
  ├─ Snyk
  ├─ SonarQube/SonarCloud
  ├─ Checkmarx
  ├─ Veracode
  ├─ Fortify
  ├─ Semgrep
  ├─ ESLint (via SARIF formatter)
  └─ Any tool that generates SARIF
```

**Enable Code Scanning (generic):**

```
Repository → Settings → Code security and analysis
  → Code scanning
      └─ [Set up] → Third-party
```

### Comparison: CodeQL vs Third-Party

**Comparative Table:**

| Aspect | CodeQL (GitHub) | Third-Party Tool |
|---------|----------------|---------------------|
| **Setup** | 1-click (default setup) | Requires configuration |
| **Cost** | Included in GHAS | Separate licensing |
| **Hosting** | GitHub-hosted | Self-hosted or SaaS |
| **Languages** | 15+ languages | Varies by tool |
| **Integration** | Native | Via SARIF upload |
| **Queries** | Open source | Proprietary |
| **Customization** | High (custom queries) | Varies |
| **Speed** | Minutes to hours | Varies |
| **False Positives** | Low-medium | Varies |
| **SARIF Support** | Yes | Yes |

### Steps to Use CodeQL

**Default Setup (recommended for most):**

```
1. Repository → Settings → Code security
2. Click [Set up] → Default
3. Select languages (auto-detected)
4. Select query suite (default: security-extended)
5. Click [Enable CodeQL]

✅ Done! GitHub automatically configures everything.
```

**Advanced Setup (for customization):**

```
1. Repository → Security → Code scanning
2. Click [Set up code scanning]
3. Select [Advanced]
4. GitHub creates workflow template:
   .github/workflows/codeql.yml
5. Customize workflow (queries, schedules, etc.)
6. Commit workflow
7. CodeQL runs automatically
```

### Steps to Use a Third-Party Tool

**Example with Snyk:**

```
1. Register on Snyk
2. Connect repository
3. Configure workflow:

# .github/workflows/snyk.yml
name: Snyk Security Scan

on:
  push:
    branches: [main]
  pull_request:

jobs:
  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Run Snyk
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
        with:
          args: --sarif-file-output=snyk.sarif
      
      - name: Upload to GitHub
        uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: snyk.sarif

4. Results appear in the Security tab
```

### Implementation Comparison

**CodeQL on GitHub Actions:**

```yaml
# .github/workflows/codeql.yml
name: "CodeQL"

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  schedule:
    - cron: '0 0 * * 1'

jobs:
  analyze:
    name: Analyze
    runs-on: ubuntu-latest
    permissions:
      security-events: write
      contents: read
    
    strategy:
      matrix:
        language: ['javascript', 'python']
    
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      
      - name: Initialize CodeQL
        uses: github/codeql-action/init@v3
        with:
          languages: ${{ matrix.language }}
          queries: security-extended
      
      - name: Autobuild
        uses: github/codeql-action/autobuild@v3
      
      - name: Perform CodeQL Analysis
        uses: github/codeql-action/analyze@v3
        with:
          category: "/language:${{matrix.language}}"
```

**CodeQL on Third-Party CI (Jenkins):**

```groovy
// Jenkinsfile
pipeline {
    agent any
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('CodeQL Analysis') {
            steps {
                script {
                    // Download CodeQL CLI
                    sh 'wget https://github.com/github/codeql-action/releases/latest/download/codeql-bundle-linux64.tar.gz'
                    sh 'tar -xvzf codeql-bundle-linux64.tar.gz'
                    
                    // Create database
                    sh './codeql/codeql database create mydb --language=java'
                    
                    // Run analysis
                    sh './codeql/codeql database analyze mydb --format=sarif-latest --output=results.sarif'
                    
                    // Upload to GitHub
                    sh '''
                        curl -X POST \
                          -H "Authorization: token ${GITHUB_TOKEN}" \
                          -H "Content-Type: application/json" \
                          https://api.github.com/repos/owner/repo/code-scanning/sarifs \
                          -d @results.sarif
                    '''
                }
            }
        }
    }
}
```

### Upload SARIF from Third-Party Tools

**Via API Endpoint:**

```bash
# Generate SARIF with third-party tool
tool-scan --output=results.sarif

# Compress SARIF (required)
gzip results.sarif

# Upload via API
curl -X POST \
  -H "Authorization: token $GITHUB_TOKEN" \
  -H "Content-Type: application/octet-stream" \
  --data-binary @results.sarif.gz \
  "https://api.github.com/repos/OWNER/REPO/code-scanning/sarifs" \
  -d '{
    "commit_sha": "'$GITHUB_SHA'",
    "ref": "refs/heads/main",
    "sarif": "'$(base64 -w0 results.sarif.gz)'"
  }'
```

**Via GitHub Action:**

```yaml
- name: Upload SARIF
  uses: github/codeql-action/upload-sarif@v3
  with:
    sarif_file: results.sarif
    category: my-custom-tool
```

**SARIF Format:**

```json
{
  "$schema": "https://raw.githubusercontent.com/oasis-tcs/sarif-spec/master/Schemata/sarif-schema-2.1.0.json",
  "version": "2.1.0",
  "runs": [
    {
      "tool": {
        "driver": {
          "name": "MySecurityTool",
          "version": "1.0.0",
          "informationUri": "https://example.com/tool"
        }
      },
      "results": [
        {
          "ruleId": "SQL001",
          "level": "error",
          "message": {
            "text": "SQL injection vulnerability detected"
          },
          "locations": [
            {
              "physicalLocation": {
                "artifactLocation": {
                  "uri": "src/database.js"
                },
                "region": {
                  "startLine": 42,
                  "startColumn": 15,
                  "endLine": 42,
                  "endColumn": 50
                }
              }
            }
          ]
        }
      ]
    }
  ]
}
```

**Links:**
- https://docs.github.com/en/code-security/code-scanning/integrating-with-code-scanning/about-integration-with-code-scanning
- https://docs.github.com/en/code-security/code-scanning/integrating-with-code-scanning/uploading-a-sarif-file-to-github
- https://docs.github.com/en/code-security/code-scanning/integrating-with-code-scanning/sarif-support-for-code-scanning
- https://docs.oasis-open.org/sarif/sarif/v2.1.0/sarif-v2.1.0.html
- https://github.com/github/codeql-action/tree/main/upload-sarif

---
<a id="d4-2"></a>
## 4.2 Describe and enable code scanning

### Role in the SDLC

```
┌──────────────────────────────────────────────────┐
│ CODE SCANNING IN THE DEVELOPMENT CYCLE           │
└──────────────────────────────────────────────────┘

1. DEVELOPMENT
   ├─ Local CodeQL CLI (pre-commit)
   ├─ IDE extensions (real-time)
   └─ Pre-push hooks

2. COMMIT & PUSH
   ├─ CodeQL runs in PR
   ├─ Results in <15 min
   └─ Alerts in PR checks

3. CODE REVIEW
   ├─ Reviewer sees CodeQL alerts
   ├─ Discuss vulnerabilities in-line
   └─ Block merge if critical

4. MERGE TO MAIN
   ├─ Full scan on main branch
   ├─ Baseline established
   └─ Security debt tracking

5. SCHEDULED SCANS
   ├─ Weekly full scan
   ├─ Detect new vulnerabilities in existing code
   └─ Monitor third-party advisories

6. RELEASE
   ├─ Verify 0 critical/high issues before deploy
   ├─ Generate security report
   └─ SBOM includes code scan results
```

### Frequency of Workflows

**On Push (every commit):**

```yaml
on:
  push:
    branches: [main, develop]

# Pros:
✅ Immediate bug detection
✅ Fast feedback
✅ Prevents accumulation of issues

# Cons:
❌ Consumes GitHub Actions minutes
❌ Can be slow in large repos
❌ Many scans if team is large
```

**On Pull Request (recommended):**

```yaml
on:
  pull_request:
    branches: [main]

# Pros:
✅ Catch issues before merge
✅ Code review context
✅ Fewer scans than on-push
✅ Required status check

# Cons:
❌ Does not scan main continuously
❌ False sense of security if no PRs are used
```

**Scheduled (complementary):**

```yaml
on:
  schedule:
    - cron: '0 0 * * 1'  # Monday 00:00

# Pros:
✅ Detects new vulnerabilities in existing code
✅ New CodeQL queries are included
✅ Predictable minutes consumption
✅ Does not affect developer flow

# Cons:
❌ Late feedback
❌ Can accumulate issues
```

**Comparison of Frequencies:**

| Trigger | When it runs | Use case | Minutes consumption |
|---------|---------------|----------|---------------------|
| **push** | Every commit | Fast CI/CD | High |
| **pull_request** | In PRs | Gate for merge | Medium |
| **schedule** | Weekly | Maintenance | Low |
| **workflow_dispatch** | Manual | Testing, audits | Minimum |
| **push + schedule** | Both | Hybrid (recommended) | Medium-high |

### Select Trigger Events

**Pattern 1: Active Development**

```yaml
# For large teams, fast development
on:
  pull_request:
    branches: [main]
    paths:
      - '**.java'
      - '**.js'
      - '!tests/**'  # Exclude tests
  schedule:
    - cron: '0 2 * * *'  # Daily 2 AM
```

**Pattern 2: Critical Project**

```yaml
# Maximum security
on:
  push:
    branches: [main, release/*]
  pull_request:
    branches: [main]
  schedule:
    - cron: '0 0 * * 1,4'  # Monday and Thursday
```

**Pattern 3: Public Open Source**

```yaml
# Balance security and minutes
on:
  pull_request:
    branches: [main]
  push:
    branches: [main]
    paths:
      - 'src/**'
  schedule:
    - cron: '0 0 * * 0'  # Sunday
```

**Trigger by Specific Paths:**

```yaml
on:
  push:
    branches: [main]
    paths:
      # Include
      - 'src/**'
      - 'lib/**'
      # Exclude
      - '!docs/**'
      - '!**.md'
      - '!tests/**'
```

### Edit CodeQL Workflow

**Default Template:**

```yaml
# .github/workflows/codeql.yml
name: "CodeQL"

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]
  schedule:
    - cron: '0 0 * * 1'

jobs:
  analyze:
    name: Analyze
    runs-on: ubuntu-latest
    permissions:
      actions: read
      contents: read
      security-events: write

    strategy:
      fail-fast: false
      matrix:
        language: [ 'javascript', 'python' ]

    steps:
    - name: Checkout repository
      uses: actions/checkout@v4

    - name: Initialize CodeQL
      uses: github/codeql-action/init@v3
      with:
        languages: ${{ matrix.language }}

    - name: Autobuild
      uses: github/codeql-action/autobuild@v3

    - name: Perform CodeQL Analysis
      uses: github/codeql-action/analyze@v3
      with:
        category: "/language:${{matrix.language}}"
```

**Production Customization:**

```yaml
name: "CodeQL Advanced"

on:
  push:
    branches: [main, release/*]
    paths:
      - 'src/**'
      - 'lib/**'
  pull_request:
    branches: [main]
  schedule:
    - cron: '0 2 * * 1'  # Monday 2 AM
  workflow_dispatch:  # Manual trigger

jobs:
  analyze:
    name: Analyze (${{ matrix.language }})
    runs-on: ${{ matrix.os }}
    timeout-minutes: 360
    
    permissions:
      actions: read
      contents: read
      security-events: write
      pull-requests: write  # To comment on PRs
    
    strategy:
      fail-fast: false
      matrix:
        include:
          - language: javascript
            os: ubuntu-latest
            build-mode: none
          - language: java
            os: ubuntu-latest
            build-mode: manual
          - language: python
            os: ubuntu-latest
            build-mode: none
    
    steps:
    - name: Checkout repository
      uses: actions/checkout@v4
      with:
        fetch-depth: 0  # Full history for better analysis
    
    - name: Setup Java
      if: matrix.language == 'java'
      uses: actions/setup-java@v4
      with:
        distribution: 'temurin'
        java-version: '17'
        cache: 'maven'
    
    - name: Initialize CodeQL
      uses: github/codeql-action/init@v3
      with:
        languages: ${{ matrix.language }}
        
        # Query suites
        queries: +security-and-quality
        
        # Config file (optional)
        config-file: ./.github/codeql/codeql-config.yml
        
        # Additional packs
        packs: |
          codeql/javascript-queries
          company/custom-queries
    
    # Manual build for compiled languages
    - name: Build Java
      if: matrix.language == 'java' && matrix.build-mode == 'manual'
      run: |
        mvn clean install -DskipTests
    
    - name: Perform CodeQL Analysis
      uses: github/codeql-action/analyze@v3
      with:
        category: "/language:${{matrix.language}}"
        output: sarif-results
        upload: true
        
    - name: Filter SARIF
      if: github.event_name == 'pull_request'
      uses: advanced-security/filter-sarif@v1
      with:
        patterns: |
          -**/tests/**
          -**/node_modules/**
        input: sarif-results/${{ matrix.language }}.sarif
        output: filtered.sarif
    
    - name: Upload filtered SARIF
      if: github.event_name == 'pull_request'
      uses: github/codeql-action/upload-sarif@v3
      with:
        sarif_file: filtered.sarif
```

### View Code Scanning Results

**Security Tab:**

```
Repository → Security → Code scanning

Available views:
  ├─ Open alerts (default)
  ├─ Closed alerts
  ├─ Dismissed alerts
  └─ Fixed alerts

Filters:
  ├─ Severity: Critical, High, Medium, Low
  ├─ Tool: CodeQL, Snyk, etc.
  ├─ Branch: main, develop, etc.
  ├─ Language: Java, JavaScript, etc.
  └─ Rule: CWE-89, CWE-79, etc.
```

**Pull Request:**

```
PR → Checks → CodeQL

Status:
  ✅ CodeQL / Analyze (javascript) — No new alerts
  ❌ CodeQL / Analyze (java) — 2 new alerts found
  
Details:
  ├─ 1 high severity
  │   └─ SQL Injection in UserController.java:42
  │
  └─ 1 medium severity
      └─ Path traversal in FileHandler.java:15
```

**Alert Details:**

```
Click on alert to view:

┌─────────────────────────────────────────────────┐
│ 🔴 SQL Injection (CWE-89)                       │
├─────────────────────────────────────────────────┤
│ Severity: High                                  │
│ Security: 8.1 (CVSS)                            │
│ Rule: java/sql-injection                        │
│                                                 │
│ Location:                                       │
│   File: src/UserController.java:42              │
│   Method: getUser()                             │
│                                                 │
│ Data flow:                                      │
│   Source: request.getParameter("id")            │
│         ↓                                       │
│   Sink: executeQuery(query)                     │
│                                                 │
│ [Show paths] [Dismiss] [Create issue]           │
└─────────────────────────────────────────────────┘
```

**Links:**
- https://docs.github.com/en/code-security/code-scanning/introduction-to-code-scanning/about-code-scanning
- https://docs.github.com/en/code-security/code-scanning/enabling-code-scanning/configuring-default-setup-for-code-scanning
- https://docs.github.com/en/code-security/code-scanning/creating-an-advanced-setup-for-code-scanning/configuring-advanced-setup-for-code-scanning
- https://docs.github.com/en/code-security/code-scanning/managing-code-scanning-alerts/about-code-scanning-alerts
- https://docs.github.com/en/code-security/code-scanning/managing-code-scanning-alerts/managing-code-scanning-alerts-for-your-repository

---
<a id="d4-3"></a>
## 4.3 Troubleshooting CodeQL workflows

### Common Errors and Solutions

**Error 1: "No language detected"**

```yaml
Error:
  No CodeQL languages found to analyze

Cause:
  - No code in supported languages
  - Incorrect paths in workflow

Solution:
# Specify languages explicitly
- name: Initialize CodeQL
  uses: github/codeql-action/init@v3
  with:
    languages: javascript, python
```

**Error 2: "Build failed"**

```yaml
Error:
  Autobuild failed for language 'java'

Cause:
  - Missing dependencies
  - Incorrect build command
  - Timeout

Solution 1: Manual build
- name: Build
  run: |
    mvn clean compile -DskipTests

Solution 2: Specify build command
- name: Initialize CodeQL
  with:
    languages: java
    build-mode: manual
    
- run: mvn clean compile

Solution 3: Increase timeout
jobs:
  analyze:
    timeout-minutes: 360  # 6 hours
```

**Error 3: "Database creation failed"**

```yaml
Error:
  CodeQL database creation failed

Cause:
  - Malformed code
  - Circular dependencies
  - Out of memory

Solution:
# Increase memory
jobs:
  analyze:
    runs-on: ubuntu-latest-large  # More CPU/RAM
    
# Or self-hosted runner with more resources
    runs-on: [self-hosted, linux, x64, large]
```

### Custom Configuration File

**Create Custom Config:**

```yaml
# .github/codeql/codeql-config.yml
name: "CodeQL Custom Config"

# Disable default queries
disable-default-queries: false

# Queries to run
queries:
  - uses: security-and-quality
  - uses: security-extended

# Custom queries
query-filters:
  - exclude:
      id: js/useless-expression

# Paths to ignore
paths-ignore:
  - 'tests/**'
  - 'docs/**'
  - '**/node_modules'
  - 'vendor/**'

# Paths to include
paths:
  - 'src/**'
  - 'lib/**'

# Query packs
packs:
  javascript:
    - codeql/javascript-queries
    - company/custom-js-queries@1.0.0
  java:
    - codeql/java-queries
```

**Use Config in Workflow:**

```yaml
- name: Initialize CodeQL
  uses: github/codeql-action/init@v3
  with:
    languages: ${{ matrix.language }}
    config-file: ./.github/codeql/codeql-config.yml
```

### Show Paths (Data Flow)

**What is "Show Paths"?**

A feature that displays the data flow from the source (user input) to the sink (vulnerable function) of a vulnerability.

**Visual Example:**

```java
// UserController.java

public User getUser(HttpRequest request) {
    // ① Source: User input
    String userId = request.getParameter("id");
    
    // ② Flow: Variable assignment
    String query = "SELECT * FROM users WHERE id = " + userId;
    
    // ③ Sink: SQL execution
    return database.executeQuery(query);  // ← VULNERABLE
}
```

**Show Paths Displays:**

```
Path 1 of 1 for SQL Injection

Step 1: Source
  Location: UserController.java:23
  request.getParameter("id")
  ↓
  Type: HttpServletRequest parameter

Step 2: Flow through concatenation
  Location: UserController.java:24
  "SELECT * FROM users WHERE id = " + userId
  ↓
  Taint preserved through string concatenation

Step 3: Sink
  Location: UserController.java:25
  database.executeQuery(query)
  ↓
  Unsanitized data flows into SQL query

Recommendation:
  Use prepared statements:
  PreparedStatement stmt = conn.prepareStatement(
    "SELECT * FROM users WHERE id = ?"
  );
  stmt.setString(1, userId);
```

### Alert Documentation

**Each alert includes:**

```
Alert: SQL Injection

CWE: CWE-89
  Common Weakness Enumeration
  "Improper Neutralization of Special Elements in SQL Command"
  
CVSS: 8.1 (High)
  Attack Vector: Network
  Attack Complexity: Low
  Privileges Required: None
  User Interaction: None
  Scope: Unchanged
  Confidentiality: High
  Integrity: High
  Availability: Low

Description:
  User-controlled data flows into a SQL query without
  sanitization, allowing SQL injection attacks.

Recommendation:
  1. Use parameterized queries (prepared statements)
  2. Validate/sanitize all user input
  3. Use ORM frameworks
  4. Principle of least privilege for DB user

Examples:
  [Click to see code examples]

References:
  - OWASP SQL Injection
  - CWE-89
  - MITRE ATT&CK T1190
```

### Determining if an Alert Should be Dismissed

**Dismissal Criteria:**

```yaml
✅ Dismiss if:
  - False positive (code is not actually vulnerable)
  - Code is in test files (not production)
  - Vulnerability is not exploitable (contextual mitigation)
  - Risk accepted (documented exception)
  - Will fix later (added to backlog)

❌ DO NOT dismiss if:
  - It is a true positive
  - It is in production code
  - It is exploitable
  - No mitigation exists
  - Critical/high severity without analysis
```

**Dismissal Reasons:**

```
Repository → Security → Code scanning → Alert

[Dismiss alert]
  ├─ False positive
  │   └─ "CodeQL flagged this but it's not actually vulnerable"
  │
  ├─ Won't fix
  │   └─ "Risk accepted, not fixing"
  │
  ├─ Used in tests
  │   └─ "This code is only in test files"
  │
  └─ Won't fix (other reason)
      └─ Custom reason required
```

**Document Dismissal:**

```
Reason: Won't fix
Comment:
  This SQL query is internal-only and the input comes from 
  a trusted admin interface with authentication & authorization.
  
  Risk accepted by: Security Team
  Date: 2026-04-27
  Review date: 2027-04-27
  
  Additional context:
  - Query runs in read-only replica
  - User must have admin role
  - All admin actions are logged
  - WAF rules prevent SQL metacharacters
```

**Links:**
- https://docs.github.com/en/code-security/code-scanning/troubleshooting-code-scanning
- https://docs.github.com/en/code-security/code-scanning/creating-an-advanced-setup-for-code-scanning/codeql-code-scanning-for-compiled-languages
- https://docs.github.com/en/code-security/code-scanning/creating-an-advanced-setup-for-code-scanning/customizing-your-advanced-setup-for-code-scanning
- https://codeql.github.com/docs/codeql-cli/creating-codeql-databases/
- https://docs.github.com/en/code-security/code-scanning/troubleshooting-code-scanning/server-error
- https://docs.github.com/en/code-security/code-scanning/troubleshooting-code-scanning/not-found-errors

---
<a id="d4-4"></a>
## 4.4 CodeQL internals

### Limitations of CodeQL

**For Compiled Languages:**

```yaml
Compiled Languages (Java, C++, C#, Go):
  Require:
    - ✅ Full build
    - ✅ All dependencies
    - ✅ Successful compilation
    - ✅ Longer analysis time
  
  Limitations:
    - ❌ Cannot analyze code that does not compile
    - ❌ Requires build configuration
    - ❌ Higher resource consumption
    - ❌ Slower (minutes to hours)
```

**For Interpreted Languages:**

```yaml
Interpreted Languages (JavaScript, Python, Ruby):
  Advantages:
    - ✅ No build required
    - ✅ Faster analysis
    - ✅ Fewer resources
    - ✅ Simpler setup
  
  Limitations:
    - ⚠️ Limited static analysis
    - ⚠️ Less precise type inference
    - ⚠️ May miss some dynamic flows
```

### Language Compatibility

**Support Matrix:**

| Language | Support | Build Mode | Typical Time |
|----------|---------|------------|---------------|
| **JavaScript/TypeScript** | ✅ Full | none | 2-10 min |
| **Python** | ✅ Full | none | 2-10 min |
| **Java** | ✅ Full | manual/autobuild | 10-60 min |
| **C#** | ✅ Full | manual/autobuild | 10-60 min |
| **C/C++** | ✅ Full | manual/autobuild | 20-120 min |
| **Go** | ✅ Full | manual/autobuild | 5-30 min |
| **Ruby** | ✅ Full | none | 2-10 min |
| **Kotlin** | ✅ Via java-kotlin | manual/autobuild | 10-60 min |
| **Swift** | ✅ Full | manual/autobuild | 10-60 min |
| **Rust** | ⚠️ Beta | manual | 10-60 min |
| **PHP** | ❌ Not supported | - | - |
| **Scala** | ❌ Not supported | - | - |

**Supported Frameworks by Language:**

```yaml
JavaScript/TypeScript:
  - Express.js ✅
  - React ✅
  - Angular ✅
  - Vue ✅
  - Node.js ✅
  - Next.js ✅

Python:
  - Django ✅
  - Flask ✅
  - FastAPI ✅
  - Tornado ✅

Java:
  - Spring Boot ✅
  - Jakarta EE ✅
  - Struts ✅
  - Play Framework ✅

C#:
  - ASP.NET Core ✅
  - .NET Framework ✅
  - Entity Framework ✅
```

### Purpose of SARIF Category

**What is a SARIF Category?**

A unique identifier that groups results from a specific analysis.

**Why it matters:**

```yaml
Without Category:
  - CodeQL runs multiple times
  - Results overwrite each other
  - You only see the latest scan
  - Historical context is lost

With Category:
  - Each analysis has a unique ID
  - Results are accumulated
  - You see the temporal evolution
  - You can compare branches/languages
```

**Usage Example:**

```yaml
# Multiple analyses in the same workflow

- name: CodeQL Analysis - Security
  uses: github/codeql-action/analyze@v3
  with:
    category: "/language:${{matrix.language}}/suite:security"

- name: CodeQL Analysis - Quality
  uses: github/codeql-action/analyze@v3
  with:
    category: "/language:${{matrix.language}}/suite:quality"

# Result: Two separate sets of alerts
```

**Naming Conventions:**

```yaml
Recommended Categories:

By Language:
  "/language:javascript"
  "/language:python"

By Suite:
  "/suite:security-extended"
  "/suite:security-and-quality"

By Branch:
  "/branch:main"
  "/branch:develop"

By Environment:
  "/env:production"
  "/env:staging"

Combined:
  "/language:java/suite:security/branch:main"
```

**Links:**
- https://codeql.github.com/docs/codeql-overview/about-codeql/
- https://codeql.github.com/docs/codeql-language-guides/
- https://docs.github.com/en/code-security/code-scanning/introduction-to-code-scanning/about-code-scanning-with-codeql
- https://codeql.github.com/docs/codeql-cli/about-codeql-databases/
- https://docs.github.com/en/code-security/code-scanning/managing-your-code-scanning-configuration/codeql-query-suites
- https://codeql.github.com/docs/writing-codeql-queries/
