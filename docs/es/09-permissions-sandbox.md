# 9. Permisos, sandbox y modos de ejecución

Todo producto de agentes tiene mandos para "¿cuánto puede hacer sin preguntar?" Equivócate en cualquiera de las dos direcciones y lo sufrirás: demasiado restrictivo y el agente es inútil, demasiado permisivo y te arrepentirás la primera vez que ejecute `rm -rf` o haga push a `main`.

## Qué vas a aprender

- Los modos de ejecución que ofrecen la mayoría de agentes (plan, ask, auto-accept).
- Cómo elegir autonomía por herramienta, no por sesión.
- Un default seguro para un equipo que empieza.

## El espectro de autonomía

Todo harness — Claude Code, Cursor, Codex, Aider, Continue — expone aproximadamente el mismo dial, aunque las etiquetas cambien:

- **Modo plan** — el agente puede leer y pensar, pero no editar ficheros ni ejecutar comandos. Produce un plan que revisas antes de que nada cambie. Ideal para explorar un código nuevo o acotar un refactor.
- **Modo ask (por defecto)** — cada tool call que muta estado (escritura, bash, red) te pide aprobación. Lento pero seguro.
- **Modo accept-edits** — las ediciones de ficheros se aplican solas, pero los comandos y llamadas de red siguen preguntando. Un buen término medio cuando ya confías en el agente para una tarea.
- **Modo bypass / auto-completo** — el agente se ejecuta sin confirmación. Útil en sandboxes y CI. Peligroso en tu portátil.

!!! note "Nombres distintos, la misma idea"
    Claude Code los llama plan/default/accept-edits/bypass. Cursor tiene agent modes. Codex CLI tiene `suggest`/`auto-edit`/`full-auto`. La semántica se mapea limpiamente entre productos.

## Por qué auto-completo para todo es una trampa de principiante

La primera vez que pones un agente en auto-completo sobre código real, se siente mágico. La décima vez, ha hecho force push a una rama, borrado tu virtualenv o abierto doce PRs en el repo equivocado. El coste de un clic extra de confirmación es un segundo. El coste de un comando destructivo no deseado puede ser horas de recuperación y confianza perdida.

Auto-completo es una herramienta para entornos donde los errores son baratos: contenedores efímeros, ramas desechables, directorios scratch. No es un default para tu working tree principal.

## Permisos por herramienta

El mejor modelo mental no es "cuánto confío en esta sesión" sino "cuánto confío en esta capacidad." Los permisos deberían ponerse por herramienta:

- **Permitir siempre:** `Read`, `Grep`, `Glob`, `LS`. Las operaciones de solo lectura no pueden corromper el estado. Dejar al agente grepear libremente lo hace drásticamente más útil.
- **Auto-permitir con scope:** `Edit`, `Write` dentro del directorio del proyecto. Bloquea escrituras fuera del repo.
- **Allowlist:** `Bash`. Permite comandos concretos (`pnpm test`, `pnpm lint`, `git status`, `git diff`) sin confirmación; pregunta para cualquier otra cosa.
- **Preguntar siempre:** git destructivo (`push`, `reset --hard`, `checkout .`), instalaciones de paquetes, llamadas de red a hosts desconocidos, cualquier cosa que toque `~/.ssh` o `.env`.

Una allowlist por herramienta te da casi toda la velocidad del auto-completo con casi nada del radio de explosión.

## Sandboxing y compromisos

Los permisos deciden qué puede intentar el agente. Los sandboxes deciden qué pasa cuando lo intenta igual. Las opciones habituales:

- **Git worktrees.** El aislamiento más barato. El agente trabaja en una copia de trabajo separada del mismo repo, así un borrado accidental o un mal commit no tocan tu checkout principal. Rápido, local, sin overhead de contenedor.
- **Contenedores (Docker, devcontainers).** La vista del filesystem y la red del agente están acotadas a un contenedor. Aislamiento fuerte al coste de un arranque más lento y un loop editar/depurar más incómodo.
- **VMs efímeras o sandboxes en la nube.** Servicios como GitHub Codespaces, Daytona, E2B o sandboxes alojados por el proveedor. Mejor aislamiento, mayor latencia, coste extra. El encaje correcto para agentes autónomos de larga duración.
- **Sin sandbox.** Ejecutando directamente sobre tu portátil. Feedback máximo, aislamiento cero — apóyate fuerte en permisos y hooks.

!!! warning "Los sandboxes no son gratis"
    Cada capa de aislamiento añade fricción: hot reloads más lentos, depurador más difícil de adjuntar, desajustes de rutas confusos. Elige el sandbox más ligero que cubra tu riesgo real. Para el trabajo del día a día en un repo de confianza, un worktree más buenos hooks le gana a un contenedor pesado.

## Defaults seguros para el primer mes

Si estás montando un agente para un equipo que nunca ha usado uno:

1. Empieza en **modo ask**, por herramienta.
2. Auto-permite `Read`, `Grep`, `Glob`, `LS`.
3. Auto-permite `Edit` y `Write` solo dentro del directorio del proyecto.
4. Allowlist de comandos `Bash` seguros: tu test runner, linter, formateador, `git status`, `git diff`, `git log`.
5. Exige confirmación para cualquier `git push`, cualquier `rm`, cualquier instalación de paquetes, cualquier fetch de red.
6. Corre dentro de un git worktree para que "deshacer" sea `git worktree remove`.
7. Habilita los hooks del primer día del capítulo 7 para que formato y escaneo de secretos pasen automáticamente.

Tras dos semanas de uso real, sabrás qué prompts apruebas siempre — promuévelos a la allowlist. También sabrás qué herramientas sigue usando mal el agente — aprétalas.

### Cuándo relajar — deliberadamente

Relaja permisos porque has aprendido algo, no porque estés frustrado. Buenas razones:

- "He aceptado este comando 50 veces seguidas sin sorpresas" → a la allowlist.
- "Esta tarea corre en un sandbox que puedo tirar" → modo bypass está bien ahí.
- "CI ejecuta el agente sobre una rama desechable" → auto-completo es lo correcto.

Malas razones:

- "Las confirmaciones son molestas y tengo prisa."
- "Es un cambio pequeño, qué podría salir mal."

!!! tip "Promueve, no bypasses"
    Cuando una confirmación se ponga pesada, añade el comando a tu allowlist. No conmutes toda la sesión a modo bypass — eso pierde la seguridad para todas las demás herramientas.

!!! success "Idea clave"
    Elige autonomía por herramienta, no por sesión. El coste de una confirmación es ínfimo; el coste de un `git push --force` no deseado es enorme.
