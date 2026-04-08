# 2. Tu primer AGENTS.md

`AGENTS.md` (o `CLAUDE.md`, `.cursorrules`, etc.) es el fichero que el agente lee antes de cada tarea. Es tu palanca más directa para moldear su comportamiento. El error más común es volcar dentro toda la documentación del proyecto. No es para eso.

## Qué vas a aprender

- Para qué sirve `AGENTS.md` y para qué no.
- El principio: **instrucciones, no enciclopedia**.
- Una plantilla mínima que puedes adaptar.

## El mismo fichero, muchos nombres

Cada coding agent importante tiene un fichero de "instrucciones del proyecto" que se inyecta en el contexto en cada turno:

| Herramienta | Fichero |
|-------------|---------|
| OpenAI Codex CLI, Aider, genérico | `AGENTS.md` |
| Claude Code | `CLAUDE.md` (también lee `AGENTS.md`) |
| Cursor | `.cursorrules` o `.cursor/rules/*.mdc` |
| GitHub Copilot | `.github/copilot-instructions.md` |
| Continue | `.continuerules` |

El nombre del fichero cambia; el rol es idéntico: un fichero corto, versionado, que le dice al agente cómo *este repo en particular* quiere que se trabaje en él. En este libro lo llamaremos `AGENTS.md` de forma genérica.

!!! note "Se carga en cada turno"
    A diferencia del README, este fichero está *siempre* en contexto. Cada línea cuesta tokens en cada turno. Por eso importa la brevedad — y por eso es tan poderoso.

## Qué meter dentro

Mete cosas que el agente **no pueda inferir del propio código** y que **cambien su comportamiento**. Buenos candidatos:

- **Cómo ejecutar cosas**: el comando exacto para tests, lint, typecheck, build. No "usamos pytest" — `pytest -xvs tests/`.
- **Fuente de verdad**: qué directorio es generado, cuál es escrito a mano, dónde se definen los tipos.
- **Convenciones** que el linter no caza: "usa `Result<T, E>` no excepciones para errores de dominio", "todo acceso a BD pasa por la capa de repositorios".
- **Cosas que evitar**: "no edites ficheros bajo `vendor/`", "no actualices dependencias sin preguntar", "no añadas dependencias top-level nuevas".
- **Rarezas locales**: "los tests de integración necesitan Docker corriendo", "node_modules está en la raíz del repo, no por paquete".

## Qué dejar fuera

- Análisis arquitectónicos profundos. El agente leerá el código si lo necesita.
- Historia y justificaciones. Útiles para humanos, ruido para agentes.
- Cualquier cosa obvia desde la estructura del repo (`src/` contiene fuentes — sí, gracias).
- La lista completa de cada endpoint, cada modelo, cada tabla. Deja que las herramientas de búsqueda hagan ese trabajo.
- Texto de marketing del README.

!!! warning "Las enciclopedias se ignoran"
    Un `AGENTS.md` de 2000 líneas no hace al agente más listo. Empuja señal útil fuera de la ventana de contexto y te entrena a no leer el fichero cuando algo necesita cambiar. Mantenlo en menos de una página si puedes.

## El principio "instrucciones, no enciclopedia"

Lee cada línea de tu `AGENTS.md` y pregúntate: *"si borro esta línea, ¿se comportará peor el agente en alguna tarea real?"* Si la respuesta honesta es no, bórrala. Cada línea que sobreviva debería ser un cambio de comportamiento esperando a ocurrir.

## Un ejemplo trabajado

Aquí tienes un `AGENTS.md` realista para un pequeño servicio web en Python. Tiene unas 40 líneas, y cada línea está ahí por una razón.

```markdown
# AGENTS.md

Esto es `billing-service`, una app FastAPI que emite facturas.
Python 3.11, gestionado con `uv`.

## Comandos

- Instalar deps:       `uv sync`
- Correr tests:        `uv run pytest -xvs`
- Correr un test:      `uv run pytest -xvs tests/test_invoices.py::test_total`
- Lint + format:       `uv run ruff check --fix && uv run ruff format`
- Type check:          `uv run mypy src/`
- Correr local:        `uv run uvicorn billing.app:app --reload`

Siempre corre lint, typecheck y los tests antes de declarar una tarea hecha.

## Estructura

- `src/billing/`        — código de aplicación (escrito a mano, fuente de verdad)
- `src/billing/api/`    — routers FastAPI, finos; aquí no hay lógica de negocio
- `src/billing/domain/` — lógica de dominio pura, sin I/O, totalmente unit-testeada
- `src/billing/db/`     — modelos SQLAlchemy y repositorios
- `tests/`              — pytest, refleja la estructura de `src/billing/`
- `migrations/`         — Alembic, generado; no editar a mano

## Convenciones

- El dinero es `decimal.Decimal`, nunca `float`. Round half-even.
- Los errores de dominio son clases de excepción explícitas en
  `billing.domain.errors`, no `ValueError` genéricos.
- Todo acceso a BD pasa por un repositorio en `billing/db/repositories/`.
  Routers y servicios nunca importan `sqlalchemy` directamente.
- Las funciones públicas tienen type hints y un docstring de una línea.

## Cosas que NO hacer

- No añadas dependencias top-level nuevas sin preguntar.
- No edites ficheros bajo `migrations/versions/` a mano;
  genera una nueva migración con `alembic revision --autogenerate`.
- No captures `Exception` de forma amplia; captura el error de dominio específico.
- No commitees cambios a `.env` ni a nada en `secrets/`.
```

Fíjate en lo que *no* está ahí: ninguna descripción de qué es una factura, ninguna lista de endpoints, ningún diagrama de arquitectura, ninguna historia. El agente puede encontrar todo eso leyendo el código. Lo que no puede encontrar leyendo el código son *los comandos que realmente usas*, *qué directorios son generados* y *qué evitar*.

## Cómo hacerlo evolucionar

Tu `AGENTS.md` es un fichero vivo, pero debería crecer despacio y con evidencia.

!!! tip "La regla de las dos veces"
    Añade una línea al `AGENTS.md` la segunda vez que el agente se equivoque en lo mismo. Una vez puede ser una casualidad. Dos veces significa que la información falta del contexto, y una sola línea puede arreglarlo para siempre.

A la inversa, poda. Si una línea se añadió por una convención antigua y esa convención ha cambiado, bórrala — las instrucciones obsoletas son peores que no tener instrucciones, porque el agente las seguirá con confianza.

## Idea clave

!!! tip "Idea clave"
    Un `AGENTS.md` corto y opinionado que se lee en cada turno bate a uno de 2000 líneas que se ignora. Empieza con 30 líneas. Gánate cada incorporación.
