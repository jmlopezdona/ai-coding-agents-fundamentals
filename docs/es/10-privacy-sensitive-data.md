# 10. Privacidad y datos sensibles: qué sale de tu máquina

Cada token que tu agente lee — un fichero, la salida de un comando, el cuerpo de una llamada a una herramienta MCP, una página web descargada — acaba en la petición que envía al proveedor del modelo. La palabra "local" confunde aquí: el editor corre local, el proceso del agente corre local, pero el *razonamiento* ocurre en las GPUs de otra persona. Si no te sentirías cómodo pegando un texto en un formulario web de terceros, tampoco deberías dejar que el agente lo lea.

Esa única constatación es el capítulo entero. El resto es aprender a actuar en consecuencia.

## Qué vas a aprender

- Por qué cada token en la ventana de contexto ha salido, por definición, de tu máquina.
- Las categorías de datos que tienes que mantener fuera del agente, y las formas habituales en que se cuelan igualmente.
- Cómo evaluar el plan y la política de datos de un proveedor *antes* de adoptarlo en una empresa.
- Las defensas locales en capas que cazan lo que la política no puede.
- Un checklist del primer mes que puedes ejecutar de verdad.

## El modelo mental: los tokens viajan

Cualquier cosa que entre en la ventana de contexto del agente ha sido transmitida al proveedor del modelo. No "puede que lo haya sido". *Lo ha sido*. No existe un modo only-local para un LLM alojado: la petición, con cada byte de contexto adjunto, cruza la red.

Esto incluye cosas que no se sienten como "input":

- Los ficheros que el agente lee con `Read` o `grep`.
- La salida de cada comando bash que ejecuta (`cat secrets.env`, `psql -c 'select * from users limit 10'`, `kubectl get secret ...`).
- Resultados de herramientas MCP — filas devueltas por un MCP de base de datos, tickets devueltos por un MCP de Jira, páginas devueltas por un MCP de navegador.
- Mensajes de error, stack traces y logs que el agente vuelve a pegar para depurar.
- Contenido web descargado, incluyendo comentarios HTML ocultos y metadatos.

Si está en la ventana, está en el cable. Tu trabajo es decidir, conscientemente, qué estás dispuesto a meter en esa ventana.

## Categorías de datos a vigilar

Los sospechosos habituales, más o menos ordenados por lo caro que sale el error:

- **PII de clientes** — nombres, emails, teléfonos, direcciones, DNIs.
- **Datos de salud** — cualquier cosa tocada por HIPAA o regímenes equivalentes.
- **Datos financieros y de pago** — números de tarjeta, detalles de cuenta, cualquier cosa dentro del alcance PCI.
- **Secretos** — API keys, tokens OAuth, claves SSH privadas, certificados TLS, contraseñas de base de datos, contenido de `.env`.
- **Código propietario bajo NDA** — código de clientes, targets de adquisición, productos pre-lanzamiento.
- **Datos regulados** — cualquier cosa cubierta por GDPR, HIPAA, PCI-DSS, SOX o normativa sectorial.
- **Borradores legales y de M&A** — contratos, term sheets, dossieres de due diligence.
- **Estrategia interna** — roadmaps no publicados, modelos de pricing, presentaciones de board.

Para la mayoría, la pregunta no es "¿se lo va a memorizar el modelo?" — es "¿mi contrato con el proveedor permite realmente que esta categoría de datos salga de la empresa?".

## Vectores de fuga habituales

Casi todos los incidentes reales empiezan igual: alguien no se dio cuenta de que el agente estaba leyendo un fichero.

- El agente abre `.env` "para entender cómo está configurado el servicio" y el fichero entero acaba en el contexto.
- Un developer pega un dump de la base de datos de producción en el chat para depurar una consulta que falla.
- El agente hace tail de los logs de prod que por casualidad contienen emails de clientes y tokens de sesión.
- El agente hace `curl` a un endpoint de admin interno "para ver la forma de la respuesta" y el JSON entero va directo a la ventana.
- Un servidor MCP está conectado a un sistema sensible (CRM, ticketing, data warehouse) sin filtrado, así que cualquier consulta trae filas reales de clientes.
- Un README o comentario de issue descargado contiene un payload de prompt injection que pide al agente leer `~/.aws/credentials` e incluir el contenido en su siguiente mensaje.

Ninguno de estos parece imprudente en el momento. Todos parecen "el agente estaba simplemente haciendo su trabajo".

## El primer filtro: plan del proveedor y política de entrenamiento

Esta es la decisión más importante con diferencia, y la que la mayoría de equipos equivoca por defecto. Antes de hablar de hooks y sandboxes, tienes que elegir un canal cuyo contrato diga que tus datos no se usan para entrenar y no se retienen más allá de lo que aceptes explícitamente.

El panorama, a grandes rasgos:

- **APIs directas (Anthropic, OpenAI, Google AI, etc.).** Por defecto *no* entrenan con inputs ni outputs de API. Este es el canal "seguro por defecto" para uso corporativo: pagas por token, obtienes un DPA, y la retención es típicamente corta y configurable (incluyendo retención cero bajo petición en algunos proveedores).
- **Tiers Enterprise y Team de productos de consumo (ChatGPT Enterprise/Team, Claude Team/Enterprise, Copilot Business/Enterprise, Gemini for Workspace).** Garantías contractuales de no entrenamiento, SOC 2, residencia de datos configurable, BAAs disponibles para cargas HIPAA. Están diseñados para que los compre una empresa, no un individuo.
- **Planes Free y Pro individuales (ChatGPT Free/Plus, Claude Free/Pro, Gemini de consumo).** Las políticas varían y cambian. En muchos casos, los inputs *pueden* usarse para mejorar el modelo salvo que el usuario haga opt-out manualmente en un panel de ajustes. Estos tiers están pensados para individuos y **no son apropiados para datos corporativos**, por muy bueno que sea el modelo que hay detrás.
- **Productos integrados de terceros (Cursor, Windsurf, Aider con backend SaaS, y similares).** Tu flujo de datos depende de dos políticas a la vez: la del propio producto y la del proveedor del modelo subyacente. Lee las dos. Algunas de estas herramientas ofrecen "bring your own key" para que tu tráfico vaya directamente por un canal de API que tú controlas — ese modo suele ser la forma más segura de usarlas en una empresa.

!!! warning "Sin DPA, no hay datos de empresa"
    Si la página de precios de una herramienta solo lista Free / Pro / Team sin un Data Processing Agreement descargable, todavía no es un producto listo para empresa — por muy bueno que sea el modelo. Espera al tier enterprise, o enruta a través de una API de proveedor con la que ya tengas contrato.

## Qué verificar antes de adoptar una herramienta en una empresa

Cuando un defensor interno dice "vamos a sacarle X al equipo", este es el checklist que impide que firmes un juicio:

1. **Política escrita de no entrenamiento** — en el contrato, no en un post de blog ni en un tuit.
2. **Retención de datos** — ¿se almacenan inputs y outputs? ¿cuánto tiempo? ¿hay retención cero bajo petición?
3. **Subprocesadores** — usando el término GDPR. ¿Quién más en la cadena toca los datos? ¿Están listados y cambian con aviso previo?
4. **Region pinning** — ¿puedes forzar procesamiento solo en US o solo en EU?
5. **Certificaciones** — SOC 2 Type II, ISO 27001, HIPAA BAA, FedRAMP donde aplique. Pide los reports actuales.
6. **Audit logs** — ¿puedes ver qué prompts enviaron tus usuarios, qué herramientas corrieron y qué ficheros se leyeron?
7. **SSO y SCIM** — la base para provisionar, desprovisionar y revisar accesos.
8. **DLP / scrubbing de PII nativo** — ¿el propio producto detecta y redacta secretos y PII obvios antes de que lleguen al modelo?

Ninguna de estas preguntas es ingeniosa. Son el mismo checklist que ya aplicas a cualquier SaaS que toque datos de clientes; las herramientas de IA no son una excepción especial.

## Defensas en capas en el lado local

Incluso con el plan correcto, quieres cinturón y tirantes mecánicos en la propia máquina. La mayoría de estos se conectan a la misma maquinaria del capítulo 7:

- **Datos en sandbox.** Trabaja contra fixtures sintéticos, no contra dumps reales de producción. Mantén un dataset pequeño y explícitamente "seguro para compartir" y apunta el agente ahí. Si necesitas datos de prod para un bug, anonimízalos primero.
- **Redacción y scrubbing.** Un hook `PreToolUse` o `UserPromptSubmit` que pase el payload saliente por un escáner de secretos (gitleaks, trufflehog) y un scrubber de PII (Microsoft Presidio, regex propios) antes de que el modelo lo vea. Bloquea ante coincidencias; no te limites a avisar.
- **Allowlist y blocklist de rutas.** El agente no debería poder leer `/etc`, `~/.ssh`, `~/.aws`, `~/.kube`, el export de tu gestor de contraseñas ni nada fuera del workspace. Fuerza esto en el harness, no pidiéndolo por favor.
- **Permisos de servidores MCP.** Usuarios de base de datos de solo lectura, scopes OAuth mínimos, nada de credenciales de prod para el agente. Asume que a cada herramienta MCP se le acabará pidiendo todo lo que pueda devolver — dale lo mínimo con lo que pueda funcionar.
- **Telemetría off donde aplique.** Muchas integraciones de editor y productos de agente tienen opt-outs para telemetría, crash reports y toggles de "ayuda a mejorar el modelo". Revísalos una vez, de forma centralizada, y documenta el estado elegido.

## El factor humano

El modo de fallo que ninguna política ni hook arregla del todo es el developer que, bajo presión de deadline, pega un stack trace de producción — con tokens y user IDs todavía dentro — en el chat "solo para ver qué piensa el agente". La tecnología puede hacerlo más difícil, pero no imposible.

La única defensa real es decidir qué es y qué no es aceptable *antes* de que llegue la presión, y entrenar a la gente al mismo nivel que la entrenas en phishing e higiene de contraseñas. Escríbelo en una página. Camina al equipo por ello. Hazlo otra vez cuando entre alguien nuevo. La política tiene que existir antes del incidente, no redactarse en respuesta a uno.

## Checklist del primer mes

Si estás montando esto para un equipo por primera vez:

- Elige una herramienta o plan listo para empresa. No una cuenta Pro personal.
- Firma un DPA con el proveedor (y, cuando aplique, un BAA).
- Configura una allowlist de rutas del workspace; bloquea explícitamente `~/.ssh`, `~/.aws`, `.env*` y cualquier directorio de secretos.
- Instala un hook de escaneo de secretos (`PreToolUse` o equivalente) que bloquee ante coincidencias.
- Construye un dataset sintético pequeño y hazlo el default para la depuración asistida por el agente.
- Escribe una política de una página cubriendo qué categorías de datos están permitidas, qué canal usar y qué hacer ante una sospecha de fuga.
- Corre un tabletop exercise de 30 minutos: "un developer pegó un dump de prod en el agente — ¿qué pasa ahora?".

Nada de esto es heroico. Todo se paga solo la primera vez que no tienes que hacer una llamada telefónica incómoda.

!!! success "Idea clave"
    Cada token viaja. Tu primera línea de defensa es el proveedor y el plan que eliges — nada de lo que hagas en local arregla un mal contrato. Datos en sandbox, allowlists de rutas, hooks de redacción, servidores MCP con mínimo privilegio y formación continua se apilan encima de esa elección, no la sustituyen.
