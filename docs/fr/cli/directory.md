---
title: Annuaire
summary: "Référence de la CLI pour `openclaw directory` (self, peers, groups)"
read_when:
  - Vous souhaitez rechercher les identifiants de contacts/groupes/self pour un canal
  - Vous développez un adaptateur d’annuaire de canal
---

<div id="openclaw-directory">
  # `openclaw directory`
</div>

Recherches dans l’annuaire pour les canaux qui prennent en charge cette fonctionnalité (contacts/pairs, groupes et « moi »).

<div id="common-flags">
  ## Options courantes
</div>

- `--channel <name>` : ID ou alias du canal (obligatoire lorsque plusieurs canaux sont configurés ; automatique lorsqu’un seul est configuré)
- `--account <id>` : ID de compte (par défaut : valeur par défaut du canal)
- `--json` : sortie au format JSON

<div id="notes">
  ## Remarques
</div>

- `directory` est conçu pour vous aider à trouver des ID que vous pouvez coller dans d’autres commandes (en particulier `openclaw message send --target ...`).
- Pour de nombreux canaux, les résultats sont pilotés par la configuration (listes d’autorisation / groupes configurés) plutôt que par un annuaire de fournisseur en temps réel.
- La sortie par défaut est composée de `id` (et parfois `name`) séparés par une tabulation&nbsp;: utilisez `--json` pour les scripts.

<div id="using-results-with-message-send">
  ## Utiliser les résultats avec `message send`
</div>

```bash
openclaw directory peers list --channel slack --query "U0"
openclaw message send --channel slack --target user:U012ABCDEF --message "hello"
```


<div id="id-formats-by-channel">
  ## Formats d’ID (par canal)
</div>

- WhatsApp : `+15551234567` (MP), `1234567890-1234567890@g.us` (groupe)
- Telegram : `@username` ou identifiant de discussion numérique ; les groupes utilisent des identifiants numériques
- Slack : `user:U…` et `channel:C…`
- Discord : `user:<id>` et `channel:<id>`
- Matrix (plugin) : `user:@user:server`, `room:!roomId:server` ou `#alias:server`
- Microsoft Teams (plugin) : `user:<id>` et `conversation:<id>`
- Zalo (plugin) : identifiant utilisateur (Bot API)
- Zalo Personal / `zalouser` (plugin) : identifiant de fil de discussion (MP/groupe) depuis `zca` (`me`, `friend list`, `group list`)

<div id="self-me">
  ## Moi (« me »)
</div>

```bash
openclaw directory self --channel zalouser
```


<div id="peers-contactsusers">
  ## Pairs (contacts/utilisateurs)
</div>

```bash
openclaw directory peers list --channel zalouser
openclaw directory peers list --channel zalouser --query "name"
openclaw directory peers list --channel zalouser --limit 50
```


<div id="groups">
  ## Groupes
</div>

```bash
openclaw directory groups list --channel zalouser
openclaw directory groups list --channel zalouser --query "work"
openclaw directory groups members --channel zalouser --group-id <id>
```
