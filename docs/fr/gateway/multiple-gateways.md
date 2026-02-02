---
title: Plusieurs Gateways
summary: "Exécuter plusieurs Gateways OpenClaw sur un seul hôte (isolation, ports et profils)"
read_when:
  - Exécuter plusieurs Gateways sur la même machine
  - Vous avez besoin de config/état/ports isolés pour chaque Gateway
---

<div id="multiple-gateways-same-host">
  # Plusieurs Gateways (même hôte)
</div>

La plupart des configurations devraient utiliser une seule Gateway, car une Gateway unique peut gérer plusieurs connexions de messagerie et plusieurs agents. Si vous avez besoin d’une isolation ou d’une redondance plus forte (par exemple, un bot de secours), exécutez des Gateways distinctes avec des profils et des ports isolés.

<div id="isolation-checklist-required">
  ## Liste de vérification d’isolation (obligatoire)
</div>

- `OPENCLAW_CONFIG_PATH` — fichier de configuration par instance
- `OPENCLAW_STATE_DIR` — sessions, identifiants et caches par instance
- `agents.defaults.workspace` — racine de l’espace de travail par instance
- `gateway.port` (ou `--port`) — unique par instance
- Les ports dérivés (navigateur/canvas) doivent être distincts

S’ils sont partagés, vous aurez des conditions de concurrence sur la configuration et des conflits de ports.

<div id="recommended-profiles-profile">
  ## Recommandé : profils (`--profile`)
</div>

Les profils définissent automatiquement la portée de `OPENCLAW_STATE_DIR` + `OPENCLAW_CONFIG_PATH` et suffixent les noms de services.

```bash
# main
openclaw --profile main setup
openclaw --profile main gateway --port 18789

# rescue
openclaw --profile rescue setup
openclaw --profile rescue gateway --port 19001
```

Services par profil :

```bash
openclaw --profile main gateway install
openclaw --profile rescue gateway install
```


<div id="rescue-bot-guide">
  ## Guide du bot de secours
</div>

Exécutez un deuxième Gateway sur le même hôte avec :

- profil/configuration
- répertoire d’état
- espace de travail
- port de base (plus ports dérivés)

Cela permet de garder le bot de secours isolé du bot principal afin qu’il puisse déboguer ou appliquer des modifications de configuration si le bot principal est hors service.

Espacement des ports : laissez au moins 20 ports entre les ports de base afin que les ports dérivés du navigateur/canvas/CDP n’entrent jamais en conflit.

<div id="how-to-install-rescue-bot">
  ### Comment installer le bot de secours
</div>

```bash
# Main bot (existing or fresh, without --profile param)
# Runs on port 18789 + Chrome CDC/Canvas/... Ports 
openclaw onboard
openclaw gateway install

# Bot de secours (profil et ports isolés)
openclaw --profile rescue onboard
# Notes : 
# - le nom de l'espace de travail sera suffixé par -rescue par défaut
# - le port doit être au minimum 18789 + 20 ports, 
#   il est préférable de choisir un port de base complètement différent, comme 19789,
# - le reste de l'intégration est identique à la procédure normale

# To install the service (if not happened automatically during onboarding)
openclaw --profile rescue gateway install
```


<div id="port-mapping-derived">
  ## Mappage de ports (dérivé)
</div>

Port de base = `gateway.port` (ou `OPENCLAW_GATEWAY_PORT` / `--port`).

- port du service de contrôle du navigateur = base + 2 (interface loopback uniquement)
- `canvasHost.port = base + 4`
- les ports CDP du profil de navigateur sont alloués automatiquement à partir de `browser.controlPort + 9 .. + 108`

Si vous redéfinissez l’un de ces ports dans la configuration ou les variables d’environnement, vous devez garantir qu’ils restent uniques pour chaque instance.

<div id="browsercdp-notes-common-footgun">
  ## Notes sur le navigateur/CDP (piège courant)
</div>

- **Ne** fixez **pas** `browser.cdpUrl` aux mêmes valeurs sur plusieurs instances.
- Chaque instance a besoin de son propre port de contrôle du navigateur et de sa propre plage CDP (dérivés de son port de Gateway).
- Si vous avez besoin de ports CDP explicites, définissez `browser.profiles.<name>.cdpPort` pour chaque instance.
- Chrome distant : utilisez `browser.profiles.<name>.cdpUrl` (par profil, par instance).

<div id="manual-env-example">
  ## Exemple d&#39;environnement configuré manuellement
</div>

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/main.json \
OPENCLAW_STATE_DIR=~/.openclaw-main \
openclaw gateway --port 18789

OPENCLAW_CONFIG_PATH=~/.openclaw/rescue.json \
OPENCLAW_STATE_DIR=~/.openclaw-rescue \
openclaw gateway --port 19001
```


<div id="quick-checks">
  ## Contrôles rapides
</div>

```bash
openclaw --profile main status
openclaw --profile rescue status
openclaw --profile rescue browser status
```
