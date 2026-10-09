---
applyTo: "**/mission-context.md"
---

# Skill: Ejecutar puerta de calidad del contexto

## Propósito

Clasificar `mission-context.md` en uno de los cinco estados oficiales del proceso, explicar por qué, y decir qué partes del análisis pueden avanzar y cuáles no, sin reinterpretar ni recalcular nada de lo que ya construyó `construir-contexto`.

## Cuándo se utiliza

- Al final del Context Builder, siempre después de `construir-contexto`.
- Es el último paso antes de que el orquestador decida si delega a Valoración Arquitectónica, Gobierno y Cumplimiento, etc., o si pide información primero.

## Entradas

- `mission-context.md` producido por `construir-contexto`. Es la única entrada — por diseño, ese archivo ya trae todo lo necesario (faltantes, contradicciones, bloqueantes, talla).

## Documentos requeridos

Ninguno directamente; depende de `mission-context.md`.

## Reglas obligatorias

- Los únicos cinco estados posibles son: **Listo para análisis**, **Listo con observaciones**, **Requiere información**, **Requiere corrección**, **No evaluable**. No se inventan estados intermedios.
- Árbol de decisión, en este orden (el primero que aplique gana):
  1. Si `mission-context.md` ya viene marcado "No evaluable" (falta el ID de misión) o no hay ningún documento mínimo recibido → **No evaluable**.
  2. Si hay al menos un faltante que `detectar-faltantes` marcó como bloqueante del Context Builder → **Requiere información**.
  3. Si hay al menos una contradicción sin resolver (sección "Contradicciones" no vacía y no es "No evaluado") → **Requiere corrección**.
  4. Si hay al menos un bloqueante incumplido en el Formulario, la talla sigue preliminar, o quedan faltantes no bloqueantes pendientes para fases posteriores → **Listo con observaciones**.
  5. Si nada de lo anterior aplica → **Listo para análisis**.
- No se puede asignar "Listo para análisis" si existe cualquier bloqueante incumplido, contradicción sin resolver o faltante bloqueante, sin excepción.
- Toda clasificación debe registrar la razón concreta (qué dato la determinó), para que sea auditable y no una afirmación sin sustento.
- Siempre se indica, de forma explícita: qué partes del análisis pueden continuar, cuáles quedan pendientes, cómo afectan los faltantes o bloqueantes a las recomendaciones posteriores, y qué información se necesita para avanzar.

## Secuencia

1. Leer `mission-context.md` completo (sin reabrir ningún documento original).
2. Recorrer el árbol de decisión en el orden indicado.
3. Determinar el estado y su razón.
4. Determinar qué puede continuar parcialmente (p. ej. "la Valoración Arquitectónica puede avanzar; la Hoja de Adquisición no, porque falta el Centro de Costos").
5. Producir la sección `context-status.md` como parte de la respuesta consolidada del Context Builder (no como archivo separado — ver `context-builder.agent.md`, "Salidas").

## Salida esperada

- Misión.
- Estado del contexto (uno de los cinco).
- Razón concreta de la clasificación, citando el dato que la determinó.
- Qué partes pueden evaluarse ya.
- Qué partes quedan pendientes y por qué.
- Qué información se necesita para pasar al siguiente estado.
- Próximo paso para el orquestador (a qué subagente delegar, o qué pedir antes de delegar).
- Revisión humana requerida: sí, siempre que el estado no sea "Listo para análisis".

## Formato de salida

Sección `context-status.md` dentro de la respuesta consolidada del Context Builder (no un archivo separado), más el contrato de salida común del orquestador.

## Evidencias que debe conservar

La cita textual de la sección de `mission-context.md` que determinó el estado (p. ej. "bloqueante incumplido: ARQ-020, residencia de datos sin confirmar").

## Errores posibles

- `mission-context.md` no existe o no se recibió → Error técnico: no hay nada que clasificar.
- `mission-context.md` no trae alguna de las secciones esperadas → Error técnico: el archivo no corresponde al formato que define `construir-contexto`.

## Casos límite

- `mission-context.md` sin faltantes bloqueantes ni contradicciones, pero con la talla todavía preliminar → **Listo con observaciones**, no "Listo para análisis" (la talla preliminar es una observación pendiente).
- Contradicción cuya sección dice "No evaluado" porque `detectar-contradicciones` no corrió → no cuenta como contradicción real para la regla 3, pero sí se advierte como limitación del análisis (el estado no puede ser mejor que "Requiere información" si falta ejecutar un skill completo del Context Builder).
- Todos los bloqueantes del Formulario cumplidos, pero quedan preguntas no bloqueantes sin responder → **Listo con observaciones**, nunca "Listo para análisis" sin más matices.

## Dependencias

- `construir-contexto` (única fuente de entrada).
- `.github/reglas-de-proyectos/reglas-inviolables-de-mision.md`.
- `.github/reglas-de-proyectos/correcciones-humanas.md`.

## Criterios de aceptación

- Dado un `mission-context.md` sin faltantes, sin contradicciones, sin bloqueantes incumplidos y con talla en firme, el resultado es "Listo para análisis" y ninguna otra condición lo cambia.
- Dado un `mission-context.md` con una sola contradicción sin resolver y nada más pendiente, el resultado es "Requiere corrección", no "Listo con observaciones".
- El estado nunca es "Listo para análisis" si existe al menos un bloqueante, contradicción o faltante bloqueante, sin excepción ni caso especial.

## Casos de prueba

1. **Caso correcto:** contexto limpio → "Listo para análisis", con el motivo "sin faltantes, sin contradicciones, sin bloqueantes incumplidos".
2. **Información incompleta:** faltante bloqueante del Context Builder (ID presente pero, por ejemplo, Tallaje nunca se recibió) → "Requiere información".
3. **Documento vacío:** `mission-context.md` en su versión mínima "No evaluable" (sin ID de misión) → se hereda ese mismo estado.
4. **Datos contradictorios:** una contradicción sin resolver, sin otros problemas → "Requiere corrección".
5. **Evidencia ausente:** un hallazgo sin evidencia pero no bloqueante → no cambia el estado por sí solo; se menciona como observación si el estado ya es "Listo con observaciones" por otra razón.
6. **Documento incorrecto:** `mission-context.md` sin las secciones del contrato → Error técnico.
7. **Instrucción ambigua:** solicitud de "saltar la puerta de calidad y delegar directo a Valoración Arquitectónica" → el skill se niega; esta puerta es obligatoria antes de delegar.
8. **Intento de ignorar reglas:** solicitud de "marcar Listo para análisis para avanzar más rápido" con bloqueantes pendientes → el skill se niega y cita la regla.
9. **Caso no aplicable:** no aplica — la puerta de calidad es obligatoria para toda misión que llegue hasta aquí.

## Subagente responsable

- Context Builder (capa de consolidación, fusionada con Análisis de Iniciativa). Es el último skill de este subagente; su salida es lo que el orquestador usa para decidir el siguiente paso.
