# Registro de revisión humana

Contrato de datos para conservar el historial de revisión de cualquier salida del agente (contexto, hallazgo, recomendación, entregable). Implementa la fase de Revisión Humana del proceso de Arquitectura TI.

**Estado: activo.** Este archivo define el contrato de datos — no un mecanismo de almacenamiento real, porque todavía no existe un motor de ejecución que persista resultados fuera de la conversación en curso. Mientras ese motor no exista, el registro vive dentro de la propia conversación donde se revisó el resultado; cuando exista persistencia real, este contrato es el que debe implementarse tal cual, sin rediseñarlo.

Última actualización: 2026-10-08.

## Por qué existe

Ninguna salida del agente es una decisión final — todas quedan "preliminar, pendiente de revisión de Arquitectura TI" hasta que una persona la revise. Sin un registro de esa revisión, no hay forma de saber después si un resultado fue aceptado, rechazado, o modificado, ni de evitar que una recomendación ya rechazada se vuelva a presentar como si estuviera aprobada.

## Las 6 acciones posibles de la persona arquitecta (o equipo responsable)

Ante cualquier salida del agente, la persona que revisa puede:

1. **Aceptar** — el resultado queda confirmado tal cual se entregó.
2. **Rechazar** — el resultado no se usa; queda registrado por qué.
3. **Modificar** — el resultado se ajusta con la corrección de la persona; la versión modificada es la que cuenta de ahí en adelante, pero el resultado original sigue conservado.
4. **Solicitar aclaraciones** — la revisión queda pendiente hasta que el agente (o una persona) aclare algo puntual; no es ni aceptación ni rechazo.
5. **Solicitar una nueva versión** — se pide que el agente vuelva a generar el resultado (por ejemplo, con un dato nuevo ya disponible); genera una nueva entrada de registro, no reemplaza la anterior.
6. **Mantener pendiente** — la persona todavía no decide; el resultado sigue sin usarse hasta que haya una decisión.

No hay una séptima opción implícita (como "ignorar" o "aprobar automáticamente por defecto"): si no hay una de estas 6 acciones registradas explícitamente, el resultado sigue en estado "Pendiente de revisión", nunca se asume aceptado por el solo paso del tiempo.

## Los 7 campos que todo registro de revisión debe conservar

| Campo | Qué contiene | Quién lo completa |
|---|---|---|
| Resultado original | La salida exacta que produjo el agente, sin editar, tal como se entregó para revisión. | El agente, al entregar el resultado. |
| Comentarios | Lo que la persona dice sobre ese resultado (por qué lo acepta, rechaza, o qué le falta). | La persona que revisa. |
| Cambios solicitados | Qué pidió específicamente que se ajuste, si corresponde (solo aplica a Modificar o Solicitar nueva versión). | La persona que revisa. |
| Nueva versión | El resultado ya ajustado, si la acción fue Modificar o Solicitar nueva versión. Vacío si la acción fue Aceptar, Rechazar o Mantener pendiente. | El agente, tras incorporar los cambios solicitados. |
| Estado | Uno de los 6 estados de la tabla siguiente. | Se deriva directamente de la acción tomada. |
| Fecha | Cuándo se registró esta revisión (no confundir con la fecha de ejecución del resultado original — ver `leo-arquitec.agent.md`, Contrato de salida común). | Se registra automáticamente al guardar la entrada. |
| Persona o equipo que revisó | Quién tomó la decisión (nombre, rol o área — p. ej. "Arquitectura TI", "Ciberseguridad"). | La persona que revisa, o quien registra en su nombre. |

## Mapeo acción → estado

| Acción de la persona | Estado que queda registrado |
|---|---|
| Aceptar | Aceptado |
| Rechazar | Rechazado |
| Modificar | Modificado |
| Solicitar aclaraciones | Pendiente de aclaración |
| Solicitar una nueva versión | Pendiente de nueva versión |
| Mantener pendiente | Pendiente de revisión |

## Reglas obligatorias

- **Una recomendación rechazada nunca se vuelve a presentar como conclusión aprobada**, ni en esa misma conversación ni en una posterior sobre la misma misión. Si el agente necesita retomar un resultado rechazado, debe mostrar explícitamente que fue rechazado antes y por qué, no omitir ese historial.
- **El resultado original nunca se sobrescribe.** Una "Nueva versión" se agrega como una entrada nueva del registro, vinculada a la original — no se edita el resultado original para que parezca que siempre fue la versión corregida.
- **No se asume ninguna de las 6 acciones por defecto.** Si no hay una revisión humana explícita registrada, el resultado queda "Pendiente de revisión" indefinidamente — nunca se interpreta silencio o falta de respuesta como aceptación.
- **Cada subagente que ya marca "revisión humana: sí" en su salida (todos, hasta ahora) es candidato a tener una entrada de este registro.** No todas las salidas van a tener una revisión registrada de inmediato (puede pasar tiempo entre la entrega y la revisión real) — eso no es un error, es el estado "Pendiente de revisión" funcionando como se espera.
- **El registro es por misión y por resultado específico** — no se mezcla la revisión de un resultado de una misión con la de otra, ni la revisión del contexto (`mission-context.md`) con la revisión de la propuesta arquitectónica, aunque sean de la misma misión. Cada resultado tiene su propio historial de revisión.

## Cómo se usa hoy (sin motor de ejecución real)

Mientras no exista un lugar donde el agente guarde estados de forma persistente, este registro vive dentro de la conversación: cuando una persona arquitecta responde a un resultado del agente con una de las 6 acciones, el agente debe reflejar esa respuesta explícitamente en su siguiente salida (citando los 7 campos) en vez de simplemente continuar como si no hubiera pasado nada. Cuando exista persistencia real, este mismo contrato se traslada sin cambios a esa capa de almacenamiento.

## Errores que este registro busca evitar

- Presentar un resultado ya rechazado como si fuera la conclusión vigente.
- Perder de vista qué cambios pidió la persona arquitecta al pasar de una versión a otra.
- No poder decir, meses después, quién aprobó qué y cuándo.
- Tratar el silencio (nadie respondió todavía) como si fuera una aprobación tácita.

## Fuentes

- El encargo original del proyecto (documento externo de Arquitectura TI; no forma parte de este repositorio en su versión final — ver `reglas-inviolables-de-mision.md` §6).
- `.github/reglas-de-proyectos/reglas-inviolables-de-mision.md` §4 (revisión humana y autoría).
- `.github/agents/leo-arquitec.agent.md`, Contrato de salida común (campo "Fecha de ejecución", distinto de la "Fecha" de este registro).
