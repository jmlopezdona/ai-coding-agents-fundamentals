# 1. Contexto, herramientas, bucle — el modelo mental

Todo coding agent, sea del proveedor que sea, está hecho de las mismas tres piezas: **lo que ve** (contexto), **lo que puede hacer** (herramientas) y **cómo decide** (bucle). Si interiorizas estas tres, puedes razonar sobre cualquier producto de agentes sin reaprender el modelo mental.

## Qué vas a aprender

- Las tres piezas que tiene cualquier agente y cómo interactúan.
- A preguntar "¿qué ve el agente ahora mismo?" y "¿qué puede hacer realmente?" antes de depurar comportamientos extraños.
- Por qué la mayoría de fallos de agentes son fallos de contexto o de herramientas, no de modelo.

## Contexto: lo que el modelo ve en este turno

El contexto es todo lo que acaba en la entrada del modelo en un turno dado. No es "el proyecto" — el modelo no tiene una vista mágica de tu repo. Solo sabe lo que se ha colocado explícitamente en su ventana de contexto.

El contexto típico de un turno incluye:

- **System prompt** — fijado por el harness (Claude Code, Cursor, Codex, Aider...). Define el rol del agente, las herramientas disponibles, las reglas de seguridad.
- **Instrucciones del proyecto** — tu `AGENTS.md`, `CLAUDE.md`, `.cursorrules`, etc. (Capítulo 2.)
- **Mensaje del usuario** — lo que escribiste.
- **Historial de conversación** — mensajes previos del usuario, respuestas del modelo, tool calls, tool results.
- **Ficheros leídos hasta ahora** — cada resultado de `read_file` ya es parte del contexto.
- **Resultados de búsqueda, salidas de comandos** — lo mismo.

```
[system prompt]
[AGENTS.md]
[user] "arregla el test que falla en billing_test.py"
[assistant] read_file(billing_test.py)
[tool_result] <contenido de billing_test.py>
[assistant] read_file(billing.py)
[tool_result] <contenido de billing.py>
[assistant] run_shell("pytest billing_test.py")
[tool_result] FAILED ... AssertionError: 10.05 != 10.04
[assistant] ...pensando el arreglo
```

!!! note "Las ventanas de contexto son finitas"
    Incluso con ventanas de 200k o 1M tokens, el contexto es un presupuesto. Leer un fichero de 50k líneas se come ese presupuesto. Los buenos agentes (y los buenos usuarios) leen estrechamente: buscar primero, abrir la rebanada relevante, no el fichero entero.

## Herramientas: los verbos que tiene el agente

Las herramientas son los verbos del agente. Sin ellas, el modelo solo puede hablar. Con ellas, puede actuar. Un harness típico de coding agent expone algún subconjunto de:

- **Sistema de ficheros**: `read_file`, `write_file`, `edit_file`, `list_dir`, `glob`.
- **Búsqueda**: `grep`, búsqueda semántica, lookup de símbolos.
- **Ejecución**: `run_shell` (a menudo en sandbox), `run_tests`.
- **Red**: `fetch_url`, `web_search` (a veces).
- **VCS**: `git_diff`, `git_commit` (a veces).
- **Servidores MCP**: herramientas arbitrarias del usuario (bases de datos, Jira, APIs internas).

Distintos productos exponen distintas herramientas. Cursor se apoya en su propio buscador indexado. Aider usa git agresivamente. Claude Code expone un conjunto pequeño y afilado con permisos explícitos. Codex CLI ejecuta todo en sandbox por defecto. Los *nombres* difieren; las *categorías* son casi universales.

!!! warning "Lo que el agente no tiene, no puede hacerlo"
    Si tu agente no tiene `run_shell`, no puede correr tus tests por más claro que se lo pidas. Si no tiene acceso a la web, no puede consultar la documentación actual. Conocer el toolset es la mitad de conocer al agente.

## El bucle: cómo se toman las decisiones

El bucle es engañosamente simple:

```
while not done:
    response = model(context)
    if response.has_tool_calls:
        results = harness.execute(response.tool_calls)
        context.append(response, results)
    else:
        return response   # respuesta final al usuario
```

Eso es todo. El modelo no tiene un planificador oculto. No tiene un debugger vigilándolo. En cada turno ve el contexto acumulado y decide: llamar a otra herramienta, o parar y responder. La "inteligencia" de la ejecución de un agente es la suma de muchas pequeñas decisiones locales, cada una tomada sobre el contexto que existe en ese momento.

### Un turno individual vs una tarea completa

Un **turno** es una ronda de salida del modelo. Una **tarea** es la secuencia completa de turnos desde tu prompt hasta la respuesta final del agente. Una tarea trivial ("renombra esta variable") puede ser de 2 turnos. Una tarea real ("añade una funcionalidad con tests") puede ser de 30. Cada turno es independiente en el sentido de que el modelo vuelve a leer todo el contexto desde cero — no tiene memoria persistente entre turnos más allá de lo que esté en ese contexto.

## Diagnóstico: ¿fue contexto, herramientas o bucle?

Cuando un agente hace algo absurdo, resiste el impulso de reescribir tu prompt inmediatamente. Recorre las tres capas en orden.

!!! tip "Las tres preguntas de diagnóstico"
    1. **Contexto** — ¿Vio lo que tenía que ver? ¿Leyó el fichero que se suponía que debía editar? ¿Tenía tu `AGENTS.md`? ¿Empujaron los turnos antiguos la información relevante fuera de la ventana?
    2. **Herramientas** — ¿Tenía el verbo que necesitaba? Si "adivinó" código en vez de leer el fichero, quizá no tenía un `read_file` operativo. Si no corrió los tests, quizá `run_shell` está deshabilitado.
    3. **Bucle** — ¿Terminó demasiado pronto (se rindió antes de verificar)? ¿Entró en un bucle infinito por una herramienta inestable? ¿Tropezó con un error de herramienta y se rindió en vez de reintentar?

Algunos ejemplos concretos:

- *"Alucinó una función que no existe."* Casi siempre contexto: nunca leyó el fichero donde vive la función.
- *"Dice que el test pasa pero no pasa."* Casi siempre bucle o herramientas: nunca corrió el test, o corrió uno distinto.
- *"Editó el fichero equivocado."* Contexto: no buscó primero; adivinó la ruta.

La ingeniería de prompts es la *última* palanca, no la primera. Arreglar el contexto (apuntarle a los ficheros correctos, recortar ruido, añadir una línea al `AGENTS.md`) y arreglar las herramientas (darle acceso a shell, darle un test runner real) resuelve la inmensa mayoría del mal comportamiento de un agente.

## Idea clave

!!! tip "Idea clave"
    Cuando algo va mal, hazte las tres preguntas en orden: ¿vio lo que tenía que ver? ¿tenía las herramientas adecuadas? ¿el bucle terminó limpiamente? Esto bate al prompt-tweaking nueve de cada diez veces.
