---
title: Processus enfant
summary: "Cycle de vie du Gateway sous macOS (launchd)"
read_when:
  - Intégrer l’application macOS au cycle de vie du Gateway
---

<div id="gateway-lifecycle-on-macos">
  # Cycle de vie du Gateway sur macOS
</div>

L’app macOS **gère le Gateway via launchd** par défaut et ne lance pas
le Gateway comme processus enfant. Elle tente d’abord de se connecter à une instance
Gateway déjà en cours d’exécution sur le port configuré ; si aucune n’est joignable,
elle active le service launchd via le CLI `openclaw` externe (sans runtime intégré).
Cela vous fournit un démarrage automatique fiable à l’ouverture de session et un
redémarrage en cas de plantage.

Le mode processus enfant (Gateway lancé directement par l’app) **n’est pas utilisé**
à ce jour. Si vous avez besoin d’un couplage plus étroit avec l’UI, exécutez le
Gateway manuellement dans un terminal.

<div id="default-behavior-launchd">
  ## Comportement par défaut (launchd)
</div>

* L’app installe un LaunchAgent par utilisateur libellé `bot.molt.gateway`
  (ou `bot.molt.<profile>` lorsque vous utilisez `--profile`/`OPENCLAW_PROFILE` ; l’ancien préfixe `com.openclaw.*` reste pris en charge).
* Lorsque le mode Local est activé, l’app s’assure que le LaunchAgent est chargé et
  démarre le Gateway si nécessaire.
* Les journaux sont écrits dans le chemin de journal launchd du Gateway (visible dans les paramètres de débogage).

Commandes courantes :

```bash
launchctl kickstart -k gui/$UID/bot.molt.gateway
launchctl bootout gui/$UID/bot.molt.gateway
```

Remplacez le libellé par `bot.molt.<profile>` lorsque vous exécutez un profil nommé.


<div id="unsigned-dev-builds">
  ## Builds de développement non signés
</div>

`scripts/restart-mac.sh --no-sign` est destiné aux builds rapides en local lorsque vous n’avez pas
de clés de signature. Pour empêcher launchd de pointer vers un binaire relay non signé, il :

* écrit `~/.openclaw/disable-launchagent`.

Les exécutions signées de `scripts/restart-mac.sh` suppriment cette dérogation si le marqueur est
présent. Pour réinitialiser manuellement :

```bash
rm ~/.openclaw/disable-launchagent
```


<div id="attach-only-mode">
  ## Mode « attach-only »
</div>

Pour forcer l’app macOS à **ne jamais installer ni gérer launchd**, lancez-la avec
`--attach-only` (ou `--no-launchd`). Cela définit `~/.openclaw/disable-launchagent`,
de sorte que l’app ne fait que s’attacher à un Gateway déjà en cours d’exécution. Vous pouvez activer le même
comportement dans les paramètres de débogage.

<div id="remote-mode">
  ## Mode distant
</div>

Le mode distant ne démarre jamais de Gateway local. L'application établit un tunnel SSH vers l'hôte distant et s'y connecte via ce tunnel.

<div id="why-we-prefer-launchd">
  ## Pourquoi nous préférons launchd
</div>

- Démarrage automatique à l’ouverture de session.
- Mécanismes intégrés de redémarrage/KeepAlive.
- Journaux et supervision prévisibles.

Si un véritable mode de processus enfant devait à nouveau être nécessaire, il devrait être documenté comme un mode de développement distinct et explicitement réservé au dev.