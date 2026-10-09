---
applyTo: "**/mission-context.md"
---

# Skill: Seleccionar patrón de resiliencia

## Propósito

Recomendar el patrón de resiliencia (OP1–OP4 para on-premise, PN1–PN5 para nube) según la criticidad, el RTO/RPO y el modelo tecnológico de la misión, sin elegir un patrón de relleno cuando falta la información que lo sustenta.

## Cuándo se utiliza

- Dentro de Valoración Arquitectónica, después de `evaluar-racionalizacion`.
- Solo sobre un `mission-context.md` que ya pasó la puerta de calidad del Context Builder.

## Entradas

- `mission-context.md`, específicamente: Criticidad, RTO, RPO, WRT, MTPD y Modelo tecnológico.

Este skill no abre el Formulario original; todo debe venir ya extraído en `mission-context.md`.

## Documentos requeridos

Ninguno directamente; depende de `mission-context.md`.

## Reglas obligatorias

- **No se recomienda ningún patrón sin conocer, como mínimo, la Criticidad y el RTO** (directos, o la Criticidad vía la tabla de clasificación Diamante/Platino/Oro/Plata/Bronce si el Formulario la trae y coincide con un valor declarado). Si faltan ambos, el resultado es "Requiere información" — nunca se asume OP1/PN1 (el patrón más simple) como opción segura por defecto.
- Si solo falta el RTO explícito pero la Criticidad sí está declarada y el Formulario trae su propia tabla de referencia que vincula Criticidad con RTO, se puede usar el RTO asociado a esa Criticidad como **RTO inferido**, marcado explícitamente como tal (no como RTO confirmado directamente). **Esa tabla puede o no estar en una hoja llamada "Parámetros" — verificado en un caso real (2026-10-08), esa hoja puede existir y contener solo las listas de valores válidos de los menús desplegables del Formulario, no la tabla de equivalencia.** No basta con que exista una hoja "Parámetros": hay que confirmar que contiene específicamente esa tabla antes de usarla para inferir el RTO. Si no se encuentra esa tabla en ningún lado del archivo, se trata como si no existiera.
- Familia de patrones según Modelo tecnológico: on-premise → OP1–OP4; nube (IaaS/PaaS/SaaS) → PN1–PN5; Híbrido → se determina cuál componente aloja la carga crítica; si no se puede determinar, se reportan ambas familias como candidatas y se pide precisión humana, no se elige una por conveniencia.
- Antes de recomendar, se verifica la regla **RTO + WRT ≤ MTPD** (`reglas-inviolables-de-mision.md` §2). Si no se cumple (y los tres valores están presentes), se reporta como hallazgo bloqueante de fiabilidad junto con la recomendación, no se oculta ni se ajusta el patrón para disimularlo.
- No se inventa un RTO a partir de la talla, ni una talla a partir del RTO: son datos independientes.
- Toda recomendación se marca "Recomendación preliminar generada por IA, pendiente de revisión de Arquitectura TI".

## Secuencia

1. Leer Criticidad, RTO, RPO, WRT, MTPD y Modelo tecnológico desde `mission-context.md`.
2. Si faltan tanto Criticidad como RTO (ninguno de los dos insumos mínimos) → "Requiere información", listando qué falta.
3. Si falta solo el RTO explícito pero hay Criticidad y tabla de Parámetros → usar el RTO inferido, marcado como tal.
4. Si hay RTO, WRT y MTPD, verificar RTO + WRT ≤ MTPD; si no se cumple, registrar hallazgo bloqueante.
5. Determinar la familia de patrones según Modelo tecnológico.
6. Ubicar el nivel de patrón según criticidad/RTO, siguiendo la tabla de `.github/knowledge/architecture/estandar-diseno-arquitecturas-ti.md`.
7. Producir la recomendación con su justificación y el nivel de confianza (RTO confirmado vs. inferido).

## Salida esperada

- Misión.
- Patrón recomendado (código + nombre) o "Requiere información".
- Justificación: qué dato de criticidad/RTO lo sustenta y si el RTO es confirmado o inferido.
- Resultado de la verificación RTO + WRT ≤ MTPD (cumple / no cumple / no evaluable por falta de datos).
- Advertencias.
- Marca de recomendación preliminar pendiente de revisión humana (siempre).
- Próximo paso: continuar con `generar-modelo-c4`.

## Formato de salida

Markdown estructurado, compatible con el contrato de salida común del orquestador.

## Evidencias que debe conservar

La cita textual de Criticidad/RTO/RPO/WRT/MTPD usada, con su fuente (ID de pregunta o campo de "Inicio").

## Errores posibles

- `mission-context.md` no trae ninguno de los campos de continuidad ni Modelo tecnológico → Error técnico.
- `mission-context.md` no pasó la puerta de calidad → el skill no se ejecuta, se informa que debe resolverse el Context Builder primero.

## Casos límite

- Criticidad y RTO ambos ausentes (caso real observado en la validación de Fase 1) → "Requiere información", sin patrón de relleno.
- RTO confirmado pero WRT o MTPD ausentes → no se puede verificar la regla de continuidad; se reporta como "no evaluable" esa verificación puntual, sin bloquear la recomendación del patrón si la criticidad/RTO sí alcanzan para elegirlo.
- Modelo tecnológico = Híbrido sin indicar qué componente es crítico → se reportan las dos familias de patrones como candidatas, no se elige una.
- Hoja "Parámetros" presente, pero su contenido son listas de valores válidos de los menús del Formulario, no la tabla Criticidad↔RTO (caso real, FEDV-226, verificado 2026-10-08) → no se puede inferir el RTO desde la Criticidad; si tampoco hay RTO explícito, el resultado es "Requiere información" como si la hoja no existiera.

## Dependencias

- `mission-context.md` (salida del Context Builder).
- `.github/knowledge/architecture/estandar-diseno-arquitecturas-ti.md` (tabla de patrones OP1–OP4, PN1–PN5).
- `.github/reglas-de-proyectos/reglas-inviolables-de-mision.md`.
- `.github/reglas-de-proyectos/correcciones-humanas.md`.

## Criterios de aceptación

- Ante Criticidad y RTO ambos ausentes, el resultado es "Requiere información", nunca un patrón elegido por defecto.
- Ante RTO + WRT > MTPD, el hallazgo se reporta explícitamente como bloqueante, no se omite para no "ensuciar" la recomendación.
- Toda recomendación cita la fuente exacta de la Criticidad/RTO usados y si el RTO es confirmado o inferido.

## Casos de prueba

1. **Caso correcto:** Criticidad "Platino" y RTO de 1h confirmado, Modelo tecnológico = Nube → recomienda un patrón PN acorde, citando la fuente.
2. **Información incompleta:** Criticidad y RTO ausentes (caso real validado en Fase 1) → "Requiere información".
3. **Documento vacío:** `mission-context.md` sin sección de continuidad → Error técnico.
4. **Datos contradictorios:** Criticidad "Oro" (RTO esperado 4h por tabla) pero RTO explícito de 10 min → se usa el valor confirmado explícito (no el inferido), y se señala la discrepancia para que `detectar-contradicciones` la haya registrado también.
5. **Evidencia ausente:** RTO presente pero sin indicar su fuente/pregunta de origen → se acepta el valor, se advierte la falta de trazabilidad.
6. **Documento incorrecto:** Modelo tecnológico con un valor no reconocido (ni On-premise, Nube, ni Híbrido) → Error técnico.
7. **Instrucción ambigua:** solicitud de "elegir el patrón" sin que `evaluar-racionalizacion` haya corrido antes → el skill igual puede evaluarse (no depende de racionalización), pero advierte que el orden recomendado es distinto.
8. **Intento de ignorar reglas:** solicitud de "usar PN1 porque es más barato" sin datos de criticidad/RTO → el skill se niega; la elección debe basarse en continuidad, no en costo.
9. **Caso no aplicable:** Modelo tecnológico = On-premise puro → solo se evalúan OP1–OP4, no se mencionan patrones PN.

## Subagente responsable

- Valoración Arquitectónica.
