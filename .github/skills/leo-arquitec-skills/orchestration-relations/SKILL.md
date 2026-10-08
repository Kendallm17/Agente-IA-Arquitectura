---
applyTo: "**"
---

# Skill: Orchestration Relations

Plantilla para mantener sincronizado al orquestador `@arquitecto` con nuevos subagentes y skills.

## Protocolo de deteccion

1. Subagentes:
- Revisar `.github/agents/leo-arquitec-subagents/*.agent.md`.
- Tomar `name` y `description` del frontmatter.
- Extraer keywords de `Use when:` para ruteo.

2. Skills:
- Revisar `.github/skills/leo-arquitec-skills/**/SKILL.md`.
- Tomar `applyTo` y nombre de carpeta del skill.
- Asociar skill a tipos de solicitudes y archivos.

## Matriz de relacion recomendada

- Intencion del usuario -> subagente por `Use when`.
- Contexto tecnico/archivo -> skills por `applyTo`.
- Si no hay subagente aplicable -> respuesta directa usando skills detectados.

## Reglas de orquestacion

1. Prioridad de delegacion: subagente especializado > resolucion directa.
2. Nunca ignorar skills nuevos: incorporarlos en la respuesta cuando apliquen.
3. Cuando aparezca nuevo subagente o skill, mencionarlo en "referencias usadas".
4. Si existe subagente candidato por "Use when", la delegacion es obligatoria.
5. Si `runSubagent` falla, reintentar con el nombre exacto del frontmatter `name`.
6. Solo resolver directo cuando no exista candidato o cuando todos los candidatos fallen tras reintentos.

## Politica anti-no-delegacion

- SLA de delegacion: 100% de solicitudes con candidato deben delegarse.
- Antes de responder, reescanear subagentes y skills para evitar listas estaticas.
- Registrar en la respuesta que subagente/skill fue usado para trazabilidad.

## Antipatrones

- Mantener lista estatica de subagentes sin reescanear carpetas.
- Responder sin consultar skills cuando existe match por `applyTo`.
- Responder directo cuando habia subagente candidato disponible.
