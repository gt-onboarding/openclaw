---
title: Dépannage
summary: "Centre de dépannage : symptômes → vérifications → correctifs"
read_when:
  - Vous voyez une erreur et cherchez comment la corriger
  - L’installateur indique « success » mais la CLI ne fonctionne pas
---

<div id="troubleshooting">
  # Dépannage
</div>

<div id="first-60-seconds">
  ## Les 60 premières secondes
</div>

Exécute-les dans l’ordre suivant :

```bash
openclaw status
openclaw status --all
openclaw gateway probe
openclaw logs --follow
openclaw doctor
```

Si Gateway est joignable, effectuez des sondes approfondies :

```bash
openclaw status --deep
```

<div id="common-it-broke-cases">
  ## Pannes courantes
</div>

<div id="openclaw-command-not-found">
  ### `openclaw: command not found`
</div>

C’est presque toujours lié au PATH Node/npm. Commencez par ici :

* [Install (Node/npm PATH sanity)](/fr/install#nodejs--npm-path-sanity)

<div id="installer-fails-or-you-need-full-logs">
  ### Le programme d’installation échoue (ou vous avez besoin des logs complets)
</div>

Relancez le programme d’installation en mode verbeux pour voir la trace complète et la sortie npm :

```bash
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --verbose
```

Pour les installations en bêta :

```bash
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --beta --verbose
```

Vous pouvez également définir `OPENCLAW_VERBOSE=1` au lieu du paramètre de ligne de commande.

<div id="gateway-unauthorized-cant-connect-or-keeps-reconnecting">
  ### Gateway « non autorisé », impossible de se connecter ou se reconnecte en boucle
</div>

* [Dépannage de Gateway](/fr/gateway/troubleshooting)
* [Authentification de Gateway](/fr/gateway/authentication)

<div id="control-ui-fails-on-http-device-identity-required">
  ### Échec du Control UI via HTTP (identité de l’appareil requise)
</div>

* [Dépannage du Gateway](/fr/gateway/troubleshooting)
* [Control UI](/fr/web/control-ui#insecure-http)

<div id="docsopenclawai-shows-an-ssl-error-comcastxfinity">
  ### `docs.openclaw.ai` affiche une erreur SSL (Comcast/Xfinity)
</div>

Certaines connexions Comcast/Xfinity bloquent `docs.openclaw.ai` via Xfinity Advanced Security.
Désactivez Advanced Security ou ajoutez `docs.openclaw.ai` à la liste d’autorisation, puis réessayez.

* Aide pour Xfinity Advanced Security : https://www.xfinity.com/support/articles/using-xfinity-xfi-advanced-security
* Vérifications rapides : essayez un partage de connexion mobile ou un VPN pour confirmer qu’il s’agit d’un filtrage au niveau du FAI

<div id="service-says-running-but-rpc-probe-fails">
  ### Le service apparaît comme en cours d’exécution, mais la sonde RPC échoue
</div>

* [Dépannage de Gateway](/fr/gateway/troubleshooting)
* [Processus/service en arrière-plan](/fr/gateway/background-process)

<div id="modelauth-failures-rate-limit-billing-all-models-failed">
  ### Échecs de modèles / d’authentification (limitation de débit, facturation, « tous les modèles ont échoué »)
</div>

* [Modèles](/fr/cli/models)
* [Concepts OAuth / authentification](/fr/concepts/oauth)

<div id="model-says-model-not-allowed">
  ### `/model` affiche `model not allowed`
</div>

Cela signifie généralement que `agents.defaults.models` est configuré comme une liste d’autorisation. Lorsqu’elle n’est pas vide, seules ces clés fournisseur/modèle peuvent être sélectionnées.

* Vérifiez la liste d’autorisation : `openclaw config get agents.defaults.models`
* Ajoutez le modèle souhaité (ou videz la liste d’autorisation), puis réessayez `/model`
* Utilisez `/models` pour parcourir les fournisseurs/modèles autorisés

<div id="when-filing-an-issue">
  ### Lorsque vous créez un ticket
</div>

Collez un rapport sans données sensibles :

```bash
openclaw status --all
```

Si possible, joignez les dernières lignes de logs pertinentes issues de `openclaw logs --follow`.
