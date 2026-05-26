# Glosario — Cauciones y API Post-Trade A3

Definiciones operativas para integradores de backoffice. El agente debe consultar este archivo antes de inferir significados de negocio.

## Operaciones y estados

| Término | Contexto API | Definición |
|---------|--------------|------------|
| **Caución** | `CAU-ARS`, `CAU-USD`, `CAUC` | Operación de mutuo con garantía mobiliaria: una parte presta dinero (Colocador) y espera recibir el capital más intereses al vencimiento, y la otra parte toma el dinero (Tomador) dejando activos en garantías, con obligación de devolver capital más intereses al vencimiento. Se negocia en el segmento de cauciones del mercado `XMAB`. |
| **Tomador** | `Side=G` | Parte que toma fondos prestados y entrega activos en garantía, obligándose a devolver el capital más intereses al vencimiento. |
| **Colocador** | `Side=F` | Parte que coloca fondos (presta dinero) a cambio de una tasa de interés pactada (TNA en `Rate`). |
| **Interferencia de ofertas** | `TrdType=0` | Operación originada por el cruce de ofertas en el libro de la rueda electrónica (concertación estándar en mercado). |
| **Derivación** | `TrdType=49`, mismo `ExecID` | Operación complementaria registrada Fuera de Rueda, vinculada a la operación original por el mismo `ExecID`, para canalizar la liquidación a través de otro Agente Liquidador (p. ej. vía sociedad depositaria). Por ejemplo, ALyC AA opera una colocadora para la cuenta 11 y luego deriva a una Sociedad Depositaria por lo que la operación original colocadora en cuenta 11 se da de baja por medio de la operación de baja colocador. |
| **Baja Tomador / Baja Colocador** | `Side=5` / `Side=6` | Lado de la operación de derivación, de asignación o de corrección que revierte el efecto de la original: si la madre fue tomador (`G`), la derivada lleva baja tomador (`5`); si fue colocador (`F`), baja colocador (`6`). |
| **Asignación** | `TrdType=3` | Modificación posterior de la cuenta asignada a una operación ya concertada. |
| **Give-up** | `TrdType=61` | Traspaso de una operación concertada por un ALyC a otro ALyC para su compensación/liquidación. |
| **Boleta** | `TradeID`, `TradeNumber` | Identificador unívoco de la operación en el mercado/cámara. |
| **Operación definitiva** | `TrdRptStatus=0` | Estado que tienen las operaciones de caución. |
| **Operación anulada** | `TrdRptStatus=3` | Estado que tienen las operaciones cuando son anuladas. En cauciones no aplica estado transitorio (`4`). |
| **ExecID** | Operaciones relacionadas | Identificador de la ejecución en el sistema de negociación. La operación madre y su derivación (u otros eventos ligados) comparten el mismo `ExecID`. |
| **Broken Date** | `SettlType=B` | Valor fijo para indicar que la operación tiene una fecha de liquidación que se publica en el método `SettlDate`. |
| **Rueda Electrónica / Fuera de Rueda** | `MarketSegmentID`, `VenueType` `R`/`C` | Modalidad de negociación en la que se generó una operación: en pantalla (`R`) o fuera de rueda (`C`), p. ej. derivaciones y ciertos ajustes. |
| **Rate (TNA)** | `Rate` en `TrdCapRptSideGrp` | Tasa nominal anual expresada en porcentaje que determina el costo/rendimiento de la caución. |
| **StartCash / EndCash** | Montos en boleta | Capital inicial de la operación y monto a devolver al vencimiento (capital + intereses). |

## Participantes y cuentas

| Término | Contexto API | Definición |
|---------|--------------|------------|
| **ALyC** | `alyc`, `ClearingMemberCode` | Agente de Liquidación y Compensación: participante habilitado ante el CCP que compensa y liquida operaciones propias y de terceros. |
| **CCP / A3 CCP** | Procesos post-17 h | Contraparte central de clearing: intermedia entre las partes, calcula márgenes y ejecuta compensación y liquidación después del cierre de rueda. |
| **Cuenta de compensación** | `CompensationAccount` | Cuenta del ALyC ante el CCP que agrupa márgenes requeridos y stock de activos de las cuentas de neteo asociadas. Cada ALyC tiene una Cuenta de compensación para cartera propia (debajo cuelgan cuentas de neteo propias del ALyC), una Cuenta de compensación de cartera de terceros (debajo cuelgan cuentas comitentes del ALyC) y dependiendo el caso podría tener algunas Cuentas de compensación adicionales. Los conceptos de Cuenta de Integración de Márgenes (CIM) y Cuenta de Liquidación son sinónimos de la entidad Cuenta de Compensación. |
| **Cuenta de registro / comitente** | `Account` en `TradeCaptureReport` | Código de la cuenta comitente ante el ALyC, estas cuentas son las que registran operaciones. |
| **Cuenta de neteo** | `Account` en garantías/márgenes, `NettingAccount` | Cuenta donde se netean márgenes y garantías; asociada 1:1 a la de registro (~99 % mismo número, distinto significado en API). |
| **Supletorias** | `CollAppIType=24` | Finalidad de garantías supletorias vinculada a la operatoria de cauciones. Es un requerimiento a nivel cuenta propia del ALyC, que sería un cargo por apalancamiento que se calcula teniendo en cuenta operatoria propia y de sus clientes. |
| **CIM** | Parámetro `CIM` en `MT506` | Cuenta de integración de márgenes: agrupa el neteo de requerimientos de margen y del stock de activos valorizados para ese esquema. |
| **Party** | margen//neteo en `MT506` | Identificador compuesto: cuenta de compensación//cuenta de neteo del participante en la posición de garantía. |

## Garantías y márgenes

| Término | Contexto API | Definición |
|---------|--------------|------------|
| **Finalidad / Reference** | `Cauciones $`, `Cauciones U$S`, `Supletorias`, `CollAppIType` 3/4/24 | “Caja” de margen y garantías por operatoria (cauciones pesos, cauciones dólares o supletorias). |
| **Aforo** | `SHAI`, `Haircut` en `CollateralList` | Porcentaje del valor de mercado del activo que el CCP acepta para integrar en garantía (p. ej. 90 = se considera el 90 % del valor). |
| **SECV** | `MT506` | Cantidad nominal o cantidad del activo puesta en garantía. |
| **MRKT** | `MT506`, `MT536` | Precio de mercado del activo usado para valorizar la garantía. |
| **Margen requerido** | `RequiredMargin`, `Margin` | Exigencia de garantías valorizado en pesos que el CCP calcula para cubrir el riesgo de la posición en una finalidad. |
| **Activo integrado** | `IntegratedAsset` | Valorización de las garantías depositadas e integradas que cubren (total o parcialmente) el margen. |
| **Saldo neto integrado** | `IntegratedNetBalance`, `FinalTotalIntegratedNetBalance` | Diferencia entre activos integrados valorizados y margen requerido; indica excedente o déficit de cobertura. |
