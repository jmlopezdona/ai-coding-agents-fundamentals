# 0. Qué es realmente un AI coding agent

La mayoría de developers conocieron los LLMs a través del autocomplete (Copilot) o del chat (ChatGPT). Un *agente* no es ninguna de las dos cosas. El salto cualitativo no está en el modelo — está en el **bucle**: un modelo que puede leer ficheros, ejecutar herramientas, observar resultados y decidir el siguiente paso, repetidamente, hasta terminar una tarea.

## Qué vas a aprender

- La diferencia entre autocomplete, chat y agente.
- Por qué el bucle — no el modelo — es lo que marca la diferencia.
- Un primer vocabulario: herramientas, contexto, turno, resultado de herramienta.

## Esquema

1. **Autocomplete** — predice el siguiente token. Sin estado, sin herramientas, sin decisiones.
2. **Chat** — conversación multi-turno. Aún sin herramientas ni acceso al sistema de ficheros.
3. **Agente** — modelo + herramientas + bucle. Lee, edita, ejecuta comandos, observa, itera.
4. Un ejemplo trabajado: "arregla este test que falla" en cada uno de los tres modos.
5. Por qué esto cambia cómo deberías pensar en revisar código generado por IA.

## Idea clave

Deja de comparar los agentes con autocomplete. Son máquinas distintas. La unidad de trabajo ya no es "una sugerencia que aceptas" — es "una tarea que delegas y verificas."
