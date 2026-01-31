---
title: Gmail Pub/Sub
summary: "Push de Gmail Pub/Sub conectado a webhooks de OpenClaw mediante gogcli"
read_when:
  - Conectar los activadores de la bandeja de entrada de Gmail con OpenClaw
  - Configurar el push de Pub/Sub para activar al agente
---

<div id="gmail-pubsub-openclaw">
  # Gmail Pub/Sub -&gt; OpenClaw
</div>

Objetivo: Gmail watch -&gt; push de Pub/Sub -&gt; `gog gmail watch serve` -&gt; webhook de OpenClaw.

<div id="prereqs">
  ## Requisitos previos
</div>

* `gcloud` instalado y con sesión iniciada ([guía de instalación](https://docs.cloud.google.com/sdk/docs/install-sdk)).
* `gog` (gogcli) instalado y autorizado para la cuenta de Gmail ([gogcli.sh](https://gogcli.sh/)).
* Hooks de OpenClaw habilitados (consulta [Webhooks](/es/automation/webhook)).
* `tailscale` con sesión iniciada ([tailscale.com](https://tailscale.com/)). La configuración admitida usa Tailscale Funnel para el endpoint público HTTPS.
  Otros servicios de túnel pueden funcionar, pero son de tipo DIY/no admitidos y requieren configuración manual.
  Por ahora, solo damos soporte a Tailscale.

Ejemplo de configuración de hook (habilita la asignación preconfigurada de Gmail):

```json5
{
  hooks: {
    enabled: true,
    token: "OPENCLAW_HOOK_TOKEN",
    path: "/hooks",
    presets: ["gmail"]
  }
}
```

Para entregar el resumen de Gmail a una interfaz de chat, sobrescribe el preset con un mapeo
que defina `deliver` y, opcionalmente, `channel`/`to`:

```json5
{
  hooks: {
    enabled: true,
    token: "OPENCLAW_HOOK_TOKEN",
    presets: ["gmail"],
    mappings: [
      {
        match: { path: "gmail" },
        action: "agent",
        wakeMode: "now",
        name: "Gmail",
        sessionKey: "hook:gmail:{{messages[0].id}}",
        messageTemplate:
          "New email from {{messages[0].from}}\nSubject: {{messages[0].subject}}\n{{messages[0].snippet}}\n{{messages[0].body}}",
        model: "openai/gpt-5.2-mini",
        deliver: true,
        channel: "last"
        // to: "+15551234567" // para: número de teléfono de destino
      }
    ]
  }
}
```

Si quieres un canal fijo, configura `channel` + `to`. De lo contrario, `channel: "last"`
usa la última ruta de entrega (recurre a WhatsApp como respaldo).

Para forzar un modelo más barato para ejecuciones de Gmail, configura `model` en la asignación
(`provider/model` o alias). Si impones `agents.defaults.models`, inclúyelo allí.

Para definir un modelo predeterminado y un nivel de razonamiento específicamente para hooks de Gmail, agrega
`hooks.gmail.model` / `hooks.gmail.thinking` en tu configuración:

```json5
{
  hooks: {
    gmail: {
      model: "openrouter/meta-llama/llama-3.3-70b-instruct:free",
      thinking: "off"
    }
  }
}
```

Notas:

* El `model`/`thinking` por hook en el mapeo sigue anulando estos valores predeterminados.
* Orden de fallback: `hooks.gmail.model` → `agents.defaults.model.fallbacks` → primario (auth/límites de velocidad/timeouts).
* Si se configura `agents.defaults.models`, el modelo de Gmail debe estar en la lista de permitidos.
* El contenido del hook de Gmail se encapsula dentro de límites de seguridad para contenido externo de forma predeterminada.
  Para deshabilitarlo (peligroso), configura `hooks.gmail.allowUnsafeExternalContent: true`.

Para personalizar aún más el procesamiento del payload, agrega `hooks.mappings` o un módulo de transformación JS/TS
en `hooks.transformsDir` (consulta [Webhooks](/es/automation/webhook)).

<div id="wizard-recommended">
  ## Asistente (recomendado)
</div>

Usa el asistente de OpenClaw para configurarlo todo (instala dependencias en macOS mediante brew):

```bash
openclaw webhooks gmail setup \
  --account openclaw@gmail.com
```

Valores predeterminados:

* Usa Tailscale Funnel para el endpoint público de push.
* Escribe la configuración `hooks.gmail` para `openclaw webhooks gmail run`.
* Habilita el preset del hook de Gmail (`hooks.presets: ["gmail"]`).

Nota sobre la ruta: cuando `tailscale.mode` está habilitado, OpenClaw establece
automáticamente `hooks.gmail.serve.path` en `/` y mantiene la ruta pública en
`hooks.gmail.tailscale.path` (por defecto `/gmail-pubsub`) porque Tailscale
elimina el prefijo de ruta establecido antes de hacer proxy.
Si necesitas que el backend reciba la ruta con prefijo, establece
`hooks.gmail.tailscale.target` (o `--tailscale-target`) en una URL completa como
`http://127.0.0.1:8788/gmail-pubsub` y haz que coincida con `hooks.gmail.serve.path`.

¿Quieres un endpoint personalizado? Usa `--push-endpoint <url>` o `--tailscale off`.

Nota sobre la plataforma: en macOS el asistente instala `gcloud`, `gogcli` y `tailscale`
mediante Homebrew; en Linux primero instálalos manualmente.

Inicio automático del Gateway (recomendado):

* Cuando `hooks.enabled=true` y `hooks.gmail.account` está configurado, el Gateway inicia
  `gog gmail watch serve` al arrancar y renueva automáticamente el watch.
* Establece `OPENCLAW_SKIP_GMAIL_WATCHER=1` para desactivarlo (útil si ejecutas tú mismo el daemon).
* No ejecutes el daemon manual al mismo tiempo, o te encontrarás con
  `listen tcp 127.0.0.1:8788: bind: address already in use`.

Daemon manual (inicia `gog gmail watch serve` + auto-renovación):

```bash
openclaw webhooks gmail run
```

<div id="one-time-setup">
  ## Configuración única
</div>

1. Selecciona el proyecto de GCP **al que pertenece el cliente OAuth** utilizado por `gog`.

```bash
gcloud auth login
gcloud config set project <project-id>
```

Nota: la función watch de Gmail requiere que el tema de Pub/Sub esté en el mismo proyecto que el cliente OAuth.

2. Habilita las API:

```bash
gcloud services enable gmail.googleapis.com pubsub.googleapis.com
```

3. Crea un tema:

```bash
gcloud pubsub topics create gog-gmail-watch
```

4. Permite la publicación push de Gmail:

```bash
gcloud pubsub topics add-iam-policy-binding gog-gmail-watch \
  --member=serviceAccount:gmail-api-push@system.gserviceaccount.com \
  --role=roles/pubsub.publisher
```

<div id="start-the-watch">
  ## Inicia el watch
</div>

```bash
gog gmail watch start \
  --account openclaw@gmail.com \
  --label INBOX \
  --topic projects/<project-id>/topics/gog-gmail-watch
```

Guarda el `history_id` del resultado (para depuración).

<div id="run-the-push-handler">
  ## Ejecutar el controlador de push
</div>

Ejemplo en local (autenticación mediante token compartido):

```bash
gog gmail watch serve \
  --account openclaw@gmail.com \
  --bind 127.0.0.1 \
  --port 8788 \
  --path /gmail-pubsub \
  --token <shared> \
  --hook-url http://127.0.0.1:18789/hooks/gmail \
  --hook-token OPENCLAW_HOOK_TOKEN \
  --include-body \
  --max-bytes 20000
```

Notas:

* `--token` protege el endpoint de push (`x-gog-token` o `?token=`).
* `--hook-url` apunta a OpenClaw `/hooks/gmail` (mapeado; ejecución aislada + resumen al principal).
* `--include-body` y `--max-bytes` controlan el fragmento del cuerpo enviado a OpenClaw.

Recomendado: `openclaw webhooks gmail run` envuelve el mismo flujo y renueva automáticamente el watch.

<div id="expose-the-handler-advanced-unsupported">
  ## Exponer el manejador (avanzado, no compatible)
</div>

Si necesitas un túnel que no use Tailscale, configúralo manualmente y usa la URL pública en la suscripción push (no compatible, sin salvaguardas):

```bash
cloudflared tunnel --url http://127.0.0.1:8788 --no-autoupdate
```

Utiliza la URL generada como endpoint de push:

```bash
gcloud pubsub subscriptions create gog-gmail-watch-push \
  --topic gog-gmail-watch \
  --push-endpoint "https://<public-url>/gmail-pubsub?token=<shared>"
```

Producción: utiliza un endpoint HTTPS estable y configura JWT de OIDC para Pub/Sub; luego ejecuta:

```bash
gog gmail watch serve --verify-oidc --oidc-email <svc@...>
```

<div id="test">
  ## Prueba
</div>

Envía un mensaje a la bandeja de entrada supervisada:

```bash
gog gmail send \
  --account openclaw@gmail.com \
  --to openclaw@gmail.com \
  --subject "watch test" \
  --body "ping"
```

Comprueba el estado e historial del watch:

```bash
gog gmail watch status --account openclaw@gmail.com
gog gmail history --account openclaw@gmail.com --since <historyId>
```

<div id="troubleshooting">
  ## Solución de problemas
</div>

* `Invalid topicName`: el proyecto no coincide (el tema no pertenece al proyecto del cliente OAuth).
* `User not authorized`: falta el rol `roles/pubsub.publisher` en el tema.
* Mensajes vacíos: el push de Gmail solo incluye `historyId`; recupéralos mediante `gog gmail history`.

<div id="cleanup">
  ## Limpieza
</div>

```bash
gog gmail watch stop --account openclaw@gmail.com
gcloud pubsub subscriptions delete gog-gmail-watch-push
gcloud pubsub topics delete gog-gmail-watch
```
