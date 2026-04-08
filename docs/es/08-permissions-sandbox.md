# 8. Permisos, sandbox y modos de ejecución

Todo producto de agentes tiene mandos para "¿cuánto puede hacer sin preguntar?" Equivócate en cualquiera de las dos direcciones y lo sufrirás: demasiado restrictivo y el agente es inútil, demasiado permisivo y te arrepentirás la primera vez que ejecute `rm -rf` o haga push a `main`.

## Qué vas a aprender

- Los modos de ejecución que ofrecen la mayoría de agentes (plan, ask, auto-accept).
- Cómo elegir autonomía por herramienta, no por sesión.
- Un default seguro para un equipo que empieza.

## Esquema

1. El espectro de autonomía: solo-plan → preguntar-cada-tool → auto-accept-allowlist → auto-completo.
2. Por qué "auto-completo para todo" es una trampa de principiante.
3. Permisos por herramienta: confía en `Read` y `Grep`, controla `Bash` y `Write`.
4. Sandboxes: contenedores, worktrees, entornos efímeros.
5. Defaults seguros para el primer mes de un equipo.
6. Cuándo relajar permisos y cómo hacerlo deliberadamente, no por frustración.

## Idea clave

Elige autonomía por herramienta, no por sesión. El coste de una confirmación es ínfimo; el coste de un `git push --force` no deseado es enorme.
