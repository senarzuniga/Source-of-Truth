# 🧠 Source of Truth — Central AI Agent & Pipeline Governance

> **Versión:** 1.0.0 | **Estado:** Producción | **Mantenido por:** Arquitectura AI

Este repositorio es la **fuente única de verdad** para todos los agentes de IA, pipelines de CI/CD y estándares de calidad del ecosistema. Todos los repositorios deben referenciar y heredar estos estándares.

---

## 📦 Estructura del repositorio

```
Source-of-Truth/
├── .github/
│   └── workflows/
│       ├── main-pipeline.yml        # Pipeline completo de producción
│       ├── agent-evaluator.yml      # Evaluación continua de agentes
│       └── governance.yml           # Auditoría y gobernanza automática
├── agents/
│   ├── behavior-standard.md         # Estándar de comportamiento de agentes
│   ├── base-prompts/
│   │   ├── agent-base.md            # Prompt base para todos los agentes
│   │   └── auditor-agent.md         # Prompt del agente auditor
│   └── evaluation/
│       ├── metrics.yml              # Métricas de evaluación de agentes
│       └── validation-schema.json   # Esquema de validación de respuestas
├── governance/
│   ├── policies.md                  # Políticas de gobernanza
│   ├── security-policy.md           # Política de seguridad
│   └── approval-rules.yml           # Reglas de aprobación
├── standards/
│   ├── code-quality.md              # Estándares de calidad de código
│   ├── testing-standards.md         # Estándares de testing
│   └── documentation-standards.md  # Estándares de documentación
├── config/
│   ├── source-of-truth.yml          # Configuración central
│   └── pipeline-config.yml          # Configuración del pipeline
├── templates/
│   ├── repo-config.yml              # Plantilla de config para nuevos repos
│   └── agent-config.yml             # Plantilla de config para agentes
└── docs/
    ├── architecture.md              # Arquitectura general del sistema
    ├── integration-guide.md         # Guía de integración
    └── getting-started.md           # Guía de inicio rápido
```

---

## 🚀 Inicio rápido

### Vincular un nuevo repositorio

1. Copia `templates/repo-config.yml` a tu repositorio como `.source-of-truth.yml`
2. Actualiza los valores con la información de tu proyecto
3. Incluye el workflow de referencia en `.github/workflows/`
4. Ejecuta la validación inicial

```bash
# Verificar conformidad con el Source of Truth
curl -sSL https://raw.githubusercontent.com/senarzuniga/Source-of-Truth/main/config/source-of-truth.yml
```

---

## 🔥 Principios fundamentales

Todo agente y repositorio vinculado debe cumplir:

| Principio | Descripción |
|-----------|-------------|
| **Iterativo** | Trabajo en modo cascada, mejora continua |
| **Basado en hipótesis** | Generar múltiples hipótesis antes de decidir |
| **Validado** | Decisiones respaldadas por datos, tests y evidencia |
| **Auditable** | Todo cambio debe ser trazable y revisable |
| **Seguro** | Sin cambios directos sin validación humana en producción |

---

## 📚 Documentación

- [Arquitectura del sistema](docs/architecture.md)
- [Guía de integración](docs/integration-guide.md)
- [Inicio rápido](docs/getting-started.md)
- [Estándar de comportamiento de agentes](agents/behavior-standard.md)
- [Políticas de gobernanza](governance/policies.md)

---

## 🔗 Repositorios vinculados

Los repositorios que referencian este Source of Truth aparecen automáticamente en el registro de gobernanza al configurar `.source-of-truth.yml`.

---

## 📈 Versioning

Este repositorio sigue [Semantic Versioning](https://semver.org/). Los cambios de estándares se publican como releases etiquetados.

| Versión | Fecha | Cambios |
|---------|-------|---------|
| 1.0.0 | 2026-04-08 | Release inicial |
