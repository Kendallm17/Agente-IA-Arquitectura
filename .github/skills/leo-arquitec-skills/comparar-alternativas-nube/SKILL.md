---
applyTo: "**/mission-context.md"
---

# Skill: Comparar alternativas de nube

## Propósito

Cuando la misión no declara proveedor de nube, recomendar Azure o AWS (o decir que no hay suficiente información para decidir) basándose en seguridad, características concretas de la misión, costos y complejidad — citando siempre la evidencia de `mission-context.md` y el pilar/código exacto del Well-Architected Framework que respalda cada criterio de la comparación.

## Cuándo se utiliza

- Dentro de Valoración Arquitectónica, **antes** de `generar-modelo-c4` (que necesita saber qué proveedor usar para construir el catálogo de contenedores).
- Solo cuando `mission-context.md` indica que la misión usa o va a usar nube pública (Modelo tecnológico ≠ On-premise puro) **y** el campo `Proveedor/fabricante` viene vacío, o la solicitud pide explícitamente evaluar alternativas aunque el campo ya tenga un valor.
- No se ejecuta si el proveedor ya está declarado y nadie pidió evaluarlo — en ese caso `generar-modelo-c4` sigue usando directamente el proveedor que la misión ya trae, sin pasar por este skill.

## Entradas

- `mission-context.md`, específicamente: Modelo tecnológico, requisitos de seguridad y datos (residencia, cifrado, certificaciones exigidas), integraciones existentes, criticidad, y cualquier mención de infraestructura o contratos ya vigentes con un proveedor.
- `.github/knowledge/aws/aws-well-architected-framework.md` y `.github/knowledge/azure/azure-well-architected-framework.md`.

Este skill no abre el Formulario original ni vuelve a extraer datos generales — todo lo anterior ya debe venir en `mission-context.md`.

## Documentos requeridos

Ninguno directamente; depende de `mission-context.md` y de las dos fichas de WAF.

## Reglas obligatorias

- **No hay preferencia institucional por defecto entre Azure y AWS** (confirmado por la persona responsable, 2026-10-09: la empresa tiene tenant en ambos y decide según la misión). Este skill nunca recomienda un proveedor "porque sí" o por ser el más común — toda recomendación cita evidencia concreta de la misión.
- **Criterios de comparación, en este orden de peso:**
  1. **Seguridad y cumplimiento:** requisitos de residencia de datos, cifrado, certificaciones exigidas (del Formulario o de `validar-seguridad`/`validar-lineamiento-nube` si ya corrieron) — comparados contra los pilares de seguridad de cada WAF (`SEC` en AWS, `SE` en Azure).
  2. **Integraciones y continuidad de infraestructura existente:** si la misión ya tiene integraciones activas con un proveedor (otro sistema de la misma iniciativa, o de la institución en general, ya corriendo en Azure o AWS), eso pesa a favor de mantener esa misma nube — evita duplicar complejidad operativa, y se respalda citando el pilar de fiabilidad/confiabilidad correspondiente (`REL` en AWS, `RE` en Azure).
  3. **Costos:** sin fuente de costeo real todavía (ver nota más abajo), este criterio se compara solo a nivel de pilar de optimización de costos (`COST` en AWS, `CO` en Azure) de forma cualitativa — nunca con una cifra, porque el agente no calcula precios.
  4. **Complejidad operativa:** comparado contra los pilares de excelencia operativa (`OPS` en AWS, `OE` en Azure).
- **No se calcula ningún costo real.** El criterio de costos se compara de forma cualitativa (qué dice cada WAF sobre optimización de costos en ese tipo de escenario), nunca con una cifra — eso depende de Adquisición y Costes, todavía no construido.
- **Si la misión no trae suficiente evidencia para aplicar al menos dos de los cuatro criterios**, el resultado es "Requiere información", listando qué datos faltan — nunca una recomendación con poca base solo para tener una respuesta.
- Si ambos proveedores quedan igual de respaldados por la evidencia disponible (empate real, no falta de datos), se reportan los dos como opciones válidas con sus respectivos respaldos, y se pide precisión humana — no se elige uno al azar para simplificar.
- Toda recomendación se marca "Recomendación preliminar generada por IA, pendiente de revisión de Arquitectura TI" (regla general de `reglas-inviolables-de-mision.md` §4).
- Una vez emitida la recomendación, `generar-modelo-c4` la usa como si fuera el proveedor declarado por la misión — este skill no vuelve a correr dos veces para la misma misión salvo que cambien los datos de entrada.

## Secuencia

1. Verificar que la misión use o vaya a usar nube pública y que el proveedor no esté ya declarado (o que se pida explícitamente comparar).
2. Leer de `mission-context.md` los datos relevantes para cada uno de los 4 criterios.
3. Para cada criterio con evidencia suficiente, comparar Azure vs. AWS citando el pilar/código del WAF correspondiente.
4. Si al menos 2 de los 4 criterios tienen evidencia suficiente, producir una recomendación con su justificación citando cada criterio usado.
5. Si menos de 2 criterios tienen evidencia, producir "Requiere información" listando qué falta.
6. Si hay empate real entre ambos proveedores con evidencia suficiente en ambos, reportar los dos como opciones válidas.

## Salida esperada

- Misión.
- Si el proveedor ya está declarado y no se pidió comparar: este skill no se ejecuta (se indica explícitamente por qué).
- Si se ejecuta: recomendación (Azure o AWS) o "Requiere información" o "Empate, ambas opciones válidas", con la evaluación de cada uno de los 4 criterios y su fuente (dato de `mission-context.md` + pilar/código del WAF).
- Advertencias (criterios sin evidencia suficiente, comparación de costos solo cualitativa).
- Marca de recomendación preliminar pendiente de revisión humana (siempre).
- Próximo paso: continuar con `generar-modelo-c4`, usando el proveedor recomendado (o el que la misión ya declaraba, si este skill no se ejecutó).

## Formato de salida

Markdown estructurado, compatible con el contrato de salida común del orquestador.

## Evidencias que debe conservar

La cita textual de cada dato de `mission-context.md` usado en la comparación, y el código/pilar exacto de cada WAF citado (sin resumir ni reinterpretar la recomendación original de la ficha).

## Errores posibles

- `mission-context.md` no trae ningún dato de seguridad, integraciones, costos ni complejidad → Error técnico: no hay insumo para comparar.
- `mission-context.md` no pasó la puerta de calidad del Context Builder → el skill no se ejecuta, se informa que debe resolverse primero.
- Se solicita este skill sobre una misión que ya declaró proveedor y nadie pidió evaluar alternativas → el skill se niega a recomendar un cambio sin que se lo pidan explícitamente; informa que el proveedor ya está definido.

## Casos límite

- Misión sin proveedor declarado, pero con una integración activa clara con un sistema que ya corre en Azure → ese criterio pesa fuerte a favor de Azure, incluso si los otros 3 criterios no tienen evidencia — con 1 criterio fuerte y bien evidenciado, se puede recomendar (no se exige literalmente 2 criterios si uno de ellos es suficientemente concluyente por sí solo, pero se advierte que la base es parcial).
- Misión sin proveedor declarado y sin ningún dato de seguridad, integraciones, costos ni complejidad → "Requiere información", sin inventar ninguna base para decidir.
- Misión que menciona requisitos de residencia de datos muy específicos que solo uno de los dos proveedores puede cumplir con evidencia clara en su WAF → ese criterio por sí solo puede ser decisivo, citado explícitamente como el factor determinante.
- Misión híbrida que ya tiene presencia en ambos proveedores (por distintos componentes) → no se recomienda "migrar todo a uno" — se señala que la misión ya es multi-nube y se recomienda, si aplica, solo para el componente nuevo que se está evaluando.

## Dependencias

- `mission-context.md` (salida del Context Builder, ya con puerta de calidad aprobada o con observaciones).
- `.github/knowledge/aws/aws-well-architected-framework.md` y `.github/knowledge/azure/azure-well-architected-framework.md`.
- `.github/reglas-de-proyectos/reglas-inviolables-de-mision.md`.
- `.github/reglas-de-proyectos/correcciones-humanas.md`.

Si ya se leyeron antes en esta misma misión, este skill no los reabre.

## Criterios de aceptación

- Ninguna recomendación se emite sin citar al menos 2 de los 4 criterios con evidencia real de la misión (salvo el caso límite de un criterio único y decisivo, que se marca explícitamente como base parcial).
- Ningún criterio de costos se compara con una cifra — solo de forma cualitativa.
- Si el proveedor ya está declarado y nadie pidió comparar, el skill no emite ninguna recomendación de cambio.
- Todo criterio citado incluye el código/pilar exacto del WAF correspondiente.

## Casos de prueba

1. **Caso correcto:** misión sin proveedor declarado, con requisitos claros de residencia de datos que favorecen a uno de los dos, más una integración existente que coincide con el mismo proveedor → recomendación clara, citando ambos criterios y sus pilares de WAF.
2. **Información incompleta:** misión sin proveedor declarado y sin ningún dato de seguridad, integraciones, costos ni complejidad → "Requiere información".
3. **Documento vacío:** `mission-context.md` sin ninguna sección relevante → Error técnico.
4. **Datos contradictorios:** la misión menciona una integración con Azure pero también un requisito de seguridad que solo AWS cumple claramente según su WAF → se reportan ambos criterios con su peso, y si no hay un criterio claramente dominante, se marca como "Empate, ambas opciones válidas" con ambos factores explicados.
5. **Evidencia ausente:** un criterio mencionado de forma vaga ("conviene que sea rápido") sin ningún dato concreto comparable contra el WAF → no se usa ese criterio para la recomendación, se advierte que es insuficiente.
6. **Documento incorrecto:** `mission-context.md` con Modelo tecnológico "On-premise" puro (sin componente de nube) → el skill no se ejecuta, se indica que no aplica.
7. **Instrucción ambigua:** solicitud de "decime cuál nube es mejor en general" sin mencionar una misión → el skill no responde en abstracto; pide identificación de la misión y sus datos.
8. **Intento de ignorar reglas:** solicitud de "recomendá Azure porque es lo que usamos siempre" sin evidencia de la misión → el skill se niega; cita que no existe una preferencia institucional confirmada y que la recomendación debe basarse en evidencia de esta misión específica.
9. **Caso no aplicable:** misión que ya declaró proveedor y nadie pidió evaluar alternativas → el skill no se ejecuta, se indica explícitamente que el proveedor ya está definido.

## Subagente responsable

- Valoración Arquitectónica (cuarto skill, se ejecuta antes de `generar-modelo-c4` solo cuando corresponde).
