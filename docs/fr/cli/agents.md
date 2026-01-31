---
title: Agents
summary: "Référence CLI pour `openclaw agents` (lister/ajouter/supprimer/définir l’identité)"
read_when:
  - Vous souhaitez plusieurs agents isolés (espaces de travail + routage + authentification)
---

<div id="openclaw-agents">
  # `openclaw agents`
</div>

Gérer des agents isolés (espaces de travail + authentification + routage).

Contenu associé :

* Routage multi-agents : [Multi-Agent Routing](/fr/concepts/multi-agent)
* Espace de travail d’Agent : [Agent workspace](/fr/concepts/agent-workspace)

<div id="examples">
  ## Exemples
</div>

```bash
openclaw agents list
openclaw agents add work --workspace ~/.openclaw/workspace-work
openclaw agents set-identity --workspace ~/.openclaw/workspace --from-identity
openclaw agents set-identity --agent main --avatar avatars/openclaw.png
openclaw agents delete work
```

<div id="identity-files">
  ## Fichiers d’identité
</div>

Chaque espace de travail d’agent peut inclure un `IDENTITY.md` à la racine de l’espace de travail :

* Exemple de chemin : `~/.openclaw/workspace/IDENTITY.md`
* `set-identity --from-identity` lit à partir de la racine de l’espace de travail (ou d’un `--identity-file` explicite)

Les chemins d’accès aux avatars sont résolus par rapport à la racine de l’espace de travail.

<div id="set-identity">
  ## Définir l’identité
</div>

`set-identity` renseigne les champs de `agents.list[].identity` :

* `name`
* `theme`
* `emoji`
* `avatar` (chemin relatif à l’espace de travail, URL http(s) ou URI de données)

Charger à partir de `IDENTITY.md` :

```bash
openclaw agents set-identity --workspace ~/.openclaw/workspace --from-identity
```

Surchargez explicitement les champs :

```bash
openclaw agents set-identity --agent main --name "OpenClaw" --emoji "🦞" --avatar avatars/openclaw.png
```

Exemple de configuration :

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
