# DOMINIO 3: CONFIGURAR Y USAR DEPENDABOT Y DEPENDENCY REVIEW (35%)

<a id="d3-1"></a>
## 3.1 Herramientas para gestionar vulnerabilidades en dependencias

### Dependency Graph

**Definición:**
Representación visual y analítica de todas las dependencias de un proyecto, incluyendo dependencias directas y transitivas (indirectas).

**¿Cómo se genera?**

```
1. GitHub detecta archivos de manifest:
   ├─ package.json, package-lock.json (npm/Node.js)
   ├─ Gemfile, Gemfile.lock (Ruby)
   ├─ requirements.txt, Pipfile.lock (Python)
   ├─ pom.xml, build.gradle (Java/Maven/Gradle)
   ├─ go.mod, go.sum (Go)
   ├─ Cargo.toml, Cargo.lock (Rust)
   ├─ composer.json, composer.lock (PHP)
   └─ *.csproj, packages.config (NuGet/.NET)

2. Parser extrae dependencias:
   ├─ Direct dependencies (en manifest)
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

**Visualización del Dependency Graph:**

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

**Ecosistemas soportados:**

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

Para ecosistemas NO soportados nativamente o builds custom:

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

**Definición:**
Lista completa y formal de todos los componentes de software, bibliotecas y dependencias que componen una aplicación.

**Formato SPDX (usado por GitHub):**

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

**Generar SBOM desde GitHub:**

```bash
# Via API
curl -H "Authorization: token $GITHUB_TOKEN" \
     -H "Accept: application/vnd.github+json" \
     https://api.github.com/repos/OWNER/REPO/dependency-graph/sbom \
     > sbom.json

# Via GitHub CLI
gh api /repos/OWNER/REPO/dependency-graph/sbom > sbom.json

# En GitHub Actions
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

**Casos de uso de SBOM:**

```yaml
1. Compliance:
   - NIST guidelines require SBOM
   - Executive Order 14028 (US)
   - EU Cyber Resilience Act

2. Supply chain security:
   - Identificar componentes con vulnerabilidades conocidas
   - Auditoría de licencias
   - Track de actualizaciones

3. Incident response:
   - "¿Estamos afectados por Log4Shell?"
   - Query SBOM por "log4j" → Sí/No inmediato
   - Version exacta → Patch priority

4. Procurement:
   - Vendors deben proporcionar SBOM
   - Verificar componentes antes de compra
   - Due diligence automatizado
```

### Vulnerabilidad de dependencia

**Definición:**
Defecto de seguridad conocido (CVE) en una dependencia que puede ser explotado para comprometer la aplicación.

**Anatomía de una vulnerabilidad:**

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

**Severidad (CVSS Score):**

| Score | Rating | Prioridad | SLA |
|-------|--------|-----------|-----|
| 9.0-10.0 | Critical | P0 | 24 horas |
| 7.0-8.9 | High | P1 | 7 días |
| 4.0-6.9 | Medium | P2 | 30 días |
| 0.1-3.9 | Low | P3 | 90 días |

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

Actualización: Múltiples veces al día
```

### Dependabot Alerts

**¿Qué son?**
Notificaciones automáticas cuando una dependencia tiene una vulnerabilidad conocida.

**Cómo funcionan:**

```
1. Dependency Graph identifica dependencias
     ↓
2. GitHub Advisory Database tiene nueva CVE
     ↓
3. Match: lodash@4.17.15 → CVE-2021-23337
     ↓
4. Calcular severidad: High (CVSS 7.2)
     ↓
5. Crear alert en Security tab
     ↓
6. Notify:
     ├─ Email a repository admins
     ├─ Web notification
     ├─ Webhook (si configurado)
     └─ Security Overview
```

**Formato de alerta:**

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

**¿Qué son?**
Pull requests automáticos que actualizan dependencias vulnerables a versiones parcheadas.

**Funcionamiento:**

```
1. Dependabot alert creada para lodash@4.17.15
     ↓
2. Dependabot verifica si hay versión parcheada
     ├─ lodash@4.17.21 existe
     └─ Es backward compatible (patch/minor)
     ↓
3. Crear branch: dependabot/npm_and_yarn/lodash-4.17.21
     ↓
4. Update package.json y lockfile
     ↓
5. Run tests (si hay CI configurado)
     ↓
6. Abrir PR con detalles:
     ├─ Changelog
     ├─ Commits
     ├─ Compatibility score
     └─ Release notes
     ↓
7. Developer review + merge
     ↓
8. Alert auto-closed
```

**Ejemplo de PR de Dependabot:**

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

**¿Qué es?**
Feature que analiza cambios de dependencias en PRs y bloquea merge si se introducen vulnerabilidades.

**Diferencia clave:**

```
Dependabot Alerts:
  - Escanea dependencias existentes
  - Reactivo (alerta después de merge)
  - Security tab

Dependency Review:
  - Escanea cambios en PR
  - Proactivo (bloquea antes de merge)
  - PR checks
```

**Funcionamiento:**

```
Developer crea PR:
  package.json: lodash@4.17.15 → lodash@4.17.10 (downgrade!)
     ↓
Dependency Review Action ejecuta:
     ↓
Compara:
  Base branch (main): lodash@4.17.15 (sin vulnerabilidades)
  PR branch: lodash@4.17.10 (CVE-2020-8203: HIGH)
     ↓
Resultado:
  ❌ Check failed: 1 high severity vulnerability introduced
     ↓
Bloquea merge:
  - PR status: ❌ Dependency review — Changes introduce known vulnerabilities
  - Requires: Fix before merge
```

**Configuración de Dependency Review Action:**

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
          # Fail on high/critical only
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

### Generación de alertas para dependencias vulnerables

**Pipeline completo:**

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

### Diferencia entre Dependabot y Dependency Review

**Tabla comparativa completa:**

| Aspecto | Dependabot Alerts | Dependabot Security Updates | Dependency Review |
|---------|------------------|----------------------------|-------------------|
| **Cuándo actúa** | Después de commit | Después de alert | Durante PR |
| **Objetivo** | Detectar vulnerabilidades existentes | Automatizar fixes | Prevenir nuevas vulnerabilidades |
| **Ubicación** | Security tab | Pull requests tab | PR checks |
| **Acción** | Crear alerta | Crear PR de fix | Bloquear/aprobar merge |
| **Reactivo/Proactivo** | Reactivo | Reactivo | Proactivo |
| **Requiere GHAS** | No (públicos), Sí (privados) | No (públicos), Sí (privados) | Sí |
| **Bloquea código** | No | No | Sí (configurable) |
| **Auto-remediation** | No | Sí (PR) | No |
| **Scope** | Todo el repo | Dependencias vulnerables | Cambios en PR |
| **Configuración** | Settings → Dependabot | Settings → Dependabot | GitHub Actions workflow |

**Flujo combinado ideal:**

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

**Cuándo usar cada herramienta:**

```yaml
Dependabot Alerts:
  Usa para:
    - ✅ Monitoring continuo de dependencias
    - ✅ Detectar vulnerabilidades en main branch
    - ✅ Compliance reporting
    - ✅ Security overview metrics

Dependabot Security Updates:
  Usa para:
    - ✅ Automatizar updates de seguridad
    - ✅ Reducir tiempo de remediación
    - ✅ Keep dependencies current
    - ✅ Batch updates (via grouping)

Dependency Review:
  Usa para:
    - ✅ Gate PRs con vulnerabilidades
    - ✅ Prevenir regresiones de seguridad
    - ✅ License compliance
    - ✅ Enforce security policies
    - ✅ Educate developers at PR time
```
**Enlaces:**
- https://docs.github.com/en/code-security/supply-chain-security/understanding-your-software-supply-chain/about-the-dependency-graph
- https://docs.github.com/en/code-security/dependabot
- https://docs.github.com/en/code-security/supply-chain-security/understanding-your-software-supply-chain/about-dependency-review

---
<a id="d3-2"></a>
## 3.2 Configuración predeterminada para alertas de Dependabot

### Repositorios Públicos

**Configuración automática:**

```yaml
Dependabot Alerts: ✅ HABILITADO por defecto
Dependency Graph: ✅ HABILITADO por defecto
Dependabot Security Updates: ✅ HABILITADO por defecto

Características incluidas:
  - Alertas automáticas de vulnerabilidades
  - PRs de seguridad automáticos
  - Notificaciones por email
  - Security tab visible
  - Dependency graph público
```

**No requiere:**
- ❌ Licencia GHAS
- ❌ Configuración manual
- ❌ GitHub Actions minutes (los PRs son gratuitos)

### Repositorios Privados

**Sin GHAS:**
```yaml
Dependabot Alerts: ✅ HABILITADO por defecto (desde 2022)
Dependency Graph: ✅ HABILITADO por defecto
Dependabot Security Updates: ❌ DESHABILITADO (requiere habilitar)
Dependency Review: ❌ NO DISPONIBLE (requiere GHAS)
```

**Con GHAS (GitHub Code Security):**
```yaml
Dependabot Alerts: ✅ HABILITADO
Dependency Graph: ✅ HABILITADO
Dependabot Security Updates: ✅ Puede habilitarse
Dependency Review: ✅ DISPONIBLE
Custom Auto-triage Rules: ✅ DISPONIBLE
```

### Tabla comparativa de configuración predeterminada

| Feature | Público | Privado sin GHAS | Privado con GHAS |
|---------|---------|------------------|------------------|
| **Dependency Graph** | ✅ Auto | ✅ Auto | ✅ Auto |
| **Dependabot Alerts** | ✅ Auto | ✅ Auto | ✅ Auto |
| **Dependabot Security Updates** | ✅ Auto | Opt-in | Opt-in |
| **Dependabot Version Updates** | Opt-in | Opt-in | Opt-in |
| **Dependency Review** | ❌ | ❌ | ✅ Requiere config |
| **Custom Auto-triage** | ❌ | ❌ | ✅ |
| **Security Overview** | ❌ | ❌ | ✅ |
| **Grouped Updates** | Opt-in | Opt-in | Opt-in |

### Verificar configuración actual

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

**Enlaces:**
- https://docs.github.com/en/code-security/dependabot/dependabot-alerts/about-dependabot-alerts
- https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/enabling-features-for-your-repository/managing-security-and-analysis-settings-for-your-repository
- https://docs.github.com/en/code-security/getting-started/securing-your-repository

---
<a id="d3-3"></a>
## 3.3 Permisos y roles para Dependabot

### Permisos para HABILITAR alertas de Dependabot

**A nivel de repositorio:**

| Rol | Habilitar Dependabot Alerts | Habilitar Security Updates | Configurar dependabot.yml |
|-----|----------------------------|---------------------------|---------------------------|
| **Read** | ❌ | ❌ | ❌ |
| **Triage** | ❌ | ❌ | ❌ |
| **Write** | ❌ | ❌ | ✅ (via PR) |
| **Maintain** | ❌ | ❌ | ✅ |
| **Admin** | ✅ | ✅ | ✅ |

**A nivel de organización:**

| Rol | Habilitar para org | Policies | Bulk enable |
|-----|-------------------|----------|-------------|
| **Member** | ❌ | ❌ | ❌ |
| **Owner** | ✅ | ✅ | ✅ |
| **Security Manager** | ✅ | ✅ | ✅ |

### Permisos para VER alertas de Dependabot

**Importante:** Las alertas de Dependabot tienen visibilidad diferente que otras alertas de seguridad.

| Rol | Ver Dependabot Alerts | Ver detalles | Dismiss alerts | Ver PRs |
|-----|----------------------|--------------|----------------|---------|
| **Read** | ✅ | ✅ | ❌ | ✅ |
| **Triage** | ✅ | ✅ | ❌ | ✅ |
| **Write** | ✅ | ✅ | ✅ | ✅ |
| **Maintain** | ✅ | ✅ | ✅ | ✅ |
| **Admin** | ✅ | ✅ | ✅ | ✅ |
| **Security Manager** | ✅ | ✅ | ✅ | ✅ |

**Diferencia con Code Scanning:**
```yaml
Code Scanning:
  - Solo Admin y Security Manager ven alertas
  
Dependabot:
  - Todos los colaboradores con Read+ ven alertas
  - Razón: Developers necesitan ver dependencias para su trabajo
```

### Configuración de acceso granular

**Otorgar acceso a team:**

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
# Agregar security manager a org
gh api \
  --method PUT \
  -H "Accept: application/vnd.github+json" \
  /orgs/ORG/security-managers/teams/TEAM_SLUG

# Listar security managers
gh api /orgs/ORG/security-managers/teams
```

**Custom notification groups:**

```yaml
# No hay configuración nativa para custom groups
# Solución: Usar webhooks + automation

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

**Enlaces:**
- https://docs.github.com/en/organizations/managing-user-access-to-your-organizations-repositories/repository-roles-for-an-organization
- https://docs.github.com/en/code-security/dependabot/dependabot-alerts/configuring-dependabot-alerts
- https://docs.github.com/en/organizations/managing-peoples-access-to-your-organization-with-roles/managing-security-managers-in-your-organization
- https://docs.github.com/en/code-security/dependabot/working-with-dependabot/configuring-access-to-private-registries-for-dependabot

---
<a id="d3-4"></a>
## 3.4 Habilitar Dependabot para repositorios privados

### Método 1: Via Web UI (Individual)

**Paso a paso:**

```
1. Ir al repositorio
   └─ Settings tab

2. Navegar a Code security and analysis
   └─ Sidebar izquierdo

3. Habilitar Dependency graph (si no está habilitado)
   ├─ Click [Enable]
   └─ Esperar 1-2 minutos para el análisis inicial

4. Habilitar Dependabot alerts
   ├─ Click [Enable]
   └─ Confirmar

5. [Opcional] Habilitar Dependabot security updates
   ├─ Click [Enable]
   └─ Esto permite PRs automáticos
```

### Método 2: Via GitHub CLI

```bash
# Habilitar todo de una vez
gh api \
  --method PATCH \
  -H "Accept: application/vnd.github+json" \
  /repos/OWNER/REPO \
  -f has_dependency_graph=true

# Habilitar vulnerability alerts
gh api \
  --method PUT \
  -H "Accept: application/vnd.github+json" \
  /repos/OWNER/REPO/vulnerability-alerts

# Habilitar security updates
gh api \
  --method PUT \
  -H "Accept: application/vnd.github+json" \
  /repos/OWNER/REPO/automated-security-fixes

# Verificar
gh api repos/OWNER/REPO | jq '{
  dependency_graph: .has_dependency_graph,
  alerts: .vulnerability_alerts_enabled,
  security_updates: .automated_security_fixes_enabled
}'
```

### Método 3: Via API (Programático)

```python
import requests

GITHUB_TOKEN = "ghp_..."
ORG = "my-org"

headers = {
    "Authorization": f"token {GITHUB_TOKEN}",
    "Accept": "application/vnd.github+json"
}

# Obtener todos los repos privados
repos_response = requests.get(
    f"https://api.github.com/orgs/{ORG}/repos",
    headers=headers,
    params={"type": "private", "per_page": 100}
)

for repo in repos_response.json():
    repo_name = repo["full_name"]
    
    print(f"Enabling Dependabot for {repo_name}...")
    
    # Habilitar vulnerability alerts
    alerts_response = requests.put(
        f"https://api.github.com/repos/{repo_name}/vulnerability-alerts",
        headers=headers
    )
    
    # Habilitar automated security fixes
    fixes_response = requests.put(
        f"https://api.github.com/repos/{repo_name}/automated-security-fixes",
        headers=headers
    )
    
    if alerts_response.status_code == 204 and fixes_response.status_code == 204:
        print(f"  ✅ {repo_name}: Dependabot enabled")
    else:
        print(f"  ❌ {repo_name}: Error - {alerts_response.status_code}")
```

### Método 4: Bulk habilitación (Organization level)

**Via UI:**

```
Organization Settings
  → Code security and analysis
  → Dependabot
      ├─ [Enable for all repositories]
      │   └─ Seleccionar:
      │       ├─ All repositories
      │       ├─ All private repositories
      │       └─ Selected repositories
      │
      └─ [✓] Automatically enable for new repositories
          ├─ New public repositories
          └─ New private repositories
```

**Script de habilitación masiva:**

```bash
#!/bin/bash
# enable-dependabot-all-repos.sh

ORG="my-org"
TOKEN="$GITHUB_TOKEN"

# Obtener todos los repos
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

**Enlaces:**

- https://docs.github.com/en/code-security/dependabot/dependabot-alerts/configuring-dependabot-alerts
- https://docs.github.com/en/rest/repos/repos#enable-vulnerability-alerts
- https://docs.github.com/en/rest/repos/repos#enable-automated-security-fixes
- https://docs.github.com/en/enterprise-cloud@latest/code-security/dependabot/dependabot-alerts/configuring-dependabot-alerts#managing-dependabot-alerts-for-your-organization

---
<a id="d3-5"></a>
## 3.5 Habilitar Dependabot para organizaciones

### Políticas a nivel de organización

**Opción 1: Habilitar para todos**

```
Organization Settings
  → Code security and analysis
  → Dependabot alerts
      └─ [Enable for all repositories]
          ├─ Apply to: All repositories
          └─ [Confirm]
```

**Opción 2: Política por defecto para nuevos repos**

```
Organization Settings
  → Code security and analysis
  → Dependabot alerts
      └─ [✓] Automatically enable for new repositories
          ├─ Public repositories: [✓] Enabled
          └─ Private repositories: [✓] Enabled
```

**Opción 3: Políticas condicionales (GitHub Enterprise)**

```yaml
# Solo disponible en GitHub Enterprise Cloud
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

### Configuración de notificaciones a nivel org

```yaml
# Configurar destinatarios por defecto

Organization Settings
  → Code security
  → Dependabot alerts
      └─ Default notification settings
          ├─ Notify: security@company.com
          ├─ Slack: #security-alerts
          └─ Webhook: https://api.company.com/webhooks/dependabot
```

### Reporte y compliance

```bash
# Script para generar reporte de coverage

#!/bin/bash
ORG="my-org"

echo "Dependabot Coverage Report for $ORG"
echo "===================================="
echo ""

total_repos=0
enabled_repos=0
disabled_repos=0

# Obtener todos los repos
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

**Enlaces:**

- https://docs.github.com/en/organizations/keeping-your-organization-secure/managing-security-settings-for-your-organization/managing-security-and-analysis-settings-for-your-organization
- https://docs.github.com/en/code-security/dependabot/dependabot-alerts/configuring-dependabot-alerts#managing-dependabot-alerts-for-your-organization
- https://docs.github.com/en/enterprise-cloud@latest/admin/policies/enforcing-policies-for-your-enterprise/enforcing-policies-for-code-security-and-analysis-for-your-enterprise

---
<a id="d3-6"></a>
## 3.6 Crear archivo de configuración de Dependabot (dependabot.yml)

### Estructura básica

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

### Ecosistemas soportados

```yaml
# Todos los ecosistemas disponibles:

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

### Configuración de grupos (Grouping updates)

**Agrupar updates para reducir PRs:**

```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
    
    # Agrupar por tipo de dependencia
    groups:
      # Grupo para todas las dev dependencies
      development-dependencies:
        dependency-type: "development"
        update-types:
          - "minor"
          - "patch"
      
      # Grupo para production dependencies (solo patches)
      production-dependencies:
        dependency-type: "production"
        update-types:
          - "patch"
      
      # Grupo por patrón de nombre
      react-related:
        patterns:
          - "react*"
          - "@types/react*"
      
      # Grupo para testing frameworks
      testing-frameworks:
        patterns:
          - "jest"
          - "@testing-library/*"
          - "vitest"
```

**Ejemplo de agrupación avanzada:**

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
        # No agrupar - un PR por major update
      
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

**Resultado de grouping:**

```
Sin grouping:
  ├─ PR #1: Bump eslint from 8.0.0 to 8.0.1
  ├─ PR #2: Bump eslint-config-airbnb from 19.0.0 to 19.0.1
  ├─ PR #3: Bump eslint-plugin-import from 2.25.0 to 2.25.1
  └─ ... (10 PRs más para tooling)

Con grouping:
  └─ PR #1: Bump build-tools group (13 dependencies)
      ├─ eslint: 8.0.0 → 8.0.1
      ├─ eslint-config-airbnb: 19.0.0 → 19.0.1
      └─ ... (11 más)
```

### Configuración de automerge

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
    
    # Auto-merge configuration (requires GitHub CLI or Actions)
    # Note: Dependabot no tiene auto-merge nativo
    # Se configura via branch protection rules o Actions
```

**GitHub Actions workflow para auto-merge:**

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

### Configuración avanzada de scheduling

```yaml
version: 2
updates:
  # Diferentes schedules para diferentes ecosistemas
  
  # npm: Diario (alta frecuencia de updates)
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "daily"
      time: "03:00"
      timezone: "UTC"
  
  # Docker: Semanal (menos frecuente)
  - package-ecosystem: "docker"
    directory: "/"
    schedule:
      interval: "weekly"
      day: "sunday"
      time: "04:00"
  
  # GitHub Actions: Mensual (muy estables)
  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "monthly"
    
  # Terraform: Solo security updates
  - package-ecosystem: "terraform"
    directory: "/infrastructure"
    schedule:
      interval: "weekly"
    open-pull-requests-limit: 0  # No version updates
    # Solo security updates a través de Dependabot alerts
```

### Ignore configurations

```yaml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
    
    # Ignorar dependencias específicas
    ignore:
      # Nunca actualizar React (pinned version)
      - dependency-name: "react"
        update-types: ["version-update:semver-major"]
      
      # Ignorar patches de lodash (demasiados updates)
      - dependency-name: "lodash"
        update-types: ["version-update:semver-patch"]
      
      # Ignorar todas las major versions de Angular
      - dependency-name: "@angular/*"
        update-types: ["version-update:semver-major"]
      
      # Ignorar versiones específicas que rompen
      - dependency-name: "webpack"
        versions: ["5.x"]
```

### Configuración para monorepos

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

### Validación del archivo

```bash
# GitHub no tiene validador oficial, pero puedes usar:

# 1. Linter YAML básico
yamllint .github/dependabot.yml

# 2. Verificar sintaxis con Python
python -c "import yaml; yaml.safe_load(open('.github/dependabot.yml'))"

# 3. Verificar después de commit
# GitHub validará el archivo automáticamente
# Errores aparecerán en:
# Repository → Insights → Dependency graph → Dependabot

# 4. Test in workflow
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

**Enlaces:**
- https://docs.github.com/en/code-security/dependabot/dependabot-version-updates/configuration-options-for-the-dependabot.yml-file
- https://docs.github.com/en/code-security/dependabot/dependabot-version-updates/configuring-dependabot-version-updates
- https://docs.github.com/en/code-security/dependabot/dependabot-version-updates/about-dependabot-version-updates
- https://docs.github.com/en/code-security/dependabot/working-with-dependabot/keeping-your-actions-up-to-date-with-dependabot
- https://docs.github.com/en/code-security/dependabot/dependabot-version-updates/grouping-dependabot-updates

---
<a id="d3-7"></a>
# 3.7 Custom Auto-triage Rules para Dependabot

**Disponible con GitHub Code Security**

### ¿Qué son las auto-triage rules?

Reglas automáticas para gestionar alertas de Dependabot a escala:
- Auto-dismiss alertas de baja prioridad
- Auto-snooze hasta que haya patch disponible
- Auto-trigger security updates para criterios específicos

### Crear reglas de auto-triage

**Via UI:**

```
Repository/Organization Settings
  → Code security
  → Dependabot
  → Auto-triage rules
      → [New rule]
```

**Ejemplo 1: Descartar low severity hasta patch**

```yaml
Rule name: Dismiss low severity without patch
Conditions:
  - Severity: Low
  - State: Open
  - Has patch: No

Action: Dismiss
Reason: "Low severity, will address when patch available"
```

**Ejemplo 2: Auto-approve patches de desarrollo**

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
# Crear auto-triage rule
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

### Casos de uso comunes

**1. Gestión de deuda técnica:**

```yaml
Rule: Snooze medium severity for legacy code
Conditions:
  - Severity: Medium
  - File path: /legacy/**
Action: Snooze for 90 days
Reason: "Legacy code scheduled for deprecation"
```

**2. Priorización por ecosistema:**

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

**3. Desarrollo vs Producción:**

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

**Enlaces:**

- https://docs.github.com/en/code-security/dependabot/dependabot-auto-triage-rules/about-dependabot-auto-triage-rules
- https://docs.github.com/en/code-security/dependabot/dependabot-auto-triage-rules/customizing-auto-triage-rules-to-prioritize-dependabot-alerts
- https://github.blog/changelog/2024-01-24-dependabot-auto-triage-rules-and-custom-rules-public-beta/

---
<a id="d3-8"></a>
## 3.8 Dependency Review Workflow

### Configurar Dependency Review Action

**Configuración básica:**

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

### Configuración avanzada con umbrales

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
          # Fallar solo en severidad moderate o superior
          fail-on-severity: moderate
          
          # Fallar en scopes específicos
          fail-on-scopes: runtime, production
          
          # Licencias permitidas
          allow-licenses: |
            MIT
            Apache-2.0
            BSD-2-Clause
            BSD-3-Clause
            ISC
            0BSD
          
          # Licencias denegadas
          deny-licenses: |
            GPL-3.0
            AGPL-3.0
            LGPL-3.0
          
          # Comentar resumen en PR
          comment-summary-in-pr: always
          
          # Fallar en dependencias con OpenSSF Scorecard bajo
          warn-on-openssf-scorecard-level: 3
          
          # Configurar GitHub token
          github-token: ${{ secrets.GITHUB_TOKEN }}
          
          # Base y head para comparación
          base-ref: ${{ github.event.pull_request.base.sha }}
          head-ref: ${{ github.event.pull_request.head.sha }}
```

### Configuración de licencias

**Definir políticas de licencia:**

```yaml
# .github/workflows/dependency-review.yml
- name: 'Dependency Review with License Check'
  uses: actions/dependency-review-action@v4
  with:
    # Permitir solo licencias open source permisivas
    allow-licenses: |
      MIT
      Apache-2.0
      BSD-2-Clause
      BSD-3-Clause
      ISC
      Unlicense
    
    # Denegar copyleft estricto
    deny-licenses: |
      GPL-2.0
      GPL-3.0
      AGPL-3.0
      LGPL-3.0
    
    # Permitir excepciones para dependencias específicas
    allow-dependencies-licenses: |
      pkg:npm/some-gpl-package@1.0.0, GPL-3.0
    
    # Acción en violación de licencia
    license-check: true
    fail-on-severity: high
```

**Matriz de licencias comunes:**

| Licencia | Tipo | Permitida corporativamente | Requiere atribución |
|----------|------|---------------------------|---------------------|
| MIT | Permisiva | ✅ | ✅ |
| Apache-2.0 | Permisiva | ✅ | ✅ |
| BSD-3-Clause | Permisiva | ✅ | ✅ |
| ISC | Permisiva | ✅ | ✅ |
| GPL-3.0 | Copyleft | ⚠️ | ✅ |
| AGPL-3.0 | Copyleft fuerte | ❌ | ✅ |
| Unlicense | Public Domain | ✅ | ❌ |
| Proprietary | Propietaria | ⚠️ | Varía |

### Configurar umbrales de severidad personalizados

```yaml
# Diferentes políticas por tipo de proyecto

# Aplicación crítica de producción
- name: 'Dependency Review - Production'
  uses: actions/dependency-review-action@v4
  with:
    fail-on-severity: low  # Muy estricto
    fail-on-scopes: runtime
    
# Proyecto interno de desarrollo
- name: 'Dependency Review - Dev Tool'
  uses: actions/dependency-review-action@v4
  with:
    fail-on-severity: high  # Más relajado
    fail-on-scopes: runtime, production
```

### Integración con branch protection

```yaml
# Configurar en GitHub UI
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

### Casos de uso avanzados

**1. Diferentes políticas por directorio:**

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

**2. Monitoreo sin bloqueo (warning mode):**

```yaml
- name: 'Dependency Review - Warn Only'
  uses: actions/dependency-review-action@v4
  with:
    fail-on-severity: critical  # Solo critical bloquea
    warn-only: true  # Comentar pero no fallar
    comment-summary-in-pr: always
```

**3. Custom output y reporting:**

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

**Enlaces:**
- https://docs.github.com/en/code-security/supply-chain-security/understanding-your-software-supply-chain/about-dependency-review
- https://docs.github.com/en/code-security/supply-chain-security/understanding-your-software-supply-chain/configuring-dependency-review
- https://github.com/actions/dependency-review-action
- https://docs.github.com/en/code-security/supply-chain-security/understanding-your-software-supply-chain/dependency-review-enforcement
- https://github.com/actions/dependency-review-action#configuration-options

---
<a id="d3-9"></a>
## 3.9 Identificar y corregir dependencias vulnerables

### Identificar vulnerabilidades

**Desde Dependabot Alert:**

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

*Desde Pull Request:**

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

### Solucionar desde Security tab

**Flujo de remediación:**

```
1. Ir a Security → Dependabot alerts
   
2. Click en alert específica
   
3. Revisar detalles:
   ├─ Leer descripción del CVE
   ├─ Entender el impacto
   └─ Verificar si el código afectado se usa
   
4. Opciones de remediación:
   
   Opción A: Aceptar PR de Dependabot
     ├─ Click [Review security update]
     ├─ Ver PR con cambios
     ├─ Verificar tests pasan
     ├─ [Merge pull request]
     └─ Alert se cierra automáticamente
   
   Opción B: Actualizar manualmente
     ├─ Click [Dismiss alert]
     ├─ Reason: "Fix is being prepared"
     ├─ Actualizar package.json:
     │   "lodash": "4.17.15"  →  "lodash": "^4.17.21"
     ├─ npm update lodash
     ├─ git commit & push
     └─ Alert se cierra automáticamente
   
   Opción C: Eliminar dependencia
     ├─ Si no es necesaria
     ├─ npm uninstall lodash
     ├─ Refactor código para no usar lodash
     ├─ git commit & push
     └─ Alert se cierra automáticamente
```

### Solucionar desde Pull Request

**Cuando Dependency Review bloquea el PR:**

```
Scenario: PR introduce axios@0.21.0 (vulnerable)

1. Ver el error en PR checks:
   ❌ Dependency Review — High severity vulnerability

2. Click en "Details" del check

3. Ver reporte:
   Package: axios@0.21.0
   Vulnerability: CVE-2021-3749
   Fix: axios@0.27.0+

4. Actualizar en el PR:
   # En tu branch local
   npm update axios@^0.27.0
   git add package*.json
   git commit -m "fix: Update axios to resolve CVE-2021-3749"
   git push

5. Dependency Review re-ejecuta:
   ✅ Dependency Review — No vulnerabilities detected

6. PR ahora puede mergearse
```

### Actualizar dependencias transitivas

**Problema:** La vulnerabilidad está en una dependencia indirecta

```
Dependency tree:
  my-app
    └─ express@4.16.0
        └─ lodash@4.17.15  ← Vulnerable (transitiva)
```

**Solución 1: Actualizar la dependencia padre**

```bash
# Actualizar express a versión que use lodash seguro
npm update express
# o
npm install express@latest

# Verificar árbol de dependencias
npm list lodash
# my-app@1.0.0
# └─ express@4.18.2
#     └─ lodash@4.17.21  ✅ Fixed
```

**Solución 2: Resolution/override (npm 8.3+)**

```json
// package.json
{
  "overrides": {
    "lodash": "^4.17.21"
  }
}

// Para yarn
{
  "resolutions": {
    "lodash": "^4.17.21"
  }
}

// Para pnpm
{
  "pnpm": {
    "overrides": {
      "lodash": "^4.17.21"
    }
  }
}
```

**Solución 3: npm audit fix**

```bash
# Auto-fix vulnerabilities
npm audit fix

# Fix aggressive (puede romper compatibilidad)
npm audit fix --force

# Ver qué se va a arreglar sin aplicar
npm audit fix --dry-run

# Generar reporte
npm audit --json > audit-report.json
```

### Testing después de actualización

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

### Merge de Dependabot PRs

**Best practices:**

```yaml
✅ DO:
  - Review changelog de la actualización
  - Verificar breaking changes
  - Run tests completos
  - Check bundle size impact
  - Verificar deprecation warnings
  - Test en staging antes de prod

❌ DON'T:
  - Auto-merge major versions sin review
  - Ignorar test failures
  - Skip security patches
  - Merge sin verificar compatibility
```

**Workflow de merge:**

```bash
# 1. Checkout PR de Dependabot
gh pr checkout 123

# 2. Verificar cambios
git diff main HEAD

# 3. Run tests localmente
npm test

# 4. Build y verificar
npm run build

# 5. Si todo OK, merge
gh pr merge 123 --squash --delete-branch

# 6. Verificar que alert se cierra
gh api repos/:owner/:repo/dependabot/alerts | \
  jq '.[] | select(.state == "fixed")'
```

**Enlaces:**
- https://docs.github.com/en/code-security/dependabot/working-with-dependabot/managing-pull-requests-for-dependency-updates
- https://docs.github.com/en/code-security/dependabot/dependabot-security-updates/about-dependabot-security-updates
- https://docs.github.com/en/code-security/dependabot/dependabot-security-updates/configuring-dependabot-security-updates
- https://docs.github.com/en/code-security/dependabot/working-with-dependabot/automating-dependabot-with-github-actions
- https://docs.npmjs.com/cli/v8/commands/npm-audit
- https://docs.github.com/en/code-security/supply-chain-security/understanding-your-software-supply-chain/troubleshooting-the-dependency-graph

---
