---
title: Zalouser
summary: "Prise en charge des comptes personnels Zalo via zca-cli (connexion par QR code), fonctionnalités et configuration"
read_when:
  - Configuration de Zalo Personal avec OpenClaw
  - Débogage de la connexion ou du flux de messages Zalo Personal
---

<div id="zalo-personal-unofficial">
  # Zalo personnel (non officiel)
</div>

Statut : expérimental. Cette intégration automatise un **compte Zalo personnel** via `zca-cli`.

> **Avertissement :** Il s&#39;agit d&#39;une intégration non officielle et cela peut entraîner la suspension ou le bannissement de votre compte. Utilisez-la à vos risques et périls.

<div id="plugin-required">
  ## Plugin requis
</div>

Zalo Personal est fourni sous forme de plugin et n&#39;est pas inclus dans l&#39;installation de base.

* Installez via la CLI : `openclaw plugins install @openclaw/zalouser`
* Ou à partir d&#39;un clone du dépôt : `openclaw plugins install ./extensions/zalouser`
* Détails : [Plugins](/fr/plugin)

<div id="prerequisite-zca-cli">
  ## Prérequis : zca-cli
</div>

La machine hébergeant le Gateway doit avoir le binaire `zca` disponible dans le `PATH`.

* Vérifiez : `zca --version`
* S&#39;il n&#39;est pas présent, installez zca-cli (voir `extensions/zalouser/README.md` ou la documentation officielle de zca-cli).

<div id="quick-setup-beginner">
  ## Configuration rapide (débutant)
</div>

1. Installez le plugin (voir ci-dessus).
2. Connectez-vous (QR, sur la machine exécutant le Gateway) :
   * `openclaw channels login --channel zalouser`
   * Scannez le code QR affiché dans le terminal avec l’application mobile Zalo.
3. Activez le canal :

```json5
{
  channels: {
    zalouser: {
      enabled: true,
      dmPolicy: "appairage"
    }
  }
}
```

4. Redémarrez Gateway (ou terminez l’onboarding).
5. Par défaut, l’accès par DM passe par l’appairage ; approuvez le code d’appairage lors du premier contact.

<div id="what-it-is">
  ## Présentation
</div>

* Utilise `zca listen` pour recevoir les messages entrants.
* Utilise `zca msg ...` pour envoyer des réponses (texte/média/lien).
* Conçu pour des cas d’usage de « compte personnel » où l’API Zalo Bot n’est pas disponible.

<div id="naming">
  ## Convention de nommage
</div>

L&#39;identifiant de canal est `zalouser` afin d&#39;indiquer clairement qu&#39;il automatise un **compte utilisateur Zalo personnel** (non officiel). Nous gardons `zalo` réservé pour une éventuelle future intégration à l&#39;API Zalo officielle.

<div id="finding-ids-directory">
  ## Recherche des ID (annuaire)
</div>

Utilisez la CLI de l’annuaire pour découvrir les pairs et groupes ainsi que leurs ID :

```bash
openclaw directory self --channel zalouser
openclaw directory peers list --channel zalouser --query "name"
openclaw directory groups list --channel zalouser --query "work"
```

<div id="limits">
  ## Limites
</div>

* Le texte envoyé est segmenté en blocs d’environ 2000 caractères (limites du client Zalo).
* Le streaming est désactivé par défaut.

<div id="access-control-dms">
  ## Contrôle d’accès (DM)
</div>

`channels.zalouser.dmPolicy` prend les valeurs suivantes : `pairing | allowlist | open | disabled` (où `open` indique que ce réglage autorise l’acceptation de messages sans restriction de la part de n’importe quel utilisateur ; valeur par défaut : `pairing`).
`channels.zalouser.allowFrom` accepte des identifiants ou des noms d’utilisateurs. L’assistant de configuration convertit les noms en identifiants via `zca friend find` lorsque c’est possible.

Approuver via :

* `openclaw pairing list zalouser`
* `openclaw pairing approve zalouser <code>`

<div id="group-access-optional">
  ## Accès aux groupes (optionnel)
</div>

* Valeur par défaut : `channels.zalouser.groupPolicy = "open"` (groupes autorisés, acceptation sans restriction de messages provenant de n’importe quel groupe). Utilisez `channels.defaults.groupPolicy` pour remplacer cette valeur par défaut lorsqu’aucune valeur n’est définie.
* Restreindre à une liste d’autorisation avec :
  * `channels.zalouser.groupPolicy = "allowlist"`
  * `channels.zalouser.groups` (les clés sont des ID ou des noms de groupes)
* Bloquer tous les groupes : `channels.zalouser.groupPolicy = "disabled"`.
* L’assistant de configuration peut demander des listes d’autorisation de groupes.
* Au démarrage, OpenClaw résout les noms de groupes/utilisateurs présents dans les listes d’autorisation en identifiants et journalise la correspondance ; les entrées non résolues sont conservées telles que saisies.

Exemple :

```json5
{
  channels: {
    zalouser: {
      groupPolicy: "allowlist",
      groups: {
        "123456789": { allow: true },
        "Work Chat": { allow: true }
      }
    }
  }
}
```

<div id="multi-account">
  ## Multi-comptes
</div>

Les comptes correspondent aux profils zca. Exemple :

```json5
{
  channels: {
    zalouser: {
      enabled: true,
      defaultAccount: "default",
      accounts: {
        work: { enabled: true, profile: "work" }
      }
    }
  }
}
```

<div id="troubleshooting">
  ## Dépannage
</div>

**`zca` introuvable :**

* Installez zca-cli et vérifiez qu’il est présent dans le `PATH` du processus Gateway.

**La connexion ne reste pas active :**

* `openclaw channels status --probe`
* Reconnectez-vous : `openclaw channels logout --channel zalouser && openclaw channels login --channel zalouser`