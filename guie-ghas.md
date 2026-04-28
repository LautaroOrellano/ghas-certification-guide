# GUÍA COMPLETA CERTIFICACIÓN GITHUB ADVANCED SECURITY (GHAS)
## Guía de Estudio Nivel Superior - Examen GH-500

---

# TABLA DE CONTENIDO

1. [Dominio 1: Características y Funcionalidades de Seguridad de GHAS (15%)](#dominio1)
2. [Dominio 2: Configurar y Usar el Escaneo de Secretos (15%)](#dominio2)
3. [Dominio 3: Configurar y Usar Dependabot y Dependency Review (35%)](#dominio3)
4. [Dominio 4: Configurar y Usar el Análisis de Código con CodeQL (25%)](#dominio4)
5. [Dominio 5: Mejores Prácticas de GHAS, Resultados Y Medidas Correctivas (10%)](#dominio5)
6. [Enlaces y Recursos Adicionales](#recursos)

---

# DOMINIO 1: CARACTERÍSTICAS Y FUNCIONALIDADES DE SEGURIDAD DE GHAS (15%) {#dominio1}

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
