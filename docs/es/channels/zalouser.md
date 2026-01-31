---
title: Zalouser
summary: "Soporte para cuentas personales de Zalo mediante zca-cli (inicio de sesión mediante código QR), capacidades y configuración"
read_when:
  - Configurar Zalo Personal en OpenClaw
  - Depurar el inicio de sesión o el flujo de mensajes de Zalo Personal
---

<div id="zalo-personal-unofficial">
  # Zalo Personal (no oficial)
</div>

Estado: experimental. Esta integración automatiza una **cuenta personal de Zalo** mediante `zca-cli`.

> **Advertencia:** Esta es una integración no oficial y puede resultar en la suspensión/bloqueo de la cuenta. Úsala bajo tu propia responsabilidad.

<div id="plugin-required">
  ## Complemento requerido
</div>

Zalo Personal se distribuye como un complemento y no viene incluido con la instalación básica.

* Instalar mediante la CLI: `openclaw plugins install @openclaw/zalouser`
* O desde un checkout del código fuente: `openclaw plugins install ./extensions/zalouser`
* Detalles: [Complementos](/es/plugin)

<div id="prerequisite-zca-cli">
  ## Requisito previo: zca-cli
</div>

La máquina del Gateway debe tener el binario `zca` disponible en el `PATH`.

* Comprueba: `zca --version`
* Si no está disponible, instala zca-cli (consulta `extensions/zalouser/README.md` o la documentación oficial de zca-cli).

<div id="quick-setup-beginner">
  ## Configuración rápida (para principiantes)
</div>

1. Instala el complemento (consulta la sección anterior).
2. Inicia sesión (código QR, en la máquina donde se ejecuta el Gateway):
   * `openclaw channels login --channel zalouser`
   * Escanea el código QR que aparece en la terminal con la app móvil de Zalo.
3. Habilita el canal:

```json5
{
  channels: {
    zalouser: {
      enabled: true,
      dmPolicy: "pairing"
    }
  }
}
```

4. Reinicia el Gateway (o finaliza la incorporación).
5. El acceso por DM utiliza el emparejamiento por defecto; aprueba el código de emparejamiento en el primer contacto.

<div id="what-it-is">
  ## Qué es
</div>

* Usa `zca listen` para recibir mensajes entrantes.
* Usa `zca msg ...` para enviar respuestas (texto/contenido multimedia/enlaces).
* Diseñado para casos de uso de cuentas personales donde Zalo Bot API no está disponible.

<div id="naming">
  ## Nomenclatura
</div>

El `channel id` es `zalouser` para dejar claro que esto automatiza una **cuenta personal de usuario de Zalo** (no oficial). Mantenemos `zalo` reservado para una posible integración oficial futura con la api de Zalo.

<div id="finding-ids-directory">
  ## Cómo encontrar IDs (directorio)
</div>

Usa la CLI de directorio para descubrir pares/grupos y sus IDs:

```bash
openclaw directory self --channel zalouser
openclaw directory peers list --channel zalouser --query "name"
openclaw directory groups list --channel zalouser --query "work"
```

<div id="limits">
  ## Límites
</div>

* El texto saliente se divide en fragmentos de ~2000 caracteres (límites del cliente de Zalo).
* El streaming está bloqueado por defecto.

<div id="access-control-dms">
  ## Control de acceso (mensajes directos)
</div>

`channels.zalouser.dmPolicy` admite: `pairing | allowlist | open | disabled` (valor predeterminado: `pairing`; `open` permite aceptar mensajes sin restricciones de cualquier usuario).
`channels.zalouser.allowFrom` acepta identificadores (ID) de usuario o nombres. El asistente resuelve los nombres a ID mediante `zca friend find` cuando está disponible.

Aprueba mediante:

* `openclaw pairing list zalouser`
* `openclaw pairing approve zalouser <code>`

<div id="group-access-optional">
  ## Acceso a grupos (opcional)
</div>

* Predeterminado: `channels.zalouser.groupPolicy = "open"` (open: acepta mensajes de cualquier grupo, sin restricciones). Usa `channels.defaults.groupPolicy` para anular el valor predeterminado cuando no esté establecido.
* Restringir a una lista de permitidos con:
  * `channels.zalouser.groupPolicy = "allowlist"`
  * `channels.zalouser.groups` (las claves son IDs o nombres de grupo)
* Bloquear todos los grupos: `channels.zalouser.groupPolicy = "disabled"`.
* El asistente de configuración puede solicitar listas de permitidos de grupos.
* Al inicio, OpenClaw resuelve los nombres de grupos/usuarios en las listas de permitidos a IDs y registra el mapeo; las entradas que no se puedan resolver se mantienen tal como se escribieron.

Ejemplo:

```json5
{
  channels: {
    zalouser: {
      groupPolicy: "allowlist",
      groups: {
        "123456789": { allow: true },
        "Work Chat": { allow: true }
      }
    }
  }
}
```

<div id="multi-account">
  ## Varias cuentas
</div>

Las cuentas corresponden a perfiles de zca. Ejemplo:

```json5
{
  channels: {
    zalouser: {
      enabled: true,
      defaultAccount: "default",
      accounts: {
        work: { enabled: true, profile: "work" }
      }
    }
  }
}
```

<div id="troubleshooting">
  ## Solución de problemas
</div>

**`zca` no se encuentra:**

* Instala zca-cli y asegúrate de que esté en el `PATH` del proceso del Gateway.

**El inicio de sesión no persiste:**

* `openclaw channels status --probe`
* Vuelve a iniciar sesión: `openclaw channels logout --channel zalouser && openclaw channels login --channel zalouser`