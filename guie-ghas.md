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
