# 11. Memoria y persistencia entre sesiones

Los agentes tienen varias formas de "recordar" cosas: la conversación en vivo, ficheros de memoria persistente, planes, listas de tareas y el propio repositorio. Cada una tiene una vida útil distinta, un público distinto y un modo de fallo distinto. Tratarlas como intercambiables es uno de los errores más comunes al empezar, y una de las mayores fuentes de pérdida silenciosa de calidad cuando un equipo escala.

## Qué vas a aprender

- Las cuatro capas de persistencia, más la quinta capa más amplia de artefactos de conocimiento del proyecto.
- Por qué volcarlo todo en "memoria" es mala idea.
- Un árbol de decisión para saber dónde pertenece cada pieza de información.
- Cómo escribir un fichero de memoria del que no te arrepientas en tres meses.

## Las cuatro capas de persistencia

Piensa en la memoria del agente como una pila de capas, cada una con una vida útil más corta que la que tiene debajo.

### Capa 1: Contexto en sesión (la ventana de contexto)

Es todo lo que el modelo "ve" en este momento: el system prompt, las definiciones de herramientas, el historial de la conversación, los ficheros que ha leído y las salidas de las herramientas. Vive solo durante la tarea actual. Cuando la sesión acaba (o la ventana se compacta), desaparece.

Úsala para: el problema que estás resolviendo ahora mismo, razonamientos intermedios, contenido de ficheros que el agente acaba de abrir, planes en borrador.

!!! warning "El contexto no es gratis"
    Cada token que metes en el contexto cuesta latencia, dinero y atención. Una ventana de 200k no significa que debas llenarla. Los agentes razonan peor en el último 20% de una ventana llena que en el primer 20%.

### Capa 2: Planes y listas de tareas (memoria de sesión)

Herramientas como la lista de todos de Claude Code, el panel de planes de Cursor o el historial de chat de Aider dan al agente un cuaderno temporal que sobrevive dentro de una sesión pero no más allá. Aquí vive el trabajo multi-paso: "estoy en el paso 3 de 7, el paso 2 falló por X, este es el reintento."

Úsala para: la tarea multi-paso actual, decisiones en curso, cosas a las que quieres que el agente vuelva antes de acabar la sesión.

### Capa 3: Memoria de proyecto (AGENTS.md / CLAUDE.md)

Esta es la primera capa realmente duradera. Ficheros como `AGENTS.md`, `CLAUDE.md`, `.cursorrules` o `.aider.conf.yml` viven en el repo y se cargan automáticamente al inicio de cada sesión. Describen *el proyecto*: cómo ejecutar los tests, el estilo de arquitectura, las convenciones de nombres, qué directorios están prohibidos, cómo desplegar.

Al vivir en el repo, se comparten con todo el equipo y se versionan con el código. Aquí es donde pertenece el conocimiento a nivel de proyecto — no en la memoria privada de nadie.

### Capa 4: Memoria de usuario persistente

La mayoría de los agentes de código también ofrecen una memoria privada por usuario — el `~/.claude/CLAUDE.md` de Claude Code, las reglas de usuario de Cursor, las instrucciones de usuario de Codex. Esta capa sobrevive entre proyectos y sesiones. Es para hechos sobre *ti*: "prefiero pnpm antes que npm", "trabajo en macOS con fish shell", "no sugieras emojis en mensajes de commit".

!!! tip "La regla de oro"
    Si un compañero también se beneficiaría de saberlo, pertenece al repo (capa 3), no a tu memoria personal (capa 4).

### Capa 5: Artefactos de conocimiento del proyecto (bajo demanda)

Las cuatro capas anteriores responden a "¿qué sabe siempre el agente?". Hay un cuerpo de conocimiento mucho más amplio que pertenece al proyecto pero que *no* debe vivir en `AGENTS.md` porque es demasiado grande y demasiado raras veces necesario en cada turno. Esta es la **capa de conocimiento bajo demanda**: artefactos largos que el agente lee solo cuando son relevantes para la tarea actual.

Artefactos típicos:

- **Documentación funcional** — specs, requisitos, historias de usuario cerradas.
- **Documentación de arquitectura** — diagramas C4, modelos de dominio, contratos entre servicios.
- **ADRs** (Architecture Decision Records) — el *por qué* histórico de las decisiones.
- **Planes de ejecución** — roadmaps, planes de migración, fases en curso.
- **Runbooks operacionales** — qué hacer cuando X falla, cómo desplegar Y.
- **Post-mortems e incident reports** — el aprendizaje histórico.
- **Glosarios y modelos de dominio** — el vocabulario del negocio.
- **Documentación de APIs internas** — specs OpenAPI, esquemas de eventos.

Por qué importa: cuando el agente está implementando "añadir un endpoint para X", el contexto técnico vive en el código, pero el contexto de *por qué* y *cómo encaja* vive en estos documentos. Sin ellos, el agente toma decisiones razonables localmente pero globalmente incoherentes con el sistema.

Esto es distinto de las capas anteriores:

- **No es `AGENTS.md`** — ese fichero es corto y siempre activo. Estos artefactos son largos y bajo demanda.
- **No es una skill** — las skills son procedimentales ("cómo hacer X"). Estos son descriptivos ("qué es X y por qué").

Hay tres buenas formas de exponerlos al agente:

1. **Ficheros versionados en el repo** (`docs/`, `adr/`) — el agente los lee con su tool de filesystem. Mejor cuando los artefactos pertenecen al proyecto y deben viajar con el código.
2. **Skills que los referencian** como ficheros de soporte vía progressive disclosure (capítulo 5) — el `SKILL.md` se carga siempre, el fichero de referencia largo solo cuando la skill se activa.
3. **Un servidor MCP** sobre el wiki/CMS corporativo (Confluence, Notion, SharePoint — capítulo 8) — cuando los artefactos viven fuera del repo y se comparten entre proyectos.

La decisión: **en el repo cuando son del proyecto y versionables; en MCP cuando son corporativos y compartidos.**

La pieza clave — y la parte que la mayoría de equipos se pierden — es que estos artefactos tienen que ser **descubribles desde algún sitio que el agente ya lea**. Un puntero corto en `AGENTS.md` suele bastar:

```markdown
## Project knowledge

- Architecture overview: `docs/architecture/overview.md`
- ADRs: `docs/adr/` (read the index first)
- Payments domain — read `docs/adr/0007-payments.md` before touching `app/payments/`
- Runbooks: `docs/runbooks/`
```

!!! warning "Dos anti-patrones que evitar"
    **Anti-patrón 1: volcarlo todo en `AGENTS.md`.** Pegar el documento de arquitectura inline revienta la ventana de contexto en cada turno. Apunta a él, no lo pegues.

    **Anti-patrón 2: tener los artefactos pero nunca enlazarlos.** Si `AGENTS.md` no le dice al agente que `docs/adr/` existe, el agente no lo encontrará. Es como si el artefacto no existiera.

## Un árbol de decisión

Cuando quieras que el agente "recuerde" algo, pregúntate:

1. **¿Es relevante solo para la tarea actual?** → Déjalo en el contexto. No hagas nada.
2. **¿Es relevante para el resto de esta sesión?** → Ponlo en el plan o la lista de todos.
3. **¿Es un hecho corto y siempre cierto del proyecto?** → `AGENTS.md` en el repo.
4. **¿Es conocimiento largo del proyecto (arquitectura, ADRs, runbooks)?** → Capa 5 — documento versionado en el repo (o MCP si vive en la wiki), con un puntero desde `AGENTS.md`.
5. **¿Es una preferencia personal sobre cómo trabajas *tú*?** → Fichero de memoria de usuario.
6. **¿Es una observación puntual que no volverás a necesitar?** → No lo escribas en ningún sitio.

## Un ejemplo de fichero de memoria

Aquí tienes un `AGENTS.md` pequeño y bien acotado — del tipo que realmente ayuda:

```markdown
# Project: billing-api

## Stack
- Python 3.12, FastAPI, SQLAlchemy 2.x, Postgres 16.
- Tests: pytest + pytest-asyncio. Run with `make test`.
- Lint/format: `make lint` (ruff + mypy strict).

## Conventions
- All new endpoints go under `app/api/v1/`.
- Database models live in `app/models/`. Never import them from `app/api/`
  directly — always go through a repository in `app/repositories/`.
- Money is always `Decimal`, never `float`. Currency is a separate field.

## Definition of done
- `make lint test` passes.
- New endpoints have a request/response schema and an integration test.
- No changes to `alembic/versions/` without an accompanying migration note.

## Off-limits
- `infra/` and `.github/` — ask before touching.
```

Fíjate en lo que *no* aparece: sin historial de bugs pasados, sin changelog, sin opiniones personales, sin una explicación de 30 líneas sobre REST. Es corto, escaneable, y cada línea se gana su sitio.

## El anti-patrón: la memoria como cuaderno

El modo de fallo más común es usar la memoria como un diario. Cada vez que algo sale ligeramente mal, se añade una línea: "Recuerda que el 2026-03-14 el test de X falló por Y." Al cabo de un mes, el fichero de memoria tiene 2000 líneas de anécdotas viejas que empujan fuera las instrucciones que el agente realmente necesita.

La memoria no es un log. Es un conjunto *curado* de hechos no obvios. Si una nota no va a importar la semana que viene, no pertenece allí. Poda sin piedad — borrar líneas de un fichero de memoria es casi siempre una mejora neta.

!!! success "Idea clave"
    Cada capa de persistencia tiene su trabajo. El repo (`AGENTS.md`) es la única capa que tus compañeros pueden ver — cualquier cosa importante para el *equipo* pertenece ahí, no a la memoria privada de un developer concreto. En la duda, prefiere la memoria más corta y el repo más largo.
