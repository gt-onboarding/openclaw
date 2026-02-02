---
title: Agenti
summary: "Riferimento CLI per `openclaw agents` (elenca/aggiungi/elimina/imposta l'identità)"
read_when:
  - Vuoi gestire più agenti isolati (spazi di lavoro + instradamento + autenticazione)
---

<div id="openclaw-agents">
  # `openclaw agents`
</div>

Gestisci agenti isolati (spazio di lavoro + autenticazione + routing).

Correlati:

* Routing multi-agente: [Routing multi-agente](/it/concepts/multi-agent)
* Spazio di lavoro dell&#39;agente: [Spazio di lavoro dell&#39;agente](/it/concepts/agent-workspace)

<div id="examples">
  ## Esempi
</div>

```bash
openclaw agents list
openclaw agents add work --workspace ~/.openclaw/workspace-work
openclaw agents set-identity --workspace ~/.openclaw/workspace --from-identity
openclaw agents set-identity --agent main --avatar avatars/openclaw.png
openclaw agents delete work
```

<div id="identity-files">
  ## File di identità
</div>

Ogni spazio di lavoro dell&#39;agente può includere un file `IDENTITY.md` nella directory principale dello spazio di lavoro:

* Percorso di esempio: `~/.openclaw/workspace/IDENTITY.md`
* `set-identity --from-identity` legge dalla directory principale dello spazio di lavoro (o da un `--identity-file` esplicito)

I percorsi degli avatar sono interpretati come relativi rispetto alla directory principale dello spazio di lavoro.

<div id="set-identity">
  ## Imposta l&#39;identità
</div>

`set-identity` scrive i campi in `agents.list[].identity`:

* `name`
* `theme`
* `emoji`
* `avatar` (percorso relativo allo spazio di lavoro, URL http(s) o data URI)

Carica da `IDENTITY.md`:

```bash
openclaw agents set-identity --workspace ~/.openclaw/workspace --from-identity
```

Esegui l&#39;override dei campi in modo esplicito:

```bash
openclaw agents set-identity --agent main --name "OpenClaw" --emoji "🦞" --avatar avatars/openclaw.png
```

Esempio di configurazione:

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
