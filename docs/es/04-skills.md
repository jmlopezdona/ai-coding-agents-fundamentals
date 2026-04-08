# 4. Skills: conocimiento bajo demanda

Las skills son paquetes de instrucciones y recursos que el agente carga **solo cuando son relevantes**. Resuelven un problema que `AGENTS.md` no puede: cómo dar al agente conocimiento profundo de dominio sin inflar su contexto en cada turno.

## Qué vas a aprender

- Qué es una skill y en qué se diferencia del contenido de `AGENTS.md`.
- Cómo se descubren y activan las skills.
- La anatomía de un archivo de skill, con un ejemplo real de frontmatter.
- Una regla de decisión simple para *skill vs `AGENTS.md`*.

## El problema del exceso

`AGENTS.md` (o `CLAUDE.md`, `.cursorrules`, `AGENT.md` — el nombre varía, la idea no) se carga en el contexto en cada turno. Eso es exactamente lo que quieres para cosas que son *siempre* ciertas: el lenguaje, el gestor de paquetes, el comando de build, los innegociables del equipo.

¿Pero qué pasa con las 200 líneas que explican cómo escribir una migración de Flyway? ¿Las 80 líneas sobre tus convenciones custom de MapStruct? ¿El runbook para desplegar el worker legacy? Si lo metes todo en `AGENTS.md`, cada turno — incluso "renombra esta variable" — paga el coste en tokens. Peor: la atención del agente se diluye: con demasiada guía siempre activa, ninguna destaca.

!!! note "El contexto es un presupuesto, no un cubo"
    Cada token que cargas es un token que compite con la tarea real. El objetivo no es cargar *más*; es cargar *lo correcto*.

## Skills como conocimiento de carga perezosa

Una skill es un trozo autocontenido de guía que vive en disco y *solo* se trae al contexto cuando la tarea actual coincide con su descripción de trigger. Piénsalo como `import` para prompts: el módulo existe, pero solo se carga cuando hace falta.

Este es el modelo que usan las skills de Claude Code, los archivos de reglas de Cursor con triggers `globs:`, las instrucciones contextuales de Codex y herramientas de la comunidad como las convenciones de Aider repartidas en archivos.

## Anatomía de una skill

Una skill típica es un directorio con un `SKILL.md` (o equivalente) en la raíz. El archivo tiene dos partes: **frontmatter** que le dice al agente *cuándo* cargarla, y un **cuerpo** que le dice al agente *qué hacer* una vez cargada.

### Ejemplo de frontmatter de SKILL.md

```markdown
---
name: writing-flyway-migrations
description: >
  Use when creating, editing, or reviewing Flyway database migrations
  for the Postgres-backed services in this repo. Triggers on changes
  under src/main/resources/db/migration or mentions of "migration",
  "schema change", "ALTER TABLE", or "Flyway".
version: 1.2.0
---

# Writing Flyway migrations

## Naming
- File pattern: `V{yyyyMMddHHmm}__{snake_case_description}.sql`
- Never edit a migration that has been merged to `main`.

## Safety rules
- Always wrap destructive changes (DROP, ALTER COLUMN TYPE) in a
  two-step migration: add the new shape, backfill, then remove the old.
- Never use `CREATE INDEX` without `CONCURRENTLY` on tables > 100k rows.

## Required checklist
1. Migration runs cleanly on an empty DB.
2. Migration runs cleanly on a copy of production schema.
3. Rollback strategy documented in the PR description.

## Resources
- See `examples/V202401151200__add_user_email_index.sql` for a reference.
```

El campo `description` es la línea más importante del archivo. Es lo que el agente lee para decidir si cargar la skill. Las descripciones vagas ("ayuda con bases de datos") nunca disparan; las precisas ("usar al editar archivos bajo `db/migration` o cuando el usuario mencione Flyway") disparan de forma fiable.

## Descubrimiento: cómo se activan las skills

La mayoría de agentes hacen algo así en cada turno:

1. Leen la lista de skills disponibles (solo nombres y descripciones, barato).
2. Las comparan con la tarea actual — rutas tocadas, palabras clave del mensaje del usuario, salida reciente de herramientas.
3. Cargan el cuerpo de cualquier skill cuya descripción coincida claramente.

!!! tip "Escribe descripciones para el matcher, no para humanos"
    Incluye los nombres exactos de archivos, patrones glob, nombres de librerías y verbos que usa tu equipo. "Revisar PRs" es un peor trigger que "Usar al ejecutar `/review-pr`, cuando el usuario pegue una URL de PR de GitHub, o cuando se pida revisar un diff."

## Revelación progresiva: archivos referenciados bajo demanda

Una skill no es solo un único `SKILL.md`. Es un directorio, y ese directorio puede empaquetar material adicional: documentos de referencia más largos, archivos de ejemplo, plantillas, snippets de oro, un cheatsheet para una API oscura. El truco es que nada de eso se carga de entrada. El agente lee el cuerpo del `SKILL.md` y *solo si* decide que un archivo referenciado concreto es relevante para la tarea actual lo abre.

Esto es revelación progresiva: la skill anuncia lo que hay disponible, y el agente trae el material profundo bajo demanda. La huella se mantiene pequeña — una descripción, un cuerpo corto, una lista de punteros — hasta que una tarea realmente necesita el recetario de migraciones de 400 líneas o el volcado del schema legacy.

!!! note "Por qué importa"
    Compara con meter el recetario en `AGENTS.md`: pagarías el coste en tokens en *cada* turno, para cada tarea, para siempre. Con un archivo referenciado dentro de una skill, solo lo pagas cuando la skill dispara *y* el archivo referenciado es relevante. El mismo conocimiento, a una fracción del coste.

## Scripts ejecutables empaquetados

Una skill también puede traer scripts — `scripts/validate.sh`, `generate.py`, `scaffold.ts`, lo que sea — que el agente ejecuta directamente en el entorno local cuando la skill aplica. Esta es la alternativa in-process a MCP: sin servidor, sin protocolo, sin ciclo de vida. Solo código que vive junto a las instrucciones y que el agente invoca cuando es relevante.

Los scripts encajan muy bien con trabajo determinista: validar una migración generada, andamiar un módulo nuevo desde una plantilla, correr un codemod, transformar un payload, comprobar que un archivo cumple un schema. El cuerpo de la skill le dice al agente *cuándo* ejecutar el script y *cómo interpretar* la salida.

!!! tip "Skills con scripts vs MCP"
    Las skills con scripts empaquetados son más simples de distribuir — son solo archivos en el repo, versionados junto al código. Se ejecutan localmente, en el propio entorno del agente, con las mismas credenciales y el mismo filesystem. Los servidores MCP son más pesados (un proceso corriendo, un protocolo, auth) pero pueden envolver sistemas *remotos* y ser compartidos entre proyectos y clientes. Regla general: si el comportamiento es local y determinista, un script dentro de una skill es el camino más ligero. Si necesita llamar a un servicio remoto o ser reutilizado por varios agentes, tira de MCP.

### Ejemplo de SKILL.md con referencias y un script

```markdown
---
name: writing-flyway-migrations
description: >
  Use when creating or reviewing Flyway migrations under
  src/main/resources/db/migration, or on mentions of "migration",
  "ALTER TABLE", or "Flyway".
version: 1.3.0
---

# Writing Flyway migrations

Follow the two-step rule for destructive changes. Full cookbook in
`reference/destructive-changes.md` — open it when the change touches
an existing column or index.

For the file name and header, use the template in
`templates/migration.sql.tmpl`.

## Validation

After writing the migration, run:

    scripts/validate-migration.sh path/to/VXXXX__name.sql

The script checks naming, idempotency, and that `CONCURRENTLY` is
used on indexed tables > 100k rows. Fix any reported issue before
handing the change back to the user.
```

El directorio en disco se ve así:

```
writing-flyway-migrations/
├── SKILL.md
├── reference/
│   └── destructive-changes.md
├── templates/
│   └── migration.sql.tmpl
└── scripts/
    └── validate-migration.sh
```

Solo `SKILL.md` se carga cuando la skill dispara. El documento de referencia y la plantilla se abren bajo demanda; el script se ejecuta bajo demanda.

## Descubrimiento: automático vs explícito

Hay dos formas en las que una skill acaba en el contexto, y no son mutuamente excluyentes.

- **Descubrimiento automático.** El runtime del agente escanea las skills disponibles en cada turno, lee solo los nombres y las descripciones, y decide cuáles aplican a la tarea actual. Es lo que describe la sección del matcher más arriba. Es potente pero depende enteramente de la calidad de la implementación del agente — y de lo precisas que sean tus descripciones de trigger.
- **Referencia explícita.** Una definición de agente o subagente lista las skills a las que debe tener acceso siempre. Por ejemplo, un subagente dedicado de "migraciones" puede estar cableado para cargar siempre la skill `writing-flyway-migrations`, sin importar lo que piense el matcher. Más predecible, menos mágico, y útil cuando quieres comportamiento garantizado para un rol concreto.

!!! tip "Usa ambos"
    El descubrimiento automático es ideal para la larga cola de skills que aplican ocasionalmente a muchas tareas. La referencia explícita es ideal para las pocas skills que *definen* para qué sirve un subagente. Una configuración bien hecha usa los dos: automático para amplitud, cableado explícito para el camino crítico.

## Claude Code: skills invocables por el usuario

Claude Code añade un matiz que merece la pena destacar: las skills pueden ser invocadas no solo por el agente (descubiertas automáticamente o cableadas explícitamente) sino también **por el usuario directamente**, igual que invocarías un slash command. El usuario escribe el nombre de la skill, la skill se carga, y el agente trabaja con ella.

Esto difumina la línea entre "skill" y "comando". En la práctica, las skills están reemplazando progresivamente al antiguo concepto de slash command en Claude Code, porque una skill empaqueta todo lo que un comando solía hacer *y más*: instrucciones, archivos referenciados, scripts ejecutables y metadatos más ricos — todo en un único paquete versionado.

!!! note "Específico del proveedor"
    Este comportamiento invocable por el usuario es una decisión de Claude Code. Otros agentes pueden exponer las skills solo al runtime, o mantener los slash commands como un concepto completamente aparte. Revisa la documentación de tu agente antes de asumir que la misma ergonomía aplica.

## Ejemplos que compensan

- **`writing-migrations`** — reglas de schema, naming, cambios destructivos en dos pasos.
- **`reviewing-prs`** — tu rúbrica de revisión, las severidades, el tono.
- **`spring-boot-conventions`** — layout de paquetes, patrones de configuración, estilo de testing.
- **`incident-response`** — runbook para el agente on-call: dónde están los logs, a quién llamar, qué *no* tocar.

Cada uno de estos sería ruido en `AGENTS.md` para el 90% de las tareas. Como skills, son invisibles hasta que son útiles.

## La regla de decisión

!!! success "Skill vs AGENTS.md"
    - **¿Siempre cierto?** Pertenece a `AGENTS.md`. (Comando de build, lenguaje, reglas de lint.)
    - **¿A veces cierto?** Pertenece a una skill. (Reglas de migración, runbook de despliegue, refactorizaciones de nicho.)
    - **¿Cierto solo una vez?** Pertenece al prompt. No sobre-ingenierices.

## Modos de fallo

!!! warning "Las skills también se pudren"
    - **Skills que nunca disparan.** La descripción es demasiado vaga — el agente no puede saber cuándo cargarla. Arregla el trigger, no el cuerpo.
    - **Skills que siempre disparan.** La descripción es demasiado amplia y la skill se carga en cada turno. Eso es `AGENTS.md` con pasos extra.
    - **Skills obsoletas.** La librería se actualizó, las convenciones cambiaron, nadie tocó el archivo. Trata las skills como código: revísalas y versiónalas.

## Idea clave

!!! success "Idea clave"
    Las skills son cómo escalas conocimiento sin escalar contexto. Si una guía solo importa para el 10% de las tareas, no pertenece a `AGENTS.md` — pertenece a una skill con una descripción de trigger precisa.
