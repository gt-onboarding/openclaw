---
title: Santé
summary: "Étapes de contrôle de la santé de la connectivité des canaux"
read_when:
  - Diagnostic de la santé du canal WhatsApp
---

<div id="health-checks-cli">
  # Vérifications d'état (CLI)
</div>

Bref guide pour vérifier la connectivité de vos canaux sans avoir à deviner.

<div id="quick-checks">
  ## Vérifications rapides
</div>

- `openclaw status` — récapitulatif local : accessibilité et mode du Gateway, indication de mise à jour, ancienneté de l’authentification des canaux liés, sessions + activité récente.
- `openclaw status --all` — diagnostic local complet (en lecture seule, avec couleurs, sans risque à copier-coller pour le débogage).
- `openclaw status --deep` — sonde également le Gateway en cours d’exécution (sondes par canal lorsque c’est pris en charge).
- `openclaw health --json` — demande au Gateway en cours d’exécution un instantané complet d’état de santé (WS uniquement ; aucun socket Baileys direct).
- Envoyez `/status` comme message seul dans WhatsApp/WebChat pour obtenir une réponse d’état sans invoquer l’agent.
- Logs : suivez `/tmp/openclaw/openclaw-*.log` et filtrez par `web-heartbeat`, `web-reconnect`, `web-auto-reply`, `web-inbound`.

<div id="deep-diagnostics">
  ## Diagnostics détaillés
</div>

- Identifiants sur disque : `ls -l ~/.openclaw/credentials/whatsapp/<accountId>/creds.json` (`mtime` doit être récent).
- Stockage des sessions : `ls -l ~/.openclaw/agents/<agentId>/sessions/sessions.json` (le chemin peut être surchargé dans la config). Le nombre et les destinataires récents sont visibles via `status`.
- Flux de reconnexion : `openclaw channels logout && openclaw channels login --verbose` lorsque les codes d’état 409–515 ou `loggedOut` apparaissent dans les journaux. (Remarque : le flux de connexion via QR code redémarre automatiquement une fois pour le statut 515 après l’appairage.)

<div id="when-something-fails">
  ## En cas de problème
</div>

- `logged out` ou statut 409–515 → reliez à nouveau avec `openclaw channels logout` puis `openclaw channels login`.
- Gateway injoignable → démarrez-la : `openclaw gateway --port 18789` (utilisez `--force` si le port est déjà utilisé).
- Aucun message entrant → vérifiez que le téléphone associé est en ligne et que l’expéditeur est autorisé (`channels.whatsapp.allowFrom`) ; pour les discussions de groupe, assurez-vous que la liste d’autorisation et les règles de mention correspondent (`channels.whatsapp.groups`, `agents.list[].groupChat.mentionPatterns`).

<div id="dedicated-health-command">
  ## Commande dédiée « health »
</div>

`openclaw health --json` demande au Gateway en cours d'exécution de fournir un instantané de son état de santé (sans ouvrir de socket de canal direct depuis la CLI). Il indique, lorsque c'est disponible, l'ancienneté des identifiants / informations d'authentification liées, des résumés de sondes par canal, un résumé du stockage des sessions, ainsi que la durée de la sonde. Il renvoie un code de sortie non nul si le Gateway est injoignable ou si la sonde échoue ou expire. Utilisez `--timeout <ms>` pour remplacer la valeur par défaut de 10 s.