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
