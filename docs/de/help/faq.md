---
title: FAQ
summary: "Häufig gestellte Fragen zur Einrichtung, Konfiguration und Nutzung von OpenClaw"
---

<div id="faq">
  # FAQ
</div>

Kurzantworten und vertiefte Hinweise zur Fehlerdiagnose für Praxis-Setups (lokale Entwicklung, VPS, Multi-Agent-Setups, OAuth/API-Schlüssel, Modell-Failover). Für Laufzeitdiagnosen siehe [Troubleshooting](/de/gateway/troubleshooting). Die vollständige Konfigurationsreferenz findest du unter [Configuration](/de/gateway/configuration).

<div id="table-of-contents">
  ## Inhaltsverzeichnis
</div>

* [Schnellstart und Ersteinrichtung](#quick-start-and-firstrun-setup)
  * [Ich stecke fest – wie komme ich am schnellsten wieder weiter?](#im-stuck-whats-the-fastest-way-to-get-unstuck)
  * [Was ist der empfohlene Weg, OpenClaw zu installieren und einzurichten?](#whats-the-recommended-way-to-install-and-set-up-openclaw)
  * [Wie öffne ich das Dashboard nach dem Onboarding?](#how-do-i-open-the-dashboard-after-onboarding)
  * [Wie authentifiziere ich das Dashboard (Token) auf localhost vs. remote?](#how-do-i-authenticate-the-dashboard-token-on-localhost-vs-remote)
  * [Welche Runtime benötige ich?](#what-runtime-do-i-need)
  * [Läuft es auf dem Raspberry Pi?](#does-it-run-on-raspberry-pi)
  * [Gibt es Tipps für Installationen auf dem Raspberry Pi?](#any-tips-for-raspberry-pi-installs)
  * [Es bleibt bei „wake up my friend“ hängen / das Onboarding wird nicht abgeschlossen. Was jetzt?](#it-is-stuck-on-wake-up-my-friend-onboarding-will-not-hatch-what-now)
  * [Kann ich mein Setup auf einen neuen Rechner (Mac mini) migrieren, ohne das Onboarding neu zu durchlaufen?](#can-i-migrate-my-setup-to-a-new-machine-mac-mini-without-redoing-onboarding)
  * [Wo sehe ich, was in der neuesten Version neu ist?](#where-do-i-see-whats-new-in-the-latest-version)
  * [Ich kann nicht auf docs.openclaw.ai zugreifen (SSL-Fehler). Was jetzt?](#i-cant-access-docsopenclawai-ssl-error-what-now)
  * [Was ist der Unterschied zwischen stable und beta?](#whats-the-difference-between-stable-and-beta)
* [Wie installiere ich die Beta-Version und worin besteht der Unterschied zwischen Beta und Dev?](#how-do-i-install-the-beta-version-and-whats-the-difference-between-beta-and-dev)
  * [Wie kann ich die neueste Version testen?](#how-do-i-try-the-latest-bits)
  * [Wie lange dauern Installation und Onboarding in der Regel?](#how-long-does-install-and-onboarding-usually-take)
  * [Installationsprogramm hängt? Wie bekomme ich mehr Ausgaben?](#installer-stuck-how-do-i-get-more-feedback)
  * [Windows-Installation meldet, dass Git nicht gefunden wurde oder openclaw nicht erkannt wird](#windows-install-says-git-not-found-or-openclaw-not-recognized)
  * [Die Dokumentation hat meine Frage nicht beantwortet – wie bekomme ich eine bessere Antwort?](#the-docs-didnt-answer-my-question-how-do-i-get-a-better-answer)
  * [Wie installiere ich OpenClaw unter Linux?](#how-do-i-install-openclaw-on-linux)
  * [Wie installiere ich OpenClaw auf einem VPS?](#how-do-i-install-openclaw-on-a-vps)
  * [Wo finde ich die Installationsanleitungen für Cloud/VPS?](#where-are-the-cloudvps-install-guides)
  * [Kann ich OpenClaw anweisen, sich selbst zu aktualisieren?](#can-i-ask-openclaw-to-update-itself)
  * [Was macht der Onboarding-Assistent genau?](#what-does-the-onboarding-wizard-actually-do)
  * [Benötige ich ein Claude- oder OpenAI-Abonnement, um das auszuführen?](#do-i-need-a-claude-or-openai-subscription-to-run-this)
  * [Kann ich ein Claude-Max-Abonnement ohne API-Schlüssel nutzen?](#can-i-use-claude-max-subscription-without-an-api-key)
  * [Wie funktioniert die „setup-token“-Authentifizierung von Anthropic?](#how-does-anthropic-setuptoken-auth-work)
  * [Wo finde ich ein Setup-Token von Anthropic?](#where-do-i-find-an-anthropic-setuptoken)
  * [Unterstützt OpenClaw die Authentifizierung über ein Claude-Abonnement (Claude Code OAuth)?](#do-you-support-claude-subscription-auth-claude-code-oauth)
  * [Warum erhalte ich `HTTP 429: rate_limit_error` von Anthropic?](#why-am-i-seeing-http-429-ratelimiterror-from-anthropic)
  * [Wird AWS Bedrock unterstützt?](#is-aws-bedrock-supported)
  * [Wie funktioniert die Codex-Authentifizierung?](#how-does-codex-auth-work)
  * [Unterstützt ihr die abonnementbasierte OpenAI-Authentifizierung (Codex OAuth)?](#do-you-support-openai-subscription-auth-codex-oauth)
  * [Wie richte ich Gemini-CLI-OAuth ein](#how-do-i-set-up-gemini-cli-oauth)
  * [Ist ein lokales Modell für lockere Chats ausreichend?](#is-a-local-model-ok-for-casual-chats)
  * [Wie stelle ich sicher, dass der Traffic gehosteter Modelle in einer bestimmten Region bleibt?](#how-do-i-keep-hosted-model-traffic-in-a-specific-region)
  * [Muss ich dafür einen Mac Mini kaufen?](#do-i-have-to-buy-a-mac-mini-to-install-this)
  * [Benötige ich einen Mac mini für die iMessage-Unterstützung?](#do-i-need-a-mac-mini-for-imessage-support)
  * [Wenn ich einen Mac mini kaufe, um OpenClaw darauf zu betreiben, kann ich ihn mit meinem MacBook Pro verbinden?](#if-i-buy-a-mac-mini-to-run-openclaw-can-i-connect-it-to-my-macbook-pro)
  * [Kann ich Bun nutzen?](#can-i-use-bun)
  * [Telegram: Was gehört in `allowFrom`?](#telegram-what-goes-in-allowfrom)
  * [Können mehrere Personen eine WhatsApp-Nummer mit verschiedenen OpenClaw-Instanzen verwenden?](#can-multiple-people-use-one-whatsapp-number-with-different-openclaw-instances)
  * [Kann ich einen „Fast Chat“-agent und einen „Opus for Coding“-agent ausführen?](#can-i-run-a-fast-chat-agent-and-an-opus-for-coding-agent)
  * [Funktioniert Homebrew unter Linux?](#does-homebrew-work-on-linux)
  * [Was ist der Unterschied zwischen der hackbaren Git-Installation und der npm-Installation?](#whats-the-difference-between-the-hackable-git-install-and-npm-install)
  * [Kann ich später zwischen npm- und Git-Installationen wechseln?](#can-i-switch-between-npm-and-git-installs-later)
  * [Sollte ich das Gateway auf meinem Laptop oder auf einem VPS laufen lassen?](#should-i-run-the-gateway-on-my-laptop-or-a-vps)
  * [Wie wichtig ist es, OpenClaw auf einem dedizierten Rechner auszuführen?](#how-important-is-it-to-run-openclaw-on-a-dedicated-machine)
  * [Was sind die Mindestanforderungen für den VPS und das empfohlene Betriebssystem?](#what-are-the-minimum-vps-requirements-and-recommended-os)
  * [Kann ich OpenClaw in einer VM ausführen und welche Anforderungen gibt es?](#can-i-run-openclaw-in-a-vm-and-what-are-the-requirements)
* [Was ist OpenClaw?](#what-is-openclaw)
  * [Was ist OpenClaw in einem Absatz?](#what-is-openclaw-in-one-paragraph)
  * [Was ist das Wertversprechen?](#whats-the-value-proposition)
  * [Ich habe es gerade eingerichtet – was sollte ich als Erstes tun?](#i-just-set-it-up-what-should-i-do-first)
  * [Was sind die fünf wichtigsten alltäglichen Anwendungsfälle für OpenClaw?](#what-are-the-top-five-everyday-use-cases-for-openclaw)
  * [Kann OpenClaw bei Lead-Generierung, Outreach, Anzeigen und Blogs für ein SaaS-Produkt helfen?](#can-openclaw-help-with-lead-gen-outreach-ads-and-blogs-for-a-saas)
  * [Welche Vorteile gibt es im Vergleich zu Claude Code für die Webentwicklung?](#what-are-the-advantages-vs-claude-code-for-web-development)
* [Fähigkeiten und Automatisierung](#skills-and-automation)
  * [Wie passe ich Fähigkeiten an, ohne das Repo „dirty“ zu machen?](#how-do-i-customize-skills-without-keeping-the-repo-dirty)
  * [Kann ich Fähigkeiten aus einem benutzerdefinierten Ordner laden?](#can-i-load-skills-from-a-custom-folder)
  * [Wie kann ich verschiedene Modelle für unterschiedliche Aufgaben verwenden?](#how-can-i-use-different-models-for-different-tasks)
  * [Der Bot friert bei rechenintensiven Aufgaben ein. Wie kann ich das auslagern?](#the-bot-freezes-while-doing-heavy-work-how-do-i-offload-that)
  * [Cron oder Erinnerungen werden nicht ausgelöst. Was sollte ich überprüfen?](#cron-or-reminders-do-not-fire-what-should-i-check)
  * [Wie installiere ich Fähigkeiten unter Linux?](#how-do-i-install-skills-on-linux)
  * [Kann OpenClaw Aufgaben nach Zeitplan oder dauerhaft im Hintergrund ausführen?](#can-openclaw-run-tasks-on-a-schedule-or-continuously-in-the-background)
  * [Kann ich Apple-/macOS-exklusive Fähigkeiten unter Linux ausführen?](#can-i-run-applemacosonly-skills-from-linux)
  * [Gibt es eine Notion- oder HeyGen-Integration?](#do-you-have-a-notion-or-heygen-integration)
  * [Wie installiere ich die Chrome-Erweiterung zur Browser-Übernahme?](#how-do-i-install-the-chrome-extension-for-browser-takeover)
* [Sandboxing und Speicherverwaltung](#sandboxing-and-memory)
  * [Gibt es eine dedizierte Dokumentation zur sandbox?](#is-there-a-dedicated-sandboxing-doc)
  * [Wie binde ich einen Host-Ordner in die sandbox ein?](#how-do-i-bind-a-host-folder-into-the-sandbox)
  * [Wie funktioniert Memory?](#how-does-memory-work)
  * [Memory „vergisst“ ständig Dinge. Wie sorge ich dafür, dass sie erhalten bleiben?](#memory-keeps-forgetting-things-how-do-i-make-it-stick)
  * [Bleibt Memory für immer bestehen? Wo liegen die Grenzen?](#does-memory-persist-forever-what-are-the-limits)
  * [Erfordert die semantische Memory-Suche einen OpenAI-API-Key?](#does-semantic-memory-search-require-an-openai-api-key)
* [Wo die Daten auf dem Datenträger liegen](#where-things-live-on-disk)
  * [Werden alle Daten, die mit OpenClaw verwendet werden, lokal gespeichert?](#is-all-data-used-with-openclaw-saved-locally)
  * [Wo speichert OpenClaw seine Daten?](#where-does-openclaw-store-its-data)
  * [Wo sollten AGENTS.md / SOUL.md / USER.md / MEMORY.md abgelegt werden?](#where-should-agentsmd-soulmd-usermd-memorymd-live)
  * [Was ist die empfohlene Backup-Strategie?](#whats-the-recommended-backup-strategy)
  * [Wie deinstalliere ich OpenClaw vollständig?](#how-do-i-completely-uninstall-openclaw)
  * [Können Agenten auch außerhalb des Arbeitsbereichs arbeiten?](#can-agents-work-outside-the-workspace)
  * [Ich bin im Remote-Modus – wo befindet sich der Sitzungsspeicher?](#im-in-remote-mode-where-is-the-session-store)
* [Konfigurationsgrundlagen](#config-basics)
  * [In welchem Format liegt die Konfiguration vor? Wo befindet sie sich?](#what-format-is-the-config-where-is-it)
  * [Ich habe `gateway.bind: "lan"` (oder `"tailnet"`) gesetzt und jetzt lauscht nichts mehr / die UI sagt „unauthorized“](#i-set-gatewaybind-lan-or-tailnet-and-now-nothing-listens-the-ui-says-unauthorized)
  * [Warum benötige ich jetzt ein Token auf localhost?](#why-do-i-need-a-token-on-localhost-now)
  * [Muss ich nach Änderungen an der Konfiguration neu starten?](#do-i-have-to-restart-after-changing-config)
  * [Wie aktiviere ich Websuche (und Web-Fetch)?](#how-do-i-enable-web-search-and-web-fetch)
  * [`config.apply` hat meine Konfiguration gelöscht. Wie kann ich sie wiederherstellen und das in Zukunft vermeiden?](#configapply-wiped-my-config-how-do-i-recover-and-avoid-this)
  * [Wie betreibe ich ein zentrales Gateway mit spezialisierten Workern über mehrere Geräte hinweg?](#how-do-i-run-a-central-gateway-with-specialized-workers-across-devices)
  * [Kann der OpenClaw-Browser headless laufen?](#can-the-openclaw-browser-run-headless)
  * [Wie nutze ich Brave zur Browser-Steuerung?](#how-do-i-use-brave-for-browser-control)
* [Remote-Gateways + Knoten](#remote-gateways-nodes)
  * [Wie werden Befehle zwischen Telegram, dem Gateway und Knoten weitergeleitet?](#how-do-commands-propagate-between-telegram-the-gateway-and-nodes)
  * [Wie kann mein Agent auf meinen Computer zugreifen, wenn das Gateway auf einem entfernten Server gehostet ist?](#how-can-my-agent-access-my-computer-if-the-gateway-is-hosted-remotely)
  * [Tailscale ist verbunden, aber ich bekomme keine Antworten. Was nun?](#tailscale-is-connected-but-i-get-no-replies-what-now)
  * [Können zwei OpenClaw-Instanzen miteinander kommunizieren (lokal + VPS)?](#can-two-openclaw-instances-talk-to-each-other-local-vps)
  * [Brauche ich separate VPS-Instanzen für mehrere Agenten?](#do-i-need-separate-vpses-for-multiple-agents)
  * [Gibt es einen Vorteil, einen Knoten auf meinem persönlichen Laptop zu nutzen statt per SSH von einem VPS?](#is-there-a-benefit-to-using-a-node-on-my-personal-laptop-instead-of-ssh-from-a-vps)
  * [Führen Knoten einen Gateway-Dienst aus?](#do-nodes-run-a-gateway-service)
  * [Gibt es eine API-/RPC-Methode, um die Konfiguration anzuwenden?](#is-there-an-api-rpc-way-to-apply-config)
  * [Was ist eine minimale „sinnvolle“ Konfiguration für eine erste Installation?](#whats-a-minimal-sane-config-for-a-first-install)
  * [Wie richte ich Tailscale auf einem VPS ein und verbinde mich von meinem Mac aus?](#how-do-i-set-up-tailscale-on-a-vps-and-connect-from-my-mac)
  * [Wie verbinde ich einen Mac-Knoten mit einem entfernten Gateway (Tailscale Serve)?](#how-do-i-connect-a-mac-node-to-a-remote-gateway-tailscale-serve)
  * [Soll ich auf einem zweiten Laptop installieren oder einfach einen Knoten hinzufügen?](#should-i-install-on-a-second-laptop-or-just-add-a-node)
* [Umgebungsvariablen und Laden von .env-Dateien](#env-vars-and-env-loading)
  * [Wie lädt OpenClaw Umgebungsvariablen?](#how-does-openclaw-load-environment-variables)
  * [„Ich habe das Gateway über den Dienst gestartet und meine Env-Vars sind verschwunden.“ Was nun?](#i-started-the-gateway-via-the-service-and-my-env-vars-disappeared-what-now)
  * [Ich habe `COPILOT_GITHUB_TOKEN` gesetzt, aber der Modellstatus zeigt „Shell-Env: off“. Warum?](#i-set-copilotgithubtoken-but-models-status-shows-shell-env-off-why)
* [Sitzungen &amp; mehrere Chats](#sessions-multiple-chats)
  * [Wie starte ich ein neues Gespräch?](#how-do-i-start-a-fresh-conversation)
  * [Werden Sitzungen automatisch zurückgesetzt, wenn ich nie `/new` sende?](#do-sessions-reset-automatically-if-i-never-send-new)
  * [Gibt es eine Möglichkeit, ein Team aus OpenClaw-Instanzen aufzubauen – einen CEO und viele Agenten?](#is-there-a-way-to-make-a-team-of-openclaw-instances-one-ceo-and-many-agents)
  * [Warum wurde der Kontext mitten in der Aufgabe abgeschnitten? Wie verhindere ich das?](#why-did-context-get-truncated-midtask-how-do-i-prevent-it)
  * [Wie setze ich OpenClaw vollständig zurück, ohne es zu deinstallieren?](#how-do-i-completely-reset-openclaw-but-keep-it-installed)
  * [Ich erhalte „context too large“-Fehlermeldungen – wie setze ich zurück oder komprimiere?](#im-getting-context-too-large-errors-how-do-i-reset-or-compact)
  * [Warum sehe ich „LLM request rejected: messages.N.content.X.tool&#95;use.input: Field required“?](#why-am-i-seeing-llm-request-rejected-messagesncontentxtooluseinput-field-required)
  * [Warum bekomme ich alle 30 Minuten Herzschlag-Nachrichten?](#why-am-i-getting-heartbeat-messages-every-30-minutes)
  * [Muss ich ein „Bot-Konto“ zu einer WhatsApp-Gruppe hinzufügen?](#do-i-need-to-add-a-bot-account-to-a-whatsapp-group)
  * [Wie erhalte ich die JID einer WhatsApp-Gruppe?](#how-do-i-get-the-jid-of-a-whatsapp-group)
  * [Warum antwortet OpenClaw nicht in einer Gruppe?](#why-doesnt-openclaw-reply-in-a-group)
  * [Teilen Gruppen/Threads Kontext mit DMs?](#do-groupsthreads-share-context-with-dms)
  * [Wie viele Arbeitsbereiche und Agenten kann ich erstellen?](#how-many-workspaces-and-agents-can-i-create)
  * [Kann ich mehrere Bots oder Chats gleichzeitig ausführen (Slack), und wie richte ich das ein?](#can-i-run-multiple-bots-or-chats-at-the-same-time-slack-and-how-should-i-set-that-up)
* [Modelle: Voreinstellungen, Auswahl, Aliasse, Wechsel](#models-defaults-selection-aliases-switching)
  * [Was ist das „Standardmodell“?](#what-is-the-default-model)
  * [Welches Modell empfehlt ihr?](#what-model-do-you-recommend)
  * [Wie wechsle ich das Modell, ohne meine Konfiguration zu löschen?](#how-do-i-switch-models-without-wiping-my-config)
  * [Kann ich selbstgehostete Modelle (llama.cpp, vLLM, Ollama) verwenden?](#can-i-use-selfhosted-models-llamacpp-vllm-ollama)
  * [Welche Modelle verwenden OpenClaw, Flawd und Krill?](#what-do-openclaw-flawd-and-krill-use-for-models)
  * [Wie wechsle ich Modelle im laufenden Betrieb (ohne Neustart)?](#how-do-i-switch-models-on-the-fly-without-restarting)
  * [Kann ich GPT 5.2 für Alltagsaufgaben und Codex 5.2 fürs Programmieren verwenden?](#can-i-use-gpt-52-for-daily-tasks-and-codex-52-for-coding)
  * [Warum sehe ich „Model … is not allowed“ und bekomme dann keine Antwort?](#why-do-i-see-model-is-not-allowed-and-then-no-reply)
  * [Warum sehe ich „Unknown model: minimax/MiniMax-M2.1“?](#why-do-i-see-unknown-model-minimaxminimaxm21)
  * [Kann ich MiniMax als Standard und OpenAI für komplexe Aufgaben verwenden?](#can-i-use-minimax-as-my-default-and-openai-for-complex-tasks)
  * [Sind opus / sonnet / gpt eingebaute Shortcuts?](#are-opus-sonnet-gpt-builtin-shortcuts)
  * [Wie definiere oder überschreibe ich Modell‑Shortcuts (Aliasse)?](#how-do-i-defineoverride-model-shortcuts-aliases)
  * [Wie füge ich Modelle von anderen Anbietern wie OpenRouter oder Z.AI hinzu?](#how-do-i-add-models-from-other-providers-like-openrouter-or-zai)
* [Modell-Failover und „All models failed“](#model-failover-and-all-models-failed)
  * [Wie funktioniert das Failover?](#how-does-failover-work)
  * [Was bedeutet dieser Fehler?](#what-does-this-error-mean)
  * [Checkliste zur Behebung von `No credentials found for profile "anthropic:default"`](#fix-checklist-for-no-credentials-found-for-profile-anthropicdefault)
  * [Warum wurde außerdem auch Google Gemini ausprobiert und ist fehlgeschlagen?](#why-did-it-also-try-google-gemini-and-fail)
* [Auth-Profile: Was sie sind und wie du sie verwaltest](#auth-profiles-what-they-are-and-how-to-manage-them)
  * [Was ist ein Auth-Profil?](#what-is-an-auth-profile)
  * [Was sind typische Profil-IDs?](#what-are-typical-profile-ids)
  * [Kann ich festlegen, welches Auth-Profil zuerst verwendet wird?](#can-i-control-which-auth-profile-is-tried-first)
  * [OAuth vs API-Schlüssel: Was ist der Unterschied?](#oauth-vs-api-key-whats-the-difference)
* [Gateway: Ports, „läuft bereits“ und Remote-Modus](#gateway-ports-already-running-and-remote-mode)
  * [Welchen Port verwendet das Gateway?](#what-port-does-the-gateway-use)
  * [Warum zeigt `openclaw gateway status` `Runtime: running`, aber `RPC probe: failed`?](#why-does-openclaw-gateway-status-say-runtime-running-but-rpc-probe-failed)
  * [Warum zeigt `openclaw gateway status` `Config (cli)` und `Config (service)` unterschiedlich?](#why-does-openclaw-gateway-status-show-config-cli-and-config-service-different)
  * [Was bedeutet „another gateway instance is already listening“?](#what-does-another-gateway-instance-is-already-listening-mean)
  * [Wie führe ich OpenClaw im Remote-Modus aus (Client verbindet sich mit einem entfernten Gateway)?](#how-do-i-run-openclaw-in-remote-mode-client-connects-to-a-gateway-elsewhere)
  * [Die Control UI zeigt „unauthorized“ (oder verbindet sich ständig neu). Was nun?](#the-control-ui-says-unauthorized-or-keeps-reconnecting-what-now)
  * [Ich habe `gateway.bind: "tailnet"` gesetzt, aber der Bind-Vorgang schlägt fehl / nichts lauscht](#i-set-gatewaybind-tailnet-but-it-cant-bind-nothing-listens)
  * [Kann ich mehrere Gateways auf demselben Host ausführen?](#can-i-run-multiple-gateways-on-the-same-host)
  * [Was bedeutet „invalid handshake“ / Code 1008?](#what-does-invalid-handshake-code-1008-mean)
* [Logging und Debugging](#logging-and-debugging)
  * [Wo finde ich die Logs?](#where-are-logs)
  * [Wie starte, stoppe oder starte ich den Gateway-Dienst neu?](#how-do-i-startstoprestart-the-gateway-service)
  * [Ich habe mein Terminal unter Windows geschlossen – wie starte ich OpenClaw neu?](#i-closed-my-terminal-on-windows-how-do-i-restart-openclaw)
  * [Der Gateway läuft, aber Antworten kommen nie an. Was sollte ich prüfen?](#the-gateway-is-up-but-replies-never-arrive-what-should-i-check)
  * [&quot;Disconnected from gateway: no reason&quot; – was nun?](#disconnected-from-gateway-no-reason-what-now)
  * [Telegram setMyCommands schlägt mit Netzwerkfehlern fehl. Was sollte ich prüfen?](#telegram-setmycommands-fails-with-network-errors-what-should-i-check)
  * [TUI zeigt keine Ausgabe. Was sollte ich prüfen?](#tui-shows-no-output-what-should-i-check)
  * [Wie stoppe ich den Gateway vollständig und starte ihn dann neu?](#how-do-i-completely-stop-then-start-the-gateway)
  * [ELI5: `openclaw gateway restart` vs `openclaw gateway`](#eli5-openclaw-gateway-restart-vs-openclaw-gateway)
  * [Was ist der schnellste Weg, um mehr Details zu erhalten, wenn etwas fehlschlägt?](#whats-the-fastest-way-to-get-more-details-when-something-fails)
* [Medien &amp; Anhänge](#media-attachments)
  * [Mein Skill hat ein Bild/PDF erzeugt, aber es wurde nichts verschickt](#my-skill-generated-an-imagepdf-but-nothing-was-sent)
* [Sicherheit und Zugriffskontrolle](#security-and-access-control)
  * [Ist es sicher, OpenClaw für eingehende DMs öffentlich erreichbar zu machen?](#is-it-safe-to-expose-openclaw-to-inbound-dms)
  * [Ist Prompt Injection nur bei öffentlichen Bots ein Problem?](#is-prompt-injection-only-a-concern-for-public-bots)
  * [Sollte mein Bot eine eigene E-Mail-Adresse, ein eigenes GitHub-Konto oder eine eigene Telefonnummer haben](#should-my-bot-have-its-own-email-github-account-or-phone-number)
  * [Kann ich ihm Autonomie über meine Textnachrichten geben und ist das sicher](#can-i-give-it-autonomy-over-my-text-messages-and-is-that-safe)
  * [Kann ich günstigere Modelle für persönliche Assistenzaufgaben verwenden?](#can-i-use-cheaper-models-for-personal-assistant-tasks)
  * [Ich habe `/start` in Telegram ausgeführt, aber keinen Kopplungscode erhalten](#i-ran-start-in-telegram-but-didnt-get-a-pairing-code)
  * [WhatsApp: Schreibt es meine Kontakte an? Wie funktioniert die Kopplung?](#whatsapp-will-it-message-my-contacts-how-does-pairing-work)
* [Chat-Befehle, Aufgaben abbrechen und „es hört nicht auf“](#chat-commands-aborting-tasks-and-it-wont-stop)
  * [Wie verhindere ich, dass interne Systemnachrichten im Chat angezeigt werden?](#how-do-i-stop-internal-system-messages-from-showing-in-chat)
  * [Wie stoppe oder breche ich eine laufende Aufgabe ab?](#how-do-i-stopcancel-a-running-task)
  * [Wie sende ich eine Discord‑Nachricht aus Telegram? („Cross-context messaging denied“)](#how-do-i-send-a-discord-message-from-telegram-crosscontext-messaging-denied)
  * [Warum wirkt es so, als würde der Bot Schnellfeuer‑Nachrichten „ignorieren“?](#why-does-it-feel-like-the-bot-ignores-rapidfire-messages)

<div id="first-60-seconds-if-somethings-broken">
  ## Erste 60 Sekunden, wenn etwas nicht funktioniert
</div>

1. **Kurzer Status (erster Check)**
   ```bash
   openclaw status
   ```
   Schnelle lokale Zusammenfassung: OS + Update, Gateway-/Dienst-Erreichbarkeit, Agenten/Sitzungen, Anbieter-Konfiguration + Laufzeitprobleme (wenn das Gateway erreichbar ist).

2. **Einfügbarer Bericht (sicher teilbar)**
   ```bash
   openclaw status --all
   ```
   Diagnose im Read-only-Modus mit Log-Ausschnitt (Tokens geschwärzt).

3. **Daemon- + Port-Status**
   ```bash
   openclaw gateway status
   ```
   Zeigt Supervisor-Laufzeit vs. RPC-Erreichbarkeit, die Ziel-URL des Probes und welche Konfiguration der Dienst voraussichtlich verwendet hat.

4. **Tiefe Prüfungen**
   ```bash
   openclaw status --deep
   ```
   Führt Gateway-Health-Checks + Anbieter-Prüfungen aus (erfordert ein erreichbares Gateway). Siehe [Health](/de/gateway/health).

5. **Das neueste Log verfolgen**
   ```bash
   openclaw logs --follow
   ```
   Wenn RPC nicht erreichbar ist, verwende stattdessen:
   ```bash
   tail -f "$(ls -t /tmp/openclaw/openclaw-*.log | head -1)"
   ```
   Datei-Logs sind getrennt von Dienst-Logs; siehe [Logging](/de/logging) und [Troubleshooting](/de/gateway/troubleshooting).

6. **Den Doctor ausführen (Reparaturen)**
   ```bash
   openclaw doctor
   ```
   Repariert/migriert Konfiguration/Zustand + führt Health-Checks aus. Siehe [Doctor](/de/gateway/doctor).

7. **Gateway-Snapshot**
   ```bash
   openclaw health --json
   openclaw health --verbose   # zeigt die Ziel-URL + den Konfigurationspfad bei Fehlern
   ```
   Fordert vom laufenden Gateway einen vollständigen Snapshot an (nur über WS verfügbar). Siehe [Health](/de/gateway/health).

<div id="quick-start-and-first-run-setup">
  ## Schnellstart und Ersteinrichtung
</div>

<div id="im-stuck-whats-the-fastest-way-to-get-unstuck">
  ### Ich stecke fest – wie komme ich am schnellsten wieder weiter?
</div>

Verwende einen lokalen AI-Agent, der **deinen Rechner sehen kann**. Das ist deutlich effektiver
als in Discord zu fragen, weil die meisten „Ich stecke fest“-Fälle **lokale Konfigurations- oder
Umgebungsprobleme** sind, die externe Helfer nicht einsehen können.

* **Claude Code**: https://www.anthropic.com/claude-code/
* **OpenAI Codex**: https://openai.com/codex/

Diese Tools können das Repo lesen, Befehle ausführen, Logs einsehen und dir helfen,
dein Setup auf Maschinenebene zu reparieren (PATH, Services, Berechtigungen, Auth-Dateien). Gib
ihnen den **vollständigen Checkout des Quellcodes** über die hackbare (git)-Installation:

```bash
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --install-method git
```

Dadurch wird OpenClaw **aus einem Git-Checkout installiert**, sodass der agent den Code und die Doku lesen und über die exakte Version, die du verwendest, nachdenken kann. Du kannst später jederzeit wieder auf die stabile Version wechseln, indem du das Installationsprogramm ohne `--install-method git` erneut ausführst.

Tipp: Bitte den agent, die Fehlerbehebung **zu planen und zu überwachen** (Schritt für Schritt), und führe dann nur die notwendigen Befehle aus. So bleiben Änderungen klein und leichter nachvollziehbar.

Wenn du einen echten Bug oder Fix findest, erstelle bitte ein GitHub-Issue oder sende einen PR:
https://github.com/openclaw/openclaw/issues
https://github.com/openclaw/openclaw/pulls

Starte mit diesen Befehlen (teile die Ausgaben, wenn du um Hilfe bittest):

```bash
openclaw status
openclaw models status
openclaw doctor
```

Wozu sie dienen:

* `openclaw status`: schneller Überblick über Gateway-/agent-Status + Basis-Konfiguration.
* `openclaw models status`: prüft anbieter-Authentifizierung + Modellverfügbarkeit.
* `openclaw doctor`: validiert und repariert gängige Konfigurations-/Zustandsprobleme.

Weitere nützliche CLI-Checks: `openclaw status --all`, `openclaw logs --follow`,
`openclaw gateway status`, `openclaw health --verbose`.

Schneller Debug-Zyklus: [Erste 60 Sekunden, wenn etwas nicht funktioniert](#first-60-seconds-if-somethings-broken).
Installationsdokumentation: [Installation](/de/install), [Installer-Flags](/de/install/installer), [Aktualisierung](/de/install/updating).

<div id="whats-the-recommended-way-to-install-and-set-up-openclaw">
  ### Was ist die empfohlene Vorgehensweise, OpenClaw zu installieren und einzurichten?
</div>

Das Repo empfiehlt, OpenClaw direkt aus dem Quellcode zu starten und den Onboarding-Assistenten zu verwenden:

```bash
curl -fsSL https://openclaw.bot/install.sh | bash
openclaw onboard --install-daemon
```

Der Wizard kann UI-Assets auch automatisch erstellen. Nach dem Onboarding betreibst du das Gateway in der Regel auf Port **18789**.

Aus dem Quellcode (Contributors/Dev):

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm build
pnpm ui:build # installiert UI-Abhängigkeiten beim ersten Durchlauf automatisch
openclaw onboard
```

Wenn du OpenClaw noch nicht global installiert hast, führe `pnpm openclaw onboard` aus.

<div id="how-do-i-open-the-dashboard-after-onboarding">
  ### Wie öffne ich das Dashboard nach dem Onboarding
</div>

Der Assistent öffnet direkt nach dem Onboarding deinen Browser mit einer tokenisierten Dashboard-URL und gibt den vollständigen Link (mit Token) auch in der Zusammenfassung aus. Lass diesen Tab geöffnet; falls der Browser nicht gestartet wurde, kopiere den angezeigten Link und füge ihn auf demselben Rechner ein. Tokens bleiben lokal auf deinem Host – es wird nichts aus dem Browser abgerufen.

<div id="how-do-i-authenticate-the-dashboard-token-on-localhost-vs-remote">
  ### Wie authentifiziere ich das Dashboard-Token auf localhost vs. remote
</div>

**Localhost (gleiche Maschine):**

* Öffne `http://127.0.0.1:18789/`.
* Falls nach einer Authentifizierung gefragt wird, führe `openclaw dashboard` aus und verwende den tokenisierten Link (`?token=...`).
* Das Token ist derselbe Wert wie `gateway.auth.token` (oder `OPENCLAW_GATEWAY_TOKEN`) und wird von der UI nach dem ersten Laden gespeichert.

**Nicht auf localhost:**

* **Tailscale Serve** (empfohlen): Loopback-Bindung beibehalten, `openclaw gateway --tailscale serve` ausführen, `https://<magicdns>/` öffnen. Wenn `gateway.auth.allowTailscale` auf `true` steht, reichen die Identity-Header für die Authentifizierung aus (kein Token nötig).
* **Tailnet-Bindung**: `openclaw gateway --bind tailnet --token "<token>"` ausführen, `http://<tailscale-ip>:18789/` öffnen, Token in den Dashboard-Einstellungen einfügen.
* **SSH-Tunnel**: `ssh -N -L 18789:127.0.0.1:18789 user@host` und dann `http://127.0.0.1:18789/?token=...` aus `openclaw dashboard` im Browser öffnen.

Siehe [Dashboard](/de/web/dashboard) und [Web surfaces](/de/web) für Bind-Modi und Details zur Authentifizierung.

<div id="what-runtime-do-i-need">
  ### Welche Laufzeitumgebung brauche ich
</div>

Node **&gt;= 22** ist erforderlich. `pnpm` wird empfohlen. Bun wird für das Gateway **nicht empfohlen**.

<div id="does-it-run-on-raspberry-pi">
  ### Läuft es auf dem Raspberry Pi
</div>

Ja. Das Gateway ist ressourcenschonend – in der Doku sind **512MB–1GB RAM**, **1 Kern** und etwa **500MB**
Speicherplatz als ausreichend für die private Nutzung angegeben, und es wird erwähnt, dass ein **Raspberry Pi 4 es ausführen kann**.

Wenn du etwas mehr Spielraum möchtest (Logs, Medien, andere Dienste), sind **2GB empfohlen**, aber das ist
kein hartes Minimum.

Tipp: Ein kleiner Pi/VPS kann das Gateway hosten, und du kannst **knoten** auf deinem Laptop/Smartphone koppeln für
lokalen Bildschirm-/Kamera-/Canvas-Zugriff oder die Ausführung von Befehlen. Siehe [Nodes](/de/nodes).

<div id="any-tips-for-raspberry-pi-installs">
  ### Gibt es Tipps für Raspberry-Pi-Installationen
</div>

Kurzfassung: Es funktioniert, aber rechne mit ein paar Ecken und Kanten.

* Verwende ein **64-Bit**-OS und setze Node auf Version &gt;= 22.
* Bevorzuge die **hackbare (Git-)Installation**, damit du Logs sehen und schnell aktualisieren kannst.
* Starte ohne Channels/Fähigkeiten und füge sie dann nacheinander hinzu.
* Wenn du auf seltsame Probleme mit Binärdateien stößt, ist es meist ein **ARM-Kompatibilitäts**problem.

Dokumentation: [Linux](/de/platforms/linux), [Installation](/de/install).

<div id="it-is-stuck-on-wake-up-my-friend-onboarding-will-not-hatch-what-now">
  ### Beim Onboarding bleibt „Wake up, my friend“ hängen und es wird nicht fertig. Was jetzt?
</div>

Dieser Bildschirm setzt voraus, dass das Gateway erreichbar und authentifiziert ist. Die TUI sendet
&quot;Wake up, my friend!&quot; beim ersten Start automatisch. Wenn du diese Zeile siehst, aber **keine Antwort**
bekommst und die Token bei 0 bleiben, wurde der agent nie gestartet.

1. Starte das Gateway neu:

```bash
openclaw gateway restart
```

2. Status + Authentifizierung prüfen:

```bash
openclaw status
openclaw models status
openclaw logs --follow
```

3. Wenn es immer noch hängt, führen Sie Folgendes aus:

```bash
openclaw doctor
```

Wenn das Gateway remote ist, stelle sicher, dass die Tunnel- bzw. Tailscale-Verbindung aktiv ist und dass die UI auf das richtige Gateway zeigt. Siehe [Remotezugriff](/de/gateway/remote).

<div id="can-i-migrate-my-setup-to-a-new-machine-mac-mini-without-redoing-onboarding">
  ### Kann ich mein Setup auf einen neuen Mac mini migrieren, ohne das Onboarding erneut durchlaufen zu müssen
</div>

Ja. Kopiere das **State-Verzeichnis** und den **Arbeitsbereich** und führe dann einmal Doctor aus. So bleibt dein Bot „genau derselbe“ (Speicher, Sitzungsverlauf, Authentifizierung und Channel-Status), solange du **beide** Verzeichnisse kopierst:

1. Installiere OpenClaw auf dem neuen Rechner.
2. Kopiere `$OPENCLAW_STATE_DIR` (Standard: `~/.openclaw`) vom alten Rechner.
3. Kopiere deinen Arbeitsbereich (Standard: `~/.openclaw/workspace`).
4. Führe `openclaw doctor` aus und starte den Gateway-Dienst neu.

Damit bleiben Konfiguration, Auth-Profile, WhatsApp-Zugangsdaten, Sitzungen und Speicher erhalten. Wenn du im Remote-Modus bist, denke daran, dass der Gateway-Host den Sitzungsspeicher und den Arbeitsbereich besitzt.

**Wichtig:** Wenn du nur deinen Arbeitsbereich in GitHub committest/pusht, sicherst du **Speicher + Bootstrap-Dateien**, aber **nicht** den Sitzungsverlauf oder Auth-Daten. Diese liegen unter `~/.openclaw/` (zum Beispiel `~/.openclaw/agents/<agentId>/sessions/`).

Verwandt: [Migration](/de/install/migrating), [Wo die Daten auf der Festplatte liegen](/de/help/faq#where-does-openclaw-store-its-data),
[Agent-Arbeitsbereich](/de/concepts/agent-workspace), [Doctor](/de/gateway/doctor),
[Remote-Modus](/de/gateway/remote).

<div id="where-do-i-see-whats-new-in-the-latest-version">
  ### Wo sehe ich, was in der neuesten Version neu ist
</div>

Sieh dir das Changelog auf GitHub an:\
https://github.com/openclaw/openclaw/blob/main/CHANGELOG.md

Die neuesten Einträge stehen ganz oben. Wenn der oberste Abschnitt mit **Unreleased** markiert ist, ist der nächste datierte Abschnitt die zuletzt ausgelieferte Version. Die Einträge sind nach **Highlights**, **Changes** und **Fixes** gruppiert (plus Docs/sonstige Abschnitte, falls nötig).

<div id="i-cant-access-docsopenclawai-ssl-error-what-now">
  ### Ich kann nicht auf docs.openclaw.ai zugreifen – SSL-Fehler. Was nun?
</div>

Einige Comcast/Xfinity-Verbindungen blockieren `docs.openclaw.ai` fälschlicherweise über Xfinity Advanced Security. Deaktiviere es oder setze `docs.openclaw.ai` auf die Allowlist und versuche es dann erneut. Weitere Details: [Troubleshooting](/de/help/troubleshooting#docsopenclawai-shows-an-ssl-error-comcastxfinity).
Bitte hilf uns, die Blockierung zu entfernen, indem du das hier meldest: https://spa.xfinity.com/check&#95;url&#95;status.

Wenn du die Seite weiterhin nicht erreichen kannst, ist die Dokumentation auch gespiegelt auf GitHub verfügbar:
https://github.com/openclaw/openclaw/tree/main/docs

<div id="whats-the-difference-between-stable-and-beta">
  ### Was ist der Unterschied zwischen stable und beta
</div>

**stable** und **beta** sind **npm‑dist‑Tags**, keine separaten Codezweige:

* `latest` = stable
* `beta` = frühes Build zum Testen

Wir veröffentlichen Builds unter **beta**, testen sie und sobald ein Build stabil ist, **befördern wir genau diese Version zu `latest`**. Deshalb können beta und stable auf **dieselbe Version** zeigen.

Änderungen ansehen:\
https://github.com/openclaw/openclaw/blob/main/CHANGELOG.md

<div id="how-do-i-install-the-beta-version-and-whats-the-difference-between-beta-and-dev">
  ### Wie installiere ich die Beta-Version, und was ist der Unterschied zwischen Beta und Dev?
</div>

**Beta** ist der npm‑dist‑Tag `beta` (kann mit `latest` übereinstimmen).
**Dev** ist der aktuelle Stand von `main` (git); wenn veröffentlicht, verwendet er den npm‑dist‑Tag `dev`.

Einzeiler (macOS/Linux):

```bash
curl -fsSL --proto '=https' --tlsv1.2 https://openclaw.bot/install.sh | bash -s -- --beta
```

```bash
curl -fsSL --proto '=https' --tlsv1.2 https://openclaw.bot/install.sh | bash -s -- --install-method git
```

Windows-Installer (PowerShell):
https://openclaw.ai/install.ps1

Weitere Details: [Entwicklungskanäle](/de/install/development-channels) und [Installer-Optionen](/de/install/installer).

<div id="how-long-does-install-and-onboarding-usually-take">
  ### Wie lange dauern Installation und Onboarding normalerweise?
</div>

Grobe Richtwerte:

* **Installation:** 2–5 Minuten
* **Onboarding:** 5–15 Minuten, abhängig davon, wie viele Kanäle/Modelle du konfigurierst

Wenn etwas hängen bleibt, verwende [Installer stuck](/de/help/faq#installer-stuck-how-do-i-get-more-feedback)
und die schnelle Debug-Schleife in [Im stuck](/de/help/faq#im-stuck--whats-the-fastest-way-to-get-unstuck).

<div id="how-do-i-try-the-latest-bits">
  ### Wie kann ich den neuesten Stand ausprobieren
</div>

Zwei Optionen:

1. **Dev-Channel (git checkout):**

```bash
openclaw update --channel dev
```

Damit wechselst du auf den Branch `main` und aktualisierst aus dem Quell-Repository.

2. **Hackbare Installation (über die Installerseite):**

```bash
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --install-method git
```

Damit erhältst du ein lokales Repository, das du bearbeiten und anschließend über Git aktualisieren kannst.

Wenn du stattdessen einen sauberen, manuell erstellten Klon bevorzugst, verwende:

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm build
```

Dokumentation: [Update](/de/cli/update), [Entwicklungskanäle](/de/install/development-channels),
[Installation](/de/install).

<div id="installer-stuck-how-do-i-get-more-feedback">
  ### Installer hängt – wie bekomme ich mehr Feedback?
</div>

Führe den Installer mit **ausführlicher Ausgabe** erneut aus:

```bash
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --verbose
```

Beta-Installation mit ausführlicher Ausgabe:

```bash
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --beta --verbose
```

Für eine leicht anpassbare (Git-)Installation:

```bash
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --install-method git --verbose
```

Weitere Optionen: [Installer-Flags](/de/install/installer).

<div id="windows-install-says-git-not-found-or-openclaw-not-recognized">
  ### Windows-Installation meldet, dass git nicht gefunden wurde oder openclaw nicht erkannt wird
</div>

Zwei häufige Windows-Probleme:

**1) npm-Fehler `spawn git` / `git not found`**

* Installiere **Git for Windows** und stelle sicher, dass `git` in deinem PATH liegt.
* Schließe PowerShell und öffne es erneut, dann starte das Installationsprogramm noch einmal.

**2) openclaw wird nach der Installation nicht erkannt**

* Dein globaler npm-bin-Ordner ist nicht im PATH.
* Prüfe den Pfad:
  ```powershell
  npm config get prefix
  ```
* Stelle sicher, dass `<prefix>\\bin` im PATH ist (auf den meisten Systemen ist das `%AppData%\\npm`).
* Schließe PowerShell und öffne es nach dem Aktualisieren des PATH erneut.

Wenn du das möglichst reibungsloseste Windows-Setup möchtest, verwende **WSL2** statt nativem Windows.
Dokumentation: [Windows](/de/platforms/windows).

<div id="the-docs-didnt-answer-my-question-how-do-i-get-a-better-answer">
  ### Die Dokumentation hat meine Frage nicht beantwortet – wie bekomme ich eine bessere Antwort
</div>

Verwende die **hackbare (Git-)Installation**, damit du den vollständigen Quellcode und die Dokumentation lokal hast, und frage dann
deinen Bot (oder Claude/Codex) *aus diesem Verzeichnis heraus*, damit er das Repository lesen und präzise antworten kann.

```bash
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --install-method git
```

Weitere Details: [Installation](/de/install) und [Installer-Optionen](/de/install/installer).

<div id="how-do-i-install-openclaw-on-linux">
  ### Wie installiere ich OpenClaw unter Linux
</div>

Kurzfassung: Befolge die Linux-Anleitung und führe anschließend den Onboarding-Assistenten aus.

* Schneller Linux-Einstieg + Service-Installation: [Linux](/de/platforms/linux).
* Vollständige Schritt-für-Schritt-Anleitung: [Erste Schritte](/de/start/getting-started).
* Installationsprogramm + Updates: [Installation &amp; Updates](/de/install/updating).

<div id="how-do-i-install-openclaw-on-a-vps">
  ### Wie installiere ich OpenClaw auf einem VPS?
</div>

Jeder Linux-VPS ist geeignet. Installiere OpenClaw auf dem Server und greife dann per SSH/Tailscale auf das Gateway zu.

Anleitungen: [exe.dev](/de/platforms/exe-dev), [Hetzner](/de/platforms/hetzner), [Fly.io](/de/platforms/fly).\
Remote-Zugriff: [Gateway remote](/de/gateway/remote).

<div id="where-are-the-cloudvps-install-guides">
  ### Wo finde ich die Cloud-VPS-Installationsanleitungen
</div>

Wir pflegen einen **Hosting-Hub** mit den gängigen Anbietern. Wähle einen aus und folge der Anleitung:

* [VPS hosting](/de/vps) (alle Anbieter an einem Ort)
* [Fly.io](/de/platforms/fly)
* [Hetzner](/de/platforms/hetzner)
* [exe.dev](/de/platforms/exe-dev)

So funktioniert es in der Cloud: Das **Gateway läuft auf dem Server**, und du greifst
von deinem Laptop/Smartphone über die Control UI (oder Tailscale/SSH) darauf zu. Dein Status und Arbeitsbereich
liegen auf dem Server, behandle den Host also als maßgebliche Quelle („source of truth“) und sichere ihn.

Du kannst **Knoten** (Mac/iOS/Android/headless) mit diesem Cloud-Gateway koppeln, um auf
lokalen Bildschirm/Kamera/Canvas zuzugreifen oder Befehle auf deinem Laptop auszuführen, während das
Gateway in der Cloud bleibt.

Hub: [Platforms](/de/platforms). Remotezugriff: [Gateway remote](/de/gateway/remote).
Knoten: [Nodes](/de/nodes), [Nodes CLI](/de/cli/nodes).

<div id="can-i-ask-openclaw-to-update-itself">
  ### Kann ich OpenClaw bitten, sich selbst zu aktualisieren?
</div>

Kurzantwort: **möglich, aber nicht empfohlen**. Der Update-Vorgang kann den
Gateway neu starten (wodurch die aktive Sitzung verloren geht), eventuell einen
sauberen `git checkout` erfordern und eine Bestätigung abfragen. Sicherer ist es,
Updates als Operator in einer Shell auszuführen.

Verwende die CLI:

```bash
openclaw update
openclaw update status
openclaw update --channel stable|beta|dev
openclaw update --tag <dist-tag|version>
openclaw update --no-restart
```

Wenn du unbedingt aus einem agent heraus automatisieren musst:

```bash
openclaw update --yes --no-restart
openclaw gateway restart
```

Dokumentation: [Update](/de/cli/update), [Aktualisierung](/de/install/updating).

<div id="what-does-the-onboarding-wizard-actually-do">
  ### Was macht der Onboarding‑Assistent eigentlich
</div>

`openclaw onboard` ist der empfohlene Weg für die Einrichtung. Im **lokalen Modus** führt er dich durch:

* **Modell/Auth‑Setup** (Anthropic‑**setup-token** empfohlen für Claude‑Abonnements, OpenAI Codex OAuth wird unterstützt, API‑Schlüssel sind optional, lokale LM‑Studio‑Modelle werden unterstützt)
* **Arbeitsbereich**‑Speicherort + Bootstrap‑Dateien
* **Gateway‑Einstellungen** (bind/port/auth/Tailscale)
* **Anbieter** (WhatsApp, Telegram, Discord, Mattermost (Plugin), Signal, iMessage)
* **Daemon‑Installation** (LaunchAgent auf macOS; systemd‑User‑Unit unter Linux/WSL2)
* **Health‑Checks** und **Fähigkeiten**‑Auswahl

Er warnt außerdem, wenn dein konfiguriertes Modell unbekannt ist oder die Authentifizierung fehlt.

<div id="do-i-need-a-claude-or-openai-subscription-to-run-this">
  ### Brauche ich ein Claude- oder OpenAI-Abonnement, um das zu verwenden?
</div>

Nein. Du kannst OpenClaw entweder mit **API-Schlüsseln** (Anthropic/OpenAI/weitere) oder mit
**rein lokalen Modellen** betreiben, sodass deine Daten auf deinem Gerät bleiben. Abonnements (Claude
Pro/Max oder OpenAI Codex) sind optionale Methoden, um diese Anbieter zu authentifizieren.

Dokumentation: [Anthropic](/de/providers/anthropic), [OpenAI](/de/providers/openai),
[Lokale Modelle](/de/gateway/local-models), [Modelle](/de/concepts/models).

<div id="can-i-use-claude-max-subscription-without-an-api-key">
  ### Kann ich ein Claude-Max-Abonnement ohne API-Schlüssel verwenden?
</div>

Ja. Sie können sich mit einem **setup-token** authentifizieren,
anstatt einen API-Schlüssel zu verwenden. Dies ist der Weg über das Abonnement.

Claude-Pro/Max-Abonnements **enthalten keinen API-Schlüssel**, daher ist dies der
korrekte Ansatz für Abonnementkonten. Wichtig: Sie müssen mit
Anthropic klären, dass diese Nutzung gemäß deren Abonnementrichtlinie und -bedingungen zulässig ist.
Wenn Sie den explizitesten und offiziell unterstützten Weg möchten, verwenden Sie einen Anthropic-API-Schlüssel.

<div id="how-does-anthropic-setuptoken-auth-work">
  ### Wie funktioniert die Anthropic-Setup-Token-Authentifizierung
</div>

`claude setup-token` erzeugt einen **Token-String** über die Claude Code CLI (er ist in der Webkonsole nicht verfügbar). Du kannst ihn auf **jedem beliebigen Rechner** ausführen. Wähle im Assistenten **Anthropic token (paste setup-token)** oder füge ihn mit `openclaw models auth paste-token --provider anthropic` ein. Der Token wird als Auth-Profil für den **anthropic**-Anbieter gespeichert und wie ein API-Schlüssel verwendet (keine automatische Aktualisierung). Weitere Details: [OAuth](/de/concepts/oauth).

<div id="where-do-i-find-an-anthropic-setuptoken">
  ### Wo finde ich ein Anthropic-Setup-Token?
</div>

Es ist **nicht** in der Anthropic Console zu finden. Das Setup-Token wird von der **Claude Code CLI** auf **jedem beliebigen Rechner** generiert:

```bash
claude setup-token
```

Kopiere das ausgegebene Token und wähle dann **Anthropic token (paste setup-token)** im Assistenten. Wenn du es auf dem Gateway-Host ausführen möchtest, verwende `openclaw models auth setup-token --provider anthropic`. Falls du `claude setup-token` an anderer Stelle ausgeführt hast, füge es auf dem Gateway-Host mit `openclaw models auth paste-token --provider anthropic` ein. Siehe [Anthropic](/de/providers/anthropic).

<div id="do-you-support-claude-subscription-auth-claude-promax">
  ### Unterstützt OpenClaw die Authentifizierung über Claude-Abos (Claude Pro/Max)
</div>

Ja — über **setup-token**. OpenClaw verwendet keine Claude Code CLI OAuth-Tokens mehr; verwende einen setup-token oder einen Anthropic API-Schlüssel. Generiere das Token an beliebiger Stelle und füge es auf dem Gateway-Host ein. Siehe [Anthropic](/de/providers/anthropic) und [OAuth](/de/concepts/oauth).

Hinweis: Der Zugriff über Claude-Abos unterliegt den Nutzungsbedingungen von Anthropic. Für den Produktivbetrieb oder Mehrbenutzer-Workloads sind API-Schlüssel in der Regel die sicherere Wahl.

<div id="why-am-i-seeing-http-429-ratelimiterror-from-anthropic">
  ### Warum erhalte ich den HTTP‑429‑Fehler ratelimiterror von Anthropic
</div>

Das bedeutet, dass dein **Anthropic‑Kontingent/Rate‑Limit** für das aktuelle Zeitfenster aufgebraucht ist. Wenn du ein **Claude‑Abonnement** verwendest (Setup‑Token oder Claude Code OAuth), warte, bis das Zeitfenster zurückgesetzt wird, oder upgrade deinen Tarif. Wenn du einen **Anthropic‑API‑Schlüssel** verwendest, prüfe in der Anthropic Console Nutzung und Abrechnung und erhöhe bei Bedarf die Limits.

Tipp: Richte ein **Fallback‑Modell** ein, damit OpenClaw weiter antworten kann, während ein anbieter durch Rate‑Limits eingeschränkt ist. Siehe [Models](/de/cli/models) und [OAuth](/de/concepts/oauth).

<div id="is-aws-bedrock-supported">
  ### Wird AWS Bedrock unterstützt
</div>

Ja – über pi‑ais **Amazon Bedrock (Converse)**-Anbieter mit **manueller Konfiguration**. Du musst AWS-Anmeldedaten und Region auf dem Gateway-Host bereitstellen und einen Bedrock-Anbieter-Eintrag in deiner Modellkonfiguration hinzufügen. Siehe [Amazon Bedrock](/de/bedrock) und [Modellanbieter](/de/providers/models). Wenn du einen verwalteten Key-Workflow bevorzugst, ist ein OpenAI-kompatibler Proxy vor Bedrock weiterhin eine gültige Option.

<div id="how-does-codex-auth-work">
  ### Wie funktioniert die Codex-Authentifizierung?
</div>

OpenClaw unterstützt **OpenAI Code (Codex)** über OAuth (Anmeldung mit ChatGPT). Der Assistent kann den OAuth-Ablauf ausführen und setzt bei Bedarf das Standardmodell auf `openai-codex/gpt-5.2`. Siehe [Modellanbieter](/de/concepts/model-providers) und [Assistent](/de/start/wizard).

<div id="do-you-support-openai-subscription-auth-codex-oauth">
  ### Unterstützt ihr OpenAI Code (Codex) Subscription-OAuth?
</div>

Ja. OpenClaw unterstützt **OpenAI Code (Codex) Subscription-OAuth** vollständig. Der Onboarding-Assistent
kann den OAuth-Flow für dich durchlaufen.

Siehe [OAuth](/de/concepts/oauth), [Modellanbieter](/de/concepts/model-providers) und [Assistent](/de/start/wizard).

<div id="how-do-i-set-up-gemini-cli-oauth">
  ### Wie richte ich Gemini CLI OAuth ein
</div>

Gemini CLI verwendet einen **Plugin-Auth-Flow** und keine Client-ID oder kein Secret in `openclaw.json`.

Schritte:

1. Plugin aktivieren: `openclaw plugins enable google-gemini-cli-auth`
2. Anmelden: `openclaw models auth login --provider google-gemini-cli --set-default`

Dadurch werden OAuth-Tokens in Auth-Profilen auf dem Gateway-Host gespeichert. Details: [Modellanbieter](/de/concepts/model-providers).

<div id="is-a-local-model-ok-for-casual-chats">
  ### Ist ein lokales Modell für lockere Chats okay?
</div>

In der Regel nein. OpenClaw braucht großen Kontext und starke Sicherheit; kleine Modelle kürzen den Kontext und können Daten leaken. Wenn es unbedingt sein muss, betreibe lokal (LM Studio) den **größten** MiniMax-M2.1-Build, den du lokal ausführen kannst, und sieh dir [/gateway/local-models](/de/gateway/local-models) an. Kleinere/quantisierte Modelle erhöhen das Prompt-Injection-Risiko – siehe [Security](/de/gateway/security).

<div id="how-do-i-keep-hosted-model-traffic-in-a-specific-region">
  ### Wie halte ich den Datenverkehr gehosteter Modelle in einer bestimmten Region
</div>

Wähle regionsgebundene Endpunkte. OpenRouter stellt US-gehostete Optionen für MiniMax, Kimi und GLM bereit; wähle die US-gehostete Variante, um Daten in der Region zu halten. Du kannst Anthropic/OpenAI weiterhin zusammen mit diesen aufführen, indem du `models.mode: "merge"` verwendest, sodass Fallbacks verfügbar bleiben, während der von dir ausgewählte, regionsgebundene Anbieter respektiert wird.

<div id="do-i-have-to-buy-a-mac-mini-to-install-this">
  ### Muss ich einen Mac Mini kaufen, um das zu installieren
</div>

Nein. OpenClaw läuft auf macOS oder Linux (Windows via WSL2). Ein Mac mini ist optional – manche Leute
kaufen einen als Always‑On‑Host, aber ein kleiner VPS, ein Heimserver oder ein Raspberry‑Pi‑ähnlicher Mini‑Rechner funktioniert genauso gut.

Du benötigst einen Mac nur **für macOS‑exklusive Tools**. Für iMessage kannst du das Gateway auf Linux betreiben
und `imsg` auf einem beliebigen Mac per SSH ausführen, indem du `channels.imessage.cliPath` auf einen SSH‑Wrapper setzt.
Wenn du andere macOS‑exklusive Tools nutzen willst, betreibe das Gateway auf einem Mac oder koppel einen macOS‑Knoten.

Dokumentation: [iMessage](/de/channels/imessage), [Nodes](/de/nodes), [Mac-Remote-Modus](/de/platforms/mac/remote).

<div id="do-i-need-a-mac-mini-for-imessage-support">
  ### Brauche ich einen Mac mini für iMessage-Unterstützung?
</div>

Du brauchst **ein macOS-Gerät**, das in Nachrichten angemeldet ist. Es muss **kein** Mac mini sein – jeder Mac funktioniert. Die iMessage-Integrationen von OpenClaw laufen auf macOS (BlueBubbles oder `imsg`), während das Gateway woanders laufen kann.

Übliche Setups:

* Führe das Gateway auf Linux/VPS aus und setze `channels.imessage.cliPath` auf einen SSH-Wrapper, der `imsg` auf dem Mac ausführt.
* Führe alles auf dem Mac aus, wenn du das einfachste Einzelrechner-Setup möchtest.

Dokumentation: [iMessage](/de/channels/imessage), [BlueBubbles](/de/channels/bluebubbles),
[Mac-Remote-Modus](/de/platforms/mac/remote).

<div id="if-i-buy-a-mac-mini-to-run-openclaw-can-i-connect-it-to-my-macbook-pro">
  ### Wenn ich einen Mac mini kaufe, um OpenClaw auszuführen, kann ich ihn mit meinem MacBook Pro verbinden
</div>

Ja. Der **Mac mini kann das Gateway ausführen**, und dein MacBook Pro kann sich als
**Knoten** (Begleitgerät) verbinden. Knoten führen das Gateway nicht aus – sie stellen
zusätzliche Funktionen wie Bildschirm/Kamera/Canvas und `system.run` auf diesem Gerät bereit.

Typisches Muster:

* Gateway auf dem Mac mini (immer eingeschaltet).
* MacBook Pro führt die macOS‑App oder einen Knoten‑Host aus und koppelt sich mit dem Gateway.
* Verwende `openclaw nodes status` / `openclaw nodes list`, um ihn anzuzeigen.

Doku: [Nodes](/de/nodes), [Nodes CLI](/de/cli/nodes).

<div id="can-i-use-bun">
  ### Kann ich Bun verwenden
</div>

Bun wird **nicht empfohlen**. Wir sehen Laufzeitfehler, insbesondere mit WhatsApp und Telegram.
Verwende **Node.js** für stabile Gateways.

Wenn du trotzdem mit Bun experimentieren willst, mach das auf einem nicht produktiven Gateway
ohne WhatsApp/Telegram.

<div id="telegram-what-goes-in-allowfrom">
  ### Telegram: Was gehört in allowFrom?
</div>

`channels.telegram.allowFrom` ist **die Telegram-Benutzer-ID des menschlichen Absenders** (numerisch, empfohlen) oder der `@username`. Das ist nicht der Benutzername des Bots.

Sicherer (kein Drittanbieter-Bot):

* Schicke deinem Bot eine DM, führe dann `openclaw logs --follow` aus und lies dort `from.id` ab.

Offizielle Bot-API:

* Schicke deinem Bot eine DM, rufe dann `https://api.telegram.org/bot<bot_token>/getUpdates` auf und lies `message.from.id` ab.

Drittanbieter (weniger privat):

* Schicke eine DM an `@userinfobot` oder `@getidsbot`.

Siehe [/channels/telegram](/de/channels/telegram#access-control-dms--groups).

<div id="can-multiple-people-use-one-whatsapp-number-with-different-openclaw-instances">
  ### Können mehrere Personen eine WhatsApp-Nummer mit unterschiedlichen OpenClaw-Instanzen verwenden?
</div>

Ja, über **Multi-Agent-Routing**. Verknüpfe die WhatsApp‑**DM** jedes Absenders (Peer `kind: "dm"`, Absender im E.164-Format wie `+15551234567`) mit einer anderen `agentId`, sodass jede Person ihren eigenen Arbeitsbereich und eigenen Sitzungsspeicher erhält. Antworten werden weiterhin vom **selben WhatsApp-Konto** gesendet, und die DM-Zugriffskontrolle (`channels.whatsapp.dmPolicy` / `channels.whatsapp.allowFrom`) ist global pro WhatsApp-Konto. Siehe [Multi-Agent Routing](/de/concepts/multi-agent) und [WhatsApp](/de/channels/whatsapp).

<div id="can-i-run-a-fast-chat-agent-and-an-opus-for-coding-agent">
  ### Kann ich einen schnellen Chat-agent und einen Opus-Coding-agent ausführen
</div>

Ja. Verwende Multi-Agent-Routing: Gib jedem agent sein eigenes Standardmodell und binde dann eingehende Routen (Anbieter-Konto oder bestimmte Peers) an jeden agent. Eine Beispielkonfiguration findest du unter [Multi-Agent-Routing](/de/concepts/multi-agent). Siehe auch [Modelle](/de/concepts/models) und [Konfiguration](/de/gateway/configuration).

<div id="does-homebrew-work-on-linux">
  ### Funktioniert Homebrew unter Linux?
</div>

Ja. Homebrew unterstützt auch Linux (Linuxbrew). Schnelle Einrichtung:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
echo 'eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"' >> ~/.profile
eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"
brew install <formula>
```

Wenn du OpenClaw über systemd ausführst, stelle sicher, dass die Service-Umgebungsvariable `PATH` `/home/linuxbrew/.linuxbrew/bin` (oder deinen Brew-Prefix) enthält, damit `brew`-installierte Tools in Non-Login-Shells verfügbar sind.
Neuere Builds setzen außerdem gängige Benutzer-`bin`-Verzeichnisse in systemd-Services unter Linux an den Anfang (zum Beispiel `~/.local/bin`, `~/.npm-global/bin`, `~/.local/share/pnpm`, `~/.bun/bin`) und berücksichtigen `PNPM_HOME`, `NPM_CONFIG_PREFIX`, `BUN_INSTALL`, `VOLTA_HOME`, `ASDF_DATA_DIR`, `NVM_DIR` und `FNM_DIR`, falls gesetzt.

<div id="whats-the-difference-between-the-hackable-git-install-and-npm-install">
  ### Was ist der Unterschied zwischen der hackbaren Git-Installation und der npm-Installation
</div>

* **Hackbare (Git-)Installation:** Checkout des vollständigen Quellcodes, editierbar, ideal für Mitwirkende.
  Du baust lokal und kannst Code/Docs patchen.
* **npm-Installation:** globale CLI-Installation, kein Repo, ideal zum „einfach nur laufen lassen“.
  Updates kommen über npm‑dist‑Tags.

Dokumentation: [Erste Schritte](/de/start/getting-started), [Aktualisieren](/de/install/updating).

<div id="can-i-switch-between-npm-and-git-installs-later">
  ### Kann ich später zwischen npm- und git-Installationen wechseln?
</div>

Ja. Installiere die andere Variante und führe dann Doctor aus, damit der Gateway-Dienst auf den neuen Entry-Point zeigt.
Dadurch **werden deine Daten nicht gelöscht** – es wird nur die OpenClaw-Code-Installation geändert. Dein Zustand
(`~/.openclaw`) und dein Arbeitsbereich (`~/.openclaw/workspace`) bleiben unangetastet.

Von npm → git:

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm build
openclaw doctor
openclaw gateway restart
```

Von Git → npm:

```bash
npm install -g openclaw@latest
openclaw doctor
openclaw gateway restart
```

Doctor erkennt eine Abweichung beim Gateway-Service-Entrypoint und bietet an, die Service-Konfiguration an die aktuelle Installation anzupassen (verwende `--repair` bei Automatisierung).

Backup-Tipps: siehe [Backup-Strategie](/de/help/faq#whats-the-recommended-backup-strategy).

<div id="should-i-run-the-gateway-on-my-laptop-or-a-vps">
  ### Sollte ich den Gateway auf meinem Laptop oder einem VPS ausführen
</div>

Kurzfassung: **Wenn du 24/7‑Zuverlässigkeit willst, nutze einen VPS**. Wenn du
möglichst wenig Aufwand willst und Schlafmodus/Neustarts okay sind, führe ihn lokal aus.

**Laptop (lokaler Gateway)**

* **Vorteile:** keine Serverkosten, direkter Zugriff auf lokale Dateien, sichtbares Browserfenster.
* **Nachteile:** Schlafmodus/Netzwerkabbrüche = Disconnects, OS‑Updates/Neustarts unterbrechen den Betrieb, der Laptop muss wach bleiben.

**VPS / Cloud**

* **Vorteile:** durchgängig verfügbar (always‑on), stabiles Netzwerk, keine Laptop‑Schlafprobleme, einfacher dauerhaft am Laufen zu halten.
* **Nachteile:** läuft oft headless (Screenshots verwenden), nur Remote‑Dateizugriff, du musst für Updates per SSH darauf zugreifen.

**OpenClaw-spezifische Anmerkung:** WhatsApp/Telegram/Slack/Mattermost (Plugin)/Discord funktionieren alle problemlos von einem VPS. Der einzige wirkliche Kompromiss ist **Headless‑Browser** vs. sichtbares Fenster. Siehe [Browser](/de/tools/browser).

**Empfohlener Standard:** VPS, wenn du zuvor Gateway‑Disconnects hattest. Lokal ist ideal, wenn du den Mac aktiv nutzt und lokalen Dateizugriff oder UI‑Automatisierung mit einem sichtbaren Browser möchtest.

<div id="how-important-is-it-to-run-openclaw-on-a-dedicated-machine">
  ### Wie wichtig ist es, OpenClaw auf einer dedizierten Maschine zu betreiben?
</div>

Nicht erforderlich, aber **für Zuverlässigkeit und Isolation empfohlen**.

* **Dedizierter Host (VPS/Mac mini/Pi):** immer eingeschaltet, weniger Unterbrechungen durch Ruhezustand/Neustarts, sauberere Berechtigungen, leichter dauerhaft am Laufen zu halten.
* **Normal genutzter Laptop/Desktop:** völlig in Ordnung für Tests und aktive Nutzung, aber rechne mit Pausen, wenn die Maschine in den Ruhezustand geht oder Updates installiert.

Wenn du das Beste aus beiden Welten möchtest, betreibe das Gateway auf einem dedizierten Host und koppel deinen Laptop als **Knoten** für lokale Bildschirm‑/Kamera‑/Exec‑Tools. Siehe [Nodes](/de/nodes).
Für Sicherheitshinweise lies [Security](/de/gateway/security).

<div id="what-are-the-minimum-vps-requirements-and-recommended-os">
  ### Was sind die minimalen VPS-Anforderungen und das empfohlene Betriebssystem
</div>

OpenClaw ist ressourcenschonend. Für ein einfaches Gateway + einen Chat-Kanal:

* **Absolutes Minimum:** 1 vCPU, 1GB RAM, ~500MB Speicherplatz.
* **Empfohlen:** 1–2 vCPU, 2GB RAM oder mehr als Puffer (Logs, Medien, mehrere Kanäle). Node-Tools und Browserautomatisierung können ressourcenhungrig sein.

OS: Verwende **Ubuntu LTS** (oder ein anderes aktuelles Debian/Ubuntu). Der Linux-Installationspfad ist dort am besten erprobt.

Dokumentation: [Linux](/de/platforms/linux), [VPS-Hosting](/de/vps).

<div id="can-i-run-openclaw-in-a-vm-and-what-are-the-requirements">
  ### Kann ich OpenClaw in einer VM ausführen und was sind die Anforderungen
</div>

Ja. Behandle eine VM genauso wie einen VPS: Sie muss immer eingeschaltet und erreichbar sein und
genug RAM für den Gateway und alle von dir aktivierten Channels haben.

Basisempfehlungen:

* **Absolutes Minimum:** 1 vCPU, 1GB RAM.
* **Empfohlen:** 2GB RAM oder mehr, wenn du mehrere Channels, Browser-Automatisierung oder Media-Tools betreibst.
* **OS:** Ubuntu LTS oder ein anderes modernes Debian/Ubuntu.

Wenn du Windows verwendest, ist **WSL2 das einfachste VM-Setup** und bietet die beste
Tooling-Kompatibilität. Siehe [Windows](/de/platforms/windows), [VPS hosting](/de/vps).
Wenn du macOS in einer VM betreibst, siehe [macOS VM](/de/platforms/macos-vm).

<div id="what-is-openclaw">
  ## Was ist OpenClaw?
</div>

<div id="what-is-openclaw-in-one-paragraph">
  ### Was ist OpenClaw in einem Absatz
</div>

OpenClaw ist ein persönlicher KI-Assistent, den du auf deinen eigenen Geräten betreibst. Er antwortet auf den Messaging-Plattformen, die du bereits nutzt (WhatsApp, Telegram, Slack, Mattermost (Plugin), Discord, Google Chat, Signal, iMessage, WebChat) und unterstützt auf geeigneten Plattformen außerdem Sprachinteraktion und ein Live-Canvas. Das **Gateway** ist die durchgängig laufende Control Plane; der Assistent ist das Produkt.

<div id="whats-the-value-proposition">
  ### Was ist das Wertversprechen?
</div>

OpenClaw ist nicht „nur ein Claude-Wrapper“. Es ist eine **lokal‑first‑Control‑Plane**, mit der du
einen leistungsfähigen Assistenten auf **deiner eigenen Hardware** betreiben kannst – erreichbar
aus den Chat-Apps, die du bereits nutzt, mit zustandsbehafteten Sitzungen, Memory und Tools –
ohne die Kontrolle über deine Workflows an ein gehostetes SaaS abzugeben.

Highlights:

* **Deine Geräte, deine Daten:** betreibe das Gateway, wo du willst (Mac, Linux, VPS), und
  behalte Arbeitsbereich + Sitzungshistorie lokal.
* **Echte Kanäle, keine Web-sandbox:** WhatsApp/Telegram/Slack/Discord/Signal/iMessage/etc.,
  plus mobile Sprachsteuerung und Canvas auf unterstützten Plattformen.
* **Modell-agnostisch:** nutze Anthropic, OpenAI, MiniMax, OpenRouter etc. mit per‑agent‑Routing
  und Failover.
* **Nur-lokal-Option:** betreibe lokale Modelle, sodass **alle Daten auf deinem Gerät bleiben
  können**, wenn du das möchtest.
* **Multi-agent-Routing:** getrennte Agenten pro Kanal, Account oder Aufgabe, jeweils mit eigenem
  Arbeitsbereich und eigenen Defaults.
* **Open Source und hackbar:** inspiziere, erweitere und hoste selbst – ohne Vendor Lock‑in.

Doku: [Gateway](/de/gateway), [Channels](/de/channels), [Multi‑agent](/de/concepts/multi-agent),
[Memory](/de/concepts/memory).

<div id="i-just-set-it-up-what-should-i-do-first">
  ### Ich habe es gerade eingerichtet – was sollte ich als Erstes tun
</div>

Gute erste Projekte:

* Eine Website erstellen (WordPress, Shopify oder eine einfache statische Site).
* Einen Prototyp für eine mobile App erstellen (Ablauf, Screens, API-Plan).
* Dateien und Ordner organisieren (Aufräumen, Benennung, Tags).
* Gmail verbinden und Zusammenfassungen oder Follow-ups automatisieren.

Es kann große Aufgaben bewältigen, funktioniert aber am besten, wenn du sie in Phasen aufteilst und Sub-Agents für parallele Arbeit einsetzt.

<div id="what-are-the-top-five-everyday-use-cases-for-openclaw">
  ### Was sind die fünf wichtigsten alltäglichen Anwendungsfälle für OpenClaw?
</div>

Typische Alltagsanwendungen sehen meist so aus:

* **Persönliche Briefings:** Zusammenfassungen von E-Mail-Posteingang, Kalender und für dich relevanten Nachrichten.
* **Recherche und Entwürfe:** schnelle Recherchen, Zusammenfassungen und erste Entwürfe für E-Mails oder Dokumente.
* **Erinnerungen und Follow-ups:** cron- oder Herzschlag-gesteuerte Hinweise und Checklisten.
* **Browser-Automatisierung:** Formulare ausfüllen, Daten sammeln und wiederkehrende Webaufgaben ausführen.
* **Geräteübergreifende Koordination:** Sende eine Aufgabe von deinem Smartphone, lass das Gateway sie auf einem Server ausführen und erhalte das Ergebnis im Chat zurück.

<div id="can-openclaw-help-with-lead-gen-outreach-ads-and-blogs-for-a-saas">
  ### Kann OpenClaw bei Lead-Gen-Outreach, Anzeigen und Blogs für ein SaaS helfen?
</div>

Ja, für **Recherche, Qualifizierung und Entwürfe**. Es kann Websites scannen, Shortlists
erstellen, potenzielle Kunden zusammenfassen und Entwürfe für Outreach- oder Anzeigentexte schreiben.

Für **Outreach- oder Anzeigen-Kampagnen** solltest du immer einen Menschen im Loop behalten. Vermeide Spam, halte dich an lokale Gesetze und
Plattformrichtlinien und prüfe alles, bevor es versendet wird. Das sicherste Vorgehen ist, OpenClaw Entwürfe erstellen zu lassen, die du dann freigibst.

Doku: [Sicherheit](/de/gateway/security).

<div id="what-are-the-advantages-vs-claude-code-for-web-development">
  ### Was sind die Vorteile im Vergleich zu Claude Code für Webentwicklung
</div>

OpenClaw ist ein **persönlicher Assistent** und eine Koordinationsschicht, kein IDE-Ersatz. Verwende
Claude Code oder Codex für den schnellsten direkten Coding-Workflow im Repo. Verwende OpenClaw, wenn du
dauerhaften Speicher, geräteübergreifenden Zugriff und Tool-Orchestrierung möchtest.

Vorteile:

* **Persistenter Speicher + Arbeitsbereich** über Sitzungen hinweg
* **Plattformübergreifender Zugriff** (WhatsApp, Telegram, TUI, WebChat)
* **Tool-Orchestrierung** (Browser, Dateien, Terminplanung, Hooks)
* **Always-on-Gateway** (auf einem VPS ausführen, von überall nutzen)
* **Knoten** für lokalen Browser/Bildschirm/Kamera/exec

Showcase: https://openclaw.ai/showcase

<div id="skills-and-automation">
  ## Fähigkeiten und Automatisierung
</div>

<div id="how-do-i-customize-skills-without-keeping-the-repo-dirty">
  ### Wie passe ich Fähigkeiten an, ohne das Repo dauerhaft zu verändern
</div>

Verwende verwaltete Überschreibungen, anstatt eine lokale Kopie des Repos direkt zu bearbeiten. Lege deine Änderungen in `~/.openclaw/skills/<name>/SKILL.md` ab (oder füge einen Ordner über `skills.load.extraDirs` in `~/.openclaw/openclaw.json` hinzu). Die Priorität ist `<workspace>/skills` &gt; `~/.openclaw/skills` &gt; mitgeliefert, sodass verwaltete Überschreibungen Vorrang haben, ohne git anzufassen. Nur Änderungen, die „upstream-würdig“ sind, sollten im Repo liegen und als PRs rausgehen.

<div id="can-i-load-skills-from-a-custom-folder">
  ### Kann ich Fähigkeiten aus einem benutzerdefinierten Ordner laden?
</div>

Ja. Fügen Sie zusätzliche Verzeichnisse über `skills.load.extraDirs` in `~/.openclaw/openclaw.json` hinzu (niedrigste Priorität). Die Standardreihenfolge der Prioritäten bleibt: `<workspace>/skills` → `~/.openclaw/skills` → mitgeliefert → `skills.load.extraDirs`. `clawhub` installiert standardmäßig in `./skills`, was OpenClaw als `<workspace>/skills` behandelt.

<div id="how-can-i-use-different-models-for-different-tasks">
  ### Wie kann ich unterschiedliche Modelle für unterschiedliche Aufgaben verwenden?
</div>

Aktuell werden folgende Muster unterstützt:

* **Cron-Jobs**: Isolierte Jobs können je Job ein `model`-Override festlegen.
* **Sub-Agenten**: Leite Aufgaben an separate Agenten mit unterschiedlichen Standardmodellen weiter.
* **On-Demand-Umschaltung**: Verwende `/model`, um das aktuelle Modell der Sitzung jederzeit zu wechseln.

Siehe [Cron-Jobs](/de/automation/cron-jobs), [Multi-Agent-Routing](/de/concepts/multi-agent) und [Slash-Befehle](/de/tools/slash-commands).

<div id="the-bot-freezes-while-doing-heavy-work-how-do-i-offload-that">
  ### Der Bot friert bei rechenintensiven Aufgaben ein – wie lagere ich das aus
</div>

Verwende **Subagenten** für lange oder parallele Aufgaben. Subagenten laufen in ihrer eigenen Sitzung,
geben eine Zusammenfassung zurück und halten deinen Hauptchat reaktionsfähig.

Bitte deinen Bot, „einen Subagenten für diese Aufgabe zu starten“, oder verwende `/subagents`.
Nutze `/status` im Chat, um zu sehen, was das Gateway gerade tut (und ob es ausgelastet ist).

Token-Tipp: Lange Aufgaben und Subagenten verbrauchen beide Tokens. Wenn die Kosten ein Problem sind, setze
ein günstigeres Modell für Subagenten über `agents.defaults.subagents.model`.

Doku: [Subagenten](/de/tools/subagents).

<div id="cron-or-reminders-do-not-fire-what-should-i-check">
  ### Cron oder Erinnerungen werden nicht ausgelöst – was sollte ich prüfen
</div>

Cron läuft innerhalb des Gateway-Prozesses. Wenn das Gateway nicht kontinuierlich läuft,
werden geplante Jobs nicht ausgeführt.

Checkliste:

* Stelle sicher, dass Cron aktiviert ist (`cron.enabled`) und `OPENCLAW_SKIP_CRON` nicht gesetzt ist.
* Prüfe, ob das Gateway 24/7 läuft (kein Sleep/keine Neustarts).
* Überprüfe die Zeitzoneneinstellungen für den Job (`--tz` vs. Host-Zeitzone).

Debug:

```bash
openclaw cron run <jobId> --force
openclaw cron runs --id <jobId> --limit 50
```

Dokumentation: [Cron-Jobs](/de/automation/cron-jobs), [Cron vs Herzschlag](/de/automation/cron-vs-heartbeat).

<div id="how-do-i-install-skills-on-linux">
  ### Wie installiere ich Fähigkeiten unter Linux
</div>

Verwende **ClawHub** (CLI) oder lege Fähigkeiten in deinem Arbeitsbereich ab. Die macOS Skills UI ist unter Linux nicht verfügbar.
Stöbere in den Fähigkeiten auf https://clawhub.com.

Installiere die ClawHub CLI (wähle einen Paketmanager aus):

```bash
npm i -g clawhub
```

```bash
pnpm add -g clawhub
```

<div id="can-openclaw-run-tasks-on-a-schedule-or-continuously-in-the-background">
  ### Kann OpenClaw Aufgaben nach Zeitplan oder dauerhaft im Hintergrund ausführen
</div>

Ja. Verwende den Gateway-Scheduler:

* **Cron jobs** für geplante oder wiederkehrende Aufgaben (bleiben über Neustarts hinweg bestehen).
* **Heartbeat** für periodische Prüfungen der „Hauptsitzung“.
* **Isolated jobs** für autonome Agenten, die Zusammenfassungen posten oder in Chats liefern.

Doku: [Cron jobs](/de/automation/cron-jobs), [Cron vs Heartbeat](/de/automation/cron-vs-heartbeat),
[Heartbeat](/de/gateway/heartbeat).

**Kann ich Apple-macOS-only-Fähigkeiten von Linux aus ausführen**

Nicht direkt. macOS-Fähigkeiten werden durch `metadata.openclaw.os` plus erforderliche Binaries begrenzt, und Fähigkeiten erscheinen nur dann im System-Prompt, wenn sie auf dem **Gateway-Host** zulässig sind. Unter Linux werden `darwin`-only-Fähigkeiten (wie `imsg`, `apple-notes`, `apple-reminders`) nicht geladen, es sei denn, du überschreibst dieses Gating.

Du hast drei unterstützte Varianten:

**Option A – Gateway auf einem Mac ausführen (am einfachsten).**\
Führe das Gateway dort aus, wo die macOS-Binaries vorhanden sind, und verbinde dich dann von Linux im [Remote-Modus](#how-do-i-run-openclaw-in-remote-mode-client-connects-to-a-gateway-elsewhere) oder über Tailscale. Die Fähigkeiten werden normal geladen, weil der Gateway-Host macOS ist.

**Option B – einen macOS-Knoten verwenden (kein SSH).**\
Führe das Gateway unter Linux aus, paare einen macOS-Knoten (Menüleisten-App) und setze **Node Run Commands** auf „Always Ask“ oder „Always Allow“ auf dem Mac. OpenClaw kann macOS-only-Fähigkeiten als zulässig behandeln, wenn die erforderlichen Binaries auf dem Knoten vorhanden sind. Der Agent führt diese Fähigkeiten über das `nodes`-Werkzeug aus. Wenn du „Always Ask“ wählst, fügt das Bestätigen von „Always Allow“ in der Eingabeaufforderung diesen Befehl zur Allowlist hinzu.

**Option C – macOS-Binaries über SSH proxyen (fortgeschritten).**\
Lass das Gateway unter Linux laufen, aber sorge dafür, dass die erforderlichen CLI-Binaries auf SSH-Wrapper verweisen, die auf einem Mac ausgeführt werden. Überschreibe dann die Fähigkeit, um Linux zuzulassen, sodass sie zulässig bleibt.

1. Erstelle einen SSH-Wrapper für das Binary (Beispiel: `imsg`):
   ```bash
   #!/usr/bin/env bash
   set -euo pipefail
   exec ssh -T user@mac-host /opt/homebrew/bin/imsg "$@"
   ```
2. Lege den Wrapper auf dem Linux-Host in den `PATH` (zum Beispiel `~/bin/imsg`).
3. Überschreibe die Skill-Metadaten (Arbeitsbereich oder `~/.openclaw/skills`), um Linux zuzulassen:
   ```markdown
   ---
   name: imsg
   description: iMessage/SMS CLI for listing chats, history, watch, and sending.
   metadata: {"openclaw":{"os":["darwin","linux"],"requires":{"bins":["imsg"]}}}
   ---
   ```
4. Starte eine neue Sitzung, damit der Fähigkeiten-Snapshot aktualisiert wird.

Speziell für iMessage kannst du `channels.imessage.cliPath` auch auf einen SSH-Wrapper zeigen lassen (OpenClaw benötigt nur stdio). Siehe [iMessage](/de/channels/imessage).

<div id="do-you-have-a-notion-or-heygen-integration">
  ### Gibt es eine Notion‑ oder HeyGen‑Integration
</div>

Derzeit nicht integriert.

Optionen:

* **Benutzerdefinierte Fähigkeit / Plugin:** am besten für zuverlässigen API‑Zugriff (sowohl Notion als auch HeyGen haben APIs).
* **Browser‑Automatisierung:** funktioniert ohne Code, ist aber langsamer und fehleranfälliger.

Wenn du Kontext pro Kunde beibehalten möchtest (Agentur‑Workflows), ist ein einfaches Muster:

* Eine Notion‑Seite pro Kunde (Kontext + Präferenzen + aktive Arbeit).
* Weise den agent an, diese Seite zu Beginn einer Sitzung abzurufen.

Wenn du eine native Integration möchtest, eröffne einen Feature Request oder baue eine Fähigkeit,
die diese APIs ansteuert.

Fähigkeiten installieren:

```bash
clawhub install <skill-slug>
clawhub update --all
```

ClawHub wird im Verzeichnis `./skills` in deinem aktuellen Verzeichnis installiert (oder greift auf deinen konfigurierten OpenClaw-Arbeitsbereich zurück); OpenClaw behandelt dieses Verzeichnis in der nächsten Sitzung als `<workspace>/skills`. Um Fähigkeiten zwischen mehreren Agenten zu teilen, lege sie unter `~/.openclaw/skills/<name>/SKILL.md` ab. Einige Fähigkeiten setzen über Homebrew installierte Binärdateien voraus; unter Linux bedeutet das Linuxbrew (siehe den oben stehenden Homebrew-Linux-FAQ-Eintrag). Siehe [Skills](/de/tools/skills) und [ClawHub](/de/tools/clawhub).

<div id="how-do-i-install-the-chrome-extension-for-browser-takeover">
  ### Wie installiere ich die Chrome-Erweiterung für die Browser-Übernahme?
</div>

Verwende den integrierten Installer und lade anschließend die entpackte Erweiterung in Chrome:

```bash
openclaw browser extension install
openclaw browser extension path
```

Dann in Chrome → `chrome://extensions` → „Developer mode“ aktivieren → „Load unpacked“ → diesen Ordner auswählen.

Vollständige Anleitung (inkl. entferntem Gateway + Sicherheitshinweisen): [Chrome-Erweiterung](/de/tools/chrome-extension)

Wenn das Gateway auf demselben Rechner wie Chrome läuft (Standard-Setup), brauchst du in der Regel **nichts Weiteres**.
Wenn das Gateway woanders läuft, starte einen Knoten-Host auf dem Browser-Rechner, damit das Gateway Browser-Aktionen als Proxy weiterleiten kann.
Du musst trotzdem die Erweiterungsschaltfläche auf dem Tab anklicken, den du steuern möchtest (sie verknüpft sich nicht automatisch mit dem Tab).

<div id="sandboxing-and-memory">
  ## Sandboxing und Speicher
</div>

<div id="is-there-a-dedicated-sandboxing-doc">
  ### Gibt es eine eigene Dokumentation zum Sandboxing
</div>

Ja. Siehe [Sandboxing](/de/gateway/sandboxing). Für Docker-spezifisches Setup (Gateway komplett in Docker oder sandbox-Images) siehe [Docker](/de/install/docker).

**Kann ich DMs privat halten, aber Gruppen öffentlich in einer sandbox mit einem Agent betreiben**

Ja – sofern dein privater Datenverkehr **DMs** und dein öffentlicher Datenverkehr **Gruppen** sind.

Verwende `agents.defaults.sandbox.mode: "non-main"`, sodass Gruppen-/Kanal-Sitzungen (non-main-Keys) in Docker laufen, während die Haupt-DM-Sitzung auf dem Host bleibt. Beschränke dann, welche Tools in Sandbox-Sitzungen verfügbar sind, über `tools.sandbox.tools`.

Schritt-für-Schritt-Anleitung + Beispielkonfiguration: [Gruppen: persönliche DMs + öffentliche Gruppen](/de/concepts/groups#pattern-personal-dms-public-groups-single-agent)

Zentrale Konfigurationsreferenz: [Gateway-Konfiguration](/de/gateway/configuration#agentsdefaultssandbox)

<div id="how-do-i-bind-a-host-folder-into-the-sandbox">
  ### Wie binde ich einen Host-Ordner in die Sandbox ein
</div>

Setze `agents.defaults.sandbox.docker.binds` auf `["host:path:mode"]` (z. B. `"/home/user/src:/src:ro"`). Globale und agent-spezifische Binds werden zusammengeführt; agent-spezifische Binds werden ignoriert, wenn `scope: "shared"` ist. Verwende `:ro` für alle sensiblen Daten und denke daran, dass Binds die Dateisystem-Isolierung der Sandbox umgehen. Siehe [Sandboxing](/de/gateway/sandboxing#custom-bind-mounts) und [Sandbox vs Tool Policy vs Elevated](/de/gateway/sandbox-vs-tool-policy-vs-elevated#bind-mounts-security-quick-check) für Beispiele und Sicherheitshinweise.

<div id="how-does-memory-work">
  ### Wie funktioniert Memory
</div>

OpenClaw-Memory besteht einfach aus Markdown-Dateien im arbeitsbereich des agents:

* Tägliche Notizen in `memory/YYYY-MM-DD.md`
* Kuratierte Langzeitnotizen in `MEMORY.md` (nur Haupt-/private Sitzungen)

OpenClaw führt außerdem einen **stillschweigenden Memory-Flush vor der Kompaktierung** aus, um das Modell daran zu erinnern, vor der automatischen Kompaktierung dauerhafte Notizen zu schreiben. Dies läuft nur, wenn der arbeitsbereich schreibbar ist (schreibgeschützte sandboxes überspringen ihn). Siehe [Memory](/de/concepts/memory).

<div id="memory-keeps-forgetting-things-how-do-i-make-it-stick">
  ### Der Speicher vergisst ständig Dinge – wie sorge ich dafür, dass sie bleiben
</div>

Bitte den Bot, **die Information in den Speicher zu schreiben**. Langfristige Notizen gehören in `MEMORY.md`,
kurzfristiger Kontext geht in `memory/YYYY-MM-DD.md`.

Dies ist noch ein Bereich, den wir verbessern. Es hilft, das Modell daran zu erinnern, Erinnerungen zu speichern;
es weiß, was zu tun ist. Wenn es weiterhin Dinge vergisst, überprüfe, ob das Gateway bei jedem Lauf denselben
arbeitsbereich verwendet.

Dokumentation: [Speicher](/de/concepts/memory), [Agent-arbeitsbereich](/de/concepts/agent-workspace).

<div id="does-semantic-memory-search-require-an-openai-api-key">
  ### Benötigt die semantische Speichersuche einen OpenAI-API-Schlüssel?
</div>

Nur wenn du **OpenAI-Embeddings** verwendest. Codex OAuth deckt Chat/Completions ab und
gewährt **keinen** Zugriff auf Embeddings. **Die Anmeldung mit Codex (OAuth oder
dem Codex-CLI-Login)** hilft daher nicht für die semantische Speichersuche. OpenAI-Embeddings
benötigen weiterhin einen echten API-Schlüssel (`OPENAI_API_KEY` oder `models.providers.openai.apiKey`).

Wenn du keinen Anbieter explizit setzt, wählt OpenClaw automatisch einen Anbieter aus, sobald es
einen API-Schlüssel findet (Auth-Profile, `models.providers.*.apiKey` oder Umgebungsvariablen).
OpenClaw bevorzugt OpenAI, wenn ein OpenAI-Schlüssel gefunden wird, andernfalls Gemini, wenn ein Gemini-Schlüssel
gefunden wird. Wenn keiner der Schlüssel verfügbar ist, bleibt die Speichersuche deaktiviert, bis du
sie konfigurierst. Wenn du einen lokalen Modellpfad konfiguriert und verfügbar hast, bevorzugt OpenClaw
`local`.

Wenn du lieber lokal bleiben möchtest, setze `memorySearch.provider = "local"` (und optional
`memorySearch.fallback = "none"`). Wenn du Gemini-Embeddings verwenden möchtest, setze
`memorySearch.provider = "gemini"` und stelle `GEMINI_API_KEY` bereit (oder
`memorySearch.remote.apiKey`). Wir unterstützen **OpenAI-, Gemini- oder lokale** Embedding-Modelle – siehe
[Memory](/de/concepts/memory) für Details zur Einrichtung.

<div id="does-memory-persist-forever-what-are-the-limits">
  ### Bleibt der Speicher für immer bestehen? Welche Grenzen gibt es?
</div>

Speicherdateien werden auf der Festplatte gespeichert und bleiben erhalten, bis du sie löschst. Die Grenze ist dein
Speicherplatz, nicht das Modell. Der **Sitzungskontext** ist weiterhin durch das Kontextfenster des Modells
begrenzt, daher können lange Unterhaltungen komprimiert oder gekürzt werden. Deshalb gibt es die
Speichersuche – sie holt nur die relevanten Teile zurück in den Kontext.

Dokumentation: [Memory](/de/concepts/memory), [Context](/de/concepts/context).

<div id="where-things-live-on-disk">
  ## Wo Daten auf dem Datenträger liegen
</div>

<div id="is-all-data-used-with-openclaw-saved-locally">
  ### Werden alle Daten, die mit OpenClaw verwendet werden, lokal gespeichert?
</div>

Nein – **der Zustand von OpenClaw ist lokal**, aber **externe Dienste sehen weiterhin, was du ihnen sendest**.

* **Standardmäßig lokal:** Sitzungen, Speicherdateien, Konfiguration und Arbeitsbereich liegen auf dem Gateway-Host
  (`~/.openclaw` + dein Arbeitsbereichsverzeichnis).
* **Zwangsläufig extern:** Nachrichten, die du an Modellanbieter (Anthropic/OpenAI/etc.) sendest, gehen an
  deren APIs, und Chat-Plattformen (WhatsApp/Telegram/Slack/etc.) speichern Nachrichtendaten auf ihren
  Servern.
* **Du kontrollierst die Datenspur:** Wenn du lokale Modelle verwendest, bleiben Prompts auf deinem Rechner, aber der
  Kanalverkehr läuft weiterhin über die Server des jeweiligen Kanals.

Siehe auch: [Agent-Arbeitsbereich](/de/concepts/agent-workspace), [Speicher](/de/concepts/memory).

<div id="where-does-openclaw-store-its-data">
  ### Wo speichert OpenClaw seine Daten?
</div>

Alles befindet sich unter `$OPENCLAW_STATE_DIR` (Standard: `~/.openclaw`):

| Pfad | Zweck |
|------|--------|
| `$OPENCLAW_STATE_DIR/openclaw.json` | Hauptkonfiguration (JSON5) |
| `$OPENCLAW_STATE_DIR/credentials/oauth.json` | Veralteter OAuth‑Import (bei der ersten Verwendung in Auth‑Profile kopiert) |
| `$OPENCLAW_STATE_DIR/agents/<agentId>/agent/auth-profiles.json` | Auth‑Profile (OAuth + API‑Schlüssel) |
| `$OPENCLAW_STATE_DIR/agents/<agentId>/agent/auth.json` | Laufzeit‑Auth‑Cache (wird automatisch verwaltet) |
| `$OPENCLAW_STATE_DIR/credentials/` | Provider‑Zustand (z. B. `whatsapp/<accountId>/creds.json`) |
| `$OPENCLAW_STATE_DIR/agents/` | Zustand pro Agent (agentDir + Sitzungen) |
| `$OPENCLAW_STATE_DIR/agents/<agentId>/sessions/` | Gesprächsverlauf &amp; Zustand (pro Agent) |
| `$OPENCLAW_STATE_DIR/agents/<agentId>/sessions/sessions.json` | Sitzungsmetadaten (pro Agent) |

Veralteter Einzelagent‑Pfad: `~/.openclaw/agent/*` (migriert durch `openclaw doctor`).

Dein **Arbeitsbereich** (AGENTS.md, Memory‑Dateien, Fähigkeiten usw.) ist getrennt und wird über `agents.defaults.workspace` konfiguriert (Standard: `~/.openclaw/workspace`).

<div id="where-should-agentsmd-soulmd-usermd-memorymd-live">
  ### Wo sollten AGENTSmd SOULmd USERmd MEMORYmd liegen
</div>

Diese Dateien liegen im **Agent-Arbeitsbereich**, nicht in `~/.openclaw`.

* **Arbeitsbereich (pro Agent)**: `AGENTS.md`, `SOUL.md`, `IDENTITY.md`, `USER.md`,
  `MEMORY.md` (oder `memory.md`), `memory/YYYY-MM-DD.md`, optional `HEARTBEAT.md`.
* **State-Verzeichnis (`~/.openclaw`)**: Konfiguration, Zugangsdaten, Auth-Profile, Sitzungen, Logs
  und gemeinsame Fähigkeiten (`~/.openclaw/skills`).

Der Standard-Arbeitsbereich ist `~/.openclaw/workspace`, konfigurierbar über:

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } }
}
```

Wenn der Bot nach einem Neustart „vergisst“, überprüfe, ob das Gateway bei jedem Start denselben
Arbeitsbereich verwendet (und denk daran: Der Remote-Modus nutzt den **Arbeitsbereich des Gateway-Hosts**,
nicht dein lokales Laptop).

Tipp: Wenn du ein dauerhaft beibehaltenes Verhalten oder eine bevorzugte Einstellung möchtest, bitte den Bot, **es in
AGENTS.md oder MEMORY.md zu schreiben**, statt dich auf den Chatverlauf zu verlassen.

Siehe [Agent-Arbeitsbereich](/de/concepts/agent-workspace) und [Memory](/de/concepts/memory).

<div id="whats-the-recommended-backup-strategy">
  ### Was ist die empfohlene Backup-Strategie
</div>

Lege deinen **Agent-Arbeitsbereich** in ein **privates** Git-Repository und sichere ihn
an einem privaten Ort (zum Beispiel in einem privaten GitHub-Repository). Dadurch werden Memory-Daten plus AGENTS/SOUL/USER-
Dateien erfasst, und du kannst den „Geist“ des Assistenten später wiederherstellen.

Lege **nichts** unter `~/.openclaw` per Commit an (Credentials, Sitzungen, Tokens).
Wenn du eine vollständige Wiederherstellung benötigst, sichere sowohl den Arbeitsbereich als auch das State-Verzeichnis
separat (siehe die obenstehende Migrationsfrage).

Dokumentation: [Agent-Arbeitsbereich](/de/concepts/agent-workspace).

<div id="how-do-i-completely-uninstall-openclaw">
  ### Wie deinstalliere ich OpenClaw vollständig
</div>

Siehe die ausführliche Anleitung: [Deinstallation](/de/install/uninstall).

<div id="can-agents-work-outside-the-workspace">
  ### Können Agenten außerhalb des Arbeitsbereichs arbeiten
</div>

Ja. Der Arbeitsbereich ist das **Standard‑cwd** und der Memory‑Anker, keine strikte sandbox.
Relative Pfade werden innerhalb des Arbeitsbereichs aufgelöst, aber absolute Pfade
können auf andere Speicherorte des Hosts zugreifen, sofern sandboxing nicht aktiviert ist. Wenn du
Isolation brauchst, verwende [`agents.defaults.sandbox`](/de/gateway/sandboxing) oder
sandbox‑Einstellungen pro agent. Wenn ein Repo das Standard‑Arbeitsverzeichnis sein
soll, setze den `workspace` dieses agents auf das Repo‑Root. Das OpenClaw‑Repo ist nur
Quellcode; halte den Arbeitsbereich getrennt, es sei denn, du möchtest ausdrücklich, dass der
agent darin arbeitet.

Beispiel (Repo als Standard‑cwd):

```json5
{
  agents: {
    defaults: {
      workspace: "~/Projects/my-repo"
    }
  }
}
```

<div id="im-in-remote-mode-where-is-the-session-store">
  ### Ich bin im Remote-Modus – wo befindet sich der Sitzungsspeicher?
</div>

Der Sitzungszustand wird vom **Gateway-Host** verwaltet. Wenn du im Remote-Modus bist, liegt der relevante Sitzungsspeicher auf dem Remote-Rechner, nicht auf deinem lokalen Laptop. Siehe [Sitzungsverwaltung](/de/concepts/session).

<div id="config-basics">
  ## Konfigurationsgrundlagen
</div>

<div id="what-format-is-the-config-where-is-it">
  ### In welchem Format liegt die Konfiguration vor, und wo befindet sie sich?
</div>

OpenClaw liest eine optionale **JSON5**-Konfigurationsdatei aus `$OPENCLAW_CONFIG_PATH` (Standard: `~/.openclaw/openclaw.json`):

```
$OPENCLAW_CONFIG_PATH
```

Falls die Datei fehlt, werden relativ sichere Standardwerte verwendet (einschließlich eines Standardarbeitsbereichs `~/.openclaw/workspace`).

<div id="i-set-gatewaybind-lan-or-tailnet-and-now-nothing-listens-the-ui-says-unauthorized">
  ### Ich habe gatewaybind auf lan oder tailnet gesetzt, und jetzt lauscht nichts mehr – die UI zeigt „unauthorized“ an
</div>

Nicht-Loopback-Binds **erfordern Authentifizierung**. Konfiguriere `gateway.auth.mode` + `gateway.auth.token` (oder verwende `OPENCLAW_GATEWAY_TOKEN`).

```json5
{
  gateway: {
    bind: "lan",
    auth: {
      mode: "token",
      token: "replace-me"
    }
  }
}
```

Hinweise:

* `gateway.remote.token` ist nur für **Remote-CLI-Aufrufe** bestimmt; es aktiviert keine lokale Gateway-Authentifizierung.
* Die Control UI authentifiziert sich über `connect.params.auth.token` (in den App-/UI-Einstellungen gespeichert). Vermeide, Token in URLs zu verwenden.

<div id="why-do-i-need-a-token-on-localhost-now">
  ### Warum brauche ich jetzt ein Token auf localhost
</div>

Der Assistent erzeugt standardmäßig ein Gateway-Token (auch auf dem Loopback-Interface), sodass sich **lokale WS-Clients authentifizieren müssen**. Dadurch wird verhindert, dass andere lokale Prozesse auf das Gateway zugreifen. Füge das Token in die Einstellungen der Control UI (oder deine Client-Konfiguration) ein, um eine Verbindung herzustellen.

Wenn du **wirklich** ein offenes Loopback-Interface willst, entferne `gateway.auth` aus deiner Konfiguration. Der Befehl `openclaw doctor` kann dir jederzeit ein Token erzeugen: `openclaw doctor --generate-gateway-token`.

<div id="do-i-have-to-restart-after-changing-config">
  ### Muss ich nach einer Konfigurationsänderung neu starten
</div>

Das Gateway überwacht die Konfiguration und unterstützt Hot‑Reload:

* `gateway.reload.mode: "hybrid"` (Standard): wendet sichere Änderungen im laufenden Betrieb an, Neustart für kritische Änderungen
* `hot`, `restart`, `off` werden ebenfalls unterstützt

<div id="how-do-i-enable-web-search-and-web-fetch">
  ### Wie aktiviere ich Websuche und Web-Fetch
</div>

`web_fetch` funktioniert ohne API-Schlüssel. `web_search` benötigt einen Brave-Search-API-Schlüssel. **Empfohlen:** `openclaw configure --section web` ausführen, um ihn in `tools.web.search.apiKey` zu speichern. Alternative über die Umgebung: die Umgebungsvariable `BRAVE_API_KEY` für den Gateway-Prozess setzen.

```json5
{
  tools: {
    web: {
      search: {
        enabled: true,
        apiKey: "BRAVE_API_KEY_HERE",
        maxResults: 5
      },
      fetch: {
        enabled: true
      }
    }
  }
}
```

Hinweise:

* Wenn du Allowlists verwendest, füge `web_search`/`web_fetch` oder `group:web` hinzu.
* `web_fetch` ist standardmäßig aktiviert (sofern nicht ausdrücklich deaktiviert).
* Daemons lesen Umgebungsvariablen aus `~/.openclaw/.env` (oder aus der Service-Umgebung).

Doku: [Web-Tools](/de/tools/web).

<div id="how-do-i-run-a-central-gateway-with-specialized-workers-across-devices">
  ### Wie betreibe ich ein zentrales Gateway mit spezialisierten Workern über mehrere Geräte hinweg?
</div>

Das gängige Muster ist **ein Gateway** (z. B. Raspberry Pi) plus **Knoten** und **Agenten**:

* **Gateway (zentral):** verwaltet Kanäle (Signal/WhatsApp), Routing und Sitzungen.
* **Knoten (Geräte):** Macs/iOS/Android werden als Peripheriegeräte verbunden und stellen lokale Tools (`system.run`, `canvas`, `camera`) bereit.
* **Agenten (Worker):** separate Logik-/Arbeitsbereiche für spezielle Rollen (z. B. „Hetzner Ops“, „Persönliche Daten“).
* **Sub-Agenten:** starten Hintergrundarbeit aus einem Hauptagenten, wenn du Parallelität benötigst.
* **TUI:** stellt die Verbindung zum Gateway her und erlaubt das Umschalten zwischen Agenten und Sitzungen.

Dokumentation: [Knoten](/de/nodes), [Remotezugriff](/de/gateway/remote), [Multi-Agent-Routing](/de/concepts/multi-agent), [Sub-Agenten](/de/tools/subagents), [TUI](/de/tui).

<div id="can-the-openclaw-browser-run-headless">
  ### Kann der OpenClaw-Browser im Headless-Modus laufen
</div>

Ja, das ist eine Konfigurationsoption:

```json5
{
  browser: { headless: true },
  agents: {
    defaults: {
      sandbox: { browser: { headless: true } }
    }
  }
}
```

Standardwert ist `false` (mit sichtbarem Fenster). Headless erhöht auf manchen Websites die Wahrscheinlichkeit, dass Anti‑Bot‑Prüfungen ausgelöst werden. Siehe [Browser](/de/tools/browser).

Headless verwendet **dieselbe Chromium‑Engine** und funktioniert für die meisten Automatisierungsaufgaben (Formulare, Klicks, Scraping, Logins). Die wichtigsten Unterschiede:

* Kein sichtbares Browserfenster (verwende Screenshots, wenn du visuelle Ausgaben brauchst).
* Einige Websites sind bei Automatisierung im Headless‑Modus strenger (CAPTCHAs, Anti‑Bot).
  Zum Beispiel blockiert X/Twitter häufig Headless‑Sitzungen.

<div id="how-do-i-use-brave-for-browser-control">
  ### Wie verwende ich Brave für die Browsersteuerung
</div>

Setze `browser.executablePath` auf die Brave-Programmdatei (oder einen beliebigen Chromium-basierten Browser) und starte das Gateway neu.
Sieh dir die vollständigen Beispielkonfigurationen unter [Browser](/de/tools/browser#use-brave-or-another-chromium-based-browser) an.

<div id="remote-gateways-nodes">
  ## Remote-Gateways + Knoten
</div>

<div id="how-do-commands-propagate-between-telegram-the-gateway-and-nodes">
  ### Wie werden Befehle zwischen Telegram, dem Gateway und Knoten weitergeleitet?
</div>

Telegram-Nachrichten werden vom **Gateway** verarbeitet. Das Gateway führt den Agent aus und ruft Knoten erst dann über den **Gateway-WebSocket** auf, wenn ein Knoten-Tool benötigt wird:

Telegram → Gateway → Agent → `node.*` → Node → Gateway → Telegram

Knoten sehen keinen eingehenden Anbieter-Traffic; sie erhalten nur Knoten-RPC-Aufrufe.

<div id="how-can-my-agent-access-my-computer-if-the-gateway-is-hosted-remotely">
  ### Wie kann mein Agent auf meinen Computer zugreifen, wenn das Gateway remote gehostet wird
</div>

Kurzfassung: **kopple deinen Computer als Knoten**. Das Gateway läuft woanders, kann aber
`node.*`‑Tools (Bildschirm, Kamera, System) auf deinem lokalen Rechner über den Gateway‑WebSocket aufrufen.

Typische Einrichtung:

1. Führe das Gateway auf dem Always‑on‑Host aus (VPS/Home‑Server).
2. Hänge den Gateway‑Host und deinen Computer in dasselbe Tailnet.
3. Stelle sicher, dass der Gateway‑WS erreichbar ist (Tailnet‑Bind oder SSH‑Tunnel).
4. Öffne die macOS‑App lokal und verbinde dich im Modus **Remote über SSH** (oder direktes Tailnet),
   damit sie sich als Knoten registrieren kann.
5. Genehmige den Knoten im Gateway:
   ```bash
   openclaw nodes pending
   openclaw nodes approve <requestId>
   ```

Es ist keine separate TCP‑Bridge erforderlich; Knoten verbinden sich über den Gateway‑WebSocket.

Sicherheitshinweis: Das Koppeln eines macOS‑Knotens erlaubt `system.run` auf dieser Maschine. Kopple nur
Geräte, denen du vertraust, und lies dir [Sicherheit](/de/gateway/security) durch.

Dokumentation: [Nodes](/de/nodes), [Gateway‑Protokoll](/de/gateway/protocol), [macOS‑Remote‑Modus](/de/platforms/mac/remote), [Sicherheit](/de/gateway/security).

<div id="tailscale-is-connected-but-i-get-no-replies-what-now">
  ### Tailscale ist verbunden, aber ich bekomme keine Antworten. Was nun?
</div>

Prüfe zuerst die Grundlagen:

* Gateway läuft: `openclaw gateway status`
* Gateway-Status: `openclaw status`
* Channel-Status: `openclaw channels status`

Überprüfe danach Authentifizierung und Routing:

* Wenn du Tailscale Serve verwendest, stelle sicher, dass `gateway.auth.allowTailscale` korrekt gesetzt ist.
* Wenn du dich über einen SSH-Tunnel verbindest, prüfe, ob der lokale Tunnel aktiv ist und auf den richtigen Port zeigt.
* Stelle sicher, dass deine Allowlist (DM oder Gruppe) dein Konto enthält.

Doku: [Tailscale](/de/gateway/tailscale), [Remote access](/de/gateway/remote), [Channels](/de/channels).

<div id="can-two-openclaw-instances-talk-to-each-other-local-vps">
  ### Können zwei OpenClaw-Instanzen auf einem lokalen VPS miteinander kommunizieren
</div>

Ja. Es gibt keine integrierte „Bot-zu-Bot“-Bridge, aber du kannst das auf ein paar
zuverlässige Arten einrichten:

**Am einfachsten:** Verwende einen normalen Chat-Kanal, auf den beide Bots zugreifen können (Telegram/Slack/WhatsApp).
Lass Bot A eine Nachricht an Bot B senden und Bot B dann wie üblich antworten.

**CLI-Bridge (generisch):** Führe ein Skript aus, das das andere Gateway mit
`openclaw agent --message ... --deliver` aufruft und einen Chat adressiert, in dem der andere Bot
lauscht. Wenn ein Bot auf einem entfernten VPS läuft, richte deine CLI über SSH/Tailscale auf dieses entfernte Gateway aus (siehe [Remotezugriff](/de/gateway/remote)).

Beispiel (ausgeführt von einer Maschine, die das Ziel-Gateway erreichen kann):

```bash
openclaw agent --message "Hallo vom lokalen Bot" --deliver --channel telegram --reply-to <chat-id>
```

Tipp: Füge einen Schutzmechanismus hinzu, damit die beiden Bots nicht endlos in einer Schleife hängen bleiben (Nur-Erwähnungen, Channel-Allowlists oder eine „nicht auf Bot-Nachrichten antworten“-Regel).

Doku: [Remotezugriff](/de/gateway/remote), [Agent CLI](/de/cli/agent), [Agent send](/de/tools/agent-send).

<div id="do-i-need-separate-vpses-for-multiple-agents">
  ### Brauche ich separate VPS für mehrere Agenten
</div>

Nein. Ein Gateway kann mehrere Agenten hosten, jeweils mit eigenem Arbeitsbereich, eigenen Modell-Standardeinstellungen
und eigenem Routing. Das ist das normale Setup und deutlich günstiger und einfacher, als
für jeden Agent einen eigenen VPS zu betreiben.

Verwende separate VPS nur, wenn du harte Isolation (Sicherheitsgrenzen) oder sehr
unterschiedliche Konfigurationen brauchst, die du nicht gemeinsam nutzen möchtest. Andernfalls betreibst du ein Gateway
und verwendest mehrere Agenten oder Sub-Agenten.

<div id="is-there-a-benefit-to-using-a-node-on-my-personal-laptop-instead-of-ssh-from-a-vps">
  ### Gibt es einen Vorteil, auf meinem persönlichen Laptop einen Knoten zu verwenden, statt per SSH von einem VPS aus zu arbeiten
</div>

Ja – Knoten sind der primäre Weg, deinen Laptop von einem entfernten Gateway aus zu erreichen, und sie
ermöglichen mehr als nur Shell-Zugriff. Das Gateway läuft auf macOS/Linux (Windows via WSL2) und ist
ressourcenschonend (ein kleiner VPS oder eine Raspberry-Pi-Klasse-Box reicht aus; 4 GB RAM sind mehr als genug), daher ist ein typisches
Setup ein Always-on-Host plus dein Laptop als Knoten.

* **Kein eingehendes SSH erforderlich.** Knoten bauen eine ausgehende Verbindung zum Gateway-WS (WebSocket) auf und verwenden Gerätekopplung.
* **Sicherere Ausführungssteuerung.** `system.run` wird auf diesem Laptop durch Knoten-Allowlists/-Freigaben abgesichert.
* **Mehr Geräte-Tools.** Knoten stellen neben `system.run` auch `canvas`, `camera` und `screen` zur Verfügung.
* **Lokale Browser-Automatisierung.** Lass das Gateway auf einem VPS laufen, aber führe Chrome lokal aus und leite die Steuerung
  mit der Chrome-Erweiterung + einem Knoten-Host auf dem Laptop weiter.

SSH ist für gelegentlichen Shell-Zugriff in Ordnung, aber Knoten sind einfacher für laufende Agent-Workflows und
Geräteautomatisierung.

Dokumentation: [Nodes](/de/nodes), [Nodes CLI](/de/cli/nodes), [Chrome extension](/de/tools/chrome-extension).

<div id="should-i-install-on-a-second-laptop-or-just-add-a-node">
  ### Sollte ich auf einem zweiten Laptop installieren oder einfach einen Knoten hinzufügen
</div>

Wenn du auf dem zweiten Laptop nur **lokale Tools** (screen/camera/exec) brauchst, füge ihn als
**Knoten** hinzu. Damit behältst du ein einzelnes Gateway und vermeidest doppelte Konfiguration. Lokale Knoten-Tools sind
derzeit nur für macOS verfügbar, wir planen aber, sie auf andere Betriebssysteme auszuweiten.

Installiere ein zweites Gateway nur, wenn du **strikte Isolation** oder zwei vollständig getrennte Bots benötigst.

Dokumentation: [Nodes](/de/nodes), [Nodes-CLI](/de/cli/nodes), [Mehrere Gateways](/de/gateway/multiple-gateways).

<div id="do-nodes-run-a-gateway-service">
  ### Betreiben Knoten einen Gateway-Dienst
</div>

Nein. Pro Host sollte nur **ein Gateway** laufen, es sei denn, du betreibst absichtlich isolierte Profile (siehe [Multiple gateways](/de/gateway/multiple-gateways)). Knoten sind Peripheriegeräte, die sich mit dem Gateway verbinden (iOS/Android-Knoten oder macOS-„Knotenmodus“ in der Menüleisten-App). Für headless Knoten-Hosts und die Steuerung per CLI siehe [Node host CLI](/de/cli/node).

Für Änderungen an `gateway`, `discovery` und `canvasHost` ist ein vollständiger Neustart erforderlich.

<div id="is-there-an-api-rpc-way-to-apply-config">
  ### Gibt es eine API-/RPC-Methode zum Anwenden der Konfiguration?
</div>

Ja. `config.apply` validiert und schreibt die vollständige Konfiguration und startet das Gateway als Teil des Vorgangs neu.

<div id="configapply-wiped-my-config-how-do-i-recover-and-avoid-this">
  ### configapply hat meine Config gelöscht Wie kann ich sie wiederherstellen und das vermeiden
</div>

`config.apply` ersetzt die **gesamte Config**. Wenn du nur ein partielles Objekt sendest, wird
alles andere entfernt.

Wiederherstellen:

* Aus einem Backup wiederherstellen (git oder eine kopierte `~/.openclaw/openclaw.json`).
* Wenn du kein Backup hast, `openclaw doctor` erneut ausführen und Kanäle/Modelle neu konfigurieren.
* Wenn das unerwartet war, melde einen Bug und füge deine letzte bekannte Config oder ein vorhandenes Backup bei.
* Ein lokaler Coding-Agent kann oft aus Logs oder Verlauf eine funktionierende Config rekonstruieren.

Vermeiden:

* Verwende `openclaw config set` für kleine Änderungen.
* Verwende `openclaw configure` für interaktive Bearbeitung.

Doku: [Config](/de/cli/config), [Configure](/de/cli/configure), [Doctor](/de/gateway/doctor).

<div id="whats-a-minimal-sane-config-for-a-first-install">
  ### Wie sieht eine minimal sinnvolle Konfiguration für die erste Installation aus
</div>

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
  channels: { whatsapp: { allowFrom: ["+15555550123"] } }
}
```

Dadurch wird dein arbeitsbereich festgelegt und eingeschränkt, wer den Bot ausführen darf.

<div id="how-do-i-set-up-tailscale-on-a-vps-and-connect-from-my-mac">
  ### Wie richte ich Tailscale auf einem VPS ein und verbinde mich von meinem Mac aus
</div>

Minimale Schritte:

1. **Auf dem VPS installieren + anmelden**
   ```bash
   curl -fsSL https://tailscale.com/install.sh | sh
   sudo tailscale up
   ```
2. **Auf deinem Mac installieren + anmelden**
   * Verwende die Tailscale-App und melde dich im selben Tailnet an.
3. **MagicDNS aktivieren (empfohlen)**
   * Aktiviere in der Tailscale-Admin-Konsole MagicDNS, damit der VPS einen stabilen Namen hat.
4. **Den Tailnet-Hostnamen verwenden**
   * SSH: `ssh user@your-vps.tailnet-xxxx.ts.net`
   * Gateway WS: `ws://your-vps.tailnet-xxxx.ts.net:18789`

Wenn du die Control UI ohne SSH verwenden willst, nutze Tailscale Serve auf dem VPS:

```bash
openclaw gateway --tailscale serve
```

Dadurch bleibt das Gateway an die Loopback-Schnittstelle gebunden und stellt HTTPS über Tailscale bereit. Siehe [Tailscale](/de/gateway/tailscale).

<div id="how-do-i-connect-a-mac-node-to-a-remote-gateway-tailscale-serve">
  ### Wie verbinde ich einen Mac-Knoten mit einem Remote-Gateway über Tailscale Serve
</div>

Serve stellt die **Gateway Control UI + WS** bereit. Knoten verbinden sich über denselben Gateway-WS-Endpunkt.

Empfohlene Einrichtung:

1. **Stell sicher, dass sich der VPS und der Mac im selben Tailnet befinden**.
2. **Verwende die macOS-App im Remote-Modus** (das SSH-Ziel kann der Tailnet-Hostname sein).
   Die App tunnelt den Gateway-Port und verbindet sich als Knoten.
3. **Genehmige den Knoten** auf dem Gateway:
   ```bash
   openclaw nodes pending
   openclaw nodes approve <requestId>
   ```

Dokumentation: [Gateway-Protokoll](/de/gateway/protocol), [Discovery](/de/gateway/discovery), [macOS-Remote-Modus](/de/platforms/mac/remote).

<div id="env-vars-and-env-loading">
  ## Umgebungsvariablen und Laden von .env-Dateien
</div>

<div id="how-does-openclaw-load-environment-variables">
  ### Wie lädt OpenClaw Umgebungsvariablen?
</div>

OpenClaw liest Umgebungsvariablen aus dem Parent-Prozess (Shell, launchd/systemd, CI usw.) und lädt zusätzlich:

* `.env` aus dem aktuellen Arbeitsverzeichnis
* ein globales Fallback-`.env` aus `~/.openclaw/.env` (alias `$OPENCLAW_STATE_DIR/.env`)

Keine der `.env`-Dateien überschreibt bereits gesetzte Umgebungsvariablen.

Du kannst außerdem Inline-Umgebungsvariablen in der Konfiguration definieren (werden nur angewendet, wenn sie in der Prozessumgebung fehlen):

```json5
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: { GROQ_API_KEY: "gsk-..." }
  }
}
```

Siehe [/environment](/de/environment) für die vollständige Prioritäten- und Quellenübersicht.

<div id="i-started-the-gateway-via-the-service-and-my-env-vars-disappeared-what-now">
  ### Ich habe das Gateway über den Service gestartet und meine Env‑Variablen sind verschwunden – was nun?
</div>

Zwei häufige Lösungen:

1. Lege die fehlenden Schlüssel in `~/.openclaw/.env` ab, damit sie auch dann geladen werden, wenn der Service deine Shell‑Umgebung nicht erbt.
2. Aktiviere den Shell‑Import (optionale Komfortfunktion):

```json5
{
  env: {
    shellEnv: {
      enabled: true,
      timeoutMs: 15000
    }
  }
}
```

Dies startet deine Login-Shell und importiert nur erwartete Schlüssel, die noch fehlen (überschreibt niemals bestehende Werte). Entsprechende Umgebungsvariablen:
`OPENCLAW_LOAD_SHELL_ENV=1`, `OPENCLAW_SHELL_ENV_TIMEOUT_MS=15000`.

<div id="i-set-copilotgithubtoken-but-models-status-shows-shell-env-off-why">
  ### Ich habe COPILOTGITHUBTOKEN gesetzt, aber `models status` zeigt „Shell env off“. Warum?
</div>

`openclaw models status` zeigt an, ob der **Shell-Umgebungsimport** aktiviert ist. „Shell env: off“
bedeutet **nicht**, dass deine Umgebungsvariablen fehlen – es bedeutet nur, dass OpenClaw deine
Login-Shell nicht automatisch lädt.

Wenn der Gateway als Dienst (launchd/systemd) läuft, erbt er deine Shell-Umgebung nicht.
Behebe das, indem du eine der folgenden Optionen nutzt:

1. Lege das Token in `~/.openclaw/.env` ab:
   ```
   COPILOT_GITHUB_TOKEN=...
   ```
2. Oder aktiviere den Shell-Import (`env.shellEnv.enabled: true`).
3. Oder füge es in deinen `env`-Block der Konfiguration ein (gilt nur, falls es fehlt).

Starte dann den Gateway neu und prüfe den Status erneut:

```bash
openclaw models status
```

Copilot-Tokens werden aus `COPILOT_GITHUB_TOKEN` (bzw. `GH_TOKEN` / `GITHUB_TOKEN`) eingelesen.
Siehe [/concepts/model-providers](/de/concepts/model-providers) und [/environment](/de/environment).

<div id="sessions-multiple-chats">
  ## Sitzungen &amp; mehrere Chats
</div>

<div id="how-do-i-start-a-fresh-conversation">
  ### Wie starte ich einen neuen Chat
</div>

Sende `/new` oder `/reset` als eigene Nachricht. Siehe [Sitzungsverwaltung](/de/concepts/session).

<div id="do-sessions-reset-automatically-if-i-never-send-new">
  ### Setzen sich Sitzungen automatisch zurück, wenn ich nie etwas Neues sende
</div>

Ja. Sitzungen laufen nach `session.idleMinutes` (Standardwert **60**) ab. Die **nächste**
Nachricht startet eine neue Sitzungs-ID für diesen Chat-Schlüssel. Dadurch werden
Transkripte nicht gelöscht – es wird nur eine neue Sitzung gestartet.

```json5
{
  session: {
    idleMinutes: 240
  }
}
```

<div id="is-there-a-way-to-make-a-team-of-openclaw-instances-one-ceo-and-many-agents">
  ### Gibt es eine Möglichkeit, aus OpenClaw-Instanzen ein Team zu machen – einen CEO und viele Agenten
</div>

Ja, über **Multi-Agent-Routing** und **Subagenten**. Du kannst einen koordinierenden
Agent und mehrere Worker-Agenten mit eigenen Arbeitsbereichen und Modellen erstellen.

Allerdings ist das eher als **spaßiges Experiment** zu sehen. Es ist token-intensiv und oft
weniger effizient, als einen einzelnen Bot mit separaten Sitzungen zu verwenden. Das typische Modell,
das wir uns vorstellen, ist ein Bot, mit dem du sprichst, der unterschiedliche Sitzungen für parallele Arbeit nutzt. Dieser
Bot kann bei Bedarf auch Subagenten starten.

Doku: [Multi-Agent-Routing](/de/concepts/multi-agent), [Subagenten](/de/tools/subagents), [Agents-CLI](/de/cli/agents).

<div id="why-did-context-get-truncated-midtask-how-do-i-prevent-it">
  ### Warum wurde der Kontext mitten in einer Aufgabe abgeschnitten, und wie verhindere ich das
</div>

Der Sitzungskontext ist durch das Kontextfenster des Modells begrenzt. Lange Unterhaltungen, große Tool-Ausgaben oder viele Dateien können Komprimierung oder Abschneiden auslösen.

Was hilft:

* Bitte den Bot, den aktuellen Stand zusammenzufassen und in eine Datei zu schreiben.
* Verwende `/compact` vor langen Aufgaben und `/new` beim Themenwechsel.
* Halte wichtigen Kontext im Arbeitsbereich und bitte den Bot, ihn wieder daraus einzulesen.
* Verwende Sub-Agenten für lange oder parallele Aufgaben, damit der Hauptchat klein bleibt.
* Wähle ein Modell mit einem größeren Kontextfenster, wenn das häufig passiert.

<div id="how-do-i-completely-reset-openclaw-but-keep-it-installed">
  ### Wie setze ich OpenClaw vollständig zurück, ohne es zu deinstallieren?
</div>

Verwende den Reset-Befehl:

```bash
openclaw reset
```

Nicht-interaktiver kompletter Reset:

```bash
openclaw reset --scope full --yes --non-interactive
```

Führe anschließend das Onboarding erneut aus:

```bash
openclaw onboard --install-daemon
```

Hinweise:

* Der Onboarding-Assistent bietet ebenfalls die Option **Reset** an, falls eine vorhandene Konfiguration gefunden wird. Siehe [Wizard](/de/start/wizard).
* Wenn du Profile verwendet hast (`--profile` / `OPENCLAW_PROFILE`), setze jedes Zustandsverzeichnis zurück (Standardpfade sind `~/.openclaw-<profile>`).
* Dev-Reset: `openclaw gateway --dev --reset` (nur für Entwicklung; löscht die Dev-Konfiguration + Zugangsdaten + Sitzungen + Arbeitsbereich).

<div id="im-getting-context-too-large-errors-how-do-i-reset-or-compact">
  ### Ich erhalte „context too large“-Fehler – wie kann ich zurücksetzen oder komprimieren?
</div>

Verwende eine der folgenden Optionen:

* **Compact** (behält die Unterhaltung, fasst aber ältere Dialogschritte zusammen):
  ```
  /compact
  ```
  oder `/compact <instructions>`, um die Zusammenfassung zu steuern.

* **Reset** (neue Sitzungs-ID für denselben Chat-Schlüssel):
  ```
  /new
  /reset
  ```

Wenn das weiterhin auftritt:

* Aktiviere oder justiere **Session-Pruning** (`agents.defaults.contextPruning`), um alte Tool-Ausgaben zu kürzen.
* Verwende ein Modell mit einem größeren Kontextfenster.

Dokumentation: [Compaction](/de/concepts/compaction), [Session pruning](/de/concepts/session-pruning), [Session management](/de/concepts/session).

<div id="why-am-i-seeing-llm-request-rejected-messagesncontentxtooluseinput-field-required">
  ### Warum sehe ich Meldungen „LLM request rejected messagesNcontentXtooluseinput Field required“
</div>

Dies ist ein Anbieter-Validierungsfehler: Das Modell hat einen `tool_use`-Block ohne das erforderliche
`input` ausgegeben. In der Regel bedeutet das, dass der Sitzungsverlauf veraltet oder beschädigt ist (oft nach langen Threads
oder einer Tool-/Schema-Änderung).

Lösung: Starte eine neue Sitzung mit `/new` (eigenständige Nachricht).

<div id="why-am-i-getting-heartbeat-messages-every-30-minutes">
  ### Warum erhalte ich alle 30 Minuten Herzschlag-Benachrichtigungen?
</div>

Herzschläge werden standardmäßig alle **30m** ausgeführt. Du kannst sie anpassen oder deaktivieren:

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "2h"   // oder „0m" zum Deaktivieren
      }
    }
  }
}
```

Wenn `HEARTBEAT.md` existiert, aber effektiv leer ist (nur Leerzeilen und Markdown-Überschriften wie `# Heading`), überspringt OpenClaw den Herzschlaglauf, um API-Aufrufe zu sparen. Wenn die Datei nicht vorhanden ist, läuft der Herzschlag trotzdem und das Modell entscheidet, was zu tun ist.

Überschreibungen pro Agent verwenden `agents.list[].heartbeat`. Dokumentation: [Herzschlag](/de/gateway/heartbeat).

<div id="do-i-need-to-add-a-bot-account-to-a-whatsapp-group">
  ### Muss ich ein Bot‑Konto zu einer WhatsApp‑Gruppe hinzufügen?
</div>

Nein. OpenClaw läuft über **dein eigenes Konto**, also wenn du in der Gruppe bist, kann OpenClaw sie sehen.
Standardmäßig sind Antworten in Gruppen blockiert, bis du Absender zulässt (`groupPolicy: "allowlist"`).

Wenn du möchtest, dass nur **du** Antworten in Gruppen auslösen kannst:

```json5
{
  channels: {
    whatsapp: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15551234567"]
    }
  }
}
```

<div id="how-do-i-get-the-jid-of-a-whatsapp-group">
  ### Wie bekomme ich die JID einer WhatsApp‑Gruppe
</div>

Option 1 (am schnellsten): Logs mit `tail` verfolgen und eine Testnachricht in der Gruppe senden:

```bash
openclaw logs --follow --json
```

Suche nach einer `chatId` (oder `from`), die auf `@g.us` endet, zum Beispiel:
`1234567890-1234567890@g.us`.

Option 2 (falls bereits konfiguriert/auf der Allowlist): Gruppen aus der Konfiguration auflisten:

```bash
openclaw directory groups list --channel whatsapp
```

Dokumentation: [WhatsApp](/de/channels/whatsapp), [Directory](/de/cli/directory), [Logs](/de/cli/logs).

<div id="why-doesnt-openclaw-reply-in-a-group">
  ### Warum antwortet OpenClaw in einer Gruppe nicht?
</div>

Zwei häufige Ursachen:

* Mention-Gating ist aktiviert (Standard). Du musst den Bot per @-Erwähnung ansprechen (oder einer in `mentionPatterns` definierten Erwähnung entsprechen).
* Du hast `channels.whatsapp.groups` ohne `"*"` konfiguriert, und die Gruppe ist nicht in der Allowlist enthalten.

Siehe [Gruppen](/de/concepts/groups) und [Gruppennachrichten](/de/concepts/group-messages).

<div id="do-groupsthreads-share-context-with-dms">
  ### Teilen Gruppen-Threads den Kontext mit DMs
</div>

Direkte Chats werden standardmäßig in die Hauptsitzung zusammengeführt. Gruppen/Kanäle haben ihre eigenen Sitzungsschlüssel, und Telegram-Themen / Discord-Threads sind separate Sitzungen. Siehe [Gruppen](/de/concepts/groups) und [Gruppennachrichten](/de/concepts/group-messages).

<div id="how-many-workspaces-and-agents-can-i-create">
  ### Wie viele Arbeitsbereiche und Agenten kann ich erstellen
</div>

Es gibt keine harten Limits. Dutzende (sogar Hunderte) sind in Ordnung, aber achte auf:

* **Wachstum des Speicherbedarfs:** Sitzungen + Transkripte liegen unter `~/.openclaw/agents/<agentId>/sessions/`.
* **Token-Kosten:** Mehr Agenten bedeuten mehr gleichzeitige Modellnutzung.
* **Betriebsaufwand:** Auth-Profile pro Agent, Arbeitsbereiche und Channel-Routing.

Tipps:

* Halte einen **aktiven** arbeitsbereich pro agent (`agents.defaults.workspace`).
* Bereinige alte Sitzungen (lösche JSONL- oder Store-Einträge), wenn der belegte Speicherplatz wächst.
* Verwende `openclaw doctor`, um verwaiste Arbeitsbereiche und nicht zueinander passende Profile zu finden.

<div id="can-i-run-multiple-bots-or-chats-at-the-same-time-slack-and-how-should-i-set-that-up">
  ### Kann ich mehrere Bots oder Chats gleichzeitig in Slack ausführen und wie sollte ich das einrichten?
</div>

Ja. Verwende **Multi‑Agent Routing**, um mehrere isolierte Agenten auszuführen und eingehende Nachrichten nach Kanal/Konto/Peer zu routen. Slack wird als Kanal unterstützt und kann an bestimmte Agenten gebunden werden.

Browser‑Zugriff ist leistungsfähig, aber nicht „alles, was ein Mensch kann“ – Anti‑Bot‑Mechanismen, CAPTCHAs und MFA können Automatisierung trotzdem blockieren. Für die zuverlässigste Browser‑Steuerung verwende das Chrome‑Extension‑Relay auf der Maschine, auf der der Browser läuft (und betreibe das Gateway an einem beliebigen anderen Ort).

Empfohlene Einrichtung:

* Always‑on‑Gateway‑Host (VPS/Mac mini).
* Ein agent pro Rolle (Bindings).
* Slack‑Kanal/Kanäle, die an diese Agenten gebunden sind.
* Lokaler Browser über Extension‑Relay (oder ein Knoten), wenn nötig.

Doku: [Multi‑Agent Routing](/de/concepts/multi-agent), [Slack](/de/channels/slack),
[Browser](/de/tools/browser), [Chrome extension](/de/tools/chrome-extension), [Nodes](/de/nodes).

<div id="models-defaults-selection-aliases-switching">
  ## Modelle: Standardwerte, Auswahl, Aliasse und Wechsel
</div>

<div id="what-is-the-default-model">
  ### Was ist das Standardmodell
</div>

Das Standardmodell von OpenClaw ist das, was du wie folgt festlegst:

```
agents.defaults.model.primary
```

Modelle werden im Format `provider/model` angegeben (Beispiel: `anthropic/claude-opus-4-5`). Wenn du den Anbieter weglässt, setzt OpenClaw derzeit `anthropic` als vorübergehenden Fallback im Zuge der Abschaffung älterer Voreinstellungen ein – du solltest aber trotzdem **explizit** `provider/model` setzen.

<div id="what-model-do-you-recommend">
  ### Welches Modell empfiehlst du?
</div>

**Empfohlenes Standardmodell:** `anthropic/claude-opus-4-5`.\
**Gute Alternative:** `anthropic/claude-sonnet-4-5`.\
**Zuverlässig (weniger Charakter):** `openai/gpt-5.2` – fast so gut wie Opus, nur mit weniger Persönlichkeit.\
**Budget:** `zai/glm-4.7`.

MiniMax M2.1 hat eine eigene Doku: [MiniMax](/de/providers/minimax) und
[Lokale Modelle](/de/gateway/local-models).

Faustregel: Verwende für Arbeiten mit hohem Risiko oder großer Tragweite das **beste Modell, das du dir leisten kannst**, und ein günstigeres
Modell für Routine-Chat oder Zusammenfassungen. Du kannst Modelle pro agent routen und Sub-Agenten verwenden, um
lange Aufgaben zu parallelisieren (jeder Sub-Agent verbraucht Tokens). Siehe [Modelle](/de/concepts/models) und
[Sub-Agenten](/de/tools/subagents).

Wichtige Warnung: Schwächere oder stark quantisierte Modelle sind anfälliger für Prompt-Injection
und unsicheres Verhalten. Siehe [Sicherheit](/de/gateway/security).

Mehr Kontext: [Modelle](/de/concepts/models).

<div id="can-i-use-selfhosted-models-llamacpp-vllm-ollama">
  ### Kann ich selbstgehostete Modelle wie llamacpp, vLLM, Ollama verwenden
</div>

Ja. Wenn dein lokaler Server eine OpenAI-kompatible api bereitstellt, kannst du
einen benutzerdefinierten Anbieter darauf konfigurieren. Ollama wird direkt
unterstützt und ist der einfachste Weg.

Sicherheitshinweis: Kleinere oder stark quantisierte Modelle sind anfälliger
für Prompt-Injection. Wir empfehlen dringend **große Modelle** für jeden Bot, der
Tools verwenden kann. Wenn du trotzdem kleine Modelle einsetzen möchtest,
aktiviere die sandbox und strikte Tool-Allowlists.

Dokumentation: [Ollama](/de/providers/ollama), [Lokale Modelle](/de/gateway/local-models),
[Modell-Anbieter](/de/concepts/model-providers), [Sicherheit](/de/gateway/security),
[Sandboxing](/de/gateway/sandboxing).

<div id="how-do-i-switch-models-without-wiping-my-config">
  ### Wie wechsle ich Modelle, ohne meine Config zu überschreiben
</div>

Verwende **Modellbefehle** oder bearbeite nur die **model**-Felder. Vermeide vollständige Config-Ersetzungen.

Sichere Optionen:

* `/model` im Chat (schnell, pro Sitzung)
* `openclaw models set ...` (aktualisiert nur die Modell-Config)
* `openclaw configure --section models` (interaktiv)
* `agents.defaults.model` in `~/.openclaw/openclaw.json` bearbeiten

Vermeide `config.apply` mit einem Teilobjekt, es sei denn, du willst die gesamte Config ersetzen.
Wenn du die Config doch überschrieben hast, stelle sie aus einem Backup wieder her oder führe `openclaw doctor` erneut aus, um sie zu reparieren.

Dokumentation: [Models](/de/concepts/models), [Configure](/de/cli/configure), [Config](/de/cli/config), [Doctor](/de/gateway/doctor).

<div id="what-do-openclaw-flawd-and-krill-use-for-models">
  ### Welche Modelle werden in OpenClaw, Flawd und Krill verwendet
</div>

* **OpenClaw + Flawd:** Anthropic Opus (`anthropic/claude-opus-4-5`) – siehe [Anthropic](/de/providers/anthropic).
* **Krill:** MiniMax M2.1 (`minimax/MiniMax-M2.1`) – siehe [MiniMax](/de/providers/minimax).

<div id="how-do-i-switch-models-on-the-fly-without-restarting">
  ### Wie wechsle ich Modelle im laufenden Betrieb, ohne einen Neustart?
</div>

Verwende den Befehl `/model` als separate Nachricht:

```
/model sonnet
/model haiku
/model opus
/model gpt
/model gpt-mini
/model gemini
/model gemini-flash
```

Du kannst verfügbare Modelle mit `/model`, `/model list` oder `/model status` auflisten.

`/model` (und `/model list`) zeigt eine kompakte, nummerierte Auswahlliste. Triff deine Auswahl anhand der Nummer:

```
/model 3
```

Sie können außerdem ein bestimmtes Auth-Profil für den Anbieter erzwingen (pro Sitzung):

```
/model opus@anthropic:default
/model opus@anthropic:work
```

Tipp: `/model status` zeigt, welcher Agent aktiv ist, welche `auth-profiles.json`-Datei verwendet wird und welches Auth-Profil als Nächstes verwendet wird.
Es zeigt außerdem den konfigurierten Anbieter-Endpunkt (`baseUrl`) und den API-Modus (`api`), sofern verfügbar.

**Wie löse ich die Anheftung eines mit profile gesetzten Profils?**

Führe `/model` erneut aus, **ohne** den `@profile`-Suffix:

```
/model anthropic/claude-opus-4-5
```

Wenn du zum Standard zurückkehren möchtest, wähle ihn über `/model` aus (oder sende `/model &lt;Standard-Anbieter/Modell&gt;`).
Verwende `/model status`, um zu bestätigen, welches Auth-Profil aktiv ist.

<div id="can-i-use-gpt-52-for-daily-tasks-and-codex-52-for-coding">
  ### Kann ich GPT 5.2 für tägliche Aufgaben und Codex 5.2 zum Programmieren verwenden?
</div>

Ja. Lege eines als Standard fest und wechsle bei Bedarf:

* **Schnellwechsel (pro Sitzung):** `/model gpt-5.2` für tägliche Aufgaben, `/model gpt-5.2-codex` zum Programmieren.
* **Standard + Wechsel:** setze `agents.defaults.model.primary` auf `openai-codex/gpt-5.2` und wechsle dann beim Programmieren zu `openai-codex/gpt-5.2-codex` (oder umgekehrt).
* **Sub-Agents:** leite Programmieraufgaben an Sub-Agents mit einem anderen Standardmodell weiter.

Siehe [Modelle](/de/concepts/models) und [Slash-Befehle](/de/tools/slash-commands).

<div id="why-do-i-see-model-is-not-allowed-and-then-no-reply">
  ### Warum sehe ich „Model is not allowed“ und anschließend keine Antwort
</div>

Wenn `agents.defaults.models` gesetzt ist, wird es zur **Allowlist** für `/model` und alle
Sitzungs-Overrides. Wenn du ein Modell auswählst, das nicht in dieser Liste steht, erhältst du:

```
Model "provider/model" is not allowed. Use /model to list available models.
```

Dieser Fehler wird **anstelle** einer normalen Antwort zurückgegeben. Lösung: Füge das Modell zu
`agents.defaults.models` hinzu, entferne die Allowlist oder wähle ein Modell mit `/model list` aus.

<div id="why-do-i-see-unknown-model-minimaxminimaxm21">
  ### Warum sehe ich „Unknown model minimaxMiniMaxM21“
</div>

Das bedeutet, dass der **Anbieter nicht konfiguriert ist** (es wurde keine MiniMax-Anbieterkonfiguration oder kein Auth-Profil gefunden), sodass das Modell nicht aufgelöst werden kann. Ein Fix für diese Erkennung ist in **2026.1.12** enthalten (zum Zeitpunkt des Schreibens noch nicht veröffentlicht).

Checkliste zur Behebung:

1. Auf **2026.1.12** aktualisieren (oder aus dem `main`-Branch ausführen) und anschließend das Gateway neu starten.
2. Sicherstellen, dass MiniMax konfiguriert ist (Wizard oder JSON) oder dass ein MiniMax-API-Key
   in Umgebungs-/Auth-Profilen existiert, sodass der Anbieter eingebunden werden kann.
3. Die exakte Modell-ID verwenden (Groß-/Kleinschreibung beachten): `minimax/MiniMax-M2.1` oder
   `minimax/MiniMax-M2.1-lightning`.
4. Ausführen:
   ```bash
   openclaw models list
   ```
   und aus der Liste auswählen (oder `/model list` im Chat).

Siehe [MiniMax](/de/providers/minimax) und [Modelle](/de/concepts/models).

<div id="can-i-use-minimax-as-my-default-and-openai-for-complex-tasks">
  ### Kann ich MiniMax als Standard verwenden und OpenAI für komplexe Aufgaben?
</div>

Ja. Verwende **MiniMax als Standard** und wechsle das Modell **pro Sitzung**, wenn nötig.
Fallbacks sind für **Fehler**, nicht für „anspruchsvolle Aufgaben“, verwende daher `/model` oder einen separaten Agent.

**Option A: Wechsel pro Sitzung**

```json5
{
  env: { MINIMAX_API_KEY: "sk-...", OPENAI_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "minimax/MiniMax-M2.1" },
      models: {
        "minimax/MiniMax-M2.1": { alias: "minimax" },
        "openai/gpt-5.2": { alias: "gpt" }
      }
    }
  }
}
```

Dann:

```
/model gpt
```

**Option B: getrennte Agenten**

* Standard für Agent A: MiniMax
* Standard für Agent B: OpenAI
* Nach Agent routen oder mit `/agent` wechseln

Dokumentation: [Modelle](/de/concepts/models), [Multi-Agent-Routing](/de/concepts/multi-agent), [MiniMax](/de/providers/minimax), [OpenAI](/de/providers/openai).

<div id="are-opus-sonnet-gpt-builtin-shortcuts">
  ### Sind opus, sonnet, gpt eingebaute Kurzbefehle?
</div>

Ja. OpenClaw bringt einige Standardkürzel mit (sie werden nur angewendet, wenn das Modell in `agents.defaults.models` existiert):

* `opus` → `anthropic/claude-opus-4-5`
* `sonnet` → `anthropic/claude-sonnet-4-5`
* `gpt` → `openai/gpt-5.2`
* `gpt-mini` → `openai/gpt-5-mini`
* `gemini` → `google/gemini-3-pro-preview`
* `gemini-flash` → `google/gemini-3-flash-preview`

Wenn du einen eigenen Alias mit demselben Namen setzt, setzt sich dein Wert durch.

<div id="how-do-i-defineoverride-model-shortcuts-aliases">
  ### Wie definiere oder überschreibe ich Modellkürzel (Aliases)?
</div>

Aliases werden über `agents.defaults.models.<modelId>.alias` definiert. Beispiel:

```json5
{
  agents: {
    defaults: {
      model: { primary: "anthropic/claude-opus-4-5" },
      models: {
        "anthropic/claude-opus-4-5": { alias: "opus" },
        "anthropic/claude-sonnet-4-5": { alias: "sonnet" },
        "anthropic/claude-haiku-4-5": { alias: "haiku" }
      }
    }
  }
}
```

Dann wird `/model sonnet` (oder `/<alias>`, wenn unterstützt) dieser Modell-ID zugeordnet.

<div id="how-do-i-add-models-from-other-providers-like-openrouter-or-zai">
  ### Wie füge ich Modelle von anderen Anbietern wie OpenRouter oder ZAI hinzu?
</div>

OpenRouter (Bezahlung pro Token; viele Modelle):

```json5
{
  agents: {
    defaults: {
      model: { primary: "openrouter/anthropic/claude-sonnet-4-5" },
      models: { "openrouter/anthropic/claude-sonnet-4-5": {} }
    }
  },
  env: { OPENROUTER_API_KEY: "sk-or-..." }
}
```

Z.AI (GLM-Modelle):

```json5
{
  agents: {
    defaults: {
      model: { primary: "zai/glm-4.7" },
      models: { "zai/glm-4.7": {} }
    }
  },
  env: { ZAI_API_KEY: "..." }
}
```

Wenn du auf einen Anbieter oder ein Modell verweist, aber der erforderliche API-Schlüssel für diesen Anbieter fehlt, erhältst du einen Authentifizierungsfehler zur Laufzeit (z. B. `No API key found for provider "zai"`).

**„No API key found for provider“ nach dem Hinzufügen eines neuen agent**

Das bedeutet in der Regel, dass der **neue agent** einen leeren Auth-Store hat. Auth ist pro agent und
wird gespeichert in:

```
~/.openclaw/agents/<agentId>/agent/auth-profiles.json
```

Optionen zur Behebung:

* Führe `openclaw agents add <id>` aus und konfiguriere die Authentifizierung im Assistenten.
* Oder kopiere `auth-profiles.json` aus dem `agentDir` des Hauptagenten in das `agentDir` des neuen agenten.

Verwende **nicht** dasselbe `agentDir` für mehrere Agenten; das verursacht Authentifizierungs-/Sitzungskonflikte.

<div id="model-failover-and-all-models-failed">
  ## Modell-Failover und „Alle Modelle sind fehlgeschlagen“
</div>

<div id="how-does-failover-work">
  ### Wie funktioniert Failover
</div>

Failover erfolgt in zwei Stufen:

1. **Rotation des Auth-Profils** innerhalb desselben Anbieters.
2. **Modell-Fallback** auf das nächste Modell in `agents.defaults.model.fallbacks`.

Cooldowns werden auf fehlschlagende Profile angewendet (exponentielles Backoff), sodass OpenClaw weiter antworten kann, selbst wenn ein Anbieter ein Rate-Limit erreicht oder vorübergehend ausfällt.

<div id="what-does-this-error-mean">
  ### Was bedeutet dieser Fehler?
</div>

```
No credentials found for profile "anthropic:default"
```

Das bedeutet, dass das System versucht hat, das Auth-Profil mit der ID `anthropic:default` zu verwenden, aber im erwarteten Auth-Store keine entsprechenden Zugangsdaten dafür gefunden hat.

<div id="fix-checklist-for-no-credentials-found-for-profile-anthropicdefault">
  ### Checkliste zur Behebung von „No credentials found for profile anthropicdefault“
</div>

* **Bestätige, wo Auth‑Profile liegen** (neue vs. Legacy‑Pfade)
  * Aktuell: `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`
  * Legacy: `~/.openclaw/agent/*` (migriert durch `openclaw doctor`)
* **Stelle sicher, dass deine Umgebungsvariable vom Gateway geladen wird**
  * Wenn du `ANTHROPIC_API_KEY` in deiner Shell setzt, das Gateway aber über systemd/launchd startest, erbt es die Variable möglicherweise nicht. Lege sie in `~/.openclaw/.env` ab oder aktiviere `env.shellEnv`.
* **Stelle sicher, dass du den richtigen agent bearbeitest**
  * Multi‑agent‑Setups bedeuten, dass es mehrere `auth-profiles.json`‑Dateien geben kann.
* **Kurzcheck für Modell/Auth‑Status**
  * Verwende `openclaw models status`, um konfigurierte Modelle zu sehen und ob Anbieter authentifiziert sind.

**Checkliste zur Behebung von „No credentials found for profile anthropic“**

Das bedeutet, dass die Ausführung an ein Anthropic‑Auth‑Profil gebunden ist, das Gateway
es aber nicht in seinem Auth‑Speicher finden kann.

* **Verwende ein Setup‑Token**
  * Führe `claude setup-token` aus und füge es dann mit `openclaw models auth setup-token --provider anthropic` ein.
  * Wenn das Token auf einer anderen Maschine erstellt wurde, verwende `openclaw models auth paste-token --provider anthropic`.
* **Wenn du stattdessen einen API‑Schlüssel verwenden möchtest**
  * Trage `ANTHROPIC_API_KEY` in `~/.openclaw/.env` auf dem **Gateway‑Host** ein.
  * Leere jede festgelegte Reihenfolge, die ein fehlendes Profil erzwingt:
    ```bash
    openclaw models auth order clear --provider anthropic
    ```
* **Stelle sicher, dass du Befehle auf dem Gateway‑Host ausführst**
  * Im Remote‑Modus liegen Auth‑Profile auf der Gateway‑Maschine, nicht auf deinem Laptop.

<div id="why-did-it-also-try-google-gemini-and-fail">
  ### Warum wurde auch Google Gemini ausprobiert und ist fehlgeschlagen
</div>

Wenn deine Modellkonfiguration Google Gemini als Fallback enthält (oder du zu einer Gemini-Kurzschreibweise gewechselt hast), versucht OpenClaw dies beim Modell-Fallback. Wenn du keine Google-Credentials konfiguriert hast, siehst du `No API key found for provider "google"`.

Lösung: Entweder Google-Authentifizierung bereitstellen oder Google-Modelle in `agents.defaults.model.fallbacks` / Aliassen entfernen bzw. vermeiden, damit der Fallback nicht dorthin geroutet wird.

**„LLM request rejected message thinking signature required google antigravity“**

Ursache: Der Sitzungsverlauf enthält **thinking-Blöcke ohne Signaturen** (oft aus einem abgebrochenen/teilweisen Stream). Google Antigravity erfordert Signaturen für thinking-Blöcke.

Lösung: OpenClaw entfernt jetzt nicht signierte thinking-Blöcke für Google Antigravity Claude. Wenn die Meldung weiterhin erscheint, starte eine **neue Sitzung** oder setze `/thinking off` für diesen agent.

<div id="auth-profiles-what-they-are-and-how-to-manage-them">
  ## Auth-Profile: Was sie sind und wie Sie sie verwalten
</div>

Verwandt: [/concepts/oauth](/de/concepts/oauth) (OAuth-Flows, Token-Speicherung, Multi-Account-Muster)

<div id="what-is-an-auth-profile">
  ### Was ist ein Auth-Profil
</div>

Ein Auth-Profil ist ein benannter Anmeldedatensatz (OAuth- oder API-Schlüssel), der einem Anbieter zugeordnet ist. Profile befinden sich in:

```
~/.openclaw/agents/<agentId>/agent/auth-profiles.json
```

<div id="what-are-typical-profile-ids">
  ### Was sind typische Profil‑IDs
</div>

OpenClaw verwendet anbieter‑präfixierte IDs wie:

* `anthropic:default` (üblich, wenn keine E‑Mail‑Identität vorhanden ist)
* `anthropic:<email>` für OAuth‑Identitäten
* benutzerdefinierte IDs deiner Wahl (z. B. `anthropic:work`)

<div id="can-i-control-which-auth-profile-is-tried-first">
  ### Kann ich steuern, welches Auth-Profil zuerst ausprobiert wird?
</div>

Ja. Die Konfiguration unterstützt optionale Metadaten für Profile und eine Reihenfolge pro anbieter (`auth.order.<provider>`). Dabei werden **keine** Secrets gespeichert; es werden IDs auf anbieter/Modus abgebildet und die Rotationsreihenfolge festgelegt.

OpenClaw kann ein Profil vorübergehend überspringen, wenn es sich in einem kurzen **Cooldown** befindet (Rate Limits/Timeouts/Auth-Fehler) oder in einem längeren **deaktivierten** Zustand (Abrechnung/unzureichendes Guthaben). Um das zu überprüfen, führe `openclaw models status --json` aus und prüfe `auth.unusableProfiles`. Feineinstellung: `auth.cooldowns.billingBackoffHours*`.

Du kannst außerdem eine **pro-Agent**-Reihenfolge-Überschreibung festlegen (gespeichert in der `auth-profiles.json` dieses Agents) über die CLI:

```bash
# Verwendet standardmäßig den konfigurierten Standard-Agent (--agent weglassen)
openclaw models auth order get --provider anthropic

# Lock rotation to a single profile (only try this one)
openclaw models auth order set --provider anthropic anthropic:default

# Or set an explicit order (fallback within provider)
openclaw models auth order set --provider anthropic anthropic:work anthropic:default

# Clear override (fall back to config auth.order / round-robin)
openclaw models auth order clear --provider anthropic
```

Um gezielt einen agent anzusprechen:

```bash
openclaw models auth order set --provider anthropic --agent main anthropic:default
```

<div id="oauth-vs-api-key-whats-the-difference">
  ### OAuth vs API-Key – Was ist der Unterschied
</div>

OpenClaw unterstützt beides:

* **OAuth** nutzt häufig Abonnementzugänge (wo zutreffend).
* **API-Keys** verwenden eine nutzungsbasierte Abrechnung pro Token.

Der Assistent unterstützt explizit Anthropic-Setup-Token und OpenAI-Codex-OAuth und kann API-Keys für Sie speichern.

<div id="gateway-ports-already-running-and-remote-mode">
  ## Gateway: Ports, „läuft bereits“ und Remote-Modus
</div>

<div id="what-port-does-the-gateway-use">
  ### Welchen Port verwendet das Gateway
</div>

`gateway.port` steuert den einzelnen multiplexed Port für WebSocket + HTTP (Control UI, Hooks usw.).

Reihenfolge:

```
--port > OPENCLAW_GATEWAY_PORT > gateway.port > default 18789
```

<div id="why-does-openclaw-gateway-status-say-runtime-running-but-rpc-probe-failed">
  ### Warum zeigt `openclaw gateway status` „Runtime running“ an, aber die RPC-Prüfung ist fehlgeschlagen?
</div>

Weil „running“ die Sicht des **Supervisors** ist (launchd/systemd/schtasks). Die RPC-Prüfung ist der eigentliche CLI-Aufruf, der sich mit dem Gateway-WebSocket verbindet und `status` aufruft.

Verwende `openclaw gateway status` und verlasse dich auf diese Zeilen:

* `Probe target:` (die URL, die die Prüfung tatsächlich verwendet hat)
* `Listening:` (was tatsächlich an den Port gebunden ist)
* `Last gateway error:` (häufige Ursache, wenn der Prozess läuft, aber der Port nicht auf Verbindungen lauscht)

<div id="why-does-openclaw-gateway-status-show-config-cli-and-config-service-different">
  ### Warum zeigt openclaw gateway status unterschiedliche Werte für Config cli und Config service an
</div>

Du bearbeitest eine Config-Datei, während der Dienst mit einer anderen läuft (häufig passen `--profile` und `OPENCLAW_STATE_DIR` nicht zusammen).

Lösung:

```bash
openclaw gateway install --force
```

Führen Sie den Befehl mit demselben `--profile` bzw. in derselben Umgebung aus, die der Dienst verwenden soll.

<div id="what-does-another-gateway-instance-is-already-listening-mean">
  ### Was bedeutet „another gateway instance is already listening“?
</div>

OpenClaw erzwingt einen Laufzeit-Lock, indem es den WS-Listener (WebSocket-Listener) direkt beim Start bindet (Standard: `ws://127.0.0.1:18789`). Wenn dieses Binden mit `EADDRINUSE` fehlschlägt, wird ein `GatewayLockError` ausgelöst, der darauf hinweist, dass bereits eine andere Instanz auf diesem Port lauscht.

Lösung: Beende die andere Instanz, gib den Port frei oder starte mit `openclaw gateway --port <port>`.

<div id="how-do-i-run-openclaw-in-remote-mode-client-connects-to-a-gateway-elsewhere">
  ### Wie führe ich OpenClaw im Remote-Modus aus, sodass sich ein Client mit einem entfernten Gateway verbindet?
</div>

Setze `gateway.mode: "remote"` und gib eine Remote-WebSocket-URL an, optional mit Token/Passwort:

```json5
{
  gateway: {
    mode: "remote",
    remote: {
      url: "ws://gateway.tailnet:18789",
      token: "your-token",
      password: "your-password"
    }
  }
}
```

Hinweise:

* `openclaw gateway` startet nur, wenn `gateway.mode` auf `local` gesetzt ist (oder du das Override-Flag übergibst).
* Die macOS-App überwacht die Konfigurationsdatei und wechselt den Modus live, sobald sich diese Werte ändern.

<div id="the-control-ui-says-unauthorized-or-keeps-reconnecting-what-now">
  ### Die Control UI zeigt „unauthorized“ an oder verbindet sich ständig neu – was nun
</div>

Dein Gateway läuft mit aktivierter Authentifizierung (`gateway.auth.*`), aber die UI sendet nicht das passende Token/Passwort.

Fakten (aus dem Code):

* Die Control UI speichert das Token im `localStorage`-Schlüssel des Browsers `openclaw.control.settings.v1`.
* Die UI kann `?token=...` (und/oder `?password=...`) einmalig importieren und entfernt es dann aus der URL.

Lösung:

* Am schnellsten: `openclaw dashboard` (gibt einen tokenisierten Link aus, kopiert ihn, versucht ihn zu öffnen; zeigt einen SSH-Hinweis bei Headless-Setups).
* Wenn du noch kein Token hast: `openclaw doctor --generate-gateway-token`.
* Bei Remote-Zugriff zuerst tunneln: `ssh -N -L 18789:127.0.0.1:18789 user@host` und dann `http://127.0.0.1:18789/?token=...` öffnen.
* Setze `gateway.auth.token` (oder `OPENCLAW_GATEWAY_TOKEN`) auf dem Gateway-Host.
* Füge in den Control-UI-Einstellungen dasselbe Token ein (oder aktualisiere mit einem einmaligen `?token=...`-Link).
* Immer noch Probleme? Führe `openclaw status --all` aus und folge [Troubleshooting](/de/gateway/troubleshooting). Details zur Authentifizierung findest du unter [Dashboard](/de/web/dashboard).

<div id="i-set-gatewaybind-tailnet-but-it-cant-bind-nothing-listens">
  ### Ich habe gatewaybind auf tailnet gesetzt, aber es kann nichts binden, nichts lauscht
</div>

Das `tailnet`-Binding wählt eine Tailscale-IP aus deinen Netzwerk-Interfaces (100.64.0.0/10). Wenn die Maschine nicht in Tailscale eingebunden ist (oder das Interface down ist), gibt es nichts, woran gebunden werden kann.

Lösung:

* Starte Tailscale auf diesem Host (damit er eine 100.x-Adresse hat), oder
* wechsle zu `gateway.bind: "loopback"` / `"lan"`.

Hinweis: `tailnet` ist explizit. `auto` bevorzugt loopback; verwende `gateway.bind: "tailnet"`, wenn du eine Bindung ausschließlich ans Tailnet möchtest.

<div id="can-i-run-multiple-gateways-on-the-same-host">
  ### Kann ich mehrere Gateways auf demselben Host ausführen?
</div>

In der Regel nein – ein Gateway kann mehrere Messaging‑Kanäle und Agenten ausführen. Verwende mehrere Gateways nur, wenn du Redundanz (z. B. Rescue‑Bot) oder harte Isolation benötigst.

Ja, aber du musst Folgendes isolieren:

* `OPENCLAW_CONFIG_PATH` (Konfiguration pro Instanz)
* `OPENCLAW_STATE_DIR` (Statusdaten pro Instanz)
* `agents.defaults.workspace` (Arbeitsbereichs‑Isolation)
* `gateway.port` (eindeutige Ports)

Schnell‑Setup (empfohlen):

* Verwende `openclaw --profile <name> …` pro Instanz (erstellt automatisch `~/.openclaw-<name>`).
* Setze einen eindeutigen `gateway.port` in jeder Profil‑Config (oder übergib `--port` für manuelle Starts).
* Installiere einen eigenen Service pro Profil: `openclaw --profile <name> gateway install`.

Profile werden auch als Suffix an Servicenamen angehängt (`bot.molt.<profile>`; Legacy `com.openclaw.*`, `openclaw-gateway-<profile>.service`, `OpenClaw Gateway (<profile>)`).
Komplette Anleitung: [Mehrere Gateways](/de/gateway/multiple-gateways).

<div id="what-does-invalid-handshake-code-1008-mean">
  ### Was bedeutet Handshake-Fehlercode 1008
</div>

Das Gateway ist ein **WebSocket-Server** und erwartet, dass die erste Nachricht
ein `connect`‑Frame ist. Wenn es etwas anderes erhält, schließt es die Verbindung
mit **Code 1008** (Richtlinienverstoß).

Häufige Ursachen:

* Du hast die **HTTP**‑URL im Browser (`http://...`) statt in einem WS‑Client geöffnet.
* Du hast den falschen Port oder Pfad verwendet.
* Ein Proxy oder Tunnel hat Auth-Header entfernt oder eine Nicht-Gateway-Anfrage geschickt.

Schnelle Lösungen:

1. Verwende die WS‑URL: `ws://<host>:18789` (oder `wss://...` bei HTTPS).
2. Öffne den WS‑Port nicht in einem normalen Browser-Tab.
3. Wenn Authentifizierung aktiv ist, sende das Token/Passwort im `connect`‑Frame mit.

Wenn du die CLI oder TUI verwendest, sollte die URL so aussehen:

```
openclaw tui --url ws://<host>:18789 --token <token>
```

Details zum Protokoll: [Gateway-Protokoll](/de/gateway/protocol).

<div id="logging-and-debugging">
  ## Logging und Debugging
</div>

<div id="where-are-logs">
  ### Wo befinden sich die Logs?
</div>

Dateibasierte Logs (strukturiert):

```
/tmp/openclaw/openclaw-YYYY-MM-DD.log
```

Du kannst einen festen Pfad über `logging.file` setzen. Das Log-Level für Dateien wird über `logging.level` gesteuert. Die Ausführlichkeit der Konsolenausgabe wird über `--verbose` und `logging.consoleLevel` gesteuert.

Schnellstes Tail des Logs:

```bash
openclaw logs --follow
```

Service-/Supervisor-Protokolle (wenn das Gateway über launchd/systemd betrieben wird):

* macOS: `$OPENCLAW_STATE_DIR/logs/gateway.log` und `gateway.err.log` (Standard: `~/.openclaw/logs/...`; Profile verwenden `~/.openclaw-<profile>/logs/...`)
* Linux: `journalctl --user -u openclaw-gateway[-<profile>].service -n 200 --no-pager`
* Windows: `schtasks /Query /TN "OpenClaw Gateway (<profile>)" /V /FO LIST`

Siehe [Troubleshooting](/de/gateway/troubleshooting#log-locations) für weitere Informationen.

<div id="how-do-i-startstoprestart-the-gateway-service">
  ### Wie kann ich den Gateway-Dienst starten, stoppen oder neu starten
</div>

Verwende die Gateway-Hilfsprogramme:

```bash
openclaw gateway status
openclaw gateway restart
```

Wenn du das Gateway manuell startest, kann `openclaw gateway --force` den Port wieder übernehmen. Siehe [Gateway](/de/gateway).

<div id="i-closed-my-terminal-on-windows-how-do-i-restart-openclaw">
  ### Ich habe mein Terminal unter Windows geschlossen – wie starte ich OpenClaw neu?
</div>

Es gibt **zwei Windows-Installationsmodi**:

**1) WSL2 (empfohlen):** das Gateway läuft unter Linux.

Öffne PowerShell, wechsle in deine WSL-Umgebung und starte dann neu:

```powershell
wsl
openclaw gateway status
openclaw gateway restart
```

Falls du den Dienst noch nicht installiert hast, starte ihn im Vordergrund:

```bash
openclaw gateway run
```

**2) Native Windows (nicht empfohlen):** Das Gateway läuft direkt unter Windows.

Öffne PowerShell und führe Folgendes aus:

```powershell
openclaw gateway status
openclaw gateway restart
```

Wenn Sie es manuell ausführen (ohne Dienst), verwenden Sie:

```powershell
openclaw gateway run
```

Dokumentation: [Windows (WSL2)](/de/platforms/windows), [Runbook für den Gateway-Service](/de/gateway).

<div id="the-gateway-is-up-but-replies-never-arrive-what-should-i-check">
  ### Der Gateway läuft, aber Antworten kommen nie an – was sollte ich überprüfen?
</div>

Beginne mit einem schnellen Health-Check:

```bash
openclaw status
openclaw models status
openclaw channels status
openclaw logs --follow
```

Häufige Ursachen:

* Model-Authentifizierung ist auf dem **Gateway-Host** nicht geladen (überprüfe `models status`).
* Kanal-Kopplung/Allowlist blockiert Antworten (überprüfe Kanal-Konfiguration und Logs).
* WebChat/Dashboard ist geöffnet, aber ohne das richtige Token.

Wenn du remote zugreifst, prüfe, ob der Tunnel bzw. die Tailscale-Verbindung aktiv ist und ob der
Gateway-WebSocket erreichbar ist.

Dokumentation: [Channels](/de/channels), [Troubleshooting](/de/gateway/troubleshooting), [Remote access](/de/gateway/remote).

<div id="disconnected-from-gateway-no-reason-what-now">
  ### Vom Gateway getrennt, kein ersichtlicher Grund – was jetzt
</div>

Das bedeutet in der Regel, dass die UI die WebSocket-Verbindung verloren hat. Überprüfe Folgendes:

1. Läuft das Gateway? `openclaw gateway status`
2. Ist das Gateway gesund? `openclaw status`
3. Hat die UI das richtige Token? `openclaw dashboard`
4. Falls remote: Ist der Tunnel/Tailscale-Link aktiv?

Dann Logs mit `tail` verfolgen:

```bash
openclaw logs --follow
```

Dokumentation: [Dashboard](/de/web/dashboard), [Remotezugriff](/de/gateway/remote), [Fehlerbehebung](/de/gateway/troubleshooting).

<div id="telegram-setmycommands-fails-with-network-errors-what-should-i-check">
  ### Telegram setMyCommands schlägt mit Netzwerkfehlern fehl – was sollte ich überprüfen?
</div>

Beginne mit den Logs und dem Channel-Status:

```bash
openclaw channels status
openclaw channels logs --channel telegram
```

Wenn du OpenClaw auf einem VPS betreibst oder hinter einem Proxy, stelle sicher, dass ausgehendes HTTPS zugelassen ist und DNS funktioniert.
Wenn das Gateway remote läuft, vergewissere dich, dass du dir die Logs auf dem Gateway-Host ansiehst.

Dokumentation: [Telegram](/de/channels/telegram), [Channel-Fehlerbehebung](/de/channels/troubleshooting).

<div id="tui-shows-no-output-what-should-i-check">
  ### TUI zeigt keine Ausgabe – was sollte ich überprüfen?
</div>

Stelle zunächst sicher, dass das Gateway erreichbar ist und der agent ausgeführt werden kann:

```bash
openclaw status
openclaw models status
openclaw logs --follow
```

In der TUI verwendest du `/status`, um den aktuellen Status anzuzeigen. Wenn du Antworten in einem Chatkanal erwartest, stelle sicher, dass die Zustellung aktiviert ist (`/deliver on`).

Dokumentation: [TUI](/de/tui), [Slash-Befehle](/de/tools/slash-commands).

<div id="how-do-i-completely-stop-then-start-the-gateway">
  ### Wie stoppe ich das Gateway vollständig und starte es dann neu?
</div>

Wenn du den Dienst installiert hast:

```bash
openclaw gateway stop
openclaw gateway start
```

Dies stoppt/startet den **überwachten Dienst** (launchd auf macOS, systemd auf Linux).
Verwende dies, wenn der Gateway im Hintergrund als Daemon läuft.

Wenn du den Gateway im Vordergrund betreibst, beende ihn mit Strg‑C und führe dann Folgendes aus:

```bash
openclaw gateway run
```

Dokumentation: [Gateway-Service-Runbook](/de/gateway).

<div id="eli5-openclaw-gateway-restart-vs-openclaw-gateway">
  ### ELI5 openclaw gateway restart vs openclaw gateway
</div>

* `openclaw gateway restart`: startet den **Hintergrunddienst** (launchd/systemd) neu.
* `openclaw gateway`: führt das Gateway **im Vordergrund** für diese Terminal-Sitzung aus.

Wenn du den Dienst installiert hast, verwende die Gateway-Befehle. Verwende `openclaw gateway`, wenn
du einen einmaligen Lauf im Vordergrund möchtest.

<div id="whats-the-fastest-way-to-get-more-details-when-something-fails">
  ### Was ist der schnellste Weg, mehr Details zu erhalten, wenn etwas fehlschlägt
</div>

Starte das Gateway mit `--verbose`, um mehr Details in der Konsole zu erhalten. Analysiere anschließend die Protokolldatei auf Channel-Authentifizierung, Modell-Routing und RPC-Fehler.

<div id="media-attachments">
  ## Medien und Anhänge
</div>

<div id="my-skill-generated-an-imagepdf-but-nothing-was-sent">
  ### Mein Skill hat ein imagePDF erzeugt, aber nichts wurde gesendet
</div>

Ausgehende Anhänge vom Agent müssen eine Zeile `MEDIA:<path-or-url>` enthalten (alleinstehend in einer eigenen Zeile). Siehe [OpenClaw assistant setup](/de/start/openclaw) und [Agent send](/de/tools/agent-send).

Senden per CLI:

```bash
openclaw message send --target +15555550123 --message "Hier ist es" --media /path/to/file.png
```

Siehe auch:

* Der Zielkanal unterstützt ausgehende Medien und wird nicht durch die Allowlist blockiert.
* Die Datei liegt innerhalb der Größenlimits des Anbieters (Bilder werden auf maximal 2048px skaliert).

Siehe [Images](/de/nodes/images).

<div id="security-and-access-control">
  ## Sicherheit und Zugriffskontrolle
</div>

<div id="is-it-safe-to-expose-openclaw-to-inbound-dms">
  ### Ist es sicher, OpenClaw für eingehende DMs zugänglich zu machen?
</div>

Behandle eingehende DMs als nicht vertrauenswürdige Eingaben. Die Standardeinstellungen sind so gestaltet, dass sie das Risiko reduzieren:

* Das Standardverhalten auf DM‑fähigen Kanälen ist **Kopplung**:
  * Unbekannte Absender erhalten einen Kopplungscode; der Bot verarbeitet ihre Nachricht nicht.
  * Genehmige mit: `openclaw pairing approve <channel> <code>`
  * Ausstehende Anfragen sind auf **3 pro Kanal** begrenzt; prüfe `openclaw pairing list <channel>`, wenn kein Code angekommen ist.
* Das öffentliche Öffnen von DMs erfordert ein ausdrückliches Opt‑in (`dmPolicy: "open"` – erlaubt uneingeschränkten Nachrichteneingang von beliebigen Nutzern – und Allowlist `"*"`).

Führe `openclaw doctor` aus, um riskante DM-Richtlinien sichtbar zu machen.

<div id="is-prompt-injection-only-a-concern-for-public-bots">
  ### Ist Prompt-Injection nur ein Problem für öffentliche Bots
</div>

Nein. Prompt-Injection betrifft **nicht vertrauenswürdige Inhalte**, nicht nur die Frage, wer dem Bot Direktnachrichten schicken kann.
Wenn dein Assistent externe Inhalte liest (Websuche/-fetch, Browser-Seiten, E-Mails,
Dokumente, Anhänge, eingefügte Logs), können diese Inhalte Anweisungen enthalten, die versuchen,
das Modell zu kapern. Das kann passieren, selbst wenn **du der einzige Absender bist**.

Das größte Risiko besteht, wenn Tools aktiviert sind: Das Modell kann dazu gebracht werden,
Kontext zu exfiltrieren oder Tools in deinem Namen aufzurufen. Verringere den Schaden, indem du:

* einen schreibgeschützten oder tool-deaktivierten „Reader“-agent verwendest, um nicht vertrauenswürdige Inhalte zusammenzufassen
* `web_search` / `web_fetch` / `browser` für Tool-aktivierte Agenten deaktiviert lässt
* sandboxing und strikte Tool-Allowlists verwendest

Details: [Security](/de/gateway/security).

<div id="should-my-bot-have-its-own-email-github-account-or-phone-number">
  ### Sollte mein Bot ein eigenes E‑Mail-, GitHub-Konto oder eine eigene Telefonnummer haben
</div>

Ja, für die meisten Setups. Die Isolierung des Bots mit separaten Konten und Telefonnummern
begrenzt die Auswirkungen, falls etwas schiefgeht. Außerdem wird es dadurch einfacher,
Zugangsdaten zu erneuern oder den Zugriff zu widerrufen, ohne deine persönlichen Konten zu beeinträchtigen.

Fang klein an. Gewähre nur Zugriff auf die Tools und Konten, die du tatsächlich brauchst, und erweitere
später bei Bedarf.

Docs: [Security](/de/gateway/security), [Kopplung](/de/start/pairing).

<div id="can-i-give-it-autonomy-over-my-text-messages-and-is-that-safe">
  ### Kann ich ihm Autonomie über meine Textnachrichten geben und ist das sicher
</div>

Wir empfehlen **keine** vollständige Autonomie über deine persönlichen Nachrichten. Das sicherste Vorgehen ist:

* Halte DMs im **Kopplungsmodus** oder in einer strikten Allowlist.
* Verwende eine **separate Nummer oder ein separates Konto**, wenn es in deinem Namen Nachrichten senden soll.
* Lass es Entwürfe erstellen und **gib sie vor dem Senden frei**.

Wenn du experimentieren möchtest, tu das mit einem dedizierten Konto und halte es isoliert. Siehe
[Security](/de/gateway/security).

<div id="can-i-use-cheaper-models-for-personal-assistant-tasks">
  ### Kann ich günstigere Modelle für persönliche Assistenzaufgaben verwenden
</div>

Ja, **wenn** der Agent ausschließlich im Chat-Modus läuft und die Eingaben vertrauenswürdig sind. Kleinere Modellstufen sind anfälliger für Prompt-/Instruktions-Hijacking, daher solltest du sie bei tool-fähigen Agenten oder beim Lesen nicht vertrauenswürdiger Inhalte vermeiden. Wenn du ein kleineres Modell verwenden musst, beschränke die verfügbaren Tools strikt und führe es in einer sandbox aus. Siehe [Sicherheit](/de/gateway/security).

<div id="i-ran-start-in-telegram-but-didnt-get-a-pairing-code">
  ### Ich habe in Telegram /start ausgeführt, aber keinen Kopplungscode erhalten
</div>

Kopplungscodes werden **nur** gesendet, wenn ein unbekannter Absender dem Bot
schreibt und `dmPolicy: "pairing"` aktiviert ist. `/start` allein erzeugt keinen Code.

Prüfe ausstehende Anfragen:

```bash
openclaw pairing list telegram
```

Wenn du sofortigen Zugriff möchtest, füge deine Sender-ID zur Allowlist hinzu oder setze für dieses Konto `dmPolicy: "open"` (erlaubt den uneingeschränkten Empfang von Nachrichten von beliebigen Nutzern).

<div id="whatsapp-will-it-message-my-contacts-how-does-pairing-work">
  ### Schreibt WhatsApp meine Kontakte an Wie funktioniert die Kopplung
</div>

Nein. Die Standardrichtlinie für WhatsApp-DMs ist **Kopplung**. Unbekannte Absender erhalten nur einen Kopplungscode und ihre Nachricht wird **nicht verarbeitet**. OpenClaw antwortet nur auf Chats, die es erhält, oder auf explizite Sendeaktionen, die du auslöst.

Genehmige die Kopplung mit:

```bash
openclaw pairing approve whatsapp <code>
```

Ausstehende Anfragen auflisten:

```bash
openclaw pairing list whatsapp
```

Wizard-Telefonnummer-Abfrage: Sie wird verwendet, um deine **Allowlist/Owner** festzulegen, damit deine eigenen DMs zugelassen werden. Sie wird nicht zum automatischen Versenden verwendet. Wenn du deine persönliche WhatsApp-Nummer verwendest, nutze diese Nummer und aktiviere `channels.whatsapp.selfChatMode`.

<div id="chat-commands-aborting-tasks-and-it-wont-stop">
  ## Chat-Befehle, Aufgaben abbrechen und „es hört einfach nicht auf“
</div>

<div id="how-do-i-stop-internal-system-messages-from-showing-in-chat">
  ### Wie verhindere ich, dass interne Systemnachrichten im Chat angezeigt werden?
</div>

Die meisten internen Nachrichten oder Tool-Nachrichten werden nur angezeigt, wenn **verbose** oder **reasoning** für diese Sitzung aktiviert ist.

Korrigiere das direkt in dem Chat, in dem du es siehst:

```
/verbose off
/reasoning off
```

Wenn es weiterhin zu viel Ausgabe gibt, prüfe die Sitzungseinstellungen in der Control UI und setze `verbose`
auf **inherit**. Stelle außerdem sicher, dass du kein Bot-Profil mit `verboseDefault` auf `on` in der Konfiguration verwendest.

Dokumentation: [Thinking and verbose](/de/tools/thinking), [Sicherheit](/de/gateway/security#reasoning--verbose-output-in-groups).

<div id="how-do-i-stopcancel-a-running-task">
  ### Wie stoppe oder breche ich eine laufende Aufgabe ab?
</div>

Sende eine dieser Optionen **als eigenständige Nachricht** (ohne Slash):

```
stop
abort
esc
wait
exit
interrupt
```

Dies sind Abbruch-Trigger (keine Slash-Befehle).

Für Hintergrundprozesse (aus dem `exec`-Tool) kannst du den Agent anweisen, Folgendes auszuführen:

```
process action:kill sessionId:XXX
```

Übersicht über Slash-Befehle: siehe [Slash commands](/de/tools/slash-commands).

Die meisten Befehle müssen als **eigenständige** Nachricht gesendet werden, die mit `/` beginnt, aber einige Shortcuts (wie `/status`) funktionieren auch inline für Absender in der Allowlist.

<div id="how-do-i-send-a-discord-message-from-telegram-crosscontext-messaging-denied">
  ### Wie sende ich von Telegram aus eine Discord‑Nachricht – Cross‑Context‑Messaging verweigert
</div>

OpenClaw unterbindet standardmäßig **anbieterübergreifende** Nachrichten. Wenn ein Tool‑Aufruf an Telegram gebunden ist, wird er nicht an Discord gesendet, es sei denn, du erlaubst das ausdrücklich.

Aktiviere anbieterübergreifende Nachrichten für den agent:

```json5
{
  agents: {
    defaults: {
      tools: {
        message: {
          crossContext: {
            allowAcrossProviders: true,
            marker: { enabled: true, prefix: "[von {channel}] " }
          }
        }
      }
    }
  }
}
```

Starte das Gateway nach dem Bearbeiten der Konfiguration neu. Wenn du dies nur für einen einzelnen
agent möchtest, setze es stattdessen unter `agents.list[].tools.message`.

<div id="why-does-it-feel-like-the-bot-ignores-rapidfire-messages">
  ### Warum wirkt es so, als würde der Bot schnell aufeinanderfolgende Nachrichten ignorieren?
</div>

Der Queue‑Modus steuert, wie neue Nachrichten mit einer laufenden Ausführung interagieren. Verwende `/queue`, um den Modus zu ändern:

* `steer` - neue Nachrichten lenken die aktuelle Aufgabe um
* `followup` - Nachrichten der Reihe nach einzeln verarbeiten
* `collect` - Nachrichten sammeln und einmalig antworten (Standard)
* `steer-backlog` - jetzt steuern, dann den Backlog abarbeiten
* `interrupt` - aktuelle Ausführung abbrechen und neu starten

Du kannst Optionen wie `debounce:2s cap:25 drop:summarize` für Followup‑Modi hinzufügen.

<div id="answer-the-exact-question-from-the-screenshotchat-log">
  ## Beantworte exakt die Frage aus dem Screenshot/Chat-Log
</div>

**F: „Was ist das Standardmodell für Anthropic mit einem API-Schlüssel?“**

**A:** In OpenClaw sind Zugangsdaten und Modellauswahl voneinander getrennt. Das Setzen von `ANTHROPIC_API_KEY` (oder das Speichern eines Anthropic API-Schlüssels in Auth-Profilen) aktiviert die Authentifizierung, aber das eigentliche Standardmodell ist das, was du in `agents.defaults.model.primary` konfigurierst (zum Beispiel `anthropic/claude-sonnet-4-5` oder `anthropic/claude-opus-4-5`). Wenn du `No credentials found for profile "anthropic:default"` siehst, bedeutet das, dass das Gateway keine Anthropic-Zugangsdaten in der erwarteten `auth-profiles.json` für den laufenden Agenten finden konnte.

***

Immer noch Probleme? Frag auf [Discord](https://discord.com/invite/clawd) oder eröffne eine [GitHub-Diskussion](https://github.com/openclaw/openclaw/discussions).