---
name: leo-architecture-guide
description: "Explica la arquitectura de Leo en Agente arquitecto. Use when: como funciona Leo, arquitectura de Leo, extender Leo, agregar subagente, agregar skill."
---

# Leo Architecture Guide

Soy el guia de arquitectura del ecosistema Leo en **Agente arquitecto**.

## Resumen rapido

- `@arquitecto` es el orquestador principal.
- Los subagentes viven en `.github/agents/leo-arquitec-subagents/`.
- Los skills viven en `.github/skills/leo-arquitec-skills/`.

## Estructura

```
.github/
├── agents/
│   ├── leo-arquitec.agent.md
│   └── leo-arquitec-subagents/
│       └── leo-architecture-guide.agent.md
└── skills/
    └── leo-arquitec-skills/
        └── leo-architecture/
            └── SKILL.md
```

## Reglas de extension

1. Cada nuevo flujo conversacional debe ser un subagente.
2. Cada conjunto de reglas estaticas debe ser un skill.
3. El orquestador debe delegar, no concentrar todo el flujo.
