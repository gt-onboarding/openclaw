---
title: Configurar
summary: "Referencia de la CLI de `openclaw configure` (asistentes de configuración interactivos)"
read_when:
  - Quieres ajustar de forma interactiva credenciales, dispositivos o valores predeterminados de los agentes
---

<div id="openclaw-configure">
  # `openclaw configure`
</div>

Asistente interactivo para configurar credenciales, dispositivos y valores predeterminados de agentes.

Nota: La sección **Model** ahora incluye una selección múltiple para la
lista de permitidos `agents.defaults.models` (lo que aparece en `/model` y en el selector de modelos).

Consejo: `openclaw config` sin un subcomando abre el mismo asistente. Usa
`openclaw config get|set|unset` para ediciones no interactivas.

Relacionado:

* Referencia de configuración del Gateway: [Configuration](/es/gateway/configuration)
* CLI de configuración: [Config](/es/cli/config)

Notas:

* Elegir dónde se ejecuta el Gateway siempre actualiza `gateway.mode`. Puedes seleccionar &quot;Continuar&quot; sin otras secciones si eso es todo lo que necesitas.
* Los servicios orientados a canales (Slack/Discord/Matrix/Microsoft Teams) te pedirán listas de permitidos de canales/salas durante la configuración. Puedes introducir nombres o ID; el asistente resuelve nombres a ID cuando es posible.

<div id="examples">
  ## Ejemplos
</div>

```bash
openclaw configure
openclaw configure --section models --section channels
```
