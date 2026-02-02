---
title: Envío del agente
summary: "Ejecuciones directas de `openclaw agent` en la CLI (con entrega opcional)"
read_when:
  - Al agregar o modificar el punto de entrada de la CLI del agente
---

<div id="openclaw-agent-direct-agent-runs">
  # `openclaw agent` (ejecuciones directas de agentes)
</div>

`openclaw agent` ejecuta un único turno de un agente sin necesidad de recibir un mensaje de chat entrante.
De forma predeterminada se ejecuta **a través del Gateway**; añade `--local` para forzar el entorno de ejecución
integrado en la máquina actual.

<div id="behavior">
  ## Comportamiento
</div>

- Obligatorio: `--message <text>`
- Selección de sesión:
  - `--to <dest>` deriva la clave de sesión (los destinos de grupo/canal preservan el aislamiento; los chats directos se agrupan en `main`), **o**
  - `--session-id <id>` reutiliza una sesión existente por ID, **o**
  - `--agent <id>` dirige la ejecución directamente a un agente configurado (usa la clave de sesión `main` de ese agente)
- Ejecuta el mismo runtime de agente integrado que las respuestas entrantes normales.
- Las flags de razonamiento/verbose persisten en el almacén de sesión.
- Salida:
  - por defecto: imprime el texto de la respuesta (más líneas `MEDIA:<url>`)
  - `--json`: imprime el payload estructurado + metadatos
- Entrega opcional de respuesta a un canal con `--deliver` + `--channel` (los formatos de destino coinciden con `openclaw message --target`).
- Usa `--reply-channel`/`--reply-to`/`--reply-account` para anular la entrega sin cambiar la sesión.

Si el Gateway no está disponible, la CLI **recurre** a la ejecución local integrada.

<div id="examples">
  ## Ejemplos
</div>

```bash
openclaw agent --to +15555550123 --message "status update"
openclaw agent --agent ops --message "Summarize logs"
openclaw agent --session-id 1234 --message "Summarize inbox" --thinking medium
openclaw agent --to +15555550123 --message "Trace logs" --verbose on --json
openclaw agent --to +15555550123 --message "Summon reply" --deliver
openclaw agent --agent ops --message "Generate report" --deliver --reply-channel slack --reply-to "#reports"
```


<div id="flags">
  ## Flags
</div>

- `--local`: ejecutar localmente (requiere claves API del proveedor de modelos en tu shell)
- `--deliver`: enviar la respuesta al canal elegido
- `--channel`: canal de entrega (`whatsapp|telegram|discord|googlechat|slack|signal|imessage`, predeterminado: `whatsapp`)
- `--reply-to`: sobrescribir el destino de entrega
- `--reply-channel`: sobrescribir el canal de entrega
- `--reply-account`: sobrescribir el ID de la cuenta de entrega
- `--thinking <off|minimal|low|medium|high|xhigh>`: guardar el nivel de razonamiento (solo modelos GPT-5.2 + Codex)
- `--verbose <on|full|off>`: guardar el nivel de verbosidad
- `--timeout <seconds>`: sobrescribir el tiempo de espera del agente
- `--json`: salida JSON estructurada