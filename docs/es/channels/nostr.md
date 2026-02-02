---
title: Nostr
summary: "Canal de mensajes directos (DM) de Nostr mediante mensajes cifrados NIP-04"
read_when:
  - Quieres que OpenClaw reciba mensajes directos (DM) vía Nostr
  - Estás configurando mensajería descentralizada
---

<div id="nostr">
  # Nostr
</div>

**Estado:** Complemento opcional (desactivado de forma predeterminada).

Nostr es un protocolo descentralizado de redes sociales. Este canal permite que OpenClaw reciba y responda a mensajes directos (DMs) cifrados mediante NIP-04.

<div id="install-on-demand">
  ## Instalación (a demanda)
</div>

<div id="onboarding-recommended">
  ### Incorporación (recomendado)
</div>

- El asistente de incorporación (`openclaw onboard`) y el comando `openclaw channels add` muestran complementos de canal opcionales.
- Al seleccionar Nostr se te pedirá instalar el complemento bajo demanda.

Valores predeterminados de instalación:

- **Canal de desarrollo + git checkout disponible:** usa la ruta local del complemento.
- **Stable/Beta:** descarga desde npm.

Siempre puedes cambiar la opción en el aviso interactivo.

<div id="manual-install">
  ### Instalación manual
</div>

```bash
openclaw plugins install @openclaw/nostr
```

Usa un clon local del repositorio (flujos de trabajo de desarrollo):

```bash
openclaw plugins install --link <path-to-openclaw>/extensions/nostr
```

Reinicia el Gateway después de instalar o activar complementos.


<div id="quick-setup">
  ## Configuración rápida
</div>

1. Genera un par de claves de Nostr (si es necesario):

```bash
# Usando nak
nak key generate
```

2. Añade lo siguiente a la configuración:

```json
{
  "channels": {
    "nostr": {
      "privateKey": "${NOSTR_PRIVATE_KEY}"
    }
  }
}
```

3. Exporta la clave:

```bash
export NOSTR_PRIVATE_KEY="nsec1..."
```

4. Reinicia el Gateway.


<div id="configuration-reference">
  ## Referencia de configuración
</div>

| Clave | Tipo | Predeterminado | Descripción |
| --- | --- | --- | --- |
| `privateKey` | string | obligatorio | Clave privada en formato `nsec` o hex |
| `relays` | string[] | `['wss://relay.damus.io', 'wss://nos.lol']` | URLs de relays (WebSocket) |
| `dmPolicy` | string | `pairing` | Política de acceso a mensajes directos (DM) |
| `allowFrom` | string[] | `[]` | Claves públicas de remitentes permitidos |
| `enabled` | boolean | `true` | Habilitar o deshabilitar el canal |
| `name` | string | - | Nombre para mostrar |
| `profile` | object | - | Metadatos de perfil NIP-01 |

<div id="profile-metadata">
  ## Metadatos del perfil
</div>

Los datos del perfil se publican como un evento NIP-01 de `kind:0`. Puedes gestionarlos desde el Control UI (Channels -&gt; Nostr -&gt; Profile) o configurarlos directamente en la configuración.

Ejemplo:

```json
{
  "channels": {
    "nostr": {
      "privateKey": "${NOSTR_PRIVATE_KEY}",
      "profile": {
        "name": "openclaw",
        "displayName": "OpenClaw",
        "about": "Bot de asistente personal por MD",
        "picture": "https://example.com/avatar.png",
        "banner": "https://example.com/banner.png",
        "website": "https://example.com",
        "nip05": "openclaw@example.com",
        "lud16": "openclaw@example.com"
      }
    }
  }
}
```

Notas:

* Las URL de perfil deben usar `https://`.
* Importar desde relés combina los campos y mantiene las personalizaciones locales.


<div id="access-control">
  ## Control de acceso
</div>

<div id="dm-policies">
  ### Políticas de MD
</div>

- **emparejamiento** (predeterminado): los remitentes desconocidos reciben un código de emparejamiento.
- **lista de permitidos**: solo las claves públicas en `allowFrom` pueden enviar MD.
- **open**: MD públicos entrantes (la configuración `open` permite aceptar mensajes sin restricciones de cualquier usuario; requiere `allowFrom: ["*"]`).
- **desactivado**: se ignoran los MD entrantes.

<div id="allowlist-example">
  ### Ejemplo de lista de permitidos
</div>

```json
{
  "channels": {
    "nostr": {
      "privateKey": "${NOSTR_PRIVATE_KEY}",
      "dmPolicy": "allowlist",
      "allowFrom": ["npub1abc...", "npub1xyz..."]
    }
  }
}
```


<div id="key-formats">
  ## Formatos de claves
</div>

Formatos aceptados:

- **Clave privada:** `nsec...` o cadena hexadecimal de 64 caracteres
- **Claves públicas (`allowFrom`):** `npub...` o valor hexadecimal

<div id="relays">
  ## Relays
</div>

Valores predeterminados: `relay.damus.io` y `nos.lol`.

```json
{
  "channels": {
    "nostr": {
      "privateKey": "${NOSTR_PRIVATE_KEY}",
      "relays": [
        "wss://relay.damus.io",
        "wss://relay.primal.net",
        "wss://nostr.wine"
      ]
    }
  }
}
```

Consejos:

* Usa 2-3 relays para tener redundancia.
* Evita usar demasiados relays (latencia, duplicación).
* Los relays de pago pueden mejorar la fiabilidad.
* Los relays locales están bien para hacer pruebas (`ws://localhost:7777`).


<div id="protocol-support">
  ## Compatibilidad con el protocolo
</div>

| NIP | Estado | Descripción |
| --- | --- | --- |
| NIP-01 | Con soporte | Formato básico de eventos + metadatos de perfil |
| NIP-04 | Con soporte | MD cifrados (`kind:4`) |
| NIP-17 | Planificado | MD envueltos como regalo |
| NIP-44 | Planificado | Cifrado con versiones |

<div id="testing">
  ## Pruebas
</div>

<div id="local-relay">
  ### Relay local
</div>

```bash
# Iniciar strfry
docker run -p 7777:7777 ghcr.io/hoytech/strfry
```

```json
{
  "channels": {
    "nostr": {
      "privateKey": "${NOSTR_PRIVATE_KEY}",
      "relays": ["ws://localhost:7777"]
    }
  }
}
```


<div id="manual-test">
  ### Prueba manual
</div>

1) Anota la pubkey (npub) del bot en los logs.
2) Abre un cliente Nostr (Damus, Amethyst, etc.).
3) Envía un mensaje directo (DM) a la pubkey del bot.
4) Verifica la respuesta.

<div id="troubleshooting">
  ## Solución de problemas
</div>

<div id="not-receiving-messages">
  ### No se reciben mensajes
</div>

- Verifica que la clave privada sea válida.
- Asegúrate de que las URL de los relays sean accesibles y utilicen `wss://` (o `ws://` para local).
- Confirma que `enabled` no sea `false`.
- Revisa los logs del Gateway para detectar errores de conexión con los relays.

<div id="not-sending-responses">
  ### No se envían respuestas
</div>

- Comprueba que el relay acepta operaciones de escritura.
- Verifica la conectividad saliente.
- Supervisa los límites de velocidad del relay.

<div id="duplicate-responses">
  ### Respuestas duplicadas
</div>

- Es de esperar cuando se usan múltiples relays.
- Los mensajes se deduplican por ID de evento; solo la primera entrega desencadena una respuesta.

<div id="security">
  ## Seguridad
</div>

- Nunca hagas commit de claves privadas.
- Usa variables de entorno para las claves.
- Considera usar una `allowlist` (lista de permitidos) para bots en producción.

<div id="limitations-mvp">
  ## Limitaciones (MVP)
</div>

- Solo mensajes directos (sin chats grupales).
- Sin archivos multimedia adjuntos.
- Solo NIP-04 (NIP-17 gift-wrap previsto).