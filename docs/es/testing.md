---
title: Pruebas
summary: "Kit de pruebas: suites unitarias/e2e/en tiempo real, runners de Docker y qué cubre cada prueba"
read_when:
  - Ejecutar pruebas localmente o en CI
  - Añadir pruebas de regresión para errores de modelos/proveedores
  - Depurar el comportamiento del Gateway y de los agentes
---

<div id="testing">
  # Pruebas
</div>

OpenClaw tiene tres suites de Vitest (unitarias/de integración, e2e y live) y un pequeño conjunto de runners de Docker.

Este documento es una guía de “cómo hacemos las pruebas”:

* Qué cubre cada suite (y qué *no* cubre deliberadamente)
* Qué comandos ejecutar para flujos de trabajo comunes (local, pre-push, depuración)
* Cómo las pruebas live obtienen credenciales y seleccionan modelos/proveedores
* Cómo añadir pruebas de regresión para problemas reales con modelos/proveedores

<div id="quick-start">
  ## Inicio rápido
</div>

La mayoría de los días:

* Comprobación completa (recomendada antes de hacer push): `pnpm lint && pnpm build && pnpm test`

Cuando modifiques las pruebas o quieras más confianza:

* Comprobación con cobertura: `pnpm test:coverage`
* Suite E2E: `pnpm test:e2e`

Cuando estés depurando proveedores/modelos reales (requiere credenciales reales):

* Suite en vivo (modelos + sondeos de herramientas/imágenes del Gateway): `pnpm test:live`

Consejo: cuando solo necesites un caso que falle, es mejor acotar las pruebas en vivo mediante las variables de entorno de lista de permitidos descritas más abajo.

<div id="test-suites-what-runs-where">
  ## Suites de pruebas (qué se ejecuta dónde)
</div>

Piensa en las suites como “niveles crecientes de realismo” (y de inestabilidad/costo crecientes):

<div id="unit-integration-default">
  ### Unitarias / de integración (predeterminado)
</div>

* Comando: `pnpm test`
* Configuración: `vitest.config.ts`
* Archivos: `src/**/*.test.ts`
* Ámbito:
  * Pruebas unitarias puras
  * Pruebas de integración en proceso (auth del Gateway, routing, tooling, parsing, config)
  * Regresiones deterministas para errores conocidos
* Expectativas:
  * Se ejecutan en CI
  * No requieren claves reales
  * Las pruebas deben ser rápidas y estables

<div id="e2e-gateway-smoke">
  ### E2E (smoke del Gateway)
</div>

* Command: `pnpm test:e2e`
* Config: `vitest.e2e.config.ts`
* Files: `src/**/*.e2e.test.ts`
* Scope:
  * Comportamiento de extremo a extremo de múltiples instancias del Gateway
  * Interfaces WebSocket/HTTP, emparejamiento de nodos y redes más pesadas
* Expectativas:
  * Se ejecuta en CI (cuando está habilitado en el pipeline)
  * No se requieren claves reales
  * Más componentes en juego que en las pruebas unitarias (puede ser más lento)

<div id="live-real-providers-real-models">
  ### Live (proveedores reales + modelos reales)
</div>

* Command: `pnpm test:live`
* Config: `vitest.live.config.ts`
* Files: `src/**/*.live.test.ts`
* Default: **habilitado** mediante `pnpm test:live` (establece `OPENCLAW_LIVE_TEST=1`)
* Scope:
  * &quot;¿Este proveedor/modelo realmente funciona *hoy* con credenciales reales?&quot;
  * Detectar cambios de formato del proveedor, peculiaridades en las llamadas a herramientas, problemas de autenticación y comportamiento frente a límites de uso (rate limits)
* Expectations:
  * No es estable para CI por diseño (redes reales, políticas reales de los proveedores, cuotas, caídas)
  * Cuesta dinero / consume límites de uso
  * Es preferible ejecutar subconjuntos acotados en lugar de &quot;todo&quot;
  * Las ejecuciones Live cargarán `~/.profile` para recoger claves de API faltantes
  * Rotación de claves de Anthropic: establece `OPENCLAW_LIVE_ANTHROPIC_KEYS="sk-...,sk-..."` (o `OPENCLAW_LIVE_ANTHROPIC_KEY=sk-...`) o múltiples variables `ANTHROPIC_API_KEY*`; las pruebas reintentarán en caso de límites de uso

<div id="which-suite-should-i-run">
  ## ¿Qué suite debería ejecutar?
</div>

Usa esta tabla de decisiones:

* Editar lógica/pruebas: ejecuta `pnpm test` (y `pnpm test:coverage` si cambiaste muchas cosas)
* Modificar el networking del Gateway / protocolo WS / emparejamiento: añade `pnpm test:e2e`
* Depurar problemas de tipo «mi bot no responde» / fallos específicos de un proveedor / llamadas a herramientas: ejecuta un `pnpm test:live` acotado

<div id="live-model-smoke-profile-keys">
  ## En vivo: prueba de humo de modelo (claves de perfil)
</div>

Las pruebas en vivo se dividen en dos capas para poder aislar los fallos:

* El “modelo directo” nos indica que el proveedor/modelo al menos puede responder con la clave dada.
* La “prueba de humo del Gateway” nos indica que todo el pipeline Gateway+Agente funciona para ese modelo (sesiones, historial, herramientas, política de sandbox, etc.).

<div id="layer-1-direct-model-completion-no-gateway">
  ### Capa 1: completado directo de modelo (sin Gateway)
</div>

* Test: `src/agents/models.profiles.live.test.ts`
* Objetivo:
  * Enumerar los modelos detectados
  * Usar `getApiKeyForModel` para seleccionar los modelos para los que tienes credenciales
  * Ejecutar un pequeño completado por modelo (y regresiones dirigidas cuando sea necesario)
* Cómo habilitarlo:
  * `pnpm test:live` (o `OPENCLAW_LIVE_TEST=1` si invocas Vitest directamente)
* Establece `OPENCLAW_LIVE_MODELS=modern` (o `all`, alias de modern) para ejecutar realmente esta batería; de lo contrario se omite para mantener `pnpm test:live` centrado en el smoke del Gateway
* Cómo seleccionar modelos:
  * `OPENCLAW_LIVE_MODELS=modern` para ejecutar la lista de permitidos moderna (Opus/Sonnet/Haiku 4.5, GPT-5.x + Codex, Gemini 3, GLM 4.7, MiniMax M2.1, Grok 4)
  * `OPENCLAW_LIVE_MODELS=all` es un alias de la lista de permitidos moderna
  * o `OPENCLAW_LIVE_MODELS="openai/gpt-5.2,anthropic/claude-opus-4-5,..."` (lista de permitidos separada por comas)
* Cómo seleccionar proveedores:
  * `OPENCLAW_LIVE_PROVIDERS="google,google-antigravity,google-gemini-cli"` (lista de permitidos separada por comas)
* De dónde vienen las claves:
  * Por defecto: almacén de perfiles y variables de entorno como respaldo
  * Establece `OPENCLAW_LIVE_REQUIRE_PROFILE_KEYS=1` para forzar únicamente el **almacén de perfiles**
* Por qué existe esto:
  * Separa “la API del proveedor está rota / la clave es inválida” de “el pipeline de agentes del Gateway está roto”
  * Contiene regresiones pequeñas y aisladas (ejemplo: flujos de reasoning replay + tool-call de OpenAI Responses/Codex Responses)

<div id="layer-2-gateway-dev-agent-smoke-what-openclaw-actually-does">
  ### Capa 2: Gateway + smoke del agente de desarrollo (lo que “@openclaw” realmente hace)
</div>

* Prueba: `src/gateway/gateway-models.profiles.live.test.ts`
* Objetivo:
  * Levantar un Gateway en proceso
  * Crear/actualizar una sesión `agent:dev:*` (anulación de modelo por ejecución)
  * Iterar sobre modelos con claves y comprobar:
    * respuesta “significativa” (sin herramientas)
    * que una invocación real de herramienta funciona (sondeo `read`)
    * sondeos opcionales de herramientas adicionales (sondeo `exec+read`)
    * que las rutas de regresión de OpenAI (solo llamada de herramienta → mensaje de seguimiento) sigan funcionando
* Detalles de los sondeos (para que puedas explicar fallos rápidamente):
  * sondeo `read`: la prueba escribe un archivo con un nonce en el espacio de trabajo y le pide al agente que lo `read` y devuelva el nonce.
  * sondeo `exec+read`: la prueba pide al agente que use `exec` para escribir un nonce en un archivo temporal y luego lo `read` de vuelta.
  * sondeo de imagen: la prueba adjunta un PNG generado (gato + código aleatorio) y espera que el modelo devuelva `cat <CODE>`.
  * Referencia de implementación: `src/gateway/gateway-models.profiles.live.test.ts` y `src/gateway/live-image-probe.ts`.
* Cómo habilitarlo:
  * `pnpm test:live` (o `OPENCLAW_LIVE_TEST=1` si invocas Vitest directamente)
* Cómo seleccionar modelos:
  * Predeterminado: lista de permitidos moderna (Opus/Sonnet/Haiku 4.5, GPT-5.x + Codex, Gemini 3, GLM 4.7, MiniMax M2.1, Grok 4)
  * `OPENCLAW_LIVE_GATEWAY_MODELS=all` es un alias de la lista de permitidos moderna
  * O establece `OPENCLAW_LIVE_GATEWAY_MODELS="provider/model"` (o lista separada por comas) para acotar
* Cómo seleccionar proveedores (evita “OpenRouter todo”):
  * `OPENCLAW_LIVE_GATEWAY_PROVIDERS="google,google-antigravity,google-gemini-cli,openai,anthropic,zai,minimax"` (lista de permitidos separada por comas)
* Los sondeos de herramientas + imagen siempre están activos en esta prueba en vivo:
  * sondeo `read` + sondeo `exec+read` (estrés de herramientas)
  * el sondeo de imagen se ejecuta cuando el modelo anuncia compatibilidad con entrada de imágenes
  * Flujo (a alto nivel):
    * La prueba genera un PNG diminuto con “CAT” + código aleatorio (`src/gateway/live-image-probe.ts`)
    * Lo envía vía `agent` `attachments: [{ mimeType: "image/png", content: "<base64>" }]`
    * Gateway analiza los adjuntos en `images[]` (`src/gateway/server-methods/agent.ts` + `src/gateway/chat-attachments.ts`)
    * El agente incrustado reenvía un mensaje de usuario multimodal al modelo
    * Comprobación: la respuesta contiene `cat` + el código (tolerancia OCR: se permiten errores menores)

Consejo: para ver qué puedes probar en tu máquina (y los ID exactos de `provider/model`), ejecuta:

```bash
openclaw models list
openclaw models list --json
```

<div id="live-anthropic-setup-token-smoke">
  ## En vivo: prueba smoke de setup-token de Anthropic
</div>

* Prueba: `src/agents/anthropic.setup-token.live.test.ts`
* Objetivo: verificar que el comando `setup-token` de Claude Code CLI (o un perfil de setup-token pegado) pueda completar un prompt en Anthropic.
* Habilitar:
  * `pnpm test:live` (o `OPENCLAW_LIVE_TEST=1` si invocas Vitest directamente)
  * `OPENCLAW_LIVE_SETUP_TOKEN=1`
* Fuentes del token (elige una):
  * Perfil: `OPENCLAW_LIVE_SETUP_TOKEN_PROFILE=anthropic:setup-token-test`
  * Token sin procesar: `OPENCLAW_LIVE_SETUP_TOKEN_VALUE=sk-ant-oat01-...`
* Anulación de modelo (opcional):
  * `OPENCLAW_LIVE_SETUP_TOKEN_MODEL=anthropic/claude-opus-4-5`

Ejemplo de configuración:

```bash
openclaw models auth paste-token --provider anthropic --profile-id anthropic:setup-token-test
OPENCLAW_LIVE_SETUP_TOKEN=1 OPENCLAW_LIVE_SETUP_TOKEN_PROFILE=anthropic:setup-token-test pnpm test:live src/agents/anthropic.setup-token.live.test.ts
```

<div id="live-cli-backend-smoke-claude-code-cli-or-other-local-clis">
  ## En vivo: prueba básica de backend de CLI (Claude Code CLI u otras CLIs locales)
</div>

* Prueba: `src/gateway/gateway-cli-backend.live.test.ts`
* Objetivo: validar el flujo Gateway + agente usando un backend de CLI local, sin modificar tu configuración predeterminada.
* Activar:
  * `pnpm test:live` (o `OPENCLAW_LIVE_TEST=1` si invocas Vitest directamente)
  * `OPENCLAW_LIVE_CLI_BACKEND=1`
* Valores predeterminados:
  * Modelo: `claude-cli/claude-sonnet-4-5`
  * Comando: `claude`
  * Argumentos: `["-p","--output-format","json","--dangerously-skip-permissions"]`
* Anulaciones (opcionales):
  * `OPENCLAW_LIVE_CLI_BACKEND_MODEL="claude-cli/claude-opus-4-5"`
  * `OPENCLAW_LIVE_CLI_BACKEND_MODEL="codex-cli/gpt-5.2-codex"`
  * `OPENCLAW_LIVE_CLI_BACKEND_COMMAND="/full/path/to/claude"`
  * `OPENCLAW_LIVE_CLI_BACKEND_ARGS='["-p","--output-format","json","--permission-mode","bypassPermissions"]'`
  * `OPENCLAW_LIVE_CLI_BACKEND_CLEAR_ENV='["ANTHROPIC_API_KEY","ANTHROPIC_API_KEY_OLD"]'`
  * `OPENCLAW_LIVE_CLI_BACKEND_IMAGE_PROBE=1` para enviar un adjunto de imagen real (las rutas se inyectan en el prompt).
  * `OPENCLAW_LIVE_CLI_BACKEND_IMAGE_ARG="--image"` para pasar las rutas de archivos de imagen como argumentos de la CLI en lugar de inyectarlas en el prompt.
  * `OPENCLAW_LIVE_CLI_BACKEND_IMAGE_MODE="repeat"` (o `"list"`) para controlar cómo se pasan los argumentos de imagen cuando `IMAGE_ARG` está configurado.
  * `OPENCLAW_LIVE_CLI_BACKEND_RESUME_PROBE=1` para enviar un segundo turno y validar el flujo de reanudación.
* `OPENCLAW_LIVE_CLI_BACKEND_DISABLE_MCP_CONFIG=0` para mantener habilitada la configuración MCP de Claude Code CLI (de forma predeterminada, la configuración MCP se deshabilita con un archivo vacío temporal).

Ejemplo:

```bash
OPENCLAW_LIVE_CLI_BACKEND=1 \
  OPENCLAW_LIVE_CLI_BACKEND_MODEL="claude-cli/claude-sonnet-4-5" \
  pnpm test:live src/gateway/gateway-cli-backend.live.test.ts
```

<div id="recommended-live-recipes">
  ### Recetas en vivo recomendadas
</div>

Las listas de permitidos estrechas y explícitas son las más rápidas y las menos propensas a fallos:

* Un solo modelo, directo (sin Gateway):
  * `OPENCLAW_LIVE_MODELS="openai/gpt-5.2" pnpm test:live src/agents/models.profiles.live.test.ts`

* Un solo modelo, smoke test vía Gateway:
  * `OPENCLAW_LIVE_GATEWAY_MODELS="openai/gpt-5.2" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`

* Llamadas de herramientas entre varios proveedores:
  * `OPENCLAW_LIVE_GATEWAY_MODELS="openai/gpt-5.2,anthropic/claude-opus-4-5,google/gemini-3-flash-preview,zai/glm-4.7,minimax/minimax-m2.1" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`

* Enfoque en Google (clave de API de Gemini + Antigravity):
  * Gemini (clave de API): `OPENCLAW_LIVE_GATEWAY_MODELS="google/gemini-3-flash-preview" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`
  * Antigravity (OAuth): `OPENCLAW_LIVE_GATEWAY_MODELS="google-antigravity/claude-opus-4-5-thinking,google-antigravity/gemini-3-pro-high" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`

Notas:

* `google/...` usa la API de Gemini (clave de API).
* `google-antigravity/...` usa el puente OAuth de Antigravity (endpoint de agente estilo Cloud Code Assist).
* `google-gemini-cli/...` usa la CLI local de Gemini en tu máquina (autenticación independiente + peculiaridades de las herramientas).
* API de Gemini vs CLI de Gemini:
  * API: OpenClaw llama a la API alojada de Gemini de Google vía HTTP (autenticación por clave de API / perfil); esto es lo que la mayoría de los usuarios quiere decir con “Gemini”.
  * CLI: OpenClaw ejecuta un binario local `gemini`; tiene su propia autenticación y puede comportarse de forma diferente (compatibilidad con streaming/herramientas/desfase de versiones).

<div id="live-model-matrix-what-we-cover">
  ## En vivo: matriz de modelos (lo que cubrimos)
</div>

No hay una “lista de modelos de CI” fija (la ejecución en vivo es opcional), pero estos son los modelos **recomendados** que deberías probar regularmente en una máquina de desarrollo con claves de API.

<div id="modern-smoke-set-tool-calling-image">
  ### Conjunto moderno de pruebas smoke (llamadas a herramientas + imagen)
</div>

Este es el conjunto de «modelos comunes» que esperamos mantener funcionando:

* OpenAI (no Codex): `openai/gpt-5.2` (opcional: `openai/gpt-5.1`)
* OpenAI Codex: `openai-codex/gpt-5.2` (opcional: `openai-codex/gpt-5.2-codex`)
* Anthropic: `anthropic/claude-opus-4-5` (o `anthropic/claude-sonnet-4-5`)
* Google (Gemini API): `google/gemini-3-pro-preview` y `google/gemini-3-flash-preview` (evita los modelos Gemini 2.x más antiguos)
* Google (Antigravity): `google-antigravity/claude-opus-4-5-thinking` y `google-antigravity/gemini-3-flash`
* Z.AI (GLM): `zai/glm-4.7`
* MiniMax: `minimax/minimax-m2.1`

Ejecuta las pruebas smoke del Gateway con herramientas + imagen:
`OPENCLAW_LIVE_GATEWAY_MODELS="openai/gpt-5.2,openai-codex/gpt-5.2,anthropic/claude-opus-4-5,google/gemini-3-pro-preview,google/gemini-3-flash-preview,google-antigravity/claude-opus-4-5-thinking,google-antigravity/gemini-3-flash,zai/glm-4.7,minimax/minimax-m2.1" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`

<div id="baseline-tool-calling-read-optional-exec">
  ### Línea base: llamada a herramientas (Read + Exec opcional)
</div>

Elige al menos uno por familia de proveedores:

* OpenAI: `openai/gpt-5.2` (o `openai/gpt-5-mini`)
* Anthropic: `anthropic/claude-opus-4-5` (o `anthropic/claude-sonnet-4-5`)
* Google: `google/gemini-3-flash-preview` (o `google/gemini-3-pro-preview`)
* Z.AI (GLM): `zai/glm-4.7`
* MiniMax: `minimax/minimax-m2.1`

Cobertura adicional opcional (deseable):

* xAI: `xai/grok-4` (o el último disponible)
* Mistral: `mistral/`… (elige un modelo con capacidad de &quot;tools&quot; que tengas habilitado)
* Cerebras: `cerebras/`… (si tienes acceso)
* LM Studio: `lmstudio/`… (local; la llamada a herramientas depende del modo api)

<div id="vision-image-send-attachment-multimodal-message">
  ### Visión: envío de imagen (adjunto → mensaje multimodal)
</div>

Incluye al menos un modelo con capacidades de imagen en `OPENCLAW_LIVE_GATEWAY_MODELS` (variantes de Claude/Gemini/OpenAI con capacidades de visión, etc.) para probar la sonda de imágenes.

<div id="aggregators-alternate-gateways">
  ### Agregadores / gateways alternativos
</div>

Si tienes claves configuradas, también puedes hacer pruebas a través de:

* OpenRouter: `openrouter/...` (cientos de modelos; usa `openclaw models scan` para encontrar candidatos compatibles con herramientas e imágenes)
* OpenCode Zen: `opencode/...` (autenticación mediante `OPENCODE_API_KEY` / `OPENCODE_ZEN_API_KEY`)

Más proveedores que puedes incluir en la matriz en tiempo real (si tienes credenciales/config):

* Integrados: `openai`, `openai-codex`, `anthropic`, `google`, `google-vertex`, `google-antigravity`, `google-gemini-cli`, `zai`, `openrouter`, `opencode`, `xai`, `groq`, `cerebras`, `mistral`, `github-copilot`
* Vía `models.providers` (endpoints personalizados): `minimax` (nube/API), además de cualquier proxy compatible con OpenAI/Anthropic (LM Studio, vLLM, LiteLLM, etc.)

Consejo: no intentes codificar explícitamente “todos los modelos” en la documentación. La lista de referencia es lo que `discoverModels(...)` devuelve en tu máquina + las claves que haya disponibles.

<div id="credentials-never-commit">
  ## Credenciales (nunca las confirmes)
</div>

Las pruebas en vivo detectan credenciales de la misma forma que lo hace la CLI. Implicaciones prácticas:

* Si la CLI funciona, las pruebas en vivo deberían encontrar las mismas claves.

* Si una prueba en vivo indica “no creds”, depura de la misma manera que depurarías `openclaw models list` / la selección de modelo.

* Almacén de perfiles: `~/.openclaw/credentials/` (preferido; a esto se refiere “profile keys” en las pruebas)

* Configuración: `~/.openclaw/openclaw.json` (o `OPENCLAW_CONFIG_PATH`)

Si quieres basarte en claves de entorno (por ejemplo, exportadas en tu `~/.profile`), ejecuta pruebas locales después de `source ~/.profile`, o usa los runners de Docker de abajo (pueden montar `~/.profile` dentro del contenedor).

<div id="deepgram-live-audio-transcription">
  ## Deepgram en vivo (transcripción de audio)
</div>

* Prueba: `src/media-understanding/providers/deepgram/audio.live.test.ts`
* Habilitar: `DEEPGRAM_API_KEY=... DEEPGRAM_LIVE_TEST=1 pnpm test:live src/media-understanding/providers/deepgram/audio.live.test.ts`

<div id="docker-runners-optional-works-in-linux-checks">
  ## Runners de Docker (comprobaciones opcionales de “funciona en Linux”)
</div>

Estos ejecutan `pnpm test:live` dentro de la imagen Docker del repositorio, montando tu directorio de configuración local y el espacio de trabajo (y cargando `~/.profile` si está montado):

* Modelos directos: `pnpm test:docker:live-models` (script: `scripts/test-live-models-docker.sh`)
* Gateway + agente de desarrollo: `pnpm test:docker:live-gateway` (script: `scripts/test-live-gateway-models-docker.sh`)
* Asistente de onboarding (TTY, scaffolding completo): `pnpm test:docker:onboard` (script: `scripts/e2e/onboard-docker.sh`)
* Red del Gateway (dos contenedores, autenticación WS + comprobación de estado): `pnpm test:docker:gateway-network` (script: `scripts/e2e/gateway-network-docker.sh`)
* Complementos (carga de extensión personalizada + prueba básica del registro): `pnpm test:docker:plugins` (script: `scripts/e2e/plugins-docker.sh`)

Variables de entorno útiles:

* `OPENCLAW_CONFIG_DIR=...` (predeterminado: `~/.openclaw`) montado en `/home/node/.openclaw`
* `OPENCLAW_WORKSPACE_DIR=...` (predeterminado: `~/.openclaw/workspace`) montado en `/home/node/.openclaw/workspace`
* `OPENCLAW_PROFILE_FILE=...` (predeterminado: `~/.profile`) montado en `/home/node/.profile` y cargado antes de ejecutar las pruebas
* `OPENCLAW_LIVE_GATEWAY_MODELS=...` / `OPENCLAW_LIVE_MODELS=...` para restringir la ejecución
* `OPENCLAW_LIVE_REQUIRE_PROFILE_KEYS=1` para asegurar que las credenciales provengan del almacén de perfiles (no del entorno)

<div id="docs-sanity">
  ## Comprobación rápida de la documentación
</div>

Ejecuta las comprobaciones de la documentación después de editarla: `pnpm docs:list`.

<div id="offline-regression-ci-safe">
  ## Regresión offline (segura para CI)
</div>

Estas son regresiones de “pipeline real” sin proveedores reales:

* Llamadas a herramientas del Gateway (OpenAI simulado, Gateway real + bucle del agente): `src/gateway/gateway.tool-calling.mock-openai.test.ts`
* Asistente de configuración del Gateway (WS `wizard.start`/`wizard.next`, escribe la configuración + aplica autenticación): `src/gateway/gateway.wizard.e2e.test.ts`

<div id="agent-reliability-evals-skills">
  ## Evaluaciones de confiabilidad de agentes (habilidades)
</div>

Ya tenemos algunas pruebas seguras para CI que se comportan como “evaluaciones de confiabilidad de agentes”:

* Llamadas a herramientas simuladas a través del bucle real Gateway + agente (`src/gateway/gateway.tool-calling.mock-openai.test.ts`).
* Flujos de asistente de extremo a extremo que validan la conexión de sesiones y los efectos de configuración (`src/gateway/gateway.wizard.e2e.test.ts`).

Lo que aún falta para las habilidades (ver [Skills](/es/tools/skills)):

* **Toma de decisiones:** cuando las habilidades se listan en el prompt, ¿el agente elige la habilidad correcta (o evita las irrelevantes)?
* **Cumplimiento:** ¿el agente lee `SKILL.md` antes de usarla y sigue los pasos/argumentos requeridos?
* **Contratos de flujo de trabajo:** escenarios de múltiples turnos que comprueban el orden de las herramientas, el traspaso del historial de la sesión y los límites del sandbox.

Las evaluaciones futuras deberían seguir siendo deterministas:

* Un ejecutor de escenarios que use proveedores simulados para comprobar las llamadas a herramientas y su orden, las lecturas de archivos de habilidades y la conexión de sesiones.
* Una pequeña batería de escenarios centrados en habilidades (usar vs evitar, control de acceso, inyección de prompts).
* Evaluaciones en vivo opcionales (opt-in, controladas por entorno) solo después de que la batería segura para CI esté en su lugar.

<div id="adding-regressions-guidance">
  ## Añadir regresiones (orientación)
</div>

Cuando corrijas un problema de proveedor/modelo detectado en producción o en entorno real:

* Añade una regresión segura para CI si es posible (mock/stub del proveedor, o captura la transformación exacta de la estructura de la petición)
* Si es inherentemente solo en entorno real (rate limits, políticas de autenticación), mantén la prueba en vivo muy acotada y opcional mediante variables de entorno
* Prioriza apuntar a la capa más pequeña que detecte el error:
  * error de conversión/reproducción de peticiones del proveedor → prueba directa de modelos
  * error en la canalización de sesión/historial/herramientas del Gateway → smoke test en vivo del Gateway o prueba simulada del Gateway segura para CI