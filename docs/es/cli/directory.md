---
title: Directorio
summary: "Referencia de la CLI para `openclaw directory` (self, peers, groups)"
read_when:
  - Quieres consultar IDs de contactos, grupos o de tu propio usuario (self) para un canal
  - Estás desarrollando un adaptador de directorio de canales
---

<div id="openclaw-directory">
  # `openclaw directory`
</div>

Consultas de directorio para los canales que lo admiten (contactos/pares, grupos y «yo»).

<div id="common-flags">
  ## Flags comunes
</div>

- `--channel <name>`: id/alias del canal (obligatorio cuando hay varios canales configurados; automático si solo hay uno configurado)
- `--account <id>`: id de la cuenta (por defecto: la del canal)
- `--json`: salida en JSON

<div id="notes">
  ## Notas
</div>

- `directory` está pensado para ayudarte a encontrar ID que puedas pegar en otros comandos (especialmente `openclaw message send --target ...`).
- En muchos canales, los resultados provienen de la configuración (listas de permitidos / grupos configurados) en lugar de un directorio del proveedor en tiempo real.
- La salida predeterminada es `id` (y a veces `name`) separados por un tabulador; usa `--json` para automatizar con scripts.

<div id="using-results-with-message-send">
  ## Uso de los resultados con `message send`
</div>

```bash
openclaw directory peers list --channel slack --query "U0"
openclaw message send --channel slack --target user:U012ABCDEF --message "hello"
```


<div id="id-formats-by-channel">
  ## Formatos de ID (por canal)
</div>

- WhatsApp: `+15551234567` (DM), `1234567890-1234567890@g.us` (grupo)
- Telegram: `@username` o ID de chat numérico; los grupos usan IDs numéricos
- Slack: `user:U…` y `channel:C…`
- Discord: `user:<id>` y `channel:<id>`
- Matrix (complemento): `user:@user:server`, `room:!roomId:server` o `#alias:server`
- Microsoft Teams (complemento): `user:<id>` y `conversation:<id>`
- Zalo (complemento): ID de usuario (Bot API)
- Zalo Personal / `zalouser` (complemento): ID de hilo (DM/grupo) obtenido de `zca` (`me`, `friend list`, `group list`)

<div id="self-me">
  ## Yo mismo («yo»)
</div>

```bash
openclaw directory self --channel zalouser
```


<div id="peers-contactsusers">
  ## Pares (usuarios/contactos)
</div>

```bash
openclaw directory peers list --channel zalouser
openclaw directory peers list --channel zalouser --query "name"
openclaw directory peers list --channel zalouser --limit 50
```


<div id="groups">
  ## Grupos
</div>

```bash
openclaw directory groups list --channel zalouser
openclaw directory groups list --channel zalouser --query "work"
openclaw directory groups members --channel zalouser --group-id <id>
```
