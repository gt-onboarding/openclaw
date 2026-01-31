---
title: Voicewake
summary: "Modos de activación por voz y pulsar para hablar, y detalles de enrutamiento en la aplicación para Mac"
read_when:
  - Trabajando en flujos de activación por voz o PTT
---

<div id="voice-wake-push-to-talk">
  # Activación por voz y pulsar para hablar
</div>

<div id="modes">
  ## Modos
</div>

- **Modo de palabra de activación** (predeterminado): el reconocedor de voz siempre activo espera las palabras de activación (`swabbleTriggerWords`). Cuando hay coincidencia, inicia la captura, muestra la superposición con texto parcial y lo envía automáticamente después de un período de silencio.
- **Pulsar para hablar (mantener Opción derecha)**: mantén presionada la tecla Opción derecha para capturar de inmediato—no se necesita palabra de activación. La superposición aparece mientras la mantienes; al soltar, se finaliza y se envía después de un breve retraso para que puedas ajustar el texto.

<div id="runtime-behavior-wake-word">
  ## Comportamiento en tiempo de ejecución (palabra de activación)
</div>

- El reconocedor de voz se ejecuta en `VoiceWakeRuntime`.
- El activador solo se dispara cuando hay una **pausa significativa** entre la palabra de activación y la siguiente palabra (~0,55 s de separación). La superposición/campanilla puede comenzar durante la pausa incluso antes de que empiece el comando.
- Ventanas de silencio: 2,0 s cuando el habla es continua, 5,0 s si solo se oyó el activador.
- Límite forzoso: 120 s para evitar sesiones descontroladas.
- Antirrebote entre sesiones: 350 ms.
- La superposición está controlada por `VoiceWakeOverlayController` con coloración de estados confirmado/volátil.
- Después de `send`, el reconocedor se reinicia limpiamente para escuchar el siguiente activador.

<div id="lifecycle-invariants">
  ## Invariantes del ciclo de vida
</div>

- Si Voice Wake está habilitado y se han otorgado los permisos, el reconocedor de la palabra de activación debe estar escuchando (excepto durante una captura explícita de pulsar para hablar).
- La visibilidad de la superposición (incluida la ocultación manual mediante el botón X) nunca debe impedir que el reconocedor se reanude.

<div id="sticky-overlay-failure-mode-previous">
  ## Modo de fallo de superposición persistente (anterior)
</div>

Anteriormente, si la superposición quedaba atascada visible y la cerrabas manualmente, Voice Wake podía parecer “muerto” porque el intento de reinicio del runtime podía quedar bloqueado debido a que la superposición seguía visible y no se programaba ningún reinicio posterior.

Robustecimiento:

- El reinicio del runtime de Wake ya no se ve bloqueado por la visibilidad de la superposición.
- Al completarse el cierre de la superposición se desencadena un `VoiceWakeRuntime.refresh(...)` a través de `VoiceSessionCoordinator`, por lo que el cierre manual mediante la X siempre reanuda la escucha.

<div id="push-to-talk-specifics">
  ## Detalles de pulsar para hablar
</div>

- La detección del atajo usa un monitor global `.flagsChanged` para la **tecla Opción derecha** (`keyCode 61` + `.option`). Solo observamos los eventos (no los consumimos).
- El pipeline de captura reside en `VoicePushToTalk`: inicia Speech inmediatamente, transmite resultados parciales al overlay y llama a `VoiceWakeForwarder` al soltar.
- Cuando se inicia pulsar para hablar, pausamos el runtime de palabra de activación para evitar conflictos entre taps de audio; se reinicia automáticamente después de soltar.
- Permisos: requiere Micrófono + Speech; para ver eventos se necesita aprobación de Accesibilidad/Supervisión de entrada.
- Teclados externos: algunos pueden no exponer la tecla Opción derecha como se espera; ofrece un atajo alternativo si los usuarios reportan fallos.

<div id="user-facing-settings">
  ## Configuración orientada al usuario
</div>

- Conmutador **Activación por voz**: habilita la ejecución mediante palabra de activación.
- **Mantener pulsado Cmd+Fn para hablar**: habilita el monitor de pulsar para hablar (push-to-talk). Desactivado en macOS &lt; 26.
- Selectores de idioma y micrófono, medidor de nivel en tiempo real, tabla de palabras de activación, herramienta de prueba (solo local; no reenvía).
- El selector de micrófono conserva la última selección si un dispositivo se desconecta, muestra un aviso de desconexión y recurre temporalmente al valor predeterminado del sistema hasta que este vuelva.
- **Sonidos**: emite un tono al detectar la palabra de activación y al enviar; de forma predeterminada utiliza el sonido del sistema de macOS “Glass”. Puedes elegir cualquier archivo cargable por `NSSound` (p. ej., MP3/WAV/AIFF) para cada evento o elegir **Sin sonido**.

<div id="forwarding-behavior">
  ## Comportamiento de reenvío
</div>

- Cuando Voice Wake está activado, las transcripciones se reenvían al Gateway/agente activo (el mismo modo local o remoto que usa el resto de la app de macOS).
- Las respuestas se entregan al **proveedor principal utilizado más recientemente** (WhatsApp/Telegram/Discord/WebChat). Si la entrega falla, se registra el error y la ejecución sigue siendo visible en los registros de WebChat o de la sesión.

<div id="forwarding-payload">
  ## Reenvío del payload
</div>

- `VoiceWakeForwarder.prefixedTranscript(_:)` antepone la indicación para la máquina antes de enviar. Se comparte entre las rutas de palabra de activación y de pulsar para hablar.

<div id="quick-verification">
  ## Verificación rápida
</div>

- Activa el modo pulsar para hablar, mantén pulsadas Cmd+Fn, habla y suelta: la superposición debería mostrar parciales y luego enviar.
- Mientras lo mantengas pulsado, las orejas de la barra de menús deberían permanecer agrandadas (usa `triggerVoiceEars(ttl:nil)`); se encogen después de soltar.