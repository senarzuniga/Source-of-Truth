# 🔗 Guía de Integración

> **Versión:** 1.0.0 | **Source of Truth v1.0.0**

Esta guía explica cómo vincular un repositorio existente o nuevo al Source of Truth y heredar automáticamente sus estándares.

---

## 1. Prerrequisitos

Antes de integrar un repositorio:

- [ ] Tener acceso de administrador al repositorio
- [ ] Tener acceso al Source of Truth repository
- [ ] Revisar la [Arquitectura del sistema](architecture.md) para entender el modelo
- [ ] Revisar el [Estándar de comportamiento de agentes](../agents/behavior-standard.md) si el repo incluye agentes

---

## 2. Integración Paso a Paso

### Paso 1: Crear el archivo de configuración

Copia la plantilla y adapta los valores:

```bash
curl -sSL https://raw.githubusercontent.com/senarzuniga/Source-of-Truth/main/templates/repo-config.yml \
  -o .source-of-truth.yml
```

Edita `.source-of-truth.yml` con los datos de tu repositorio:

```yaml
source_of_truth:
  repository: "https://github.com/senarzuniga/Source-of-Truth"
  version: "1.0.0"
  registered_at: "2026-04-08"  # Fecha actual

repository:
  name: "mi-repositorio"
  description: "Descripción de mi proyecto"
  team: "mi-equipo"
  owner: "mi-usuario"
  type: "service"
  language: "python"
```

### Paso 2: Configurar el pipeline

Crea `.github/workflows/pipeline.yml` referenciando el pipeline base.

**Opción A: Heredar el pipeline completo** (recomendado para nuevos proyectos)

```yaml
# .github/workflows/pipeline.yml
name: "CI/CD Pipeline"

on:
  push:
    branches: [main, "release/**"]
  pull_request:
    branches: [main]

# Hereda del pipeline base del Source of Truth
# Copia el contenido de:
# https://raw.githubusercontent.com/senarzuniga/Source-of-Truth/main/.github/workflows/main-pipeline.yml
# y adapta los jobs que necesites
```

**Opción B: Pipeline mínimo** (para proyectos simples)

```yaml
name: "CI/CD Pipeline"

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

permissions:
  contents: read
  checks: write
  pull-requests: write

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: "Validate .source-of-truth.yml"
        run: |
          if [ ! -f ".source-of-truth.yml" ]; then
            echo "❌ Missing .source-of-truth.yml"
            exit 1
          fi
          echo "✅ .source-of-truth.yml found"

  test:
    runs-on: ubuntu-latest
    needs: [validate]
    steps:
      - uses: actions/checkout@v4
      # Añade tus pasos de testing aquí

  security:
    runs-on: ubuntu-latest
    needs: [validate]
    permissions:
      contents: read
      security-events: write
    steps:
      - uses: actions/checkout@v4
      # Añade tus pasos de seguridad aquí
```

### Paso 3: Configurar protección de ramas

En los ajustes de tu repositorio (Settings → Branches), protege la rama `main`:

```
Branch protection rules para 'main':
✅ Require a pull request before merging
✅ Require status checks to pass before merging
   - ci/validate
   - ci/test
✅ Require branches to be up to date before merging
✅ Do not allow bypassing the above settings
❌ Allow force pushes (desactivado)
❌ Allow deletions (desactivado)
```

### Paso 4: Configurar Dependabot

Crea `.github/dependabot.yml`:

```yaml
version: 2
updates:
  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "weekly"

  # Añade tu ecosistema de dependencias:
  - package-ecosystem: "npm"      # o "pip", "maven", "cargo", etc.
    directory: "/"
    schedule:
      interval: "weekly"
    open-pull-requests-limit: 10
```

### Paso 5: Verificar la integración

Ejecuta el Agente Auditor manualmente para verificar la conformidad inicial:

```bash
# En GitHub Actions: Actions → Governance & Audit → Run workflow
# Selecciona audit_type: "compliance_only"
```

O verifica manualmente que:

```bash
# Verificar que los archivos obligatorios existen
ls -la .source-of-truth.yml README.md CHANGELOG.md .github/workflows/

# Verificar sintaxis de .source-of-truth.yml
python3 -c "import yaml; yaml.safe_load(open('.source-of-truth.yml'))"
echo "✅ YAML válido"
```

---

## 3. Integración para Repositorios con Agentes

Si tu repositorio incluye agentes de IA, sigue estos pasos adicionales:

### Paso 1: Crear configuración del agente

```bash
mkdir -p agents/mi-agente
curl -sSL https://raw.githubusercontent.com/senarzuniga/Source-of-Truth/main/templates/agent-config.yml \
  -o agents/mi-agente/agent-config.yml
```

Edita el archivo con los datos de tu agente:

```yaml
agent:
  id: "mi-agente-v1"
  name: "Mi Agente Especializado"
  version: "1.0.0"
  type: "task"
  domain: "backend"
  
  standard:
    source_of_truth_version: "1.0.0"
    behavior_standard: "1.0.0"
    last_validated: "2026-04-08"
  
  autonomy:
    level: "L1"  # Empieza en L1, sube tras validación
```

### Paso 2: Implementar el prompt base

Copia y extiende el prompt base:

```python
# agents/mi-agente/prompts.py
SYSTEM_PROMPT = """
{base_prompt}  # Heredar de agents/base-prompts/agent-base.md

## ESPECIALIZACIÓN: Mi Agente Backend

Eres un agente especializado en desarrollo backend con Python/FastAPI.

### Herramientas disponibles:
- read_file: Lee archivos del repositorio
- search_code: Busca en el código
- create_pr: Crea Pull Requests con mejoras

### Restricciones adicionales:
- Solo modifica archivos en src/ y tests/
- No modifica archivos de infraestructura o configuración de CI
"""
```

### Paso 3: Implementar el esquema de respuesta

Tu agente debe validar sus respuestas contra el schema:

```python
import jsonschema
import json
import urllib.request

# Cargar el schema del Source of Truth
SCHEMA_URL = "https://raw.githubusercontent.com/senarzuniga/Source-of-Truth/main/agents/evaluation/validation-schema.json"

def validate_agent_response(response: dict) -> None:
    """Valida que la respuesta del agente cumple el estándar."""
    with urllib.request.urlopen(SCHEMA_URL) as f:
        schema = json.load(f)
    jsonschema.validate(response, schema)
```

### Paso 4: Habilitar evaluación en el pipeline

En `.source-of-truth.yml`, activa la evaluación de agentes:

```yaml
pipeline:
  stages:
    agent_evaluate: true  # Cambiar a true
```

---

## 4. Migración de Repositorios Existentes

Para repositorios que ya tienen pipelines propios:

### Evaluación inicial

1. Ejecuta la auditoría de conformidad manualmente
2. Revisa el reporte generado
3. Prioriza los items críticos y altos

### Plan de migración recomendado

| Semana | Actividades |
|--------|-------------|
| 1 | Añadir `.source-of-truth.yml` y configurar pipeline base |
| 2 | Implementar protección de ramas y Dependabot |
| 3 | Corregir desviaciones críticas y altas de seguridad |
| 4 | Alcanzar cobertura mínima de tests (80%) |
| 5+ | Iteración continua hacia conformidad completa |

### Solicitar excepción temporal

Si no puedes cumplir un estándar en el tiempo requerido:

```yaml
# En .source-of-truth.yml
exceptions:
  - standard: "testing.min_coverage"
    reason: "Repositorio legacy con 50,000+ líneas de código sin tests"
    approved_by: "architect-username"
    approved_at: "2026-04-08"
    expires_at: "2026-06-08"
    migration_plan: "Añadir tests al ritmo de 10% mensual"
    issue: "https://github.com/org/repo/issues/123"
```

---

## 5. Mantenimiento

### Actualizar la versión del estándar

Cuando el Source of Truth publique una nueva versión:

1. Revisa el CHANGELOG del Source of Truth
2. Identifica los cambios que afectan a tu repositorio
3. Actualiza `.source-of-truth.yml` con la nueva versión
4. Aplica los cambios requeridos por la nueva versión
5. Ejecuta la auditoría para verificar conformidad

```yaml
# .source-of-truth.yml
source_of_truth:
  version: "1.1.0"  # Actualizar cuando salga nueva versión
```

### Recibir actualizaciones automáticas

El Agente Auditor genera PRs automáticos cuando:
- El Source of Truth publica una nueva versión
- Se detectan desviaciones en tu repositorio
- Las dependencias tienen vulnerabilidades

Revisa y aprueba estos PRs periódicamente.

---

## 6. Troubleshooting

### El pipeline falla en "Validate .source-of-truth.yml"

```bash
# Verificar que el YAML es válido
python3 -c "import yaml; yaml.safe_load(open('.source-of-truth.yml')); print('OK')"

# Verificar campos requeridos
python3 -c "
import yaml
with open('.source-of-truth.yml') as f:
    c = yaml.safe_load(f)
required = ['source_of_truth', 'repository', 'standards']
print('Missing:', [k for k in required if k not in c])
"
```

### El agente auditor detecta demasiadas desviaciones

Prioriza en este orden:
1. 🔴 **Critical** — Corregir inmediatamente
2. 🟠 **High** — Corregir en menos de 7 días
3. 🟡 **Medium** — Corregir en menos de 30 días
4. 🔵 **Low** — Planificar para el siguiente sprint

### El score de conformidad es bajo

Revisa las categorías con más peso:
- `security` — Peso alto, afecta fuertemente el score
- `pipeline` — Peso alto, sin pipeline el score cae rápido
- `config` — Peso medio, configuración básica requerida

---

## 7. Soporte

Para preguntas o problemas de integración:

1. Revisa la [documentación de arquitectura](architecture.md)
2. Consulta los [estándares de comportamiento](../agents/behavior-standard.md)
3. Abre un issue en el [Source of Truth repository](https://github.com/senarzuniga/Source-of-Truth)
4. Usa el label `integration-help` para preguntas de integración
