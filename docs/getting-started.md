# 🚀 Guía de Inicio Rápido

> **Source of Truth v1.0.0** — Empieza en menos de 15 minutos

---

## ¿Qué es el Source of Truth?

El Source of Truth es la **fuente única de verdad** para todos los agentes de IA, pipelines de CI/CD y estándares de calidad del ecosistema. Cuando tu repositorio se vincula aquí, hereda automáticamente:

- ✅ Estándares de comportamiento para agentes
- ✅ Pipeline de CI/CD listo para producción
- ✅ Políticas de seguridad y gobernanza
- ✅ Estándares de calidad de código y tests
- ✅ Auditoría automática continua

---

## Inicio Rápido (5 minutos)

### 1. Crea el archivo de configuración

```bash
# Desde la raíz de tu repositorio
cat > .source-of-truth.yml << 'EOF'
source_of_truth:
  repository: "https://github.com/senarzuniga/Source-of-Truth"
  version: "1.0.0"
  registered_at: "$(date +%Y-%m-%d)"

repository:
  name: "mi-repositorio"
  description: "Mi proyecto"
  owner: "mi-usuario"
  type: "service"
  language: "python"  # python|javascript|go|java

standards:
  code_quality:
    enabled: true
    min_coverage: 80
  testing:
    enabled: true
  documentation:
    enabled: true
  security:
    enabled: true

pipeline:
  stages:
    validate: true
    test: true
    security: true
    build: true
    agent_evaluate: false
    deploy_staging: false
    deploy_production: false

auditor:
  enabled: true
EOF
echo "✅ .source-of-truth.yml creado"
```

### 2. Añade el pipeline de CI

```bash
mkdir -p .github/workflows
```

Copia el pipeline base del Source of Truth a `.github/workflows/pipeline.yml`:

```bash
curl -sSL \
  https://raw.githubusercontent.com/senarzuniga/Source-of-Truth/main/.github/workflows/main-pipeline.yml \
  -o .github/workflows/pipeline.yml
```

### 3. Configura Dependabot

```bash
cat > .github/dependabot.yml << 'EOF'
version: 2
updates:
  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "weekly"
EOF
```

### 4. Haz commit y push

```bash
git add .source-of-truth.yml .github/
git commit -m "chore: link repository to Source of Truth v1.0.0"
git push
```

### 5. Verifica en GitHub Actions

Ve a la pestaña **Actions** de tu repositorio y verifica que el pipeline se ejecuta correctamente.

---

## Inicio con Agentes (10 minutos adicionales)

Si tu repositorio incluye agentes de IA, sigue estos pasos adicionales:

### 1. Crea la configuración del agente

```bash
mkdir -p agents/mi-agente
curl -sSL \
  https://raw.githubusercontent.com/senarzuniga/Source-of-Truth/main/templates/agent-config.yml \
  -o agents/mi-agente/agent-config.yml

# Edita agent-config.yml con los datos de tu agente
# Minimum required changes:
# - agent.id: unique ID
# - agent.name: descriptive name
# - agent.standard.source_of_truth_version: "1.0.0"
# - agent.autonomy.level: L0|L1|L2|L3|L4
```

### 2. Habilita la evaluación de agentes

En `.source-of-truth.yml`:
```yaml
pipeline:
  stages:
    agent_evaluate: true  # Cambiar a true
```

### 3. Implementa el protocolo de respuesta

Tu agente debe generar respuestas que cumplan el [schema de validación](../agents/evaluation/validation-schema.json):

```python
# Ejemplo mínimo de respuesta válida
response = {
    "agent_id": "mi-agente-v1",
    "run_id": "uuid-aqui",
    "timestamp": "2026-04-08T10:00:00Z",
    "task": "Descripción de la tarea realizada",
    "analysis": {
        "problem_understood": "Entendí que el problema es...",
        "constraints": ["sin dependencias externas", "max 100 líneas"],
        "success_metrics": ["tests passing", "cobertura > 80%"]
    },
    "hypotheses": [
        {
            "id": "H1",
            "description": "Enfoque A: implementar usando biblioteca X",
            "score": 0.9,
            "evidence": "Tiene soporte oficial y documentación completa",
            "validated": True,
            "validation_method": "code_execution"
        },
        {
            "id": "H2",
            "description": "Enfoque B: implementar desde cero",
            "score": 0.6,
            "evidence": "Mayor control pero más tiempo de desarrollo",
            "validated": True,
            "validation_method": "simulation"
        },
        {
            "id": "H3",
            "description": "Enfoque C: usar servicio externo",
            "score": 0.4,
            "evidence": "Introduce dependencia externa no deseada",
            "validated": True,
            "validation_method": "data_analysis"
        }
    ],
    "selected_solution": {
        "hypothesis_id": "H1",
        "justification": "Mayor score por evidencia sólida y menor riesgo",
        "confidence": 0.9
    },
    "result": "La solución implementada es...",
    "iterations": 2,
    "self_evaluation": {
        "accuracy": 0.92,
        "limitations": "No cubre el caso edge de X",
        "future_improvements": "Podría mejorar con Y en la próxima iteración",
        "requirements_met": True
    }
}
```

---

## Estructura recomendada de tu repositorio

```
mi-repositorio/
├── .source-of-truth.yml     ← NUEVO: vinculación al Source of Truth
├── .github/
│   ├── dependabot.yml        ← NUEVO: actualización automática de deps
│   └── workflows/
│       └── pipeline.yml      ← NUEVO: pipeline heredado
├── agents/ (si aplica)
│   └── mi-agente/
│       ├── agent-config.yml  ← NUEVO: configuración del agente
│       └── ...
├── src/
│   └── ...
├── tests/
│   ├── unit/
│   └── integration/
├── README.md                 ← Actualizar con referencia al SoT
├── CHANGELOG.md              ← NUEVO si no existe
└── ...
```

---

## Checklist de integración

Verifica que completaste todos los pasos:

- [ ] `.source-of-truth.yml` creado y válido
- [ ] Pipeline de CI configurado (validate + test + security como mínimo)
- [ ] `dependabot.yml` configurado
- [ ] Protección de ramas configurada en GitHub
- [ ] README.md actualizado con referencia al Source of Truth
- [ ] CHANGELOG.md existe
- [ ] Primera ejecución del pipeline exitosa
- [ ] Auditoría inicial ejecutada (sin issues críticos)
- [ ] `agent-config.yml` creado (si aplica)

---

## Recursos adicionales

| Recurso | Descripción |
|---------|-------------|
| [Arquitectura](architecture.md) | Cómo funciona el sistema completo |
| [Guía de integración](integration-guide.md) | Integración detallada paso a paso |
| [Estándar de agentes](../agents/behavior-standard.md) | Protocolo de comportamiento |
| [Estándar de calidad](../standards/code-quality.md) | Métricas y herramientas de calidad |
| [Políticas de gobernanza](../governance/policies.md) | Reglas del ecosistema |
| [Política de seguridad](../governance/security-policy.md) | Requisitos de seguridad |

---

## Preguntas frecuentes

**¿Puedo usar el Source of Truth en un repositorio privado?**
Sí. El archivo `.source-of-truth.yml` hace referencia al Source of Truth público, pero tu repositorio puede ser privado.

**¿Qué pasa si no puedo cumplir un estándar?**
Solicita una excepción temporal añadiendo el bloque `exceptions` en `.source-of-truth.yml`. Las excepciones deben tener fecha de expiración y plan de migración.

**¿Con qué frecuencia debo actualizar la versión del estándar?**
Tienes hasta 30 días desde la publicación de una nueva versión major para migrar. Las versiones minor son retrocompatibles.

**¿El Agente Auditor puede modificar mi código directamente?**
No. El Agente Auditor solo crea Pull Requests con propuestas. Nunca hace merge sin aprobación humana.

**¿Puedo desactivar la auditoría automática?**
Sí, en `.source-of-truth.yml` establece `auditor.enabled: false`. Sin embargo, el score de conformidad se verá afectado.
