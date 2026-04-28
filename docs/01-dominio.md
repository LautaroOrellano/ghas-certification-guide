# DOMINIO 1: CARACTERÍSTICAS Y FUNCIONALIDADES DE SEGURIDAD DE GHAS (15%)
## 1.1 Comparar las caraterísticas de GHAS y su papel en el ecosistema de seguridad

### ¿Qué es GitHub Advanced Security (GHAS)?

**Desde abril 2025**, GHAS se divide en dos productos independientes:

1. **GitHub Secret Protection** ($19/mes por committer activo)
    - Secret scanning (escaneo de repositorio)
    - Push protection (protección en push)
    - Copilot secret scanning (detección AI)
    - Custom patterns (patrones personalizados)
    - Delegated bypass/dismissal
    - Security campaings
    - Security overview

2. **GitHub Code Security** (30$/mes por committer activo)
    - Code scanning con CodeQL
    - Copilot Autofix
    - Security campaigns
    - Custom auto-triage rules para Dependabot
    - Dependency review
    - Security overview

### Componentes principales de GHAS:

#### a) **Code Scanning (Escaneo de Código)**
- Motor de análisis estático que identifica vulnerabilidades de seguridad
- Utiliza CodeQL (lenguaje de consulta GitHub)
- Detecta: SQL injection, XSS, authentication bypass, etc.
- Se ejecuta en GitHub Actions o CI externo

#### b) **Secret Scanning (Escaneo de Secretos)**
- Detecta credenciales, tokens, claves API en el código
- Notifica a proveedores de servicios para revocación
- Incluye push protection para bloquear commits con secretos
- Soporta patrones personalizados

#### c) **Dependabot**
- Escaneo de dependencias vulnerables
- Alertas automáticas de vulnerabilidades
- Pull requests automáticos de actualización
- Actualizaciones de seguridad y versión

#### d) **Dependency Review**
- Revisión de dependencias vulnerables
- Bloequea merges que añaden vulnerabilidades
- Verifica licencias
- Analiza el dependency graph

#### e) **Security Overview**
- Dashboard centralizado de seguridad
- Métricas y tendencias de vulnerabilidades
- Vista a nivel organización/empresa
- Identificación de repositorios de alto riesgo

### Rol en el ecosistema de seguridad:

GHAS se integra en el **Shift Left Security** - moviendo la seguridad al inicio del SDLC:

```
Desarrollo → Commit → PR → Merge → Deploy → Producción
    ↓         ↓      ↓      ↓        ↓         ↓
  CodeQL   Secret  Dep   Code    Runtime  Monitoring
           Scan   Review Scan    Security
```

**Enlaces:**
- https://docs.github.com/en/get-started/learning-about-github/about-github-advanced-security
- https://github.com/security/advanced-security
- https://resources.github.com/evolving-github-advanced-security/

---

## 1.2 Diferenciar características para proyectos Open Source vs GHEC/GHES

### Repositorios Públicos (Open Source) - GRATUITO

**Características automáticas:**
- Secret scanning (escaneo de secretos)
- Push protection (protección en push)
- Dependency graph (gráfico de dependencias)
- Dependency alerts (alertas de dependencias vulnerables)
- Dependabot security updates (actualizaciones automáticas)

**NO incluidas:**
- Code scanning con CodeQL (requiere configuración manual pero es gratuito)
- Security overview
- Custom patterns para secret scanning
- Advanced security overview features

### Repositorios Privados con GHAS (GHEC/GHES)

**Requiere licencia de:**
- GitHub Secret Protection, o
- GitHub Code Security, o
- GitHub Enterprise (que incluía GHAS hasta abril 2025)

**Características adicionales:**

| Característica | Público | Privado sin GHAS | Privado con GHAS |
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

### GHES (GitHub Enterprise Server) - Consideraciones

**Requisitos adicionales:**
- Runners con Docker para Dependabot
- Conectividad a internet (o registry privado)
- CodeQL CLI instalado localmente
- Versión mínima: GHES 3.8+

**Características según versión:**
- GHES 3.8: Dependabot sin internet con registry privado
- GHES 3.9+: Security configurations
- GHES 3.10+: Delegated bypass

**Enlaces:**
- https://docs.github.com/en/enterprise-server@latest/admin/code-security
- https://github.com/advanced-security/advanced-security-material

---

## 1.3 Características y beneficios del Security Overview

### ¿Qué es Security Overview?

Dashboard centralizado que proporciona visibilidad del estado de seguridad a nivel:
- **Organización**: Todos los repositorios
- **Empresa**: Todas las organizaciones
- **Equipo**: Repositorios del equipo

### Características principales:

#### a) **Vistas y Métricas**

**1. Security Risk View**
- Repositorios ordenados por nivel de riesgo
- Métricas: alertas abiertas, severidad, antigüedad
- Filtros: lenguaje, equipo, arquetipo

**2. Security Coverage View**
- % de repositorios con GHAS habilitado
- Identificación de gaps de seguridad
- Estado de features por repo

**3. Alert Trends**
- Gráficos temporales de alertas
- Tendencias de remediación
- Comparativas por tipo

#### b) **Filtros Avanzados**

```yaml
Filtros disponibles:
  - Tipo de alerta: code-scanning, secret-scanning, dependabot
  - Severidad: critical, high, medium, low
  - Estado: open, closed, dismissed, fixed
  - Repositorio: nombre, arquetipo, equipo
  - Lenguaje: java, javascript, python, etc.
  - Edad de alerta: días desde creación
```

#### c) ** Capacidades de Gestión**

- **Bulk actions**: cerrar/dismissar múltiples alertas
- **Security campaigns**: coordinar remediación
- **CSV exports**: reportes para stakeholders
- **API access**: integración con SIEM/dashboards

### Beneficios:

1. **Visibilidad centralizada**: Una vista de toda la postura de seguridad
2. **Priorización**: Identificar repositorios críticos
3. **Compliance**: Demostrar cumplimiento de políticas
4. **Métricas**: KPIs de seguridad (MTTR, coverage, trends)
5. **Accountability**: Asignar ownership de remediación

### Casos de uso:

**Para Security Teams:**
- Auditoría de cobertura GHAS
- Identificación de hotspots de vulnerabilidades
- Tracking de SLAs de remediación

**Para Engineering Managers:**
- Comparar madurez de seguridad entre equipos
- Planificación de tech debt de seguridad
- Justificación de inversión en seguridad

**Para Compliance:**
- Reportes de estado para auditorías
- Evidencia de controles de seguridad
- Tracking de exceptions/waivers

**Enlaces:**
- https://docs.github.com/en/code-security/security-overview/about-security-overview
- https://docs.github.com/en/code-security/security-overview/assessing-adoption-code-security

---

## 1.4 Diferencias entre Secret Scanning y Code Scanning

### Secret Scanning

**Propósito**: Detectar credenciales y secretos expuestos

**¿Qué detecta?**
- API keys, tokens, certificados
- Contraseñas, connection strings
- Claves privadas SSH/PGP
- Credenciales cloud (AWS, Azure, GCP)

**Cómo funciona:**
1. Escanea todo el historial del repositorio
2. Usa regex patterns para match
3. Valida con providers (validity check)
4. Notifica al proveedor para revocación
5. Crea alerta en Security tab

**Tipos de patterns:**
- **Predeterminados**: 200+ patterns de GitHub
- **Partner patterns**: Validados por proveedores
- **Custom patterns**: Definidos por organización

**Ejemplo de detección:**
```javascript
// Esto generaría alerta:
const apiKey = "ghp_1234567890abcdefghijklmnopqrstuv";

// Pattern detectado: GitHub Personal Access Token
// Validez: Active (se verificó con GitHub API)
// Acción: Notificar al usuario, revocar token
```

**Push Protection:**
- Bloquea commits con secretos en tiempo real
- Permite bypass con justificación
- Configurable: block o warn

### Code Scanning

**Propósito**: Detectar vulnerabilidades y errores de código

**¿Qué detecta?**
- SQL injection, XSS, CSRF
- Path traversal, command injection
- Authentication/authorization flaws
- Resource leaks, race conditions
- CWE (Common Weakness Enumeration)

**Cómo funciona:**
1. CodeQL construye una base de datos del código
2. Ejecuta queries para encontrar patterns
3. Analiza data flow y control flow
4. Genera SARIF results
5. Crea alertas con ubicación y explicación

**Ejemplo de detección:**
```java
// Esto generaría alerta:
String query = "SELECT * FROM users WHERE id = " + userId;
// ↑ CWE-89: SQL Injection

// Query CodeQL detecta:
// - userId viene de input no sanitizado
// - Se concatena directamente en SQL
// - No hay prepared statement
// Severidad: High
// Recomendación: Usar PreparedStatement
```
### Comparación Directa

| Aspecto | Secret Scanning | Code Scanning |
|---------|----------------|---------------|
| **Objetivo** | Credenciales expuestas | Vulnerabilidades de código |
| **Tecnología** | Pattern matching (regex) | Static analysis (CodeQL) |
| **Alcance** | Todo el historial | Código en HEAD branch |
| **Velocidad** | Rápido (minutos) | Lento (minutos a horas) |
| **False positives** | Bajo (con validación) | Medio (depende de queries) |
| **Remediación** | Revocar + rotar secreto | Refactorizar código |
| **Integración** | Automática en push | GitHub Actions workflow |
| **CPU/memoria** | Bajo | Alto (compilación + análisis) |

### Cuándo usar cada uno:

**Secret Scanning:**
- ✅ Detectar exposición de credenciales
- ✅ Compliance de secretos
- ✅ Prevención de leaks en CI/CD
- ✅ Auditoría de secretos históricos

**Code Scanning:**
- ✅ Detectar bugs de seguridad
- ✅ Code review automatizado
- ✅ SAST (Static Application Security Testing)
- ✅ Mantener calidad de código

**Enlaces:**
- https://docs.github.com/en/code-security/secret-scanning
- https://docs.github.com/en/code-security/code-scanning

---

## 1.5 Ciclo de vida de desarrollo seguro con GHAS

### Integración en el SDLC

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
│  DEPLOY     │ → Security campaigns
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
### Escenario A: Seguridad Aislada (Tradicional)

**Proceso:**
1. Desarrollo completo sin checks
2. Code review manual
3. Merge a main sin validación
4. Deploy a staging
5. **Pentest/security review** → Vulnerabilidades descubiertas
6. Rollback o hotfix urgente
7. Ciclo se repite

**Problemas:**
- ❌ Vulnerabilidades descubiertas tarde
- ❌ Costo alto de remediación
- ❌ Delays en releases
- ❌ Security como bottleneck
- ❌ Cultura de "security vs velocity"

**Métricas típicas:**
- Time to fix: 30-90 días
- Cost per vulnerability: $500-$5,000
- Security debt acumulado

### Escenario B: Seguridad Integrada (GHAS)

**Proceso:**
1. Desarrollo con CodeQL en IDE
2. **Commit bloqueado** si hay secretos
3. **PR checks** bloquean merge si:
   - Code scanning encuentra critical/high
   - Dependency review detecta vulnerabilidades
4. Merge solo si pasa todos los checks
5. Dependabot crea PRs automáticos
6. Security overview monitorea todo

**Beneficios:**
- ✅ Shift left: bugs encontrados temprano
- ✅ Automatic remediation: Dependabot PRs
- ✅ No delays: checks en paralelo
- ✅ Developer ownership: contexto inmediato
- ✅ Cultura de "security enables velocity"

**Métricas típicas:**
- Time to fix: 1-7 días
- Cost per vulnerability: $50-$500
- Security debt reducido 70%

### Comparación de Impacto

| Métrica | Sin GHAS | Con GHAS | Mejora |
|---------|----------|----------|--------|
| Vulnerabilidades en producción | 50/año | 10/año | -80% |
| Tiempo de remediación | 45 días | 5 días | -89% |
| Costo por vulnerabilidad | $2,000 | $200 | -90% |
| Cobertura de tests | Manual | Automático | +100% |
| Developer satisfaction | Bajo | Alto | Mejor experiencia |

### Best Practices para Integración

**1. Fase de Plan:**
- Definir security requirements
- Threat modeling
- Alinear con compliance

**2. Fase de Code:**
- CodeQL CLI local
- IDE extensions (VS Code, IntelliJ)
- Pre-commit hooks

**3. Fase de Commit:**
- Habilitar push protection
- Educar en manejo de secretos
- Usar secret managers (Vault, etc.)

**4. Fase de PR:**
- Required status checks
- Configurar severity thresholds
- Code scanning + dependency review

**5. Fase de Merge:**
- Proteger main branch
- Require passing checks
- CODEOWNERS review

**6. Fase de Deploy:**
- Generate SBOM
- Security campaigns para deuda técnica
- Automated rollback si alertas críticas

**7. Fase de Monitor:**
- Security overview dashboard
- SLAs de remediación
- Metrics para continuous improvement

**Enlaces:**
- https://docs.github.com/en/code-security/getting-started/securing-your-repository
- https://docs.github.com/en/enterprise-cloud@latest/code-security/tutorials/adopting-github-advanced-security-at-scale

---

## 1.6 Explicar y utilizar características específicas de GHAS

### a) Identificación de dependencias vulnerables

1. **Análisis de manifiestos**
   - GitHub detecta archivos de dependencias:
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

2. **Construcción del Dependency Graph**
   - Parsea cada manifiesto
   - Identifica dependencias directas e indirectas (transitive)
   - Construye árbol de dependencias
   - Actualiza en cada commit

3. **Comparación con bases de datos**
   - **GitHub Advisory Database**: CVEs + submissions de community
   - **NVD (National Vulnerability Database)**
   - **WhiteSource/Snyk advisories** (partner data)
   - Actualización continua (múltiples veces al día)

4. **Generación de alertas**
   - Match: dependencia + versión → advisory
   - Cálculo de severidad (CVSS score)
   - Notificación según configuración
   - Creación de issue en Security tab

**Ejemplo práctico:**

```json
// package.json
{
  "dependencies": {
    "express": "4.16.0"  // ← Versión vulnerable
  }
}
```

**Resultado:**
```
Alerta Dependabot:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📦 express
🔴 Critical severity
🆔 CVE-2022-24999
📝 express <4.17.3: Prototype pollution vulnerability
🔧 Recomendación: Upgrade to 4.17.3+
✅ Auto-PR disponible
```

### b) Cómo actuar ante alertas de GHAS

**Workflow de decisión:**

```
¿Alerta GHAS aparece?
       │
       ├─→ [1] EVALUAR SEVERIDAD
       │    ├─ Critical/High → Prioridad 1 (resolver en 7 días)
       │    ├─ Medium → Prioridad 2 (resolver en 30 días)
       │    └─ Low → Prioridad 3 (resolver en 90 días)
       │
       ├─→ [2] VERIFICAR EXPLOTABILIDAD
       │    ├─ ¿El código afectado está en uso?
       │    ├─ ¿Es alcanzable por usuarios?
       │    └─ ¿Existen mitigaciones?
       │
       ├─→ [3] INVESTIGAR CONTEXTO
       │    ├─ Leer descripción del CVE
       │    ├─ Revisar PoC (Proof of Concept)
       │    ├─ Consultar vendor advisories
       │    └─ Check si hay exploit público
       │
       └─→ [4] DECIDIR ACCIÓN

ACCIÓN A: FIX (Corregir)
  → Aceptar PR de Dependabot
  → Actualizar dependencia manualmente
  → Refactorizar código vulnerable
  → Test + deploy

ACCIÓN B: MITIGATE (Mitigar)
  → Implementar workaround temporal
  → Configurar WAF rules
  → Deshabilitar feature afectada
  → Monitor while waiting for patch

ACCIÓN C: ACCEPT RISK (Aceptar riesgo)
  → Dismiss alert con justificación
  → Documentar decisión
  → Set reminder para revisitar
  → Notify security team

ACCIÓN D: FALSE POSITIVE
  → Verify no es falso positivo
  → Dismiss "Won't fix" o "Used in tests"
  → Report to GitHub si es bug del scanner
```
**Opciones de dismissal:**

```yaml
Razones válidas para dismiss:
  - won't_fix: No se va a corregir (decisión de negocio)
  - false_positive: No es realmente vulnerable
  - used_in_tests: Solo usado en tests, no producción
  - tolerable_risk: Riesgo aceptado (documentado)
```

### c) Implicaciones de ignorar una alerta

**Riesgos técnicos:**
- 🔴 Explotación en producción
- 🔴 Data breach / compromiso
- 🔴 Lateral movement por attackers
- 🔴 Cadena de supply chain attacks

**Riesgos de negocio:**
- 💰 Multas de compliance (GDPR, PCI-DSS)
- 💰 Costo de incident response
- 💰 Pérdida de reputación
- 💰 Lawsuits de usuarios afectados

**Riesgos operativos:**
- ⚠️ Acumulación de tech debt
- ⚠️ Dificultad para upgrades futuros
- ⚠️ Complejidad de remediación creciente
- ⚠️ Alert fatigue del equipo

**Best practices:**
- ✅ **NUNCA ignorar sin justificación documentada**
- ✅ Establecer SLAs por severidad
- ✅ Require approval para dismiss de Critical/High
- ✅ Audit trail de todas las decisiones
- ✅ Revisit dismissed alerts periódicamente

### d) Rol del desarrollador al descubrir una alerta

**Responsabilidades:**

**1. Triage inmediato**
```bash
# Al ver alerta en PR
1. Leer descripción completa
2. Click en "Show paths" para ver data flow
3. Entender CWE y CVE relacionados
4. Verificar si es false positive
```
**2. Comunicación**

```yaml
- Si es critical/high:
    - Notify team lead INMEDIATAMENTE
    - Tag @security-team en PR
    - Crear incident ticket
    
- Si es medium/low:
    - Comentar en PR con plan de remediación
    - Estimar effort
    - Schedule fix en próximo sprint
```
**3. Remediación**

```python
# Para code scanning:
1. Leer documentación del CWE
2. Revisar ejemplos de fix
3. Aplicar fix siguiendo best practices
4. Agregar test que verifique la corrección
5. Re-run code scanning

# Para secret scanning:
1. Revocar secreto INMEDIATAMENTE
2. Rotar a nuevo secreto
3. Actualizar en secret manager
4. Audit logs para ver si fue expuesto
5. Commit fix

# Para Dependabot:
1. Review changelog de la nueva versión
2. Check breaking changes
3. Update y run test suite
4. Merge PR de Dependabot
```

**4. Documentación**

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

**5. Prevención futura**
- Agregar linting rules
- Actualizar team guidelines
- Share learnings en team meeting
- Contribuir queries a CodeQL si es nuevo pattern

### e) Diferencias en gestión de acceso por característica

**Secret Scanning:**

| Rol | Ver alertas | Dismiss alerts | Configurar | Bypass push protection |
|-----|-------------|----------------|------------|------------------------|
| Read | ❌ | ❌ | ❌ | ❌ |
| Triage | ❌ | ❌ | ❌ | ❌ |
| Write | ❌ | ❌ | ❌ | ✅ (con justificación) |
| Maintain | ❌ | ❌ | ❌ | ✅ |
| Admin | ✅ | ✅ | ✅ | ✅ |
| Security Manager | ✅ | ✅ | ✅ | N/A |

**Code Scanning:**

| Rol | Ver alertas | Dismiss alerts | Configurar workflow | Ver SARIF |
|-----|-------------|----------------|---------------------|-----------|
| Read | ✅ | ❌ | ❌ | ✅ |
| Triage | ✅ | ❌ | ❌ | ✅ |
| Write | ✅ | ✅ | ❌ | ✅ |
| Maintain | ✅ | ✅ | ✅ | ✅ |
| Admin | ✅ | ✅ | ✅ | ✅ |

**Dependabot:**

| Rol | Ver alertas | Dismiss alerts | Ver PRs | Merge PRs | Configurar |
|-----|-------------|----------------|---------|-----------|------------|
| Read | ✅ | ❌ | ✅ | ❌ | ❌ |
| Triage | ✅ | ❌ | ✅ | ❌ | ❌ |
| Write | ✅ | ✅ | ✅ | ✅ | ❌ |
| Maintain | ✅ | ✅ | ✅ | ✅ | ✅ |
| Admin | ✅ | ✅ | ✅ | ✅ | ✅ |

**Security Overview:**

| Rol | Vista organización | Vista empresa | Exportar datos | Gestionar campaigns |
|-----|-------------------|---------------|----------------|---------------------|
| Org member | ❌ | ❌ | ❌ | ❌ |
| Org owner | ✅ | ❌ | ✅ | ✅ |
| Security manager | ✅ | ❌ | ✅ | ✅ |
| Enterprise owner | ✅ | ✅ | ✅ | ✅ |

**Configuración de notificaciones:**

```yaml
# .github/workflows/notify.yml
# Control granular de quién recibe qué

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

### f) Dónde utilizar alertas de Dependabot en el SDLC

**1. Planning / Backlog Grooming**
- Revisar alertas de Dependabot
- Priorizar por severidad
- Estimar effort de updates
- Planificar en sprints

**2. Development**
- Monitorear nuevas alertas daily
- Review Dependabot PRs
- Test compatibility de updates

**3. Code Review / PR**
- Dependency review action bloquea PRs con vulnerabilidades
- Approve Dependabot PRs después de testing
- Merge fixes antes de features

**4. CI/CD Pipeline**
```yaml
# Integrar checks en pipeline
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
- Verificar 0 critical/high alerts antes de release
- Incluir security fixes en changelog
- Comunicar updates a stakeholders

**6. Post-deployment**
- Monitorear para nuevos advisories
- Auto-merge low-risk Dependabot PRs
- Weekly review de alertas abiertas

**7. Incident Response**
- Si exploit público emerge:
  - Dependabot alerta inmediatamente
  - Emergency patch deploy
  - Postmortem y lessons learned


**Automatización recomendada:**

```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "daily"
    # Auto-merge para patches de seguridad
    open-pull-requests-limit: 10
    labels:
      - "dependencies"
      - "security"
    reviewers:
      - "org/security-team"
    assignees:
      - "security-lead"
    
  # Grouping para múltiples updates
  - package-ecosystem: "npm"
    directory: "/frontend"
    groups:
      development-dependencies:
        dependency-type: "development"
      production-dependencies:
        dependency-type: "production"
```

**Enlaces:**
- https://docs.github.com/en/code-security/dependabot
- https://docs.github.com/en/code-security/supply-chain-security/understanding-your-software-supply-chain/about-the-dependency-graph
- https://docs.github.com/en/code-security/security-overview

---
