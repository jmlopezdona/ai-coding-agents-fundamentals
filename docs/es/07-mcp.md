# 7. MCP y herramientas externas

MCP (Model Context Protocol) es una forma estándar de enchufar herramientas y fuentes de datos externas a un agente: una base de datos, un sistema de tickets, un sitio de docs, una API interna a medida. Este capítulo lo explica sin jerga y te ayuda a decidir cuándo MCP merece la pena vs un script normal.

## Qué vas a aprender

- Qué es MCP, en una página.
- Cuándo añadir un servidor MCP vs un script local o una herramienta CLI.
- Riesgos básicos de cadena de suministro al instalar servidores MCP de terceros.

## Esquema

1. El problema que resuelve MCP: cada agente necesita las mismas integraciones, cada proveedor estaba inventando la suya.
2. El modelo: el servidor expone tools/resources/prompts, el agente los descubre y los llama.
3. Cuándo MCP merece la pena: compartido en el equipo, varios agentes, necesita auth, necesita ser descubrible.
4. Cuándo basta con un script o una CLI: caso puntual, un solo usuario, entrada/salida simple.
5. Cadena de suministro: ¿quién escribió este servidor MCP? ¿a qué tiene acceso? prompt injection a través de resultados de herramientas.
6. Una lista corta de servidores MCP que compensan pronto (filesystem, GitHub, tus docs).

## Idea clave

MCP es fontanería, no magia. Úsalo cuando la integración necesita ser compartida y descubrible; tira de un script cuando no.
