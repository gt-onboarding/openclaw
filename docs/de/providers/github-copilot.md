---
title: GitHub Copilot
summary: "Melde dich aus OpenClaw heraus über den Device-Flow bei GitHub Copilot an"
read_when:
  - Du möchtest GitHub Copilot als Modellanbieter verwenden
  - Du benötigst den `openclaw models auth login-github-copilot` Flow
---

<div id="github-copilot">
  # GitHub Copilot
</div>

<div id="what-is-github-copilot">
  ## Was ist GitHub Copilot?
</div>

GitHub Copilot ist der KI-Coding-Assistent von GitHub. Er bietet Zugriff auf
Copilot-Modelle für dein GitHub-Konto und deinen Tarif. OpenClaw kann Copilot
auf zwei verschiedene Arten als Modellanbieter nutzen.

<div id="two-ways-to-use-copilot-in-openclaw">
  ## Zwei Möglichkeiten, Copilot in OpenClaw zu nutzen
</div>

<div id="1-built-in-github-copilot-provider-github-copilot">
  ### 1) Integrierter GitHub-Copilot-Anbieter (`github-copilot`)
</div>

Verwende den nativen Device-Login-Flow, um ein GitHub-Token zu erhalten, und tausche es dann während der Ausführung von OpenClaw gegen Copilot-API-Tokens ein. Dies ist der **Standardpfad** und zugleich der einfachste, da du dafür kein VS Code benötigst.

<div id="2-copilot-proxy-plugin-copilot-proxy">
  ### 2) Copilot Proxy Plugin (`copilot-proxy`)
</div>

Verwende die **Copilot Proxy**-VS-Code-Erweiterung als lokale Brücke. OpenClaw kommuniziert mit
dem `/v1`-Endpunkt des Proxys und verwendet die Modellliste, die du dort konfigurierst. Wähle
diese Option, wenn du Copilot Proxy bereits in VS Code verwendest oder den Traffic darüber routen musst.
Du musst das Plugin aktivieren und die VS-Code-Erweiterung dauerhaft laufen lassen.

Verwende GitHub Copilot als Modellanbieter (`github-copilot`). Der Anmeldebefehl startet
den GitHub Device Flow, speichert ein Auth-Profil und aktualisiert deine Konfiguration so,
dass dieses Profil verwendet wird.

<div id="cli-setup">
  ## CLI-Setup
</div>

```bash
openclaw models auth login-github-copilot
```

Du wirst aufgefordert, eine URL zu öffnen und einen einmaligen Code einzugeben. Lass das Terminal
geöffnet, bis der Vorgang abgeschlossen ist.


<div id="optional-flags">
  ### Optionale Flags
</div>

```bash
openclaw models auth login-github-copilot --profile-id github-copilot:work
openclaw models auth login-github-copilot --yes
```


<div id="set-a-default-model">
  ## Standardmodell festlegen
</div>

```bash
openclaw models set github-copilot/gpt-4o
```


<div id="config-snippet">
  ### Konfigurationsausschnitt
</div>

```json5
{
  agents: { defaults: { model: { primary: "github-copilot/gpt-4o" } } }
}
```


<div id="notes">
  ## Hinweise
</div>

- Erfordert ein interaktives TTY; starte es direkt im Terminal.
- Die Verfügbarkeit von Copilot-Modellen hängt von deinem Plan ab; wenn ein Modell abgelehnt wird, probiere
  eine andere ID (zum Beispiel `github-copilot/gpt-4.1`) aus.
- Beim Anmelden wird ein GitHub-Token im Auth-Profil-Speicher gespeichert und zur Laufzeit von OpenClaw gegen ein
  Copilot-API-Token eingetauscht.