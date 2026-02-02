---
title: Agenten
summary: "CLI-Referenz für `openclaw agents` (auflisten/hinzufügen/löschen/Identität setzen)"
read_when:
  - Wenn du mehrere isolierte Agenten (Arbeitsbereiche + Routing + Authentifizierung) verwenden möchtest
---

<div id="openclaw-agents">
  # `openclaw agents`
</div>

Isolierte Agenten verwalten (Arbeitsbereiche + Auth + Routing).

Verwandte Themen:

* Multi-Agent-Routing: [Multi-Agent-Routing](/de/concepts/multi-agent)
* Agent-Arbeitsbereich: [Agent-Arbeitsbereich](/de/concepts/agent-workspace)

<div id="examples">
  ## Beispiele
</div>

```bash
openclaw agents list
openclaw agents add work --workspace ~/.openclaw/workspace-work
openclaw agents set-identity --workspace ~/.openclaw/workspace --from-identity
openclaw agents set-identity --agent main --avatar avatars/openclaw.png
openclaw agents delete work
```

<div id="identity-files">
  ## Identitätsdateien
</div>

Jeder Agent-Arbeitsbereich kann eine `IDENTITY.md` im Wurzelverzeichnis des Arbeitsbereichs enthalten:

* Beispielpfad: `~/.openclaw/workspace/IDENTITY.md`
* `set-identity --from-identity` liest aus dem Wurzelverzeichnis des Arbeitsbereichs (oder aus einer explizit angegebenen `--identity-file`)

Avatar-Pfade werden relativ zum Wurzelverzeichnis des Arbeitsbereichs aufgelöst.

<div id="set-identity">
  ## Identität setzen
</div>

`set-identity` schreibt Felder in `agents.list[].identity`:

* `name`
* `theme`
* `emoji`
* `avatar` (arbeitsbereichsrelativer Pfad, http(s)-URL oder Data-URI)

Aus `IDENTITY.md` laden:

```bash
openclaw agents set-identity --workspace ~/.openclaw/workspace --from-identity
```

Felder explizit überschreiben:

```bash
openclaw agents set-identity --agent main --name "OpenClaw" --emoji "🦞" --avatar avatars/openclaw.png
```

Beispielkonfiguration:

```json5
{
  agents: {
    list: [
      {
        id: "main",
        identity: {
          name: "OpenClaw",
          theme: "space lobster",
          emoji: "🦞",
          avatar: "avatars/openclaw.png"
        }
      }
    ]
  }
}
```
