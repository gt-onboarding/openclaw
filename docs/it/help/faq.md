---
title: FAQ
summary: "Domande frequenti sull'installazione, la configurazione e l'utilizzo di OpenClaw"
---

<div id="faq">
  # FAQ
</div>

Risposte rapide e indicazioni più approfondite per la risoluzione dei problemi in scenari reali (sviluppo locale, VPS, configurazioni multi‑agente, chiavi OAuth/API, failover dei modelli). Per la diagnostica a runtime, consulta [Troubleshooting](/it/gateway/troubleshooting). Per il riferimento completo alla configurazione, consulta [Configuration](/it/gateway/configuration).

<div id="table-of-contents">
  ## Indice
</div>

* [Guida rapida e configurazione al primo avvio](#quick-start-and-firstrun-setup)
  * [Sono bloccato, qual è il modo più veloce per sbloccarmi?](#im-stuck-whats-the-fastest-way-to-get-unstuck)
  * [Qual è il modo consigliato per installare e configurare OpenClaw?](#whats-the-recommended-way-to-install-and-set-up-openclaw)
  * [Come apro la dashboard dopo l’onboarding?](#how-do-i-open-the-dashboard-after-onboarding)
  * [Come autentico la dashboard (token) su localhost rispetto a una macchina remota?](#how-do-i-authenticate-the-dashboard-token-on-localhost-vs-remote)
  * [Di quale runtime ho bisogno?](#what-runtime-do-i-need)
  * [Funziona su Raspberry Pi?](#does-it-run-on-raspberry-pi)
  * [Qualche consiglio per le installazioni su Raspberry Pi?](#any-tips-for-raspberry-pi-installs)
  * [È bloccato su &quot;wake up my friend&quot; / l’onboarding non si completa. E adesso?](#it-is-stuck-on-wake-up-my-friend-onboarding-will-not-hatch-what-now)
  * [Posso migrare la mia configurazione su una nuova macchina (Mac mini) senza rifare l’onboarding?](#can-i-migrate-my-setup-to-a-new-machine-mac-mini-without-redoing-onboarding)
  * [Dove posso vedere le novità dell’ultima versione?](#where-do-i-see-whats-new-in-the-latest-version)
  * [Non riesco ad accedere a docs.openclaw.ai (errore SSL). E adesso?](#i-cant-access-docsopenclawai-ssl-error-what-now)
  * [Qual è la differenza tra stable e beta?](#whats-the-difference-between-stable-and-beta)
* [Come posso installare la versione beta e qual è la differenza tra beta e dev?](#how-do-i-install-the-beta-version-and-whats-the-difference-between-beta-and-dev)
  * [Come posso provare l&#39;ultima versione disponibile?](#how-do-i-try-the-latest-bits)
  * [Quanto tempo richiedono in genere installazione e onboarding?](#how-long-does-install-and-onboarding-usually-take)
  * [Programma di installazione bloccato? Come ottenere più dettagli?](#installer-stuck-how-do-i-get-more-feedback)
  * [L&#39;installazione su Windows segnala che Git non è stato trovato o che openclaw non è riconosciuto](#windows-install-says-git-not-found-or-openclaw-not-recognized)
  * [La documentazione non ha risposto alla mia domanda: come posso ottenere una risposta migliore?](#the-docs-didnt-answer-my-question-how-do-i-get-a-better-answer)
  * [Come posso installare OpenClaw su Linux?](#how-do-i-install-openclaw-on-linux)
  * [Come faccio a installare OpenClaw su un VPS?](#how-do-i-install-openclaw-on-a-vps)
  * [Dove si trovano le guide per l&#39;installazione su cloud/VPS?](#where-are-the-cloudvps-install-guides)
  * [Posso chiedere a OpenClaw di aggiornarsi automaticamente?](#can-i-ask-openclaw-to-update-itself)
  * [Cosa fa in pratica la procedura guidata di onboarding?](#what-does-the-onboarding-wizard-actually-do)
  * [Ho bisogno di un abbonamento a Claude o OpenAI per eseguirlo?](#do-i-need-a-claude-or-openai-subscription-to-run-this)
  * [Posso utilizzare l&#39;abbonamento a Claude Max senza una chiave api](#can-i-use-claude-max-subscription-without-an-api-key)
  * [Come funziona l&#39;autenticazione &quot;setup-token&quot; di Anthropic?](#how-does-anthropic-setuptoken-auth-work)
  * [Dove posso trovare un token di configurazione Anthropic?](#where-do-i-find-an-anthropic-setuptoken)
  * [Supportate l&#39;autenticazione con abbonamento a Claude (Claude Code OAuth)?](#do-you-support-claude-subscription-auth-claude-code-oauth)
  * [Perché ricevo `HTTP 429: rate_limit_error` da Anthropic?](#why-am-i-seeing-http-429-ratelimiterror-from-anthropic)
  * [AWS Bedrock è supportato?](#is-aws-bedrock-supported)
  * [Come funziona l&#39;autenticazione di Codex?](#how-does-codex-auth-work)
  * [Supportate l&#39;autenticazione tramite abbonamento a OpenAI (Codex OAuth)?](#do-you-support-openai-subscription-auth-codex-oauth)
  * [Come configuro l&#39;OAuth per la CLI Gemini](#how-do-i-set-up-gemini-cli-oauth)
  * [Un modello locale va bene per conversazioni informali?](#is-a-local-model-ok-for-casual-chats)
  * [Come posso mantenere il traffico dei modelli ospitati all&#39;interno di una regione specifica?](#how-do-i-keep-hosted-model-traffic-in-a-specific-region)
  * [Devo acquistare un Mac mini per installarlo?](#do-i-have-to-buy-a-mac-mini-to-install-this)
  * [Mi serve un Mac mini per il supporto a iMessage?](#do-i-need-a-mac-mini-for-imessage-support)
  * [Se acquisto un Mac mini per eseguire OpenClaw, posso collegarlo al mio MacBook Pro?](#if-i-buy-a-mac-mini-to-run-openclaw-can-i-connect-it-to-my-macbook-pro)
  * [Posso usare Bun?](#can-i-use-bun)
  * [Telegram: cosa inserire in `allowFrom`?](#telegram-what-goes-in-allowfrom)
  * [Possono più persone usare lo stesso numero WhatsApp con istanze OpenClaw diverse?](#can-multiple-people-use-one-whatsapp-number-with-different-openclaw-instances)
  * [Posso eseguire un agente &quot;fast chat&quot; e un agente &quot;Opus for coding&quot;?](#can-i-run-a-fast-chat-agent-and-an-opus-for-coding-agent)
  * [Funziona Homebrew su Linux?](#does-homebrew-work-on-linux)
  * [Qual è la differenza tra l&#39;installazione &quot;hackable&quot; (git) e quella tramite npm?](#whats-the-difference-between-the-hackable-git-install-and-npm-install)
  * [Posso passare tra installazioni npm e git in un secondo momento?](#can-i-switch-between-npm-and-git-installs-later)
  * [È meglio eseguire il Gateway sul mio laptop o su un VPS?](#should-i-run-the-gateway-on-my-laptop-or-a-vps)
  * [Quanto è importante eseguire OpenClaw su una macchina dedicata?](#how-important-is-it-to-run-openclaw-on-a-dedicated-machine)
  * [Quali sono i requisiti minimi per il VPS e quale sistema operativo è consigliato?](#what-are-the-minimum-vps-requirements-and-recommended-os)
  * [Posso eseguire OpenClaw in una VM e quali sono i requisiti di sistema](#can-i-run-openclaw-in-a-vm-and-what-are-the-requirements)
* [Che cos&#39;è OpenClaw?](#what-is-openclaw)
  * [Che cos&#39;è OpenClaw, in un solo paragrafo?](#what-is-openclaw-in-one-paragraph)
  * [Qual è la proposta di valore?](#whats-the-value-proposition)
  * [L&#39;ho appena configurato: cosa dovrei fare per prima cosa?](#i-just-set-it-up-what-should-i-do-first)
  * [Quali sono i cinque principali casi d&#39;uso quotidiani per OpenClaw?](#what-are-the-top-five-everyday-use-cases-for-openclaw)
  * [OpenClaw può aiutare con lead generation, outreach, annunci e articoli per il blog per un SaaS?](#can-openclaw-help-with-lead-gen-outreach-ads-and-blogs-for-a-saas)
  * [Quali sono i vantaggi rispetto a Claude Code per lo sviluppo web?](#what-are-the-advantages-vs-claude-code-for-web-development)
* [Abilità e automazione](#skills-and-automation)
  * [Come posso personalizzare le abilità senza mantenere la repo sporca?](#how-do-i-customize-skills-without-keeping-the-repo-dirty)
  * [Posso caricare abilità da una cartella personalizzata?](#can-i-load-skills-from-a-custom-folder)
  * [Come posso usare modelli diversi per attività diverse?](#how-can-i-use-different-models-for-different-tasks)
  * [Il bot si blocca durante lavori pesanti. Come posso scaricare quel carico?](#the-bot-freezes-while-doing-heavy-work-how-do-i-offload-that)
  * [Cron o i promemoria non vengono eseguiti. Cosa dovrei verificare?](#cron-or-reminders-do-not-fire-what-should-i-check)
  * [Come installo le abilità su Linux?](#how-do-i-install-skills-on-linux)
  * [OpenClaw può eseguire attività in base a una pianificazione o continuamente in background?](#can-openclaw-run-tasks-on-a-schedule-or-continuously-in-the-background)
  * [Posso eseguire da Linux abilità disponibili solo per Apple/macOS?](#can-i-run-applemacosonly-skills-from-linux)
  * [Avete un&#39;integrazione con Notion o HeyGen?](#do-you-have-a-notion-or-heygen-integration)
  * [Come installo l&#39;estensione di Chrome per il controllo del browser?](#how-do-i-install-the-chrome-extension-for-browser-takeover)
* [Sandbox e memoria](#sandboxing-and-memory)
  * [Esiste una documentazione dedicata al sandboxing?](#is-there-a-dedicated-sandboxing-doc)
  * [Come faccio a montare una cartella dell&#39;host nella sandbox?](#how-do-i-bind-a-host-folder-into-the-sandbox)
  * [Come funziona la memoria?](#how-does-memory-work)
  * [La memoria continua a dimenticare le cose. Come faccio a far sì che le ricordi?](#memory-keeps-forgetting-things-how-do-i-make-it-stick)
  * [La memoria viene conservata per sempre? Quali sono i limiti?](#does-memory-persist-forever-what-are-the-limits)
  * [La ricerca nella memoria semantica richiede una chiave API OpenAI?](#does-semantic-memory-search-require-an-openai-api-key)
* [Dove si trovano i file su disco](#where-things-live-on-disk)
  * [Tutti i dati utilizzati con OpenClaw vengono salvati localmente?](#is-all-data-used-with-openclaw-saved-locally)
  * [Dove OpenClaw archivia i propri dati?](#where-does-openclaw-store-its-data)
  * [Dove devono trovarsi AGENTS.md / SOUL.md / USER.md / MEMORY.md?](#where-should-agentsmd-soulmd-usermd-memorymd-live)
  * [Qual è la strategia di backup consigliata?](#whats-the-recommended-backup-strategy)
  * [Come faccio a disinstallare completamente OpenClaw?](#how-do-i-completely-uninstall-openclaw)
  * [Gli agenti possono operare al di fuori dello spazio di lavoro?](#can-agents-work-outside-the-workspace)
  * [Sono in modalità remota: dove si trova l’archivio delle sessioni?](#im-in-remote-mode-where-is-the-session-store)
* [Nozioni di base sulla configurazione](#config-basics)
  * [Qual è il formato del file di config? Dove si trova?](#what-format-is-the-config-where-is-it)
  * [Ho impostato `gateway.bind: "lan"` (o `"tailnet"`) e ora non c&#39;è più nulla in ascolto / la UI dice &quot;unauthorized&quot;](#i-set-gatewaybind-lan-or-tailnet-and-now-nothing-listens-the-ui-says-unauthorized)
  * [Perché ora mi serve un token anche su localhost?](#why-do-i-need-a-token-on-localhost-now)
  * [Devo riavviare dopo aver cambiato la config?](#do-i-have-to-restart-after-changing-config)
  * [Come abilito web search (e web fetch)?](#how-do-i-enable-web-search-and-web-fetch)
  * [`config.apply` ha azzerato la mia config. Come posso ripristinarla ed evitare che succeda di nuovo?](#configapply-wiped-my-config-how-do-i-recover-and-avoid-this)
  * [Come eseguo un Gateway centrale con worker specializzati distribuiti su più dispositivi?](#how-do-i-run-a-central-gateway-with-specialized-workers-across-devices)
  * [Il browser di OpenClaw può essere eseguito in modalità headless?](#can-the-openclaw-browser-run-headless)
  * [Come posso usare Brave per controllare il browser?](#how-do-i-use-brave-for-browser-control)
* [Gateway e nodi remoti](#remote-gateways-nodes)
  * [Come si propagano i comandi tra Telegram, il Gateway e i nodi?](#how-do-commands-propagate-between-telegram-the-gateway-and-nodes)
  * [Come può il mio agente accedere al mio computer se il Gateway è eseguito in remoto?](#how-can-my-agent-access-my-computer-if-the-gateway-is-hosted-remotely)
  * [Tailscale è connesso ma non ricevo risposte. E ora?](#tailscale-is-connected-but-i-get-no-replies-what-now)
  * [Due istanze OpenClaw possono comunicare tra loro (locale + VPS)?](#can-two-openclaw-instances-talk-to-each-other-local-vps)
  * [Ho bisogno di VPS separati per più agenti?](#do-i-need-separate-vpses-for-multiple-agents)
  * [C’è un vantaggio a usare un nodo sul mio portatile personale invece di fare SSH da un VPS?](#is-there-a-benefit-to-using-a-node-on-my-personal-laptop-instead-of-ssh-from-a-vps)
  * [I nodi eseguono un servizio Gateway?](#do-nodes-run-a-gateway-service)
  * [Esiste un modo via API / RPC per applicare la configurazione?](#is-there-an-api-rpc-way-to-apply-config)
  * [Qual è una configurazione minima “sensata” per una prima installazione?](#whats-a-minimal-sane-config-for-a-first-install)
  * [Come configuro Tailscale su un VPS e mi connetto dal mio Mac?](#how-do-i-set-up-tailscale-on-a-vps-and-connect-from-my-mac)
  * [Come collego un nodo Mac a un Gateway remoto (Tailscale Serve)?](#how-do-i-connect-a-mac-node-to-a-remote-gateway-tailscale-serve)
  * [Devo installare su un secondo portatile o aggiungere solo un nodo?](#should-i-install-on-a-second-laptop-or-just-add-a-node)
* [Variabili di ambiente e caricamento di .env](#env-vars-and-env-loading)
  * [Come carica OpenClaw le variabili d&#39;ambiente?](#how-does-openclaw-load-environment-variables)
  * [«Ho avviato il Gateway tramite il servizio e le mie variabili d&#39;ambiente sono scomparse.» E adesso?](#i-started-the-gateway-via-the-service-and-my-env-vars-disappeared-what-now)
  * [Ho impostato `COPILOT_GITHUB_TOKEN`, ma lo stato dei modelli mostra «Shell env: off». Perché?](#i-set-copilotgithubtoken-but-models-status-shows-shell-env-off-why)
* [Sessioni &amp; chat multiple](#sessions-multiple-chats)
  * [Come avvio una nuova conversazione da zero?](#how-do-i-start-a-fresh-conversation)
  * [Le sessioni si reimpostano automaticamente se non invio mai `/new`?](#do-sessions-reset-automatically-if-i-never-send-new)
  * [Esiste un modo per creare un team di istanze OpenClaw, con un CEO e molti agenti?](#is-there-a-way-to-make-a-team-of-openclaw-instances-one-ceo-and-many-agents)
  * [Perché il contesto è stato troncato a metà attività? Come posso evitarlo?](#why-did-context-get-truncated-midtask-how-do-i-prevent-it)
  * [Come posso reimpostare completamente OpenClaw ma mantenerlo installato?](#how-do-i-completely-reset-openclaw-but-keep-it-installed)
  * [Ricevo errori “context too large” - come posso reimpostare o compattare?](#im-getting-context-too-large-errors-how-do-i-reset-or-compact)
  * [Perché vedo “LLM request rejected: messages.N.content.X.tool&#95;use.input: Field required”?](#why-am-i-seeing-llm-request-rejected-messagesncontentxtooluseinput-field-required)
  * [Perché ricevo messaggi di heartbeat ogni 30 minuti?](#why-am-i-getting-heartbeat-messages-every-30-minutes)
  * [Devo aggiungere un “bot account” a un gruppo WhatsApp?](#do-i-need-to-add-a-bot-account-to-a-whatsapp-group)
  * [Come faccio a ottenere il JID di un gruppo WhatsApp?](#how-do-i-get-the-jid-of-a-whatsapp-group)
  * [Perché OpenClaw non risponde in un gruppo?](#why-doesnt-openclaw-reply-in-a-group)
  * [I gruppi/thread condividono il contesto con i messaggi diretti (DM)?](#do-groupsthreads-share-context-with-dms)
  * [Quanti spazi di lavoro e agenti posso creare?](#how-many-workspaces-and-agents-can-i-create)
  * [Posso eseguire più bot o chat contemporaneamente (Slack) e come devo configurarlo?](#can-i-run-multiple-bots-or-chats-at-the-same-time-slack-and-how-should-i-set-that-up)
* [Modelli: predefiniti, selezione, alias, passaggio](#models-defaults-selection-aliases-switching)
  * [Che cos&#39;è il “modello predefinito”?](#what-is-the-default-model)
  * [Quale modello consigliate?](#what-model-do-you-recommend)
  * [Come faccio a cambiare modello senza azzerare la mia configurazione?](#how-do-i-switch-models-without-wiping-my-config)
  * [Posso usare modelli self-hosted (llama.cpp, vLLM, Ollama)?](#can-i-use-selfhosted-models-llamacpp-vllm-ollama)
  * [Che cosa usano OpenClaw, Flawd e Krill per i modelli?](#what-do-openclaw-flawd-and-krill-use-for-models)
  * [Come faccio a cambiare modello al volo (senza riavviare)?](#how-do-i-switch-models-on-the-fly-without-restarting)
  * [Posso usare GPT 5.2 per le attività quotidiane e Codex 5.2 per programmare?](#can-i-use-gpt-52-for-daily-tasks-and-codex-52-for-coding)
  * [Perché vedo “Model … is not allowed” e poi nessuna risposta?](#why-do-i-see-model-is-not-allowed-and-then-no-reply)
  * [Perché vedo “Unknown model: minimax/MiniMax-M2.1”?](#why-do-i-see-unknown-model-minimaxminimaxm21)
  * [Posso usare MiniMax come predefinito e OpenAI per i compiti più complessi?](#can-i-use-minimax-as-my-default-and-openai-for-complex-tasks)
  * [opus / sonnet / gpt sono scorciatoie predefinite?](#are-opus-sonnet-gpt-builtin-shortcuts)
  * [Come faccio a definire/sovrascrivere scorciatoie di modello (alias)?](#how-do-i-defineoverride-model-shortcuts-aliases)
  * [Come faccio ad aggiungere modelli da altri provider come OpenRouter o Z.AI?](#how-do-i-add-models-from-other-providers-like-openrouter-or-zai)
* [Failover dei modelli e “All models failed”](#model-failover-and-all-models-failed)
  * [Come funziona il failover?](#how-does-failover-work)
  * [Cosa significa questo errore?](#what-does-this-error-mean)
  * [Checklist di risoluzione per `No credentials found for profile "anthropic:default"`](#fix-checklist-for-no-credentials-found-for-profile-anthropicdefault)
  * [Perché ha provato anche con Google Gemini ed è andato in errore?](#why-did-it-also-try-google-gemini-and-fail)
* [Profili di autenticazione: cosa sono e come gestirli](#auth-profiles-what-they-are-and-how-to-manage-them)
  * [Che cos’è un profilo di autenticazione?](#what-is-an-auth-profile)
  * [Quali sono gli ID di profilo tipici?](#what-are-typical-profile-ids)
  * [Posso controllare quale profilo di autenticazione viene utilizzato per primo?](#can-i-control-which-auth-profile-is-tried-first)
  * [OAuth vs API key: qual è la differenza?](#oauth-vs-api-key-whats-the-difference)
* [Gateway: porte, “già in esecuzione” e modalità remota](#gateway-ports-already-running-and-remote-mode)
  * [Quale porta usa il Gateway?](#what-port-does-the-gateway-use)
  * [Perché `openclaw gateway status` mostra `Runtime: running` ma `RPC probe: failed`?](#why-does-openclaw-gateway-status-say-runtime-running-but-rpc-probe-failed)
  * [Perché `openclaw gateway status` mostra `Config (cli)` e `Config (service)` diverse?](#why-does-openclaw-gateway-status-show-config-cli-and-config-service-different)
  * [Cosa significa “another Gateway instance is already listening”?](#what-does-another-gateway-instance-is-already-listening-mean)
  * [Come posso eseguire OpenClaw in modalità remota (il client si connette a un Gateway altrove)?](#how-do-i-run-openclaw-in-remote-mode-client-connects-to-a-gateway-elsewhere)
  * [La Control UI mostra “unauthorized” (o continua a riconnettersi). E adesso?](#the-control-ui-says-unauthorized-or-keeps-reconnecting-what-now)
  * [Ho impostato `gateway.bind: "tailnet"` ma non riesce a eseguire il bind / nulla è in ascolto](#i-set-gatewaybind-tailnet-but-it-cant-bind-nothing-listens)
  * [Posso eseguire più Gateway sullo stesso host?](#can-i-run-multiple-gateways-on-the-same-host)
  * [Cosa significano “invalid handshake” / codice 1008?](#what-does-invalid-handshake-code-1008-mean)
* [Logging e debug](#logging-and-debugging)
  * [Dove si trovano i log?](#where-are-logs)
  * [Come avvio, arresto o riavvio il servizio Gateway?](#how-do-i-startstoprestart-the-gateway-service)
  * [Ho chiuso il terminale su Windows: come posso riavviare OpenClaw?](#i-closed-my-terminal-on-windows-how-do-i-restart-openclaw)
  * [Il Gateway è attivo ma le risposte non arrivano mai. Cosa devo controllare?](#the-gateway-is-up-but-replies-never-arrive-what-should-i-check)
  * [&quot;Disconnected from gateway: no reason&quot; - e adesso?](#disconnected-from-gateway-no-reason-what-now)
  * [Telegram setMyCommands fallisce con errori di rete. Cosa devo controllare?](#telegram-setmycommands-fails-with-network-errors-what-should-i-check)
  * [La TUI non mostra alcun output. Cosa devo controllare?](#tui-shows-no-output-what-should-i-check)
  * [Come arresto completamente e poi avvio di nuovo il Gateway?](#how-do-i-completely-stop-then-start-the-gateway)
  * [ELI5: `openclaw gateway restart` vs `openclaw gateway`](#eli5-openclaw-gateway-restart-vs-openclaw-gateway)
  * [Qual è il modo più rapido per ottenere maggiori dettagli quando qualcosa fallisce?](#whats-the-fastest-way-to-get-more-details-when-something-fails)
* [Contenuti multimediali e allegati](#media-attachments)
  * [La mia skill ha generato un&#39;immagine o un PDF, ma non è stato inviato nulla](#my-skill-generated-an-imagepdf-but-nothing-was-sent)
* [Sicurezza e controllo accessi](#security-and-access-control)
  * [È sicuro esporre OpenClaw ai DM in arrivo?](#is-it-safe-to-expose-openclaw-to-inbound-dms)
  * [La prompt injection è un rischio solo per i bot pubblici?](#is-prompt-injection-only-a-concern-for-public-bots)
  * [Il mio bot dovrebbe avere un proprio account email, GitHub o numero di telefono?](#should-my-bot-have-its-own-email-github-account-or-phone-number)
  * [Posso dargli autonomia nella gestione dei miei messaggi di testo ed è sicuro?](#can-i-give-it-autonomy-over-my-text-messages-and-is-that-safe)
  * [Posso usare modelli più economici per attività di assistente personale?](#can-i-use-cheaper-models-for-personal-assistant-tasks)
  * [Ho eseguito `/start` in Telegram ma non ho ricevuto un codice di abbinamento](#i-ran-start-in-telegram-but-didnt-get-a-pairing-code)
  * [WhatsApp: invierà messaggi ai miei contatti? Come funziona l’abbinamento?](#whatsapp-will-it-message-my-contacts-how-does-pairing-work)
* [Comandi della chat, interruzione delle attività e «non si ferma»](#chat-commands-aborting-tasks-and-it-wont-stop)
  * [Come faccio a evitare che i messaggi di sistema interni vengano mostrati in chat](#how-do-i-stop-internal-system-messages-from-showing-in-chat)
  * [Come faccio a interrompere/annullare un’attività in corso?](#how-do-i-stopcancel-a-running-task)
  * [Come faccio a inviare un messaggio su Discord da Telegram? (“Cross-context messaging denied”)](#how-do-i-send-a-discord-message-from-telegram-crosscontext-messaging-denied)
  * [Perché sembra che il bot “ignori” i messaggi inviati a raffica?](#why-does-it-feel-like-the-bot-ignores-rapidfire-messages)

<div id="first-60-seconds-if-somethings-broken">
  ## Primi 60 secondi se qualcosa non funziona
</div>

1. **Stato rapido (prima verifica)**
   ```bash
   openclaw status
   ```
   Riepilogo locale veloce: sistema operativo + aggiornamento, raggiungibilità del gateway/servizio, agenti/sessioni, configurazione dei provider + problemi di runtime (quando il gateway è raggiungibile).

2. **Report da incollare (sicuro da condividere)**
   ```bash
   openclaw status --all
   ```
   Diagnosi in sola lettura con coda dei log (token oscurati).

3. **Stato del demone + porte**
   ```bash
   openclaw gateway status
   ```
   Mostra lo stato di esecuzione del supervisore e la raggiungibilità RPC, l’URL di destinazione della sonda (probe) e quale configurazione il servizio ha probabilmente usato.

4. **Probe approfondite**
   ```bash
   openclaw status --deep
   ```
   Esegue i controlli di integrità del gateway + sonde dei provider (richiede un gateway raggiungibile). Consulta [Integrità](/it/gateway/health).

5. **Segui il log più recente**
   ```bash
   openclaw logs --follow
   ```
   Se l’RPC non è disponibile, usa in alternativa:
   ```bash
   tail -f "$(ls -t /tmp/openclaw/openclaw-*.log | head -1)"
   ```
   I log su file sono separati dai log del servizio; consulta [Logging](/it/logging) e [Risoluzione dei problemi](/it/gateway/troubleshooting).

6. **Esegui il doctor (riparazioni)**
   ```bash
   openclaw doctor
   ```
   Ripara/migra configurazione/stato + esegue i controlli di integrità. Consulta [Doctor](/it/gateway/doctor).

7. **Snapshot del Gateway**
   ```bash
   openclaw health --json
   openclaw health --verbose   # mostra l’URL di destinazione + il percorso della config in caso di errori
   ```
   Richiede al gateway in esecuzione uno snapshot completo (solo tramite WS). Consulta [Integrità](/it/gateway/health).

<div id="quick-start-and-first-run-setup">
  ## Avvio rapido e configurazione iniziale
</div>

<div id="im-stuck-whats-the-fastest-way-to-get-unstuck">
  ### Sono bloccato qual è il modo più rapido per sbloccarmi
</div>

Usa un agente IA locale che possa **vedere la tua macchina**. È molto più efficace che chiedere
su Discord, perché la maggior parte dei casi di &quot;I&#39;m stuck&quot; sono **problemi locali di configurazione o di ambiente** che
chi ti aiuta da remoto non può ispezionare.

* **Claude Code**: https://www.anthropic.com/claude-code/
* **OpenAI Codex**: https://openai.com/codex/

Questi strumenti possono leggere la repo, eseguire comandi, ispezionare i log e aiutarti a correggere la configurazione
a livello di macchina (PATH, servizi, permessi, file di auth). Fornisci loro il **checkout completo dei sorgenti** tramite
l&#39;installazione hackable (git):

```bash
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --install-method git
```

Questo installa OpenClaw **da un checkout Git**, così l&#39;agente può read il codice e la documentazione
e ragionare sull&#39;esatta versione che stai eseguendo. Puoi sempre tornare in seguito alla versione stabile
rieseguendo il programma di installazione senza `--install-method git`.

Suggerimento: chiedi all&#39;agente di **pianificare e supervisionare** la correzione (passo dopo passo), quindi esegui solo i
comandi necessari. In questo modo le modifiche restano limitate e più facili da verificare.

Se scopri un vero bug o una correzione, apri una issue su GitHub o invia una PR:
https://github.com/openclaw/openclaw/issues
https://github.com/openclaw/openclaw/pulls

Inizia con questi comandi (condividi gli output quando chiedi aiuto):

```bash
openclaw status
openclaw models status
openclaw doctor
```

Cosa fanno:

* `openclaw status`: istantanea rapida dello stato di salute del Gateway e degli agenti + configurazione di base.
* `openclaw models status`: verifica l&#39;autenticazione dei provider + la disponibilità dei modelli.
* `openclaw doctor`: convalida e ripara problemi comuni di configurazione/stato.

Altri controlli CLI utili: `openclaw status --all`, `openclaw logs --follow`,
`openclaw gateway status`, `openclaw health --verbose`.

Ciclo di debug rapido: [Primi 60 secondi quando qualcosa non funziona](#first-60-seconds-if-somethings-broken).
Documentazione sull&#39;installazione: [Installazione](/it/install), [Flag dell&#39;installer](/it/install/installer), [Aggiornamento](/it/install/updating).

<div id="whats-the-recommended-way-to-install-and-set-up-openclaw">
  ### Qual è il modo consigliato per installare e configurare OpenClaw
</div>

Il repository consiglia di eseguire OpenClaw direttamente dal sorgente e usare la procedura guidata di onboarding:

```bash
curl -fsSL https://openclaw.bot/install.sh | bash
openclaw onboard --install-daemon
```

La procedura guidata può anche generare automaticamente le risorse della UI. Dopo l&#39;onboarding, in genere esegui il Gateway sulla porta **18789**.

Dal codice sorgente (contributor/dev):

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm build
pnpm ui:build # installa automaticamente le dipendenze della UI alla prima esecuzione
openclaw onboard
```

Se non hai ancora un&#39;installazione globale di OpenClaw, esegui `pnpm openclaw onboard`.

<div id="how-do-i-open-the-dashboard-after-onboarding">
  ### Come apro la dashboard dopo l’onboarding
</div>

Il wizard ora apre il browser con un URL tokenizzato della dashboard al termine dell’onboarding e stampa anche il link completo (con token) nel riepilogo. Mantieni aperta quella scheda; se non si è aperta, copia e incolla l’URL stampato sulla stessa macchina. I token restano locali al tuo host: dal browser non viene letto o recuperato nulla.

<div id="how-do-i-authenticate-the-dashboard-token-on-localhost-vs-remote">
  ### Come faccio ad autenticare il token della dashboard in localhost e da remoto
</div>

**Localhost (stessa macchina):**

* Apri `http://127.0.0.1:18789/`.
* Se richiede autenticazione, esegui `openclaw dashboard` e usa il link con token (`?token=...`).
* Il token è lo stesso valore di `gateway.auth.token` (o `OPENCLAW_GATEWAY_TOKEN`) ed è memorizzato dalla UI dopo il primo caricamento.

**Non su localhost:**

* **Tailscale Serve** (consigliato): mantieni il bind a loopback, esegui `openclaw gateway --tailscale serve`, apri `https://<magicdns>/`. Se `gateway.auth.allowTailscale` è `true`, le intestazioni di identità sono sufficienti per l&#39;autenticazione (nessun token).
* **Bind su Tailnet**: esegui `openclaw gateway --bind tailnet --token "<token>"`, apri `http://<tailscale-ip>:18789/`, incolla il token nelle impostazioni della dashboard.
* **Tunnel SSH**: `ssh -N -L 18789:127.0.0.1:18789 user@host` quindi apri `http://127.0.0.1:18789/?token=...` utilizzando `openclaw dashboard`.

Vedi [Dashboard](/it/web/dashboard) e [Web surfaces](/it/web) per le modalità di bind e i dettagli sull&#39;autenticazione.

<div id="what-runtime-do-i-need">
  ### Quale runtime è necessario
</div>

È richiesto Node **&gt;= 22**. `pnpm` è consigliato. Bun **non è consigliato** per il Gateway.

<div id="does-it-run-on-raspberry-pi">
  ### Funziona su Raspberry Pi
</div>

Sì. Il Gateway è leggero: la documentazione indica che **512MB-1GB di RAM**, **1 core** e circa **500MB**
di disco sono sufficienti per uso personale e segnala che un **Raspberry Pi 4 può eseguirlo** senza problemi.

Se vuoi un po’ di margine in più (log, contenuti multimediali, altri servizi), **2GB sono consigliati**, ma non è
un requisito minimo tassativo.

Suggerimento: un piccolo Pi/VPS può ospitare il Gateway e puoi associare **nodi** al tuo laptop/telefono per
schermo, fotocamera, canvas locali o esecuzione di comandi. Vedi [Nodes](/it/nodes).

<div id="any-tips-for-raspberry-pi-installs">
  ### Consigli per l&#39;installazione su Raspberry Pi
</div>

Versione breve: funziona, ma aspettati qualche asperità.

* Usa un sistema operativo **a 64 bit** e mantieni Node alla versione &gt;= 22.
* Preferisci l&#39;**installazione &quot;hackable&quot; (git)** così puoi vedere i log e aggiornare velocemente.
* Inizia senza canali/abilità, poi aggiungili uno per uno.
* Se incontri strani problemi con i binari, di solito è un problema di **compatibilità ARM**.

Documentazione: [Linux](/it/platforms/linux), [Installazione](/it/install).

<div id="it-is-stuck-on-wake-up-my-friend-onboarding-will-not-hatch-what-now">
  ### È bloccato su «Wake up, my friend!»: l&#39;onboarding non parte. E adesso?
</div>

Quella schermata richiede che il Gateway sia raggiungibile e autenticato. La TUI inoltre invia
&quot;Wake up, my friend!&quot; automaticamente al primo hatch. Se vedi quella riga **senza alcuna risposta**
e i token rimangono a 0, l&#39;agente non è mai partito.

1. Riavvia il Gateway:

```bash
openclaw gateway restart
```

2. Verifica stato e autenticazione:

```bash
openclaw status
openclaw models status
openclaw logs --follow
```

3. Se continua a bloccarsi, esegui:

```bash
openclaw doctor
```

Se il Gateway è remoto, assicurati che il tunnel o la connessione Tailscale sia attiva e che la UI
punti al Gateway corretto. Vedi [Accesso remoto](/it/gateway/remote).

<div id="can-i-migrate-my-setup-to-a-new-machine-mac-mini-without-redoing-onboarding">
  ### Posso migrare la mia configurazione su un nuovo Mac mini senza rifare l’onboarding
</div>

Sì. Copia la **directory di stato** e lo **spazio di lavoro**, poi esegui Doctor una volta. In questo
modo il tuo bot resta “esattamente lo stesso” (memoria, cronologia delle sessioni, auth e stato dei canali) finché copi **entrambi** i percorsi:

1. Installa OpenClaw sulla nuova macchina.
2. Copia `$OPENCLAW_STATE_DIR` (predefinito: `~/.openclaw`) dalla vecchia macchina.
3. Copia il tuo spazio di lavoro (predefinito: `~/.openclaw/workspace`).
4. Esegui `openclaw doctor` e riavvia il servizio Gateway.

Questo preserva la configurazione, i profili di auth, le credenziali WhatsApp, le sessioni e la memoria. Se sei in
remote mode, ricorda che l’host del Gateway gestisce l’archivio delle sessioni e lo spazio di lavoro.

**Importante:** se effettui solo commit/push del tuo spazio di lavoro su GitHub, stai eseguendo il backup di **memoria + file di bootstrap**, ma **non** della cronologia delle sessioni o dell’auth. Questi si trovano
sotto `~/.openclaw/` (per esempio `~/.openclaw/agents/<agentId>/sessions/`).

Vedi anche: [Migrazione](/it/install/migrating), [Dove risiedono i dati su disco](/it/help/faq#where-does-openclaw-store-its-data),
[Spazio di lavoro dell’Agente](/it/concepts/agent-workspace), [Doctor](/it/gateway/doctor),
[Remote mode](/it/gateway/remote).

<div id="where-do-i-see-whats-new-in-the-latest-version">
  ### Dove posso vedere le novità dell&#39;ultima versione
</div>

Controlla il changelog su GitHub:\
https://github.com/openclaw/openclaw/blob/main/CHANGELOG.md

Le voci più recenti sono in alto. Se la prima sezione è contrassegnata come **Unreleased**, la successiva sezione datata è l&#39;ultima versione rilasciata. Le voci sono raggruppate in **Highlights**, **Changes** e **Fixes** (più eventuali sezioni per la documentazione/altro quando necessario).

<div id="i-cant-access-docsopenclawai-ssl-error-what-now">
  ### Non riesco ad accedere a docs.openclaw.ai, errore SSL. E adesso?
</div>

Alcune connessioni Comcast/Xfinity bloccano erroneamente `docs.openclaw.ai` tramite la funzione
Xfinity Advanced Security. Disabilitala oppure aggiungi `docs.openclaw.ai` alla lista di autorizzati,
quindi riprova. Maggiori dettagli: [Risoluzione dei problemi](/it/help/troubleshooting#docsopenclawai-shows-an-ssl-error-comcastxfinity).
Aiutaci a sbloccarlo segnalandolo qui: https://spa.xfinity.com/check&#95;url&#95;status.

Se ancora non riesci a raggiungere il sito, la documentazione è disponibile anche su GitHub:
https://github.com/openclaw/openclaw/tree/main/docs

<div id="whats-the-difference-between-stable-and-beta">
  ### Qual è la differenza tra stable e beta?
</div>

**Stable** e **beta** sono **dist‑tag di npm**, non rami di codice separati:

* `latest` = stable
* `beta` = build preliminare per i test

Rilasciamo le build su **beta**, le testiamo e, quando una build è stabile, **promuoviamo
quella stessa versione a `latest`**. Per questo beta e stable possono puntare alla
**stessa versione**.

Guarda cosa è cambiato:\
https://github.com/openclaw/openclaw/blob/main/CHANGELOG.md

<div id="how-do-i-install-the-beta-version-and-whats-the-difference-between-beta-and-dev">
  ### Come installo la versione beta e qual è la differenza tra beta e dev
</div>

**Beta** è il dist‑tag npm `beta` (può coincidere con `latest`).
**Dev** è la testa corrente di `main` (git); quando viene pubblicata, usa il dist‑tag npm `dev`.

One‑liner (macOS/Linux):

```bash
curl -fsSL --proto '=https' --tlsv1.2 https://openclaw.bot/install.sh | bash -s -- --beta
```

```bash
curl -fsSL --proto '=https' --tlsv1.2 https://openclaw.bot/install.sh | bash -s -- --install-method git
```

Programma di installazione per Windows (PowerShell):
https://openclaw.ai/install.ps1

Ulteriori dettagli: [Canali di sviluppo](/it/install/development-channels) e [flag dell&#39;installer](/it/install/installer).

<div id="how-long-does-install-and-onboarding-usually-take">
  ### Quanto tempo richiedono di solito installazione e onboarding
</div>

Indicazioni di massima:

* **Installazione:** 2-5 minuti
* **Onboarding:** 5-15 minuti a seconda di quanti canali/modelli configuri

Se si blocca, usa [Installer stuck](/it/help/faq#installer-stuck-how-do-i-get-more-feedback)
e il ciclo di debug rapido in [Im stuck](/it/help/faq#im-stuck--whats-the-fastest-way-to-get-unstuck).

<div id="how-do-i-try-the-latest-bits">
  ### Come posso provare le build più recenti?
</div>

Due opzioni:

1. **Canale di sviluppo (git checkout):**

```bash
openclaw update --channel dev
```

Questo comando passa al branch `main` e aggiorna dal repository sorgente.

2. **Installazione *hackable* (dal sito dell&#39;installer):**

```bash
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --install-method git
```

Questo ti fornisce un repository locale che puoi modificare e poi aggiornare tramite git.

Se preferisci eseguire manualmente un clone pulito, usa:

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm build
```

Documentazione: [Aggiornamento](/it/cli/update), [Canali di sviluppo](/it/install/development-channels),
[Installazione](/it/install).

<div id="installer-stuck-how-do-i-get-more-feedback">
  ### Il programma di installazione si blocca: come posso ottenere più informazioni?
</div>

Esegui di nuovo il programma di installazione con **output dettagliato (verbose)**:

```bash
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --verbose
```

Installazione beta in modalità verbosa:

```bash
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --beta --verbose
```

Per un&#39;installazione facilmente modificabile (git):

```bash
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --install-method git --verbose
```

Altre opzioni: [flag dell&#39;installer](/it/install/installer).

<div id="windows-install-says-git-not-found-or-openclaw-not-recognized">
  ### L&#39;installazione su Windows indica che git non è stato trovato o che openclaw non è riconosciuto
</div>

Due problemi comuni su Windows:

**1) errore npm spawn git / git not found**

* Installa **Git for Windows** e assicurati che `git` sia nel tuo PATH.
* Chiudi e riapri PowerShell, quindi riesegui l&#39;installer.

**2) openclaw non è riconosciuto dopo l&#39;installazione**

* La cartella bin globale di npm non è nel PATH.
* Controlla il percorso:
  ```powershell
  npm config get prefix
  ```
* Assicurati che `<prefix>\\bin` sia nel PATH (sulla maggior parte dei sistemi è `%AppData%\\npm`).
* Chiudi e riapri PowerShell dopo aver aggiornato il PATH.

Se vuoi la configurazione più semplice su Windows, usa **WSL2** invece di Windows nativo.
Documentazione: [Windows](/it/platforms/windows).

<div id="the-docs-didnt-answer-my-question-how-do-i-get-a-better-answer">
  ### La documentazione non ha risposto alla mia domanda. Come posso ottenere una risposta migliore?
</div>

Usa l&#39;**installazione hackable (git)** così da avere il codice sorgente completo e la documentazione in locale, quindi poni la tua domanda
al tuo bot (o Claude/Codex) *da quella cartella* in modo che possa leggere il repository e rispondere in modo accurato.

```bash
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --install-method git
```

Ulteriori dettagli: [Installazione](/it/install) e [Flag dell&#39;installer](/it/install/installer).

<div id="how-do-i-install-openclaw-on-linux">
  ### Come posso installare OpenClaw su Linux
</div>

Risposta breve: segui la guida per Linux, quindi esegui la procedura guidata di onboarding.

* Percorso rapido per Linux + installazione come servizio: [Linux](/it/platforms/linux).
* Procedura completa: [Guida introduttiva](/it/start/getting-started).
* Programma di installazione + aggiornamenti: [Installazione e aggiornamenti](/it/install/updating).

<div id="how-do-i-install-openclaw-on-a-vps">
  ### Come installare OpenClaw su una VPS
</div>

Qualsiasi VPS Linux funziona. Installalo sul server, quindi usa SSH o Tailscale per raggiungere il Gateway.

Guide: [exe.dev](/it/platforms/exe-dev), [Hetzner](/it/platforms/hetzner), [Fly.io](/it/platforms/fly).\
Accesso remoto: [Gateway remoto](/it/gateway/remote).

<div id="where-are-the-cloudvps-install-guides">
  ### Dove si trovano le guide di installazione per cloud/VPS
</div>

Gestiamo un **hub di hosting** con i provider più comuni. Scegline uno e segui la guida:

* [Hosting VPS](/it/vps) (tutti i provider in un unico posto)
* [Fly.io](/it/platforms/fly)
* [Hetzner](/it/platforms/hetzner)
* [exe.dev](/it/platforms/exe-dev)

Come funziona nel cloud: il **Gateway gira sul server** e vi accedi
dal tuo laptop/telefono tramite la Control UI (o Tailscale/SSH). Il tuo stato e lo spazio di lavoro
risiedono sul server, quindi considera l&#39;host come la tua fonte di verità ed esegui regolarmente dei backup.

Puoi associare **nodi** (Mac/iOS/Android/headless) a quel Gateway nel cloud per accedere
a schermo/fotocamera/canvas locali o eseguire comandi sul tuo laptop mantenendo il
Gateway nel cloud.

Hub: [Platforms](/it/platforms). Accesso remoto: [Gateway remoto](/it/gateway/remote).
Nodes: [Nodes](/it/nodes), [Nodes CLI](/it/cli/nodes).

<div id="can-i-ask-openclaw-to-update-itself">
  ### Posso chiedere a OpenClaw di aggiornarsi da solo
</div>

Risposta breve: **possibile, non consigliato**. La procedura di aggiornamento può riavviare il
Gateway (che interrompe la sessione attiva), potrebbe richiedere un `git checkout` pulito e
può chiedere una conferma. Più sicuro: esegui gli aggiornamenti da una shell come operatore.

Usa la CLI:

```bash
openclaw update
openclaw update status
openclaw update --channel stable|beta|dev
openclaw update --tag <dist-tag|version>
openclaw update --no-restart
```

Se devi automatizzare tramite un agente:

```bash
openclaw update --yes --no-restart
openclaw gateway restart
```

Documentazione: [Update](/it/cli/update), [Aggiornamento](/it/install/updating).

<div id="what-does-the-onboarding-wizard-actually-do">
  ### Cosa fa effettivamente la procedura guidata di onboarding
</div>

`openclaw onboard` è il percorso di configurazione consigliato. In **modalità locale** ti guida passo passo attraverso:

* **Configurazione modello/autenticazione** (Anthropic **setup-token** consigliato per gli abbonamenti a Claude, supporta OAuth OpenAI Codex, chiavi API facoltative, supporta i modelli locali di LM Studio)
* Posizione dello **spazio di lavoro** + file di bootstrap
* **Impostazioni Gateway** (bind/port/auth/tailscale)
* **Provider** (WhatsApp, Telegram, Discord, Mattermost (plugin), Signal, iMessage)
* **Installazione del demone** (LaunchAgent su macOS; unità utente systemd su Linux/WSL2)
* **Controlli di integrità** e selezione delle **abilità**

Inoltre ti avvisa se il modello configurato è sconosciuto o privo di autenticazione.

<div id="do-i-need-a-claude-or-openai-subscription-to-run-this">
  ### Ho bisogno di un abbonamento Claude o OpenAI per usare OpenClaw
</div>

No. Puoi eseguire OpenClaw con **chiavi API** (Anthropic/OpenAI/altri) oppure con
**modelli solo locali**, in modo che i tuoi dati rimangano sul tuo dispositivo. Gli abbonamenti (Claude
Pro/Max o OpenAI Codex) sono un modo opzionale per autenticare questi provider.

Documentazione: [Anthropic](/it/providers/anthropic), [OpenAI](/it/providers/openai),
[Modelli locali](/it/gateway/local-models), [Modelli](/it/concepts/models).

<div id="can-i-use-claude-max-subscription-without-an-api-key">
  ### Posso usare l&#39;abbonamento Claude Max senza una chiave api
</div>

Sì. Puoi autenticarti con un **setup-token**
invece di una chiave api. Questo è il flusso previsto per gli abbonamenti.

Gli abbonamenti Claude Pro/Max **non includono una chiave api**, quindi questo è l&#39;approccio
corretto per gli account in abbonamento. Importante: devi verificare con
Anthropic che questo utilizzo sia consentito ai sensi della loro policy e dei relativi termini di abbonamento.
Se vuoi l&#39;opzione più esplicita e ufficialmente supportata, usa una chiave api Anthropic.

<div id="how-does-anthropic-setuptoken-auth-work">
  ### Come funziona l&#39;autenticazione con il setuptoken di Anthropic
</div>

`claude setup-token` genera una **stringa di token di autenticazione** tramite la Claude Code CLI (non è disponibile nella console web). Puoi eseguirlo su **qualsiasi macchina**. Scegli **Anthropic token (paste setup-token)** nella procedura guidata oppure incollalo con `openclaw models auth paste-token --provider anthropic`. Il token viene memorizzato come profilo di autenticazione per il provider **anthropic** e usato come una chiave API (non viene rinnovato automaticamente). Maggiori dettagli: [OAuth](/it/concepts/oauth).

<div id="where-do-i-find-an-anthropic-setuptoken">
  ### Dove trovo il setup-token Anthropic
</div>

**Non** si trova nella console Anthropic. Il setup-token viene generato dalla **Claude Code CLI** su **qualsiasi macchina**:

```bash
claude setup-token
```

Copia il token che viene visualizzato, quindi scegli **Anthropic token (paste setup-token)** nella procedura guidata. Se vuoi eseguirlo sull&#39;host del Gateway, usa `openclaw models auth setup-token --provider anthropic`. Se hai eseguito `claude setup-token` altrove, incollalo sull&#39;host del Gateway con `openclaw models auth paste-token --provider anthropic`. Vedi [Anthropic](/it/providers/anthropic).

<div id="do-you-support-claude-subscription-auth-claude-promax">
  ### Supportate l&#39;autenticazione tramite abbonamento Claude (Claude Pro/Max)
</div>

Sì — tramite **setup-token**. OpenClaw non riutilizza più i token OAuth della Claude Code CLI; usa un setup-token o una chiave API di Anthropic. Puoi generare il token ovunque e incollarlo sull&#39;host del Gateway. Vedi [Anthropic](/it/providers/anthropic) e [OAuth](/it/concepts/oauth).

Nota: l&#39;accesso tramite abbonamento Claude è regolato dai termini di Anthropic. Per carichi di lavoro in produzione o multi‑utente, le chiavi API sono di solito l&#39;opzione più sicura.

<div id="why-am-i-seeing-http-429-ratelimiterror-from-anthropic">
  ### Perché vedo l&#39;errore HTTP 429 ratelimiterror da Anthropic
</div>

Questo significa che la tua **quota/limite di frequenza Anthropic (rate limit)** è esaurita per la finestra corrente. Se
usi un **abbonamento Claude** (setup‑token o Claude Code OAuth), attendi che la finestra si
azzerri oppure effettua l&#39;upgrade del piano. Se usi una **chiave API Anthropic**, controlla la Console Anthropic
per utilizzo/fatturazione e aumenta i limiti se necessario.

Suggerimento: imposta un **modello di fallback** così OpenClaw può continuare a rispondere mentre un provider è soggetto a rate limit.
Vedi [Models](/it/cli/models) e [OAuth](/it/concepts/oauth).

<div id="is-aws-bedrock-supported">
  ### AWS Bedrock è supportato
</div>

Sì - tramite il provider **Amazon Bedrock (Converse)** di pi‑ai con **configurazione manuale**. Devi fornire le credenziali/regione AWS sull&#39;host del Gateway e aggiungere una voce di provider Bedrock nella configurazione dei tuoi modelli. Vedi [Amazon Bedrock](/it/bedrock) e [Model providers](/it/providers/models). Se preferisci un flusso con chiave gestita, un proxy compatibile con OpenAI davanti a Bedrock è comunque un&#39;opzione valida.

<div id="how-does-codex-auth-work">
  ### Come funziona l&#39;autenticazione con Codex
</div>

OpenClaw supporta **OpenAI Code (Codex)** tramite OAuth (accesso tramite ChatGPT). Il wizard può eseguire il flusso OAuth e imposterà il modello predefinito su `openai-codex/gpt-5.2` quando appropriato. Consulta [Provider di modelli](/it/concepts/model-providers) e [Wizard](/it/start/wizard).

<div id="do-you-support-openai-subscription-auth-codex-oauth">
  ### Supportate l&#39;autenticazione OAuth per gli abbonamenti OpenAI Code (Codex)?
</div>

Sì. OpenClaw supporta pienamente **OpenAI Code (Codex) subscription OAuth**. La procedura guidata di onboarding
può eseguire il flusso OAuth per tuo conto.

Vedi [OAuth](/it/concepts/oauth), [provider di modelli](/it/concepts/model-providers) e [Procedura guidata](/it/start/wizard).

<div id="how-do-i-set-up-gemini-cli-oauth">
  ### Come configuro l&#39;OAuth di Gemini CLI
</div>

Gemini CLI utilizza un **flusso di autenticazione tramite plugin**, non un client ID o client secret in `openclaw.json`.

Passaggi:

1. Abilita il plugin: `openclaw plugins enable google-gemini-cli-auth`
2. Effettua il login: `openclaw models auth login --provider google-gemini-cli --set-default`

Questo memorizza i token OAuth nei profili di autenticazione sull&#39;host del Gateway. Dettagli: [Provider di modelli](/it/concepts/model-providers).

<div id="is-a-local-model-ok-for-casual-chats">
  ### Un modello locale va bene per chiacchierate informali
</div>

Di solito no. OpenClaw richiede un contesto ampio + un livello di sicurezza elevato; contesti piccoli troncano e possono far trapelare dati. Se proprio devi, esegui in locale la build **più grande** possibile di MiniMax M2.1 (LM Studio) e consulta [/gateway/local-models](/it/gateway/local-models). Modelli più piccoli/quantizzati aumentano il rischio di attacchi di prompt injection – vedi [Sicurezza](/it/gateway/security).

<div id="how-do-i-keep-hosted-model-traffic-in-a-specific-region">
  ### Come faccio a mantenere il traffico dei modelli ospitati in una regione specifica
</div>

Scegli endpoint vincolati a una regione specifica. OpenRouter espone opzioni ospitate negli Stati Uniti per MiniMax, Kimi e GLM; scegli la variante ospitata negli Stati Uniti per mantenere i dati all’interno di quella regione. Puoi comunque affiancare Anthropic/OpenAI a questi usando `models.mode: "merge"` in modo che i fallback restino disponibili, rispettando al contempo il provider vincolato alla regione che selezioni.

<div id="do-i-have-to-buy-a-mac-mini-to-install-this">
  ### Devo comprare un Mac mini per installarlo
</div>

No. OpenClaw viene eseguito su macOS o Linux (Windows tramite WSL2). Un Mac mini è facoltativo: alcune persone
ne comprano uno come host sempre attivo, ma vanno bene anche un piccolo VPS, un server domestico
o un box tipo Raspberry Pi.

Hai bisogno di un Mac solo **per gli strumenti disponibili esclusivamente su macOS**. Per iMessage puoi tenere il Gateway su Linux
ed eseguire `imsg` su qualsiasi Mac via SSH puntando `channels.imessage.cliPath` a un wrapper SSH.
Se ti servono altri strumenti disponibili solo su macOS, esegui il Gateway su un Mac oppure associa un nodo macOS.

Documentazione: [iMessage](/it/channels/imessage), [Nodi](/it/nodes), [Mac remote mode](/it/platforms/mac/remote).

<div id="do-i-need-a-mac-mini-for-imessage-support">
  ### Ho bisogno di un Mac mini per il supporto iMessage
</div>

Ti serve **un dispositivo macOS** con accesso a Messaggi. **Non** deve per forza essere un Mac mini:
qualsiasi Mac va bene. Le integrazioni iMessage di OpenClaw vengono eseguite su macOS (BlueBubbles o `imsg`), mentre
il Gateway può essere eseguito altrove.

Configurazioni comuni:

* Esegui il Gateway su Linux/VPS e punta `channels.imessage.cliPath` a un wrapper SSH che
  esegue `imsg` sul Mac.
* Esegui tutto sul Mac se vuoi la configurazione più semplice su una singola macchina.

Documentazione: [iMessage](/it/channels/imessage), [BlueBubbles](/it/channels/bluebubbles),
[Modalità remota Mac](/it/platforms/mac/remote).

<div id="if-i-buy-a-mac-mini-to-run-openclaw-can-i-connect-it-to-my-macbook-pro">
  ### Se compro un Mac mini per eseguire OpenClaw posso collegarlo al mio MacBook Pro
</div>

Sì. Il **Mac mini può eseguire il Gateway** e il tuo MacBook Pro può collegarsi come
**nodo** (dispositivo companion). I nodi non eseguono il Gateway: forniscono
funzionalità aggiuntive come schermo/fotocamera/canvas e `system.run` su quel dispositivo.

Schema tipico:

* Gateway sul Mac mini (sempre attivo).
* Il MacBook Pro esegue l&#39;app macOS o un host di nodo e si associa al Gateway.
* Usa `openclaw nodes status` / `openclaw nodes list` per visualizzarlo.

Documentazione: [Nodes](/it/nodes), [Nodes CLI](/it/cli/nodes).

<div id="can-i-use-bun">
  ### Posso usare Bun
</div>

Bun **non è raccomandato**. Riscontriamo bug di runtime, soprattutto con WhatsApp e Telegram.
Usa **Node** per gateway stabili.

Se vuoi comunque fare esperimenti con Bun, fallo su un gateway non in produzione
senza WhatsApp/Telegram.

<div id="telegram-what-goes-in-allowfrom">
  ### Telegram cosa inserire in allowFrom
</div>

`channels.telegram.allowFrom` è **l’ID utente Telegram della persona che invia il messaggio** (numerico, consigliato) oppure il suo `@username`. Non è lo username del bot.

Più sicuro (nessun bot di terze parti):

* Invia un DM al tuo bot, poi esegui `openclaw logs --follow` e leggi `from.id`.

Bot API ufficiale:

* Invia un DM al tuo bot, poi chiama `https://api.telegram.org/bot<bot_token>/getUpdates` e leggi `message.from.id`.

Soluzione di terze parti (meno privata):

* Invia un DM a `@userinfobot` o `@getidsbot`.

Vedi [/channels/telegram](/it/channels/telegram#access-control-dms--groups).

<div id="can-multiple-people-use-one-whatsapp-number-with-different-openclaw-instances">
  ### Più persone possono usare lo stesso numero di WhatsApp con istanze OpenClaw diverse
</div>

Sì, tramite **instradamento multi‑agente**. Associa il **DM** WhatsApp di ogni mittente (peer `kind: "dm"`, mittente in formato E.164 come `+15551234567`) a un `agentId` diverso, così ogni persona avrà il proprio spazio di lavoro e il proprio archivio delle sessioni. Le risposte provengono comunque dallo **stesso account WhatsApp** e il controllo di accesso ai DM (`channels.whatsapp.dmPolicy` / `channels.whatsapp.allowFrom`) è globale per singolo account WhatsApp. Vedi [Instradamento multi‑agente](/it/concepts/multi-agent) e [WhatsApp](/it/channels/whatsapp).

<div id="can-i-run-a-fast-chat-agent-and-an-opus-for-coding-agent">
  ### Posso eseguire un agente di chat veloce e un agente Opus per il coding
</div>

Sì. Usa l’instradamento multi‑agente: assegna a ogni agente il proprio modello predefinito, quindi collega gli instradamenti in ingresso (account del provider o peer specifici) a ciascun agente. Un esempio di configurazione è disponibile in [Instradamento multi‑agente](/it/concepts/multi-agent). Consulta anche [Modelli](/it/concepts/models) e [Configurazione](/it/gateway/configuration).

<div id="does-homebrew-work-on-linux">
  ### Homebrew è compatibile con Linux
</div>

Sì. Homebrew supporta Linux (Linuxbrew). Configurazione rapida:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
echo 'eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"' >> ~/.profile
eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"
brew install <formula>  # sostituisci <formula> con il nome del pacchetto desiderato
```

Se esegui OpenClaw tramite systemd, assicurati che il PATH del servizio includa `/home/linuxbrew/.linuxbrew/bin` (o il tuo prefisso brew) in modo che gli strumenti installati con `brew` siano disponibili nelle shell non di login.
Le build recenti inoltre prepongono al PATH le comuni directory bin dell&#39;utente nei servizi systemd su Linux (ad esempio `~/.local/bin`, `~/.npm-global/bin`, `~/.local/share/pnpm`, `~/.bun/bin`) e rispettano `PNPM_HOME`, `NPM_CONFIG_PREFIX`, `BUN_INSTALL`, `VOLTA_HOME`, `ASDF_DATA_DIR`, `NVM_DIR` e `FNM_DIR` quando impostate.

<div id="whats-the-difference-between-the-hackable-git-install-and-npm-install">
  ### Qual è la differenza tra l&#39;installazione hackable via git e l&#39;installazione via npm
</div>

* **Installazione hackable (git):** checkout completo del sorgente, modificabile, ideale per chi contribuisce.
  Esegui le build in locale e puoi applicare patch a codice e/o documentazione.
* **Installazione npm:** installazione globale della CLI, nessun repository locale, ideale per chi vuole solo eseguirlo.
  Gli aggiornamenti arrivano tramite dist‑tag di npm.

Documentazione: [Per iniziare](/it/start/getting-started), [Aggiornamenti](/it/install/updating).

<div id="can-i-switch-between-npm-and-git-installs-later">
  ### Posso passare da un&#39;installazione npm a una git in seguito
</div>

Sì. Installa l&#39;altra variante, poi esegui Doctor in modo che il servizio Gateway punti al nuovo entrypoint.
Questo **non elimina i tuoi dati**: cambia solo l&#39;installazione del codice di OpenClaw. Il tuo stato
(`~/.openclaw`) e lo spazio di lavoro (`~/.openclaw/workspace`) rimangono intatti.

Da npm → git:

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm build
openclaw doctor
openclaw gateway restart
```

Da Git → npm:

```bash
npm install -g openclaw@latest
openclaw doctor
openclaw gateway restart
```

Doctor rileva una mancata corrispondenza dell&#39;entrypoint del servizio Gateway e propone di riscrivere la configurazione del servizio per allinearla all&#39;installazione corrente (usa `--repair` in automazione).

Suggerimenti per il backup: vedi [Strategia di backup](/it/help/faq#whats-the-recommended-backup-strategy).

<div id="should-i-run-the-gateway-on-my-laptop-or-a-vps">
  ### Dovrei eseguire il Gateway sul mio laptop o su un VPS
</div>

Risposta breve: **se vuoi affidabilità 24/7, usa un VPS**. Se vuoi il minimo attrito e ti sta bene che il portatile vada in sospensione/si riavvii, eseguilo in locale.

**Laptop (Gateway locale)**

* **Pro:** nessun costo server, accesso diretto ai file locali, finestra del browser sempre visibile.
* **Contro:** sospensione/cali di rete = disconnessioni, aggiornamenti/riavvii dell’OS interrompono l’esecuzione, il portatile deve restare acceso.

**VPS / cloud**

* **Pro:** sempre attivo, rete stabile, nessun problema di sospensione del laptop, più facile da mantenere in esecuzione.
* **Contro:** spesso gira senza interfaccia grafica (usa screenshot), accesso ai file solo in remoto, devi usare SSH per gli aggiornamenti.

**Nota specifica per OpenClaw:** WhatsApp/Telegram/Slack/Mattermost (plugin)/Discord funzionano tutti senza problemi da un VPS. Il vero compromesso è solo tra **browser headless** e finestra visibile. Vedi [Browser](/it/tools/browser).

**Impostazione predefinita consigliata:** VPS se in passato hai avuto disconnessioni del Gateway. L’esecuzione in locale è ottima quando stai usando attivamente il Mac e vuoi accesso ai file locali o automazione della UI con un browser visibile.

<div id="how-important-is-it-to-run-openclaw-on-a-dedicated-machine">
  ### Quanto è importante eseguire OpenClaw su una macchina dedicata
</div>

Non è obbligatorio, ma **consigliato per affidabilità e isolamento**.

* **Host dedicato (VPS/Mac mini/Pi):** sempre attivo, meno interruzioni per sospensione/riavvio, gestione dei permessi più semplice, più facile da mantenere in esecuzione.
* **Laptop/desktop condiviso:** va benissimo per test e uso attivo, ma aspettati pause quando la macchina va in sospensione o installa aggiornamenti.

Se vuoi il meglio di entrambi i mondi, mantieni il Gateway su un host dedicato e associa il tuo laptop come **nodo** per strumenti locali per schermo/fotocamera/exec. Vedi [Nodes](/it/nodes).
Per indicazioni sulla sicurezza, leggi [Security](/it/gateway/security).

<div id="what-are-the-minimum-vps-requirements-and-recommended-os">
  ### Quali sono i requisiti minimi per un VPS e quale sistema operativo è consigliato
</div>

OpenClaw è leggero. Per un Gateway di base + un canale chat:

* **Minimo assoluto:** 1 vCPU, 1 GB di RAM, ~500 MB di disco.
* **Consigliato:** 1-2 vCPU, 2 GB di RAM o più per avere margine (log, contenuti multimediali, più canali). Gli strumenti dei nodi e l&#39;automazione del browser possono essere molto esigenti in termini di risorse.

Sistema operativo: usa **Ubuntu LTS** (o qualsiasi distribuzione Debian/Ubuntu moderna). Il percorso di installazione Linux è stato testato soprattutto lì.

Documentazione: [Linux](/it/platforms/linux), [VPS hosting](/it/vps).

<div id="can-i-run-openclaw-in-a-vm-and-what-are-the-requirements">
  ### Posso eseguire OpenClaw in una VM e quali sono i requisiti
</div>

Sì. Tratta una VM come una VPS: deve essere sempre accesa, raggiungibile e avere
RAM sufficiente per il Gateway e per tutti i canali che abiliti.

Indicazioni di base:

* **Minimo indispensabile:** 1 vCPU, 1GB di RAM.
* **Consigliato:** 2GB di RAM o più se esegui più canali, automazione del browser o strumenti multimediali.
* **OS:** Ubuntu LTS o un&#39;altra distribuzione Debian/Ubuntu moderna.

Se usi Windows, **WSL2 è la configurazione basata su VM più semplice** e offre la migliore
compatibilità con gli strumenti. Vedi [Windows](/it/platforms/windows), [VPS hosting](/it/vps).
Se stai eseguendo macOS in una VM, vedi [macOS VM](/it/platforms/macos-vm).

<div id="what-is-openclaw">
  ## Che cos&#39;è OpenClaw?
</div>

<div id="what-is-openclaw-in-one-paragraph">
  ### Che cos&#39;è OpenClaw in un paragrafo
</div>

OpenClaw è un assistente AI personale che esegui direttamente sui tuoi dispositivi. Risponde nei canali di messaggistica che già utilizzi (WhatsApp, Telegram, Slack, Mattermost (plugin), Discord, Google Chat, Signal, iMessage, WebChat) e può anche gestire interazioni vocali e un Canvas in tempo reale sulle piattaforme supportate. Il **Gateway** è il piano di controllo sempre attivo; l&#39;assistente è il prodotto.

<div id="whats-the-value-proposition">
  ### Qual è la proposta di valore
</div>

OpenClaw non è “solo un wrapper per Claude”. È un **control plane local-first** che ti permette di eseguire un
assistente avanzato sul **tuo hardware**, raggiungibile dalle app di chat che già usi, con
sessioni con stato, memoria e strumenti, **senza** cedere il controllo dei tuoi workflow a un
SaaS ospitato da terzi.

Punti chiave:

* **I tuoi dispositivi, i tuoi dati:** esegui il Gateway dove vuoi (Mac, Linux, VPS) e mantieni
  lo spazio di lavoro + la cronologia delle sessioni in locale.
* **Canali reali, non una sandbox web:** WhatsApp/Telegram/Slack/Discord/Signal/iMessage/etc,
  più voce su dispositivi mobili e Canvas sulle piattaforme supportate.
* **Indipendente dal modello:** usa Anthropic, OpenAI, MiniMax, OpenRouter, ecc., con instradamento
  per‑agente e failover.
* **Opzione solo locale:** esegui modelli locali così che **tutti i dati possano rimanere sul tuo dispositivo**
  se lo desideri.
* **Instradamento multi‑agente:** separa gli agenti per canale, account o attività, ciascuno con il proprio
  spazio di lavoro e i propri default.
* **Open source e modificabile:** ispeziona, estendi ed esegui in self‑hosting senza lock‑in rispetto al fornitore.

Documentazione: [Gateway](/it/gateway), [Channels](/it/channels), [Multi‑agent](/it/concepts/multi-agent),
[Memory](/it/concepts/memory).

<div id="i-just-set-it-up-what-should-i-do-first">
  ### L&#39;ho appena configurato: da dove comincio
</div>

Alcuni buoni progetti iniziali:

* Crea un sito web (WordPress, Shopify o un semplice sito statico).
* Prototipa un&#39;app mobile (struttura, schermate, piano per le api).
* Organizza file e cartelle (pulizia, convenzioni di denominazione, tag).
* Collega Gmail e automatizza riepiloghi o follow-up.

Può gestire attività di grandi dimensioni, ma funziona meglio se le suddividi in fasi e
usi sottoagenti per il lavoro in parallelo.

<div id="what-are-the-top-five-everyday-use-cases-for-openclaw">
  ### Quali sono i cinque principali casi d&#39;uso quotidiani di OpenClaw
</div>

I vantaggi quotidiani di solito si presentano così:

* **Briefing personali:** riepiloghi della posta in arrivo, del calendario e delle notizie che ti interessano.
* **Ricerca e stesura:** ricerche rapide, riepiloghi e prime bozze per email o documenti.
* **Promemoria e follow-up:** solleciti e checklist guidati da cron o da eventi di heartbeat.
* **Automazione del browser:** compilazione di moduli, raccolta di dati e ripetizione di attività sul web.
* **Coordinamento tra dispositivi:** invia un&#39;attività dal telefono, lascia che il Gateway la esegua su un server e ricevi il risultato in chat.

<div id="can-openclaw-help-with-lead-gen-outreach-ads-and-blogs-for-a-saas">
  ### OpenClaw può aiutare con campagne di lead gen, annunci e blog per un SaaS
</div>

Sì, per **ricerca, qualificazione e stesura**. Può analizzare siti, creare shortlist,
riassumere i potenziali clienti e scrivere bozze di testi per outreach o annunci.

Per **campagne di outreach o annunci**, mantieni sempre una supervisione umana. Evita lo spam, rispetta le leggi locali e
le policy delle piattaforme e rivedi sempre tutto prima dell’invio. Lo schema più sicuro è lasciare
che OpenClaw prepari la bozza e che tu la approvi.

Documentazione: [Sicurezza](/it/gateway/security).

<div id="what-are-the-advantages-vs-claude-code-for-web-development">
  ### Quali sono i vantaggi rispetto a Claude Code per lo sviluppo web
</div>

OpenClaw è un **assistente personale** e un livello di coordinamento, non un sostituto di un IDE. Usa
Claude Code o Codex per il ciclo di scrittura del codice più rapido all&#39;interno di un repo. Usa OpenClaw quando
ti serve memoria a lungo termine, accesso multi-dispositivo e orchestrazione degli strumenti.

Vantaggi:

* **Memoria persistente + spazio di lavoro** tra le sessioni
* **Accesso multipiattaforma** (WhatsApp, Telegram, TUI, WebChat)
* **Orchestrazione degli strumenti** (browser, file, pianificazione, hook)
* **Gateway sempre attivo** (esegui su un VPS, interagisci da qualsiasi luogo)
* **Nodi** per browser/schermo/camera/exec locali

Showcase: https://openclaw.ai/showcase

<div id="skills-and-automation">
  ## Abilità e automazione
</div>

<div id="how-do-i-customize-skills-without-keeping-the-repo-dirty">
  ### Come posso personalizzare le abilità senza sporcare il repository
</div>

Usa degli override gestiti invece di modificare la copia del repository. Metti le tue modifiche in `~/.openclaw/skills/<name>/SKILL.md` (oppure aggiungi una cartella tramite `skills.load.extraDirs` in `~/.openclaw/openclaw.json`). La precedenza è `<workspace>/skills` &gt; `~/.openclaw/skills` &gt; abilità incluse nel bundle, quindi gli override gestiti hanno la priorità senza toccare git. Solo le modifiche che vale la pena proporre a monte dovrebbero restare nel repository ed essere inviate come PR.

<div id="can-i-load-skills-from-a-custom-folder">
  ### Posso caricare abilità da una cartella personalizzata?
</div>

Sì. Aggiungi directory aggiuntive tramite `skills.load.extraDirs` in `~/.openclaw/openclaw.json` (precedenza più bassa). L&#39;ordine di precedenza predefinito è: `<workspace>/skills` → `~/.openclaw/skills` → abilità incluse nel pacchetto → `skills.load.extraDirs`. Per impostazione predefinita, `clawhub` installa in `./skills`, che OpenClaw tratta come `<workspace>/skills`.

<div id="how-can-i-use-different-models-for-different-tasks">
  ### Come posso usare modelli diversi per attività diverse?
</div>

Al momento i pattern supportati sono:

* **Cron job**: i job isolati possono impostare un override del parametro `model` per ogni job.
* **Sub-agenti**: instrada le attività verso agenti separati con modelli predefiniti diversi.
* **Switch on-demand**: usa `/model` per cambiare in qualsiasi momento il modello della sessione corrente.

Consulta [Cron job](/it/automation/cron-jobs), [Instradamento Multi-Agente](/it/concepts/multi-agent) e [Slash command](/it/tools/slash-commands).

<div id="the-bot-freezes-while-doing-heavy-work-how-do-i-offload-that">
  ### Il bot si blocca durante lavori pesanti Come posso alleggerire il carico
</div>

Usa **sottoagenti** per le attività lunghe o parallele. I sottoagenti vengono eseguiti in una propria sessione,
restituiscono un riepilogo e mantengono reattiva la tua chat principale.

Chiedi al tuo bot di &quot;creare un sottoagente per questa attività&quot; oppure usa `/subagents`.
Usa `/status` in chat per vedere cosa sta facendo il Gateway in questo momento (e se è occupato).

Suggerimento sui token: le attività lunghe e i sottoagenti consumano entrambi token. Se il costo è un problema, imposta un
modello più economico per i sottoagenti tramite `agents.defaults.subagents.model`.

Documentazione: [Sottoagenti](/it/tools/subagents).

<div id="cron-or-reminders-do-not-fire-what-should-i-check">
  ### Cron o promemoria non vengono attivati: cosa devo controllare
</div>

Cron viene eseguito all&#39;interno del processo del Gateway. Se il Gateway non è in esecuzione in modo continuativo,
i job pianificati non verranno eseguiti.

Checklist:

* Verifica che cron sia abilitato (`cron.enabled`) e che `OPENCLAW_SKIP_CRON` non sia impostata.
* Controlla che il Gateway sia in esecuzione 24/7 (senza sospensioni né riavvii).
* Verifica le impostazioni del fuso orario per il job (`--tz` rispetto al fuso orario dell&#39;host).

Debug:

```bash
openclaw cron run <jobId> --force
openclaw cron runs --id <jobId> --limit 50
```

Documentazione: [cron job](/it/automation/cron-jobs), [Cron vs heartbeat](/it/automation/cron-vs-heartbeat).

<div id="how-do-i-install-skills-on-linux">
  ### Come posso installare le abilità su Linux
</div>

Usa **ClawHub** (CLI) oppure copia le abilità nel tuo spazio di lavoro. La UI delle abilità su macOS non è disponibile su Linux.
Esplora le abilità su https://clawhub.com.

Installa la CLI di ClawHub (scegli un gestore di pacchetti):

```bash
npm i -g clawhub
```

```bash
pnpm add -g clawhub
```

<div id="can-openclaw-run-tasks-on-a-schedule-or-continuously-in-the-background">
  ### OpenClaw può eseguire attività pianificate o in modo continuo in background
</div>

Sì. Usa lo scheduler del Gateway:

* **Cron job** per attività pianificate o ricorrenti (persistono tra i riavvii).
* **Heartbeat** per controlli periodici della “sessione principale”.
* **Job isolati** per agenti autonomi che pubblicano riepiloghi o li inviano alle chat.

Documentazione: [Cron jobs](/it/automation/cron-jobs), [Cron vs Heartbeat](/it/automation/cron-vs-heartbeat),
[Heartbeat](/it/gateway/heartbeat).

**Posso eseguire abilità solo per Apple macOS da Linux**

Non direttamente. Le abilità macOS sono limitate da `metadata.openclaw.os` più i binari richiesti, e le abilità compaiono nel system prompt solo quando sono idonee sull’**host del Gateway**. Su Linux, le abilità solo per `darwin` (come `imsg`, `apple-notes`, `apple-reminders`) non verranno caricate a meno che tu non sovrascriva questo vincolo.

Hai tre modelli supportati:

**Opzione A - eseguire il Gateway su un Mac (la più semplice).**\
Esegui il Gateway dove esistono i binari macOS, quindi connettiti da Linux in [modalità remota](#how-do-i-run-openclaw-in-remote-mode-client-connects-to-a-gateway-elsewhere) o tramite Tailscale. Le abilità si caricano normalmente perché l’host del Gateway è macOS.

**Opzione B - usare un nodo macOS (senza SSH).**\
Esegui il Gateway su Linux, abbina un nodo macOS (app nella barra dei menu) e imposta **Node Run Commands** su &quot;Always Ask&quot; o &quot;Always Allow&quot; sul Mac. OpenClaw può considerare idonee le abilità solo macOS quando i binari richiesti esistono sul nodo. L’agente esegue queste abilità tramite lo strumento `nodes`. Se scegli &quot;Always Ask&quot;, approvare &quot;Always Allow&quot; nel prompt aggiunge quel comando alla lista di autorizzati.

**Opzione C - mettere in proxy i binari macOS via SSH (avanzato).**\
Mantieni il Gateway su Linux, ma fai in modo che i binari CLI richiesti puntino a wrapper SSH che vengono eseguiti su un Mac. Quindi sovrascrivi l’abilità per consentire Linux in modo che rimanga idonea.

1. Crea un wrapper SSH per il binario (esempio: `imsg`):
   ```bash
   #!/usr/bin/env bash
   set -euo pipefail
   exec ssh -T user@mac-host /opt/homebrew/bin/imsg "$@"
   ```
2. Metti il wrapper nel `PATH` sull’host Linux (per esempio `~/bin/imsg`).
3. Sovrascrivi i metadati dell’abilità (spazio di lavoro o `~/.openclaw/skills`) per consentire Linux:
   ```markdown
   ---
   name: imsg
   description: iMessage/SMS CLI for listing chats, history, watch, and sending.
   metadata: {"openclaw":{"os":["darwin","linux"],"requires":{"bins":["imsg"]}}}
   ---
   ```
4. Avvia una nuova sessione in modo che l’istantanea delle abilità venga aggiornata.

Per iMessage in particolare, puoi anche puntare `channels.imessage.cliPath` a un wrapper SSH (OpenClaw richiede solo stdio). Vedi [iMessage](/it/channels/imessage).

<div id="do-you-have-a-notion-or-heygen-integration">
  ### Hai un&#39;integrazione con Notion o HeyGen
</div>

Non è integrata nativamente al momento.

Opzioni:

* **Abilità / plugin personalizzato:** ideale per un accesso API affidabile (sia Notion che HeyGen espongono API).
* **Automazione del browser:** funziona senza codice ma è più lenta e più fragile.

Se vuoi mantenere il contesto per cliente (flussi di lavoro per agenzie), uno schema semplice è:

* Una pagina Notion per cliente (contesto + preferenze + lavoro attivo).
* Chiedi all&#39;agente di recuperare quella pagina all&#39;inizio di una sessione.

Se vuoi un&#39;integrazione nativa, apri una feature request o sviluppa un&#39;abilità
che utilizzi queste API.

Installa le abilità:

```bash
clawhub install <skill-slug>
clawhub update --all
```

ClawHub si installa in `./skills` all&#39;interno della directory corrente (oppure, in alternativa, nello spazio di lavoro OpenClaw configurato); OpenClaw la considera come `<workspace>/skills` per la sessione successiva. Per condividere le abilità tra più agenti, inseriscile in `~/.openclaw/skills/<name>/SKILL.md`. Alcune abilità richiedono binari installati tramite Homebrew; su Linux ciò significa Linuxbrew (vedi la voce della FAQ di Homebrew per Linux riportata sopra). Consulta [Skills](/it/tools/skills) e [ClawHub](/it/tools/clawhub).

<div id="how-do-i-install-the-chrome-extension-for-browser-takeover">
  ### Come installo l&#39;estensione di Chrome per prendere il controllo del browser
</div>

Usa il programma di installazione integrato, quindi carica in Chrome l&#39;estensione non pacchettizzata:

```bash
openclaw browser extension install
openclaw browser extension path
```

Poi in Chrome → `chrome://extensions` → abilita “Modalità sviluppatore” → “Carica estensione non pacchettizzata” → seleziona quella cartella.

Guida completa (incluso Gateway remoto + note sulla sicurezza): [Chrome extension](/it/tools/chrome-extension)

Se il Gateway è in esecuzione sulla stessa macchina di Chrome (configurazione predefinita), di solito **non** ti serve nient&#39;altro.
Se il Gateway è in esecuzione altrove, esegui un node sulla macchina del browser in modo che il Gateway possa fare da proxy alle azioni del browser.
Devi comunque cliccare il pulsante dell&#39;estensione sulla scheda che vuoi controllare (non si collega automaticamente).

<div id="sandboxing-and-memory">
  ## Sandbox e memoria
</div>

<div id="is-there-a-dedicated-sandboxing-doc">
  ### Esiste una documentazione dedicata al sandboxing
</div>

Sì. Consulta [Sandboxing](/it/gateway/sandboxing). Per la configurazione specifica di Docker (gateway completo in Docker o immagini sandbox), vedi [Docker](/it/install/docker).

**Posso mantenere i DM personali ma rendere pubblici i gruppi in sandbox con un solo agente**

Sì, se il tuo traffico privato sono i **DM** e il tuo traffico pubblico sono i **gruppi**.

Usa `agents.defaults.sandbox.mode: "non-main"` in modo che le sessioni di gruppo/canale (chiavi non-main) vengano eseguite in Docker, mentre la sessione DM principale resta sull&#39;host. Poi limita quali strumenti sono disponibili nelle sessioni in sandbox tramite `tools.sandbox.tools`.

Procedura guidata di configurazione + esempio di configurazione: [Groups: personal DMs + public groups](/it/concepts/groups#pattern-personal-dms-public-groups-single-agent)

Riferimento di configurazione principale: [Gateway configuration](/it/gateway/configuration#agentsdefaultssandbox)

<div id="how-do-i-bind-a-host-folder-into-the-sandbox">
  ### Come collegare una cartella dell&#39;host alla sandbox
</div>

Imposta `agents.defaults.sandbox.docker.binds` su `["host:path:mode"]` (ad esempio `"/home/user/src:/src:ro"`). I bind globali e per agente vengono uniti; i bind per agente vengono ignorati quando `scope: "shared"`. Usa `:ro` per qualsiasi elemento sensibile e ricorda che i bind aggirano le barriere del filesystem della sandbox. Consulta [Sandboxing](/it/gateway/sandboxing#custom-bind-mounts) e [Sandbox vs Tool Policy vs Elevated](/it/gateway/sandbox-vs-tool-policy-vs-elevated#bind-mounts-security-quick-check) per esempi e note sulla sicurezza.

<div id="how-does-memory-work">
  ### Come funziona la memoria
</div>

La memoria di OpenClaw consiste semplicemente in file Markdown nello spazio di lavoro dell&#39;agente:

* Note giornaliere in `memory/YYYY-MM-DD.md`
* Note curate a lungo termine in `MEMORY.md` (solo per le sessioni principali/private)

OpenClaw esegue anche un **flush silenzioso della memoria prima della compattazione** per ricordare al modello
di scrivere note persistenti prima della compattazione automatica. Questo viene eseguito solo quando lo spazio di lavoro
è scrivibile (le sandbox in sola lettura lo ignorano). Vedi [Memoria](/it/concepts/memory).

<div id="memory-keeps-forgetting-things-how-do-i-make-it-stick">
  ### La memoria continua a dimenticare le cose Come faccio a renderla persistente
</div>

Chiedi al bot di **scrivere il fatto nella memoria**. Le note a lungo termine vanno in `MEMORY.md`,
il contesto a breve termine va in `memory/YYYY-MM-DD.md`.

Questa è ancora un&#39;area che stiamo migliorando. È utile ricordare al modello di salvare le informazioni in memoria;
saprà cosa fare. Se continua a dimenticare, verifica che il Gateway stia usando lo stesso
spazio di lavoro a ogni esecuzione.

Documentazione: [Memoria](/it/concepts/memory), [Spazio di lavoro dell&#39;Agente](/it/concepts/agent-workspace).

<div id="does-semantic-memory-search-require-an-openai-api-key">
  ### La ricerca di memoria semantica richiede una API key OpenAI?
</div>

Solo se usi gli **OpenAI embeddings**. L’OAuth di Codex copre chat/completions e
**non** concede accesso agli embeddings, quindi **accedere con Codex (OAuth o il
login Codex CLI)** non aiuta per la ricerca di memoria semantica. Gli OpenAI embeddings
hanno comunque bisogno di una vera chiave API (`OPENAI_API_KEY` o `models.providers.openai.apiKey`).

Se non imposti esplicitamente un provider, OpenClaw seleziona automaticamente un provider quando
riesce a risolvere una chiave API (profili di autenticazione, `models.providers.*.apiKey` o variabili d’ambiente).
Preferisce OpenAI se trova una chiave OpenAI, altrimenti Gemini se trova una chiave Gemini.
Se nessuna delle due chiavi è disponibile, la ricerca di memoria rimane disabilitata finché non
la configuri. Se hai configurato e disponibile un percorso di modello locale, OpenClaw
preferisce `local`.

Se preferisci restare in locale, imposta `memorySearch.provider = "local"` (e facoltativamente
`memorySearch.fallback = "none"`). Se vuoi gli embeddings di Gemini, imposta
`memorySearch.provider = "gemini"` e fornisci `GEMINI_API_KEY` (o
`memorySearch.remote.apiKey`). Supportiamo modelli di embedding **OpenAI, Gemini o locali**:
vedi [Memory](/it/concepts/memory) per i dettagli di configurazione.

<div id="does-memory-persist-forever-what-are-the-limits">
  ### La memoria persiste per sempre? Quali sono i limiti
</div>

I file di memoria risiedono su disco e persistono finché non li elimini. Il limite è il tuo spazio di archiviazione, non il modello. Il **contesto di sessione** è comunque limitato dalla finestra di contesto (context window) del modello, quindi le conversazioni lunghe possono essere compattate o troncate. È per questo che esiste la ricerca nella memoria: recupera solo le parti rilevanti e le reintroduce nel contesto.

Documentazione: [Memoria](/it/concepts/memory), [Contesto](/it/concepts/context).

<div id="where-things-live-on-disk">
  ## Dove risiedono i dati su disco
</div>

<div id="is-all-data-used-with-openclaw-saved-locally">
  ### Tutti i dati usati con OpenClaw vengono salvati in locale
</div>

No: **lo stato di OpenClaw è locale**, ma **i servizi esterni vedono comunque ciò che invii loro**.

* **Locale per impostazione predefinita:** sessioni, file di memoria, configurazione e spazio di lavoro risiedono sull&#39;host del Gateway
  (`~/.openclaw` + la tua directory di spazio di lavoro).
* **Remoto per necessità:** i messaggi che invii ai provider di modelli (Anthropic/OpenAI/etc.) vengono inviati
  alle loro API e le piattaforme di chat (WhatsApp/Telegram/Slack/etc.) archiviano i dati dei messaggi sui loro
  server.
* **Controlli tu l&#39;impronta:** usare modelli locali mantiene i prompt sulla tua macchina, ma il traffico dei canali
  passa comunque attraverso i server del canale.

Correlato: [Spazio di lavoro dell&#39;Agente](/it/concepts/agent-workspace), [Memoria](/it/concepts/memory).

<div id="where-does-openclaw-store-its-data">
  ### Dove OpenClaw archivia i propri dati
</div>

Tutto si trova sotto `$OPENCLAW_STATE_DIR` (predefinito: `~/.openclaw`):

| Path | Scopo |
|------|-------|
| `$OPENCLAW_STATE_DIR/openclaw.json` | Configurazione principale (JSON5) |
| `$OPENCLAW_STATE_DIR/credentials/oauth.json` | Importazione OAuth legacy (copiata nei profili di autenticazione al primo utilizzo) |
| `$OPENCLAW_STATE_DIR/agents/<agentId>/agent/auth-profiles.json` | Profili di autenticazione (OAuth + API key) |
| `$OPENCLAW_STATE_DIR/agents/<agentId>/agent/auth.json` | Cache di autenticazione a runtime (gestita automaticamente) |
| `$OPENCLAW_STATE_DIR/credentials/` | Stato dei provider (es. `whatsapp/<accountId>/creds.json`) |
| `$OPENCLAW_STATE_DIR/agents/` | Stato per agente (agentDir + sessioni) |
| `$OPENCLAW_STATE_DIR/agents/<agentId>/sessions/` | Cronologia e stato delle conversazioni (per agente) |
| `$OPENCLAW_STATE_DIR/agents/<agentId>/sessions/sessions.json` | Metadati della sessione (per agente) |

Percorso legacy per agente singolo: `~/.openclaw/agent/*` (migrato da `openclaw doctor`).

Il tuo **spazio di lavoro** (AGENTS.md, file di memoria, abilità, ecc.) è separato e viene configurato tramite `agents.defaults.workspace` (predefinito: `~/.openclaw/workspace`).

<div id="where-should-agentsmd-soulmd-usermd-memorymd-live">
  ### Dove devono risiedere AGENTSmd SOULmd USERmd MEMORYmd
</div>

Questi file risiedono nello **spazio di lavoro dell&#39;agente**, non in `~/.openclaw`.

* **Spazio di lavoro (per agente)**: `AGENTS.md`, `SOUL.md`, `IDENTITY.md`, `USER.md`,
  `MEMORY.md` (o `memory.md`), `memory/YYYY-MM-DD.md`, `HEARTBEAT.md` facoltativo.
* **Directory di stato (`~/.openclaw`)**: configurazione, credenziali, profili di autenticazione, sessioni, log
  e abilità condivise (`~/.openclaw/skills`).

Lo spazio di lavoro predefinito è `~/.openclaw/workspace`, configurabile tramite:

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } }
}
```

Se il bot “si dimentica” dopo un riavvio, verifica che il Gateway stia usando lo stesso
spazio di lavoro a ogni avvio (e ricorda: la modalità remota usa lo **spazio di lavoro dell’host del Gateway**,
non il tuo portatile locale).

Suggerimento: se vuoi un comportamento o una preferenza persistente, chiedi al bot di **scriverli in
AGENTS.md o MEMORY.md** anziché fare affidamento sulla cronologia della chat.

Vedi [Spazio di lavoro dell’Agente](/it/concepts/agent-workspace) e [Memoria](/it/concepts/memory).

<div id="whats-the-recommended-backup-strategy">
  ### Qual è la strategia di backup consigliata
</div>

Metti lo **spazio di lavoro del tuo agente** in una repo Git **privata** ed esegui il backup in una posizione
privata (per esempio un repository privato su GitHub). In questo modo includi la memoria + i file
AGENTS/SOUL/USER e potrai ripristinare la “mente” dell&#39;assistente in seguito.

**Non** eseguire il commit di nulla all&#39;interno di `~/.openclaw` (credenziali, sessioni, token).
Se hai bisogno di un ripristino completo, esegui il backup sia dello spazio di lavoro sia della directory
di stato separatamente (vedi la domanda sulla migrazione sopra).

Documentazione: [Spazio di lavoro dell’agente](/it/concepts/agent-workspace).

<div id="how-do-i-completely-uninstall-openclaw">
  ### Come disinstallare completamente OpenClaw
</div>

Consulta la guida dedicata: [Disinstallazione](/it/install/uninstall).

<div id="can-agents-work-outside-the-workspace">
  ### Gli agenti possono lavorare al di fuori dello spazio di lavoro
</div>

Sì. Lo spazio di lavoro è la **cwd predefinita** e l&#39;ancora di memoria, non una sandbox rigida.
I percorsi relativi vengono risolti all&#39;interno dello spazio di lavoro, ma i percorsi assoluti
possono accedere ad altre posizioni dell&#39;host a meno che la sandbox non sia abilitata. Se ti serve
isolamento, usa [`agents.defaults.sandbox`](/it/gateway/sandboxing) o impostazioni di sandbox per singolo agente. Se vuoi
che una repo sia la directory di lavoro predefinita, imposta lo `workspace` di quell’agente
alla root della repo. La repo di OpenClaw è solo codice sorgente; mantieni lo
spazio di lavoro separato a meno che tu non voglia esplicitamente che l’agente lavori al suo interno.

Esempio (repo come cwd predefinita):

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
  ### Sono in modalità remota: dove si trova l&#39;archivio delle sessioni
</div>

Lo stato della sessione è di proprietà dell&#39;**host del Gateway**. Se sei in modalità remota, l&#39;archivio delle sessioni che ti interessa si trova sulla macchina remota, non sul tuo laptop locale. Vedi [Gestione delle sessioni](/it/concepts/session).

<div id="config-basics">
  ## Nozioni di base sulla configurazione
</div>

<div id="what-format-is-the-config-where-is-it">
  ### In che formato è la config e dove si trova
</div>

OpenClaw legge un file di config **JSON5** opzionale da `$OPENCLAW_CONFIG_PATH` (percorso predefinito: `~/.openclaw/openclaw.json`):

```
$OPENCLAW_CONFIG_PATH
```

Se il file non è presente, usa impostazioni predefinite abbastanza sicure (compreso uno spazio di lavoro predefinito in `~/.openclaw/workspace`).

<div id="i-set-gatewaybind-lan-or-tailnet-and-now-nothing-listens-the-ui-says-unauthorized">
  ### Ho impostato gatewaybind su lan o tailnet e ora non c&#39;è nulla in ascolto, la UI indica unauthorized
</div>

I bind non-loopback **richiedono l&#39;autenticazione**. Configura `gateway.auth.mode` + `gateway.auth.token` (oppure usa `OPENCLAW_GATEWAY_TOKEN`).

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

Note:

* `gateway.remote.token` è esclusivamente per **chiamate CLI remote**; non abilita l&#39;autenticazione locale del Gateway.
* La Control UI si autentica tramite `connect.params.auth.token` (salvato nelle impostazioni dell&#39;app/UI). Evita di includere token negli URL.

<div id="why-do-i-need-a-token-on-localhost-now">
  ### Perché ora ho bisogno di un token su localhost
</div>

Il wizard genera per impostazione predefinita un token del Gateway (anche su loopback), quindi **i client WS locali devono autenticarsi**. Questo impedisce ad altri processi locali di chiamare il Gateway. Incolla il token nelle impostazioni della Control UI (o nella configurazione del tuo client) per connetterti.

Se **davvero** vuoi un loopback aperto, rimuovi `gateway.auth` dalla tua configurazione. Doctor può generare un token per te in qualsiasi momento: `openclaw doctor --generate-gateway-token`.

<div id="do-i-have-to-restart-after-changing-config">
  ### Devo riavviare dopo aver modificato la configurazione
</div>

Il Gateway monitora la configurazione e supporta l’hot reload:

* `gateway.reload.mode: "hybrid"` (predefinito): applica al volo le modifiche sicure, richiede il riavvio per quelle critiche
* `hot`, `restart`, `off` sono anch’essi supportati

<div id="how-do-i-enable-web-search-and-web-fetch">
  ### Come abilito web search e web fetch
</div>

`web_fetch` funziona senza una chiave API. `web_search` richiede una chiave Brave Search API. **Consigliato:** esegui `openclaw configure --section web` per salvarla in `tools.web.search.apiKey`. Alternativa tramite variabile d&#39;ambiente: imposta la variabile `BRAVE_API_KEY` per il processo del Gateway.

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

Note:

* Se usi la lista di autorizzati, aggiungi `web_search`/`web_fetch` oppure `group:web`.
* `web_fetch` è abilitato per impostazione predefinita (a meno che non venga esplicitamente disabilitato).
* I daemon leggono le variabili d&#39;ambiente da `~/.openclaw/.env` (o dall&#39;ambiente del servizio).

Documentazione: [Strumenti web](/it/tools/web).

<div id="how-do-i-run-a-central-gateway-with-specialized-workers-across-devices">
  ### Come eseguire un Gateway centrale con worker specializzati su più dispositivi
</div>

Lo schema più comune è **un Gateway** (ad es. Raspberry Pi) più **nodi** e **agenti**:

* **Gateway (centrale):** gestisce i canali (Signal/WhatsApp), l’instradamento e le sessioni.
* **Nodi (dispositivi):** Mac/iOS/Android si connettono come periferiche ed espongono strumenti locali (`system.run`, `canvas`, `camera`).
* **Agenti (worker):** cervelli/spazi di lavoro separati per ruoli specializzati (ad es. “Hetzner ops”, “Dati personali”).
* **Sub‑agenti:** avviano lavoro in background da un agente principale quando ti serve parallelismo.
* **TUI:** si connette al Gateway e consente di passare da un agente/sessione all’altro.

Documentazione: [Nodes](/it/nodes), [Remote access](/it/gateway/remote), [Multi-Agent Routing](/it/concepts/multi-agent), [Sub-agents](/it/tools/subagents), [TUI](/it/tui).

<div id="can-the-openclaw-browser-run-headless">
  ### Il browser di OpenClaw può essere eseguito in modalità headless
</div>

Sì, è un&#39;opzione di configurazione:

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

Il valore predefinito è `false` (con interfaccia grafica). La modalità headless ha più probabilità di attivare i controlli anti‑bot su alcuni siti. Vedi [Browser](/it/tools/browser).

La modalità headless usa **lo stesso motore Chromium** e funziona per la maggior parte delle attività di automazione (moduli, clic, scraping, accessi). Le principali differenze:

* Nessuna finestra del browser visibile (usa screenshot/catture schermo se hai bisogno di elementi visivi).
* Alcuni siti sono più rigidi riguardo all&#39;automazione in modalità headless (CAPTCHA, anti‑bot).
  Ad esempio, X/Twitter blocca spesso le sessioni headless.

<div id="how-do-i-use-brave-for-browser-control">
  ### Come usare Brave per controllare il browser
</div>

Imposta `browser.executablePath` sul percorso del binario di Brave (o di un qualsiasi browser basato su Chromium) e riavvia il Gateway.
Consulta gli esempi completi di configurazione in [Browser](/it/tools/browser#use-brave-or-another-chromium-based-browser).

<div id="remote-gateways-nodes">
  ## Gateway e nodi remoti
</div>

<div id="how-do-commands-propagate-between-telegram-the-gateway-and-nodes">
  ### In che modo i comandi si propagano tra Telegram, il Gateway e i nodi
</div>

I messaggi di Telegram sono gestiti dal **Gateway**. Il Gateway esegue l&#39;agente e
solo allora chiama i nodi tramite il **Gateway WebSocket** quando è necessario usare uno strumento del nodo:

Telegram → Gateway → Agente → `node.*` → Nodo → Gateway → Telegram

I nodi non vedono il traffico in ingresso dal provider; ricevono solo chiamate RPC del nodo.

<div id="how-can-my-agent-access-my-computer-if-the-gateway-is-hosted-remotely">
  ### Come può il mio agente accedere al mio computer se il Gateway è ospitato in remoto
</div>

Risposta breve: **abbina il tuo computer come nodo**. Il Gateway viene eseguito altrove, ma può
chiamare gli strumenti `node.*` (screen, camera, system) sulla tua macchina locale tramite il WebSocket del Gateway.

Configurazione tipica:

1. Esegui il Gateway sull&#39;host sempre attivo (VPS/server domestico).
2. Collega l&#39;host del Gateway e il tuo computer alla stessa tailnet.
3. Assicurati che il WS del Gateway sia raggiungibile (bind sulla tailnet o tunnel SSH).
4. Apri l&#39;app macOS in locale e connettiti in modalità **Remote over SSH** (o direttamente via tailnet)
   così può registrarsi come nodo.
5. Approva il nodo sul Gateway:
   ```bash
   openclaw nodes pending
   openclaw nodes approve <requestId>
   ```

Non è necessario alcun bridge TCP separato; i nodi si connettono tramite il WebSocket del Gateway.

Promemoria di sicurezza: l&#39;abbinamento di un nodo macOS consente `system.run` su quella macchina. Abbina solo dispositivi di cui ti fidi e rivedi [Security](/it/gateway/security).

Documentazione: [Nodes](/it/nodes), [Gateway protocol](/it/gateway/protocol), [macOS remote mode](/it/platforms/mac/remote), [Security](/it/gateway/security).

<div id="tailscale-is-connected-but-i-get-no-replies-what-now">
  ### Tailscale è connesso ma non ricevo risposte. Che cosa posso fare?
</div>

Controlla le basi:

* Gateway è in esecuzione: `openclaw gateway status`
* Stato di salute del Gateway: `openclaw status`
* Stato di salute dei canali: `openclaw channels status`

Poi verifica autenticazione e instradamento:

* Se usi Tailscale Serve, assicurati che `gateway.auth.allowTailscale` sia impostato correttamente.
* Se ti connetti tramite tunnel SSH, conferma che il tunnel locale sia attivo e punti alla porta corretta.
* Conferma che la tua lista di autorizzati (DM o gruppo) includa il tuo account.

Documentazione: [Tailscale](/it/gateway/tailscale), [Accesso remoto](/it/gateway/remote), [Canali](/it/channels).

<div id="can-two-openclaw-instances-talk-to-each-other-local-vps">
  ### Due istanze OpenClaw possono comunicare tra loro su un VPS locale
</div>

Sì. Non esiste un bridge &quot;bot-to-bot&quot; integrato, ma puoi collegarle in alcuni
modi affidabili:

**Il più semplice:** usa un normale canale di chat a cui entrambi i bot hanno accesso (Telegram/Slack/WhatsApp).
Fai in modo che il Bot A invii un messaggio al Bot B, quindi lascia che il Bot B risponda come al solito.

**Bridge via CLI (generico):** esegui uno script che chiama l&#39;altro Gateway con
`openclaw agent --message ... --deliver`, indirizzato a una chat in cui l&#39;altro bot
è in ascolto. Se un bot si trova su un VPS remoto, punta la tua CLI a quel Gateway remoto
tramite SSH/Tailscale (vedi [Accesso remoto](/it/gateway/remote)).

Schema di esempio (da eseguire da una macchina che può raggiungere il Gateway di destinazione):

```bash
openclaw agent --message "Ciao dal bot locale" --deliver --channel telegram --reply-to <chat-id>
```

Suggerimento: aggiungi un guardrail così che i due bot non si rispondano all&#39;infinito (solo menzione, liste di autorizzati per il canale o una regola &quot;non rispondere ai messaggi dei bot&quot;).

Documentazione: [Accesso remoto](/it/gateway/remote), [CLI Agente](/it/cli/agent), [Agent send](/it/tools/agent-send).

<div id="do-i-need-separate-vpses-for-multiple-agents">
  ### Ho bisogno di VPS separati per più agenti?
</div>

No. Un unico Gateway può ospitare più agenti, ognuno con il proprio spazio di lavoro, le proprie impostazioni predefinite del modello
e il proprio instradamento. Questa è la configurazione normale ed è molto più economica e semplice
di eseguire un VPS separato per ogni agente.

Usa VPS separati solo se ti serve un isolamento rigoroso (confini di sicurezza) o configurazioni
molto diverse che non vuoi condividere. Altrimenti, mantieni un solo Gateway e
usa più agenti o sottoagenti.

<div id="is-there-a-benefit-to-using-a-node-on-my-personal-laptop-instead-of-ssh-from-a-vps">
  ### C’è un vantaggio a usare un nodo sul mio laptop personale invece di usare SSH da un VPS
</div>

Sì: i nodi sono il modo principale per raggiungere il tuo laptop da un Gateway remoto e
abilitano molto più del semplice accesso alla shell. Il Gateway viene eseguito su macOS/Linux (Windows via WSL2) ed è
leggero (va bene un piccolo VPS o una macchina tipo Raspberry Pi; 4 GB di RAM sono più che sufficienti), quindi una configurazione comune è un host sempre attivo più il tuo laptop come nodo.

* **Nessun SSH in ingresso richiesto.** I nodi si connettono in uscita al Gateway via WebSocket e usano l’abbinamento del dispositivo.
* **Controlli di esecuzione più sicuri.** `system.run` è protetto da liste di autorizzati/approvazioni del nodo su quel laptop.
* **Più strumenti del dispositivo.** I nodi espongono `canvas`, `camera` e `screen` oltre a `system.run`.
* **Automazione del browser locale.** Tieni il Gateway su un VPS, ma esegui Chrome in locale e inoltra il controllo
  con l’estensione di Chrome + un nodo host sul laptop.

SSH va bene per l’accesso alla shell sporadico, ma i nodi sono più semplici per flussi di lavoro continuativi degli agenti e
per l’automazione del dispositivo.

Documentazione: [Nodes](/it/nodes), [Nodes CLI](/it/cli/nodes), [Chrome extension](/it/tools/chrome-extension).

<div id="should-i-install-on-a-second-laptop-or-just-add-a-node">
  ### Dovrei installarlo su un secondo laptop o aggiungere solo un nodo
</div>

Se sul secondo laptop ti servono solo **strumenti locali** (screen/camera/exec), aggiungilo come
**nodo**. In questo modo mantieni un unico Gateway ed eviti di duplicare la configurazione. Gli strumenti locali del nodo sono
attualmente disponibili solo su macOS, ma prevediamo di estenderli ad altri sistemi operativi.

Installa un secondo Gateway solo quando ti serve **forte isolamento** o due bot completamente separati.

Documentazione: [Nodi](/it/nodes), [CLI dei nodi](/it/cli/nodes), [Gateway multipli](/it/gateway/multiple-gateways).

<div id="do-nodes-run-a-gateway-service">
  ### I nodi eseguono un servizio Gateway
</div>

No. Solo **un Gateway** dovrebbe essere in esecuzione per host, a meno che tu non stia eseguendo intenzionalmente profili isolati (vedi [Gateway multipli](/it/gateway/multiple-gateways)). I nodi sono periferiche che si connettono
al Gateway (nodi iOS/Android o la “modalità nodo” su macOS nell&#39;app della barra dei menu). Per gli
host di nodi headless e il controllo via CLI, vedi [CLI host nodo](/it/cli/node).

È richiesto un riavvio completo per le modifiche a `gateway`, `discovery` e `canvasHost`.

<div id="is-there-an-api-rpc-way-to-apply-config">
  ### Esiste un modo via API/RPC per applicare la configurazione
</div>

Sì. `config.apply` convalida e scrive l&#39;intera configurazione e riavvia il Gateway come parte dell&#39;operazione.

<div id="configapply-wiped-my-config-how-do-i-recover-and-avoid-this">
  ### configapply ha cancellato la mia config Come posso ripristinarla ed evitare che succeda di nuovo
</div>

`config.apply` sostituisce **l&#39;intera config**. Se invii un oggetto parziale, tutto
il resto viene rimosso.

Per ripristinarla:

* Ripristina da un backup (git o una copia di `~/.openclaw/openclaw.json`).
* Se non hai alcun backup, esegui di nuovo `openclaw doctor` e riconfigura canali/modelli.
* Se questo comportamento era inaspettato, apri una segnalazione di bug e includi la tua ultima config nota o qualsiasi backup.
* Un agente di coding locale può spesso ricostruire una config funzionante partendo dai log o dalla cronologia.

Per evitarlo:

* Usa `openclaw config set` per modifiche piccole.
* Usa `openclaw configure` per modifiche interattive.

Documentazione: [Config](/it/cli/config), [Configure](/it/cli/configure), [Doctor](/it/gateway/doctor).

<div id="whats-a-minimal-sane-config-for-a-first-install">
  ### Qual è la configurazione minima consigliata per una prima installazione
</div>

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
  channels: { whatsapp: { allowFrom: ["+15555550123"] } }
}
```

Questo imposta lo spazio di lavoro e limita chi può avviare il bot.

<div id="how-do-i-set-up-tailscale-on-a-vps-and-connect-from-my-mac">
  ### Come configuro Tailscale su un VPS e mi collego dal Mac
</div>

Procedura minima:

1. **Installa + effettua l’accesso sul VPS**
   ```bash
   curl -fsSL https://tailscale.com/install.sh | sh
   sudo tailscale up
   ```
2. **Installa + effettua l’accesso sul tuo Mac**
   * Usa l’app Tailscale ed esegui l’accesso alla stessa tailnet.
3. **Abilita MagicDNS (consigliato)**
   * Nella console di amministrazione di Tailscale, abilita MagicDNS in modo che il VPS abbia un nome stabile.
4. **Usa l’hostname della tailnet**
   * SSH: `ssh user@your-vps.tailnet-xxxx.ts.net`
   * Gateway WS: `ws://your-vps.tailnet-xxxx.ts.net:18789`

Se vuoi accedere alla Control UI senza usare SSH, utilizza Tailscale Serve sul VPS:

```bash
openclaw gateway --tailscale serve
```

Questo mantiene il Gateway vincolato all&#39;interfaccia di loopback ed espone HTTPS tramite Tailscale. Vedi [Tailscale](/it/gateway/tailscale).

<div id="how-do-i-connect-a-mac-node-to-a-remote-gateway-tailscale-serve">
  ### Come collegare un nodo Mac a un Gateway remoto con Tailscale Serve
</div>

Serve espone **Gateway Control UI + WS**. I nodi si connettono tramite lo stesso endpoint WS del Gateway.

Configurazione consigliata:

1. **Assicurati che il VPS e il Mac siano nella stessa tailnet**.
2. **Usa l&#39;app macOS in modalità Remote** (la destinazione SSH può essere l&#39;hostname della tailnet).
   L&#39;app effettuerà il tunneling della porta del Gateway e si connetterà come nodo.
3. **Approva il nodo** sul Gateway:
   ```bash
   openclaw nodes pending
   openclaw nodes approve <requestId>
   ```

Documentazione: [Gateway protocol](/it/gateway/protocol), [Discovery](/it/gateway/discovery), [macOS remote mode](/it/platforms/mac/remote).

<div id="env-vars-and-env-loading">
  ## Variabili d&#39;ambiente e caricamento da .env
</div>

<div id="how-does-openclaw-load-environment-variables">
  ### Come OpenClaw carica le variabili d&#39;ambiente
</div>

OpenClaw legge le variabili d&#39;ambiente dal processo padre (shell, launchd/systemd, CI, ecc.) e carica inoltre:

* `.env` dalla directory di lavoro corrente
* un `.env` globale di fallback da `~/.openclaw/.env` (ovvero `$OPENCLAW_STATE_DIR/.env`)

Nessuno dei file `.env` sovrascrive le variabili d&#39;ambiente esistenti.

Puoi anche definire variabili d&#39;ambiente inline nella configurazione (applicate solo se mancanti nell&#39;env del processo):

```json5
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: { GROQ_API_KEY: "gsk-..." }
  }
}
```

Consulta [/environment](/it/environment) per l&#39;ordine di precedenza completo e le relative fonti.

<div id="i-started-the-gateway-via-the-service-and-my-env-vars-disappeared-what-now">
  ### Ho avviato il Gateway come servizio e le mie variabili d&#39;ambiente sono sparite. E adesso?
</div>

Due soluzioni comuni:

1. Inserisci le chiavi mancanti in `~/.openclaw/.env` così vengono lette anche quando il servizio non eredita l&#39;ambiente della tua shell.
2. Abilita l&#39;importazione dalla shell (comoda funzionalità opzionale, opt‑in):

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

Questo esegue la tua shell di login e importa solo le chiavi di ambiente attese che risultano mancanti (non le sovrascrive mai). Variabili d&#39;ambiente equivalenti:
`OPENCLAW_LOAD_SHELL_ENV=1`, `OPENCLAW_SHELL_ENV_TIMEOUT_MS=15000`.

<div id="i-set-copilotgithubtoken-but-models-status-shows-shell-env-off-why">
  ### Ho impostato COPILOTGITHUBTOKEN ma lo stato dei modelli mostra “Shell env: off”. Perché?
</div>

`openclaw models status` indica se **shell env import** è abilitato. “Shell env: off”
**non** significa che le tue variabili d’ambiente mancano: significa solo che OpenClaw non carica
automaticamente la tua shell di login.

Se il Gateway viene eseguito come servizio (launchd/systemd), non erediterà
il tuo ambiente shell. Puoi risolvere in uno di questi modi:

1. Metti il token in `~/.openclaw/.env`:
   ```
   COPILOT_GITHUB_TOKEN=...
   ```
2. Oppure abilita l’import dell’ambiente della shell (`env.shellEnv.enabled: true`).
3. Oppure aggiungilo al blocco `env` della tua configurazione (si applica solo se mancante).

Quindi riavvia il Gateway e controlla di nuovo:

```bash
openclaw models status
```

I token di Copilot vengono letti da `COPILOT_GITHUB_TOKEN` (oppure da `GH_TOKEN` / `GITHUB_TOKEN`).
Consulta [/concepts/model-providers](/it/concepts/model-providers) e [/environment](/it/environment).

<div id="sessions-multiple-chats">
  ## Sessioni e chat multiple
</div>

<div id="how-do-i-start-a-fresh-conversation">
  ### Come avviare una nuova conversazione
</div>

Invia `/new` o `/reset` come messaggio a sé stante. Consulta [Gestione delle sessioni](/it/concepts/session).

<div id="do-sessions-reset-automatically-if-i-never-send-new">
  ### Le sessioni si reimpostano automaticamente se non invio mai nuovi messaggi
</div>

Sì. Le sessioni scadono dopo `session.idleMinutes` (valore predefinito **60**). Il **messaggio successivo**
avvia un nuovo ID di sessione per quella chiave di chat. Questo non elimina
le trascrizioni: avvia semplicemente una nuova sessione.

```json5
{
  session: {
    idleMinutes: 240
  }
}
```

<div id="is-there-a-way-to-make-a-team-of-openclaw-instances-one-ceo-and-many-agents">
  ### Esiste un modo per avere in un team di istanze OpenClaw un “CEO” e molti agenti
</div>

Sì, tramite **instradamento multi‑agente** e **sotto‑agenti**. Puoi creare un agente
coordinatore e diversi agenti esecutori con i propri spazi di lavoro e modelli.

Detto questo, è meglio considerarlo un **esperimento divertente**. È molto dispendioso in termini di token ed è spesso
meno efficiente rispetto all’uso di un solo bot con sessioni separate. Il modello tipico che
immaginiamo è un bot con cui interagisci, con sessioni diverse per il lavoro in parallelo. Quel
bot può anche generare sotto‑agenti quando necessario.

Documentazione: [Instradamento multi‑agente](/it/concepts/multi-agent), [Sotto‑agenti](/it/tools/subagents), [Agents CLI](/it/cli/agents).

<div id="why-did-context-get-truncated-midtask-how-do-i-prevent-it">
  ### Perché il contesto è stato troncato a metà attività? Come posso evitarlo
</div>

Il contesto di sessione è limitato dalla finestra del modello. Chat lunghe, output voluminosi degli strumenti o molti
file possono attivare la compattazione o il troncamento.

Cosa puoi fare:

* Chiedi al bot di riassumere lo stato attuale e di scriverlo in un file.
* Usa `/compact` prima di attività lunghe e `/new` quando cambi argomento.
* Tieni il contesto importante nello spazio di lavoro e chiedi al bot di rileggerlo.
* Usa sottoagenti per lavori lunghi o in parallelo, così la chat principale rimane più piccola.
* Scegli un modello con una finestra di contesto più ampia se questo succede spesso.

<div id="how-do-i-completely-reset-openclaw-but-keep-it-installed">
  ### Come posso ripristinare completamente OpenClaw senza disinstallarlo
</div>

Usa il comando reset:

```bash
openclaw reset
```

Reset completo non interattivo:

```bash
openclaw reset --scope full --yes --non-interactive
```

Quindi riesegui la procedura di onboarding:

```bash
openclaw onboard --install-daemon
```

Note:

* Anche la procedura guidata di onboarding offre **Reset** se rileva una configurazione esistente. Consulta [Wizard](/it/start/wizard).
* Se hai utilizzato dei profili (`--profile` / `OPENCLAW_PROFILE`), azzera ogni directory di stato (i valori predefiniti sono `~/.openclaw-<profile>`).
* Reset di sviluppo: `openclaw gateway --dev --reset` (solo sviluppo; elimina configurazione di sviluppo + credenziali + sessioni + spazio di lavoro).

<div id="im-getting-context-too-large-errors-how-do-i-reset-or-compact">
  ### Ricevo errori di contesto troppo grande: come faccio a reimpostare o compattare?
</div>

Usa una di queste opzioni:

* **Compact** (mantiene la conversazione ma riassume i messaggi più vecchi):
  ```
  /compact
  ```
  oppure `/compact <instructions>` per guidare il riassunto.

* **Reset** (nuovo ID di sessione per la stessa chiave di chat):
  ```
  /new
  /reset
  ```

Se continua a succedere:

* Abilita o regola il **session pruning** (`agents.defaults.contextPruning`) per eliminare l&#39;output degli strumenti più vecchio.
* Usa un modello con una finestra di contesto più ampia.

Documentazione: [Compaction](/it/concepts/compaction), [Session pruning](/it/concepts/session-pruning), [Gestione delle sessioni](/it/concepts/session).

<div id="why-am-i-seeing-llm-request-rejected-messagesncontentxtooluseinput-field-required">
  ### Perché vedo messaggi di richiesta LLM rifiutata messagesNcontentXtooluseinput Field required
</div>

Si tratta di un errore di validazione del provider: il modello ha emesso un blocco `tool_use` senza l’`input` obbligatorio. Di solito significa che la cronologia della sessione è obsoleta o danneggiata (spesso dopo thread molto lunghi o dopo una modifica di tool/schema).

Soluzione: avvia una nuova sessione con `/new` (come messaggio singolo).

<div id="why-am-i-getting-heartbeat-messages-every-30-minutes">
  ### Perché ricevo messaggi di heartbeat ogni 30 minuti
</div>

Gli heartbeat vengono eseguiti ogni **30m** per impostazione predefinita. Puoi modificarne la frequenza o disattivarli:

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "2h"   // oppure "0m" per disabilitarlo
      }
    }
  }
}
```

Se `HEARTBEAT.md` esiste ma è di fatto vuoto (solo righe vuote e intestazioni
markdown come `# Heading`), OpenClaw salta l&#39;esecuzione dell&#39;heartbeat per
risparmiare chiamate api. Se il file manca, l&#39;heartbeat viene comunque eseguito
e il modello decide cosa fare.

Le impostazioni specifiche per singolo agente usano `agents.list[].heartbeat`. Documentazione: [Heartbeat](/it/gateway/heartbeat).

<div id="do-i-need-to-add-a-bot-account-to-a-whatsapp-group">
  ### Devo aggiungere un account bot a un gruppo WhatsApp
</div>

No. OpenClaw viene eseguito **sul tuo account**, quindi se sei nel gruppo OpenClaw può vederlo.
Per impostazione predefinita, le risposte nel gruppo sono bloccate finché non autorizzi i mittenti (`groupPolicy: "allowlist"`).

Se vuoi che solo **tu** possa attivare le risposte nel gruppo:

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
  ### Come ottenere il JID di un gruppo WhatsApp
</div>

Opzione 1 (la più veloce): esegui tail sui log e Invia un messaggio di prova nel gruppo:

```bash
openclaw logs --follow --json
```

Cerca `chatId` (o `from`) che termina con `@g.us`, ad esempio:
`1234567890-1234567890@g.us`.

Opzione 2 (se già configurato/in lista di autorizzati): elenca i gruppi dalla configurazione:

```bash
openclaw directory groups list --channel whatsapp
```

Documentazione: [WhatsApp](/it/channels/whatsapp), [Directory](/it/cli/directory), [Logs](/it/cli/logs).

<div id="why-doesnt-openclaw-reply-in-a-group">
  ### Perché OpenClaw non risponde in un gruppo
</div>

Due cause comuni:

* Il controllo tramite @mention è attivo (impostazione predefinita). Devi menzionare il bot con @ (o corrispondere a `mentionPatterns`).
* Hai configurato `channels.whatsapp.groups` senza `"*"` e il gruppo non è nella lista di autorizzati.

Vedi [Gruppi](/it/concepts/groups) e [Messaggi di gruppo](/it/concepts/group-messages).

<div id="do-groupsthreads-share-context-with-dms">
  ### I gruppi e i thread condividono il contesto con i DM
</div>

Le chat dirette confluiscono nella sessione principale per impostazione predefinita. I gruppi/canali hanno le proprie chiavi di sessione e i topic di Telegram/i thread di Discord sono sessioni separate. Consulta [Gruppi](/it/concepts/groups) e [Messaggi di gruppo](/it/concepts/group-messages).

<div id="how-many-workspaces-and-agents-can-i-create">
  ### Quanti spazi di lavoro e agenti posso creare
</div>

Non ci sono limiti rigidi. Decine (anche centinaia) vanno bene, ma fai attenzione a:

* **Crescita dello spazio su disco:** le sessioni + le trascrizioni si trovano in `~/.openclaw/agents/<agentId>/sessions/`.
* **Costo dei token:** più agenti significa un utilizzo del modello più concorrente.
* **Overhead operativo:** profili di autenticazione per agente, spazi di lavoro e instradamento dei canali.

Suggerimenti:

* Mantieni uno spazio di lavoro **attivo** per agente (`agents.defaults.workspace`).
* Elimina le vecchie sessioni (cancella i file JSONL o archivia le relative voci) se l&#39;utilizzo del disco aumenta.
* Usa `openclaw doctor` per individuare spazi di lavoro orfani e incongruenze nei profili.

<div id="can-i-run-multiple-bots-or-chats-at-the-same-time-slack-and-how-should-i-set-that-up">
  ### Posso eseguire più bot o chat contemporaneamente in Slack e come dovrei configurarli
</div>

Sì. Usa **Multi‑Agent Routing** per eseguire più agenti isolati e instradare i messaggi in ingresso in base a
canale/account/interlocutore. Slack è supportato come canale e può essere associato ad agenti specifici.

L’accesso al browser è potente ma non “può fare qualsiasi cosa che possa fare un umano” — sistemi anti‑bot, CAPTCHA e MFA possono
ancora bloccare l’automazione. Per il controllo del browser più affidabile, usa il relay dell’estensione Chrome
sulla macchina che esegue il browser (e tieni il Gateway dove preferisci).

Configurazione consigliata:

* Host Gateway sempre attivo (VPS/Mac mini).
* Un agente per ruolo (binding/associazione).
* Canali Slack associati a quegli agenti.
* Browser locale tramite relay dell’estensione (o un nodo) quando necessario.

Documentazione: [Multi‑Agent Routing](/it/concepts/multi-agent), [Slack](/it/channels/slack),
[Browser](/it/tools/browser), [Chrome extension](/it/tools/chrome-extension), [Nodes](/it/nodes).

<div id="models-defaults-selection-aliases-switching">
  ## Modelli: predefiniti, selezione, alias e passaggio
</div>

<div id="what-is-the-default-model">
  ### Qual è il modello predefinito
</div>

Il modello predefinito di OpenClaw è quello che imposti come:

```
agents.defaults.model.primary
```

I modelli sono indicati nel formato `provider/model` (esempio: `anthropic/claude-opus-4-5`). Se ometti il provider, OpenClaw al momento considera `anthropic` come fallback temporaneo per la deprecazione, ma dovresti comunque specificare **esplicitamente** `provider/model`.

<div id="what-model-do-you-recommend">
  ### Quale modello consigliate
</div>

**Predefinito consigliato:** `anthropic/claude-opus-4-5`.\
**Buona alternativa:** `anthropic/claude-sonnet-4-5`.\
**Affidabile (meno personalità):** `openai/gpt-5.2` - quasi all’altezza di Opus, solo con meno personalità.\
**Economico:** `zai/glm-4.7`.

MiniMax M2.1 ha una propria documentazione: [MiniMax](/it/providers/minimax) e
[Modelli locali](/it/gateway/local-models).

Regola generale: usa il **miglior modello che puoi permetterti** per il lavoro critico e un modello più economico per chat di routine o riepiloghi. Puoi instradare i modelli per agente e usare sotto-agenti per
parallelizzare attività lunghe (ogni sotto-agente consuma token). Vedi [Modelli](/it/concepts/models) e
[Sotto-agenti](/it/tools/subagents).

Avviso importante: i modelli più deboli / eccessivamente quantizzati sono più vulnerabili alla prompt
injection e a comportamenti non sicuri. Vedi [Sicurezza](/it/gateway/security).

Ulteriori dettagli: [Modelli](/it/concepts/models).

<div id="can-i-use-selfhosted-models-llamacpp-vllm-ollama">
  ### Posso usare modelli self-hosted llamacpp vLLM Ollama
</div>

Sì. Se il tuo server locale espone un&#39;API compatibile con OpenAI, puoi
configurare un provider personalizzato che punti a quel server. Ollama è
supportato direttamente ed è il percorso più semplice.

Nota sulla sicurezza: i modelli più piccoli o pesantemente quantizzati sono più
vulnerabili alla prompt injection. Raccomandiamo vivamente **modelli di grandi
dimensioni** per qualsiasi bot che possa usare tool. Se vuoi comunque usare
modelli piccoli, abilita il sandboxing e liste di autorizzati rigorose per i tool.

Documentazione: [Ollama](/it/providers/ollama), [Modelli locali](/it/gateway/local-models),
[Provider di modelli](/it/concepts/model-providers), [Sicurezza](/it/gateway/security),
[Sandboxing](/it/gateway/sandboxing).

<div id="how-do-i-switch-models-without-wiping-my-config">
  ### Come cambio modello senza azzerare la mia configurazione
</div>

Usa i **comandi del modello** o modifica solo i campi **model**. Evita di sostituire l&#39;intera configurazione.

Opzioni sicure:

* `/model` in chat (veloce, per singola sessione)
* `openclaw models set ...` (aggiorna solo la configurazione del modello)
* `openclaw configure --section models` (interattivo)
* modifica `agents.defaults.model` in `~/.openclaw/openclaw.json`

Evita `config.apply` con un oggetto parziale a meno che tu non voglia sostituire l&#39;intera configurazione.
Se hai sovrascritto la configurazione, ripristina da un backup o riesegui `openclaw doctor` per riparare.

Documentazione: [Models](/it/concepts/models), [Configure](/it/cli/configure), [Config](/it/cli/config), [Doctor](/it/gateway/doctor).

<div id="what-do-openclaw-flawd-and-krill-use-for-models">
  ### Quali modelli utilizzano OpenClaw, Flawd e Krill
</div>

* **OpenClaw + Flawd:** Anthropic Opus (`anthropic/claude-opus-4-5`) - vedi [Anthropic](/it/providers/anthropic).
* **Krill:** MiniMax M2.1 (`minimax/MiniMax-M2.1`) - vedi [MiniMax](/it/providers/minimax).

<div id="how-do-i-switch-models-on-the-fly-without-restarting">
  ### Come faccio a passare da un modello all&#39;altro al volo senza riavviare
</div>

Usa il comando `/model` come messaggio a sé stante:

```
/model sonnet
/model haiku
/model opus
/model gpt
/model gpt-mini
/model gemini
/model gemini-flash
```

Puoi visualizzare l&#39;elenco dei modelli disponibili con `/model`, `/model list` o `/model status`.

`/model` (e `/model list`) mostra un selettore compatto numerato. Seleziona il modello tramite il numero:

```
/model 3
```

Puoi anche imporre un profilo di autenticazione specifico per il provider (per sessione):

```
/model opus@anthropic:default
/model opus@anthropic:work
```

Suggerimento: `/model status` mostra quale agente è attivo, quale file `auth-profiles.json` viene utilizzato e quale profilo di autenticazione verrà utilizzato successivamente.
Mostra anche l&#39;endpoint del provider configurato (`baseUrl`) e la modalità API (`api`) se disponibile.

**Come sblocco un profilo che ho impostato con profile**

Esegui di nuovo `/model` **senza** il suffisso `@profile`:

```
/model anthropic/claude-opus-4-5
```

Se vuoi tornare al valore predefinito, selezionalo da `/model` (o invia `/model &lt;default provider/model&gt;`).
Usa `/model status` per verificare quale profilo di autenticazione è attivo.

<div id="can-i-use-gpt-52-for-daily-tasks-and-codex-52-for-coding">
  ### Posso usare GPT 5.2 per le attività quotidiane e Codex 5.2 per la programmazione
</div>

Sì. Impostane uno come predefinito e passa all&#39;altro quando serve:

* **Cambio rapido (per sessione):** `/model gpt-5.2` per le attività quotidiane, `/model gpt-5.2-codex` per la programmazione.
* **Predefinito + cambio:** imposta `agents.defaults.model.primary` su `openai-codex/gpt-5.2`, poi passa a `openai-codex/gpt-5.2-codex` quando programmi (o viceversa).
* **Sub-agenti:** instrada le attività di programmazione verso sub-agenti con un modello predefinito diverso.

Consulta [Modelli](/it/concepts/models) e [Slash commands](/it/tools/slash-commands).

<div id="why-do-i-see-model-is-not-allowed-and-then-no-reply">
  ### Perché vedo Model is not allowed e poi nessuna risposta
</div>

Se `agents.defaults.models` è impostato, funge da **lista di autorizzati** per `/model` e per qualsiasi
override di sessione. Scegliere un modello che non è in quella lista genera:

```
Model "provider/model" is not allowed. Use /model to list available models.
```

Quell&#39;errore viene restituito **anziché** una normale risposta. Soluzione: aggiungi il modello a
`agents.defaults.models`, rimuovi la lista di autorizzati oppure scegli un modello da `/model list`.

<div id="why-do-i-see-unknown-model-minimaxminimaxm21">
  ### Perché vedo Unknown model minimaxMiniMaxM21
</div>

Questo significa che il **provider non è configurato** (non è stata trovata alcuna
configurazione o alcun profilo di autenticazione del provider MiniMax), quindi il
modello non può essere risolto. Una correzione per questo problema di rilevamento è inclusa
in **2026.1.12** (non ancora rilasciata al momento della stesura).

Checklist per la correzione:

1. Aggiorna a **2026.1.12** (o esegui dal sorgente `main`), quindi riavvia il Gateway.
2. Assicurati che MiniMax sia configurato (wizard o JSON), o che esista una
   MiniMax API key nei profili env/auth in modo che il provider possa essere iniettato.
3. Usa l’ID esatto del modello (case‑sensitive): `minimax/MiniMax-M2.1` o
   `minimax/MiniMax-M2.1-lightning`.
4. Esegui:
   ```bash
   openclaw models list
   ```
   e scegli dall’elenco (oppure `/model list` in chat).

Consulta [MiniMax](/it/providers/minimax) e [Models](/it/concepts/models).

<div id="can-i-use-minimax-as-my-default-and-openai-for-complex-tasks">
  ### Posso usare MiniMax come modello predefinito e OpenAI per i task complessi
</div>

Sì. Usa **MiniMax come modello predefinito** e cambia modello **per sessione** quando necessario.
I fallback servono per gli **errori**, non per i “task complessi”, quindi usa `/model` o un agente separato.

**Opzione A: cambia modello per sessione**

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

Poi:

```
/model gpt
```

**Opzione B: agenti separati**

* Agente A predefinito: MiniMax
* Agente B predefinito: OpenAI
* Instrada in base all’agente oppure usa `/agent` per cambiare

Documentazione: [Modelli](/it/concepts/models), [Instradamento multi‑agente](/it/concepts/multi-agent), [MiniMax](/it/providers/minimax), [OpenAI](/it/providers/openai).

<div id="are-opus-sonnet-gpt-builtin-shortcuts">
  ### Opus, sonnet e gpt sono scorciatoie integrate
</div>

Sì. OpenClaw include alcune scorciatoie predefinite (applicate solo quando il modello esiste in `agents.defaults.models`):

* `opus` → `anthropic/claude-opus-4-5`
* `sonnet` → `anthropic/claude-sonnet-4-5`
* `gpt` → `openai/gpt-5.2`
* `gpt-mini` → `openai/gpt-5-mini`
* `gemini` → `google/gemini-3-pro-preview`
* `gemini-flash` → `google/gemini-3-flash-preview`

Se definisci un tuo alias con lo stesso nome, ha la precedenza il valore che hai impostato.

<div id="how-do-i-defineoverride-model-shortcuts-aliases">
  ### Come definire o sovrascrivere scorciatoie e alias per i modelli
</div>

Gli alias provengono da `agents.defaults.models.<modelId>.alias`. Esempio:

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

Quindi `/model sonnet` (oppure `/<alias>` quando supportato) corrisponde a quell&#39;ID di modello.

<div id="how-do-i-add-models-from-other-providers-like-openrouter-or-zai">
  ### Come faccio ad aggiungere modelli da altri provider come OpenRouter o ZAI
</div>

OpenRouter (pagamento a consumo per token; molti modelli):

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

Z.AI (modelli GLM):

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

Se fai riferimento a un provider/modello ma la chiave del provider richiesta non è presente, otterrai un errore di autenticazione in fase di esecuzione (ad esempio `No API key found for provider "zai"`).

**No API key found for provider dopo aver aggiunto un nuovo agente**

Di solito significa che il **nuovo agente** ha un archivio di autenticazione vuoto. L’autenticazione è per agente ed è
memorizzata in:

```
~/.openclaw/agents/<agentId>/agent/auth-profiles.json
```

Opzioni per risolvere:

* Esegui `openclaw agents add <id>` e configura l&#39;autenticazione durante la procedura guidata.
* Oppure copia `auth-profiles.json` dall&#39;`agentDir` dell&#39;agente principale nell&#39;`agentDir` del nuovo agente.

**Non** riutilizzare `agentDir` tra agenti; provoca collisioni di autenticazione/sessione.

<div id="model-failover-and-all-models-failed">
  ## Failover dei modelli ed errore «All models failed»
</div>

<div id="how-does-failover-work">
  ### Come funziona il failover
</div>

Il failover avviene in due fasi:

1. **Rotazione del profilo di autenticazione** all&#39;interno dello stesso provider.
2. **Fallback del modello** al modello successivo in `agents.defaults.model.fallbacks`.

I periodi di cooldown si applicano ai profili non riusciti (backoff esponenziale), in modo che OpenClaw possa continuare a rispondere anche quando un provider è soggetto a rate‑limit o presenta errori temporanei.

<div id="what-does-this-error-mean">
  ### Che cosa significa questo errore
</div>

```
No credentials found for profile "anthropic:default"
```

Significa che il sistema ha provato a utilizzare l&#39;ID del profilo di autenticazione `anthropic:default`, ma non è riuscito a trovare le relative credenziali nell&#39;archivio di autenticazione previsto.

<div id="fix-checklist-for-no-credentials-found-for-profile-anthropicdefault">
  ### Checklist di risoluzione per No credentials found for profile anthropicdefault
</div>

* **Verifica dove si trovano i profili di autenticazione** (percorsi nuovi vs legacy)
  * Attuale: `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`
  * Legacy: `~/.openclaw/agent/*` (migrato da `openclaw doctor`)
* **Verifica che la variabile di ambiente sia caricata dal Gateway**
  * Se imposti `ANTHROPIC_API_KEY` nella tua shell ma esegui il Gateway tramite systemd/launchd, potrebbe non ereditarla. Mettila in `~/.openclaw/.env` oppure abilita `env.shellEnv`.
* **Assicurati di modificare l’agente corretto**
  * Configurazioni multi‑agente implicano che possano esistere più file `auth-profiles.json`.
* **Verifica di base lo stato di modelli/autenticazione**
  * Usa `openclaw models status` per vedere i modelli configurati e se i provider sono autenticati.

**Checklist di risoluzione per No credentials found for profile anthropic**

Questo significa che l’esecuzione è vincolata a un profilo di autenticazione Anthropic, ma il Gateway
non riesce a trovarlo nel proprio archivio di autenticazione.

* **Usa un setup-token**
  * Esegui `claude setup-token`, poi incollalo con `openclaw models auth setup-token --provider anthropic`.
  * Se il token è stato creato su un’altra macchina, usa `openclaw models auth paste-token --provider anthropic`.
* **Se invece vuoi usare una chiave API**
  * Metti `ANTHROPIC_API_KEY` in `~/.openclaw/.env` sull’**host del Gateway**.
  * Pulisci qualsiasi ordine fissato che impone un profilo mancante:
    ```bash
    openclaw models auth order clear --provider anthropic
    ```
* **Verifica di eseguire i comandi sull’host del Gateway**
  * In modalità remota, i profili di autenticazione risiedono sulla macchina del Gateway, non sul tuo laptop.

<div id="why-did-it-also-try-google-gemini-and-fail">
  ### Perché ha provato anche Google Gemini e non è riuscito
</div>

Se la configurazione del tuo modello include Google Gemini come fallback (o se sei passato a una scorciatoia Gemini), OpenClaw lo proverà durante il fallback del modello. Se non hai configurato le credenziali Google, vedrai `No API key found for provider "google"`.

Soluzione: fornisci l&#39;autenticazione Google oppure rimuovi/evita i modelli Google in `agents.defaults.model.fallbacks` / alias, in modo che il fallback non inoltri lì.

**Messaggio di richiesta LLM rifiutata che indica che è richiesta una firma per Google Antigravity**

Causa: la cronologia della sessione contiene **blocchi di thinking senza firma** (spesso da uno stream interrotto/parziale). Google Antigravity richiede firme per i blocchi di thinking.

Soluzione: OpenClaw ora rimuove i blocchi di thinking senza firma per Google Antigravity Claude. Se il problema si presenta ancora, avvia una **nuova sessione** oppure imposta `/thinking off` per quell&#39;agente.

<div id="auth-profiles-what-they-are-and-how-to-manage-them">
  ## Profili di autenticazione: cosa sono e come gestirli
</div>

Correlato: [/concepts/oauth](/it/concepts/oauth) (flussi OAuth, memorizzazione dei token, schemi multi-account)

<div id="what-is-an-auth-profile">
  ### Che cos&#39;è un profilo di autenticazione
</div>

Un profilo di autenticazione è un record di credenziali denominato (OAuth o chiave API) associato a un provider. I profili sono archiviati in:

```
~/.openclaw/agents/<agentId>/agent/auth-profiles.json
```

<div id="what-are-typical-profile-ids">
  ### Quali sono gli ID tipici dei profili
</div>

OpenClaw usa ID con prefisso del provider, ad esempio:

* `anthropic:default` (comune quando non esiste alcuna identità email)
* `anthropic:<email>` per le identità OAuth
* ID personalizzati a tua scelta (ad es. `anthropic:work`)

<div id="can-i-control-which-auth-profile-is-tried-first">
  ### Posso controllare quale profilo di autenticazione viene provato per primo
</div>

Sì. La configurazione supporta metadati opzionali per i profili e un ordinamento per provider (`auth.order.<provider>`). Questo **non** memorizza segreti; associa gli ID a provider/modalità e imposta l’ordine di rotazione.

OpenClaw può saltare temporaneamente un profilo se è in un breve stato di **cooldown** (rate limit/timeout/errori di autenticazione) o in uno stato di **disabled** più lungo (fatturazione/crediti insufficienti). Per verificarlo, esegui `openclaw models status --json` e controlla `auth.unusableProfiles`. Per la regolazione, vedi: `auth.cooldowns.billingBackoffHours*`.

Puoi anche impostare un override dell’ordine **per agente** (memorizzato in `auth-profiles.json` di quell’agente) tramite la CLI:

```bash
# Defaults to the configured default agent (omit --agent)
openclaw models auth order get --provider anthropic

# Lock rotation to a single profile (only try this one)
openclaw models auth order set --provider anthropic anthropic:default

# Or set an explicit order (fallback within provider)
openclaw models auth order set --provider anthropic anthropic:work anthropic:default

# Cancella l'override (ritorna a config auth.order / round-robin)
openclaw models auth order clear --provider anthropic
```

Per indirizzare un agente specifico:

```bash
openclaw models auth order set --provider anthropic --agent main anthropic:default
```

<div id="oauth-vs-api-key-whats-the-difference">
  ### OAuth vs API key: qual è la differenza
</div>

OpenClaw supporta entrambi:

* **OAuth** spesso sfrutta l’accesso tramite abbonamento (quando applicabile).
* Le **chiavi API** utilizzano una fatturazione pay‑per‑token.

Il wizard supporta esplicitamente Anthropic setup-token e OpenAI Codex OAuth e può memorizzare per te le chiavi API.

<div id="gateway-ports-already-running-and-remote-mode">
  ## Gateway: porte, “already running” e modalità remota
</div>

<div id="what-port-does-the-gateway-use">
  ### Quale porta usa il Gateway
</div>

`gateway.port` controlla l&#39;unica porta multiplexata per WebSocket + HTTP (Control UI, hook, ecc.).

Precedenza:

```
--port > OPENCLAW_GATEWAY_PORT > gateway.port > default 18789
```

<div id="why-does-openclaw-gateway-status-say-runtime-running-but-rpc-probe-failed">
  ### Perché `openclaw gateway status` mostra Runtime running ma RPC probe failed
</div>

Perché “running” è la vista del **supervisore** (launchd/systemd/schtasks). La probe RPC è la CLI che si connette effettivamente al WebSocket del Gateway e chiama `status`.

Usa `openclaw gateway status` e considera attendibili queste righe:

* `Probe target:` (l’URL che la probe ha effettivamente usato)
* `Listening:` (ciò che è effettivamente in ascolto su quella porta)
* `Last gateway error:` (causa principale tipica quando il processo è attivo ma la porta non è in ascolto)

<div id="why-does-openclaw-gateway-status-show-config-cli-and-config-service-different">
  ### Perché `openclaw gateway status` mostra Config CLI e Config service diversi
</div>

Stai modificando un file di configurazione mentre il servizio ne sta usando un altro (spesso a causa di una mancata corrispondenza tra `--profile` e `OPENCLAW_STATE_DIR`).

Soluzione:

```bash
openclaw gateway install --force
```

Esegui quel comando dallo stesso `--profile`/ambiente che vuoi usare per il servizio.

<div id="what-does-another-gateway-instance-is-already-listening-mean">
  ### Cosa significa &quot;another gateway instance is already listening&quot;
</div>

OpenClaw applica un lock di runtime effettuando immediatamente all&#39;avvio il binding del listener WebSocket (predefinito `ws://127.0.0.1:18789`). Se il binding fallisce con `EADDRINUSE`, genera `GatewayLockError` indicando che un&#39;altra istanza è già in ascolto.

Soluzione: arresta l&#39;altra istanza, libera la porta oppure esegui con `openclaw gateway --port <port>`.

<div id="how-do-i-run-openclaw-in-remote-mode-client-connects-to-a-gateway-elsewhere">
  ### Come eseguire OpenClaw in modalità remota con il client che si connette a un Gateway remoto
</div>

Configura `gateway.mode: "remote"` e punta a un URL WebSocket remoto, eventualmente specificando un token/password:

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

Note:

* `openclaw gateway` si avvia solo quando `gateway.mode` è `local` (a meno che tu non passi il flag di override).
* L&#39;app macOS monitora il file di configurazione e cambia modalità in tempo reale quando questi valori cambiano.

<div id="the-control-ui-says-unauthorized-or-keeps-reconnecting-what-now">
  ### La Control UI mostra “unauthorized” o continua a riconnettersi. E adesso?
</div>

Il tuo Gateway è in esecuzione con l’autenticazione abilitata (`gateway.auth.*`), ma la UI non sta inviando il token/password corrispondente.

Fatti (dal codice):

* La Control UI memorizza il token nella chiave localStorage del browser `openclaw.control.settings.v1`.
* La UI può importare `?token=...` (e/o `?password=...`) una sola volta, poi lo rimuove dall’URL.

Come risolvere:

* Metodo più rapido: `openclaw dashboard` (stampa e copia un link con token, prova ad aprirlo; mostra un suggerimento SSH se in modalità headless).
* Se non hai ancora un token: `openclaw doctor --generate-gateway-token`.
* Se il Gateway è remoto, crea prima un tunnel: `ssh -N -L 18789:127.0.0.1:18789 user@host` poi apri `http://127.0.0.1:18789/?token=...`.
* Imposta `gateway.auth.token` (o `OPENCLAW_GATEWAY_TOKEN`) sull’host del Gateway.
* Nelle impostazioni della Control UI, incolla lo stesso token (o aggiorna con un link monouso `?token=...`).
* Ancora bloccato? Esegui `openclaw status --all` e segui [Troubleshooting](/it/gateway/troubleshooting). Vedi [Dashboard](/it/web/dashboard) per i dettagli sull’autenticazione.

<div id="i-set-gatewaybind-tailnet-but-it-cant-bind-nothing-listens">
  ### Ho impostato gateway.bind su tailnet ma non riesce a effettuare il bind, nessun processo è in ascolto
</div>

Il bind `tailnet` sceglie un IP Tailscale dalle tue interfacce di rete (100.64.0.0/10). Se la macchina non è su Tailscale (o l’interfaccia non è attiva), non c’è nulla a cui fare il bind.

Soluzione:

* Avvia Tailscale su quell’host (così avrà un indirizzo 100.x), oppure
* Passa a `gateway.bind: "loopback"` / `"lan"`.

Nota: `tailnet` è esplicito. `auto` preferisce loopback; usa `gateway.bind: "tailnet"` quando vuoi un bind solo su tailnet.

<div id="can-i-run-multiple-gateways-on-the-same-host">
  ### Posso eseguire più Gateway sullo stesso host
</div>

Di solito no: un singolo Gateway può gestire più canali di messaggistica e agenti. Usa più Gateway solo quando ti serve ridondanza (es: bot di emergenza) o isolamento rigido.

Sì, ma devi isolarli:

* `OPENCLAW_CONFIG_PATH` (configurazione per istanza)
* `OPENCLAW_STATE_DIR` (stato per istanza)
* `agents.defaults.workspace` (isolamento dello spazio di lavoro)
* `gateway.port` (porte univoche)

Configurazione rapida (consigliata):

* Usa `openclaw --profile <name> …` per ogni istanza (crea automaticamente `~/.openclaw-<name>`).
* Imposta un valore `gateway.port` univoco nella configurazione di ogni profilo (oppure passa `--port` per esecuzioni manuali).
* Installa un servizio per ogni profilo: `openclaw --profile <name> gateway install`.

I profili aggiungono anche un suffisso ai nomi dei servizi (`bot.molt.<profile>`; legacy `com.openclaw.*`, `openclaw-gateway-<profile>.service`, `OpenClaw Gateway (<profile>)`).
Guida completa: [Multiple gateways](/it/gateway/multiple-gateways).

<div id="what-does-invalid-handshake-code-1008-mean">
  ### Cosa significa il codice di handshake non valido 1008
</div>

Il Gateway è un **server WebSocket** e si aspetta che il primo messaggio sia
un frame `connect`. Se riceve qualsiasi altra cosa, chiude la connessione
con il **codice 1008** (violazione dei criteri/policy).

Cause comuni:

* Hai aperto l’URL **HTTP** in un browser (`http://...`) invece che in un client WS.
* Hai usato la porta o il path sbagliato.
* Un proxy o un tunnel ha rimosso gli header di autenticazione o ha inviato una richiesta non destinata al Gateway.

Soluzioni rapide:

1. Usa l’URL WS: `ws://<host>:18789` (o `wss://...` se HTTPS).
2. Non aprire la porta WS in una normale scheda del browser.
3. Se l’autenticazione è attiva, includi il token/password nel frame `connect`.

Se stai usando la CLI o la TUI, l’URL dovrebbe avere questo aspetto:

```
openclaw tui --url ws://<host>:18789 --token <token>
```

Dettagli sul protocollo: [Protocollo del Gateway](/it/gateway/protocol).

<div id="logging-and-debugging">
  ## Log e debug
</div>

<div id="where-are-logs">
  ### Dove si trovano i log
</div>

Log su file (in formato strutturato):

```
/tmp/openclaw/openclaw-YYYY-MM-DD.log
```

Puoi impostare un percorso fisso per il file di log tramite `logging.file`. Il livello di log del file è controllato da `logging.level`. La verbosità della console è controllata da `--verbose` e `logging.consoleLevel`.

Tail dei log più rapido:

```bash
openclaw logs --follow
```

Log del servizio/supervisor (quando il Gateway viene eseguito tramite launchd/systemd):

* macOS: `$OPENCLAW_STATE_DIR/logs/gateway.log` e `gateway.err.log` (predefinito: `~/.openclaw/logs/...`; i profili usano `~/.openclaw-<profile>/logs/...`)
* Linux: `journalctl --user -u openclaw-gateway[-<profile>].service -n 200 --no-pager`
* Windows: `schtasks /Query /TN "OpenClaw Gateway (<profile>)" /V /FO LIST`

Consulta [Troubleshooting](/it/gateway/troubleshooting#log-locations) per ulteriori dettagli.

<div id="how-do-i-startstoprestart-the-gateway-service">
  ### Come faccio ad avviare, arrestare o riavviare il servizio Gateway?
</div>

Utilizza gli helper del Gateway:

```bash
openclaw gateway status
openclaw gateway restart
```

Se esegui il Gateway manualmente, puoi usare `openclaw gateway --force` per riacquisire la porta. Consulta [Gateway](/it/gateway).

<div id="i-closed-my-terminal-on-windows-how-do-i-restart-openclaw">
  ### Ho chiuso il terminale su Windows, come faccio a riavviare OpenClaw?
</div>

Esistono **due modalità di installazione su Windows**:

**1) WSL2 (consigliato):** il Gateway viene eseguito all&#39;interno di Linux.

Apri PowerShell, entra in WSL e poi riavvia:

```powershell
wsl
openclaw gateway status
openclaw gateway restart
```

Se non hai ancora installato il servizio, avvialo in foreground:

```bash
openclaw gateway run
```

**2) Windows nativo (non consigliato):** il Gateway viene eseguito direttamente su Windows.

Apri PowerShell ed esegui:

```powershell
openclaw gateway status
openclaw gateway restart
```

Se lo esegui manualmente (non come servizio), usa:

```powershell
openclaw gateway run
```

Documentazione: [Windows (WSL2)](/it/platforms/windows), [runbook del servizio Gateway](/it/gateway).

<div id="the-gateway-is-up-but-replies-never-arrive-what-should-i-check">
  ### Il Gateway è attivo ma le risposte non arrivano mai. Cosa devo controllare?
</div>

Inizia con una rapida verifica dello stato:

```bash
openclaw status
openclaw models status
openclaw channels status
openclaw logs --follow
```

Cause comuni:

* Autenticazione del modello non caricata sull&#39;**host del Gateway** (controlla `models status`).
* Abbinamento del canale o lista di autorizzati che bloccano le risposte (controlla la configurazione del canale e i log).
* WebChat/Dashboard è aperta senza il token corretto.

Se stai accedendo da remoto, verifica che il tunnel/Tailscale sia attivo e che il
WebSocket del Gateway sia raggiungibile.

Documentazione: [Canali](/it/channels), [Risoluzione dei problemi](/it/gateway/troubleshooting), [Accesso remoto](/it/gateway/remote).

<div id="disconnected-from-gateway-no-reason-what-now">
  ### Disconnesso dal Gateway senza motivo: e adesso?
</div>

Di solito significa che la UI ha perso la connessione WebSocket. Verifica:

1. Il Gateway è in esecuzione? `openclaw gateway status`
2. Il Gateway funziona correttamente? `openclaw status`
3. La UI ha il token corretto? `openclaw dashboard`
4. Se è in remoto, il tunnel/collegamento Tailscale è attivo?

Poi esegui il tail dei log:

```bash
openclaw logs --follow
```

Documentazione: [Dashboard](/it/web/dashboard), [Accesso remoto](/it/gateway/remote), [Risoluzione dei problemi](/it/gateway/troubleshooting).

<div id="telegram-setmycommands-fails-with-network-errors-what-should-i-check">
  ### setMyCommands di Telegram non riesce a causa di errori di rete Cosa devo controllare
</div>

Parti dai log e dallo stato del canale:

```bash
openclaw channels status
openclaw channels logs --channel telegram
```

Se sei su un VPS o dietro un proxy, verifica che le connessioni HTTPS in uscita siano consentite e che il DNS funzioni.
Se il Gateway è remoto, assicurati di controllare i log sull&#39;host del Gateway.

Documentazione: [Telegram](/it/channels/telegram), [Risoluzione dei problemi per i canali](/it/channels/troubleshooting).

<div id="tui-shows-no-output-what-should-i-check">
  ### La TUI non mostra alcun output: cosa devo controllare?
</div>

Per prima cosa verifica che il Gateway sia raggiungibile e che l’agente possa essere avviato:

```bash
openclaw status
openclaw models status
openclaw logs --follow
```

Nella TUI, usa `/status` per vedere lo stato attuale. Se ti aspetti risposte in un canale di chat, assicurati che il recapito dei messaggi sia abilitato (`/deliver on`).

Documentazione: [TUI](/it/tui), [Comandi slash](/it/tools/slash-commands).

<div id="how-do-i-completely-stop-then-start-the-gateway">
  ### Come arrestare completamente e riavviare il Gateway
</div>

Se hai installato il servizio:

```bash
openclaw gateway stop
openclaw gateway start
```

Questo arresta/avvia il **servizio supervisionato** (launchd su macOS, systemd su Linux).
Usalo quando il Gateway è in esecuzione in background come demone.

Se lo stai eseguendo in primo piano, interrompilo con Ctrl‑C, quindi:

```bash
openclaw gateway run
```

Documentazione: [Runbook del servizio Gateway](/it/gateway).

<div id="eli5-openclaw-gateway-restart-vs-openclaw-gateway">
  ### ELI5 differenza tra `openclaw gateway restart` e `openclaw gateway`
</div>

* `openclaw gateway restart`: riavvia il **servizio in background** (launchd/systemd).
* `openclaw gateway`: esegue il Gateway **in primo piano** per questa sessione del terminale.

Se hai installato il servizio, usa i comandi `openclaw gateway`. Usa `openclaw gateway` quando
vuoi un&#39;esecuzione singola, in primo piano.

<div id="whats-the-fastest-way-to-get-more-details-when-something-fails">
  ### Qual è il modo più rapido per ottenere maggiori dettagli quando qualcosa va storto?
</div>

Avvia il Gateway con `--verbose` per ottenere più dettagli in console. Quindi controlla il file di log per individuare eventuali errori di autenticazione dei canali, instradamento dei modelli e RPC.

<div id="media-attachments">
  ## Contenuti multimediali e allegati
</div>

<div id="my-skill-generated-an-imagepdf-but-nothing-was-sent">
  ### Il mio skill ha generato un imagePDF ma poi non è stato inviato nulla
</div>

Gli allegati in uscita dall&#39;agente devono includere una riga `MEDIA:&lt;path-or-url&gt;` (su una riga separata). Consulta [Configurazione dell&#39;assistente OpenClaw](/it/start/openclaw) e [Invio agente](/it/tools/agent-send).

Invio tramite CLI:

```bash
openclaw message send --target +15555550123 --message "Ecco a te" --media /path/to/file.png
```

Verifica anche:

* Il canale di destinazione supporta l’invio di contenuti multimediali in uscita e non è soggetto a restrizioni dalla lista di autorizzati.
* Il file rientra nei limiti di dimensione del provider (le immagini vengono ridimensionate fino a un massimo di 2048 px).

Vedi [Images](/it/nodes/images).

<div id="security-and-access-control">
  ## Sicurezza e controllo degli accessi
</div>

<div id="is-it-safe-to-expose-openclaw-to-inbound-dms">
  ### È sicuro esporre OpenClaw ai DM in ingresso
</div>

Considera i DM in ingresso come input non attendibile. I valori predefiniti sono progettati per ridurre il rischio:

* Il comportamento predefinito sui canali che supportano i DM è l’**abbinamento**:
  * I mittenti sconosciuti ricevono un codice di abbinamento; il bot non elabora il loro messaggio.
  * Approva con: `openclaw pairing approve <channel> <code>`
  * Le richieste in sospeso sono limitate a **3 per canale**; controlla `openclaw pairing list <channel>` se un codice non è arrivato.
* Aprire i DM al pubblico richiede un opt‑in esplicito (`dmPolicy: "open"` e allowlist `"*"`).

Esegui `openclaw doctor` per individuare policy DM potenzialmente rischiose.

<div id="is-prompt-injection-only-a-concern-for-public-bots">
  ### La prompt injection è un problema solo per i bot pubblici
</div>

No. La prompt injection riguarda i **contenuti non attendibili**, non solo chi può inviare DM al bot.
Se il tuo assistente legge contenuti esterni (web search/fetch, pagine del browser, email,
documenti, allegati, log incollati), quei contenuti possono includere istruzioni che cercano
di prendere il controllo del modello. Questo può succedere anche se **sei l&#39;unico mittente**.

Il rischio maggiore si presenta quando gli strumenti sono abilitati: il modello può essere indotto
a esfiltrare contesto o a chiamare strumenti per conto tuo. Riduci il potenziale impatto:

* usando un agente &quot;reader&quot; in sola lettura o con strumenti disabilitati per riassumere contenuti non attendibili
* mantenendo `web_search` / `web_fetch` / `browser` disattivati per gli agenti con strumenti abilitati
* usando sandbox e liste di autorizzati rigorose per gli strumenti

Dettagli: [Security](/it/gateway/security).

<div id="should-my-bot-have-its-own-email-github-account-or-phone-number">
  ### Il mio bot dovrebbe avere un proprio account email, GitHub o numero di telefono
</div>

Sì, nella maggior parte delle configurazioni. Isolare il bot con account e numeri
di telefono separati riduce la portata dei danni se qualcosa va storto. Questo rende
anche più semplice ruotare le credenziali o revocare l&#39;accesso senza influire sui tuoi account personali.

Parti in piccolo. Concedi l&#39;accesso solo agli strumenti e agli account di cui hai effettivamente bisogno e amplia
in seguito se necessario.

Documentazione: [Sicurezza](/it/gateway/security), [Abbinamento](/it/start/pairing).

<div id="can-i-give-it-autonomy-over-my-text-messages-and-is-that-safe">
  ### Posso darle autonomia sui miei messaggi di testo ed è sicuro?
</div>

**Non** consigliamo di darle piena autonomia sui tuoi messaggi personali. Il modello più sicuro è:

* Tieni i DM in **modalità di abbinamento** o con una lista di autorizzati molto ristretta.
* Usa un **numero o account separato** se vuoi che invii messaggi per tuo conto.
* Lasciale preparare le bozze, poi **approvale prima dell&#39;invio**.

Se vuoi fare esperimenti, fallo su un account dedicato e tienilo isolato. Vedi
[Sicurezza](/it/gateway/security).

<div id="can-i-use-cheaper-models-for-personal-assistant-tasks">
  ### Posso usare modelli più economici per attività di assistente personale
</div>

Sì, **se** l&#39;agente è limitato alla chat e l&#39;input è considerato attendibile. I modelli di fascia inferiore sono più suscettibili al dirottamento delle istruzioni, quindi evitali per agenti con strumenti abilitati o quando leggi contenuti non attendibili. Se devi usare un modello più piccolo, limita rigorosamente l’accesso agli strumenti ed esegui l’agente all&#39;interno di una sandbox. Vedi [Sicurezza](/it/gateway/security).

<div id="i-ran-start-in-telegram-but-didnt-get-a-pairing-code">
  ### Ho eseguito /start in Telegram ma non ho ricevuto un codice di abbinamento
</div>

I codici di abbinamento vengono inviati **solo** quando un mittente sconosciuto scrive al bot e
`dmPolicy: "pairing"` è attivata. `/start` da solo non genera un codice.

Controlla le richieste in sospeso:

```bash
openclaw pairing list telegram
```

Se vuoi avere accesso immediato, aggiungi il tuo ID mittente alla lista di autorizzati oppure imposta `dmPolicy: "open"` (impostazione che permette di accettare messaggi da chiunque)
per quell&#39;account.

<div id="whatsapp-will-it-message-my-contacts-how-does-pairing-work">
  ### WhatsApp invierà messaggi ai miei contatti? Come funziona l&#39;abbinamento
</div>

No. La policy DM predefinita di WhatsApp è **abbinamento**. I mittenti sconosciuti ricevono solo un codice di abbinamento e il loro messaggio **non viene elaborato**. OpenClaw risponde solo alle chat che riceve o agli invii espliciti che attivi tu.

Approva l&#39;abbinamento con:

```bash
openclaw pairing approve whatsapp <code>
```

Elenca le richieste pendenti:

```bash
openclaw pairing list whatsapp
```

Prompt del numero di telefono della procedura guidata: viene usato per impostare la tua **lista di autorizzati/proprietario** in modo che i tuoi messaggi diretti siano autorizzati. Non viene usato per l’invio automatico. Se esegui sul tuo numero WhatsApp personale, usa quel numero e abilita `channels.whatsapp.selfChatMode`.

<div id="chat-commands-aborting-tasks-and-it-wont-stop">
  ## Comandi della chat, annullamento delle attività e quando «non si ferma»
</div>

<div id="how-do-i-stop-internal-system-messages-from-showing-in-chat">
  ### Come faccio a impedire che i messaggi di sistema interni vengano visualizzati in chat
</div>

La maggior parte dei messaggi interni o relativi agli strumenti viene visualizzata solo quando **verbose** o **reasoning** sono abilitati
per quella sessione.

Intervieni nella chat in cui li vedi:

```
/verbose off
/reasoning off
```

Se è ancora troppo rumoroso, controlla le impostazioni della sessione nella Control UI e imposta verbose su **inherit**. Verifica inoltre di non utilizzare un profilo bot con `verboseDefault` impostato su `on` nella configurazione.

Documentazione: [Thinking and verbose](/it/tools/thinking), [Security](/it/gateway/security#reasoning--verbose-output-in-groups).

<div id="how-do-i-stopcancel-a-running-task">
  ### Come posso interrompere o annullare un&#39;attività in esecuzione
</div>

Invia uno qualsiasi di questi **come messaggio singolo** (senza lo slash iniziale):

```
stop
abort
esc
wait
exit
interrupt
```

Questi sono trigger di interruzione (non slash command).

Per i processi in background avviati tramite lo strumento exec, puoi chiedere all’agente di eseguire:

```
process action:kill sessionId:XXX
```

Panoramica dei comandi slash: consulta [Slash commands](/it/tools/slash-commands).

La maggior parte dei comandi deve essere inviata come messaggio **autonomo** che inizia con `/`, ma alcune scorciatoie (come `/status`) funzionano anche inline per i mittenti presenti nella lista di autorizzati.

<div id="how-do-i-send-a-discord-message-from-telegram-crosscontext-messaging-denied">
  ### Come posso inviare un messaggio Discord da Telegram (messaggistica cross‑contesto negata)
</div>

OpenClaw blocca la messaggistica **cross‑provider** per impostazione predefinita. Se una chiamata di uno strumento è vincolata
a Telegram, il messaggio non verrà inviato a Discord a meno che tu non lo permetta esplicitamente.

Abilita la messaggistica cross‑provider per l&#39;agente:

```json5
{
  agents: {
    defaults: {
      tools: {
        message: {
          crossContext: {
            allowAcrossProviders: true,
            marker: { enabled: true, prefix: "[da {channel}] " }
          }
        }
      }
    }
  }
}
```

Riavvia il Gateway dopo aver modificato la configurazione. Se vuoi applicare questa impostazione solo a un singolo agente, impostala in `agents.list[].tools.message` invece.

<div id="why-does-it-feel-like-the-bot-ignores-rapidfire-messages">
  ### Perché sembra che il bot ignori i messaggi inviati in rapida successione
</div>

La modalità coda controlla come i nuovi messaggi interagiscono con un&#39;esecuzione già in corso. Usa `/queue` per cambiare modalità:

* `steer` - i nuovi messaggi reindirizzano l&#39;attività corrente
* `followup` - elabora i messaggi uno alla volta
* `collect` - raggruppa i messaggi e risponde una sola volta (modalità predefinita)
* `steer-backlog` - reindirizza subito, poi elabora l&#39;arretrato
* `interrupt` - interrompe l&#39;esecuzione corrente e ne avvia una nuova

Puoi aggiungere opzioni come `debounce:2s cap:25 drop:summarize` per le modalità followup.

<div id="answer-the-exact-question-from-the-screenshotchat-log">
  ## Rispondi esattamente alla domanda dallo screenshot/log della chat
</div>

**D: “Qual è il modello predefinito per Anthropic con una chiave API?”**

**R:** In OpenClaw, credenziali e selezione del modello sono separate. Impostare `ANTHROPIC_API_KEY` (o memorizzare una chiave API Anthropic nei profili di autenticazione) abilita l’autenticazione, ma il modello predefinito effettivo è quello che configuri in `agents.defaults.model.primary` (ad esempio, `anthropic/claude-sonnet-4-5` o `anthropic/claude-opus-4-5`). Se vedi `No credentials found for profile "anthropic:default"`, significa che il Gateway non ha trovato le credenziali Anthropic nel file `auth-profiles.json` previsto per l’agente in esecuzione.

***

Ancora bloccato? Chiedi su [Discord](https://discord.com/invite/clawd) o apri una [discussione su GitHub](https://github.com/openclaw/openclaw/discussions).