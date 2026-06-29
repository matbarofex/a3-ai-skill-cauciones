# Changelog

Formato basado en Keep a Changelog y versionado SemVer.

Referencias de versión (fuente de verdad):
- **Versión de la skill:** `SKILL.md` → frontmatter `metadata.version` (línea 7).
- **Versión del manual/API:** `SKILL.md` → frontmatter `metadata.source-doc` (línea 8, v1.67).

## [Unreleased]

## [1.1.0] - 2026-06-29

### Changed

- Cambio de criterio de lados en asignación, give-up y derivación
- Se deja de documentar Side 5/6 (Baja Tomador/Colocador) en cauciones
- Cancelación por operación de lado opuesto (G↔F) con mismo ExecID, cuenta, instrumento, precio y cantidad
- Nuevos ejemplos TrdType 3 (asignación) y 61 (give-up)
- Corrección ejemplo derivación TrdType 49

### Added

- Nuevo endpoint `AccountDetails`

- Ejemplo POST garantía FCI con campos cuotapartista

### Fixed

- Corrección parámetro GET `NewCollateralReport` (`CollRptID`)

- Corrección semántica `DepositaryAccountList` (`marketAccount`)

- `NewCollateralReport` POST: `Qty` como string con decimales

- Campos condicionales FCI en `NewCollateralReport` POST

- Frecuencia recomendada `MarginBalance` (≤ 1 req/min)

## [1.0.0] - 2026-05-27

### Added

- `SKILL.md` como punto de entrada (scope, reglas, endpoints, constantes y gotchas).
- `references/reference.md` con parámetros y campos por endpoint.
- `references/diccionario-campos.md` con diccionario consolidado de campos (técnico + lectura operativa).
- `references/glosario.md` con definiciones de negocio.
- `references/buenas-practicas.md` con horarios, polling y throttling.
- `references/errores-http.md` con manejo de errores HTTP (GET/POST).
- `references/examples.md` con payloads y ejemplos operativos.
- `references/circuito-cauciones-explicado.md` con descripción funcional del circuito por método.
- `README.md` con descripción, instalación (rutas por cliente) y canales de soporte/issues.
