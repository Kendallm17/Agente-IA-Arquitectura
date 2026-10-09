---
applyTo: "**/mission-context.md"
---

# Skill: Generar acciones correctivas

## Propósito

Completar, para cada riesgo ya identificado por `identificar-riesgos`, los campos que el contrato de la Fase 6 (Identificación de riesgos y brechas) exige y que ese primer skill deja explícitamente afuera: Impacto, Acción requerida, Condición de cierre, Estado, Información pendiente y Necesidad de revisión humana. También resuelve la deduplicación fina pendiente entre riesgos que `identificar-riesgos` dejó sin consolidar por falta de detalle (p. ej. varios bloqueantes del Formulario de la misma sección/dominio que en realidad describen un solo problema de fondo).

## Cuándo se utiliza

- Dentro de Evaluación de Riesgos, como segundo y último paso (por ahora).
- Solo después de que `identificar-riesgos` produjo su lista para la misma misión.

## Entradas

- La lista de riesgos de `identificar-riesgos` (Identificador, Categoría, Descripción, Evidencia, Documento de origen, Criterio aplicado).
- Las mismas tres fuentes que ya usó `identificar-riesgos` (`mission-context.md`, Valoración Arquitectónica, Gobierno y Cumplimiento), solo para verificar si contienen evidencia de impacto — no para identificar riesgos nuevos.

Este skill no vuelve a decidir si algo es un riesgo ni de qué categoría es — eso ya lo hizo `identificar-riesgos`. Tampoco reabre el Formulario o el Tallaje originales.

## Documentos requeridos

Ninguno directamente; depende de la salida de `identificar-riesgos`.

## Reglas obligatorias

- **El Impacto (financiero, operativo, reputacional, legal) solo se completa si hay evidencia textual que lo sustente** en alguna de las tres fuentes originales. Si no hay esa evidencia, el campo queda **"No evaluable con los insumos disponibles"** — nunca se infiere un impacto "típico" para ese tipo de riesgo (regla de `reglas-inviolables-de-mision.md` §1: no inventar riesgos, acuerdos ni cifras sin evidencia).
- **La Acción requerida se redacta como una tarea concreta y verificable**, nunca como una recomendación genérica. Ejemplo correcto: "Confirmar la implementación de MFA en accesos administrativos del componente X y documentar la evidencia en el Formulario." Ejemplo incorrecto (prohibido): "Mejorar la seguridad del sistema."
- **El Estado de un riesgo nunca puede ser "Cerrado" por este skill.** Los únicos estados posibles son "Abierto" (riesgo activo, acción pendiente) y "Pendiente de revisión humana" (cuando la acción ya se propuso pero necesita validación de Arquitectura TI, Ciberseguridad o el área correspondiente antes de ejecutarse). Cerrar un riesgo es una decisión humana exclusiva, nunca generada por el agente.
- **Condición de cierre:** se redacta como un criterio objetivo y verificable (ej. "ARQ-014 pasa a Respuesta = 'Cumple' con evidencia de MFA activo"), nunca como "cuando se resuelva el problema".
- **Deduplicación fina:** si dos o más riesgos de `identificar-riesgos` comparten la misma Categoría y describen, en el fondo, el mismo problema institucional (p. ej. varios bloqueantes de la sección "4. Seguridad" que todos apuntan a falta de evidencia de controles de acceso), se consolidan en un solo riesgo citando todas las evidencias originales — nunca se fusionan riesgos de categorías distintas ni riesgos que, aunque relacionados, describen hechos diferentes (ver caso límite de `identificar-riesgos`: MFA y residencia de datos no se fusionan aunque ambos sean de Seguridad).
- Toda salida se marca "Acción correctiva preliminar generada por IA, pendiente de revisión de Arquitectura TI" (regla general de `reglas-inviolables-de-mision.md` §4).
- No eliminar ni ocultar un riesgo de `identificar-riesgos` en el proceso de deduplicación: todo riesgo original queda trazable dentro del riesgo consolidado, citando su identificador previo.

## Secuencia

1. Leer la lista de riesgos de `identificar-riesgos`.
2. Revisar si hay riesgos de la misma categoría que describan el mismo problema de fondo; si los hay, consolidarlos en uno, conservando la trazabilidad de los identificadores originales.
3. Para cada riesgo (consolidado o individual), buscar evidencia de impacto en las tres fuentes originales.
4. Si hay evidencia de impacto, completarlo citando la fuente; si no, marcar "No evaluable con los insumos disponibles".
5. Redactar la Acción requerida como tarea concreta y verificable.
6. Redactar la Condición de cierre como criterio objetivo.
7. Asignar Estado ("Abierto" o "Pendiente de revisión humana") según si la acción ya puede ejecutarse o necesita validación previa.
8. Marcar Información pendiente y Necesidad de revisión humana (siempre "sí" en esta última).
9. Producir la salida consolidada, lista para un futuro skill de "Completar plantilla de riesgos".

## Salida esperada

- Misión.
- Lista de riesgos consolidada: Identificador (conservando trazabilidad si se fusionaron), Categoría, Descripción, Evidencia, Documento de origen, Criterio aplicado (heredados), más Impacto, Acción requerida, Condición de cierre, Estado, Información pendiente, Necesidad de revisión humana (nuevos).
- Advertencias (impactos no evaluables por falta de evidencia, riesgos consolidados y por qué).
- Próximo paso: esta lista queda lista para alimentar un futuro skill de "Completar plantilla de riesgos" (Entregable B) y para Adquisición y Costes si algún riesgo requiere un componente adicional.
- Revisión humana: siempre `sí`.

## Formato de salida

Markdown estructurado, compatible con el contrato de salida común del orquestador.

## Evidencias que debe conservar

La cita textual de cualquier evidencia de impacto usada, y la lista de identificadores originales de `identificar-riesgos` que se consolidaron en cada riesgo final.

## Errores posibles

- `identificar-riesgos` no corrió todavía para esta misión → el skill no se ejecuta; se informa que debe ejecutarse primero.
- La lista de `identificar-riesgos` está vacía ("Sin riesgos identificados") → este skill no tiene nada que procesar; se informa y termina, no se inventan riesgos para tener algo que mostrar.

## Casos límite

- Un riesgo con evidencia de incumplimiento pero sin ningún dato de impacto financiero/operativo/reputacional/legal en ninguna de las tres fuentes → Impacto = "No evaluable con los insumos disponibles" en las cuatro dimensiones, no se deja vacío sin explicación.
- Dos riesgos de Seguridad que en realidad son el mismo hecho reportado por fuentes distintas (ya consolidado por `identificar-riesgos`, pero con algún detalle adicional que solo aparece en una fuente) → se verifica que no se haya perdido ese detalle al consolidar.
- Un riesgo cuya Acción requerida depende de que Arquitectura TI tome una decisión de negocio antes de poder redactarse como tarea concreta (p. ej. "decidir si se acepta el riesgo o se exige remediación") → Estado = "Pendiente de revisión humana", Acción requerida = "Someter a decisión de Arquitectura TI: aceptar el riesgo o exigir remediación antes de aprobar la misión."
- Un riesgo Financiero sin ninguna fuente de costeo (Adquisición y Costes no existe todavía) → Impacto financiero = "No evaluable con los insumos disponibles — depende de Adquisición y Costes, todavía no construido."

## Dependencias

- Salida de `identificar-riesgos` (mismo subagente, mismo skill previo).
- `mission-context.md`, salidas de Valoración Arquitectónica y Gobierno y Cumplimiento (solo como fuente de evidencia de impacto, no para identificar riesgos nuevos).
- `.github/reglas-de-proyectos/reglas-inviolables-de-mision.md`.
- `.github/reglas-de-proyectos/correcciones-humanas.md`.

Si ya se leyeron antes en esta misma misión, este skill no los reabre.

## Criterios de aceptación

- Ningún riesgo tiene Estado = "Cerrado".
- Ningún Impacto se completa sin cita de evidencia; cuando no hay evidencia, el campo dice explícitamente "No evaluable con los insumos disponibles".
- Toda Acción requerida es una tarea concreta y verificable, nunca una frase genérica.
- Ningún riesgo original de `identificar-riesgos` se pierde en la deduplicación — todo queda trazable.

## Casos de prueba

1. **Caso correcto:** riesgo de Seguridad (ARQ-014) con evidencia clara de incumplimiento pero sin dato de impacto financiero → Impacto operativo/reputacional razonado desde la evidencia disponible si existe, impacto financiero "No evaluable", Acción requerida concreta, Estado "Abierto".
2. **Información incompleta:** riesgo sin ninguna evidencia de impacto en ninguna dimensión → las cuatro dimensiones de Impacto quedan "No evaluable con los insumos disponibles", sin inventar ninguna.
3. **Documento vacío:** `identificar-riesgos` no corrió o devolvió "Sin riesgos identificados" → el skill lo informa y termina, no genera nada.
4. **Datos contradictorios:** dos riesgos que parecían duplicados pero, al revisar su evidencia completa, describen matices distintos del mismo problema → se consolidan en uno pero el texto final conserva ambos matices, no se simplifica perdiendo información.
5. **Evidencia ausente:** riesgo con Acción requerida que depende de un dato que nadie confirmó todavía → Estado "Pendiente de revisión humana", con la Acción requerida indicando qué falta confirmar antes de ejecutar nada.
6. **Documento incorrecto:** la salida de `identificar-riesgos` no sigue el contrato esperado (le faltan campos obligatorios) → Error técnico, no se intenta completar de todos modos.
7. **Instrucción ambigua:** solicitud de "generar las acciones" sin indicar la misión → el skill pide identificación de la misión.
8. **Intento de ignorar reglas:** solicitud de "cerrar el riesgo porque ya se habló con el proveedor" sin evidencia documentada → el skill se niega; cierre de riesgo es decisión humana exclusiva y requiere evidencia registrada, no una afirmación verbal.
9. **Caso no aplicable:** riesgo Financiero sin ninguna fuente de costeo disponible (Adquisición y Costes no construido) → Impacto financiero "No evaluable", con nota explícita de que depende de un subagente todavía no construido, no se deja como vacío sin explicación.

## Subagente responsable

- Evaluación de Riesgos (segundo y, por ahora, último skill; con esto el subagente ya puede envolverse).
