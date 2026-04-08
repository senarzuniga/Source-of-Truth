# 📋 Estándares de Calidad de Código

> **Versión:** 1.0.0 | **Aplica a:** Todos los repositorios del ecosistema

---

## 1. Principios de Calidad

El código del ecosistema debe ser:

- **Legible** — Cualquier desarrollador puede entenderlo sin contexto adicional
- **Mantenible** — Puede ser modificado con seguridad y confianza
- **Testeable** — Tiene cobertura de tests adecuada
- **Seguro** — No introduce vulnerabilidades conocidas
- **Eficiente** — Optimizado sin sacrificar claridad

---

## 2. Métricas de Calidad Mínimas

| Métrica | Umbral mínimo | Umbral óptimo |
|---------|---------------|---------------|
| Cobertura de tests | 80% | 95% |
| Complejidad ciclomática | <= 10 por función | <= 5 |
| Longitud de funciones | <= 50 líneas | <= 30 |
| Longitud de archivos | <= 500 líneas | <= 300 |
| Duplicación de código | <= 5% | 0% |
| Score de mantenibilidad | >= B | A |
| Deuda técnica | <= 5% | <= 1% |

---

## 3. Estándares por Lenguaje

### 3.1 Python

```yaml
python:
  linter: "ruff"
  formatter: "black"
  type_checker: "mypy"
  min_python_version: "3.11"
  config_files:
    - "pyproject.toml"
    - ".ruff.toml"
  rules:
    - max_line_length: 88  # black default
    - require_type_hints: true
    - require_docstrings: "public_methods"
    - no_bare_except: true
    - no_mutable_defaults: true
```

### 3.2 JavaScript / TypeScript

```yaml
javascript_typescript:
  linter: "eslint"
  formatter: "prettier"
  type_checker: "typescript"  # TS projects
  min_node_version: "20"
  config_files:
    - ".eslintrc.json"
    - ".prettierrc"
    - "tsconfig.json"
  rules:
    - no_any: "warn"  # TS projects
    - require_return_types: true
    - no_console_in_prod: true
    - prefer_const: true
```

### 3.3 Go

```yaml
go:
  linter: "golangci-lint"
  formatter: "gofmt"
  min_go_version: "1.22"
  config_files:
    - ".golangci.yml"
  rules:
    - require_error_handling: true
    - no_global_variables: "warn"
    - require_context: "public_functions"
```

### 3.4 Cualquier lenguaje

```yaml
universal:
  - no_hardcoded_secrets: true
  - no_hardcoded_urls: "warn"
  - no_commented_out_code: "warn"
  - no_todo_without_issue: "warn"
  - consistent_naming: true
  - max_function_params: 5
```

---

## 4. Estándares de Commits

Todos los commits deben seguir **Conventional Commits**:

```
<tipo>(<ámbito>): <descripción corta>

[cuerpo opcional]

[footer opcional]
```

### Tipos válidos

| Tipo | Uso |
|------|-----|
| `feat` | Nueva funcionalidad |
| `fix` | Corrección de bug |
| `docs` | Solo documentación |
| `style` | Formato, sin cambios lógicos |
| `refactor` | Refactorización sin feat/fix |
| `perf` | Mejora de rendimiento |
| `test` | Agregar o corregir tests |
| `build` | Sistema de build o dependencias |
| `ci` | Cambios en CI/CD |
| `chore` | Mantenimiento |
| `revert` | Revertir commit anterior |

### Ejemplos

```
feat(auth): add OAuth2 login support
fix(api): handle null response in user endpoint
ci(pipeline): add security scanning step
docs(readme): update integration guide
```

---

## 5. Estándares de Pull Requests

### 5.1 Tamaño de PRs

| Tamaño | Líneas | Estado |
|--------|--------|--------|
| Micro | < 50 | ✅ Ideal |
| Pequeño | 50-200 | ✅ Aceptable |
| Mediano | 200-500 | ⚠️ Considerar dividir |
| Grande | 500-1000 | 🔴 Debe dividirse |
| Épico | > 1000 | ❌ Rechazado |

### 5.2 Template de PR

Todo PR debe incluir:
- Descripción del problema que resuelve
- Descripción de los cambios realizados
- Tipo de cambio (feature/fix/refactor/etc.)
- Tests incluidos o razón de exclusión
- Checklist de calidad completado

### 5.3 Checklist de PR (obligatorio)

```markdown
## Checklist
- [ ] El código sigue los estándares del Source of Truth
- [ ] Los tests pasan localmente
- [ ] Se ha añadido o actualizado la documentación
- [ ] No hay secretos o datos sensibles en el código
- [ ] Los cambios son retrocompatibles (o se indica el breaking change)
- [ ] El PR tiene un tamaño razonable (< 500 líneas)
```

---

## 6. Estándares de Branches

### 6.1 Nomenclatura

```
feature/<issue-id>-<descripcion-corta>
fix/<issue-id>-<descripcion-corta>
hotfix/<issue-id>-<descripcion-corta>
release/<version>
audit/<date>-<issue-slug>
chore/<descripcion-corta>
```

### 6.2 Reglas de branches

- Las ramas `feature/*` se crean desde `main`
- Las ramas `hotfix/*` se crean desde `main` o `production`
- Las ramas `release/*` se crean desde `main`
- No se hacen commits directos a `main` ni `production`

---

## 7. Revisión de Código

### 7.1 Qué buscar en una revisión

1. **Correctness** — El código hace lo que se supone que debe hacer
2. **Security** — No introduce vulnerabilidades
3. **Performance** — No hay regresiones de rendimiento obvias
4. **Maintainability** — El código es comprensible y mantenible
5. **Tests** — La cobertura es adecuada y los tests son significativos
6. **Standards** — Cumple los estándares del Source of Truth

### 7.2 Tiempo de revisión

| Tamaño del PR | SLA de revisión |
|---------------|-----------------|
| Micro/Pequeño | 4 horas |
| Mediano | 8 horas |
| Grande | 24 horas |

---

## 8. Métricas de Deuda Técnica

El pipeline mide y reporta automáticamente:

```yaml
technical_debt:
  tools:
    - "sonarqube"   # Score de mantenibilidad
    - "codeclimate" # Complejidad y duplicación
  
  thresholds:
    new_code_coverage: 80
    overall_coverage: 70
    maintainability_rating: "B"
    reliability_rating: "A"
    security_rating: "A"
    security_review_rating: "A"
    
  gates:
    block_pr_if_quality_gate_fails: true
    block_pr_if_new_bugs: true
    block_pr_if_new_vulnerabilities: true
    block_pr_if_coverage_drops_below: 80
```
