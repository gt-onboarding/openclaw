---
title: Désinstallation
summary: "Désinstaller complètement OpenClaw (CLI, service, état, espace de travail)"
read_when:
  - Vous souhaitez supprimer OpenClaw d'une machine
  - Le service Gateway continue de s'exécuter après la désinstallation
---

<div id="uninstall">
  # Désinstallation
</div>

Deux options :

- **Méthode simple** si `openclaw` est toujours installé.
- **Suppression manuelle du service** si la CLI n’est plus installée mais que le service est toujours en cours d’exécution.

<div id="easy-path-cli-still-installed">
  ## Méthode simple (CLI encore installé)
</div>

Solution recommandée : utilisez le programme de désinstallation intégré :

```bash
openclaw uninstall
```

Mode non interactif (automatisation / npx) :

```bash
openclaw uninstall --all --yes --non-interactive
npx -y openclaw uninstall --all --yes --non-interactive
```

Étapes manuelles (même résultat) :

1. Arrêtez le service Gateway :

```bash
openclaw gateway stop
```

2. Désinstallez le service Gateway (launchd/systemd/schtasks) :

```bash
openclaw gateway uninstall
```

3. Supprimer l&#39;état et la configuration :

```bash
rm -rf "${OPENCLAW_STATE_DIR:-$HOME/.openclaw}"
```

Si vous avez défini `OPENCLAW_CONFIG_PATH` vers un emplacement personnalisé en dehors du répertoire d&#39;état, supprimez également ce fichier.

4. Supprimez votre espace de travail (facultatif, supprime les fichiers d&#39;agent) :

```bash
rm -rf ~/.openclaw/workspace
```

5. Désinstallez la CLI (choisissez la méthode que vous avez utilisée) :

```bash
npm rm -g openclaw
pnpm remove -g openclaw
bun remove -g openclaw
```

6. Si vous avez installé l’app macOS :

```bash
rm -rf /Applications/OpenClaw.app
```

Notes :

* Si vous avez utilisé des profils (`--profile` / `OPENCLAW_PROFILE`), répétez l’étape 3 pour chaque répertoire d’état (les valeurs par défaut sont `~/.openclaw-<profile>`).
* En mode distant, le répertoire d’état se trouve sur l’hôte du **Gateway** ; vous devez donc également y exécuter les étapes 1 à 4.


<div id="manual-service-removal-cli-not-installed">
  ## Désinstallation manuelle du service (CLI non installée)
</div>

Utilisez cette procédure si le service Gateway continue de s’exécuter alors que la commande `openclaw` n’est plus disponible.

<div id="macos-launchd">
  ### macOS (launchd)
</div>

L’étiquette par défaut est `bot.molt.gateway` (ou `bot.molt.<profile>` ; l’ancienne forme `com.openclaw.*` peut encore être présente) :

```bash
launchctl bootout gui/$UID/bot.molt.gateway
rm -f ~/Library/LaunchAgents/bot.molt.gateway.plist
```

Si vous avez utilisé un profil, remplacez le libellé et le nom du plist par `bot.molt.<profile>`. Supprimez tout plist hérité `com.openclaw.*` s&#39;il est présent.


<div id="linux-systemd-user-unit">
  ### Linux (unité utilisateur systemd)
</div>

Le nom de l’unité par défaut est `openclaw-gateway.service` (ou `openclaw-gateway-<profile>.service`) :

```bash
systemctl --user disable --now openclaw-gateway.service
rm -f ~/.config/systemd/user/openclaw-gateway.service
systemctl --user daemon-reload
```


<div id="windows-scheduled-task">
  ### Windows (tâche planifiée)
</div>

Le nom de la tâche par défaut est `OpenClaw Gateway` (ou `OpenClaw Gateway (<profile>)`).
Le script associé à la tâche se trouve dans votre répertoire d’état.

```powershell
schtasks /Delete /F /TN "OpenClaw Gateway"
Remove-Item -Force "$env:USERPROFILE\.openclaw\gateway.cmd"
```

Si vous avez utilisé un profil, supprimez la tâche correspondante ainsi que `~\.openclaw-<profile>\gateway.cmd`.


<div id="normal-install-vs-source-checkout">
  ## Installation classique vs installation depuis les sources
</div>

<div id="normal-install-installsh-npm-pnpm-bun">
  ### Installation standard (install.sh / npm / pnpm / bun)
</div>

Si vous avez utilisé `https://openclaw.bot/install.sh` ou `install.ps1`, la CLI a été installée avec `npm install -g openclaw@latest`.
Supprimez-la avec `npm rm -g openclaw` (ou `pnpm remove -g` / `bun remove -g` si vous l'avez installée de cette manière).

<div id="source-checkout-git-clone">
  ### Source checkout (git clone)
</div>

Si vous exécutez OpenClaw depuis un checkout de dépôt (`git clone` + `openclaw ...` / `bun run openclaw ...`) :

1) Désinstallez le service Gateway **avant** de supprimer le dépôt (utilisez la procédure simplifiée ci‑dessus ou la suppression manuelle du service).
2) Supprimez le répertoire du dépôt.
3) Supprimez l’état et l’espace de travail comme indiqué ci‑dessus.