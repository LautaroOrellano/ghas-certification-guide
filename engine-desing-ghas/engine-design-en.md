> 🌐 **Available in:** **English 🇬🇧** | [Español 🇪🇸](engine-design.md)

---

```Shell
GitHub Security Ecosystem
│
├── 🔧 Database / Context
│   └── Dependency Graph
│       ├─ Detects repo dependencies
│       ├─ Supports:
│       │   ├─ Dependabot Alerts
│       │   └─ Dependency Review
│       └─ ❗ Without this → Dependabot loses core functionality
│
├── 🛡️ Detection Layer (vulnerability detection)
│
│   ├── Code Scanning
│   │   └── CodeQL
│   │       ├─ Analyzes static code
│   │       ├─ Uses data flow / taint tracking
│   │       └─ Detects:
│   │           ├─ SQL Injection
│   │           ├─ XSS
│   │           └─ Security bugs
│
│   ├── Dependabot
│   │   ├─ Requires: Dependency Graph
│   │   ├─ Uses: GitHub Advisory Database
│   │   ├─ Detects:
│   │   │   └─ Vulnerabilities in libraries
│   │   └─ Features:
│   │       ├─ Dependabot Alerts
│   │       ├─ Dependabot Security Updates
│   │       └─ Dependabot Version Updates
│
│   ├── Secret Scanning
│   │   ├─ Detects secrets in code
│   │   ├─ Uses:
│   │   │   ├─ Regex patterns
│   │   │   └─ Partner validators
│   │   └─ Features:
│   │       ├─ Push Protection
│   │       ├─ Custom patterns (GHAS)
│   │       └─ Partner alerts
│
│   └── Dependency Review
│       ├─ Analyzes PRs
│       └─ Detects new vulnerable dependencies
│
├── ⚙️ Execution Layer
│
│   ├── GitHub Actions
│   │   ├─ Runs:
│   │   │   ├─ CodeQL scans
│   │   │   └─ Dependency review
│   │   └─ ❗ Without Actions:
│   │       ├─ No automatic Code Scanning
│   │       └─ No Dependency Review
│   │
│   └── Webhooks / API
│       └─ External integration (SIEM, dashboards)
│
├── 🚨 Alerts Layer (result)
│
│   ├── Code Scanning Alerts
│   ├── Dependabot Alerts
│   ├── Secret Scanning Alerts
│   └── Dependency Review warnings
│
│   👉 All feed into:
│       └─ Security Overview
│
├── 📊 Visualization & Governance
│
│   ├── Security Overview
│   │   ├─ MTTR
│   │   ├─ Coverage
│   │   └─ Trends
│   │
│   └── Audit Log
│       ├─ Events (bypass, changes, access)
│       └─ Base for SIEM
│
├── 🔔 Notification Layer
│
│   ├─ Emails
│   ├─ GitHub UI alerts
│   ├─ PR comments
│   └─ Integrations (Slack, SIEM)
│
└── 🧠 Governance Layer
    ├─ SLAs (remediation time)
    ├─ Accountability (assignees / CODEOWNERS)
    └─ Security policies
```
