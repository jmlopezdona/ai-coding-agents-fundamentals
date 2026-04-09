# AI Coding Agents — Fundamentos

Una guía práctica y agnóstica de proveedor para empezar a trabajar en serio con AI coding agents — antes de necesitar un harness completo a su alrededor.

## Para quién es esta guía

Equipos técnicos que están *empezando* a operar AI coding agents (Claude Code, Codex, Cursor, Copilot…) y quieren un modelo mental sólido antes de chocarse con el dolor de escalar. No requiere experiencia previa con agentes — solo con ingeniería de software.

!!! warning "Este espacio se mueve rápido"
    Los AI coding agents evolucionan en semanas, no en años. Una característica que esta guía describe como "ausente" en la herramienta X puede que ya exista cuando la leas; un nombre específico de un proveedor puede haberse renombrado; una fila "Sí/No" de una comparativa puede haber cambiado. Trata las comparativas como una foto de la intención, no como una especificación autoritativa — y revisa la documentación actual de cada herramienta antes de tomar decisiones. El **modelo mental** (contexto, herramientas, bucle, verificación, hooks, skills, subagentes) es lo que se mantiene estable; las implementaciones no.

## Cómo leerla

Los 14 capítulos avanzan progresivamente. Si vienes de cero, léelos en orden. Si ya tienes algo de experiencia, salta al capítulo que responda a tu pregunta actual.

## Capítulos

- [0. Qué es realmente un AI coding agent](00-what-is-an-agent.md)
- [1. Contexto, herramientas, bucle — el modelo mental](01-context-tools-loop.md)
- [2. Tu primer AGENTS.md](02-first-agents-md.md)
- [3. Rules e instructions: el mismo problema, distintos formatos](03-rules-instructions.md)
- [4. Slash commands y prompts reutilizables](04-slash-commands.md)
- [5. Skills: conocimiento bajo demanda](05-skills.md)
- [6. Subagentes y delegación](06-subagents.md)
- [7. Garantías deterministas: hooks, agentes especializados y workflows externos](07-hooks.md)
- [8. MCP y herramientas externas](08-mcp.md)
- [9. Permisos, sandbox y modos de ejecución](09-permissions-sandbox.md)
- [10. Privacidad y datos sensibles](10-privacy-sensitive-data.md)
- [11. Memoria y persistencia entre sesiones](11-memory-persistence.md)
- [12. Anti-patrones de los primeros meses](12-anti-patterns.md)
- [13. Señales de que necesitas un harness](13-signs-you-need-a-harness.md)

## Qué sigue

Esta guía es el primer volumen de una trilogía. Cuando deje de bastarte — cuando tu equipo empiece a sentir el dolor de operar agentes a escala — los siguientes pasos son:

👉 **[Spec-Driven Development](https://jmlopezdona.github.io/ai-coding-agents-sdd/es/)** — cómo especificar la intención con claridad y mantener esa especificación viva mientras el código evoluciona.

👉 **[Harness Engineering — construir software a escala con agentes](https://jmlopezdona.github.io/ai-coding-agents-harness/es/)** — cómo construir las guías, sensores, bucles, sandboxes, subagentes y contexto estructurado que convierten un LLM en un agente del que un equipo puede fiarse.
