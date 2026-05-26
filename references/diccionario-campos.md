# Diccionario de campos - API Post-Trade A3 (Cauciones)

Documento para **proveedores de sistemas** (clientes integradores).

Este archivo unifica:
- diccionario técnico de campos (manual A3),
- y criterio operativo para implementar sin ambigüedades.

## Cómo usar este diccionario

- **`Dónde aparece`**: endpoint(s) donde el campo es relevante.
- **`Qué validar en integración`**: chequeo concreto que conviene implementar.
- **`Nota de negocio`**: solo cuando evita errores de interpretación.

---

## 1) Operaciones (`TradeCaptureReport`)

| Campo | Dónde aparece | Definición API | Qué validar en integración | Nota de negocio |
|---|---|---|---|---|
| `TradeID` | TradeCaptureReport | Número de boleta. | Persistir como id de operación. | Usar con `ExecID`, `Side`, `Account` para reconciliación. |
| `TradeNumber` | TradeCaptureReport, AccruedFees | Número de boleta. | Conciliar costos (`AccruedFees`) por `TradeNumber` + `ExecID`. | - |
| `OrderType` | TradeCaptureReport | Tipo de orden (`1` simple en esta integración). | Aceptar `1` y loguear cualquier valor no esperado. | - |
| `ExecID` | TradeCaptureReport, AccruedFees | Código de ejecución en motor de negociación. | Mantener trazabilidad por `ExecID`. | Derivaciones/asignaciones/give-up comparten `ExecID` con la madre. |
| `RootPartyRole` | TradeCaptureReport | Rol del participante (`12` operador). | Tratar como informativo. | - |
| `VenueType` | TradeCaptureReport | Tipo de mercado: `R` rueda, `C` fuera de rueda. | Mapear `R`/`C` en catálogo interno. | Útil para distinguir operación original vs derivación/ajuste. |
| `MarketID` | TradeCaptureReport (filtro y respuesta) | Mercado asociado (`XMAB` en cauciones). | Permitir filtro opcional `XMAB` solo aquí. | Gotcha: en otros endpoints puede venir `ROFX` aunque sea caución. |
| `MarketSegmentID` | TradeCaptureReport | Segmento de negociación (`Rueda Electrónica` / `Fuera de Rueda`). | Mapear ambos valores de texto. | - |
| `SecurityID` | TradeCaptureReport | Sigla de instrumento. | Filtrar cauciones por `CAU-ARS`/`CAU-USD`. | - |
| `SecurityIDSource` | TradeCaptureReport | Fuente de sigla (`H` clearing house). | Tratar como informativo. | - |
| `CFICode` | TradeCaptureReport | Código ISO 10962 (`RPXXXX` caución). | Validar `RPXXXX` para identificar cauciones. | - |
| `LastQty` | TradeCaptureReport | Cantidad de operación. | Tratar como monto operado en cauciones. | - |
| `LastPx` | TradeCaptureReport | Valor fijo (`1`). | Alertar si distinto de `1`. | - |
| `Currency` | TradeCaptureReport | Moneda de liquidación (`ARS` / `USD`). | Validar consistencia con instrumento (`CAU-ARS` / `CAU-USD`). | - |
| `TrdType` (`trdType`) | TradeCaptureReport | Tipo de ejecución (`0`,`3`,`49`,`61`). | Mapear y no descartar `3`/`49`/`61`. | Impacta cómo se interpreta el ciclo de vida de la operación. |
| `TrdRptStatus` | TradeCaptureReport | Estado de operación (`0` definitiva, `3` anulada). | Procesar `0`; tratar `3` como reversa/anulación. | En cauciones no aplica `4`. |
| `SettlCurrency` | TradeCaptureReport | Descripción de moneda de liquidación. | Tratar como etiqueta descriptiva. | - |
| `TradeDate` | TradeCaptureReport | Fecha de operación. | Usar formato fecha interno estándar. | - |
| `TransactTime` | TradeCaptureReport | Fecha/hora de operación. | Persistir timestamp de auditoría. | - |
| `SettlType` | TradeCaptureReport | Plazo (`B` Broken Date). | Esperar `B` para cauciones de esta integración. | Fecha efectiva en `SettlDate`. |
| `Side` | TradeCaptureReport | Lado (`G`,`F`,`5`,`6`). | Mapear los 4 valores. | `5/6` revierten efectos de madre. |
| `Account` | TradeCaptureReport | Cuenta involucrada. | Guardar como cuenta de **registro/comitente** en este endpoint. | Gotcha crítico: en garantías/márgenes cambia a cuenta de neteo. |
| `AggressorIndicator` | TradeCaptureReport | Orden agresora (`N`/`Y`). | Tratar como informativo. | - |
| `SegmentID` (`SegmentId`) | TradeCaptureReport | Segmento del instrumento (`CAUC`). | Validar `CAUC` para cauciones. | - |
| `SettlDate` | TradeCaptureReport | Fecha de liquidación. | Calcular vencimiento/cronograma con este campo. | - |
| `StartCash` | TradeCaptureReport | Monto inicial. | Persistir como capital inicial. | - |
| `EndCash` | TradeCaptureReport | Monto final. | Validar regla financiera interna (capital + intereses). | - |
| `Rate` | TradeCaptureReport | Tasa de operación. | Tratar como TNA (%) en cálculos/reportes. | - |

---

## 2) Garantías, movimientos y stock (`MT506`, `MT536`, `NewCollateralReport`, `CollateralList`)

| Campo | Dónde aparece | Definición API | Qué validar en integración | Nota de negocio |
|---|---|---|---|---|
| `Reference` | MT506, MT536, MarginRequirementReport, CollateralList | Tipo de garantía/concepto (`Cauciones $`, `Cauciones U$S`, `Supletorias`, etc.). | Filtrar por finalidades de cauciones en ERP. | Es la clave práctica para separar finalidades. |
| `Asset` | MT506, CollateralList | Activo involucrado. | Validar disponibilidad del activo en maestro interno. | - |
| `Party` | MT506 | Cuentas `Compensación//Neteo`. | Parsear estructura compuesta y validar cuentas. | - |
| `Amount` | MT506, MT536 | Monto débito/crédito valorizado. | Conciliar valorización contra cantidad/precio/aforo. | - |
| `Narrative` | MT506, MT536 | Descripción de fideicomiso. | Tratar como descriptivo/auditoría. | - |
| `MRKT` | MT506, MT536 | Precio de mercado. | Usar para valorización de garantías. | - |
| `InstrumentCode` | MT506, MT536, CollateralList | Código interno del activo. | Mantener mapeo contra maestro de instrumentos. | - |
| `InternalInstrumentCode` | CollateralList, MT506, MT536, NewCollateralReport | Código interno alternativo. | Usar este id en `POST NewCollateralReport`. | Campo operativo clave para alta/baja de garantías. |
| `SECV` | MT506 | Cantidad en garantía. | Conciliar stock final con movimientos ejecutados. | - |
| `SHAI` | MT506, MT536, CollateralList | Aforo aplicado. | Aplicar porcentaje de aforo en cálculos de cobertura. | `SHAI`/`Haircut` representan el mismo concepto. |
| `CurrencyCode` | MT506, MT536 | Moneda. | Validar consistencia con activo/finalidad. | - |
| `DepositoryAccount` | MT506, MT536, DepositaryAccountList | Cuenta depositaria (descripción). | Usar para trazabilidad y auditoría. | - |
| `DepositoryAccountCode` | MT506, MT536, NewCollateralReport, DepositaryAccountList | Código de cuenta depositaria. | Validar origen/destino antes de POST. | - |
| `ExpirationDate` | MT506, CollateralList | Fecha de vencimiento del activo. | Controlar vencimientos en reglas de elegibilidad. | - |
| `StockSourceCode` | MT506 | Origen del activo (`1` garantías, `2` mercado FCI, `3` custodia). | Mapear catálogo interno de origen. | - |
| `ClearingMemberCode` | MT506, MT536, AccruedFees, márgenes | Código ALyC miembro compensador. | Validar contra ALyC consultado. | - |
| `Status` | NewCollateralReport GET | Estado de solicitud de garantía. | Implementar polling por estado hasta final. | Los estados observados pueden variar según circuito/entorno. |
| `ClearingBusinessDate` | MT536 | Fecha operativa. | Usar para cortes diarios y conciliación. | - |
| `CollMovTransType` | MT536 | Tipo de movimiento (`Ingreso` / `Egreso`). | Validar signo/cantidad según tipo. | - |
| `CollMovDescription` | MT536 | Descripción del movimiento. | Tratar como audit trail. | - |
| `Instrument` | MT536 | Nombre del instrumento. | Informativo para reportes. | - |
| `Quantity` | MT536 | Cantidad de movimiento. | Conciliar con instrucción ejecutada. | - |
| `ClearingMember` | MT536, márgenes | Nombre del miembro compensador. | Informativo. | - |

---

## 3) Márgenes y cuentas (`MarginRequirementReport`, `MarginBalance`, reportes relacionados)

| Campo | Dónde aparece | Definición API | Qué validar en integración | Nota de negocio |
|---|---|---|---|---|
| `Date` | MarginRequirementReport, AccruedFees | Fecha de registro. | Normalizar formato y zona horaria interna. | - |
| `ProductGroup` | MarginRequirementReport | Grupo de producto. | Mapear grupo para reportes internos. | - |
| `ProductGroupCurrencyQuotation` | MarginRequirementReport | Cotización de moneda del grupo. | Usar en valorización/controles de conversión. | - |
| `ProductGroupCurrencyTotal` | MarginRequirementReport | Total de márgenes en moneda de grupo. | Verificar signo y consistencia con `Margin`. | - |
| `ProductGroupCurrency` | MarginRequirementReport | Moneda del grupo (`Pesos` / `Dólar MEP`). | Mapear catálogo de monedas de negocio. | - |
| `CompensationAccount` | MarginRequirementReport, NewCollateralReport, MT536 | Cuenta de compensación. | Validar relación con netting/registro antes de POST. | - |
| `CompensationAccountCode` | MarginRequirementReport | Código de cuenta compensación. | Mapeo interno de cuentas de compensación. | - |
| `NettingAccount` | MarginRequirementReport, MT536 | Cuenta de neteo. | Validar relación 1:1 esperada con cuenta de registro. | - |
| `NettingAccountCode` | MarginRequirementReport | Código de cuenta de neteo. | - | - |
| `Margin` | MarginRequirementReport | Margen requerido. | Conciliar por finalidad (`Reference`). | Es por posición total, no por trade aislado. |
| `MarketCode` | MarginBalance / reportes varios | Código de mercado. | Tratar como informativo si no se usa en reglas. | - |
| `Market` | MarginBalance / reportes varios | Mercado. | Informativo. | - |
| `RequiredMargin` | MarginBalance | Margen requerido total. | Comparar contra activos integrados. | - |
| `IntegratedAsset` | MarginBalance | Activos integrados valorizados. | Base para cobertura actual. | - |
| `IntegratedNetBalance` | MarginBalance | Neto entre margen y activos. | Alertar déficit/excedente según política interna. | - |
| `CurrentMargin` | MarginBalance | Margen actual. | Informativo para evolución intradía. | - |
| `FinalMargin` | MarginBalance | Margen final. | Usar para cierre diario cuando aplique. | - |
| `FinalTotalIntegratedNetBalance` | MarginBalance | Balance final integrado. | Validar cierre vs posición final del día. | - |
| `ClearingAccountCode` | Reportes varios | Código cuenta de compensación. | Mapear en maestro de cuentas. | - |
| `ClearingAccountType` | Reportes varios | Tipo de cuenta. | Informativo / mapeo. | - |
| `AccountCode` | AccruedFees / reportes | Código de cuenta. | Validar relación con cuenta consultada. | - |
| `AccountType` | Reportes varios | Tipo de cuenta. | Informativo. | - |

---

## Referencias relacionadas

- `glosario.md`: definiciones de negocio.
- `reference.md`: parámetros por endpoint.
- `examples.md`: payloads concretos.
