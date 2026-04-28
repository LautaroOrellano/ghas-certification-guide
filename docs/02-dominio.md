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
