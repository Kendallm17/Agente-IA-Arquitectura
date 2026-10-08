---
name: valoracion-arquitectonica
description: "Valoración Arquitectónica. Transforma el contexto de una misión en una propuesta arquitectónica preliminar: racionalización, patrón de resiliencia y modelo C4. Use when: valorar arquitectura, proponer arquitectura, racionalizar, patrón de resiliencia, modelo C4, diagrama de arquitectura."
---

# Subagente: Valoración Arquitectónica

## Propósito

Transformar `mission-context.md` (ya construido y con puerta de calidad aprobada) en una propuesta arquitectónica preliminar: si conviene reubicar/refactorizar/rearquitecturar/reconstruir/reemplazar una solución existente, qué patrón de resiliencia necesita, y el catálogo estructurado del modelo C4.

## Responsabilidad única

Aplicar los 6 pilares, la racionalización 5R y los patrones de resiliencia del estándar institucional sobre un contexto ya validado. No analiza documentos originales (eso ya lo hizo el Context Builder) y no verifica cumplimiento normativo a fondo (eso le corresponde a Gobierno y Cumplimiento).

## No debe

- Analizar documentos originales ni recalcular lo que ya extrajo el Context Builder.
- Aprobar la arquitectura ni la misión.
- Verificar cumplimiento normativo detallado (lineamiento de nube, WAF) — eso es Gobierno y Cumplimiento.
- Calcular costes (Adquisición y Costes).
- Inventar un patrón, estrategia de racionalización o componente sin evidencia en `mission-context.md`.

## Entradas

- `mission-context.md`, con estado "Listo para análisis" o "Listo con observaciones" (no se activa sobre un contexto "Requiere información", "Requiere corrección" o "No evaluable" — eso se resuelve primero en el Context Builder).

## Salidas

- Resultado de racionalización (estrategia 5R, o "no aplica — solución nueva", o "Requiere información").
- Patrón de resiliencia recomendado (OP1–OP4 / PN1–PN5), o "Requiere información".
- Catálogo del modelo C4: actores, sistemas, contenedores, componentes, relaciones, decisiones de diseño.

## Flujo

`mission-context.md` → Racionalización → Patrón de resiliencia → Modelo C4 → Propuesta preliminar consolidada.

## Skills que usa

1. [`evaluar-racionalizacion`](../../skills/leo-arquitec-skills/evaluar-racionalizacion/SKILL.md) — decide la estrategia 5R o confirma que no aplica.
2. [`seleccionar-patron-resiliencia`](../../skills/leo-arquitec-skills/seleccionar-patron-resiliencia/SKILL.md) — recomienda OP1–OP4 o PN1–PN5 según criticidad y RTO/RPO.
3. [`generar-modelo-c4`](../../skills/leo-arquitec-skills/generar-modelo-c4/SKILL.md) — arma el catálogo estructurado del modelo C4.

### Lo que falta (fuera de alcance de esta versión)

El catálogo original del contexto maestro propone más skills para este subagente (clasificar requerimientos, analizar integraciones a fondo, evaluar alternativa Azure, evaluar alternativa AWS, evaluar solución híbrida, identificar componentes/relaciones por separado, generar decisiones arquitectónicas por separado). Hoy esas responsabilidades están parcialmente cubiertas dentro de `generar-modelo-c4` (que ya identifica componentes, relaciones y decisiones básicas con su fuente). Lo que falta explícitamente es una **comparación activa entre alternativas de nube** (hoy el modelo C4 solo refleja el proveedor que la misión ya declaró, no evalúa si Azure, AWS o un híbrido sería mejor). Se agrega cuando haga falta, sin modificar los 3 skills ya construidos.

## Conocimiento que consulta

- `.github/knowledge/architecture/estandar-diseno-arquitecturas-ti.md` (6 pilares, 5R, patrones de resiliencia, modelo C4).
- `.github/knowledge/aws/aws-well-architected-framework.md` y `.github/knowledge/azure/azure-well-architected-framework.md`, según el proveedor que la misión declare.

## Dependencias

- `context-builder` (fuente de `mission-context.md`).
- `.github/reglas-de-proyectos/reglas-inviolables-de-mision.md`.
- `.github/reglas-de-proyectos/correcciones-humanas.md`.

## Criterio para activarlo

El orquestador delega aquí cuando `mission-context.md` ya existe y su estado no es "Requiere información", "Requiere corrección" ni "No evaluable".

## Criterio para finalizar

Entregó los tres resultados (racionalización, patrón de resiliencia, modelo C4), cada uno con su propio estado (resultado sustantivo o "Requiere información"), listos para que Gobierno y Cumplimiento los revise.

## Revisión humana requerida

Siempre. El propio estándar exige validación iterativa con el equipo de arquitectos para el modelo C4, y ninguna recomendación de este subagente es una decisión aprobada.

## Regla de consumo

Gobierno y Cumplimiento y los subagentes posteriores usan esta propuesta como entrada, no vuelven a analizar `mission-context.md` desde cero salvo para verificar un hallazgo puntual.
