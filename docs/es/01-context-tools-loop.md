# 1. Contexto, herramientas, bucle — el modelo mental

Todo coding agent, sea del proveedor que sea, está hecho de las mismas tres piezas: **lo que ve** (contexto), **lo que puede hacer** (herramientas) y **cómo decide** (bucle). Si interiorizas estas tres, puedes razonar sobre cualquier producto de agentes sin reaprender el modelo mental.

## Qué vas a aprender

- Las tres piezas que tiene cualquier agente y cómo interactúan.
- A preguntar "¿qué ve el agente ahora mismo?" y "¿qué puede hacer realmente?" antes de depurar comportamientos extraños.
- Por qué la mayoría de fallos de agentes son fallos de contexto o de herramientas, no de modelo.

## Esquema

1. **Contexto** — system prompt, user prompt, ficheros leídos, resultados de herramientas, memoria. Todo lo que el modelo "ve" en este turno.
2. **Herramientas** — leer/escribir/editar ficheros, ejecutar shell, buscar código, etc. Los verbos que tiene el agente.
3. **Bucle** — el modelo emite tool calls → el harness los ejecuta → vuelven los resultados → el modelo decide el siguiente paso → se repite hasta terminar.
4. Diagrama: un turno individual vs una tarea completa.
5. Ejercicio de diagnóstico: "el agente hizo algo absurdo" → ¿fue contexto, herramientas o bucle?

## Idea clave

Cuando algo va mal, hazte las tres preguntas en orden: ¿vio lo que tenía que ver? ¿tenía las herramientas adecuadas? ¿el bucle terminó limpiamente? Esto bate al prompt-tweaking nueve de cada diez veces.
