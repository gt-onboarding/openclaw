---
title: Configurer
summary: "Référence CLI pour `openclaw configure` (assistant de configuration interactif)"
read_when:
  - Vous souhaitez ajuster de manière interactive les identifiants, les appareils ou les valeurs par défaut des agents
---

<div id="openclaw-configure">
  # `openclaw configure`
</div>

Assistant interactif de configuration pour définir les identifiants, les appareils et les paramètres par défaut des agents.

Remarque : la section **Model** inclut désormais une sélection multiple pour la
liste d’autorisation `agents.defaults.models` (ce qui apparaît dans `/model` et dans le sélecteur de modèles).

Astuce : `openclaw config` sans sous-commande ouvre le même assistant. Utilisez
`openclaw config get|set|unset` pour des modifications non interactives.

Liens associés :

* Référence de configuration du Gateway : [Configuration](/fr/gateway/configuration)
* CLI de configuration : [Config](/fr/cli/config)

Notes :

* Le choix de l’emplacement d’exécution du Gateway met toujours à jour `gateway.mode`. Vous pouvez sélectionner « Continue » sans remplir les autres sections si c’est tout ce dont vous avez besoin.
* Les services orientés canaux (Slack/Discord/Matrix/Microsoft Teams) demandent des listes d’autorisation de canaux/salons lors de la configuration. Vous pouvez saisir des noms ou des identifiants ; l’assistant résout les noms en identifiants lorsque c’est possible.

<div id="examples">
  ## Exemples
</div>

```bash
openclaw configure
openclaw configure --section models --section channels
```
