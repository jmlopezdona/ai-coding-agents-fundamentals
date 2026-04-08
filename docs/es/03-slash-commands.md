# 3. Slash commands y prompts reutilizables

Cuando te encuentres tecleando el mismo prompt por quinta vez, es momento de cristalizarlo como un slash command (`/commit`, `/review-pr`, `/test`). Los comandos son cómo conviertes prompts puntuales en convenciones de equipo.

## Qué vas a aprender

- Cuándo merece la pena que un prompt se convierta en comando.
- Anatomía de un buen comando: alcance, entradas, salida esperada.
- Cómo los comandos componen valor a través del equipo.

## Esquema

1. La regla de la "quinta vez": los prompts que repites se convierten en comandos.
2. Anatomía: un nombre claro, una sola responsabilidad, entradas explícitas, salida predecible.
3. Ejemplos: `/commit`, `/review-pr`, `/test`, `/explain`, `/migrate`.
4. Dónde viven los comandos (por usuario vs por repo) y por qué los del repo ganan en equipos.
5. Modos de fallo: comandos que intentan hacer demasiado, comandos que nadie actualiza, comandos que esconden contexto importante.
6. Versionado y revisión — sí, los comandos también son código.

## Idea clave

Un slash command es una llamada a función para prompts. Aplica la misma higiene: una sola responsabilidad, interfaz clara, code review.
