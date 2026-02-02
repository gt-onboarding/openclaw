---
title: Localisation
summary: "Analyse de la localisation pour les canaux entrants (Telegram + WhatsApp) et champs de contexte"
read_when:
  - Ajout ou modification de l’analyse de la localisation pour un canal
  - Utilisation des champs de contexte de localisation dans les prompts ou les outils des agents
---

<div id="channel-location-parsing">
  # Analyse des localisations dans les canaux
</div>

OpenClaw normalise les emplacements partagés depuis les canaux de discussion en :

- texte lisible ajouté au corps du message entrant, et
- champs structurés dans la charge utile de contexte de la réponse automatique.

Actuellement pris en charge :

- **Telegram** (épingles de localisation + lieux + partages de position en direct)
- **WhatsApp** (`locationMessage` + `liveLocationMessage`)
- **Matrix** (`m.location` avec `geo_uri`)

<div id="text-formatting">
  ## Mise en forme du texte
</div>

Les emplacements sont affichés sous forme de lignes lisibles sans crochets :

* Épingle :
  * `📍 48.858844, 2.294351 ±12m`
* Lieu identifié :
  * `📍 Eiffel Tower — Champ de Mars, Paris (48.858844, 2.294351 ±12m)`
* Partage de position en direct :
  * `🛰 Position en direct : 48.858844, 2.294351 ±12m`

Si le canal inclut une légende ou un commentaire, celui-ci est ajouté à la ligne suivante :

```
📍 48.858844, 2.294351 ±12m
Rendez-vous ici
```


<div id="context-fields">
  ## Champs de contexte
</div>

Lorsqu'une localisation est présente, ces champs sont ajoutés à `ctx` :

- `LocationLat` (nombre)
- `LocationLon` (nombre)
- `LocationAccuracy` (nombre, en mètres ; facultatif)
- `LocationName` (chaîne de caractères ; facultatif)
- `LocationAddress` (chaîne de caractères ; facultatif)
- `LocationSource` (`pin | place | live`)
- `LocationIsLive` (booléen)

<div id="channel-notes">
  ## Notes sur les canaux
</div>

- **Telegram** : les lieux correspondent à `LocationName/LocationAddress` ; les localisations en direct utilisent `live_period`.
- **WhatsApp** : `locationMessage.comment` et `liveLocationMessage.caption` sont ajoutés comme ligne de légende.
- **Matrix** : `geo_uri` est interprété comme une position épinglée ; l’altitude est ignorée et `LocationIsLive` vaut toujours `false`.