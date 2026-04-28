# DOMINIO 2: CONFIGURAR Y USAR EL ESCANEO DE SECRETOS (15%)

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

**1. Configurar notificaciones:**

```
Settings → Notifications
  └─ Security alerts
      ├─ [✓] Email notifications
      │   └─ security@company.com
      ├─ [✓] Web notifications
      └─ [✓] Slack integration
          └─ #security-alerts
```

**2. Configurar custom patterns (opcional):**

```
Settings → Code security → Secret scanning
  → Custom patterns
      → [New pattern]
          ├─ Name: Internal API Key
          ├─ Secret format: INT-[A-Z0-9]{32}
          ├─ Test string: INT-ABC123...
          └─ [Save pattern]
```

**3. Configurar bypass policies:**

```
Organization Settings → Code security
  → Secret scanning
      → Push protection
          ├─ [✓] Allow bypasses
          ├─ [✓] Require bypass reason
          ├─ [ ] Require approval (Enterprise only)
          └─ Bypass expires: [7 days ▼]
```

**4. Configurar CODEOWNERS para alertas:**

```bash
# .github/CODEOWNERS
# Security team owns all secret scanning alerts

* @org/developers
/.github/workflows/* @org/devops
**/secrets.yml @org/security
**/config*.* @org/security
```

### Troubleshooting común

**Problema 1**: "Advanced Security not available"
```
Causa: No hay licencias GHAS disponibles
Solución: Comprar más seats o liberar seats no usados

# Check usage:
gh api /orgs/:org/settings/billing/advanced-security
```

**Problema 2**: "Secret scanning failed to start"
```
Causa: Repository es fork sin GHAS habilitado en parent
Solución: Habilitar GHAS en parent repository o desconectar fork

# Desconectar fork:
# Settings → Danger Zone → "Detach fork"
```

**Problema 3**: "No scan results after 1 hour"
```
Causa: Repository muy grande o muchos commits
Solución: Esperar más tiempo o contactar GitHub Support

# Monitor status:
gh api /repos/:owner/:repo/code-scanning/analyses | \
  jq '.[0].created_at'
```

**Problema 4**: "Push protection not working"
```
Causa: Feature no habilitado o bypass configurado
Solución:
# Verify:
gh api /repos/:owner/:repo | \
  jq '.security_and_analysis.secret_scanning_push_protection.status'

# Should be: "enabled"
```

### Best practices

```yaml
✅ Rollout strategy:
  1. Pilot con 5-10 repos no-críticos
  2. Evaluar false positives
  3. Ajustar custom patterns
  4. Expandir a 50% de repos
  5. Habilitar push protection
  6. Full rollout a 100%

✅ Team enablement:
  - Training session sobre secret management
  - Documentación de workflows
  - Runbooks para common scenarios
  - Regular retros de alerts

✅ Monitoring:
  - Weekly reports de:
      - Nuevas alertas
      - Alertas resueltas
      - Bypass requests
      - False positive rate
  - Dashboard con métricas clave
  - Alerting para critical findings

❌ Evitar:
  - Habilitar en producción sin testing
  - No configurar notificaciones
  - Ignorar alertas por "alert fatigue"
  - No entrenar al equipo
  - Deshabilitar push protection "temporalmente"
```

**Enlaces:**
- https://docs.github.com/en/code-security/secret-scanning/configuring-secret-scanning-for-your-repositories
- https://docs.github.com/en/code-security/secret-scanning/introduction/about-push-protection

---

## 2.5 Respuestas apropiadas a alertas de Secret Scanning

### Workflow de decisión

```
Alerta de secret scanning aparece
        ↓
[1] EVALUAR VALIDEZ
        ├─ Active → 🔴 CRITICAL
        ├─ Inactive → 🟡 MEDIUM
        └─ Unknown → 🟠 HIGH
        ↓
[2] VERIFICAR EXPOSICIÓN
        ├─ ¿Cuánto tiempo expuesto?
        ├─ ¿Repositorio público o privado?
        ├─ ¿Quién tiene acceso?
        └─ ¿Se usó el secreto?
        ↓
[3] EVALUAR IMPACTO
        ├─ ¿Qué recursos protege?
        ├─ ¿Cuál es el blast radius?
        ├─ ¿Hay datos sensibles accesibles?
        └─ ¿Compliance implications?
        ↓
[4] DECIDIR ACCIÓN
```

### Matriz de decisión

| Validez | Tipo de repo | Tiempo expuesto | Acción |
|---------|--------------|-----------------|--------|
| **Active** | Público | Cualquiera | 🚨 EMERGENCY |
| **Active** | Privado | >24h | 🔴 URGENT |
| **Active** | Privado | <24h | 🟠 HIGH |
| **Inactive** | Cualquiera | Cualquiera | 🟡 MEDIUM |
| **Unknown** | Público | >7d | 🔴 URGENT |
| **Unknown** | Privado | Cualquiera | 🟠 HIGH |

### Acciones por tipo de secreto

#### A) GitHub Personal Access Token

**Si es Active:**

```bash
# 1. REVOCAR INMEDIATAMENTE
https://github.com/settings/tokens
   → Locate token
   → [Delete] o [Revoke]

# 2. VERIFICAR USO
gh api /user/events | jq '.[] | select(.created_at > "2026-04-26")'
# Revisar:
# - IPs de acceso
# - Repositories accedidos
# - Actions realizadas

# 3. ROTAR
# Crear nuevo token con scopes mínimos necesarios
gh auth login --scopes repo,read:org

# 4. ACTUALIZAR DEPENDENCIAS
# CI/CD pipelines
# GitHub Actions secrets
# Aplicaciones que usan el token

# 5. LIMPIAR CÓDIGO
git filter-repo --invert-paths --path config.js
git push --force

# 6. DOCUMENTAR INCIDENT
# Crear postmortem:
# - Timeline de exposición
# - Scope de compromiso
# - Actions tomadas
# - Prevention measures
```

**Si es Inactive:**

```bash
# 1. VERIFICAR REVOCACIÓN
# Confirmar que el token ya no funciona

# 2. LIMPIAR CÓDIGO
# Remover referencias al token
git rm config/secrets.js
git commit -m "Remove revoked GitHub token"

# 3. DISMISS ALERT
# In GitHub UI:
# Reason: "Won't fix - token already revoked"
# Comment: "Token was revoked on 2026-04-20"
```

#### B) AWS Access Key

**Si es Active:**

```bash
# 1. REVOCAR INMEDIATAMENTE
aws iam delete-access-key \
  --access-key-id AKIA... \
  --user-name compromised-user

# 2. AUDIT AWS CloudTrail
aws cloudtrail lookup-events \
  --lookup-attributes AttributeKey=Username,AttributeValue=compromised-user \
  --start-time 2026-04-01 \
  --end-time 2026-04-27 \
  > cloudtrail-audit.json

# Buscar:
# - EC2 instances launched
# - S3 buckets accessed
# - IAM changes
# - Unusual regions/IPs

# 3. VERIFICAR RECURSOS NO AUTORIZADOS
# EC2 instances de cryptomining
aws ec2 describe-instances --filters "Name=key-name,Values=*"

# S3 buckets expuestos
aws s3api list-buckets

# Lambda functions sospechosas
aws lambda list-functions

# 4. ROTAR CREDENCIALES
aws iam create-access-key --user-name production-app

# 5. APLICAR LEAST PRIVILEGE
aws iam attach-user-policy \
  --user-name production-app \
  --policy-arn arn:aws:iam::aws:policy/ReadOnlyAccess

# 6. HABILITAR MFA
aws iam enable-mfa-device \
  --user-name production-app \
  --serial-number arn:aws:iam::123456:mfa/app \
  --authentication-code1 123456 \
  --authentication-code2 789012

# 7. CONFIGURAR ALERTAS
# CloudWatch alarm para API calls no autorizadas
aws cloudwatch put-metric-alarm \
  --alarm-name UnauthorizedAPICalls \
  --alarm-actions arn:aws:sns:us-east-1:123456:security-alerts

# 8. BILLING REVIEW
# Check por charges inesperados
aws ce get-cost-and-usage \
  --time-period Start=2026-04-01,End=2026-04-27 \
  --granularity DAILY \
  --metrics UnblendedCost
```

**Caso real - Cryptomining:**

```bash
# Scenario: AWS key leaked, used for cryptomining

# 1. Detected:
#    - CloudWatch: EC2 CPU 100% en us-west-2
#    - 50 c5.24xlarge instances (!!!!)
#    - Cost: $120/hour = $86,400/month

# 2. Response:
# Terminate todas las instances sospechosas
aws ec2 terminate-instances \
  --instance-ids $(aws ec2 describe-instances \
    --filters "Name=instance-state-name,Values=running" \
    --query "Reservations[*].Instances[?LaunchTime>'2026-04-26'][].InstanceId" \
    --output text)

# 3. Revoke key
aws iam delete-access-key --access-key-id AKIA...

# 4. Contact AWS Support
# Request billing adjustment (goodwill credit)

# 5. Enable AWS GuardDuty
aws guardduty create-detector --enable

# 6. Setup billing alerts
aws budgets create-budget \
  --budget file://budget.json \
  --notifications-with-subscribers file://alerts.json
```

#### C) Database Connection String

**Si es Active:**

```sql
-- 1. VERIFICAR CONEXIONES ACTIVAS
SELECT pid, usename, application_name, client_addr, backend_start
FROM pg_stat_activity
WHERE usename = 'compromised_user'
ORDER BY backend_start DESC;

-- 2. KILL SESIONES SOSPECHOSAS
SELECT pg_terminate_backend(pid)
FROM pg_stat_activity
WHERE usename = 'compromised_user'
  AND client_addr NOT IN ('10.0.1.5', '10.0.1.6');

-- 3. CAMBIAR PASSWORD
ALTER USER compromised_user WITH PASSWORD 'new_secure_password_12345!';

-- 4. REVISAR AUDIT LOGS
SELECT * FROM pg_stat_statements
WHERE userid = (SELECT oid FROM pg_user WHERE usename = 'compromised_user')
ORDER BY calls DESC
LIMIT 100;

-- 5. CHECK POR DATA EXFILTRATION
-- Queries con grandes resultsets
SELECT query, calls, total_time, rows
FROM pg_stat_statements
WHERE rows > 10000
ORDER BY total_time DESC;

-- 6. ROTAR CREDENCIALES EN APLICACIONES
-- Update connection strings in:
# - Kubernetes secrets
# - AWS Secrets Manager
# - Environment variables
# - Application configs

-- 7. RESTRICT ACCESS
-- Limitar por IP
-- pg_hba.conf:
host    dbname    username    10.0.1.0/24    md5
host    dbname    username    0.0.0.0/0      reject

-- 8. ENABLE SSL/TLS REQUIRED
ALTER USER compromised_user SET ssl TO on;

-- 9. AUDIT SCHEMA CHANGES
SELECT schemaname, tablename, usename, query_start
FROM pg_stat_activity
WHERE query LIKE '%ALTER%' OR query LIKE '%DROP%';
```

#### D) API Keys (Stripe, Twilio, etc.)

**Si es Active:**

```javascript
// 1. REVOKE KEY
// Stripe Dashboard:
// Developers → API keys → [Reveal] → [Roll key]

// 2. REVIEW API CALLS
const stripe = require('stripe')('sk_live_...');

const charges = await stripe.charges.list({
  created: {
    gte: Math.floor(Date.parse('2026-04-26') / 1000)
  },
  limit: 100
});

// Look for:
// - Unauthorized charges
// - Refunds needed
// - Unusual patterns

// 3. ALERT CUSTOMERS (if needed)
// If customer data was accessed:
// - Email notification
// - Credit monitoring offer
// - Incident report

// 4. UPDATE APPLICATIONS
// Kubernetes:
kubectl create secret generic stripe-secret \
  --from-literal=api-key=sk_live_NEW_KEY \
  --dry-run=client -o yaml | kubectl apply -f -

// Restart pods to pick up new secret
kubectl rollout restart deployment/payment-service

// 5. IMPLEMENT KEY ROTATION
// Set up automatic rotation
// AWS Secrets Manager with Lambda:
exports.handler = async (event) => {
  const newKey = await rotateStripeKey();
  await updateSecret('stripe-api-key', newKey);
  await notifySlack('#security', 'Stripe key rotated');
};

// 6. ENABLE WEBHOOK SIGNATURE VERIFICATION
// stripe-webhook.js
const sig = request.headers['stripe-signature'];
let event;
try {
  event = stripe.webhooks.constructEvent(
    request.body,
    sig,
    process.env.STRIPE_WEBHOOK_SECRET
  );
} catch (err) {
  return response.status(400).send(`Webhook Error: ${err.message}`);
}

// 7. SETUP MONITORING
// CloudWatch/Datadog alerts for:
// - Unusual charge volumes
// - Failed charges spike
// - API errors increase
// - Charges from new countries
```

### False Positives

**Cómo identificar:**

```yaml
Indicadores de false positive:
  - Secret en archivo de test: *_test.py, *_spec.js
  - Secret en comentario explicativo
  - Secret en documentación (README, docs/)
  - Secret es ejemplo/placeholder: "your-api-key-here"
  - Secret es hash/checksum, no credencial
  - Pattern match pero no es secreto real
```

**Cómo manejar:**

```bash
# 1. VERIFICAR que es false positive
# NO ASUMIR sin verificación

# 2. DISMISS ALERT
# GitHub UI → Alert → Dismiss
# Reason: "False positive"
# Comment: "This is a test fixture, not a real API key. File: tests/fixtures/sample.json"

# 3. PREVENIR RECURRENCIA
# Opción A: Exclude path from scanning
# .github/secret_scanning.yml
paths-ignore:
  - 'tests/**'
  - 'docs/**'
  - '**/*.md'

# Opción B: Comment in code
# Algunos scanners respetan:
# secretlint-disable-next-line
API_KEY = "example_key_12345"

# Opción C: Use placeholder values
# Good:
API_KEY = "sk_test_YOUR_KEY_HERE"
# Bad:
API_KEY = "sk_test_4eC39HqLyjWDarjtT1zdp7dc"  # ← Looks real!
```

### Workflow automatizado con GitHub Actions

```yaml
# .github/workflows/secret-alert-handler.yml
name: Secret Scanning Alert Handler

on:
  secret_scanning_alert:
    types: [created]

jobs:
  handle-alert:
    runs-on: ubuntu-latest
    steps:
      - name: Get alert details
        id: alert
        uses: actions/github-script@v7
        with:
          script: |
            const alert = context.payload.alert;
            return {
              secret_type: alert.secret_type,
              validity: alert.validity,
              created_at: alert.created_at
            };

      - name: Post to Slack
        if: steps.alert.outputs.validity == 'active'
        uses: slackapi/slack-github-action@v1
        with:
          channel-id: 'C01234567'
          slack-message: |
            🚨 CRITICAL: Active secret detected!
            Type: ${{ steps.alert.outputs.secret_type }}
            Repo: ${{ github.repository }}
            Action required: Revoke immediately
        env:
          SLACK_BOT_TOKEN: ${{ secrets.SLACK_BOT_TOKEN }}

      - name: Create PagerDuty incident
        if: |
          steps.alert.outputs.validity == 'active' &&
          contains(steps.alert.outputs.secret_type, 'aws')
        uses: peter-murray/pagerduty-incident-action@v1
        with:
          pagerduty-token: ${{ secrets.PAGERDUTY_TOKEN }}
          incident-title: "AWS credentials leaked in ${{ github.repository }}"
          incident-urgency: "high"

      - name: Auto-dismiss if test file
        if: contains(github.event.alert.locations[0].path, 'test')
        uses: actions/github-script@v7
        with:
          script: |
            await github.rest.secretScanning.updateAlert({
              owner: context.repo.owner,
              repo: context.repo.name,
              alert_number: context.payload.alert.number,
              state: 'resolved',
              resolution: 'false_positive',
              resolution_comment: 'Auto-dismissed: found in test file'
            });
```

### SLAs recomendados

| Severidad | Validez | Repo tipo | SLA para resolución |
|-----------|---------|-----------|---------------------|
| Critical | Active | Public | 1 hora |
| Critical | Active | Private | 4 horas |
| High | Active | Any | 24 horas |
| High | Unknown | Public | 24 horas |
| Medium | Inactive | Any | 7 días |
| Medium | Unknown | Private | 7 días |
| Low | Inactive | Private | 30 días |

### Checklist de respuesta

```markdown
## Secret Scanning Alert Response Checklist

### Immediate Actions (0-1 hour)
- [ ] Verify alert authenticity (not false positive)
- [ ] Check validity status (active/inactive/unknown)
- [ ] Assess repository exposure (public/private/internal)
- [ ] Identify secret type and associated resources
- [ ] Escalate to security team if critical

### Short-term Actions (1-24 hours)
- [ ] Revoke/rotate compromised secret
- [ ] Audit logs for unauthorized access
- [ ] Identify all services using the secret
- [ ] Update secret in all locations
- [ ] Test applications after rotation
- [ ] Notify stakeholders if needed

### Medium-term Actions (1-7 days)
- [ ] Review security posture of affected resources
- [ ] Implement additional monitoring
- [ ] Clean secret from git history
- [ ] Update documentation and runbooks
- [ ] Conduct team training if needed

### Long-term Actions (ongoing)
- [ ] Implement secret management solution (Vault, etc.)
- [ ] Set up automated secret rotation
- [ ] Enable push protection if not enabled
- [ ] Review and update custom patterns
- [ ] Quarterly audit of all secrets
- [ ] Document lessons learned
```

**Enlaces:**
- https://docs.github.com/en/code-security/secret-scanning/managing-alerts-from-secret-scanning
- https://docs.github.com/en/code-security/secret-scanning/secret-scanning-partnership-program

---

## 2.6 Personalizar el comportamiento de Secret Scanning

### Configurar destinatarios de alertas

**A nivel de repositorio:**

```
Settings → Code security and analysis
  → Secret scanning
    → Alert notifications
        ├─ [✓] Email notifications
        │   └─ Recipients:
        │       ├─ Repository administrators (default)
        │       ├─ Security managers
        │       └─ Custom: security@company.com
        ├─ [✓] Web notifications
        └─ [✓] Integrations
            ├─ Slack: #security-alerts
            ├─ PagerDuty: Security-Oncall
            └─ Webhook: https://api.company.com/security/webhooks
```

**Granular access control:**

```yaml
# Via GitHub API
# Grant read access to non-admin team

PUT /repos/:owner/:repo/teams/:team_slug
{
  "permission": "pull"  # read access
}

# Grant secret scanning access specifically
PUT /repos/:owner/:repo/teams/:team_slug/security-managers
{
  "team_id": 12345
}

# Via GitHub CLI
gh api \
  --method PUT \
  -H "Accept: application/vnd.github+json" \
  /repos/OWNER/REPO/collaborators/USERNAME \
  -f permission='maintain'  # Can view and dismiss alerts
```

**Roles y permisos detallados:**

| Action | Read | Triage | Write | Maintain | Admin | Security Manager |
|--------|------|--------|-------|----------|-------|------------------|
| View alerts | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ |
| Comment on alerts | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ |
| Dismiss alerts | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ |
| Reopen alerts | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ |
| Configure scanning | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ |
| View audit log | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ |

**Notification routing avanzado:**

```javascript
// GitHub App / Webhook handler
// Route alerts based on secret type

const routingRules = {
  github_pat: {
    notify: ['#eng-leads', 'security@company.com'],
    severity: 'high',
    oncall: false
  },
  aws_access_key: {
    notify: ['#cloud-team', '#security-oncall'],
    severity: 'critical',
    oncall: true,
    pagerduty: 'cloud-security'
  },
  stripe_api_key: {
    notify: ['#payments-team', 'cfo@company.com'],
    severity: 'critical',
    oncall: true
  },
  generic_password: {
    notify: ['#security-alerts'],
    severity: 'medium',
    oncall: false
  }
};

app.post('/webhooks/secret-scanning', async (req, res) => {
  const alert = req.body.alert;
  const routing = routingRules[alert.secret_type] || routingRules.generic_password;
  
  // Send to Slack
  for (const channel of routing.notify) {
    await slack.post(channel, {
      text: `🔐 Secret detected: ${alert.secret_type}`,
      severity: routing.severity,
      repo: alert.repository.full_name,
      validity: alert.validity
    });
  }
  
  // Page oncall if critical
  if (routing.oncall && alert.validity === 'active') {
    await pagerduty.createIncident({
      title: `Active ${alert.secret_type} leaked`,
      service: routing.pagerduty,
      urgency: 'high'
    });
  }
  
  res.sendStatus(200);
});
```

### Excluir archivos del escaneo

**Opción 1: .gitignore (no suficiente)**

```bash
# ⚠️ .gitignore NO afecta a secret scanning
# Secret scanning escanea TODO el historial, incluso archivos ignorados

# .gitignore
secrets.json
.env
```

**Opción 2: Path exclusions (Repository level)**

```yaml
# .github/secret_scanning.yml
# Excluir paths específicos

paths-ignore:
  - 'tests/**'
  - 'test/**'
  - '__tests__/**'
  - '**/*.test.js'
  - '**/*.spec.ts'
  - 'docs/**'
  - '**/*.md'
  - 'examples/**'
  - 'node_modules/**'  # Usually auto-ignored
  - 'vendor/**'
  - '.github/workflows/**'  # Si usas secretos de ejemplo en workflows
```

**Opción 3: File-level exclusions (Custom patterns)**

```yaml
# Excluir archivos específicos por nombre
# Settings → Secret scanning → Custom patterns

# Pattern name: Internal test fixture
# Pattern:  TEST_API_KEY_[A-Z0-9]{32}
# Exclude paths:
#   - tests/fixtures/sample.json
#   - tests/integration/*.test.js
```

**Opción 4: Inline comments (Limited support)**

```javascript
// Algunos patterns pueden ser suprimidos con comentarios
// NO TODOS LOS SCANNERS lo soportan

// gitleaks:ignore
const API_KEY = "test_key_12345";

// secret-scanner-ignore
const PASSWORD = "example_password";

// ⚠️ Esto NO funciona para GitHub secret scanning nativo
// Solo funciona para algunas herramientas terceras
```

**Opción 5: Organization-level exclusions**

```yaml
# Organization Settings → Secret scanning
# → Path exclusions (aplica a todos los repos)

Global exclusions:
  - '**/test/**'
  - '**/tests/**'
  - '**/__tests__/**'
  - '**/*.test.*'
  - '**/*.spec.*'
  - '**/examples/**'
  - '**/docs/**'
```

**Best practices para exclusiones:**

```yaml
✅ EXCLUIR:
  - Test fixtures con datos fake
  - Documentación con ejemplos
  - Vendored dependencies (ya escaneadas)
  - Generated files (build artifacts)

❌ NO EXCLUIR:
  - Production code
  - Configuration files
  - Scripts de deployment
  - Infrastructure as code
  - CI/CD workflows (a menos que sea solo ejemplos)

⚠️ CAUTION:
  - Over-excluding crea blind spots
  - Revisar exclusiones trimestralmente
  - Document WHY cada exclusion existe
```

**Verificar efectividad de exclusiones:**

```bash
# Test if exclusion works
# 1. Commit un secreto falso en path excluido
echo "test_key_12345" > tests/fixtures/fake-secret.json
git add tests/fixtures/fake-secret.json
git commit -m "test: Add test fixture"
git push

# 2. Verificar que NO se genera alerta
gh api /repos/:owner/:repo/secret-scanning/alerts | \
  jq '.[] | select(.locations[].path | contains("tests/fixtures"))'

# 3. Si aparece alerta = exclusion no funcionó
# 4. Si NO aparece = exclusion efectiva ✅
```

### Habilitar Custom Patterns

**¿Por qué custom patterns?**

GitHub patterns cubren 200+ tipos de secretos comunes, pero NO cubren:
- ❌ Internal API keys de sistemas propietarios
- ❌ Legacy authentication schemes
- ❌ Custom encryption keys
- ❌ Organization-specific token formats

**Requisitos:**

- ✅ GitHub Advanced Security (Secret Protection)
- ✅ Organization owner o Security manager role
- ✅ Knowledge de regex patterns

**Creación de custom pattern paso a paso:**

**Step 1**: Identificar el pattern

```regex
# Ejemplo: Internal API key
# Format: ACME-[PROJECT]-[A-Z0-9]{32}
# Examples:
#   ACME-WEB-ABC123DEF456GHI789JKL012MNO345PQ
#   ACME-API-XYZ987UVW654TSR321QPO098NML765KJ

# Regex:
ACME-[A-Z]{3,10}-[A-Z0-9]{32}
```

**Step 2**: Test the pattern

```
GitHub UI → Organization Settings
  → Code security and analysis
    → Secret scanning
      → Custom patterns
        → [New pattern]

Pattern details:
  ├─ Name: ACME Internal API Key
  ├─ Secret format (regex):
  │   ACME-[A-Z]{3,10}-[A-Z0-9]{32}
  │
  ├─ Test strings:
  │   ✅ ACME-WEB-ABC123DEF456GHI789JKL012MNO345PQ
  │   ✅ ACME-MOBILE-XYZ987UVW654TSR321QPO098NML76
  │   ❌ ACME-WEB-SHORT
  │   ❌ acme-web-lowercase
  │
  └─ [Publish pattern]
```

**Step 3**: Advanced regex features

```regex
# Before secret (context):
^[\s]*(?:api[_-]?key|token)[\s]*[=:]["']?

# Secret pattern:
ACME-[A-Z]{3,10}-[A-Z0-9]{32}

# After secret (context):
["']?[\s]*$

# Full pattern:
(?:api[_-]?key|token)[\s]*[=:]["']?ACME-[A-Z]{3,10}-[A-Z0-9]{32}["']?

# Matches:
# ✅ API_KEY="ACME-WEB-ABC..."
# ✅ token: 'ACME-API-XYZ...'
# ✅ apiKey = "ACME-MOBILE-..."
# ❌ Just "ACME-WEB-..." by itself (no context)
```

**Step 4**: Dry run mode

```yaml
# Start with "alert only" mode
# Don't enable push protection yet

Pattern settings:
  ├─ Push protection: [ ] Disabled (first)
  ├─ Alert on matches: [✓] Enabled
  └─ Test period: 2 weeks

After 2 weeks:
  ├─ Review all alerts
  ├─ Calculate false positive rate
  ├─ If FP < 10%: Enable push protection
  └─ If FP > 10%: Refine pattern
```

**Ejemplos de custom patterns útiles:**

**1. Database connection strings:**
```regex
# Pattern:
(postgres|mysql|mongodb)://[a-zA-Z0-9_-]+:[^@\s]+@[^/\s]+

# Matches:
# postgres://user:password@host:5432/db
# mysql://admin:secret@192.168.1.1:3306/mydb
# mongodb://dbuser:dbpass@mongo.example.com:27017/prod
```

**2. JWT tokens:**
```regex
# Pattern:
eyJ[A-Za-z0-9_-]{10,}\.eyJ[A-Za-z0-9_-]{10,}\.[A-Za-z0-9_-]{10,}

# Matches:
# eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIx...
```

**3. Private keys:**
```regex
# Pattern:
-----BEGIN (RSA |EC )?PRIVATE KEY-----[^-]*-----END (RSA |EC )?PRIVATE KEY-----

# Matches:
# -----BEGIN RSA PRIVATE KEY-----
# MIIEpAIBAAKCAQEA...
# -----END RSA PRIVATE KEY-----
```

**4. API keys con checksums:**
```regex
# Pattern:
sk_[a-z]{4}_[a-zA-Z0-9]{24,99}

# Matches Stripe keys:
# sk_live_51H7...(varies)
# sk_test_51H7...(varies)
```

**Pattern testing workflow:**

```python
# test_custom_patterns.py
import re

def test_pattern(pattern, test_cases):
    """Test custom pattern against known cases"""
    compiled = re.compile(pattern)
    
    results = {
        'true_positives': [],
        'false_negatives': [],
        'false_positives': []
    }
    
    for case in test_cases:
        match = compiled.search(case['text'])
        
        if case['should_match'] and match:
            results['true_positives'].append(case)
        elif case['should_match'] and not match:
            results['false_negatives'].append(case)
        elif not case['should_match'] and match:
            results['false_positives'].append(case)
    
    return results

# Test cases
ACME_PATTERN = r'ACME-[A-Z]{3,10}-[A-Z0-9]{32}'

test_cases = [
    {
        'text': 'API_KEY="ACME-WEB-ABC123DEF456GHI789JKL012MNO345PQ"',
        'should_match': True,
        'note': 'Valid ACME API key'
    },
    {
        'text': 'const token = "ACME-MOBILE-XYZ987UVW654TSR321QPO098";',
        'should_match': True,
        'note': 'Mobile app token'
    },
    {
        'text': 'ACME-WEB-SHORT',
        'should_match': False,
        'note': 'Too short'
    },
    {
        'text': '<!-- Example: ACME-PROJECT-ABC123DEF456GHI789JKL012MNO345PQ -->',
        'should_match': False,
        'note': 'In comment, likely example'
    }
]

results = test_pattern(ACME_PATTERN, test_cases)

print(f"True Positives: {len(results['true_positives'])}")
print(f"False Negatives: {len(results['false_negatives'])}")
print(f"False Positives: {len(results['false_positives'])}")

# Ideal: TP high, FN low, FP very low
# Acceptable: FP rate < 10%
```

**Managing custom patterns at scale:**

```yaml
# Terraform configuration for custom patterns
# Infraestructure as Code

resource "github_organization_secret_scanning_pattern" "acme_api_key" {
  pattern = "ACME-[A-Z]{3,10}-[A-Z0-9]{32}"
  name    = "ACME Internal API Key"
  
  # Optional: antes/después de secret para contexto
  before_secret = "^[\\s]*(?:api[_-]?key|token)[\\s]*[=:]"
  after_secret  = "[\\s]*$"
  
  # Dry run primero
  enabled = true
  push_protection_enabled = false  # Enable después de testing
}

resource "github_organization_secret_scanning_pattern" "internal_jwt" {
  pattern = "eyJ[A-Za-z0-9_-]{10,}\\.eyJ[A-Za-z0-9_-]{10,}\\.[A-Za-z0-9_-]{10,}"
  name    = "Internal JWT Token"
  enabled = true
}

# Apply:
# terraform apply
# Review:
# terraform plan
```

**Monitoring custom patterns:**

```bash
# Get all alerts for custom patterns
gh api /orgs/:org/secret-scanning/alerts \
  --jq '.[] | select(.secret_type_display_name | contains("ACME")) | {
    number,
    secret_type: .secret_type_display_name,
    state,
    resolution,
    created_at,
    repository: .repository.full_name
  }'

# Metrics to track:
# - Total alerts generated
# - False positive rate
# - Time to resolution
# - Bypass requests
# - Developer feedback
```

**Best practices:**

```yaml
✅ DO:
  - Start specific, broaden gradually
  - Test thoroughly before enabling push protection
  - Document pattern purpose and examples
  - Review quarterly for effectiveness
  - Engage developers in pattern design
  - Monitor false positive rates

❌ DON'T:
  - Create overly broad patterns
  - Enable push protection without testing
  - Ignore developer feedback on false positives
  - Set and forget
  - Copy patterns from internet without testing
```

**Enlaces:**
- https://docs.github.com/en/code-security/secret-scanning/defining-custom-patterns-for-secret-scanning
- https://docs.github.com/en/code-security/secret-scanning/managing-alerts-from-secret-scanning/viewing-alerts

---
