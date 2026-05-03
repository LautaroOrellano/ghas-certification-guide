# GUÍA ESTRATÉGICA PARA EL EXAMEN GHAS (GH-500)
## Decision-Making Framework, Exam Tips & Practice Questions

> **Complemento esencial a la Guía Técnica GHAS**  
> Esta guía se enfoca en **cómo aprobar el examen**.

---

## 📋 TABLA DE CONTENIDOS

1. [Decision-Making Framework](#decision-framework)
2. [Gobernanza y Roles Avanzados](#gobernanza-avanzada)
3. [Security Overview en Profundidad](#security-overview-profundo)
4. [Comparativas Críticas](#comparativas-criticas)
5. [Trampas Comunes del Examen](#exam-traps)
6. [Escenarios del Mundo Real](#escenarios-reales)
7. [100 Preguntas de Práctica](#preguntas-practica)
8. [Estrategia de Examen](#estrategia-examen)

---

# DECISION-MAKING FRAMEWORK {#decision-framework}

## 🎯 La regla de oro del examen

> **El examen NO pregunta "¿cómo funciona X?"**  
> **El examen pregunta "¿qué usarías en este escenario?"**

## Framework de 4 pasos

```
┌─────────────────────────────────────────────────────┐
│ 1. IDENTIFICAR EL PROBLEMA                          │
│    ¿Qué está pasando? ¿Cuál es el objetivo?        │
└─────────────────┬───────────────────────────────────┘
                  │
┌─────────────────▼───────────────────────────────────┐
│ 2. CLASIFICAR EL MOMENTO                            │
│    ¿Antes del commit? ¿En PR? ¿Después del merge?  │
└─────────────────┬───────────────────────────────────┘
                  │
┌─────────────────▼───────────────────────────────────┐
│ 3. SELECCIONAR LA FEATURE                           │
│    ¿Qué feature de GHAS resuelve esto?             │
└─────────────────┬───────────────────────────────────┘
                  │
┌─────────────────▼───────────────────────────────────┐
│ 4. VERIFICAR PREREQUISITOS                          │
│    ¿Qué se necesita? ¿Permisos? ¿Licencias?        │
└─────────────────────────────────────────────────────┘
```

---

## 📊 Matriz de Decisión Rápida

### Por TIPO DE PROBLEMA

| Problema | Feature Principal | Feature Secundario | Cuándo actúa |
|----------|------------------|-------------------|--------------|
| **Secreto en código** | Secret scanning | Push protection | Después / Antes |
| **Código vulnerable** | CodeQL | - | Durante PR |
| **Dependencia vulnerable** | Dependabot alerts | Security updates | Después |
| **Prevenir vuln en PR** | Dependency review | CodeQL | Durante PR |
| **Actualizar deps automáticamente** | Dependabot security updates | - | Después de alert |
| **Visibilidad org-wide** | Security Overview | - | Continuo |
| **Cumplimiento/compliance** | Security policies | Rulesets | Continuo |

### Por MOMENTO EN EL SDLC

```
ANTES DEL COMMIT:
  └─ Push protection
      ├─ Bloquea commits con secretos
      └─ Requiere: Secret scanning habilitado

DURANTE EL PR:
  ├─ CodeQL (análisis de código)
  ├─ Dependency review (nuevas vulnerabilidades)
  └─ Branch protection rules
      └─ Requiere: Status checks configurados

DESPUÉS DEL MERGE:
  ├─ Secret scanning (detección)
  ├─ Dependabot alerts (monitoreo continuo)
  ├─ CodeQL scheduled scans
  └─ Security campaigns (remediación)

CONTINUO:
  └─ Security Overview (métricas y tendencias)
```

---

## 🎓 Escenarios de Decisión (Tipo Examen)

### Escenario 1: Secretos

**Pregunta**: Un desarrollador acaba de pushear un AWS access key a un repositorio público.

**¿Qué sucede? (selecciona todas las correctas)**

A) ✅ Secret scanning detecta el secreto  
B) ✅ GitHub notifica al desarrollador  
C) ✅ GitHub notifica a AWS  
D) ❌ El secreto es automáticamente revocado  
E) ✅ Se crea una alerta en Security tab  

**Explicación**:
- Secret scanning detecta automáticamente en repos públicos (A, E)
- Notifica al owner del repo Y al service provider (B, C)
- NO revoca automáticamente - eso debe hacerlo el usuario o AWS (D es falso)

**¿Qué deberías hacer?**

```
1. INMEDIATO (< 5 min):
   └─ Revocar el AWS access key en AWS Console

2. CORTO PLAZO (< 1 hora):
   ├─ Revisar CloudTrail logs
   ├─ Verificar uso no autorizado
   └─ Rotar credenciales

3. PREVENCIÓN:
   ├─ Habilitar push protection
   └─ Training al equipo
```

---

### Escenario 2: Dependencias

**Pregunta**: Tu equipo quiere que los PRs sean bloqueados automáticamente si introducen dependencias con vulnerabilidades HIGH o CRITICAL.

**¿Qué configuración necesitas?**

A) Dependabot alerts  
B) Dependabot security updates  
C) Dependency review + branch protection ← **CORRECTO**  
D) CodeQL  

**Explicación**:
- **Dependabot alerts**: Solo NOTIFICA, no bloquea
- **Security updates**: Crea PRs de fix, no bloquea
- **Dependency review**: Analiza cambios en PR + branch protection = BLOQUEO
- **CodeQL**: Analiza código, no dependencias

**Configuración completa**:

```yaml
# 1. Workflow de dependency review
# .github/workflows/dependency-review.yml
name: Dependency Review
on: [pull_request]

jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/dependency-review-action@v4
        with:
          fail-on-severity: high

# 2. Branch protection rule
Settings → Branches → Branch protection rules
  ✓ Require status checks to pass
    ✓ Dependency Review
```

---

### Escenario 3: CodeQL

**Pregunta**: Un desarrollador pregunta por qué CodeQL tarda 30 minutos en su repositorio Java, pero solo 5 minutos en JavaScript.

**¿Cuál es la razón correcta?**

A) Java tiene más vulnerabilidades  
B) Java requiere compilación, JavaScript no ← **CORRECTO**  
C) CodeQL tiene menos queries para JavaScript  
D) El runner es más lento para Java  

**Explicación**:

| Lenguaje | Tipo | Build Required | Análisis |
|----------|------|----------------|----------|
| **Java** | Compilado | ✅ SÍ | Lento (10-60 min) |
| **JavaScript** | Interpretado | ❌ NO | Rápido (2-10 min) |
| **Python** | Interpretado | ❌ NO | Rápido (2-10 min) |
| **C++** | Compilado | ✅ SÍ | Lento (20-120 min) |

**Proceso de CodeQL para lenguajes compilados**:

```
1. Initialize CodeQL
   └─ Setup CodeQL database
   
2. BUILD (← AQUÍ está el tiempo)
   ├─ Download dependencies
   ├─ Compile código
   └─ Generate artifacts
   
3. Analyze
   └─ Run queries on database
   
4. Upload results
```

---

### Escenario 4: Permisos

**Pregunta**: Un contractor externo necesita ver alertas de Dependabot en repositorios privados, pero NO debe poder modificar código ni configuración.

**¿Qué rol le otorgas?**

A) Read ← **CORRECTO**  
B) Triage  
C) Write  
D) Security Manager  

**Explicación detallada**:

| Rol | Ver Dependabot Alerts | Dismiss Alerts | Ver Código | Modificar Código |
|-----|----------------------|----------------|-----------|------------------|
| **Read** | ✅ | ❌ | ✅ | ❌ |
| **Triage** | ✅ | ❌ | ✅ | ❌ |
| **Write** | ✅ | ✅ | ✅ | ✅ |
| **Security Manager** | ✅ | ✅ | ✅ | ❌ |

**¿Por qué Read y no Security Manager?**

- **Security Manager**: Es un rol a nivel ORGANIZACIÓN, no repositorio
- **Security Manager**: Tiene permisos de seguridad en TODOS los repos
- **Read**: Cumple exactamente el requisito sin over-privileging

**Diferencia CRÍTICA**:

```yaml
Security Manager (Organization-level):
  ✅ Ver security alerts en TODOS los repos de la org
  ✅ Configurar security features
  ✅ Manage security campaigns
  ❌ No puede modificar código

Read (Repository-level):
  ✅ Ver código
  ✅ Ver Dependabot alerts (solo en ese repo)
  ❌ No puede modificar nada
```

---

### Escenario 5: Org vs Repo Settings

**Pregunta**: La organización tiene secret scanning habilitado para "all repositories". Un admin de repo lo deshabilita a nivel de repositorio.

**¿Qué sucede?**

A) Se deshabilita (repo override org) ← **Depende de enforcement**  
B) Permanece habilitado (org override repo)  
C) GitHub bloquea la acción  
D) Requiere approval de org owner  

**Explicación - Jerarquía de Settings**:

```
┌─────────────────────────────────────────┐
│ ENTERPRISE                               │
│  ├─ Policies (enforce, allow, disable)  │
│  └─ Si "enforce" → No se puede cambiar  │
└──────────────┬──────────────────────────┘
               │
┌──────────────▼──────────────────────────┐
│ ORGANIZATION                             │
│  ├─ Can enable for all repos            │
│  ├─ Can auto-enable for new repos       │
│  └─ Si no hay enterprise policy:        │
│     repo admin puede override           │
└──────────────┬──────────────────────────┘
               │
┌──────────────▼──────────────────────────┐
│ REPOSITORY                               │
│  └─ Settings aplican solo si:           │
│     - No hay enterprise enforcement     │
│     - Org permite override              │
└─────────────────────────────────────────┘
```

**Caso A**: Sin enterprise policy
- Org habilita secret scanning
- Repo admin puede deshabilitar
- ✅ **Se deshabilita a nivel repo**

**Caso B**: Con enterprise enforcement
- Enterprise policy: "Enforced"
- Repo admin intenta deshabilitar
- ❌ **Opción está grayed out / bloqueada**

**Respuesta completa**:
> Depende de si hay enterprise policy enforcement. Sin enforcement: el repo puede override. Con enforcement: no se puede cambiar.

---

## 🔄 Decision Trees (Árboles de Decisión)

### Decision Tree: "Tengo una vulnerabilidad, ¿qué hago?"

```
¿Dónde se detectó la vulnerabilidad?
    │
    ├─ En DEPENDENCIA
    │   │
    │   ├─ ¿Ya está en main?
    │   │   ├─ SÍ → Dependabot alert existe
    │   │   │        └─ ¿Hay PR de Dependabot?
    │   │   │            ├─ SÍ → Review y merge PR
    │   │   │            └─ NO → Crear issue/PR manual
    │   │   │
    │   │   └─ NO → Está en PR
    │   │            └─ Dependency review debe haberlo bloqueado
    │   │                ├─ Si no lo bloqueó: Configurar dependency review
    │   │                └─ Si lo bloqueó: No mergear hasta fix
    │   │
    │   └─ ¿Es directa o transitiva?
    │       ├─ Directa → Actualizar en package.json
    │       └─ Transitiva → Actualizar parent dependency
    │                       o usar overrides/resolutions
    │
    └─ En CÓDIGO (CodeQL)
        │
        ├─ ¿Dónde está?
        │   ├─ Production code → FIX inmediato
        │   ├─ Test code → FIX (menor prioridad)
        │   └─ Example/docs → Dismiss o fix
        │
        ├─ ¿Severidad?
        │   ├─ Critical → SLA: 24h
        │   ├─ High → SLA: 7 días
        │   ├─ Medium → SLA: 30 días
        │   └─ Low → SLA: 90 días
        │
        └─ ¿Es explotable?
            ├─ SÍ → Prioridad máxima
            │        └─ Ver "Show paths" para entender flujo
            │            └─ Implementar fix según recomendación
            │
            └─ NO → ¿Por qué?
                ├─ Código no alcanzable → Documentar + dismiss
                ├─ Input ya sanitizado → Documentar + dismiss
                ├─ Mitigaciones existentes → Risk acceptance
                └─ False positive → Dismiss + reportar a GitHub
```

---

### Decision Tree: "¿Qué feature de GHAS habilito primero?"

```
¿Qué tipo de organización eres?
    │
    ├─ OPEN SOURCE (repos públicos)
    │   │
    │   └─ ✅ Ya tienes GRATIS:
    │       ├─ Secret scanning
    │       ├─ Push protection
    │       ├─ Dependabot alerts
    │       └─ Dependabot security updates
    │       
    │       📋 Setup recomendado:
    │       1. Habilitar CodeQL (manual pero gratis)
    │       2. Configure branch protection
    │       3. Setup Dependabot version updates (dependabot.yml)
    │
    └─ ENTERPRISE (repos privados)
        │
        ├─ ¿Tienes GHAS license?
        │   │
        │   ├─ NO → Prioridad:
        │   │        1. Comprar GHAS (o al menos GitHub Code Security)
        │   │        2. Mientras tanto: Solo Dependabot alerts disponible
        │   │
        │   └─ SÍ → Rollout strategy:
        │
        └─ FASE 1 (Semana 1-2): VISIBILIDAD
            │   ├─ Habilitar Dependency graph
            │   ├─ Habilitar Dependabot alerts
            │   └─ Habilitar Security Overview
            │       └─ Identificar estado actual
            │
            ├─ FASE 2 (Semana 3-4): DETECCIÓN
            │   ├─ Habilitar Secret scanning
            │   ├─ Configurar custom patterns (si aplica)
            │   └─ Enable CodeQL (default setup) en 10-20% repos
            │       └─ Piloto en repos no-críticos
            │
            ├─ FASE 3 (Mes 2): PREVENCIÓN
            │   ├─ Habilitar Push protection
            │   ├─ Setup Dependency review
            │   ├─ Configure branch protection rules
            │   └─ Expandir CodeQL a 50% repos
            │
            ├─ FASE 4 (Mes 3): REMEDIACIÓN
            │   ├─ Habilitar Dependabot security updates
            │   ├─ Configure auto-merge policies
            │   ├─ Setup Security campaigns
            │   └─ CodeQL en 100% repos
            │
            └─ FASE 5 (Mes 4+): OPTIMIZACIÓN
                ├─ Custom CodeQL queries
                ├─ Advanced workflows
                ├─ Integration con SIEM
                ├─ Automated governance
                └─ Continuous improvement
```

---

### Decision Tree: "Un PR tiene alertas de seguridad, ¿puedo mergear?"

```
PR tiene alertas de seguridad
    │
    ├─ ¿Qué tipo de alerta?
    │
    ├─── SECRET SCANNING
    │     │
    │     ├─ ¿Push protection está habilitado?
    │     │   ├─ SÍ → El commit fue bloqueado
    │     │   │        └─ Este escenario no debería existir
    │     │   │            └─ ¿Hubo bypass?
    │     │   │                ├─ Con justificación válida → Revisar caso
    │     │   │                └─ Sin justificación → BLOCK merge
    │     │   │
    │     │   └─ NO → Secreto está en el branch
    │     │            └─ ❌ NO MERGEAR
    │     │                ├─ 1. Revocar secreto INMEDIATAMENTE
    │     │                ├─ 2. Limpiar historial (git filter-repo)
    │     │                ├─ 3. Force push cleaned branch
    │     │                └─ 4. Re-review PR
    │     │
    ├─── CODEQL
    │     │
    │     ├─ ¿Severidad?
    │     │   │
    │     │   ├─ CRITICAL/HIGH
    │     │   │   └─ ❌ NO MERGEAR (por policy)
    │     │   │       └─ Requiere fix O exception approval
    │     │   │           ├─ Fix → Mejor opción
    │     │   │           └─ Exception → Requiere:
    │     │   │               ├─ Security team approval
    │     │   │               ├─ Documented justification
    │     │   │               ├─ Mitigations in place
    │     │   │               └─ Time-bound (review en X meses)
    │     │   │
    │     │   ├─ MEDIUM
    │     │   │   └─ ⚠️ Decisión según policy
    │     │   │       ├─ Strict policy → Block
    │     │   │       └─ Permissive → Warn + merge
    │     │   │           └─ Crear issue para fix
    │     │   │
    │     │   └─ LOW
    │     │       └─ ✅ Generalmente OK mergear
    │     │           └─ Add to backlog
    │     │
    │     └─ ¿Es false positive?
    │         ├─ SÍ → Dismiss alert
    │         │        └─ Document reason
    │         │            └─ ✅ Merge OK
    │         │
    │         └─ NO → Ver árbol de severidad arriba
    │
    └─── DEPENDENCY REVIEW
          │
          ├─ ¿Introduce vulnerabilidades nuevas?
          │   │
          │   ├─ SÍ
          │   │   ├─ ¿Severidad?
          │   │   │   ├─ Critical/High → ❌ BLOQUEADO por workflow
          │   │   │   │                    └─ Update dependency antes de merge
          │   │   │   │
          │   │   │   └─ Medium/Low → Según config de fail-on-severity
          │   │   │                     ├─ Si está configurado bloquear → ❌ NO MERGE
          │   │   │                     └─ Si no → ⚠️ Warning, puede mergear
          │   │   │
          │   │   └─ ¿Licencia prohibida?
          │   │       ├─ SÍ → ❌ BLOQUEADO
          │   │       │        └─ Remover dependencia o buscar alternativa
          │   │       │
          │   │       └─ NO → Continuar
          │   │
          │   └─ NO → ✅ Dependency review PASS
          │            └─ OK para merge (desde perspectiva dependencies)
          │
          └─ ¿Branch protection requiere este check?
              ├─ SÍ → Debe pasar para mergear
              └─ NO → Es informativo solamente
```

---

# GOBERNANZA Y ROLES AVANZADOS {#gobernanza-avanzada}

## 🏢 Jerarquía de Permisos Completa

### Nivel 1: Repository Roles

| Rol | Permisos de Seguridad |
|-----|----------------------|
| **Read** | • Ver código<br>• Ver Dependabot alerts<br>• Ver discusiones |
| **Triage** | • Todo de Read<br>• Gestionar issues y PRs<br>• **NO puede ver Code scanning ni Secret scanning** |
| **Write** | • Todo de Triage<br>• Push código<br>• Dismiss Dependabot alerts<br>• **NO puede ver Code scanning ni Secret scanning** |
| **Maintain** | • Todo de Write<br>• Gestionar settings (limitado)<br>• **NO puede ver Code scanning ni Secret scanning** |
| **Admin** | • Todo de Maintain<br>• **VER y gestionar Code scanning**<br>• **VER y gestionar Secret scanning**<br>• Configurar GHAS features<br>• Gestionar branch protection |

**⚠️ CRÍTICO PARA EL EXAMEN**:
- Solo **Admin** puede ver alertas de Code scanning y Secret scanning en repos privados
- **Dependabot alerts** son visibles para **Read+** (excepción a la regla)

---

### Nivel 2: Organization Roles

| Rol | Alcance | Permisos GHAS |
|-----|---------|---------------|
| **Member** | Repos asignados | Según rol en cada repo |
| **Outside Collaborator** | Repos específicos | Según rol asignado |
| **Owner** | Toda la org | • Todos los permisos<br>• Configure org-wide security<br>• Manage billing<br>• Enforce policies |
| **Security Manager** | Toda la org (security only) | • **Ver** security alerts en todos los repos<br>• **Gestionar** security settings<br>• **NO puede** ver/modificar código<br>• **NO puede** cambiar repo settings no-security |

**Security Manager en Detalle**:

```yaml
✅ PUEDE:
  - Ver Code scanning alerts (todos los repos)
  - Ver Secret scanning alerts (todos los repos)
  - Ver Dependabot alerts (todos los repos)
  - Dismiss/reopen alerts
  - Configurar security features
  - Manage security campaigns
  - Access Security Overview
  - Configure custom patterns
  - Manage security policies

❌ NO PUEDE:
  - Ver código (a menos que tenga rol adicional en repo)
  - Modificar código
  - Merge PRs
  - Cambiar branch protection (si no es security-related)
  - Manage billing
  - Add/remove members
  - Delete repositories
```

**Cuándo usar Security Manager**:

```
✅ Usar cuando:
  - Equipo de seguridad centralizado
  - Auditoría cross-repo
  - Compliance officer
  - Security consultant externo

❌ NO usar cuando:
  - Necesita modificar código
  - Necesita gestionar repos
  - Es un developer que necesita ver alerts en SU repo
    └─ Mejor: Repository role (Read/Write/Admin)
```

---

### Nivel 3: Enterprise Policies

**Solo disponible en GitHub Enterprise Cloud**

```yaml
Enterprise Admin puede:
  ├─ Enforced: TODAS las orgs DEBEN tener habilitado
  │   └─ Orgs no pueden deshabilitar
  │   └─ Repos no pueden deshabilitar
  │
  ├─ Enabled: Habilitado por default, orgs pueden override
  │   └─ Org puede deshabilitar
  │   └─ Repos pueden deshabilitar (si org permite)
  │
  └─ Disabled: Deshabilitado enterprise-wide
      └─ Orgs NO pueden habilitar

Aplica a:
  - Secret scanning
  - Push protection
  - Dependabot alerts
  - Dependabot security updates
  - Code scanning
```

**Ejemplo de Enforcement**:

```
Escenario 1: Enterprise enforced secret scanning
  Enterprise: Enforced ✅
    └─ Organization A: Enforced (no choice)
        └─ Repo 1: Enforced (no choice)
        └─ Repo 2: Enforced (no choice)
    └─ Organization B: Enforced (no choice)
        └─ Repo 3: Enforced (no choice)

Escenario 2: Enterprise enabled (not enforced)
  Enterprise: Enabled ⚙️
    ├─ Organization A: Enabled (can change)
    │   ├─ Repo 1: Enabled (can change)
    │   └─ Repo 2: Disabled (admin chose to disable)
    │
    └─ Organization B: Disabled (owner chose to disable)
        └─ Repo 3: Disabled (inherited from org)
```

---

## 🔐 Security Policies y Rulesets

### Branch Protection Rules (Legacy)

```yaml
Limitaciones:
  ❌ Solo protege branches específicos
  ❌ Configuración por repo
  ❌ No se puede aplicar a org-wide
  ❌ Difícil de mantener a escala

Usa cuando:
  - Repo individual
  - Configuración simple
  - No tienes Enterprise
```

### Repository Rulesets (Nuevo - Recomendado)

```yaml
Ventajas:
  ✅ Aplica a múltiples branches con patterns
  ✅ Puede ser org-wide
  ✅ Bypass con approval workflow
  ✅ Más flexible y granular

Niveles:
  1. Repository ruleset
     └─ Aplica solo a ese repo
  
  2. Organization ruleset
     └─ Aplica a repos seleccionados o todos
     
  3. Enterprise ruleset (GHEC)
     └─ Aplica a todas las orgs

Ejemplo de Configuración:
  Name: Security Standards
  Target: Default branch + release/*
  
  Rules:
    ✓ Require status checks to pass:
      - CodeQL / Analyze (javascript)
      - Dependency Review
    
    ✓ Require pull request before merging
      - Require 1 approval
      - Dismiss stale approvals
      - Require review from code owners
    
    ✓ Block force pushes
    ✓ Require signed commits
  
  Bypass:
    - Organization admins (with approval)
    - Security team (no approval needed)
```

**Comparación Crítica**:

| Feature | Branch Protection | Rulesets |
|---------|------------------|----------|
| **Scope** | Por branch | Por pattern |
| **Level** | Repository | Repo/Org/Enterprise |
| **Bypass** | Simple on/off | Approval workflow |
| **Status checks** | Lista simple | Conditions + matrix |
| **Maintenance** | Manual por repo | Centralizado |
| **Recomendado** | ❌ Legacy | ✅ Usar esto |

---

## 📊 Security Policies Matrix

### ¿Qué feature se puede configurar dónde?

| Feature | Repository | Organization | Enterprise |
|---------|-----------|--------------|------------|
| **Secret Scanning** | ✅ Enable/Disable | ✅ Enable for all<br>✅ Auto-enable new | ✅ Enforce policy |
| **Push Protection** | ✅ Enable/Disable | ✅ Enable for all<br>✅ Delegated bypass | ✅ Enforce policy |
| **Custom Patterns** | ❌ | ✅ Org-wide patterns | ❌ |
| **Code Scanning** | ✅ Configure workflow | ✅ Default setup org-wide | ✅ Enforce policy |
| **CodeQL Queries** | ✅ Custom config | ✅ Org default queries | ❌ |
| **Dependabot Alerts** | ✅ Enable/Disable | ✅ Enable for all | ✅ Enforce policy |
| **Security Updates** | ✅ Enable/Disable | ✅ Enable for all | ✅ Enforce policy |
| **Dependency Review** | ✅ Workflow config | ✅ Required workflow | ✅ Policy |
| **Rulesets** | ✅ Repo rulesets | ✅ Org rulesets | ✅ Enterprise rulesets |
| **Security Overview** | ❌ | ✅ Org-level view | ✅ Enterprise view |

---

## 🎯 Governance Best Practices

### Estrategia de Rollout Empresarial

```yaml
Fase 1: DISCOVERY (Mes 1)
  Objetivo: Entender estado actual
  
  Acciones:
    1. Enable Security Overview
       └─ Identificar qué repos ya tienen GHAS
    
    2. Enable Dependency Graph everywhere
       └─ Visibilidad de dependencias
    
    3. Run security assessment
       └─ Generar baseline metrics
    
  Métricas:
    - % repos con GHAS habilitado
    - Total de alertas por tipo
    - Repos de alto riesgo (críticas abiertas)

Fase 2: PILOT (Mes 2)
  Objetivo: Validar configuración
  
  Selección de repos:
    ✓ 5-10 repos no-críticos
    ✓ Diferentes lenguajes
    ✓ Equipos diversos
    ✓ Representativos
  
  Acciones:
    1. Enable todas las features
    2. Configure workflows
    3. Train teams
    4. Collect feedback
  
  Success Criteria:
    - <10% false positive rate
    - <7 días MTTR para high severity
    - >80% developer satisfaction

Fase 3: EXPANSION (Mes 3-4)
  Objetivo: Escalar a 50% repos
  
  Priorización:
    1. Repos con datos sensibles
    2. Production apps
    3. Public-facing services
    4. Compliance-required
  
  Automation:
    - Script de habilitación bulk
    - Auto-enable para new repos
    - Centralized policies

Fase 4: FULL ROLLOUT (Mes 5-6)
  Objetivo: 100% coverage
  
  Enforcement:
    - Organization policies
    - Required workflows
    - Branch protection universal
  
  Governance:
    - Security campaigns
    - Regular audits
    - Metrics dashboards

Fase 5: OPTIMIZATION (Ongoing)
  Objetivo: Continuous improvement
  
  Activities:
    - Custom queries
    - Integration con SIEM
    - Automated remediation
    - Developer education
```

---

### Organization Settings - Checklist Completo

```yaml
Code security and analysis:
  
  Dependency graph:
    [✓] Enable for all repositories
    [✓] Automatically enable for new repositories
  
  Dependabot:
    [✓] Enable Dependabot alerts for all repositories
    [✓] Enable Dependabot security updates for all repositories
    [✓] Automatically enable for new repositories
    
    Grouping (GitHub Code Security required):
      [✓] Enable auto-triage rules
      - Rule 1: Auto-dismiss low severity without patch
      - Rule 2: Auto-create PRs for critical in production
  
  Code scanning:
    [✓] Automatically enable for new repositories
    [ ] Require approval for new workflows (si quieres control)
    
    Default setup:
      Languages: [✓] Auto-detect
      Query suite: security-extended
      
  Secret scanning:
    [✓] Enable for all repositories
    [✓] Enable push protection for all repositories
    [✓] Automatically enable for new repositories
    
    Push protection:
      [✓] Allow bypasses
      [✓] Require justification for bypass
      [ ] Require approval for bypass (Enterprise only)
      Bypass expires: 7 days
    
    Custom patterns:
      - Pattern 1: Internal API keys
      - Pattern 2: Database connection strings
      - Pattern 3: JWT tokens
  
  Security managers:
    Teams:
      - @org/security-team
      - @org/compliance-team

Member privileges:
  Base permissions: Read
  
  Repository creation:
    [✓] Allow members to create repositories
    [ ] Allow members to create public repositories (si es privado)
  
  Repository visibility change:
    [ ] Allow members to change repository visibilities

Actions permissions:
  [✓] Allow GitHub Actions
  [✓] Allow actions created by GitHub
  [✓] Allow actions by Marketplace verified creators
  [ ] Allow specified actions (mejor control)

Webhooks:
  - Webhook 1: Security alerts → SIEM
  - Webhook 2: Dependabot → Slack #security
```

---

# SECURITY OVERVIEW EN PROFUNDIDAD {#security-overview-profundo}

## 📊 Capabilities de Security Overview

### Vistas Disponibles

```yaml
1. SECURITY RISK
   Propósito: Identificar repos de alto riesgo
   
   Ordena por:
     - Número de alertas críticas
     - Número de alertas high
     - Age of oldest alert
     - Combined risk score
   
   Usa para:
     - Priorizar remediation efforts
     - Identificar hotspots
     - Executive dashboards

2. SECURITY COVERAGE
   Propósito: Adoption tracking
   
   Muestra:
     - % repos con GHAS enabled
     - % repos con cada feature
     - Gaps en coverage
   
   Usa para:
     - Rollout planning
     - Compliance reporting
     - Identify laggards

3. SECURITY ALERTS
   Propósito: Detalle de todas las alertas
   
   Vista:
     - Todas las alertas de la org
     - Filtrable por tipo, severity, estado
     - Bulk actions disponibles
   
   Usa para:
     - Daily triage
     - Bulk dismiss
     - Trend analysis

4. SECURITY CAMPAIGNS (Enterprise)
   Propósito: Coordinated remediation
   
   Permite:
     - Agrupar alertas similares
     - Assign to teams
     - Track progress
     - Set deadlines
   
   Usa para:
     - Log4Shell-type incidents
     - Tech debt sprints
     - Compliance deadlines
```

---

## 📈 Métricas Clave

### Métricas de Cobertura (Coverage)

```yaml
1. GHAS Adoption Rate
   Formula: (Repos con GHAS / Total repos) × 100
   
   Benchmark:
     - Excelente: >90%
     - Bueno: 70-90%
     - Necesita mejora: <70%
   
   Acciones si bajo:
     - Identificar repos sin GHAS
     - Priorizar por criticidad
     - Auto-enable para nuevos

2. Feature Adoption por Tipo
   Métricas individuales:
     - % con Secret scanning: Target 100%
     - % con Code scanning: Target 100%
     - % con Dependabot: Target 100%
     - % con Push protection: Target 90%+
   
   Segmentación:
     - Por equipo
     - Por lenguaje
     - Por criticidad

3. Configuration Quality
   Indicadores:
     - % repos con branch protection
     - % repos con required status checks
     - % repos con custom CodeQL config
   
   Quality Score:
     - Gold: Todas las features + custom config
     - Silver: Todas las features + basics
     - Bronze: Features básicas
     - None: Sin GHAS
```

### Métricas de Riesgo (Risk)

```yaml
1. Open Alerts por Severidad
   
   Critical:
     Total: 5
     Avg age: 3 días
     Oldest: 10 días ⚠️
     SLA compliance: 80% (target: 100%)
   
   High:
     Total: 23
     Avg age: 12 días
     Oldest: 45 días ⚠️
     SLA compliance: 65% (target: 90%)
   
   Medium:
     Total: 156
     Avg age: 34 días
     
   Low:
     Total: 412
     (tracked but not prioritized)

2. Alert Age Distribution
   
   Buckets:
     0-7 días: 45 alerts ✅ (fresh)
     8-30 días: 123 alerts ⚠️ (aging)
     31-90 días: 67 alerts 🔴 (stale)
     90+ días: 12 alerts 🔴🔴 (critical backlog)
   
   Action thresholds:
     - Any alert >90 días: Executive escalation
     - Critical >7 días: Manager review
     - High >30 días: Team retrospective

3. Vulnerability Density
   
   Formula: Alertas / 1000 LOC
   
   Benchmarks:
     - Excellent: <0.5
     - Good: 0.5-2.0
     - Needs improvement: >2.0
   
   Segmentación:
     - Por repositorio
     - Por equipo
     - Por lenguaje
```

### Métricas de Remediación (Remediation)

```yaml
1. Mean Time To Resolve (MTTR)
   
   Por severidad:
     Critical: 2.3 días (target: <1 día)
     High: 8.5 días (target: <7 días)
     Medium: 25 días (target: <30 días)
     Low: 65 días (target: <90 días)
   
   Trend: ↓ -15% vs last month ✅

2. Fix Rate
   
   Formula: (Alerts fixed / Total alerts) × 100
   
   Current month: 78% ✅
   Last month: 72%
   Trend: ↑ improving
   
   Breakdown:
     - Dependabot auto-fixed: 45%
     - Developer fixed: 33%
     - Dismissed (valid): 15%
     - Still open: 7%

3. Recurrence Rate
   
   Formula: (Re-opened alerts / Total fixed) × 100
   
   Current: 3% ✅ (target: <5%)
   
   Causes:
     - Dependency re-downgrade: 60%
     - Code regression: 30%
     - False dismissal: 10%
   
   Prevention:
     - Lock dependency versions
     - Regression tests
     - Better triage training

4. Security Debt Trend
   
   Total alerts over time:
     Jan: 450
     Feb: 423 ↓
     Mar: 389 ↓
     Apr: 412 ↑ ⚠️
     
   Analysis:
     - ↑ en Abril debido a nueva CVE wave
     - Underlying trend: improving
     - Velocity: -15 alerts/month average
```

---

## 🎯 Uso Práctico de Security Overview

### Dashboard para Executives (C-level)

```yaml
Report mensual debe incluir:

1. EXECUTIVE SUMMARY
   "Nuestra postura de seguridad mejoró 12% este mes"
   
   Key Metrics:
     - Total critical/high alerts: 28 (↓15%)
     - MTTR críticas: 2.3 días (↓0.7 días)
     - GHAS coverage: 94% (↑4%)
     - Security debt: $150k (↓$25k)
   
   Status: 🟢 On track

2. RISK HEAT MAP
   
   Repos de Alto Riesgo:
     🔴 payment-service: 5 critical, 12 high
     🔴 user-api: 3 critical, 8 high
     🟡 frontend-app: 0 critical, 15 high
   
   Action: CTO review requerido

3. COMPLIANCE STATUS
   
   PCI-DSS:
     ✅ 100% de payment repos tienen GHAS
     ✅ All critical alerts <7 días old
     ⚠️ 2 repos sin code scanning
   
   SOC2:
     ✅ Security policies enforced
     ✅ Audit trail completo
   
   Overall: 🟢 Compliant

4. COST AVOIDANCE
   
   Vulnerabilidades prevenidas: 45
   Estimated breach cost avoided: $2.3M
   GHAS investment: $150k/year
   ROI: 15x ✅
```

### Dashboard para Security Team

```yaml
Daily Dashboard:

1. TRIAGE QUEUE
   
   New alerts (últimas 24h):
     Critical: 2 🔴
       - CVE-2024-1234 en payment-service
       - Secret leak en user-api
     High: 5 🟡
     Medium: 12
   
   Action items:
     [Assign] CVE-2024-1234 → @security-team
     [Create incident] Secret leak

2. SLA TRACKING
   
   At risk (próximas a violar SLA):
     - payment-service: critical alert día 6/7 ⚠️
     - api-gateway: high alert día 28/30 ⚠️
   
   Action: Ping responsible teams

3. CAMPAIGN PROGRESS
   
   "Log4j Remediation":
     Target: 45 repos
     Completed: 38 (84%) ✅
     In progress: 5
     Blocked: 2
     Deadline: 3 días
   
   Status: On track

4. TRENDS
   
   Week-over-week:
     New alerts: +8 (↑)
     Fixed alerts: +15 (↑↑)
     Net change: -7 (↓) ✅
   
   Emerging patterns:
     - ↑ SQL injection findings (new CodeQL query)
     - ↑ npm dependencies vulnerabilities
```

### Dashboard para Engineering Managers

```yaml
Team Performance Dashboard:

1. TEAM SECURITY SCORE
   
   Frontend Team:
     Alerts per 1000 LOC: 1.2 🟡
     MTTR: 9 días 🟡
     Coverage: 100% ✅
     Grade: B+
   
   Backend Team:
     Alerts per 1000 LOC: 0.6 ✅
     MTTR: 5 días ✅
     Coverage: 100% ✅
     Grade: A

2. SPRINT HEALTH
   
   Current sprint:
     Security stories: 3/5 completed
     Dependabot PRs: 8/12 merged
     Alerts opened: 4
     Alerts fixed: 7
     Net: -3 ✅

3. BLOCKERS
   
   PRs blocked by security checks: 2
     - PR #456: High severity CodeQL
     - PR #789: New vulnerable dependency
   
   Action needed: Dev review

4. TRAINING NEEDS
   
   Based on alert patterns:
     - SQL injection: 5 occurrences
       → Recommend: OWASP training
     - Hardcoded secrets: 3 occurrences
       → Recommend: Secret management workshop
```

---

## 🔍 Filtros Avanzados en Security Overview

### Construcción de Queries Complejas

```yaml
Ejemplo 1: "Todas las alertas críticas en repos de producción"

Filtros:
  is:open
  severity:critical
  archived:false
  topic:production

Resultado:
  12 alerts across 5 repositories
  
Acción:
  [Bulk assign] → @security-oncall


Ejemplo 2: "Dependencias vulnerables en Node.js con patches disponibles"

Filtros:
  is:open
  ecosystem:npm
  has:patch
  severity:high,critical

Resultado:
  23 alerts
  
Acción:
  [Security campaign] "Node.js March Updates"
  Deadline: End of week


Ejemplo 3: "CodeQL alerts sin revisar en PRs activos"

Filtros:
  is:open
  tool:CodeQL
  state:unreviewed
  pr:open

Resultado:
  8 alerts
  
Acción:
  [Notify] PR authors


Ejemplo 4: "Secretos activos de AWS expuestos"

Filtros:
  is:open
  tool:secret-scanning
  secret-type:aws_access_key_id
  validity:active

Resultado:
  3 alerts 🚨
  
Acción:
  [IMMEDIATE] Revoke via AWS Console
  [Create incident] INC-2024-045
```

### Saved Filters (Best Practices)

```yaml
Pre-configurar filtros comunes:

1. "Daily Triage"
   is:open
   created:>$(date -7days)
   severity:critical,high
   
   Úsalo: Cada mañana

2. "SLA at Risk"
   is:open
   severity:critical
   created:<$(date -6days)
   
   Úsalo: Monitoring diario

3. "Team: Backend"
   is:open
   team:@org/backend
   
   Úsalo: Team standups

4. "Compliance Report"
   repository:payment-*
   archived:false
   
   Úsalo: Quarterly audits

5. "Quick Wins"
   is:open
   severity:medium,low
   has:patch
   
   Úsalo: Downtime/learning time
```

---

## 📥 Exportar y Reportar

### CSV Export

```yaml
Uso:
  Security Overview → [Export] → CSV

Contiene:
  - Alert ID
  - Repository
  - Severity
  - CWE / CVE
  - Created date
  - Age (días)
  - Status
  - Assigned to
  
Úsalo para:
  - Executive reports
  - Trend analysis en Excel
  - Integration con BI tools
  - Compliance documentation
```

### API para Automatización

```bash
# Get all critical alerts
gh api graphql -f query='
  query($org: String!) {
    organization(login: $org) {
      repositories(first: 100) {
        nodes {
          name
          vulnerabilityAlerts(first: 10, states: OPEN) {
            nodes {
              securityAdvisory {
                severity
                description
                publishedAt
              }
              vulnerableManifestPath
            }
          }
        }
      }
    }
  }
' -f org=my-org

# Create weekly report
curl -H "Authorization: token $GITHUB_TOKEN" \
     https://api.github.com/orgs/my-org/code-scanning/alerts \
  | jq '[.[] | select(.state=="open") | {
      repo: .repository.name,
      severity: .rule.security_severity_level,
      age: (now - (.created_at | fromdateiso8601)) / 86400 | floor
    }] | group_by(.severity) | map({
      severity: .[0].severity,
      count: length,
      avg_age: (map(.age) | add / length)
    })'
```

---

# COMPARATIVAS CRÍTICAS {#comparativas-criticas}

## 🔄 Dependabot: Alerts vs Security Updates vs Version Updates

### Tabla Maestra

| Aspecto | Dependabot Alerts | Dependabot Security Updates | Dependabot Version Updates |
|---------|------------------|----------------------------|---------------------------|
| **Propósito** | Detectar vulnerabilidades | Auto-fix vulnerabilidades | Mantener deps actualizadas |
| **Qué detecta** | Solo vulnerabilidades (CVEs) | Solo vulnerabilidades | Todas las actualizaciones |
| **Cuándo actúa** | Al detectar vuln | Después de alert | Según schedule |
| **Acción** | Crea alerta | Crea PR de fix | Crea PR de update |
| **Automático** | ✅ Sí | ✅ Sí | ✅ Sí |
| **Requiere config** | ❌ No | ❌ No | ✅ Sí (dependabot.yml) |
| **Gratis público** | ✅ | ✅ | ✅ |
| **Gratis privado** | ✅ | ✅ | ✅ |
| **Requiere GHAS** | ❌ No (desde 2022) | ❌ No | ❌ No |
| **Frequency** | Continuo | Al crear alert | Configurable |
| **Scope** | Solo vulnerable | Solo vulnerable | Todas |
| **PR creados** | 0 | 1 por vuln | Muchos (según config) |
| **Severidad** | Todas | Solo con patch | N/A |

### Casos de Uso

```yaml
Usa Dependabot ALERTS cuando:
  ✓ Quieres visibilidad de vulnerabilidades
  ✓ Quieres monitoreo continuo
  ✓ No estás listo para PRs automáticos

Usa Dependabot SECURITY UPDATES cuando:
  ✓ Quieres auto-remediación
  ✓ Confías en semantic versioning
  ✓ Tienes buenos tests
  ✓ Quieres reducir MTTR

Usa Dependabot VERSION UPDATES cuando:
  ✓ Quieres prevenir tech debt
  ✓ Quieres latest features
  ✓ Quieres mantener deps current
  ✓ NO solo para seguridad
```

### Flow Diagram

```
┌──────────────────────────────────────────────┐
│ Nueva CVE publicada para lodash@4.17.15      │
└────────────────┬─────────────────────────────┘
                 │
┌────────────────▼─────────────────────────────┐
│ DEPENDABOT ALERTS                             │
│ ✅ Detecta que tu repo usa lodash@4.17.15    │
│ ✅ Crea alerta en Security tab                │
│ ✅ Notifica a admins                          │
└────────────────┬─────────────────────────────┘
                 │
                 ├─ ¿Security Updates habilitado?
                 │
                 ├─ NO → Alert permanece
                 │        Developer debe actuar manualmente
                 │
                 └─ SÍ ↓
┌──────────────────────────────────────────────┐
│ DEPENDABOT SECURITY UPDATES                   │
│ ✅ Verifica que lodash@4.17.21 existe        │
│ ✅ Verifica compatibilidad (semver)          │
│ ✅ Crea branch: dependabot/npm_and_yarn/...  │
│ ✅ Actualiza package.json + lockfile         │
│ ✅ Abre PR con changelog                     │
└────────────────┬─────────────────────────────┘
                 │
                 ├─ Developer review
                 ├─ Tests run (CI)
                 ├─ Merge PR
                 │
                 └─ Alert auto-cerrada ✅
                 
┌──────────────────────────────────────────────┐
│ DEPENDABOT VERSION UPDATES (separado)        │
│                                              │
│ Según schedule (ej: weekly):                 │
│ ✅ Check lodash current: 4.17.21            │
│ ✅ Check lodash latest: 5.0.0               │
│ ✅ Crea PR para upgrade a 5.0.0             │
│ ⚠️  MAJOR version - requiere review         │
└──────────────────────────────────────────────┘
```

---

## 🔐 Secret Scanning vs Push Protection

### Comparativa Fundamental

| Aspecto | Secret Scanning | Push Protection |
|---------|----------------|-----------------|
| **Momento** | Después del commit | Antes del commit |
| **Acción** | Detecta y alerta | Bloquea push |
| **Scope** | Todo el historial | Nuevos commits |
| **Retroactivo** | ✅ Sí | ❌ No |
| **Preventivo** | ❌ No | ✅ Sí |
| **Puede ser bypasseado** | N/A | ✅ Sí (con justificación) |
| **Gratis público** | ✅ | ✅ |
| **Gratis privado** | ❌ (requiere GHAS) | ❌ (requiere GHAS) |

### Timeline Comparison

```
WITHOUT Push Protection:
  Developer escribe código con secret
    ↓
  git commit
    ✅ Success
    ↓
  git push
    ✅ Success (secret ahora en GitHub)
    ↓
  [30 segundos después]
  Secret Scanning detecta
    ↓
  Alert creada
  Notificación enviada
    ↓
  ⚠️ PERO: Secret YA está expuesto en historial
  ⚠️ Bots ya lo escanearon
  ⚠️ Daño potencial ya ocurrió

WITH Push Protection:
  Developer escribe código con secret
    ↓
  git commit
    ✅ Success (local)
    ↓
  git push
    ❌ BLOCKED!
    
    remote: ❌ Push protection: Secret detected
    remote: 
    remote: GitHub Personal Access Token found
    remote:   Location: src/config.js:15
    remote:   Type: github_pat
    remote:   Status: Active
    remote: 
    remote: To push anyway:
    remote:   1. Revoke the secret
    remote:   2. Remove from code
    remote:   3. Request bypass (justification required)
    
    ↓
  Developer DEBE actuar ANTES de push
    ↓
  ✅ Secret nunca llega a GitHub
  ✅ No hay exposición
  ✅ No hay alertas
```

### Configuración Recomendada

```yaml
✅ HABILITAR AMBOS:

Secret Scanning:
  Propósito: Safety net
  Detecta: Secretos históricos + nuevos
  Casos:
    - Migración de repos existentes
    - Secretos en branches viejos
    - Bypasses de push protection

Push Protection:
  Propósito: Prevención
  Bloquea: Nuevos commits
  Casos:
    - Prevenir exposición inicial
    - Educar developers
    - Cumplir políticas

Juntos:
  Defense in depth ✅
  Push protection previene
  Secret scanning detecta si algo pasa
```

---

## 📊 CodeQL: Default Setup vs Advanced Setup

### Comparación Completa

| Aspecto | Default Setup | Advanced Setup |
|---------|--------------|----------------|
| **Configuración** | 1-click | Workflow manual |
| **Tiempo setup** | 30 segundos | 10-30 minutos |
| **Archivo requerido** | Ninguno | `.github/workflows/codeql.yml` |
| **Customización** | Limitada | Completa |
| **Query suites** | `default` (fixed) | Cualquiera |
| **Custom queries** | ❌ No | ✅ Sí |
| **Build control** | Automático | Manual posible |
| **Scheduled scans** | Weekly (fixed) | Configurable |
| **Languages** | Auto-detect | Especificar manual |
| **Paths filters** | ❌ No | ✅ Sí |
| **Config file** | ❌ No | ✅ Sí (codeql-config.yml) |
| **Runners** | GitHub-hosted | GitHub o self-hosted |
| **Recomendado para** | 80% de repos | Power users, custom needs |

### Cuándo Usar Cada Uno

```yaml
Usa DEFAULT SETUP cuando:
  ✓ Repo nuevo o standard
  ✓ No necesitas customización
  ✓ Lenguajes estándar (JS, Python, Ruby, Go)
  ✓ Quieres empezar rápido
  ✓ No tienes expertise en CodeQL
  ✓ Codebase <100k LOC
  ✓ Default queries son suficientes

Usa ADVANCED SETUP cuando:
  ✓ Necesitas custom queries
  ✓ Lenguajes compilados con build complejo
  ✓ Quieres control de schedule
  ✓ Necesitas filtrar paths
  ✓ Quieres múltiples query suites
  ✓ Self-hosted runners
  ✓ Monorepo con múltiples languages
  ✓ Compliance requiere specific queries
  ✓ Integración con CI/CD externo
```

### Migration Path

```yaml
Migrar de Default a Advanced:

1. Export current config:
   Settings → Code security → Code scanning
   → View CodeQL default setup
   → [Generate advanced setup file]

2. Commit workflow:
   .github/workflows/codeql.yml creado
   
3. Customize:
   - Add custom queries
   - Configure schedule
   - Add path filters
   - Etc.

4. Default setup auto-disabled
   (no conflict)

⚠️ NO puedes tener ambos simultáneamente
```

---

## 🔍 Dependency Review vs Dependabot Alerts

### Diferencia Fundamental

```yaml
DEPENDABOT ALERTS:
  └─ Reactivo
      └─ "Ya tienes este problema"
          └─ Aparece en Security tab
              └─ Después del merge

DEPENDENCY REVIEW:
  └─ Proactivo
      └─ "Esto crearía un problema"
          └─ Aparece en PR checks
              └─ Antes del merge
```

### Tabla Comparativa

| Aspecto | Dependabot Alerts | Dependency Review |
|---------|------------------|-------------------|
| **Scope** | Repo completo | Solo cambios en PR |
| **Momento** | Post-merge | Pre-merge |
| **Objetivo** | Detectar existente | Prevenir nuevos |
| **Ubicación** | Security tab | PR checks |
| **Bloquea merge** | ❌ No | ✅ Sí (configurable) |
| **Implementación** | Automático | GitHub Action required |
| **Requiere GHAS** | ❌ No | ✅ Sí |
| **Verifica** | Vulnerabilidades | Vulnerabilidades + licencias + score |
| **PRs generados** | Security update PRs | Ninguno (solo check) |

### Flow Completo

```
Escenario: Developer actualiza axios de 0.21.0 a 0.21.4

┌─────────────────────────────────────────────┐
│ Developer modifica package.json              │
│   axios: "0.21.0" → "0.21.4"                │
└────────────┬────────────────────────────────┘
             │
┌────────────▼────────────────────────────────┐
│ git commit & push                            │
└────────────┬────────────────────────────────┘
             │
┌────────────▼────────────────────────────────┐
│ Abre PR a main                               │
└────────────┬────────────────────────────────┘
             │
             ├─ DEPENDENCY REVIEW ejecuta
             │
┌────────────▼────────────────────────────────┐
│ Dependency Review Action                     │
│ Compara:                                     │
│   Base (main): axios@0.21.0                 │
│   Head (PR):   axios@0.21.4                 │
│                                              │
│ Resultado:                                   │
│   ✅ Actualización de seguridad             │
│   ✅ Resuelve CVE-2021-3749                 │
│   ✅ No introduce nuevas vulns              │
│   ✅ Licencia compatible                    │
│                                              │
│ Check: ✅ PASS                               │
└────────────┬────────────────────────────────┘
             │
┌────────────▼────────────────────────────────┐
│ Merge permitido                              │
└────────────┬────────────────────────────────┘
             │
┌────────────▼────────────────────────────────┐
│ Post-merge:                                  │
│ Dependabot alert para axios@0.21.0          │
│ automáticamente se cierra                    │
│ (vulnerabilidad resuelta)                    │
└─────────────────────────────────────────────┘


Escenario 2: Developer downgrade vulnerable

┌─────────────────────────────────────────────┐
│ Developer modifica package.json              │
│   axios: "0.27.0" → "0.21.0" (downgrade!)   │
└────────────┬────────────────────────────────┘
             │
             [... git commit, push, PR ...]
             │
┌────────────▼────────────────────────────────┐
│ Dependency Review Action                     │
│ Compara:                                     │
│   Base (main): axios@0.27.0 (seguro)        │
│   Head (PR):   axios@0.21.0 (vulnerable)    │
│                                              │
│ Resultado:                                   │
│   ❌ Introduce CVE-2021-3749                │
│   ❌ Severity: High                          │
│                                              │
│ Check: ❌ FAIL                               │
└────────────┬────────────────────────────────┘
             │
┌────────────▼────────────────────────────────┐
│ PR BLOQUEADO                                 │
│ No puede mergear hasta:                      │
│   1. Actualizar axios a versión segura      │
│   2. O override check (si permitido)        │
└─────────────────────────────────────────────┘
```

### Configuración Complementaria

```yaml
# Ambos habilitados = Defense in depth

1. Dependabot Alerts (siempre activo):
   └─ Detecta vulnerabilidades en main
   └─ Genera security update PRs

2. Dependency Review (en PRs):
   └─ Bloquea nuevas vulnerabilidades
   └─ Previene regresiones

Resultado:
  ✅ Nada vulnerable llega a main
  ✅ Lo que ya está se detecta y arregla
  ✅ Coverage 360°
```

---

## 🛡️ Code Scanning Tools: CodeQL vs Third-party

### Comparación Estratégica

| Aspecto | CodeQL | Third-party (Snyk, Sonar, etc) |
|---------|--------|-------------------------------|
| **Provider** | GitHub (gratis con GHAS) | Vendor externo (costo adicional) |
| **Integration** | Nativa (1-click) | SARIF upload |
| **Languages** | 15+ | Varía por tool |
| **Customization** | Open source queries | Depende del vendor |
| **Database** | GitHub Advisory | Vendor DB + público |
| **False positives** | Bajo-medio | Varía |
| **Data flow analysis** | ✅ Excelente | Varía |
| **Taint tracking** | ✅ Sí | Algunos |
| **Supply chain** | ❌ (usa Dependabot) | Algunos tienen |
| **License** | Gratis con GHAS | Licensing separado |
| **Support** | GitHub Support | Vendor support |

### Cuándo Usar Qué

```yaml
Usa SOLO CodeQL cuando:
  ✓ Tienes GHAS license
  ✓ Languages soportados por CodeQL
  ✓ No necesitas features específicas de vendors
  ✓ Quieres simplificar tooling
  ✓ Budget limitado

Usa CodeQL + Third-party cuando:
  ✓ Defense in depth
  ✓ Compliance requiere múltiples tools
  ✓ Lenguajes no soportados por CodeQL
  ✓ Features específicas (ej: Snyk container scanning)
  ✓ Ya tienes investment en tool

Usa SOLO Third-party cuando:
  ✓ No tienes GHAS
  ✓ Ya tienes contrato enterprise con vendor
  ✓ Tool específico es requirement (ej: SonarQube enterprise)
```

---

# TRAMPAS COMUNES DEL EXAMEN {#exam-traps}

## 🎯 Top 20 Trampas que Reprueba a Candidatos

### Trampa #1: Dependabot Auto-merge

```yaml
❌ INCORRECTO:
"Dependabot automáticamente mergea los PRs de seguridad"

✅ CORRECTO:
Dependabot CREA PRs, pero NO auto-mergea por default.
Requieres configurar auto-merge explícitamente via:
  - GitHub Actions workflow
  - Branch protection auto-merge setting
  - Third-party automation

Por qué es trampa:
  - Mucha gente asume "automated" = auto-merge
  - El nombre "security UPDATES" suena automático
  - Necesitas leer la documentación con cuidado
```

### Trampa #2: CodeQL en Lenguajes Compilados

```yaml
❌ INCORRECTO:
"CodeQL funciona sin build en Java"

✅ CORRECTO:
Java, C++, C#, Go requieren BUILD completo.
JavaScript, Python, Ruby NO requieren build.

Detalles:
  Compilados:
    - CodeQL intercepta compilador
    - Requiere todas las dependencias
    - Autobuild puede fallar
    - Manual build común

  Interpretados:
    - Parse directo del source
    - No requiere build
    - Más rápido
    - Type inference limitado

Por qué es trampa:
  - Fácil confundir con otros SAST tools
  - Autobuild existe pero no siempre funciona
  - Workflow falla y no sabes por qué
```

### Trampa #3: Secret Scanning Coverage

```yaml
❌ INCORRECTO:
"Secret scanning detecta TODOS los secretos"

✅ CORRECTO:
Solo detecta:
  - ~200 patterns predefinidos (GitHub + partners)
  - Custom patterns que TÚ defines
  - NO detecta secretos de formatos desconocidos
  - NO detecta secretos ofuscados

Ejemplos de NO DETECTA:
  ❌ password = base64_encode("mysecret")
  ❌ API_KEY = "custom-" + generateRandomString()
  ❌ secrets en archivos binarios/imágenes
  ❌ secretos en formatos propietarios sin pattern

Por qué es trampa:
  - Nombre "secret SCANNING" suena comprehensivo
  - La gente asume AI/ML detecta todo
  - Genera falsa sensación de seguridad
```

### Trampa #4: Push Protection Scope

```yaml
❌ INCORRECTO:
"Push protection previene ALL secrets en el repo"

✅ CORRECTO:
Push protection solo bloquea NUEVOS commits.
NO bloquea:
  - Secretos en historial existente
  - Secretos en branches ya pusheados
  - Secretos en repos forkeados
  - Secretos antes de habilitar la feature

Flow:
  1. Habilitas push protection HOY
  2. Secretos de AYER no son bloqueados
  3. Solo NUEVOS pushes son verificados

Por qué es trampa:
  - "Protection" suena total
  - No es retroactivo
  - Complementa a secret scanning, no reemplaza
```

### Trampa #5: Triage Role

```yaml
❌ INCORRECTO:
"Triage role puede ver code scanning alerts"

✅ CORRECTO:
Triage NO puede ver:
  - Code scanning alerts
  - Secret scanning alerts

Triage SÍ puede ver:
  - Dependabot alerts (excepción!)
  - Código del repo
  - Issues y PRs

Jerarquía de visibilidad:
  Code/Secret Scanning:
    - Admin: ✅
    - Maintain: ❌
    - Write: ❌
    - Triage: ❌
    - Read: ❌

  Dependabot:
    - Admin: ✅
    - Maintain: ✅
    - Write: ✅
    - Triage: ✅
    - Read: ✅

Por qué es trampa:
  - Inconsistencia entre features
  - "Triage" suena como security role
  - Fácil asumir que puede ver todo security
```

### Trampa #6: Organization vs Repository Settings

```yaml
❌ INCORRECTO:
"Organization settings siempre overridean repository settings"

✅ CORRECTO:
Depende de si hay ENFORCEMENT:

Sin Enterprise Policy:
  Org settings = default
  Repo puede override ✅

Con Enterprise Enforcement:
  Enterprise mandatorio
  Org NO puede cambiar
  Repo NO puede cambiar

Hierarchy:
  Enterprise (enforced)
    ↓ overrides
  Organization (default)
    ↓ can be overridden by
  Repository (specific)

Ejemplo:
  Enterprise: No enforcement
  Org: Secret scanning enabled for all
  Repo Admin: Puede deshabilitar ✅

  Enterprise: Enforced
  Org: No choice, must enable
  Repo Admin: Cannot disable ❌

Por qué es trampa:
  - Jerarquía no es obvia
  - Enforcement vs default confunde
  - Varía entre GHEC y sin Enterprise
```

### Trampa #7: GHAS License Scope

```yaml
❌ INCORRECTO:
"GHAS license cubre unlimited repos"

✅ CORRECTO:
GHAS se cobra por ACTIVE COMMITTER, no por repo.

Active committer = alguien que:
  - Hizo commit a un repo con GHAS habilitado
  - En los últimos 90 días
  - En cualquier branch

Pricing:
  - ~$49/active committer/mes (legacy)
  - Desde 2025:
    - Secret Protection: $19/committer
    - Code Security: $30/committer

Ejemplos:
  100 repos, 10 committers = $490/mes (legacy)
  100 repos, 100 committers = $4,900/mes

Por qué es trampa:
  - Mucha gente asume "per repo"
  - "Advanced Security" suena como flat fee
  - Costos escalan con team size
```

### Trampa #8: Dependabot Configuration File

```yaml
❌ INCORRECTO:
"Necesitas dependabot.yml para Dependabot alerts"

✅ CORRECTO:

dependabot.yml NO ES REQUERIDO para:
  ✅ Dependabot alerts
  ✅ Dependabot security updates

dependabot.yml SÍ ES REQUERIDO para:
  ✅ Dependabot VERSION updates
  ✅ Scheduled updates
  ✅ Grouping
  ✅ Customización de PRs

Sin dependabot.yml:
  - Alerts: ✅ Funcionan
  - Security updates: ✅ Funcionan
  - Version updates: ❌ No funcionan

Con dependabot.yml:
  - Todo lo anterior: ✅
  - Plus: schedule, groups, labels, etc.

Por qué es trampa:
  - Confusión entre features
  - Mucha gente crea yml innecesariamente
  - O asume que sin yml no hay Dependabot
```

### Trampa #9: CodeQL Query Suites

```yaml
❌ INCORRECTO:
"security-extended incluye todas las queries de seguridad"

✅ CORRECTO:

Query suites hierarchy:
  security
    ↓ subset de
  security-extended
    ↓ subset de
  security-and-quality

Queries count (approx):
  security: ~100
  security-extended: ~200
  security-and-quality: ~300

security-extended NO incluye:
  ❌ Quality queries
  ❌ Experimental queries
  ❌ Custom queries

Para TODO:
  queries: +security-and-quality

Por qué es trampa:
  - "extended" suena como "todo"
  - Naming confunde
  - Cada suite tiene propósito específico
```

### Trampa #10: Dependency Graph Availability

```yaml
❌ INCORRECTO:
"Dependency graph requiere GHAS license"

✅ CORRECTO:

Dependency graph es GRATIS para:
  ✅ Repos públicos
  ✅ Repos privados

NO requiere GHAS.
Disponible para todos.

Requiere GHAS:
  - Secret scanning (privados)
  - Code scanning
  - Dependency REVIEW (no graph)

Gratis siempre:
  - Dependency graph
  - Dependabot alerts
  - Security advisories

Por qué es trampa:
  - Confusión con otros features
  - "Advanced" suena como paid
  - Es base para Dependabot (también gratis)
```

### Trampa #11: Secret Validity Check

```yaml
❌ INCORRECTO:
"GitHub revoca automáticamente secretos detectados"

✅ CORRECTO:

GitHub:
  ✅ Detecta secreto
  ✅ Verifica validez (algunos providers)
  ✅ Notifica a usuario
  ✅ Notifica a provider (algunos)
  ❌ NO revoca automáticamente

Partner actions:
  GitHub tokens: Revocados por GitHub ✅
  AWS keys: AWS puede revocar (su decisión)
  Otros: Depende del partner

Tu responsabilidad:
  ✅ Revisar alerta
  ✅ Revocar secreto
  ✅ Rotar credenciales
  ✅ Limpiar historial

Por qué es trampa:
  - "Notify provider" suena como revoke
  - GitHub tokens son caso especial
  - Mucha gente asume auto-remediation
```

### Trampa #12: CodeQL Language Support

```yaml
❌ INCORRECTO:
"CodeQL soporta PHP y Scala"

✅ CORRECTO (2026):

Fully supported:
  ✅ JavaScript/TypeScript
  ✅ Python
  ✅ Java/Kotlin
  ✅ C/C++
  ✅ C#
  ✅ Go
  ✅ Ruby
  ✅ Swift

Beta/Experimental:
  ⚠️ Rust (beta)

NOT supported:
  ❌ PHP
  ❌ Scala
  ❌ Perl
  ❌ Lua
  ❌ R

Workarounds:
  - Use third-party SAST
  - Upload SARIF results
  - Community queries (experimental)

Por qué es trampa:
  - Lenguajes populares no soportados
  - Beta != production ready
  - Fácil asumir "todos los lenguajes"
```

### Trampa #13: SARIF Upload Limit

```yaml
❌ INCORRECTO:
"Puedo subir SARIF de tamaño ilimitado"

✅ CORRECTO:

Límites SARIF:
  - File size: 10 MB (compressed)
  - Results per file: 25,000 results
  - Files per analysis: 20 files

Si excedes:
  ❌ Upload falla
  
Workarounds:
  - Split results en múltiples files
  - Filter low-severity alerts
  - Compress SARIF (gzip)
  - Upload por categoría

Ejemplo:
  CodeQL + Snyk + SonarQube = 3 SARIFs
  Cada uno <10MB
  Total <20 files

Por qué es trampa:
  - Límites no obvios
  - Repos grandes pueden exceder
  - Error solo aparece en upload
```

### Trampa #14: Security Manager Permissions

```yaml
❌ INCORRECTO:
"Security Manager puede modificar código"

✅ CORRECTO:

Security Manager PUEDE:
  ✅ Ver security alerts (todos los repos)
  ✅ Dismiss/reopen alerts
  ✅ Configure security features
  ✅ Access Security Overview
  ✅ Manage security campaigns
  ✅ Create custom patterns

Security Manager NO PUEDE:
  ❌ Ver código (sin role adicional)
  ❌ Modificar código
  ❌ Merge PRs
  ❌ Delete repos
  ❌ Manage billing
  ❌ Add/remove org members

Para ver + modificar código:
  Necesitas: Security Manager + Repo role (Write/Admin)

Por qué es trampa:
  - "Manager" suena con muchos permisos
  - Es security-scoped only
  - Separation of duties
```

### Trampa #15: Dependabot PR Limits

```yaml
❌ INCORRECTO:
"Dependabot crea PRs ilimitados"

✅ CORRECTO:

Default limits:
  - Version updates: 5 PRs abiertos
  - Security updates: Ilimitados

Configurable:
  open-pull-requests-limit: 10

Por ecosistema:
  - npm: limit aplica
  - pip: limit aplica
  - docker: limit aplica
  - etc.

Comportamiento:
  - Si hay 5 PRs abiertos
  - Dependabot NO crea más
  - Hasta que mergees o cierres

Override:
  # dependabot.yml
  open-pull-requests-limit: 0  # Disable version updates
  open-pull-requests-limit: 20 # Increase limit

Por qué es trampa:
  - "Automated" suena como unlimited
  - Confunde cuando PRs dejan de aparecer
  - Security updates no tienen límite (diferente)
```

### Trampa #16: Branch Protection Bypass

```yaml
❌ INCORRECTO:
"Admins pueden siempre bypassear branch protection"

✅ CORRECTO:

Depende de configuración:

Branch protection rule:
  [✓] Do not allow bypassing the above settings
    → NADIE puede bypass (ni admins)
  
  [ ] Do not allow bypassing...
    → Admins pueden bypass

Repository settings:
  [ ] Allow repository administrators to bypass
    → Explicit setting

Enterprise enforcement:
  Si enforced: NO bypass
  Si no enforced: Según repo settings

Rulesets (nuevo):
  Bypass permissions granulares
    - Organization admins
    - Security team
    - Specific users
  Con/sin approval workflow

Por qué es trampa:
  - Admin != God mode
  - "Do not allow bypassing" aplica a TODOS
  - Rulesets cambian el modelo
```

### Trampa #17: CodeQL Database Retention

```yaml
❌ INCORRECTO:
"CodeQL databases se guardan permanentemente"

✅ CORRECTO:

Retention:
  - GitHub-hosted: 30 días
  - Después: Database deleted
  - SARIF results: Permanentes

Implicaciones:
  - No puedes re-run queries después de 30 días
  - Need to re-build database
  - Results persisten

Para análisis histórico:
  ✅ Download database
  ✅ Store in blob storage
  ✅ Re-upload cuando necesites

Workflow:
  - name: Upload CodeQL DB
    uses: actions/upload-artifact@v4
    with:
      name: codeql-database
      path: codeql-db

Por qué es trampa:
  - "Database" suena permanente
  - Results vs database confunde
  - 30 días pasa rápido
```

### Trampa #18: Custom Pattern Scope

```yaml
❌ INCORRECTO:
"Custom patterns se pueden crear a nivel repository"

✅ CORRECTO:

Custom patterns solo a nivel:
  ✅ Organization
  ❌ Repository
  ❌ Enterprise

Scope:
  - Crear pattern en Org settings
  - Aplica a TODOS los repos de la org
  - No hay patterns por repo

Implicaciones:
  - Patterns deben ser generales
  - No puedes tener patterns repo-specific
  - Cuidado con false positives

Workaround:
  - Path exclusions en pattern
  - Test exhaustivamente antes de org-wide

Por qué es trampa:
  - Code scanning tiene repo config
  - Intuitivo pensar que secrets también
  - Limitación de arquitectura
```

### Trampa #19: Dependency Review License Checks

```yaml
❌ INCORRECTO:
"Dependency review solo verifica vulnerabilidades"

✅ CORRECTO:

Dependency review verifica:
  ✅ Vulnerabilidades (CVEs)
  ✅ Licencias
  ✅ OpenSSF Scorecard
  ✅ Dependency changes

License checks:
  allow-licenses:
    - MIT
    - Apache-2.0
  
  deny-licenses:
    - GPL-3.0
    - AGPL-3.0

Bloquea si:
  - Nueva dependencia con GPL-3.0
  - Even si no tiene vulnerabilidades

Use case:
  - Compliance requirements
  - Commercial licensing
  - Open source policies

Por qué es trampa:
  - "Dependency" suena solo technical
  - Licenses son legal issue
  - Fácil olvidar configurar
```

### Trampa #20: Secret Scanning en Forks

```yaml
❌ INCORRECTO:
"Push protection aplica a forks públicos"

✅ CORRECTO:

Public fork de public repo:
  ✅ Secret scanning: Enabled
  ❌ Push protection: Disabled

Por qué:
  - Forks son para contribuir
  - Push protection bloquearía contributors
  - Parent repo tiene protection

Private fork:
  ✅ Hereda settings de parent
  ✅ Puede tener push protection
  ⚠️ Requiere GHAS license

Workflow correcto:
  1. Contributor forkea
  2. Push a fork (sin protection)
  3. Abre PR a parent
  4. Parent tiene protection
  5. Secret scanning detecta en parent

Por qué es trampa:
  - Esperarías protection en forks
  - Lógica de OSS contribution diferente
  - Fácil asumir herencia completa
```

---

## 🚨 Red Flags en Respuestas

### Palabras que indican respuesta incorrecta:

```yaml
❌ "siempre"
  - GHAS tiene muchos "depende de"
  - Rara vez algo es absoluto

❌ "nunca"
  - Exceptions existen
  - Configuración varía

❌ "automáticamente" (sin contexto)
  - Muchas cosas requieren opt-in
  - "Auto" puede ser misleading

❌ "todos" / "unlimited"
  - Hay límites en todo
  - Scope es limitado

❌ "gratis" (sin calificar)
  - Público vs privado
  - GHAS license requirements

✅ "depende de..."
✅ "si X está configurado..."
✅ "por default, pero..."
✅ "en repos públicos..."
✅ "con GHAS license..."
```

---

# ESCENARIOS DEL MUNDO REAL {#escenarios-reales}

## 🏢 Caso 1: Startup Tech Escalando

### Contexto

```yaml
Company: TechStartup Inc.
Repos: 50
Developers: 25
Tech stack: Node.js, Python, React
Current state:
  - No GHAS
  - Solo Dependabot alerts (gratis)
  - Security ad-hoc
Budget: Limited
Goal: Aprobar SOC2 audit en 6 meses
```

### Pregunta del Examen

> **TechStartup necesita cumplir SOC2. Tienen $50k budget para security tooling. ¿Cuál es la mejor estrategia de implementación de GHAS?**

**Opciones:**

A) Comprar GHAS para todos los repos inmediatamente  
B) Empezar con repos críticos (10), expandir gradualmente  
C) Solo usar features gratuitas hasta que crezcan  
D) Contratar third-party tool en lugar de GHAS  

**Respuesta Correcta: B**

### Análisis Completo

```yaml
Por qué B es correcto:

1. COST ANALYSIS:
   25 active committers × $49/mo (legacy) = $14,700/year
   Fit dentro de budget ✅

2. SOC2 Requirements:
   ✅ Vulnerability management
   ✅ Secure SDLC
   ✅ Code review process
   ✅ Access controls
   GHAS cumple todos

3. Phased Rollout (B):
   
   Month 1-2: Phase 1 (10 repos críticos)
     - Payment processing
     - User authentication
     - API gateway
     - Customer data
     
     Enable:
       ✅ Secret scanning + push protection
       ✅ CodeQL (default setup)
       ✅ Dependabot security updates
       ✅ Dependency review
     
     Cost: $14.7k/year (full team)
     Impact: 70% del risk cubierto
   
   Month 3-4: Phase 2 (20 repos)
     - Internal tools
     - Admin panels
     - Monitoring
     
     Same features
     Cost: Ya pagado (per committer)
     Impact: 90% risk covered
   
   Month 5-6: Phase 3 (20 repos restantes)
     - Documentation
     - Scripts
     - Experimental
     
     Cost: Ya pagado
     Impact: 100% coverage

4. SOC2 Audit Prep:
   Month 5:
     ✅ Security Overview screenshots
     ✅ Metrics dashboard
     ✅ Policy documentation
     ✅ Alert resolution logs
     ✅ Training records
   
   Month 6:
     ✅ Audit ready

Por qué A es incorrecto:
  - Overwhelming for teams
  - Posible alert fatigue
  - Costos inmediatos
  - No da tiempo para learning curve

Por qué C es incorrecto:
  - Features gratuitas insuficientes para SOC2
  - No code scanning en repos privados
  - No secret scanning en privados
  - Auditor rechazaría

Por qué D es incorrecto:
  - Third-party típicamente más caro
  - Menos integration con GitHub
  - GHAS tiene mejor ROI para GitHub-native orgs
```

---

## 🏥 Caso 2: Healthcare Enterprise

### Contexto

```yaml
Company: HealthCorp
Repos: 500+
Developers: 300
Compliance: HIPAA, SOC2, ISO27001
Current: Ya tiene GHAS enterprise-wide
Issue: 2,500 open security alerts
Goal: Reducir a <100 en Q1
```

### Pregunta del Examen

> **HealthCorp tiene 2,500 alerts de GHAS. El management quiere <100 para fin de trimestre. ¿Cuál es la estrategia MÁS EFECTIVA?**

**Opciones:**

A) Dismiss todas las alertas medium/low  
B) Crear Security Campaign priorizando por impacto  
C) Contratar 50 developers para fix todo  
D) Deshabilitar GHAS hasta que tengan capacidad  

**Respuesta Correcta: B**

### Solución Detallada

```yaml
Step 1: TRIAGE Y CLASSIFICATION

Analizar 2,500 alerts:
  Critical: 45
  High: 234
  Medium: 987
  Low: 1,234

Por tipo:
  Secret scanning: 123 (5%)
    - Active: 12 🔴
    - Inactive: 111
  
  Code scanning: 1,456 (58%)
    - SQL injection: 23
    - XSS: 45
    - Path traversal: 12
    - etc.
  
  Dependabot: 921 (37%)
    - Critical: 15
    - High: 156
    - Medium: 750

Por repo:
  Top 10 repos: 1,200 alerts (48%)
  Next 40 repos: 800 alerts (32%)
  Long tail 450 repos: 500 alerts (20%)

Step 2: SECURITY CAMPAIGNS

Campaign 1: "Active Secrets" (Week 1)
  Target: 12 active secrets
  Priority: P0
  Action: Immediate revocation
  Owner: Security team
  Deadline: 3 días
  
  Result: 12 fixed ✅

Campaign 2: "Critical Vulns in Production" (Week 1-2)
  Target: 45 critical alerts en production apps
  Priority: P0
  Action: Fix or mitigate
  Owner: By team
  Deadline: 7 días
  
  Result: 40 fixed, 5 mitigated ✅

Campaign 3: "High Severity Dependabot" (Week 2-4)
  Target: 156 high severity deps
  Priority: P1
  Action: Merge Dependabot PRs
  Owner: By team
  Deadline: 30 días
  
  Automation:
    - Auto-approve patch updates
    - CI/CD testing
    - Batch merging
  
  Result: 140 fixed ✅

Campaign 4: "SQL Injection Elimination" (Week 3-6)
  Target: 23 SQL injection findings
  Priority: P1
  Action: Refactor to prepared statements
  Owner: Backend team
  Deadline: 45 días
  
  Result: 23 fixed ✅

Campaign 5: "XSS Cleanup" (Week 4-8)
  Target: 45 XSS vulnerabilities
  Priority: P1
  Action: Output encoding
  Owner: Frontend team
  Deadline: 60 días
  
  Result: 45 fixed ✅

Campaign 6: "Medium Severity Triage" (Week 5-12)
  Target: 987 medium alerts
  Priority: P2
  Action: Fix or document dismiss
  Owner: All teams
  Deadline: 90 días
  
  Strategy:
    - Bulk dismiss false positives (documented)
    - Fix legitimate issues
    - Risk accept with approval
  
  Result:
    Fixed: 345
    Dismissed (valid): 542
    Accepted risk: 100

Step 3: PROCESS IMPROVEMENTS

Prevention:
  ✅ Mandatory dependency review on PRs
  ✅ Push protection enabled org-wide
  ✅ CodeQL required status check
  ✅ Security training for all devs
  ✅ Weekly security champions meeting

Automation:
  ✅ Auto-merge Dependabot patches
  ✅ Auto-triage low severity (custom rules)
  ✅ Slack notifications for critical
  ✅ Weekly digest reports

Step 4: RESULTS

End of Q1:
  Starting: 2,500 alerts
  Fixed: 565
  Dismissed (valid): 653
  Risk accepted: 105
  
  Remaining: 1,177
  
  But focus on:
    Critical: 0 ✅
    High: 50 (78% reduction)
    Medium: 400 (59% reduction)
    Low: 727 (41% reduction)

Recalibrated goal:
  Total <100 alerts was unrealistic
  New metric:
    - 0 critical ✅
    - <50 high ✅
    - Manageable backlog ✅

Compliance status:
  ✅ HIPAA: No active vulnerabilities in PHI systems
  ✅ SOC2: <7 day MTTR for critical
  ✅ ISO27001: Security metrics tracked

Por qué otras opciones son incorrectas:

A) Dismiss todas las medium/low:
  - Compliance violation
  - No addressing root causes
  - Medium alerts pueden ser serious

C) Contratar 50 developers:
  - Costo prohibitivo ($10M+/year)
  - Onboarding time
  - Not sustainable

D) Deshabilitar GHAS:
  - Compliance violation
  - Pérdida de visibilidad
  - Irresponsible
```

---

## 💳 Caso 3: Fintech con Compliance Estricto

### Contexto

```yaml
Company: PaymentPro
Industry: Fintech (payment processor)
Repos: 75 (todos privados)
Compliance: PCI-DSS Level 1, SOC2 Type II
Issue: Auditor encontró secrets en historial de Git
Severity: Critical finding
Deadline: 30 días para remediar
```

### Pregunta del Examen

> **PaymentPro descubre que hay AWS keys en el historial de Git de 15 repos. El auditor dio 30 días para remediar completamente. ¿Qué secuencia de acciones es correcta?**

**Opciones:**

A) 1. Habilitar push protection → 2. Esperar a que prevenga futuros  
B) 1. Revocar keys → 2. Limpiar historial → 3. Habilitar protection → 4. Audit  
C) 1. Limpiar historial → 2. Notificar clientes → 3. Habilitar scanning  
D) 1. Crear nuevos repos → 2. Migrar código → 3. Delete repos viejos  

**Respuesta Correcta: B**

### Plan de Remediación Completo

```yaml
FASE 1: IMMEDIATE RESPONSE (Día 1 - 2 horas)

Hour 1: Assess Damage
  
  Tasks:
    1. Run secret scanning en 15 repos
       gh api /repos/:owner/:repo/secret-scanning/alerts
    
    2. Identificar todos los secretos:
       - AWS access keys: 23 instances
       - AWS secret keys: 23 instances
       - RDS passwords: 5 instances
       - API keys: 12 instances
       
       Total: 63 secrets exposed
    
    3. Verificar validez:
       aws sts get-caller-identity --profile 
       
       Active: 18 ✅ (CRÍTICO)
       Inactive: 45
    
    4. Check CloudTrail logs:
       aws cloudtrail lookup-events \
         --lookup-attributes AttributeKey=Username,AttributeValue=compromised-key \
         --start-time 2024-01-01
       
       Unauthorized access: NONE detected ✅
       But: Exposure window = risk

Hour 2: Revoke Everything

  Tasks:
    1. Revoke todas las keys (active + inactive):
       aws iam delete-access-key \
         --access-key-id AKIA... \
         --user-name app-user
       
       Script para bulk revocation
    
    2. Rotate credentials:
       - Generar nuevas keys
       - Update en secret managers (Vault/AWS Secrets Manager)
       - Deploy new configs
    
    3. Test applications:
       - Verificar que apps funcionan con nuevas keys
       - No downtime
    
    4. Document timeline:
       - Exposure period
       - Actions taken
       - Verification de no compromise

FASE 2: CLEAN HISTORY (Día 1-3)

Tool: git-filter-repo (mejor que BFG)

Step 1: Identify exact commits
  
  git log --all --full-history -- '**/config.js' | grep -i "aws"
  
  Find:
    - Commit SHAs con secrets
    - Filenames
    - Branches affected

Step 2: Create backup

  # Backup completo
  git clone --mirror https://github.com/org/repo.git repo-backup
  
  # Store en S3
  tar -czf repo-backup.tar.gz repo-backup/
  aws s3 cp repo-backup.tar.gz s3://backups/

Step 3: Clean with git-filter-repo

  # Install
  pip install git-filter-repo
  
  # Create replacement file
  cat > ../secrets-replacement.txt <<EOF
  AKIAIOSFODNN7EXAMPLE==>REDACTED_AWS_KEY
  wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY==>REDACTED_SECRET
  EOF
  
  # Run filter
  git filter-repo --replace-text ../secrets-replacement.txt
  
  # Verify
  git log --all --full-history -S 'AKIAIOSFODNN7EXAMPLE'
  # Should return 0 results

Step 4: Force push

  # Warning: Destructive!
  git push --force --all
  git push --force --tags
  
  # Notify team via Slack:
  "🚨 FORCE PUSH to payment-api repo
   Reason: Secret remediation
   Action: All developers must re-clone
   Do NOT pull, do git clone fresh"

Step 5: Verify cleanup

  # Clone fresh
  git clone https://github.com/org/repo.git verify
  cd verify
  
  # Search for any secrets
  git log --all --full-history -S 'AKIA'
  git log --all --full-history -S 'aws_secret'
  
  # Should be clean ✅

Step 6: Handle forks

  # List all forks
  gh api /repos/:owner/:repo/forks
  
  # Contact fork owners
  # Request they update from parent
  # Or delete old forks

FASE 3: ENABLE PROTECTION (Día 3-5)

Step 1: Organization-wide secret scanning

  Organization Settings
    → Code security
    → Secret scanning
      [✓] Enable for all repositories
      [✓] Enable push protection for all repositories
      [✓] Automatically enable for new repositories

Step 2: Custom patterns (Payment-specific)

  # Stripe keys
  Pattern: sk_live_[0-9a-zA-Z]{24,99}
  
  # Internal API keys
  Pattern: PAYMENTPRO-[A-Z0-9]{32}
  
  # Database URLs
  Pattern: postgresql://[^:]+:[^@]+@[^/]+/payment_db

Step 3: Push protection configuration

  Settings:
    [✓] Allow bypasses: NO
    [✓] Require justification: YES (si permite bypasses)
    Bypass expires: 7 days (si permite)
  
  For PCI compliance: NO bypasses allowed

Step 4: Branch protection for all repos

  Script para bulk apply:
  
  repos=$(gh api /orgs/paymentpro/repos --jq '.[].name')
  
  for repo in $repos; do
    gh api -X PUT /repos/paymentpro/$repo/branches/main/protection \
      -f required_status_checks[strict]=true \
      -f required_status_checks[contexts][]="secret-scanning" \
      -f enforce_admins=true \
      -f restrictions=null
  done

FASE 4: AUDIT & DOCUMENTATION (Día 5-7)

Step 1: Generate evidence

  1. Screenshots:
     - Secret scanning enabled ✅
     - Push protection enabled ✅
     - All alerts resolved ✅
  
  2. Reports:
     - CSV export de Security Overview
     - Timeline de remediation
     - Git history verification
  
  3. Policies:
     - Updated secret management policy
     - Incident response playbook
     - Developer training materials

Step 2: External audit

  1. Bring in third-party:
     - Security consultant
     - Verify history clean
     - Pentest for any lingering exposure
  
  2. AWS audit:
     - Review all IAM policies
     - Verify no unauthorized access
     - Check CloudTrail completeness

Step 3: Compliance documentation

  For PCI-DSS:
    ✅ 6.3.2: Secure development practices
    ✅ 6.5.3: Insecure cryptographic storage
    ✅ 12.10.1: Incident response
  
  For SOC2:
    ✅ CC6.1: Logical access controls
    ✅ CC7.2: System monitoring
    ✅ CC9.2: Risk mitigation

Step 4: Submit to auditor

  Package:
    1. Executive summary
    2. Timeline of discovery → resolution
    3. Technical evidence (screenshots, logs)
    4. Process improvements
    5. Prevention measures
    6. Training completion
    7. Monitoring setup
  
  Outcome: Finding closed ✅

FASE 5: PREVENTION (Día 7-30)

Step 1: Developer training

  Week 1:
    - Lunch & learn: Secret management
    - Hands-on: Using AWS Secrets Manager
    - Demo: Push protection in action
  
  Week 2:
    - Code review guidelines update
    - Secret scanning workshop
    - Q&A session

Step 2: Process changes

  New requirements:
    ✅ All secrets in Vault/AWS Secrets Manager
    ✅ No hardcoded credentials ever
    ✅ Pre-commit hooks (local)
    ✅ Quarterly secret rotation
    ✅ Monthly access reviews

Step 3: Monitoring

  Setup:
    - Security Overview dashboard
    - Weekly reports to leadership
    - Slack alerts for new secrets
    - PagerDuty for active secrets
  
  SLAs:
    - Active secret: Revoke within 1 hour
    - Inactive secret: Clean within 24 hours

Step 4: Continuous improvement

  Monthly:
    - Review dismissed alerts
    - Update custom patterns
    - Team retrospectives
  
  Quarterly:
    - External pentest
    - Compliance audit prep
    - Policy review

RESULTS:

Day 30 Status:
  ✅ All secrets revoked
  ✅ All repos history cleaned
  ✅ Push protection enabled org-wide
  ✅ 0 active secrets in any repo
  ✅ Custom patterns deployed
  ✅ 100% team trained
  ✅ Auditor finding closed
  ✅ PCI-DSS compliant

Cost:
  - GHAS: $14,700/year (already had)
  - External audit: $15,000
  - Training: $5,000
  - Tools (git-filter-repo, etc): $0
  Total: $20,000
  
  vs.
  
  Cost of breach: $5M+ (estimated)
  ROI: 250x

Lessons learned:
  - Prevention >> Remediation
  - Git history is permanent (kinda)
  - Automation is key
  - Training prevents recurrence
```

---

[Continuará con las 100 preguntas de práctica en la siguiente sección...]

---

# 100 PREGUNTAS DE PRÁCTICA {#preguntas-practica}

## Formato del Examen Real

```yaml
Tipo de preguntas:
  - Multiple choice (1 respuesta)
  - Multiple select (2+ respuestas)
  - Scenario-based
  - True/False

Distribución por dominio:
  - Dominio 1 (Características GHAS): ~10 preguntas
  - Dominio 2 (Secret Scanning): ~10 preguntas
  - Dominio 3 (Dependabot): ~25 preguntas
  - Dominio 4 (CodeQL): ~18 preguntas
  - Dominio 5 (Best Practices): ~7 preguntas

Total: ~70 preguntas
Tiempo: 120 minutos
Passing score: 70%
```

---

## DOMINIO 1: Características y Funcionalidades de GHAS

### Pregunta 1

**Una organización quiere visibilidad centralizada del estado de seguridad de todos sus repositorios. ¿Qué feature de GHAS proporciona esto?**

A) Dependabot alerts  
B) Security Overview  
C) Code scanning dashboard  
D) Secret scanning alerts  

<details>
<summary>Ver respuesta</summary>

**Respuesta: B) Security Overview**

**Explicación:**
- Security Overview es un dashboard centralizado a nivel organización/empresa
- Proporciona métricas, trends y filtros cross-repo
- Otras opciones son feature-specific, no overview

**Por qué otras son incorrectas:**
- A) Dependabot alerts: Solo muestra vulnerabilidades de dependencias
- C) Code scanning dashboard: Solo para code scanning
- D) Secret scanning alerts: Solo secretos

**Recursos:**
- https://docs.github.com/en/code-security/security-overview/about-security-overview
</details>

---

### Pregunta 2

**¿Qué features de GHAS están disponibles GRATIS para repositorios públicos? (Selecciona todas las correctas)**

A) Secret scanning  
B) Push protection  
C) CodeQL code scanning  
D) Dependabot alerts  
E) Security Overview  

<details>
<summary>Ver respuesta</summary>

**Respuestas: A, B, C, D**

**Explicación:**
- Repositorios públicos tienen casi todas las features gratis
- Security Overview NO está disponible para públicos

**Detalles:**
- ✅ Secret scanning: Gratis y auto-habilitado
- ✅ Push protection: Gratis (desde 2025)
- ✅ CodeQL: Gratis pero requiere setup manual
- ✅ Dependabot alerts: Gratis y auto-habilitado
- ❌ Security Overview: Solo para orgs con GHAS

**Recursos:**
- https://docs.github.com/en/get-started/learning-about-github/about-github-advanced-security
</details>

---

### Pregunta 3

**Tu empresa tiene GitHub Enterprise Cloud. El CISO quiere FORZAR que todos los repositorios en todas las organizaciones tengan secret scanning habilitado, sin excepciones. ¿Dónde configuras esto?**

A) Organization settings → Code security  
B) Repository settings → Security  
C) Enterprise policies → Code security  
D) GitHub Support ticket  

<details>
<summary>Ver respuesta</summary>

**Respuesta: C) Enterprise policies → Code security**

**Explicación:**
- Enterprise policies permiten ENFORCEMENT
- "Sin excepciones" = necesitas enforced policy
- Organization/Repo settings pueden ser overridden

**Jerarquía:**
```
Enterprise (enforced) ← ESTO
  ↓ no puede cambiar
Organization
  ↓ no puede cambiar
Repository
```

**Recursos:**
- https://docs.github.com/en/enterprise-cloud@latest/admin/policies/enforcing-policies-for-your-enterprise/enforcing-policies-for-code-security-and-analysis-for-your-enterprise
</details>

---

### Pregunta 4

**¿Cuál es la diferencia principal entre Dependabot alerts y Dependency review?**

A) Dependabot alerts es gratis, Dependency review requiere GHAS  
B) Dependabot alerts detecta existente, Dependency review previene nuevo  
C) Dependabot alerts es para npm, Dependency review para todos los ecosistemas  
D) No hay diferencia, son nombres diferentes para lo mismo  

<details>
<summary>Ver respuesta</summary>

**Respuesta: B) Dependabot alerts detecta existente, Dependency review previene nuevo**

**Explicación:**
- Dependabot alerts: REACTIVO (post-merge)
- Dependency review: PROACTIVO (pre-merge, en PR)

**Comparación:**
| | Dependabot Alerts | Dependency Review |
|-|-------------------|-------------------|
| Cuándo | Después de merge | Durante PR |
| Acción | Crea alerta | Bloquea merge |
| Ubicación | Security tab | PR checks |

**Por qué A es incorrecta:**
- Dependabot alerts ES gratis, pero ese no es la diferencia PRINCIPAL
- La diferencia clave es reactivo vs proactivo

**Recursos:**
- https://docs.github.com/en/code-security/supply-chain-security/understanding-your-software-supply-chain/about-dependency-review
</details>

---

### Pregunta 5

**Un developer en tu equipo reporta que CodeQL tarda 45 minutos en analizar el repositorio Java, pero solo 5 minutos para JavaScript. ¿Cuál es la razón MÁS probable?**

A) Java tiene más vulnerabilidades que JavaScript  
B) Java requiere compilación, JavaScript no  
C) CodeQL tiene menos queries para JavaScript  
D) El repositorio Java es más grande  

<details>
<summary>Ver respuesta</summary>

**Respuesta: B) Java requiere compilación, JavaScript no**

**Explicación:**
- Java es lenguaje compilado → requiere build completo
- JavaScript es interpretado → parse directo

**Proceso:**
```
Java:
  1. Setup CodeQL
  2. BUILD (← tiempo aquí)
  3. Análisis
  4. Upload

JavaScript:
  1. Setup CodeQL
  2. Análisis (no build)
  3. Upload
```

**Por qué otras son incorrectas:**
- A) Número de vulnerabilidades no afecta tiempo de scan
- C) JavaScript tiene queries similares
- D) Tamaño afecta pero no es la razón principal (build es el factor)

**Recursos:**
- https://codeql.github.com/docs/codeql-overview/supported-languages-and-frameworks/
</details>

---

## DOMINIO 2: Secret Scanning

### Pregunta 6

**Un developer acaba de pushear un AWS access key a un repositorio público. ¿Qué sucede automáticamente? (Selecciona todas las correctas)**

A) GitHub detecta el secreto  
B) GitHub notifica al developer  
C) GitHub revoca el AWS key  
D) AWS es notificado del leak  
E) El commit es revertido  

<details>
<summary>Ver respuesta</summary>

**Respuestas: A, B, D**

**Explicación:**

**Lo que SÍ pasa:**
- ✅ Secret scanning detecta (A)
- ✅ Notificación al repo admin/developer (B)
- ✅ AWS es notificado por GitHub (D)

**Lo que NO pasa:**
- ❌ GitHub NO revoca (C) - es responsabilidad del usuario/AWS
- ❌ Commit NO se revierte (E) - permanece en historial

**Proceso real:**
```
1. Push con secret
2. Secret scanning detecta (30 seg después)
3. Alert creada en Security tab
4. Email a repo admin
5. Notificación a AWS (partner notification)
6. AWS puede tomar acción (revoke, notify account owner)
```

**Recursos:**
- https://docs.github.com/en/code-security/secret-scanning/introduction/about-secret-scanning
</details>

---

### Pregunta 7

**Push protection está habilitado en tu repositorio. Un developer intenta pushear un GitHub Personal Access Token. ¿Qué puede hacer el developer? (Selecciona todas las correctas)**

A) Remover el secret del código y push de nuevo  
B) Request bypass con justificación (si está permitido)  
C) Force push para sobreescribir el check  
D) Pushear a diferente branch para evitar el check  

<details>
<summary>Ver respuesta</summary>

**Respuestas: A, B**

**Explicación:**

**Opciones válidas:**
- ✅ A) Remover secret y push: Solución correcta
- ✅ B) Request bypass: Si está configurado allow_bypass=true

**Opciones NO válidas:**
- ❌ C) Force push NO evade push protection
- ❌ D) Diferentes branches NO evaden (aplica a todos los pushes)

**Push protection aplica a:**
- ✅ Todos los branches
- ✅ Todos los commits
- ✅ Force pushes
- ✅ Tags

**Bypass workflow:**
```
IF allow_bypass = true:
  Developer puede:
    1. Click "Bypass protection"
    2. Proveer justificación
    3. Push permitido
    4. Alerta creada igual
    5. Bypass logged para audit

IF allow_bypass = false:
  Developer debe:
    1. Remover secreto
    2. Commit fix
    3. Push again
```

**Recursos:**
- https://docs.github.com/en/code-security/secret-scanning/introduction/about-push-protection
</details>

---

### Pregunta 8

**Tu organización quiere detectar secretos en el formato "ACME-API-[32 caracteres alfanuméricos]". ¿Dónde creas este custom pattern?**

A) Repository settings → Secret scanning  
B) Organization settings → Code security → Custom patterns  
C) Enterprise settings → Policies  
D) .github/secret-scanning.yml file  

<details>
<summary>Ver respuesta</summary>

**Respuesta: B) Organization settings → Code security → Custom patterns**

**Explicación:**
- Custom patterns solo se pueden crear a nivel ORGANIZACIÓN
- NO a nivel repository
- NO a nivel enterprise

**Pattern:**
```regex
ACME-API-[A-Z0-9]{32}
```

**Scope:**
- Creado en org settings
- Aplica a TODOS los repos de la org
- No hay patterns repo-specific

**Recursos:**
- https://docs.github.com/en/code-security/secret-scanning/defining-custom-patterns-for-secret-scanning
</details>

---

### Pregunta 9

**Una alerta de secret scanning muestra "Validity: Active" para un GitHub token. ¿Qué significa esto?**

A) El secreto fue detectado recientemente  
B) El token es válido y funcional  
C) El token está en código activamente usado  
D) El secreto está en la rama activa (main)  

<details>
<summary>Ver respuesta</summary>

**Respuesta: B) El token es válido y funcional**

**Explicación:**

**Estados de validez:**
- **Active**: GitHub verificó el token y AÚN funciona
- **Inactive**: Token revocado o expirado
- **Unknown**: No se pudo verificar

**Proceso de validity check:**
```
1. Secret detectado
2. GitHub intenta usar el token
3. API call a GitHub
4. ¿Funciona?
   - SÍ → Active ✅ CRITICAL
   - NO → Inactive (low priority)
   - Error → Unknown
```

**Acción requerida para "Active":**
- 🚨 Revocar INMEDIATAMENTE
- 🚨 Rotar a nuevo token
- 🚨 Audit logs de uso
- 🚨 Limpiar historial

**Recursos:**
- https://docs.github.com/en/code-security/secret-scanning/managing-alerts-from-secret-scanning/evaluating-alerts
</details>

---

### Pregunta 10

**¿Cuál es la principal diferencia entre secret scanning y push protection?**

A) Secret scanning es gratis, push protection requiere GHAS  
B) Secret scanning detecta después, push protection previene antes  
C) Secret scanning es para públicos, push protection para privados  
D) Secret scanning usa AI, push protection usa regex  

<details>
<summary>Ver respuesta</summary>

**Respuesta: B) Secret scanning detecta después, push protection previene antes**

**Explicación:**

**Timeline comparison:**
```
SIN push protection:
  Code → Commit → Push → (30 seg) → Secret detected
  ⚠️ Secret YA está en GitHub

CON push protection:
  Code → Commit → Push → BLOCKED
  ✅ Secret NUNCA llega a GitHub
```

**Por qué otras son incorrectas:**
- A) Ambos requieren GHAS para repos privados (desde 2025 push protection gratis en públicos)
- C) Ambos funcionan en públicos y privados
- D) Ambos usan regex patterns, no AI

**Recursos:**
- https://docs.github.com/en/code-security/secret-scanning/introduction/about-push-protection
</details>

---

## DOMINIO 3: Dependabot y Dependency Review

### Pregunta 11

**¿Qué archivo de configuración es REQUERIDO para Dependabot alerts en repositorios privados?**

A) dependabot.yml  
B) .github/dependabot-config.json  
C) Ninguno (Dependabot alerts es automático)  
D) security.yml  

<details>
<summary>Ver respuesta</summary>

**Respuesta: C) Ninguno (Dependabot alerts es automático)**

**Explicación:**

**Dependabot alerts:**
- ✅ NO requiere dependabot.yml
- ✅ Automático al habilitar
- ✅ Gratis para repos públicos y privados

**dependabot.yml solo para:**
- Dependabot VERSION updates
- Scheduled updates
- Grouping
- Custom configurations

**Confusión común:**
```yaml
❌ INCORRECTO: "Necesito dependabot.yml para alerts"

✅ CORRECTO:
  - Dependabot alerts: Sin config
  - Dependabot security updates: Sin config
  - Dependabot version updates: Requiere dependabot.yml
```

**Recursos:**
- https://docs.github.com/en/code-security/dependabot/dependabot-alerts/about-dependabot-alerts
</details>

---

### Pregunta 12

**Un PR introduce lodash@4.17.10 (vulnerable). Dependency review está configurado con `fail-on-severity: high`. ¿Qué sucede?**

A) El PR se mergea normalmente  
B) El PR es bloqueado y no puede mergearse  
C) Se crea una alerta pero no bloquea  
D) Dependabot crea un PR de fix  

<details>
<summary>Ver respuesta</summary>

**Respuesta: B) El PR es bloqueado y no puede mergearse**

**Explicación:**

**Lodash@4.17.10 tiene:**
- CVE-2020-8203
- Severity: High
- Prototype pollution

**Dependency review workflow:**
```
1. PR changes lodash version
2. Dependency review Action ejecuta
3. Detecta CVE-2020-8203 (High)
4. fail-on-severity: high → FAIL
5. Check status: ❌
6. Merge button: BLOCKED
```

**Developer debe:**
- Update lodash a versión segura (4.17.21+)
- Push cambio
- Dependency review re-ejecuta
- Check pasa ✅
- Merge permitido

**Recursos:**
- https://github.com/actions/dependency-review-action
</details>

---

### Pregunta 13

**Tu equipo quiere agrupar todas las actualizaciones de React en un solo PR semanal. ¿Cómo lo configuras?**

A) GitHub Actions workflow custom  
B) dependabot.yml con groups  
C) Organization settings → Dependabot  
D) No es posible, Dependabot crea PRs individuales  

<details>
<summary>Ver respuesta</summary>

**Respuesta: B) dependabot.yml con groups**

**Explicación:**

**Configuración:**
```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
    
    groups:
      react-ecosystem:
        patterns:
          - "react"
          - "react-dom"
          - "@types/react"
          - "@types/react-dom"
        update-types:
          - "minor"
          - "patch"
```

**Resultado:**
```
SIN grouping:
  PR #1: Bump react from 18.2.0 to 18.3.0
  PR #2: Bump react-dom from 18.2.0 to 18.3.0
  PR #3: Bump @types/react from 18.0.0 to 18.0.1
  PR #4: Bump @types/react-dom from 18.0.0 to 18.0.1

CON grouping:
  PR #1: Bump react-ecosystem group
    - react: 18.2.0 → 18.3.0
    - react-dom: 18.2.0 → 18.3.0
    - @types/react: 18.0.0 → 18.0.1
    - @types/react-dom: 18.0.0 → 18.0.1
```

**Recursos:**
- https://docs.github.com/en/code-security/dependabot/dependabot-version-updates/grouping-dependabot-updates
</details>

---

### Pregunta 14

**¿Qué rol mínimo necesita un usuario para VER alertas de Dependabot en un repositorio privado?**

A) Admin  
B) Write  
C) Read  
D) Security Manager  

<details>
<summary>Ver respuesta</summary>

**Respuesta: C) Read**

**Explicación:**

**Dependabot alerts visibilidad:**
| Rol | Ver Alerts | Dismiss Alerts |
|-----|-----------|----------------|
| Read | ✅ | ❌ |
| Triage | ✅ |
