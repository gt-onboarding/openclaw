---
title: Indicadores de escritura
summary: "Cuándo muestra OpenClaw indicadores de escritura y cómo configurarlos"
read_when:
  - Cambiar el comportamiento o los valores predeterminados de los indicadores de escritura
---

<div id="typing-indicators">
  # Indicadores de escritura
</div>

Los indicadores de escritura se envían al canal de chat mientras una ejecución está en curso. Usa
`agents.defaults.typingMode` para controlar **cuándo** comienza la escritura y `typingIntervalSeconds`
para controlar **con qué frecuencia** se actualiza.

<div id="defaults">
  ## Valores predeterminados
</div>

Cuando `agents.defaults.typingMode` **no está definido**, OpenClaw mantiene el comportamiento heredado:

- **Chats directos**: el indicador de escritura comienza inmediatamente una vez que se inicia el bucle del modelo.
- **Chats de grupo con una mención**: el indicador de escritura comienza de inmediato.
- **Chats de grupo sin una mención**: el indicador de escritura comienza solo cuando el texto del mensaje empieza a transmitirse.
- **Ejecuciones de heartbeat (latido)**: el indicador de escritura está desactivado.

<div id="modes">
  ## Modos
</div>

Configura `agents.defaults.typingMode` en uno de los siguientes valores:

- `never` — sin indicador de escritura, nunca.
- `instant` — comienza a escribir **tan pronto como comienza el bucle del modelo**, incluso si la ejecución
  luego devuelve solo el token de respuesta silenciosa.
- `thinking` — comienza a escribir con el **primer delta de razonamiento** (requiere
  `reasoningLevel: "stream"` para la ejecución).
- `message` — comienza a escribir con el **primer delta de texto no silencioso** (ignora
  el token silencioso `NO_REPLY`).

Orden de “qué tan pronto se activa”:
`never` → `message` → `thinking` → `instant`

<div id="configuration">
  ## Configuración
</div>

```json5
{
  agent: {
    typingMode: "thinking",
    typingIntervalSeconds: 6
  }
}
```

Puedes anular el modo o la cadencia para cada sesión:

```json5
{
  session: {
    typingMode: "message",
    typingIntervalSeconds: 4
  }
}
```


<div id="notes">
  ## Notas
</div>

- El modo `message` no mostrará el indicador de escritura para respuestas silenciosas (por ejemplo, el token `NO_REPLY`
  usado para suprimir la salida).
- `thinking` solo se activa si la ejecución transmite el razonamiento (`reasoningLevel: "stream"`).
  Si el modelo no emite deltas de razonamiento, el indicador de escritura no se iniciará.
- Los latidos nunca muestran el indicador de escritura, sin importar el modo.
- `typingIntervalSeconds` controla la **cadencia de actualización**, no el momento de inicio.
  El valor predeterminado es 6 segundos.