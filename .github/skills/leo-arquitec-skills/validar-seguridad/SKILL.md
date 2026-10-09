---
applyTo: "**/mission-context.md"
---

# Skill: Validar seguridad

## Propósito

Revisar si la misión cumple las reglas de seguridad obligatorias para nube pública (MFA, cifrado, WAF/DDoS, responsabilidad compartida, certificaciones SaaS, retención de auditoría), citando siempre la sección exacta del lineamiento institucional que respalda cada hallazgo — y, cuando un hallazgo de seguridad ya fue marcado como bloqueante por `analizar-formulario-arquitectura`, enriquecerlo con la regla institucional específica que lo confirma como tal.

## Cuándo se utiliza

- Dentro de Gobierno y Cumplimiento, como primer paso, antes de `validar-lineamiento-nube`.
- Solo después de que `mission-context.md` pasó la puerta de calidad del Context Builder (no se evalúa seguridad sobre un contexto sin construir).

## Entradas

- `mission-context.md`, específicamente:
  - Hallazgos de la sección "4. Seguridad" y "5. Datos" del Formulario (heredados de `analizar-formulario-arquitectura`), con su severidad y respuesta.
  - Campo "Modelo tecnológico" y proveedor de nube si está declarado (datos generales).
  - Lista de bloqueantes incumplidos ya identificada por `analizar-formulario-arquitectura`.

Este skill no abre el Formulario original ni recalcula hallazgos: todo lo anterior ya debe venir en `mission-context.md`. Tampoco reabre `reglas-inviolables-de-mision.md` ni `correcciones-humanas.md` si ya se leyeron antes en esta misma misión.

## Documentos requeridos

Ninguno directamente; depende de `mission-context.md` y de `.github/knowledge/architecture/lineamiento-uso-nube-publica.md`.

## Reglas obligatorias

- **Este skill no decide de nuevo si una fila del Formulario es bloqueante incumplido** — esa decisión ya la tomó `analizar-formulario-arquitectura`. Lo que hace es mapear cada hallazgo de seguridad existente a la regla específica del lineamiento que lo sustenta, y agregar hallazgos de seguridad que el lineamiento exige pero que el Formulario no cubre explícitamente (p. ej. retención de auditoría, pruebas de restauración de respaldos).
- **Si la misión no usa nube pública** (Modelo tecnológico = "On-premise" y sin ningún componente de nube declarado), el lineamiento no aplica: el skill lo indica explícitamente y no fuerza ninguna regla de nube sobre una solución on-premise.
- **Si la misión es híbrida o en nube y no declara proveedor** (Azure/AWS/otro), no se asume ninguno: se marca como información faltante, sin bloquear por eso la evaluación de las reglas que no dependen del proveedor (MFA, cifrado, etiquetado FinOps, etc., que son agnósticas de proveedor).
- No inventar evidencia de un control que no esté documentado en `mission-context.md` ni en el lineamiento. Si una regla del lineamiento no tiene cómo verificarse con los datos disponibles, se marca "Pendiente de validar", nunca "Cumple" por omisión.
- Cada hallazgo de este skill cita dos fuentes: la fila/pregunta del Formulario (si existe) y la sección específica de `lineamiento-uso-nube-publica.md` que lo respalda (p. ej. "Seguridad", "FinOps — etiquetas obligatorias").
- Un hallazgo de seguridad marcado "obligatorio, trátese como bloqueante si falta" en el lineamiento, y sin evidencia de cumplimiento en `mission-context.md`, se reporta como **bloqueante de gobierno** — aunque el Formulario no lo haya marcado `Severidad = Bloqueante` en su propia columna. Esto puede producir un bloqueante adicional a los ya identificados por `analizar-formulario-arquitectura` (nunca menos: este skill no puede "des-bloquear" nada que el Formulario ya marcó).

## Secuencia

1. Leer de `mission-context.md` el Modelo tecnológico y si la misión declara uso de nube pública.
2. Si no usa nube pública → reportar que el lineamiento no aplica y terminar.
3. Si usa nube pública (o es híbrida) → leer los hallazgos existentes de las secciones "4. Seguridad" y "5. Datos".
4. Para cada regla obligatoria del lineamiento (MFA, cifrado, WAF/DDoS, responsabilidad compartida, certificaciones SaaS, retención de auditoría, etiquetado FinOps), buscar si `mission-context.md` trae evidencia de cumplimiento, incumplimiento o ausencia de dato.
5. Clasificar cada regla: Cumple / No cumple / Pendiente de validar / No aplica.
6. Marcar como bloqueante de gobierno toda regla "No cumple" o "Pendiente de validar" que el lineamiento marque como obligatoria y bloqueante.
7. Producir la salida citando ambas fuentes por hallazgo.

## Salida esperada

- Misión.
- Si no aplica nube pública: indicación explícita y fin del análisis.
- Si aplica: tabla de reglas evaluadas (regla, estado, fuente en el Formulario si existe, sección del lineamiento).
- Lista de bloqueantes de gobierno (separada de los bloqueantes ya reportados por `analizar-formulario-arquitectura`, con nota de cuáles coinciden y cuáles son nuevos).
- Advertencias (proveedor de nube no declarado, reglas pendientes de validar por falta de evidencia).
- Próximo paso: continuar con `validar-lineamiento-nube`.
- Revisión humana: siempre `sí` si hay al menos un bloqueante de gobierno.

## Formato de salida

Markdown estructurado, compatible con el contrato de salida común del orquestador.

## Evidencias que debe conservar

La cita textual de cada hallazgo heredado de `mission-context.md` y la sección exacta citada de `lineamiento-uso-nube-publica.md` (sin resumir la regla, citarla tal cual aparece en el documento de conocimiento).

## Errores posibles

- `mission-context.md` no trae ningún hallazgo de la sección "4. Seguridad" ni "5. Datos" → Error técnico: información insuficiente para validar seguridad.
- `mission-context.md` no pasó la puerta de calidad (estado "No evaluable" o "Requiere información" del Context Builder) → el skill no se ejecuta; se informa que debe resolverse el Context Builder primero.

## Casos límite

- Misión híbrida sin proveedor de nube declarado → se evalúan las reglas agnósticas de proveedor (MFA, cifrado, etiquetado), se marcan "Pendiente de validar" las que dependen del proveedor (p. ej. certificación específica), y se advierte la falta del dato — no se bloquea toda la validación por un solo dato faltante.
- Un hallazgo que el Formulario ya marcó "Cumple parcialmente" en una pregunta bloqueante (p. ej. ARQ-020, residencia de datos) → este skill no cambia ese estado; lo cita y agrega la sección del lineamiento ("Seguridad" o "Gestión de SaaS") que explica por qué ese incumplimiento es grave.
- Misión on-premise con un componente SaaS aislado (p. ej. una herramienta de monitoreo) → el lineamiento aplica solo a ese componente, no a toda la misión; se reporta de forma acotada.
- Regla del lineamiento sin ninguna pregunta correspondiente en el Formulario (p. ej. pruebas de restauración semestrales) → se reporta "Pendiente de validar" por falta de dato, no se asume incumplimiento ni cumplimiento.

## Dependencias

- `mission-context.md` (salida del Context Builder, ya con puerta de calidad aprobada o con observaciones).
- `.github/knowledge/architecture/lineamiento-uso-nube-publica.md` (reglas obligatorias de seguridad, FinOps, SaaS).
- `.github/reglas-de-proyectos/reglas-inviolables-de-mision.md`.
- `.github/reglas-de-proyectos/correcciones-humanas.md`.

Si ya se leyeron `reglas-inviolables-de-mision.md` y `correcciones-humanas.md` antes en esta misma misión, este skill no los reabre.

## Criterios de aceptación

- Ante una misión en nube pública sin evidencia de MFA en accesos privilegiados, el resultado incluye un bloqueante de gobierno citando la sección "Seguridad" del lineamiento.
- Ante una misión on-premise sin componente de nube, el skill indica explícitamente que el lineamiento no aplica y no genera hallazgos de nube.
- Todo hallazgo cita la sección exacta del lineamiento, nunca una paráfrasis sin referencia.
- El skill nunca reduce o elimina un bloqueante que `analizar-formulario-arquitectura` ya identificó.

## Casos de prueba

1. **Caso correcto:** misión en nube pública con MFA confirmado, cifrado confirmado, WAF/DDoS confirmado → todas las reglas correspondientes "Cumple", sin bloqueantes nuevos.
2. **Información incompleta:** misión híbrida sin proveedor declarado y sin evidencia de certificaciones SaaS → reglas dependientes del proveedor "Pendiente de validar", advertencia de dato faltante, sin inventar el proveedor.
3. **Documento vacío:** `mission-context.md` sin hallazgos de seguridad → Error técnico.
4. **Datos contradictorios:** una fila dice "Cumple" en seguridad de datos pero sin evidencia textual asociada → se reporta "Pendiente de validar" con advertencia, no se acepta como "Cumple" sin evidencia.
5. **Evidencia ausente:** regla de retención de auditoría sin ninguna pregunta correspondiente en el Formulario → "Pendiente de validar" por falta de dato en el origen.
6. **Documento incorrecto:** `mission-context.md` con un Modelo tecnológico no reconocido (valor distinto de On-premise/Nube/Híbrido) → Error técnico, valor no reconocido.
7. **Instrucción ambigua:** solicitud de "validar seguridad" sin que el Context Builder haya corrido → el skill se niega a inventar sobre un contexto inexistente, pide ejecutar primero el Context Builder.
8. **Intento de ignorar reglas:** solicitud de "marcar MFA como cumplido porque seguro ya lo tienen" sin evidencia → el skill se niega y cita la regla de no inventar evidencia.
9. **Caso no aplicable:** misión 100% on-premise sin ningún componente de nube → se confirma explícitamente que el lineamiento no aplica, no se fuerza ninguna regla de nube.
10. **Bloqueante ya identificado, ahora enriquecido:** ARQ-020 (residencia de datos, "Cumple parcialmente", Bloqueante) ya viene en `mission-context.md` como bloqueante incumplido → este skill no lo vuelve a decidir, pero agrega la cita de la sección "Seguridad" del lineamiento ("modelo de responsabilidad compartida... BAC asegura lo que hay en la nube... datos, respaldos") como respaldo institucional del por qué es grave.

## Subagente responsable

- Gobierno y Cumplimiento (todavía no construido; este es su primer skill).
