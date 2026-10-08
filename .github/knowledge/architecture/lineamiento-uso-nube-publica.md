---
name: lineamiento-uso-nube-publica
description: "Conocimiento institucional: Lineamiento de Uso de Nube Pública. Consultar cuando se valore seguridad, resiliencia, DevOps, operación, FinOps o contratación SaaS de una solución en nube pública. Use when: nube pública, Azure, AWS, CCoE, FinOps, landing zone, etiquetado, SaaS."
applyTo: "**/*nube*|**/*cloud*|**/*azure*|**/*aws*|**/*saas*"
---

# Conocimiento: Lineamiento de Uso de Nube Pública

## Propósito

Proveer al Agente Evaluador las reglas institucionales obligatorias para analizar, valorar y recomendar arquitecturas que usen nube pública (IaaS, PaaS, SaaS), de forma que ninguna recomendación contradiga este lineamiento.

## Fuente

- Documento: "Uso de nube pública" (PDF institucional, proporcionado por Arquitectura TI).
- Tipo: lineamiento corporativo obligatorio.
- Última revisión registrada en este repo: la entregada por la persona responsable el 2026-10-07. Verificar vigencia con Arquitectura TI antes de usarla en una evaluación crítica.

## Cuándo se consulta

- Al evaluar si una solución puede desplegarse en nube pública.
- Al definir componentes, servicios de nube, resiliencia, seguridad o costos de una propuesta.
- Al revisar el cumplimiento de gobierno de nube de una misión.

## Reglas obligatorias

### Alcance y obligatoriedad
- Aplica a toda aplicación financiera, almacenamiento/procesamiento de datos, IaaS/PaaS/SaaS, IA/automatización en nube pública e integraciones entre on-premise y nube.
- Es obligatorio para usuarios internos, proveedores, equipos de TI y áreas de cumplimiento/auditoría.
- Solo se usan servicios de nube pública **previamente aprobados** por Tecnología y Seguridad de la Información, mediante el proceso de Misiones, Dirección Tecnológica y Compras estratégicas.
- Todo despliegue debe estar asociado a un requerimiento formalizado en una misión y aprobado; se exige evaluación de al menos 3 opciones salvo excepción ya aprobada.

### Gobierno
- Existe el Equipo Ejecutivo de Gobierno de Nube (alineamiento estratégico, 3 sesiones/año) y el CCoE (Cloud Center of Excellence), responsable de lineamientos, arquitectura, automatización, seguridad, educación, optimización de valor y KPIs.
- Cada país designa Champions de nube para Infraestructura, DevOps, FinOps, Ciberseguridad y CloudOps.

### Pilares de arquitectura (coinciden con el Estándar de Diseño de Arquitecturas de TI)
Excelencia operativa, Seguridad, Fiabilidad, Rendimiento, Optimización de costos, Sostenibilidad. Ver `estandar-diseno-arquitecturas-ti.md` para el detalle operativo de cada pilar.

### Seguridad (obligatorio, trátese como bloqueante si falta)
- Autenticación multifactor (MFA) para todo acceso administrativo o privilegiado; RBAC revisado periódicamente.
- Cifrado de datos en tránsito y en reposo con TLS 1.2+ y AES-256; datos sensibles requieren llaves custodiadas.
- Modelo de responsabilidad compartida: el proveedor asegura la nube, BAC asegura lo que hay en la nube (configuración, accesos, datos, respaldos).
- Servicios expuestos a Internet requieren WAF y protección DDoS (capa 7); no se permite comunicación directa Internet → red interna.
- Validación anual de controles de seguridad para servicios IaaS/PaaS/SaaS registrados en CMDB, con evidencia (SOC 2 Type II, ISO/IEC 27001, NIST CSF o PCI DSS para SaaS).
- Evaluación de riesgos previa obligatoria: no se despliega en nube sin aseguramiento y validación de Ciberseguridad.
- Registros de auditoría con retención mínima de 5 años o lo que exija la regulación.

### Arquitectura de integración, telecomunicaciones e infraestructura
- Modelo de integración preferente: asíncrono, con identificador de transacción trazable.
- Toda API publicada requiere control de acceso y protección contra uso no autorizado/DDoS.
- Interconexión nube–premisas: canales redundantes, firewall de siguiente generación, hub de concentración de conexiones regional.
- Landing Zone obligatoria como punto de partida: IAM, seguridad/cumplimiento, monitoreo, IaC, gobernanza de costos.
- Etiquetado obligatorio sin excepción en todo recurso de nube (ver tabla de etiquetas FinOps). Nomenclatura obligatoria.

### DevOps
- CI/CD como código; análisis estático y validaciones de seguridad antes de integrar; mínimo 3 aprobaciones en Pull Request (desarrollador, supervisor técnico, subgerente).
- Git como estándar de control de versiones. Scrum como metodología de gestión de tareas.
- Shift-left security (DevSecOps): escaneo de dependencias y contenedores, políticas de cumplimiento automatizadas.

### Operación y continuidad
- Inventario de componentes de nube actualizado al menos 2 veces al año en CMDB (campos: Nombre, País, Administrador por, Estado de instalación, Tipo de aplicación, Tipo de instalación, Descripción, Criticidad de negocio, Criticidad de TI, Proveedor, Rol del área, Plataforma).
- BIA obligatorio para procesos críticos en nube: define RTO, RPO, impacto financiero/operativo/reputacional/legal.
- Respaldos: redundantes, automatizados, cifrados, verificables; frecuencia según RPO; pruebas de restauración semestrales (críticos) o anuales (resto).
- Ambientes de desarrollo/pruebas en suscripción separada de producción.

### FinOps — etiquetas obligatorias (sin excepción, sin tildes)
`Ambiente`, `CentroCostos`, `Direccion`, `Gerencia`, `IDCargoSAP`, `Proyecto`, `Responsable`, `Servicio`. `IDCargoSAP` es indispensable para asignar el costo. Los costos se asignan al área que los provoca (Patrocinador), nunca a TI por defecto.

### Gestión de SaaS (requisitos mínimos de contratación)
- SLA con disponibilidad mínima, tiempos de respuesta/resolución por severidad, penalizaciones, métricas monitoreadas.
- Certificaciones de seguridad reconocidas (SOC 2 Type 2, ISO 27001/27017/27018, CSA).
- Cláusulas de decomiso y destrucción de datos post-retiro, con certificación formal.
- Escalabilidad/licenciamiento proporcional al uso, sin penalización por crecimiento.
- Cumplimiento regulatorio local (p. ej. Ley de Protección de Datos Personales, Conassif 5-24 según país).

## Proceso de aplicación por el agente

1. Identificar si la misión usa o propone nube pública.
2. Verificar que el servicio esté dentro del alcance autorizado (o marcarlo como pendiente de aprobación).
3. Aplicar las reglas de seguridad, continuidad, DevOps, operación y FinOps como criterios de valoración.
4. Señalar como hallazgo bloqueante cualquier incumplimiento de una regla marcada como obligatoria aquí.
5. Citar la sección específica de este documento como fuente de cada hallazgo (trazabilidad).

## Salida esperada al ser consultado

- Lista de reglas aplicables a la misión evaluada, con su cumplimiento (cumple / no cumple / pendiente de validar).
- Referencia a la sección del lineamiento que sustenta cada hallazgo.
- Indicación de si el incumplimiento es bloqueante.

## Errores y casos límite

- No inferir aprobación de nube si no hay evidencia del proceso de Misiones.
- No asumir que SaaS está fuera de FinOps de costo variable: SaaS sí se gestiona en visibilidad financiera, solo queda fuera del control de costo por decisiones de uso.
- Si la misión no específica proveedor de nube, no asumir Azure o AWS: marcar como información faltante.

## Casos de prueba

1. Misión que expone un servicio a Internet sin WAF/DDoS → hallazgo bloqueante de seguridad.
2. Misión sin etiqueta `IDCargoSAP` → hallazgo bloqueante de FinOps (no se puede asignar costo).
3. Misión que usa SaaS sin certificación de seguridad declarada → hallazgo bloqueante.
4. Misión con RTO/RPO definidos y patrón de resiliencia coherente → sin hallazgo.
5. Misión on-premise sin componente de nube → este documento no aplica; el agente lo indica y no fuerza reglas de nube.

## Consultado por

- Subagente de Valoración Arquitectónica.
- Subagente de Gobierno y Cumplimiento.
- Subagente de Evaluación de Riesgos.
