# 9. Memoria y persistencia entre sesiones

Los agentes tienen varias formas de "recordar" cosas: contexto en conversación, ficheros de memoria persistente, planes, listas de tareas, y el propio repo. Cada una tiene una vida útil y un propósito distinto. Mezclarlas es uno de los errores más comunes al empezar.

## Qué vas a aprender

- Las cuatro capas de persistencia y para qué sirve cada una.
- Por qué volcarlo todo en "memoria" es mala idea.
- Un árbol de decisión: ¿dónde pertenece esta información?

## Esquema

1. Capa 1: **contexto de conversación** — vive durante la tarea actual.
2. Capa 2: **planes y tareas** — viven durante la sesión actual.
3. Capa 3: **memoria** — sobrevive entre sesiones, pero debería ser rara y curada.
4. Capa 4: **el repo** — `AGENTS.md`, skills, comandos. La única capa verdaderamente duradera.
5. Árbol de decisión: efímero → contexto; esta sesión → plan/tareas; futuras sesiones sobre *ti* → memoria; futuras sesiones sobre *el proyecto* → repo.
6. Anti-patrón: usar la memoria como cuaderno. La memoria es para hechos no obvios sobre el usuario y el proyecto, no para un log.

## Idea clave

Cada capa de persistencia tiene su trabajo. El repo es la única que tus compañeros pueden ver — cualquier cosa importante para el *equipo* pertenece ahí, no a la memoria de un developer concreto.
