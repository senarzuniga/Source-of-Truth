# 🏛️ Políticas de Gobernanza

> **Versión:** 1.0.0 | **Obligatorio para todos los repositorios del ecosistema**

---

## 1. Principio de Gobernanza

El Source of Truth establece un sistema de gobernanza descentralizada con control centralizado de estándares. Cada repositorio tiene autonomía operativa, pero debe adherirse a los estándares definidos aquí.

```
Source of Truth (Central)
       │
       ├── Define estándares y políticas
       ├── Publica versiones versionadas
       ├── Audita conformidad automáticamente
       └── Propone mejoras via PRs
       
Repositorios vinculados
       │
       ├── Heredan estándares automáticamente
       ├── Implementan pipeline base
       ├── Reportan métricas al central
       └── Proponen extensiones de estándares
```

---

## 2. Roles y Responsabilidades

### 2.1 Arquitecto del Source of Truth
- Define y aprueba cambios en estándares
- Revisa y aprueba PRs al Source of Truth
- Gestiona el versionado de estándares
- Escala incidentes de seguridad

### 2.2 Propietarios de Repositorio (Repo Owners)
- Mantienen la conformidad de su repositorio con el estándar
- Revisan y aprueban PRs generados por agentes
- Reportan problemas o limitaciones del estándar
- Migran a nuevas versiones del estándar en el plazo establecido

### 2.3 Agentes de IA
- Operan dentro de los permisos asignados
- Generan PRs en lugar de cambios directos
- Documentan cada acción con logs completos
- Se detienen ante incertidumbre o situaciones no contempladas

### 2.4 Agente Auditor
- Audita repositorios según schedule configurado
- Genera reportes de conformidad
- Propone correcciones via PRs
- Escala desviaciones críticas inmediatamente

---

## 3. Flujo de Cambios

### 3.1 Cambios en el Source of Truth

```
Propuesta de cambio
       │
       ▼
   Discusión en Issue
       │
       ▼
   PR con los cambios
       │
       ▼
   Revisión por Arquitecto
       │
       ▼
   Aprobación (2+ revisores para cambios major)
       │
       ▼
   Merge a main
       │
       ▼
   Publicación de nueva versión
       │
       ▼
   Notificación a repos vinculados
       │
       ▼
   PRs automáticos de migración (si aplica)
```

### 3.2 Cambios en repositorios vinculados

Los repositorios deben cumplir la política de su rama principal:
- **main/master**: Solo merges desde PRs aprobados
- **staging**: Merges automáticos permitidos desde PRs de agentes con score >= 0.90
- **develop**: Merges automáticos permitidos con tests passing

---

## 4. Niveles de Conformidad

| Nivel | Score global | Estado | Consecuencia |
|-------|-------------|--------|--------------|
| ✅ Compliant | >= 0.90 | Conforme | Sin acción requerida |
| ⚠️ Warning | 0.75 - 0.89 | Advertencia | Plan de mejora requerido en 14 días |
| 🔴 Non-Compliant | 0.60 - 0.74 | No conforme | Corrección requerida en 7 días |
| ⛔ Critical | < 0.60 | Crítico | Bloqueo inmediato, escalación |

---

## 5. Proceso de Aprobación de Agentes

Todo nuevo agente que se integre al ecosistema debe:

1. **Declarar** su versión de estándar compatible en `agent-config.yml`
2. **Pasar** la suite de validación de comportamiento
3. **Operar** en modo L0/L1 durante 5 ejecuciones iniciales
4. **Ser promovido** a niveles superiores solo tras validación exitosa
5. **Renovar** la validación si cambia de versión o dominio

---

## 6. Gestión de Incidentes

### 6.1 Clasificación de incidentes

| Tipo | Descripción | SLA de respuesta |
|------|-------------|-----------------|
| P0 - Crítico | Agente actuando fuera de permisos, secretos expuestos | 1 hora |
| P1 - Alto | Desviación de seguridad, fallo de pipeline en producción | 4 horas |
| P2 - Medio | Conformidad degradada, métricas por debajo del umbral | 24 horas |
| P3 - Bajo | Advertencias de calidad, documentación desactualizada | 7 días |

### 6.2 Escalación

```
P0/P1: Haltar agente → Notificar Arquitecto → Incident report → Post-mortem
P2: Crear issue → Asignar propietario → PR de corrección → Verificar
P3: Registrar en backlog → Planificar corrección → Verificar en auditoría siguiente
```

---

## 7. Auditoría y Trazabilidad

### 7.1 Qué se registra

Todo cambio en el ecosistema genera un registro con:
- Quién inició el cambio (humano o agente)
- Qué archivos fueron modificados
- Cuándo ocurrió (timestamp ISO 8601)
- Por qué (referencia a issue o tarea)
- Resultado (éxito, fallo, pendiente)

### 7.2 Retención de logs

| Tipo de log | Retención |
|-------------|-----------|
| Logs de agentes | 90 días |
| Logs de pipeline | 30 días |
| Reportes de auditoría | 365 días |
| Logs de incidentes | 2 años |

---

## 8. Cumplimiento y Excepciones

### 8.1 Excepciones permitidas

Un repositorio puede solicitar una excepción temporal a un estándar si:
1. Existe una razón técnica documentada
2. El Arquitecto del Source of Truth aprueba
3. La excepción tiene una fecha de expiración (<= 30 días)
4. Existe un plan de migración aprobado

### 8.2 Proceso de excepción

```
1. Crear issue con label "exception-request"
2. Describir: qué estándar, por qué, cuánto tiempo, plan de migración
3. Revisión y aprobación del Arquitecto
4. Registrar en governance/exceptions.yml (si aplica)
5. Revisión automática al vencer el plazo
```

---

## 9. Evolución del Source of Truth

Este estándar es un documento vivo. Para proponer cambios:

1. Abre un issue con el label `standard-proposal`
2. Describe el problema actual y la mejora propuesta
3. Incluye impacto estimado en repos vinculados
4. El Arquitecto revisa y agenda para discusión
5. Los cambios aprobados se implementan en la siguiente versión minor o major

Los cambios **retrocompatibles** → versión minor (1.x.0)
Los cambios **incompatibles** → versión major (x.0.0)
