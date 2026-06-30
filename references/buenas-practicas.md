# Buenas prácticas y throttling — API BackOffice (Cauciones)

Fuente: BO-Buenas_practicas. Solo aplica a métodos usados en integración Cauciones.

**Ambiente:** único demo → `https://demoapi.anywhereportfolio.com.ar`

## Horarios

| Ventana | Qué hacer |
|---------|-----------|
| **10:00–17:00** | Rueda activa. `TradeCaptureReport` transitorias/anuladas hasta **12 req/min**. `MarginBalance` intradía. |
| **Después de 17:00** | Procesos **CCP** (compensación/liquidación). Operaciones **definitivas** y MT536/AccruedFees/MarginRequirementReport: consultar **post CCP**. |
| Sin `ClosingProcesses` en doc Cauciones | Usar hora fin rueda + margen; si integran API BO completa, `ClosingProcesses` indica fin de proceso (`ProcessTypeCode` según mercado). |

## Distribución de requests

Los límites de throttling son **por ventana de tiempo**. Ej.: 2 req/seg → 1 cada 500 ms. Más rápido → **429** hasta que pase la ventana.

Respuesta 429:

```json
{
  "error": {
    "status_code": 429,
    "status": "Too Many Requests"
  }
}
```

Implementar: backoff exponencial + jitter; no reintentar en bucle tight.

## Por método (Cauciones)

| Método | Buena práctica | Throttling |
|--------|----------------|------------|
| **AuthToken** | **1 vez por día** (token 24 h) | 1 / 60 s |
| **SecurityList** | 1 vez por día (no cambia en rueda) | 1 / s |
| **TradeCaptureReport** | Transitorias/anuladas: hasta **12/min** en rueda. **Definitivas**: **2/día** post compensación/liquidación | 2 / s |
| **CollateralList** | 1 vez por día | 2 / s |
| **MT506** | **3/min** (custodia intradía puede cambiar stock) | 2 / s |
| **MT536** | **1/min**; ideal **fin de día post proceso diario** | 2 / s |
| **MarginRequirementReport** | **2/día** post compensación/liquidación | 1 / s |
| **NewCollateralReport** | **POST** alta + **GET** estado: **2/min** total | 2 / s |
| **DepositaryAccountList** | Consultar bajo demanda o cache diario | 1 / s |
| **Fee** | 1 vez por día (cambios ~mensuales) | 1 / s |
| **AccruedFees** | 1 vez por día, post proceso diario | 1 / s |
| **MarginBalance** | **≤ 1 req/min** intradía | 2 / s |
| **AccountDetails** | Consultar bajo demanda (detalle de cuenta por `accountCode`) | 1 / s |

## Estados de NewCollateralReport

Estados observados en circuito: `Inicial`, `Confirmado`, `Aprobado Riesgos`, `Procesado`, `Ejecutado`, `Anulado`.

Recomendación: al consultar estado, espaciar polling (por ejemplo cada 30-60 s) para no consumir la cuota de `2/min` compartida con POST.

## Scheduler sugerido (día hábil)

```
10:00  AuthToken (si no hay token del día)
10:05  SecurityList + CollateralList + Fee (cache)
10:00–17:00  TradeCaptureReport (≤12/min) + MT506 (≤3/min) + MarginBalance (≤1/min)
17:00+  Esperar CCP
Post-CCP  TradeCaptureReport definitivas (2/día) + MT536 + MarginRequirementReport + AccruedFees
```

## Métodos del PDF no usados en Cauciones (referencia)

Útiles si el proveedor amplía integración: `ClosingProcesses`, `AccountBalance`, `PositionReport`, `ExecutionReport`, MFCI (`NewOrderSingle`, etc.). Ver [apihub.primary.com.ar](https://apihub.primary.com.ar/).
