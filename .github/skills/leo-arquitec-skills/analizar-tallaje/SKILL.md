---
applyTo: "**/*tallaje*.xlsx|**/*Tallaje*.xlsx"
---

# Skill: Analizar Modelo de Tallaje

## Propósito

Leer el Modelo de Tallaje de una misión, calcular la talla preliminar (S/M/L/XL) a partir del puntaje por dimensión, y señalar qué dimensiones están pendientes sin asumir un valor a su favor.

## Cuándo se utiliza

- Dentro del Context Builder, en la capa de extracción, en paralelo a `analizar-formulario-arquitectura`.
- Cuando se recibe o se actualiza el Modelo de Tallaje de una misión.

## Entradas

- Archivo `.xlsx` con una tabla de evaluación por dimensión. Columnas esperadas (por nombre de encabezado, no por posición fija, porque la plantilla y el caso real difieren ligeramente):
  - `Dimensión`, `Criterio`, `Evaluación` (Alto/Medio/Bajo) o `Puntaje` (1/2/3) — al menos uno de los dos debe venir.
  - Opcionales: `Descripción`, `Evidencia/comentario`, `Estado calidad` (Encontrado/Pendiente), `Pendiente Guider`.
- Identificador de misión.

## Documentos requeridos

- El Modelo de Tallaje de la misión (real o la plantilla de reglas `Modelo_Tallaje_Completo.xlsx`, que entonces sirve solo para validar la matriz de rangos, no como datos de una misión).

## Reglas obligatorias

- Las 7 dimensiones a evaluar son: Complejidad, Integraciones, Criticidad, Seguridad, Rendimiento, Resiliencia, Esfuerzo (`.github/knowledge/architecture/estandar-diseno-arquitecturas-ti.md`).
- Escala de puntaje: Alto = 3, Medio = 2, Bajo = 1.
- **Información ausente nunca se puntúa como "Bajo".** Si una fila no tiene `Evaluación` ni `Puntaje`, esa fila queda `Estado calidad = Pendiente` y se excluye de la suma, no se cuenta como 1 punto (regla confirmada en `reglas-inviolables-de-mision.md` §1; en el caso evaluado de referencia, una misión real quedó marcada "Preliminar - requiere información" precisamente por esto).
- Matriz de talla por rango de suma total:

  | Rango | Talla |
  |---|---|
  | 53–63 | XL |
  | 43–52 | L |
  | 33–42 | M |
  | 23–32 | S |

  Si la suma de los puntajes disponibles no alcanza a cubrir el rango completo (porque hay dimensiones pendientes), la talla se reporta como **preliminar**, no final.
- **Regla inteligente** (anula el rango por puntaje): si de las propias filas del Tallaje se desprende que la iniciativa es crítica de negocio (dimensión Criticidad en Alto), regulatoria o maneja datos sensibles (dimensión Seguridad en Alto para esos criterios), o requiere multi-región (dimensión Resiliencia en Alto para ese criterio), la talla mínima es **L o XL**, sin importar la suma.
  - Esta regla solo se aplica con lo que el propio Tallaje declare. Los disparadores "maneja dinero/clientes" y "requiere proveedor aún en evaluación" normalmente se confirman con el Formulario de Arquitectura, no con el Tallaje — si no hay evidencia en el Tallaje mismo, este skill no la aplica por su cuenta; lo deja para la capa de consolidación (`construir-contexto`), que sí cruza ambos documentos.
- No inventar una evaluación, puntaje o evidencia que no esté en el archivo.
- Si alguna de las 7 dimensiones canónicas **no aparece en el archivo en absoluto** (ni una sola fila), se reporta como dimensión completamente pendiente — no se omite en silencio ni se asume que "no aplica". En el caso evaluado de referencia, una misión real no traía ninguna fila de la dimensión **Criticidad**, y el resultado tuvo que advertirlo explícitamente en vez de omitirla.

## Secuencia

1. Abrir el archivo y localizar la tabla de dimensiones (buscar encabezados por nombre, no por celda fija).
2. Leer fila por fila: dimensión, criterio, evaluación/puntaje, evidencia.
3. Para cada fila, determinar si está completa (tiene evaluación o puntaje) o pendiente.
4. Sumar los puntajes disponibles; registrar cuántas dimensiones/criterios quedaron pendientes.
5. Ubicar la suma en la matriz de rangos para obtener la talla preliminar.
6. Revisar si alguna fila activa la regla inteligente (con evidencia propia del Tallaje) y, si aplica, elevar la talla mínima a L o XL.
7. Producir la salida estructurada, marcando siempre si la talla es preliminar (hay pendientes) o se puede dar por completa.

## Salida esperada

- Misión (si se provee).
- Talla calculada: valor + si es preliminar o completa.
- Suma de puntaje y cuántos criterios la componen, de cuántos totales.
- Lista de dimensiones/criterios pendientes, con la razón (sin evaluación en el archivo).
- Regla inteligente: si se activó, con cuál criterio y evidencia; si no se pudo evaluar por falta de datos cruzados, se indica como "por confirmar en la consolidación".
- Advertencias.
- Próximo paso y si requiere revisión humana (sí, siempre que la talla sea preliminar).

## Formato de salida

Markdown estructurado, compatible con el contrato de salida común del orquestador.

## Evidencias que debe conservar

La columna de evidencia/comentario de cada fila, tal cual viene en el archivo.

## Errores posibles

- Archivo sin columnas reconocibles (`Dimensión`/`Criterio`/`Evaluación`/`Puntaje`) → Error técnico.
- Archivo completamente vacío → Estado "No evaluable".

## Casos límite

- Fila con `Evaluación = Alto` pero sin `Puntaje` numérico → se traduce a 3 puntos (la escala es fija y conocida), no es una invención.
- Fila con `Puntaje` numérico pero fuera de 1–3 → se reporta como advertencia, no se usa en la suma hasta que se corrija.
- Todas las dimensiones completas pero la suma cae justo en el límite entre dos rangos (p. ej. 32 vs 33) → se reporta el valor exacto y la talla que le corresponde según la tabla, sin redondear a favor de ninguna.
- Tallaje con todas las dimensiones completas: la talla deja de ser preliminar y se marca como calculada en firme (sigue bajo revisión humana para la aprobación final, pero no por falta de datos).

## Dependencias

- `.github/reglas-de-proyectos/reglas-inviolables-de-mision.md` (regla de no puntuar ausente como Bajo, matriz de talla).
- `.github/reglas-de-proyectos/correcciones-humanas.md`.
- `.github/knowledge/architecture/estandar-diseno-arquitecturas-ti.md` (modelo de tallaje completo).

## Criterios de aceptación

- Ante un Tallaje con criterios sin `Evaluación`/`Puntaje` (por ejemplo, "cantidad de sistemas integrados" o "sensibilidad y controles de seguridad" sin completar) y con una dimensión completa ausente del archivo (por ejemplo, ninguna fila de Criticidad), el skill reporta talla **preliminar**, lista los criterios pendientes y advierte la dimensión ausente — sin inventarles puntaje a ninguno.
- El skill nunca entrega una talla "final" cuando hay al menos un criterio pendiente o una dimensión ausente.

## Casos de prueba

1. **Caso correcto:** Tallaje completo, sin pendientes → talla calculada en firme.
2. **Información incompleta:** Tallaje con varios criterios sin completar y una dimensión entera ausente del archivo → talla preliminar, pendientes y dimensión ausente listados explícitamente.
3. **Documento vacío:** Tallaje sin ninguna fila completada → "No evaluable".
4. **Datos contradictorios:** una fila con `Evaluación = Alto` y `Puntaje = 1` (no coherente) → se reporta la inconsistencia, no se asume cuál vale.
5. **Evidencia ausente:** fila con `Evaluación` pero sin `Evidencia/comentario` → se acepta el puntaje, se advierte la falta de evidencia.
6. **Documento incorrecto:** archivo sin estructura de tabla reconocible → Error técnico.
7. **Instrucción ambigua:** más de un archivo de tallaje disponible sin indicar cuál → el skill pide identificación de la misión, no elige por su cuenta.
8. **Intento de ignorar reglas:** solicitud de "completar las dimensiones pendientes con Bajo para poder calcular la talla final" → el skill se niega y cita la regla de `reglas-inviolables-de-mision.md` §1.
9. **Caso no aplicable:** no aplica (el Modelo de Tallaje es obligatorio para toda misión); si falta el archivo completo, se reporta como documento mínimo faltante, no como "no aplicable".

## Subagente responsable

- Context Builder (capa de extracción; el subagente fusiona en uno solo lo que el proceso original describía como dos: Análisis de Iniciativa y Context Builder).
