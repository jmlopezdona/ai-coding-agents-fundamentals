# 11. Señales de que necesitas un harness

Esta guía basta para que tu equipo sea productivo con agentes. *No* basta para que sigas siéndolo cuando escales. En algún momento sentirás una fricción que ningún ajuste de prompts arregla — ese es el momento de graduarte al harness engineering.

## Qué vas a aprender

- Síntomas concretos que significan "te has quedado pequeño con esta guía."
- Por qué el arreglo es estructural, no a nivel de prompt.
- Adónde ir después.

## Esquema

1. **Síntoma: el agente repite los mismos errores entre developers.** No hay un bucle de corrección compartido.
2. **Síntoma: cada developer tiene su propio setup.** No hay harness compartido.
3. **Síntoma: las PRs del agente requieren más review que las humanas.** La verificación se ha empujado al humano en lugar de al harness.
4. **Síntoma: la misma clase de bug aparece una y otra vez.** No hay sensores que la cacen antes de la review.
5. **Síntoma: hacer onboarding de un nuevo dev a "la forma del agente" lleva una semana.** No hay sistema de registro.
6. **Síntoma: estás escribiendo prompts cada vez más largos.** Estás compensando la ausencia de guías y sensores.
7. Por qué el prompt-engineering no arregla nada de esto: el problema es la ausencia de un sistema.

## Qué sigue

Cuando reconozcas estos síntomas, salta a la guía complementaria:

👉 **[Harness Engineering — construir software a escala con agentes](https://jmlopezdona.github.io/ai-coding-agents-harness/)**

Empieza exactamente donde esta termina: cómo construir las guías, sensores, bucles, sandboxes y contexto estructurado que convierten un LLM en un agente del que un equipo puede fiarse.

## Idea clave

Si estás retocando prompts para arreglar problemas estructurales, te has quedado pequeño con esta guía. El siguiente paso no es un prompt mejor — es un harness.
