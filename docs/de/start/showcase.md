---
title: "Showcase"
description: "Praxisnahe OpenClaw-Projekte aus der Community"
summary: "Community-Projekte und Integrationen mit OpenClaw"
---

<div id="showcase">
  # Showcase
</div>

Echte Projekte aus der Community. Sieh dir an, was Leute mit OpenClaw entwickeln.

<Info>
**Möchtest du vorgestellt werden?** Teile dein Projekt im [#showcase auf Discord](https://discord.gg/clawd) oder [markiere @openclaw auf X](https://x.com/openclaw).
</Info>

<div id="openclaw-in-action">
  ## 🎥 OpenClaw in Aktion
</div>

Vollständige Setup-Anleitung (28 Min.) von VelvetShark.

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
    title="OpenClaw: Die selbstgehostete KI, die Siri hätte sein sollen (vollständiges Setup)"
    style={{ position: "absolute", top: 0, left: 0, width: "100%", height: "100%" }}
    frameBorder="0"
    loading="lazy"
    allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
    allowFullScreen
  />
</div>

[Auf YouTube ansehen](https://www.youtube.com/watch?v=SaWSPZoPX34)

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
    title="OpenClaw Showcase-Video"
    style={{ position: "absolute", top: 0, left: 0, width: "100%", height: "100%" }}
    frameBorder="0"
    loading="lazy"
    allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
    allowFullScreen
  />
</div>

[Auf YouTube ansehen](https://www.youtube.com/watch?v=mMSKQvlmFuQ)

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
    title="OpenClaw Community-Showcase"
    style={{ position: "absolute", top: 0, left: 0, width: "100%", height: "100%" }}
    frameBorder="0"
    loading="lazy"
    allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
    allowFullScreen
  />
</div>

[Auf YouTube ansehen](https://www.youtube.com/watch?v=5kkIJNUGFho)

<div id="fresh-from-discord">
  ## 🆕 Direkt aus Discord
</div>

<CardGroup cols={2}>
  <Card title="PR-Review → Feedback per Telegram" icon="code-pull-request" href="https://x.com/i/status/2010878524543131691">
    **@bangnokia** • `review` `github` `telegram`

    OpenCode führt die Änderung durch → öffnet einen PR → OpenClaw prüft das Diff und antwortet in Telegram mit „kleinen Vorschlägen“ plus einer klaren Merge-Entscheidung (einschließlich kritischer Korrekturen, die zuerst umgesetzt werden müssen).

    <img src="/assets/showcase/pr-review-telegram.jpg" alt="OpenClaw PR-Review-Feedback in Telegram" />
  </Card>

  <Card title="Weinkeller-Skill in wenigen Minuten" icon="wine-glass" href="https://x.com/i/status/2010916352454791216">
    **@prades&#95;maxime** • `skills` `local` `csv`

    Hat „Robby“ (@openclaw) um eine lokale Weinkeller-Fähigkeit gebeten. Der Assistent fordert einen Beispiel-CSV-Export sowie den Speicherort an und erstellt und testet die Fähigkeit anschließend schnell (962 Flaschen im Beispiel).

    <img src="/assets/showcase/wine-cellar-skill.jpg" alt="OpenClaw erstellt eine lokale Weinkeller-Fähigkeit aus CSV" />
  </Card>

  <Card title="Tesco-Shop-Autopilot" icon="cart-shopping" href="https://x.com/i/status/2009724862470689131">
    **@marchattonhere** • `automation` `browser` `shopping`

    Wöchentlicher Essensplan → Stammprodukte → Lieferzeitfenster buchen → Bestellung bestätigen. Keine APIs, nur Browsersteuerung.

    <img src="/assets/showcase/tesco-shop.jpg" alt="Tesco-Shop-Automatisierung per Chat" />
  </Card>

  <Card title="SNAG Screenshot zu Markdown" icon="scissors" href="https://github.com/am-will/snag">
    **@am-will** • `devtools` `screenshots` `markdown`

    Mit einem Tastenkürzel einen Bildschirmbereich markieren → Gemini Vision → sofort Markdown in der Zwischenablage.

    <img src="/assets/showcase/snag.png" alt="SNAG Screenshot-zu-Markdown-Tool" />
  </Card>

  <Card title="Agenten-UI" icon="window-maximize" href="https://releaseflow.net/kitze/agents-ui">
    **@kitze** • `ui` `skills` `sync`

    Desktop-App zur Verwaltung von Fähigkeiten und Befehlen für Agenten, Claude, Codex und OpenClaw.

    <img src="/assets/showcase/agents-ui.jpg" alt="Agents-UI-App" />
  </Card>

  <Card title="Telegram-Sprachnachrichten (papla.media)" icon="microphone" href="https://papla.media/docs">
    **Community** • `voice` `tts` `telegram`

    Wrappt papla.media TTS und sendet die Ergebnisse als Telegram-Sprachnachrichten (kein nerviges Autoplay).

    <img src="/assets/showcase/papla-tts.jpg" alt="Telegram-Sprachnachricht-Ausgabe von TTS" />
  </Card>

  <Card title="CodexMonitor" icon="eye" href="https://clawhub.com/odrobnik/codexmonitor">
    **@odrobnik** • `devtools` `codex` `brew`

    Mit Homebrew installiertes Hilfstool zum Auflisten, Inspizieren und Überwachen lokaler OpenAI-Codex-Sitzungen (CLI + VS Code).

    <img src="/assets/showcase/codexmonitor.png" alt="CodexMonitor auf ClawHub" />
  </Card>

  <Card title="Steuerung für Bambu-3D-Drucker" icon="print" href="https://clawhub.com/tobiasbischoff/bambu-cli">
    **@tobiasbischoff** • `hardware` `3d-printing` `skill`

    BambuLab-Drucker steuern und Fehler beheben: Status, Aufträge, Kamera, AMS, Kalibrierung und mehr.

    <img src="/assets/showcase/bambu-cli.png" alt="Bambu CLI-Skill auf ClawHub" />
  </Card>

  <Card title="Wiener Linien (Verkehrsbetriebe Wien)" icon="train" href="https://clawhub.com/hjanuschka/wienerlinien">
    **@hjanuschka** • `travel` `transport` `skill`

    Echtzeit-Abfahrten, Störungsmeldungen, Aufzugsstatus und Routenplanung für den öffentlichen Nahverkehr in Wien.

    <img src="/assets/showcase/wienerlinien.png" alt="Wiener Linien-Skill auf ClawHub" />
  </Card>

  <Card title="ParentPay-Schulessen" icon="utensils" href="#">
    **@George5562** • `automation` `browser` `parenting`

    Automatisierte Buchung von Schulessen in Großbritannien über ParentPay. Nutzt Mauskoordinaten, um zuverlässig auf Tabellenzellen zu klicken.
  </Card>

  <Card title="R2 Upload (Schick mir meine Dateien)" icon="cloud-arrow-up" href="https://clawhub.com/skills/r2-upload">
    **@julianengel** • `files` `r2` `presigned-urls`

    Uploads nach Cloudflare R2/S3 und Erzeugen sicherer, vorab signierter Download-Links. Perfekt für entfernte OpenClaw-Instanzen.
  </Card>

  <Card title="iOS-App über Telegram" icon="mobile" href="#">
    **@coard** • `ios` `xcode` `testflight`

    Komplette iOS-App mit Karten und Sprachaufnahmen gebaut und ausschließlich über einen Telegram-Chat auf TestFlight bereitgestellt.

    <img src="/assets/showcase/ios-testflight.jpg" alt="iOS app on TestFlight" />
  </Card>

  <Card title="Gesundheitsassistent für den Oura Ring" icon="heart-pulse" href="#">
    **@AS** • `health` `oura` `calendar`

    Persönlicher KI-Gesundheitsassistent, der Daten des Oura-Rings mit Kalender, Terminen und Trainingsplan verknüpft.

    <img src="/assets/showcase/oura-health.png" alt="Oura ring health assistant" />
  </Card>

  <Card title="Kevs Dream-Team (14+ Agenten)" icon="robot" href="https://github.com/adam91holt/orchestrated-ai-articles">
    **@adam91holt** • `multi-agent` `orchestration` `architecture` `manifesto`

    14+ Agenten unter einem Gateway mit einem Opus-4.5-Orchestrator, der an Codex-Worker delegiert. Umfassender [technischer Beitrag](https://github.com/adam91holt/orchestrated-ai-articles), der das Dream-Team-Roster, Modellauswahl, Sandboxing, Webhooks, Herzschläge und Delegationsabläufe abdeckt. [Clawdspace](https://github.com/adam91holt/clawdspace) für Agent-Sandboxing. [Blogpost](https://adams-ai-journey.ghost.io/2026-the-year-of-the-orchestrator/).
  </Card>

  <Card title="Linear CLI" icon="terminal" href="https://github.com/Finesssee/linear-cli">
    **@NessZerra** • `devtools` `linear` `cli` `issues`

    CLI für Linear, die sich in agentenbasierte Workflows (Claude Code, OpenClaw) integriert. Verwalte Issues, Projekte und Workflows direkt aus dem Terminal. Erste externe PR gemergt!
  </Card>

  <Card title="Beeper CLI" icon="message" href="https://github.com/blqke/beepcli">
    **@jules** • `messaging` `beeper` `cli` `automation`

    Lesen, Senden und Archivieren von Nachrichten mit Beeper Desktop. Nutzt die lokale Beeper MCP api, damit Agenten all deine Chats (iMessage, WhatsApp usw.) zentral verwalten können.
  </Card>
</CardGroup>

<div id="automation-workflows">
  ## 🤖 Automatisierung & Workflows
</div>

<CardGroup cols={2}>

<Card title="Steuerung des Winix-Luftreinigers" icon="wind" href="https://x.com/antonplex/status/2010518442471006253">
  **@antonplex** • `automation` `hardware` `air-quality`

  Claude Code hat die Steuerung des Luftreinigers entdeckt und verifiziert, anschließend übernimmt OpenClaw die Regelung der Raumluftqualität.

  <img src="/assets/showcase/winix-air-purifier.jpg" alt="Winix-Luftreinigersteuerung über OpenClaw" />
</Card>

<Card title="Schöne Himmelsaufnahmen mit der Kamera" icon="camera" href="https://x.com/signalgaining/status/2010523120604746151">
  **@signalgaining** • `automation` `camera` `skill` `images`

  Ausgelöst von einer Dachkamera: Lass OpenClaw ein Himmelsfoto machen, sobald es schön aussieht – es hat einen Skill entworfen und das Bild aufgenommen.

  <img src="/assets/showcase/roof-camera-sky.jpg" alt="Himmels-Snapshot der Dachkamera, aufgenommen von OpenClaw" />
</Card>

<Card title="Visuelle Morning-Briefing-Szene" icon="robot" href="https://x.com/buddyhadry/status/2010005331925954739">
  **@buddyhadry** • `automation` `briefing` `images` `telegram`

  Ein geplanter Prompt generiert jeden Morgen ein einzelnes „Szenen“-Bild (Wetter, Aufgaben, Datum, Lieblingspost/-zitat) über eine OpenClaw-Persona.
</Card>

<Card title="Padelplatz-Buchung" icon="calendar-check" href="https://github.com/joshp123/padel-cli">
  **@joshp123** • `automation` `booking` `cli`
  
  Playtomic-Verfügbarkeitsprüfer + Buchungs-CLI. Verpasse nie wieder einen freien Court.
  
  <img src="/assets/showcase/padel-screenshot.jpg" alt="padel-cli Screenshot" />
</Card>

<Card title="Buchhaltungs-Intake" icon="file-invoice-dollar">
  **Community** • `automation` `email` `pdf`
  
  Sammelt PDFs aus E-Mails und bereitet Unterlagen für den Steuerberater vor. So läuft die monatliche Buchhaltung im Autopilot-Modus.
</Card>

<Card title="Couch Potato Dev Mode" icon="couch" href="https://davekiss.com">
  **@davekiss** • `telegram` `website` `migration` `astro`

  Komplette persönliche Website via Telegram neu aufgebaut, während Netflix lief – Notion → Astro, 18 Beiträge migriert, DNS zu Cloudflare. Ohne je einen Laptop zu öffnen.
</Card>

<Card title="Job-Suchagent" icon="briefcase">
  **@attol8** • `automation` `api` `skill`

  Durchsucht Stellenanzeigen, gleicht sie mit CV-Schlüsselwörtern ab und liefert passende Angebote mit Links zurück. In 30 Minuten mit der JSearch API gebaut.
</Card>

<Card title="Jira-Skill-Builder" icon="diagram-project" href="https://x.com/jdrhyne/status/2008336434827002232">
  **@jdrhyne** • `automation` `jira` `skill` `devtools`

  OpenClaw wurde mit Jira verbunden und hat dann spontan einen neuen Skill generiert (bevor er auf ClawHub existierte).
</Card>

<Card title="Todoist-Skill via Telegram" icon="list-check" href="https://x.com/iamsubhrajyoti/status/2009949389884920153">
  **@iamsubhrajyoti** • `automation` `todoist` `skill` `telegram`

  Hat Todoist-Aufgaben automatisiert und OpenClaw den Skill direkt im Telegram-Chat generieren lassen.
</Card>

<Card title="TradingView-Analyse" icon="chart-line">
  **@bheem1798** • `finance` `browser` `automation`

  Loggt sich per Browser-Automatisierung in TradingView ein, erstellt Screenshots von Charts und führt auf Abruf technische Analysen durch. Keine API nötig – nur Browsersteuerung.
</Card>

<Card title="Slack-Auto-Support" icon="slack">
  **@henrymascot** • `slack` `automation` `support`

  Überwacht den Slack-Kanal des Unternehmens, antwortet hilfreich und leitet Benachrichtigungen an Telegram weiter. Behebte eigenständig einen Produktions-Bug in einer produktiven App – ganz ohne Aufforderung.
</Card>

</CardGroup>

<div id="knowledge-memory">
  ## 🧠 Wissen & Memory
</div>

<CardGroup cols={2}>

<Card title="xuezh Chinese Learning" icon="language" href="https://github.com/joshp123/xuezh">
  **@joshp123** • `learning` `voice` `skill`
  
  Lern-Engine für Chinesisch mit Aussprache-Feedback und Lernabläufen über OpenClaw.
  
  <img src="/assets/showcase/xuezh-pronunciation.jpeg" alt="xuezh pronunciation feedback" />
</Card>

<Card title="WhatsApp Memory Vault" icon="vault">
  **Community** • `memory` `transcription` `indexing`
  
  Importiert vollständige WhatsApp-Exporte, transkribiert über 1.000 Sprachnotizen, gleicht sie mit Git-Logs ab und erzeugt verlinkte Markdown-Berichte.
</Card>

<Card title="Karakeep Semantic Search" icon="magnifying-glass" href="https://github.com/jamesbrooksco/karakeep-semantic-search">
  **@jamesbrooksco** • `search` `vector` `bookmarks`
  
  Fügt Karakeep-Lesezeichen eine Vektorsuche mit Qdrant + OpenAI/Ollama-Embeddings hinzu.
</Card>

<Card title="Inside-Out-2 Memory" icon="brain">
  **Community** • `memory` `beliefs` `self-model`
  
  Eigenständiger Memory-Manager, der Sitzungsdateien in Erinnerungen → Überzeugungen → ein sich entwickelndes Selbstmodell verwandelt.
</Card>

</CardGroup>

<div id="voice-phone">
  ## 🎙️ Stimme & Telefon
</div>

<CardGroup cols={2}>

<Card title="Clawdia Phone Bridge" icon="phone" href="https://github.com/alejandroOPI/clawdia-bridge">
  **@alejandroOPI** • `voice` `vapi` `bridge`
  
  Vapi-Sprachassistent ↔ OpenClaw HTTP-Bridge. Nahezu in Echtzeit telefonieren mit deinem agent.
</Card>

<Card title="OpenRouter Transcription" icon="microphone" href="https://clawhub.com/obviyus/openrouter-transcribe">
  **@obviyus** • `transcription` `multilingual` `skill`

  Mehrsprachige Audio-Transkription über OpenRouter (Gemini, etc.). Auf ClawHub verfügbar.
</Card>

</CardGroup>

<div id="infrastructure-deployment">
  ## 🏗️ Infrastruktur & Deployment
</div>

<CardGroup cols={2}>

<Card title="Home Assistant Add-on" icon="home" href="https://github.com/ngutman/openclaw-ha-addon">
  **@ngutman** • `homeassistant` `docker` `raspberry-pi`
  
  OpenClaw Gateway auf Home Assistant OS mit Unterstützung für SSH-Tunnel und persistentem Zustand.
</Card>

<Card title="Home Assistant Skill" icon="toggle-on" href="https://clawhub.com/skills/homeassistant">
  **ClawHub** • `homeassistant` `skill` `automation`
  
  Steuere und automatisiere Home-Assistant-Geräte per natürlicher Sprache.
</Card>

<Card title="Nix Packaging" icon="snowflake" href="https://github.com/openclaw/nix-openclaw">
  **@openclaw** • `nix` `packaging` `deployment`
  
  Umfassende, Nix-basierte OpenClaw-Konfiguration für reproduzierbare Deployments.
</Card>

<Card title="CalDAV Calendar" icon="calendar" href="https://clawhub.com/skills/caldav-calendar">
  **ClawHub** • `calendar` `caldav` `skill`
  
  Kalender-Fähigkeit mit khal/vdirsyncer. Selbstgehostete Kalenderintegration.
</Card>

</CardGroup>

<div id="home-hardware">
  ## 🏠 Zuhause & Hardware
</div>

<CardGroup cols={2}>

<Card title="GoHome Automation" icon="house-signal" href="https://github.com/joshp123/gohome">
  **@joshp123** • `home` `nix` `grafana`
  
  Nix-native Hausautomatisierung mit OpenClaw als Schnittstelle und ansprechenden Grafana-Dashboards.
  
  <img src="/assets/showcase/gohome-grafana.png" alt="GoHome-Grafana-Dashboard" />
</Card>

<Card title="Roborock Vacuum" icon="robot" href="https://github.com/joshp123/gohome/tree/main/plugins/roborock">
  **@joshp123** • `vacuum` `iot` `plugin`
  
  Steuere den Roborock-Saugroboter per natürlicher Sprache.
  
  <img src="/assets/showcase/roborock-screenshot.jpg" alt="Roborock-Status" />
</Card>

</CardGroup>

<div id="community-projects">
  ## 🌟 Community-Projekte
</div>

<CardGroup cols={2}>

<Card title="StarSwap Marketplace" icon="star" href="https://star-swap.com/">
  **Community** • `marketplace` `astronomy` `webapp`
  
  Umfassender Marktplatz für Astronomieausrüstung. Auf Basis des OpenClaw-Ökosystems aufgebaut.
</Card>

</CardGroup>

---

<div id="submit-your-project">
  ## Reiche dein Projekt ein
</div>

Hast du etwas, das du teilen möchtest? Wir würden es gerne vorstellen!

<Steps>
  <Step title="Teile es">
    Poste im [#showcase-Channel auf Discord](https://discord.gg/clawd) oder veröffentliche einen Tweet mit [@openclaw](https://x.com/openclaw)
  </Step>
  <Step title="Füge Details hinzu">
    Erzähl uns, was es macht, verlinke das Repo bzw. die Demo und teile einen Screenshot, falls du einen hast
  </Step>
  <Step title="Werde vorgestellt">
    Wir nehmen herausragende Projekte auf dieser Seite auf
  </Step>
</Steps>