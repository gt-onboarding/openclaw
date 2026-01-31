---
title: Formato de Markdown
summary: "Pipeline de formato Markdown para canales salientes"
read_when:
  - Estás cambiando el formato de Markdown o la fragmentación (chunking) para canales salientes
  - Estás añadiendo un nuevo formateador de canal o mapeo de estilos
  - Estás depurando regresiones de formato en canales
---

<div id="markdown-formatting">
  # Formato de Markdown
</div>

OpenClaw formatea el Markdown de salida convirtiéndolo en una representación
intermedia (IR) compartida antes de generar la salida específica de cada canal. La IR
mantiene el texto original intacto mientras transporta los rangos de estilo/enlaces,
de modo que la fragmentación y el renderizado se mantengan consistentes en todos los canales.

<div id="goals">
  ## Objetivos
</div>

* **Consistencia:** un único paso de análisis, múltiples renderizadores.
* **Segmentación segura:** dividir el texto antes de renderizar para que el formato en línea nunca se rompa entre fragmentos.
* **Compatibilidad con canales:** mapear el mismo IR a Slack mrkdwn, HTML de Telegram y rangos de estilo de Signal sin volver a analizar el Markdown.

<div id="pipeline">
  ## Pipeline
</div>

1. **Analizar Markdown -&gt; IR**
   * IR es texto plano con intervalos de estilo (negrita/cursiva/tachado/código/spoiler) e intervalos de enlace.
   * Los desplazamientos se expresan en unidades de código UTF-16, de modo que los rangos de estilo de Signal se alineen con su API.
   * Las tablas solo se analizan cuando un canal opta por la conversión de tablas.
2. **Dividir IR en fragmentos (format-first)**
   * La división en fragmentos ocurre sobre el texto IR antes del renderizado.
   * El formato en línea no se divide entre fragmentos; los intervalos se recortan por fragmento.
3. **Renderizar por canal**
   * **Slack:** tokens mrkdwn (negrita/cursiva/tachado/código), enlaces como `<url|label>`.
   * **Telegram:** etiquetas HTML (`<b>`, `<i>`, `<s>`, `<code>`, `<pre><code>`, `<a href>`).
   * **Signal:** texto plano + intervalos `text-style`; los enlaces se convierten en `label (url)` cuando la etiqueta es distinta.

<div id="ir-example">
  ## Ejemplo de IR
</div>

Markdown de entrada:

```markdown
Hello **world** — see [docs](https://docs.openclaw.ai).
```

IR (esquemático):

```json
{
  "text": "Hello world — see docs.",
  "styles": [
    { "start": 6, "end": 11, "style": "bold" }
  ],
  "links": [
    { "start": 19, "end": 23, "href": "https://docs.openclaw.ai" }
  ]
}
```

<div id="where-it-is-used">
  ## Dónde se utiliza
</div>

* Los adaptadores de salida de Slack, Telegram y Signal se representan a partir del IR.
* Otros canales (WhatsApp, iMessage, MS Teams, Discord) siguen usando texto sin formato o
  sus propias reglas de formato, con conversión de tablas Markdown aplicada antes de
  dividir en fragmentos cuando está habilitada.

<div id="table-handling">
  ## Manejo de tablas
</div>

Las tablas de Markdown no se admiten de manera uniforme en todos los clientes de chat. Usa
`markdown.tables` para controlar la conversión por canal (y por cuenta).

* `code`: muestra las tablas como bloques de código (valor predeterminado para la mayoría de los canales).
* `bullets`: convierte cada fila en viñetas (valor predeterminado para Signal y WhatsApp).
* `off`: desactiva el análisis y la conversión de tablas; el texto de la tabla en bruto se transmite tal cual.

Claves de configuración:

```yaml
channels:
  discord:
    markdown:
      tables: code
    accounts:
      work:
        markdown:
          tables: off
```

<div id="chunking-rules">
  ## Reglas de fragmentación
</div>

* Los límites de fragmentos provienen de los adaptadores de canal y su configuración, y se aplican al texto IR.
* Los bloques de código delimitados se conservan como un único bloque con un salto de línea final para que los canales
  los representen correctamente.
* Los prefijos de listas y los prefijos de citas en bloque forman parte del texto IR, por lo que la fragmentación
  no se realiza en mitad de un prefijo.
* Los estilos en línea (negrita/cursiva/tachado/código en línea/spoiler) nunca se dividen entre
  fragmentos; el renderizador vuelve a abrir los estilos dentro de cada fragmento.

Si necesitas más información sobre el comportamiento de la fragmentación entre canales, consulta
[Streaming + fragmentación](/es/concepts/streaming).

<div id="link-policy">
  ## Política de enlaces
</div>

* **Slack:** `[label](url)` -&gt; `<url|label>`; las URL sin formato permanecen sin formato. La detección automática de enlaces (autolink)
  está desactivada durante el análisis para evitar enlazarlos dos veces.
* **Telegram:** `[label](url)` -&gt; `<a href="url">label</a>` (modo de análisis HTML).
* **Signal:** `[label](url)` -&gt; `label (url)` a menos que la etiqueta coincida con la URL.

<div id="spoilers">
  ## Spoilers
</div>

Los marcadores de spoiler (`||spoiler||`) solo se procesan en Signal, donde se convierten en
rangos de estilo SPOILER. Otros canales los tratan como texto sin formato.

<div id="how-to-add-or-update-a-channel-formatter">
  ## Cómo agregar o actualizar un formateador de canal
</div>

1. **Analizar una vez:** usa la función auxiliar compartida `markdownToIR(...)` con opciones adecuadas para el canal (autolink, estilo de encabezado, prefijo de cita en bloque).
2. **Renderizar:** implementa un renderizador con `renderMarkdownWithMarkers(...)` y un mapa de marcadores de estilo (o rangos de estilo de Signal).
3. **Segmentar:** llama a `chunkMarkdownIR(...)` antes de renderizar; renderiza cada segmento.
4. **Integrar el adaptador:** actualiza el adaptador de salida del canal para usar el nuevo segmentador y el renderizador.
5. **Probar:** agrega o actualiza tests de formato y un test de entrega saliente si el canal usa segmentación.

<div id="common-gotchas">
  ## Problemas frecuentes
</div>

* Los tokens entre paréntesis angulares de Slack (`<@U123>`, `<#C123>`, `<https://...>`) deben
  conservarse; escapa el HTML sin procesar de forma segura.
* El HTML de Telegram requiere escapar el texto fuera de las etiquetas para evitar que el marcado se rompa.
* Los rangos de estilo de Signal dependen de desplazamientos UTF-16; no uses desplazamientos basados en puntos de código.
* Conserva las nuevas líneas finales en los bloques de código delimitados para que los marcadores de cierre queden en
  su propia línea.