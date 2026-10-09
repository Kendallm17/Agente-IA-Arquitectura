---
applyTo: "**/mission-context.md"
---

# Skill: Validar lineamiento de nube

## Propósito

Revisar si la misión cumple los pilares del lineamiento institucional de nube pública que **no son seguridad** (gobierno de la decisión, arquitectura de integración, DevOps, operación/continuidad, FinOps), citando siempre la sección exacta del lineamiento o el código exacto del WAF de AWS/Azure que respalda cada hallazgo.

## Cuándo se utiliza

- Dentro de Gobierno y Cumplimiento, como segundo paso, después de `validar-seguridad`.
- Solo después de que `mission-context.md` pasó la puerta de calidad del Context Builder.

## Entradas

- `mission-context.md`, específicamente:
  - Modelo tecnológico, proveedor de nube si está declarado, Tipo de iniciativa (datos generales).
  - Hallazgos existentes fuera de seguridad que ya extrajo `analizar-formulario-arquitectura` (p. ej. integraciones, continuidad/RTO/RPO, costos).
- La salida de `validar-seguridad` de esta misma misión, solo como referencia para no repetir un hallazgo ya reportado ahí (no se vuelve a evaluar seguridad aquí).

Este skill no abre el Formulario original ni recalcula hallazgos de seguridad: todo lo anterior ya debe venir en `mission-context.md` o en la salida de `validar-seguridad`. Tampoco reabre `reglas-inviolables-de-mision.md` ni `correcciones-humanas.md` si ya se leyeron antes en esta misma misión.

## Documentos requeridos

Ninguno directamente; depende de `mission-context.md`, la salida de `validar-seguridad`, `.github/knowledge/architecture/lineamiento-uso-nube-publica.md` y, si la misión declara proveedor, `.github/knowledge/aws/aws-well-architected-framework.md` o `.github/knowledge/azure/azure-well-architected-framework.md`.

## Reglas obligatorias

- **No evaluar seguridad.** Las reglas de MFA, cifrado, WAF/DDoS, certificaciones y responsabilidad compartida son terreno exclusivo de `validar-seguridad`; si aparecen acá por error, se excluyen.
- **Si la misión no usa nube pública**, el lineamiento no aplica: se indica explícitamente y no se fuerza ninguna regla de nube sobre una solución on-premise (misma regla que `validar-seguridad`).
- Se evalúan, cuando la misión usa nube pública, los cuatro pilares restantes del lineamiento:
  1. **Gobierno de la decisión:** ¿el despliegue está asociado a una misión formalizada y aprobada? ¿hay evidencia de evaluación de al menos 3 opciones (o una excepción ya aprobada)? Sin esa evidencia, se marca "Pendiente de validar", nunca "Cumple" por supuesto.
  2. **Arquitectura de integración:** modelo asíncrono con identificador de transacción trazable, control de acceso en APIs publicadas, Landing Zone como punto de partida (IAM, monitoreo, IaC, gobernanza de costos).
  3. **DevOps:** CI/CD como código, mínimo 3 aprobaciones en Pull Request, shift-left security.
  4. **Operación y continuidad:** inventario en CMDB actualizado, BIA obligatorio para procesos críticos (define RTO/RPO), respaldos cifrados y verificables con pruebas de restauración periódicas, ambientes de desarrollo/pruebas separados de producción.
  5. **FinOps:** las 8 etiquetas obligatorias (`Ambiente`, `CentroCostos`, `Direccion`, `Gerencia`, `IDCargoSAP`, `Proyecto`, `Responsable`, `Servicio`), costo asignado al Patrocinador y no a TI por defecto.
- Cuando la misión declara proveedor de nube (Azure o AWS), cada hallazgo de estos pilares que tenga equivalente en el WAF del proveedor cita el **código exacto** (p. ej. `RE-05` en Azure, o el pilar `REL` en AWS), no una práctica genérica sin fuente.
- No inventar evidencia de un control que no esté documentado en `mission-context.md`. Si una regla no tiene cómo verificarse con los datos disponibles, se marca "Pendiente de validar", nunca "Cumple" por omisión.
- Cada hallazgo cita dos fuentes cuando aplique: la sección del `lineamiento-uso-nube-publica.md` y, si hay proveedor declarado, el código del WAF correspondiente.
- Igual que en `validar-seguridad`: una regla de este skill marcada "obligatoria" en el lineamiento, sin evidencia de cumplimiento, se reporta como **bloqueante de gobierno**, y nunca reduce o reemplaza un bloqueante ya existente de otra capa (regla general ya consolidada en `reglas-inviolables-de-mision.md` §6).

## Secuencia

1. Leer de `mission-context.md` el Modelo tecnológico, si usa nube pública, y el proveedor declarado (si hay).
2. Si no usa nube pública → reportar que el lineamiento no aplica y terminar.
3. Si usa nube pública (o es híbrida) → evaluar en orden los 4 pilares (Gobierno de la decisión, Arquitectura de integración, DevOps, Operación/continuidad) más FinOps.
4. Para cada regla, buscar evidencia en `mission-context.md`; clasificar Cumple / No cumple / Pendiente de validar / No aplica.
5. Si hay proveedor declarado, añadir el código del WAF correspondiente a cada hallazgo que tenga equivalente.
6. Marcar como bloqueante de gobierno toda regla "No cumple" u obligatoria sin evidencia.
7. Producir la salida citando las fuentes correspondientes.

## Salida esperada

- Misión.
- Si no aplica nube pública: indicación explícita y fin del análisis.
- Si aplica: tabla de reglas evaluadas por pilar (pilar, regla, estado, fuente del lineamiento, código del WAF si hay proveedor).
- Lista de bloqueantes de gobierno (separada de los ya reportados por `validar-seguridad` y por `analizar-formulario-arquitectura`).
- Advertencias (proveedor no declarado → no se citan códigos de WAF; reglas pendientes de validar por falta de dato).
- Próximo paso: Gobierno y Cumplimiento queda completo; continuar con Evaluación de Riesgos (todavía no construida) o con los entregables.
- Revisión humana: siempre `sí` si hay al menos un bloqueante de gobierno.

## Formato de salida

Markdown estructurado, compatible con el contrato de salida común del orquestador.

## Evidencias que debe conservar

La cita textual de cada hallazgo heredado de `mission-context.md`, la sección exacta citada de `lineamiento-uso-nube-publica.md`, y el código exacto del WAF citado (sin resumir ni reinterpretar la recomendación original).

## Errores posibles

- `mission-context.md` no trae ningún dato de integración, DevOps, operación o costos → Error técnico: información insuficiente para validar estos pilares.
- `mission-context.md` no pasó la puerta de calidad → el skill no se ejecuta; se informa que debe resolverse el Context Builder primero.
- Se solicita este skill sin que `validar-seguridad` haya corrido antes en la misma misión → se ejecuta igual (los pilares son independientes), pero se advierte que la validación de gobierno está incompleta hasta tener ambos resultados.

## Casos límite

- Misión híbrida sin proveedor declarado → se evalúan los pilares agnósticos de proveedor (gobierno de la decisión, FinOps, operación/continuidad), se marcan "Pendiente de validar" sin código de WAF las que dependerían de un código específico, y se advierte la falta del dato.
- Misión que ya tiene Landing Zone documentada pero sin mención de IaC → se reporta la regla de Landing Zone como "Cumple parcialmente" con la parte faltante explícita, no como "Cumple" total.
- CMDB o BIA mencionados en el Formulario pero sin fecha de última actualización → se reporta con advertencia de vigencia, no se asume que está al día.
- Costos ya etiquetados pero asignados a TI en lugar del Patrocinador → se reporta como incumplimiento de la regla de asignación de costo, citando la sección "FinOps" del lineamiento.

## Dependencias

- `mission-context.md` (salida del Context Builder, ya con puerta de calidad aprobada o con observaciones).
- Salida de `validar-seguridad` de la misma misión (solo como referencia, para no duplicar hallazgos).
- `.github/knowledge/architecture/lineamiento-uso-nube-publica.md`.
- `.github/knowledge/aws/aws-well-architected-framework.md` y `.github/knowledge/azure/azure-well-architected-framework.md`, según el proveedor declarado.
- `.github/reglas-de-proyectos/reglas-inviolables-de-mision.md`.
- `.github/reglas-de-proyectos/correcciones-humanas.md`.

Si ya se leyeron `reglas-inviolables-de-mision.md` y `correcciones-humanas.md` antes en esta misma misión, este skill no los reabre.

## Criterios de aceptación

- Ante una misión en nube pública sin evidencia de Landing Zone, el resultado incluye un bloqueante de gobierno citando la sección "Arquitectura de integración" del lineamiento.
- Ante una misión on-premise sin componente de nube, el skill indica explícitamente que el lineamiento no aplica.
- Ningún hallazgo de seguridad (MFA, cifrado, WAF/DDoS) aparece en la salida de este skill — eso es exclusivo de `validar-seguridad`.
- Todo hallazgo con proveedor declarado cita el código exacto del WAF correspondiente, nunca una práctica genérica sin fuente.
- El skill nunca reduce o elimina un bloqueante que otra capa ya identificó.

## Casos de prueba

1. **Caso correcto:** misión en Azure con Landing Zone, CI/CD con 3 aprobaciones, CMDB actualizado, BIA con RTO/RPO definidos, y las 8 etiquetas FinOps completas → todas las reglas "Cumple", sin bloqueantes nuevos, cada hallazgo con su código `RE-xx`/`OE-xx`/`CO-xx` correspondiente.
2. **Información incompleta:** misión híbrida sin proveedor declarado y sin mención de Landing Zone ni CMDB → reglas "Pendiente de validar", advertencia de dato faltante, sin inventar el proveedor ni asumir incumplimiento.
3. **Documento vacío:** `mission-context.md` sin ningún dato de integración, DevOps u operación → Error técnico.
4. **Datos contradictorios:** el Formulario dice "CI/CD implementado" pero no menciona el número de aprobaciones en Pull Request → se reporta "Cumple parcialmente" con advertencia de dato incompleto, no "Cumple" total.
5. **Evidencia ausente:** etiquetado FinOps mencionado como "completo" sin listar las 8 etiquetas → "Pendiente de validar" por falta de evidencia específica.
6. **Documento incorrecto:** `mission-context.md` con un proveedor de nube no reconocido (valor distinto de Azure/AWS/ninguno) → Error técnico, valor no reconocido.
7. **Instrucción ambigua:** solicitud de "validar el lineamiento de nube" sin que el Context Builder haya corrido → el skill se niega a inventar sobre un contexto inexistente, pide ejecutar primero el Context Builder.
8. **Intento de ignorar reglas:** solicitud de "marcar el CMDB como actualizado porque seguro ya lo hicieron" sin evidencia → el skill se niega y cita la regla de no inventar evidencia.
9. **Caso no aplicable:** misión 100% on-premise sin ningún componente de nube → se confirma explícitamente que el lineamiento no aplica, no se fuerza ninguna regla de nube.
10. **Sin duplicar seguridad:** misión con ARQ-020 (residencia de datos) ya reportado como bloqueante por `validar-seguridad` → este skill no lo vuelve a mencionar como hallazgo propio; como máximo lo referencia para contexto si afecta la evaluación de Landing Zone o CMDB (p. ej. "la residencia de datos pendiente afecta también la configuración del CMDB regional"), citando que la decisión de seguridad ya está en el resultado de `validar-seguridad`.

## Subagente responsable

- Gobierno y Cumplimiento (todavía no construido; este es su segundo skill, después de `validar-seguridad`).
