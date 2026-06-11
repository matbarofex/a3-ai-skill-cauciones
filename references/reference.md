# Referencia API Post-Trade — Cauciones (A3 v1.67)

**Ambiente único:** `https://demoapi.anywhereportfolio.com.ar`

Envelope éxito: `{ "Status", "Code", "Value" }`. HTTP GET: 200, 400, 401, 429 — ver [errores-http.md](errores-http.md).

### `Reference` — garantías (cauciones)

Valores habituales en MT506, MT536, CollateralList, márgenes: **`Cauciones $`**, **`Cauciones U$S`**, **`Supletorias`**.

---

## AuthToken

```
POST /AuthToken/AuthToken
```

| Parámetro | Ubicación | Tipo | Req |
|-----------|-----------|------|-----|
| nombreUsuario | query o body | string | sí |
| password | query o body | string | sí |

---

## SecurityList

```
GET /PreTrade/SecurityList
```

| Parámetro | Req | Valores relevantes Cauciones |
|-----------|-----|------------------------------|
| SecurityIDSource | sí | `4` ISIN, `M` Market, `H` Clearing House |
| MarketID | sí | `XMAB` |
| SecurityExchange | no | `XMAB` |
| SecurityType | no | `TD` (Time Deposit / caución en ejemplos) |
| SecurityGroup | no | ej. `Caución Pesos`, `Caución Dólares` |

Respuesta: `MarketSegmentID: "Caución"`, `SegmentID: "CAUC"`, instrumentos `CAU-ARS`, `CAU-USD`, `CFICode: RPXXXX`.

---

## AccountDetails

```
GET /PreTrade/AccountDetails
```

Consulta unitaria de una cuenta de registro. **Único parámetro:** `accountCode`. Sin relación específica con el flujo de Cauciones.

| Parámetro | Tipo | Req | Descripción |
|-----------|------|-----|-------------|
| accountCode | string | sí | Código de la Cuenta de Registro |

HTTP esperados: **200** (OK), **400** (bad request). También aplican 401/429 — ver [errores-http.md](errores-http.md).

Campos principales de respuesta: `AccountCode`, `Account`, `CompensationAccountCode`, `CompensationAccount`, `NettingAccountCode`, `PartyId`, `CreationDate`, `ClearingMemberCode`, `ClearingMember`, `AccountType`, `UnderlyingOwner`, `AccountRegisterType`.

Estructuras anidadas:

- **`PartySubGrp`**: array de grupos; cada ítem se interpreta por `PartySubIDSource` (ver [diccionario-campos.md](diccionario-campos.md) sección AccountDetails).
- **`PosTransType`**: array de grupos; cada ítem se interpreta por `PosTransTypeIDSource` (`1` = MetodoCancelacionID, `2` = MetodoCancelacionAgroID).

Throttling: 1 req/s. Ver [buenas-practicas.md](buenas-practicas.md).

---

## TradeCaptureReport

```
GET /PosTrade/TradeCaptureReport
```

| Parámetro | Req | Descripción |
|-----------|-----|-------------|
| DateFrom | sí | YYYYMMDD |
| DateTo | sí | YYYYMMDD |
| MarketID | no | Filtro opcional. `XMAB` solo aquí para cauciones/TV. No confundir con `MarketID` en respuestas de otros métodos (puede ser `ROFX`). |
| segmentID | no | Filtro opcional. `CAUC` |
| MarketSegmentID | no | `Rueda Electrónica`, `Fuera de Rueda` |
| CFICode | no | `RPXXXX` |
| TrdRptStatus | no | `0` Definitiva, `3` Anulada (Cauciones) |

### Campos respuesta (operaciones)

| Campo | Cauciones |
|-------|-----------|
| TradeID / TradeNumber | Número boleta |
| ExecID | ID negociación; derivación comparte ExecID |
| TrdType | 0 interferencia, 49 derivación, 3 asignación, 61 give-up |
| Side | G tomador, F colocador, 5/6 bajas |
| LastQty | Monto operación |
| LastPx | Siempre 1 |
| Rate | Tasa % |
| StartCash / EndCash | Monto inicial/final |
| SettlType | `B` Broken Date |
| SegmentId | `CAUC` — discriminar caución si la consulta es solo por fechas |
| MarketID (respuesta) | En boleta de caución suele ser `XMAB`; en otros métodos el mismo campo puede venir `ROFX` |
| Account | Cuenta de **registro/comitente** (no neteo) |
| Currency | `ARS`, `USD` (MEP en SettlCurrency) |

---

## MT506 — Stock de activos

```
GET /PosTrade/MT506
```

| Parámetro | Req | Notas |
|-----------|-----|-------|
| date | sí | YYYYMMDD |
| alyc / ALyC | sí | Código ALyC |
| CIM | no | Cuenta integración márgenes |
| AssetCode | no | Código activo |
| Format | no | `json` (default), `swift` |
| PageNumber, PageSize | no | Paginación |

Campos clave: `Reference` (`Cauciones $`, `Cauciones U$S`, `Supletorias`), `Asset`, `SECV`, `SHAI`, `MRKT`, `Party` (formato `margen//neteo`).

---

## MT536 — Movimientos de garantías

```
GET /PosTrade/MT536
```

**Recomendado:** solo `dateFrom`, `dateTo` y `alyc`. Traer todos los movimientos del día y filtrar en el ERP por `Reference`: `Cauciones $`, `Cauciones U$S`, `Supletorias`.

| Parámetro | Req | Notas |
|-----------|-----|-------|
| dateFrom | sí | YYYYMMDD |
| dateTo | sí | YYYYMMDD |
| alyc / ALyC | sí | |
| ClearingAccount | no | Opcional |
| Classification | no | **No enviar** para cauciones. Filtro API (7=Cauciones) existe en el manual pero no es el patrón recomendado. |
| Format | no | json / swift |
| MarketFCI | no | bool |

---

## CollateralList

```
GET /PosTrade/CollateralList?date=YYYYMMDD
```

| date | sí |

Devuelve: `InternalInstrumentCode`, `Asset`, `Price`, `Haircut`, `Reference` (`Cauciones $`, `Cauciones U$S`, `Supletorias`, etc.).

---

## MarginRequirementReport

```
GET /PosTrade/MarginRequirementReport
```

| Parámetro | Req |
|-----------|-----|
| date | sí |
| viewDetails | no | `true` desglosa por grupo producto |

Estructura: `ProductGroup` / `References[]` con `Reference` (`Cauciones $`, `Cauciones U$S`, `Supletorias`, …) y `Margin`.

---

## NewCollateralReport

**Alta instrucción**

```
POST /PosTrade/NewCollateralReport
Content-Type: application/json
Authorization: <token>
```

**Consulta estado**

```
GET /PosTrade/NewCollateralReport?CollRptID=<id>
```

| Parámetro | Tipo | Req | Descripción |
|-----------|------|-----|-------------|
| CollRptID | string | sí | ID de la instrucción (el enviado en POST como `ExternalCollRptID` o el devuelto en respuesta 200) |

Límite conjunto: 2 req/min (POST + GET). Ver [buenas-practicas.md](buenas-practicas.md).

Body JSON (POST) — campos documentados:

| Campo | Tipo | Obligatorio | Descripción |
|-------|------|-------------|-------------|
| InternalInstrumentCode | integer | sí | Código activo (de `CollateralList`) |
| Side | integer | sí | 1 Ingreso, 2 Egreso |
| ExternalCollRptID | string | sí | ID backoffice / idempotencia (20 chars) |
| Currency | string | sí | Código de moneda activa |
| CollAppIType | integer | sí | 3 Cauciones $, 4 Cauciones U$S, 24 Supletorias |
| OriginType | integer | sí | Combinación válida depende de `Side` (en circuito: 1 ingreso, 3 egreso) |
| OriginDepositoryAccountCode | integer | sí | Cuenta depositaria origen |
| DestinationDepositoryAccountCode | integer | sí | Cuenta depositaria destino |
| CompensationAccount | string | condicional | Requerida en escenarios MC / multi-CEL |
| Observations (`Notes`) | string | no | Observaciones libres |
| Details | array | sí (funcional) | Lista de detalles |
| Details[].Account | string | sí | Cuenta del detalle |
| Details[].Fund | string | no | Fondo asociado al detalle |
| Details[].Qty | string | no* | Cantidad del detalle; admite decimales con punto (ej. `"2412.1192385"`) (*en validación API puede comportarse como condicional) |
| ShareholderNumber | string | condicional (FCI) | Número de cuotapartista — requerido si el activo es FCI |
| ShareholderBusinessName | string | condicional (FCI) | Razón social del cuotapartista — requerido si el activo es FCI |
| ShareholderTIN | string | condicional (FCI) | CUIT/CUIL del cuotapartista — requerido si el activo es FCI |

`Fund`: valores permitidos según doc A3 (ej. `A3 RF Propio`, `A3 RF Tercero`). Ver [glosario.md](glosario.md).

---

## MarginBalance (tiempo real)

```
GET /Risk/MarginBalance
```

| Parámetro | Valores |
|-----------|---------|
| marketID | 1 = A3 Mercados, 2 = UFEX |
| balanceType | 1 = Consolidado |

Campos: `RequiredMargin`, `IntegratedAsset`, `IntegratedNetBalance`, `FinalMargin`, `Detail[]` por cuenta neteo. En `Detail`, `Reference`: `Cauciones $`, `Cauciones U$S`, `Supletorias`.

**Buena práctica:** no más de **1 req/min** intradía. Ver [buenas-practicas.md](buenas-practicas.md).

---

## Fee

```
GET /PosTrade/Fee
```

Filtros opcionales: `entity` (A3 Mercados), `tradingSession`, `product`, `securityType` (`Caución`), `execType`.

---

## AccruedFees

```
GET /PosTrade/AccruedFees
```

| Parámetro | Req |
|-----------|-----|
| date | sí |
| accountCode, symbol, productAlias, tradingSession, execType, entity | no |

`Trade[]` enlaza `TradeNumber`, `Qty`, `FeeAmt` por operación.

---

## DepositaryAccountList

```
GET /PosTrade/DepositaryAccountList
```

| Parámetro | Req | Notas |
|-----------|-----|-------|
| marketAccount | no | `true` = cuentas del mercado; `false` = solo cuentas del ALYC/agente integrador |

Campos clave de respuesta: `DepositaryAccountCode`, `DepositaryAccount`, `AccountType`, `Entity`, `Currency`, `CollateralAccount`, `Owner`, `TaxId`, `AccountTypeCode`.

---

## Diccionario ampliado

PDF oficial: https://apihub.primary.com.ar/assets/docs/PrimaryAPI-BO.pdf

Consolidado operativo (definición técnica + lectura de negocio): [diccionario-campos.md](diccionario-campos.md)

### Status garantías (NewCollateralReport)

Estados observados en circuito: `Inicial` → `Confirmado` / `Aprobado Riesgos` → `Procesado` → `Ejecutado` (también puede aparecer `Anulado`).

### Classification MT536 (solo referencia del manual)


### Monedas NewCollateralReport

Siempre "ARS".
