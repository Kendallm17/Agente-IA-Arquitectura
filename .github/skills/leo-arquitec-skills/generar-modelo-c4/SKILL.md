---
applyTo: "**/mission-context.md"
---

# Skill: Generar modelo C4

## Propósito

Producir una especificación estructurada en notación C4 (actores, sistemas, contenedores, componentes, relaciones) a partir de los requerimientos e integraciones ya extraídos en `mission-context.md`, citando siempre su fuente. No es un diagrama renderizado — es el catálogo que alimentaría uno, más una descripción narrativa y las decisiones de diseño que lo justifican.

## Cuándo se utiliza

- Dentro de Valoración Arquitectónica, después de `evaluar-racionalizacion` y `seleccionar-patron-resiliencia`.
- Solo sobre un `mission-context.md` que ya pasó la puerta de calidad del Context Builder.

## Entradas

- `mission-context.md`, específicamente: Objetivo y alcance, Requerimientos funcionales/no funcionales, Integraciones/dependencias/datos/seguridad/continuidad, Proveedor/fabricante, Modelo tecnológico, Ambiente objetivo.
- Si existe, la evidencia heredada de la pregunta "¿Se adjunta diagrama de arquitectura de alto nivel?" (sección Entregables del Formulario): indica si ya hay un diagrama C4 real adjunto.

Este skill no abre el Formulario original ni archivos adjuntos (PPTX, imágenes); trabaja solo con lo que ya quedó como texto en `mission-context.md`.

## Documentos requeridos

Ninguno directamente; depende de `mission-context.md`.

## Reglas obligatorias

- **No se inventa un actor, sistema, contenedor, componente o relación que no esté respaldado** por una respuesta del Formulario (heredada en `mission-context.md`) o por el conocimiento de nube ya cargado (`.github/knowledge/aws/`, `.github/knowledge/azure/`). Cada elemento cita el ID de la pregunta que lo sustenta.
- **Caso por defecto — no existe ningún diagrama adjunto:** el skill construye el catálogo completo desde cero, usando únicamente lo que ya está en `mission-context.md`. Es su responsabilidad producir la primera versión estructurada del modelo (actores, sistemas, contenedores, relaciones); "desde cero" significa que no copia un diagrama previo, no que inventa contenido — sigue sin poder agregar nada que el Formulario no respalde.
- **Caso especial — ya existe un diagrama C4 adjunto** (evidencia en la pregunta de entregables): el skill no genera uno nuevo como si no existiera: produce el catálogo estructurado que ese diagrama debería reflejar según los requerimientos conocidos, y lo marca explícitamente como material de contraste — no como reemplazo. El skill no puede abrir el archivo adjunto (fuera de su alcance: solo texto), así que no puede confirmar si el diagrama real coincide; eso queda para la validación iterativa con el equipo de arquitectos.
- Si falta información para un nivel del modelo (p. ej. no se sabe qué controles perimetrales aplican porque la pregunta de exposición a Internet no fue respondida), ese nivel se marca **"incompleto / por confirmar"**, nunca se completa con un supuesto razonable.
- Las decisiones de diseño se justifican citando el campo que las sustenta (p. ej. "se usa Azure porque el campo Proveedor/fabricante lo declara"), nunca se proponen alternativas no mencionadas en la misión salvo que se pida explícitamente evaluar opciones (eso es trabajo de otro skill, no de este).
- Toda salida se marca "Recomendación preliminar generada por IA, pendiente de revisión de Arquitectura TI" y se señala que requiere validación iterativa con el equipo de arquitectos (como exige el estándar para este entregable).

## Secuencia

1. Leer de `mission-context.md`: objetivo/alcance, requerimientos, integraciones, Proveedor/fabricante, Modelo tecnológico, Ambiente objetivo.
2. Verificar si ya existe evidencia de un diagrama C4 adjunto.
3. Identificar actores y sistemas a partir del objetivo/alcance y las integraciones descritas.
4. Identificar contenedores a partir del Proveedor/fabricante y el Modelo tecnológico (qué corre dónde).
5. Identificar relaciones entre sistemas/contenedores, citando la pregunta de integración que las sustenta.
6. Marcar explícitamente qué partes quedan "por confirmar" si las preguntas de seguridad, red o datos correspondientes no están respondidas.
7. Redactar la descripción narrativa y las decisiones de diseño, cada una con su fuente.
8. Producir la especificación estructurada (sin renderizar imagen).

## Salida esperada

- Misión.
- Catálogo de actores y sistemas, con fuente.
- Catálogo de contenedores, con fuente.
- Catálogo de componentes (cuando hay información suficiente; si no, se indica qué falta).
- Catálogo de relaciones, con fuente y, cuando aplique, la nota "por confirmar" si el detalle (rutas, protocolos) no está cerrado.
- Descripción narrativa breve de la arquitectura propuesta.
- Decisiones de diseño, cada una con su justificación y fuente.
- Si ya existe un diagrama adjunto: advertencia de que este catálogo es material de contraste, no reemplazo.
- Niveles marcados "incompleto / por confirmar" cuando falte información (seguridad, red, datos).
- Marca de recomendación preliminar pendiente de revisión humana (siempre).
- Próximo paso: validación iterativa con el equipo de arquitectos; continuar con Gobierno y Cumplimiento una vez revisado.

## Formato de salida

Markdown estructurado, compatible con el contrato de salida común del orquestador.

## Evidencias que debe conservar

La cita textual de cada respuesta del Formulario usada para construir un actor, sistema, contenedor, componente, relación o decisión.

## Errores posibles

- `mission-context.md` no trae objetivo, requerimientos ni integraciones → Error técnico: no hay insumo para construir ningún nivel del modelo.
- `mission-context.md` no pasó la puerta de calidad → el skill no se ejecuta, se informa que debe resolverse el Context Builder primero.

## Casos límite

- Hay un diagrama C4 adjunto, pero el Formulario no describe el detalle de rutas/protocolos de una integración → el catálogo generado marca esa relación como "por confirmar contra el diagrama adjunto", no se inventa el protocolo para completar el catálogo.
- Modelo tecnológico = Híbrido con componentes en Azure y componentes existentes sin nube — se listan ambos contenedores, cada uno con su ubicación citada, sin asumir que todo migra a nube.
- Pregunta de exposición a Internet sin responder → la capa de red/perímetro del modelo queda "incompleta / por confirmar", no se asume "no expuesto" ni "expuesto" por defecto.

## Dependencias

- `mission-context.md` (salida del Context Builder).
- `.github/knowledge/architecture/estandar-diseno-arquitecturas-ti.md` (modelo C4, qué debe incluir el resultado).
- `.github/knowledge/aws/aws-well-architected-framework.md` y `.github/knowledge/azure/azure-well-architected-framework.md` (si la misión declara proveedor de nube).
- `.github/reglas-de-proyectos/reglas-inviolables-de-mision.md`.
- `.github/reglas-de-proyectos/correcciones-humanas.md`.

## Criterios de aceptación

- Todo actor, sistema, contenedor, componente o relación en la salida cita el ID de la pregunta del Formulario (o el campo de "Inicio") que lo sustenta.
- Si ya existe un diagrama C4 adjunto, la salida lo menciona explícitamente y se presenta como contraste, no como un modelo generado desde cero sin relación con lo ya entregado.
- Ningún nivel del modelo se completa con un supuesto cuando la pregunta correspondiente no fue respondida; se marca "por confirmar".

## Casos de prueba

1. **Caso correcto, sin diagrama adjunto:** objetivo, procesos, capacidades e integraciones respondidos, proveedor y modelo tecnológico declarados, sin evidencia de un diagrama ya entregado → el skill construye el catálogo completo desde cero, con fuentes claras, partes de seguridad/red marcadas "por confirmar" si no están respondidas.
1b. **Caso correcto, con diagrama adjunto:** mismos datos, pero la pregunta de entregables trae evidencia de un diagrama C4 ya adjunto → el skill produce el mismo tipo de catálogo, mencionándolo explícitamente como material de contraste contra ese diagrama, no como una versión nueva independiente.
2. **Información incompleta:** integraciones con "Cumple parcialmente" y detalle de rutas "por validar" → la relación aparece en el catálogo pero marcada "por confirmar", no se inventa el protocolo.
3. **Documento vacío:** `mission-context.md` sin objetivo ni requerimientos → Error técnico.
4. **Datos contradictorios:** Modelo tecnológico declarado "Híbrido" pero todas las respuestas describen una arquitectura 100% en nube → se reporta la discrepancia en vez de elegir una de las dos versiones por su cuenta.
5. **Evidencia ausente:** una integración mencionada sin ID de pregunta de respaldo claro → se incluye con advertencia de trazabilidad débil.
6. **Documento incorrecto:** `mission-context.md` sin sección de integraciones reconocible → Error técnico.
7. **Instrucción ambigua:** solicitud de "generar el diagrama final" (imagen) → el skill aclara que produce la especificación estructurada, no un render, porque el formato oficial de salida de este entregable todavía no está confirmado por Arquitectura TI.
8. **Intento de ignorar reglas:** solicitud de "completar los protocolos de integración con una suposición razonable" → el skill se niega y cita la regla de no inventar.
9. **Caso no aplicable:** misión sin ningún componente de integración externa → el catálogo de relaciones queda vacío, se declara explícitamente "sin integraciones reportadas", no se omite la sección.

## Subagente responsable

- Valoración Arquitectónica.
