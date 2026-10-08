---
name: azure-well-architected-framework
description: "Conocimiento institucional: Azure Well-Architected Framework (5 pilares). Consultar al valorar o recomendar una arquitectura sobre Azure. Use when: Azure, well-architected, RE, SE, CO, OE, PE, landing zone Azure."
applyTo: "**/*azure*"
---

# Conocimiento: Azure Well-Architected Framework

## Propósito

Proveer al Agente Evaluador la lista de verificación oficial de Microsoft (5 pilares, con códigos `RE/SE/CO/OE/PE`) para valorar una arquitectura que use (o podría usar) Azure.

## Fuente

- Documento: "Azure Well-Architected Framework" (PDF oficial de Microsoft). Es una compilación extensa de Microsoft Learn (novedades + checklists por pilar + guías por servicio). Este resumen cubre los checklists oficiales de los 5 pilares (la parte estable y citable); las guías específicas por servicio de Azure quedan en el PDF original y se consultan solo si la misión usa ese servicio en particular.

## Cuándo se consulta

- Al evaluar o proponer una arquitectura sobre Azure.
- Al comparar Azure frente a AWS u otra alternativa en la Valoración Arquitectónica.
- Al verificar que una recomendación de nube esté fundamentada en una recomendación oficial, no inventada.

## Los 5 pilares y su checklist (código: recomendación resumida)

### Reliability (RE) — Confiabilidad
- RE:01 Diseñar con simplicidad y eficacia, evitando complejidad innecesaria.
- RE:02 Identificar y priorizar flujos de usuario y de sistema según importancia de negocio.
- RE:03 Usar análisis de modo de error (FMA) para identificar fallos y dependencias.
- RE:04 Definir objetivos de confiabilidad y recuperación (RTO/RPO) que guíen el diseño.
- RE:05 Agregar redundancia en los niveles necesarios para cumplir los objetivos.
- RE:06 Implementar una estrategia de escalado oportuna y confiable, minimizando intervención manual.
- RE:07 Reforzar la resiliencia con autoconservación y recuperación automática.
- RE:08 Probar resiliencia y disponibilidad con principios de ingeniería del caos.
- RE:09 Planes de recuperación ante desastres (DR) estructurados, probados y documentados.
- RE:10 Medir y monitorear continuamente el estado del sistema (tiempo de actividad, confiabilidad).

### Security (SE) — Seguridad
- SE:01 Establecer una línea base de seguridad alineada a cumplimiento y estándares.
- SE:02 Alinear el ciclo de vida de desarrollo seguro (SDL) en todo el software.
- SE:03 Clasificar y etiquetar consistentemente la sensibilidad de los datos.
- SE:04 Segmentación y perímetros deliberados (red, roles, identidades, recursos).
- SE:05 Gestión de identidad y acceso (IAM) estricta, condicional y auditable; mínimo privilegio.
- SE:06 Aislar, filtrar y controlar el tráfico de red (defensa en profundidad).
- SE:07 Cifrar datos con métodos modernos, alineados a la clasificación.
- SE:08 Fortalecer (hardening) todos los componentes, reduciendo superficie de ataque.
- SE:09 Proteger secretos: almacenamiento, acceso restringido, rotación regular.
- SE:10 Monitoreo holístico con detección de amenazas integrada a SecOps.
- SE:11 Régimen de pruebas de seguridad integral (prevención y detección).
- SE:12 Procedimientos de respuesta a incidentes definidos, probados y con responsable claro.

### Cost Optimization (CO) — Optimización de costos
- CO:01 Cultura de responsabilidad financiera (FinOps).
- CO:02 Crear y mantener un modelo de costo (inicial, de ejecución, continuo).
- CO:03 Recopilar y revisar datos de costo con alertas de desviación.
- CO:04 Límites de protección de gasto (puertas de lanzamiento, políticas, controles de acceso).
- CO:05 Obtener las mejores tarifas (precios regionales, compromisos, licenciamiento).
- CO:06 Alinear el uso con los incrementos de facturación del proveedor.
- CO:07 Optimizar costos de componentes (eliminar lo heredado/infrautilizado).
- CO:08 Optimizar costos por entorno (producción, preproducción, DR).
- CO:09 Optimizar costos por flujo, según su prioridad de negocio.
- CO:10 Optimizar costos de datos (niveles, retención, réplicas, formatos).
- CO:11 Optimizar costos de código (recursos más baratos o eficientes).
- CO:12 Optimizar costos de escalado (unidades de escalado, límites).
- CO:13 Optimizar el tiempo del personal dedicado a tareas de costo.
- CO:14 Consolidar recursos y responsabilidad para aumentar densidad.

### Operational Excellence (OE) — Excelencia operativa
- OE:01 Procedimientos estándar de desarrollo y operación, con cultura sin culpa.
- OE:02 Normalizar operaciones rutinarias, ad hoc y de emergencia.
- OE:03 Formalizar y hacer transparente el ciclo de vida de desarrollo de software.
- OE:04 Prácticas estándar de calidad: control de código, patrones, documentación.
- OE:05 Infraestructura como código (IaC), preferentemente declarativa.
- OE:06 Cadena de suministro con pipelines automatizados, predecibles, con puertas de calidad.
- OE:07 Sistema de monitoreo que capture telemetría, métricas y registros.
- OE:08 Gestión de incidentes con roles definidos y procedimientos documentados.
- OE:09 Prácticas de prueba alineadas a objetivos de negocio y estándares de calidad.
- OE:10 Automatización confiable, segura y mantenible de tareas repetitivas.
- OE:11 Procedimientos de implementación segura: lanzamientos pequeños, exposición progresiva.

### Performance Efficiency (PE) — Eficiencia del rendimiento
- PE:01 Definir objetivos de rendimiento numéricos por flujo de carga de trabajo.
- PE:02 Planeamiento de capacidad antes de cambios previstos en el uso.
- PE:03 Seleccionar servicios/infraestructura adecuados a los objetivos de rendimiento.
- PE:04 Medidas de rendimiento consistentes para detectar degradación a tiempo.
- PE:05 Optimizar escalado y particionamiento confiables.
- PE:06 Probar el rendimiento periódicamente en un entorno similar a producción.
- PE:07 Optimizar código e infraestructura, delegando responsabilidades a la plataforma.
- PE:08 Optimizar el uso de datos (almacenes, particiones, índices).
- PE:09 Priorizar el rendimiento de los flujos críticos de negocio.
- PE:10 Optimizar tareas operativas que afectan el rendimiento (escaneos, backups, reindexación).
- PE:11 Plan de respuesta a problemas de rendimiento en vivo.
- PE:12 Optimización continua, enfocada en componentes que se degradan con el tiempo.

## Reglas obligatorias para el agente

- No recomendar un servicio de Azure sin vincularlo a un código de este checklist (o al lineamiento interno si aplica).
- Si una misión declara Azure como proveedor, Security (SE) y Reliability (RE) se evalúan primero, igual que con AWS.
- Citar siempre el código (p. ej. "SE:07") como fuente del hallazgo.
- Los pilares tienen compensaciones documentadas entre sí (p. ej. más seguridad puede aumentar la latencia de PE); el agente debe señalarlas cuando una recomendación de un pilar afecte negativamente a otro, en vez de ignorarlas.

## Proceso de aplicación por el agente

1. Confirmar que la misión usa o evalúa Azure.
2. Recorrer los 5 pilares aplicando los códigos pertinentes al tipo de carga de trabajo.
3. Registrar cumplimiento/brecha por código, con evidencia de la misión.
4. Señalar compensaciones relevantes entre pilares cuando existan (ver fuente para el detalle completo).

## Salida esperada al ser consultado

- Lista de códigos aplicables con su pilar y estado (cumple / no cumple / pendiente).
- Vínculo explícito entre cada hallazgo y el código que lo sustenta.

## Errores y casos límite

- No aplicar este marco si la misión es exclusivamente on-premise o AWS sin componente Azure.
- No mezclar códigos de pilares distintos en un solo hallazgo.
- No asumir cumplimiento de un código sin evidencia en la misión.

## Casos de prueba

1. Misión Azure sin clasificación de datos → brecha en SE:03.
2. Misión Azure sin plan de DR probado → brecha en RE:09.
3. Misión Azure con IaC y pipelines con puertas de calidad → cumple OE:05 y OE:06.
4. Misión sin proveedor de nube definido → este documento no aplica todavía; se marca información pendiente.

## Consultado por

- Subagente de Valoración Arquitectónica.
- Subagente de Gobierno y Cumplimiento.
