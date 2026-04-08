# 4. Skills: conocimiento bajo demanda

Las skills son paquetes de instrucciones y recursos que el agente carga **solo cuando son relevantes**. Resuelven un problema que `AGENTS.md` no puede: cómo dar al agente conocimiento profundo de dominio sin inflar su contexto en cada turno.

## Qué vas a aprender

- Qué es una skill y en qué se diferencia del contenido de `AGENTS.md`.
- Cómo se descubren y activan las skills.
- Cuándo crear una skill vs cuándo añadir una línea a `AGENTS.md`.

## Esquema

1. El problema del exceso: por qué no puedes meterlo todo en `AGENTS.md`.
2. Las skills como conocimiento de carga perezosa: solo presentes en el contexto cuando son relevantes.
3. Anatomía de una skill: descripción del trigger, cuerpo, recursos opcionales.
4. Descubrimiento: cómo decide el agente cargar una skill.
5. Ejemplos: una skill de "escribir migraciones", una de "revisar PRs", una de "convenciones Spring Boot".
6. Regla de decisión: siempre cierto → `AGENTS.md`. A veces cierto → skill.

## Idea clave

Las skills son cómo escalas conocimiento sin escalar contexto. Si una guía solo importa para el 10% de las tareas, no pertenece a `AGENTS.md`.
