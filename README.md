# a3-ai-skill-cauciones

Skill documental para asistir integraciones de backoffice con la API Post-Trade de A3 en el marco de la fusión entre Matba Rofex y MAE (dando origen a A3 Mercados) y la migración en curso de **Cauciones**, donde se migrará la operatoria desde SIOPEL a E-trader.

## Qué cubre

- Identificación de operaciones de caución (`CAUC`, `RPXXXX`, `CAU-ARS`/`CAU-USD`).
- Lectura de operaciones, márgenes y garantías por endpoint.
- Instrucciones de garantías (`NewCollateralReport`), campos FCI y seguimiento de estado.
- Detalle de cuenta (`AccountDetails`) por código de registro.
- Conciliación de movimientos (`MT536`) y stock final (`MT506`).
- Reglas y gotchas de implementación (semántica de `Account`, uso de `MarketID`, etc.).

## Para qué usar este skill

Usalo cuando el proveedor ya integra la API para otros productos y necesita adaptar/extender a Cauciones sin reimplementar todo.

No impone un flujo único: cada proveedor puede implementar los endpoints que necesite según su arquitectura.

## Disclaimer

- Esta skill resume y organiza la documentación de integración de Cauciones.
- Ante dudas o casos no cubiertos, validar siempre contra la documentación oficial de A3 y canales de soporte.
- Este material acelera integraciones, pero no reemplaza la validación funcional/técnica del equipo integrador.

## Archivo principal

- `SKILL.md`: punto de entrada del skill (scope, reglas, endpoints y referencias).

## Instalación / uso

Instalar copiando este directorio como skill en tu entorno de agente (Cursor/Claude u otro cliente compatible con skills Markdown).

La **ruta de instalación depende de la convención del cliente** (lista completa: `https://agentskills.io/clients`).

Ejemplos:

```bash
# Claude Code
git clone <repo-url> .claude/skills/a3-ai-skill-cauciones

# Otros clientes compatibles con agentskills.io (ej.: Cursor; lista completa en https://agentskills.io/clients)
git clone <repo-url> .agents/skills/a3-ai-skill-cauciones
```

Requisito importante:

- el nombre de carpeta del skill debe ser `a3-ai-skill-cauciones`,
- y debe coincidir con `name` en el frontmatter de `SKILL.md`.

## Referencias internas

- `references/reference.md`: parámetros y campos por endpoint.
- `references/diccionario-campos.md`: diccionario consolidado de campos (definición técnica + lectura de negocio).
- `references/glosario.md`: definiciones de negocio.
- `references/buenas-practicas.md`: horarios, polling y throttling.
- `references/errores-http.md`: manejo HTTP GET/POST.
- `references/examples.md`: payloads y ejemplos operativos.
- `references/circuito-cauciones-explicado.md`: lectura funcional del ciclo de vida por endpoint.

## Versionado

- Esquema: SemVer.
- Versión actual: `1.0.0`.
- Historial de cambios: `CHANGELOG.md`.

## Fuente funcional del contenido

- Manual oficial A3 (PrimaryAPI-BO):  
[https://apihub.primary.com.ar/assets/docs/PrimaryAPI-BO.pdf .](https://apihub.primary.com.ar/assets/docs/PrimaryAPI-BO.pdf)
- Manual oficial A3, específico de la migración de Cauciones: [https://a3mercados.com.ar/docs/cauciones-api-post-trade-a3/ .](https://a3mercados.com.ar/docs/cauciones-api-post-trade-a3/)

## Soporte

- Contacto operativo: `mpi@primary.com.ar`
- Issues / sugerencias: vía repositorio GitHub: [https://github.com/matbarofex/a3-ai-skill-cauciones .](https://github.com/matbarofex/a3-ai-skill-cauciones)

## Licencia

- `Proprietary` (ver frontmatter en `SKILL.md`).

