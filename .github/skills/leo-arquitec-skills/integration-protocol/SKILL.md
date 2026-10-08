---
applyTo: "**"
---

# Skill: Integration Protocol

Reglas para integrar nuevos skills y subagentes al ecosistema de `@arquitecto`.

## Cuando ejecutar este protocolo

- Cuando el usuario diga que agrego un skill.
- Cuando el usuario diga que agrego un subagente.
- Cuando Leo no delega por falta de relacion actualizada.

## Pasos obligatorios

1. Reescaneo:
- Subagentes: `.github/agents/leo-arquitec-subagents/*.agent.md`.
- Skills: `.github/skills/leo-arquitec-skills/**/SKILL.md`.

2. Parseo minimo:
- Subagente: `name`, `description`, keywords de `Use when`.
- Skill: `applyTo` y nombre de carpeta.

3. Enlace operativo:
- Intencion -> subagente (por coincidencia de keywords de `Use when`).
- Contexto de archivo/tarea -> skill (por `applyTo`).

4. Actualizacion de orquestador:
- Agregar subagente en "Subagentes disponibles" si no estaba.
- Agregar reglas de "Deteccion de contexto" para el nuevo subagente.
- Agregar skill/subagente en "Referencias".

5. Validacion:
- Probar al menos una frase de usuario que deba delegar al subagente nuevo.
- Confirmar que el skill nuevo es consultado en su dominio.

## Antipatrones

- Agregar archivos nuevos sin incorporarlos al ruteo del orquestador.
- No leer `Use when` y delegar por suposicion.
- Ignorar `applyTo` y usar skills fuera de contexto.
