# 6. Hooks y automatización

Los hooks son comandos que el harness ejecuta automáticamente ante eventos (antes de una tool call, después de una edición, antes de un commit). Son cómo impones cosas en las que no deberías confiar a la memoria del agente — formato, lint, escaneo de secretos, ejecución de tests.

## Qué vas a aprender

- La diferencia entre "pedir al agente que haga X" y "hacer que el harness imponga X."
- Hooks habituales que merece la pena montar el primer día.
- Por qué los hooks baten a las instrucciones para cualquier cosa crítica de seguridad.

## Esquema

1. La poca fiabilidad del "por favor ejecuta siempre el linter".
2. Eventos de hook: pre-tool, post-tool, pre-commit, session-start, session-end.
3. Hooks del primer día: formateador, linter, escáner de secretos, runner de tests sobre ficheros cambiados.
4. Hooks que bloquean vs hooks que avisan.
5. Depurar fallos de hooks sin saltárselos (`--no-verify` es mal olor).
6. El puente con la guía de harness: los hooks son donde empieza realmente el harness engineering.

## Idea clave

Si tiene que pasar siempre, tiene que ser un hook. Las instrucciones son esperanzas; los hooks son garantías.
