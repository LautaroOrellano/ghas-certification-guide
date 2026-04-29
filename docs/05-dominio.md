# DOMINIO 5: MEJORES PRÁCTICAS, RESULTADOS Y MEDIDAS CORRECTIVAS (10%) {#dominio5}

<a id="d5-1"></a>
## 5.1 Usar CVE y CWE para describir alertas

### CVE (Common Vulnerabilities and Exposures)

**¿Qué es un CVE?**

Identificador único para vulnerabilidades de seguridad conocidas públicamente.

**Formato:**
```
CVE-YYYY-NNNNN

Ejemplo: CVE-2021-44228 (Log4Shell)
  ├─ CVE: Prefijo
  ├─ 2021: Año de publicación
  └─ 44228: Número secuencial
```

**Información de un CVE:**

```yaml
CVE-2021-44228: Log4Shell

Severity: Critical (10.0 CVSS)

Description:
  Apache Log4j2 2.0-beta9 through 2.15.0 (excluding security releases)
  JNDI features used in configuration, log messages, and parameters
  do not protect against attacker controlled LDAP and other JNDI related
  endpoints. An attacker who can control log messages or log message
  parameters can execute arbitrary code loaded from LDAP servers when
  message lookup substitution is enabled.

CVSS Vector: CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:C/C:H/I:H/A:H
  - Attack Vector: Network
  - Attack Complexity: Low
  - Privileges Required: None
  - User Interaction: None
  - Scope: Changed
  - Confidentiality: High
  - Integrity: High
  - Availability: High

Affected:
  - Apache Log4j 2.0-beta9 to 2.15.0

Patched:
  - Apache Log4j 2.17.1+ (Java 8)
  - Apache Log4j 2.12.4 (Java 7)
  - Apache Log4j 2.3.2 (Java 6)

References:
  - https://nvd.nist.gov/vuln/detail/CVE-2021-44228
  - https://logging.apache.org/log4j/2.x/security.html
```

### CWE (Common Weakness Enumeration)

**¿Qué es un CWE?**

Categorización de tipos de debilidades de software que conducen a vulnerabilidades.

**Ejemplos comunes:**

```yaml
CWE-79: Cross-site Scripting (XSS)
  Description: Software no sanitiza input antes de output a HTML
  
  Example:
    <div>Welcome, <%= user.name %></div>
    
    Attack:
      user.name = "<script>alert('XSS')</script>"
    
    Result:
      <div>Welcome, <script>alert('XSS')</script></div>
  
  Fix:
    <div>Welcome, <%= escapeHtml(user.name) %></div>

CWE-89: SQL Injection
  Description: SQL queries construidas con concatenación de input
  
  Example:
    query = "SELECT * FROM users WHERE id = " + userId
    
    Attack:
      userId = "1 OR 1=1"
    
    Result:
      SELECT * FROM users WHERE id = 1 OR 1=1
      # Returns all users
  
  Fix:
    PreparedStatement stmt = conn.prepareStatement(
      "SELECT * FROM users WHERE id = ?"
    );
    stmt.setInt(1, userId);

CWE-22: Path Traversal
  Description: Input usado en file paths sin validación
  
  Example:
    filename = request.getParameter("file")
    file = new File("/uploads/" + filename)
    
    Attack:
      filename = "../../etc/passwd"
    
    Result:
      /uploads/../../etc/passwd → /etc/passwd
  
  Fix:
    Path basePath = Paths.get("/uploads").normalize()
    Path filePath = basePath.resolve(filename).normalize()
    if (!filePath.startsWith(basePath)) {
      throw new SecurityException("Invalid path")
    }

CWE-502: Deserialization of Untrusted Data
  Description: Deserializar objetos de fuentes no confiables
  
  Example:
    ObjectInputStream in = new ObjectInputStream(request.getInputStream());
    User user = (User) in.readObject();
    
    Attack:
      # Payload malicioso que ejecuta código
    
  Fix:
    # Usar JSON en lugar de serialización Java
    # O whitelist de clases permitidas

CWE-798: Hard-coded Credentials
  Description: Credenciales en el código fuente
  
  Example:
    String password = "admin123"
    
  Fix:
    String password = System.getenv("DB_PASSWORD")
```

### Top 25 CWEs más peligrosos (2024)

```yaml
1. CWE-787: Out-of-bounds Write
2. CWE-79: Cross-site Scripting
3. CWE-89: SQL Injection
4. CWE-416: Use After Free
5. CWE-78: OS Command Injection
6. CWE-20: Improper Input Validation
7. CWE-125: Out-of-bounds Read
8. CWE-22: Path Traversal
9. CWE-352: Cross-Site Request Forgery
10. CWE-434: Unrestricted Upload of File with Dangerous Type
11. CWE-862: Missing Authorization
12. CWE-476: NULL Pointer Dereference
13. CWE-287: Improper Authentication
14. CWE-190: Integer Overflow
15. CWE-502: Deserialization of Untrusted Data
16. CWE-77: Command Injection
17. CWE-119: Buffer Errors
18. CWE-798: Hard-coded Credentials
19. CWE-918: Server-Side Request Forgery (SSRF)
20. CWE-306: Missing Authentication
21. CWE-362: Race Condition
22. CWE-269: Improper Privilege Management
23. CWE-94: Code Injection
24. CWE-863: Incorrect Authorization
25. CWE-276: Incorrect Default Permissions
```

### Describir una alerta usando CVE y CWE

**Template de descripción:**

```markdown
## Alert: SQL Injection in UserController

### Identification
- **CWE**: CWE-89 (SQL Injection)
- **CVE**: N/A (vulnerability in custom code, not third-party)
- **Severity**: High (CVSS 8.1)

### Description
User-controlled input from HTTP request parameter 'userId' flows
directly into SQL query without sanitization, enabling SQL injection
attacks.

### Location
- **File**: src/main/java/com/example/UserController.java
- **Line**: 42
- **Method**: getUserById()

### Data Flow
1. **Source**: HttpServletRequest.getParameter("userId")
2. **Flow**: Variable userId concatenated into SQL string
3. **Sink**: Statement.executeQuery(query)

### Impact
An attacker can:
- Bypass authentication
- Read sensitive data (PII, credentials)
- Modify or delete data
- Execute admin operations
- Potential for remote code execution (via xp_cmdshell, etc.)

### Remediation

#### Option 1: Prepared Statement (Recommended)
String query = "SELECT * FROM users WHERE id = ?";
PreparedStatement stmt = connection.prepareStatement(query);
stmt.setInt(1, Integer.parseInt(userId));
ResultSet rs = stmt.executeQuery();


#### Option 2: Input Validation
if (!userId.matches("\\d+")) {
    throw new IllegalArgumentException("Invalid user ID");
}
// Still use prepared statement


#### Option 3: ORM Framework

User user = entityManager.find(User.class, userId);


### References
- CWE-89: https://cwe.mitre.org/data/definitions/89.html
- OWASP SQL Injection: https://owasp.org/www-community/attacks/SQL_Injection
- NIST SP 800-53: SI-10 (Information Input Validation)

### Timeline
- **Discovered**: 2026-04-27
- **SLA**: Fix within 7 days (High severity)
- **Target**: 2026-05-04
```

**Enalces:**
- https://cve.mitre.org/
- https://nvd.nist.gov/vuln
- https://cwe.mitre.org/
- https://www.first.org/cvss/calculator/
- https://cwe.mitre.org/top25/
- https://owasp.org/www-community/vulnerabilities/

---
<a id="d5-2"></a>
## 5.2 Proceso de toma de decisiones para cerrar/descartar alertas

### Framework de decisión

```
┌─────────────────────────────────────────────┐
│ NUEVA ALERTA APARECE                        │
└──────────────┬──────────────────────────────┘
               │
┌──────────────▼──────────────────────────────┐
│ PASO 1: VERIFICAR AUTENTICIDAD              │
│ ¿Es un verdadero positivo?                  │
└──────────────┬──────────────────────────────┘
               │
        ┌──────┴──────┐
       SÍ            NO
        │              │
        │    ┌─────────▼──────────────┐
        │    │ DISMISS: False Positive│
        │    │ Documentar por qué     │
        │    └────────────────────────┘
        │
┌───────▼───────────────────────────────────┐
│ PASO 2: EVALUAR EXPLOTABILIDAD            │
│ ¿El código vulnerable está en uso?        │
│ ¿Es accesible por usuarios/atacantes?     │
└──────────────┬────────────────────────────┘
               │
        ┌──────┴──────┐
       SÍ            NO
        │              │
        │    ┌─────────▼──────────────────┐
        │    │ DISMISS: Used in tests     │
        │    │ O bajo riesgo              │
        │    │ Documentar contexto        │
        │    └────────────────────────────┘
        │
┌───────▼─────────────────────────────────────┐
│ PASO 3: EVALUAR IMPACTO                     │
│ ¿Qué es el worst-case scenario?             │
│ ¿Qué datos están en riesgo?                 │
└──────────────┬──────────────────────────────┘
               │
┌──────────────▼──────────────────────────────┐
│ PASO 4: VERIFICAR MITIGACIONES              │
│ ¿Existen controles compensatorios?          │
│ - WAF rules                                 │
│ - Network segmentation                      │
│ - Input validation en otro layer            │
│ - Authentication/authorization checks       │
└──────────────┬──────────────────────────────┘
               │
        ┌──────┴──────┐
  Mitigaciones       No
   suficientes    mitigado
        │              │
        ▼              ▼
┌─────────────┐  ┌─────────────┐
│ ACCEPT RISK │  │   FIX       │
│ (documented)│  │ (prioritize)│
└─────────────┘  └─────────────┘
```

### Documentación de decisiones

**Template de documentación:**

```yaml
Alert ID: GH-SA-2024-001
Date: 2026-04-27
Decision: DISMISS
Reason: Won't Fix

Context:
  - Finding: SQL Injection in ReportGenerator.java
  - Severity: High
  - CWE: CWE-89

Analysis:
  1. Explotabilidad:
     - Code is in admin-only reporting feature
     - Requires authenticated admin user
     - Admin panel is IP-restricted to corp network
     - All admin actions are logged and monitored
  
  2. Impact if exploited:
     - Attacker would need to:
       a) Compromise admin credentials (2FA enabled)
       b) Access from whitelisted IP
       c) Bypass API rate limiting
     - Data at risk: Reports (already accessible to admins)
  
  3. Existing controls:
     - Network: IP whitelist, VPN required
     - Authentication: SSO + 2FA mandatory
     - Authorization: RBAC with admin role
     - Monitoring: SIEM alerts on admin SQL queries
     - Audit: All actions logged and reviewed weekly

Decision Rationale:
  Risk is ACCEPTED based on:
  - Low likelihood (multiple controls must fail)
  - Limited impact (attacker already has admin access)
  - Cost-benefit: Refactoring legacy report system
    would take 3 months for minimal security gain

Conditions for Acceptance:
  - Annual review of this decision
  - Monitor for any suspicious admin activity
  - Include in next major refactor (Q3 2026)

Approved By:
  - Security Team Lead: @security-lead
  - Engineering Manager: @eng-manager
  - CTO: @cto

Review Date: 2027-04-27
```

### Decisión basada en datos

**Métricas a considerar:**

```yaml
Technical Metrics:
  - CVSS Score: 7.5 (High)
  - Exploitability: 3.9 (High)
  - Attack Complexity: Low
  - Privileges Required: None
  - User Interaction: None

Business Metrics:
  - Data sensitivity: PII (High)
  - System criticality: Payment processing (Critical)
  - User base: 1M+ users
  - Revenue impact: $10M/year
  - Compliance: PCI-DSS, GDPR

Risk Metrics:
  - Likelihood: Medium (some barriers)
  - Impact: High (data breach)
  - Detection: Medium (some monitoring)
  - Response: Fast (incident response ready)

Cost Metrics:
  - Fix effort: 2 developer-days
  - Testing effort: 1 day
  - Deployment risk: Low
  - Cost of breach: $500k-$5M (estimated)

Decision:
  FIX IMMEDIATELY
  - High impact on critical system
  - Low cost to fix
  - Compliance requirements
```

**Enlaces:**
- https://docs.github.com/en/code-security/code-scanning/managing-code-scanning-alerts/managing-code-scanning-alerts-for-your-repository#dismissing-or-deleting-alerts
- https://docs.github.com/en/code-security/secret-scanning/managing-alerts-from-secret-scanning
- https://docs.github.com/en/code-security/dependabot/dependabot-alerts/viewing-and-updating-dependabot-alerts
- https://docs.github.com/en/code-security/security-overview/filtering-alerts-in-security-overview

---
<a id="d5-3"></a>
## 5.3 CodeQL query suites

### Query suites predeterminados

```yaml
security-and-quality:
  Description: Todas las queries de seguridad y calidad
  Use case: Análisis comprehensivo
  Queries: ~300
  Runtime: 15-30 min
  Recomendado para: CI regular

security-extended:
  Description: Todas las queries de seguridad (más cobertura)
  Use case: Security-focused analysis
  Queries: ~200
  Runtime: 10-20 min
  Recomendado para: PRs, security reviews

security:
  Description: Queries de seguridad core (menos false positives)
  Use case: Quick security check
  Queries: ~100
  Runtime: 5-10 min
  Recomendado para: Fast feedback

code-scanning:
  Description: Optimizado para code scanning (balance)
  Use case: GitHub Code Scanning default
  Queries: ~150
  Runtime: 8-15 min
  Recomendado para: Default setup
```

### Comparación de query suites

| Suite | Queries | Security | Quality | False Positives | Runtime |
|-------|---------|----------|---------|-----------------|---------|
| **security** | ~100 | ✅✅✅ | ❌ | Bajo | Rápido |
| **security-extended** | ~200 | ✅✅✅✅ | ❌ | Medio | Medio |
| **security-and-quality** | ~300 | ✅✅✅ | ✅✅✅ | Medio-Alto | Lento |
| **code-scanning** | ~150 | ✅✅✅ | ✅ | Bajo-Medio | Medio |

### Configurar query suite

```yaml
# Default setup
- name: Initialize CodeQL
  uses: github/codeql-action/init@v3
  with:
    queries: security-extended  # Suite recomendado

# Multiple suites
- name: Initialize CodeQL
  uses: github/codeql-action/init@v3
  with:
    queries: |
      security-extended
      +security-and-quality  # '+' agrega queries

# Custom queries
- name: Initialize CodeQL
  uses: github/codeql-action/init@v3
  with:
    queries: |
      security-extended
      codeql/javascript-queries:Experimental/Security/*.ql
      company/custom-queries@main
```

**Enlaces:**
- https://docs.github.com/en/code-security/code-scanning/managing-your-code-scanning-configuration/codeql-query-suites
- https://codeql.github.com/docs/codeql-cli/creating-codeql-query-suites/
- https://github.com/github/codeql/tree/main/suites
- https://docs.github.com/en/code-security/code-scanning/managing-your-code-scanning-configuration/built-in-codeql-query-suites

---
<a id="d5-4"></a>
## 5.4 Cómo CodeQL analiza el código

### Lenguajes compilados vs interpretados

**Lenguajes compilados (Java, C++, C#):**

```
1. EXTRACCIÓN:
   ├─ CodeQL intercepta el compilador
   ├─ Captura AST (Abstract Syntax Tree)
   ├─ Captura type information
   ├─ Captura control flow
   └─ Genera CodeQL database

2. CONSTRUCCIÓN DE DATABASE:
   ├─ Source files → Parse → AST
   ├─ AST → Semantic analysis → Type info
   ├─ Control flow graph
   ├─ Data flow graph
   └─ Call graph

3. ANÁLISIS:
   ├─ Load CodeQL queries
   ├─ Execute queries on database
   ├─ Find patterns (data flow, taint)
   └─ Generate alerts

Ventajas:
  ✅ Type information precisa
  ✅ Control flow completo
  ✅ Call graph exacto
  ✅ Menos false positives

Desventajas:
  ❌ Requiere build
  ❌ Más lento
  ❌ Más complejo setup
```

**Lenguajes interpretados (JavaScript, Python):**

```
1. EXTRACCIÓN:
   ├─ Parse source files directamente
   ├─ No requiere build
   ├─ Infiere types (heurísticas)
   └─ Genera CodeQL database

2. CONSTRUCCIÓN DE DATABASE:
   ├─ Source files → Parse → AST
   ├─ Type inference (best effort)
   ├─ Control flow (basado en sintaxis)
   ├─ Data flow (conservative analysis)
   └─ Call graph (puede ser incompleto)

3. ANÁLISIS:
   ├─ Load CodeQL queries
   ├─ Execute queries on database
   ├─ Handle dynamic features
   └─ Generate alerts

Ventajas:
  ✅ No build required
  ✅ Setup simple
  ✅ Análisis rápido
  ✅ Bueno para scripts

Desventajas:
  ❌ Type info imprecisa
  ❌ Más false positives
  ❌ Puede perder flows dinámicos
```

### Ejemplo de análisis

**Código vulnerable:**

```javascript
// app.js
const express = require('express');
const db = require('./db');

app.get('/user', (req, res) => {
    const userId = req.query.id;  // ← Source
    
    const query = `SELECT * FROM users WHERE id = ${userId}`;  // ← Taint flow
    
    db.execute(query)  // ← Sink
       .then(result => res.json(result));
});
```

**CodeQL analysis:**

```
1. PARSE:
   AST:
     FunctionExpression (line 4)
       └─ BlockStatement
           ├─ VariableDeclaration (line 5)
           │   └─ req.query.id [SOURCE]
           ├─ VariableDeclaration (line 7)
           │   └─ TemplateLiteral [TAINT]
           └─ CallExpression (line 9)
               └─ db.execute [SINK]

2. DATA FLOW:
   userId (line 5)
     → comes from req.query.id
     → marked as TAINTED (user input)
   
   query (line 7)
     → includes ${userId}
     → TAINT propagates
   
   db.execute(query) (line 9)
     → receives TAINTED data
     → MATCHES pattern: SQL sink + tainted data

3. QUERY EXECUTION:
   Query: javascript/sql-injection
   
   Predicates:
     - isSource(req.query.id) ✅
     - isSink(db.execute) ✅
     - flowPath(source → sink) ✅
   
   Result: ALERT GENERATED
     CWE-89: SQL Injection
     Severity: High
```

**Enlaces:**
- https://codeql.github.com/docs/codeql-overview/about-codeql/
- https://codeql.github.com/docs/codeql-language-guides/basic-query-for-javascript-analysis/
- https://codeql.github.com/docs/writing-codeql-queries/about-data-flow-analysis/
- https://codeql.github.com/docs/codeql-cli/creating-codeql-databases/
- https://docs.github.com/en/code-security/code-scanning/introduction-to-code-scanning/about-code-scanning-with-codeql#about-codeql

---
<a id="d5-5"></a>
## 5.5 Roles y responsabilidades

### Development Team

```yaml
Responsibilities:
  - Fix security vulnerabilities found by GHAS
  - Write secure code following best practices
  - Review and merge Dependabot PRs
  - Respond to code scanning alerts in PRs
  - Participate in security training

Activities:
  Daily:
    - Review code scanning results in PRs
    - Merge Dependabot security updates
  
  Weekly:
    - Triage new security alerts
    - Update dependencies
  
  Monthly:
    - Security retrospective
    - Review dismissed alerts

KPIs:
  - Mean Time To Fix (MTTF): < 7 days for high severity
  - % PRs with 0 new vulnerabilities: > 95%
  - Dependabot PR merge rate: > 90%
```

### Security Team

```yaml
Responsibilities:
  - Configure and maintain GHAS features
  - Define security policies and standards
  - Triage and prioritize security findings
  - Provide security guidance to developers
  - Manage exceptions and risk acceptance
  - Security awareness training

Activities:
  Daily:
    - Monitor critical alerts
    - Support developers on security questions
  
  Weekly:
    - Review security metrics
    - Audit dismissed alerts
    - Update custom patterns
  
  Monthly:
    - Security posture report
    - Policy updates
    - Tool configuration review

KPIs:
  - % repositories with GHAS enabled: 100%
  - Critical/high alerts open: < 10
  - Security training completion: 100%
  - Mean Time To Detect (MTTD): < 24 hours
```

### DevOps/Platform Team

```yaml
Responsibilities:
  - Maintain GitHub Actions runners
  - Configure CI/CD pipelines with security checks
  - Manage GitHub organization settings
  - Automate GHAS workflows
  - Monitor system performance

Activities:
  Daily:
    - Monitor workflow executions
    - Troubleshoot failed scans
  
  Weekly:
    - Review Actions usage and costs
    - Update workflow templates
  
  Monthly:
    - Capacity planning
    - Workflow optimization

KPIs:
  - CodeQL scan success rate: > 95%
  - Average scan time: < 15 min
  - Actions minutes usage: within budget
```

### Engineering Leadership

```yaml
Responsibilities:
  - Allocate resources for security work
  - Set security priorities
  - Enforce security gates
  - Sponsor security initiatives
  - Escalate critical findings

Activities:
  Weekly:
    - Review security dashboard
    - Discuss blocking issues
  
  Monthly:
    - Security metrics review
    - Budget for security tooling
  
  Quarterly:
    - Security roadmap planning
    - Risk assessment

KPIs:
  - Security debt trend: decreasing
  - Developer security training: 100%
  - Time allocated to security: 10-15% of sprint
```

**Enlaces:**
- https://docs.github.com/en/organizations/managing-user-access-to-your-organizations-repositories/repository-roles-for-an-organization
- https://docs.github.com/en/organizations/managing-peoples-access-to-your-organization-with-roles/managing-security-managers-in-your-organization
- https://docs.github.com/en/code-security/adopting-github-advanced-security-at-scale/introduction-to-adopting-github-advanced-security-at-scale
- https://resources.github.com/security/adopting-github-advanced-security-at-scale/

---
<a id="d5-6"></a>
## 5.6 Severity thresholds para PR checks

### Configurar thresholds

**Default behavior:**

```yaml
# Sin configuración, code scanning REPORTA pero no BLOQUEA
# Para bloquear, necesitas:

1. Configurar severity threshold en workflow
2. Configurar branch protection rules
```

**Configuración en workflow:**

```yaml
# .github/workflows/codeql.yml
- name: Perform CodeQL Analysis
  uses: github/codeql-action/analyze@v3
  with:
    category: "/language:${{matrix.language}}"
    
    # No bloquea por default
    # Para bloquear, usar GitHub branch protection
```

**Branch protection rules:**

```
Repository → Settings → Branches → Branch protection rule

Protect matching branches: main

✓ Require status checks to pass before merging
  ✓ Require branches to be up to date before merging
  
  Status checks that are required:
    ✓ CodeQL / Analyze (javascript)
    ✓ CodeQL / Analyze (python)

Configurar qué bloquea:
  - Critical/High: Block merge
  - Medium: Warning (no block)
  - Low: Info only
```

**Custom threshold con filtros:**

```yaml
# .github/workflows/codeql-check.yml
name: CodeQL Security Gate

on:
  pull_request:
    branches: [main]

jobs:
  security-gate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Get CodeQL alerts
        id: alerts
        run: |
          gh api graphql -f query='
            query($owner:String!, $name:String!, $pr:Int!) {
              repository(owner:$owner, name:$name) {
                pullRequest(number:$pr) {
                  headRefOid
                }
              }
            }' -f owner=${{ github.repository_owner }} \
               -f name=${{ github.event.repository.name }} \
               -F pr=${{ github.event.pull_request.number }}
          
          # Contar alerts por severidad
          critical=$(gh api /repos/${{ github.repository }}/code-scanning/alerts \
            --jq '[.[] | select(.state=="open" and .rule.security_severity_level=="critical")] | length')
          
          high=$(gh api /repos/${{ github.repository }}/code-scanning/alerts \
            --jq '[.[] | select(.state=="open" and .rule.security_severity_level=="high")] | length')
          
          echo "critical=$critical" >> $GITHUB_OUTPUT
          echo "high=$high" >> $GITHUB_OUTPUT
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      
      - name: Enforce threshold
        if: steps.alerts.outputs.critical != '0' || steps.alerts.outputs.high != '0'
        run: |
          echo "❌ Security gate failed!"
          echo "Critical alerts: ${{ steps.alerts.outputs.critical }}"
          echo "High alerts: ${{ steps.alerts.outputs.high }}"
          echo ""
          echo "Fix all critical and high severity alerts before merging."
          exit 1
```

**Enlaces:**
- https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches
- https://docs.github.com/en/code-security/code-scanning/managing-code-scanning-alerts/triaging-code-scanning-alerts-in-pull-requests
- https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/about-rulesets
- https://docs.github.com/en/code-security/code-scanning/managing-your-code-scanning-configuration/editing-your-configuration-of-default-setup

---
<a id="d5-7"></a>
## 5.7 Filtros y clasificación para priorización

### Secret scanning filters

**Filtrar por validez:**

```yaml
# En Security tab
Filters:
  - Validity: Active  # ← MÁXIMA PRIORIDAD
  - Validity: Inactive
  - Validity: Unknown

Workflow de priorización:
  1. Active secrets: Fix NOW (< 1 hour)
  2. Unknown validity: Investigate and fix (< 24 hours)
  3. Inactive secrets: Clean up code (< 7 days)
```

**Filtrar por tipo:**

```yaml
Priority 1 (Critical):
  - AWS credentials
  - Azure service principals
  - GCP service account keys
  - Database passwords
  - Payment processor keys (Stripe, PayPal)

Priority 2 (High):
  - GitHub tokens
  - OAuth secrets
  - API keys (generic)
  - Private keys (SSH, PGP)

Priority 3 (Medium):
  - Test/development secrets
  - Deprecated keys
  - Limited-scope tokens
```

### Dependabot filters

**Filtrar por severidad + scope:**

```yaml
Priority 1:
  - Severity: Critical
  - Scope: Runtime/Production
  - Fix: Immediate

Priority 2:
  - Severity: High
  - Scope: Runtime/Production
  - Fix: 7 days

Priority 3:
  - Severity: Medium
  - Scope: Production
  - Fix: 30 days

Priority 4:
  - Severity: Low
  - Scope: Any
  - Fix: 90 days

Defer:
  - Scope: Development only
  - Fix: Next release cycle
```

### Code scanning filters

**Múltiples dimensiones:**

```yaml
# En Security → Code scanning

Filters:
  Severity:
    - Critical: Fix immediately
    - High: Fix within sprint
    - Medium: Backlog
    - Low: Technical debt

  CWE Category:
    - Injection (89, 79): High priority
    - Auth (287, 306): High priority
    - Crypto (327, 338): Medium priority

  Location:
    - Production code: High priority
    - Test code: Low priority
    - Example code: Dismiss

  Age:
    - New (< 7 days): High priority
    - Recent (7-30 days): Medium
    - Old (> 30 days): Lower (tech debt)

  Branch:
    - main: Highest priority
    - develop: High priority
    - feature/*: Medium priority
```

### Sorting strategies

```yaml
Sort by:
  1. Severity (desc) → Critical first
  2. Age (asc) → Newest first
  3. Exploitability (desc) → Easy to exploit first
  4. Impact (desc) → High impact first

Example query:
  severity:critical,high
  state:open
  sort:created-desc
  branch:main

Result: Critical & high alerts in main, newest first
```

**Enlaces:**
- https://docs.github.com/en/code-security/security-overview/filtering-alerts-in-security-overview
- https://docs.github.com/en/code-security/secret-scanning/managing-alerts-from-secret-scanning/evaluating-alerts
- https://docs.github.com/en/code-security/dependabot/dependabot-alerts/viewing-and-updating-dependabot-alerts
- https://docs.github.com/en/code-security/code-scanning/managing-code-scanning-alerts/assessing-code-scanning-alerts-for-your-repository

---
<a id="d5-8"></a>
## 5.8 CodeQL y Dependency Review con rulesets

### Repository rulesets

**¿Qué son?**

Nueva forma de configurar branch protection y policies (sucesor de branch protection rules).

**Crear ruleset para code security:**

```
Repository → Settings → Rules → Rulesets

[Create ruleset]

Name: Security Standards
Target: Default branch

Enforcement status: Active

Rules:
  ✓ Require status checks to pass
    Required checks:
      - CodeQL / Analyze (javascript)
      - CodeQL / Analyze (python)
      - Dependency Review
      - Secret Scanning
    
  ✓ Require pull request before merging
    Required approvals: 1
    Dismiss stale approvals: Yes
    Require review from code owners: Yes
  
  ✓ Block force pushes
  
  ✓ Require signed commits
```

**Aplicar rulesets a múltiples repos:**

```yaml
# Organization-level ruleset
Organization Settings → Rules → Rulesets

[Create organization ruleset]

Name: Org-wide Security Standards
Target: All repositories
  - Include: All repositories
  - Exclude: archived repositories

Rules: [same as above]

Bypass:
  - Organization admins
  - Security team
  
  With approval from:
    - 2 approvers required
    - From: Security team
```

### CodeQL con rulesets

**Enforcement:**

```yaml
# Ruleset configuration
Required checks:
  - CodeQL / Analyze (javascript)
  - CodeQL / Analyze (java)
  - CodeQL / Analyze (python)

Merge blocked if:
  ❌ CodeQL scan failed
  ❌ CodeQL found critical/high alerts
  ❌ CodeQL not run yet

Merge allowed if:
  ✅ CodeQL passed (0 critical/high)
  ✅ Medium/low alerts present (warning)
```

### Dependency Review con rulesets

```yaml
Required checks:
  - Dependency Review

Merge blocked if:
  ❌ New critical vulnerabilities
  ❌ New high vulnerabilities (configurable)
  ❌ Disallowed licenses
  ❌ Low OpenSSF Scorecard

Merge allowed if:
  ✅ No new vulnerabilities
  ✅ Only medium/low vulnerabilities
  ✅ All licenses approved
```

**Enlaces:**
- https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/about-rulesets
- https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/creating-rulesets-for-a-repository
- https://docs.github.com/en/organizations/managing-organization-settings/managing-rulesets-for-repositories-in-your-organization
- https://docs.github.com/en/enterprise-cloud@latest/admin/policies/enforcing-policies-for-your-enterprise/enforcing-policies-for-code-security-and-analysis-for-your-enterprise

---
<a id="d5-9"></a>
## 5.9 Configurar escaneo temprano

### Code scanning en PR

**Beneficios:**

```yaml
✅ Catch vulnerabilities antes de merge
✅ Developer context (el código está fresco)
✅ Feedback inmediato
✅ Prevent security debt
✅ Educate developers
```

**Configuración:**

```yaml
# .github/workflows/codeql.yml
name: "CodeQL"

on:
  pull_request:  # ← KEY: Escanea en PRs
    branches: [main, develop]
    paths:
      - '**.js'
      - '**.java'
      - '**.py'

jobs:
  analyze:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0  # Full history para mejor análisis
      
      - uses: github/codeql-action/init@v3
        with:
          languages: javascript, java, python
          queries: security-extended
      
      - uses: github/codeql-action/autobuild@v3
      
      - uses: github/codeql-action/analyze@v3
        with:
          category: "/language:${{matrix.language}}/pr:${{github.event.pull_request.number}}"
```

### Secret scanning push protection

**Habilitar:**

```yaml
Repository Settings
  → Code security and analysis
  → Secret scanning
      └─ [✓] Push protection

Organization Settings (aplicar a todos)
  → Code security
  → Secret scanning
      └─ [✓] Enable push protection for all repositories
```

**Developer experience:**

```bash
$ git push origin feature-branch

remote: 
remote: ❌ PUSH PROTECTION: Secret detected
remote: 
remote: GitHub Personal Access Token found in:
remote:   src/config.js:10
remote: 
remote: This secret is active and can be used to access
remote: your GitHub account.
remote: 
remote: To push anyway:
remote:   1. Revoke the secret
remote:   2. Remove it from the commit history
remote:   3. Request bypass (requires justification)
remote: 
To github.com:org/repo.git
 ! [remote rejected] feature-branch -> feature-branch (push declined due to secret)
```

### Dependency Review en PR

**Configuración:**

```yaml
# .github/workflows/dependency-review.yml
name: 'Dependency Review'

on: 
  pull_request:  # ← Ejecuta en TODOS los PRs
    branches: [main]

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
          fail-on-severity: moderate  # Block medium+
          comment-summary-in-pr: always  # Siempre comentar
```

**PR experience:**

```
Pull Request #123
  ├─ Checks
  │   ├─ ✅ CI / test
  │   ├─ ✅ CodeQL / Analyze (javascript)
  │   ├─ ❌ Dependency Review — 1 high severity vulnerability
  │   └─ ✅ Secret Scanning
  │
  └─ Dependency Review comment:
      
      ⚠️ Dependency Review found 1 high severity vulnerability
      
      | Package | Version | Vulnerability | Severity |
      |---------|---------|---------------|----------|
      | lodash | 4.17.15 | CVE-2021-23337 | High |
      
      Recommendation: Update to lodash@4.17.21
      
      This PR cannot be merged until vulnerabilities are resolved.
```

**Enlaces:**
- https://docs.github.com/en/code-security/code-scanning/creating-an-advanced-setup-for-code-scanning/configuring-advanced-setup-for-code-scanning#configuring-frequency
- https://docs.github.com/en/code-security/secret-scanning/introduction/about-push-protection
- https://docs.github.com/en/code-security/secret-scanning/using-advanced-secret-scanning-and-push-protection-features/push-protection-for-repositories-and-organizations
- https://docs.github.com/en/code-security/supply-chain-security/understanding-your-software-supply-chain/configuring-dependency-review
- https://github.com/actions/dependency-review-action

---























