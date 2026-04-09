# 6. Subagentes y delegación

Un subagente es un agente nuevo que el agente principal lanza para tratar una sub-tarea de forma aislada. Hay tres razones para usarlos: **paralelización** (ejecutar sub-tareas independientes a la vez — búsquedas, análisis, builds, etc.), **protección del contexto** (mantener el hilo principal limpio de salidas grandes y ruidosas) y **orquestación autónoma** (un agente que coordina a otros agentes para llevar una tarea mayor de punta a punta).

## Qué vas a aprender

- Las tres razones reales para delegar a un subagente.
- Cuándo hacer el trabajo en línea en el hilo principal.
- Cómo dar un briefing a un subagente para que devuelva algo realmente útil.
- Cómo la orquestación multi-agente escala la atención de una sola persona en tareas largas.
- El único modo de fallo que arruina la delegación: externalizar el pensamiento que te tocaba a ti.

## Las tres razones reales

### 1. Protección del contexto

La conversación del agente principal es preciosa. Cada token gastado en una salida de `grep` de 4.000 líneas es un token no gastado en el problema real. Un subagente puede ejecutar ese `grep`, leer las coincidencias y devolver *un párrafo* de síntesis. El ruido se queda en el contexto del subagente y muere con él.

### 2. Paralelización

Tres preguntas independientes — "¿dónde se gestiona auth?", "¿cómo es el modelo de usuario?", "¿cómo se envían los emails?" — pueden ser respondidas por tres subagentes a la vez. Hacerlas secuencialmente en el hilo principal solo es más lento sin ningún beneficio.

### 3. Orquestación autónoma

Un agente puede estar construido específicamente para coordinar a otros agentes — ver "Orquestación multi-agente" más abajo. Esto es cualitativamente distinto de las dos primeras razones: no va de proteger una conversación conducida por una persona, va de escalar la atención de una sola persona a trabajo que de otra forma requeriría supervisión constante.

!!! note "Dos razones protegen el contexto principal, una lo extiende"
    Paralelización y protección del contexto son sobre que una sesión conducida por una persona se mantenga afilada. La orquestación es sobre sacar a la persona del bucle interno por completo en tareas bien definidas y verificables. Si no aplica ninguna de las tres, no necesitas un subagente.

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
| Coordinar una tarea multi-paso de punta a punta con verificación | No | Sí — patrón orquestador (ver más abajo) |

El patrón para sesiones conducidas por una persona: **delega la recolección, quédate con la decisión.** El patrón para orquestación: **delega el bucle, quédate con el objetivo y el criterio de aceptación.**

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

## Orquestación multi-agente: agentes que llaman a agentes

Todo lo anterior describe a una persona conduciendo un único agente que ocasionalmente delega. Pero el mismo primitivo compone un nivel más arriba: un agente puede estar construido como un **orquestador** cuya tarea principal es llamar a *otros* agentes, leer sus salidas, decidir el siguiente paso y correr el bucle de forma autónoma hasta que una tarea mayor esté hecha. El humano hace el briefing al orquestador una vez; el orquestador coordina el resto.

Una forma realista:

- Un agente **orquestador** recibe "saca un nuevo endpoint para X".
- Llama a un subagente *research* para mapear el código existente.
- Llama a un subagente *planner* para producir un plan paso a paso.
- Llama a un subagente *coder* para implementar cada paso.
- Llama a un subagente *reviewer* (o ejecuta tests/lint como herramientas) para verificar.
- Si el reviewer falla, vuelve al coder con el feedback.
- Solo cuando el bucle de verificación cierra, devuelve el control al humano.

Cada subagente se mantiene estrecho y sin estado; el orquestador lleva la intención de alto nivel y la síntesis. Así es como escalas la atención de una sola persona a trabajo largo y multi-paso — y es el puente natural entre "yo conduzco al agente" y "el agente lleva una fase del SDLC de punta a punta".

!!! warning "La orquestación multiplica tanto el leverage como el radio de impacto"
    Un subagente que entrega código roto desperdicia minutos. Un orquestador autónomo que entrega código roto durante una hora desperdicia una hora. Los setups multi-agente necesitan bucles de verificación **más fuertes**, condiciones de parada más claras y sandboxing más estricto — no más laxo. No tires de orquestación hasta que tu historia de verificación de un solo agente sea sólida (ver capítulo 1).

## El modo de fallo fatal

!!! warning "Nunca delegues la síntesis que te tocaba a ti"
    En una sesión conducida por una persona, en el momento en que le pides a un subagente "...y luego decide qué enfoque deberíamos tomar y empieza a implementarlo", has cedido la parte más importante del trabajo. El subagente no tiene tu contexto completo, no conoce los trade-offs que ya has sopesado y no carga con la responsabilidad del resultado.

    La orquestación parece una excepción pero no lo es: un orquestador *sí* sintetiza sobre sus subagentes — ese es su trabajo. Lo que se queda con la persona es un nivel por encima: decidir **qué problema resolver**, **qué significa "hecho"** y **cómo se va a verificar el éxito**. Delega eso, y ninguna cantidad de orquestación te salvará.

## Idea clave

!!! success "Idea clave"
    Los subagentes son para proteger contexto, paralelizar y orquestar trabajo autónomo — no para descargar el pensamiento que define la tarea. Delega la recolección y el bucle; quédate con el objetivo, los trade-offs y el criterio de aceptación.
