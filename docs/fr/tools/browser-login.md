---
title: Connexion au navigateur
summary: "Connexions manuelles pour l'automatisation du navigateur + publication sur X/Twitter"
read_when:
  - Vous avez besoin de vous connecter à des sites web pour l'automatisation du navigateur
  - Vous voulez publier des mises à jour sur X/Twitter
---

<div id="browser-login-xtwitter-posting">
  # Connexion via le navigateur + publication sur X / Twitter
</div>

<div id="manual-login-recommended">
  ## Connexion manuelle (recommandée)
</div>

Lorsqu’un site nécessite une connexion, **connectez‑vous manuellement** dans le profil de navigateur de l’**hôte** (le navigateur openclaw).

Ne communiquez **pas** vos identifiants au modèle. Les connexions automatisées peuvent souvent déclencher des défenses anti‑bots et verrouiller votre compte.

Retour à la documentation principale du navigateur : [Browser](/fr/tools/browser).

<div id="which-chrome-profile-is-used">
  ## Quel profil Chrome est utilisé ?
</div>

OpenClaw contrôle un **profil Chrome dédié** (nommé `openclaw`, avec une UI à teinte orangée). Il est distinct de votre profil de navigation habituel.

Deux moyens simples d’accéder à ce profil :

1. **Demandez à l’agent d’ouvrir le navigateur**, puis connectez‑vous vous‑même.
2. **Ouvrez‑le via la CLI** :

```bash
openclaw browser start
openclaw browser open https://x.com
```

Si vous avez plusieurs profils, utilisez `--browser-profile <name>` (par défaut : `openclaw`).

<div id="xtwitter-recommended-flow">
  ## X/Twitter : workflow recommandé
</div>

* **Lecture/recherche/threads :** utilisez le skill CLI **bird** (sans navigateur, stable).
  * Dépôt : https://github.com/steipete/bird
* **Publication de mises à jour :** utilisez le navigateur **host** (connexion manuelle).

<div id="sandboxing-host-browser-access">
  ## Sandboxing et accès au navigateur de l’hôte
</div>

Les sessions de navigateur en sandbox sont **plus susceptibles** de déclencher la détection de bots. Pour X/Twitter (et d’autres sites stricts), privilégiez le navigateur **hôte**.

Si l’agent est en sandbox, l’outil de navigation utilise par défaut la sandbox. Pour autoriser le contrôle par l’hôte :

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main",
        browser: {
          allowHostControl: true
        }
      }
    }
  }
}
```

Ciblez ensuite le navigateur de l’hôte :

```bash
openclaw browser open https://x.com --browser-profile openclaw --target host
```

Ou désactivez la sandbox pour l’agent qui poste les mises à jour.
