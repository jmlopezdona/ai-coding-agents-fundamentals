# 0. Qué es realmente un AI coding agent

La mayoría de developers conocieron los LLMs a través del autocomplete (Copilot) o del chat (ChatGPT). Un *agente* no es ninguna de las dos cosas. El salto cualitativo no está en el modelo — está en el **bucle**: un modelo que puede leer ficheros, ejecutar herramientas, observar resultados y decidir el siguiente paso, repetidamente, hasta terminar una tarea.

## Qué vas a aprender

- La diferencia entre autocomplete, chat y agente.
- Por qué el bucle — no el modelo — es lo que marca la diferencia.
- Un primer vocabulario: herramientas, contexto, turno, resultado de herramienta.

## Tres máquinas, no una

Es tentador pensar en Copilot, ChatGPT y Claude Code como tres sabores de "la misma cosa de IA". No lo son. Son tres sistemas arquitectónicamente distintos que casualmente comparten un LLM en su núcleo. Confundirlos es la causa raíz de la mayoría de las malas opiniones sobre las herramientas de IA para programar — tanto del hype como del rechazo.

### Autocomplete: predecir el siguiente token

Herramientas como la finalización en línea de GitHub Copilot, Tabnine o el primer Codex se comportan como un predictor de texto muy listo. Tú escribes, el modelo propone los siguientes tokens, tú aceptas o rechazas. No hay memoria entre completados, no hay herramientas, no hay decisiones. El modelo nunca lee un fichero que no abriste, nunca corre un test, nunca se pregunta "¿esto funcionó?"

```python
def parse_iso_date(s: str) -> datetime:
    # cursor aquí — autocomplete sugiere el cuerpo
```

La unidad de trabajo es **una sugerencia**. El revisor eres **tú, en cada pulsación**.

### Chat: conversación multi-turno

ChatGPT, Claude.ai, Gemini en una pestaña del navegador — son conversacionales. Mantienen el historial de diálogo, pueden razonar entre turnos y producir salidas largas y estructuradas. Pero por defecto siguen sin poder tocar tu sistema de ficheros, ejecutar tus tests o comprobar si el snippet que acaban de escribir compila. Tú pegas dentro, tú pegas fuera.

La unidad de trabajo es **una respuesta**. El revisor eres **tú, después de cada réplica**.

### Agente: modelo + herramientas + bucle

Claude Code, el modo agente de Cursor, OpenAI Codex CLI, Aider, Cline, Continue — esto es distinto. El modelo está conectado a un conjunto de **herramientas** (leer fichero, escribir fichero, ejecutar comando shell, buscar código, descargar URL...) y corre dentro de un **bucle**: emite una llamada a una herramienta, el harness la ejecuta, el resultado vuelve como un nuevo mensaje y el modelo decide qué hacer a continuación. Sigue hasta que cree que la tarea está hecha — o te hace una pregunta.

```
user:    "el test en billing_test.py está fallando, arréglalo"
agent:   read_file(billing_test.py)
agent:   read_file(billing.py)
agent:   run_shell("pytest billing_test.py")     -> rojo
agent:   edit_file(billing.py, ...)
agent:   run_shell("pytest billing_test.py")     -> verde
agent:   "arreglado: el redondeo usaba floor en vez de round-half-even"
```

La unidad de trabajo es **una tarea**. El revisor eres **tú, sobre el diff y el resultado**.

## La misma tarea, de tres formas

Imagina: *"el test `test_invoice_total` está fallando, arréglalo."*

- **Autocomplete** no puede ayudar. No sabe que el test existe.
- **Chat** puede ayudar si pegas el test, la función que falla y el stack trace — y te devuelve código que tú tienes que pegar, guardar y volver a ejecutar. Tú eres el bucle.
- **Agente** abre el test, lee la función, ejecuta el test para ver el fallo real, edita el código, vuelve a ejecutar el test e informa. El bucle está automatizado.

Esa última frase es la idea central de este libro.

## Un primer vocabulario

!!! note "Palabras que verás en cada capítulo"
    - **Contexto** — todo lo que el modelo puede ver ahora mismo: tu mensaje, system prompt, ficheros que ha leído, resultados previos de herramientas.
    - **Herramienta** — una función que el agente puede llamar (p. ej. `read_file`, `run_shell`).
    - **Tool call** — la petición del modelo para invocar una herramienta, con sus argumentos.
    - **Tool result** — lo que vuelve, alimentado en el siguiente turno.
    - **Turno** — una ronda de "el modelo piensa → el modelo emite algo → el harness reacciona."
    - **Harness** — el programa alrededor del modelo que ejecuta las herramientas, gestiona ficheros, aplica permisos. Claude Code, Cursor, Aider, etc. *son* harnesses.

## Por qué esto cambia cómo revisas el código

Cuando usas autocomplete, revisas a nivel de pulsación. Cuando usas chat, revisas a nivel de snippet. Cuando usas un agente, revisas a nivel de **tarea** — el diff más la evidencia que el agente reunió (qué tests corrió, qué ficheros tocó, qué observó).

!!! warning "No revises a un agente como el PR de un junior"
    No puedes cazar cada línea. En su lugar: comprueba que el diff sea pequeño y enfocado, comprueba que los tests que dice haber corrido existen y pasan, y trata cualquier cosa fuera del alcance pedido como una bandera roja.

Por eso también las historias de "la IA hizo un destrozo" casi siempre vienen de gente usando un agente como si fuera autocomplete: aceptando sugerencias una a una, sin mirar el diff global, sin preguntar qué tests se ejecutaron.

## Idea clave

!!! tip "Idea clave"
    Deja de comparar los agentes con autocomplete. Son máquinas distintas. La unidad de trabajo ya no es "una sugerencia que aceptas" — es "una tarea que delegas y verificas."
