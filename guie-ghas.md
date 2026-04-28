# GUÍA COMPLETA CERTIFICACIÓN GITHUB ADVANCED SECURITY (GHAS)
## Guía de Estudio Nivel Superior - Examen GH-500

---

# TABLA DE CONTENIDO

1. [Dominio 1: Características y Funcionalidades de Seguridad de GHAS (15%)](docs/dominio-1.md)
2. [Dominio 2: Configurar y Usar el Escaneo de Secretos (15%)](docs/dominio-2.md)
3. [Dominio 3: Configurar y Usar Dependabot y Dependency Review (35%)](docs/dominio-3.md)
4. [Dominio 4: Configurar y Usar el Análisis de Código con CodeQL (25%)](docs/dominio-4.md)
5. [Dominio 5: Mejores Prácticas de GHAS, Resultados Y Medidas Correctivas (10%)](docs/dominio-5.md)
6. [Enlaces y Recursos Adicionales](docs/recursos.md)

---

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

<h1 id="dominio2">DOMINIO 2: CONFIGURAR Y USAR EL ESCANEO DE SECRETOS (15%)</h1>

## 2.1 Describir el escaneo de secretos

### ¿Qué es Secret Scanning?

**Definición**: Feature de GHAS que detecta credenciales, tokens y otros secretos que han sido commiteados accidentalmente en repositorios.

**Funcionamiento interno:**

1. **Escaneo inicial**: Al habilitar, escanea todo el historial del repositorio
2. **Escaneo continuo**: Cada push nuevo es escaneado automáticamente
3. **Pattern matching**: Usa regex patterns para identificar secretos
4. **Validación**: Verifica con el provider si el secreto es válido
5. **Alertas**: Notifica al usuario y al service provider

### Tipos de secretos detectados

**Categorías principales:**

```yaml
Tokens de autenticación:
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

Certificados y claves:
  - Private SSH keys
  - PGP private keys
  - TLS/SSL certificates
  - Code signing certificates

Credenciales de base de datos:
  - MongoDB connection strings
  - PostgreSQL passwords
  - MySQL credentials
  - Redis authentication

Cloud credentials:
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
- 200+ patterns predefinidos
- Validación automática con providers
- Revocación automática posible
- Actualizados por GitHub
- **Ejemplos**: AWS, Stripe, Slack, Azure

**Custom Patterns (User-defined):**
- Patrones específicos de organización
- Regex personalizados
- No hay validación automática
- Mantenimiento manual
- **Ejemplos**: Internal API keys, proprietary tokens

### Validity Checks (Comprobaciones de validez)

**¿Qué son?**

Cuando secret scanning detecta un secreto, intenta verificar si aún es válido:

```
Secreto detectado → Pattern match
        ↓
¿Provider soporta validación?
        ├─ SÍ → Llamar API del provider
        │        ├─ Activo ✅ → CRITICAL alert
        │        ├─ Inactivo ❌ → Low priority
        │        └─ Unknown ⚠️ → Medium priority
        │
        └─ NO → Crear alerta sin validación
```

**Estados de validez:**

| Estado | Significado | Prioridad | Acción |
|--------|-------------|-----------|--------|
| **Active** | Secreto válido y activo | 🔴 Critical | Revocar INMEDIATAMENTE |
| **Inactive** | Secreto revocado/expirado | 🟢 Low | Limpiar código |
| **Unknown** | No se pudo verificar | 🟡 Medium | Investigar manualmente |
| **No check** | Provider no soporta validación | 🟡 Medium | Asumir activo |

**Providers con validity checks:**

- ✅ GitHub (tokens)
- ✅ AWS (IAM keys)
- ✅ Google Cloud
- ✅ Azure
- ✅ Stripe
- ✅ Slack
- ✅ Twilio
- ✅ Dropbox
- ❌ Muchos custom patterns

**Ejemplo práctico:**

```python
# secrets.py - INCORRECTO ❌
GITHUB_TOKEN = "ghp_AbCd1234567890EfGhIjKlMnOpQrStUv"
AWS_KEY = "AKIA2345678901234567"

# Secret scanning detecta:
# 1. GitHub token
#    → Valida con GitHub API
#    → Estado: Active ✅
#    → Alerta: CRITICAL
#    → Acción: Token auto-revocado por GitHub
#
# 2. AWS key
#    → Valida con AWS
#    → Estado: Active ✅
#    → Alerta: CRITICAL
#    → Acción: Notificar al AWS account owner
```

### Arquitectura de Secret Scanning

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

**Enlaces:**
- https://docs.github.com/en/code-security/secret-scanning/introduction/about-secret-scanning
- https://docs.github.com/en/code-security/secret-scanning/introduction/supported-secret-scanning-patterns

---

## 2.2 Describir Push Protection

### ¿Qué es Push Protection?

**Definición**: Feature que **bloquea en tiempo real** cualquier push que contenga secretos, previniendo que lleguen al repositorio.

**Diferencia con secret scanning tradicional:**

```
Secret Scanning (tradicional):
  Developer commit → Push → Repository → Scan → Alert
  ❌ Secreto YA está en historial

Push Protection:
  Developer commit → Push BLOQUEADO → Notificación → Fix → Push again
  ✅ Secreto NUNCA llega al repositorio
```
### Cómo funciona Push Protection


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

### Configuración de Push Protection

**Niveles de habilitación:**

```yaml
Repositorio:
  Settings → Code security → Secret scanning
  ├─ Enable secret scanning ✅
  └─ Enable push protection ✅

Organización:
  Settings → Code security → Secret scanning
  ├─ Enable for all repositories
  ├─ Enable for new repositories
  └─ Enable push protection

Empresa:
  Settings → Policies → Advanced Security
  └─ Push protection policy for all orgs
```

**Opciones de bypass:**

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

### Bypass workflow

**Cuando un developer necesita bypass:**


```
Developer encuentra push bloqueado
        ↓
Click "Bypass protection"
        ↓
Proveer justificación:
  - "Testing vulnerability fix"
  - "False positive - not real secret"
  - "Legacy code - will fix in separate PR"
        ↓
    ┌───┴────┐
    │ Policy │
    └───┬────┘
        │
   ┌────┴────┐
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

**Audit trail:**

Todo bypass queda registrado:

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

### Experiencia del developer

**Sin push protection:**
```bash
$ git push origin main
Enumerating objects: 5, done.
Counting objects: 100% (5/5), done.
Writing objects: 100% (3/3), 289 bytes | 289.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0)
To github.com:company/api.git
   abc123..def456  main -> main

# ⚠️ Secreto está en el repositorio
# ⚠️ Alert aparece 30 segundos después
```

**Con push protection:**
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

# ✅ Secreto NO está en el repositorio
# ✅ Developer puede actuar ANTES de exposición
```

### Casos de uso y best practices

**Escenarios donde push protection es crítico:**

1. **Servicios cloud (AWS, Azure, GCP)**
   - Keys tienen acceso a recursos costosos
   - Explotación puede resultar en cryptomining
   - Facturas de $10k+ en 24 horas

2. **Payment processors (Stripe, PayPal)**
   - Acceso a transacciones financieras
   - PCI-DSS compliance requirements
   - Riesgo legal y reputacional

3. **Database credentials**
   - Acceso a customer PII
   - GDPR compliance
   - Data breach notifications

4. **Third-party APIs**
   - Quota exhaustion
   - Account suspension
   - Service disruption

**Best practices:**

```yaml
✅ DO:
  - Habilitar push protection en todos los repos
  - Require bypass justification
  - Set up delegated bypass for sensitive repos
  - Educate developers on secret management
  - Use secret managers (Vault, AWS Secrets Manager)
  - Rotate secrets regularly
  - Monitor bypass patterns

❌ DON'T:
  - Allow bypasses sin approval para production repos
  - Ignorar push protection alerts
  - Hardcode secrets "temporalmente"
  - Use comentarios como excusa para bypass
  - Disable push protection para "convenience"
```

**Integración con secret managers:**

```javascript
// ❌ INCORRECTO - Hardcoded
const apiKey = "sk_live_1234567890";

// ✅ CORRECTO - Secret manager
const apiKey = await secretManager.getSecret('stripe_api_key');

// ✅ CORRECTO - Environment variables
const apiKey = process.env.STRIPE_API_KEY;

// ✅ CORRECTO - GitHub Secrets (Actions)
// En workflow:
# ${{ secrets.STRIPE_API_KEY }}
```

### Limitaciones de Push Protection

**No protege contra:**
- ❌ Secrets en archivos binarios (imágenes, PDFs)
- ❌ Secrets obfuscados intencionalmente
- ❌ Secrets en repositorios privados no monitoreados
- ❌ Secrets compartidos verbalmente o por email
- ❌ Secrets en wikis, issues, discussions

**Workarounds necesarios:**
- Pre-commit hooks locales
- IDE plugins (VS Code extension)
- Git hooks en workstation
- Security awareness training

**Enlaces:**
- https://docs.github.com/en/code-security/secret-scanning/introduction/about-push-protection
- https://docs.github.com/en/code-security/secret-scanning/using-advanced-secret-scanning-and-push-protection-features/delegated-bypass-for-push-protection

---

## 2.3 Disponibilidad de Secret Scanning por tipo de repositorio

### Repositorios Públicos

**Secret scanning: ✅ GRATIS (habilitado por defecto)**

Características incluidas:
- ✅ Escaneo automático de todo el historial
- ✅ 200+ partner patterns
- ✅ Validity checks con providers
- ✅ Notificaciones a service providers
- ✅ Revocación automática (algunos providers)
- ✅ Push protection (NUEVO desde 2025)
- ❌ Custom patterns (no disponible)
- ❌ Security overview (no disponible)
- ❌ Delegated bypass (no disponible)

**Razón**: Proteger el ecosistema open source y prevenir leaked credentials.

### Repositorios Privados SIN GHAS

**Secret scanning: ❌ NO DISPONIBLE**

Para habilitar, necesitas:
- **GitHub Secret Protection** ($19/mes por committer), o
- **GitHub Enterprise** (incluía GHAS hasta abril 2025)

Sin licencia:
- ❌ No hay escaneo de secretos
- ❌ No hay push protection
- ❌ No hay alertas
- ⚠️ Riesgo: secretos pueden estar expuestos sin detección

### Repositorios Privados CON GHAS (GitHub Secret Protection)

**Secret scanning: ✅ COMPLETO**

Características adicionales:
- ✅ Escaneo de repositorios privados
- ✅ Push protection
- ✅ Custom patterns (organization-level)
- ✅ Delegated bypass workflows
- ✅ Delegated alert dismissal
- ✅ Security overview
- ✅ Security campaigns
- ✅ Copilot secret scanning (AI-powered)
- ✅ Advanced analytics

**Comparación completa:**

| Feature | Público | Privado sin GHAS | Privado con Secret Protection |
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

### Habilitación por nivel

**A nivel de repositorio:**
```yaml
Settings → Code security and analysis
  └─ Secret scanning
      ├─ [✓] Secret scanning (Free for public, GHAS for private)
      └─ [✓] Push protection (Free for public, GHAS for private)
```

**A nivel de organización:**
```yaml
Settings → Code security and analysis
  ├─ Enable for all existing repositories
  ├─ Enable for new repositories
  └─ Configure default settings
      ├─ Push protection: Enabled
      ├─ Bypass allowed: Require justification
      └─ Custom patterns: [Add patterns]
```

**A nivel de empresa:**
```yaml
Policies → Advanced Security
  ├─ Enforce for all organizations
  ├─ Allow organizations to override
  └─ Billing (per active committer)
```

### Casos especiales
**Forked repositories:**
- Public fork of public: ✅ Secret scanning enabled
- Private fork of private: ⚠️ Depends on parent's GHAS license
- Private fork of public: ❌ Requires GHAS license

**Archived repositories:**
- ✅ Secret scanning continúa activo
- ❌ No se generan nuevas alertas (no hay nuevos commits)
- ℹ️ Alertas existentes permanecen visibles

**Template repositories:**
- Settings se copian a repos creados desde template
- GHAS se requiere en cada repo, no se hereda

**Mirrored repositories:**
- ✅ Secret scanning funciona en mirrors
- ⚠️ Alertas se crean en el repo espejo
- ℹ️ Push protection aplica en mirror, no en origen

**Enlaces:**
- https://docs.github.com/en/get-started/learning-about-github/about-github-advanced-security#about-advanced-security-features

---

## 2.4 Habilitar Secret Scanning para repositorios privados

### Prerrequisitos

**Licencias requeridas:**
- ✅ GitHub Secret Protection ($19/active committer/mes), o
- ✅ GitHub Enterprise (legacy bundle)

**Permisos necesarios:**
- Repository: Admin role
- Organization: Owner o Security manager
- Enterprise: Enterprise owner

**Verificación de elegibilidad:**

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

### Habilitación paso a paso

#### Método 1: Via Web UI (Repositorio individual)

**Paso 1**: Navegar a Settings
```
Repositorio → Settings tab
```

**Paso 2**: Ir a Security
```
Sidebar → Code security and analysis
```

**Paso 3**: Habilitar Advanced Security
```
[ ] GitHub Advanced Security
    └─ [Enable] ← Click aquí primero
```

**Paso 4**: Habilitar Secret Scanning
```
[✓] GitHub Advanced Security (ahora habilitado)
    ├─ [ ] Secret scanning
    │   └─ [Enable] ← Click para habilitar
    └─ [ ] Push protection
        └─ [Enable] ← Opcional pero recomendado
```

**Paso 5**: Esperar escaneo inicial
```
⏳ Scanning repository history...
   Commits scanned: 1,234 / 5,678
   ETA: 2 minutes

✅ Initial scan complete
   0 secrets found
```

#### Método 2: Via API (Programático)

**Habilitar GHAS:**
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

**Verificar estado:**
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

#### Método 3: Via GitHub CLI

```bash
# Habilitar GHAS + Secret Scanning
gh api -X PATCH /repos/:owner/:repo \
  -f security_and_analysis[advanced_security][status]=enabled \
  -f security_and_analysis[secret_scanning][status]=enabled \
  -f security_and_analysis[secret_scanning_push_protection][status]=enabled

# Verificar
gh api /repos/:owner/:repo | jq '.security_and_analysis'
```

#### Método 4: Bulk habilitación (Organización)

**Via UI:**
```
Organization Settings
  → Code security and analysis
  → Configure security and analysis features
      ├─ [Enable all] ← Habilitar para todos los repos
      └─ [✓] Automatically enable for new repositories
```

**Via script (Python):**
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
    
    # Habilitar GHAS + Secret Scanning
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

### Configuración post-habilitación



































