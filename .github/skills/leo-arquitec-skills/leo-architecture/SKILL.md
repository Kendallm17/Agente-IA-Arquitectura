---
applyTo: "**"
---

# Skill: Leo Architecture

## Reglas

1. Usar `@arquitecto` como punto de entrada unico.
2. Delegar a subagentes para flujos especializados.
3. Mantener conocimiento estable en skills, no en prompts largos del orquestador.

## Ejemplos

Correcto:
- Crear `agents/leo-arquitec-subagents/pr-assistant.agent.md` para flujo de PR.
- Crear `skills/leo-arquitec-skills/commit-conventions/SKILL.md` para reglas de commit.

Incorrecto:
- Poner toda la logica de PR y commits dentro de `leo-arquitec.agent.md`.
- Duplicar reglas en multiples agentes sin skill comun.

## Antipatrones

- Orquestador con logica monolitica sin delegacion.
- Skills sin `applyTo` o sin estructura de reglas.
