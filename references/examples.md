# Ejemplos de integración — Cauciones A3

## 1. Cliente HTTP mínimo (pseudocódigo)

```typescript
const BASE = process.env.A3_API_BASE ?? "https://demoapi.anywhereportfolio.com.ar";

async function getToken(): Promise<string> {
  const res = await fetch(`${BASE}/AuthToken/AuthToken`, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      nombreUsuario: process.env.A3_USER,
      password: process.env.A3_PASSWORD,
    }),
  });
  const data = await res.json();
  if (data.Code !== "200") throw new Error(`Auth failed: ${data.Status}`);
  return data.Value;
}

async function a3Get(path: string, token: string, params: Record<string, string>) {
  const qs = new URLSearchParams(params).toString();
  const res = await fetch(`${BASE}${path}?${qs}`, {
    headers: { Authorization: token },
  });
  return res.json();
}
```

## 2. Operaciones del día (Cauciones)

```
GET /PosTrade/TradeCaptureReport?DateFrom=20260521&DateTo=20260521
Authorization: <token>
```

Opcional (filtro en API):

```
GET .../TradeCaptureReport?DateFrom=20260521&DateTo=20260521&MarketID=XMAB&segmentID=CAUC
```

### Tomadora (Side G)

```json
{
  "TradeID": 32115134,
  "TradeNumber": 32115134,
  "TrdRptStatus": "0",
  "TrdType": 0,
  "OrderType": 1,
  "ExecID": 26052112381128988,
  "RootParties": [
    {
      "RootPartyID": "",
      "RootPartyIDSource": "D",
      "RootPartyRole": "12"
    }
  ],
  "VenueType": "R",
  "MarketID": "XMAB",
  "MarketSegmentID": "Rueda Electrónica",
  "Instrument": [
    {
      "SecurityID": "CAU-ARS",
      "SecurityIDSource": "H",
      "CFICode": "RPXXXX"
    }
  ],
  "LastQty": 1000000,
  "LastPx": 1,
  "Currency": "ARS",
  "SettlCurrency": "Pesos",
  "TradeDate": "2026-05-21",
  "TransactTime": "2026-05-21T12:38:11",
  "SettlType": "B",
  "SettlDate": "2026-05-22",
  "TrdCapRptSideGrp": [
    {
      "Side": "G",
      "Account": "1234",
      "Rate": 30,
      "StartCash": 1000000,
      "EndCash": 1000821.92,
      "AggressorIndicator": "N"
    }
  ],
  "SegmentId": "CAUC"
}
```

### Colocadora (Side F)

Misma estructura con `Side: "F"`, `SecurityID: "CAU-ARS"`, `Currency: "ARS"`.

### Par tomador + derivación

Dos registros en `Value[]`:

1. `TrdType: 0`, `Side: "G"`, `VenueType: "R"`, `MarketSegmentID: "Rueda Electrónica"`
2. `TrdType: 49`, `Side: "5"`, `VenueType: "C"`, `MarketSegmentID: "Fuera de Rueda"`, **mismo ExecID**

Reconciliación backoffice: agrupar por `ExecID`; operación madre `TrdType=0`, evento derivación `TrdType=49`.

## 3. Derechos de mercado (AccruedFees)

```http
GET /PosTrade/AccruedFees?date=20260521&alyc=123
Authorization: <token>
```

```json
{
  "Status": "OK",
  "Code": "200",
  "Value": [
    {
      "FeeDescription": "Derecho Registro Caución Caución Pesos (Caución A3) ",
      "ClearingMemberCode": "123",
      "ClearingMember": "TEST ALYC",
      "BillingAccountCode": "1123",
      "BillingAccount": "TEST ALYC",
      "AccountCode": "1234",
      "Account": "Descripcion de la cuenta",
      "ClearingAccountCode": "1123",
      "ClearingAccount": "TEST ALYC",
      "Symbol": "CAU-ARS",
      "ProductAlias": "Caución Pesos",
      "Product": "Caución Pesos",
      "TradingSession": "",
      "ExecType": "",
      "OrderType": "",
      "FeeCurr": "ARS",
      "Date": "2026-05-21T00:00:00",
      "FeeType": "Facturable",
      "Trade": [
        {
          "TradeNumber": 32115134,
          "ExecId": 26052112381128988,
          "Qty": 1000.82,
          "FeeAmt": 0.0005,
          "BaseCurrencyTotal": 1000.82
        }
      ],
      "FeeID": 101703,
      "Entity": "MATBA ROFEX",
      "Status": "No Facturado",
      "Rate": 1
    }
  ]
}
```

## 4. Requerimientos de márgenes (MarginRequirementReport)

```json
{
  "Status": "OK",
  "Code": "200",
  "Value": [
    {
      "ClearingMemberCode": "123",
      "ClearingMember": "TEST ALYC",
      "Date": "2026-04-29",
      "ProductGroup": "Cauciones $",
      "ProductGroupCurrencyQuotation": 1.0,
      "ProductGroupCurrencyTotal": 10000.0,
      "ProductGroupCurrency": "Pesos",
      "Accounts": [
        {
          "CompensationAccount": "TEST ALYC",
          "CompensationAccountCode": "1123",
          "SubAccounts": [
            {
              "NettingAccount": "Descripcion de la cuenta",
              "NettingAccountCode": "1234",
              "References": [
                {
                  "Reference": "Cauciones $",
                  "Currency": "Pesos",
                  "Margin": 10000.0
                }
              ]
            }
          ]
        }
      ]
    },
    {
      "ClearingMemberCode": "123",
      "ClearingMember": "TEST ALYC",
      "Date": "2026-04-29",
      "ProductGroup": "Cauciones USD",
      "ProductGroupCurrencyQuotation": 1200.48,
      "ProductGroupCurrencyTotal": -100013700.0,
      "ProductGroupCurrency": "Dólar MEP",
      "Accounts": [
        {
          "CompensationAccount": "TEST ALYC",
          "CompensationAccountCode": "1123",
          "SubAccounts": [
            {
              "NettingAccount": "Cuenta de prueba2",
              "NettingAccountCode": "4321",
              "References": [
                {
                  "Reference": "Cauciones U$S",
                  "Currency": "Pesos",
                  "Margin": -120064446576.0
                }
              ]
            }
          ]
        }
      ]
    }
  ]
}
```

## 5. Activos para garantías (CollateralList)

```http
GET /PosTrade/CollateralList?date=20260521
Authorization: <token>
```

```json
{
  "Status": "OK",
  "Code": "200",
  "Value": [
    {
      "InternalInstrumentCode": 2827,
      "Asset": "AL30",
      "InstrumentCode": "5921",
      "ExpirationDate": "2030-07-09T00:00:00",
      "PriceDate": "2025-05-21T00:00:00",
      "Price": 804.976772,
      "Currency": "Pesos",
      "Haircut": 90,
      "Reference": "Cauciones $",
      "SecuritySubType": "Títulos - Soberano doméstico",
      "SecuritySubTypeCode": 5
    },
    {
      "InternalInstrumentCode": 2833,
      "Asset": "AL29",
      "InstrumentCode": "5927",
      "ExpirationDate": "2029-07-09T00:00:00",
      "PriceDate": "2025-05-21T00:00:00",
      "Price": 854.962207,
      "Currency": "Pesos",
      "Haircut": 90,
      "Reference": "Margen SPOT TV",
      "SecuritySubType": "Títulos - Soberano doméstico",
      "SecuritySubTypeCode": 5
    }
  ]
}
```

## 6. Ingreso garantía Cauciones $

```http
POST /PosTrade/NewCollateralReport
Authorization: <token>
Content-Type: application/json

{
  "InternalInstrumentCode": 2827,
  "Side": 1,
  "ExternalCollRptID": "Codigo_Interno2",
  "Currency": "ARS",
  "CollAppIType": 3,
  "OriginType": 1,
  "OriginDepositoryAccountCode": 19,
  "DestinationDepositoryAccountCode": 9364,
  "CompensationAccount": "1123",
  "Observations": "",
  "Details": [
    { "Account": "1234", "Fund": "A3 RF Tercero", "Qty": "400000" }
  ]
}
```

`CollAppIType: 4` para Cauciones U$S.

### Estado de instrucción (GET) — ingreso

```http
GET /PosTrade/NewCollateralReport?CollRptID=Codigo_Interno
Authorization: <token>
```

```json
{
  "Status": "OK",
  "Code": "200",
  "Value": [{ "CollRptID": "Codigo_Interno", "Status": "Inicial" }]
}
```

```json
{
  "Status": "OK",
  "Code": "200",
  "Value": [{ "CollRptID": "Codigo_Interno", "Status": "Confirmado" }]
}
```

```json
{
  "Status": "OK",
  "Code": "200",
  "Value": [{ "CollRptID": "Codigo_Interno", "Status": "Ejecutado" }]
}
```

## 6b. Ingreso garantía FCI

Campos `Shareholder*` requeridos cuando el activo es FCI. `CollAppIType: 3` = Cauciones $.

```http
POST /PosTrade/NewCollateralReport
Authorization: <token>
Content-Type: application/json

{
  "InternalInstrumentCode": 2827,
  "Side": 1,
  "ExternalCollRptID": "34",
  "Currency": "ARS",
  "CollAppIType": 3,
  "OriginType": 1,
  "OriginDepositoryAccountCode": 9364,
  "DestinationDepositoryAccountCode": 19,
  "CompensationAccount": "1234",
  "ShareholderNumber": "123891",
  "ShareholderBusinessName": "Cuotapartista TEST",
  "ShareholderTIN": "20426250013",
  "Observations": "",
  "Details": [
    { "Account": "55412", "Fund": "FGOT", "Qty": "1000" }
  ]
}
```

## 7. Egreso de garantía Cauciones $

```http
POST /PosTrade/NewCollateralReport
Authorization: <token>
Content-Type: application/json

{
  "InternalInstrumentCode": 2827,
  "Side": 2,
  "ExternalCollRptID": "Codigo_Interno2",
  "Currency": "ARS",
  "CollAppIType": 3,
  "OriginType": 3,
  "OriginDepositoryAccountCode": 19,
  "DestinationDepositoryAccountCode": 9364,
  "CompensationAccount": "1123",
  "Observations": "",
  "Details": [
    { "Account": "1234", "Fund": "A3 RF Tercero", "Qty": "100" }
  ]
}
```

### Estado de instrucción (GET) — egreso

```http
GET /PosTrade/NewCollateralReport?CollRptID=Codigo_Interno2
Authorization: <token>
```

```json
{
  "Status": "OK",
  "Code": "200",
  "Value": [{ "CollRptID": "Codigo_Interno2", "Status": "Inicial" }]
}
```

```json
{
  "Status": "OK",
  "Code": "200",
  "Value": [{ "CollRptID": "Codigo_Interno2", "Status": "Aprobado Riesgos" }]
}
```

```json
{
  "Status": "OK",
  "Code": "200",
  "Value": [{ "CollRptID": "Codigo_Interno2", "Status": "Procesado" }]
}
```

```json
{
  "Status": "OK",
  "Code": "200",
  "Value": [{ "CollRptID": "Codigo_Interno2", "Status": "Ejecutado" }]
}
```

### Estado de instrucción (GET) — anulación/rechazo

```
GET /PosTrade/NewCollateralReport?CollRptID=Codigo_Interno2
```

```json
{
  "CollRptID": "Codigo_Interno2",
  "Status": "Anulado"
}
```

## 8. Movimientos de activos (MT536)

```http
GET /PosTrade/MT536?dateFrom=20260521&dateTo=20260521&alyc=123
Authorization: <token>
```

```json
{
  "Status": "OK",
  "Code": "200",
  "Value": [
    {
      "ClearingBusinessDate": "2026-05-21",
      "CollMovTransType": "Ingreso",
      "CollMovDescription": "Informe de pago",
      "Party": [
        {
          "ClearingMember": "123",
          "CompensationAccount": "1123",
          "NettingAccount": "1234"
        }
      ],
      "InstrumentCode": "5921",
      "Instrument": "AL30",
      "DepositoryAccount": "Caja de Valores Dep 303",
      "Narrative": "A3 RF Tercero",
      "Reference": "Cauciones $",
      "Quantity": 400000,
      "SettlDate": "2030-07-09",
      "MRKT": 804.976772,
      "SHAI": 90,
      "Amount": 289791637.92,
      "InternalInstrumentCode": 2827,
      "CurrencyCode": "ARS",
      "Trades": [{}]
    },
    {
      "ClearingBusinessDate": "2026-05-21",
      "CollMovTransType": "Egreso",
      "CollMovDescription": "Solicitud de extracción",
      "Party": [
        {
          "ClearingMember": "123",
          "CompensationAccount": "1123",
          "NettingAccount": "1234"
        }
      ],
      "InstrumentCode": "5921",
      "Instrument": "AL30",
      "DepositoryAccount": "Caja de Valores Dep 303",
      "Narrative": "A3 RF Tercero",
      "Reference": "Cauciones $",
      "Quantity": -100,
      "SettlDate": "2030-07-09",
      "MRKT": 804.976772,
      "SHAI": 90,
      "Amount": 72447.90948,
      "InternalInstrumentCode": 2827,
      "CurrencyCode": "ARS",
      "Trades": [{}]
    }
  ]
}
```

## 9. Stock de activos (MT506)

```http
GET /PosTrade/MT506?date=20260521&alyc=123
Authorization: <token>
```

```json
{
  "Reference": "Cauciones $",
  "Asset": "AL30",
  "TradeDate": "2026-05-21T00:00:00",
  "Party": "1123\\1234",
  "Amount": 28906715.8825,
  "Narrative": "A3 RF Tercero",
  "MRKT": 804.976772,
  "InstrumentCode": "5921",
  "InternalInstrumentCode": "2827",
  "SECV": 39900,
  "SHAI": 90,
  "CurrencyCode": "ARS",
  "DepositoryAccount": "Caja de Valores Dep 303",
  "DepositoryAccountCode": "19",
  "ExpirationDate": "2030-07-09T00:00:00",
  "ExternalAccountCode": "133",
  "StockSourceCode": 1,
  "ClearingMemberCode": "123"
}
```

## 10. Cuentas depositarias (DepositaryAccountList)

```http
GET /PosTrade/DepositaryAccountList?marketAccount=true
Authorization: <token>
```

```json
{
  "Status": "OK",
  "Code": "200",
  "Value": [
    {
      "DepositaryAccountCode": 2,
      "DepositaryAccount": "Cuenta depositaria descripción",
      "AccountType": "Cuenta Corriente",
      "Entity": "MERCADO A TÉRMINO DE ROSARIO S.A.",
      "Currency": "ARS",
      "DatanetEnable": false,
      "MutualFundsAccount": false,
      "MTMAccount": false,
      "CollateralAccount": true,
      "Owner": "ROFEX  INVERSORA S.A.",
      "TaxId": "30528994012",
      "AccountTypeCode": 2
    },
    {
      "DepositaryAccountCode": 3,
      "DepositaryAccount": "Cuenta depositaria descripción2",
      "AccountType": "Cuenta Corriente",
      "Entity": "MERCADO A TÉRMINO DE ROSARIO S.A.",
      "Currency": "ARS",
      "DatanetEnable": false,
      "MutualFundsAccount": false,
      "MTMAccount": false,
      "CollateralAccount": true,
      "Owner": "ROFEX  INVERSORA S.A.",
      "TaxId": "30528994012",
      "AccountTypeCode": 2
    }
  ]
}
```

## 11. Detalle de cuenta (AccountDetails)

```http
GET /PreTrade/AccountDetails?accountCode=22300
Authorization: <token>
```

```json
{
  "Status": "OK",
  "Code": "200",
  "Value": [
    {
      "AccountCode": "22300",
      "Account": "TEST S.A.",
      "CompensationAccountCode": "1999",
      "CompensationAccount": "Compensacion Testing S.A.",
      "NettingAccountCode": "22300",
      "PartyId": "30123456782",
      "CreationDate": "2021-09-30T00:00:00",
      "ClearingMemberCode": "123",
      "ClearingMember": "BROKER TEST S.A.",
      "AccountType": 1,
      "UnderlyingOwner": true,
      "AccountRegisterType": 2,
      "PosTransType": [
        [
          { "PosTransTypeID": "", "PosTransTypeIDSource": 1 },
          { "PosTransTypeID": "1", "PosTransTypeIDSource": 2 }
        ]
      ],
      "PartySubGrp": [
        [
          { "PartySubID": "", "PartySubIDSource": 5 },
          { "PartySubID": "Calle Falsa 123 (1200)", "PartySubIDSource": 6 },
          { "PartySubID": "+54 (11) 1234-5678", "PartySubIDSource": 7 },
          { "PartySubID": "TESTING@PRIMARY.COM.AR", "PartySubIDSource": 8 },
          { "PartySubID": "TEST S A", "PartySubIDSource": 2 },
          { "PartySubID": 2, "PartySubIDSource": 4009 },
          { "PartySubID": 2, "PartySubIDSource": 4003 },
          { "PartySubID": true, "PartySubIDSource": 4005 },
          { "PartySubID": 2, "PartySubIDSource": 4022 }
        ]
      ]
    }
  ]
}
```

Campos anidados: ver reglas de extracción en [diccionario-campos.md](diccionario-campos.md) sección 4.

## 12. Mapeo sugerido a entidades backoffice

| API | Entidad ERP sugerida |
|-----|----------------------|
| TradeCaptureReport | Boleta / operación monetaria |
| TrdType 49 + Side 5/6 | Evento de baja / derivación |
| MT506 | Posición garantía por comitente |
| NewCollateralReport | Orden de garantía (estado ↔ Status API) |
| MarginBalance | Panel riesgo intradía |
| AccountDetails | Maestro comitente / detalle de cuenta |
| AccruedFees | Asiento costos / derechos mercado |

## 13. Errores HTTP

| HTTP | Acción |
|------|--------|
| 401 | Renovar token (1 AuthToken/día) |
| 400 | Revisar parámetros; no reintentar igual |
| 429 | Backoff; espaciar según [buenas-practicas.md](buenas-practicas.md) |

| Síntoma lógico | Acción |
|----------------|--------|
| Value vacío post 17h | Esperar fin CCP; reconsultar |
| ExecID sin derivación | Consultar después de cierre rueda |

## 14. Variables de entorno (.env ejemplo)

```env
A3_API_BASE=https://demoapi.anywhereportfolio.com.ar
A3_USER=
A3_PASSWORD=
A3_ALYC_CODE=
A3_DEFAULT_MARKET_ID=XMAB
A3_SEGMENT_CAUC=CAUC
```
