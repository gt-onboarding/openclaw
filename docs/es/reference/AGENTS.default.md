---
title: AGENTS.default
summary: "Instrucciones predeterminadas del agente de OpenClaw y catálogo de habilidades para la configuración del asistente personal"
read_when:
  - Al iniciar una nueva sesión de agente de OpenClaw
  - Al habilitar o auditar habilidades predeterminadas
---

<div id="agentsmd-openclaw-personal-assistant-default">
  # AGENTS.md — Asistente personal de OpenClaw (por defecto)
</div>

<div id="first-run-recommended">
  ## Primera ejecución (recomendado)
</div>

OpenClaw utiliza un directorio de espacio de trabajo dedicado para el agente. Valor predeterminado: `~/.openclaw/workspace` (configurable mediante `agents.defaults.workspace`).

1. Crea el espacio de trabajo (si aún no existe):

```bash
mkdir -p ~/.openclaw/workspace
```

2. Copia las plantillas de espacio de trabajo predeterminadas al espacio de trabajo:

```bash
cp docs/reference/templates/AGENTS.md ~/.openclaw/workspace/AGENTS.md
cp docs/reference/templates/SOUL.md ~/.openclaw/workspace/SOUL.md
cp docs/reference/templates/TOOLS.md ~/.openclaw/workspace/TOOLS.md
```

3. Opcional: si quieres la lista de habilidades del asistente personal, reemplaza AGENTS.md por este archivo:

```bash
cp docs/reference/AGENTS.default.md ~/.openclaw/workspace/AGENTS.md
```

4. Opcional: selecciona un espacio de trabajo distinto configurando `agents.defaults.workspace` (admite `~`):

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } }
}
```


<div id="safety-defaults">
  ## Valores predeterminados de seguridad
</div>

- No envíes directorios ni secretos al chat.
- No ejecutes comandos destructivos a menos que se te pida explícitamente.
- No envíes respuestas parciales/en streaming a plataformas de mensajería externas (solo respuestas finales).

<div id="session-start-required">
  ## Inicio de la sesión (obligatorio)
</div>

- Lee `SOUL.md`, `USER.md`, `memory.md` y los archivos de hoy y de ayer en `memory/`.
- Hazlo antes de responder.

<div id="soul-required">
  ## Alma (obligatoria)
</div>

- `SOUL.md` define la identidad, el tono y los límites. Manténlo actualizado.
- Si cambias `SOUL.md`, díselo al usuario.
- Eres una instancia nueva en cada sesión; la continuidad se conserva en estos archivos.

<div id="shared-spaces-recommended">
  ## Espacios compartidos (recomendado)
</div>

- No eres la voz del usuario; ten cuidado en chats grupales o canales públicos.
- No compartas datos privados, información de contacto ni notas internas.

<div id="memory-system-recommended">
  ## Sistema de memoria (recomendado)
</div>

- Registro diario: `memory/YYYY-MM-DD.md` (crea `memory/` si es necesario).
- Memoria a largo plazo: `memory.md` para hechos duraderos, preferencias y decisiones.
- Al inicio de la sesión, ejecuta read sobre hoy + ayer + `memory.md` si existe.
- Captura: decisiones, preferencias, restricciones, frentes abiertos.
- Evita almacenar secretos salvo que se solicite explícitamente.

<div id="tools-skills">
  ## Herramientas y habilidades
</div>

- Las herramientas se almacenan en las habilidades; consulta el `SKILL.md` de cada habilidad cuando lo necesites.
- Mantén las notas específicas del entorno en `TOOLS.md` (Notas para habilidades).

<div id="backup-tip-recommended">
  ## Consejo sobre copias de seguridad (recomendado)
</div>

Si tratas este espacio de trabajo como la “memoria” de Clawd, conviértelo en un repositorio de Git (preferiblemente privado) para que `AGENTS.md` y tus archivos de memoria queden respaldados.

```bash
cd ~/.openclaw/workspace
git init
git add AGENTS.md
git commit -m "Add Clawd workspace"
# Opcional: añadir un remoto privado y hacer push
```


<div id="what-openclaw-does">
  ## Qué hace OpenClaw
</div>

- Ejecuta el Gateway de WhatsApp y el agente de programación en Pi para que el asistente pueda leer/escribir chats, obtener contexto y ejecutar habilidades a través del Mac anfitrión.
- La app de macOS gestiona permisos (grabación de pantalla, notificaciones, micrófono) y expone la CLI `openclaw` a través de su binario incluido.
- Los chats directos se consolidan en la sesión `main` del agente de forma predeterminada; los grupos se mantienen aislados como `agent:<agentId>:<channel>:group:<id>` (salas/canales: `agent:<agentId>:<channel>:channel:<id>`); los latidos mantienen activas las tareas en segundo plano.

<div id="core-skills-enable-in-settings-skills">
  ## Habilidades principales (activa en Configuración → Habilidades)
</div>

- **mcporter** — Runtime/CLI de servidor de herramientas para gestionar backends de habilidades externas.
- **Peekaboo** — Capturas de pantalla rápidas en macOS con análisis opcional de visión por IA.
- **camsnap** — Captura fotogramas, clips o alertas de movimiento desde cámaras de seguridad RTSP/ONVIF.
- **oracle** — CLI de Agente preparada para OpenAI con repetición de sesiones y control del navegador.
- **eightctl** — Controla tu sueño desde la terminal.
- **imsg** — Envía, read y transmite iMessage y SMS.
- **wacli** — CLI de WhatsApp: sincroniza, busca, send.
- **discord** — Acciones de Discord: reaccionar, stickers, encuestas. Usa objetivos `user:<id>` o `channel:<id>` (los ID numéricos sin prefijo son ambiguos).
- **gog** — CLI de Google Suite: Gmail, Calendar, Drive, Contacts.
- **spotify-player** — Cliente de Spotify en la terminal para buscar, poner en cola y controlar la reproducción.
- **sag** — Voz de ElevenLabs con experiencia de uso tipo `say` de macOS; transmite a los altavoces por defecto.
- **Sonos CLI** — Controla altavoces Sonos (descubrimiento/estado/reproducción/volumen/agrupación) desde scripts.
- **blucli** — Reproduce, agrupa y automatiza reproductores BluOS desde scripts.
- **OpenHue CLI** — Control de iluminación Philips Hue para escenas y automatizaciones.
- **OpenAI Whisper** — Reconocimiento de voz a texto local para dictado rápido y transcripciones de buzón de voz.
- **Gemini CLI** — Modelos de Google Gemini desde la terminal para preguntas y respuestas rápidas.
- **bird** — CLI de X/Twitter para tuitear, responder, read hilos y buscar sin navegador.
- **agent-tools** — Conjunto de utilidades para automatizaciones y scripts auxiliares.

<div id="usage-notes">
  ## Notas de uso
</div>

- Prefiere usar la CLI `openclaw` para scripting; la app de macOS gestiona los permisos.
- Ejecuta las instalaciones desde la pestaña Skills; oculta el botón si ya hay un binario presente.
- Mantén los latidos habilitados para que el asistente pueda programar recordatorios, supervisar bandejas de entrada y activar capturas con la cámara.
- La UI de Canvas se ejecuta a pantalla completa con superposiciones nativas. Evita colocar controles críticos en las esquinas superior izquierda/derecha ni en el borde inferior; añade márgenes explícitos en el diseño y no confíes en los safe-area insets del sistema.
- Para verificación basada en navegador, usa `openclaw browser` (tabs/status/screenshot) con el perfil de Chrome gestionado por OpenClaw.
- Para inspección del DOM, usa `openclaw browser eval|query|dom|snapshot` (y `--json`/`--out` cuando necesites salida legible por máquina).
- Para interacciones, usa `openclaw browser click|type|hover|drag|select|upload|press|wait|navigate|back|evaluate|run` (click/type requieren referencias al snapshot; usa `evaluate` para selectores CSS).