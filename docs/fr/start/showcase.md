---
title: "Vitrine"
description: "Projets OpenClaw réels issus de la communauté"
summary: "Projets et intégrations conçus par la communauté et propulsés par OpenClaw"
---

<div id="showcase">
  # Exemples
</div>

Des projets concrets de la communauté. Découvrez ce que les gens construisent avec OpenClaw.

<Info>
**Vous voulez être mis en avant&nbsp;?** Partagez votre projet dans [#showcase sur Discord](https://discord.gg/clawd) ou [mentionnez @openclaw sur X](https://x.com/openclaw).
</Info>

<div id="openclaw-in-action">
  ## 🎥 OpenClaw en action
</div>

Tutoriel complet de configuration (28 min) par VelvetShark.

<div
  style={{
    position: "relative",
    paddingBottom: "56.25%",
    height: 0,
    overflow: "hidden",
    borderRadius: 16,
  }}
>
  <iframe
    src="https://www.youtube-nocookie.com/embed/SaWSPZoPX34"
    title="OpenClaw : l'IA auto-hébergée que Siri aurait dû être (configuration complète)"
    style={{ position: "absolute", top: 0, left: 0, width: "100%", height: "100%" }}
    frameBorder="0"
    loading="lazy"
    allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
    allowFullScreen
  />
</div>

[Voir sur YouTube](https://www.youtube.com/watch?v=SaWSPZoPX34)

<div
  style={{
    position: "relative",
    paddingBottom: "56.25%",
    height: 0,
    overflow: "hidden",
    borderRadius: 16,
  }}
>
  <iframe
    src="https://www.youtube-nocookie.com/embed/mMSKQvlmFuQ"
    title="Vidéo de présentation d’OpenClaw"
    style={{ position: "absolute", top: 0, left: 0, width: "100%", height: "100%" }}
    frameBorder="0"
    loading="lazy"
    allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
    allowFullScreen
  />
</div>

[Voir sur YouTube](https://www.youtube.com/watch?v=mMSKQvlmFuQ)

<div
  style={{
    position: "relative",
    paddingBottom: "56.25%",
    height: 0,
    overflow: "hidden",
    borderRadius: 16,
  }}
>
  <iframe
    src="https://www.youtube-nocookie.com/embed/5kkIJNUGFho"
    title="Présentation de la communauté OpenClaw"
    style={{ position: "absolute", top: 0, left: 0, width: "100%", height: "100%" }}
    frameBorder="0"
    loading="lazy"
    allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
    allowFullScreen
  />
</div>

[Voir sur YouTube](https://www.youtube.com/watch?v=5kkIJNUGFho)

<div id="fresh-from-discord">
  ## 🆕 En direct de Discord
</div>

<CardGroup cols={2}>
  <Card title="Revue de PR → Commentaires Telegram" icon="code-pull-request" href="https://x.com/i/status/2010878524543131691">
    **@bangnokia** • `review` `github` `telegram`

    OpenCode finalise la modification → ouvre une PR → OpenClaw examine le diff et répond dans Telegram avec « quelques suggestions mineures » ainsi qu’un verdict de fusion clair (y compris les correctifs critiques à appliquer en priorité).

    <img src="/assets/showcase/pr-review-telegram.jpg" alt="Commentaires de revue de PR OpenClaw envoyés dans Telegram" />
  </Card>

  <Card title="Une compétence de cave à vin en quelques minutes" icon="wine-glass" href="https://x.com/i/status/2010916352454791216">
    **@prades&#95;maxime** • `skills` `local` `csv`

    A demandé à « Robby » (@openclaw) une compétence de cave à vin locale. Il demande un exemple d’export CSV et l’emplacement où le stocker, puis génère et teste la compétence très rapidement (962 bouteilles dans l’exemple).

    <img src="/assets/showcase/wine-cellar-skill.jpg" alt="OpenClaw construisant une compétence locale de cave à vin à partir d’un CSV" />
  </Card>

  <Card title="Autopilote pour Tesco Shop" icon="cart-shopping" href="https://x.com/i/status/2009724862470689131">
    **@marchattonhere** • `automation` `browser` `shopping`

    Plan hebdomadaire de repas → articles habituels → réserver un créneau de livraison → confirmer la commande. Aucune API, juste un pilotage du navigateur.

    <img src="/assets/showcase/tesco-shop.jpg" alt="Automatisation des courses Tesco via chat" />
  </Card>

  <Card title="SNAG : capture d’écran vers Markdown" icon="scissors" href="https://github.com/am-will/snag">
    **@am-will** • `devtools` `screenshots` `markdown`

    Raccourci clavier sur une zone de l’écran → Gemini Vision → Markdown instantané dans votre presse-papiers.

    <img src="/assets/showcase/snag.png" alt="SNAG, outil de conversion de capture d’écran en Markdown" />
  </Card>

  <Card title="UI des agents" icon="window-maximize" href="https://releaseflow.net/kitze/agents-ui">
    **@kitze** • `ui` `skills` `sync`

    Application de bureau pour gérer les compétences et les commandes des Agents, de Claude, de Codex et d’OpenClaw.

    <img src="/assets/showcase/agents-ui.jpg" alt="Application UI pour Agents" />
  </Card>

  <Card title="Messages vocaux Telegram (papla.media)" icon="microphone" href="https://papla.media/docs">
    **Communauté** • `voice` `tts` `telegram`

    Intègre le TTS de papla.media et envoie les résultats sous forme de notes vocales Telegram (sans lecture automatique agaçante).

    <img src="/assets/showcase/papla-tts.jpg" alt="Note vocale Telegram générée à partir du TTS" />
  </Card>

  <Card title="CodexMonitor" icon="eye" href="https://clawhub.com/odrobnik/codexmonitor">
    **@odrobnik** • `devtools` `codex` `brew`

    Utilitaire installé via Homebrew pour lister/inspecter/surveiller les sessions OpenAI Codex locales (CLI + VS Code).

    <img src="/assets/showcase/codexmonitor.png" alt="CodexMonitor sur ClawHub" />
  </Card>

  <Card title="Contrôle de l’imprimante 3D Bambu" icon="print" href="https://clawhub.com/tobiasbischoff/bambu-cli">
    **@tobiasbischoff** • `hardware` `3d-printing` `skill`

    Contrôlez et dépannez les imprimantes BambuLab : état, tâches d’impression, caméra, AMS, étalonnage et plus encore.

    <img src="/assets/showcase/bambu-cli.png" alt="Compétence Bambu CLI sur ClawHub" />
  </Card>

  <Card title="Transports publics de Vienne (Wiener Linien)" icon="train" href="https://clawhub.com/hjanuschka/wienerlinien">
    **@hjanuschka** • `travel` `transport` `skill`

    Départs en temps réel, perturbations, état des ascenseurs et calcul d’itinéraires pour les transports en commun de Vienne.

    <img src="/assets/showcase/wienerlinien.png" alt="Skill Wiener Linien sur ClawHub" />
  </Card>

  <Card title="ParentPay Repas scolaires" icon="utensils" href="#">
    **@George5562** • `automation` `browser` `parenting`

    Réservation automatique des repas de cantine au Royaume-Uni via ParentPay. Utilise les coordonnées de la souris pour cliquer de façon fiable sur les cellules du tableau.
  </Card>

  <Card title="Téléversement R2 (Envoie-moi mes fichiers)" icon="cloud-arrow-up" href="https://clawhub.com/skills/r2-upload">
    **@julianengel** • `files` `r2` `presigned-urls`

    Téléversez vers Cloudflare R2/S3 et générez des URL de téléchargement sécurisées pré-signées. Parfait pour les instances OpenClaw distantes.
  </Card>

  <Card title="Application iOS via Telegram" icon="mobile" href="#">
    **@coard** • `ios` `xcode` `testflight`

    A créé une application iOS complète avec cartes et enregistrement vocal, déployée sur TestFlight entièrement depuis une conversation Telegram.

    <img src="/assets/showcase/ios-testflight.jpg" alt="Application iOS sur TestFlight" />
  </Card>

  <Card title="Assistant de santé Oura Ring" icon="heart-pulse" href="#">
    **@AS** • `health` `oura` `calendar`

    Assistant santé personnel basé sur l’IA intégrant les données de la bague Oura avec le calendrier, les rendez-vous et le planning de la salle de sport.

    <img src="/assets/showcase/oura-health.png" alt="Assistant santé pour bague Oura" />
  </Card>

  <Card title="La dream team de Kev (14+ Agents)" icon="robot" href="https://github.com/adam91holt/orchestrated-ai-articles">
    **@adam91holt** • `multi-agent` `orchestration` `architecture` `manifesto`

    Plus de 14 agents au sein d’un seul Gateway avec un orchestrateur Opus 4.5 déléguant à des workers Codex. [Article technique détaillé](https://github.com/adam91holt/orchestrated-ai-articles) couvrant l’effectif de la Dream Team, la sélection de modèles, le sandboxing, les webhooks, les signaux de vie et les flux de délégation. [Clawdspace](https://github.com/adam91holt/clawdspace) pour le sandboxing des agents. [Article de blog](https://adams-ai-journey.ghost.io/2026-the-year-of-the-orchestrator/).
  </Card>

  <Card title="Linear CLI" icon="terminal" href="https://github.com/Finesssee/linear-cli">
    **@NessZerra** • `devtools` `linear` `cli` `issues`

    CLI pour Linear, qui s’intègre à des workflows pilotés par des agents (Claude Code, OpenClaw). Gérez les issues, projets et workflows depuis le terminal. Première PR externe intégrée !
  </Card>

  <Card title="Beeper CLI" icon="message" href="https://github.com/blqke/beepcli">
    **@jules** • `messaging` `beeper` `cli` `automation`

    Lire, envoyer et archiver des messages via Beeper Desktop. Utilise l’API MCP locale de Beeper afin que les agents puissent gérer l’ensemble de vos conversations (iMessage, WhatsApp, etc.) depuis un seul endroit.
  </Card>
</CardGroup>

<div id="automation-workflows">
  ## 🤖 Automatisation & Workflows
</div>

<CardGroup cols={2}>

<Card title="Contrôle du purificateur d’air Winix" icon="wind" href="https://x.com/antonplex/status/2010518442471006253">
  **@antonplex** • `automation` `hardware` `air-quality`

  Claude Code a découvert et confirmé les commandes du purificateur, puis OpenClaw prend le relais pour gérer la qualité de l’air de la pièce.

  <img src="/assets/showcase/winix-air-purifier.jpg" alt="Contrôle du purificateur d’air Winix via OpenClaw" />
</Card>

<Card title="Beaux clichés du ciel" icon="camera" href="https://x.com/signalgaining/status/2010523120604746151">
  **@signalgaining** • `automation` `camera` `skill` `images`

  Déclenché par une caméra sur le toit : demande à OpenClaw de prendre une photo du ciel dès qu’il est joli — il a conçu une compétence et pris le cliché.

  <img src="/assets/showcase/roof-camera-sky.jpg" alt="Capture du ciel par la caméra de toit via OpenClaw" />
</Card>

<Card title="Scène visuelle de briefing matinal" icon="robot" href="https://x.com/buddyhadry/status/2010005331925954739">
  **@buddyhadry** • `automation` `briefing` `images` `telegram`

  Une invite programmée génère chaque matin une seule image de « scène » (météo, tâches, date, publication/citation favorite) via une persona OpenClaw.
</Card>

<Card title="Réservation de court de padel" icon="calendar-check" href="https://github.com/joshp123/padel-cli">
  **@joshp123** • `automation` `booking` `cli`
  
  Outil de vérification de disponibilité Playtomic + CLI de réservation. Ne manque plus jamais un court disponible.
  
  <img src="/assets/showcase/padel-screenshot.jpg" alt="Capture d’écran de padel-cli" />
</Card>

<Card title="Collecte comptable" icon="file-invoice-dollar">
  **Community** • `automation` `email` `pdf`
  
  Récupère les PDF depuis les e-mails et prépare les documents pour le conseiller fiscal. Comptabilité mensuelle en pilote automatique.
</Card>

<Card title="Mode dev canapé" icon="couch" href="https://davekiss.com">
  **@davekiss** • `telegram` `website` `migration` `astro`

  Reconstruction complète du site personnel via Telegram en regardant Netflix — Notion → Astro, 18 articles migrés, DNS vers Cloudflare. N’a jamais ouvert son ordinateur portable.
</Card>

<Card title="Agent de recherche d’emploi" icon="briefcase">
  **@attol8** • `automation` `api` `skill`

  Recherche des offres d’emploi, les compare aux mots-clés du CV et renvoie les opportunités pertinentes avec liens. Créé en 30 minutes en utilisant JSearch API.
</Card>

<Card title="Générateur de compétences Jira" icon="diagram-project" href="https://x.com/jdrhyne/status/2008336434827002232">
  **@jdrhyne** • `automation` `jira` `skill` `devtools`

  OpenClaw a été connecté à Jira, puis a généré une nouvelle compétence à la volée (avant qu’elle n’existe sur ClawHub).
</Card>

<Card title="Compétence Todoist via Telegram" icon="list-check" href="https://x.com/iamsubhrajyoti/status/2009949389884920153">
  **@iamsubhrajyoti** • `automation` `todoist` `skill` `telegram`

  A automatisé les tâches Todoist et fait générer la compétence par OpenClaw directement dans le chat Telegram.
</Card>

<Card title="Analyse TradingView" icon="chart-line">
  **@bheem1798** • `finance` `browser` `automation`

  Se connecte à TradingView via automatisation du navigateur, capture des captures d’écran des graphiques et réalise une analyse technique à la demande. Aucune API nécessaire — juste le contrôle du navigateur.
</Card>

<Card title="Support automatique sur Slack" icon="slack">
  **@henrymascot** • `slack` `automation` `support`

  Surveille un canal Slack d’entreprise, répond de façon utile et transfère les notifications vers Telegram. A corrigé de manière autonome un bug de production dans une application déployée sans qu’on le lui demande.
</Card>

</CardGroup>

<div id="knowledge-memory">
  ## 🧠 Connaissances & mémoire
</div>

<CardGroup cols={2}>

<Card title="xuezh Chinese Learning" icon="language" href="https://github.com/joshp123/xuezh">
  **@joshp123** • `learning` `voice` `skill`
  
  Moteur d'apprentissage du chinois avec retour sur la prononciation et parcours d'apprentissage via OpenClaw.
  
  <img src="/assets/showcase/xuezh-pronunciation.jpeg" alt="retour sur la prononciation xuezh" />
</Card>

<Card title="WhatsApp Memory Vault" icon="vault">
  **Community** • `memory` `transcription` `indexing`
  
  Importe des exports WhatsApp complets, transcrit plus de 1&nbsp;000 notes vocales, recoupe avec les logs Git et génère des rapports Markdown avec liens croisés.
</Card>

<Card title="Karakeep Semantic Search" icon="magnifying-glass" href="https://github.com/jamesbrooksco/karakeep-semantic-search">
  **@jamesbrooksco** • `search` `vector` `bookmarks`
  
  Ajoute la recherche vectorielle aux favoris Karakeep en utilisant Qdrant + les embeddings OpenAI/Ollama.
</Card>

<Card title="Inside-Out-2 Memory" icon="brain">
  **Community** • `memory` `beliefs` `self-model`
  
  Gestionnaire de mémoire distinct qui transforme les fichiers de session en souvenirs → croyances → modèle de soi évolutif.
</Card>

</CardGroup>

<div id="voice-phone">
  ## 🎙️ Voix & Téléphone
</div>

<CardGroup cols={2}>

<Card title="Passerelle téléphonique Clawdia" icon="phone" href="https://github.com/alejandroOPI/clawdia-bridge">
  **@alejandroOPI** • `voice` `vapi` `bridge`
  
  Assistant vocal Vapi ↔ passerelle HTTP OpenClaw. Appels téléphoniques presque en temps réel avec votre agent.
</Card>

<Card title="Transcription OpenRouter" icon="microphone" href="https://clawhub.com/obviyus/openrouter-transcribe">
  **@obviyus** • `transcription` `multilingual` `skill`

  Transcription audio multilingue via OpenRouter (Gemini, etc.). Disponible sur ClawHub.
</Card>

</CardGroup>

<div id="infrastructure-deployment">
  ## 🏗️ Infrastructure et déploiement
</div>

<CardGroup cols={2}>

<Card title="Module complémentaire Home Assistant" icon="home" href="https://github.com/ngutman/openclaw-ha-addon">
  **@ngutman** • `homeassistant` `docker` `raspberry-pi`
  
  Gateway OpenClaw exécuté sur Home Assistant OS, avec prise en charge des tunnels SSH et état persistant.
</Card>

<Card title="Compétence Home Assistant" icon="toggle-on" href="https://clawhub.com/skills/homeassistant">
  **ClawHub** • `homeassistant` `skill` `automation`
  
  Contrôlez et automatisez les appareils Home Assistant à l’aide du langage naturel.
</Card>

<Card title="Packaging Nix" icon="snowflake" href="https://github.com/openclaw/nix-openclaw">
  **@openclaw** • `nix` `packaging` `deployment`
  
  Configuration OpenClaw « nixifiée » avec batteries incluses pour des déploiements reproductibles.
</Card>

<Card title="Calendrier CalDAV" icon="calendar" href="https://clawhub.com/skills/caldav-calendar">
  **ClawHub** • `calendar` `caldav` `skill`
  
  Compétence calendrier utilisant khal/vdirsyncer. Intégration de calendrier auto-hébergée.
</Card>

</CardGroup>

<div id="home-hardware">
  ## 🏠 Maison & matériel
</div>

<CardGroup cols={2}>

<Card title="GoHome Automation" icon="house-signal" href="https://github.com/joshp123/gohome">
  **@joshp123** • `home` `nix` `grafana`
  
  Domotique native pour Nix, avec OpenClaw comme interface et de superbes tableaux de bord Grafana.
  
  <img src="/assets/showcase/gohome-grafana.png" alt="Tableau de bord Grafana GoHome" />
</Card>

<Card title="Roborock Vacuum" icon="robot" href="https://github.com/joshp123/gohome/tree/main/plugins/roborock">
  **@joshp123** • `vacuum` `iot` `plugin`
  
  Contrôlez votre aspirateur robot Roborock via des conversations naturelles.
  
  <img src="/assets/showcase/roborock-screenshot.jpg" alt="Statut Roborock" />
</Card>

</CardGroup>

<div id="community-projects">
  ## 🌟 Projets de la communauté
</div>

<CardGroup cols={2}>

<Card title="StarSwap Marketplace" icon="star" href="https://star-swap.com/">
  **Communauté** • `marketplace` `astronomy` `webapp`
  
  Place de marché complète pour du matériel d’astronomie. Construite autour de l’écosystème OpenClaw.
</Card>

</CardGroup>

---

<div id="submit-your-project">
  ## Proposez votre projet
</div>

Vous avez quelque chose à partager ? Nous serions ravis de le mettre en avant !

<Steps>
  <Step title="Partagez-le">
    Publiez dans [#showcase sur Discord](https://discord.gg/clawd) ou tweetez [@openclaw](https://x.com/openclaw)
  </Step>
  <Step title="Ajoutez des détails">
    Dites-nous ce que fait votre projet, ajoutez un lien vers le dépôt ou la démo, et partagez une capture d'écran si vous en avez une
  </Step>
  <Step title="Mise en avant">
    Nous ajouterons les projets les plus remarquables à cette page
  </Step>
</Steps>