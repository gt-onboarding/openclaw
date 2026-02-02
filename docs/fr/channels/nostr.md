---
title: Nostr
summary: "Canal DM Nostr via messages chiffrés NIP-04"
read_when:
  - Vous souhaitez qu’OpenClaw reçoive des DM via Nostr
  - Vous configurez une messagerie décentralisée
---

<div id="nostr">
  # Nostr
</div>

**Statut :** plugin facultatif (désactivé par défaut).

Nostr est un protocole décentralisé pour les réseaux sociaux. Ce canal permet à OpenClaw de recevoir et de répondre à des messages privés (DM) chiffrés via NIP-04.

<div id="install-on-demand">
  ## Installer (à la demande)
</div>

<div id="onboarding-recommended">
  ### Intégration (recommandé)
</div>

- L’assistant d’intégration (`openclaw onboard`) et `openclaw channels add` répertorient les plugins de canal optionnels.
- Si vous sélectionnez Nostr, vous serez invité à installer le plugin à la demande.

Valeurs par défaut de l’installation :

- **Canal de développement + dépôt git disponible :** utilise le chemin local du plugin.
- **Stable/Beta :** télécharge le plugin depuis npm.

Vous pouvez toujours modifier le choix proposé dans l’invite.

<div id="manual-install">
  ### Installation manuelle
</div>

```bash
openclaw plugins install @openclaw/nostr
```

Utilisez un clone local (workflows de développement) :

```bash
openclaw plugins install --link <path-to-openclaw>/extensions/nostr
```

Redémarrez Gateway après avoir installé ou activé des plugins.


<div id="quick-setup">
  ## Configuration rapide
</div>

1. Générez une paire de clés Nostr (si nécessaire) :

```bash
# Utilisation de nak
nak key generate
```

2. Ajoutez à la configuration :

```json
{
  "channels": {
    "nostr": {
      "privateKey": "${NOSTR_PRIVATE_KEY}"
    }
  }
}
```

3. Exportez la clé :

```bash
export NOSTR_PRIVATE_KEY="nsec1..."
```

4. Redémarrez Gateway.


<div id="configuration-reference">
  ## Référence de configuration
</div>

| Clé | Type | Valeur par défaut | Description |
| --- | --- | --- | --- |
| `privateKey` | string | obligatoire | Clé privée au format `nsec` ou hexadécimal |
| `relays` | string[] | `['wss://relay.damus.io', 'wss://nos.lol']` | URL de relais (WebSocket) |
| `dmPolicy` | string | `pairing` | Politique d'accès aux messages privés (DM) |
| `allowFrom` | string[] | `[]` | Clés publiques des expéditeurs autorisés |
| `enabled` | boolean | `true` | Activer/désactiver le canal |
| `name` | string | - | Nom d’affichage |
| `profile` | object | - | Métadonnées de profil NIP-01 |

<div id="profile-metadata">
  ## Métadonnées de profil
</div>

Les données de profil sont publiées sous la forme d’un événement NIP-01 `kind:0`. Vous pouvez les gérer depuis le Control UI (Channels -&gt; Nostr -&gt; Profile) ou les définir directement dans la configuration.

Exemple :

```json
{
  "channels": {
    "nostr": {
      "privateKey": "${NOSTR_PRIVATE_KEY}",
      "profile": {
        "name": "openclaw",
        "displayName": "OpenClaw",
        "about": "Bot assistant personnel par message privé",
        "picture": "https://example.com/avatar.png",
        "banner": "https://example.com/banner.png",
        "website": "https://example.com",
        "nip05": "openclaw@example.com",
        "lud16": "openclaw@example.com"
      }
    }
  }
}
```

Notes :

* Les URL de profil doivent utiliser `https://`.
* L’importation à partir des relais fusionne les champs et préserve les modifications locales.


<div id="access-control">
  ## Contrôle d’accès
</div>

<div id="dm-policies">
  ### Politiques de DM
</div>

- **pairing** (par défaut) : les expéditeurs inconnus reçoivent un code d’appairage.
- **allowlist** : seules les clés publiques répertoriées dans `allowFrom` peuvent envoyer des DM.
- **open** : DM entrants publics (le paramètre « open » autorise l’acceptation sans restriction de messages de n’importe quel utilisateur ; nécessite `allowFrom: ["*"]`).
- **disabled** : ignore les DM entrants.

<div id="allowlist-example">
  ### Exemple de liste d’autorisation
</div>

```json
{
  "channels": {
    "nostr": {
      "privateKey": "${NOSTR_PRIVATE_KEY}",
      "dmPolicy": "allowlist",
      "allowFrom": ["npub1abc...", "npub1xyz..."]
    }
  }
}
```


<div id="key-formats">
  ## Formats de clés
</div>

Formats acceptés :

- **Clé privée :** `nsec...` ou chaîne hexadécimale de 64 caractères
- **Clés publiques (`allowFrom`) :** `npub...` ou valeur hexadécimale

<div id="relays">
  ## Relais
</div>

Valeurs par défaut : `relay.damus.io` et `nos.lol`.

```json
{
  "channels": {
    "nostr": {
      "privateKey": "${NOSTR_PRIVATE_KEY}",
      "relays": [
        "wss://relay.damus.io",
        "wss://relay.primal.net",
        "wss://nostr.wine"
      ]
    }
  }
}
```

Conseils :

* Utilisez 2 à 3 relais pour la redondance.
* Évitez d’utiliser trop de relais (latence, duplication).
* Des relais payants peuvent améliorer la fiabilité.
* Les relais locaux conviennent pour les tests (`ws://localhost:7777`).


<div id="protocol-support">
  ## Prise en charge du protocole
</div>

| NIP | Statut | Description |
| --- | --- | --- |
| NIP-01 | Pris en charge | Format d’événement de base + métadonnées de profil |
| NIP-04 | Pris en charge | DM chiffrés (`kind:4`) |
| NIP-17 | Prévu | DM « gift-wrapped » |
| NIP-44 | Prévu | Chiffrement versionné |

<div id="testing">
  ## Tests
</div>

<div id="local-relay">
  ### Relais local
</div>

```bash
# Démarrer strfry
docker run -p 7777:7777 ghcr.io/hoytech/strfry
```

```json
{
  "channels": {
    "nostr": {
      "privateKey": "${NOSTR_PRIVATE_KEY}",
      "relays": ["ws://localhost:7777"]
    }
  }
}
```


<div id="manual-test">
  ### Test manuel
</div>

1) Notez la clé publique (npub) du bot à partir des logs.
2) Ouvrez un client Nostr (Damus, Amethyst, etc.).
3) Envoyez un message privé (DM) à la clé publique du bot.
4) Vérifiez la réponse.

<div id="troubleshooting">
  ## Dépannage
</div>

<div id="not-receiving-messages">
  ### Vous ne recevez pas de messages
</div>

- Vérifiez que la clé privée est valide.
- Assurez-vous que les URL de relais sont joignables et utilisent `wss://` (ou `ws://` en local).
- Vérifiez que `enabled` n'est pas défini sur `false`.
- Consultez les journaux du Gateway pour d'éventuelles erreurs de connexion au relais.

<div id="not-sending-responses">
  ### Réponses non envoyées
</div>

- Vérifiez que le relais accepte les opérations d'écriture.
- Vérifiez la connectivité sortante.
- Surveillez les limites de débit du relais.

<div id="duplicate-responses">
  ### Réponses en double
</div>

- Comportement attendu lors de l’utilisation de plusieurs relais.
- Les messages sont dédupliés par ID d’événement ; seule la première réception déclenche une réponse.

<div id="security">
  ## Sécurité
</div>

- Ne validez jamais de clés privées.
- Utilisez des variables d’environnement pour les clés.
- Envisagez d’utiliser `allowlist` pour les bots de production.

<div id="limitations-mvp">
  ## Limitations (MVP)
</div>

- Messages privés uniquement (pas de discussions de groupe).
- Aucune pièce jointe multimédia.
- NIP-04 uniquement (emballage-cadeau NIP-17 prévu).