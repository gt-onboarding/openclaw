---
title: Permisos
summary: "Persistencia de permisos en macOS (TCC) y requisitos de firma"
read_when:
  - Depuración de solicitudes de permiso de macOS que no aparecen o se quedan bloqueadas
  - Empaquetar o firmar la app de macOS
  - Cambiar identificadores de bundle o rutas de instalación de la app
---

<div id="macos-permissions-tcc">
  # Permisos de macOS (TCC)
</div>

Las autorizaciones de permisos en macOS son frágiles. TCC asocia cada autorización de permiso con la firma de código de la app, el identificador del bundle y la ruta en disco. Si cualquiera de esos elementos cambia, macOS trata la app como nueva y puede descartar u ocultar los diálogos de solicitud de permiso.

<div id="requirements-for-stable-permissions">
  ## Requisitos para permisos estables
</div>

- Misma ruta: ejecuta la app desde una ubicación fija (para OpenClaw, `dist/OpenClaw.app`).
- Mismo identificador de bundle: cambiar el bundle ID crea una nueva identidad de permisos.
- App firmada: las compilaciones sin firmar o firmadas ad hoc no conservan los permisos.
- Firma consistente: usa un certificado real de Apple Development o Developer ID
  para que la firma se mantenga estable entre recompilaciones.

Las firmas ad hoc generan una nueva identidad en cada compilación. macOS olvidará las
concesiones anteriores y los avisos pueden desaparecer por completo hasta que se
eliminen las entradas obsoletas.

<div id="recovery-checklist-when-prompts-disappear">
  ## Lista de comprobación de recuperación cuando desaparecen las solicitudes de permiso
</div>

1. Cierra la app.
2. Elimina la entrada de la app en Ajustes del Sistema -&gt; Privacidad y seguridad.
3. Vuelve a abrir la app desde la misma ruta y vuelve a conceder los permisos.
4. Si la solicitud de permiso aún no aparece, restablece las entradas de TCC con `tccutil` e inténtalo de nuevo.
5. Algunos permisos solo vuelven a aparecer después de reiniciar por completo macOS.

Restablecimientos de ejemplo (sustituye el identificador de bundle según sea necesario):

```bash
sudo tccutil reset Accessibility bot.molt.mac
sudo tccutil reset ScreenCapture bot.molt.mac
sudo tccutil reset AppleEvents
```

Si estás probando permisos, firma siempre con un certificado real. Las compilaciones ad‑hoc solo resultan aceptables para ejecuciones locales rápidas donde los permisos no son relevantes.
