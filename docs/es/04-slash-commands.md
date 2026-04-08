# 4. Slash commands y prompts reutilizables

Cuando te encuentres tecleando el mismo prompt por quinta vez, es momento de cristalizarlo como un slash command (`/commit`, `/review-pr`, `/test`). Los comandos son cómo conviertes prompts puntuales en convenciones de equipo — un vocabulario compartido en el que todos, incluido el agente, están de acuerdo.

## Qué vas a aprender

- Cuándo merece la pena que un prompt se convierta en comando.
- La anatomía de un buen comando: alcance, entradas, salida esperada.
- Dónde viven los comandos y cómo componen valor en un equipo.
- Los modos de fallo que convierten comandos útiles en minas ocultas.

## La regla de la "quinta vez"

La primera vez que escribes un prompt, estás explorando. La segunda y la tercera, lo estás refinando. Para cuando has tecleado por quinta vez "mira el diff, escribe un mensaje de commit convencional, agrupa cambios relacionados y no incluyas líneas de co-autor", ya no estás explorando — tienes una receta. Las recetas viven en un archivo, no en tu memoria muscular.

!!! tip "Heurística"
    Si dos compañeros escribirían versiones sutilmente distintas del mismo prompt y obtendrían resultados sutilmente distintos, ese prompt está pidiendo a gritos ser un comando.

La misma lógica aplica a `.claude/commands/` de Claude Code, a las reglas-como-comandos de Cursor, a los prompts de Codex o al archivo de convenciones de Aider. La herramienta cambia; la disciplina no.

## Anatomía de un buen comando

Un buen slash command se comporta como una pequeña función pura:

- **Un nombre claro** — `/commit`, no `/haz-lo-de-git`.
- **Una sola responsabilidad** — un verbo, un resultado.
- **Entradas explícitas** — qué archivos, qué argumentos, qué supuestos.
- **Una salida predecible** — el usuario debería saber qué obtendrá antes de ejecutarlo.

### Ejemplo de archivo de comando

Aquí tienes un comando `/commit` mínimo, escrito en el formato que esperan la mayoría de agentes (Markdown con frontmatter y cuerpo):

```markdown
---
description: Stage relevant changes and write a conventional commit message.
argument-hint: "[optional scope]"
---

You are preparing a commit for the current repository.

1. Run `git status` and `git diff` to see what changed.
2. Group related changes; ignore generated files and lockfiles unless they
   are the point of the change.
3. Write a Conventional Commits message:
   - type(scope): short summary (max 72 chars)
   - blank line
   - body explaining *why*, not *what*
4. Do NOT add Co-Authored-By lines.
5. Show me the message and the `git add` plan before committing.

Scope hint from the user: $ARGUMENTS
```

Fíjate en lo que este comando *no* hace: no hace push, no decide si hacer amend, no abre un PR. Eso son comandos separados.

## Dónde viven los comandos

La mayoría de agentes soportan dos ámbitos:

- **Por usuario** (p. ej. `~/.claude/commands/`) — tu caja de herramientas personal.
- **Por repositorio** (p. ej. `.claude/commands/` versionado en git) — la caja de herramientas compartida del equipo.

!!! note "Por repo gana en equipos"
    Los comandos personales son geniales para `/journal` o `/explain-this-error`. Pero cualquier cosa que toque el código — commits, PRs, migraciones, ejecución de tests — debería vivir en el repo para que los revisores puedan leerlo, cambiarlo y confiar en él.

## Ejemplos que vale la pena copiar

| Comando | Responsabilidad |
|---|---|
| `/commit` | Preparar y escribir un commit convencional. |
| `/review-pr <número>` | Traer el diff de un PR y revisarlo contra los estándares del proyecto. |
| `/test` | Ejecutar el comando de test correcto para los archivos tocados y resumir los fallos. |
| `/explain <ruta>` | Producir una explicación arquitectónica corta de un archivo o módulo. |
| `/migrate <de> <a>` | Aplicar una refactorización conocida (p. ej. JUnit 4 → JUnit 5). |

Cada uno cabe en una pantalla. Cada uno tiene un solo trabajo.

## Modos de fallo

!!! warning "Las cuatro formas en que los comandos se pudren"
    1. **Scope creep.** `/commit` empieza haciendo push, luego abre PRs, luego corre tests. Divídelo.
    2. **Comandos obsoletos.** Nadie actualizó `/test` cuando cambió el test runner. Trata los comandos como código: revísalos, actualízalos, bórralos.
    3. **Contexto oculto.** Un comando que inyecta silenciosamente 4.000 tokens de "estilo de la casa" hace los resultados irreproducibles. Sé explícito sobre lo que carga.
    4. **Nombres mágicos.** `/do-it` no es un comando, es una moneda al aire.

## Versionado y revisión

Los comandos son código. Viven en git, pasan por pull requests y merecen mensajes de commit que expliquen por qué cambiaron. Cuando `/review-pr` produzca peores revisiones el próximo trimestre, querrás que `git blame` te diga quién suavizó la rúbrica y por qué.

Una regla útil: cualquier cambio a un comando del repo debería ser revisado por al menos una persona que *use* ese comando, no solo por la que lo escribió. Los comandos moldean el comportamiento de todo el equipo — merecen el mismo cuidado que una librería compartida.

## Idea clave

!!! success "Idea clave"
    Un slash command es una llamada a función para prompts. Aplica la misma higiene que aplicarías a cualquier función: una sola responsabilidad, interfaz clara, code review, y un nombre del que no te avergonzarías leer en voz alta en un standup.
