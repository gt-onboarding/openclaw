---
title: Complemento
summary: "Complementos/extensiones de OpenClaw: descubrimiento, configuración y seguridad"
read_when:
  - Agregar o modificar complementos/extensiones
  - Documentar reglas de instalación o carga de complementos
---

<div id="plugins-extensions">
  # Complementos (extensiones)
</div>

<div id="quick-start-new-to-plugins">
  ## Inicio rápido (¿eres nuevo con los complementos?)
</div>

Un complemento es simplemente un **pequeño módulo de código** que amplía OpenClaw con
funcionalidades adicionales (comandos, herramientas y Gateway RPC).

La mayoría de las veces usarás complementos cuando quieras una funcionalidad
que todavía no está integrada en el núcleo de OpenClaw (o cuando quieras
mantener las funciones opcionales fuera de tu instalación principal).

Vía rápida:

1. Mira qué ya está cargado:

```bash
openclaw plugins list
```

2. Instala un complemento oficial (por ejemplo, Voice Call):

```bash
openclaw plugins install @openclaw/voice-call
```

3. Reinicia el Gateway y después configúralo en `plugins.entries.<id>.config`.

Consulta [Voice Call](/es/plugins/voice-call) para ver un ejemplo concreto de complemento.

<div id="available-plugins-official">
  ## Complementos disponibles (oficiales)
</div>

* Microsoft Teams solo está disponible como complemento a partir de 2026.1.15; instala `@openclaw/msteams` si usas Teams.
* Memory (Core): complemento integrado de búsqueda en memoria (activado de forma predeterminada mediante `plugins.slots.memory`)
* Memory (LanceDB): complemento integrado de memoria a largo plazo (recuperación/captura automática; establece `plugins.slots.memory = "memory-lancedb"`)
* [Voice Call](/es/plugins/voice-call): `@openclaw/voice-call`
* [Zalo Personal](/es/plugins/zalouser): `@openclaw/zalouser`
* [Matrix](/es/channels/matrix): `@openclaw/matrix`
* [Nostr](/es/channels/nostr): `@openclaw/nostr`
* [Zalo](/es/channels/zalo): `@openclaw/zalo`
* [Microsoft Teams](/es/channels/msteams): `@openclaw/msteams`
* Google Antigravity OAuth (autenticación de proveedor): incluido como `google-antigravity-auth` (desactivado de forma predeterminada)
* Gemini CLI OAuth (autenticación de proveedor): incluido como `google-gemini-cli-auth` (desactivado de forma predeterminada)
* Qwen OAuth (autenticación de proveedor): incluido como `qwen-portal-auth` (desactivado de forma predeterminada)
* Copilot Proxy (autenticación de proveedor): puente local de Copilot Proxy para VS Code; distinto del inicio de sesión en dispositivo integrado `github-copilot` (incluido, desactivado de forma predeterminada)

Los complementos de OpenClaw son **módulos TypeScript** cargados en tiempo de ejecución mediante jiti. **La validación de configuración no ejecuta código del complemento**; en su lugar utiliza el manifiesto del complemento y el JSON Schema. Consulta el [Plugin manifest](/es/plugins/manifest).

Los complementos pueden registrar:

* Métodos RPC del Gateway
* Manejadores HTTP del Gateway
* Herramientas de agente
* Comandos de la CLI
* Servicios en segundo plano
* Validación de configuración opcional
* **Habilidades** (incluyendo directorios `skills` en el manifiesto del complemento)
* **Comandos de respuesta automática** (se ejecutan sin invocar al agente de IA)

Los complementos se ejecutan **en el mismo proceso** que el Gateway, así que trátalos como código confiable.
Guía para creación de herramientas: [Plugin agent tools](/es/plugins/agent-tools).

<div id="runtime-helpers">
  ## Funciones auxiliares de tiempo de ejecución
</div>

Los complementos pueden acceder a determinadas funciones auxiliares principales mediante `api.runtime`. Para TTS de telefonía:

```ts
const result = await api.runtime.tts.textToSpeechTelephony({
  text: "Hello from OpenClaw",
  cfg: api.config,
});
```

Notas:

* Usa la configuración central de `messages.tts` (OpenAI o ElevenLabs).
* Devuelve un búfer de audio PCM + frecuencia de muestreo. Los complementos deben remuestrear y codificar para los proveedores.
* Edge TTS no es compatible para telefonía.

<div id="discovery-precedence">
  ## Descubrimiento y precedencia
</div>

OpenClaw escanea, en este orden:

1. Rutas de configuración

* `plugins.load.paths` (archivo o directorio)

2. Extensiones del espacio de trabajo

* `<workspace>/.openclaw/extensions/*.ts`
* `<workspace>/.openclaw/extensions/*/index.ts`

3. Extensiones globales

* `~/.openclaw/extensions/*.ts`
* `~/.openclaw/extensions/*/index.ts`

4. Extensiones incluidas (se distribuyen con OpenClaw, **desactivadas de forma predeterminada**)

* `<openclaw>/extensions/*`

Los complementos incluidos deben habilitarse explícitamente mediante `plugins.entries.<id>.enabled`
u `openclaw plugins enable <id>`. Los complementos instalados se habilitan de forma predeterminada,
pero se pueden desactivar del mismo modo.

Cada complemento debe incluir un archivo `openclaw.plugin.json` en su directorio raíz. Si una ruta
apunta a un archivo, la raíz del complemento es el directorio de ese archivo y este debe contener el
manifiesto.

Si varios complementos se resuelven al mismo ID, gana la primera coincidencia en el orden anterior
y se ignoran las copias con menor precedencia.

<div id="package-packs">
  ### Conjuntos de paquetes
</div>

El directorio de un complemento puede incluir un `package.json` con `openclaw.extensions`:

```json
{
  "name": "my-pack",
  "openclaw": {
    "extensions": ["./src/safety.ts", "./src/tools.ts"]
  }
}
```

Cada entrada se convierte en un complemento. Si el paquete declara varias extensiones, el ID del complemento
pasa a ser `name/<fileBase>`.

Si tu complemento importa dependencias de npm, instálalas en ese directorio para que
`node_modules` esté disponible (`npm install` / `pnpm install`).

<div id="channel-catalog-metadata">
  ### Metadatos del catálogo de canales
</div>

Los complementos de canal pueden anunciar metadatos de incorporación mediante `openclaw.channel` e
instrucciones de instalación mediante `openclaw.install`. Esto mantiene el catálogo principal sin datos específicos de cada complemento.

Ejemplo:

```json
{
  "name": "@openclaw/nextcloud-talk",
  "openclaw": {
    "extensions": ["./index.ts"],
    "channel": {
      "id": "nextcloud-talk",
      "label": "Nextcloud Talk",
      "selectionLabel": "Nextcloud Talk (self-hosted)",
      "docsPath": "/channels/nextcloud-talk",
      "docsLabel": "nextcloud-talk",
      "blurb": "Chat autoalojado mediante bots webhook de Nextcloud Talk.",
      "order": 65,
      "aliases": ["nc-talk", "nc"]
    },
    "install": {
      "npmSpec": "@openclaw/nextcloud-talk",
      "localPath": "extensions/nextcloud-talk",
      "defaultChoice": "npm"
    }
  }
}
```

OpenClaw también puede combinar **catálogos de canales externos** (por ejemplo, una
exportación de un registro MPM). Coloca un archivo JSON en una de las siguientes rutas:

* `~/.openclaw/mpm/plugins.json`
* `~/.openclaw/mpm/catalog.json`
* `~/.openclaw/plugins/catalog.json`

O configura `OPENCLAW_PLUGIN_CATALOG_PATHS` (o `OPENCLAW_MPM_CATALOG_PATHS`) para que apunte a uno
o más archivos JSON (delimitados por comas, punto y coma o como en `PATH`). Cada archivo debe
contener `{ "entries": [ { "name": "@scope/pkg", "openclaw": { "channel": {...}, "install": {...} } } ] }`.

<div id="plugin-ids">
  ## IDs de complementos
</div>

IDs de complementos predeterminados:

* Paquete: campo `name` de `package.json`
* Archivo independiente: nombre base del archivo (`~/.../voice-call.ts` → `voice-call`)

Si un complemento exporta `id`, OpenClaw lo utiliza, pero muestra una advertencia cuando no coincide con el ID configurado.

<div id="config">
  ## Configuración
</div>

```json5
{
  plugins: {
    enabled: true,
    allow: ["voice-call"],
    deny: ["untrusted-plugin"],
    load: { paths: ["~/Projects/oss/voice-call-extension"] },
    entries: {
      "voice-call": { enabled: true, config: { provider: "twilio" } }
    }
  }
}
```

Fields:

* `enabled`: interruptor principal (predeterminado: true)
* `allow`: lista de permitidos (opcional)
* `deny`: lista de denegados (opcional; `deny` prevalece)
* `load.paths`: archivos/directorios de complementos adicionales
* `entries.<id>`: interruptores y configuración por complemento

Los cambios de configuración **requieren un reinicio del Gateway**.

Reglas de validación (estrictas):

* Los IDs de complemento desconocidos en `entries`, `allow`, `deny` o `slots` son **errores**.
* Las claves `channels.<id>` desconocidas son **errores** a menos que un manifiesto de complemento declare
  el ID del canal.
* La configuración del complemento se valida usando el JSON Schema incrustado en
  `openclaw.plugin.json` (`configSchema`).
* Si un complemento está deshabilitado, su configuración se conserva y se emite una **advertencia**.

<div id="plugin-slots-exclusive-categories">
  ## Slots de complementos (categorías exclusivas)
</div>

Algunas categorías de complementos son **exclusivas** (solo uno activo a la vez). Usa
`plugins.slots` para seleccionar qué complemento es el propietario del slot:

```json5
{
  plugins: {
    slots: {
      memory: "memory-core" // o "none" para deshabilitar los complementos de memoria
    }
  }
}
```

Si varios complementos declaran `kind: "memory"`, solo se carga el que esté seleccionado. Los demás se deshabilitan con diagnósticos.

<div id="control-ui-schema-labels">
  ## Control UI (schema + etiquetas)
</div>

La Control UI usa `config.schema` (JSON Schema + `uiHints`) para renderizar mejores formularios.

OpenClaw amplía `uiHints` en tiempo de ejecución en función de los complementos detectados:

* Agrega etiquetas por complemento para `plugins.entries.<id>` / `.enabled` / `.config`
* Fusiona sugerencias opcionales de campos de configuración proporcionadas por el complemento bajo:
  `plugins.entries.<id>.config.<field>`

Si quieres que los campos de configuración de tu complemento muestren buenas etiquetas/marcadores de posición (y marquen los secretos como sensibles),
proporciona `uiHints` junto con tu JSON Schema en el manifiesto del complemento.

Ejemplo:

```json
{
  "id": "my-plugin",
  "configSchema": {
    "type": "object",
    "additionalProperties": false,
    "properties": {
      "apiKey": { "type": "string" },
      "region": { "type": "string" }
    }
  },
  "uiHints": {
    "apiKey": { "label": "API Key", "sensitive": true },
    "region": { "label": "Región", "placeholder": "us-east-1" }
  }
}
```

<div id="cli">
  ## CLI
</div>

```bash
openclaw plugins list
openclaw plugins info <id>
openclaw plugins install <path>                 # copia un archivo/directorio local en ~/.openclaw/extensions/<id>
openclaw plugins install ./extensions/voice-call # relative path ok
openclaw plugins install ./plugin.tgz           # install from a local tarball
openclaw plugins install ./plugin.zip           # install from a local zip
openclaw plugins install -l ./extensions/voice-call # link (no copy) for dev
openclaw plugins install @openclaw/voice-call # install from npm
openclaw plugins update <id>
openclaw plugins update --all
openclaw plugins enable <id>
openclaw plugins disable <id>
openclaw plugins doctor
```

`plugins update` solo funciona con instalaciones de npm que estén registradas bajo `plugins.installs`.

Los complementos también pueden registrar sus propios comandos de primer nivel (ejemplo: `openclaw voicecall`).

<div id="plugin-api-overview">
  ## API de complementos (resumen)
</div>

Los complementos exportan:

* Una función: `(api) => { ... }`
* Un objeto: `{ id, name, configSchema, register(api) { ... } }`

<div id="plugin-hooks">
  ## Hooks de complementos
</div>

Los complementos pueden proporcionar hooks y registrarlos en tiempo de ejecución. Esto permite que un complemento incorpore automatización basada en eventos sin tener que instalar por separado un paquete de hooks.

<div id="example">
  ### Ejemplo
</div>

```
import { registerPluginHooksFromDir } from "openclaw/plugin-sdk";

export default function register(api) {
  registerPluginHooksFromDir(api, "./hooks");
}
```

Notas:

* Los directorios de hooks siguen la estructura normal de hooks (`HOOK.md` + `handler.ts`).
* Las reglas de elegibilidad de hooks se siguen aplicando (requisitos de sistema operativo/binarios/entorno/configuración).
* Los hooks gestionados por complementos aparecen en `openclaw hooks list` con `plugin:<id>`.
* No puedes habilitar ni deshabilitar hooks gestionados por complementos mediante `openclaw hooks`; en su lugar, habilita o deshabilita el complemento.

<div id="provider-plugins-model-auth">
  ## Complementos de proveedores (autenticación de modelos)
</div>

Los complementos pueden registrar flujos de **autenticación de proveedores de modelos** para que los usuarios puedan configurar OAuth o
claves de API directamente en OpenClaw (sin necesidad de scripts externos).

Registra un proveedor mediante `api.registerProvider(...)`. Cada proveedor expone uno
o más métodos de autenticación (OAuth, clave de API, código de dispositivo, etc.). Estos métodos se utilizan en:

* `openclaw models auth login --provider <id> [--method <id>]`

Ejemplo:

```ts
api.registerProvider({
  id: "acme",
  label: "AcmeAI",
  auth: [
    {
      id: "oauth",
      label: "OAuth",
      kind: "oauth",
      run: async (ctx) => {
        // Ejecuta el flujo OAuth y devuelve los perfiles de autenticación.
        return {
          profiles: [
            {
              profileId: "acme:default",
              credential: {
                type: "oauth",
                provider: "acme",
                access: "...",
                refresh: "...",
                expires: Date.now() + 3600 * 1000,
              },
            },
          ],
          defaultModel: "acme/opus-1",
        };
      },
    },
  ],
});
```

Notas:

* `run` recibe un `ProviderAuthContext` con las funciones auxiliares `prompter`, `runtime`,
  `openUrl` y `oauth.createVpsAwareHandlers`.
* Devuelve `configPatch` cuando necesites añadir modelos predeterminados o configuración del proveedor.
* Devuelve `defaultModel` para que `--set-default` pueda actualizar los valores predeterminados del agente.

<div id="register-a-messaging-channel">
  ### Registrar un canal de mensajería
</div>

Los complementos pueden registrar **complementos de canal** que se comportan como canales integrados
(WhatsApp, Telegram, etc.). La configuración del canal se define en `channels.<id>` y la valida
el código de tu complemento de canal.

```ts
const myChannel = {
  id: "acmechat",
  meta: {
    id: "acmechat",
    label: "AcmeChat",
    selectionLabel: "AcmeChat (API)",
    docsPath: "/channels/acmechat",
    blurb: "demo channel plugin.",
    aliases: ["acme"],
  },
  capabilities: { chatTypes: ["direct"] },
  config: {
    listAccountIds: (cfg) => Object.keys(cfg.channels?.acmechat?.accounts ?? {}),
    resolveAccount: (cfg, accountId) =>
      (cfg.channels?.acmechat?.accounts?.[accountId ?? "default"] ?? { accountId }),
  },
  outbound: {
    deliveryMode: "direct",
    sendText: async () => ({ ok: true }),
  },
};

export default function (api) {
  api.registerChannel({ plugin: myChannel });
}
```

Notas:

* Coloca la configuración bajo `channels.<id>` (no en `plugins.entries`).
* `meta.label` se usa para las etiquetas en las listas de la CLI/UI.
* `meta.aliases` agrega identificadores alternativos para la normalización y las entradas de la CLI.
* `meta.preferOver` enumera los identificadores de canal para omitir la habilitación automática cuando ambos están configurados.
* `meta.detailLabel` y `meta.systemImage` permiten que las UIs muestren etiquetas e íconos de canal más detallados.

<div id="write-a-new-messaging-channel-stepbystep">
  ### Crear un nuevo canal de mensajería (paso a paso)
</div>

Usa esto cuando quieras una **nueva interfaz de chat** (un “canal de mensajería”), no un proveedor de modelos.
La documentación de proveedores de modelos está en `/providers/*`.

1. Elige un id + estructura de configuración

* Toda la configuración del canal vive bajo `channels.<id>`.
* Usa preferentemente `channels.<id>.accounts.<accountId>` para configuraciones con múltiples cuentas.

2. Define los metadatos del canal

* `meta.label`, `meta.selectionLabel`, `meta.docsPath`, `meta.blurb` controlan las listas de la CLI/UI.
* `meta.docsPath` debe apuntar a una página de documentación como `/channels/<id>`.
* `meta.preferOver` permite que un complemento reemplace otro canal (la activación automática lo prioriza).
* `meta.detailLabel` y `meta.systemImage` los usa la UI para textos/iconos de detalle.

3. Implementa los adaptadores requeridos

* `config.listAccountIds` + `config.resolveAccount`
* `capabilities` (tipos de chat, medios, hilos, etc.)
* `outbound.deliveryMode` + `outbound.sendText` (para envío básico)

4. Agrega adaptadores opcionales según sea necesario

* `setup` (asistente), `security` (política de mensajes directos), `status` (estado/salud y diagnósticos)
* `gateway` (inicio/detención/inicio de sesión), `mentions`, `threading`, `streaming`
* `actions` (acciones de mensaje), `commands` (comportamiento de comandos nativos)

5. Registra el canal en tu complemento

* `api.registerChannel({ plugin })`

Ejemplo mínimo de configuración:

```json5
{
  channels: {
    acmechat: {
      accounts: {
        default: { token: "ACME_TOKEN", enabled: true }
      }
    }
  }
}
```

Complemento mínimo de canal (solo de salida):

```ts
const plugin = {
  id: "acmechat",
  meta: {
    id: "acmechat",
    label: "AcmeChat",
    selectionLabel: "AcmeChat (API)",
    docsPath: "/channels/acmechat",
    blurb: "AcmeChat messaging channel.",
    aliases: ["acme"],
  },
  capabilities: { chatTypes: ["direct"] },
  config: {
    listAccountIds: (cfg) => Object.keys(cfg.channels?.acmechat?.accounts ?? {}),
    resolveAccount: (cfg, accountId) =>
      (cfg.channels?.acmechat?.accounts?.[accountId ?? "default"] ?? { accountId }),
  },
  outbound: {
    deliveryMode: "direct",
    sendText: async ({ text }) => {
      // entrega `text` a tu canal aquí
      return { ok: true };
    },
  },
};

export default function (api) {
  api.registerChannel({ plugin });
}
```

Carga el complemento (directorio de extensiones o `plugins.load.paths`), reinicia el Gateway
y luego configura `channels.&lt;id&gt;` en tu configuración.

<div id="agent-tools">
  ### Herramientas de agente
</div>

Consulta la guía específica: [Herramientas de agente de complementos](/es/plugins/agent-tools).

<div id="register-a-gateway-rpc-method">
  ### Registrar un método RPC en el Gateway
</div>

```ts
export default function (api) {
  api.registerGatewayMethod("myplugin.status", ({ respond }) => {
    respond(true, { ok: true });
  });
}
```

<div id="register-cli-commands">
  ### Registrar comandos CLI
</div>

```ts
export default function (api) {
  api.registerCli(({ program }) => {
    program.command("mycmd").action(() => {
      console.log("Hello");
    });
  }, { commands: ["mycmd"] });
}
```

<div id="register-auto-reply-commands">
  ### Registrar comandos de respuesta automática
</div>

Los complementos pueden registrar comandos *slash* personalizados que se ejecutan **sin invocar al
agente de IA**. Esto es útil para comandos de activar/desactivar, comprobaciones de estado o acciones rápidas
que no necesitan procesamiento de un LLM.

```ts
export default function (api) {
  api.registerCommand({
    name: "mystatus",
    description: "Show plugin status",
    handler: (ctx) => ({
      text: `¡El complemento está en ejecución! Canal: ${ctx.channel}`,
    }),
  });
}
```

Contexto del manejador de comandos:

* `senderId`: El ID del remitente (si está disponible)
* `channel`: El canal donde se envió el comando
* `isAuthorizedSender`: Indica si el remitente es un usuario autorizado
* `args`: Argumentos pasados después del comando (si `acceptsArgs: true`)
* `commandBody`: El texto completo del comando
* `config`: La configuración actual de OpenClaw

Opciones de comando:

* `name`: Nombre del comando (sin la `/` inicial)
* `description`: Texto de ayuda que se muestra en las listas de comandos
* `acceptsArgs`: Indica si el comando acepta argumentos (valor predeterminado: false). Si es false y se proporcionan argumentos, el comando no coincidirá y el mensaje se derivará a otros manejadores
* `requireAuth`: Indica si se requiere un remitente autorizado (valor predeterminado: true)
* `handler`: Función que devuelve `{ text: string }` (puede ser async)

Ejemplo con autorización y argumentos:

```ts
api.registerCommand({
  name: "setmode",
  description: "Establecer el modo del complemento",
  acceptsArgs: true,
  requireAuth: true,
  handler: async (ctx) => {
    const mode = ctx.args?.trim() || "default";
    await saveMode(mode);
    return { text: `Mode set to: ${mode}` };
  },
});
```

Notas:

* Los comandos de complementos se procesan **antes** que los comandos integrados y que el Agente de IA
* Los comandos se registran globalmente y funcionan en todos los canales
* Los nombres de comandos no distinguen entre mayúsculas y minúsculas (`/MyStatus` coincide con `/mystatus`)
* Los nombres de comandos deben comenzar con una letra y contener solo letras, números, guiones y guiones bajos
* Los nombres de comandos reservados (como `help`, `status`, `reset`, etc.) no pueden ser sobrescritos por complementos
* El registro duplicado de comandos entre complementos fallará y producirá un error de diagnóstico

<div id="register-background-services">
  ### Registrar servicios en segundo plano
</div>

```ts
export default function (api) {
  api.registerService({
    id: "my-service",
    start: () => api.logger.info("ready"),
    stop: () => api.logger.info("bye"),
  });
}
```

<div id="naming-conventions">
  ## Convenciones de nomenclatura
</div>

* Métodos de Gateway: `pluginId.action` (ejemplo: `voicecall.status`)
* Herramientas: `snake_case` (ejemplo: `voice_call`)
* Comandos de la CLI: kebab o camel, pero evita conflictos con los comandos principales

<div id="skills">
  ## Habilidades
</div>

Los complementos pueden incluir una habilidad en el repositorio (`skills/<name>/SKILL.md`).
Actívala con `plugins.entries.<id>.enabled` (u otros controles de configuración) y asegúrate
de que esté presente en tu espacio de trabajo o en las ubicaciones de habilidades administradas.

<div id="distribution-npm">
  ## Distribución (npm)
</div>

Empaquetado recomendado:

* Paquete principal: `openclaw` (este repositorio)
* Complementos: paquetes npm independientes bajo `@openclaw/*` (por ejemplo, `@openclaw/voice-call`)

Contrato de publicación:

* El `package.json` del complemento debe incluir `openclaw.extensions` con uno o más archivos de entrada.
* Los archivos de entrada pueden ser `.js` o `.ts` (jiti carga TS en tiempo de ejecución).
* `openclaw plugins install <npm-spec>` usa `npm pack`, lo extrae en `~/.openclaw/extensions/<id>/` y lo habilita en la configuración.
* Estabilidad de las claves de configuración: los paquetes con ámbito (scoped) se normalizan a su ID **sin ámbito** (unscoped) para `plugins.entries.*`.

<div id="example-plugin-voice-call">
  ## Complemento de ejemplo: Llamada de voz
</div>

Este repositorio incluye un complemento de llamada de voz (Twilio o registro de logs como alternativa):

* Origen: `extensions/voice-call`
* Habilidad: `skills/voice-call`
* CLI: `openclaw voicecall start|status`
* Herramienta: `voice_call`
* RPC: `voicecall.start`, `voicecall.status`
* Configuración (twilio): `provider: "twilio"` + `twilio.accountSid/authToken/from` (opcional `statusCallbackUrl`, `twimlUrl`)
* Configuración (desarrollo): `provider: "log"` (sin red)

Consulta [Llamada de voz](/es/plugins/voice-call) y `extensions/voice-call/README.md` para la configuración y el uso.

<div id="safety-notes">
  ## Notas sobre seguridad
</div>

Los complementos se ejecutan en el mismo proceso que el Gateway. Trátalos como código confiable:

* Instala solo complementos en los que confíes.
* Da preferencia al uso de la lista de permitidos `plugins.allow`.
* Reinicia el Gateway después de realizar cambios.

<div id="testing-plugins">
  ## Pruebas de complementos
</div>

Los complementos pueden (y deberían) incluir pruebas:

* Los complementos dentro del mismo repositorio pueden mantener pruebas de Vitest en `src/**` (por ejemplo: `src/plugins/voice-call.plugin.test.ts`).
* Los complementos publicados por separado deberían ejecutar su propio CI (lint/build/test) y verificar que `openclaw.extensions` apunte al punto de entrada generado (`dist/index.js`).