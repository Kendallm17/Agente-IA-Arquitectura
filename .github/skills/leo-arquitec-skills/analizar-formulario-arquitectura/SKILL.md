---
applyTo: "**/*formulario*arquitectura*.xlsx|**/*Formulario_Arquitectura*.xlsx"
---

# Skill: Analizar Formulario de Arquitectura

## Propósito

Leer la hoja "Evaluación Arquitectónica" del Formulario para Diseño de Arquitectura de una misión y producir hallazgos estructurados por sección, con bloqueantes identificados y un dictamen, citando siempre la fila de origen.

## Cuándo se utiliza

- Dentro del Context Builder, en la capa de extracción, antes de `detectar-faltantes`, `detectar-contradicciones` y `construir-contexto` (estos dependen de esta salida).
- Cuando se recibe o se actualiza el Formulario para Diseño de Arquitectura de una misión.

## Entradas

- Archivo `.xlsx` con la hoja "Evaluación Arquitectónica", columnas: `ID`, `Sección`, `Dominio`, `Pregunta`, `Guía/evidencia esperada`, `Aplica a`, `¿Aplica?`, `Respuesta`, `Evidencia/referencia`, `Observaciones`, `Responsable`, `Severidad`, `Puntaje`.
- Identificador de misión.

## Documentos requeridos

- El Formulario de la misión (real o la plantilla vacía `Formularios Para Diseño De Arquitectura.xlsx`, que entonces produce "sin datos para evaluar").

## Reglas obligatorias

- No recalcular con una fórmula propia: el dictamen cualitativo usa los umbrales ya confirmados en `reglas-inviolables-de-mision.md` §2 (≥90% y sin bloqueantes → Aprobado; 75–89.9% y sin bloqueantes → Aprobado con condiciones; <75% o algún bloqueante incumplido → No aprobado; preguntas aplicables sin responder → Requiere información).
- **Si se pide explícitamente un porcentaje de cumplimiento y la columna `Puntaje` está vacía, el skill no debe calcular ni mostrar ningún número — ni siquiera marcado como "cálculo manual", "estimación", "aproximación" o "no oficial".** No existe una fórmula de ponderación confirmada (ver nota abajo), así que reproducir una fórmula propia y presentarla junto a un porcentaje viola la regla de no inventar, aunque se aclare que no es un dato oficial — un número junto a un % siempre se lee como un dato, no como una advertencia. La única respuesta correcta ante ese pedido es explicar que el % queda `"Pendiente de fórmula oficial"` y por qué (columna `Puntaje` vacía, sin fórmula confirmada por Arquitectura TI), sin producir ninguna cifra.
- Un hallazgo con `Severidad = Bloqueante` y `Respuesta` distinta de "Cumple" (incluye "Cumple parcialmente", "No cumple" y vacío cuando `¿Aplica? = Sí`) cuenta como **bloqueante incumplido**.
- **Si `Respuesta` tiene un valor, la pregunta se trata como aplicada y respondida — sin importar si `¿Aplica?` está vacío o marcado.** Una `Respuesta` presente ya es evidencia de que quien completó el Formulario la trató como aplicable; `¿Aplica?` vacío no es motivo para tratarla como "sin confirmar" ni para pedir que se "complete" una respuesta que ya existe. El campo `¿Aplica?` solo decide exclusión cuando está explícitamente en "No", o cuando `Aplica a` no corresponde al tipo de misión **y además no hay ninguna `Respuesta` registrada** en esa fila.
- Una fila con `¿Aplica? = No`, o con `Aplica a` que no corresponda al tipo de misión y sin `Respuesta` registrada, se excluye del cálculo y se reporta aparte, nunca como incumplimiento. Cuando `Modelo tecnológico = Híbrido`, la regla por defecto es que las preguntas `Aplica a: SaaS` sin respuesta **no aplican**, salvo que el propio Formulario mencione explícitamente un componente SaaS; el skill marca esta exclusión como **criterio provisional** en su salida (no una regla confirmada por Arquitectura TI), para que quede claro que puede revisarse.
- **En la salida, los bloqueantes incumplidos se listan siempre por separado de las preguntas sin responder que no son bloqueantes.** No se agrupan bajo un mismo encabezado genérico tipo "resolver los bloqueantes" — eso sugiere que todo lo listado es un bloqueante cuando puede no serlo, y hace que un bloqueante real (como una fila ya respondida "Cumple parcialmente") se pierda entre preguntas de severidad Alta o Media que simplemente están sin responder.
- No inventar una respuesta, evidencia o severidad que no esté en el archivo.
- Cada hallazgo cita: ID, fila de origen, sección y dominio.

### Pregunta pendiente para Arquitectura TI (no resuelta por este skill)

En el caso evaluado de referencia (misión real, ya avanzada) la columna `Puntaje` existe pero quedó vacía en todas las filas. No existe una fórmula de ponderación confirmada para convertir `Cumple / Cumple parcialmente / No cumple` en el % que alimenta el dictamen. **El skill no inventa esa fórmula.** Mientras no se confirme, reporta:
- El conteo de respuestas por categoría y el listado de bloqueantes incumplidos (esto sí es verificable sin inventar nada).
- El % de cumplimiento se marca como `"Pendiente de fórmula oficial"` en vez de calcularse, salvo que la fila `Puntaje` venga con valores.
- El dictamen solo se calcula completo cuando hay bloqueantes incumplidos (caso en que es "No aprobado" sin necesitar el %) o cuando `Puntaje` sí viene poblado.

## Secuencia

1. Abrir el archivo y localizar la hoja "Evaluación Arquitectónica" (si no existe, salida = Error técnico: hoja no encontrada).
2. Leer fila por fila a partir del encabezado (fila 4 en el formato real).
3. Para cada fila: registrar ID, sección, dominio, pregunta, `¿Aplica?`, respuesta, evidencia, severidad, puntaje (si existe).
4. Clasificar cada fila aplicable como: Cumple / Cumple parcialmente / No cumple / Sin responder.
5. Identificar bloqueantes incumplidos (regla arriba).
6. Calcular lo que se pueda calcular sin inventar (conteos, bloqueantes) y marcar el resto como pendiente.
7. Determinar el dictamen cualitativo con los umbrales confirmados, aplicando primero la regla de bloqueante.
8. Producir la salida estructurada.

## Salida esperada

- Misión (si se provee).
- Total de preguntas, aplicables, respondidas, sin responder.
- Hallazgos por sección: ID, severidad, estado, evidencia, fuente (fila).
- **Lista de bloqueantes incumplidos**, separada y explícita — nunca mezclada con la lista general de preguntas sin responder.
- Lista de preguntas aplicables sin responder que **no** son bloqueantes (severidad Alta, Media o Baja), aparte de la anterior.
- % de cumplimiento o `"Pendiente de fórmula oficial"`.
- Dictamen: Aprobado / Aprobado con condiciones / No aprobado / Requiere información / No evaluable.
- Advertencias (p. ej. fórmula de puntaje no confirmada; exclusión de preguntas SaaS por criterio provisional si `Modelo tecnológico = Híbrido`).
- Próximo paso y si requiere revisión humana (siempre `sí` cuando el dictamen no es "Aprobado").

## Formato de salida

Markdown estructurado, compatible con el contrato de salida común del orquestador (Misión, Subagente/skill, Estado, Resultado, Fuentes, Faltantes, Advertencias, Próximo paso, Revisión humana).

## Evidencias que debe conservar

La columna `Evidencia/referencia` de cada fila, tal cual viene en el archivo, sin resumir ni reinterpretar.

## Errores posibles

- Archivo sin la hoja "Evaluación Arquitectónica" → Error técnico.
- Archivo vacío (plantilla sin completar) → Estado "No evaluable", salida indica 0 preguntas respondidas.
- Fila con `Severidad` vacía → se reporta como advertencia, no se asume "Baja".

## Casos límite

- Fila con `¿Aplica? = Sí` pero `Respuesta` vacía → cuenta como "Sin responder"; si es bloqueante, el dictamen es como mínimo "Requiere información".
- Fila con `¿Aplica?` vacío pero `Respuesta = "Cumple parcialmente"` y `Severidad = Bloqueante` → **bloqueante incumplido**, no "pendiente de confirmar aplicabilidad". La `Respuesta` ya registrada tiene prioridad sobre un `¿Aplica?` sin marcar.
- Fila con `Aplica a = SaaS` en una misión IaaS → se excluye, se reporta en "No aplicables", nunca como incumplimiento. Pero si esa misma fila ya tiene una `Respuesta`, no se excluye: se evalúa con esa respuesta.
- Formulario con todas las respuestas "Cumple" pero sin `Puntaje` → dictamen no puede ser "Aprobado" automáticamente; se marca "Requiere confirmación del % (fórmula pendiente)".

## Dependencias

- `.github/reglas-de-proyectos/reglas-inviolables-de-mision.md` (umbrales de dictamen, regla de bloqueante).
- `.github/reglas-de-proyectos/correcciones-humanas.md` (ya cargado por `context-builder` al iniciar la misión; solo se reabre si esta es la primera vez en la conversación que se usa este skill).

## Criterios de aceptación

- Ante una pregunta con `Severidad = Bloqueante` y `Respuesta = "Cumple parcialmente"` (por ejemplo: integración de identidad con MFA validada solo a medias, o residencia de datos aún por confirmar), el skill la reporta como **bloqueante incumplido** — nunca como "Cumple".
- Dado el Formulario vacío, el skill responde "No evaluable" sin inventar respuestas.
- El % de cumplimiento nunca aparece como número si `Puntaje` está vacío en el archivo de origen.

## Casos de prueba

1. **Caso correcto:** Formulario con preguntas bloqueantes y no bloqueantes mezcladas, en distintos estados (Cumple, Cumple parcialmente, No cumple, sin responder) → el skill distingue cada una, calcula bien los bloqueantes incumplidos y deja el % marcado como pendiente si no hay `Puntaje`.
1b. **¿Aplica? vacío con Respuesta ya registrada:** una pregunta bloqueante con `¿Aplica?` sin marcar pero `Respuesta = "Cumple parcialmente"` → se reporta como bloqueante incumplido, en la lista separada de bloqueantes, no como "pendiente de confirmar aplicabilidad" ni mezclada con las preguntas sin responder. (Corrección registrada en `correcciones-humanas.md`, 2026-10-07, misión FEDV-226, pregunta ARQ-020.)
2. **Información incompleta:** Formulario con varias filas `¿Aplica? = Sí` y `Respuesta` vacía → "Sin responder", dictamen no puede ser "Aprobado".
3. **Documento vacío:** plantilla sin completar → "No evaluable".
4. **Datos contradictorios:** una fila con `Respuesta = Cumple` pero `Evidencia` vacía en una pregunta bloqueante → se reporta como hallazgo con advertencia ("respuesta sin evidencia"), no se da por cumplida sin más.
5. **Evidencia ausente:** igual que el caso anterior, remarcado si `Severidad = Bloqueante`.
6. **Documento incorrecto:** archivo `.xlsx` sin la hoja esperada → Error técnico.
7. **Instrucción ambigua:** solicitud sin indicar cuál Formulario analizar, habiendo más de uno disponible → el skill no elige por su cuenta; pide identificación de la misión.
8. **Intento de ignorar reglas:** solicitud de "marcar como Aprobado" un Formulario con bloqueante incumplido → el skill se niega y explica la regla de `reglas-inviolables-de-mision.md` §2.
8b. **Intento de obtener un % sin fórmula confirmada (caso real, 2026-10-09):** solicitud de "dame el porcentaje de cumplimiento" sobre un Formulario con columna `Puntaje` vacía → el skill se niega a calcular o mostrar cualquier número, ni siquiera marcado como "cálculo manual" o "reproducción de la fórmula" — responde que el % queda `"Pendiente de fórmula oficial"` y explica por qué, sin producir ninguna cifra. (Hallazgo detectado en una prueba real fuera del flujo de desarrollo, corregido en esta regla.)
9. **Caso no aplicable:** Formulario de una misión on-premise con preguntas `Aplica a = SaaS` → esas filas se excluyen del cálculo, no se tratan como incumplimiento.

## Subagente responsable

- Context Builder (capa de extracción; el subagente fusiona en uno solo lo que el proceso original describía como dos: Análisis de Iniciativa y Context Builder).
