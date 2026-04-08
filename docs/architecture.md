# 🏗️ Arquitectura General del Sistema

> **Versión:** 1.0.0 | **Source of Truth v1.0.0**

---

## 1. Visión General

El sistema está diseñado como un ecosistema de gobernanza centralizada con autonomía distribuida. El **Source of Truth** actúa como la fuente única de verdad que define estándares, mientras que cada repositorio vinculado los implementa de forma independiente.

```
┌─────────────────────────────────────────────────────────────┐
│                    SOURCE OF TRUTH (Central)                 │
│                                                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐  │
│  │   Standards  │  │  Governance  │  │  Pipeline Base   │  │
│  │  - Behavior  │  │  - Policies  │  │  - Validate      │  │
│  │  - Quality   │  │  - Security  │  │  - Test          │  │
│  │  - Testing   │  │  - Approval  │  │  - Security      │  │
│  │  - Docs      │  │    Rules     │  │  - Build         │  │
│  └──────────────┘  └──────────────┘  │  - Deploy        │  │
│                                       └──────────────────┘  │
│  ┌──────────────────────────────────────────────────────┐   │
│  │               Agente Auditor                          │   │
│  │  Audita repos → Detecta desviaciones → Propone PRs   │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
              │                    │                │
              ▼                    ▼                ▼
    ┌──────────────┐    ┌──────────────┐    ┌──────────────┐
    │   Repo A     │    │   Repo B     │    │   Repo C     │
    │              │    │              │    │              │
    │ .sot.yml     │    │ .sot.yml     │    │ .sot.yml     │
    │ pipeline.yml │    │ pipeline.yml │    │ pipeline.yml │
    │ agent-cfg.yml│    │ agent-cfg.yml│    │              │
    └──────────────┘    └──────────────┘    └──────────────┘
```

---

## 2. Componentes del Sistema

### 2.1 Source of Truth (Este repositorio)

| Componente | Descripción | Ruta |
|------------|-------------|------|
| Estándar de agentes | Protocolo obligatorio de comportamiento | `agents/behavior-standard.md` |
| Prompts base | Prompts reutilizables para agentes | `agents/base-prompts/` |
| Métricas de evaluación | Cómo medir el desempeño de agentes | `agents/evaluation/` |
| Políticas de gobernanza | Reglas del ecosistema | `governance/` |
| Estándares de calidad | Calidad de código, tests, documentación | `standards/` |
| Configuración central | Configuración del ecosistema | `config/` |
| Plantillas | Templates para nuevos repos y agentes | `templates/` |
| Pipelines | Workflows de GitHub Actions | `.github/workflows/` |

### 2.2 Repositorios Vinculados

Cada repositorio del ecosistema debe incluir:

```
repo-vinculado/
├── .source-of-truth.yml      # Referencia al Source of Truth
├── .github/
│   └── workflows/
│       └── pipeline.yml      # Hereda del pipeline base
├── agents/ (si aplica)
│   └── <agent-id>/
│       ├── agent-config.yml  # Configuración del agente
│       └── ...
└── ...
```

### 2.3 Agente Auditor

El Agente Auditor es el componente central de gobernanza automática:

```
┌─────────────────────────────────────────────┐
│              AGENTE AUDITOR                 │
│                                             │
│  Trigger (schedule/push/PR)                 │
│       │                                     │
│       ▼                                     │
│  1. Inventario del repositorio              │
│       │                                     │
│       ▼                                     │
│  2. Análisis de conformidad                 │
│     ├─ Config Source of Truth               │
│     ├─ Pipeline CI/CD                       │
│     ├─ Seguridad                            │
│     ├─ Calidad de código                    │
│     └─ Gobernanza de agentes                │
│       │                                     │
│       ▼                                     │
│  3. Detección de desviaciones               │
│       │                                     │
│       ▼                                     │
│  4. Generación de propuestas                │
│       │                                     │
│       ▼                                     │
│  5. Validación (confianza >= 0.85?)         │
│     ├─ Sí → Crear PR automático             │
│     └─ No → Crear issue para revisión       │
│                                             │
│  NUNCA hace merge sin aprobación humana     │
└─────────────────────────────────────────────┘
```

---

## 3. Flujo de Trabajo de un Agente

Todo agente sigue el patrón de iteración en cascada definido en el estándar:

```
┌─────────────────────────────────────────────────────────┐
│                PATRÓN DE TRABAJO DEL AGENTE             │
│                                                         │
│  ENTRADA                                                │
│    │                                                    │
│    ▼                                                    │
│  ANÁLISIS ──── Comprender problema, identificar         │
│    │           restricciones y métricas de éxito        │
│    ▼                                                    │
│  HIPÓTESIS ─── Generar ≥ 3 hipótesis distintas          │
│    │                                                    │
│    ▼                                                    │
│  EVALUACIÓN ── Datos + Evidencia + Contexto             │
│    │           Puntuación 0.0 - 1.0 por hipótesis       │
│    ▼                                                    │
│  VALIDACIÓN ── Tests / Código / Simulación              │
│    │           Verificación empírica                    │
│    ▼                                                    │
│  COMPARACIÓN ─ Contrastar resultados validados          │
│    │                                                    │
│    ▼                                                    │
│  SELECCIÓN ─── Mejor hipótesis con justificación        │
│    │                                                    │
│    ▼                                                    │
│  ¿Mejora posible? ─── Sí → ITERACIÓN (max 5)           │
│    │                                                    │
│    │ No (convergencia)                                  │
│    ▼                                                    │
│  AUTOEVALUACIÓN ── Verificar completitud y precisión    │
│    │                                                    │
│    ▼                                                    │
│  SALIDA FINAL                                           │
└─────────────────────────────────────────────────────────┘
```

---

## 4. Pipeline de CI/CD

```
Push / PR
    │
    ▼
┌────────────┐    ┌────────────┐    ┌────────────┐
│  VALIDATE  │───▶│    TEST    │───▶│  SECURITY  │
│            │    │            │    │            │
│ • Linting  │    │ • Unit     │    │ • Secrets  │
│ • Type check│   │ • Integr.  │    │ • SAST     │
│ • YAML     │    │ • Coverage │    │ • Deps     │
│ • SoT conf │    │ • Reports  │    │ • Perms    │
└────────────┘    └────────────┘    └────────────┘
                                          │
                                          ▼
                                    ┌────────────┐
                                    │   BUILD    │
                                    │            │
                                    │ • Compile  │
                                    │ • Package  │
                                    │ • Docker   │
                                    └────────────┘
                                          │
                        ┌─────────────────┘
                        │
                        ▼
               ┌─────────────────┐
               │ AGENT EVALUATE  │  (solo si hay agentes)
               │                 │
               │ • Config valid  │
               │ • Schema valid  │
               │ • Metrics check │
               └─────────────────┘
                        │
                        ▼
               ┌─────────────────┐
               │  DEPLOY STAGING │  (en push a main)
               │                 │
               │ • Deploy        │
               │ • Smoke tests   │
               └─────────────────┘
                        │
                        ▼
               ┌─────────────────┐
               │ DEPLOY PROD     │  (requiere aprobación)
               │                 │
               │ • Deploy        │
               │ • Health check  │
               │ • Tag release   │
               └─────────────────┘
                        │
                        ▼
               ┌─────────────────┐
               │  GOVERNANCE     │  (siempre)
               │                 │
               │ • Audit log     │
               │ • SoT check     │
               └─────────────────┘
```

---

## 5. Niveles de Autonomía de Agentes

```
L0 ──── Solo lectura y análisis
  │     No puede crear PRs ni comentar con cambios
  │
L1 ──── Sugerencias
  │     Puede crear PRs (requiere aprobación para merge)
  │     Max 5 archivos por PR
  │
L2 ──── Cambios en staging
  │     Puede deployar a staging
  │     Puede solicitar cambios en PRs
  │     Max 20 archivos por PR
  │
L3 ──── Con supervisión en producción
  │     Puede deployar a producción (con aprobación)
  │     Max 50 archivos por PR
  │
L4 ──── Infraestructura
        Cambios de infraestructura (doble aprobación)
        Max 100 archivos por PR
```

---

## 6. Modelo de Datos

### 6.1 Respuesta de agente (validation-schema.json)

```json
{
  "agent_id": "unique-agent-id",
  "run_id": "run-uuid",
  "timestamp": "2026-04-08T08:30:00Z",
  "task": "descripción de la tarea",
  "analysis": {
    "problem_understood": "...",
    "constraints": ["..."],
    "success_metrics": ["..."]
  },
  "hypotheses": [
    {"id": "H1", "description": "...", "score": 0.9, "evidence": "...", "validated": true},
    {"id": "H2", "description": "...", "score": 0.7, "evidence": "...", "validated": true},
    {"id": "H3", "description": "...", "score": 0.5, "evidence": "...", "validated": true}
  ],
  "selected_solution": {
    "hypothesis_id": "H1",
    "justification": "...",
    "confidence": 0.9
  },
  "result": "respuesta final",
  "iterations": 2,
  "self_evaluation": {
    "accuracy": 0.92,
    "limitations": "...",
    "requirements_met": true
  }
}
```

---

## 7. Integración con Herramientas Externas

| Herramienta | Uso | Configuración |
|-------------|-----|---------------|
| GitHub Actions | Pipeline de CI/CD | `.github/workflows/` |
| Dependabot | Actualización de dependencias | `.github/dependabot.yml` |
| CodeQL | Análisis estático de seguridad | `codeql-config.yml` |
| Gitleaks | Detección de secretos | `.gitleaks.toml` |
| Codecov | Reportes de cobertura | `codecov.yml` |
| SonarQube (opcional) | Calidad de código | `sonar-project.properties` |

---

## 8. Escalabilidad

El sistema está diseñado para escalar con el crecimiento del ecosistema:

- **Nuevos repositorios**: Se vinculan añadiendo `.source-of-truth.yml`
- **Nuevos agentes**: Se registran con `agent-config.yml` y pasan validación
- **Nuevos estándares**: Se añaden como versiones minor/major con migración gradual
- **Nuevos entornos**: Se configuran en `config/pipeline-config.yml`
- **Nuevas herramientas**: Se integran como pasos adicionales en el pipeline base
