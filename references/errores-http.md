# Códigos HTTP — API Post-Trade Cauciones

## GET (todos los endpoints de consulta)

| HTTP | Significado | Acción integrador |
|------|-------------|-------------------|
| **200** | OK (revisar `Code` en body: `"200"`) | Procesar `Value` |
| **400** | Request inválido (parámetros, formato fecha, etc.) | Corregir query/body; no reintentar igual |
| **401** | Token inválido o ausente | Renovar `AuthToken` (máx. 1/día salvo expiración) |
| **429** | Throttling excedido | Backoff; ver [buenas-practicas.md](buenas-practicas.md) |

Body éxito: `{ "Status": "OK", "Code": "200", "Value": ... }`.

Body 429 (formato documentado):

```json
{ "error": { "status_code": 429, "status": "Too Many Requests" } }
```

## POST

### `POST /PosTrade/NewCollateralReport`

| HTTP | Cuando ocurre | Response posible | Acción integrador |
|------|---------------|------------------|-------------------|
| **200** | Alta exitosa | Entero simple con `CollRptID` generado | Persistir `CollRptID` y pasar a polling GET de estado |
| **400** | Body nulo o JSON inválido | `"Json format invalid"` | Corregir body; no reintentar igual |
| **400** | Faltan campos obligatorios | `"Field ... cannot be null."` | Completar campos obligatorios; no reintentar igual |
| **400** | Faltan campos obligatorios condicionales (FCI, MC, multi-CEL) | `"Field ... cannot be null."` | Aplicar validación por escenario y reenviar |
| **400** | Combinación inválida `Side` + `OriginType` | `"The combination entered is invalid. Origin Type / Side ..."` | Corregir combinación según tipo de movimiento |
| **400** | `CompensationAccount` inválida en multi-CEL | `"Compensation account ... invalid."` | Corregir cuenta de compensación |
| **400** | `OriginDepositoryAccountCode` inválido | `"Invalid OriginDepositoryAccountCode."` | Corregir cuenta depositaria origen |
| **400** | `DestinationDepositoryAccountCode` inválido | `"Invalid DestinationDepositoryAccountCode."` | Corregir cuenta depositaria destino |
| **400** | `InternalInstrumentCode` inválido | `"Invalid InternalInstrumentCode."` | Reconsultar `CollateralList` y reenviar |
| **400** | `Currency` inválida | `"Invalid Currency."` | Corregir moneda y consistencia con activo/finalidad |
| **400** | Cuenta sin CV asignada | Mensaje tipo `"The following/s account/s ... does not have the CV account assigned."` | Gestionar alta/asignación CV antes de reenviar |
| **400** | Relación cuenta compensación / registro incorrecta | `"The Compensation Account-Registry Account relationship is not correct."` | Corregir mapeo de cuentas |
| **400** | Cuenta de compensación inactiva | `"Inactive CompensationAccount"` | Activar/cambiar cuenta antes de reenviar |
| **503** | Falla operativa no mapeada en persistencia | `"The operation could not be performed."` | Reintentar con backoff y alertar si persiste |

Tipos y obligatoriedad del body `POST /NewCollateralReport`: ver `reference.md` (sección `NewCollateralReport`).

### `POST /AuthToken/AuthToken`

En la práctica, seguir mismos criterios base que GET para `401`/`429` y loguear body completo ante error.

## Notas

- No hay lista publicada de códigos de negocio en body para GET más allá de `Code` en envelope.
- Catálogo de errores de `NewCollateralReport` cargado desde tabla operativa de análisis.
- El `Status` de `NewCollateralReport` en GET (`Inicial`, `Confirmado`, `Aprobado Riesgos`, `Procesado`, `Ejecutado`, `Anulado`) es estado de proceso, no código HTTP.
