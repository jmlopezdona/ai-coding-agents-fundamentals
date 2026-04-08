# 2. Tu primer AGENTS.md

`AGENTS.md` (o `CLAUDE.md`, `.cursorrules`, etc.) es el fichero que el agente lee antes de cada tarea. Es tu palanca más directa para moldear su comportamiento. El error más común es volcar dentro toda la documentación del proyecto. No es para eso.

## Qué vas a aprender

- Para qué sirve `AGENTS.md` y para qué no.
- El principio: **instrucciones, no enciclopedia**.
- Una plantilla mínima que puedes adaptar.

## Esquema

1. El fichero según el proveedor: `AGENTS.md`, `CLAUDE.md`, `.cursorrules`, `.github/copilot-instructions.md`. Misma idea, distintos nombres.
2. Qué meter dentro: cómo correr los tests, dónde está la fuente de verdad, convenciones que el agente no puede inferir del código, cosas que evitar.
3. Qué dejar fuera: análisis arquitectónicos, historia, cualquier cosa obvia desde la estructura del repo.
4. El principio "instrucciones, no enciclopedia" — cada línea debería cambiar el comportamiento.
5. Un ejemplo mínimo trabajado para un repo típico Node.js / Python / Java.
6. Cómo hacer evolucionar el fichero: añade una línea cada vez que el agente se equivoque dos veces en lo mismo.

## Idea clave

Un `AGENTS.md` corto y opinionado que se lee en cada turno bate a uno de 2000 líneas que se ignora. Empieza con 30 líneas. Gánate cada incorporación.
