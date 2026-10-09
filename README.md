# Agente Evaluador de Arquitectura TI (Arquitecto, sobre la base Leo)

Este README es el punto de entrada del proyecto. Si sos una persona o una IA que recién llega a este repositorio, leé esto primero — te dice qué es, qué no es, cómo está organizado y cómo comportarte dentro de él.

## Qué es este proyecto

Un agente construido con GitHub Copilot / modelos de lenguaje, sobre una arquitectura de tres niveles (orquestador → subagentes → skills) heredada del framework **Leo**, cuyo objetivo es apoyar a las personas arquitectas de TI en el análisis de factibilidad de iniciativas tecnológicas (misiones o FED): revisar documentos, identificar riesgos y brechas, y preparar los cuatro entregables oficiales del proceso.

**No es un chatbot de preguntas y respuestas.** Recibe los documentos que la misión ya completó, extrae lo que puede, y solo pregunta lo que de verdad no puede obtener de los documentos.

## Qué NO es este agente

- **No sustituye a la persona arquitecta.** Toda salida es preliminar, pendiente de revisión humana.
- **No aprueba arquitecturas ni acepta riesgos.** Eso lo decide siempre una persona.
- **Criterios Ready están completamente fuera de alcance.** No se evalúan, no se generan, no se piden como documento obligatorio, no bloquean nada.
- **No inventa información.** Ni datos, ni precios, ni RTO/RPO, ni riesgos sin evidencia. Si falta algo, lo dice explícitamente en vez de completarlo.
- **No cambia el proceso de Arquitectura TI.** Cualquier mejora que lo toque se marca como propuesta pendiente de aprobación, nunca se aplica en silencio.

El detalle completo de este contrato está consolidado en [`.github/reglas-de-proyectos/reglas-inviolables-de-mision.md`](.github/reglas-de-proyectos/reglas-inviolables-de-mision.md).

## Identidad del orquestador

- Nombre: **Arquitecto**
- Handle: `@arquitecto`
- Archivo: [`.github/agents/leo-arquitec.agent.md`](.github/agents/leo-arquitec.agent.md)
- Rol: entiende la solicitud, identifica la misión, delega a quien corresponda, consolida resultados, pide revisión humana. No hace el análisis él mismo.

## Cómo está organizado el repositorio

```
LeoGeneradorDesc/
├── README.md                         ← este archivo
│
└── .github/                          ← TODO lo que forma parte de la versión final del agente
    ├── agents/
    │   ├── leo-arquitec.agent.md             → orquestador (@arquitecto)
    │   └── leo-arquitec-subagents/
    │       ├── context-builder.agent.md
    │       ├── valoracion-arquitectonica.agent.md
    │       ├── gobierno-cumplimiento.agent.md
    │       ├── evaluacion-riesgos.agent.md
    │       ├── leo-architecture-guide.agent.md    (genérico de Leo)
    │       └── leo-ecosystem-integrator.agent.md  (genérico de Leo)
    ├── skills/leo-arquitec-skills/
    │   ├── delegacion-aislada/                (rol de aislamiento, independiente de herramienta)
    │   ├── analizar-formulario-arquitectura/
    │   ├── analizar-tallaje/
    │   ├── detectar-faltantes/
    │   ├── detectar-contradicciones/
    │   ├── construir-contexto/
    │   ├── puerta-calidad-contexto/
    │   ├── evaluar-racionalizacion/
    │   ├── seleccionar-patron-resiliencia/
    │   ├── comparar-alternativas-nube/
    │   ├── generar-modelo-c4/
    │   ├── validar-seguridad/          (primer skill de Gobierno y Cumplimiento)
    │   ├── validar-lineamiento-nube/   (segundo skill de Gobierno y Cumplimiento)
    │   ├── identificar-riesgos/        (primer skill de Evaluación de Riesgos)
    │   ├── generar-acciones-correctivas/ (segundo skill de Evaluación de Riesgos)
    │   ├── extraer-componentes-arquitectura/ (primer skill de Adquisición y Costes, subagente todavía no completo)
    │   ├── preparar-parametros-costeo/       (segundo skill de Adquisición y Costes)
    │   └── (3 skills genéricos de Leo: leo-architecture, orchestration-relations, integration-protocol)
    ├── reglas-de-proyectos/
    │   ├── reglas-inviolables-de-mision.md    (reglas duras, activas)
    │   ├── correcciones-humanas.md            (memoria de errores en producción)
    │   └── registro-revision-humana.md        (contrato de datos para aceptar/rechazar/modificar un resultado)
    └── knowledge/                     ← lo que el agente "sabe" y puede citar
        ├── architecture/
        │   ├── lineamiento-uso-nube-publica.md
        │   └── estandar-diseno-arquitecturas-ti.md
        ├── aws/aws-well-architected-framework.md
        └── azure/azure-well-architected-framework.md
```

**Regla de ubicación — la más importante del proyecto:** `.github/` es la **versión final** del agente; es lo único que se empaqueta y se entrega, junto con este `README.md`. Todo el material usado para construir, decidir y validar el agente (documentos originales, planes de trabajo, fixtures de prueba, catálogos explicativos) se maneja aparte, fuera de este repositorio, y **nunca se referencia desde acá** ni desde ningún archivo dentro de `.github/`. Si un skill necesita algo de un documento externo, esa información se copia a su propio contrato dentro de `.github/` (ver `construir-contexto/SKILL.md`: trae su propia plantilla embebida, no apunta a otro documento), y si se trata de conocimiento reutilizable, vive en `.github/knowledge/`.

## Arquitectura agentic (herencia de Leo)

Tres niveles, de arriba hacia abajo:

1. **Orquestador** (`@arquitecto`): entiende, delega, consolida. Nunca hace el trabajo de un subagente que ya existe.
2. **Subagentes** (`.github/agents/leo-arquitec-subagents/*.agent.md`): agrupan un conjunto de skills ya construidos y validados bajo una responsabilidad única. Se construyen **después** de sus skills, no antes — primero se prueba que cada skill funciona solo, y recién entonces se envuelve.
3. **Skills** (`.github/skills/leo-arquitec-skills/*/SKILL.md`): una tarea pequeña, concreta y verificable. Cada uno tiene Propósito, Entradas, Reglas obligatorias, Secuencia, Salida esperada, Errores, Casos límite, Dependencias, Criterios de aceptación y Casos de prueba — sin esas secciones, no es un skill terminado.

Delegación: el orquestador busca primero un subagente por intención (`Use when`), si no hay un subagente lo resuelve con un skill por patrón de archivo (`applyTo`), y si no hay ninguno lo informa en vez de improvisar.

## Cómo comportarse si sos una IA trabajando en este repo

1. **Leé `.github/reglas-de-proyectos/reglas-inviolables-de-mision.md` y `correcciones-humanas.md` antes de generar cualquier salida de dominio TI.** Son las reglas que ya nadie puede romper y los errores ya corregidos.
2. **No inventes.** Si un dato, precio, fórmula o regla de negocio no está confirmado, decilo explícitamente ("Pendiente de confirmación", "Requiere información") en vez de completarlo con un supuesto razonable.
3. **No construyas de una sola vez.** El método de este proyecto es: especificar un skill (Intención → Estructura completa) → validarlo a mano contra un caso real → recién ahí construir el siguiente, y solo envolver un subagente cuando todos sus skills ya existen y están probados.
4. **Nunca references, desde un archivo dentro de `.github/` (ni desde este README), algo que esté fuera de este repositorio.** `.github/` (más este `README.md`) es la versión final que se entrega; cualquier otro material de desarrollo se maneja aparte y no debe nombrarse ni enlazarse desde acá.
5. **Los skills y subagentes del dominio TI se mantienen concisos** — lo necesario para cumplir su objetivo, sin relleno. Las fichas de `.github/knowledge/` pueden ser más extensas porque son material de referencia, no tareas ejecutables.

## Estado actual

- **Completo y validado:** Context Builder (6 skills), Valoración Arquitectónica (3 skills), Gobierno y Cumplimiento (2 skills), Evaluación de Riesgos (2 skills).
- **No iniciado:** Adquisición y Costes, Generación de Entregables (los 4 entregables oficiales).

## Cómo usar el agente hoy (todavía no hay interfaz gráfica)

Hoy todo lo que hay en `.github/agents/` y `.github/skills/` es **Markdown — contratos y reglas, no código.** No hay ningún programa que los lea y los ejecute solo. Se vuelven reales cuando una IA conversacional (GitHub Copilot Chat, Claude Code, o cualquier otra con acceso a los archivos del proyecto) los lee y los sigue como instrucción, igual que seguiría cualquier otra indicación tuya.

### Qué necesitás

- Un editor con este repositorio abierto (por ejemplo, VS Code).
- Un chat de IA conectado a ese editor que pueda leer los archivos del workspace.
- Los documentos de una misión, completados: el Formulario para Diseño de Arquitectura y el Modelo de Tallaje.
- Nada más. No hay instalación, no hay servidor, no hay claves ni configuración adicional.

### Paso a paso

1. **Abrí una conversación nueva** con el chat de IA (nueva de verdad — sin historial previo, para probarlo como lo haría alguien que nunca vio este proyecto).
2. **Decile que actúe como el orquestador**, señalándole dónde están sus instrucciones y **limitándole el acceso a `.github/`**. Por ejemplo:

   ```
   Actuá como el orquestador @arquitecto de este proyecto.
   Tus instrucciones están en .github/agents/leo-arquitec.agent.md — seguilas al pie de la letra:
   identificá los subagentes y skills disponibles dentro de .github/, delegá en vez de resolver todo vos mismo,
   no inventes información, y pedí revisión humana cuando corresponda.

   Usá únicamente lo que está dentro de .github/ (agentes, skills, reglas, conocimiento). No uses como
   referencia ningún otro archivo o carpeta que no esté dentro de .github/.

   Te paso los documentos de la misión FEDV-XXX:
   - Formulario para Diseño de Arquitectura: [adjuntar el archivo o pegar su contenido]
   - Modelo de Tallaje: [adjuntar el archivo o pegar su contenido]

   Construí el contexto de esta misión.
   ```

3. **Observá qué hace la IA por su cuenta:** si encuentra sola el subagente `context-builder`, si delega en sus 6 skills en el orden correcto, si te dice honestamente cuando falta un dato en vez de inventarlo, si te marca cada recomendación como preliminar.
4. **Si algo no coincide con lo que el skill promete en su propia especificación**, esa es una corrección real de producción — ahí sí empieza a llenarse `.github/reglas-de-proyectos/correcciones-humanas.md`.

### Una aclaración importante

Si tu chat tiene soporte nativo para "agentes personalizados" por `@mención` (una función específica de algunas herramientas), puede que escribir `@arquitecto` directamente ya lo invoque. Si no lo reconoce, no es un error — simplemente esa función no está activada, y el método del paso 2 (decirle explícitamente qué archivo leer) funciona siempre, sin depender de ninguna configuración especial.
