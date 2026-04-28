# DOMINIO 3: CONFIGURAR Y USAR DEPENDABOT Y DEPENDENCY REVIEW (35%)

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



















































































