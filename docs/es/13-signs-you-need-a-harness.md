# 13. Señales de que necesitas un harness

Esta guía basta para que tu equipo sea productivo con agentes de código. *No* basta para que sigas siéndolo cuando escales más allá de un puñado de developers o un puñado de repositorios. En algún momento sentirás un tipo de fricción que ningún ajuste de prompts arregla. Ese es el momento de graduarte de "usar un agente" a especificar bien lo que quieres (**SDD**) y, después, a construir el sistema que lo hace posible (**harness engineering**).

## Qué vas a aprender

- Qué es realmente un harness.
- Síntomas concretos que significan "te has quedado pequeño con esta guía."
- Por qué el arreglo es estructural, no a nivel de prompt.
- Adónde ir después.

## ¿Qué es un "harness"?

El modelo es solo una parte de un agente de código. Todo lo que está *alrededor* del modelo — el runtime que da forma a lo que ve, lo que puede hacer y lo que ocurre con su salida — es el harness. En concreto:

- **Hooks.** Código determinista que corre antes o después de las llamadas a herramientas (`PreToolUse`, `PostToolUse`, `Stop`). Aquí es donde fuerzas reglas, ejecutas linters, bloqueas comandos peligrosos e inyectas contexto.
- **Sandbox.** El entorno en el que corre el agente — devcontainer, Docker, una VM, una shell restringida. Define qué puede leer, escribir y alcanzar por red.
- **Subagentes.** Subprocesos especializados con sus propios prompts y acceso a herramientas: un revisor, un runner de tests, un investigador. Permiten descomponer el trabajo y mantener los contextos pequeños.
- **Memoria.** El sistema en capas del capítulo 11 — memoria de proyecto, memoria de usuario, skills, comandos.
- **MCP y herramientas.** El catálogo de herramientas que el agente puede llamar: sistema de ficheros, shell, búsqueda, servidores MCP propios para tus sistemas internos.
- **Guías y sensores.** Documentos que enseñan al agente *cómo* hacer algo, y checks que detectan cuando se ha salido del camino.

Un harness no es un producto — es un ensamblaje. Claude Code, Cursor, Codex y Aider vienen con un harness por defecto. Durante un tiempo, el por defecto basta. Y luego no.

Conviene dejar esto explícito: lo que aprendes en *Fundamentals* es, de hecho, a usar el harness por defecto que traen estas herramientas — el de Claude Code, el de Cursor, el de Copilot, el de Codex. Para muchos equipos y muchos proyectos, ese por defecto es suficiente, y conviene exprimirlo todo lo que dé. Cuando decimos "necesitas un harness" no queremos decir que antes no usaras ninguno, sino que el genérico se queda corto frente a las particularidades de *tu* sistema: tus reglas, tu dominio, tus integraciones, tu nivel de criticidad. Necesitas uno *tailored* — un harness construido para el sistema concreto que estás creando, manteniendo, modernizando o refactorizando. Y sí, cabe esperar que con el tiempo los harnesses por defecto absorban cada vez más capacidades y la frontera del *tailoring* necesario se vaya estrechando — pero mientras esa frontera exista, alguien en tu equipo tiene que cruzarla.

Ese cruce no es directo. Antes de invertir en construir un harness propio, casi siempre conviene pasar por **Spec-Driven Development (SDD)**: aprender a especificar la intención con claridad y a mantener esa especificación viva mientras el código evoluciona. La razón es práctica — un harness más sofisticado da al agente más autonomía, y la autonomía sin una spec clara amplifica el ruido en lugar de la productividad. SDD es el puente: te obliga a poner por escrito *qué* quieres antes de automatizar el *cómo*. Solo entonces tiene sentido el siguiente salto, **harness engineering**: llevar la ingeniería a construir el harness que permite implementar sistemas con un alto grado de autonomía, a escala, y a lo largo de todo el SDLC. Fundamentals → SDD → Harness Engineering es el camino.

## Síntomas de que te has quedado pequeño con el por defecto

Aquí tienes el checklist. Si más de dos o tres te resultan familiares, estás pasado el punto en el que prompts mejores van a ayudar.

### El agente repite los mismos errores entre developers

Alice le enseña a su sesión a no tocar la carpeta de migraciones. La sesión de Bob lo hace al día siguiente igualmente. No hay un bucle de corrección compartido — cada developer está reentrenando en privado al mismo agente.

**Qué falta:** una regla compartida, reforzada en el harness (hook o permiso), no en el prompt de una sola persona.

### Cada developer tiene su propio setup

Date una vuelta por el equipo. Alice usa Claude Code con su propio árbol `~/.claude`. Bob usa Cursor con reglas personalizadas que nadie más ha visto. Carol usa Codex con un modelo distinto directamente. Ninguna de sus configuraciones está en control de versiones.

**Qué falta:** un harness a nivel de repo, versionado. Comandos compartidos, skills compartidas, hooks compartidos.

### Las PRs del agente necesitan más review que las humanas

Estás dedicando más tiempo a revisar la salida del agente del que habrías pasado escribiéndolo tú. La verificación se ha empujado al humano en lugar de al harness.

**Qué falta:** sensores automatizados — linters, tests, type checks, escáneres de seguridad — corriendo *antes* de que la PR llegue a un revisor humano, y un bucle de feedback que haga que el agente arregle su propia salida.

### La misma clase de bug sigue volviendo

Revisas tres PRs en una semana y las tres tienen el mismo error: un null check que falta, un boundary de transacción mal puesto, un secret hardcodeado. Lo corriges cada vez. Vuelve.

**Qué falta:** un sensor que cace esta clase de bug automáticamente, más una guía que el agente cargue siempre que toque el código relevante.

### Hacer onboarding de un nuevo dev a "la forma del agente" lleva una semana

No hay ningún documento al que puedas apuntar a alguien nuevo. El conocimiento vive en cabezas y en chats privados. Cada persona nueva redescubre las mismas lecciones.

**Qué falta:** un sistema de registro. El harness *es* el onboarding — si vive en el repo, los nuevos lo heredan desde el día uno.

### Tus prompts son cada vez más largos

Te das cuenta de que empiezas cada sesión con un preámbulo de 20 líneas: "Recuerda ejecutar los tests, recuerda las reglas de migración, recuerda no tocar X, recuerda Y, recuerda Z..." Estás haciendo a mano lo que un harness debería hacer automáticamente.

**Qué falta:** guías que se cargan bajo demanda, hooks que fuerzan reglas y contexto inyectado por el harness en lugar de por ti.

### Tienes miedo de dejar al agente corriendo solo

Cada sesión es babysitting. Nunca lanzas una tarea y vuelves en 30 minutos. El riesgo de que el agente haga algo destructivo se siente demasiado alto.

**Qué falta:** un sandbox real, un modelo de permisos real y sensores que puedan parar una ejecución cuando algo pinta mal.

### Estás usando el agente en varias fases del SDLC

Cuando el mismo agente ayuda en diseño, código, QA y operaciones, necesitas memoria, skills y servidores MCP compartidos que viajen con él — no tres configuraciones desconectadas.

**Qué falta:** un único harness que arrastre contexto, skills y acceso a herramientas entre fases, para que el agente no pierda todo lo que sabe en cuanto la tarea cambia de forma.

## Por qué el prompt-engineering no arregla nada de esto

Todos estos síntomas comparten una causa raíz: **el problema no es lo que se le dice al modelo, es la ausencia de un sistema alrededor del modelo.** Un prompt más largo no puede reemplazar a un hook. Un fichero de memoria mejor no puede reemplazar a un sandbox. Una instrucción ingeniosa no puede reemplazar a un sensor automatizado. En cuanto estás intentando usar palabras para tapar huecos estructurales, estás peleando la batalla equivocada.

## Qué sigue

Cuando reconozcas estos síntomas, salta a las guías complementarias. Esta guía es el primer volumen de una trilogía; las otras dos toman el relevo desde ángulos distintos:

!!! info "Spec-Driven Development"
    Siguiente paso: **[Spec-Driven Development](https://jmlopezdona.github.io/ai-coding-agents-sdd/)**

    El cuello de botella ya no es escribir código, sino especificar la intención con claridad y mantener esa especificación viva a medida que el código evoluciona. Esta guía cubre los ciclos y patrones para que el agente ejecute consistentemente lo que de verdad querías — y cuándo esa disciplina ayuda y cuándo se convierte en burocracia.

!!! info "Harness Engineering"
    Y después: **[Harness Engineering — construir software a escala con agentes](https://jmlopezdona.github.io/ai-coding-agents-harness/)**

    Empieza exactamente donde esta termina: cómo construir las guías, sensores, bucles, sandboxes, subagentes y contexto estructurado que convierten un LLM en un agente del que un equipo puede fiarse de verdad.

!!! success "Idea clave"
    Si estás retocando prompts para arreglar problemas estructurales, te has quedado pequeño con esta guía. El siguiente paso no es un prompt mejor: es aprender a especificar bien la intención (SDD) y, cuando eso ya no escale, construir el harness que la sostiene.
