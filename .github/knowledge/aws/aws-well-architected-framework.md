---
name: aws-well-architected-framework
description: "Conocimiento institucional: AWS Well-Architected Framework (6 pilares). Consultar al valorar o recomendar una arquitectura sobre AWS. Use when: AWS, well-architected, OPS, SEC, REL, PERF, COST, SUS."
applyTo: "**/*aws*"
---

# Conocimiento: AWS Well-Architected Framework

## Propósito

Proveer al Agente Evaluador los principios de diseño y las preguntas oficiales de AWS para valorar una arquitectura que use (o podría usar) AWS, y para fundamentar recomendaciones con la fuente correcta.

## Fuente

- Documento: "AWS Well-Architected Framework" (PDF oficial de AWS). Publicación: 6 de noviembre de 2024. 1094 páginas; este resumen cubre la visión general de los 6 pilares (principios de diseño + catálogo de preguntas). El apéndice completo con cada práctica recomendada en detalle queda en el PDF original; se cita por pilar y número de pregunta (p. ej. "OPS 7") para que quien lo necesite localice el detalle.

## Cuándo se consulta

- Al evaluar o proponer una arquitectura sobre AWS.
- Al comparar AWS frente a Azure u otra alternativa en la Valoración Arquitectónica.
- Al verificar que una recomendación de nube esté fundamentada en un principio reconocido, no inventado.

## Los 6 pilares y sus principios de diseño

### 1. Excelencia operativa (OPS)
Organizar equipos en torno a resultados de negocio; implementar observabilidad para información práctica; automatizar con seguridad; cambios frecuentes, pequeños y reversibles; refinar procedimientos operativos con regularidad; anticipar el fracaso; aprender de todos los eventos y métricas; usar servicios administrados.

Preguntas: OPS1 prioridades · OPS2 estructura organizacional · OPS3 cultura · OPS4 observabilidad · OPS5 reducir defectos y mejorar el flujo · OPS6 mitigar riesgos de implementación · OPS7 preparación para dar soporte · OPS8 uso de la observabilidad · OPS9 comprensión del estado de operaciones · OPS10 gestión de eventos · OPS11 evolución de operaciones.

### 2. Seguridad (SEC)
Bases de identidad sólidas (mínimo privilegio, sin credenciales de larga duración); trazabilidad (auditoría y alertas en tiempo real); seguridad en todas las capas (defensa en profundidad); automatización de prácticas de seguridad; protección de datos en tránsito y en reposo; alejar a las personas del acceso directo a datos; preparación para eventos de seguridad.

Preguntas: SEC1 uso seguro de la carga de trabajo · SEC2 identidades de personas y máquinas · SEC3 autenticación · SEC4 detección e investigación · SEC5 protección de redes · SEC7 clasificación de datos · SEC8 datos en reposo · SEC9 datos en tránsito · SEC10 respuesta y recuperación de incidentes · SEC11 seguridad de aplicaciones.

### 3. Fiabilidad (REL)
Recuperación automática ante errores; probar los procedimientos de recuperación; escalado horizontal para aumentar disponibilidad agregada; no adivinar la capacidad (monitorear demanda y automatizar); gestionar cambios mediante automatización.

Preguntas: REL1 cuotas de servicio y restricciones · REL2 topología de red · REL4 diseño de interacciones para evitar errores · REL5 mitigar/tolerar errores · REL6 supervisión de recursos · REL7 adaptación a cambios de demanda · REL9 respaldo de datos · REL10 aislamiento de errores · REL11 soportar fallas de componentes · REL12 pruebas de fiabilidad · REL13 planificación de DR.

### 4. Eficiencia del rendimiento (PERF)
Democratizar tecnologías avanzadas (consumirlas como servicio); adoptar un enfoque global en minutos (multi-región); usar arquitecturas sin servidor; experimentar con más frecuencia; considerar la "simpatía mecánica" (elegir el enfoque que mejor se adapte a la carga).

Preguntas: PERF1 selección de recursos/patrones · PERF2 cómputo · PERF3 datos · PERF4 red · PERF5 proceso para mejorar continuamente el rendimiento.

### 5. Optimización de costos (COST)
Implementar administración financiera en la nube; adoptar un modelo de consumo (pagar solo por lo usado); evaluar la eficacia global; eliminar gasto en tareas pesadas no diferenciadas; analizar y atribuir gastos.

Preguntas: COST1 administración financiera · COST2 control del uso · COST3 supervisión de uso y costo · COST4 retiro de recursos · COST5 costo al seleccionar servicios · COST6 tipo/tamaño/número de recursos · COST7 modelos de fijación de precios · COST8 transferencia de datos · COST9 gestión de demanda y aprovisionamiento · COST10 evaluación de servicios nuevos.

### 6. Sostenibilidad (SUS)
Comprender el impacto; establecer objetivos de sostenibilidad; maximizar el uso (evitar recursos infrautilizados); anticipar y adoptar hardware/software más eficiente; usar servicios administrados; reducir el impacto indirecto en el cliente final.

Preguntas: SUS1 selección de regiones · SUS2 alinear recursos a la demanda · SUS3 patrones de software/arquitectura · SUS4 gestión de datos · SUS5 hardware y servicios · SUS6 procesos organizativos.

## Reglas obligatorias para el agente

- No recomendar un servicio de AWS sin vincularlo a un principio o pregunta de este marco (o al lineamiento interno si aplica).
- Si una misión declara AWS como proveedor, el pilar de Seguridad y el de Fiabilidad se evalúan primero (contienen las preguntas con mayor severidad típica: identidad, cifrado, DR).
- Citar siempre el código de pregunta (p. ej. "SEC9") como fuente del hallazgo, no una paráfrasis sin trazabilidad.
- Cuando la profundidad de una pregunta no esté cubierta en este resumen, indicarlo como "requiere consulta al documento completo (página de la sección correspondiente)" en vez de inventar la práctica recomendada.

## Proceso de aplicación por el agente

1. Confirmar que la misión usa o evalúa AWS.
2. Recorrer los 6 pilares aplicando las preguntas pertinentes al tipo de carga de trabajo.
3. Registrar cumplimiento/brecha por pregunta, con evidencia de la misión.
4. Priorizar Seguridad y Fiabilidad cuando haya datos sensibles o criticidad alta (coherente con `lineamiento-uso-nube-publica.md`).

## Salida esperada al ser consultado

- Lista de preguntas aplicables con su código, pilar y estado (cumple / no cumple / pendiente).
- Vínculo explícito entre cada hallazgo y el principio de diseño que lo sustenta.

## Errores y casos límite

- No aplicar este marco si la misión es exclusivamente on-premise o Azure sin componente AWS.
- No fusionar preguntas de pilares distintos en un solo hallazgo.
- No asumir cumplimiento de una pregunta sin evidencia en la misión.

## Casos de prueba

1. Misión AWS sin MFA documentado → brecha en SEC3.
2. Misión AWS sin plan de DR → brecha en REL13.
3. Misión AWS con autoescalado y pruebas de carga documentadas → cumple REL7 y PERF5.
4. Misión sin proveedor de nube definido → este documento no aplica todavía; se marca información pendiente.

## Consultado por

- Subagente de Valoración Arquitectónica.
- Subagente de Gobierno y Cumplimiento.
