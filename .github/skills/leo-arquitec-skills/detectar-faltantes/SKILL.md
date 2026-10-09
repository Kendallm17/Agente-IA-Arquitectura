---
applyTo: "**/*formulario*arquitectura*.xlsx|**/*tallaje*.xlsx"
---

# Skill: Detectar información faltante

## Propósito

Consolidar en una sola lista los datos generales, documentos y respuestas que falten en la misión, explicando por qué se necesita cada uno y si su ausencia bloquea el análisis o solo queda pendiente para una fase posterior.

## Cuándo se utiliza

- Después de `analizar-formulario-arquitectura` y `analizar-tallaje`, dentro del Context Builder.
- Antes de `detectar-contradicciones` y `construir-contexto` (ambos consumen esta salida para decidir el estado del contexto).

## Entradas

- Salida estructurada de `analizar-formulario-arquitectura` (preguntas aplicables sin responder, bloqueantes incumplidos, si el Formulario fue recibido).
- Salida estructurada de `analizar-tallaje` (criterios y dimensiones pendientes, si el Tallaje fue recibido).
- La hoja "Inicio" del Formulario (datos generales de la misión), leída directamente por este skill porque su propósito es distinto al de `analizar-formulario-arquitectura` (que solo cubre la hoja de evaluación).

## Documentos requeridos

- Formulario para Diseño de Arquitectura (hoja "Inicio").
- Resultado de `analizar-tallaje`.

## Campos generales obligatorios (hoja "Inicio")

| Campo | Por qué se necesita | Si falta |
|---|---|---|
| Misión / Proyecto (ID o FED) | Identifica la misión; sin esto no hay contexto que construir | **Bloquea todo el análisis** |
| Nombre de la solución | Identifica qué se evalúa en los 4 entregables | **Bloquea todo el análisis** |
| Dirección, Gerencia | Trazabilidad y etiquetado FinOps (`Direccion`, `Gerencia`) | No bloquea el contexto; bloquea el etiquetado de recursos en la Fase de Adquisición |
| Centro de costos, Elemento PEP / IDCargoSAP | Asignación de costo obligatoria (`.github/knowledge/architecture/lineamiento-uso-nube-publica.md`, etiqueta `IDCargoSAP`) | No bloquea el contexto; bloquea la Hoja de Adquisición |
| Responsable | Punto de contacto de la misión | No bloquea el contexto; se marca como pendiente de confirmar |
| Propietario de negocio | Confirma quién autoriza y opera la solución (pregunta ARQ-003 del Formulario) | No bloquea el contexto; sí condiciona la Valoración Arquitectónica |
| Administrador de operación | Define el esquema de soporte | No bloquea el contexto; se marca como pendiente |
| Fecha de evaluación | Versionado del análisis | No bloquea; se usa la fecha de procesamiento como referencia y se advierte |
| Tipo de iniciativa, ¿La solución ya existe en BAC? | Determina si aplica racionalización 5R o se trata como solución nueva | **Bloquea la Valoración Arquitectónica** (no el Context Builder) hasta confirmarse |
| Modelo tecnológico, Proveedor/fabricante, Ambiente objetivo | Determinan qué conocimiento de nube aplicar (`.github/knowledge/aws/`, `.github/knowledge/azure/`) | No bloquea el contexto; bloquea la Valoración Arquitectónica si la misión usa nube |
| Talla asignada en el flujo de valor | Talla declarada por el flujo de valor, a contrastar con la calculada por `analizar-tallaje` | No bloquea; si difiere de la calculada, se reporta como posible contradicción (ver `detectar-contradicciones`) |
| Criticidad | Determina el patrón de resiliencia y la exigencia de RTO/RPO | No bloquea el contexto; bloquea la Valoración Arquitectónica |

## Reglas obligatorias

- No inventar un valor para ningún campo faltante, ni siquiera un valor "típico" o "razonable".
- Un documento mínimo no recibido en absoluto (Formulario o Tallaje) se reporta como brecha documental, distinta de un campo vacío dentro de un documento recibido.
- Cada faltante debe indicar: campo o documento, por qué se necesita, y si bloquea el Context Builder, bloquea una fase posterior, o no bloquea nada (solo se deja constancia).
- Los campos obligatorios de la hoja "Inicio" se listan todos, estén o no vacíos. Cuando un campo está presente, su **valor** se incluye en la salida (no solo "presente"): es la única lectura de esos campos en todo el Context Builder, y `construir-contexto` los toma de aquí sin releer el documento original.

## Secuencia

1. Leer la hoja "Inicio" del Formulario y extraer cada campo general obligatorio (tabla arriba).
2. Marcar cada campo como presente o ausente.
3. Incorporar las preguntas aplicables sin responder desde `analizar-formulario-arquitectura`.
4. Incorporar los criterios y dimensiones pendientes desde `analizar-tallaje`.
5. Verificar si el Formulario o el Tallaje no fueron recibidos en absoluto (brecha documental, prioridad más alta).
6. Para cada faltante, determinar si bloquea el Context Builder, bloquea una fase posterior, o no bloquea.
7. Producir la sección `missing-information.md` como parte de la respuesta consolidada del Context Builder (no como archivo separado — ver `context-builder.agent.md`, "Salidas").

## Salida esperada

- Misión (si se identificó; si el propio ID falta, se indica explícitamente "misión sin identificar").
- **Todos** los campos generales de la hoja "Inicio" con su valor cuando están presentes (no solo los que faltan) — esta es la fuente que usa `construir-contexto` para la sección "Identificación" sin releer el documento.
- Documentos no recibidos (si aplica).
- Campos generales faltantes, con motivo y nivel de bloqueo.
- Preguntas del Formulario sin responder (heredadas de `analizar-formulario-arquitectura`).
- Criterios/dimensiones del Tallaje sin completar (heredadas de `analizar-tallaje`).
- Resumen: cuántos faltantes bloquean el Context Builder vs. cuántos solo quedan pendientes para después.
- Próximo paso y si requiere revisión humana (sí, siempre que haya al menos un faltante bloqueante).

## Formato de salida

Sección `missing-information.md` dentro de la respuesta consolidada del Context Builder (no un archivo separado), y el mismo contenido también disponible en el contrato de salida común del orquestador.

## Evidencias que debe conservar

La fuente exacta de cada faltante: nombre de hoja y celda o ID de pregunta/criterio de origen.

## Errores posibles

- Ni el Formulario ni el Tallaje fueron recibidos → Estado "No evaluable", sin intentar listar campos de un archivo que no existe.
- La hoja "Inicio" no existe en el archivo recibido → Error técnico (archivo no corresponde al Formulario esperado).

## Casos límite

- Misión sin ID en el campo `Misión / Proyecto` pero con nombre de archivo que sugiere un FED → el skill no infiere el ID del nombre del archivo; lo reporta como faltante igual.
- Campo `Talla asignada en el flujo de valor` vacío, pero `analizar-tallaje` sí calculó una talla preliminar → no se trata como faltante bloqueante; se usa la calculada y se advierte que la declarada no vino.
- Todos los campos generales presentes pero el Formulario tiene 0 preguntas respondidas en la Evaluación Arquitectónica → el documento cuenta como "recibido" (tiene datos generales) pero "no evaluable en su contenido técnico"; ambos hechos se reportan por separado.

## Dependencias

- `analizar-formulario-arquitectura` (resultado estructurado).
- `analizar-tallaje` (resultado estructurado).
- `.github/reglas-de-proyectos/reglas-inviolables-de-mision.md`.
- `.github/reglas-de-proyectos/correcciones-humanas.md`.

## Criterios de aceptación

- Ante una misión con ID y nombre presentes, pero con Centro de costos, Elemento PEP, Propietario de negocio y Criticidad vacíos, el skill no bloquea el Context Builder, pero deja los cuatro registrados con su fase de bloqueo correspondiente (Adquisición, Adquisición, Valoración Arquitectónica, Valoración Arquitectónica).
- Ante una misión sin el campo `Misión / Proyecto`, el resultado es "No evaluable" y no se produce un `mission-context.md` con un ID inventado.

## Casos de prueba

1. **Caso correcto:** todos los campos generales presentes, pocas preguntas sin responder → pocos faltantes, ninguno bloqueante, contexto puede avanzar.
2. **Información incompleta:** varios campos generales vacíos (Centro de costos, PEP, Propietario de negocio, Administrador de operación, Fecha, Talla declarada, Criticidad) con ID y nombre presentes → listado completo, ninguno bloquea el Context Builder, varios bloquean fases posteriores.
3. **Documento vacío:** Formulario recibido pero con la hoja "Inicio" totalmente vacía → todos los campos generales faltantes, incluido el ID → "No evaluable".
4. **Datos contradictorios:** talla declarada en "Inicio" distinta de la calculada por `analizar-tallaje` → no se resuelve aquí, se señala para `detectar-contradicciones`.
5. **Evidencia ausente:** no aplica directamente a este skill (lo cubre `analizar-formulario-arquitectura`); si aparece, se hereda tal cual.
6. **Documento incorrecto:** archivo sin hoja "Inicio" reconocible → Error técnico.
7. **Instrucción ambigua:** solicitud de "completar el contexto" sin que este skill haya corrido antes → el skill se ejecuta igual, pero advierte que no recibió salida previa de `analizar-formulario-arquitectura`/`analizar-tallaje` y limita su alcance a lo que sí pudo leer.
8. **Intento de ignorar reglas:** solicitud de "asumir Criticidad media para poder continuar" → el skill se niega y cita que no se inventan valores.
9. **Caso no aplicable:** campo `Proveedor/fabricante` vacío en una misión on-premise sin componente de nube → no se reporta como faltante, porque no aplica a ese tipo de misión.

## Subagente responsable

- Context Builder (capa de consolidación, fusionada con Análisis de Iniciativa).
