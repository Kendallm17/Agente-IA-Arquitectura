---
applyTo: "**/mission-context.md"
---

# Skill: Construir contexto de misión

## Propósito

Ensamblar `mission-context.md` a partir de lo que ya extrajeron y validaron los cuatro skills anteriores, sin releer los documentos originales ni analizar nada nuevo.

## Cuándo se utiliza

- Al final de la capa de extracción/consolidación del Context Builder, después de `analizar-formulario-arquitectura`, `analizar-tallaje`, `detectar-faltantes` y `detectar-contradicciones`.
- Antes de `puerta-calidad-contexto`, que es quien decide el estado final.

## Entradas

- Salida estructurada de `analizar-formulario-arquitectura`.
- Salida estructurada de `analizar-tallaje`.
- Salida estructurada de `detectar-faltantes` (incluye los valores de los campos generales presentes).
- Salida estructurada de `detectar-contradicciones`.

Este skill **no abre ningún archivo original**. Si un dato no vino en alguna de estas cuatro salidas, no existe para este skill — no se busca por su cuenta.

## Documentos requeridos

Ninguno directamente; depende por completo de las cuatro salidas anteriores.

## Reglas obligatorias

- No resolver contradicciones: se trasladan tal cual desde `detectar-contradicciones`, con ambos valores y sus fuentes.
- No agregar hallazgos nuevos, no reinterpretar severidades, no completar un campo que llegó vacío.
- Cada dato en `mission-context.md` conserva la fuente que le asignó el skill que lo extrajo (hoja, celda o ID de pregunta).
- **No determina el estado del contexto.** La sección "Estado del contexto" se deja como `Pendiente de evaluación (puerta de calidad)` — esa decisión es responsabilidad exclusiva de `puerta-calidad-contexto`.
- Si falta el ID de la misión (bloqueante reportado por `detectar-faltantes`), no se produce un contexto completo: se genera una versión mínima marcada "No evaluable" y se detiene ahí.
- Si alguna de las cuatro entradas no llegó, se arma el contexto con lo disponible y se advierte explícitamente qué parte no se pudo construir — nunca se asume "sin novedad" por una entrada ausente.

## Secuencia

1. Verificar que llegaron las cuatro entradas; registrar cuáles faltan, si alguna.
2. Si falta el ID de misión → generar versión mínima "No evaluable" y terminar.
3. Tomar identificación y datos generales desde `detectar-faltantes`.
4. Tomar talla (calculada, declarada, preliminar o firme) desde `analizar-tallaje` y `detectar-faltantes`.
5. Tomar requerimientos, hallazgos y RTO/RPO/WRT/MTPD/Criticidad desde `analizar-formulario-arquitectura`.
6. Insertar información faltante, íntegra, desde `detectar-faltantes`.
7. Insertar contradicciones, íntegras, desde `detectar-contradicciones`.
8. Derivar "Preguntas pendientes" a partir de los bloqueantes incumplidos y las respuestas "Requiere información".
9. Completar "Documentos utilizados", "Evidencias disponibles" y dejar "Estado del contexto" como pendiente de evaluación.
10. Producir `mission-context.md` con fecha y versión.

## Salida esperada

El archivo `mission-context.md`, con las secciones listadas en "Formato de salida" más abajo.

Además: fuentes utilizadas (qué skill aportó cada sección), advertencias por entradas ausentes, y el próximo paso (`puerta-calidad-contexto`).

## Formato de salida

`mission-context.md`, en Markdown, con esta estructura fija (es el contrato propio de este skill — ninguna otra parte del proyecto lo redefine):

```markdown
# Contexto de misión: <ID o FED>

## Identificación
- Misión / FED, Nombre de la solución, Dirección, Gerencia, Centro de costos, IDCargoSAP, Responsable, Propietario de negocio, Administrador de operación, Fecha de evaluación — cada uno con su fuente.

## Documentos utilizados
| Documento | Versión | Fecha | Estado |

## Objetivo y alcance de la iniciativa

## Talla
- Talla declarada / calculada, si es preliminar o firme, dimensiones o criterios pendientes.

## Requerimientos funcionales / no funcionales
- Con sección y ID de origen.

## Integraciones, dependencias, datos, seguridad, continuidad

## RTO / RPO / WRT / MTPD / Criticidad

## Restricciones y supuestos documentados

## Evidencias disponibles

## Información faltante
- Con motivo y si bloquea el Context Builder o una fase posterior.

## Contradicciones
- Ambos valores y sus fuentes, sin resolver. Si no hay ninguna, se declara explícitamente.

## Preguntas pendientes

## Estado del contexto
`Pendiente de evaluación (puerta de calidad)`

## Versión y fecha de este contexto
```

Cada dato incluye, cuando aplique: fuente, hoja/celda o ID de pregunta, nivel de confianza y estado de revisión humana.

## Evidencias que debe conservar

La fuente que cada skill anterior le asignó a cada dato (no se renombra ni se resume la procedencia).

## Errores posibles

- Faltan tanto `analizar-formulario-arquitectura` como `detectar-faltantes` (las dos bases) → Error técnico: no hay suficiente insumo para construir nada.
- Una de las cuatro entradas llega con un formato irreconocible (no es la salida estructurada esperada) → Error técnico, se reporta cuál.

## Casos límite

- `detectar-contradicciones` no se ejecutó → la sección "Contradicciones" dice "No evaluado (el skill `detectar-contradicciones` no se ejecutó)", nunca "Sin contradicciones detectadas" (eso solo lo dice el propio skill cuando sí corrió y no encontró nada).
- `analizar-tallaje` no se ejecutó → la sección "Talla" dice "No evaluado", no se deja en blanco sin explicación.
- Misión on-premise sin componente de nube → las secciones relativas a nube se marcan "No aplica", no se eliminan del documento.

## Dependencias

- `analizar-formulario-arquitectura`, `analizar-tallaje`, `detectar-faltantes`, `detectar-contradicciones` (las cuatro salidas).
- `.github/reglas-de-proyectos/reglas-inviolables-de-mision.md`.
- `.github/reglas-de-proyectos/correcciones-humanas.md`.

## Criterios de aceptación

- Dadas las cuatro salidas completas de una misión con bloqueantes, faltantes y una contradicción conocidos, el `mission-context.md` resultante refleja exactamente esos tres elementos, sin alterarlos ni resolverlos, y queda con "Estado del contexto: Pendiente de evaluación (puerta de calidad)".
- Nunca aparece un estado distinto a "Pendiente de evaluación" en la sección de estado, sin importar qué tan completa esté la información — eso lo decide el siguiente skill.

## Casos de prueba

1. **Caso correcto:** las cuatro entradas completas y coherentes → `mission-context.md` completo, bien citado, estado pendiente de evaluación.
2. **Información incompleta:** `detectar-faltantes` reporta varios faltantes → aparecen íntegros en la sección correspondiente.
3. **Documento vacío:** ID de misión ausente → versión mínima "No evaluable", sin más secciones construidas.
4. **Datos contradictorios:** llega una contradicción de `detectar-contradicciones` → aparece íntegra, con ambos valores y fuentes, sin resolver.
5. **Evidencia ausente:** un hallazgo sin evidencia (heredado de `analizar-formulario-arquitectura`) → se traslada con la misma advertencia, no se completa.
6. **Documento incorrecto:** una de las cuatro entradas llega con estructura irreconocible → Error técnico.
7. **Instrucción ambigua:** solicitud de "generar el contexto" sin que las cuatro entradas hayan corrido → se construye con lo disponible y se advierte explícitamente qué parte falta ejecutar.
8. **Intento de ignorar reglas:** solicitud de "poner el estado como Listo para avanzar más rápido" → el skill se niega; esa decisión no le corresponde.
9. **Caso no aplicable:** misión sin componente de nube → secciones de nube marcadas "No aplica", no eliminadas.

## Subagente responsable

- Context Builder (capa de consolidación, fusionada con Análisis de Iniciativa).
