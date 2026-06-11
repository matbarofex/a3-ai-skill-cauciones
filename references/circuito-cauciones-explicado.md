# Circuito Cauciones — lectura por endpoint (sin flujo obligatorio)

Este documento describe qué aporta cada endpoint al ciclo de vida de cauciones.  
No define un paso a paso obligatorio: cada proveedor implementa los métodos que necesita según su producto y arquitectura.

## Criterio general

- `examples.md` es la fuente única de payloads.
- Este archivo explica para qué sirve cada método dentro del ciclo de vida.
- Se puede implementar por módulos, no necesariamente en secuencia lineal.

## Contexto operativo del ciclo de vida

- En rueda (10:00 a 17:00) se generan operaciones.
- Post 17:00 se ejecutan procesos de compensación y liquidación, y se consolidan datos de derechos, márgenes y garantías.
- No hay eventos push: el seguimiento es por polling GET/POST.

**Referencia de implementación:** `examples.md` -> `1. Cliente HTTP mínimo (pseudocódigo)`.

## Endpoints de operaciones

### `TradeCaptureReport`

Sirve para obtener operaciones y detectar cuáles corresponden a cauciones (`SegmentId=CAUC`, `CFICode=RPXXXX`, `CAU-`*).  
Es la base para trazabilidad operativa (`TradeID/TradeNumber`, `ExecID`, `Side`, `Account`, fechas e importes).

Para el caso tomador, permite identificar la operación tomadora en pesos y su vencimiento.

**Ejemplos:** `examples.md` -> `2. Operaciones del día (Cauciones)` (`Tomadora`, `Colocadora`, `Par tomador + derivación`).

## Endpoints de costos y riesgo

### `AccruedFees` (+ referencia de tarifa en `Fee`)

Sirve para consultar derechos de mercado asociados a la operatoria.  
Funcionalmente se usa para conciliación económica por operación/ejecución.

**Ejemplo:** `examples.md` -> `3. Derechos de mercado (AccruedFees)`.

### `MarginRequirementReport`

Sirve para ver requerimientos de márgenes por posición completa de cuenta, no por trade individual.  
El análisis se hace por finalidad (`Reference`), por ejemplo `Cauciones $` o `Cauciones U$S`.

**Ejemplo:** `examples.md` -> `4. Requerimientos de márgenes (MarginRequirementReport)`.

## Endpoints de elegibilidad de garantías

### `CollateralList`

Sirve para conocer qué activos pueden integrarse como garantía y bajo qué condiciones.  
Acá se obtiene `InternalInstrumentCode` y `Haircut` (aforo: porcentaje del valor que computa como garantía).

**Ejemplo:** `examples.md` -> `5. Activos para garantías (CollateralList)`.

### `AccountDetails`

Sirve para consultar el detalle de una cuenta de registro (CUIT, razón social, cuenta de neteo/compensación, tipo de cuenta).  
No tiene relación específica con Cauciones; cada proveedor lo usa según necesidad de maestro de comitentes.

**Ejemplo:** `examples.md` -> `11. Detalle de cuenta (AccountDetails)`.

### `DepositaryAccountList`

Sirve para resolver cuentas depositarias disponibles antes de enviar instrucciones de garantías.  
`marketAccount=true` devuelve cuentas del mercado; `marketAccount=false` solo las del ALYC/agente.

**Ejemplo:** `examples.md` -> `10. Cuentas depositarias (DepositaryAccountList)`.

## Endpoints de instrucciones de garantías

### `NewCollateralReport` (POST)

Sirve para alta de instrucciones de ingreso (`Side=1`) o egreso (`Side=2`) de activos en garantía.  
Se usa con idempotencia (`ExternalCollRptID`) y con finalidad (`CollAppIType`) según cauciones pesos/dólares/supletorias.

**Ejemplos:**

- `examples.md` -> `6. Ingreso garantía Cauciones $`
- `examples.md` -> `6b. Ingreso garantía FCI`
- `examples.md` -> `7. Egreso de garantía Cauciones $`

### `NewCollateralReport` (GET)

Sirve para consultar estado de la instrucción (proceso asíncrono) por parámetro `CollRptID`.  
Estados observados del ciclo: `Inicial`, `Confirmado`, `Aprobado Riesgos`, `Procesado`, `Ejecutado`, `Anulado`.

**Ejemplos:**

- `examples.md` -> `Estado de instrucción (GET) — ingreso`
- `examples.md` -> `Estado de instrucción (GET) — egreso`
- `examples.md` -> `Estado de instrucción (GET) — anulación/rechazo`

## Endpoints de resultado y conciliación de garantías

### `MT536`

Sirve para ver movimientos ejecutados de activos (ingreso/egreso).  
Es la evidencia operativa de ejecución de la instrucción.

**Ejemplo:** `examples.md` -> `8. Movimientos de activos (MT536)`.

### `MT506`

Sirve para consultar stock consolidado de activos en garantía (foto final por cuenta/finalidad).  
Se usa para validar cantidad, valuación y aforo aplicado después de movimientos.

**Ejemplo:** `examples.md` -> `9. Stock de activos (MT506)`.

## Cómo usar este documento en una implementación real

- Si necesitás solo registrar operaciones, podés implementar `TradeCaptureReport` y conciliación económica.
- Si además cubrís riesgo, sumás `MarginRequirementReport` + `CollateralList`.
- Si también integrás movimientos de garantía, incorporás `NewCollateralReport` (POST/GET), `MT536` y `MT506`.

En todos los casos, los payloads y casos concretos se toman de `examples.md`.