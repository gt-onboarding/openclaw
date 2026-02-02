---
title: Superposición de voz
summary: "Ciclo de vida de la superposición de voz cuando la palabra de activación y el modo pulsar para hablar se superponen"
read_when:
  - Ajustar el comportamiento de la superposición de voz
---

<div id="voice-overlay-lifecycle-macos">
  # Ciclo de vida del overlay de voz (macOS)
</div>

Audiencia: colaboradores de la aplicación para macOS. Objetivo: mantener el overlay de voz predecible cuando se solapan la palabra de activación y el modo pulsar para hablar.

<div id="current-intent">
  ### Intención actual
</div>

- Si el overlay ya es visible por la palabra de activación y el usuario pulsa el atajo de teclado, la sesión del atajo de teclado *adopta* el texto existente en lugar de restablecerlo. El overlay permanece visible mientras se mantenga pulsado el atajo. Cuando el usuario lo suelta: enviar si hay texto (tras recortar espacios en blanco), de lo contrario descartar.
- La palabra de activación por sí sola sigue enviando automáticamente al detectar silencio; el modo pulsar‑para‑hablar (push‑to‑talk) envía inmediatamente al soltar.

<div id="implemented-dec-9-2025">
  ### Implementado (9 de diciembre de 2025)
</div>

- Las sesiones de superposición ahora llevan un token por captura (palabra de activación o pulsar para hablar). Las actualizaciones parciales/finales/send/dismiss/level se descartan cuando el token no coincide, evitando callbacks desfasados.
- Pulsar para hablar adopta cualquier texto visible de la superposición como prefijo (por lo que, al pulsar el atajo de teclado mientras la superposición de activación está visible, se mantiene el texto y se añade el nuevo dictado). Espera hasta 1,5 s una transcripción final antes de recurrir al texto actual.
- El registro de tonos/superposición se emite con nivel `info` en las categorías `voicewake.overlay`, `voicewake.ptt` y `voicewake.chime` (inicio de sesión, parcial, final, send, dismiss, motivo del tono).

<div id="next-steps">
  ### Próximos pasos
</div>

1. **VoiceSessionCoordinator (actor)**
   - Gestiona exactamente una `VoiceSession` a la vez.
   - API (basada en tokens): `beginWakeCapture`, `beginPushToTalk`, `updatePartial`, `endCapture`, `cancel`, `applyCooldown`.
   - Descarta callbacks que llevan tokens obsoletos (evita que reconocedores antiguos vuelvan a abrir el overlay).
2. **VoiceSession (model)**
   - Campos: `token`, `source` (wakeWord|pushToTalk), texto confirmado/volátil, flags de sonido, temporizadores (envío automático, inactividad), `overlayMode` (display|editing|sending), fecha límite de enfriamiento.
3. **Vinculación del overlay**
   - `VoiceSessionPublisher` (`ObservableObject`) refleja la sesión activa en SwiftUI.
   - `VoiceWakeOverlayView` se renderiza solo a través del publisher; nunca muta singletons globales directamente.
   - Las acciones de usuario del overlay (`sendNow`, `dismiss`, `edit`) llaman de vuelta al coordinador con el token de la sesión.
4. **Ruta de envío unificada**
   - En `endCapture`: si el texto recortado está vacío → descartar; de lo contrario `performSend(session:)` (reproduce el sonido de envío una vez, reenvía, cierra).
   - Push-to-talk: sin demora; wake-word: demora opcional para envío automático.
   - Aplica un breve enfriamiento al runtime de wake después de que termine push-to-talk para que la wake-word no se vuelva a activar inmediatamente.
5. **Registro (logging)**
   - El coordinador emite logs `.info` en el subsistema `bot.molt`, categorías `voicewake.overlay` y `voicewake.chime`.
   - Eventos clave: `session_started`, `adopted_by_push_to_talk`, `partial`, `finalized`, `send`, `dismiss`, `cancel`, `cooldown`.

<div id="debugging-checklist">
  ### Lista de comprobación de depuración
</div>

- Haz streaming de los registros mientras reproduces una superposición persistente:

  ```bash
  sudo log stream --predicate 'subsystem == "bot.molt" AND category CONTAINS "voicewake"' --level info --style compact
  ```
- Comprueba que solo haya un token de sesión activo; las callbacks obsoletas deberían ser descartadas por el coordinador.
- Asegúrate de que, al soltar el botón de push-to-talk, siempre se llame a `endCapture` con el token activo; si el texto está vacío, espera `dismiss` sin sonido ni send.

<div id="migration-steps-suggested">
  ### Pasos de migración (recomendados)
</div>

1. Agrega `VoiceSessionCoordinator`, `VoiceSession` y `VoiceSessionPublisher`.
2. Refactoriza `VoiceWakeRuntime` para crear/actualizar/cerrar sesiones en lugar de interactuar directamente con `VoiceWakeOverlayController`.
3. Refactoriza `VoicePushToTalk` para adoptar las sesiones existentes y llamar a `endCapture` al soltar; aplica un periodo de enfriamiento en tiempo de ejecución.
4. Conecta `VoiceWakeOverlayController` al publisher; elimina las llamadas directas desde el runtime/PTT.
5. Agrega pruebas de integración para la adopción de sesiones, el periodo de enfriamiento y el descarte de texto vacío.