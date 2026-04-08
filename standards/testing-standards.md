# 🧪 Estándares de Testing

> **Versión:** 1.0.0 | **Aplica a:** Todos los repositorios del ecosistema

---

## 1. Pirámide de Testing

El ecosistema sigue la pirámide de testing estándar:

```
         /\
        /E2E\          (5-10% de tests, lentos, costosos)
       /------\
      /Integra-\       (20-30% de tests, velocidad media)
     /  ción    \
    /------------\
   /  Unit Tests  \    (60-75% de tests, rápidos, baratos)
  /________________\
```

---

## 2. Tipos de Tests

### 2.1 Tests unitarios

**Propósito:** Verificar el comportamiento de unidades individuales de código.

**Características obligatorias:**
- Rápidos (< 100ms por test)
- Independientes (sin dependencias externas)
- Deterministas (mismo resultado siempre)
- Aislados (usan mocks/stubs para dependencias)

**Cobertura mínima:** 80% de líneas y ramas

```yaml
unit_tests:
  speed: "< 100ms per test"
  isolation: "full mocking of external dependencies"
  coverage_minimum: 80
  naming_convention: "test_<what>_when_<condition>_then_<expected>"
  run_on:
    - "every_commit"
    - "pre_push"
    - "pull_request"
```

### 2.2 Tests de integración

**Propósito:** Verificar la interacción entre componentes del sistema.

**Características obligatorias:**
- Usar datos de prueba controlados
- Limpiar el estado después de cada test
- Probar rutas happy path y rutas de error
- Verificar contratos de APIs

```yaml
integration_tests:
  speed: "< 5s per test"
  isolation: "real dependencies, test database"
  coverage_minimum: 60
  run_on:
    - "pull_request"
    - "main_branch_push"
```

### 2.3 Tests End-to-End (E2E)

**Propósito:** Verificar flujos completos de usuario desde la interfaz hasta el backend.

**Características obligatorias:**
- Cubrir los flujos críticos del negocio
- Ejecutar en entorno similar a producción
- Generar screenshots/videos en fallos
- Ser estables (sin flakiness)

```yaml
e2e_tests:
  tool: "playwright"  # o cypress
  speed: "< 30s per test"
  run_on:
    - "pull_request_to_main"
    - "staging_deploy"
  retries: 2
  record_on_failure: true
```

### 2.4 Tests de rendimiento

**Propósito:** Verificar que el sistema cumple los requisitos de rendimiento bajo carga.

```yaml
performance_tests:
  tool: "k6"  # o locust, jmeter
  run_on: "weekly"  # o pre_release
  thresholds:
    p95_response_time: "< 500ms"
    p99_response_time: "< 1000ms"
    error_rate: "< 1%"
    throughput: "> 100 rps"
```

### 2.5 Tests de seguridad

Ver [security-policy.md](../governance/security-policy.md) para detalles.

---

## 3. Testing de Agentes de IA

Los agentes requieren una categoría adicional de tests:

### 3.1 Tests de comportamiento

Verifican que el agente sigue el protocolo estándar:

```python
def test_agent_generates_minimum_hypotheses():
    """El agente debe generar al menos 3 hipótesis."""
    response = agent.process("Solve problem X")
    assert len(response.hypotheses) >= 3

def test_agent_validates_all_hypotheses():
    """Todas las hipótesis deben ser validadas."""
    response = agent.process("Solve problem X")
    for hypothesis in response.hypotheses:
        assert hypothesis.validated == True

def test_agent_performs_self_evaluation():
    """El agente debe realizar autoevaluación."""
    response = agent.process("Solve problem X")
    assert response.self_evaluation is not None
    assert response.self_evaluation.accuracy > 0
```

### 3.2 Tests de conformidad con el estándar

```python
def test_response_conforms_to_schema():
    """La respuesta debe cumplir el schema de validación."""
    response = agent.process("Solve problem X")
    validate(response.to_dict(), validation_schema)  # No lanza excepción

def test_agent_respects_permission_boundaries():
    """El agente no debe exceder sus permisos."""
    # Intentar acción fuera del nivel de autonomía
    with pytest.raises(PermissionError):
        agent.deploy_to_production_without_approval()
```

### 3.3 Tests de calidad de output

```python
def test_selected_solution_is_best_hypothesis():
    """La solución seleccionada debe tener el mayor score."""
    response = agent.process("Solve problem X")
    max_score = max(h.score for h in response.hypotheses)
    selected = next(h for h in response.hypotheses
                    if h.id == response.selected_solution.hypothesis_id)
    assert selected.score == max_score
```

---

## 4. Herramientas de Testing por Lenguaje

| Lenguaje | Unit | Integration | E2E | Coverage |
|----------|------|------------|-----|----------|
| Python | pytest | pytest + testcontainers | playwright | coverage.py |
| JS/TS | jest/vitest | jest + supertest | playwright/cypress | istanbul |
| Go | testing | testify | playwright | go test -cover |
| Java | JUnit 5 | Spring Test | selenium | jacoco |

---

## 5. Configuración de CI para Tests

```yaml
# Ejemplo de configuración de jobs de testing en GitHub Actions
test_matrix:
  unit:
    timeout_minutes: 10
    fail_fast: false

  integration:
    timeout_minutes: 20
    requires: ["unit"]
    services:
      - "postgres"
      - "redis"

  e2e:
    timeout_minutes: 30
    requires: ["integration"]
    environment: "staging"

  performance:
    timeout_minutes: 60
    schedule: "weekly"
    environment: "staging"
```

---

## 6. Gestión de Test Data

### 6.1 Reglas de datos de prueba

- ❌ No usar datos reales de producción en tests
- ❌ No hardcodear IDs o valores que cambien entre entornos
- ✅ Usar factories o fixtures para generar datos
- ✅ Limpiar datos creados durante tests
- ✅ Usar datos representativos de casos reales

### 6.2 Ambientes de testing

```yaml
environments:
  local:
    description: "Entorno del desarrollador"
    data: "mocks + local database"
    external_services: "mocked"

  ci:
    description: "Entorno de CI/CD"
    data: "generated fixtures"
    external_services: "test instances"

  staging:
    description: "Entorno pre-producción"
    data: "anonymized production snapshot"
    external_services: "real test accounts"
```

---

## 7. Reporte de Tests

El pipeline genera automáticamente:

```yaml
test_reports:
  formats:
    - junit_xml      # Para integración con CI
    - html           # Para revisión humana
    - coverage_xml   # Para análisis de cobertura

  publish:
    - "github_checks"
    - "pr_comment"
    - "artifacts"

  alerts:
    on_coverage_drop: true
    on_flaky_test: true
    on_new_failure: true
```

---

## 8. Tests como Documentación

Los tests deben ser legibles como documentación:

```python
# BIEN: El test documenta el comportamiento esperado
def test_user_login_with_invalid_password_returns_401():
    """
    Dado un usuario registrado
    Cuando se intenta login con contraseña incorrecta
    Entonces debe retornar 401 Unauthorized
    """
    user = create_user(email="test@example.com", password="correct_password")
    response = client.post("/auth/login", json={
        "email": "test@example.com",
        "password": "wrong_password"
    })
    assert response.status_code == 401
    assert "Invalid credentials" in response.json()["message"]

# MAL: Test que no documenta comportamiento
def test_login():
    r = client.post("/auth/login", json={"email": "t@t.com", "p": "x"})
    assert r.status_code == 401
```
