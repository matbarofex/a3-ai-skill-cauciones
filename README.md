# a3-ai-skill-cauciones

Skill documental para asistir integraciones de backoffice con la API Post-Trade de A3 en el dominio de **Cauciones**.

## Qué cubre

- Identificación de operaciones de caución (`CAUC`, `RPXXXX`, `CAU-ARS`/`CAU-USD`).
- Lectura de operaciones, márgenes y garantías por endpoint.
- Instrucciones de garantías (`NewCollateralReport`) y seguimiento de estado.
- Conciliación de movimientos (`MT536`) y stock final (`MT506`).
- Reglas y gotchas de implementación (semántica de `Account`, uso de `MarketID`, etc.).

## Para qué usar este skill

Usalo cuando el proveedor ya integra la API para otros productos y necesita adaptar/extender a Cauciones sin reimplementar todo.

No impone un flujo único: cada proveedor puede implementar los endpoints que necesite según su arquitectura.

## Archivo principal

- `SKILL.md`: punto de entrada del skill (scope, reglas, endpoints y referencias).

## Referencias internas

- `references/reference.md`: parámetros y campos por endpoint.
- `references/glosario.md`: definiciones de negocio.
- `references/buenas-practicas.md`: horarios, polling y throttling.
- `references/errores-http.md`: manejo HTTP GET/POST.
- `references/examples.md`: payloads y ejemplos operativos.
- `circuito-cauciones-explicado.md`: lectura funcional del ciclo de vida por endpoint.

## Fuente funcional del contenido

- Manual oficial A3 (PrimaryAPI-BO):  
  [https://apihub.primary.com.ar/assets/docs/PrimaryAPI-BO.pdf](https://apihub.primary.com.ar/assets/docs/PrimaryAPI-BO.pdf)

## Soporte

- Contacto operativo: `mpi@primary.com.ar`
