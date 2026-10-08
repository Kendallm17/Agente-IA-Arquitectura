---
applyTo: "**"
---

# Skill: Delegación aislada

Define el rol de aislamiento de contexto entre el orquestador y los subagentes. Es independiente de cualquier herramienta: el contrato se cumple con Markdown y con la capacidad de cada plataforma, sin depender de una función concreta.

## Propósito

Ejecutar el trabajo de un subagente sin que su contexto interno contamine al orquestador ni a otros subagentes, y devolver solo un resultado estructurado.

## Cuándo se usa

- Cuando el orquestador delega una tarea a un subagente con `Use when` coincidente.
- Cuando dos subagentes deben trabajar sobre la misma misión sin compartir razonamiento intermedio.

## Entradas

- Identificador de misión.
- Nombre exacto del subagente (campo `name` de su frontmatter).
- Solicitud o tarea delegada.
- Contexto autorizado de la misión (solo lo necesario para la tarea).

## Reglas obligatorias

1. El subagente recibe solo el contexto de la misión que necesita. No recibe el historial completo del orquestador.
2. El subagente no comparte razonamiento intermedio con el orquestador; solo devuelve su salida estructurada.
3. Cada subagente se ejecuta con sus propias reglas y skills; no hereda reglas de otro subagente.
4. Ningún resultado de un subagente se presenta como decisión humana.
5. Si la plataforma no ofrece aislamiento real, el orquestador ejecuta el contrato del subagente leyendo su archivo, manteniendo las mismas reglas de entrada y salida.
6. Ningún paso de este skill depende de una herramienta, interfaz ni ruta local.

## Proceso

1. Identificar el subagente por su `name` exacto en `.github/agents/leo-arquitec-subagents/`.
2. Preparar la entrada mínima: misión, tarea y contexto autorizado.
3. Ejecutar el contrato del subagente con aislamiento disponible en la plataforma.
4. Recibir la salida estructurada.
5. Validar que la salida cumple el formato esperado (ver Salida esperada).
6. Entregar la salida al orquestador para consolidación.

## Salida esperada

- Identificador de misión.
- Subagente ejecutado.
- Estado (Completado, Completado con observaciones, Requiere información, Requiere corrección, No evaluable, Error técnico).
- Resultado estructurado.
- Fuentes utilizadas.
- Advertencias.
- Revisión humana requerida (sí/no).

## Errores y casos límite

- **Nombre de subagente no encontrado:** reintentar una vez con el nombre exacto del frontmatter; si falla, informar y no delegar en silencio.
- **Salida sin formato:** solicitar nueva salida con el formato indicado; no aceptarla como válida.
- **Aislamiento no disponible:** aplicar la regla 5 y registrar la advertencia.
- **Subagente sin "Use when":** puede ejecutarse por nombre o tema, pero se registra la advertencia.

## Casos de prueba

1. Delegación correcta: subagente encontrado, salida con formato completo.
2. Nombre incorrecto: reintento con nombre exacto y, si falla, informe sin delegación silenciosa.
3. Salida incompleta: rechazada y solicitada de nuevo.
4. Aislamiento no disponible: el contrato se ejecuta igual, con advertencia registrada.
5. Intento de pasar contexto de otra misión: bloqueado.

## Dependencias

- Subagentes en `.github/agents/leo-arquitec-subagents/`.
- Reglas en `.github/reglas-de-proyectos/`.

## Subagente responsable

- Orquestador `@arquitecto` (`.github/agents/leo-arquitec.agent.md`).
