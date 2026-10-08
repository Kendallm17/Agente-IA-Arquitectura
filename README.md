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

El detalle completo de este contrato está en [`CONTEXTO DEL AGENTE.md`](CONTEXTO%20DEL%20AGENTE.md) (el encargo original del proyecto) y en [`.github/reglas-de-proyectos/reglas-inviolables-de-mision.md`](.github/reglas-de-proyectos/reglas-inviolables-de-mision.md) (las reglas ya consolidadas).

## Identidad del orquestador

- Nombre: **Arquitecto**
- Handle: `@arquitecto`
- Archivo: [`.github/agents/leo-arquitec.agent.md`](.github/agents/leo-arquitec.agent.md)
- Rol: entiende la solicitud, identifica la misión, delega a quien corresponda, consolida resultados, pide revisión humana. No hace el análisis él mismo.

## Cómo está organizado el repositorio

```
LeoGeneradorDesc/
├── README.md                        ← este archivo
├── CONTEXTO DEL AGENTE.md            ← el encargo original del proyecto (qué se pidió construir)
├── CATALOGO-DE-ELEMENTOS.md          ← qué existe hoy, explicado en lenguaje simple
├── DUDAS-PENDIENTES.md               ← preguntas abiertas para la persona responsable
│
├── .github/                          ← TODO lo que forma parte de la versión final del agente
│   ├── agents/
│   │   ├── leo-arquitec.agent.md             → orquestador (@arquitecto)
│   │   └── leo-arquitec-subagents/
│   │       ├── context-builder.agent.md
│   │       ├── valoracion-arquitectonica.agent.md
│   │       ├── leo-architecture-guide.agent.md    (genérico de Leo)
│   │       └── leo-ecosystem-integrator.agent.md  (genérico de Leo)
│   ├── skills/leo-arquitec-skills/
│   │   ├── delegacion-aislada/                (rol de aislamiento, independiente de herramienta)
│   │   ├── analizar-formulario-arquitectura/
│   │   ├── analizar-tallaje/
│   │   ├── detectar-faltantes/
│   │   ├── detectar-contradicciones/
│   │   ├── construir-contexto/
│   │   ├── puerta-calidad-contexto/
│   │   ├── evaluar-racionalizacion/
│   │   ├── seleccionar-patron-resiliencia/
│   │   ├── generar-modelo-c4/
│   │   └── (3 skills genéricos de Leo: leo-architecture, orchestration-relations, integration-protocol)
│   ├── reglas-de-proyectos/
│   │   ├── reglas-inviolables-de-mision.md    (reglas duras, activas)
│   │   └── correcciones-humanas.md            (memoria de errores en producción)
│   └── knowledge/                    ← lo que el agente "sabe" y puede citar
│       ├── architecture/
│       │   ├── lineamiento-uso-nube-publica.md
│       │   └── estandar-diseno-arquitecturas-ti.md
│       ├── aws/aws-well-architected-framework.md
│       └── azure/azure-well-architected-framework.md
│
├── documentosAgente/                 ← insumos originales (PDFs, Excel) — solo desarrollo
├── propuestas/                       ← documentos de planificación — solo desarrollo
└── tests/                            ← solo desarrollo
    ├── fixtures/FEDV-226/            ← caso real anonimizado, usado para validar skills
    └── results/                      ← validaciones ya corridas
```

**Regla de ubicación — la más importante del proyecto:** `.github/` es la **versión final** del agente; es lo único que se empaqueta y se entrega. Todo lo que está fuera de `.github/` (`documentosAgente/`, `propuestas/`, `tests/`, `CONTEXTO DEL AGENTE.md`, `DUDAS-PENDIENTES.md`, `CATALOGO-DE-ELEMENTOS.md`, este mismo `README.md`) es **material de desarrollo**: ayuda a construir, decidir y validar, pero no se entrega. Por eso, **ningún archivo dentro de `.github/` puede referenciar un archivo fuera de `.github/`** — ni como ruta, ni como dependencia, ni como "fuente". Si un skill necesita algo de un documento externo, esa información se copia a su propio contrato dentro de `.github/` (ver `construir-contexto/SKILL.md`: trae su propia plantilla embebida, no apunta a otro documento), y si se trata de conocimiento reutilizable, vive en `.github/knowledge/`, no afuera.

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
4. **Si te surge una duda que no podés resolver solo con el repositorio, preguntá antes de agregarla a `DUDAS-PENDIENTES.md`** — ese archivo lo revisa la persona responsable, no se llena por cuenta propia.
5. **Actualizá `CATALOGO-DE-ELEMENTOS.md`** cada vez que construyas algo nuevo — es la explicación en lenguaje simple de qué existe, pensada para que la persona responsable no tenga que leer código ni Markdown técnico para saber qué hay. Y al cerrar una sesión de trabajo, agregá una entrada nueva (fechada, arriba de las anteriores) en su sección "Qué puede hacer el agente hoy": un resumen honesto de qué podría hacer el agente si alguien lo probara en ese momento, y qué todavía no puede — es el historial de avance, no se sobrescribe ni se borra.
6. **Nunca references, desde un archivo dentro de `.github/`, algo que esté fuera de `.github/`.** Esa carpeta es la versión final; lo de afuera es solo desarrollo y no se entrega. Si hace falta mover algo que hoy está afuera porque el agente lo necesita en producción, se mueve adentro de `.github/` (como se hizo con `knowledge/`), no se referencia desde lejos.
7. **Los skills y subagentes del dominio TI se mantienen concisos** — lo necesario para cumplir su objetivo, sin relleno. Las fichas de `.github/knowledge/` pueden ser más extensas porque son material de referencia, no tareas ejecutables.

## Estado actual

Ver [`CATALOGO-DE-ELEMENTOS.md`](CATALOGO-DE-ELEMENTOS.md) para el detalle en lenguaje simple, y [`propuestas/analisis-fase-1-context-builder.md`](propuestas/analisis-fase-1-context-builder.md) §16 para el plan de fases y qué está completado.

Resumen: Fase 0 y Fase 1 (Context Builder) completas y validadas. Fase 2 (Valoración Arquitectónica) completa y validada; falta Gobierno y Cumplimiento dentro de esa misma fase. Fases 3 a 5 (Riesgos, Adquisición, Entregables) no iniciadas.

## Cómo usar el agente hoy (todavía no hay interfaz gráfica)

Hoy todo lo que hay en `.github/agents/` y `.github/skills/` es **Markdown — contratos y reglas, no código.** No hay ningún programa que los lea y los ejecute solo. Se vuelven reales cuando una IA conversacional (GitHub Copilot Chat, Claude Code, o cualquier otra con acceso a los archivos del proyecto) los lee y los sigue como instrucción, igual que seguiría cualquier otra indicación tuya.

### Qué necesitás

- Un editor con este repositorio abierto (por ejemplo, VS Code).
- Un chat de IA conectado a ese editor que pueda leer los archivos del workspace (GitHub Copilot Chat o Claude Code — lo que ya estés usando para construir esto).
- Los documentos de una misión: el Formulario para Diseño de Arquitectura y el Modelo de Tallaje, completados (reales, o los de prueba en `tests/fixtures/FEDV-226/` — recordá que esa carpeta es solo de desarrollo, no se usa en producción).
- Nada más. No hay instalación, no hay servidor, no hay claves ni configuración adicional.

### Paso a paso

1. **Abrí una conversación nueva** con el chat de IA (nueva de verdad — sin el historial de cómo construimos el agente, para probarlo como lo haría alguien que nunca vio este proyecto).
2. **Decile que actúe como el orquestador**, señalándole dónde están sus instrucciones y **limitándole el acceso a `.github/`** (más los documentos de la misión que le estés pasando). Por ejemplo:

   ```
   Actuá como el orquestador @arquitecto de este proyecto.
   Tus instrucciones están en .github/agents/leo-arquitec.agent.md — seguilas al pie de la letra:
   reescaneá los subagentes y skills disponibles dentro de .github/, delegá en vez de resolver todo vos mismo,
   no inventes información, y pedí revisión humana cuando corresponda.

   Para esta prueba, usá únicamente lo que está dentro de .github/ (agentes, skills, reglas, conocimiento).
   No abras ni uses como referencia ningún archivo de documentosAgente/, propuestas/, tests/ ni ningún otro
   lugar fuera de .github/ — esas carpetas no existen en la versión final del agente, y si las usás ahora,
   la prueba no refleja cómo se va a comportar realmente.

   Te paso los documentos de la misión FEDV-XXX:
   - Formulario para Diseño de Arquitectura: [adjuntar el archivo o pegar su contenido]
   - Modelo de Tallaje: [adjuntar el archivo o pegar su contenido]

   Construí el contexto de esta misión.
   ```

   **Por qué importa esta aclaración:** en la primera prueba real, la IA citó `Modelo_Tallaje_Completo.xlsx` (que vive en `documentosAgente/`, fuera de `.github/`) como referencia de las reglas de tallaje — un atajo que no va a estar disponible en la versión final, porque esas reglas ya deberían estar completas en `.github/knowledge/architecture/estandar-diseno-arquitecturas-ti.md`. Si la IA necesitó salir de `.github/` para resolver algo, es señal de que a esa ficha de conocimiento le falta algo, no de que está bien usar el atajo.

3. **Observá qué hace la IA por su cuenta:** si encuentra sola el subagente `context-builder`, si delega en sus 6 skills en el orden correcto, si te dice honestamente cuando falta un dato en vez de inventarlo, si te marca cada recomendación como preliminar.
4. **Comparalo contra lo que ya documentamos**, para saber si se comportó como se diseñó: [`tests/results/validacion-context-builder-fedv226.md`](tests/results/validacion-context-builder-fedv226.md) y [`tests/results/validacion-valoracion-arquitectonica-fedv226.md`](tests/results/validacion-valoracion-arquitectonica-fedv226.md).
5. **Si algo no coincide**, esa es una corrección real de producción — ahí sí empieza a llenarse `.github/reglas-de-proyectos/correcciones-humanas.md`.

### Una aclaración importante

Si tu chat tiene soporte nativo para "agentes personalizados" por `@mención` (una función específica de algunas herramientas), puede que escribir `@arquitecto` directamente ya lo invoque. Si no lo reconoce, no es un error — simplemente esa función no está activada, y el método del paso 2 (decirle explícitamente qué archivo leer) funciona siempre, sin depender de ninguna configuración especial.
