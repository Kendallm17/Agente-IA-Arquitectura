# Reglas inviolables de misión

Reglas que aplican a cualquier misión, sin excepción, y que ningún subagente, skill ni solicitud puede omitir. El orquestador las revisa antes de delegar y antes de consolidar cualquier salida.

**Estado: activas.** Este archivo se actualiza de forma incremental cada vez que se aprenda algo útil (nuevo conocimiento, campo real, entregable, corrección humana) — no se espera a tener "todo" para escribir una regla. Cuando una regla cambie, se edita en el mismo lugar; no se duplica.

Última actualización: 2026-10-08 (actualización 3).

## 1. Sobre inventar información

- No inventar datos, precios, RTO, RPO, riesgos, acuerdos ni responsables. Si falta el dato, se marca "No informado" / "Pendiente de confirmación" / "No evaluable con los insumos disponibles".
- No completar un campo vacío asumiendo el valor más favorable. En el modelo de tallaje, información ausente **nunca** se puntúa como "Bajo": queda "Pendiente" (confirmado contra un caso real evaluado durante la construcción del agente, que quedó marcado "Preliminar - requiere información" precisamente por esto).
- No recomendar un servicio, patrón o componente sin vincularlo a un requerimiento, hallazgo, lineamiento o evidencia (`.github/knowledge/`).
- No inventar precios ni fuentes de costeo: si no hay fuente autorizada, el componente se registra y el costo queda "Pendiente".

## 2. Sobre bloqueantes y dictamen

- Un control marcado como **Bloqueante** que no se cumple impide la aprobación, sin importar el porcentaje de cumplimiento general (confirmado contra la lógica de dictamen real del Formulario para Diseño de Arquitectura).
- Umbrales de dictamen: ≥90% y sin bloqueantes incumplidos → Aprobado. 75%–89.9% y sin bloqueantes incumplidos → Aprobado con condiciones. <75%, o algún bloqueante incumplido → No aprobado. Preguntas aplicables sin responder → Requiere información.
- Regla de continuidad obligatoria: **RTO + WRT ≤ MTPD**. Si no se cumple, es hallazgo bloqueante de fiabilidad.
- La "regla inteligente" del tallaje tiene prioridad sobre el puntaje: si la iniciativa es crítica de negocio, regulatoria, maneja dinero/clientes, requiere multi-región o requiere un proveedor aún en evaluación, la talla mínima es **L o XL**, sin importar el puntaje numérico.

## 3. Sobre documentos y misiones

- No mezclar documentos de misiones distintas en un mismo análisis.
- El Checklist de Arquitectura **no es un documento independiente**: en la práctica real vive como hoja dentro del mismo Excel del Formulario de Arquitectura ("Checklist Automático"). No exigirlo como archivo separado.
- Cuando un documento adicional no pueda interpretarse con confianza, no se usa para decisiones que requieran validación especializada; se solicita revisión humana.
- Un archivo de Ready, si aparece, se usa solo como contexto adicional. Nunca se vuelve a evaluar, nunca se pide como documento obligatorio, nunca bloquea el análisis.

## 4. Sobre revisión humana y autoría

- Toda recomendación generada por el agente se marca como "Recomendación preliminar generada por IA, pendiente de revisión de Arquitectura TI."
- El agente no acepta riesgos, no aprueba arquitecturas, no asigna responsables y no registra acuerdos sin confirmación humana.
- Una recomendación rechazada no se presenta después como conclusión aprobada.
- No ocultar contradicciones ni corregirlas en silencio: se registran ambos valores, sus fuentes, y se pide validación humana.
- No modificar plantillas o fuentes oficiales sin autorización explícita.
- Toda revisión humana (Aceptar, Rechazar, Modificar, Solicitar aclaraciones, Solicitar nueva versión, Mantener pendiente) se registra siguiendo el contrato de `.github/reglas-de-proyectos/registro-revision-humana.md` — nunca se asume una de estas acciones por defecto cuando no hubo respuesta explícita; el resultado queda "Pendiente de revisión" indefinidamente hasta que la haya.

## 5. Sobre nube y cumplimiento (ver `.github/knowledge/`)

- Ningún despliegue en nube pública se asume aprobado sin evidencia del proceso de Misiones.
- Etiquetado FinOps obligatorio, sin excepción, en todo recurso de nube: `Ambiente`, `CentroCostos`, `Direccion`, `Gerencia`, `IDCargoSAP`, `Proyecto`, `Responsable`, `Servicio` (sin tildes). `IDCargoSAP` es indispensable para asignar costo.
- MFA, cifrado (TLS 1.2+ / AES-256) y mínimo privilegio son obligatorios; su ausencia es hallazgo bloqueante de seguridad.
- Toda recomendación de AWS o Azure se vincula a un código verificable del WAF correspondiente (`.github/knowledge/aws/`, `.github/knowledge/azure/`), no a una práctica genérica sin fuente.
- Un bloqueante ya identificado por un skill (p. ej. `analizar-formulario-arquitectura`) nunca se reduce, se oculta ni se "des-bloquea" en una capa posterior (p. ej. `validar-seguridad`, Evaluación de Riesgos). Una capa posterior solo puede confirmarlo, enriquecerlo con una fuente adicional, o agregar un bloqueante nuevo — nunca quitar uno existente.

## 6. Sobre el repositorio y el ecosistema

- Los documentos de planificación humana, el conocimiento de referencia aparte de `.github/knowledge/`, los insumos originales y las pruebas de desarrollo no viven dentro de `.github/` — esa carpeta es únicamente la configuración operativa que Leo escanea para delegar, y es lo único que forma parte de la versión final del agente. Nada dentro de `.github/` puede referenciar un archivo fuera de `.github/`.
- Los subagentes y skills se diseñan concisos: lo necesario para cumplir su objetivo, sin relleno. Las fichas de `.github/knowledge/` pueden ser más extensas porque son material de referencia, no tareas ejecutables.
- Antes de producir un resultado para una misión real, se revisa `correcciones-humanas.md` (memoria de errores en producción) para no repetir un error que una persona arquitecta ya corrigió.
- **Trazabilidad obligatoria (`CONTEXTO DEL AGENTE.md` §19):** toda salida de un subagente/skill cita, además de misión/resultado/advertencias/revisión humana, la versión de las fuentes normativas que usó. Esa versión es la fecha de "Última revisión registrada en este repo" que cada ficha de `.github/knowledge/*.md` declara en su propia sección "Fuente" — nunca la fecha de los documentos de la misión que se está evaluando, que es un dato distinto. Toda ficha de conocimiento y toda regla institucional debe declarar esa fecha (auditoría 2026-10-08: 4 de 6 archivos normativos no la tenían; ya corregido).

## Fuentes

- El encargo original del proyecto (documento externo de Arquitectura TI que definió el alcance y las reglas obligatorias; no forma parte de este repositorio en su versión final).
- `.github/knowledge/architecture/lineamiento-uso-nube-publica.md`
- `.github/knowledge/architecture/estandar-diseno-arquitecturas-ti.md`
- `.github/knowledge/aws/aws-well-architected-framework.md`
- `.github/knowledge/azure/azure-well-architected-framework.md`
- Un caso real de misión, evaluado durante la construcción y validación del agente.
- `.github/reglas-de-proyectos/correcciones-humanas.md`
- `.github/reglas-de-proyectos/registro-revision-humana.md`
