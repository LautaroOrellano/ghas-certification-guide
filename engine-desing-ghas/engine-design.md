```Shell
GitHub Security Ecosystem
│
├── 🔧 Base de datos / contexto
│   └── Dependency Graph
│       ├─ Detecta dependencias del repo
│       ├─ Soporta:
│       │   ├─ Dependabot Alerts
│       │   └─ Dependency Review
│       └─ ❗ Sin esto → Dependabot pierde funcionalidad clave
│
├── 🛡️ Detection Layer (detección de vulnerabilidades)
│
│   ├── Code Scanning
│   │   └── CodeQL
│   │       ├─ Analiza código estático
│   │       ├─ Usa data flow / taint tracking
│   │       └─ Detecta:
│   │           ├─ SQL Injection
│   │           ├─ XSS
│   │           └─ Bugs de seguridad
│
│   ├── Dependabot
│   │   ├─ Requiere: Dependency Graph
│   │   ├─ Usa: GitHub Advisory Database
│   │   ├─ Detecta:
│   │   │   └─ Vulnerabilidades en librerías
│   │   └─ Funcionalidades:
│   │       ├─ Dependabot Alerts
│   │       ├─ Dependabot Security Updates
│   │       └─ Dependabot Version Updates
│
│   ├── Secret Scanning
│   │   ├─ Detecta secretos en código
│   │   ├─ Usa:
│   │   │   ├─ Regex patterns
│   │   │   └─ Validadores de partners
│   │   └─ Features:
│   │       ├─ Push Protection
│   │       ├─ Custom patterns (GHAS)
│   │       └─ Partner alerts
│
│   └── Dependency Review
│       ├─ Analiza PRs
│       └─ Detecta nuevas dependencias vulnerables
│
├── ⚙️ Execution Layer
│
│   ├── GitHub Actions
│   │   ├─ Ejecuta:
│   │   │   ├─ CodeQL scans
│   │   │   └─ Dependency review
│   │   └─ ❗ Sin Actions:
│   │       ├─ No hay Code Scanning automático
│   │       └─ No hay Dependency Review
│
│   └── Webhooks / API
│       └─ Integración externa (SIEM, dashboards)
│
├── 🚨 Alerts Layer (resultado)
│
│   ├── Code Scanning Alerts
│   ├── Dependabot Alerts
│   ├── Secret Scanning Alerts
│   └── Dependency Review warnings
│
│   👉 Todas alimentan:
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
│       ├─ Eventos (bypass, cambios, accesos)
│       └─ Base para SIEM
│
├── 🔔 Notification Layer
│
│   ├─ Emails
│   ├─ GitHub UI alerts
│   ├─ PR comments
│   └─ Integraciones (Slack, SIEM)
│
└── 🧠 Governance Layer
    ├─ SLAs (tiempo de remediación)
    ├─ Accountability (assignees / CODEOWNERS)
    └─ Policies de seguridad
```







































