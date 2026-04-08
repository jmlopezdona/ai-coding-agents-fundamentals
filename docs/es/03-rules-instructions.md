# 3. Rules e instructions: el mismo problema, distintos formatos

Todo equipo acaba queriendo que el agente *simplemente deje* de hacer algo — deje de inventar rutas de import, deje de usar `any`, deje de reformatear ficheros que no se le pidió tocar. Reescribir "por favor no hagas X" en cada turno no es una solución. Ese es el problema que resuelven las rules e instructions: guía persistente que el agente lleva a cada tarea sin que tengas que recordársela.

## Qué vas a aprender

- Qué son realmente las "rules" e "instructions", y en qué se diferencian de un prompt.
- Por qué algunas herramientas ofrecen un sistema de rules dedicado y otras lo funden en `AGENTS.md`.
- Cómo lo manejan los principales coding agents, uno al lado del otro.
- La diferencia entre rules, `AGENTS.md` y skills — y la trampa de duplicarlas.

## Rules, en una frase

Una rule es **guía siempre-activa (o activa-por-scope)** que el agente ve sin que se la pidas. "Usa `Result<T, E>` en lugar de lanzar excepciones." "Nunca edites ficheros bajo `generated/`." "Todos los componentes React son funciones con hooks." Son cortas, declarativas, y viajan con el repo (o con tu perfil de usuario) en vez de vivir en un prompt puntual.

Las rules *no* son un prompt — un prompt es una petición única. Las rules *tampoco* son una skill — una skill es conocimiento bajo demanda que el agente carga cuando cree que es relevante. Las rules son la radiación de fondo: siempre están ahí, moldeando el comportamiento mencione o no la tarea actual.

## ¿Por qué un mecanismo aparte?

Aquí está lo interesante: algunos agentes exponen un sistema de rules dedicado (Cursor, Windsurf, Cline, Continue), y otros no (Claude Code, Codex CLI). Los dos grupos resuelven el **mismo** problema de fondo — dar al agente guía siempre-activa sin que el usuario tenga que reescribirla — simplemente discrepan sobre la forma de la solución.

El bando de rules dedicadas argumenta: las rules son algo distinto del briefing del proyecto. Merecen sus propios ficheros, su propia lógica de activación (por glob, por tipo de tarea, siempre-activa) y su propio flujo de review. Poder escribir `react.mdc` que solo aplique a ficheros `*.tsx` es genuinamente útil.

El bando de fundirlo-en-AGENTS.md argumenta: cada mecanismo extra es otro sitio al que mirar cuando el comportamiento se tuerce. Un fichero, una fuente de verdad, sin confusiones de "¿lo metí en rules o en CLAUDE.md?".

Los dos son defendibles. Ninguno está mal. Pero si saltas entre herramientas, necesitas saber qué modelo usa cada una.

## Cómo lo manejan las herramientas principales

| Herramienta | Fichero | Alcance | ¿Condicional? |
|-------------|---------|---------|---------------|
| Cursor | `.cursor/rules/*.mdc` | Repo + usuario | Sí — frontmatter `alwaysApply`, `globs`, `description` |
| GitHub Copilot | `.github/copilot-instructions.md`, `.github/instructions/*.instructions.md`, settings de usuario | Repo (global), repo (glob-scoped), usuario | Sí — vía glob `applyTo` en ficheros de instrucciones |
| Windsurf | `.windsurfrules` + reglas globales | Repo + usuario | Limitado |
| Cline | Fichero `.clinerules` o directorio `.clinerules/` | Repo | Activación por fichero |
| Continue | Rules en config + `.continuerules` | Repo + usuario | Parcial |
| Aider | `CONVENTIONS.md` vía `--read` | Repo (carga explícita) | No — eliges cuándo cargarlo |
| Claude Code | `CLAUDE.md` / `AGENTS.md` | Repo + usuario | No — un único fichero siempre-activo |
| Codex CLI | `AGENTS.md` | Repo + usuario | No — un único fichero siempre-activo |

La columna que más importa es la última. Las **rules condicionales** — "aplica esto solo cuando el agente toque TypeScript" — son la razón principal por la que un sistema dedicado se gana el sueldo. Si no necesitas eso, `AGENTS.md` hace el trabajo.

## Rules vs AGENTS.md vs skills

Estos tres mecanismos se confunden constantemente. La distinción clave es **cuándo el contenido está en contexto**:

| Mecanismo | Cuándo se carga | Contenido típico |
|-----------|-----------------|------------------|
| Rules | Siempre, o bajo match de glob/scope | Frases cortas de do/don't, convenciones |
| `AGENTS.md` | Siempre (cada turno) | Briefing de proyecto, comandos, convenciones, guardrails |
| Skills | Bajo demanda, cuando el agente decide que aplican | Conocimiento how-to, procedimientos, material de referencia |

- Las **rules** son tensas y declarativas: "siempre", "nunca", "prefiere".
- **`AGENTS.md`** es el briefing del proyecto: qué es este repo, cómo se ejecuta, qué importa. Las rules suelen ser *un subconjunto* de lo que vive en `AGENTS.md`.
- Las **skills** son pull, no push: el agente tira de ellas cuando la tarea lo pide y se quedan fuera del contexto el resto del tiempo.

## La trampa de la duplicación

El modo de fallo más común es escribir la misma convención en los tres sitios. "Usa `Result<T, E>`" termina en `.cursor/rules/errors.mdc`, en `AGENTS.md` y en una skill `error-handling`. Ahora tienes tres copias que mantener sincronizadas, y cuando una se desvía el agente recibe señales contradictorias.

!!! warning "Una casa por convención"
    Elige un único sitio para cada rule. Si está en `AGENTS.md`, sácala de la carpeta de rules. Si está en una skill, no la repitas en `AGENTS.md`. La duplicación parece "ser minucioso" y se comporta como un bug.

Una heurística simple: si la guía aplica a *todas* las tareas de este repo, ponla en `AGENTS.md`. Si solo aplica a un tipo de fichero o a un subdirectorio, usa una rule con scope (si tu agente lo soporta) o acepta que una línea corta en `AGENTS.md` ya es suficiente.

## Por qué algunos agentes deliberadamente no tienen sistema de rules

Claude Code y Codex CLI tomaron una decisión consciente: sin mecanismo de rules aparte, todo va a `CLAUDE.md`/`AGENTS.md`. El argumento es que **la fragmentación es el enemigo real** — cuantos más sitios de los que el agente saque instrucciones, más difícil razonar sobre por qué hizo lo que hizo.

El trade-off es real. Pierdes rules con scope por glob. Pierdes la posibilidad de tener un set "solo TypeScript" que se active en silencio cuando toca. A cambio, ganas un único fichero que auditar, un único fichero que revisar en PRs, y una única respuesta a "¿de dónde salió ese comportamiento?".

Hay una segunda razón más profunda: es un enfoque más **agéntico** que programático. En vez de declarar "si el fichero matchea `**/*.tsx`, aplica estas reglas", levantas un **subagente especializado** para ese dominio (p. ej. un subagente `frontend-react`) y le das sus propios skills, su propio briefing y sus propias herramientas. La decisión de enrutado se mueve de un glob estático al propio agente: cuando la tarea es de React, delega al subagente de React; cuando es de infra, delega al subagente de infra. Cada subagente carga sólo lo que necesita, bajo demanda. Las rules con scope por glob intentan inyectar el contexto correcto en base a rutas de ficheros; los subagentes + skills inyectan el contexto correcto en base a la *tarea*. Esto último compone mejor a medida que el proyecto crece.

Si tu proyecto es pequeño-a-mediano y mayormente de un solo lenguaje, el enfoque de fichero único gana. Si tienes un monorepo polígloto, puedes optar por un sistema de rules dedicado con scope por glob, o tirar de subagentes especializados — ambos resuelven el mismo problema, pero desde extremos distintos.

## Ejemplo: una rule pequeña de Cursor con scope

```markdown
---
description: TypeScript conventions for the web app
globs: ["apps/web/**/*.ts", "apps/web/**/*.tsx"]
alwaysApply: false
---

- Prefer `type` over `interface` unless declaration merging is needed.
- Never use `any`. If the type is truly unknown, use `unknown` and narrow.
- Components are function components; no classes.
- Props types live next to the component, named `<Component>Props`.
- Do not add new runtime dependencies without asking.
```

Fíjate en la forma: un frontmatter que le dice al agente *cuándo* aplicar la rule, y un cuerpo corto, declarativo y escaneable. Ese es el formato que un sistema de rules premia.

## Idea clave

!!! tip "Idea clave"
    Rules, `AGENTS.md` y skills son tres respuestas a la misma pregunta: *¿cómo le doy a este agente guía persistente sin reescribirla?* Elige el mecanismo que bendice tu herramienta, elige una casa por convención, y resiste la tentación de copiar la misma rule en tres ficheros "por si acaso".
