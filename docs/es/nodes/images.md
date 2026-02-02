---
title: Imágenes
summary: "Reglas para el manejo de imágenes y contenido multimedia en las respuestas de send, Gateway y agentes"
read_when:
  - Modificar el pipeline de medios o los archivos adjuntos
---

<div id="image-media-support-2025-12-05">
  # Compatibilidad con imágenes y contenido multimedia — 2025-12-05
</div>

El canal de WhatsApp se ejecuta mediante **Baileys Web**. Este documento recoge las reglas actuales para el manejo de contenido multimedia para send, el Gateway y las respuestas de los agentes.

<div id="goals">
  ## Objetivos
</div>

- Enviar contenido multimedia con texto opcional mediante `openclaw message send --media`.
- Permitir que las respuestas automáticas desde la bandeja de entrada web incluyan contenido multimedia junto con texto.
- Mantener límites razonables y predecibles por tipo.

<div id="cli-surface">
  ## Interfaz de la CLI
</div>

- `openclaw message send --media <path-or-url> [--message <caption>]`
  - `--media` opcional; el pie de foto puede estar vacío para envíos solo de contenido multimedia.
  - `--dry-run` imprime la carga útil resultante; `--json` emite `{ channel, to, messageId, mediaUrl, caption }`.

<div id="whatsapp-web-channel-behavior">
  ## Comportamiento del canal WhatsApp Web
</div>

- Entrada: ruta de archivo local **o** URL HTTP(S).
- Flujo: cargar en un `Buffer`, detectar el tipo de medio y construir el payload correcto:
  - **Imágenes:** redimensionar y recomprimir a JPEG (lado máximo 2048 px) apuntando a `agents.defaults.mediaMaxMb` (por defecto 5 MB), con tope de 6 MB.
  - **Audio/Voice/Video:** pasa sin modificaciones hasta 16 MB; el audio se envía como nota de voz (`ptt: true`).
  - **Documentos:** cualquier otro tipo de archivo, hasta 100 MB, conservando el nombre de archivo cuando esté disponible.
- Reproducción estilo GIF de WhatsApp: enviar un MP4 con `gifPlayback: true` (CLI: `--gif-playback`) para que los clientes móviles lo reproduzcan en bucle dentro del chat.
- La detección de MIME prioriza los magic bytes, luego los encabezados y después la extensión del archivo.
- El texto de la leyenda (caption) proviene de `--message` o `reply.text`; se permite una leyenda vacía.
- Registro (logging): en modo no verboso muestra `↩️`/`✅`; en modo verboso incluye el tamaño y la ruta/URL de origen.

<div id="auto-reply-pipeline">
  ## Canalización de respuesta automática
</div>

- `getReplyFromConfig` devuelve `{ text?, mediaUrl?, mediaUrls? }`.
- Cuando hay contenido multimedia, el componente de envío web resuelve rutas locales o URL utilizando la misma canalización que `openclaw message send`.
- Si se proporcionan varios elementos multimedia, se envían secuencialmente.

<div id="inbound-media-to-commands-pi">
  ## Medios entrantes a comandos (Pi)
</div>

- Cuando los mensajes web entrantes incluyen contenido multimedia, OpenClaw lo descarga a un archivo temporal y expone variables de plantilla:
  - `{{MediaUrl}}` pseudo-URL para el medio entrante.
  - `{{MediaPath}}` ruta temporal local escrita antes de ejecutar el comando.
- Cuando está habilitado un sandbox de Docker por sesión, los medios entrantes se copian en el espacio de trabajo del sandbox y `MediaPath`/`MediaUrl` se reescriben a una ruta relativa como `media/inbound/<filename>`.
- El procesamiento/entendimiento de medios (si se configura mediante `tools.media.*` o `tools.media.models` compartidos) se ejecuta antes del procesado de plantillas y puede insertar bloques `[Image]`, `[Audio]` y `[Video]` en `Body`.
  - Audio establece `{{Transcript}}` y usa la transcripción para el análisis de comandos, de modo que los comandos con barra inclinada (slash) sigan funcionando.
  - Las descripciones de vídeo e imagen conservan cualquier texto de subtítulo para el análisis de comandos.
- De forma predeterminada solo se procesa el primer archivo adjunto de imagen/audio/vídeo que coincida; configura `tools.media.<cap>.attachments` para procesar varios archivos adjuntos.

<div id="limits-errors">
  ## Límites y errores
</div>

**Límites de envío saliente (envío web de WhatsApp)**

- Imágenes: límite de ~6 MB después de la recomprensión.
- Audio/voz/vídeo: límite de 16 MB; documentos: límite de 100 MB.
- Archivos multimedia demasiado grandes o ilegibles → error claro en los logs y se omite la respuesta.

**Límites de comprensión de medios (transcripción/descripción)**

- Imagen (valor predeterminado): 10 MB (`tools.media.image.maxBytes`).
- Audio (valor predeterminado): 20 MB (`tools.media.audio.maxBytes`).
- Vídeo (valor predeterminado): 50 MB (`tools.media.video.maxBytes`).
- Los archivos multimedia demasiado grandes se omiten en la fase de comprensión, pero las respuestas igualmente se envían con el cuerpo original.

<div id="notes-for-tests">
  ## Notas para pruebas
</div>

- Cubre los flujos de send + reply para casos de imágenes/audio/documentos.
- Valida la recompresión de imágenes (límite de tamaño) y el indicador de nota de voz para audio.
- Asegúrate de que las respuestas multimedia se expandan en sends secuenciales.