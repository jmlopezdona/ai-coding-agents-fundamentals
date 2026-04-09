# 7. Garantías deterministas: hooks, agentes especializados y workflows externos

Hay pasos que tienen que pasar — formato, lint, type checks, escaneo de secretos, tests — y "le pedí al agente que se acordara" no es una estrategia. Este capítulo va sobre los mecanismos que hacen que esos pasos sean fiables: **hooks** cuando el harness los ofrece, **agentes especializados** cuando no (o cuando hace falta juicio), y **workflows externos** (CI, GitHub Actions, git hooks) como red de seguridad. En la práctica los combinas en capas.

Esos mismos mecanismos también devuelven los fallos a quien esté iterando: el propio agente en el **inner loop** (para que se autocorrija sin esperar a un humano), y el humano o un agente upstream en el **outer loop** (para que la colaboración se base en señal real, no en confianza).

## Qué vas a aprender

- La diferencia entre "pedir al agente que haga X" y "hacer que algo distinto del agente lo imponga".
- Tres mecanismos para garantías deterministas, y cuándo encaja cada uno.
- Cómo esos mecanismos devuelven feedback al inner loop (autocorrección del agente) y al outer loop (revisión humano/agente).
- Qué herramientas soportan qué mecanismo hoy.
- Hooks habituales que merece la pena montar el primer día y cómo depurarlos.
- Por qué los hooks actúan como **sensores**, no solo como automatizadores.

## Las instrucciones son esperanzas, las garantías son configuración

Todo equipo que adopta un agente de código pasa por el mismo ciclo. Alguien escribe en `AGENTS.md` o `CLAUDE.md`: "Ejecuta siempre `pnpm lint` después de editar TypeScript." Durante un tiempo funciona. Luego cambia el modelo, el contexto se alarga, la tarea urge, y la instrucción deja silenciosamente de cumplirse. El agente publica código que rompe CI.

La lección es simple: un system prompt es una sugerencia que el modelo puede ignorar bajo presión. Si un paso tiene que pasar siempre, no puede vivir en prosa — tiene que vivir en configuración que controle algo *distinto del modelo*.

## Tres formas de garantizar que un paso ocurra

No hay un único mecanismo — hay tres, con trade-offs distintos:

| Opción | Determinismo | Latencia del feedback | Dónde encaja |
|---|---|---|---|
| **Hooks del harness** | Alto — los ejecuta el harness, no el modelo | Inmediato (mismo turno) | Si tu herramienta los soporta (Claude Code, Codex CLI, parcial en Aider) |
| **Agente especializado** | Medio-alto — tarea unitaria que el modelo casi siempre ejecuta bien | Mismo turno o turno siguiente | Si no tienes hooks, o si la validación necesita juicio, no solo un comando |
| **Workflows externos** (CI, Actions, pre-commit) | Máximo — vive completamente fuera del agente | Tardío (al PR / al push) | Como red de seguridad, o cuando el agente no está en el bucle (p. ej. Copilot coding agent) |

Cada uno cierra un bucle distinto:

- **Hooks** cierran el **inner loop**: el fallo se convierte en contexto que el agente lee en su siguiente paso, a mitad de turno, sin intervención humana.
- **Agentes especializados** también cierran el inner loop, pero un turno más tarde — el orquestador delega "ejecuta el validador" a un subagente focal y lee su salida.
- **Workflows externos** cierran el **outer loop**: el humano (o un orquestador upstream) ve el fallo durante la revisión, y la siguiente iteración arranca desde ahí.

No eliges uno. Los **apilas**: hooks para lo que el harness soporte, agentes especializados para lo que los hooks no expresen, CI como red final.

### Soporte de las herramientas hoy

| Herramienta | Hooks de evento | Sub-agentes especializados | Integración con workflows externos |
|---|---|---|---|
| Claude Code | Sí (`PreToolUse`, `PostToolUse`, `Stop`, `SessionStart`...) | Sí | Sí (cualquier CI) |
| Codex CLI | Sí, equivalente | Sí | Sí |
| Cursor | Parcial (commands, sin hooks de evento generales) | Limitado | Sí |
| Aider | Limitado (`--lint-cmd`, `--test-cmd` al cerrar turno) | No | Sí |
| GitHub Copilot | **No** tiene hooks de evento | No (Copilot coding agent corre como un único agente) | Sí — se apoya en git hooks + GitHub Actions |

Si tu herramienta no expone hooks, el determinismo sigue teniendo que vivir en algún sitio — empújalo a agentes especializados y workflows externos.

!!! note "El principio es el mismo en todas las herramientas"
    El harness impone, el modelo pide. Que "el harness" sea un hook, un subagente delegado o un job de CI es un detalle de implementación.

## Por qué los hooks (cuando los tienes) son la opción más fuerte

## Eventos de hook que usarás de verdad

La mayoría de harnesses expone una superficie de eventos similar. Los útiles son:

- **PreToolUse** — se dispara antes de una tool call. Puede bloquearla. Útil para políticas ("no dejes al agente editar `/etc`") y redacción.
- **PostToolUse** — se dispara tras una tool call con éxito. Útil para formatear, lintar y ejecutar tests rápidos sobre ficheros cambiados.
- **PreCommit / PrePush** — hooks clásicos de git, siguen siendo valiosos. El escaneo de secretos vive aquí.
- **SessionStart / SessionEnd** — buen sitio para loggear, hacer snapshot de estado o imprimir la rama actual y los ficheros sucios para que el agente arranque con los pies en la tierra.
- **UserPromptSubmit** — se dispara cuando el usuario envía un mensaje; útil para inyectar contexto o bloquear peticiones obviamente peligrosas.

## Hooks del primer día

Antes de afinar prompts, monta estos cuatro. Se amortizan en un día:

1. **Formateador en edición.** Prettier, Black, gofmt, rustfmt. Elimina toda una clase de diffs inútiles.
2. **Linter / typechecker al cerrar el turno.** ESLint, Ruff, golangci-lint, tsc, mypy. Caza los errores favoritos del agente (imports sin usar, variables sombreadas, tipos rotos) — pero ejecútalos una vez al cerrar el turno, no en cada edición (ver "Por edición vs al cerrar turno" más abajo).
3. **Escáner de secretos pre-commit.** gitleaks o trufflehog. El agente acabará pegando una API key en algún sitio.
4. **Tests afectados.** Ejecuta los tests que tocan los ficheros cambiados. Feedback rápido, no CI completa.

### Un fragmento de `settings.json` de ejemplo

Aquí tienes una configuración de hooks al estilo Claude Code. Cursor y Codex usan esquemas distintos pero la forma es similar.

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "npx prettier --write \"$CLAUDE_FILE_PATHS\""
          }
        ]
      }
    ],
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "npx eslint . && npx tsc --noEmit"
          }
        ]
      }
    ],
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "./scripts/deny-dangerous-commands.sh"
          }
        ]
      }
    ]
  }
}
```

El hook `PostToolUse` ejecuta Prettier sobre cualquier fichero que el agente acabe de escribir — barato, idempotente, sin fallos. El hook `Stop` ejecuta ESLint y `tsc` una sola vez cuando el agente termina su turno, produciendo una pasada de verificación coherente en lugar de ruido tras cada edición. El hook `PreToolUse` inspecciona comandos bash y sale con código distinto de cero para bloquear cosas como `rm -rf /` o `git push --force` a `main`.

## Por edición vs al cerrar turno: cuándo ejecutar cada cosa

No toda comprobación pertenece a cada edición. La regla útil:

- **Por edición (`PostToolUse`)** es para **formateadores**: Prettier, Black, gofmt, rustfmt, ruff format. Son baratos, idempotentes, no pueden "fallar" de verdad y evitan que el agente vea jamás código mal formateado. Ejecutarlos en cada escritura es el default correcto.
- **Al cerrar turno (`Stop` / `SubagentStop`)** es para **linters, typecheckers y tests**: ESLint, mypy, tsc, pytest. Son caros, ruidosos y fallan a menudo a mitad de un refactor — un símbolo puede estar momentáneamente ausente, un import temporalmente roto. Ejecutarlos una sola vez al cerrar el turno sobre todo lo que cambió es más rápido y produce una señal de feedback única y coherente.

La regla útil: **formatea por edición, valida al cerrar el turno.**

!!! warning "La validación pesada por edición hace más daño que bien"
    Ejecutar `tsc` después de cada `Edit` desperdicia tokens, ralentiza el bucle e inunda al agente con errores transitorios de código en construcción. El agente entonces "arregla" cosas que no estaban rotas — solo estaban a medio hacer. Espera a que cierre el turno.

## Los hooks como sensores: cómo se entera el agente de los fallos

Que un hook se ejecute es solo la mitad de la historia. La otra mitad es: **¿se entera el agente de que se ejecutó, y de lo que encontró?**

La respuesta vive en el código de salida del hook:

- **Exit 0** — silencio. El agente continúa como si nada.
- **Exit distinto de cero (o stderr)** — el harness típicamente inyecta el stderr del hook de vuelta al contexto del agente como un mensaje del sistema. El agente lo ve como si fuera feedback de una herramienta, abre el fichero, lee el error, lo arregla y reintenta. El bucle de verificación se cierra sin intervención humana.
- **Los hooks `PreToolUse` pueden ser bloqueantes**: un exit no-cero aborta la acción *antes* de que ocurra. Así es como funcionan de verdad guardarraíles tipo "no hagas `rm -rf` fuera del workspace".

Este es el puente con la "verificación antes de devolver el control" del capítulo 1. Un hook bien diseñado no es solo un automatizador — es un **sensor** que convierte efectos colaterales en feedback sobre el que el agente puede actuar. El fallo de `tsc` al cerrar el turno se convierte en la primera tarea del siguiente turno. El agente se autocorrige.

!!! warning "El anti-patrón: hooks que solo loguean"
    Un hook que redirige su salida a `/tmp/agent-hooks.log` y sale con 0 es decoración. Nada vuelve al contexto del agente, así que los fallos son invisibles hasta que un humano lee el log. Si quieres que el agente reaccione, el fallo tiene que ser visible para él.

!!! tip "Un hook no vale por haberse ejecutado"
    Vale porque el agente se enteró de que se ejecutó y actuó sobre el resultado. Cablea el código de salida con intención.

## Bloquear vs avisar

No todo hook debería hacer fallar la tool call. A grandes rasgos:

- **Bloquea** ante violaciones de política, secretos, comandos peligrosos, sintaxis rota.
- **Avisa** (loggea pero no falla) ante detalles de estilo, tests lentos o reglas de lint consultivas.

Un hook que bloquea demasiado agresivamente entrena al agente — y al humano — a buscar bypasses. Un hook que solo avisa se ignora. Elige la severidad correcta para cada regla.

!!! warning "`--no-verify` es mal olor"
    Si tú o el agente tiráis de `git commit --no-verify`, la respuesta correcta casi nunca es "saltarse el hook." Es o "arregla el problema de fondo" o "el hook está mal, arregla el hook." Saltárselos es como los guardarraíles se pudren.

## Depurar fallos de hooks

Cuando un hook falla, resiste la tentación de saltártelo. En su lugar:

1. Ejecuta el comando del hook a mano en tu shell con las mismas entradas.
2. Comprueba que las variables de entorno que inyecta el harness (como `$CLAUDE_FILE_PATHS`) son lo que esperas.
3. Haz el comando idempotente — los hooks que fallan al reejecutarse son dolorosos.
4. Mantén los hooks rápidos. Un hook de 30 segundos en cada edición destruirá el flow.

!!! tip "Loggea, no adivines"
    Redirige la salida del hook a un fichero (`>> /tmp/agent-hooks.log 2>&1`) durante la puesta a punto. Depurarás en minutos en vez de en horas.

## Apilar las tres opciones en la práctica

Un setup realista para un servicio TypeScript podría verse así:

- **Hooks** — Prettier en cada edición; ESLint + `tsc` en `Stop`; `gitleaks` en pre-commit. Cubren las cosas que nunca deberían estar en duda y que se expresan como un único comando.
- **Agente especializado** — un subagente `migration-validator` al que el orquestador llama antes de cualquier cambio en base de datos. Ejecuta la migración en una BD desechable, comprueba operaciones destructivas y devuelve un informe estructurado. Un hook no puede expresar ese juicio; un subagente focal casi siempre sí.
- **Workflows externos** — GitHub Actions sobre el PR: suite de tests completa, escaneo de seguridad con Trivy, smoke tests de rendimiento. Caza lo que se haya escapado, y es contra lo que el humano revisa en el outer loop.

Cada capa alimenta el siguiente bucle. Los hooks autocorrigen al agente en segundos. Los agentes especializados cazan lo que los hooks no pueden, en la misma sesión. CI caza lo que el agente se perdió y lo expone al humano, que lo arregla o le pide al agente que lo arregle.

## El puente con el harness engineering

Aquí es donde "usar un agente" se convierte en "hacer ingeniería de un harness." Una vez tienes hooks, agentes especializados y CI conectados — y los tres devuelven señal al bucle al que pertenecen — dejas de preocuparte de si el agente se acordó de las reglas. Las reglas ya no son trabajo del agente. Ese traspaso de responsabilidad, del prompt a la fontanería, es el mayor salto de madurez que hace un equipo.

!!! success "Idea clave"
    El determinismo no viene de un único mecanismo. **Los hooks** cubren lo que el harness puede expresar. **Los agentes especializados** cubren lo que los hooks no pueden. **Los workflows externos** son la red debajo de los dos. Cada uno alimenta el inner loop del agente o el outer loop del humano — y un paso que no alimenta *algún* bucle no es una garantía, es decoración.
