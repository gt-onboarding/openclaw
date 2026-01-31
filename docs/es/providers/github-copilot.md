---
title: Github Copilot
summary: "Inicia sesión en GitHub Copilot desde OpenClaw usando el flujo de dispositivo"
read_when:
  - Quieres usar GitHub Copilot como proveedor de modelos
  - Necesitas usar el flujo `openclaw models auth login-github-copilot`
---

<div id="github-copilot">
  # GitHub Copilot
</div>

<div id="what-is-github-copilot">
  ## ¿Qué es GitHub Copilot?
</div>

GitHub Copilot es el asistente de programación con IA de GitHub. Proporciona acceso a los modelos de Copilot
en tu cuenta y plan de GitHub. OpenClaw puede usar Copilot como proveedor
de modelos de dos formas diferentes.

<div id="two-ways-to-use-copilot-in-openclaw">
  ## Dos maneras de usar Copilot en OpenClaw
</div>

<div id="1-built-in-github-copilot-provider-github-copilot">
  ### 1) Proveedor integrado de GitHub Copilot (`github-copilot`)
</div>

Usa el flujo nativo de inicio de sesión mediante dispositivo para obtener un token de GitHub y luego cámbialo por
tokens de la API de Copilot cuando se ejecute OpenClaw. Esta es la ruta **predeterminada** y más sencilla
porque no requiere VS Code.

<div id="2-copilot-proxy-plugin-copilot-proxy">
  ### 2) Complemento Copilot Proxy (`copilot-proxy`)
</div>

Usa la extensión de VS Code **Copilot Proxy** como puente local. OpenClaw se comunica con
el endpoint `/v1` del proxy y utiliza la lista de modelos que configures allí. Elige
esta opción cuando ya estés ejecutando Copilot Proxy en VS Code o necesites enrutar el tráfico a través de él.
Debes habilitar el complemento y mantener la extensión de VS Code en ejecución.

Usa GitHub Copilot como proveedor de modelos (`github-copilot`). El comando de inicio de sesión ejecuta
el flujo de dispositivo de GitHub, guarda un perfil de autenticación y actualiza tu configuración para usar
ese perfil.

<div id="cli-setup">
  ## Configuración de la CLI
</div>

```bash
openclaw models auth login-github-copilot
```

Se te pedirá que visites una URL e introduzcas un código de un solo uso. Mantén la terminal
abierta hasta que termine.


<div id="optional-flags">
  ### Flags opcionales
</div>

```bash
openclaw models auth login-github-copilot --profile-id github-copilot:work
openclaw models auth login-github-copilot --yes
```


<div id="set-a-default-model">
  ## Configurar un modelo predeterminado
</div>

```bash
openclaw models set github-copilot/gpt-4o
```


<div id="config-snippet">
  ### Fragmento de configuración
</div>

```json5
{
  agents: { defaults: { model: { primary: "github-copilot/gpt-4o" } } }
}
```


<div id="notes">
  ## Notas
</div>

- Requiere un TTY interactivo; ejecútalo directamente en una terminal.
- La disponibilidad del modelo de Copilot depende de tu plan; si un modelo es rechazado, prueba
  con otro ID (por ejemplo, `github-copilot/gpt-4.1`).
- El inicio de sesión almacena un token de GitHub en el almacén de perfiles de autenticación y lo canjea por un
  token de la API de Copilot cuando se ejecuta OpenClaw.