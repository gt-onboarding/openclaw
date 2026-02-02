---
title: Googlechat
summary: "Estado de soporte, capacidades y configuración de la aplicación Google Chat"
read_when:
  - Trabajando en funciones del canal de Google Chat
---

<div id="google-chat-chat-api">
  # Google Chat (Chat API)
</div>

Estado: compatible con DMs y espacios mediante webhooks de la API de Google Chat (solo HTTP).

<div id="quick-setup-beginner">
  ## Configuración rápida (principiante)
</div>

1. Crea un proyecto de Google Cloud y habilita la **Google Chat API**.
   * Ve a: [Google Chat API Credentials](https://console.cloud.google.com/apis/api/chat.googleapis.com/credentials)
   * Habilita la API si aún no está habilitada.
2. Crea una **Service Account**:
   * Pulsa **Create Credentials** &gt; **Service Account**.
   * Ponle el nombre que quieras (p. ej., `openclaw-chat`).
   * Deja los permisos en blanco (pulsa **Continue**).
   * Deja los principals con acceso en blanco (pulsa **Done**).
3. Crea y descarga la **JSON Key**:
   * En la lista de service accounts, haz clic en la que acabas de crear.
   * Ve a la pestaña **Keys**.
   * Haz clic en **Add Key** &gt; **Create new key**.
   * Selecciona **JSON** y pulsa **Create**.
4. Guarda el archivo JSON descargado en el host de tu Gateway (p. ej., `~/.openclaw/googlechat-service-account.json`).
5. Crea una aplicación de Google Chat en la [Google Cloud Console Chat Configuration](https://console.cloud.google.com/apis/api/chat.googleapis.com/hangouts-chat):
   * Rellena la **Application info**:
     * **App name**: (p. ej. `OpenClaw`)
     * **Avatar URL**: (p. ej. `https://openclaw.ai/logo.png`)
     * **Description**: (p. ej. `Personal AI Assistant`)
   * Habilita **Interactive features**.
   * En **Functionality**, marca **Join spaces and group conversations**.
   * En **Connection settings**, selecciona **HTTP endpoint URL**.
   * En **Triggers**, selecciona **Use a common HTTP endpoint URL for all triggers** y configúralo con la URL pública de tu Gateway seguida de `/googlechat`.
     * *Consejo: Ejecuta `openclaw status` para encontrar la URL pública de tu Gateway.*
   * En **Visibility**, marca **Make this Chat app available to specific people and groups in &lt;Your Domain&gt;**.
   * Introduce tu dirección de correo electrónico (p. ej. `user@example.com`) en el cuadro de texto.
   * Haz clic en **Save** al final de la página.
6. **Habilita el estado de la app**:
   * Después de guardar, **recarga la página**.
   * Busca la sección **App status** (normalmente cerca de la parte superior o inferior después de guardar).
   * Cambia el estado a **Live - available to users**.
   * Haz clic en **Save** de nuevo.
7. Configura OpenClaw con la ruta de la service account + webhook audience:
   * Variable de entorno: `GOOGLE_CHAT_SERVICE_ACCOUNT_FILE=/path/to/service-account.json`
   * O configuración: `channels.googlechat.serviceAccountFile: "/path/to/service-account.json"`.
8. Configura el tipo y el valor de webhook audience (debe coincidir con la configuración de tu app de Chat).
9. Inicia el Gateway. Google Chat realizará solicitudes POST a tu ruta de webhook.

<div id="add-to-google-chat">
  ## Agregar a Google Chat
</div>

Una vez que el Gateway se esté ejecutando y tu correo electrónico se haya agregado a la lista de visibilidad:

1. Ve a [Google Chat](https://chat.google.com/).
2. Haz clic en el ícono **+** (más) junto a **Direct Messages**.
3. En la barra de búsqueda (donde normalmente agregas personas), escribe el **App name** que configuraste en Google Cloud Console.
   * **Nota**: El bot *no* aparecerá en la lista de exploración de &quot;Marketplace&quot; porque es una aplicación privada. Debes buscarlo por nombre.
4. Selecciona tu bot en los resultados.
5. Haz clic en **Add** o **Chat** para iniciar una conversación 1:1 (uno a uno).
6. Envía «Hello» para activar el asistente.

<div id="public-url-webhook-only">
  ## URL pública (solo Webhook)
</div>

Los webhooks de Google Chat requieren un endpoint HTTPS público. Por seguridad, **expón únicamente la ruta `/googlechat`** a Internet. Mantén el panel de control de OpenClaw y otros endpoints sensibles en tu red privada.

<div id="option-a-tailscale-funnel-recommended">
  ### Opción A: Tailscale Funnel (recomendada)
</div>

Usa Tailscale Serve para el panel privado y Funnel para la ruta pública del webhook. Esto mantiene `/` privado mientras expone solo `/googlechat`.

1. **Comprueba a qué dirección está enlazado tu Gateway:**
   ```bash
   ss -tlnp | grep 18789
   ```
   Anota la dirección IP (por ejemplo, `127.0.0.1`, `0.0.0.0` o tu IP de Tailscale como `100.x.x.x`).

2. **Expón el panel solo a la tailnet (puerto 8443):**
   ```bash
   # Si está enlazado a localhost (127.0.0.1 o 0.0.0.0):
   tailscale serve --bg --https 8443 http://127.0.0.1:18789

   # Si está enlazado solo a la IP de Tailscale (por ejemplo, 100.106.161.80):
   tailscale serve --bg --https 8443 http://100.106.161.80:18789
   ```

3. **Expón públicamente solo la ruta del webhook:**
   ```bash
   # Si está enlazado a localhost (127.0.0.1 o 0.0.0.0):
   tailscale funnel --bg --set-path /googlechat http://127.0.0.1:18789/googlechat

   # Si está enlazado solo a la IP de Tailscale (por ejemplo, 100.106.161.80):
   tailscale funnel --bg --set-path /googlechat http://100.106.161.80:18789/googlechat
   ```

4. **Autoriza el nodo para acceso con Funnel:**
   Si se te solicita, visita la URL de autorización que aparece en la salida para habilitar Funnel para este nodo en la política de tu tailnet.

5. **Verifica la configuración:**
   ```bash
   tailscale serve status
   tailscale funnel status
   ```

Tu URL pública de webhook será:
`https://<node-name>.<tailnet>.ts.net/googlechat`

Tu panel privado permanece limitado a la tailnet:
`https://<node-name>.<tailnet>.ts.net:8443/`

Usa la URL pública (sin `:8443`) en la configuración de la app de Google Chat.

> Nota: Esta configuración persiste tras los reinicios. Para eliminarla más adelante, ejecuta `tailscale funnel reset` y `tailscale serve reset`.

<div id="option-b-reverse-proxy-caddy">
  ### Opción B: Proxy inverso (Caddy)
</div>

Si usas un proxy inverso como Caddy, configura el proxy solo para la ruta específica:

```caddy
your-domain.com {
    reverse_proxy /googlechat* localhost:18789
}
```

Con esta configuración, cualquier petición a `your-domain.com/` se ignorará o devolverá un 404, mientras que `your-domain.com/googlechat` se enruta de forma segura a OpenClaw.

<div id="option-c-cloudflare-tunnel">
  ### Opción C: Cloudflare Tunnel
</div>

Configura las reglas de entrada (ingress) de tu túnel para que solo enruten la ruta del webhook:

* **Ruta**: `/googlechat` -&gt; `http://localhost:18789/googlechat`
* **Regla predeterminada**: HTTP 404 (No encontrado)

<div id="how-it-works">
  ## Cómo funciona
</div>

1. Google Chat envía solicitudes POST de webhook al Gateway. Cada solicitud incluye un encabezado `Authorization: Bearer <token>`.
2. OpenClaw verifica el token contra los valores configurados de `audienceType` + `audience`:
   * `audienceType: "app-url"` → el audience es tu URL de webhook HTTPS.
   * `audienceType: "project-number"` → el audience es el número de proyecto de Cloud.
3. Los mensajes se enrutan por espacio:
   * Los DM usan la clave de sesión `agent:<agentId>:googlechat:dm:<spaceId>`.
   * Los espacios usan la clave de sesión `agent:<agentId>:googlechat:group:<spaceId>`.
4. El acceso por DM usa emparejamiento de forma predeterminada. Los remitentes desconocidos reciben un código de emparejamiento; apruébalo con:
   * `openclaw pairing approve googlechat <code>`
5. Los espacios de grupo requieren una mención con @ de forma predeterminada. Usa `botUser` si la detección de menciones necesita el nombre de usuario de la aplicación.

<div id="targets">
  ## Destinos
</div>

Usa estos identificadores para el envío y la lista de permitidos:

* Mensajes directos: `users/<userId>` o `users/<email>` (se aceptan direcciones de correo electrónico).
* Espacios: `spaces/<spaceId>`.

<div id="config-highlights">
  ## Aspectos clave de la configuración
</div>

```json5
{
  channels: {
    "googlechat": {
      enabled: true,
      serviceAccountFile: "/path/to/service-account.json",
      audienceType: "app-url",
      audience: "https://gateway.example.com/googlechat",
      webhookPath: "/googlechat",
      botUser: "users/1234567890", // opcional; ayuda con la detección de menciones
      dm: {
        policy: "pairing",
        allowFrom: ["users/1234567890", "name@example.com"]
      },
      groupPolicy: "allowlist",
      groups: {
        "spaces/AAAA": {
          allow: true,
          requireMention: true,
          users: ["users/1234567890"],
          systemPrompt: "Short answers only."
        }
      },
      actions: { reactions: true },
      typingIndicator: "message",
      mediaMaxMb: 20
    }
  }
}
```

Notas:

* Las credenciales de la cuenta de servicio también se pueden pasar directamente mediante `serviceAccount` (cadena JSON).
* La ruta predeterminada del webhook es `/googlechat` si `webhookPath` no está configurado.
* Las reacciones están disponibles a través de la herramienta `reactions` y de `channels action` cuando `actions.reactions` está habilitado.
* `typingIndicator` admite `none`, `message` (valor predeterminado) y `reaction` (las reacciones requieren OAuth del usuario).
* Los archivos adjuntos se descargan mediante la Chat API y se almacenan en el pipeline de medios (tamaño limitado por `mediaMaxMb`).

<div id="troubleshooting">
  ## Solución de problemas
</div>

<div id="405-method-not-allowed">
  ### 405 Method Not Allowed
</div>

Si en Google Cloud Logs Explorer aparecen errores como:

```
status code: 405, reason phrase: HTTP error response: HTTP/1.1 405 Method Not Allowed
```

Esto significa que el controlador de webhook no está registrado. Causas frecuentes:

1. **Canal no configurado**: Falta la sección `channels.googlechat` en tu configuración. Verifícalo con:
   ```bash
   openclaw config get channels.googlechat
   ```
   Si devuelve «Config path not found», añade la configuración (consulta [Aspectos destacados de la configuración](#config-highlights)).

2. **Complemento no habilitado**: Comprueba el estado del complemento:
   ```bash
   openclaw plugins list | grep googlechat
   ```
   Si muestra «disabled», añade `plugins.entries.googlechat.enabled: true` a tu configuración.

3. **Gateway no reiniciado**: Después de añadir la configuración, reinicia el Gateway:
   ```bash
   openclaw gateway restart
   ```

Verifica que el canal esté en ejecución:

```bash
openclaw channels status
# Debería mostrar: Google Chat default: enabled, configured, ...
```

<div id="other-issues">
  ### Otros problemas
</div>

* Comprueba `openclaw channels status --probe` para ver errores de autenticación o una configuración de `audience` que falte.
* Si no llegan mensajes, confirma la URL del webhook de la app de Chat y las suscripciones de eventos.
* Si el filtrado por menciones bloquea las respuestas, ajusta `botUser` al nombre de recurso de usuario de la app y comprueba `requireMention`.
* Usa `openclaw logs --follow` mientras envías un mensaje de prueba para ver si las solicitudes llegan al Gateway.

Documentación relacionada:

* [Configuración del Gateway](/es/gateway/configuration)
* [Seguridad](/es/gateway/security)
* [Reacciones](/es/tools/reactions)