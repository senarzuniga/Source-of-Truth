# 🔐 Política de Seguridad

> **Versión:** 1.0.0 | **Clasificación:** Crítica — Cumplimiento obligatorio

---

## 1. Principios de Seguridad

El ecosistema opera bajo los siguientes principios de seguridad:

1. **Mínimo privilegio** — Cada agente y pipeline recibe solo los permisos mínimos necesarios
2. **Defensa en profundidad** — Múltiples capas de control de seguridad
3. **Zero trust** — Ningún componente es confiable por defecto
4. **Aprobación humana obligatoria** — Cambios críticos requieren revisión humana
5. **Auditoría completa** — Todo acceso y cambio queda registrado

---

## 2. Gestión de Secretos

### 2.1 Reglas obligatorias

- ❌ **PROHIBIDO**: Hardcodear secretos, tokens o contraseñas en el código
- ❌ **PROHIBIDO**: Almacenar secretos en variables de entorno en texto claro
- ❌ **PROHIBIDO**: Loggear secretos o tokens (ni parcialmente)
- ✅ **OBLIGATORIO**: Usar GitHub Secrets para todos los secretos en pipelines
- ✅ **OBLIGATORIO**: Rotar secretos al menos cada 90 días
- ✅ **OBLIGATORIO**: Usar `secrets: inherit` solo cuando sea estrictamente necesario

### 2.2 Herramientas de detección

El pipeline verifica automáticamente:
- `gitleaks` — Detección de secretos en commits
- `truffleHog` — Análisis de historial de git
- `detect-secrets` — Pre-commit hook de detección

### 2.3 Respuesta ante exposición de secretos

Si se detecta un secreto expuesto:
1. **Revocar inmediatamente** el secreto/token expuesto
2. **Notificar** al equipo de seguridad
3. **Investigar** el alcance de la exposición
4. **Rotar** todos los secretos relacionados
5. **Generar** un incident report (P0)

---

## 3. Permisos de GitHub Actions

### 3.1 Permisos por defecto (restrictivo)

Todos los workflows deben declarar permisos explícitamente:

```yaml
permissions:
  contents: read    # Solo lectura por defecto
```

### 3.2 Permisos adicionales (justificados)

Solo se otorgan permisos adicionales cuando son necesarios:

| Permiso | Uso permitido | Requiere justificación |
|---------|---------------|----------------------|
| `contents: write` | Publicar releases | Sí |
| `pull-requests: write` | Crear/actualizar PRs | Sí |
| `issues: write` | Crear issues de auditoría | Sí |
| `packages: write` | Publicar paquetes | Sí |
| `id-token: write` | OIDC auth (cloud deploy) | Sí |
| `actions: write` | Gestionar workflows | Solo admins |

### 3.3 Tokens de agentes

- Los tokens de agentes tienen scopes específicos y limitados
- Los tokens expiran automáticamente (máximo 24h)
- Los tokens se generan via OIDC cuando sea posible (no secrets estáticos)

---

## 4. Protección de Ramas

### 4.1 Configuración obligatoria para rama principal

```yaml
branch_protection:
  branch: main
  required_status_checks:
    strict: true
    contexts:
      - "ci/validate"
      - "ci/test"
      - "ci/security"
  required_pull_request_reviews:
    required_approving_review_count: 1
    dismiss_stale_reviews: true
    require_code_owner_reviews: false
  enforce_admins: false
  restrictions: null
  allow_force_pushes: false
  allow_deletions: false
```

### 4.2 Ramas protegidas

| Rama | Protección requerida |
|------|---------------------|
| `main` | PRs con aprobación + tests passing |
| `staging` | PRs + tests passing |
| `production` | PRs + aprobación + tests passing + security scan |

---

## 5. Análisis de Vulnerabilidades

### 5.1 Frecuencia

| Tipo de análisis | Frecuencia |
|-----------------|------------|
| Dependencias (Dependabot) | Continuo |
| SAST (análisis estático) | En cada PR |
| Secrets scan | En cada commit |
| DAST (análisis dinámico) | Semanal en staging |
| Penetration testing | Trimestral |

### 5.2 Respuesta a CVEs

| Severidad | Tiempo máximo de corrección |
|-----------|----------------------------|
| Critical (CVSS >= 9.0) | 24 horas |
| High (CVSS 7.0-8.9) | 7 días |
| Medium (CVSS 4.0-6.9) | 30 días |
| Low (CVSS < 4.0) | 90 días |

### 5.3 Herramientas de análisis

```yaml
security_tools:
  dependency_scan:
    - tool: "dependabot"
      config: ".github/dependabot.yml"
  sast:
    - tool: "codeql"
      languages: ["javascript", "python", "go", "java"]
  secrets:
    - tool: "gitleaks"
      config: ".gitleaks.toml"
  container:
    - tool: "trivy"
      severity: ["CRITICAL", "HIGH"]
```

---

## 6. Seguridad de Agentes de IA

### 6.1 Principios específicos de seguridad para agentes

- Los agentes **no pueden** autoescalar sus propios permisos
- Los agentes **no pueden** acceder a secretos fuera de su contexto
- Los agentes **no pueden** ejecutar código arbitrario en producción
- Los agentes **deben** ser auditables en cada acción
- Los agentes **deben** respetar los límites de rate limiting

### 6.2 Sandboxing de agentes

```yaml
agent_sandbox:
  network_access: "restricted"    # Solo dominios permitidos
  file_system: "read_only"        # Solo lectura por defecto
  execution_timeout: 3600         # 1 hora máximo
  max_api_calls_per_run: 1000
  allowed_tools:
    - "read_file"
    - "search_code"
    - "create_pr"
    - "create_issue"
  blocked_tools:
    - "delete_repo"
    - "manage_secrets"
    - "modify_branch_protection"
```

### 6.3 Revisión de outputs de agentes

Antes de aplicar cualquier cambio propuesto por un agente:
1. Revisar el diff completo del PR
2. Verificar que no hay cambios no relacionados con la tarea
3. Confirmar que no hay código malicioso o backdoors
4. Aprobar explícitamente antes del merge

---

## 7. Respuesta a Incidentes de Seguridad

### 7.1 Contacto de seguridad

Para reportar vulnerabilidades de seguridad, **no abrir issues públicos**.
Usar el canal de seguridad privado del repositorio (Security Advisories).

### 7.2 Playbook de respuesta

```
1. DETECTAR      → Alertas automáticas, reporte humano
2. CONTENER      → Revocar accesos, deshabilitar componente afectado
3. EVALUAR       → Determinar alcance e impacto
4. ERRADICAR     → Eliminar la causa raíz
5. RECUPERAR     → Restaurar servicio de forma segura
6. POST-MORTEM   → Documentar lecciones aprendidas
7. MEJORAR       → Actualizar controles de seguridad
```

### 7.3 Comunicación

| Audiencia | Cuándo comunicar | Canal |
|-----------|-----------------|-------|
| Equipo técnico | Inmediatamente | Canal de incidentes |
| Management | En P0/P1 | Escalación directa |
| Usuarios afectados | Tras contención | Notificación oficial |
| Público | Si aplica (datos) | Disclosure responsable |
