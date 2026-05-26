---
name: a3-ai-skill-cauciones
description: Migra o extiende integración BackOffice existente con API Post-Trade A3 para Cauciones (TradeCaptureReport, garantías por finalidad, márgenes). Usar cuando el proveedor ya consume la API para futuros/opciones/TV y debe adaptar a XMAB/CAUC, o ante A3, Cauciones, migración Matba, MAE, ALyC, MT506, MT536, MarginBalance.
license: Proprietary
metadata:
  author: A3 Mercados
  version: 1.0.0
  source-doc: PrimaryAPI-BO.pdf (v1.67)
  contact: mpi@primary.com.ar
---

# API Post-Trade A3 — Cauciones (backoffice)

Skill para proveedores de backoffice que integran **Cauciones** (doc v1.67, `CAUC`, `XMAB`). Fusión Matba Rofex + MAE → A3 Mercados; migración por etapas; **etapa actual: Cauciones**.

## Integración existente (leer primero)

La mayoría de proveedores **ya consume** la misma API para futuros, opciones, Títulos Valores (TIVA), etc. Cauciones agrega filtros, finalidades y reglas de negocio; **no reimplementar** lo que ya funciona.

1. Aplicar solo los **deltas** de la matriz por endpoint
2. Usar [reference.md](references/reference.md) / [glosario.md](references/glosario.md) para detalle técnico y de negocio.

## Entorno y documentación

| Recurso | Detalle |
|---------|---------|
| **Único ambiente de pruebas** | Demo: `https://demoapi.anywhereportfolio.com.ar` |
| Credenciales demo | `mpi@primary.com.ar` |
| Manual completo de la API con diccionario campos (200+ págs) | [PrimaryAPI-BO.pdf](https://apihub.primary.com.ar/assets/docs/PrimaryAPI-BO.pdf) — **no duplicar en skill**; link + campos Cauciones en [reference.md](references/reference.md) |
| Hub APIs | [apihub.primary.com.ar](https://apihub.primary.com.ar/) |
| Glosario negocio | [glosario.md](references/glosario.md) |


## Horarios y modelo de datos

- **Rueda:** 10:00–17:00 (sesión de negociación).
- **Post 17:00:** procesos **CCP** (compensación/liquidación). Datos definitivos del día: consultar **después** de CCP.
- La API **no envía eventos** al integrador (no hay webhooks): todo es **polling** con GET/POST.
- Fechas: `YYYYMMDD`. Sin fecha en algunos métodos → última info disponible post-CCP.

## Flujo de integración (solo recomendación)

**No es obligatorio.** Cada proveedor usa los métodos que necesite según su producto.

Ejemplo de secuencia posible (sistema sin BO API previo):

```
AuthToken (1/día)
→ TradeCaptureReport (operaciones; ver sección abajo)
→ [opcional] SecurityList, CollateralList, MT506, MT536, MarginBalance,
   MarginRequirementReport, NewCollateralReport, Fee, AccruedFees
```

Horarios y throttling: [buenas-practicas.md](references/buenas-practicas.md). HTTP: [errores-http.md](references/errores-http.md).

## Autenticación

`POST /AuthToken/AuthToken` — body `{ "nombreUsuario", "password" }`. Header `Authorization: <token>` (24 h). **Máximo 1 solicitud de token por día.**

## Constantes Cauciones

Nota: para advertencias críticas de implementación ver sección **Gotchas (leer antes de codear)**.

| Concepto | Dónde aplica | Valor |
|----------|--------------|-------|
| MarketID (consulta) | Solo `TradeCaptureReport` (opcional) | `XMAB` |
| segmentID (consulta) | Solo `TradeCaptureReport` (opcional) | `CAUC` |
| SegmentId / CFICode (respuesta) | Identificar caución en boleta | `CAUC`, `RPXXXX` |
| SecurityID | Instrumento | `CAU-ARS`, `CAU-USD` |
| TrdRptStatus | Operaciones | `0` Definitiva, `3` Anulada |
| Side | Operaciones | `G`/`F`/`5`/`6` — [glosario.md](references/glosario.md) |
| TrdType | Operaciones | `0` Interferencia, `49` Derivación |
| Reference (garantías) | MT506, MT536, CollateralList, márgenes | `Cauciones $`, `Cauciones U$S`, `Supletorias` |
| CollAppIType | `NewCollateralReport` | `3` Cauciones $, `4` Cauciones U$S, `24` Supletorias |

## Gotchas (leer antes de codear)

- **`MarketID` no es regla global:** `XMAB` aplica como filtro en `TradeCaptureReport`; en otros métodos puede aparecer `ROFX` aunque la operatoria sea cauciones.
- **`Account` cambia semántica por endpoint:** en `TradeCaptureReport` es cuenta de registro/comitente; en garantías y márgenes es cuenta de neteo ya que el tipo de información es diferente.
- **`MT536` sin filtro `Classification=7` como patrón:** para cauciones, filtrar por `Reference` (`Cauciones $`, `Cauciones U$S`, `Supletorias`).
- **Datos definitivos del día:** usar información consolidada después de procesos de compensación y liquidación (post 17:00). El horario de finalización de los procesos y publicación de información no es fijo y puede variar, según la duración de los procesos (en base al volumen operado) de la CCP.
- **`MarginBalance` (límite oficial):** `Soporta 1 request cada 5 segundos.`.
- **`NewCollateralReport` POST:** usar catálogo de errores de `references/errores-http.md` y loguear status + body completo ante errores operativos.

## Endpoints

| Método | HTTP | Ruta |
|--------|------|------|
| AuthToken | POST | `/AuthToken/AuthToken` |
| SecurityList | GET | `/PreTrade/SecurityList` |
| TradeCaptureReport | GET | `/PosTrade/TradeCaptureReport` |
| MT506 | GET | `/PosTrade/MT506` |
| MT536 | GET | `/PosTrade/MT536` |
| CollateralList | GET | `/PosTrade/CollateralList` |
| MarginRequirementReport | GET | `/PosTrade/MarginRequirementReport` |
| NewCollateralReport (alta) | **POST** | `/PosTrade/NewCollateralReport` |
| NewCollateralReport (estado) | GET | `/PosTrade/NewCollateralReport` |
| DepositaryAccountList | GET | `/PosTrade/DepositaryAccountList` |
| MarginBalance | GET | `/Risk/MarginBalance` |
| Fee | GET | `/PosTrade/Fee` |
| AccruedFees | GET | `/PosTrade/AccruedFees` |

## Campo `Account` (según método)

| Método | Significado de `Account` |
|--------|---------------------------|
| **TradeCaptureReport** | Código de **cuenta de registro / comitente**. |
| **Garantías** (MT506, MT536, `NewCollateralReport` `Details`, márgenes) | Código de **cuenta de neteo**. |

Una cuenta de registro tiene una cuenta de neteo asociada; en ~99 % coinciden en número, pero el **significado del campo cambia por endpoint**. No unificar en un solo mapper sin discriminar origen.

## TradeCaptureReport (Cauciones)

**Recomendado:** consultar solo con fechas y filtrar cauciones en el backoffice:

```
GET .../TradeCaptureReport?DateFrom=YYYYMMDD&DateTo=YYYYMMDD
```

Identificar cauciones en respuesta: `SegmentId=CAUC`, `CFICode=RPXXXX`, instrumentos `CAU-*`.

**Alternativa válida:** agregar `MarketID=XMAB` y/o `segmentID=CAUC` en la URL si el proveedor prefiere filtrar en origen.

- En rueda: transitorias/anuladas (hasta 12/min) si aplica al mercado.
- Definitivas: 2/día post-CCP (buenas prácticas).
- Derivación: `TrdType=49`, mismo `ExecID` que madre (`TrdType=0`).
- `Account` en boleta = cuenta de registro/comitente.

## MT536 (movimientos de garantías)

**Recomendado:** solo `dateFrom`, `dateTo` y `alyc`. Filtrar en el ERP por `Reference`: `Cauciones $`, `Cauciones U$S`, `Supletorias`.

**No recomendado:** `Classification=7` (existe en el manual; cada proveedor puede usar otro criterio, pero no es el patrón habitual).

## NewCollateralReport

- **POST**: ingreso instrucción (`ExternalCollRptID` idempotente, max 20 chars).
- **GET**: consulta estado (comparte límite 2/min con POST).
- `Side=1` (ingreso) suele usar `OriginType=1`; `Side=2` (egreso) suele usar `OriginType=3`.
- Estados observados en circuito: `Inicial`, `Confirmado`, `Aprobado Riesgos`, `Procesado`, `Ejecutado`, `Anulado`.

## Implementación

1. Token 1/día; renovar solo si 401 o expiración.
2. Respetar throttling → manejar 429 con backoff.
3. GET: solo esperar HTTP 200, 400, 401, 429 (ver [errores-http.md](references/errores-http.md)).
4. POST `NewCollateralReport`: usar catálogo de errores documentado en [errores-http.md](references/errores-http.md).
5. Reconciliación operaciones: `TradeID` + `Account` (registro) + `Side`; derivaciones por `ExecID`.

## Archivos

| Archivo | Uso |
|---------|-----|
| [reference.md](references/reference.md) | Parámetros y campos |
| [diccionario-campos.md](references/diccionario-campos.md) | Diccionario consolidado de campos (técnico + lectura de negocio) |
| [buenas-practicas.md](references/buenas-practicas.md) | Frecuencias y 429 |
| [errores-http.md](references/errores-http.md) | HTTP GET/POST |
| [examples.md](references/examples.md) | Código ejemplo |
| [circuito-cauciones-explicado.md](references/circuito-cauciones-explicado.md) | Narrativa completa del circuito por método |
| [glosario.md](references/glosario.md) | Definiciones negocio |

Soporte: `mpi@primary.com.ar`
