---
title: Windows
summary: "Prise en charge de Windows (WSL2) et état de l'application compagnon"
read_when:
  - Installation d'OpenClaw sur Windows
  - Consultation de l'état de l'application compagnon Windows
---

<div id="windows-wsl2">
  # Windows (WSL2)
</div>

OpenClaw sur Windows est à installer de préférence **via WSL2** (Ubuntu recommandé). Le
CLI + Gateway s&#39;exécutent sous Linux, ce qui assure un environnement d&#39;exécution homogène et
améliore largement la compatibilité des outils (Node/Bun/pnpm, binaires Linux, compétences). Les installations
natives sous Windows ne sont pas testées et sont plus problématiques.

Des applications compagnon natives pour Windows sont prévues.

<div id="install-wsl2">
  ## Installation (WSL2)
</div>

* [Prise en main](/fr/start/getting-started) (à utiliser dans WSL2)
* [Installation et mises à jour](/fr/install/updating)
* Guide officiel WSL2 (Microsoft) : https://learn.microsoft.com/windows/wsl/install

<div id="gateway">
  ## Gateway
</div>

* [Runbook du Gateway](/fr/gateway)
* [Configuration](/fr/gateway/configuration)

<div id="gateway-service-install-cli">
  ## Installation du service Gateway via la CLI
</div>

Dans WSL2 :

```
openclaw onboard --install-daemon
```

Ou :

```
openclaw gateway install
```

Ou :

```
openclaw configure
```

Sélectionnez **Gateway service** lorsque vous y êtes invité.

Réparer/migrer :

```
openclaw doctor
```

<div id="advanced-expose-wsl-services-over-lan-portproxy">
  ## Avancé : exposer les services WSL sur le LAN (portproxy)
</div>

WSL dispose de son propre réseau virtuel. Si une autre machine doit accéder à un service
s’exécutant **dans WSL** (SSH, un serveur TTS local ou le Gateway), vous devez
rediriger un port Windows vers l’adresse IP actuelle de WSL. L’adresse IP de WSL change après les redémarrages ; vous devrez donc peut‑être actualiser la règle de redirection.

Exemple (PowerShell **en tant qu’administrateur**) :

```powershell
$Distro = "Ubuntu-24.04"
$ListenPort = 2222
$TargetPort = 22

$WslIp = (wsl -d $Distro -- hostname -I).Trim().Split(" ")[0]
if (-not $WslIp) { throw "IP WSL introuvable." }

netsh interface portproxy add v4tov4 listenaddress=0.0.0.0 listenport=$ListenPort `
  connectaddress=$WslIp connectport=$TargetPort
```

Autoriser le port dans le Pare-feu Windows (une seule fois) :

```powershell
New-NetFirewallRule -DisplayName "WSL SSH $ListenPort" -Direction Inbound `
  -Protocol TCP -LocalPort $ListenPort -Action Allow
```

Rafraîchissez le portproxy après chaque redémarrage de WSL :

```powershell
netsh interface portproxy delete v4tov4 listenport=$ListenPort listenaddress=0.0.0.0 | Out-Null
netsh interface portproxy add v4tov4 listenport=$ListenPort listenaddress=0.0.0.0 `
  connectaddress=$WslIp connectport=$TargetPort | Out-Null
```

Notes :

* Les connexions SSH depuis une autre machine doivent cibler l’**IP de l’hôte Windows** (exemple : `ssh user@windows-host -p 2222`).
* Les nœuds distants doivent pointer vers une URL de Gateway **accessible** (pas `127.0.0.1`) ; utilisez
  `openclaw status --all` pour vérifier.
* Utilisez `listenaddress=0.0.0.0` pour un accès sur le LAN ; `127.0.0.1` le limite à la machine locale.
* Si vous voulez automatiser cela, créez une tâche planifiée pour exécuter l’étape de rafraîchissement
  à la connexion.

<div id="step-by-step-wsl2-install">
  ## Installation pas à pas de WSL2
</div>

<div id="1-install-wsl2-ubuntu">
  ### 1) Installer WSL2 + Ubuntu
</div>

Ouvrez PowerShell en tant qu’administrateur :

```powershell
wsl --install
# Ou choisissez explicitement une distribution :
wsl --list --online
wsl --install -d Ubuntu-24.04
```

Redémarrez l’ordinateur si Windows vous le demande.

<div id="2-enable-systemd-required-for-gateway-install">
  ### 2) Activer systemd (obligatoire pour l&#39;installation du Gateway)
</div>

Dans votre terminal WSL :

```bash
sudo tee /etc/wsl.conf >/dev/null <<'EOF'
[boot]
systemd=true
EOF
```

Ensuite, à partir de PowerShell :

```powershell
wsl --shutdown
```

Relancez Ubuntu, puis vérifiez :

```bash
systemctl --user status
```

<div id="3-install-openclaw-inside-wsl">
  ### 3) Installer OpenClaw (dans WSL)
</div>

Suivez la procédure de prise en main Linux dans WSL :

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm ui:build # installe automatiquement les dépendances de l'UI à la première exécution
pnpm build
openclaw onboard
```

Guide complet : [Prise en main](/fr/start/getting-started)

<div id="windows-companion-app">
  ## Application compagnon pour Windows
</div>

Nous n&#39;avons pas encore d&#39;application compagnon pour Windows. Les contributions sont les bienvenues si vous souhaitez participer à sa réalisation.