# 6. Subagentes y delegación

Un subagente es un agente nuevo que el agente principal lanza para tratar una sub-tarea de forma aislada. Hay dos razones para usarlos: **paralelización** (búsquedas independientes a la vez) y **protección del contexto** (mantener el hilo principal limpio de salidas grandes y ruidosas).

## Qué vas a aprender

- Las dos razones reales para delegar a un subagente.
- Cuándo hacer el trabajo en línea en el hilo principal.
- Cómo dar un briefing a un subagente para que devuelva algo realmente útil.
- El único modo de fallo que arruina la delegación: externalizar tu propio pensamiento.

## Las dos razones reales

### 1. Protección del contexto

La conversación del agente principal es preciosa. Cada token gastado en una salida de `grep` de 4.000 líneas es un token no gastado en el problema real. Un subagente puede ejecutar ese `grep`, leer las coincidencias y devolver *un párrafo* de síntesis. El ruido se queda en el contexto del subagente y muere con él.

### 2. Paralelización

Tres preguntas independientes — "¿dónde se gestiona auth?", "¿cómo es el modelo de usuario?", "¿cómo se envían los emails?" — pueden ser respondidas por tres subagentes a la vez. Hacerlas secuencialmente en el hilo principal solo es más lento sin ningún beneficio.

!!! note "Ambas razones tratan del contexto principal"
    O lo estás protegiendo del ruido, o lo estás llenando más rápido paralelizando. Si no aplica ninguna, no necesitas un subagente.

## La anti-razón

!!! warning "No delegues porque suena moderno"
    Delegar "renombra esta variable" a un subagente es más lento, más caro y más difícil de depurar que simplemente hacerlo. El coste de levantar un subagente — briefing, espera, parsear la respuesta — es real. Págalo solo cuando el ahorro sea real.

## En línea vs delegar: chuleta

| Situación | Hazlo en línea | Delega a subagente |
|---|---|---|
| Editar un archivo que ya tienes delante | Sí | No |
| Buscar un patrón en 200 archivos | No | Sí — devolver solo coincidencias |
| Leer tres archivos no relacionados en paralelo | No | Sí — un subagente cada uno |
| Decidir qué enfoque tomar | Sí | No — nunca delegues la decisión |
| Resumir un log de 2.000 líneas | No | Sí — devolver el resumen |
| Escribir el cambio de código final | Sí | No — la síntesis es tuya |

El patrón: **delega la recolección, quédate con la decisión.**

## Briefing a un subagente

Un subagente no comparte tu conversación. Entra en frío. Dale el briefing como se lo darías a un colega que acaba de sentarse en tu mesa: objetivo, contexto, restricciones, salida esperada.

### Un ejemplo concreto de prompt

Supón que la tarea principal es "añadir rate limiting a la API pública". Antes de escribir código quieres saber cómo está organizado el stack de middleware existente. Aquí tienes un buen briefing de subagente:

```text
Goal: Map the HTTP middleware stack of this Spring Boot service so I can
decide where to insert a rate-limiter.

Context:
- Repo root: /Users/me/work/payments-api
- Framework: Spring Boot 3.3, Java 21
- I am about to add a rate limiter for the /v1/public/** routes
- I have NOT yet decided between Bucket4j and Resilience4j

Please investigate:
1. List every class annotated with @Configuration that registers a
   filter, interceptor, or WebMvcConfigurer customization.
2. For each, note the order (Ordered / @Order) and which URL patterns
   it applies to.
3. Identify the current authentication filter and where in the chain
   it sits.

Constraints:
- Read-only. Do not modify any files.
- Do not recommend a library — I'll decide that myself.
- Budget: ~5 minutes of exploration.

Expected output:
- A table: filter/interceptor name -> order -> URL pattern -> file path
- A 3-sentence summary of where a new filter would naturally fit
- Nothing else
```

Fíjate en la estructura: un único objetivo, el contexto que el subagente necesita (y solo ese), restricciones explícitas (read-only, sin recomendación de librería) y un formato de salida preciso. El subagente devolverá ~30 líneas útiles en lugar de 3.000 ruidosas.

!!! tip "Si no puedes escribir el briefing, no estás listo para delegar"
    Un briefing vago produce una respuesta vaga. Si te cuesta especificar la salida esperada, probablemente la tarea no esté lo bastante definida como para delegarla todavía.

## Delegación paralela

Cuando las sub-tareas son independientes, lánzalas en paralelo en un mismo turno. Un turno típico de "explorar un código desconocido" puede lanzar tres subagentes a la vez:

- Uno mapeando la capa HTTP.
- Uno mapeando la capa de persistencia.
- Uno mapeando los background jobs.

Cada uno devuelve un resumen; el agente principal los une. Toda la exploración termina en el tiempo del subagente más lento, no en la suma.

## Subagentes especializados

Algunos equipos definen subagentes con nombre para roles comunes: un subagente *research* (read-only, búsqueda web + código), un *planner* (produce un plan, no escribe código), un subagente *code-search* (greps, lee, devuelve rutas y snippets). La especialización es útil cuando el rol se reutiliza lo bastante como para justificar un briefing estable — si no, un subagente ad-hoc con un buen prompt vale.

## El modo de fallo fatal

!!! warning "Nunca delegues el paso de síntesis"
    En el momento en que le pides a un subagente "...y luego decide qué enfoque deberíamos tomar y empieza a implementarlo", has cedido la parte más importante del trabajo. El subagente no tiene tu contexto completo, no conoce los trade-offs que ya has sopesado y no carga con la responsabilidad del resultado. La síntesis — *qué significa todo esto y qué deberíamos hacer?* — es tuya. Siempre.

## Idea clave

!!! success "Idea clave"
    Los subagentes son para proteger contexto y paralelizar — no para descargar el pensamiento. Delega la recolección, quédate con la decisión, y nunca delegues el paso de síntesis.
