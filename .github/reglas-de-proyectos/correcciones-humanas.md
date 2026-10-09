# Correcciones humanas

Última entrada agregada: 2026-10-07 (ver la sección fechada más abajo; este archivo no tiene "última actualización" fija porque crece con cada corrección real — la fecha vigente es la de la entrada más reciente).

Memoria de errores **en producción**: cada vez que una persona arquitecta corrige un resultado del agente ya en uso (un hallazgo mal calculado, un dictamen incorrecto, una recomendación sin evidencia, una talla mal asignada, un entregable con un formato equivocado, etc.) durante la evaluación de una misión real, la corrección se registra aquí para que el agente no la repita.

Este archivo **no** es para decisiones de construcción del repositorio (esas quedan en el historial de la conversación de desarrollo y, si se vuelven permanentes, en `reglas-inviolables-de-mision.md`). Es para errores del agente ya operando: la etapa de producción, no la de desarrollo.

Todo subagente y skill del dominio TI revisa este archivo antes de producir un resultado del mismo tipo que uno ya corregido aquí.

## Formato de cada entrada

```markdown
## [AAAA-MM-DD] Misión <ID o FED> — Resumen corto del error

- **Resultado que entregó el agente:** ...
- **Corrección de la persona arquitecta:** ...
- **Causa raíz:** (regla mal aplicada, dato mal leído, fuente ausente, etc.)
- **Regla a aplicar en adelante:** ...
- **Skill o subagente responsable del error:** ...
- **¿Se elevó a `reglas-inviolables-de-mision.md`?** Sí/No — si sí, enlazar la sección.
```

Las entradas se agregan al final (orden cronológico) y no se borran, aunque ya estén reflejadas en una regla consolidada: son la evidencia de por qué existe esa regla.

---

## [2026-10-07] Misión FEDV-226 — ARQ-020 no se reportó como bloqueante incumplido

- **Resultado que entregó el agente:** al correr `context-builder` sobre el Formulario y el Tallaje de FEDV-226, en "Pendientes priorizados" dijo *"Confirmar si ARQ-020 aplica y, si aplica, completar su respuesta"* — tratándola como si todavía no tuviera respuesta.
- **Corrección de la persona arquitecta:** ARQ-020 ("¿Dónde se almacenan, respaldan y procesan los datos?", severidad Bloqueante) ya tiene `Respuesta = "Cumple parcialmente"` en el Formulario real. Al tener esa respuesta y ser Bloqueante, debía reportarse como **bloqueante incumplido** — exactamente igual que ARQ-014, que sí se identificó bien en la misma corrida.
- **Causa raíz:** la columna `¿Aplica?` de ARQ-020 está vacía en el Formulario (nunca se marcó explícitamente "Sí"), aunque sí tiene una `Respuesta`. La regla del skill no cubría ese caso: decía qué hacer si `¿Aplica?=Sí` y la `Respuesta` está vacía, pero no qué hacer si `¿Aplica?` está vacío y la `Respuesta` sí está presente. El agente pareció priorizar la ausencia de `¿Aplica?` por sobre la `Respuesta` ya registrada.
- **Regla a aplicar en adelante:** una `Respuesta` presente es evidencia suficiente de que la pregunta se trató como aplicable, sin importar si `¿Aplica?` quedó sin marcar. Además, los bloqueantes incumplidos deben listarse siempre separados de las preguntas sin responder que no son bloqueantes, para que un caso como este no se diluya en una lista genérica.
- **Skill o subagente responsable del error:** `analizar-formulario-arquitectura`, dentro de `context-builder`.
- **¿Se elevó a `reglas-inviolables-de-mision.md`?** No — se corrigió directamente en la especificación del skill (`.github/skills/leo-arquitec-skills/analizar-formulario-arquitectura/SKILL.md`, reglas obligatorias, casos límite y casos de prueba); es una aclaración de lógica interna del skill, no una regla de negocio institucional nueva.
