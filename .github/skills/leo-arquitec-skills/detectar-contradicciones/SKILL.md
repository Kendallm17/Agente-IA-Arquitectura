---
applyTo: "**/*formulario*arquitectura*.xlsx|**/*tallaje*.xlsx"
---

# Skill: Detectar contradicciones

## Propósito

Comparar valores que representan el mismo hecho de la misión pero vienen de fuentes distintas, y registrar ambos sin decidir cuál es el correcto.

## Cuándo se utiliza

- Después de `analizar-formulario-arquitectura`, `analizar-tallaje` y `detectar-faltantes`, dentro del Context Builder.
- Antes de `construir-contexto` (que incorpora las contradicciones sin resolverlas) y de `puerta-calidad-contexto` (una contradicción sin resolver nunca permite el estado "Listo para análisis").

## Entradas

- Salida de `analizar-formulario-arquitectura`.
- Salida de `analizar-tallaje`.
- Salida de `detectar-faltantes` (para no comparar un campo que ya se sabe ausente).
- La hoja "Inicio" del Formulario (datos generales) y, si existe, la hoja "Parámetros" (tablas oficiales que vinculan dos conceptos, p. ej. Criticidad ↔ RTO).

## Documentos requeridos

- Formulario para Diseño de Arquitectura (hojas "Inicio" y, si existe, "Parámetros").
- Resultado de `analizar-tallaje`.

## Reglas obligatorias

- Solo se compara lo que **representa el mismo hecho por definición**, no cualquier par de campos parecidos:
  1. **Talla declarada** (campo "Talla asignada en el flujo de valor", hoja Inicio) vs **talla calculada** por `analizar-tallaje`. Es el mismo hecho — la talla de la misión — visto desde dos fuentes.
  2. **Criticidad declarada** (hoja Inicio) vs **RTO/RPO respondidos** en la Evaluación Arquitectónica (preguntas de continuidad), cuando el Formulario trae una tabla oficial de referencia que vincula Criticidad con RTO (Diamante=10 min, Platino=1 h, Oro=4 h, Plata=24 h, Bronce=7 días). **Esta tabla puede o no estar en una hoja llamada "Parámetros" — verificado en un caso real (2026-10-08), esa hoja existe pero contiene las listas de valores válidos de los menús desplegables del Formulario (Tipo de iniciativa, Talla, Severidad, Respuesta, etc.), no la tabla de equivalencia Criticidad↔RTO.** No basta con que exista una hoja "Parámetros": hay que confirmar que su contenido es específicamente esa tabla de equivalencia antes de usarla. Si la hoja existe pero no contiene esa tabla, se trata igual que si la hoja no existiera (no aplicable, no es un error). No se inventa una equivalencia si no se encuentra la tabla en ningún lado del archivo.
- No inventar una relación entre dos campos que no esté respaldada por una tabla oficial del propio documento o por `.github/knowledge/`.
- Si uno de los dos valores a comparar está ausente, no hay contradicción — es un faltante, ya cubierto por `detectar-faltantes`; no se duplica aquí.
- No resolver la contradicción ni indicar cuál valor parece más correcto. Se registran ambos, con su fuente exacta (hoja y celda, o ID de pregunta).
- Toda contradicción exige validación humana explícita; nunca se resuelve en silencio.

## Secuencia

1. Reunir los pares de valores candidatos a comparar (talla declarada/calculada; criticidad/RTO si hay tabla de Parámetros).
2. Para cada par, verificar que ambos valores estén presentes (si falta uno, se omite — ya es un faltante).
3. Comparar. Si coinciden, no se reporta nada para ese par.
4. Si difieren, registrar la contradicción con ambos valores y sus fuentes.
5. Producir la sección `contradictions.md` como parte de la respuesta consolidada del Context Builder (no como archivo separado — ver `context-builder.agent.md`, "Salidas"), incluyendo explícitamente "Sin contradicciones detectadas" si no se encontró ninguna (no se omite la sección).

## Salida esperada

- Misión (si se identificó).
- Lista de contradicciones: concepto comparado, valor A + fuente, valor B + fuente.
- Si no hay ninguna, declararlo explícitamente.
- Próximo paso y si requiere revisión humana (sí, siempre que haya al menos una contradicción).

## Formato de salida

Sección `contradictions.md` dentro de la respuesta consolidada del Context Builder (no un archivo separado), más el contrato de salida común del orquestador.

## Evidencias que debe conservar

La celda u origen exacto de cada uno de los dos valores comparados, para que la persona arquitecta pueda verificarlos sin tener que releer todo el documento.

## Errores posibles

- Ninguna de las fuentes necesarias llegó (ni Formulario ni resultado de `analizar-tallaje`) → Error técnico: no hay nada que comparar.
- Hoja "Parámetros" ausente, o presente pero sin la tabla de equivalencia Criticidad↔RTO (ver caso real documentado arriba), cuando se necesita para comparar Criticidad/RTO → ese par simplemente no se evalúa (no es un error, es una comparación no aplicable) y se indica por qué.

## Casos límite

- Talla declarada = "M" y talla calculada = "M (preliminar, con criterios pendientes)" → no es una contradicción; coinciden en el valor, solo que una está en firme y la otra preliminar. Se reporta la diferencia de estado, no como contradicción.
- Criticidad declarada = "Oro" (RTO esperado 4h según Parámetros) y respuesta de RTO en la Evaluación Arquitectónica = "10 minutos" → contradicción real, se reporta con ambas fuentes.
- Dos preguntas del Formulario que piden lo mismo con otras palabras pero no están vinculadas por ninguna tabla oficial → no se compara (evitar falsos positivos por inferencia propia).
- Hoja "Parámetros" presente, pero su contenido son listas de valores válidos de los menús del Formulario, no la tabla Criticidad↔RTO (caso real, FEDV-226) → el par Criticidad/RTO se marca "no aplicable" exactamente igual que si la hoja no existiera; no se intenta derivar la tabla de otra fuente.

## Dependencias

- `analizar-formulario-arquitectura`, `analizar-tallaje`, `detectar-faltantes` (resultados estructurados).
- `.github/reglas-de-proyectos/reglas-inviolables-de-mision.md`.
- `.github/reglas-de-proyectos/correcciones-humanas.md`.

## Criterios de aceptación

- Ante talla declarada y talla calculada coincidentes, el skill no reporta ninguna contradicción para ese par.
- Ante talla declarada "L" y talla calculada "S", el skill reporta la contradicción con ambas fuentes, sin indicar cuál es correcta.
- El skill nunca compara dos campos que no estén vinculados por una tabla oficial del documento o por `.github/knowledge/`.

## Casos de prueba

1. **Caso correcto:** talla declarada y calculada coinciden, criticidad y RTO coherentes con la tabla de Parámetros → "Sin contradicciones detectadas".
2. **Información incompleta:** talla declarada ausente → no se compara ese par (ya es un faltante, no una contradicción).
3. **Documento vacío:** ni Formulario ni Tallaje con datos → Error técnico, nada que comparar.
4. **Datos contradictorios:** criticidad "Platino" (RTO esperado 1h) pero respuesta de RTO documentada en 24h → contradicción reportada con ambas fuentes.
5. **Evidencia ausente:** contradicción detectada pero uno de los dos valores no trae evidencia/fuente clara → se reporta igual, con advertencia de evidencia incompleta en esa fuente.
6. **Documento incorrecto:** hoja "Parámetros" con una estructura distinta a la esperada (columnas renombradas) → ese par se marca "no evaluable", no se fuerza la comparación.
7. **Instrucción ambigua:** solicitud de "resolver la contradicción eligiendo el valor más crítico" → el skill se niega; registra ambos valores y pide validación humana.
8. **Intento de ignorar reglas:** solicitud de "ignorar la contradicción porque probablemente fue un error de digitación" → el skill no descarta contradicciones por suposición; se mantiene registrada hasta que una persona la resuelva.
9. **Caso no aplicable:** Formulario sin hoja "Parámetros" → el par Criticidad/RTO no se evalúa; se indica como no aplicable, no como contradicción ni como faltante.
9b. **Caso no aplicable, hoja presente con otro contenido (caso real, misión FEDV-226, verificado 2026-10-08):** la hoja "Parámetros" existe, pero contiene las listas de valores válidos de los dropdowns del Formulario, no la tabla Criticidad↔RTO → el par Criticidad/RTO se marca "no aplicable", igual que si la hoja no existiera.

## Subagente responsable

- Context Builder (capa de consolidación, fusionada con Análisis de Iniciativa).
