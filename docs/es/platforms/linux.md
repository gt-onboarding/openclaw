---
title: Linux
summary: "Compatibilidad con Linux y estado de la aplicación complementaria"
read_when:
  - Buscando el estado de la aplicación complementaria de Linux
  - Planificando la cobertura de plataformas o contribuciones
---

<div id="linux-app">
  # Aplicación para Linux
</div>

El Gateway es totalmente compatible con Linux. **Node es el entorno de ejecución recomendado**.
No se recomienda usar Bun con el Gateway (errores con WhatsApp/Telegram).

Están previstas aplicaciones nativas de acompañamiento para Linux. Las contribuciones son bienvenidas si quieres ayudar a desarrollar una.

<div id="beginner-quick-path-vps">
  ## Ruta rápida para principiantes (VPS)
</div>

1. Instala Node.js 22 o superior
2. `npm i -g openclaw@latest`
3. `openclaw onboard --install-daemon`
4. Desde tu portátil: `ssh -N -L 18789:127.0.0.1:18789 <user>@<host>`
5. Abre `http://127.0.0.1:18789/` y pega tu token

Guía VPS paso a paso: [exe.dev](/es/platforms/exe-dev)

<div id="install">
  ## Instalar
</div>

* [Primeros pasos](/es/start/getting-started)
* [Instalación y actualizaciones](/es/install/updating)
* Flujos opcionales: [Bun (experimental)](/es/install/bun), [Nix](/es/install/nix), [Docker](/es/install/docker)

<div id="gateway">
  ## Gateway
</div>

* [Runbook del Gateway](/es/gateway)
* [Configuración](/es/gateway/configuration)

<div id="gateway-service-install-cli">
  ## Instalación del servicio Gateway (CLI)
</div>

Utiliza uno de los siguientes métodos:

```
openclaw onboard --install-daemon
```

O bien:

```
openclaw gateway install
```

O bien:

```
openclaw configure
```

Selecciona **Gateway service** cuando se te solicite.

Reparar/migrar:

```
openclaw doctor
```

<div id="system-control-systemd-user-unit">
  ## Control del sistema (unidad de usuario de systemd)
</div>

OpenClaw instala de forma predeterminada un servicio **de usuario** de systemd. Usa un servicio
**de sistema** para servidores compartidos o siempre encendidos. El ejemplo completo
de la unidad y las instrucciones se encuentran en el [runbook del Gateway](/es/gateway).

Configuración mínima:

Crea `~/.config/systemd/user/openclaw-gateway[-<profile>].service`:

```
[Unit]
Description=OpenClaw Gateway (profile: <profile>, v<version>)
After=network-online.target
Wants=network-online.target

[Service]
ExecStart=/usr/local/bin/openclaw gateway --port 18789
Restart=always
RestartSec=5

[Install]
WantedBy=default.target
```

Habilitarla:

```
systemctl --user enable --now openclaw-gateway[-<profile>].service
```
