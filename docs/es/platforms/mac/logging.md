---
title: Registro
summary: "Registro en OpenClaw: archivo de registro de diagnóstico con rotación + indicadores de privacidad del sistema de registro unificado"
read_when:
  - Captura de registros de macOS o investigación del registro de datos privados
  - Depuración de problemas del ciclo de vida de la activación por voz o de la sesión
---

<div id="logging-macos">
  # Registros (macOS)
</div>

<div id="rolling-diagnostics-file-log-debug-pane">
  ## Registro de diagnósticos en archivo rotativo (panel Debug)
</div>

OpenClaw enruta los registros de la app de macOS a través de swift-log (registro unificado de forma predeterminada) y puede escribir un registro local en archivo rotativo en disco cuando necesites una captura duradera.

- Verbosidad: **Debug pane → Logs → App logging → Verbosity**
- Activar: **Debug pane → Logs → App logging → “Write rolling diagnostics log (JSONL)”**
- Ubicación: `~/Library/Logs/OpenClaw/diagnostics.jsonl` (rota automáticamente; a los archivos antiguos se les añade el sufijo `.1`, `.2`, …)
- Borrar: **Debug pane → Logs → App logging → “Clear”**

Notas:

- Esto está **desactivado de forma predeterminada**. Actívalo solo mientras estés depurando activamente.
- Trata el archivo como confidencial; no lo compartas sin revisarlo antes.

<div id="unified-logging-private-data-on-macos">
  ## Datos privados de Unified Logging en macOS
</div>

Unified Logging oculta la mayor parte de las cargas de datos, a menos que un subsistema opte por `privacy -off`. Según el artículo de Peter sobre las [triquiñuelas de privacidad de logging en macOS](https://steipete.me/posts/2025/logging-privacy-shenanigans) (2025), esto se controla mediante un plist en `/Library/Preferences/Logging/Subsystems/` cuya clave es el nombre del subsistema. Solo las nuevas entradas de registro recogen el flag, así que actívalo antes de reproducir un problema.

<div id="enable-for-openclaw-botmolt">
  ## Habilitar para OpenClaw (`bot.molt`)
</div>

* Primero escribe el plist en un archivo temporal y luego instálalo de forma atómica como root:

```bash
cat <<'EOF' >/tmp/bot.molt.plist
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>DEFAULT-OPTIONS</key>
    <dict>
        <key>Enable-Private-Data</key>
        <true/>
    </dict>
</dict>
</plist>
EOF
sudo install -m 644 -o root -g wheel /tmp/bot.molt.plist /Library/Preferences/Logging/Subsystems/bot.molt.plist
```

* No es necesario reiniciar; `logd` detecta el archivo rápidamente, pero solo las nuevas líneas de registro incluirán datos privados.
* Consulta la salida más detallada con el helper existente, por ejemplo: `./scripts/clawlog.sh --category WebChat --last 5m`.


<div id="disable-after-debugging">
  ## Desactivar después de depurar
</div>

- Elimina la anulación de configuración: `sudo rm /Library/Preferences/Logging/Subsystems/bot.molt.plist`.
- Opcionalmente ejecuta `sudo log config --reload` para forzar que logd descarte la anulación de inmediato.
- Ten en cuenta que esta salida puede incluir números de teléfono y cuerpos de mensajes; mantén el plist solo mientras necesites activamente ese nivel adicional de detalle.