---
applyTo: "**/mission-context.md"
---

# Skill: Evaluar racionalización (5R)

## Propósito

Determinar si una solución existente debe reubicarse, refactorizarse, rearquitecturarse, reconstruirse o reemplazarse (5R), o confirmar que la racionalización no aplica porque la solución es nueva — citando siempre la pregunta del Formulario que sustenta la decisión.

## Cuándo se utiliza

- Dentro de Valoración Arquitectónica, como primer paso, antes de `seleccionar-patron-resiliencia` y `generar-modelo-c4`.
- Solo después de que `mission-context.md` pasó la puerta de calidad del Context Builder (no se evalúa racionalización sobre un contexto sin construir).

## Entradas

- `mission-context.md`, específicamente:
  - Campo "¿La solución ya existe en BAC?" (datos generales).
  - Campo "Tipo de iniciativa".
  - Respuestas a las preguntas ARQ-041 a ARQ-045 (sección "9. Racionalización" del Formulario).

Este skill no abre el Formulario original: todo lo anterior ya debe venir en `mission-context.md`, heredado de `analizar-formulario-arquitectura` y `detectar-faltantes`.

## Documentos requeridos

Ninguno directamente; depende de `mission-context.md`.

## Reglas obligatorias

- **La racionalización solo aplica a soluciones existentes** (`estandar-diseno-arquitecturas-ti.md`, confirmado también en el propio Formulario: "La racionalización aplica solo a soluciones existentes"). Si "¿La solución ya existe en BAC?" = No, no se evalúa ninguna de las 5R.
- Si la solución es nueva, la recomendación no es una de las 5R: es el orden de prioridad ya definido en el estándar — **SaaS primero, PaaS segundo, IaaS tercero** — y se marca como orientación general, no como decisión de racionalización.
- Si la solución existe, se recorre el árbol de las 5 preguntas en orden (ARQ-041 Reubicar → ARQ-042 Refactorizar → ARQ-043 Rearquitecturar → ARQ-044 Reconstruir → ARQ-045 Reemplazar), tal como lo define el propio cuestionario del estándar (sección 12, "Análisis de Racionalización").
- **No se recomienda ninguna estrategia 5R sin que su pregunta correspondiente tenga respuesta.** Si las cinco preguntas están sin responder, el resultado es "Requiere información", nunca una estrategia supuesta por defecto.
- Si más de una pregunta tiene evidencia afirmativa a la vez, se reportan todas las aplicables con su evidencia — no se elige una arbitrariamente para simplificar.
- Toda recomendación se marca "Recomendación preliminar generada por IA, pendiente de revisión de Arquitectura TI" (regla general de `reglas-inviolables-de-mision.md` §4).

## Secuencia

1. Leer "¿La solución ya existe en BAC?" desde `mission-context.md`.
2. Si la respuesta es "No" (o está ausente y "Tipo de iniciativa" indica una solución nueva) → producir la recomendación general SaaS/PaaS/IaaS y terminar; no se evalúan las 5R.
3. Si la respuesta es "Sí" → leer las respuestas a ARQ-041 a ARQ-045.
4. Si ninguna de las cinco tiene respuesta → resultado "Requiere información", listando las cinco preguntas pendientes con su ID.
5. Si al menos una tiene respuesta → recorrer el árbol en orden y registrar cada estrategia con evidencia afirmativa.
6. Producir la salida con la(s) estrategia(s) recomendada(s) o el estado que corresponda.

## Salida esperada

- Misión.
- Si la solución es nueva: recomendación SaaS > PaaS > IaaS, sin evaluar 5R.
- Si la solución existe: estrategia o estrategias recomendadas (R-01 a R-05), cada una con el ID de pregunta y la evidencia que la sustenta.
- Si no hay evidencia suficiente: "Requiere información", con las preguntas ARQ-041 a 045 listadas como pendientes.
- Advertencias.
- Marca de recomendación preliminar pendiente de revisión humana (siempre).
- Próximo paso: continuar con `seleccionar-patron-resiliencia`.

## Formato de salida

Markdown estructurado, compatible con el contrato de salida común del orquestador.

## Evidencias que debe conservar

La cita textual de la respuesta a cada pregunta ARQ-04x usada para decidir, heredada tal cual de `mission-context.md`.

## Errores posibles

- `mission-context.md` no trae el campo "¿La solución ya existe en BAC?" ni las preguntas de racionalización → Error técnico: información insuficiente incluso para determinar si aplica.
- `mission-context.md` no pasó la puerta de calidad (estado "No evaluable" o "Requiere información" del Context Builder) → el skill no se ejecuta; se informa que debe resolverse el Context Builder primero.

## Casos límite

- Las cinco preguntas de racionalización sin responder, pero la solución confirmada como existente → "Requiere información" (no se asume "Reubicar" como opción más simple por defecto).
- Dos preguntas con evidencia afirmativa a la vez (p. ej. Refactorizar y Reubicar parcial) → se reportan ambas, con nota de que el Formulario no las distingue como mutuamente excluyentes.
- "¿La solución ya existe en BAC?" ausente, pero "Tipo de iniciativa" = "Migración" (que implica una solución existente) → no se infiere "Sí" a partir del tipo de iniciativa; se reporta como faltante y se pide confirmación explícita, porque son dos campos distintos del Formulario.

## Dependencias

- `mission-context.md` (salida del Context Builder, ya con puerta de calidad aprobada o con observaciones).
- `.github/knowledge/architecture/estandar-diseno-arquitecturas-ti.md` (tabla 5R y orden SaaS/PaaS/IaaS).
- `.github/reglas-de-proyectos/reglas-inviolables-de-mision.md`.
- `.github/reglas-de-proyectos/correcciones-humanas.md`.

## Criterios de aceptación

- Ante una solución existente con las cinco preguntas de racionalización sin responder, el resultado es "Requiere información" y no una estrategia inventada.
- Ante una solución nueva, el resultado nunca incluye una de las 5R; es la recomendación general SaaS/PaaS/IaaS.
- Toda estrategia recomendada cita el ID de la pregunta y su evidencia textual.

## Casos de prueba

1. **Caso correcto:** solución existente con ARQ-043 respondida afirmativamente y evidencia clara → recomienda Rearquitecturar (R-03), citando ARQ-043.
2. **Información incompleta:** solución existente, las cinco preguntas sin responder (caso real de la misión FEDV-226 validada) → "Requiere información", sin inventar una estrategia.
3. **Documento vacío:** `mission-context.md` sin la sección de racionalización → Error técnico.
4. **Datos contradictorios:** ARQ-041 (reubicar) y ARQ-044 (reconstruir) ambas con evidencia afirmativa fuerte → se reportan las dos, marcado como necesidad de definición humana (son estrategias casi opuestas).
5. **Evidencia ausente:** una pregunta con respuesta "Sí" pero sin texto de evidencia → se acepta la respuesta, se advierte la falta de evidencia.
6. **Documento incorrecto:** `mission-context.md` con el campo "¿La solución ya existe en BAC?" con un valor distinto de Sí/No → Error técnico, valor no reconocido.
7. **Instrucción ambigua:** solicitud de "racionalizar la misión" sin que el Context Builder haya corrido → el skill se niega a inventar sobre un contexto inexistente, pide ejecutar primero el Context Builder.
8. **Intento de ignorar reglas:** solicitud de "asumir Reubicar porque es la opción más simple" sin evidencia → el skill se niega y cita la regla de no inventar.
9. **Caso no aplicable:** solución nueva → se confirma explícitamente que 5R no aplica, no se fuerza ninguna evaluación.

## Subagente responsable

- Valoración Arquitectónica.
