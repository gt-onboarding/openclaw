---
title: Preguntas frecuentes
summary: "Preguntas frecuentes sobre la instalación, configuración y uso de OpenClaw"
---

<div id="faq">
  # Preguntas frecuentes (FAQ)
</div>

Respuestas rápidas y solución de problemas más en profundidad para entornos reales (desarrollo local, VPS, multiagente, claves OAuth/API, conmutación por error de modelos). Para diagnósticos en tiempo de ejecución, consulta [Solución de problemas](/es/gateway/troubleshooting). Para la referencia completa de configuración, consulta [Configuración](/es/gateway/configuration).

<div id="table-of-contents">
  ## Índice de contenidos
</div>

* [Inicio rápido y configuración de la primera ejecución](#quick-start-and-firstrun-setup)
  * [Estoy atascado, ¿cuál es la forma más rápida de desbloquearme?](#im-stuck-whats-the-fastest-way-to-get-unstuck)
  * [¿Cuál es la forma recomendada de instalar y configurar OpenClaw?](#whats-the-recommended-way-to-install-and-set-up-openclaw)
  * [¿Cómo abro el dashboard después del onboarding?](#how-do-i-open-the-dashboard-after-onboarding)
  * [¿Cómo autentico el dashboard (token) en localhost vs remoto?](#how-do-i-authenticate-the-dashboard-token-on-localhost-vs-remote)
  * [¿Qué entorno de ejecución necesito?](#what-runtime-do-i-need)
  * [¿Se ejecuta en Raspberry Pi?](#does-it-run-on-raspberry-pi)
  * [¿Algún consejo para instalaciones en Raspberry Pi?](#any-tips-for-raspberry-pi-installs)
  * [Se queda atascado en &quot;wake up my friend&quot; / el onboarding no termina de arrancar. ¿Y ahora qué?](#it-is-stuck-on-wake-up-my-friend-onboarding-will-not-hatch-what-now)
  * [¿Puedo migrar mi configuración a una máquina nueva (Mac mini) sin repetir el onboarding?](#can-i-migrate-my-setup-to-a-new-machine-mac-mini-without-redoing-onboarding)
  * [¿Dónde veo qué hay de nuevo en la última versión?](#where-do-i-see-whats-new-in-the-latest-version)
  * [No puedo acceder a docs.openclaw.ai (error SSL). ¿Y ahora qué?](#i-cant-access-docsopenclawai-ssl-error-what-now)
  * [¿Cuál es la diferencia entre stable y beta?](#whats-the-difference-between-stable-and-beta)
* [¿Cómo instalo la versión beta y en qué se diferencia de dev?](#how-do-i-install-the-beta-version-and-whats-the-difference-between-beta-and-dev)
  * [¿Cómo pruebo las versiones más recientes?](#how-do-i-try-the-latest-bits)
  * [¿Cuánto tiempo suele llevar la instalación y la configuración inicial?](#how-long-does-install-and-onboarding-usually-take)
  * [¿El instalador se ha quedado atascado? ¿Cómo obtengo más información?](#installer-stuck-how-do-i-get-more-feedback)
  * [La instalación en Windows indica que no se encuentra git o que no se reconoce openclaw](#windows-install-says-git-not-found-or-openclaw-not-recognized)
  * [La documentación no respondió mi pregunta; ¿cómo puedo obtener una mejor respuesta?](#the-docs-didnt-answer-my-question-how-do-i-get-a-better-answer)
  * [¿Cómo instalo OpenClaw en Linux?](#how-do-i-install-openclaw-on-linux)
  * [¿Cómo instalo OpenClaw en un VPS?](#how-do-i-install-openclaw-on-a-vps)
  * [¿Dónde están las guías de instalación para la nube/VPS?](#where-are-the-cloudvps-install-guides)
  * [¿Puedo hacer que OpenClaw se actualice solo?](#can-i-ask-openclaw-to-update-itself)
  * [¿Qué hace realmente el asistente de configuración inicial?](#what-does-the-onboarding-wizard-actually-do)
  * [¿Necesito una suscripción a Claude u OpenAI para usar esto?](#do-i-need-a-claude-or-openai-subscription-to-run-this)
  * [¿Puedo usar la suscripción a Claude Max sin una clave de API?](#can-i-use-claude-max-subscription-without-an-api-key)
  * [¿Cómo funciona la autenticación &quot;setup-token&quot; de Anthropic?](#how-does-anthropic-setuptoken-auth-work)
  * [¿Dónde puedo encontrar el token de configuración de Anthropic?](#where-do-i-find-an-anthropic-setuptoken)
  * [¿Son compatibles con la autenticación por suscripción de Claude (Claude Code OAuth)?](#do-you-support-claude-subscription-auth-claude-code-oauth)
  * [¿Por qué aparece el error `HTTP 429: rate_limit_error` de Anthropic?](#why-am-i-seeing-http-429-ratelimiterror-from-anthropic)
  * [¿Es compatible AWS Bedrock?](#is-aws-bedrock-supported)
  * [¿Cómo funciona la autenticación en Codex?](#how-does-codex-auth-work)
  * [¿Son compatibles con la autenticación por suscripción de OpenAI (Codex OAuth)?](#do-you-support-openai-subscription-auth-codex-oauth)
  * [¿Cómo configuro OAuth en la CLI de Gemini?](#how-do-i-set-up-gemini-cli-oauth)
  * [¿Es adecuado un modelo local para chats informales?](#is-a-local-model-ok-for-casual-chats)
  * [¿Cómo puedo mantener el tráfico de modelos alojados en una región específica?](#how-do-i-keep-hosted-model-traffic-in-a-specific-region)
  * [¿Necesito comprar un Mac mini para instalarlo?](#do-i-have-to-buy-a-mac-mini-to-install-this)
  * [¿Necesito un Mac mini para usar iMessage?](#do-i-need-a-mac-mini-for-imessage-support)
  * [Si compro un Mac mini para ejecutar OpenClaw, ¿puedo conectarlo a mi MacBook Pro?](#if-i-buy-a-mac-mini-to-run-openclaw-can-i-connect-it-to-my-macbook-pro)
  * [¿Puedo usar Bun?](#can-i-use-bun)
  * [Telegram: ¿qué debe ir en `allowFrom`?](#telegram-what-goes-in-allowfrom)
  * [¿Pueden varias personas usar el mismo número de WhatsApp con diferentes instancias de OpenClaw?](#can-multiple-people-use-one-whatsapp-number-with-different-openclaw-instances)
  * [¿Puedo ejecutar un agente de «chat rápido» y un agente de «Opus para programación»?](#can-i-run-a-fast-chat-agent-and-an-opus-for-coding-agent)
  * [¿Funciona Homebrew en Linux?](#does-homebrew-work-on-linux)
  * [¿Cuál es la diferencia entre la instalación “hackable” (git) y la instalación con npm?](#whats-the-difference-between-the-hackable-git-install-and-npm-install)
  * [¿Puedo cambiar entre instalaciones de npm y de git más adelante?](#can-i-switch-between-npm-and-git-installs-later)
  * [¿Debería ejecutar el Gateway en mi portátil o en un VPS?](#should-i-run-the-gateway-on-my-laptop-or-a-vps)
  * [¿Qué tan importante es ejecutar OpenClaw en una máquina dedicada?](#how-important-is-it-to-run-openclaw-on-a-dedicated-machine)
  * [¿Cuáles son los requisitos mínimos para un VPS y el sistema operativo recomendado?](#what-are-the-minimum-vps-requirements-and-recommended-os)
  * [¿Puedo ejecutar OpenClaw en una VM y cuáles son los requisitos?](#can-i-run-openclaw-in-a-vm-and-what-are-the-requirements)
* [¿Qué es OpenClaw?](#what-is-openclaw)
  * [¿Qué es OpenClaw, en un párrafo?](#what-is-openclaw-in-one-paragraph)
  * [¿Cuál es la propuesta de valor?](#whats-the-value-proposition)
  * [Acabo de configurarlo, ¿qué debería hacer primero?](#i-just-set-it-up-what-should-i-do-first)
  * [¿Cuáles son los cinco principales casos de uso cotidianos de OpenClaw?](#what-are-the-top-five-everyday-use-cases-for-openclaw)
  * [¿Puede OpenClaw ayudar con la generación de leads, outreach, anuncios y blogs para un SaaS?](#can-openclaw-help-with-lead-gen-outreach-ads-and-blogs-for-a-saas)
  * [¿Cuáles son las ventajas frente a Claude Code para desarrollo web?](#what-are-the-advantages-vs-claude-code-for-web-development)
* [Habilidades y automatización](#skills-and-automation)
  * [¿Cómo personalizo habilidades sin ensuciar el repositorio?](#how-do-i-customize-skills-without-keeping-the-repo-dirty)
  * [¿Puedo cargar habilidades desde una carpeta personalizada?](#can-i-load-skills-from-a-custom-folder)
  * [¿Cómo puedo usar diferentes modelos para distintas tareas?](#how-can-i-use-different-models-for-different-tasks)
  * [El bot se congela mientras realiza trabajo pesado. ¿Cómo puedo delegar esa carga de trabajo?](#the-bot-freezes-while-doing-heavy-work-how-do-i-offload-that)
  * [Cron o los recordatorios no se ejecutan. ¿Qué debo revisar?](#cron-or-reminders-do-not-fire-what-should-i-check)
  * [¿Cómo instalo habilidades en Linux?](#how-do-i-install-skills-on-linux)
  * [¿Puede OpenClaw ejecutar tareas de forma programada o continuamente en segundo plano?](#can-openclaw-run-tasks-on-a-schedule-or-continuously-in-the-background)
  * [¿Puedo ejecutar habilidades exclusivas de Apple/macOS desde Linux?](#can-i-run-applemacosonly-skills-from-linux)
  * [¿Tienen una integración con Notion o HeyGen?](#do-you-have-a-notion-or-heygen-integration)
  * [¿Cómo instalo la extensión de Chrome para controlar el navegador?](#how-do-i-install-the-chrome-extension-for-browser-takeover)
* [Sandbox y memoria](#sandboxing-and-memory)
  * [¿Hay una documentación específica sobre sandboxing?](#is-there-a-dedicated-sandboxing-doc)
  * [¿Cómo monto una carpeta del host dentro del sandbox?](#how-do-i-bind-a-host-folder-into-the-sandbox)
  * [¿Cómo funciona la memoria?](#how-does-memory-work)
  * [La memoria sigue olvidando cosas. ¿Cómo hago para que no las olvide?](#memory-keeps-forgetting-things-how-do-i-make-it-stick)
  * [¿La memoria persiste para siempre? ¿Cuáles son los límites?](#does-memory-persist-forever-what-are-the-limits)
  * [¿La búsqueda de memoria semántica requiere una clave de API de OpenAI?](#does-semantic-memory-search-require-an-openai-api-key)
* [Dónde se almacenan los datos en disco](#where-things-live-on-disk)
  * [¿Todos los datos que se usan con OpenClaw se guardan localmente?](#is-all-data-used-with-openclaw-saved-locally)
  * [¿Dónde guarda OpenClaw sus datos?](#where-does-openclaw-store-its-data)
  * [¿Dónde deben ubicarse AGENTS.md / SOUL.md / USER.md / MEMORY.md?](#where-should-agentsmd-soulmd-usermd-memorymd-live)
  * [¿Cuál es la estrategia de copia de seguridad recomendada?](#whats-the-recommended-backup-strategy)
  * [¿Cómo desinstalo completamente OpenClaw?](#how-do-i-completely-uninstall-openclaw)
  * [¿Pueden los agentes trabajar fuera del espacio de trabajo?](#can-agents-work-outside-the-workspace)
  * [Estoy en modo remoto: ¿dónde se encuentra el almacén de sesiones?](#im-in-remote-mode-where-is-the-session-store)
* [Aspectos básicos de la configuración](#config-basics)
  * [¿En qué formato está la configuración? ¿Dónde se encuentra?](#what-format-is-the-config-where-is-it)
  * [Configuré `gateway.bind: "lan"` (o `"tailnet"`) y ahora no hay nada escuchando / la UI indica «unauthorized»](#i-set-gatewaybind-lan-or-tailnet-and-now-nothing-listens-the-ui-says-unauthorized)
  * [¿Por qué ahora necesito un token en localhost?](#why-do-i-need-a-token-on-localhost-now)
  * [¿Tengo que reiniciar después de cambiar la configuración?](#do-i-have-to-restart-after-changing-config)
  * [¿Cómo habilito la búsqueda en la web (y la obtención de contenido web)?](#how-do-i-enable-web-search-and-web-fetch)
  * [`config.apply` borró mi configuración. ¿Cómo la recupero y evito que vuelva a ocurrir?](#configapply-wiped-my-config-how-do-i-recover-and-avoid-this)
  * [¿Cómo ejecuto un Gateway central con workers especializados en varios dispositivos?](#how-do-i-run-a-central-gateway-with-specialized-workers-across-devices)
  * [¿Puede ejecutarse el navegador de OpenClaw en modo headless?](#can-the-openclaw-browser-run-headless)
  * [¿Cómo uso Brave para el control del navegador?](#how-do-i-use-brave-for-browser-control)
* [Gateways y nodos remotos](#remote-gateways-nodes)
  * [¿Cómo se propagan los comandos entre Telegram, el Gateway y los nodos?](#how-do-commands-propagate-between-telegram-the-gateway-and-nodes)
  * [¿Cómo puede mi agente acceder a mi computadora si el Gateway está alojado de forma remota?](#how-can-my-agent-access-my-computer-if-the-gateway-is-hosted-remotely)
  * [Tailscale está conectado pero no recibo ninguna respuesta. ¿Y ahora qué?](#tailscale-is-connected-but-i-get-no-replies-what-now)
  * [¿Pueden dos instancias de OpenClaw comunicarse entre sí (local + VPS)?](#can-two-openclaw-instances-talk-to-each-other-local-vps)
  * [¿Necesito VPS independientes para varios agentes?](#do-i-need-separate-vpses-for-multiple-agents)
  * [¿Hay alguna ventaja en usar un nodo en mi portátil personal en lugar de usar SSH desde un VPS?](#is-there-a-benefit-to-using-a-node-on-my-personal-laptop-instead-of-ssh-from-a-vps)
  * [¿Los nodos ejecutan un servicio de Gateway?](#do-nodes-run-a-gateway-service)
  * [¿Hay alguna forma de aplicar la configuración mediante API o RPC?](#is-there-an-api-rpc-way-to-apply-config)
  * [¿Cuál es una configuración mínima «razonable» para una primera instalación?](#whats-a-minimal-sane-config-for-a-first-install)
  * [¿Cómo configuro Tailscale en un VPS y me conecto desde mi Mac?](#how-do-i-set-up-tailscale-on-a-vps-and-connect-from-my-mac)
  * [¿Cómo conecto un nodo Mac a un Gateway remoto (Tailscale Serve)?](#how-do-i-connect-a-mac-node-to-a-remote-gateway-tailscale-serve)
  * [¿Debería instalarlo en un segundo portátil o simplemente añadir un nodo?](#should-i-install-on-a-second-laptop-or-just-add-a-node)
* [Variables de entorno y carga desde .env](#env-vars-and-env-loading)
  * [¿Cómo carga OpenClaw las variables de entorno?](#how-does-openclaw-load-environment-variables)
  * [«Inicié el Gateway como servicio y mis variables de entorno desaparecieron». ¿Y ahora qué?](#i-started-the-gateway-via-the-service-and-my-env-vars-disappeared-what-now)
  * [He definido `COPILOT_GITHUB_TOKEN`, pero el estado de los modelos muestra «Shell env: off». ¿Por qué?](#i-set-copilotgithubtoken-but-models-status-shows-shell-env-off-why)
* [Sesiones y varios chats](#sessions-multiple-chats)
  * [¿Cómo inicio una conversación nueva desde cero?](#how-do-i-start-a-fresh-conversation)
  * [¿Las sesiones se reinician automáticamente si nunca envío `/new`?](#do-sessions-reset-automatically-if-i-never-send-new)
  * [¿Hay alguna forma de crear un equipo de instancias de OpenClaw, con un CEO y muchos agentes?](#is-there-a-way-to-make-a-team-of-openclaw-instances-one-ceo-and-many-agents)
  * [¿Por qué se truncó el contexto a mitad de una tarea? ¿Cómo lo evito?](#why-did-context-get-truncated-midtask-how-do-i-prevent-it)
  * [¿Cómo restablezco por completo OpenClaw pero lo mantengo instalado?](#how-do-i-completely-reset-openclaw-but-keep-it-installed)
  * [Estoy recibiendo errores de “context too large”: ¿cómo lo reinicio o lo compacto?](#im-getting-context-too-large-errors-how-do-i-reset-or-compact)
  * [¿Por qué veo “LLM request rejected: messages.N.content.X.tool&#95;use.input: Field required”?](#why-am-i-seeing-llm-request-rejected-messagesncontentxtooluseinput-field-required)
  * [¿Por qué estoy recibiendo mensajes de latido cada 30 minutos?](#why-am-i-getting-heartbeat-messages-every-30-minutes)
  * [¿Necesito añadir una “cuenta de bot” a un grupo de WhatsApp?](#do-i-need-to-add-a-bot-account-to-a-whatsapp-group)
  * [¿Cómo obtengo el JID de un grupo de WhatsApp?](#how-do-i-get-the-jid-of-a-whatsapp-group)
  * [¿Por qué OpenClaw no responde en un grupo?](#why-doesnt-openclaw-reply-in-a-group)
  * [¿Los grupos/hilos comparten contexto con los mensajes directos (DM)?](#do-groupsthreads-share-context-with-dms)
  * [¿Cuántos espacios de trabajo y agentes puedo crear?](#how-many-workspaces-and-agents-can-i-create)
  * [¿Puedo ejecutar varios bots o chats al mismo tiempo (Slack) y cómo debo configurarlo?](#can-i-run-multiple-bots-or-chats-at-the-same-time-slack-and-how-should-i-set-that-up)
* [Modelos: valores predeterminados, selección, alias, conmutación](#models-defaults-selection-aliases-switching)
  * [¿Qué es el “modelo predeterminado”?](#what-is-the-default-model)
  * [¿Qué modelo recomiendan?](#what-model-do-you-recommend)
  * [¿Cómo cambio de modelo sin borrar mi configuración?](#how-do-i-switch-models-without-wiping-my-config)
  * [¿Puedo usar modelos autoalojados (llama.cpp, vLLM, Ollama)?](#can-i-use-selfhosted-models-llamacpp-vllm-ollama)
  * [¿Qué usan OpenClaw, Flawd y Krill para los modelos?](#what-do-openclaw-flawd-and-krill-use-for-models)
  * [¿Cómo cambio de modelo sobre la marcha (sin reiniciar)?](#how-do-i-switch-models-on-the-fly-without-restarting)
  * [¿Puedo usar GPT 5.2 para tareas diarias y Codex 5.2 para programar?](#can-i-use-gpt-52-for-daily-tasks-and-codex-52-for-coding)
  * [¿Por qué veo “Model … is not allowed” y luego no hay respuesta?](#why-do-i-see-model-is-not-allowed-and-then-no-reply)
  * [¿Por qué veo “Unknown model: minimax/MiniMax-M2.1”?](#why-do-i-see-unknown-model-minimaxminimaxm21)
  * [¿Puedo usar MiniMax como mi predeterminado y OpenAI para tareas complejas?](#can-i-use-minimax-as-my-default-and-openai-for-complex-tasks)
  * [¿Son opus / sonnet / gpt accesos directos integrados?](#are-opus-sonnet-gpt-builtin-shortcuts)
  * [¿Cómo defino/sobrescribo accesos directos de modelos (alias)?](#how-do-i-defineoverride-model-shortcuts-aliases)
  * [¿Cómo agrego modelos de otros proveedores como OpenRouter o Z.AI?](#how-do-i-add-models-from-other-providers-like-openrouter-or-zai)
* [Conmutación por error de modelos y «Todos los modelos han fallado»](#model-failover-and-all-models-failed)
  * [¿Cómo funciona la conmutación por error?](#how-does-failover-work)
  * [¿Qué significa este error?](#what-does-this-error-mean)
  * [Lista de comprobación para solucionar `No credentials found for profile "anthropic:default"`](#fix-checklist-for-no-credentials-found-for-profile-anthropicdefault)
  * [¿Por qué también probó con Google Gemini y falló?](#why-did-it-also-try-google-gemini-and-fail)
* [Perfiles de autenticación: qué son y cómo administrarlos](#auth-profiles-what-they-are-and-how-to-manage-them)
  * [¿Qué es un perfil de autenticación?](#what-is-an-auth-profile)
  * [¿Cuáles son los identificadores de perfil típicos?](#what-are-typical-profile-ids)
  * [¿Puedo controlar qué perfil de autenticación se intenta primero?](#can-i-control-which-auth-profile-is-tried-first)
  * [OAuth vs clave de API: ¿cuál es la diferencia?](#oauth-vs-api-key-whats-the-difference)
* [Gateway: puertos, «ya se está ejecutando» y modo remoto](#gateway-ports-already-running-and-remote-mode)
  * [¿Qué puerto usa el Gateway?](#what-port-does-the-gateway-use)
  * [¿Por qué `openclaw gateway status` muestra `Runtime: running` pero `RPC probe: failed`?](#why-does-openclaw-gateway-status-say-runtime-running-but-rpc-probe-failed)
  * [¿Por qué `openclaw gateway status` muestra `Config (cli)` y `Config (service)` como distintas?](#why-does-openclaw-gateway-status-show-config-cli-and-config-service-different)
  * [¿Qué significa “another gateway instance is already listening”?](#what-does-another-gateway-instance-is-already-listening-mean)
  * [¿Cómo ejecuto OpenClaw en modo remoto (cliente se conecta a un Gateway en otro lugar)?](#how-do-i-run-openclaw-in-remote-mode-client-connects-to-a-gateway-elsewhere)
  * [La Control UI muestra “unauthorized” (o sigue reconectándose). ¿Qué hago ahora?](#the-control-ui-says-unauthorized-or-keeps-reconnecting-what-now)
  * [Configuré `gateway.bind: "tailnet"` pero no puede enlazar / no hay nada escuchando](#i-set-gatewaybind-tailnet-but-it-cant-bind-nothing-listens)
  * [¿Puedo ejecutar múltiples Gateways en el mismo host?](#can-i-run-multiple-gateways-on-the-same-host)
  * [¿Qué significan “invalid handshake” / código 1008?](#what-does-invalid-handshake-code-1008-mean)
* [Registro y depuración](#logging-and-debugging)
  * [¿Dónde están los logs?](#where-are-logs)
  * [¿Cómo inicio/detengo/reinicio el servicio del Gateway?](#how-do-i-startstoprestart-the-gateway-service)
  * [Cerré mi terminal en Windows: ¿cómo reinicio OpenClaw?](#i-closed-my-terminal-on-windows-how-do-i-restart-openclaw)
  * [El Gateway está en ejecución, pero las respuestas nunca llegan. ¿Qué debo revisar?](#the-gateway-is-up-but-replies-never-arrive-what-should-i-check)
  * [&quot;Disconnected from gateway: no reason&quot; - ¿y ahora qué?](#disconnected-from-gateway-no-reason-what-now)
  * [Telegram setMyCommands falla con errores de red. ¿Qué debo revisar?](#telegram-setmycommands-fails-with-network-errors-what-should-i-check)
  * [La TUI no muestra ninguna salida. ¿Qué debo revisar?](#tui-shows-no-output-what-should-i-check)
  * [¿Cómo detengo por completo y luego inicio el Gateway?](#how-do-i-completely-stop-then-start-the-gateway)
  * [ELI5: `openclaw gateway restart` vs `openclaw gateway`](#eli5-openclaw-gateway-restart-vs-openclaw-gateway)
  * [¿Cuál es la forma más rápida de obtener más detalles cuando algo falla?](#whats-the-fastest-way-to-get-more-details-when-something-fails)
* [Contenido multimedia y archivos adjuntos](#media-attachments)
  * [Mi skill generó una imagen o un PDF, pero no se envió nada](#my-skill-generated-an-imagepdf-but-nothing-was-sent)
* [Seguridad y control de acceso](#security-and-access-control)
  * [¿Es seguro exponer OpenClaw a DMs entrantes?](#is-it-safe-to-expose-openclaw-to-inbound-dms)
  * [¿La inyección de prompts solo es una preocupación para bots públicos?](#is-prompt-injection-only-a-concern-for-public-bots)
  * [¿Mi bot debería tener su propia cuenta de correo electrónico, de GitHub o su propio número de teléfono?](#should-my-bot-have-its-own-email-github-account-or-phone-number)
  * [¿Puedo darle autonomía sobre mis mensajes de texto y es eso seguro?](#can-i-give-it-autonomy-over-my-text-messages-and-is-that-safe)
  * [¿Puedo usar modelos más baratos para tareas de asistente personal?](#can-i-use-cheaper-models-for-personal-assistant-tasks)
  * [Ejecuté `/start` en Telegram pero no obtuve un código de emparejamiento](#i-ran-start-in-telegram-but-didnt-get-a-pairing-code)
  * [WhatsApp: ¿le enviará mensajes a mis contactos? ¿Cómo funciona el emparejamiento?](#whatsapp-will-it-message-my-contacts-how-does-pairing-work)
* [Comandos de chat, cancelación de tareas y «no se detiene»](#chat-commands-aborting-tasks-and-it-wont-stop)
  * [¿Cómo impido que los mensajes internos del sistema se muestren en el chat?](#how-do-i-stop-internal-system-messages-from-showing-in-chat)
  * [¿Cómo detengo/cancelo una tarea en ejecución?](#how-do-i-stopcancel-a-running-task)
  * [¿Cómo envío un mensaje de Discord desde Telegram? (“Cross-context messaging denied”)](#how-do-i-send-a-discord-message-from-telegram-crosscontext-messaging-denied)
  * [¿Por qué parece que el bot “ignora” los mensajes enviados muy rápido y de forma consecutiva?](#why-does-it-feel-like-the-bot-ignores-rapidfire-messages)

<div id="first-60-seconds-if-somethings-broken">
  ## Primeros 60 segundos si algo falla
</div>

1. **Estado rápido (primera comprobación)**
   ```bash
   openclaw status
   ```
   Resumen local rápido: sistema operativo + actualización, disponibilidad del gateway/servicio, agentes/sesiones, configuración de proveedores + problemas de tiempo de ejecución (cuando el gateway es accesible).

2. **Informe para pegar (seguro para compartir)**
   ```bash
   openclaw status --all
   ```
   Diagnóstico de solo lectura con la cola del log (tokens redactados).

3. **Estado del daemon + puerto**
   ```bash
   openclaw gateway status
   ```
   Muestra el estado de ejecución del supervisor frente a la accesibilidad RPC, la URL de destino de la sonda y qué configuración es probable que haya usado el servicio.

4. **Comprobaciones profundas**
   ```bash
   openclaw status --deep
   ```
   Ejecuta comprobaciones de estado del gateway + sondas a proveedores (requiere un gateway accesible). Consulta [Health](/es/gateway/health).

5. **Seguir el último log**
   ```bash
   openclaw logs --follow
   ```
   Si RPC no está disponible, usa en su lugar:
   ```bash
   tail -f "$(ls -t /tmp/openclaw/openclaw-*.log | head -1)"
   ```
   Los logs de archivo son independientes de los logs del servicio; consulta [Logging](/es/logging) y [Troubleshooting](/es/gateway/troubleshooting).

6. **Ejecutar el doctor (reparaciones)**
   ```bash
   openclaw doctor
   ```
   Repara/migra configuración/estado + ejecuta comprobaciones de estado. Consulta [Doctor](/es/gateway/doctor).

7. **Instantánea del Gateway**
   ```bash
   openclaw health --json
   openclaw health --verbose   # muestra la URL de destino + la ruta de configuración en caso de errores
   ```
   Le pide al gateway en ejecución una instantánea completa (solo vía WS). Consulta [Health](/es/gateway/health).

<div id="quick-start-and-first-run-setup">
  ## Inicio rápido y configuración de la primera ejecución
</div>

<div id="im-stuck-whats-the-fastest-way-to-get-unstuck">
  ### Estoy atascado, ¿cuál es la forma más rápida de salir del atasco?
</div>

Usa un agente de IA local que pueda **ver tu máquina**. Eso es mucho más efectivo que preguntar
en Discord, porque la mayoría de los casos de &quot;estoy atascado&quot; son **problemas locales de configuración o de entorno** que
las personas que te ayudan de forma remota no pueden inspeccionar.

* **Claude Code**: https://www.anthropic.com/claude-code/
* **OpenAI Codex**: https://openai.com/codex/

Estas herramientas pueden leer el repositorio, ejecutar comandos, inspeccionar registros y ayudar a corregir la configuración
a nivel de máquina (PATH, servicios, permisos, archivos de autenticación). Dales el **checkout completo del código fuente** mediante
la instalación hackeable (git):

```bash
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --install-method git
```

Esto instala OpenClaw **desde un checkout de Git**, para que el agente pueda leer el código y la documentación y
razonar sobre la versión exacta que estás ejecutando. Siempre puedes volver a la versión estable más tarde
volviendo a ejecutar el instalador sin `--install-method git`.

Consejo: pídele al agente que **planifique y supervise** la solución (paso a paso), y luego ejecute solo los
comandos necesarios. Eso mantiene los cambios pequeños y más fáciles de auditar.

Si descubres un bug real o una corrección, abre un issue en GitHub o envía un PR:
https://github.com/openclaw/openclaw/issues
https://github.com/openclaw/openclaw/pulls

Empieza con estos comandos (comparte los resultados cuando pidas ayuda):

```bash
openclaw status
openclaw models status
openclaw doctor
```

Qué hacen:

* `openclaw status`: instantánea rápida del estado del Gateway/agente + configuración básica.
* `openclaw models status`: comprueba la autenticación del proveedor + la disponibilidad de modelos.
* `openclaw doctor`: valida y repara problemas comunes de configuración/estado.

Otras comprobaciones útiles de la CLI: `openclaw status --all`, `openclaw logs --follow`,
`openclaw gateway status`, `openclaw health --verbose`.

Ciclo rápido de depuración: [Primeros 60 segundos si algo falla](#first-60-seconds-if-somethings-broken).
Documentación de instalación: [Instalar](/es/install), [Flags del instalador](/es/install/installer), [Actualización](/es/install/updating).

<div id="whats-the-recommended-way-to-install-and-set-up-openclaw">
  ### ¿Cuál es la manera recomendada de instalar y configurar OpenClaw?
</div>

El repositorio recomienda ejecutar OpenClaw desde el código fuente y usar el asistente de configuración inicial:

```bash
curl -fsSL https://openclaw.bot/install.sh | bash
openclaw onboard --install-daemon
```

El asistente también puede generar automáticamente recursos de UI. Después del onboarding, sueles ejecutar el Gateway en el puerto **18789**.

Desde el código fuente (colaboradores/desarrolladores):

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm build
pnpm ui:build # instala automáticamente las dependencias de UI en la primera ejecución
openclaw onboard
```

Si aún no tienes una instalación global de openclaw, ejecuta `pnpm openclaw onboard`.

<div id="how-do-i-open-the-dashboard-after-onboarding">
  ### ¿Cómo abro el panel después de la incorporación inicial?
</div>

El asistente ahora abre tu navegador con una URL tokenizada del panel justo después de la incorporación inicial y también muestra el enlace completo (con token) en el resumen. Mantén esa pestaña abierta; si no se abrió, copia y pega la URL mostrada en la misma máquina. Los tokens permanecen locales en tu host: no se obtiene nada desde el navegador.

<div id="how-do-i-authenticate-the-dashboard-token-on-localhost-vs-remote">
  ### ¿Cómo autentico el token del panel en localhost frente a remoto?
</div>

**Localhost (misma máquina):**

* Abre `http://127.0.0.1:18789/`.
* Si solicita autenticación, ejecuta `openclaw dashboard` y usa el enlace con token (`?token=...`).
* El token es el mismo valor que `gateway.auth.token` (o `OPENCLAW_GATEWAY_TOKEN`) y la UI lo almacena después de la primera carga.

**Fuera de localhost:**

* **Tailscale Serve** (recomendado): mantén el bind a loopback, ejecuta `openclaw gateway --tailscale serve`, abre `https://<magicdns>/`. Si `gateway.auth.allowTailscale` es `true`, las cabeceras de identidad satisfacen la autenticación (no hace falta token).
* **Tailnet bind**: ejecuta `openclaw gateway --bind tailnet --token "<token>"`, abre `http://<tailscale-ip>:18789/` y pega el token en la configuración del panel.
* **Túnel SSH**: `ssh -N -L 18789:127.0.0.1:18789 user@host` y luego abre `http://127.0.0.1:18789/?token=...` usando `openclaw dashboard`.

Consulta [Dashboard](/es/web/dashboard) y [Web surfaces](/es/web) para ver detalles sobre modos de bind y autenticación.

<div id="what-runtime-do-i-need">
  ### ¿Qué entorno de ejecución necesito?
</div>

Se requiere Node **&gt;= 22**. Se recomienda `pnpm`. Bun **no se recomienda** para el Gateway.

<div id="does-it-run-on-raspberry-pi">
  ### ¿Se ejecuta en Raspberry Pi?
</div>

Sí. El Gateway es ligero: la documentación indica que **512MB-1GB de RAM**, **1 núcleo** y unos **500MB**
de disco son suficientes para uso personal, y señala que un **Raspberry Pi 4 puede ejecutarlo**.

Si quieres un margen adicional (logs, contenido multimedia, otros servicios), **se recomiendan 2GB**, pero no es
un requisito mínimo estricto.

Consejo: una Pi/VPS pequeña puede alojar el Gateway, y puedes emparejar **nodos** en tu portátil o teléfono para
pantalla/cámara/canvas locales o ejecución de comandos. Consulta [Nodes](/es/nodes).

<div id="any-tips-for-raspberry-pi-installs">
  ### ¿Algún consejo para instalaciones en Raspberry Pi?
</div>

Versión corta: funciona, pero espera algunos detalles sin pulir.

* Usa un sistema operativo **de 64 bits** y mantén Node &gt;= 22.
* Da preferencia a la **instalación “hackeable” (git)** para poder ver los logs y actualizar rápido.
* Empieza sin canales/habilidades y luego añádelos uno por uno.
* Si te encuentras con problemas extraños con binarios, normalmente es un problema de **compatibilidad ARM**.

Docs: [Linux](/es/platforms/linux), [Install](/es/install).

<div id="it-is-stuck-on-wake-up-my-friend-onboarding-will-not-hatch-what-now">
  ### Está atascado en el onboarding de &quot;Wake up, my friend!&quot; y no llega a arrancar. ¿Qué hago ahora?
</div>

Esa pantalla depende de que el Gateway sea accesible y esté autenticado. La TUI también envía
&quot;Wake up, my friend!&quot; automáticamente en el primer inicio. Si ves esa línea **sin respuesta**
y los tokens se quedan en 0, el agente nunca se llegó a ejecutar.

1. Reinicia el Gateway:

```bash
openclaw gateway restart
```

2. Verifica el estado y la autenticación:

```bash
openclaw status
openclaw models status
openclaw logs --follow
```

3. Si sigue colgándose, ejecuta:

```bash
openclaw doctor
```

Si el Gateway está en una máquina remota, asegúrate de que el túnel o la conexión de Tailscale estén activos y de que la UI
apunte al Gateway correcto. Consulta [Acceso remoto](/es/gateway/remote).

<div id="can-i-migrate-my-setup-to-a-new-machine-mac-mini-without-redoing-onboarding">
  ### ¿Puedo migrar mi configuración a una nueva Mac mini sin repetir el onboarding?
</div>

Sí. Copia el **directorio de estado** y el **espacio de trabajo**, luego ejecuta Doctor una vez. Esto
mantiene tu bot “exactamente igual” (memoria, historial de sesiones, autenticación y estado
de los canales) siempre que copies **ambas** ubicaciones:

1. Instala OpenClaw en la nueva máquina.
2. Copia `$OPENCLAW_STATE_DIR` (por defecto: `~/.openclaw`) desde la máquina anterior.
3. Copia tu espacio de trabajo (por defecto: `~/.openclaw/workspace`).
4. Ejecuta `openclaw doctor` y reinicia el servicio del Gateway.

Esto conserva la configuración, los perfiles de autenticación, las credenciales de WhatsApp, las sesiones y la memoria. Si estás en
modo remoto, recuerda que el host del Gateway es el propietario del almacén de sesiones y del espacio de trabajo.

**Importante:** si solo haces commit/push de tu espacio de trabajo a GitHub, estás haciendo copia de seguridad de la
**memoria + archivos de arranque**, pero **no** del historial de sesiones ni de la autenticación. Esos viven
bajo `~/.openclaw/` (por ejemplo `~/.openclaw/agents/<agentId>/sessions/`).

Relacionado: [Migrating](/es/install/migrating), [Where things live on disk](/es/help/faq#where-does-openclaw-store-its-data),
[Agent workspace](/es/concepts/agent-workspace), [Doctor](/es/gateway/doctor),
[Remote mode](/es/gateway/remote).

<div id="where-do-i-see-whats-new-in-the-latest-version">
  ### ¿Dónde veo qué hay de nuevo en la última versión?
</div>

Consulta el changelog de GitHub:\
https://github.com/openclaw/openclaw/blob/main/CHANGELOG.md

Las entradas más recientes están arriba. Si la sección superior está marcada como **Unreleased**, la siguiente sección con fecha es la última versión publicada. Las entradas se agrupan en **Highlights**, **Changes** y **Fixes** (además de secciones de documentación u otros cuando sea necesario).

<div id="i-cant-access-docsopenclawai-ssl-error-what-now">
  ### No puedo acceder a docs.openclaw.ai (error de SSL) ¿qué hago ahora?
</div>

Algunas conexiones de Comcast/Xfinity bloquean incorrectamente `docs.openclaw.ai` mediante Xfinity Advanced Security. Desactiva esa opción o incluye `docs.openclaw.ai` en la lista de permitidos y vuelve a intentarlo. Más detalles: [Solución de problemas](/es/help/troubleshooting#docsopenclawai-shows-an-ssl-error-comcastxfinity). Ayúdanos a desbloquearlo notificándolo aquí: https://spa.xfinity.com/check&#95;url&#95;status.

Si aún no puedes acceder al sitio, la documentación también está disponible como espejo en GitHub:
https://github.com/openclaw/openclaw/tree/main/docs

<div id="whats-the-difference-between-stable-and-beta">
  ### ¿Cuál es la diferencia entre estable y beta?
</div>

**Stable** y **beta** son **dist‑tags de npm**, no líneas de código independientes:

* `latest` = estable
* `beta` = versión temprana para pruebas

Publicamos versiones en **beta**, las probamos y, cuando una versión es sólida, **promocionamos esa misma versión a `latest`**. Por eso beta y estable pueden apuntar a la **misma versión**.

Consulta qué ha cambiado:\
https://github.com/openclaw/openclaw/blob/main/CHANGELOG.md

<div id="how-do-i-install-the-beta-version-and-whats-the-difference-between-beta-and-dev">
  ### ¿Cómo instalo la versión beta y cuál es la diferencia entre beta y dev?
</div>

**Beta** es la etiqueta de distribución (`dist-tag`) de npm `beta` (puede coincidir con `latest`).
**Dev** es la cabeza actual de `main` (git); cuando se publica, utiliza la etiqueta de distribución de npm `dev`.

Comandos en una sola línea (macOS/Linux):

```bash
curl -fsSL --proto '=https' --tlsv1.2 https://openclaw.bot/install.sh | bash -s -- --beta
```

```bash
curl -fsSL --proto '=https' --tlsv1.2 https://openclaw.bot/install.sh | bash -s -- --install-method git
```

Instalador para Windows (PowerShell):
https://openclaw.ai/install.ps1

Más detalles: [Canales de desarrollo](/es/install/development-channels) y [Opciones del instalador](/es/install/installer).

<div id="how-long-does-install-and-onboarding-usually-take">
  ### ¿Cuánto suelen tardar la instalación y la configuración inicial?
</div>

Guía orientativa:

* **Instalación:** 2-5 minutos
* **Configuración inicial:** 5-15 minutos, según cuántos canales/modelos configures

Si se queda colgado, usa [Installer stuck](/es/help/faq#installer-stuck-how-do-i-get-more-feedback)
y el bucle rápido de depuración en [Im stuck](/es/help/faq#im-stuck--whats-the-fastest-way-to-get-unstuck).

<div id="how-do-i-try-the-latest-bits">
  ### ¿Cómo pruebo las últimas novedades?
</div>

Dos opciones:

1. **Canal de desarrollo (git checkout):**

```bash
openclaw update --channel dev
```

Esto cambia a la rama `main` y actualiza desde el origen.

2. **Instalación fácilmente modificable (desde el sitio del instalador):**

```bash
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --install-method git
```

Eso te da un repositorio local que puedes editar y luego actualizar mediante Git.

Si prefieres hacer un clon limpio de forma manual, usa:

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm build
```

Documentación: [Actualización](/es/cli/update), [Canales de desarrollo](/es/install/development-channels),
[Instalación](/es/install).

<div id="installer-stuck-how-do-i-get-more-feedback">
  ### Instalador atascado: ¿cómo obtengo más información?
</div>

Vuelve a ejecutar el instalador con **salida detallada**:

```bash
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --verbose
```

Instalación beta con salida detallada:

```bash
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --beta --verbose
```

Para una instalación desde el código fuente (git):

```bash
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --install-method git --verbose
```

Más opciones: [Opciones del instalador](/es/install/installer).

<div id="windows-install-says-git-not-found-or-openclaw-not-recognized">
  ### La instalación en Windows muestra que no se encuentra git o que openclaw no se reconoce
</div>

Dos problemas comunes en Windows:

**1) error de npm `spawn git` / `git not found`**

* Instala **Git for Windows** y asegúrate de que `git` esté en tu PATH.
* Cierra y vuelve a abrir PowerShell, luego vuelve a ejecutar el instalador.

**2) openclaw no se reconoce después de la instalación**

* Tu carpeta global `bin` de npm no está en el PATH.
* Verifica la ruta:
  ```powershell
  npm config get prefix
  ```
* Asegúrate de que `<prefix>\\bin` esté en el PATH (en la mayoría de los sistemas es `%AppData%\\npm`).
* Cierra y vuelve a abrir PowerShell después de actualizar el PATH.

Si quieres la configuración más sencilla en Windows, usa **WSL2** en lugar de Windows nativo.
Documentación: [Windows](/es/platforms/windows).

<div id="the-docs-didnt-answer-my-question-how-do-i-get-a-better-answer">
  ### La documentación no respondió mi pregunta: ¿cómo obtengo una mejor respuesta?
</div>

Usa la **instalación hackeable (git)** para tener todo el código fuente y la documentación en tu máquina, luego pídele
a tu bot (o a Claude/Codex) *desde esa carpeta* que lea el repositorio y responda con precisión.

```bash
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --install-method git
```

Más información: [Instalación](/es/install) y [flags del instalador](/es/install/installer).

<div id="how-do-i-install-openclaw-on-linux">
  ### ¿Cómo instalo OpenClaw en Linux?
</div>

Respuesta rápida: sigue la guía de Linux y luego ejecuta el asistente de configuración inicial.

* Ruta rápida para Linux + instalación como servicio: [Linux](/es/platforms/linux).
* Recorrido completo: [Primeros pasos](/es/start/getting-started).
* Instalador + actualizaciones: [Instalación y actualizaciones](/es/install/updating).

<div id="how-do-i-install-openclaw-on-a-vps">
  ### ¿Cómo instalo OpenClaw en un VPS?
</div>

Cualquier VPS con Linux sirve. Instálalo en el servidor y luego usa SSH/Tailscale para acceder al Gateway.

Guías: [exe.dev](/es/platforms/exe-dev), [Hetzner](/es/platforms/hetzner), [Fly.io](/es/platforms/fly).\
Acceso remoto: [Gateway remoto](/es/gateway/remote).

<div id="where-are-the-cloudvps-install-guides">
  ### ¿Dónde están las guías de instalación en cloudVPS?
</div>

Mantenemos un **hub de hosting** con los proveedores más comunes. Elige uno y sigue la guía:

* [VPS hosting](/es/vps) (todos los proveedores en un solo lugar)
* [Fly.io](/es/platforms/fly)
* [Hetzner](/es/platforms/hetzner)
* [exe.dev](/es/platforms/exe-dev)

Cómo funciona en la nube: el **Gateway se ejecuta en el servidor** y accedes a él
desde tu laptop/teléfono mediante el Control UI (o Tailscale/SSH). Tu estado y tu espacio de trabajo
residen en el servidor, así que trata ese host como la fuente de la verdad y haz copias de seguridad.

Puedes emparejar **nodos** (Mac/iOS/Android/headless) con ese Gateway en la nube para acceder
a la pantalla/cámara/canvas locales o ejecutar comandos en tu laptop mientras mantienes el
Gateway en la nube.

Hub: [Platforms](/es/platforms). Acceso remoto: [Gateway remote](/es/gateway/remote).
Nodos: [Nodes](/es/nodes), [Nodes CLI](/es/cli/nodes).

<div id="can-i-ask-openclaw-to-update-itself">
  ### ¿Puedo pedirle a OpenClaw que se actualice a sí mismo?
</div>

Respuesta corta: **es posible, pero no recomendable**. El proceso de actualización puede
reiniciar el Gateway (lo que elimina la sesión activa), puede necesitar un
`git checkout` limpio y puede solicitar confirmación. Es más seguro ejecutar las
actualizaciones desde una shell como operador.

Usa la CLI:

```bash
openclaw update
openclaw update status
openclaw update --channel stable|beta|dev
openclaw update --tag <dist-tag|version>
openclaw update --no-restart
```

Si necesitas automatizar desde un agente:

```bash
openclaw update --yes --no-restart
openclaw gateway restart
```

Documentación: [Actualizar](/es/cli/update), [Actualización](/es/install/updating).

<div id="what-does-the-onboarding-wizard-actually-do">
  ### ¿Qué hace realmente el asistente de incorporación
</div>

`openclaw onboard` es la ruta de configuración recomendada. En **modo local** te guía paso a paso por:

* **Configuración de modelo/auth** (Anthropic **setup-token** recomendado para suscripciones de Claude, OpenAI Codex OAuth compatible, claves API opcionales, modelos locales de LM Studio compatibles)
* Ubicación del **espacio de trabajo** + archivos de arranque
* **Configuración del Gateway** (bind/port/auth/tailscale)
* **Proveedores** (WhatsApp, Telegram, Discord, Mattermost (plugin), Signal, iMessage)
* **Instalación del daemon** (LaunchAgent en macOS; unidad de usuario systemd en Linux/WSL2)
* **Comprobaciones de estado** y selección de **habilidades**

También te avisa si el modelo que has configurado es desconocido o si falta la autenticación.

<div id="do-i-need-a-claude-or-openai-subscription-to-run-this">
  ### ¿Necesito una suscripción de Claude u OpenAI para usar esto?
</div>

No. Puedes ejecutar OpenClaw con **claves API** (Anthropic/OpenAI/otros) o con
**modelos solo locales**, de modo que tus datos permanezcan en tu dispositivo. Las suscripciones (Claude
Pro/Max u OpenAI Codex) son formas opcionales de autenticar esos proveedores.

Documentación: [Anthropic](/es/providers/anthropic), [OpenAI](/es/providers/openai),
[Modelos locales](/es/gateway/local-models), [Modelos](/es/concepts/models).

<div id="can-i-use-claude-max-subscription-without-an-api-key">
  ### ¿Puedo usar la suscripción a Claude Max sin una clave de API?
</div>

Sí. Puedes autenticarte con un **setup-token**
en lugar de una clave de API. Esta es la ruta para suscripciones.

Las suscripciones Claude Pro/Max **no incluyen una clave de API**, así que este es el
enfoque correcto para cuentas de suscripción. Importante: debes verificar con
Anthropic que este uso esté permitido según sus políticas de suscripción y términos.
Si quieres la ruta más explícita y con soporte oficial, usa una clave de API de Anthropic.

<div id="how-does-anthropic-setuptoken-auth-work">
  ### ¿Cómo funciona la autenticación setuptoken de Anthropic?
</div>

`claude setup-token` genera una **cadena de token** mediante la CLI de Claude Code (no está disponible en la consola web). Puedes ejecutarlo en **cualquier máquina**. Selecciona **Anthropic token (paste setup-token)** en el asistente o pégalo con `openclaw models auth paste-token --provider anthropic`. El token se almacena como un perfil de autenticación para el proveedor **anthropic** y se utiliza como una clave de API (sin renovación automática). Más detalles: [OAuth](/es/concepts/oauth).

<div id="where-do-i-find-an-anthropic-setuptoken">
  ### ¿Dónde puedo encontrar un setuptoken de Anthropic?
</div>

**No** está en Anthropic Console. El setup-token lo genera la **Claude Code CLI** en **cualquier máquina**:

```bash
claude setup-token
```

Copia el token que muestre y luego selecciona **Anthropic token (paste setup-token)** en el asistente. Si quieres ejecutarlo en el host del Gateway, usa `openclaw models auth setup-token --provider anthropic`. Si ejecutaste `claude setup-token` en otro lugar, pégalo en el host del Gateway con `openclaw models auth paste-token --provider anthropic`. Consulta [Anthropic](/es/providers/anthropic).

<div id="do-you-support-claude-subscription-auth-claude-promax">
  ### ¿Es compatible la autenticación por suscripción de Claude (Claude Pro/Max)?
</div>

Sí — mediante **setup-token**. OpenClaw ya no reutiliza tokens OAuth de Claude Code CLI; utiliza un setup-token o una clave de API de Anthropic. Genera el token en cualquier lugar y pégalo en el host del Gateway. Consulta [Anthropic](/es/providers/anthropic) y [OAuth](/es/concepts/oauth).

Nota: El acceso por suscripción a Claude se rige por los términos de Anthropic. Para cargas de trabajo en producción o multiusuario, las claves de API suelen ser la opción más segura.

<div id="why-am-i-seeing-http-429-ratelimiterror-from-anthropic">
  ### ¿Por qué veo un error HTTP 429 ratelimiterror de Anthropic?
</div>

Eso significa que tu **cuota/límite de tasa de Anthropic** se ha agotado para la ventana actual. Si
usas una **suscripción de Claude** (setup‑token u OAuth de Claude Code), espera a que la ventana se
restablezca o actualiza tu plan. Si usas una **clave de API de Anthropic**, revisa la Anthropic Console
para ver el uso y la facturación y aumenta los límites según sea necesario.

Sugerencia: configura un **modelo de respaldo** para que OpenClaw pueda seguir respondiendo mientras un proveedor esté sujeto a límites de tasa.
Consulta [Models](/es/cli/models) y [OAuth](/es/concepts/oauth).

<div id="is-aws-bedrock-supported">
  ### ¿Es compatible AWS Bedrock?
</div>

Sí, a través del proveedor **Amazon Bedrock (Converse)** de pi‑ai con **configuración manual**. Debes proporcionar las credenciales y la región de AWS en el host del Gateway y añadir una entrada de proveedor de Bedrock en la configuración de modelos. Consulta [Amazon Bedrock](/es/bedrock) y [Proveedores de modelos](/es/providers/models). Si prefieres un flujo gestionado de claves, un proxy compatible con OpenAI delante de Bedrock sigue siendo una opción válida.

<div id="how-does-codex-auth-work">
  ### ¿Cómo funciona la autenticación de Codex?
</div>

OpenClaw es compatible con **OpenAI Code (Codex)** mediante OAuth (inicio de sesión con ChatGPT). El asistente puede ejecutar el flujo de OAuth y establecer el modelo predeterminado en `openai-codex/gpt-5.2` cuando sea apropiado. Consulta [Proveedores de modelos](/es/concepts/model-providers) y [Asistente](/es/start/wizard).

<div id="do-you-support-openai-subscription-auth-codex-oauth">
  ### ¿Es compatible OpenClaw con la autenticación de suscripción de OpenAI Codex OAuth?
</div>

Sí. OpenClaw es totalmente compatible con **OpenAI Code (Codex) subscription OAuth**. El asistente de incorporación
puede ejecutar el flujo de OAuth por ti.

Consulta [OAuth](/es/concepts/oauth), [Proveedores de modelos](/es/concepts/model-providers) y [Asistente](/es/start/wizard).

<div id="how-do-i-set-up-gemini-cli-oauth">
  ### ¿Cómo configuro OAuth para Gemini CLI?
</div>

Gemini CLI usa un **plugin auth flow** (flujo de autenticación mediante complemento), no un ID de cliente ni un secreto en `openclaw.json`.

Pasos:

1. Habilita el complemento: `openclaw plugins enable google-gemini-cli-auth`
2. Inicia sesión: `openclaw models auth login --provider google-gemini-cli --set-default`

Esto almacena los tokens de OAuth en perfiles de autenticación en el host del Gateway. Detalles: [Model providers](/es/concepts/model-providers).

<div id="is-a-local-model-ok-for-casual-chats">
  ### ¿Es suficiente un modelo local para chats informales?
</div>

Por lo general, no. OpenClaw necesita un contexto amplio y una seguridad sólida; los contextos pequeños se truncan y filtran datos. Si es imprescindible, ejecuta localmente la versión **más grande** de MiniMax M2.1 que puedas (LM Studio) y consulta [/gateway/local-models](/es/gateway/local-models). Los modelos más pequeños o cuantizados aumentan el riesgo de ataques de *prompt injection*; consulta [Security](/es/gateway/security).

<div id="how-do-i-keep-hosted-model-traffic-in-a-specific-region">
  ### ¿Cómo mantengo el tráfico de modelos alojados en una región específica?
</div>

Elige endpoints anclados a una región. OpenRouter expone opciones alojadas en EE. UU. para MiniMax, Kimi y GLM; selecciona la variante alojada en EE. UU. para mantener los datos dentro de la región. Aun así puedes incluir Anthropic/OpenAI junto con estos usando `models.mode: "merge"` para que las alternativas sigan disponibles respetando el proveedor regional que elijas.

<div id="do-i-have-to-buy-a-mac-mini-to-install-this">
  ### ¿Tengo que comprar un Mac mini para instalar esto?
</div>

No. OpenClaw se ejecuta en macOS o Linux (Windows mediante WSL2). Un Mac mini es opcional: algunas personas
compran uno como host siempre activo, pero también sirve un VPS pequeño, un servidor doméstico o una máquina tipo Raspberry Pi.

Solo necesitas un Mac **para herramientas exclusivas de macOS**. Para iMessage, puedes mantener el Gateway en Linux
y ejecutar `imsg` en cualquier Mac a través de SSH, apuntando `channels.imessage.cliPath` a un wrapper de SSH.
Si quieres otras herramientas exclusivas de macOS, ejecuta el Gateway en un Mac o empareja un nodo con macOS.

Docs: [iMessage](/es/channels/imessage), [Nodos](/es/nodes), [Modo remoto de Mac](/es/platforms/mac/remote).

<div id="do-i-need-a-mac-mini-for-imessage-support">
  ### ¿Necesito un Mac mini para compatibilidad con iMessage?
</div>

Necesitas **algún dispositivo con macOS** con sesión iniciada en Mensajes. **No** tiene que ser un Mac mini;
cualquier Mac funciona. Las integraciones de iMessage de OpenClaw se ejecutan en macOS (BlueBubbles o `imsg`),
mientras que el Gateway puede ejecutarse en otra parte.

Configuraciones habituales:

* Ejecutar el Gateway en Linux/VPS y apuntar `channels.imessage.cliPath` a un wrapper de SSH que
  ejecute `imsg` en el Mac.
* Ejecutar todo en el Mac si quieres la configuración más sencilla en una sola máquina.

Documentación: [iMessage](/es/channels/imessage), [BlueBubbles](/es/channels/bluebubbles),
[Modo remoto de Mac](/es/platforms/mac/remote).

<div id="if-i-buy-a-mac-mini-to-run-openclaw-can-i-connect-it-to-my-macbook-pro">
  ### Si compro un Mac mini para ejecutar OpenClaw, ¿puedo conectarlo a mi MacBook Pro?
</div>

Sí. El **Mac mini puede ejecutar el Gateway**, y tu MacBook Pro puede conectarse
como **nodo** (dispositivo complementario). Los nodos no ejecutan el Gateway:
proporcionan capacidades adicionales como pantalla/cámara/lienzo y `system.run`
en ese dispositivo.

Patrón habitual:

* Gateway en el Mac mini (siempre encendido).
* El MacBook Pro ejecuta la aplicación de macOS o un host de nodo y se empareja con el Gateway.
* Usa `openclaw nodes status` / `openclaw nodes list` para verlo.

Docs: [Nodos](/es/nodes), [Nodes CLI](/es/cli/nodes).

<div id="can-i-use-bun">
  ### ¿Puedo usar Bun?
</div>

No se recomienda usar **Bun**. Hemos observado errores en tiempo de ejecución, especialmente con WhatsApp y Telegram.
Usa **Node.js** para Gateways estables.

Si aun así quieres experimentar con Bun, hazlo en un Gateway que no sea de producción,
sin WhatsApp ni Telegram.

<div id="telegram-what-goes-in-allowfrom">
  ### Telegram qué va en allowFrom
</div>

`channels.telegram.allowFrom` es **el ID de usuario de Telegram del remitente humano** (numérico, recomendado) o `@username`. No es el nombre de usuario del bot.

Más seguro (sin bot de terceros):

* Envíale un DM a tu bot, luego ejecuta `openclaw logs --follow` y lee `from.id`.

Bot API oficial:

* Envíale un DM a tu bot, luego haz una petición a `https://api.telegram.org/bot<bot_token>/getUpdates` y lee `message.from.id`.

Servicios de terceros (menos privado):

* Envíale un DM a `@userinfobot` o `@getidsbot`.

Consulta [/channels/telegram](/es/channels/telegram#access-control-dms--groups).

<div id="can-multiple-people-use-one-whatsapp-number-with-different-openclaw-instances">
  ### ¿Pueden varias personas usar un mismo número de WhatsApp con distintas instancias de OpenClaw?
</div>

Sí, mediante **enrutamiento multi‑agente**. Vincula el **DM** de WhatsApp de cada remitente (par `kind: "dm"`, remitente E.164 como `+15551234567`) a un `agentId` diferente, de modo que cada persona tenga su propio espacio de trabajo y almacén de sesiones. Las respuestas siguen saliendo de la **misma cuenta de WhatsApp**, y el control de acceso de DM (`channels.whatsapp.dmPolicy` / `channels.whatsapp.allowFrom`) es global a nivel de cuenta de WhatsApp. Consulta [Enrutamiento multi-agente](/es/concepts/multi-agent) y [WhatsApp](/es/channels/whatsapp).

<div id="can-i-run-a-fast-chat-agent-and-an-opus-for-coding-agent">
  ### ¿Puedo ejecutar un agente de chat rápido y un agente Opus para programación?
</div>

Sí. Usa enrutamiento multiagente: asigna a cada agente su propio modelo predeterminado y luego vincula las rutas entrantes (cuenta de proveedor o pares específicos) a cada agente. La configuración de ejemplo está en [Enrutamiento multiagente](/es/concepts/multi-agent). Consulta también [Modelos](/es/concepts/models) y [Configuración](/es/gateway/configuration).

<div id="does-homebrew-work-on-linux">
  ### ¿Funciona Homebrew en Linux?
</div>

Sí. Homebrew es compatible con Linux (Linuxbrew). Configuración rápida:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
echo 'eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"' >> ~/.profile
eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"
brew install <formula>
```

Si ejecutas OpenClaw a través de systemd, asegúrate de que la variable PATH del servicio incluya `/home/linuxbrew/.linuxbrew/bin` (o tu prefijo de brew) para que las herramientas instaladas con `brew` se resuelvan en shells que no sean de inicio de sesión.
Las versiones recientes también anteponen directorios de binarios de usuario comunes en servicios systemd de Linux (por ejemplo `~/.local/bin`, `~/.npm-global/bin`, `~/.local/share/pnpm`, `~/.bun/bin`) y respetan `PNPM_HOME`, `NPM_CONFIG_PREFIX`, `BUN_INSTALL`, `VOLTA_HOME`, `ASDF_DATA_DIR`, `NVM_DIR` y `FNM_DIR` cuando están definidos.

<div id="whats-the-difference-between-the-hackable-git-install-and-npm-install">
  ### ¿Cuál es la diferencia entre la instalación hackeable con git y la instalación con npm?
</div>

* **Instalación hackeable (git):** clon completo del código fuente, editable, ideal para colaboradores.
  Compilas todo localmente y puedes aplicar parches al código y la documentación.
* **Instalación con npm:** instalación global de la CLI, sin repositorio, ideal si “solo quieres ejecutarlo”.
  Las actualizaciones vienen de los dist‑tags de npm.

Documentación: [Primeros pasos](/es/start/getting-started), [Actualización](/es/install/updating).

<div id="can-i-switch-between-npm-and-git-installs-later">
  ### ¿Puedo cambiar entre instalaciones con npm y con git más adelante?
</div>

Sí. Instala la otra modalidad y luego ejecuta Doctor para que el servicio Gateway apunte al nuevo punto de entrada.
Esto **no elimina tus datos**: solo cambia la instalación del código de OpenClaw. Tu estado
(`~/.openclaw`) y espacio de trabajo (`~/.openclaw/workspace`) permanecen sin cambios.

De npm → git:

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm build
openclaw doctor
openclaw gateway restart
```

De Git → npm:

```bash
npm install -g openclaw@latest
openclaw doctor
openclaw gateway restart
```

Doctor detecta una discrepancia en el entrypoint del servicio del Gateway y ofrece reescribir la configuración del servicio para que coincida con la instalación actual (usa `--repair` en automatización).

Recomendaciones de copia de seguridad: consulta [Estrategia de copia de seguridad](/es/help/faq#whats-the-recommended-backup-strategy).

<div id="should-i-run-the-gateway-on-my-laptop-or-a-vps">
  ### ¿Debería ejecutar el Gateway en mi portátil o en un VPS?
</div>

Respuesta breve: **si quieres fiabilidad 24/7, usa un VPS**. Si quieres la
mínima fricción y no te importa la suspensión o los reinicios, ejecútalo en local.

**Portátil (Gateway local)**

* **Ventajas:** sin coste de servidor, acceso directo a archivos locales, ventana de navegador activa.
* **Desventajas:** suspensión/caídas de red = desconexiones, las actualizaciones/reinicios del SO interrumpen, el portátil tiene que permanecer despierto.

**VPS / nube**

* **Ventajas:** siempre encendido, red estable, sin problemas de suspensión del portátil, más fácil de mantener en ejecución.
* **Desventajas:** a menudo se ejecuta en modo headless (tendrás que usar capturas de pantalla), solo acceso remoto a archivos, debes usar SSH para las actualizaciones.

**Nota específica de OpenClaw:** WhatsApp/Telegram/Slack/Mattermost (complemento)/Discord funcionan bien desde un VPS. La única contrapartida real es **navegador en modo headless** frente a una ventana visible. Consulta [Browser](/es/tools/browser).

**Recomendación por defecto:** VPS si antes tuviste desconexiones del Gateway. Local es ideal cuando estás usando activamente el Mac y quieres acceso a archivos locales o automatización de la UI con un navegador visible.

<div id="how-important-is-it-to-run-openclaw-on-a-dedicated-machine">
  ### ¿Qué tan importante es ejecutar OpenClaw en una máquina dedicada?
</div>

No es obligatorio, pero **se recomienda para mayor fiabilidad y aislamiento**.

* **Host dedicado (VPS/Mac mini/Pi):** siempre encendido, menos interrupciones por suspensión/reinicio, permisos más limpios, más fácil de mantener en ejecución.
* **Portátil o equipo de sobremesa compartido:** perfectamente válido para pruebas y uso activo, pero ten en cuenta pausas cuando la máquina se suspenda o se actualice.

Si quieres lo mejor de ambos mundos, mantén el Gateway en un host dedicado y empareja tu portátil como un **nodo** para herramientas locales de pantalla/cámara/exec. Consulta [Nodes](/es/nodes).
Para obtener recomendaciones de seguridad, lee [Security](/es/gateway/security).

<div id="what-are-the-minimum-vps-requirements-and-recommended-os">
  ### ¿Cuáles son los requisitos mínimos de un VPS y el sistema operativo recomendado?
</div>

OpenClaw es liviano. Para un Gateway básico + un canal de chat:

* **Mínimo absoluto:** 1 vCPU, 1GB de RAM, ~500MB de disco.
* **Recomendado:** 1-2 vCPU, 2GB de RAM o más para tener margen (logs, contenido multimedia, múltiples canales). Las herramientas de nodo y la automatización del navegador pueden consumir muchos recursos.

SO: usa **Ubuntu LTS** (o cualquier Debian/Ubuntu moderno). La ruta de instalación en Linux es la que está mejor probada.

Docs: [Linux](/es/platforms/linux), [VPS hosting](/es/vps).

<div id="can-i-run-openclaw-in-a-vm-and-what-are-the-requirements">
  ### ¿Puedo ejecutar OpenClaw en una VM y cuáles son los requisitos?
</div>

Sí. Trata una VM igual que un VPS: debe estar siempre encendida, ser accesible y disponer de suficiente
RAM para el Gateway y cualquier canal que habilites.

Guía básica:

* **Mínimo absoluto:** 1 vCPU, 1GB RAM.
* **Recomendado:** 2GB RAM o más si ejecutas múltiples canales, automatización de navegador o herramientas de medios.
* **SO:** Ubuntu LTS u otra distribución moderna basada en Debian/Ubuntu.

Si estás en Windows, **WSL2 es la configuración de VM más sencilla** y la que ofrece la mejor compatibilidad con las herramientas.
Consulta [Windows](/es/platforms/windows), [VPS hosting](/es/vps).
Si estás ejecutando macOS en una VM, consulta [macOS VM](/es/platforms/macos-vm).

<div id="what-is-openclaw">
  ## ¿Qué es OpenClaw?
</div>

<div id="what-is-openclaw-in-one-paragraph">
  ### ¿Qué es OpenClaw en un párrafo?
</div>

OpenClaw es un asistente de IA personal que se ejecuta en tus propios dispositivos. Responde en las plataformas de mensajería que ya utilizas (WhatsApp, Telegram, Slack, Mattermost (complemento), Discord, Google Chat, Signal, iMessage, WebChat) y también puede ofrecer voz y un Canvas en vivo en las plataformas compatibles. El **Gateway** es el plano de control siempre activo; el asistente es el producto.

<div id="whats-the-value-proposition">
  ### ¿Cuál es la propuesta de valor?
</div>

OpenClaw no es “solo un wrapper de Claude”. Es un **plano de control local-first** que te permite ejecutar
un asistente potente en **tu propio hardware**, accesible desde las apps de chat que ya usas, con
sesiones con estado, memoria y herramientas, sin ceder el control de tus flujos de trabajo a un
SaaS alojado.

Aspectos destacados:

* **Tus dispositivos, tus datos:** ejecuta el Gateway donde quieras (Mac, Linux, VPS) y mantén el
  espacio de trabajo + el historial de sesión de forma local.
* **Canales reales, no un sandbox web:** WhatsApp/Telegram/Slack/Discord/Signal/iMessage/etc.,
  además de voz móvil y Canvas en las plataformas compatibles.
* **Independiente del modelo:** usa Anthropic, OpenAI, MiniMax, OpenRouter, etc., con enrutamiento
  por agente y conmutación por error.
* **Opción solo local:** ejecuta modelos locales para que **todos los datos puedan permanecer en tu dispositivo** si así lo quieres.
* **Enrutamiento multiagente:** separa agentes por canal, cuenta o tarea, cada uno con su propio
  espacio de trabajo y valores predeterminados.
* **Código abierto y hackeable:** inspecciona, extiende y autohospeda sin bloqueo de proveedor.

Documentación: [Gateway](/es/gateway), [Channels](/es/channels), [Multi‑agent](/es/concepts/multi-agent),
[Memory](/es/concepts/memory).

<div id="i-just-set-it-up-what-should-i-do-first">
  ### Ya lo configuré, ¿qué debo hacer primero?
</div>

Buenos proyectos iniciales:

* Crear un sitio web (WordPress, Shopify o un sitio estático sencillo).
* Prototipar una app móvil (esquema, pantallas, plan de API).
* Organizar archivos y carpetas (limpieza, nombres, etiquetado).
* Conectar Gmail y automatizar resúmenes o seguimientos.

Puede encargarse de tareas grandes, pero funciona mejor cuando las divides en fases y
usas subagentes para trabajo en paralelo.

<div id="what-are-the-top-five-everyday-use-cases-for-openclaw">
  ### ¿Cuáles son los cinco principales casos de uso cotidianos de OpenClaw?
</div>

Las ventajas cotidianas suelen verse así:

* **Informes personales:** resúmenes de tu bandeja de entrada, calendario y noticias que te importan.
* **Investigación y redacción:** investigación rápida, resúmenes y primeros borradores de correos o documentos.
* **Recordatorios y seguimientos:** avisos y listas de tareas impulsados por cron o por el latido (heartbeat).
* **Automatización del navegador:** completar formularios, recopilar datos y repetir tareas web.
* **Coordinación entre dispositivos:** envía una tarea desde tu teléfono, deja que el Gateway la ejecute en un servidor y recibe el resultado de vuelta en el chat.

<div id="can-openclaw-help-with-lead-gen-outreach-ads-and-blogs-for-a-saas">
  ### ¿Puede OpenClaw ayudar con anuncios de prospección y blogs para generación de leads en un SaaS?
</div>

Sí, para **investigación, cualificación y redacción de borradores**. Puede escanear sitios, crear listas cortas priorizadas,
resumir prospectos y redactar borradores de mensajes de prospección o textos publicitarios.

Para **campañas de prospección o anuncios en producción**, mantén siempre intervención humana. Evita el spam, respeta las leyes locales y
las políticas de las plataformas, y revisa todo antes de enviarlo. El patrón más seguro es dejar que
OpenClaw redacte y que tú apruebes.

Docs: [Seguridad](/es/gateway/security).

<div id="what-are-the-advantages-vs-claude-code-for-web-development">
  ### ¿Cuáles son las ventajas frente a Claude Code para el desarrollo web?
</div>

OpenClaw es un **asistente personal** y una capa de coordinación, no un sustituto de un IDE. Usa
Claude Code o Codex para el ciclo de codificación directa más rápido dentro de un repositorio. Usa OpenClaw cuando
quieras memoria duradera, acceso multidispositivo y orquestación de herramientas.

Ventajas:

* **Memoria persistente + espacio de trabajo** a través de sesiones
* **Acceso multiplataforma** (WhatsApp, Telegram, TUI, WebChat)
* **Orquestación de herramientas** (navegador, archivos, planificación, hooks)
* **Gateway siempre activo** (ejecútalo en un VPS, interactúa desde cualquier lugar)
* **Nodes** para navegador/pantalla/cámara/exec local

Showcase: https://openclaw.ai/showcase

<div id="skills-and-automation">
  ## Habilidades y automatización
</div>

<div id="how-do-i-customize-skills-without-keeping-the-repo-dirty">
  ### ¿Cómo personalizo las habilidades sin ensuciar el repositorio?
</div>

Utiliza anulaciones gestionadas en lugar de editar la copia del repositorio. Pon tus cambios en `~/.openclaw/skills/<name>/SKILL.md` (o añade una carpeta mediante `skills.load.extraDirs` en `~/.openclaw/openclaw.json`). La precedencia es `<workspace>/skills` &gt; `~/.openclaw/skills` &gt; incluidas en el paquete, de modo que las anulaciones gestionadas tienen prioridad sin tocar git. Solo las ediciones que merezcan enviarse upstream deberían vivir en el repositorio y salir como PRs.

<div id="can-i-load-skills-from-a-custom-folder">
  ### ¿Puedo cargar habilidades desde una carpeta personalizada?
</div>

Sí. Añade directorios adicionales mediante `skills.load.extraDirs` en `~/.openclaw/openclaw.json` (con la precedencia más baja). La precedencia predeterminada sigue siendo: `<workspace>/skills` → `~/.openclaw/skills` → habilidades integradas en el paquete → `skills.load.extraDirs`. `clawhub` instala en `./skills` de forma predeterminada, que OpenClaw considera como `<workspace>/skills`.

<div id="how-can-i-use-different-models-for-different-tasks">
  ### ¿Cómo puedo usar distintos modelos para diferentes tareas?
</div>

Actualmente, los patrones admitidos son:

* **Trabajos cron**: los trabajos aislados pueden definir un `model` distinto para cada trabajo.
* **Subagentes**: enruta tareas a agentes separados con diferentes modelos predeterminados.
* **Cambio bajo demanda**: usa `/model` para cambiar el modelo de la sesión actual en cualquier momento.

Consulta [Trabajos cron](/es/automation/cron-jobs), [Enrutamiento multi-agente](/es/concepts/multi-agent) y [Comandos slash](/es/tools/slash-commands).

<div id="the-bot-freezes-while-doing-heavy-work-how-do-i-offload-that">
  ### El bot se congela mientras realiza trabajo pesado ¿Cómo puedo delegar eso?
</div>

Utiliza **subagentes** para tareas largas o en paralelo. Los subagentes se ejecutan en su propia sesión,
devuelven un resumen y mantienen tu chat principal receptivo.

Pídele a tu bot que &quot;cree un subagente para esta tarea&quot; o utiliza `/subagents`.
Usa `/status` en el chat para ver qué está haciendo el Gateway en este momento (y si está ocupado).

Consejo sobre tokens: tanto las tareas largas como los subagentes consumen tokens. Si te preocupa el costo, configura un
modelo más barato para los subagentes mediante `agents.defaults.subagents.model`.

Documentación: [Sub-agents](/es/tools/subagents).

<div id="cron-or-reminders-do-not-fire-what-should-i-check">
  ### Cron o recordatorios no se ejecutan: ¿qué debo comprobar?
</div>

Cron se ejecuta dentro del proceso del Gateway. Si el Gateway no se está ejecutando de forma continua,
las tareas programadas no se ejecutarán.

Lista de comprobación:

* Confirma que cron está habilitado (`cron.enabled`) y que `OPENCLAW_SKIP_CRON` no está definido.
* Comprueba que el Gateway se esté ejecutando 24/7 (sin suspensión ni reinicios).
* Verifica la configuración de zona horaria para la tarea (`--tz` vs zona horaria del host).

Depuración:

```bash
openclaw cron run <jobId> --force
openclaw cron runs --id <jobId> --limit 50
```

Documentación: [Tareas cron](/es/automation/cron-jobs), [Cron vs latido](/es/automation/cron-vs-heartbeat).

<div id="how-do-i-install-skills-on-linux">
  ### ¿Cómo instalo habilidades en Linux?
</div>

Usa **ClawHub** (CLI) o añade habilidades a tu espacio de trabajo. La UI de habilidades de macOS no está disponible en Linux.
Explora habilidades en https://clawhub.com.

Instala la CLI de ClawHub (elige un gestor de paquetes):

```bash
npm i -g clawhub
```

```bash
pnpm add -g clawhub
```

<div id="can-openclaw-run-tasks-on-a-schedule-or-continuously-in-the-background">
  ### ¿Puede OpenClaw ejecutar tareas programadas o continuamente en segundo plano?
</div>

Sí. Usa el planificador del Gateway:

* **Trabajos cron** para tareas programadas o recurrentes (persisten entre reinicios).
* **Latido (Heartbeat)** para comprobaciones periódicas de la “sesión principal”.
* **Trabajos aislados** para agentes autónomos que publican resúmenes o envían a chats.

Documentación: [Cron jobs](/es/automation/cron-jobs), [Cron vs Heartbeat](/es/automation/cron-vs-heartbeat),
[Heartbeat](/es/gateway/heartbeat).

**¿Puedo ejecutar habilidades exclusivas de Apple macOS desde Linux?**

No directamente. Las habilidades de macOS están restringidas por `metadata.openclaw.os` además de los binarios requeridos, y las habilidades solo aparecen en el mensaje del sistema cuando son elegibles en el **host del Gateway**. En Linux, las habilidades solo para `darwin` (como `imsg`, `apple-notes`, `apple-reminders`) no se cargarán a menos que anules esa restricción.

Tienes tres patrones admitidos:

**Opción A: ejecutar el Gateway en un Mac (la más simple).**\
Ejecuta el Gateway donde existan los binarios de macOS y luego conéctate desde Linux en [modo remoto](#how-do-i-run-openclaw-in-remote-mode-client-connects-to-a-gateway-elsewhere) o a través de Tailscale. Las habilidades se cargan normalmente porque el host del Gateway es macOS.

**Opción B: usar un nodo macOS (sin SSH).**\
Ejecuta el Gateway en Linux, empareja un nodo macOS (app de la barra de menús) y configura **Node Run Commands** en &quot;Preguntar siempre&quot; o &quot;Permitir siempre&quot; en el Mac. OpenClaw puede tratar las habilidades exclusivas de macOS como elegibles cuando los binarios necesarios existen en el nodo. El agente ejecuta esas habilidades mediante la herramienta `nodes`. Si eliges &quot;Preguntar siempre&quot;, al aprobar &quot;Permitir siempre&quot; en el mensaje se añade ese comando a la lista de permitidos.

**Opción C: usar binarios de macOS a través de SSH como proxy (avanzado).**\
Mantén el Gateway en Linux, pero haz que los binarios de CLI requeridos se resuelvan a scripts envoltorio SSH que se ejecutan en un Mac. Luego anula la configuración de la habilidad para permitir Linux de forma que siga siendo elegible.

1. Crea un script envoltorio SSH para el binario (ejemplo: `imsg`):
   ```bash
   #!/usr/bin/env bash
   set -euo pipefail
   exec ssh -T user@mac-host /opt/homebrew/bin/imsg "$@"
   ```
2. Coloca el script envoltorio en el `PATH` del host Linux (por ejemplo `~/bin/imsg`).
3. Anula los metadatos de la habilidad (en el espacio de trabajo o en `~/.openclaw/skills`) para permitir Linux:
   ```markdown
   ---
   name: imsg
   description: CLI de iMessage/SMS para listar chats, historial, monitorización y envío.
   metadata: {"openclaw":{"os":["darwin","linux"],"requires":{"bins":["imsg"]}}}
   ---
   ```
4. Inicia una nueva sesión para que se actualice la instantánea de las habilidades.

Para iMessage específicamente, también puedes apuntar `channels.imessage.cliPath` a un script envoltorio SSH (OpenClaw solo necesita stdio). Consulta [iMessage](/es/channels/imessage).

<div id="do-you-have-a-notion-or-heygen-integration">
  ### ¿Tienen una integración con Notion o HeyGen?
</div>

Por ahora no hay ninguna integración nativa.

Opciones:

* **Habilidad / complemento personalizado:** mejor para un acceso confiable a la API (tanto Notion como HeyGen tienen APIs).
* **Automatización del navegador:** funciona sin código, pero es más lenta y frágil.

Si quieres mantener el contexto por cliente (flujos de trabajo de agencia), un patrón sencillo es:

* Una página de Notion por cliente (contexto + preferencias + trabajo activo).
* Pedirle al agente que obtenga esa página al inicio de una sesión.

Si quieres una integración nativa, abre una solicitud de funcionalidad o crea una habilidad
dirigida a esas APIs.

Instala habilidades:

```bash
clawhub install <skill-slug>
clawhub update --all
```

ClawHub se instala en `./skills` dentro de tu directorio actual (o, en su defecto, en tu espacio de trabajo de OpenClaw configurado); OpenClaw lo interpreta como `<workspace>/skills` en la siguiente sesión. Para habilidades compartidas entre agentes, colócalas en `~/.openclaw/skills/<name>/SKILL.md`. Algunas habilidades esperan binarios instalados mediante Homebrew; en Linux eso implica Linuxbrew (consulta la entrada anterior de las preguntas frecuentes de Homebrew en Linux). Consulta [Skills](/es/tools/skills) y [ClawHub](/es/tools/clawhub).

<div id="how-do-i-install-the-chrome-extension-for-browser-takeover">
  ### ¿Cómo instalo la extensión de Chrome para tomar el control del navegador?
</div>

Usa el instalador integrado y luego carga la extensión desempaquetada en Chrome:

```bash
openclaw browser extension install
openclaw browser extension path
```

Luego, en Chrome → `chrome://extensions` → habilita “Modo de desarrollador” → “Cargar descomprimida” → elige esa carpeta.

Guía completa (incluyendo Gateway remoto y notas de seguridad): [Extensión de Chrome](/es/tools/chrome-extension)

Si el Gateway se ejecuta en la misma máquina que Chrome (configuración predeterminada), normalmente **no** necesitas nada adicional.
Si el Gateway se ejecuta en otro lugar, ejecuta un nodo host en la máquina del navegador para que el Gateway pueda hacer de proxy para las acciones del navegador.
Aun así, tienes que hacer clic en el botón de la extensión en la pestaña que quieres controlar (no se adjunta automáticamente).

<div id="sandboxing-and-memory">
  ## Sandbox y memoria
</div>

<div id="is-there-a-dedicated-sandboxing-doc">
  ### ¿Hay documentación específica sobre sandboxing?
</div>

Sí. Consulta [Sandboxing](/es/gateway/sandboxing). Para la configuración específica de Docker (Gateway completo en Docker o imágenes de sandbox), consulta [Docker](/es/install/docker).

**¿Puedo mantener los DMs personales pero hacer que los grupos sean públicos con sandbox usando un solo agente?**

Sí, si tu tráfico privado son los **DMs** y tu tráfico público son los **grupos**.

Usa `agents.defaults.sandbox.mode: "non-main"` para que las sesiones de grupos/canales (claves non-main) se ejecuten en Docker, mientras que la sesión principal de DM permanezca en el host. Después, restringe qué herramientas están disponibles en las sesiones en sandbox mediante `tools.sandbox.tools`.

Recorrido de configuración + ejemplo de configuración: [Groups: personal DMs + public groups](/es/concepts/groups#pattern-personal-dms-public-groups-single-agent)

Referencia clave de configuración: [Gateway configuration](/es/gateway/configuration#agentsdefaultssandbox)

<div id="how-do-i-bind-a-host-folder-into-the-sandbox">
  ### ¿Cómo vinculo una carpeta del host en el sandbox?
</div>

Configura `agents.defaults.sandbox.docker.binds` como `["host:path:mode"]` (por ejemplo, `"/home/user/src:/src:ro"`). Los binds globales y por agente se combinan; los binds por agente se ignoran cuando `scope: "shared"`. Usa `:ro` para cualquier contenido sensible y recuerda que los binds eluden las barreras del sistema de archivos del sandbox. Consulta [Sandboxing](/es/gateway/sandboxing#custom-bind-mounts) y [Sandbox vs Tool Policy vs Elevated](/es/gateway/sandbox-vs-tool-policy-vs-elevated#bind-mounts-security-quick-check) para ver ejemplos y notas de seguridad.

<div id="how-does-memory-work">
  ### ¿Cómo funciona la memoria?
</div>

La memoria de OpenClaw son simplemente archivos Markdown en el espacio de trabajo del agente:

* Notas diarias en `memory/YYYY-MM-DD.md`
* Notas seleccionadas a largo plazo en `MEMORY.md` (solo sesiones principales/privadas)

OpenClaw también ejecuta un **vaciado de memoria silencioso previo a la compactación** para recordarle al modelo que escriba notas persistentes antes de la compactación automática. Esto solo se ejecuta cuando el espacio de trabajo tiene permisos de escritura (las sandbox de solo lectura lo omiten). Consulta [Memoria](/es/concepts/memory).

<div id="memory-keeps-forgetting-things-how-do-i-make-it-stick">
  ### La memoria sigue olvidando cosas ¿Cómo hago para que no se le olviden?
</div>

Pídele al bot que **escriba el hecho en la memoria**. Las notas a largo plazo pertenecen a `MEMORY.md`,
el contexto a corto plazo va en `memory/YYYY-MM-DD.md`.

Esta sigue siendo un área que estamos mejorando. Ayuda recordarle al modelo que almacene memorias;
sabrá qué hacer. Si sigue olvidando cosas, verifica que el Gateway esté usando el mismo
espacio de trabajo en cada ejecución.

Docs: [Memoria](/es/concepts/memory), [Espacio de trabajo del agente](/es/concepts/agent-workspace).

<div id="does-semantic-memory-search-require-an-openai-api-key">
  ### ¿La búsqueda de memoria semántica requiere una clave de API de OpenAI?
</div>

Solo si usas **OpenAI embeddings**. Codex OAuth cubre chat/completions y
**no** concede acceso a embeddings, por lo que **iniciar sesión con Codex (OAuth o el
login de Codex CLI)** no ayuda para la búsqueda de memoria semántica. OpenAI embeddings
siguen necesitando una clave de API real (`OPENAI_API_KEY` o `models.providers.openai.apiKey`).

Si no configuras explícitamente un proveedor, OpenClaw selecciona automáticamente un proveedor cuando
puede resolver una clave de API (perfiles de autenticación, `models.providers.*.apiKey` o variables de entorno).
Prefiere OpenAI si se resuelve una clave de OpenAI; en caso contrario, Gemini si se resuelve una clave
de Gemini. Si ninguna de las dos claves está disponible, la búsqueda de memoria permanece deshabilitada hasta que la
configures. Si tienes configurada y disponible una ruta de modelo local, OpenClaw
prefiere `local`.

Si prefieres mantenerte en local, establece `memorySearch.provider = "local"` (y opcionalmente
`memorySearch.fallback = "none"`). Si quieres embeddings de Gemini, establece
`memorySearch.provider = "gemini"` y proporciona `GEMINI_API_KEY` (o
`memorySearch.remote.apiKey`). Admitimos modelos de embeddings **OpenAI, Gemini o locales**:
consulta [Memory](/es/concepts/memory) para los detalles de configuración.

<div id="does-memory-persist-forever-what-are-the-limits">
  ### ¿La memoria persiste para siempre? ¿Cuáles son los límites?
</div>

Los archivos de memoria se guardan en disco y persisten hasta que los eliminas. El límite lo marca tu almacenamiento, no el modelo. El **contexto de la sesión** sigue estando limitado por la ventana de contexto del modelo, así que las conversaciones largas se pueden compactar o truncar. Por eso existe la búsqueda en memoria: extrae solo las partes relevantes de nuevo en el contexto.

Documentación: [Memoria](/es/concepts/memory), [Contexto](/es/concepts/context).

<div id="where-things-live-on-disk">
  ## Dónde se almacena todo en disco
</div>

<div id="is-all-data-used-with-openclaw-saved-locally">
  ### ¿Todos los datos usados con OpenClaw se guardan localmente?
</div>

No: **el estado de OpenClaw es local**, pero **los servicios externos siguen viendo lo que les envías**.

* **Local por defecto:** las sesiones, archivos de memoria, configuración y el espacio de trabajo residen en el host del Gateway
  (`~/.openclaw` + tu directorio de espacio de trabajo).
* **Remoto por necesidad:** los mensajes que envías a proveedores de modelos (Anthropic/OpenAI/etc.) van a
  sus APIs, y las plataformas de chat (WhatsApp/Telegram/Slack/etc.) almacenan los datos de los mensajes en sus
  servidores.
* **Tú controlas la huella:** usar modelos locales mantiene los prompts en tu máquina, pero el tráfico de los canales
  sigue pasando por los servidores del canal.

Relacionado: [Espacio de trabajo del Agente](/es/concepts/agent-workspace), [Memoria](/es/concepts/memory).

<div id="where-does-openclaw-store-its-data">
  ### ¿Dónde almacena OpenClaw sus datos?
</div>

Todo se almacena bajo `$OPENCLAW_STATE_DIR` (valor predeterminado: `~/.openclaw`):

| Path | Propósito |
|------|-----------|
| `$OPENCLAW_STATE_DIR/openclaw.json` | Configuración principal (JSON5) |
| `$OPENCLAW_STATE_DIR/credentials/oauth.json` | Importación OAuth heredada (copiada en los perfiles de autenticación en el primer uso) |
| `$OPENCLAW_STATE_DIR/agents/<agentId>/agent/auth-profiles.json` | Perfiles de autenticación (OAuth + claves de API) |
| `$OPENCLAW_STATE_DIR/agents/<agentId>/agent/auth.json` | Caché de autenticación en tiempo de ejecución (gestionada automáticamente) |
| `$OPENCLAW_STATE_DIR/credentials/` | Estado del proveedor (por ejemplo, `whatsapp/<accountId>/creds.json`) |
| `$OPENCLAW_STATE_DIR/agents/` | Estado por agente (agentDir + sesiones) |
| `$OPENCLAW_STATE_DIR/agents/<agentId>/sessions/` | Historial y estado de la conversación (por agente) |
| `$OPENCLAW_STATE_DIR/agents/<agentId>/sessions/sessions.json` | Metadatos de la sesión (por agente) |

Ruta heredada de agente único: `~/.openclaw/agent/*` (migrada por `openclaw doctor`).

Tu **espacio de trabajo** (AGENTS.md, archivos de memoria, habilidades, etc.) es independiente y se configura a través de `agents.defaults.workspace` (valor predeterminado: `~/.openclaw/workspace`).

<div id="where-should-agentsmd-soulmd-usermd-memorymd-live">
  ### ¿Dónde deben ubicarse AGENTSmd SOULmd USERmd MEMORYmd?
</div>

Estos archivos se encuentran en el **espacio de trabajo del agente**, no en `~/.openclaw`.

* **Espacio de trabajo (por agente)**: `AGENTS.md`, `SOUL.md`, `IDENTITY.md`, `USER.md`,
  `MEMORY.md` (o `memory.md`), `memory/YYYY-MM-DD.md` y, de forma opcional, `HEARTBEAT.md`.
* **Directorio de estado (`~/.openclaw`)**: configuración, credenciales, perfiles de autenticación, sesiones, registros
  y habilidades compartidas (`~/.openclaw/skills`).

El espacio de trabajo predeterminado es `~/.openclaw/workspace`, configurable mediante:

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } }
}
```

Si el bot “olvida” cosas después de un reinicio, confirma que el Gateway está usando el mismo
espacio de trabajo en cada inicio (y recuerda: el modo remoto usa el **espacio de trabajo del host del Gateway**,
no el de tu portátil local).

Consejo: si quieres un comportamiento o preferencia duradero, pídele al bot que **lo escriba en
AGENTS.md o MEMORY.md** en lugar de depender del historial del chat.

Consulta [Espacio de trabajo del agente](/es/concepts/agent-workspace) y [Memoria](/es/concepts/memory).

<div id="whats-the-recommended-backup-strategy">
  ### ¿Cuál es la estrategia de copia de seguridad recomendada?
</div>

Coloca tu **espacio de trabajo del agente** en un repositorio git **privado** y realiza una copia de seguridad en algún lugar privado (por ejemplo, un repositorio privado en GitHub). Esto captura la memoria y los archivos AGENTS/SOUL/USER, y te permite restaurar la “mente” del asistente más adelante.

**No** hagas commit de nada dentro de `~/.openclaw` (credenciales, sesiones, tokens). Si necesitas una restauración completa, realiza copias de seguridad tanto del espacio de trabajo como del directorio de estado por separado (consulta la pregunta de migración anterior).

Documentación: [Espacio de trabajo del Agente](/es/concepts/agent-workspace).

<div id="how-do-i-completely-uninstall-openclaw">
  ### ¿Cómo desinstalo OpenClaw por completo?
</div>

Consulta la guía específica: [Desinstalar](/es/install/uninstall).

<div id="can-agents-work-outside-the-workspace">
  ### ¿Pueden los agentes trabajar fuera del espacio de trabajo?
</div>

Sí. El espacio de trabajo es el **directorio de trabajo actual (cwd)** y el ancla de memoria, no un sandbox estricto.
Las rutas relativas se resuelven dentro del espacio de trabajo, pero las rutas absolutas pueden acceder a otras
ubicaciones del host a menos que el sandbox esté habilitado. Si necesitas aislamiento, usa
[`agents.defaults.sandbox`](/es/gateway/sandboxing) o configuraciones de sandbox por agente. Si quieres que un repositorio
sea el directorio de trabajo predeterminado, apunta el `workspace` de ese agente a la raíz del repositorio. El repositorio de OpenClaw es solo código fuente; mantén el espacio de trabajo separado a menos que quieras de forma intencional que el agente trabaje dentro de él.

Ejemplo (repositorio como cwd predeterminado):

```json5
{
  agents: {
    defaults: {
      workspace: "~/Projects/my-repo"
    }
  }
}
```

<div id="im-in-remote-mode-where-is-the-session-store">
  ### Estoy en modo remoto, ¿dónde se encuentra el almacén de sesiones?
</div>

El estado de la sesión pertenece al **host del Gateway**. Si estás en modo remoto, el almacén de sesiones que te interesa está en la máquina remota, no en tu portátil. Consulta [Gestión de sesiones](/es/concepts/session).

<div id="config-basics">
  ## Conceptos básicos de la configuración
</div>

<div id="what-format-is-the-config-where-is-it">
  ### ¿En qué formato está la configuración y dónde se encuentra?
</div>

OpenClaw lee un archivo de configuración opcional en formato **JSON5** desde `$OPENCLAW_CONFIG_PATH` (valor predeterminado: `~/.openclaw/openclaw.json`):

```
$OPENCLAW_CONFIG_PATH
```

Si falta el archivo, se utilizan valores predeterminados relativamente seguros (incluido un espacio de trabajo predeterminado en `~/.openclaw/workspace`).

<div id="i-set-gatewaybind-lan-or-tailnet-and-now-nothing-listens-the-ui-says-unauthorized">
  ### Configuré gatewaybind lan o tailnet y ahora nada escucha; la UI muestra &quot;unauthorized&quot;
</div>

Los binds en interfaces que no son loopback **requieren autenticación**. Configura `gateway.auth.mode` + `gateway.auth.token` (o usa `OPENCLAW_GATEWAY_TOKEN`).

```json5
{
  gateway: {
    bind: "lan",
    auth: {
      mode: "token",
      token: "replace-me"
    }
  }
}
```

Notas:

* `gateway.remote.token` es solo para **llamadas CLI remotas**; no habilita la autenticación local del Gateway.
* El Control UI se autentica mediante `connect.params.auth.token` (almacenado en la configuración de la app/UI). Evita incluir tokens en las URL.

<div id="why-do-i-need-a-token-on-localhost-now">
  ### ¿Por qué ahora necesito un token en localhost?
</div>

El asistente genera un token del Gateway por defecto (incluso en loopback), por lo que **los clientes WS locales deben autenticarse**. Esto impide que otros procesos locales llamen al Gateway. Pega el token en la configuración de la Control UI (o en la configuración de tu cliente) para conectarte.

Si **de verdad** quieres un loopback abierto, elimina `gateway.auth` de tu configuración. El comando `openclaw doctor` puede generar un token en cualquier momento: `openclaw doctor --generate-gateway-token`.

<div id="do-i-have-to-restart-after-changing-config">
  ### ¿Tengo que reiniciar después de cambiar la configuración?
</div>

El Gateway supervisa la configuración y permite recarga en caliente:

* `gateway.reload.mode: "hybrid"` (predeterminado): aplica en caliente los cambios seguros y reinicia para los críticos
* También se permiten `hot`, `restart` y `off`

<div id="how-do-i-enable-web-search-and-web-fetch">
  ### ¿Cómo habilito web search y web fetch?
</div>

`web_fetch` funciona sin una clave de API. `web_search` requiere una clave de API de Brave Search. **Recomendado:** ejecuta `openclaw configure --section web` para guardarla en
`tools.web.search.apiKey`. Alternativa mediante variable de entorno: establece la variable `BRAVE_API_KEY` para el proceso del Gateway.

```json5
{
  tools: {
    web: {
      search: {
        enabled: true,
        apiKey: "BRAVE_API_KEY_HERE",
        maxResults: 5
      },
      fetch: {
        enabled: true
      }
    }
  }
}
```

Notas:

* Si utilizas listas de permitidos, añade `web_search`/`web_fetch` o `group:web`.
* `web_fetch` está habilitado de forma predeterminada (a menos que se deshabilite explícitamente).
* Los demonios leen las variables de entorno de `~/.openclaw/.env` (o del entorno del servicio).

Documentación: [Herramientas web](/es/tools/web).

<div id="how-do-i-run-a-central-gateway-with-specialized-workers-across-devices">
  ### ¿Cómo ejecuto un Gateway central con workers especializados en varios dispositivos?
</div>

El patrón común es **un Gateway** (p. ej. Raspberry Pi) más **nodos** y **agentes**:

* **Gateway (central):** administra canales (Signal/WhatsApp), enrutamiento y sesiones.
* **Nodos (dispositivos):** Mac/iOS/Android se conectan como periféricos y exponen herramientas locales (`system.run`, `canvas`, `camera`).
* **Agentes (workers):** cerebros/espacios de trabajo separados para roles especiales (p. ej. “Hetzner ops”, “Datos personales”).
* **Subagentes:** generan trabajo en segundo plano desde un agente principal cuando necesitas paralelismo.
* **TUI:** se conecta al Gateway y permite cambiar entre agentes/sesiones.

Docs: [Nodos](/es/nodes), [Acceso remoto](/es/gateway/remote), [Enrutamiento multi‑agente](/es/concepts/multi-agent), [Subagentes](/es/tools/subagents), [TUI](/es/tui).

<div id="can-the-openclaw-browser-run-headless">
  ### ¿Puede ejecutarse el navegador de OpenClaw en modo headless?
</div>

Sí. Es una opción de configuración:

```json5
{
  browser: { headless: true },
  agents: {
    defaults: {
      sandbox: { browser: { headless: true } }
    }
  }
}
```

El valor predeterminado es `false` (con interfaz). El modo sin interfaz gráfica (`headless`) es más propenso a activar comprobaciones anti‑bot en algunos sitios. Consulta [Browser](/es/tools/browser).

El modo sin interfaz gráfica utiliza **el mismo motor de Chromium** y funciona para la mayoría de las tareas de automatización (formularios, clics, scraping, inicios de sesión). Las principales diferencias son:

* No hay ventana de navegador visible (usa capturas de pantalla si necesitas elementos visuales).
* Algunos sitios son más estrictos con la automatización en modo sin interfaz gráfica (CAPTCHAs, anti‑bot).
  Por ejemplo, X/Twitter suele bloquear sesiones en modo sin interfaz gráfica.

<div id="how-do-i-use-brave-for-browser-control">
  ### ¿Cómo uso Brave para el control del navegador?
</div>

Configura `browser.executablePath` con la ruta al binario de Brave (o cualquier navegador basado en Chromium) y reinicia el Gateway.
Consulta los ejemplos de configuración completos en [Browser](/es/tools/browser#use-brave-or-another-chromium-based-browser).

<div id="remote-gateways-nodes">
  ## Gateways y nodos remotos
</div>

<div id="how-do-commands-propagate-between-telegram-the-gateway-and-nodes">
  ### ¿Cómo se propagan los comandos entre Telegram, el Gateway y los nodos?
</div>

Los mensajes de Telegram los gestiona el **Gateway**. El Gateway ejecuta el agente y
solo entonces invoca nodos a través del **Gateway WebSocket** cuando se necesita una herramienta de nodo:

Telegram → Gateway → Agente → `node.*` → Nodo → Gateway → Telegram

Los nodos no ven el tráfico entrante del proveedor; solo reciben llamadas RPC de nodo.

<div id="how-can-my-agent-access-my-computer-if-the-gateway-is-hosted-remotely">
  ### ¿Cómo puede mi agente acceder a mi computadora si el Gateway está alojado de forma remota?
</div>

Respuesta corta: **empareja tu computadora como un nodo**. El Gateway se ejecuta en otro lugar, pero puede
invocar herramientas `node.*` (screen, camera, system) en tu máquina local a través del WebSocket del Gateway.

Configuración típica:

1. Ejecuta el Gateway en el host siempre encendido (VPS/servidor doméstico).
2. Coloca el host del Gateway y tu computadora en la misma tailnet.
3. Asegúrate de que el WS del Gateway sea accesible (bind en la tailnet o túnel SSH).
4. Abre la app de macOS localmente y conéctate en modo **Remote over SSH** (o tailnet directa)
   para que pueda registrarse como nodo.
5. Aprueba el nodo en el Gateway:
   ```bash
   openclaw nodes pending
   openclaw nodes approve <requestId>
   ```

No se necesita un puente TCP independiente; los nodos se conectan a través del WebSocket del Gateway.

Recordatorio de seguridad: emparejar un nodo macOS permite `system.run` en esa máquina. Solo
empareja dispositivos en los que confíes y revisa [Security](/es/gateway/security).

Documentación: [Nodes](/es/nodes), [Gateway protocol](/es/gateway/protocol), [macOS remote mode](/es/platforms/mac/remote), [Security](/es/gateway/security).

<div id="tailscale-is-connected-but-i-get-no-replies-what-now">
  ### Tailscale está conectado, pero no recibo respuestas. ¿Qué hago ahora?
</div>

Verifica primero lo básico:

* Gateway está en ejecución: `openclaw gateway status`
* Estado del Gateway: `openclaw status`
* Estado de los canales: `openclaw channels status`

A continuación, verifica la autenticación y el enrutamiento:

* Si usas Tailscale Serve, asegúrate de que `gateway.auth.allowTailscale` esté configurado correctamente.
* Si te conectas mediante un túnel SSH, confirma que el túnel local esté activo y apunte al puerto correcto.
* Confirma que tus listas de permitidos (DM o grupo) incluyan tu cuenta.

Documentación: [Tailscale](/es/gateway/tailscale), [Acceso remoto](/es/gateway/remote), [Canales](/es/channels).

<div id="can-two-openclaw-instances-talk-to-each-other-local-vps">
  ### ¿Pueden dos instancias de OpenClaw comunicarse entre sí en un VPS local?
</div>

Sí. No hay un puente integrado de &quot;bot a bot&quot;, pero puedes conectarlas de varias
formas confiables:

**La más simple:** usa un canal de chat normal al que ambos bots puedan acceder (Telegram/Slack/WhatsApp).
Haz que el Bot A envíe un mensaje al Bot B y luego deja que el Bot B responda como de costumbre.

**Puente por CLI (genérico):** ejecuta un script que llame al otro Gateway con
`openclaw agent --message ... --deliver`, apuntando a un chat donde el otro bot
esté escuchando. Si uno de los bots está en un VPS remoto, apunta tu CLI a ese Gateway
remoto mediante SSH/Tailscale (ver [Acceso remoto](/es/gateway/remote)).

Patrón de ejemplo (ejecutar desde una máquina que pueda acceder al Gateway de destino):

```bash
openclaw agent --message "Hola desde el bot local" --deliver --channel telegram --reply-to <chat-id>
```

Sugerencia: agrega una salvaguarda para que los dos bots no entren en un bucle infinito (solo menciones, listas de permitidos de canal o una regla de &quot;no responder a mensajes de bots&quot;).

Documentación: [Acceso remoto](/es/gateway/remote), [CLI del Agente](/es/cli/agent), [Agent send](/es/tools/agent-send).

<div id="do-i-need-separate-vpses-for-multiple-agents">
  ### ¿Necesito VPS separados para varios agentes?
</div>

No. Un solo Gateway puede alojar varios agentes, cada uno con su propio espacio de trabajo, valores predeterminados de modelo
y enrutamiento. Esa es la configuración habitual y es mucho más barata y sencilla que ejecutar
un VPS por agente.

Usa VPS separados solo cuando necesites aislamiento estricto (límites de seguridad) o configuraciones
muy diferentes que no quieras compartir. De lo contrario, mantén un solo Gateway y
usa múltiples agentes o subagentes.

<div id="is-there-a-benefit-to-using-a-node-on-my-personal-laptop-instead-of-ssh-from-a-vps">
  ### ¿Hay alguna ventaja en usar un nodo en mi portátil personal en lugar de hacer SSH desde un VPS?
</div>

Sí: los nodos son la forma principal de llegar a tu portátil desde un Gateway remoto, y
desbloquean más que el simple acceso a la shell. El Gateway se ejecuta en macOS/Linux (Windows vía WSL2) y es
ligero (un VPS pequeño o un equipo tipo Raspberry Pi es suficiente; 4 GB de RAM bastan), así que una
configuración común es un host siempre encendido más tu portátil como nodo.

* **Sin SSH entrante.** Los nodos se conectan hacia fuera al WebSocket del Gateway y usan emparejamiento de dispositivo.
* **Controles de ejecución más seguros.** `system.run` está condicionado por listas de permitidos/aprobaciones de nodos en ese portátil.
* **Más herramientas de dispositivo.** Los nodos exponen `canvas`, `camera` y `screen`, además de `system.run`.
* **Automatización del navegador local.** Mantén el Gateway en un VPS, pero ejecuta Chrome localmente y retransmite el control
  con la extensión de Chrome y un nodo host en el portátil.

SSH está bien para acceso puntual a la shell, pero los nodos son más simples para flujos de trabajo continuos de agentes y
automatización de dispositivos.

Docs: [Nodes](/es/nodes), [Nodes CLI](/es/cli/nodes), [Chrome extension](/es/tools/chrome-extension).

<div id="should-i-install-on-a-second-laptop-or-just-add-a-node">
  ### ¿Debería instalarlo en un segundo portátil o solo añadir un nodo?
</div>

Si en el segundo portátil solo necesitas **herramientas locales** (screen/camera/exec), añádelo como
**nodo**. Así mantienes un único Gateway y evitas tener configuraciones duplicadas. Las herramientas locales del nodo
actualmente solo están disponibles para macOS, pero planeamos extenderlas a otros sistemas operativos.

Instala un segundo Gateway solo cuando necesites **aislamiento estricto** o dos bots completamente separados.

Documentación: [Nodos](/es/nodes), [CLI de nodos](/es/cli/nodes), [Múltiples Gateways](/es/gateway/multiple-gateways).

<div id="do-nodes-run-a-gateway-service">
  ### ¿Los nodos ejecutan un servicio de Gateway?
</div>

No. Solo debe ejecutarse **un Gateway** por host, a menos que ejecutes intencionalmente perfiles aislados (consulta [Multiple gateways](/es/gateway/multiple-gateways)). Los nodos son periféricos que se conectan
al Gateway (nodos iOS/Android o el “modo nodo” de macOS en la app de la barra de menús). Para hosts de nodos sin interfaz gráfica y control por CLI, consulta [Node host CLI](/es/cli/node).

Se requiere un reinicio completo para que surtan efecto los cambios en `gateway`, `discovery` y `canvasHost`.

<div id="is-there-an-api-rpc-way-to-apply-config">
  ### ¿Hay alguna forma de aplicar la configuración mediante API RPC?
</div>

Sí. `config.apply` valida y escribe la configuración completa y reinicia el Gateway como parte de la operación.

<div id="configapply-wiped-my-config-how-do-i-recover-and-avoid-this">
  ### config.apply borró mi configuración ¿Cómo la recupero y evito que vuelva a ocurrir?
</div>

`config.apply` reemplaza **toda la configuración**. Si envías un objeto parcial, todo lo
demás se elimina.

Para recuperarla:

* Restaura desde una copia de seguridad (git o una copia de `~/.openclaw/openclaw.json`).
* Si no tienes copia de seguridad, vuelve a ejecutar `openclaw doctor` y reconfigura canales/modelos.
* Si esto fue inesperado, reporta un bug e incluye tu última configuración conocida o cualquier copia de seguridad.
* Un agente local de programación a menudo puede reconstruir una configuración funcional a partir de los logs o del historial.

Para evitarlo:

* Usa `openclaw config set` para cambios pequeños.
* Usa `openclaw configure` para ediciones interactivas.

Docs: [Config](/es/cli/config), [Configure](/es/cli/configure), [Doctor](/es/gateway/doctor).

<div id="whats-a-minimal-sane-config-for-a-first-install">
  ### ¿Cuál es la configuración mínima razonable para una primera instalación?
</div>

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
  channels: { whatsapp: { allowFrom: ["+15555550123"] } }
}
```

Esto establece tu espacio de trabajo y limita quién puede activar el bot.

<div id="how-do-i-set-up-tailscale-on-a-vps-and-connect-from-my-mac">
  ### ¿Cómo configuro Tailscale en un VPS y me conecto desde mi Mac?
</div>

Pasos mínimos:

1. **Instalar + iniciar sesión en el VPS**
   ```bash
   curl -fsSL https://tailscale.com/install.sh | sh
   sudo tailscale up
   ```
2. **Instalar + iniciar sesión en tu Mac**
   * Usa la aplicación de Tailscale e inicia sesión en el mismo tailnet.
3. **Activar MagicDNS (recomendado)**
   * En la consola de administración de Tailscale, activa MagicDNS para que el VPS tenga un nombre de host estable.
4. **Usar el hostname del tailnet**
   * SSH: `ssh user@your-vps.tailnet-xxxx.ts.net`
   * Gateway WS: `ws://your-vps.tailnet-xxxx.ts.net:18789`

Si quieres acceder al Control UI sin SSH, usa Tailscale Serve en el VPS:

```bash
openclaw gateway --tailscale serve
```

Esto mantiene el Gateway limitado a la interfaz de loopback y expone HTTPS a través de Tailscale. Consulta [Tailscale](/es/gateway/tailscale).

<div id="how-do-i-connect-a-mac-node-to-a-remote-gateway-tailscale-serve">
  ### ¿Cómo conecto un nodo de Mac a un Gateway remoto con Tailscale Serve
</div>

Serve expone la **Gateway Control UI + WS**. Los nodos se conectan mediante el mismo endpoint WS del Gateway.

Configuración recomendada:

1. **Asegúrate de que el VPS y el Mac estén en el mismo Tailnet**.
2. **Usa la app de macOS en modo remoto** (el destino SSH puede ser el nombre de host del Tailnet).
   La app creará un túnel al puerto del Gateway y se conectará como nodo.
3. **Aprueba el nodo** en el Gateway:
   ```bash
   openclaw nodes pending
   openclaw nodes approve <requestId>
   ```

Documentación: [Gateway protocol](/es/gateway/protocol), [Discovery](/es/gateway/discovery), [macOS remote mode](/es/platforms/mac/remote).

<div id="env-vars-and-env-loading">
  ## Variables de entorno y carga de archivos .env
</div>

<div id="how-does-openclaw-load-environment-variables">
  ### ¿Cómo carga OpenClaw las variables de entorno?
</div>

OpenClaw lee las variables de entorno del proceso padre (shell, launchd/systemd, CI, etc.) y además carga:

* `.env` desde el directorio de trabajo actual
* un `.env` global de respaldo desde `~/.openclaw/.env` (también conocido como `$OPENCLAW_STATE_DIR/.env`)

Ninguno de los archivos `.env` reemplaza variables de entorno existentes.

También puedes definir variables de entorno en línea en la configuración (se aplican solo si faltan en el entorno del proceso):

```json5
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: { GROQ_API_KEY: "gsk-..." }
  }
}
```

Consulta [/environment](/es/environment) para ver el orden de precedencia completo y las fuentes.

<div id="i-started-the-gateway-via-the-service-and-my-env-vars-disappeared-what-now">
  ### Inicié el Gateway a través del servicio y mis variables de entorno desaparecieron ¿Y ahora qué?
</div>

Dos soluciones habituales:

1. Coloca las variables que faltan en `~/.openclaw/.env` para que se carguen incluso cuando el servicio no herede tu entorno de shell.
2. Activa la importación del shell (comodidad opcional):

```json5
{
  env: {
    shellEnv: {
      enabled: true,
      timeoutMs: 15000
    }
  }
}
```

Esto ejecuta tu shell de inicio de sesión e importa solo las claves esperadas que falten (sin sobrescribir nunca las existentes). Variables de entorno equivalentes:
`OPENCLAW_LOAD_SHELL_ENV=1`, `OPENCLAW_SHELL_ENV_TIMEOUT_MS=15000`.

<div id="i-set-copilotgithubtoken-but-models-status-shows-shell-env-off-why">
  ### Configuré COPILOTGITHUBTOKEN pero el estado de models muestra “Shell env: off”. ¿Por qué?
</div>

`openclaw models status` indica si la **importación del entorno de shell** está habilitada. “Shell env: off”
**no** significa que falten tus variables de entorno, solo significa que OpenClaw no cargará
tu shell de inicio automáticamente.

Si el Gateway se ejecuta como servicio (launchd/systemd), no heredará tu entorno
de shell. Soluciona esto haciendo una de las siguientes:

1. Pon el token en `~/.openclaw/.env`:
   ```
   COPILOT_GITHUB_TOKEN=...
   ```
2. O habilita la importación de shell (`env.shellEnv.enabled: true`).
3. O añádelo al bloque `env` de tu configuración (solo se aplica si falta).

Luego reinicia el Gateway y vuelve a comprobar:

```bash
openclaw models status
```

Los tokens de Copilot se obtienen de `COPILOT_GITHUB_TOKEN` (también `GH_TOKEN` / `GITHUB_TOKEN`).
Consulta [/concepts/model-providers](/es/concepts/model-providers) y [/environment](/es/environment).

<div id="sessions-multiple-chats">
  ## Sesiones y múltiples chats
</div>

<div id="how-do-i-start-a-fresh-conversation">
  ### ¿Cómo inicio una conversación desde cero?
</div>

Envía `/new` o `/reset` en un mensaje independiente. Consulta [Gestión de sesiones](/es/concepts/session).

<div id="do-sessions-reset-automatically-if-i-never-send-new">
  ### ¿Las sesiones se reinician automáticamente si nunca send algo nuevo?
</div>

Sí. Las sesiones expiran después de `session.idleMinutes` (valor predeterminado: **60**). El **siguiente**
mensaje inicia un ID de sesión nuevo para esa clave de chat. Esto no elimina
las transcripciones; solo inicia una sesión nueva.

```json5
{
  session: {
    idleMinutes: 240
  }
}
```

<div id="is-there-a-way-to-make-a-team-of-openclaw-instances-one-ceo-and-many-agents">
  ### ¿Hay alguna forma de crear un equipo de instancias de OpenClaw, un CEO y muchos agentes?
</div>

Sí, mediante **enrutamiento multiagente** y **subagentes**. Puedes crear un agente
coordinador y varios agentes ejecutores, cada uno con su propio espacio de trabajo y modelos.

Dicho esto, es mejor verlo como un **experimento divertido**. Consume muchos tokens y a menudo
es menos eficiente que usar un solo bot con sesiones separadas. El modelo típico que
planteamos es un solo bot con el que hablas, con distintas sesiones para trabajo en paralelo. Ese
bot también puede crear subagentes cuando sea necesario.

Documentación: [Enrutamiento multiagente](/es/concepts/multi-agent), [Subagentes](/es/tools/subagents), [CLI de agentes](/es/cli/agents).

<div id="why-did-context-get-truncated-midtask-how-do-i-prevent-it">
  ### ¿Por qué se truncó el contexto a mitad de una tarea? ¿Cómo lo evito?
</div>

El contexto de la sesión está limitado por la ventana del modelo. Chats largos, salidas muy grandes de herramientas o muchos archivos pueden activar la compactación o el truncamiento.

Qué puede ayudar:

* Pídele al bot que resuma el estado actual y lo escriba en un archivo.
* Usa `/compact` antes de tareas largas y `/new` al cambiar de tema.
* Mantén el contexto importante en el espacio de trabajo y pídele al bot que lo vuelva a leer desde ahí.
* Usa subagentes para trabajo prolongado o en paralelo para que el chat principal se mantenga más pequeño.
* Elige un modelo con una ventana de contexto más grande si esto ocurre a menudo.

<div id="how-do-i-completely-reset-openclaw-but-keep-it-installed">
  ### ¿Cómo puedo restablecer OpenClaw por completo sin desinstalarlo?
</div>

Utiliza el comando `reset`:

```bash
openclaw reset
```

Restablecimiento completo no interactivo:

```bash
openclaw reset --scope full --yes --non-interactive
```

Luego vuelve a ejecutar el asistente de configuración:

```bash
openclaw onboard --install-daemon
```

Notas:

* El asistente de configuración inicial también ofrece la opción **Reset** si detecta una configuración ya existente. Consulta [Wizard](/es/start/wizard).
* Si usaste perfiles (`--profile` / `OPENCLAW_PROFILE`), reinicia cada directorio de estado (de forma predeterminada, `~/.openclaw-<profile>`).
* Reinicio de desarrollo: `openclaw gateway --dev --reset` (solo para desarrollo; borra la configuración de desarrollo, credenciales, sesiones y espacio de trabajo).

<div id="im-getting-context-too-large-errors-how-do-i-reset-or-compact">
  ### Estoy recibiendo errores de contexto demasiado grande, ¿cómo lo reinicio o compacto?
</div>

Usa una de estas opciones:

* **Compactar** (mantiene la conversación pero resume las intervenciones más antiguas):
  ```
  /compact
  ```
  o `/compact <instructions>` para orientar el resumen.

* **Reiniciar** (ID de sesión nueva para la misma clave de chat):
  ```
  /new
  /reset
  ```

Si sigue ocurriendo:

* Activa o ajusta la **poda de sesión** (`agents.defaults.contextPruning`) para recortar la salida antigua de las herramientas.
* Usa un modelo con una ventana de contexto más grande.

Documentación: [Compaction](/es/concepts/compaction), [Session pruning](/es/concepts/session-pruning), [Session management](/es/concepts/session).

<div id="why-am-i-seeing-llm-request-rejected-messagesncontentxtooluseinput-field-required">
  ### ¿Por qué veo el mensaje «LLM request rejected messagesNcontentXtooluseinput Field required»?
</div>

Este es un error de validación del proveedor: el modelo emitió un bloque `tool_use` sin el
`input` requerido. Normalmente significa que el historial de la sesión está obsoleto o dañado (a menudo después de hilos largos
o de un cambio en una herramienta o en el esquema).

Solución: inicia una sesión nueva con `/new` (mensaje independiente).

<div id="why-am-i-getting-heartbeat-messages-every-30-minutes">
  ### ¿Por qué recibo mensajes de heartbeat cada 30 minutos?
</div>

Los heartbeat se ejecutan cada **30m** de forma predeterminada. Puedes ajustarlos o desactivarlos:

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "2h"   // o "0m" para deshabilitarlo
      }
    }
  }
}
```

Si `HEARTBEAT.md` existe pero está efectivamente vacío (solo líneas en blanco y
encabezados de Markdown como `# Heading`), OpenClaw omite la ejecución del latido para ahorrar llamadas a la API.
Si el archivo no existe, el latido sigue ejecutándose y el modelo decide qué hacer.

Las anulaciones específicas por agente usan `agents.list[].heartbeat`. Documentación: [Latido](/es/gateway/heartbeat).

<div id="do-i-need-to-add-a-bot-account-to-a-whatsapp-group">
  ### ¿Necesito añadir una cuenta de bot a un grupo de WhatsApp?
</div>

No. OpenClaw se ejecuta en **tu propia cuenta**, así que si tú estás en el grupo, OpenClaw puede verlo.
De forma predeterminada, las respuestas en grupos están bloqueadas hasta que autorices a los remitentes (`groupPolicy: "allowlist"`).

Si quieres que solo **tú** puedas activar respuestas en grupos:

```json5
{
  channels: {
    whatsapp: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15551234567"]
    }
  }
}
```

<div id="how-do-i-get-the-jid-of-a-whatsapp-group">
  ### ¿Cómo obtengo el JID de un grupo de WhatsApp?
</div>

Opción 1 (más rápida): haz tail de los logs y envía un mensaje de prueba al grupo:

```bash
openclaw logs --follow --json
```

Busca un `chatId` (o `from`) que termine en `@g.us`, por ejemplo:
`1234567890-1234567890@g.us`.

Opción 2 (si ya está configurado o en la lista de permitidos): muestra los grupos desde la configuración:

```bash
openclaw directory groups list --channel whatsapp
```

Documentación: [WhatsApp](/es/channels/whatsapp), [Directory](/es/cli/directory), [Logs](/es/cli/logs).

<div id="why-doesnt-openclaw-reply-in-a-group">
  ### ¿Por qué OpenClaw no responde en un grupo?
</div>

Dos causas frecuentes:

* El filtrado por mención está activado (por defecto). Debes @mencionar al bot (o coincidir con `mentionPatterns`).
* Configuraste `channels.whatsapp.groups` sin `"*"` y el grupo no está incluido en la lista de permitidos.

Consulta [Grupos](/es/concepts/groups) y [Mensajes de grupo](/es/concepts/group-messages).

<div id="do-groupsthreads-share-context-with-dms">
  ### ¿Comparten contexto los grupos/hilos con los DMs?
</div>

Los chats directos se fusionan en la sesión principal por defecto. Los grupos/canales tienen sus propias claves de sesión, y los temas de Telegram y los hilos de Discord son sesiones independientes. Consulta [Groups](/es/concepts/groups) y [Group messages](/es/concepts/group-messages).

<div id="how-many-workspaces-and-agents-can-i-create">
  ### ¿Cuántos espacios de trabajo y agentes puedo crear?
</div>

Sin límites estrictos. Decenas (incluso cientos) están bien, pero presta atención a:

* **Crecimiento en disco:** las sesiones + transcripciones se almacenan en `~/.openclaw/agents/<agentId>/sessions/`.
* **Costo de tokens:** más agentes implica más uso concurrente de modelos.
* **Sobrecarga operativa:** perfiles de autenticación por agente, espacios de trabajo y enrutamiento de canales.

Consejos:

* Mantén un espacio de trabajo **activo** por agente (`agents.defaults.workspace`).
* Depura sesiones antiguas (elimina las entradas JSONL o de almacenamiento) si el uso de disco crece.
* Usa `openclaw doctor` para detectar espacios de trabajo huérfanos y desajustes de perfiles.

<div id="can-i-run-multiple-bots-or-chats-at-the-same-time-slack-and-how-should-i-set-that-up">
  ### ¿Puedo ejecutar varios bots o chats al mismo tiempo en Slack y cómo debo configurarlo?
</div>

Sí. Usa **Enrutamiento Multi‑Agente** para ejecutar varios agentes aislados y enrutar los mensajes entrantes por canal/cuenta/peer. Slack se admite como canal y se puede vincular a agentes específicos.

El acceso al navegador es potente, pero no equivale a “puede hacer todo lo que haría una persona”: los sistemas anti‑bot, los CAPTCHAs y la autenticación multifactor (MFA) aún pueden bloquear la automatización. Para el control de navegador más fiable, usa el relay de la extensión de Chrome en la máquina que ejecuta el navegador (y mantén el Gateway en cualquier otro lugar).

Configuración recomendada:

* Host del Gateway siempre encendido (VPS/Mac mini).
* Un agente por rol (vinculaciones).
* Canal(es) de Slack vinculados a esos agentes.
* Navegador local mediante relay de extensión (o un nodo) cuando sea necesario.

Docs: [Enrutamiento Multi‑Agente](/es/concepts/multi-agent), [Slack](/es/channels/slack),
[Navegador](/es/tools/browser), [Extensión de Chrome](/es/tools/chrome-extension), [Nodos](/es/nodes).

<div id="models-defaults-selection-aliases-switching">
  ## Modelos: valores predeterminados, selección, alias y cambio
</div>

<div id="what-is-the-default-model">
  ### ¿Cuál es el modelo predeterminado?
</div>

El modelo predeterminado de OpenClaw es el que establezcas como:

```
agents.defaults.model.primary
```

Los modelos se especifican como `provider/model` (por ejemplo: `anthropic/claude-opus-4-5`). Si omites el proveedor, OpenClaw actualmente asume `anthropic` como valor predeterminado provisional por motivos de compatibilidad con versiones anteriores, pero aun así debes establecer **explícitamente** `provider/model`.

<div id="what-model-do-you-recommend">
  ### ¿Qué modelo recomiendas?
</div>

**Predeterminado recomendado:** `anthropic/claude-opus-4-5`.\
**Buena alternativa:** `anthropic/claude-sonnet-4-5`.\
**Fiable (con menos carácter):** `openai/gpt-5.2` - casi tan bueno como Opus, solo con menos personalidad.\
**Económico:** `zai/glm-4.7`.

MiniMax M2.1 tiene su propia documentación: [MiniMax](/es/providers/minimax) y
[Modelos locales](/es/gateway/local-models).

Regla general: usa el **mejor modelo que puedas permitirte** para trabajo de alto riesgo, y uno más barato
para chat rutinario o resúmenes. Puedes enrutar modelos por agente y usar subagentes para
paralelizar tareas largas (cada subagente consume tokens). Consulta [Modelos](/es/concepts/models) y
[Subagentes](/es/tools/subagents).

Advertencia importante: los modelos más débiles o cuantizados en exceso son más vulnerables a la inyección
de prompts y a comportamientos no seguros. Consulta [Seguridad](/es/gateway/security).

Más contexto: [Modelos](/es/concepts/models).

<div id="can-i-use-selfhosted-models-llamacpp-vllm-ollama">
  ### ¿Puedo usar modelos autoalojados (llamacpp, vLLM, Ollama)?
</div>

Sí. Si tu servidor local expone una api compatible con OpenAI, puedes configurar un
proveedor personalizado que lo use. Ollama es compatible directamente y es la vía más sencilla.

Nota de seguridad: los modelos más pequeños o fuertemente cuantizados son más vulnerables a la
inyección de prompts. Recomendamos encarecidamente **modelos grandes** para cualquier bot que pueda usar herramientas.
Si aún así quieres modelos pequeños, habilita el sandbox y listas de permitidos estrictas para las herramientas.

Documentación: [Ollama](/es/providers/ollama), [Modelos locales](/es/gateway/local-models),
[Proveedores de modelos](/es/concepts/model-providers), [Seguridad](/es/gateway/security),
[Sandboxing](/es/gateway/sandboxing).

<div id="how-do-i-switch-models-without-wiping-my-config">
  ### Cómo cambio de modelos sin borrar mi configuración
</div>

Usa **comandos de modelo** o edita solo los campos **model**. Evita reemplazar toda la configuración.

Opciones seguras:

* `/model` en el chat (rápido, solo para la sesión actual)
* `openclaw models set ...` (actualiza solo la configuración de modelos)
* `openclaw configure --section models` (interactivo)
* edita `agents.defaults.model` en `~/.openclaw/openclaw.json`

Evita usar `config.apply` con un objeto parcial a menos que realmente quieras reemplazar toda la configuración.
Si sobrescribiste la configuración, restáurala desde una copia de seguridad o vuelve a ejecutar `openclaw doctor` para repararla.

Documentación: [Models](/es/concepts/models), [Configure](/es/cli/configure), [Config](/es/cli/config), [Doctor](/es/gateway/doctor).

<div id="what-do-openclaw-flawd-and-krill-use-for-models">
  ### ¿Qué modelos utilizan OpenClaw, Flawd y Krill?
</div>

* **OpenClaw + Flawd:** Anthropic Opus (`anthropic/claude-opus-4-5`) - consulta [Anthropic](/es/providers/anthropic).
* **Krill:** MiniMax M2.1 (`minimax/MiniMax-M2.1`) - consulta [MiniMax](/es/providers/minimax).

<div id="how-do-i-switch-models-on-the-fly-without-restarting">
  ### ¿Cómo cambio de modelo sobre la marcha sin reiniciar nada?
</div>

Usa el comando `/model` como mensaje independiente:

```
/model sonnet
/model haiku
/model opus
/model gpt
/model gpt-mini
/model gemini
/model gemini-flash
```

Puedes listar los modelos disponibles con `/model`, `/model list` o `/model status`.

`/model` (y `/model list`) muestra un selector compacto y numerado. Selecciona usando el número:

```
/model 3
```

También puedes forzar el uso de un perfil de autenticación específico para el proveedor (por sesión):

```
/model opus@anthropic:default
/model opus@anthropic:work
```

Consejo: `/model status` muestra qué agente está activo, qué archivo `auth-profiles.json` se está usando y qué perfil de autenticación se intentará a continuación.
También muestra el endpoint del proveedor configurado (`baseUrl`) y el modo de API (`api`) cuando esté disponible.

**¿Cómo desanclo un perfil que configuré con `profile`**

Vuelve a ejecutar `/model` **sin** el sufijo `@profile`:

```
/model anthropic/claude-opus-4-5
```

Si quieres volver al valor predeterminado, elígelo en `/model` (o envía `/model <default provider/model>`).
Usa `/model status` para confirmar qué perfil de autenticación está activo.

<div id="can-i-use-gpt-52-for-daily-tasks-and-codex-52-for-coding">
  ### ¿Puedo usar GPT 5.2 para tareas diarias y Codex 5.2 para programar?
</div>

Sí. Configura uno como predeterminado y cambia cuando lo necesites:

* **Cambio rápido (por sesión):** `/model gpt-5.2` para tareas diarias, `/model gpt-5.2-codex` para programación.
* **Predeterminado + cambio:** establece `agents.defaults.model.primary` en `openai-codex/gpt-5.2`, luego cambia a `openai-codex/gpt-5.2-codex` cuando programes (o al revés).
* **Subagentes:** enruta las tareas de programación a subagentes con un modelo predeterminado diferente.

Consulta [Modelos](/es/concepts/models) y [Comandos slash](/es/tools/slash-commands).

<div id="why-do-i-see-model-is-not-allowed-and-then-no-reply">
  ### ¿Por qué veo «Model is not allowed» y luego no hay respuesta?
</div>

Si `agents.defaults.models` está definido, se convierte en la **lista de permitidos** para `/model` y cualquier sobrescritura de sesión. Elegir un modelo que no esté en esa lista devuelve:

```
Model "provider/model" is not allowed. Use /model to list available models.
```

Ese error se devuelve **en lugar de** una respuesta normal. Solución: añade ese modelo a
`agents.defaults.models`, quita la lista de permitidos o selecciona un modelo con `/model list`.

<div id="why-do-i-see-unknown-model-minimaxminimaxm21">
  ### ¿Por qué veo Unknown model minimaxMiniMaxM21?
</div>

Esto significa que el **proveedor no está configurado** (no se encontró ninguna configuración ni perfil de autenticación de MiniMax), por lo que el modelo no se puede resolver. Se incluye una corrección para esta detección en **2026.1.12** (aún sin publicar en el momento de redactar esto).

Lista de comprobación para la corrección:

1. Actualiza a **2026.1.12** (o ejecútalo desde `main` en el código fuente) y después reinicia el Gateway.
2. Asegúrate de que MiniMax esté configurado (asistente o JSON), o de que exista una clave de API de MiniMax
   en variables de entorno o perfiles de autenticación (env/auth) para que se pueda inyectar el proveedor.
3. Usa el ID de modelo exacto (sensible a mayúsculas y minúsculas): `minimax/MiniMax-M2.1` o
   `minimax/MiniMax-M2.1-lightning`.
4. Ejecuta:
   ```bash
   openclaw models list
   ```
   y elige uno de la lista (o usa `/model list` en el chat).

Consulta [MiniMax](/es/providers/minimax) y [Modelos](/es/concepts/models).

<div id="can-i-use-minimax-as-my-default-and-openai-for-complex-tasks">
  ### ¿Puedo usar MiniMax como mi modelo por defecto y OpenAI para tareas complejas?
</div>

Sí. Usa **MiniMax como modelo por defecto** y cambia de modelo **por sesión** cuando sea necesario.
Los fallbacks son para **errores**, no para “tareas difíciles”, así que usa `/model` o un agente separado.

**Opción A: cambiar por sesión**

```json5
{
  env: { MINIMAX_API_KEY: "sk-...", OPENAI_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "minimax/MiniMax-M2.1" },
      models: {
        "minimax/MiniMax-M2.1": { alias: "minimax" },
        "openai/gpt-5.2": { alias: "gpt" }
      }
    }
  }
}
```

Luego:

```
/model gpt
```

**Opción B: agentes separados**

* Agente A por defecto: MiniMax
* Agente B por defecto: OpenAI
* Enruta según el agente o usa `/agent` para cambiar

Documentación: [Modelos](/es/concepts/models), [Enrutamiento multiagente](/es/concepts/multi-agent), [MiniMax](/es/providers/minimax), [OpenAI](/es/providers/openai).

<div id="are-opus-sonnet-gpt-builtin-shortcuts">
  ### ¿Hay atajos integrados para opus, sonnet y gpt?
</div>

Sí. OpenClaw incluye algunos atajos predeterminados (solo se aplican cuando el modelo existe en `agents.defaults.models`):

* `opus` → `anthropic/claude-opus-4-5`
* `sonnet` → `anthropic/claude-sonnet-4-5`
* `gpt` → `openai/gpt-5.2`
* `gpt-mini` → `openai/gpt-5-mini`
* `gemini` → `google/gemini-3-pro-preview`
* `gemini-flash` → `google/gemini-3-flash-preview`

Si defines tu propio alias con el mismo nombre, tendrá prioridad tu valor.

<div id="how-do-i-defineoverride-model-shortcuts-aliases">
  ### ¿Cómo defino o sobrescribo alias de accesos directos de modelos?
</div>

Los alias provienen de `agents.defaults.models.<modelId>.alias`. Ejemplo:

```json5
{
  agents: {
    defaults: {
      model: { primary: "anthropic/claude-opus-4-5" },
      models: {
        "anthropic/claude-opus-4-5": { alias: "opus" },
        "anthropic/claude-sonnet-4-5": { alias: "sonnet" },
        "anthropic/claude-haiku-4-5": { alias: "haiku" }
      }
    }
  }
}
```

Entonces `/model sonnet` (o `/<alias>` cuando sea compatible) se resuelve en ese ID de modelo.

<div id="how-do-i-add-models-from-other-providers-like-openrouter-or-zai">
  ### ¿Cómo puedo añadir modelos de otros proveedores como OpenRouter o ZAI?
</div>

OpenRouter (pago por token; muchos modelos):

```json5
{
  agents: {
    defaults: {
      model: { primary: "openrouter/anthropic/claude-sonnet-4-5" },
      models: { "openrouter/anthropic/claude-sonnet-4-5": {} }
    }
  },
  env: { OPENROUTER_API_KEY: "sk-or-..." }
}
```

Z.AI (modelos GLM):

```json5
{
  agents: {
    defaults: {
      model: { primary: "zai/glm-4.7" },
      models: { "zai/glm-4.7": {} }
    }
  },
  env: { ZAI_API_KEY: "..." }
}
```

Si haces referencia a un proveedor o modelo pero falta la clave de proveedor necesaria, obtendrás un error de autenticación en tiempo de ejecución (por ejemplo, `No API key found for provider "zai"`).

**&quot;No API key found for provider&quot; después de añadir un nuevo agente**

Esto suele indicar que el **nuevo agente** tiene un almacén de autenticación vacío. La autenticación es por agente y
se almacena en:

```
~/.openclaw/agents/<agentId>/agent/auth-profiles.json
```

Opciones para solucionarlo:

* Ejecuta `openclaw agents add <id>` y configura la autenticación durante el asistente de configuración.
* O copia `auth-profiles.json` desde el `agentDir` del agente principal al `agentDir` del nuevo agente.

**No** reutilices `agentDir` entre agentes; esto provoca conflictos de autenticación/sesión.

<div id="model-failover-and-all-models-failed">
  ## Conmutación por error de modelos y «Todos los modelos han fallado»
</div>

<div id="how-does-failover-work">
  ### ¿Cómo funciona el failover?
</div>

El failover se realiza en dos etapas:

1. **Rotación del perfil de autenticación** dentro del mismo proveedor.
2. **Fallback de modelo** al siguiente modelo definido en `agents.defaults.model.fallbacks`.

Se aplican períodos de enfriamiento a los perfiles que fallen (exponential backoff), de modo que OpenClaw pueda seguir respondiendo incluso cuando un proveedor esté sujeto a limitación de tasa o falle temporalmente.

<div id="what-does-this-error-mean">
  ### ¿Qué significa este error?
</div>

```
No credentials found for profile "anthropic:default"
```

Significa que el sistema intentó usar el ID de perfil de autenticación `anthropic:default`, pero no pudo encontrar credenciales para dicho perfil en el almacén de autenticación esperado.

<div id="fix-checklist-for-no-credentials-found-for-profile-anthropicdefault">
  ### Lista de comprobación para corregir No credentials found for profile anthropicdefault
</div>

* **Comprueba dónde se almacenan los perfiles de autenticación** (rutas nuevas vs heredadas)
  * Actual: `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`
  * Heredada: `~/.openclaw/agent/*` (migrado por `openclaw doctor`)
* **Comprueba que el Gateway cargue tu variable de entorno**
  * Si configuras `ANTHROPIC_API_KEY` en tu shell pero ejecutas el Gateway mediante systemd/launchd, puede que no la herede. Ponla en `~/.openclaw/.env` o habilita `env.shellEnv`.
* **Asegúrate de que estás editando el agente correcto**
  * Las configuraciones con varios agentes implican que puede haber múltiples archivos `auth-profiles.json`.
* **Verifica rápidamente el estado de modelo/autenticación**
  * Usa `openclaw models status` para ver los modelos configurados y si los proveedores están autenticados.

**Lista de comprobación para corregir No credentials found for profile anthropic**

Esto significa que la ejecución está asociada a un perfil de autenticación de Anthropic, pero el Gateway
no puede encontrarlo en su almacén de autenticación.

* **Usa un setup-token**
  * Ejecuta `claude setup-token`, luego pégalo con `openclaw models auth setup-token --provider anthropic`.
  * Si el token se creó en otra máquina, usa `openclaw models auth paste-token --provider anthropic`.
* **Si en su lugar quieres usar una clave de API**
  * Coloca `ANTHROPIC_API_KEY` en `~/.openclaw/.env` en el **host del Gateway**.
  * Borra cualquier orden fijada que fuerce un perfil inexistente:
    ```bash
    openclaw models auth order clear --provider anthropic
    ```
* **Comprueba que estés ejecutando los comandos en el host del Gateway**
  * En modo remoto, los perfiles de autenticación se almacenan en la máquina del Gateway, no en tu portátil.

<div id="why-did-it-also-try-google-gemini-and-fail">
  ### ¿Por qué también intentó usar Google Gemini y falló?
</div>

Si la configuración de tu modelo incluye Google Gemini como opción de respaldo (o cambiaste a un atajo de Gemini), OpenClaw lo intentará durante el fallback de modelos. Si no has configurado las credenciales de Google, verás `No API key found for provider "google"`.

Solución: proporciona las credenciales de autenticación de Google o elimina/evita los modelos de Google en `agents.defaults.model.fallbacks` / alias para que el fallback no dirija el tráfico allí.

**Mensaje de rechazo de la solicitud LLM indicando que se requiere firma para Google Antigravity**

Causa: el historial de la sesión contiene **bloques de thinking sin firmas** (a menudo de un stream abortado/parcial). Google Antigravity requiere firmas para los bloques de thinking.

Solución: OpenClaw ahora elimina los bloques de thinking sin firmar para Google Antigravity Claude. Si el problema persiste, inicia una **sesión nueva** o establece `/thinking off` para ese agente.

<div id="auth-profiles-what-they-are-and-how-to-manage-them">
  ## Perfiles de autenticación: qué son y cómo administrarlos
</div>

Relacionado: [/concepts/oauth](/es/concepts/oauth) (flujos OAuth, almacenamiento de tokens, patrones multicuenta)

<div id="what-is-an-auth-profile">
  ### ¿Qué es un perfil de autenticación?
</div>

Un perfil de autenticación es un registro de credenciales identificado por un nombre (OAuth o API key) vinculado a un proveedor. Los perfiles se almacenan en:

```
~/.openclaw/agents/<agentId>/agent/auth-profiles.json
```

<div id="what-are-typical-profile-ids">
  ### ¿Cuáles son los identificadores de perfil típicos?
</div>

OpenClaw usa IDs con prefijo de proveedor, como:

* `anthropic:default` (común cuando no existe una identidad de correo electrónico)
* `anthropic:<email>` para identidades de OAuth
* IDs personalizados que elijas (p. ej., `anthropic:work`)

<div id="can-i-control-which-auth-profile-is-tried-first">
  ### ¿Puedo controlar qué perfil de autenticación se intenta primero?
</div>

Sí. La configuración admite metadatos opcionales para los perfiles y un orden por proveedor (`auth.order.<provider>`). Esto **no** almacena secretos; asigna IDs a proveedor/modo y define el orden de rotación.

OpenClaw puede omitir temporalmente un perfil si está en un período corto de **cooldown** (limitaciones de tasa/timeouts/errores de autenticación) o en un estado **deshabilitado** más prolongado (facturación/créditos insuficientes). Para inspeccionar esto, ejecuta `openclaw models status --json` y revisa `auth.unusableProfiles`. Ajuste fino: `auth.cooldowns.billingBackoffHours*`.

También puedes establecer una anulación de orden **por agente** (almacenada en el `auth-profiles.json` de ese agente) mediante la CLI:

```bash
# Defaults to the configured default agent (omit --agent)
openclaw models auth order get --provider anthropic

# Lock rotation to a single profile (only try this one)
openclaw models auth order set --provider anthropic anthropic:default

# Or set an explicit order (fallback within provider)
openclaw models auth order set --provider anthropic anthropic:work anthropic:default

# Limpiar la anulación (volver a config auth.order / round-robin)
openclaw models auth order clear --provider anthropic
```

Para dirigirte a un agente específico:

```bash
openclaw models auth order set --provider anthropic --agent main anthropic:default
```

<div id="oauth-vs-api-key-whats-the-difference">
  ### OAuth vs clave de API: ¿cuál es la diferencia?
</div>

OpenClaw admite ambos:

* **OAuth** suele utilizar acceso por suscripción (cuando corresponde).
* **Claves de API** usan facturación por token consumido.

El asistente de configuración admite explícitamente el `setup-token` de Anthropic y OAuth de OpenAI Codex y puede almacenar tus claves de API.

<div id="gateway-ports-already-running-and-remote-mode">
  ## Gateway: puertos, «ya se está ejecutando» y modo remoto
</div>

<div id="what-port-does-the-gateway-use">
  ### ¿Qué puerto usa el Gateway?
</div>

`gateway.port` controla el único puerto multiplexado para WebSocket y HTTP (Control UI, hooks, etc.).

Prioridad:

```
--port > OPENCLAW_GATEWAY_PORT > gateway.port > default 18789
```

<div id="why-does-openclaw-gateway-status-say-runtime-running-but-rpc-probe-failed">
  ### ¿Por qué `openclaw gateway status` dice Runtime running pero RPC probe failed?
</div>

Porque “running” es la vista del **supervisor** (launchd/systemd/schtasks). El sondeo RPC es la CLI conectándose directamente al WebSocket del Gateway y llamando a `status`.

Usa `openclaw gateway status` y confía en estas líneas:

* `Probe target:` (la URL que el sondeo usó realmente)
* `Listening:` (lo que está realmente vinculado/asociado al puerto)
* `Last gateway error:` (causa raíz frecuente cuando el proceso está vivo pero el puerto no está escuchando)

<div id="why-does-openclaw-gateway-status-show-config-cli-and-config-service-different">
  ### ¿Por qué openclaw gateway status muestra Config cli y Config service como diferentes?
</div>

Estás editando un archivo de configuración mientras el servicio está usando otro distinto (a menudo por un desajuste entre `--profile` y `OPENCLAW_STATE_DIR`).

Solución:

```bash
openclaw gateway install --force
```

Ejecútalo desde el mismo `--profile` o entorno que quieres que utilice el servicio.

<div id="what-does-another-gateway-instance-is-already-listening-mean">
  ### ¿Qué significa que otra instancia del Gateway ya está escuchando?
</div>

OpenClaw aplica un bloqueo en tiempo de ejecución enlazando el listener de WebSocket inmediatamente al iniciar (por defecto `ws://127.0.0.1:18789`). Si el bind falla con `EADDRINUSE`, lanza `GatewayLockError`, lo que indica que otra instancia ya está escuchando.

Solución: detén la otra instancia, libera el puerto o ejecuta con `openclaw gateway --port <port>`.

<div id="how-do-i-run-openclaw-in-remote-mode-client-connects-to-a-gateway-elsewhere">
  ### ¿Cómo ejecuto OpenClaw en modo remoto con un cliente que se conecta a un Gateway remoto?
</div>

Configura `gateway.mode: "remote"` y especifícale una URL WebSocket remota, opcionalmente con un token/contraseña:

```json5
{
  gateway: {
    mode: "remote",
    remote: {
      url: "ws://gateway.tailnet:18789",
      token: "your-token",
      password: "your-password"
    }
  }
}
```

Notas:

* `openclaw gateway` solo se inicia cuando `gateway.mode` es `local` (o si pasas el flag de override).
* La app de macOS supervisa el archivo de configuración y cambia de modo en tiempo real cuando estos valores cambian.

<div id="the-control-ui-says-unauthorized-or-keeps-reconnecting-what-now">
  ### The Control UI muestra “unauthorized” o se reconecta todo el tiempo. ¿Y ahora qué?
</div>

Tu Gateway se está ejecutando con autenticación habilitada (`gateway.auth.*`), pero la UI no está enviando el token/contraseña correcto.

Hechos (según el código):

* La Control UI almacena el token en la clave de localStorage del navegador `openclaw.control.settings.v1`.
* La UI puede importar `?token=...` (y/o `?password=...`) una sola vez y luego lo elimina de la URL.

Solución:

* Lo más rápido: `openclaw dashboard` (imprime y copia el enlace tokenizado, intenta abrirlo; muestra una sugerencia sobre SSH si está en modo headless).
* Si aún no tienes un token: `openclaw doctor --generate-gateway-token`.
* Si es remoto, crea primero un túnel: `ssh -N -L 18789:127.0.0.1:18789 user@host` y luego abre `http://127.0.0.1:18789/?token=...`.
* Establece `gateway.auth.token` (o `OPENCLAW_GATEWAY_TOKEN`) en el host del Gateway.
* En la configuración de la Control UI, pega el mismo token (o actualiza usando un enlace de un solo uso con `?token=...`).
* ¿Sigues atascado? Ejecuta `openclaw status --all` y sigue [Troubleshooting](/es/gateway/troubleshooting). Consulta [Dashboard](/es/web/dashboard) para ver los detalles de autenticación.

<div id="i-set-gatewaybind-tailnet-but-it-cant-bind-nothing-listens">
  ### Configuré gatewaybind tailnet pero no puede enlazar, no hay nada escuchando
</div>

La opción de enlace `tailnet` selecciona una IP de Tailscale de tus interfaces de red (100.64.0.0/10). Si la máquina no está en Tailscale (o la interfaz está caída), no hay nada a lo que vincularse.

Solución:

* Inicia Tailscale en ese host (para que tenga una dirección 100.x), o
* Cambia a `gateway.bind: "loopback"` / `"lan"`.

Nota: `tailnet` es explícito. `auto` prefiere loopback; usa `gateway.bind: "tailnet"` cuando quieras que solo haga bind en la tailnet.

<div id="can-i-run-multiple-gateways-on-the-same-host">
  ### ¿Puedo ejecutar varios Gateways en el mismo host?
</div>

Normalmente no: un solo Gateway puede ejecutar múltiples canales de mensajería y agentes. Usa varios Gateways solo cuando necesites redundancia (p. ej., un bot de rescate) o aislamiento estricto.

Sí, pero debes aislar:

* `OPENCLAW_CONFIG_PATH` (configuración por instancia)
* `OPENCLAW_STATE_DIR` (estado por instancia)
* `agents.defaults.workspace` (aislamiento de espacio de trabajo)
* `gateway.port` (puertos únicos)

Configuración rápida (recomendada):

* Usa `openclaw --profile <name> …` por instancia (crea automáticamente `~/.openclaw-<name>`).
* Establece un `gateway.port` único en la configuración de cada perfil (o usa `--port` para ejecuciones manuales).
* Instala un servicio por perfil: `openclaw --profile <name> gateway install`.

Los perfiles también añaden un sufijo a los nombres de servicio (`bot.molt.<profile>`; legado `com.openclaw.*`, `openclaw-gateway-<profile>.service`, `OpenClaw Gateway (<profile>)`).
Guía completa: [Múltiples Gateways](/es/gateway/multiple-gateways).

<div id="what-does-invalid-handshake-code-1008-mean">
  ### ¿Qué significa el código de handshake no válido 1008?
</div>

El Gateway es un **servidor WebSocket** y espera que el primer mensaje
sea un frame `connect`. Si recibe cualquier otra cosa, cierra la conexión
con el **código 1008** (incumplimiento de la política).

Causas comunes:

* Abriste la URL **HTTP** en un navegador (`http://...`) en lugar de en un cliente WS.
* Usaste el puerto o la ruta equivocados.
* Un proxy o túnel eliminó los encabezados de autenticación o envió una solicitud que no corresponde al Gateway.

Soluciones rápidas:

1. Usa la URL de WS: `ws://<host>:18789` (o `wss://...` si es HTTPS).
2. No abras el puerto de WS en una pestaña de navegador normal.
3. Si la autenticación está activada, incluye el token/contraseña en el frame `connect`.

Si estás usando la CLI o la TUI, la URL debería verse así:

```
openclaw tui --url ws://<host>:18789 --token <token>
```

Detalles del protocolo: [Protocolo de Gateway](/es/gateway/protocol).

<div id="logging-and-debugging">
  ## Registros y depuración
</div>

<div id="where-are-logs">
  ### ¿Dónde se encuentran los registros?
</div>

Registros en archivos (estructurados):

```
/tmp/openclaw/openclaw-YYYY-MM-DD.log
```

Puedes establecer una ruta fija mediante `logging.file`. El nivel de los registros en archivo se controla con `logging.level`. La verbosidad de la consola se controla con `--verbose` y `logging.consoleLevel`.

Forma más rápida de seguir los registros:

```bash
openclaw logs --follow
```

Registros del servicio/supervisor (cuando el Gateway se ejecuta a través de launchd/systemd):

* macOS: `$OPENCLAW_STATE_DIR/logs/gateway.log` y `gateway.err.log` (predeterminado: `~/.openclaw/logs/...`; los perfiles usan `~/.openclaw-<profile>/logs/...`)
* Linux: `journalctl --user -u openclaw-gateway[-<profile>].service -n 200 --no-pager`
* Windows: `schtasks /Query /TN "OpenClaw Gateway (<profile>)" /V /FO LIST`

Consulta [Solución de problemas](/es/gateway/troubleshooting#log-locations) para más detalles.

<div id="how-do-i-startstoprestart-the-gateway-service">
  ### ¿Cómo iniciar, detener o reiniciar el servicio de Gateway?
</div>

Usa los helpers de Gateway:

```bash
openclaw gateway status
openclaw gateway restart
```

Si ejecutas el Gateway manualmente, puedes usar `openclaw gateway --force` para volver a reclamar el puerto. Consulta [Gateway](/es/gateway).

<div id="i-closed-my-terminal-on-windows-how-do-i-restart-openclaw">
  ### Cerré mi terminal en Windows, ¿cómo vuelvo a iniciar OpenClaw?
</div>

Hay **dos modos de instalación en Windows**:

**1) WSL2 (recomendado):** el Gateway se ejecuta dentro de Linux.

Abre PowerShell, entra en WSL y luego reinicia:

```powershell
wsl
openclaw gateway status
openclaw gateway restart
```

Si nunca has instalado el servicio, inícialo en primer plano:

```bash
openclaw gateway run
```

**2) Windows de forma nativa (no recomendado):** El Gateway se ejecuta directamente en Windows.

Abre PowerShell y ejecuta:

```powershell
openclaw gateway status
openclaw gateway restart
```

Si lo ejecutas manualmente (sin servicio), utiliza:

```powershell
openclaw gateway run
```

Documentación: [Windows (WSL2)](/es/platforms/windows), [runbook del servicio Gateway](/es/gateway).

<div id="the-gateway-is-up-but-replies-never-arrive-what-should-i-check">
  ### El Gateway está levantado, pero las respuestas nunca llegan. ¿Qué debo comprobar?
</div>

Empieza con una revisión rápida del estado:

```bash
openclaw status
openclaw models status
openclaw channels status
openclaw logs --follow
```

Causas comunes:

* La autenticación del modelo no está cargada en el **host del Gateway** (comprueba `models status`).
* El emparejamiento o la lista de permitidos del canal está bloqueando las respuestas (revisa la configuración del canal y los logs).
* WebChat/Dashboard está abierto sin el token correcto.

Si estás accediendo de forma remota, confirma que el túnel o la conexión Tailscale está activa y que el
WebSocket del Gateway es accesible.

Documentación: [Canales](/es/channels), [Resolución de problemas](/es/gateway/troubleshooting), [Acceso remoto](/es/gateway/remote).

<div id="disconnected-from-gateway-no-reason-what-now">
  ### Desconectado del Gateway sin razón aparente, ¿qué hago ahora?
</div>

Normalmente esto significa que la UI perdió la conexión WebSocket. Verifica:

1. ¿Se está ejecutando el Gateway? `openclaw gateway status`
2. ¿Está el Gateway en buen estado? `openclaw status`
3. ¿La UI tiene el token correcto? `openclaw dashboard`
4. Si es remoto, ¿está activo el túnel/enlace de Tailscale?

Luego haz tail de los logs:

```bash
openclaw logs --follow
```

Documentación: [Panel de control](/es/web/dashboard), [Acceso remoto](/es/gateway/remote), [Solución de problemas](/es/gateway/troubleshooting).

<div id="telegram-setmycommands-fails-with-network-errors-what-should-i-check">
  ### Telegram setMyCommands falla con errores de red ¿Qué debo comprobar?
</div>

Empieza por revisar los registros y el estado del canal:

```bash
openclaw channels status
openclaw channels logs --channel telegram
```

Si estás en un VPS o detrás de un proxy, confirma que se permita el tráfico HTTPS saliente y que la resolución DNS funcione.
Si el Gateway está en remoto, asegúrate de que estás revisando los registros en el host del Gateway.

Documentación: [Telegram](/es/channels/telegram), [Resolución de problemas de canales](/es/channels/troubleshooting).

<div id="tui-shows-no-output-what-should-i-check">
  ### La TUI no muestra salida: ¿qué debo comprobar?
</div>

Primero, confirma que el Gateway sea accesible y que el agente pueda ejecutarse:

```bash
openclaw status
openclaw models status
openclaw logs --follow
```

En la TUI, usa `/status` para ver el estado actual. Si esperas recibir respuestas en un canal
de chat, asegúrate de que la entrega esté activada (`/deliver on`).

Documentación: [TUI](/es/tui), [Comandos slash](/es/tools/slash-commands).

<div id="how-do-i-completely-stop-then-start-the-gateway">
  ### ¿Cómo detengo por completo el Gateway y luego lo vuelvo a iniciar?
</div>

Si instalaste el servicio:

```bash
openclaw gateway stop
openclaw gateway start
```

Esto detiene/inicia el **servicio supervisado** (launchd en macOS, systemd en Linux).
Utiliza esto cuando el Gateway se ejecuta en segundo plano como daemon.

Si lo estás ejecutando en primer plano, deténlo con Ctrl‑C y luego:

```bash
openclaw gateway run
```

Documentación: [Guía de operación del servicio Gateway](/es/gateway).

<div id="eli5-openclaw-gateway-restart-vs-openclaw-gateway">
  ### ELI5 diferencia entre `openclaw gateway restart` y `openclaw gateway`
</div>

* `openclaw gateway restart`: reinicia el **servicio en segundo plano** (launchd/systemd).
* `openclaw gateway`: ejecuta el Gateway **en primer plano** para esta sesión de terminal.

Si instalaste el servicio, usa los comandos del Gateway. Usa `openclaw gateway` cuando
quieras una ejecución puntual en primer plano.

<div id="whats-the-fastest-way-to-get-more-details-when-something-fails">
  ### ¿Cuál es la forma más rápida de obtener más información cuando algo falla?
</div>

Inicia el Gateway con `--verbose` para obtener más información en la consola. Luego inspecciona el archivo de registro para revisar errores de autenticación de canales, enrutamiento de modelos y RPC.

<div id="media-attachments">
  ## Contenido multimedia y archivos adjuntos
</div>

<div id="my-skill-generated-an-imagepdf-but-nothing-was-sent">
  ### Mi skill generó un imagePDF pero no se envió nada
</div>

Los archivos adjuntos salientes del agente deben incluir una línea `MEDIA:<path-or-url>` (en una línea independiente). Consulta [Configuración del asistente de OpenClaw](/es/start/openclaw) y [Envío del Agente](/es/tools/agent-send).

Envío mediante la CLI:

```bash
openclaw message send --target +15555550123 --message "Aquí tienes" --media /path/to/file.png
```

También comprueba:

* Que el canal de destino admite contenido multimedia saliente y no está bloqueado por listas de permitidos.
* Que el archivo está dentro de los límites de tamaño del proveedor (las imágenes se redimensionan a un máximo de 2048px).

Consulta [Imágenes](/es/nodes/images).

<div id="security-and-access-control">
  ## Seguridad y control de acceso
</div>

<div id="is-it-safe-to-expose-openclaw-to-inbound-dms">
  ### ¿Es seguro exponer OpenClaw a DMs entrantes?
</div>

Trata los DMs entrantes como entrada no confiable. Los valores predeterminados están diseñados para reducir el riesgo:

* El comportamiento predeterminado en canales que admiten DMs es el **emparejamiento**:
  * Los remitentes desconocidos reciben un código de emparejamiento; el bot no procesa su mensaje.
  * Aprueba con: `openclaw pairing approve <channel> <code>`
  * Las solicitudes de emparejamiento pendientes se limitan a **3 por canal**; revisa `openclaw pairing list <channel>` si un código no llegó.
* Abrir los DMs públicamente requiere optar explícitamente por ello (configurar `dmPolicy: "open"` —que permite aceptar mensajes sin restricciones de cualquier usuario— y una lista de permitidos `"*"`).

Ejecuta `openclaw doctor` para detectar políticas de DM riesgosas.

<div id="is-prompt-injection-only-a-concern-for-public-bots">
  ### ¿Es la inyección de prompts solo un problema para bots públicos?
</div>

No. La inyección de prompts tiene que ver con **contenido no confiable**, no solo con quién puede enviar DMs al bot.
Si tu asistente lee contenido externo (búsquedas/fetch web, páginas del navegador, correos,
documentos, adjuntos, registros pegados), ese contenido puede incluir instrucciones que intenten
secuestrar el modelo. Esto puede ocurrir incluso si **tú eres el único remitente**.

El mayor riesgo aparece cuando las herramientas están habilitadas: se puede engañar al modelo para
exfiltrar contexto o invocar herramientas en tu nombre. Reduce el radio de impacto:

* usando un agente &quot;lector&quot; de solo lectura o con herramientas deshabilitadas para resumir contenido no confiable
* manteniendo `web_search` / `web_fetch` / `browser` desactivados para agentes con herramientas habilitadas
* usando sandbox y listas de permitidos estrictas para herramientas

Detalles: [Seguridad](/es/gateway/security).

<div id="should-my-bot-have-its-own-email-github-account-or-phone-number">
  ### ¿Mi bot debería tener su propia cuenta de correo electrónico, cuenta de GitHub o número de teléfono?
</div>

Sí, en la mayoría de los casos. Aislar el bot con cuentas y números de teléfono separados
reduce el alcance del daño si algo sale mal. Esto también facilita rotar credenciales
o revocar el acceso sin afectar tus cuentas personales.

Empieza de forma limitada. Da acceso solo a las herramientas y cuentas que realmente necesitas y amplía
más adelante si es necesario.

Documentación: [Seguridad](/es/gateway/security), [Emparejamiento](/es/start/pairing).

<div id="can-i-give-it-autonomy-over-my-text-messages-and-is-that-safe">
  ### ¿Puedo darle autonomía sobre mis mensajes de texto y es seguro hacerlo?
</div>

**No** recomendamos darle autonomía total sobre tus mensajes personales. El enfoque más seguro es:

* Mantén los mensajes directos en **modo de emparejamiento** o con una lista de permitidos estricta.
* Usa un **número o cuenta separados** si quieres que envíe mensajes en tu nombre.
* Deja que redacte y luego **aprueba antes de enviar**.

Si quieres experimentar, hazlo en una cuenta dedicada y mantenla aislada. Consulta
[Seguridad](/es/gateway/security).

<div id="can-i-use-cheaper-models-for-personal-assistant-tasks">
  ### ¿Puedo usar modelos más baratos para tareas de asistente personal?
</div>

Sí, **si** el agente es solo de chat y la entrada es confiable. Los niveles
inferiores son más susceptibles al secuestro de instrucciones, así que evítalos para agentes con herramientas habilitadas
o cuando leas contenido no confiable. Si tienes que usar un modelo más pequeño, restringe al máximo
las herramientas y ejecútalo dentro de un sandbox. Consulta [Security](/es/gateway/security).

<div id="i-ran-start-in-telegram-but-didnt-get-a-pairing-code">
  ### Ejecuté /start en Telegram pero no recibí un código de emparejamiento
</div>

Los códigos de emparejamiento se envían **solo** cuando un remitente desconocido envía un mensaje al bot y
`dmPolicy: "pairing"` está habilitado. `/start` por sí solo no genera un código.

Revisa las solicitudes pendientes:

```bash
openclaw pairing list telegram
```

Si deseas acceso inmediato, incluye tu ID de remitente en la lista de permitidos o establece `dmPolicy: "open"` (permite aceptar mensajes sin restricciones de cualquier usuario)
para esa cuenta.

<div id="whatsapp-will-it-message-my-contacts-how-does-pairing-work">
  ### WhatsApp: ¿enviará mensajes a mis contactos? ¿Cómo funciona el emparejamiento?
</div>

No. La política predeterminada de mensajes directos de WhatsApp es el **emparejamiento**. Los remitentes desconocidos solo reciben un código de emparejamiento y su mensaje **no se procesa**. OpenClaw solo responde a los chats que recibe o a envíos explícitos que tú inicies.

Aprueba el emparejamiento con:

```bash
openclaw pairing approve whatsapp <code>
```

Listar solicitudes pendientes:

```bash
openclaw pairing list whatsapp
```

Solicitud de número de teléfono del asistente: se usa para configurar tu **lista de permitidos/propietario** de modo que se permitan tus propios mensajes directos (MD). No se utiliza para el envío automático. Si lo ejecutas con tu número personal de WhatsApp, usa ese número y habilita `channels.whatsapp.selfChatMode`.

<div id="chat-commands-aborting-tasks-and-it-wont-stop">
  ## Comandos de chat, cancelación de tareas y cuando «no se detiene»
</div>

<div id="how-do-i-stop-internal-system-messages-from-showing-in-chat">
  ### ¿Cómo evito que se muestren los mensajes internos del sistema en el chat?
</div>

La mayoría de los mensajes internos o de herramientas solo aparecen cuando **verbose** o **reasoning** están activados
para esa sesión.

Corrígelo en el chat donde lo ves:

```
/verbose off
/reasoning off
```

Si sigue siendo ruidoso, revisa la configuración de la sesión en el Control UI y configura `verbose`
en **inherit**. Confirma también que no estés usando un perfil de bot con `verboseDefault` establecido
en `on` en la configuración.

Documentación: [Thinking and verbose](/es/tools/thinking), [Security](/es/gateway/security#reasoning--verbose-output-in-groups).

<div id="how-do-i-stopcancel-a-running-task">
  ### ¿Cómo detengo o cancelo una tarea en ejecución?
</div>

Envía cualquiera de estos **como un mensaje independiente** (sin barra):

```
stop
abort
esc
wait
exit
interrupt
```

Estos son disparadores de cancelación (no comandos con /).

Para procesos en segundo plano (desde la herramienta exec), puedes pedirle al agente que ejecute:

```
process action:kill sessionId:XXX
```

Descripción general de los comandos slash: consulta [Slash commands](/es/tools/slash-commands).

La mayoría de los comandos deben enviarse como un mensaje **independiente** que comience con `/`, pero algunos atajos (como `/status`) también funcionan en línea para remitentes en la lista de permitidos.

<div id="how-do-i-send-a-discord-message-from-telegram-crosscontext-messaging-denied">
  ### ¿Cómo envío un mensaje de Discord desde Telegram? Mensajería entre contextos denegada
</div>

OpenClaw bloquea la mensajería **entre proveedores** de forma predeterminada. Si una llamada de herramienta está vinculada a Telegram, no se enviará a Discord a menos que lo permitas explícitamente.

Habilita la mensajería entre proveedores para el agente:

```json5
{
  agents: {
    defaults: {
      tools: {
        message: {
          crossContext: {
            allowAcrossProviders: true,
            marker: { enabled: true, prefix: "[from {channel}] " }
          }
        }
      }
    }
  }
}
```

Reinicia el Gateway después de editar la configuración. Si solo necesitas esto para un único
agente, configúralo en `agents.list[].tools.message` en su lugar.

<div id="why-does-it-feel-like-the-bot-ignores-rapidfire-messages">
  ### ¿Por qué parece que el bot ignora los mensajes enviados en ráfaga?
</div>

El modo de cola controla cómo interactúan los nuevos mensajes con una ejecución en curso. Usa `/queue` para cambiar de modo:

* `steer` - los mensajes nuevos redirigen la tarea actual
* `followup` - ejecuta los mensajes de uno en uno
* `collect` - agrupa mensajes y responde una sola vez (predeterminado)
* `steer-backlog` - redirige ahora y luego procesa la cola pendiente
* `interrupt` - aborta la ejecución actual y empieza de nuevo

Puedes añadir opciones como `debounce:2s cap:25 drop:summarize` para los modos de seguimiento.

<div id="answer-the-exact-question-from-the-screenshotchat-log">
  ## Responde exactamente a la pregunta de la captura/registro de chat
</div>

**P: «¿Cuál es el modelo predeterminado para Anthropic con una clave de API?»**

**R:** En OpenClaw, las credenciales y la selección de modelo están separadas. Definir `ANTHROPIC_API_KEY` (o almacenar una clave de API de Anthropic en perfiles de autenticación) habilita la autenticación, pero el modelo predeterminado real es el que configures en `agents.defaults.model.primary` (por ejemplo, `anthropic/claude-sonnet-4-5` o `anthropic/claude-opus-4-5`). Si ves `No credentials found for profile "anthropic:default"`, significa que el Gateway no pudo encontrar credenciales de Anthropic en el `auth-profiles.json` esperado para el agente en ejecución.

***

¿Sigues atascado? Pregunta en [Discord](https://discord.com/invite/clawd) o abre una [discusión en GitHub](https://github.com/openclaw/openclaw/discussions).