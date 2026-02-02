---
title: VM macOS
summary: "Exécuter OpenClaw dans une VM macOS isolée (sandbox, locale ou hébergée) lorsque vous avez besoin d’isolement ou d’iMessage"
read_when:
  - Vous voulez isoler OpenClaw de votre environnement macOS principal
  - Vous voulez une intégration iMessage (BlueBubbles) dans un sandbox
  - Vous voulez un environnement macOS réinitialisable que vous pouvez cloner
  - Vous voulez comparer les options de VM macOS locales et hébergées
---

<div id="openclaw-on-macos-vms-sandboxing">
  # OpenClaw sur des machines virtuelles macOS (sandbox)
</div>

<div id="recommended-default-most-users">
  ## Recommandation par défaut (pour la plupart des utilisateurs)
</div>

* **Petit VPS Linux** pour un Gateway toujours en ligne et à faible coût. Voir [Hébergement VPS](/fr/vps).
* **Matériel dédié** (Mac mini ou machine Linux) si vous voulez un contrôle total et une **IP résidentielle** pour l’automatisation du navigateur. De nombreux sites bloquent les IP de centres de données, donc la navigation locale fonctionne souvent mieux.
* **Hybride :** gardez le Gateway sur un VPS peu coûteux et connectez votre Mac comme **nœud** lorsque vous avez besoin d’automatisation du navigateur/de l’UI. Voir [Nodes](/fr/nodes) et [Gateway remote](/fr/gateway/remote).

Utilisez une VM macOS lorsque vous avez spécifiquement besoin de capacités propres à macOS (iMessage/BlueBubbles) ou que vous souhaitez une isolation stricte de votre Mac utilisé au quotidien.

<div id="macos-vm-options">
  ## Options de machines virtuelles macOS
</div>

<div id="local-vm-on-your-apple-silicon-mac-lume">
  ### VM locale sur votre Mac Apple Silicon (Lume)
</div>

Exécutez OpenClaw dans une VM macOS isolée (sandbox) sur votre Mac Apple Silicon existant à l’aide de [Lume](https://cua.ai/docs/lume).

Cela vous permet de bénéficier :

* D’un environnement macOS complet et isolé (votre hôte reste propre)
* De la prise en charge d’iMessage via BlueBubbles (impossible sous Linux/Windows)
* D’une réinitialisation instantanée en clonant la VM
* D’aucun coût matériel ou cloud supplémentaire

<div id="hosted-mac-providers-cloud">
  ### Fournisseurs Mac hébergés (cloud)
</div>

Si vous voulez macOS dans le cloud, les fournisseurs de Mac hébergés peuvent aussi convenir :

* [MacStadium](https://www.macstadium.com/) (Mac hébergés)
* D&#39;autres fournisseurs de Mac hébergés fonctionnent aussi ; suivez leur documentation sur les VM et SSH

Une fois que vous avez un accès SSH à une VM macOS, continuez à l&#39;étape 6 ci-dessous.

***

<div id="quick-path-lume-experienced-users">
  ## Parcours rapide (Lume, utilisateurs expérimentés)
</div>

1. Installez Lume
2. `lume create openclaw --os macos --ipsw latest`
3. Terminez l’assistant de configuration, activez l’ouverture de session à distance (SSH)
4. `lume run openclaw --no-display`
5. Connectez-vous en SSH, installez OpenClaw, configurez les canaux
6. C’est tout

***

<div id="what-you-need-lume">
  ## Ce dont vous avez besoin (Lume)
</div>

* Mac Apple Silicon (M1/M2/M3/M4)
* macOS Sequoia ou version ultérieure sur la machine hôte
* ~60 Go d’espace disque libre par VM
* ~20 minutes

***

<div id="1-install-lume">
  ## 1) Installer Lume
</div>

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/trycua/cua/main/libs/lume/scripts/install.sh)"
```

Si `~/.local/bin` n’est pas dans votre PATH :

```bash
echo 'export PATH="$PATH:$HOME/.local/bin"' >> ~/.zshrc && source ~/.zshrc
```

Vérifiez :

```bash
lume --version
```

Docs : [Installation de Lume](https://cua.ai/docs/lume/guide/getting-started/installation)

***

<div id="2-create-the-macos-vm">
  ## 2) Créer la machine virtuelle macOS
</div>

```bash
lume create openclaw --os macos --ipsw latest
```

Cette opération télécharge macOS et crée la VM. Une fenêtre VNC s’ouvre automatiquement.

Remarque : le téléchargement peut prendre un certain temps en fonction de votre connexion.

***

<div id="3-complete-setup-assistant">
  ## 3) Terminer l’assistant de configuration
</div>

Dans la fenêtre VNC :

1. Sélectionnez la langue et la région
2. Ignorez l’identifiant Apple (ou connectez-vous si vous voulez utiliser iMessage plus tard)
3. Créez un compte utilisateur (notez soigneusement le nom d’utilisateur et le mot de passe)
4. Ignorez toutes les fonctionnalités optionnelles

Une fois la configuration terminée, activez SSH :

1. Ouvrez Réglages Système → Général → Partage
2. Activez « Ouverture de session à distance »

***

<div id="4-get-the-vms-ip-address">
  ## 4) Obtenir l’adresse IP de la VM
</div>

```bash
lume get openclaw
```

Repérez l’adresse IP (généralement `192.168.64.x`).

***

<div id="5-ssh-into-the-vm">
  ## 5) Se connecter à la VM en SSH
</div>

```bash
ssh youruser@192.168.64.X
```

Remplacez `youruser` par le compte que vous avez créé et l’adresse IP par celle de votre VM.

***

<div id="6-install-openclaw">
  ## 6) Installer OpenClaw
</div>

Dans la VM :

```bash
npm install -g openclaw@latest
openclaw onboard --install-daemon
```

Suivez les étapes d’onboarding pour configurer votre fournisseur de modèles (Anthropic, OpenAI, etc.).

***

<div id="7-configure-channels">
  ## 7) Configurer les canaux
</div>

Modifiez le fichier de configuration :

```bash
nano ~/.openclaw/openclaw.json
```

Ajoutez vos canaux de communication :

```json
{
  "channels": {
    "whatsapp": {
      "dmPolicy": "allowlist",
      "allowFrom": ["+15551234567"]
    },
    "telegram": {
      "botToken": "YOUR_BOT_TOKEN"
    }
  }
}
```

Connectez-vous ensuite à WhatsApp (scannez le code QR) :

```bash
openclaw channels login
```

***

<div id="8-run-the-vm-headlessly">
  ## 8) Exécuter la VM en mode headless
</div>

Arrêtez la VM et redémarrez-la en mode headless :

```bash
lume stop openclaw
lume run openclaw --no-display
```

La machine virtuelle tourne en arrière-plan. Le démon OpenClaw maintient le Gateway en cours d’exécution.

Pour vérifier l’état :

```bash
ssh youruser@192.168.64.X "openclaw status"
```

***

<div id="bonus-imessage-integration">
  ## Bonus : intégration iMessage
</div>

C’est la fonctionnalité phare de l’exécution sur macOS. Utilisez [BlueBubbles](https://bluebubbles.app) pour ajouter iMessage à OpenClaw.

Dans la VM :

1. Téléchargez BlueBubbles depuis bluebubbles.app
2. Connectez-vous avec votre identifiant Apple
3. Activez l’API Web et définissez un mot de passe
4. Pointez les webhooks BlueBubbles vers votre Gateway (exemple : `https://your-gateway-host:3000/bluebubbles-webhook?password=&lt;password&gt;`)

Ajoutez à votre configuration OpenClaw :

```json
{
  "channels": {
    "bluebubbles": {
      "serverUrl": "http://localhost:1234",
      "password": "your-api-password",
      "webhookPath": "/bluebubbles-webhook"
    }
  }
}
```

Redémarrez Gateway. Votre agent peut maintenant envoyer et recevoir des iMessages.

Tous les détails de la configuration : [canal BlueBubbles](/fr/channels/bluebubbles)

***

<div id="save-a-golden-image">
  ## Enregistrer une image de référence
</div>

Avant toute personnalisation ultérieure, créez un instantané de cet état propre :

```bash
lume stop openclaw
lume clone openclaw openclaw-golden
```

Réinitialiser à tout moment :

```bash
lume stop openclaw && lume delete openclaw
lume clone openclaw-golden openclaw
lume run openclaw --no-display
```

***

<div id="running-247">
  ## Fonctionnement 24/7
</div>

Pour que la VM reste en fonctionnement :

* Gardez votre Mac branché sur secteur
* Désactivez la mise en veille dans Réglages Système → Économiseur d’énergie
* Utilisez `caffeinate` si nécessaire

Pour une disponibilité réellement continue, envisagez un Mac mini dédié ou un petit VPS. Voir [Hébergement VPS](/fr/vps).

***

<div id="troubleshooting">
  ## Dépannage
</div>

| Problème | Solution |
|---------|----------|
| Impossible d&#39;établir une connexion SSH à la VM | Vérifiez que « Remote Login » est activé dans les réglages système de la VM |
| L&#39;adresse IP de la VM ne s&#39;affiche pas | Attendez que la VM ait entièrement démarré, puis exécutez à nouveau `lume get openclaw` |
| Commande `lume` introuvable | Ajoutez `~/.local/bin` à votre PATH |
| Le code QR WhatsApp ne se lit pas | Assurez-vous d&#39;être connecté à la VM (et non à l&#39;hôte) lorsque vous exécutez `openclaw channels login` |

***

<div id="related-docs">
  ## Documentation associée
</div>

* [Hébergement VPS](/fr/vps)
* [Nœuds](/fr/nodes)
* [Gateway distant](/fr/gateway/remote)
* [Canal BlueBubbles](/fr/channels/bluebubbles)
* [Démarrage rapide Lume](https://cua.ai/docs/lume/guide/getting-started/quickstart)
* [Référence de la CLI Lume](https://cua.ai/docs/lume/reference/cli-reference)
* [Configuration de VM sans intervention](https://cua.ai/docs/lume/guide/fundamentals/unattended-setup) (avancé)
* [Sandboxing Docker](/fr/install/docker) (approche d’isolation alternative)