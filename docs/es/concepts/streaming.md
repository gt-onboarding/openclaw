---
title: Streaming
summary: "Comportamiento de streaming y fragmentación (respuestas por bloques, streaming de borradores, límites)"
read_when:
  - Explicar cómo funciona el streaming o la fragmentación en los canales
  - Cambiar el comportamiento de streaming por bloques o de fragmentación del canal
  - Depurar respuestas por bloques duplicadas o adelantadas, o el streaming de borradores
---

<div id="streaming-chunking">
  # Streaming + fragmentación
</div>

OpenClaw tiene dos capas de streaming separadas:

- **Streaming por bloques (canales):** emite **bloques** completos a medida que el asistente escribe. Estos son mensajes de canal normales (no deltas de tokens).
- **Streaming tipo token (solo Telegram):** actualiza una **burbuja de borrador** con texto parcial durante la generación; el mensaje final se envía al terminar.

Hoy no hay **streaming real de tokens** hacia mensajes de canales externos. El streaming de borradores de Telegram es el único punto de streaming parcial disponible.

<div id="block-streaming-channel-messages">
  ## Transmisión por bloques (mensajes de canal)
</div>

La transmisión por bloques envía la salida del asistente en fragmentos gruesos a medida que van estando disponibles.

```
Model output
  └─ text_delta/events
       ├─ (blockStreamingBreak=text_end)
       │    └─ chunker emits blocks as buffer grows
       └─ (blockStreamingBreak=message_end)
            └─ chunker flushes at message_end
                   └─ channel send (block replies)
```

Leyenda:

* `text_delta/events`: eventos de streaming del modelo (pueden ser poco frecuentes en modelos sin streaming).
* `chunker`: `EmbeddedBlockChunker` que aplica límites mínimo y máximo + preferencia de ruptura.
* `channel send`: mensajes salientes reales (respuestas en bloques).

**Controles:**

* `agents.defaults.blockStreamingDefault`: `"on"`/`"off"` (por defecto off).
* Anulaciones por canal: `*.blockStreaming` (y variantes por cuenta) para forzar `"on"`/`"off"` por canal.
* `agents.defaults.blockStreamingBreak`: `"text_end"` o `"message_end"`.
* `agents.defaults.blockStreamingChunk`: `{ minChars, maxChars, breakPreference? }`.
* `agents.defaults.blockStreamingCoalesce`: `{ minChars?, maxChars?, idleMs? }` (fusiona bloques transmitidos antes de `send`).
* Límite rígido por canal: `*.textChunkLimit` (por ejemplo, `channels.whatsapp.textChunkLimit`).
* Modo de fragmentación por canal: `*.chunkMode` (`length` por defecto, `newline` divide en líneas en blanco (límites de párrafo) antes de fragmentar por longitud).
* Límite blando de Discord: `channels.discord.maxLinesPerMessage` (17 por defecto) divide respuestas muy largas para evitar recorte en la UI.

**Semántica de los límites:**

* `text_end`: transmite bloques tan pronto como el `chunker` los emite; vacía en cada `text_end`.
* `message_end`: espera hasta que termine el mensaje del asistente y luego vacía la salida en búfer.

`message_end` sigue usando el `chunker` si el texto en búfer supera `maxChars`, por lo que puede emitir varios fragmentos al final.


<div id="chunking-algorithm-lowhigh-bounds">
  ## Algoritmo de segmentación (límites mínimo/máximo)
</div>

La segmentación en bloques se implementa mediante `EmbeddedBlockChunker`:

- **Límite mínimo:** no emitir hasta que el búfer sea >= `minChars` (a menos que se fuerce).
- **Límite máximo:** se prefieren cortes antes de `maxChars`; si se fuerza, cortar en `maxChars`.
- **Preferencia de corte:** `paragraph` → `newline` → `sentence` → `whitespace` → corte forzado.
- **Bloques de código (code fences):** nunca cortar dentro de ellos; cuando se fuerce en `maxChars`, cerrar y volver a abrir el bloque para mantener un Markdown válido.

`maxChars` se ajusta al `textChunkLimit` del canal, por lo que no puedes exceder los límites por canal.

<div id="coalescing-merge-streamed-blocks">
  ## Coalescencia (fusionar bloques en streaming)
</div>

Cuando el streaming de bloques está habilitado, OpenClaw puede **fusionar fragmentos de bloque consecutivos**
antes de enviarlos. Esto reduce el “spam de una sola línea” y sigue proporcionando
salida progresiva.

- La coalescencia espera **intervalos de inactividad** (`idleMs`) antes de vaciar el búfer.
- Los búferes están limitados por `maxChars` y se vaciarán si se supera ese valor.
- `minChars` evita que se envíen fragmentos muy pequeños hasta que se acumule suficiente texto
  (el vaciado final siempre envía el texto restante).
- El separador se deriva de `blockStreamingChunk.breakPreference`
  (`paragraph` → `\n\n`, `newline` → `\n`, `sentence` → espacio).
- Hay opciones de sobrescritura específicas por canal mediante `*.blockStreamingCoalesce` (incluidas configuraciones por cuenta).
- El valor predeterminado de `minChars` para la coalescencia se incrementa a 1500 para Signal/Slack/Discord, salvo que se sobrescriba.

<div id="human-like-pacing-between-blocks">
  ## Ritmo más humano entre bloques
</div>

Cuando la transmisión por bloques está habilitada, puedes añadir una **pausa aleatoria**
entre bloques de respuesta (después del primer bloque). Esto hace que las respuestas con múltiples burbujas se sientan
más naturales.

- Config: `agents.defaults.humanDelay` (puedes sobrescribirlo por agente mediante `agents.list[].humanDelay`).
- Modos: `off` (predeterminado), `natural` (800–2500ms), `custom` (`minMs`/`maxMs`).
- Se aplica solo a **respuestas por bloques**, no a respuestas finales ni a resúmenes de herramientas.

<div id="stream-chunks-or-everything">
  ## "Transmitir fragmentos o todo"
</div>

Esto se corresponde con:

- **Transmitir fragmentos:** `blockStreamingDefault: "on"` + `blockStreamingBreak: "text_end"` (se emite a medida que se genera). Los canales que no son de Telegram también necesitan `*.blockStreaming: true`.
- **Transmitir todo al final:** `blockStreamingBreak: "message_end"` (se envía de una vez, posiblemente en varios fragmentos si es muy largo).
- **Sin transmisión por bloques:** `blockStreamingDefault: "off"` (solo la respuesta final).

**Nota sobre canales:** Para canales que no son de Telegram, la transmisión por bloques está **desactivada a menos que**
`*.blockStreaming` se establezca explícitamente en `true`. Telegram puede transmitir borradores
(`channels.telegram.streamMode`) sin respuestas por bloques.

Recordatorio sobre la ubicación de la configuración: los valores predeterminados de `blockStreaming*` se definen bajo
`agents.defaults`, no en la configuración raíz.

<div id="telegram-draft-streaming-token-ish">
  ## Streaming de borradores en Telegram (a nivel de tokens)
</div>

Telegram es el único canal con streaming de borradores:

* Usa la Bot API `sendMessageDraft` en **chats privados con temas habilitados**.
* `channels.telegram.streamMode: "partial" | "block" | "off"`.
  * `partial`: actualiza el borrador con el texto más reciente del streaming.
  * `block`: actualiza el borrador en bloques (mismas reglas del chunker).
  * `off`: sin streaming de borradores.
* Configuración de fragmentación del borrador (solo para `streamMode: "block"`): `channels.telegram.draftChunk` (valores por defecto: `minChars: 200`, `maxChars: 800`).
* El streaming de borradores es independiente del streaming por bloques; las respuestas en bloques están desactivadas por defecto y solo se activan con `*.blockStreaming: true` en canales que no sean Telegram.
* La respuesta final sigue siendo un mensaje normal.
* `/reasoning stream` escribe el razonamiento en la burbuja del borrador (solo Telegram).

Cuando el streaming de borradores está activo, OpenClaw desactiva el streaming por bloques para esa respuesta para evitar el doble streaming.

```
Telegram (privado + topics)
  └─ sendMessageDraft (burbuja de borrador)
       ├─ streamMode=partial → actualiza el último texto
       └─ streamMode=block   → el chunker actualiza el borrador
  └─ respuesta final → mensaje normal
```

Leyenda:

* `sendMessageDraft`: burbuja de borrador de Telegram (no es un mensaje real).
* `final reply`: mensaje normal enviado en Telegram.
