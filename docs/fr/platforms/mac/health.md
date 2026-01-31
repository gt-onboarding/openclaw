---
title: Santé
summary: "Comment l'app macOS signale les états de santé de Gateway/Baileys"
read_when:
  - Débogage des indicateurs d'état de santé de l'app macOS
---

<div id="health-checks-on-macos">
  # Vérifications d’intégrité sur macOS
</div>

Comment vérifier si le canal associé est opérationnel depuis l’app dans la barre des menus.

<div id="menu-bar">
  ## Barre de menu
</div>

* La pastille d’état reflète désormais l’état de santé de Baileys :
  * Vert : lié + socket ouvert récemment.
  * Orange : connexion/nouvelle tentative.
  * Rouge : déconnecté ou sonde en échec.
* La ligne secondaire affiche « linked · auth 12m » ou indique la raison de l’échec.
* L’élément de menu « Run Health Check » déclenche une vérification à la demande.

<div id="settings">
  ## Paramètres
</div>

* L’onglet Général ajoute une carte d’état de santé affichant : l’ancienneté de l’authentification liée, le chemin/compteur de `session-store`, l’heure du dernier contrôle, le dernier code d’erreur/statut et des boutons pour « Run Health Check » / « Reveal Logs ».
* Utilise un instantané mis en cache pour que l’UI se charge instantanément et gère correctement le mode hors ligne.
* L’onglet **Channels** met en évidence l’état des canaux et les contrôles pour WhatsApp/Telegram (QR de connexion, déconnexion, test, dernière déconnexion/erreur).

<div id="how-the-probe-works">
  ## Fonctionnement de la sonde
</div>

* L’app exécute `openclaw health --json` via `ShellExecutor` toutes les ~60 s et à la demande. La sonde charge les identifiants et signale l’état sans envoyer de messages.
* Met en cache le dernier instantané valide et la dernière erreur séparément pour éviter les effets de clignotement ; affiche l’horodatage de chacun.

<div id="when-in-doubt">
  ## En cas de doute
</div>

* Vous pouvez toujours suivre le flux CLI décrit dans [Gateway health](/fr/gateway/health) (`openclaw status`, `openclaw status --deep`, `openclaw health --json`) et exécuter `tail` sur `/tmp/openclaw/openclaw-*.log` pour `web-heartbeat` / `web-reconnect`.