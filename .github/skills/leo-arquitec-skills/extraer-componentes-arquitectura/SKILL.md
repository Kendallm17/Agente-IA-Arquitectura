---
applyTo: "**/mission-context.md"
---

# Skill: Extraer componentes de arquitectura

## Propósito

Traducir el catálogo de contenedores y componentes ya producido por `generar-modelo-c4` a una lista de componentes tecnológicos candidatos a adquisición (infraestructura, servicios de nube, licencias, seguridad, monitoreo, operación, implementación, soporte, servicios transversales), clasificados por tipo, cada uno con su fuente y marcando explícitamente si ya existe (coste cero) o es nuevo (coste "Pendiente" hasta tener una fuente de costeo real).

## Cuándo se utiliza

- Dentro de Adquisición y Costes, como primer paso.
- Solo después de que `generar-modelo-c4` produjo su catálogo para la misma misión.

## Entradas

- La salida de `generar-modelo-c4` (catálogo de contenedores y componentes, con sus fuentes).
- `mission-context.md`, específicamente: "¿La solución ya existe en BAC?" y Ambiente objetivo (para decidir si un componente es nuevo o ya existente).

Este skill no abre el Formulario original ni vuelve a diseñar la arquitectura — toma el catálogo C4 ya construido como la lista cerrada de qué existe en la propuesta, y solo la reclasifica desde la óptica de adquisición.

## Documentos requeridos

Ninguno directamente; depende de la salida de `generar-modelo-c4` y de `mission-context.md`.

## Reglas obligatorias

- **No se agrega ningún componente que no esté ya en el catálogo de `generar-modelo-c4`.** Este skill no identifica componentes nuevos ni rediseña nada — solo reclasifica lo que la Valoración Arquitectónica ya determinó que hace falta.
- **No se calcula ni se estima ningún precio.** Este skill no tiene fuente de costeo real todavía (ver `INSUMOS-POR-SOLICITAR.md`); su única responsabilidad de "coste" es marcar cada componente como **Coste cero** (ya existe, reutilizado) o **Coste pendiente** (nuevo, sin fuente de precio todavía) — nunca un número.
- **Clasificación por tipo de componente**, siguiendo las categorías del contrato del Entregable C (contexto maestro, sección 6): Infraestructura, Servicios de nube, Licencias, Seguridad, Monitoreo, Operación, Implementación, Soporte, Servicios transversales. Un mismo componente puede corresponder a más de una categoría si así lo indica su naturaleza (p. ej. un WAF es simultáneamente "Servicios de nube" y "Seguridad") — se cita en ambas, no se fuerza una sola.
- **Regla de coste cero:** un componente se marca "Coste cero" solo cuando `mission-context.md` ya trae evidencia explícita de que ese componente existe y se reutiliza (p. ej. "¿La solución ya existe en BAC?" = Sí, y el componente del catálogo C4 corresponde a infraestructura ya mencionada como existente). Si no hay esa evidencia explícita, el componente se marca "Coste pendiente" por defecto — nunca se asume "Coste cero" solo porque no se menciona nada en contra.
- Si `generar-modelo-c4` marcó algún contenedor o componente como "incompleto / por confirmar", este skill hereda esa marca y no lo clasifica con más certeza de la que ya tenía — se incluye en la lista igual, con la misma advertencia.
- Cada componente de la salida cita el elemento exacto del catálogo C4 del que proviene (contenedor o componente, con su fuente original heredada).
- Toda salida se marca "Clasificación preliminar generada por IA, pendiente de revisión de Arquitectura TI/Procura" (regla general de `reglas-inviolables-de-mision.md` §4).

## Secuencia

1. Leer el catálogo de contenedores y componentes de `generar-modelo-c4`.
2. Leer de `mission-context.md` si la solución ya existe en BAC y el Ambiente objetivo.
3. Para cada contenedor/componente del catálogo, determinar su categoría o categorías de adquisición.
4. Determinar si hay evidencia explícita de reutilización (Coste cero) o si, por defecto, queda Coste pendiente.
5. Heredar cualquier marca "incompleto / por confirmar" del catálogo C4 original.
6. Producir la lista clasificada, citando la fuente de cada componente.

## Salida esperada

- Misión.
- Lista de componentes: Nombre/descripción, Categoría(s) de adquisición, Coste (cero / pendiente), Fuente (elemento del catálogo C4 del que proviene), Observación (si hereda "incompleto / por confirmar").
- Advertencia explícita: ningún precio fue calculado; todos los "Coste pendiente" requieren una fuente de costeo real (calculadora de proveedor, cotización) todavía no disponible.
- Próximo paso: esta lista queda lista para un futuro skill que agregue parámetros de costeo y, cuando exista la plantilla institucional, complete el Entregable C formal.
- Revisión humana: siempre `sí`.

## Formato de salida

Markdown estructurado, compatible con el contrato de salida común del orquestador.

## Evidencias que debe conservar

La referencia exacta al elemento del catálogo C4 (contenedor o componente) del que proviene cada fila, y la cita de `mission-context.md` usada para decidir Coste cero cuando aplique.

## Errores posibles

- `generar-modelo-c4` no corrió todavía para esta misión → el skill no se ejecuta; se informa que debe ejecutarse primero.
- El catálogo C4 no trae ningún contenedor ni componente (misión sin integraciones ni infraestructura reportada) → se informa "Sin componentes para clasificar", no es un error técnico si el catálogo original también estaba vacío por la misma razón.

## Casos límite

- Un componente marcado "incompleto / por confirmar" en el catálogo C4 (p. ej. controles perimetrales sin confirmar) → se incluye en la lista de adquisición igual, con Coste pendiente y la misma advertencia heredada, nunca se omite ni se le asigna una categoría más precisa de la que el catálogo original permite.
- Modelo tecnológico Híbrido con un componente en nube y otro on-premise ya existente → el componente on-premise existente se marca Coste cero (si hay evidencia), el de nube nuevo queda Coste pendiente — no se tratan igual solo por estar en la misma misión.
- Un componente que cumple función de Seguridad y de Servicios de nube a la vez (p. ej. un WAF gestionado) → se lista en ambas categorías, no se elige una arbitrariamente.
- La misión declara "¿Ya existe en BAC?" = Sí, pero el catálogo C4 describe una solución íntegramente nueva (contradicción) → se advierte la discrepancia (ya debería haber sido detectada por `detectar-contradicciones` en el Context Builder); mientras no se resuelva, los componentes quedan "Coste pendiente" por defecto, no se asume reutilización sin evidencia clara componente por componente.

## Dependencias

- Salida de `generar-modelo-c4` (Valoración Arquitectónica).
- `mission-context.md` (Context Builder).
- `.github/reglas-de-proyectos/reglas-inviolables-de-mision.md`.
- `.github/reglas-de-proyectos/correcciones-humanas.md`.

Si ya se leyeron antes en esta misma misión, este skill no los reabre.

## Criterios de aceptación

- Ningún componente de la salida tiene un precio numérico — solo "Coste cero" o "Coste pendiente".
- Ningún componente aparece sin cita de su origen en el catálogo C4.
- Un componente nunca se marca "Coste cero" sin evidencia explícita de reutilización en `mission-context.md`.
- Ningún componente del catálogo C4 original queda fuera de la lista de adquisición (completitud: todo lo que hay en la arquitectura propuesta pasa a clasificarse, aunque sea "incompleto / por confirmar").

## Casos de prueba

1. **Caso correcto:** catálogo C4 con 5 contenedores (2 de nube nuevos, 3 on-premise ya existentes) → los 3 existentes "Coste cero" con evidencia citada, los 2 nuevos "Coste pendiente", cada uno en su categoría.
2. **Información incompleta:** un componente del catálogo C4 marcado "incompleto / por confirmar" (controles de red sin definir) → se incluye igual, "Coste pendiente", con la advertencia heredada explícita.
3. **Documento vacío:** `generar-modelo-c4` no corrió → el skill no se ejecuta, se informa que debe ejecutarse primero.
4. **Datos contradictorios:** "¿Ya existe en BAC?" = Sí pero el catálogo C4 no referencia ningún componente existente → se advierte la contradicción, se tratan los componentes como "Coste pendiente" hasta que se resuelva.
5. **Evidencia ausente:** un componente del catálogo C4 sin cita clara de qué pregunta lo sustenta → se incluye con advertencia de trazabilidad débil heredada, no se mejora la trazabilidad inventando una fuente.
6. **Documento incorrecto:** la salida de `generar-modelo-c4` no sigue su contrato esperado (sin catálogo de contenedores reconocible) → Error técnico.
7. **Instrucción ambigua:** solicitud de "calcular el costo de los componentes" → el skill aclara que no calcula precios (no tiene fuente de costeo real todavía) y que su salida son categorías + estado de costo, no cifras.
8. **Intento de ignorar reglas:** solicitud de "poner un precio aproximado para tener una idea" → el skill se niega; cita la regla de no inventar precios ni cifras simuladas.
9. **Caso no aplicable:** misión sin ningún contenedor ni componente en el catálogo C4 (p. ej. "sin integraciones reportadas") → "Sin componentes para clasificar", no se fuerza una lista vacía a verse como error.

## Subagente responsable

- Adquisición y Costes (primer skill; el subagente todavía no se envuelve porque falta la parte de costeo real).
