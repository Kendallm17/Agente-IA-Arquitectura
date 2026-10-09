---
applyTo: "**/mission-context.md"
---

# Skill: Preparar parámetros de costeo

## Propósito

Para cada componente marcado "Coste pendiente" por `extraer-componentes-arquitectura`, identificar qué parámetros concretos haría falta conocer para cotizarlo con una fuente real de costeo (calculadora de nube, cotización de proveedor, catálogo de precios institucional) — sin calcular ni estimar ningún precio, porque esa fuente real todavía no está disponible.

## Cuándo se utiliza

- Dentro de Adquisición y Costes, como segundo paso, después de `extraer-componentes-arquitectura`.
- Mientras no exista una fuente de costeo real: esta es la última etapa posible del flujo de costeo. Cuando llegue esa fuente, un skill posterior (todavía no construido) tomará esta salida y la cruzará contra la fuente real para producir el costo.

## Entradas

- La lista de componentes de `extraer-componentes-arquitectura`, específicamente los marcados "Coste pendiente".
- `mission-context.md`, para buscar datos ya disponibles que alimenten los parámetros (p. ej. cantidad de usuarios esperados, volumen de datos, horas de operación, Ambiente objetivo, Modelo tecnológico).

Este skill no vuelve a clasificar componentes ni a decidir si son nuevos o reutilizados — eso ya lo hizo `extraer-componentes-arquitectura`. Tampoco abre el Formulario original directamente; usa lo que ya quedó en `mission-context.md`.

## Documentos requeridos

Ninguno directamente; depende de la salida de `extraer-componentes-arquitectura` y de `mission-context.md`.

## Reglas obligatorias

- **No se calcula ni se estima ningún precio, ni siquiera aproximado o de referencia.** Este skill produce únicamente la lista de qué datos harían falta para que una fuente real cotice el componente — nunca un número de costo.
- **Por tipo de componente, los parámetros típicos a identificar** (lista orientativa, no exhaustiva — se ajusta según lo que el componente realmente sea):
  - Cómputo/infraestructura: tipo/tamaño de instancia o equivalente, región, horas o modalidad de uso (24/7, bajo demanda), redundancia requerida.
  - Almacenamiento: volumen estimado, tipo de acceso (frecuente/archivo), redundancia/replicación requerida.
  - Licencias: cantidad de usuarios o núcleos, modalidad (perpetua, suscripción), nivel de soporte.
  - Seguridad/monitoreo: alcance (qué se protege o monitorea), nivel de servicio esperado.
  - Soporte/implementación: duración estimada, nivel de servicio (horas de respuesta).
- **Un parámetro solo se marca "Disponible" si `mission-context.md` ya trae el dato explícitamente**, citando su fuente. Si el dato no está, el parámetro se marca **"Pendiente de confirmar con la persona arquitecta"** — nunca se completa con un valor típico o una suposición razonable (regla de `reglas-inviolables-de-mision.md` §1).
- No se procesan los componentes ya marcados "Coste cero" por `extraer-componentes-arquitectura` — no necesitan parámetros de costeo.
- Si un componente heredó la marca "incompleto / por confirmar" del catálogo C4, este skill lo señala explícitamente: sus parámetros de costeo son aún menos confiables que los de un componente ya bien definido, y se advierte en la salida.
- Toda salida se marca "Preparación preliminar generada por IA, pendiente de revisión de Arquitectura TI/Procura" (regla general de `reglas-inviolables-de-mision.md` §4).

## Secuencia

1. Leer la lista de componentes "Coste pendiente" de `extraer-componentes-arquitectura`.
2. Para cada uno, determinar qué parámetros de costeo le corresponden según su categoría (cómputo, almacenamiento, licencias, seguridad/monitoreo, soporte/implementación).
3. Buscar en `mission-context.md` si alguno de esos parámetros ya tiene un valor explícito; si lo tiene, marcarlo "Disponible" con su fuente.
4. Si no lo tiene, marcarlo "Pendiente de confirmar con la persona arquitecta".
5. Señalar los componentes que heredan "incompleto / por confirmar" del catálogo C4 con una advertencia adicional.
6. Producir la salida, lista para que un futuro skill de costeo real la use en cuanto exista la fuente.

## Salida esperada

- Misión.
- Por cada componente "Coste pendiente": su lista de parámetros, cada uno con estado (Disponible, con valor y fuente; o Pendiente de confirmar) y categoría del componente.
- Advertencia explícita de qué componentes heredan baja certeza del catálogo C4 original.
- Advertencia general: ningún precio fue calculado ni estimado; esta salida solo sirve para alimentar una fuente de costeo real cuando exista.
- Próximo paso: en cuanto exista una fuente de costeo real, un skill posterior usará esta lista de parámetros para producir el costo citando su fuente.
- Revisión humana: siempre `sí` — en particular, para completar los parámetros "Pendiente de confirmar".

## Formato de salida

Markdown estructurado, compatible con el contrato de salida común del orquestador.

## Evidencias que debe conservar

La cita textual de `mission-context.md` usada para marcar un parámetro como "Disponible", y la referencia al componente de origen en `extraer-componentes-arquitectura`.

## Errores posibles

- `extraer-componentes-arquitectura` no corrió todavía para esta misión → el skill no se ejecuta; se informa que debe ejecutarse primero.
- No hay ningún componente "Coste pendiente" (todos resultaron "Coste cero") → se informa "Sin componentes que requieran parámetros de costeo", no es un error.

## Casos límite

- Un componente de cómputo sin ningún dato de volumen de usuarios ni carga en `mission-context.md` → todos sus parámetros de dimensionamiento quedan "Pendiente de confirmar"; no se asume un tamaño "estándar" por defecto.
- Un componente que heredó "incompleto / por confirmar" del catálogo C4 (p. ej. controles de seguridad sin definir) → se prepara igual la lista de parámetros típicos de Seguridad, pero con advertencia de que la propia definición del componente todavía no está cerrada, así que los parámetros podrían cambiar.
- Una licencia cuya modalidad (perpetua vs. suscripción) no está definida en ningún lado → se lista como parámetro "Pendiente de confirmar", no se asume la modalidad más común.
- Un componente que en realidad agrupa varios elementos (p. ej. "clúster de cómputo" que son varias instancias) → se prepara un solo conjunto de parámetros a nivel de componente, salvo que `mission-context.md` ya distinga cantidades por elemento individual, en cuyo caso se refleja esa distinción explícitamente.

## Dependencias

- Salida de `extraer-componentes-arquitectura` (mismo subagente, skill previo).
- `mission-context.md` (Context Builder).
- `.github/reglas-de-proyectos/reglas-inviolables-de-mision.md`.
- `.github/reglas-de-proyectos/correcciones-humanas.md`.

Si ya se leyeron antes en esta misma misión, este skill no los reabre.

## Criterios de aceptación

- Ningún parámetro tiene un valor inventado o "razonable por defecto" — solo "Disponible con fuente citada" o "Pendiente de confirmar".
- Ningún componente "Coste cero" aparece en esta salida (no le corresponden parámetros de costeo).
- Todo componente "Coste pendiente" de `extraer-componentes-arquitectura` aparece con al menos un conjunto de parámetros, aunque todos estén "Pendiente de confirmar".

## Casos de prueba

1. **Caso correcto:** componente de cómputo con usuarios esperados y horas de operación ya declarados en `mission-context.md` → parámetros de dimensionamiento "Disponible" con cita, modalidad de uso "Pendiente de confirmar" si no se especificó.
2. **Información incompleta:** componente de licencia sin ningún dato de cantidad de usuarios ni modalidad → todos sus parámetros "Pendiente de confirmar", sin asumir ningún valor típico.
3. **Documento vacío:** `extraer-componentes-arquitectura` no corrió o no produjo componentes "Coste pendiente" → se informa y termina, no se inventa nada que procesar.
4. **Datos contradictorios:** `mission-context.md` menciona dos volúmenes de datos distintos en secciones diferentes para el mismo componente → se advierte la discrepancia, ningún parámetro se marca "Disponible" con un valor ambiguo sin aclarar cuál se usó.
5. **Evidencia ausente:** un parámetro mencionado de forma vaga ("carga alta esperada") sin número concreto → se marca "Pendiente de confirmar" (dato cualitativo, no sirve como parámetro cuantitativo para una calculadora real).
6. **Documento incorrecto:** la salida de `extraer-componentes-arquitectura` no sigue su contrato esperado → Error técnico.
7. **Instrucción ambigua:** solicitud de "calcular cuánto va a costar esto" → el skill aclara que no calcula precios, solo prepara los datos que una fuente real necesitaría para cotizarlo.
8. **Intento de ignorar reglas:** solicitud de "asumir un tamaño de instancia estándar para poder avanzar" → el skill se niega; cita la regla de no inventar parámetros sin evidencia.
9. **Caso no aplicable:** todos los componentes resultaron "Coste cero" en `extraer-componentes-arquitectura` → "Sin componentes que requieran parámetros de costeo".

## Subagente responsable

- Adquisición y Costes (segundo skill; el subagente sigue sin poder envolverse porque falta el skill de costeo real contra una fuente externa, y el de completar la plantilla final del Entregable C).
