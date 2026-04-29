# DOMINIO 4: CONFIGURAR Y USAR EL ANÁLISIS DE CÓDIGO CON CODEQL (25%) {#dominio4}

## 4.1 Herramientas de escaneo de terceros

### Habilitar code scanning para terceros

**GitHub soporta herramientas SARIF-compatible:**

```yaml
Herramientas compatibles:
  ├─ Snyk
  ├─ SonarQube/SonarCloud
  ├─ Checkmarx
  ├─ Veracode
  ├─ Fortify
  ├─ Semgrep
  ├─ ESLint (via SARIF formatter)
  └─ Cualquier tool que genere SARIF
```

**Habilitar code scanning (generic):**

```
Repository → Settings → Code security and analysis
  → Code scanning
      └─ [Set up] → Third-party
```

### Comparación CodeQL vs Terceros

**Tabla comparativa:**

| Aspecto | CodeQL (GitHub) | Herramienta tercera |
|---------|----------------|---------------------|
| **Setup** | 1-click (default setup) | Requiere configuración |
| **Cost** | Incluido en GHAS | Licencia aparte |
| **Hosting** | GitHub-hosted | Self-hosted o SaaS |
| **Languages** | 15+ lenguajes | Varía por tool |
| **Integration** | Nativa | Via SARIF upload |
| **Queries** | Open source | Propietarias |
| **Customization** | Alta (queries custom) | Varía |
| **Speed** | Minutos-horas | Varía |
| **False positives** | Bajo-medio | Varía |
| **Sarif support** | Sí | Sí |

### Pasos para usar CodeQL

**Default setup (recomendado para mayoría):**

```
1. Repository → Settings → Code security
2. Click [Set up] → Default
3. Select languages (auto-detected)
4. Select query suite (default: security-extended)
5. Click [Enable CodeQL]

✅ Done! GitHub configura todo automáticamente.
```

**Advanced setup (para customización):**

```
1. Repository → Security → Code scanning
2. Click [Set up code scanning]
3. Select [Advanced]
4. GitHub crea workflow template:
   .github/workflows/codeql.yml
5. Customize workflow (queries, schedules, etc.)
6. Commit workflow
7. CodeQL ejecuta automáticamente
```

### Pasos para usar herramienta tercera

**Ejemplo con Snyk:**

```
1. Registrar en Snyk
2. Conectar repository
3. Configurar workflow:

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

4. Results appear in Security tab
```

### Comparación de implementación

**CodeQL en GitHub Actions:**

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

**CodeQL en CI tercero (Jenkins):**

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

### Upload SARIF desde terceros

**Via API endpoint:**

```bash
# Generar SARIF con herramienta tercera
tool-scan --output=results.sarif

# Comprimir SARIF (requerido)
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

**Formato SARIF:**

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

**Enlaces:**
- https://docs.github.com/en/code-security/code-scanning/integrating-with-code-scanning/about-integration-with-code-scanning
- https://docs.github.com/en/code-security/code-scanning/integrating-with-code-scanning/uploading-a-sarif-file-to-github
- https://docs.github.com/en/code-security/code-scanning/integrating-with-code-scanning/sarif-support-for-code-scanning
- https://docs.oasis-open.org/sarif/sarif/v2.1.0/sarif-v2.1.0.html
- https://github.com/github/codeql-action/tree/main/upload-sarif

---

## 4.2 Describir y habilitar code scanning

### Rol en el SDLC

```
┌──────────────────────────────────────────────────┐
│ CODE SCANNING EN EL CICLO DE DESARROLLO          │
└──────────────────────────────────────────────────┘

1. DEVELOPMENT
   ├─ CodeQL CLI local (pre-commit)
   ├─ IDE extensions (real-time)
   └─ Pre-push hooks

2. COMMIT & PUSH
   ├─ CodeQL ejecuta en PR
   ├─ Results en <15 min
   └─ Alerts en PR checks

3. CODE REVIEW
   ├─ Reviewer ve CodeQL alerts
   ├─ Discuss vulnerabilities in-line
   └─ Block merge si critical

4. MERGE TO MAIN
   ├─ Full scan en main branch
   ├─ Baseline establecido
   └─ Tracking de security debt

5. SCHEDULED SCANS
   ├─ Weekly full scan
   ├─ Detect new vulnerabilities en código existente
   └─ Monitor third-party advisories

6. RELEASE
   ├─ Verify 0 critical/high antes de deploy
   ├─ Generate security report
   └─ SBOM incluye code scan results
```

### Frecuencia de workflows

**On Push (cada commit):**

```yaml
on:
  push:
    branches: [main, develop]

# Pros:
✅ Detección inmediata de bugs
✅ Fast feedback
✅ Prevents accumulation of issues

# Cons:
❌ Consume GitHub Actions minutes
❌ Puede ser lento en repos grandes
❌ Muchos scans si equipo grande
```

**On Pull Request (recomendado):**

```yaml
on:
  pull_request:
    branches: [main]

# Pros:
✅ Catch antes de merge
✅ Code review context
✅ Menos scans que on-push
✅ Required status check

# Cons:
❌ No escanea main continuamente
❌ False sense of security si no PRs
```

**Scheduled (complementario):**

```yaml
on:
  schedule:
    - cron: '0 0 * * 1'  # Lunes 00:00

# Pros:
✅ Detecta nuevas vulnerabilidades en código existente
✅ Nuevas queries de CodeQL
✅ Predecible consumption de minutes
✅ No afecta developer flow

# Cons:
❌ Feedback tardío
❌ Puede acumular issues
```

**Comparación de frecuencias:**

| Trigger | Cuando ejecuta | Use case | Minutes consumption |
|---------|---------------|----------|---------------------|
| **push** | Cada commit | CI/CD rápido | Alto |
| **pull_request** | En PRs | Gate para merge | Medio |
| **schedule** | Semanalmente | Mantenimiento | Bajo |
| **workflow_dispatch** | Manual | Testing, audits | Mínimo |
| **push + schedule** | Ambos | Híbrido (recomendado) | Medio-alto |

### Seleccionar eventos trigger

**Pattern 1: Desarrollo activo**

```yaml
# Para equipos grandes, desarrollo rápido
on:
  pull_request:
    branches: [main]
    paths:
      - '**.java'
      - '**.js'
      - '!tests/**'  # Excluir tests
  schedule:
    - cron: '0 2 * * *'  # Daily 2 AM
```

**Pattern 2: Proyecto crítico**

```yaml
# Máxima seguridad
on:
  push:
    branches: [main, release/*]
  pull_request:
    branches: [main]
  schedule:
    - cron: '0 0 * * 1,4'  # Lunes y Jueves
```

**Pattern 3: Open source público**

```yaml
# Balance seguridad y minutes
on:
  pull_request:
    branches: [main]
  push:
    branches: [main]
    paths:
      - 'src/**'
  schedule:
    - cron: '0 0 * * 0'  # Domingo
```

**Trigger por paths específicos:**

```yaml
on:
  push:
    branches: [main]
    paths:
      # Incluir
      - 'src/**'
      - 'lib/**'
      # Excluir
      - '!docs/**'
      - '!**.md'
      - '!tests/**'
```

### Editar workflow de CodeQL

**Template predeterminado:**

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

**Customización para producción:**

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
    - cron: '0 2 * * 1'  # Lunes 2 AM
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
      pull-requests: write  # Para comentar en PRs
    
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
        fetch-depth: 0  # Full history para mejor análisis
    
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
        
        # Config file (opcional)
        config-file: ./.github/codeql/codeql-config.yml
        
        # Packs adicionales
        packs: |
          codeql/javascript-queries
          company/custom-queries
    
    # Manual build para lenguajes compilados
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

### Ver resultados de code scanning

**Security tab:**

```
Repository → Security → Code scanning

Views disponibles:
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

**Alert details:**

```
Click on alert para ver:

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
│ [Show paths] [Dismiss] [Create issue]          │
└─────────────────────────────────────────────────┘
```

**Enlaces:**
- https://docs.github.com/en/code-security/code-scanning/introduction-to-code-scanning/about-code-scanning
- https://docs.github.com/en/code-security/code-scanning/enabling-code-scanning/configuring-default-setup-for-code-scanning
- https://docs.github.com/en/code-security/code-scanning/creating-an-advanced-setup-for-code-scanning/configuring-advanced-setup-for-code-scanning
- https://docs.github.com/en/code-security/code-scanning/managing-code-scanning-alerts/about-code-scanning-alerts
- https://docs.github.com/en/code-security/code-scanning/managing-code-scanning-alerts/managing-code-scanning-alerts-for-your-repository

---

## 4.3 Troubleshooting de workflows CodeQL

### Errores comunes y soluciones

**Error 1: "No language detected"**

```yaml
Error:
  No CodeQL languages found to analyze

Causa:
  - No hay código en lenguajes soportados
  - Paths incorrectos en workflow

Solución:
# Especificar languages explícitamente
- name: Initialize CodeQL
  uses: github/codeql-action/init@v3
  with:
    languages: javascript, python
```

**Error 2: "Build failed"**

```yaml
Error:
  Autobuild failed for language 'java'

Causa:
  - Dependencias faltantes
  - Comando de build incorrecto
  - Timeout

Solución 1: Manual build
- name: Build
  run: |
    mvn clean compile -DskipTests

Solución 2: Especificar build command
- name: Initialize CodeQL
  with:
    languages: java
    build-mode: manual
    
- run: mvn clean compile

Solución 3: Aumentar timeout
jobs:
  analyze:
    timeout-minutes: 360  # 6 hours
```

**Error 3: "Database creation failed"**

```yaml
Error:
  CodeQL database creation failed

Causa:
  - Código mal formado
  - Dependencias circulares
  - Out of memory

Solución:
# Aumentar memoria
jobs:
  analyze:
    runs-on: ubuntu-latest-large  # Más CPU/RAM
    
# O self-hosted runner con más recursos
    runs-on: [self-hosted, linux, x64, large]
```

### Custom configuration file

**Crear config personalizado:**

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

**Usar config en workflow:**

```yaml
- name: Initialize CodeQL
  uses: github/codeql-action/init@v3
  with:
    languages: ${{ matrix.language }}
    config-file: ./.github/codeql/codeql-config.yml
```

### Show paths (data flow)

**¿Qué es "show paths"?**

Feature que muestra el flujo de datos desde la fuente (source) hasta el sumidero (sink) de una vulnerabilidad.

**Ejemplo visual:**

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

**Show paths muestra:**

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

### Documentación de alertas

**Cada alerta incluye:**

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







































