# 7. Hooks y automatización

Los hooks son comandos que el harness ejecuta automáticamente ante eventos (antes de una tool call, después de una edición, antes de un commit). Son cómo impones cosas en las que no deberías confiar a la memoria del agente — formato, lint, escaneo de secretos, ejecución de tests.

## Qué vas a aprender

- La diferencia entre "pedir al agente que haga X" y "hacer que el harness imponga X."
- Hooks habituales que merece la pena montar el primer día.
- Por qué los hooks baten a las instrucciones para cualquier cosa crítica de seguridad.

## Las instrucciones son esperanzas, los hooks son garantías

Todo equipo que adopta un agente de código pasa por el mismo ciclo. Alguien escribe en `AGENTS.md` o `CLAUDE.md`: "Ejecuta siempre `pnpm lint` después de editar TypeScript." Durante un tiempo funciona. Luego cambia el modelo, el contexto se alarga, la tarea urge, y la instrucción deja silenciosamente de cumplirse. El agente publica código que rompe CI.

La lección es simple: un system prompt es una sugerencia que el modelo puede ignorar bajo presión. Un hook es un comando que el harness ejecuta le guste o no al modelo. Si un paso tiene que pasar siempre, no puede vivir en prosa — tiene que vivir en configuración.

!!! note "Aplica a cualquier harness"
    Claude Code, Cursor, Codex CLI, Aider y otros exponen alguna forma de hook de eventos o comando pre/post. Los nombres cambian pero el principio es idéntico: el harness impone, el modelo pide.

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
2. **Linter en edición.** ESLint, Ruff, golangci-lint. Caza los errores favoritos del agente (imports sin usar, variables sombreadas).
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
          },
          {
            "type": "command",
            "command": "npx eslint --fix \"$CLAUDE_FILE_PATHS\""
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

El hook `PostToolUse` ejecuta Prettier y luego ESLint sobre cualquier fichero que el agente acabe de escribir. El hook `PreToolUse` inspecciona comandos bash y sale con código distinto de cero para bloquear cosas como `rm -rf /` o `git push --force` a `main`.

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

## El puente con el harness engineering

Los hooks son donde "usar un agente" se convierte en "hacer ingeniería de un harness." Una vez tienes formateador, linter, escáner de secretos y runner de tests conectados, dejas de preocuparte de si el agente se acordó de las reglas — porque las reglas ya no son trabajo del agente. Ese traspaso de responsabilidad, del prompt a la fontanería, es el mayor salto de madurez que hace un equipo.

!!! success "Idea clave"
    Si tiene que pasar siempre, tiene que ser un hook. Las instrucciones son esperanzas; los hooks son garantías.
