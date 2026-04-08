# 5. Subagentes y delegación

Un subagente es un agente nuevo que el agente principal lanza para tratar una sub-tarea de forma aislada. Hay dos razones para usarlos: **paralelización** (búsquedas independientes a la vez) y **protección del contexto** (mantener el hilo principal limpio de salidas grandes y ruidosas).

## Qué vas a aprender

- Cuándo delegar y cuándo hacerlo tú mismo en el hilo principal.
- Cómo los subagentes protegen el context window principal.
- El coste: un subagente no comparte tu conversación, así que el briefing importa.

## Esquema

1. Las dos razones reales para usar subagentes.
2. Anti-razón: "suena moderno." No delegues trabajo trivial.
3. Briefing a un subagente como a un colega que acaba de entrar: objetivo, contexto, restricciones, salida esperada.
4. Delegación paralela: varios subagentes independientes en un mismo turno.
5. Subagentes especializados (research, planificación, búsqueda de código) y cuándo cada uno compensa.
6. Modo de fallo: delegar la *comprensión* — hacer que el subagente decida lo que deberías haber decidido tú.

## Idea clave

Los subagentes son para proteger contexto y paralelizar — no para descargar el pensamiento. Nunca delegues el paso de síntesis.
