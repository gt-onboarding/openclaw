---
title: Ios
summary: "Aplicación de nodo de iOS: conéctala al Gateway, emparejamiento, canvas y solución de problemas"
read_when:
  - Para emparejar o reconectar el nodo de iOS
  - Para ejecutar la aplicación de iOS desde el código fuente
  - Para depurar el descubrimiento del Gateway o los comandos de canvas
---

<div id="ios-app-node">
  # Aplicación para iOS (nodo)
</div>

Disponibilidad: versión preliminar interna. La aplicación para iOS aún no se distribuye públicamente.

<div id="what-it-does">
  ## Qué hace
</div>

* Se conecta a un Gateway a través de WebSocket (LAN o tailnet).
* Expone capacidades del nodo: Canvas, instantánea de pantalla, captura de cámara, ubicación, modo conversación, activación por voz.
* Recibe comandos `node.invoke` y emite eventos de estado del nodo.

<div id="requirements">
  ## Requisitos
</div>

* Gateway en ejecución en otro dispositivo (macOS, Linux o Windows vía WSL2).
* Conectividad de red:
  * Misma LAN vía Bonjour, **o**
  * Tailnet vía DNS-SD unicast (dominio de ejemplo: `openclaw.internal.`), **o**
  * Host/puerto manual (como alternativa de respaldo).

<div id="quick-start-pair-connect">
  ## Inicio rápido (emparejar + conectar)
</div>

1. Inicia el Gateway:

```bash
openclaw gateway --port 18789
```

2. En la app de iOS, abre Settings y selecciona un Gateway detectado (o activa Manual Host e introduce el host y el puerto).

3. Aprueba la solicitud de emparejamiento en el host del Gateway:

```bash
openclaw nodes pending
openclaw nodes approve <requestId>
```

4. Verifica la conexión:

```bash
openclaw nodes status
openclaw gateway call node.list --params "{}"
```

<div id="discovery-paths">
  ## Métodos de descubrimiento
</div>

<div id="bonjour-lan">
  ### Bonjour (LAN)
</div>

El Gateway anuncia el servicio `_openclaw-gw._tcp` en `local.`. La app para iOS los lista automáticamente.

<div id="tailnet-cross-network">
  ### Tailnet (entre redes)
</div>

Si mDNS está bloqueado, utiliza una zona DNS-SD unicast (elige un dominio; ejemplo: `openclaw.internal.`) y el split DNS de Tailscale.
Consulta [Bonjour](/es/gateway/bonjour) para ver el ejemplo de CoreDNS.

<div id="manual-hostport">
  ### Host/puerto manual
</div>

En Configuración, habilita **Manual Host** e introduce el host y puerto del Gateway (por defecto `18789`).

<div id="canvas-a2ui">
  ## Canvas + A2UI
</div>

El nodo de iOS renderiza un canvas WKWebView. Utiliza `node.invoke` para controlarlo:

```bash
openclaw nodes invoke --node "iOS Node" --command canvas.navigate --params '{"url":"http://<gateway-host>:18793/__openclaw__/canvas/"}'
```

Notas:

* El host de canvas del Gateway sirve `/__openclaw__/canvas/` y `/__openclaw__/a2ui/`.
* El nodo de iOS navega automáticamente a A2UI al conectarse cuando se anuncia una URL del host de canvas.
* Vuelve al scaffold integrado con `canvas.navigate` y `{"url":""}`.

<div id="canvas-eval-snapshot">
  ### Evaluación de Canvas / instantánea
</div>

```bash
openclaw nodes invoke --node "iOS Node" --command canvas.eval --params '{"javaScript":"(() => { const {ctx} = window.__openclaw; ctx.clearRect(0,0,innerWidth,innerHeight); ctx.lineWidth=6; ctx.strokeStyle=\"#ff2d55\"; ctx.beginPath(); ctx.moveTo(40,40); ctx.lineTo(innerWidth-40, innerHeight-40); ctx.stroke(); return \"ok\"; })()"}'
```

```bash
openclaw nodes invoke --node "iOS Node" --command canvas.snapshot --params '{"maxWidth":900,"format":"jpeg"}'
```

<div id="voice-wake-talk-mode">
  ## Activación por voz + modo de conversación
</div>

* La activación por voz y el modo de conversación están disponibles en Ajustes.
* iOS puede suspender el audio en segundo plano; considera las funciones de voz como “en la medida de lo posible” cuando la app no está activa.

<div id="common-errors">
  ## Errores comunes
</div>

* `NODE_BACKGROUND_UNAVAILABLE`: lleva la app de iOS al primer plano (los comandos de canvas/camera/screen lo requieren).
* `A2UI_HOST_NOT_CONFIGURED`: el Gateway no anunció ninguna URL de host de canvas; comprueba `canvasHost` en la [configuración del Gateway](/es/gateway/configuration).
* El aviso de emparejamiento nunca aparece: ejecuta `openclaw nodes pending` y aprueba el nodo manualmente.
* La reconexión falla después de reinstalar: el token de emparejamiento del Keychain se borró; vuelve a emparejar el nodo.

<div id="related-docs">
  ## Documentación relacionada
</div>

* [Emparejamiento](/es/gateway/pairing)
* [Descubrimiento](/es/gateway/discovery)
* [Bonjour](/es/gateway/bonjour)