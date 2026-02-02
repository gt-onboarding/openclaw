---
title: Msteams
summary: "Estado de compatibilidad, capacidades y configuración del bot de Microsoft Teams"
read_when:
  - Trabajando en las funcionalidades del canal de Microsoft Teams
---

<div id="microsoft-teams-plugin">
  # Microsoft Teams (complemento)
</div>

> &quot;Abandonad toda esperanza, quienes entráis aquí.&quot;

Actualizado: 2026-01-21

Estado: se admiten texto y archivos adjuntos en mensajes directos (DM); el envío de archivos a canales/grupos requiere `sharePointSiteId` y permisos de Graph (consulta [Envío de archivos en chats de grupo](#sending-files-in-group-chats)). Las encuestas se envían mediante Adaptive Cards.

<div id="plugin-required">
  ## Complemento requerido
</div>

Microsoft Teams se distribuye como un complemento y no viene incluido con la instalación básica.

**Cambio incompatible (2026.1.15):** MS Teams se ha separado del núcleo. Si lo usas, debes instalar el complemento.

Motivo: mantiene las instalaciones básicas más ligeras y permite que las dependencias de MS Teams se actualicen de forma independiente.

Instala mediante la CLI (registro de npm):

```bash
openclaw plugins install @openclaw/msteams
```

Copia de trabajo local (cuando se ejecuta desde un repositorio Git):

```bash
openclaw plugins install ./extensions/msteams
```

Si eliges Teams durante la configuración inicial y se detecta un checkout de git,
OpenClaw te sugerirá automáticamente la ruta de instalación local.

Detalles: [Complementos](/es/plugin)

<div id="quick-setup-beginner">
  ## Configuración rápida (para principiantes)
</div>

1. Instala el complemento de Microsoft Teams.
2. Crea un **Azure Bot** (App ID + client secret + tenant ID).
3. Configura OpenClaw con esas credenciales.
4. Expón `/api/messages` (puerto 3978 de forma predeterminada) mediante una URL pública o un túnel.
5. Instala el paquete de aplicación de Teams e inicia el Gateway.

Configuración mínima:

```json5
{
  channels: {
    msteams: {
      enabled: true,
      appId: "<APP_ID>",
      appPassword: "<APP_PASSWORD>",
      tenantId: "<TENANT_ID>",
      webhook: { port: 3978, path: "/api/messages" }
    }
  }
}
```

Nota: los chats de grupo están bloqueados por defecto (`channels.msteams.groupPolicy: "allowlist"`). Para permitir respuestas de grupo, configura `channels.msteams.groupAllowFrom` (o usa `groupPolicy: "open"` para permitir aceptar mensajes de cualquier miembro sin más restricciones que las menciones).

<div id="goals">
  ## Objetivos
</div>

* Hablar con OpenClaw a través de mensajes directos (DMs) de Teams, chats de grupo o canales.
* Mantener un enrutamiento determinístico: las respuestas siempre vuelven al canal por el que llegaron.
* Usar por defecto un comportamiento seguro en los canales (menciones obligatorias, salvo que se configure lo contrario).

<div id="config-writes">
  ## Escrituras de configuración
</div>

Por defecto, Microsoft Teams puede escribir actualizaciones de configuración emitidas mediante `/config set|unset` (requiere `commands.config: true`).

Puedes desactivarlo con:

```json5
{
  channels: { msteams: { configWrites: false } }
}
```

<div id="access-control-dms-groups">
  ## Control de acceso (MD y grupos)
</div>

**Acceso MD**

* Predeterminado: `channels.msteams.dmPolicy = "pairing"`. Los remitentes desconocidos se ignorarán hasta que sean aprobados.
* `channels.msteams.allowFrom` acepta IDs de objeto de AAD, UPN o nombres para mostrar. El asistente resuelve los nombres a IDs mediante Microsoft Graph cuando las credenciales lo permiten.

**Acceso de grupo**

* Predeterminado: `channels.msteams.groupPolicy = "allowlist"` (bloqueado a menos que añadas `groupAllowFrom`). Usa `channels.defaults.groupPolicy` para anular el valor predeterminado cuando no esté establecido.
* `channels.msteams.groupAllowFrom` controla qué remitentes pueden activar en chats/canales de grupo (si no se especifica, recurre a `channels.msteams.allowFrom`).
* Establece `groupPolicy: "open"` para permitir a cualquier miembro (sigue estando limitado por menciones de forma predeterminada; la opción `open` permite aceptar mensajes sin restricciones de cualquier miembro).
* Para no permitir **ningún canal**, establece `channels.msteams.groupPolicy: "disabled"`.

Ejemplo:

```json5
{
  channels: {
    msteams: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["user@org.com"]
    }
  }
}
```

**Teams + lista de permitidos de canales**

* Acota el ámbito de las respuestas de grupo/canal enumerando equipos y canales en `channels.msteams.teams`.
* Las claves pueden ser ID o nombres de equipo; las claves de canal pueden ser ID de conversación o nombres.
* Cuando `groupPolicy="allowlist"` y hay una lista de permitidos de equipos, solo se aceptan los equipos/canales listados (restringidos por mención).
* El asistente de configuración acepta entradas `Team/Channel` y las guarda por ti.
* Al inicio, OpenClaw resuelve los nombres de equipos/canales y de la lista de permitidos de usuarios a ID (cuando los permisos de Graph lo permiten)
  y registra el mapeo; las entradas no resueltas se conservan tal como se escribieron.

Ejemplo:

```json5
{
  channels: {
    msteams: {
      groupPolicy: "allowlist",
      teams: {
        "My Team": {
          channels: {
            "General": { requireMention: true }
          }
        }
      }
    }
  }
}
```

<div id="how-it-works">
  ## Cómo funciona
</div>

1. Instala el complemento de Microsoft Teams.
2. Crea un **Azure Bot** (App ID + secret + tenant ID).
3. Crea un **paquete de aplicación de Teams** que haga referencia al bot e incluya los permisos RSC indicados a continuación.
4. Carga e instala la aplicación de Teams en un equipo (o en el ámbito personal para mensajes directos).
5. Configura `msteams` en `~/.openclaw/openclaw.json` (o mediante variables de entorno) e inicia el Gateway.
6. El Gateway escucha el tráfico de webhooks de Bot Framework en `/api/messages` por defecto.

<div id="azure-bot-setup-prerequisites">
  ## Configuración del bot de Azure (requisitos previos)
</div>

Antes de configurar OpenClaw, debes crear primero un recurso de bot de Azure.

<div id="step-1-create-azure-bot">
  ### Paso 1: Crear Azure Bot
</div>

1. Ve a [Create Azure Bot](https://portal.azure.com/#create/Microsoft.AzureBot)
2. Rellena la pestaña **Basics**:

   | Campo | Valor |
   |-------|-------|
   | **Bot handle** | El nombre de tu bot, p. ej., `openclaw-msteams` (debe ser único) |
   | **Subscription** | Selecciona tu suscripción de Azure |
   | **Resource group** | Crea uno nuevo o usa uno existente |
   | **Pricing tier** | **Free** para desarrollo y pruebas |
   | **Type of App** | **Single Tenant** (recomendado; consulta la nota siguiente) |
   | **Creation type** | **Create new Microsoft App ID** |

> **Aviso de obsolescencia:** La creación de nuevos bots multi-tenant dejó de estar admitida a partir del 31-07-2025. Usa **Single Tenant** para bots nuevos.

3. Haz clic en **Review + create** → **Create** (espera ~1-2 minutos)

<div id="step-2-get-credentials">
  ### Paso 2: Obtener credenciales
</div>

1. Ve a tu recurso de Azure Bot → **Configuration**
2. Copia **Microsoft App ID** → este es tu `appId`
3. Haz clic en **Manage Password** → irás a la App Registration
4. En **Certificates &amp; secrets** → **New client secret** → copia el **Value** → este es tu `appPassword`
5. Ve a **Overview** → copia **Directory (tenant) ID** → este es tu `tenantId`

<div id="step-3-configure-messaging-endpoint">
  ### Paso 3: Configurar el punto de conexión de mensajería
</div>

1. En Azure Bot → **Configuration**
2. Establece **Messaging endpoint** en la URL de tu webhook:
   * Producción: `https://your-domain.com/api/messages`
   * Desarrollo local: Usa un túnel (consulta [Desarrollo local](#local-development-tunneling) más abajo)

<div id="step-4-enable-teams-channel">
  ### Paso 4: Habilitar el canal de Teams
</div>

1. En Azure Bot → **Channels**
2. Haz clic en **Microsoft Teams** → Configure → Save
3. Acepta los Términos de servicio

<div id="local-development-tunneling">
  ## Desarrollo local (túneles)
</div>

Microsoft Teams no puede acceder a `localhost`. Utiliza un túnel para desarrollo local:

**Opción A: ngrok**

```bash
ngrok http 3978
# Copia la URL https, p. ej., https://abc123.ngrok.io
# Configura el endpoint de mensajería como: https://abc123.ngrok.io/api/messages
```

**Opción B: Tailscale Funnel**

```bash
tailscale funnel 3978
# Usa tu URL de funnel de Tailscale como endpoint de mensajería
```

<div id="teams-developer-portal-alternative">
  ## Teams Developer Portal (alternativa)
</div>

En lugar de crear manualmente un archivo ZIP de manifiesto, puedes usar el [Teams Developer Portal](https://dev.teams.microsoft.com/apps):

1. Haz clic en **+ New app**
2. Completa la información básica (nombre, descripción, información del desarrollador)
3. Ve a **App features** → **Bot**
4. Selecciona **Enter a bot ID manually** y pega tu Azure Bot App ID
5. Revisa los ámbitos: **Personal**, **Team**, **Group Chat**
6. Haz clic en **Distribute** → **Download app package**
7. En Teams: **Apps** → **Manage your apps** → **Upload a custom app** → selecciona el ZIP

Esto suele ser más fácil que editar a mano los manifiestos JSON.

<div id="testing-the-bot">
  ## Probando el bot
</div>

**Opción A: Azure Web Chat (verificar primero el webhook)**

1. En Azure Portal → tu recurso de Azure Bot → **Test in Web Chat**
2. Envía un mensaje: deberías ver una respuesta
3. Esto confirma que tu endpoint de webhook funciona antes de configurar Teams

**Opción B: Teams (después de instalar la app)**

1. Instala la app de Teams (sideload o catálogo de la organización)
2. Busca el bot en Teams y envíale un mensaje directo (DM)
3. Revisa los registros del Gateway para ver la actividad entrante

<div id="setup-minimal-text-only">
  ## Configuración mínima (solo texto)
</div>

1. **Instalar el complemento de Microsoft Teams**
   * Desde npm: `openclaw plugins install @openclaw/msteams`
   * Desde un checkout local: `openclaw plugins install ./extensions/msteams`

2. **Registro del bot**
   * Crea un Azure Bot (ver arriba) y anota:
     * App ID
     * Client secret (contraseña de la aplicación)
     * Tenant ID (single-tenant)

3. **Manifiesto de la aplicación de Teams**
   * Incluye una entrada `bot` con `botId = <App ID>`.
   * Ámbitos: `personal`, `team`, `groupChat`.
   * `supportsFiles: true` (requerido para el manejo de archivos en el ámbito personal).
   * Añade permisos RSC (ver abajo).
   * Crea iconos: `outline.png` (32x32) y `color.png` (192x192).
   * Comprime estos tres archivos en un único archivo `.zip`: `manifest.json`, `outline.png`, `color.png`.

4. **Configura OpenClaw**

   ```json
   {
     "msteams": {
       "enabled": true,
       "appId": "<APP_ID>",
       "appPassword": "<APP_PASSWORD>",
       "tenantId": "<TENANT_ID>",
       "webhook": { "port": 3978, "path": "/api/messages" }
     }
   }
   ```

   También puedes usar variables de entorno en lugar de claves de configuración:

   * `MSTEAMS_APP_ID`
   * `MSTEAMS_APP_PASSWORD`
   * `MSTEAMS_TENANT_ID`

5. **Endpoint del bot**
   * Establece el Azure Bot Messaging Endpoint en:
     * `https://<host>:3978/api/messages` (o la ruta/puerto que elijas).

6. **Ejecuta el Gateway**
   * El canal de Teams se inicia automáticamente cuando el complemento está instalado y existe la configuración `msteams` con credenciales.

<div id="history-context">
  ## Contexto del historial
</div>

* `channels.msteams.historyLimit` controla cuántos mensajes recientes de canal/grupo se incluyen en el prompt.
* Si no se establece, recurre a `messages.groupChat.historyLimit`. Establece `0` para desactivar (valor predeterminado: 50).
* El historial de mensajes directos (DM) se puede limitar con `channels.msteams.dmHistoryLimit` (turnos de usuario). Anulaciones por usuario: `channels.msteams.dms["<user_id>"].historyLimit`.

<div id="current-teams-rsc-permissions-manifest">
  ## Permisos RSC actuales de Teams (Manifiesto)
</div>

Estos son los **permisos resourceSpecific existentes** en nuestro manifiesto de la app de Teams. Solo se aplican dentro del equipo o chat donde la app está instalada.

**Para canales (ámbito de equipo):**

* `ChannelMessage.Read.Group` (Application) - recibir todos los mensajes del canal sin necesidad de @mention
* `ChannelMessage.Send.Group` (Application)
* `Member.Read.Group` (Application)
* `Owner.Read.Group` (Application)
* `ChannelSettings.Read.Group` (Application)
* `TeamMember.Read.Group` (Application)
* `TeamSettings.Read.Group` (Application)

**Para chats de grupo:**

* `ChatMessage.Read.Chat` (Application) - recibir todos los mensajes de chats de grupo sin necesidad de @mention

<div id="example-teams-manifest-redacted">
  ## Ejemplo de manifiesto de Teams (con datos ocultos)
</div>

Ejemplo mínimo válido con los campos obligatorios. Reemplaza los ID y las URL.

```json
{
  "$schema": "https://developer.microsoft.com/en-us/json-schemas/teams/v1.23/MicrosoftTeams.schema.json",
  "manifestVersion": "1.23",
  "version": "1.0.0",
  "id": "00000000-0000-0000-0000-000000000000",
  "name": { "short": "OpenClaw" },
  "developer": {
    "name": "Your Org",
    "websiteUrl": "https://example.com",
    "privacyUrl": "https://example.com/privacy",
    "termsOfUseUrl": "https://example.com/terms"
  },
  "description": { "short": "OpenClaw in Teams", "full": "OpenClaw in Teams" },
  "icons": { "outline": "outline.png", "color": "color.png" },
  "accentColor": "#5B6DEF",
  "bots": [
    {
      "botId": "11111111-1111-1111-1111-111111111111",
      "scopes": ["personal", "team", "groupChat"],
      "isNotificationOnly": false,
      "supportsCalling": false,
      "supportsVideo": false,
      "supportsFiles": true
    }
  ],
  "webApplicationInfo": {
    "id": "11111111-1111-1111-1111-111111111111"
  },
  "authorization": {
    "permissions": {
      "resourceSpecific": [
        { "name": "ChannelMessage.Read.Group", "type": "Application" },
        { "name": "ChannelMessage.Send.Group", "type": "Application" },
        { "name": "Member.Read.Group", "type": "Application" },
        { "name": "Owner.Read.Group", "type": "Application" },
        { "name": "ChannelSettings.Read.Group", "type": "Application" },
        { "name": "TeamMember.Read.Group", "type": "Application" },
        { "name": "TeamSettings.Read.Group", "type": "Application" },
        { "name": "ChatMessage.Read.Chat", "type": "Application" }
      ]
    }
  }
}
```

<div id="manifest-caveats-must-have-fields">
  ### Advertencias del manifiesto (campos obligatorios)
</div>

* `bots[].botId` **debe** coincidir con el Azure Bot App ID.
* `webApplicationInfo.id` **debe** coincidir con el Azure Bot App ID.
* `bots[].scopes` debe incluir las superficies en las que planeas usarlo (`personal`, `team`, `groupChat`).
* `bots[].supportsFiles: true` es obligatorio para el manejo de archivos en el ámbito personal.
* `authorization.permissions.resourceSpecific` debe incluir channel read/send si quieres tráfico de canal.

<div id="updating-an-existing-app">
  ### Actualizar una aplicación existente
</div>

Para actualizar una aplicación de Teams ya instalada (por ejemplo, para agregar permisos RSC):

1. Actualiza tu `manifest.json` con la nueva configuración
2. **Incrementa el campo `version`** (por ejemplo, `1.0.0` → `1.1.0`)
3. **Vuelve a comprimir** el manifiesto con los iconos (`manifest.json`, `outline.png`, `color.png`)
4. Sube el nuevo archivo ZIP:
   * **Opción A (Teams Admin Center):** Teams Admin Center → Teams apps → Manage apps → busca tu aplicación → Upload new version
   * **Opción B (Sideload):** En Teams → Apps → Manage your apps → Upload a custom app
5. **Para canales de equipo:** Reinstala la aplicación en cada equipo para que los nuevos permisos surtan efecto
6. **Cierra completamente y vuelve a abrir Teams** (no solo cierres la ventana) para borrar los metadatos de la aplicación en caché

<div id="capabilities-rsc-only-vs-graph">
  ## Capacidades: solo RSC frente a Graph
</div>

<div id="with-teams-rsc-only-app-installed-no-graph-api-permissions">
  ### Solo con **Teams RSC** (aplicación instalada, sin permisos de Graph API)
</div>

Funciona:

* Leer el contenido de **texto** de mensajes de canal.
* Enviar contenido de **texto** en mensajes de canal.
* Recibir archivos adjuntos en **chats personales (DM)**.

No funciona:

* **Contenido de imágenes o archivos** en canales/grupos (la carga útil solo incluye un stub HTML).
* Descargar archivos adjuntos almacenados en SharePoint/OneDrive.
* Leer el historial de mensajes (más allá del evento de webhook activo).

<div id="with-teams-rsc-microsoft-graph-application-permissions">
  ### Con **Teams RSC + permisos de aplicación de Microsoft Graph**
</div>

Permite:

* Descargar contenido hospedado (imágenes pegadas en mensajes).
* Descargar archivos adjuntos almacenados en SharePoint/OneDrive.
* Leer el historial de mensajes de canales/chats a través de Graph.

<div id="rsc-vs-graph-api">
  ### RSC vs Graph API
</div>

| Capacidad | Permisos de RSC | Graph API |
|------------|-----------------|-----------|
| **Mensajes en tiempo real** | Sí (mediante webhook) | No (solo sondeo periódico) |
| **Mensajes históricos** | No | Sí (puede consultar el historial) |
| **Complejidad de configuración** | Solo manifiesto de la app | Requiere consentimiento de administrador + flujo de tokens |
| **Funciona sin conexión** | No (debe estar en ejecución) | Sí (consultar en cualquier momento) |

**Conclusión:** RSC es para escucha en tiempo real; Graph API es para acceso histórico. Para ponerte al día con los mensajes perdidos cuando has estado sin conexión, necesitas Graph API con `ChannelMessage.Read.All` (requiere consentimiento de administrador).

<div id="graph-enabled-media-history-required-for-channels">
  ## Contenido multimedia e historial habilitados por Graph (requerido para canales)
</div>

Si necesitas imágenes/archivos en **canales** o quieres obtener el **historial de mensajes**, debes habilitar los permisos de Microsoft Graph y conceder el consentimiento de administrador.

1. En **App Registration** de Entra ID (Azure AD), agrega los **Application permissions** de Microsoft Graph:
   * `ChannelMessage.Read.All` (archivos adjuntos del canal + historial)
   * `Chat.Read.All` o `ChatMessage.Read.All` (chats de grupo)
2. **Concede el consentimiento de administrador** para el tenant.
3. Incrementa la **versión del manifiesto** de la aplicación de Teams, vuelve a subir el paquete y **reinstala la aplicación en Teams**.
4. **Cierra completamente Teams y vuelve a abrirlo** para borrar los metadatos de la aplicación en caché.

<div id="known-limitations">
  ## Limitaciones conocidas
</div>

<div id="webhook-timeouts">
  ### Tiempos de espera de webhooks
</div>

Teams entrega mensajes mediante un webhook HTTP. Si el procesamiento tarda demasiado (por ejemplo, por respuestas lentas del LLM), es posible que veas:

* Tiempos de espera del Gateway
* Reintentos del mensaje por parte de Teams (lo que puede causar duplicados)
* Respuestas omitidas

OpenClaw gestiona esto respondiendo rápidamente y enviando las respuestas de forma proactiva, pero las respuestas muy lentas aún pueden causar problemas.

<div id="formatting">
  ### Formato
</div>

El markdown de Teams es más limitado que el de Slack o Discord:

* El formato básico funciona: **negrita**, *cursiva*, `code`, enlaces
* El markdown complejo (tablas, listas anidadas) puede no mostrarse correctamente
* Se admiten Adaptive Cards para encuestas y envío de tarjetas arbitrarias (ver más abajo)

<div id="configuration">
  ## Configuración
</div>

Ajustes clave (consulta `/gateway/configuration` para patrones de canal compartidos):

* `channels.msteams.enabled`: habilitar/deshabilitar el canal.
* `channels.msteams.appId`, `channels.msteams.appPassword`, `channels.msteams.tenantId`: credenciales del bot.
* `channels.msteams.webhook.port` (valor predeterminado `3978`)
* `channels.msteams.webhook.path` (valor predeterminado `/api/messages`)
* `channels.msteams.dmPolicy`: `pairing | allowlist | open | disabled` (valor predeterminado: pairing)
* `channels.msteams.allowFrom`: lista de permitidos para mensajes directos (ID de objetos de AAD, UPN o nombres para mostrar). El asistente resuelve nombres a ID durante la configuración cuando el acceso a Graph está disponible.
* `channels.msteams.textChunkLimit`: tamaño de los fragmentos de texto saliente.
* `channels.msteams.chunkMode`: `length` (predeterminado) o `newline` para dividir por líneas en blanco (límites de párrafo) antes de fragmentar por longitud.
* `channels.msteams.mediaAllowHosts`: lista de permitidos para hosts de archivos adjuntos entrantes (valor predeterminado: dominios de Microsoft/Teams).
* `channels.msteams.requireMention`: requiere @mención en canales/grupos (true de forma predeterminada).
* `channels.msteams.replyStyle`: `thread | top-level` (consulta [Estilo de respuesta](#reply-style-threads-vs-posts)).
* `channels.msteams.teams.<teamId>.replyStyle`: anulación por equipo.
* `channels.msteams.teams.<teamId>.requireMention`: anulación por equipo.
* `channels.msteams.teams.<teamId>.tools`: anulaciones predeterminadas por equipo de la política de herramientas (`allow`/`deny`/`alsoAllow`) usadas cuando falta una anulación de canal.
* `channels.msteams.teams.<teamId>.toolsBySender`: anulaciones predeterminadas por equipo y por remitente de la política de herramientas (se admite el comodín `"*"`).
* `channels.msteams.teams.<teamId>.channels.<conversationId>.replyStyle`: anulación por canal.
* `channels.msteams.teams.<teamId>.channels.<conversationId>.requireMention`: anulación por canal.
* `channels.msteams.teams.<teamId>.channels.<conversationId>.tools`: anulaciones por canal de la política de herramientas (`allow`/`deny`/`alsoAllow`).
* `channels.msteams.teams.<teamId>.channels.<conversationId>.toolsBySender`: anulaciones por canal y por remitente de la política de herramientas (se admite el comodín `"*"`).
* `channels.msteams.sharePointSiteId`: ID de sitio de SharePoint para subidas de archivos en chats de grupo/canales (consulta [Enviar archivos en chats de grupo](#sending-files-in-group-chats)).

<div id="routing-sessions">
  ## Enrutamiento y sesiones
</div>

* Las claves de sesión siguen el formato estándar de agente (consulta [/concepts/session](/es/concepts/session)):
  * Los mensajes directos comparten la sesión principal (`agent:<agentId>:<mainKey>`).
  * Los mensajes de canal/grupo usan el ID de conversación:
    * `agent:<agentId>:msteams:channel:<conversationId>`
    * `agent:<agentId>:msteams:group:<conversationId>`

<div id="reply-style-threads-vs-posts">
  ## Estilo de respuesta: hilos vs publicaciones
</div>

Teams introdujo recientemente dos estilos de UI de canal sobre el mismo modelo de datos subyacente:

| Estilo                   | Descripción                                                | `replyStyle` recomendado  |
| ------------------------ | ---------------------------------------------------------- | ------------------------- |
| **Posts** (clásico)      | Los mensajes aparecen como tarjetas con respuestas en hilo | `thread` (predeterminado) |
| **Threads** (tipo Slack) | Los mensajes fluyen de forma lineal, similar a Slack       | `top-level`               |

**El problema:** La API de Teams no expone qué estilo de UI usa un canal. Si usas el `replyStyle` incorrecto:

* `thread` en un canal con estilo Threads → las respuestas aparecen anidadas de forma incómoda
* `top-level` en un canal con estilo Posts → las respuestas aparecen como publicaciones independientes de nivel superior en lugar de dentro del hilo

**Solución:** Configura `replyStyle` para cada canal en función de cómo esté configurado el canal:

```json
{
  "msteams": {
    "replyStyle": "thread",
    "teams": {
      "19:abc...@thread.tacv2": {
        "channels": {
          "19:xyz...@thread.tacv2": {
            "replyStyle": "top-level"
          }
        }
      }
    }
  }
}
```

<div id="attachments-images">
  ## Archivos adjuntos e imágenes
</div>

**Limitaciones actuales:**

* **DMs:** Las imágenes y los archivos adjuntos funcionan mediante las APIs de archivos del bot de Teams.
* **Canales/grupos:** Los archivos adjuntos residen en el almacenamiento de M365 (SharePoint/OneDrive). La carga útil del webhook solo incluye un fragmento HTML, no los bytes reales del archivo. **Se requieren permisos de Graph API** para descargar archivos adjuntos de canales.

Sin permisos de Graph, los mensajes en canales con imágenes se recibirán solo como texto (el contenido de la imagen no es accesible para el bot).
De forma predeterminada, OpenClaw solo descarga contenido multimedia desde nombres de host de Microsoft/Teams. Puedes anular este comportamiento con `channels.msteams.mediaAllowHosts` (usa `["*"]` para permitir cualquier host).

<div id="sending-files-in-group-chats">
  ## Envío de archivos en chats de grupo
</div>

Los bots pueden enviar archivos en mensajes directos (DMs) usando el flujo FileConsentCard (integrado). Sin embargo, **enviar archivos en chats de grupo/canales** requiere configuración adicional:

| Contexto | Cómo se envían los archivos | Configuración necesaria |
|---------|-----------------------------|-------------------------|
| **DMs** | FileConsentCard → el usuario acepta → el bot carga el archivo | Funciona sin configuración adicional |
| **Chats de grupo/canales** | Cargar en SharePoint → compartir enlace | Requiere `sharePointSiteId` + permisos de Graph |
| **Imágenes (cualquier contexto)** | En línea, codificadas en Base64 | Funciona sin configuración adicional |

<div id="why-group-chats-need-sharepoint">
  ### Por qué los chats de grupo necesitan SharePoint
</div>

Los bots no disponen de OneDrive personal (el endpoint `/me/drive` de Graph API no funciona con identidades de aplicación). Para enviar archivos en chats de grupo o canales, el bot los carga en un **sitio de SharePoint** y crea un vínculo para compartir.

<div id="setup">
  ### Configuración
</div>

1. **Añade permisos de Graph API** en Entra ID (Azure AD) → App Registration:
   * `Sites.ReadWrite.All` (Application): subir archivos a SharePoint
   * `Chat.Read.All` (Application): opcional, habilita enlaces para compartir por usuario

2. **Concede el consentimiento de administrador** para el inquilino (tenant).

3. **Obtén el ID de tu sitio de SharePoint:**
   ```bash
   # Mediante Graph Explorer o curl con un token válido:
   curl -H "Authorization: Bearer $TOKEN" \
     "https://graph.microsoft.com/v1.0/sites/{hostname}:/{site-path}"

   # Ejemplo: para un sitio en "contoso.sharepoint.com/sites/BotFiles"
   curl -H "Authorization: Bearer $TOKEN" \
     "https://graph.microsoft.com/v1.0/sites/contoso.sharepoint.com:/sites/BotFiles"

   # La respuesta incluye: "id": "contoso.sharepoint.com,guid1,guid2"
   ```

4. **Configura OpenClaw:**
   ```json5
   {
     channels: {
       msteams: {
         // ... otra configuración ...
         sharePointSiteId: "contoso.sharepoint.com,guid1,guid2"
       }
     }
   }
   ```

<div id="sharing-behavior">
  ### Comportamiento de uso compartido
</div>

| Permission | Sharing behavior |
|------------|------------------|
| `Sites.ReadWrite.All` only | Enlace de uso compartido a nivel de organización (cualquier persona de la organización puede acceder) |
| `Sites.ReadWrite.All` + `Chat.Read.All` | Enlace de uso compartido por usuario (solo los miembros del chat pueden acceder) |

El uso compartido por usuario es más seguro, ya que solo los participantes del chat pueden acceder al archivo. Si falta el permiso `Chat.Read.All`, el bot recurre al uso compartido a nivel de organización.

<div id="fallback-behavior">
  ### Comportamiento de respaldo
</div>

| Escenario | Resultado |
|----------|--------|
| Chat de grupo + archivo + `sharePointSiteId` configurado | Subir a SharePoint, enviar enlace para compartir |
| Chat de grupo + archivo + sin `sharePointSiteId` | Intentar subir a OneDrive (puede fallar), enviar solo texto |
| Chat personal + archivo | Flujo de FileConsentCard (funciona sin SharePoint) |
| Cualquier contexto + imagen | Codificada en Base64 incrustada (funciona sin SharePoint) |

<div id="files-stored-location">
  ### Ubicación donde se almacenan los archivos
</div>

Los archivos subidos se almacenan en una carpeta `/OpenClawShared/` en la biblioteca de documentos predeterminada del sitio de SharePoint que hayas configurado.

<div id="polls-adaptive-cards">
  ## Encuestas (Adaptive Cards)
</div>

OpenClaw envía encuestas de Teams como Adaptive Cards (no existe una API nativa de encuestas de Teams).

* CLI: `openclaw message poll --channel msteams --target conversation:<id> ...`
* El Gateway registra los votos en `~/.openclaw/msteams-polls.json`.
* El Gateway debe permanecer en línea para registrar los votos.
* Las encuestas aún no publican automáticamente resúmenes de resultados (consulta el archivo de almacenamiento si es necesario).

<div id="adaptive-cards-arbitrary">
  ## Adaptive Cards (arbitrarias)
</div>

Envía cualquier JSON de Adaptive Card a usuarios o conversaciones de Teams usando la herramienta `message` o la CLI.

El parámetro `card` acepta un objeto JSON de Adaptive Card. Cuando se incluye `card`, el texto del mensaje es opcional.

**Herramienta de Agente:**

```json
{
  "action": "send",
  "channel": "msteams",
  "target": "user:<id>",
  "card": {
    "type": "AdaptiveCard",
    "version": "1.5",
    "body": [{"type": "TextBlock", "text": "Hello!"}]
  }
}
```

**CLI:**

```bash
openclaw message send --channel msteams \
  --target "conversation:19:abc...@thread.tacv2" \
  --card '{"type":"AdaptiveCard","version":"1.5","body":[{"type":"TextBlock","text":"Hello!"}]}'
```

Consulta la [documentación de Adaptive Cards](https://adaptivecards.io/) para obtener el esquema de tarjetas y ejemplos. Para más detalles sobre los formatos de destino, consulta la sección [Formatos de destino](#target-formats) a continuación.

<div id="target-formats">
  ## Formatos de destino
</div>

Los destinos de MSTeams utilizan prefijos para distinguir usuarios de conversaciones:

| Tipo de destino            | Formato                                | Ejemplo                                             |
| -------------------------- | -------------------------------------- | --------------------------------------------------- |
| Usuario (por ID)           | `user:&lt;aad-object-id&gt;`           | `user:40a1a0ed-4ff2-4164-a219-55518990c197`         |
| Usuario (por nombre)       | `user:&lt;display-name&gt;`            | `user:John Smith` (requiere la Graph API)           |
| Grupo/canal                | `conversation:&lt;conversation-id&gt;` | `conversation:19:abc123...@thread.tacv2`            |
| Grupo/canal (sin procesar) | `&lt;conversation-id&gt;`              | `19:abc123...@thread.tacv2` (si contiene `@thread`) |

**Ejemplos de CLI:**

```bash
# Send to a user by ID
openclaw message send --channel msteams --target "user:40a1a0ed-..." --message "Hello"

# Enviar a un usuario por nombre para mostrar (activa la búsqueda en Graph API)
openclaw message send --channel msteams --target "user:John Smith" --message "Hello"

# Send to a group chat or channel
openclaw message send --channel msteams --target "conversation:19:abc...@thread.tacv2" --message "Hello"

# Send an Adaptive Card to a conversation
openclaw message send --channel msteams --target "conversation:19:abc...@thread.tacv2" \
  --card '{"type":"AdaptiveCard","version":"1.5","body":[{"type":"TextBlock","text":"Hello"}]}'
```

**Ejemplos de herramientas de agentes:**

```json
{
  "action": "send",
  "channel": "msteams",
  "target": "user:John Smith",
  "message": "Hello!"
}
```

```json
{
  "action": "send",
  "channel": "msteams",
  "target": "conversation:19:abc...@thread.tacv2",
  "card": {"type": "AdaptiveCard", "version": "1.5", "body": [{"type": "TextBlock", "text": "Hello"}]}
}
```

Nota: Sin el prefijo `user:`, los nombres se interpretan por defecto como de grupo o equipo. Utiliza siempre `user:` cuando te dirijas a personas por su nombre para mostrar.

<div id="proactive-messaging">
  ## Mensajería proactiva
</div>

* Los mensajes proactivos solo se pueden enviar **después de que** el usuario haya interactuado, porque es entonces cuando almacenamos las referencias de la conversación.
* Consulta `/gateway/configuration` para `dmPolicy` y la restricción mediante lista de permitidos.

<div id="team-and-channel-ids-common-gotcha">
  ## IDs de equipo y canal (Trampa común)
</div>

El parámetro de consulta `groupId` en las URLs de Teams **NO** es el ID de equipo que se usa para la configuración. Extrae los IDs de la ruta de la URL en su lugar:

**URL del equipo:**

```
https://teams.microsoft.com/l/team/19%3ABk4j...%40thread.tacv2/conversations?groupId=...
                                    └────────────────────────────┘
                                    Team ID (decodificar URL)
```

**URL del canal:**

```
https://teams.microsoft.com/l/channel/19%3A15bc...%40thread.tacv2/ChannelName?groupId=...
                                      └─────────────────────────┘
                                      Channel ID (URL-decode this)
```

**Para la configuración:**

* Team ID = segmento de la ruta después de `/team/` (descodificado de URL, por ejemplo, `19:Bk4j...@thread.tacv2`)
* Channel ID = segmento de la ruta después de `/channel/` (descodificado de URL)
* **Omite** el parámetro de consulta `groupId`

<div id="private-channels">
  ## Canales privados
</div>

Los bots tienen compatibilidad limitada en canales privados:

| Función | Canales estándar | Canales privados |
|---------|-------------------|------------------|
| Instalación del bot | Sí | Limitada |
| Mensajes en tiempo real (webhook) | Sí | Puede que no funcionen |
| Permisos RSC | Sí | Puede comportarse de manera diferente |
| @menciones | Sí | Si el bot es accesible |
| Historial de Graph API | Sí | Sí (con permisos) |

**Alternativas si los canales privados no funcionan:**

1. Usa canales estándar para las interacciones con el bot
2. Usa mensajes directos (DM): los usuarios siempre pueden escribirle al bot directamente
3. Usa Graph API para acceso al historial (requiere `ChannelMessage.Read.All`)

<div id="troubleshooting">
  ## Solución de problemas
</div>

<div id="common-issues">
  ### Problemas comunes
</div>

* **Imágenes que no se muestran en los canales:** faltan permisos de Graph o consentimiento de administrador. Reinstala la aplicación de Teams y cierra Teams por completo y vuelve a abrirlo.
* **Sin respuestas en el canal:** las menciones son obligatorias de forma predeterminada; establece `channels.msteams.requireMention=false` o configúralo por equipo/canal.
* **Incompatibilidad de versiones (Teams sigue mostrando el manifiesto antiguo):** quita y vuelve a agregar la aplicación y cierra Teams por completo para que se actualice.
* **401 Unauthorized desde el webhook:** es el comportamiento esperado al probar manualmente sin Azure JWT; significa que el endpoint es accesible pero la autenticación falló. Usa Azure Web Chat para hacer pruebas correctamente.

<div id="manifest-upload-errors">
  ### Errores al cargar el manifiesto
</div>

* **&quot;Icon file cannot be empty&quot;:** El manifiesto hace referencia a archivos de icono de 0 bytes. Crea iconos PNG válidos (32x32 para `outline.png`, 192x192 para `color.png`).
* **&quot;webApplicationInfo.Id already in use&quot;:** La aplicación sigue instalada en otro equipo/chat. Localízala y desinstálala primero, o espera 5‑10 minutos para que se propague el cambio.
* **&quot;Something went wrong&quot; on upload:** En su lugar, cárgala mediante https://admin.teams.microsoft.com, abre las herramientas de desarrollo (DevTools) del navegador (F12) → pestaña Network (Red) y revisa el cuerpo de la respuesta para ver el error real.
* **Sideload failing:** Prueba &quot;Upload an app to your org&#39;s app catalog&quot; en lugar de &quot;Upload a custom app&quot;; a menudo esto evita las restricciones de sideload.

<div id="rsc-permissions-not-working">
  ### Los permisos RSC no funcionan
</div>

1. Verifica que `webApplicationInfo.id` coincida exactamente con el App ID de tu bot
2. Vuelve a subir la aplicación y reinstálala en el equipo/chat
3. Comprueba si el administrador de tu organización ha bloqueado los permisos RSC
4. Confirma que estás usando el ámbito correcto: `ChannelMessage.Read.Group` para equipos, `ChatMessage.Read.Chat` para chats de grupo

<div id="references">
  ## Referencias
</div>

* [Create Azure Bot](https://learn.microsoft.com/en-us/azure/bot-service/bot-service-quickstart-registration) - guía de configuración de Azure Bot
* [Teams Developer Portal](https://dev.teams.microsoft.com/apps) - crear y administrar aplicaciones de Teams
* [Teams app manifest schema](https://learn.microsoft.com/en-us/microsoftteams/platform/resources/schema/manifest-schema)
* [Receive channel messages with RSC](https://learn.microsoft.com/en-us/microsoftteams/platform/bots/how-to/conversations/channel-messages-with-rsc)
* [RSC permissions reference](https://learn.microsoft.com/en-us/microsoftteams/platform/graph-api/rsc/resource-specific-consent)
* [Teams bot file handling](https://learn.microsoft.com/en-us/microsoftteams/platform/bots/how-to/bots-filesv4) (los canales y los grupos requieren Graph)
* [Proactive messaging](https://learn.microsoft.com/en-us/microsoftteams/platform/bots/how-to/conversations/send-proactive-messages)