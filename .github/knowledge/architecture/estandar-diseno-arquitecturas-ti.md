---
name: estandar-diseno-arquitecturas-ti
description: "Conocimiento institucional: Estándar para Diseño de Arquitecturas de TI. Consultar para evaluar factibilidad, aplicar los 6 pilares, racionalizar (5R), determinar resiliencia y completar el cuestionario oficial. Use when: diseño de arquitectura, racionalización, 5R, RTO, RPO, MTPD, pilares, cuestionario de arquitectura."
applyTo: "**/*formulario*arquitectura*|**/*tallaje*|**/*cuestionario*|**/*checklist*arquitectura*"
---

# Conocimiento: Estándar para Diseño de Arquitecturas de TI

## Propósito

Proveer al Agente Evaluador las reglas, pilares, cuestionario y criterios oficiales para evaluar la factibilidad y proponer una arquitectura de alto nivel, on-premise o en nube.

## Fuente

- Documento: "Diseño de arquitecturas de TI" (PDF institucional, proporcionado por Arquitectura TI).
- Esta ficha ya incorpora el mapeo de campos reales del Formulario para Diseño de Arquitectura y del Modelo de Tallaje (estructura de columnas, códigos `ARQ-0xx`, matriz de talla) obtenido al analizar ejemplos reales de misiones durante la construcción del agente.
- Última revisión registrada en este repo: la entregada por la persona responsable el 2026-10-07. Verificar vigencia con Arquitectura TI antes de usarla en una evaluación crítica.

## Cuándo se consulta

- Al iniciar la valoración arquitectónica de una misión.
- Al calcular la talla de una iniciativa.
- Al decidir entre racionalizar, reconstruir o reemplazar una solución existente.
- Al definir el patrón de resiliencia.
- Al completar o validar el Formulario para Diseño de Arquitectura.

## Reglas obligatorias

### Objetivo del estándar
La arquitectura propuesta debe ser resiliente, disponible y recuperable; cumplir estándares de seguridad; permitir control financiero; y ser compatible con desarrollo y operaciones responsables. Debe apegarse al [lineamiento-uso-nube-publica.md](lineamiento-uso-nube-publica.md) y a los demás lineamientos de la sección 10 del documento fuente (seguridad, continuidad, monitoreo, comunicaciones, DevOps).

### Los 6 pilares (obligatorio evaluar todos)
1. **Excelencia operativa:** propietarios de negocio/tecnología definidos, aprobación de Gerencia/Dirección, registro en el proceso de misiones, línea base de monitoreo, prácticas de CI/CD establecidas, revisiones de preparación operativa.
2. **Seguridad:** Zero Trust desde el diseño, MFA obligatorio, mínimo privilegio y separación de funciones, cifrado TLS 1.2+/AES-256, clasificación de datos, SIEM, pruebas de penetración, cumplimiento normativo, WAF/segmentación de red, gestión de parches.
3. **Fiabilidad:** suscripciones/cuentas por ambiente, grupos de recursos por misión, diseño de red segmentada, RPO/RTO/WRT/MTPD definidos, redundancia (zonas/regiones), recuperación ante desastres con simulacros, elasticidad, flexibilidad, documentación.
4. **Rendimiento:** elección de recursos según tipo de carga (web, datos, IA, infraestructura, seguridad), optimización continua, uso de tecnologías avanzadas (IA, serverless, contenedores), optimización de almacenamiento, gestión de bases de datos, monitoreo y análisis con SIEM/observabilidad.
5. **Optimización de costos:** arquitecturas elásticas, serverless para cargas intermitentes, servicios gestionados, apagado automático en no-producción, monitoreo de costos desde el diseño (tags, presupuestos, alertas).
6. **Sostenibilidad:** reducir recursos ociosos, consolidar cargas, priorizar proveedores con compromisos de energía renovable, elegir regiones de menor huella de carbono cuando sea viable.

### Regla de oro RTO/RPO/MTPD
`RTO + WRT ≤ MTPD`. Si no se cumple, es un hallazgo bloqueante de fiabilidad.

### Racionalización (5R) — solo aplica a soluciones existentes
| R | Estrategia | Cuándo |
|---|---|---|
| R-01 | Reubicar (lift & shift) | Mover sin cambios significativos |
| R-02 | Refactorizar | Modificación parcial hacia PaaS |
| R-03 | Rearquitecturar | Incompatible con nube o no nativa |
| R-04 | Reconstruir | Obsoleta, no justifica inversión, se reescribe |
| R-05 | Reemplazar | Existe un SaaS equivalente |

Si la solución es **nueva**, no aplica racionalización: se recomienda SaaS primero, PaaS segundo, IaaS tercero.

### Patrones de resiliencia
**On-premise:** OP1 (redundancia de hardware, 1 servidor), OP2 (alta disponibilidad, clúster en 1 sitio), OP3 (DR activo-pasivo, 2 sitios), OP4 (activo-activo, 2 sitios primarios).

**Nube:** PN1 (1 zona, 1 región — bajo impacto), PN2 (multi-zona, 1 región), PN3 (distribución geográfica, 3 zonas/2 regiones), PN4 (multi-región activo-pasivo), PN5 (multi-región activo-activo, RPO≈0).

El patrón se elige según criticidad, SLA, RTO y RPO — nunca por defecto.

### Resiliencia de los datos
Fuentes confiables con validación de integridad; cifrado en reposo y tránsito (AES-256, TLS 1.2+); acceso por mínimo privilegio; trazabilidad de consultas/modificaciones/transferencias; retención diferenciada por criticidad; archivo seguro de datos inactivos; eliminación segura al expirar la retención; clasificación obligatoria por sensibilidad (pública, interna, confidencial, crítica).

### Modelo de tallaje

Esta sección ya contiene todo lo necesario para calcular una talla — no hace falta abrir ningún archivo fuera de `.github/` para completarla.

- 7 dimensiones: Complejidad, Integraciones, Criticidad, Seguridad, Rendimiento, Resiliencia, Esfuerzo. Cada criterio se puntúa Alto=3, Medio=2, Bajo=1.
- **Regla inteligente** (anula el score): si la iniciativa es crítica de negocio, regulatoria, maneja dinero/clientes, requiere multi-región, o requiere proveedor aún en evaluación → talla mínima **L o XL**, sin importar el puntaje.
- Matriz de talla por rango de puntaje:

| Rango | Talla | Duración | Gobierno | Validación |
|---|---|---|---|---|
| 53–63 | XL | 6+ meses | Mínimo | Básica (equipo técnico) |
| 43–52 | L | 3–6 meses | Moderado | Checklist (arquitecto asignado) |
| 33–42 | M | 1–3 meses | Alto | Evaluación completa (arquitectos y negocio) |
| 23–32 | S | 1–4 semanas | Estricto | Comité (arquitectos, negocio, VB ejecutivo) |

- **No puntuar información ausente como "Bajo"**: si falta el dato, el criterio queda pendiente, no se asume el valor más favorable. (Confirmado contra un caso real evaluado durante la construcción del agente, que quedó marcado "Preliminar - requiere información" precisamente por esto.)

### Formulario para Diseño de Arquitectura — estructura real
Hojas: Inicio, Evaluación Arquitectónica, Checklist Automático, Dictamen, Plan de Acción, Parámetros.

- **Evaluación Arquitectónica:** preguntas con ID (`ARQ-001`…), Sección (Negocio, Requerimientos, Excelencia operativa, Seguridad, Datos, Fiabilidad, Proveedor SaaS, Costos, Sostenibilidad, Racionalización, Entregables), Dominio, Pregunta, Guía/evidencia esperada, Aplica a (Todos/Nube-SaaS/SaaS/Existente), ¿Aplica?, Respuesta (Cumple / Cumple parcialmente / No cumple / Pendiente), Evidencia, Observaciones, Responsable, Severidad (**Bloqueante**/Alta/Media/Baja), Puntaje.
- **Dictamen automático:**
  - No aprobado: existe bloqueante incumplido, o cumplimiento < 75%.
  - Requiere información: faltan respuestas aplicables.
  - Aprobado con condiciones: 75%–89.9%, sin bloqueantes incumplidos.
  - Aprobado: ≥ 90%, sin bloqueantes incumplidos.
- **Plan de Acción:** ID, Hallazgo/brecha, Severidad, Acción, Responsable, Fecha compromiso, Estado, Evidencia de cierre.
- Preguntas bloqueantes típicas: cumplimiento regulatorio (ARQ-004), MFA/SSO (ARQ-014), RBAC (ARQ-015), cifrado (ARQ-016), clasificación de datos (ARQ-019), residencia de datos (ARQ-020), portabilidad SaaS (ARQ-022), uso de datos por el proveedor (ARQ-023), RTO/RPO/MTPD de negocio (ARQ-024 a 026), certificaciones del proveedor SaaS (ARQ-031), SLA del proveedor (ARQ-032), continuidad del proveedor (ARQ-033), notificación de incidentes (ARQ-034), estrategia de salida (ARQ-035), diagrama de arquitectura (ARQ-046), evaluación de riesgos (ARQ-047).

### Cuestionario genérico (plantilla vacía del Formulario)
Mismas 8 secciones que el documento fuente: Preguntas Generales, Análisis de Racionalización (5R), Excelencia Operativa, Seguridad, Fiabilidad, Rendimiento, Optimización de Costos, Desarrollo y Ciclo de Vida, Evaluación de Resiliencia On-Premise/Nube. Encabezado común: Dirección, Gerencia, Proyecto, CentroCostos, Responsable, Ambiente, Servicio, IDCargoSAP, Nombre Proyecto, Talla de la arquitectura, Fecha. Incluye además una Lista de Chequeo paralela con las mismas categorías, respuesta Sí/No/N/A.

### Entregables oficiales (sección 11 del documento fuente)
A. Diagrama de Arquitectura de Alto Nivel — formulario + diagrama, validado iterativamente con arquitectos.
B. Entregables de TI y Evaluación de Riesgos — sesión colaborativa PO + arquitectos TI + ciberseguridad + administradores.
C. Hoja de Adquisición — componentes, costos (incluso costo cero), fuentes (calculadoras de proveedor, cotizaciones).
D. Hoja de Entrega con Conclusiones y Recomendaciones — consolida diseño, adquisición, ETI/riesgos, checklist de seguridad, RTO/RPO/criticidad, acuerdos, siguientes pasos.

## Proceso de aplicación por el agente

1. Determinar si la solución es nueva o existente (define si aplica racionalización 5R).
2. Calcular o verificar la talla con el modelo de 7 dimensiones y la regla inteligente.
3. Evaluar los 6 pilares usando el cuestionario/formulario como checklist de evidencia.
4. Verificar la regla RTO + WRT ≤ MTPD.
5. Determinar el patrón de resiliencia aplicable según criticidad.
6. Calcular el dictamen (aprobado / con condiciones / requiere información / no aprobado) según severidad y porcentaje de cumplimiento.
7. Registrar cada brecha en un plan de acción con responsable y fecha.

## Salida esperada al ser consultado

- Talla calculada (o pendiente) con las dimensiones que la sustentan.
- Lista de preguntas del formulario aplicables, con su severidad.
- Dictamen y justificación.
- Patrón de resiliencia recomendado con su justificación.
- Recomendación de racionalización (si aplica) con su motivación.

## Errores y casos límite

- No calcular talla con información faltante asumida como "Bajo".
- No recomendar un patrón de resiliencia sin conocer RTO/RPO/criticidad.
- No aplicar racionalización 5R a una solución nueva.
- No emitir dictamen "Aprobado" si hay al menos un bloqueante incumplido, sin importar el porcentaje.

## Casos de prueba

1. Misión nueva, sin solución previa → sin racionalización; se recomienda SaaS/PaaS/IaaS en ese orden.
2. Misión con dato financiero sensible y sin MFA documentado → bloqueante de seguridad, dictamen no aprobado.
3. Misión con 88% de cumplimiento y sin bloqueantes → aprobado con condiciones.
4. Misión con RTO 10 min pero MTPD 4h y WRT 1h (10min+1h ≤ 4h) → cumple la regla.
5. Misión crítica de negocio con puntaje de tallaje de 25 (rango S) pero que maneja dinero → la regla inteligente la sube a L o XL.
6. Modelo de tallaje con dimensión "Rendimiento" sin datos → queda "Pendiente", no se asume Bajo.

## Consultado por

- Subagente de Análisis de Iniciativa (tallaje, formulario).
- Subagente de Valoración Arquitectónica (pilares, racionalización, resiliencia).
- Subagente de Generación de Entregables (los 4 entregables oficiales).
