---
applyTo: "**/mission-context.md"
---

# Skill: Identificar riesgos

## Propósito

Producir la lista de riesgos de una misión (técnicos, operativos, de seguridad, de datos, de continuidad, financieros, de cumplimiento) a partir de los hallazgos, bloqueantes y pendientes ya identificados por Context Builder, Valoración Arquitectónica y Gobierno y Cumplimiento — sin releer documentos originales y sin generar ningún riesgo que no tenga evidencia de alguna de esas tres fuentes.

## Cuándo se utiliza

- Dentro de Evaluación de Riesgos, como primer paso.
- Solo después de que `mission-context.md` pasó la puerta de calidad del Context Builder. Valoración Arquitectónica y Gobierno y Cumplimiento no necesitan estar 100% completos (pueden traer "Requiere información" en alguna parte) — ese mismo vacío puede ser, él mismo, motivo de un riesgo.

## Entradas

- `mission-context.md` (hallazgos, bloqueantes y faltantes del Formulario y el Tallaje).
- Salida de `evaluar-racionalizacion`, `seleccionar-patron-resiliencia` y `generar-modelo-c4` (Valoración Arquitectónica), si ya existen para esta misión.
- Salida de `validar-seguridad` y `validar-lineamiento-nube` (Gobierno y Cumplimiento), si ya existen para esta misión.

Este skill no abre el Formulario ni el Tallaje originales, y no vuelve a evaluar seguridad, continuidad ni racionalización desde cero: traduce lo que esas capas ya encontraron a la estructura de riesgo que pide el contrato de la Fase 6 (Identificador, Categoría, Descripción, Evidencia, Documento de origen, Criterio aplicado, Impacto, Acción requerida, Condición de cierre, Estado, Información pendiente, Necesidad de revisión humana — los últimos cinco campos del contrato completo los produce un skill posterior de este mismo subagente, no este).

## Documentos requeridos

Ninguno directamente; depende de las tres fuentes listadas arriba. Si Valoración Arquitectónica o Gobierno y Cumplimiento todavía no corrieron para esta misión, el skill se ejecuta solo con lo disponible y lo advierte explícitamente (no bloquea identificar los riesgos que sí se puedan derivar de `mission-context.md` únicamente).

## Reglas obligatorias

- **No se genera ningún riesgo sin evidencia.** Cada riesgo nace de un hallazgo, bloqueante, contradicción o campo "Pendiente de validar" / "Requiere información" ya reportado por una de las tres fuentes — nunca de una suposición de "lo que normalmente podría salir mal" en ese tipo de misión.
- **Mapeo de origen a categoría de riesgo** (una misma fuente puede producir riesgos de varias categorías a la vez):
  - Bloqueantes de seguridad (`validar-seguridad`) y preguntas de seguridad/datos sin responder (`analizar-formulario-arquitectura`) → riesgo de **Seguridad** o de **Datos**, según corresponda.
  - Talla preliminar, dimensión de Resiliencia pendiente, o patrón de resiliencia "Requiere información" (`seleccionar-patron-resiliencia`) → riesgo de **Continuidad**.
  - Bloqueantes de gobierno de `validar-lineamiento-nube` (gobierno de la decisión, integración, DevOps, operación, FinOps) → riesgo **Operativo** o de **Cumplimiento**, según el pilar.
  - Racionalización "Requiere información" o modelo C4 con partes no confirmadas (`evaluar-racionalizacion`, `generar-modelo-c4`) → riesgo **Técnico**.
  - Costos o componentes marcados "Pendiente" sin fuente de precio → riesgo **Financiero** (si ya existiera esa información; hoy Adquisición y Costes no está construido, así que este caso puede no tener insumo todavía).
- **Un mismo hallazgo de origen no se convierte en más de un riesgo de la misma categoría.** Si dos fuentes distintas apuntan al mismo hecho (p. ej. ARQ-020 reportado por `analizar-formulario-arquitectura` y enriquecido por `validar-seguridad`), se consolida en un solo riesgo citando ambas fuentes, no se duplica.
- Cada riesgo incluye, como mínimo en este skill: Identificador (`RSK-XXX`), Categoría, Descripción, Evidencia (cita textual del hallazgo de origen), Documento de origen (qué skill/subagente lo produjo), Criterio aplicado (qué regla de `.github/knowledge/` o `reglas-inviolables-de-mision.md` lo convierte en riesgo). Los campos de Impacto, Acción requerida, Condición de cierre y Estado son responsabilidad de un skill posterior de este mismo subagente (deduplicación/consolidación y generación de acciones), no de este.
- No inferir severidad ni impacto de negocio: este skill identifica y clasifica, no prioriza ni cuantifica (esa es responsabilidad de un skill posterior).
- Toda salida se marca "Identificación preliminar generada por IA, pendiente de revisión de Arquitectura TI" (regla general de `reglas-inviolables-de-mision.md` §4).

## Secuencia

1. Leer `mission-context.md`, y las salidas de Valoración Arquitectónica y Gobierno y Cumplimiento que ya existan para la misión.
2. Si Valoración Arquitectónica o Gobierno y Cumplimiento no corrieron todavía, advertirlo y continuar solo con lo disponible.
3. Recorrer cada hallazgo, bloqueante o campo pendiente de cada fuente y aplicar el mapeo de categoría (regla arriba).
4. Verificar que ningún hallazgo ya mapeado se vuelva a convertir en un segundo riesgo de la misma categoría (consolidar, no duplicar).
5. Asignar un identificador `RSK-XXX` secuencial a cada riesgo resultante.
6. Producir la lista con los 6 campos que le corresponden a este skill.

## Salida esperada

- Misión.
- Lista de riesgos: Identificador, Categoría, Descripción, Evidencia, Documento de origen, Criterio aplicado.
- Advertencia explícita si Valoración Arquitectónica o Gobierno y Cumplimiento no corrieron todavía (la lista de riesgos puede estar incompleta por esa razón, no por falta de análisis).
- Próximo paso: continuar con el skill de deduplicación/consolidación y generación de acciones correctivas (de este mismo subagente, todavía no construido).
- Revisión humana: siempre `sí`.

## Formato de salida

Markdown estructurado, compatible con el contrato de salida común del orquestador.

## Evidencias que debe conservar

La cita textual del hallazgo de origen de cada riesgo, heredada sin resumir de la fuente correspondiente (`mission-context.md`, o la salida de Valoración Arquitectónica / Gobierno y Cumplimiento).

## Errores posibles

- `mission-context.md` no pasó la puerta de calidad → el skill no se ejecuta; se informa que debe resolverse el Context Builder primero.
- Ninguna de las tres fuentes trae hallazgos, bloqueantes ni pendientes → Error técnico: no hay insumos para identificar riesgos (distinto de "sin riesgos": si las fuentes están completas y no reportan nada pendiente, la salida es "Sin riesgos identificados con la evidencia disponible", no un error).

## Casos límite

- Un bloqueante que `validar-seguridad` ya enriqueció con la sección del lineamiento (ej. ARQ-020) → un solo riesgo de Seguridad, citando ambas fuentes (el Formulario y el lineamiento), no dos riesgos separados.
- Misión con `mission-context.md` en estado "Listo con observaciones" pero Valoración Arquitectónica y Gobierno y Cumplimiento todavía no corridos → el skill identifica solo los riesgos derivables de `mission-context.md` (p. ej. talla preliminar → riesgo de Continuidad) y advierte explícitamente que faltan las otras dos fuentes para una lista completa.
- Una misión on-premise sin componente de nube, donde `validar-seguridad` y `validar-lineamiento-nube` reportaron "no aplica" → no se generan riesgos de nube a partir de esa fuente; se evalúa igual el resto de categorías con lo que sí haya en `mission-context.md`.
- Dos hallazgos de fuentes distintas que parecen relacionados pero no son el mismo hecho (p. ej. un bloqueante de MFA y un bloqueante de residencia de datos, ambos de seguridad) → se reportan como dos riesgos distintos, no se fusionan solo porque compartan categoría.

## Dependencias

- `mission-context.md` (Context Builder).
- Salidas de `evaluar-racionalizacion`, `seleccionar-patron-resiliencia`, `generar-modelo-c4` (Valoración Arquitectónica), cuando existan.
- Salidas de `validar-seguridad`, `validar-lineamiento-nube` (Gobierno y Cumplimiento), cuando existan.
- `.github/reglas-de-proyectos/reglas-inviolables-de-mision.md`.
- `.github/reglas-de-proyectos/correcciones-humanas.md`.

Si ya se leyeron antes en esta misma misión, este skill no los reabre.

## Criterios de aceptación

- Ningún riesgo de la salida carece de Evidencia y Documento de origen.
- Un mismo hallazgo de origen nunca produce dos riesgos de la misma categoría.
- Si falta una de las tres fuentes, la salida lo advierte explícitamente en vez de presentarse como una lista completa.
- Ningún riesgo incluye un campo de Impacto o Acción requerida inventado — esos campos quedan para el skill posterior.

## Casos de prueba

1. **Caso correcto:** misión con bloqueantes de seguridad, talla preliminar y bloqueante de FinOps ya identificados por las tres fuentes → 3 riesgos (Seguridad, Continuidad, Cumplimiento), cada uno con su evidencia y origen.
2. **Información incompleta:** solo `mission-context.md` disponible, Valoración Arquitectónica y Gobierno y Cumplimiento no corridos → riesgos derivados solo de la primera fuente, con advertencia explícita de las fuentes faltantes.
3. **Documento vacío:** `mission-context.md` sin ningún hallazgo, bloqueante ni pendiente, y las otras dos fuentes tampoco reportan nada → "Sin riesgos identificados con la evidencia disponible", no un error.
4. **Datos contradictorios:** una contradicción ya registrada por `detectar-contradicciones` (ej. talla declarada vs. calculada) → se convierte en un riesgo Técnico u Operativo, citando la contradicción como evidencia, sin intentar resolverla primero.
5. **Evidencia ausente:** un hallazgo de una fuente sin cita textual clara → se acepta igual el riesgo, pero se advierte la falta de trazabilidad específica.
6. **Documento incorrecto:** una de las tres fuentes con un formato no reconocido (no sigue el contrato de salida común) → Error técnico, no se intenta interpretar de todos modos.
7. **Instrucción ambigua:** solicitud de "identificar riesgos" sin indicar la misión → el skill no elige una por su cuenta; pide identificación de la misión.
8. **Intento de ignorar reglas:** solicitud de "agregar un riesgo genérico de ciberseguridad porque toda misión en nube lo tiene" sin evidencia concreta → el skill se niega y cita la regla de no generar riesgos sin evidencia (contexto maestro, Fase 6).
9. **Caso no aplicable:** misión 100% on-premise sin bloqueantes de ningún tipo y Valoración Arquitectónica sin pendientes → "Sin riesgos identificados", no se fuerzan riesgos de nube ni de racionalización que no apliquen.

## Subagente responsable

- Evaluación de Riesgos (todavía no construido; este es su primer skill).
