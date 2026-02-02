---
title: Endurecimiento de directivas de grupo
summary: "Endurecimiento de la lista de permitidos de Telegram: normalización de prefijos y espacios en blanco"
read_when:
  - Revisar cambios históricos en la lista de permitidos de Telegram
---

<div id="telegram-allowlist-hardening">
  # Refuerzo de la lista de permitidos de Telegram
</div>

**Fecha**: 2026-01-05\
**Estado**: Completado\
**PR**: #216

<div id="summary">
  ## Resumen
</div>

Las listas de permitidos de Telegram ahora aceptan los prefijos `telegram:` y `tg:` sin distinción entre mayúsculas y minúsculas y toleran espacios en blanco accidentales. Esto alinea las comprobaciones de listas de permitidos de entrada con la normalización de las operaciones `send` de salida.

<div id="what-changed">
  ## Qué ha cambiado
</div>

* Los prefijos `telegram:` y `tg:` se tratan igual (sin distinguir entre mayúsculas y minúsculas).
* Las entradas de la lista de permitidos se recortan; las entradas vacías se omiten.

<div id="examples">
  ## Ejemplos
</div>

Todos estos son válidos para el mismo ID:

* `telegram:123456`
* `TG:123456`
* `tg:123456`

<div id="why-it-matters">
  ## Por qué es importante
</div>

Copiar y pegar desde registros o IDs de chat a menudo incluye prefijos y espacios en blanco. La normalización evita falsos negativos al determinar si responder por mensajes directos o en grupos.

<div id="related-docs">
  ## Documentación relacionada
</div>

* [Chats de grupo](/es/concepts/groups)
* [Proveedor de Telegram](/es/channels/telegram)