---
title: FAQ
summary: "Foire aux questions sur l’installation, la configuration et l’utilisation d’OpenClaw"
---

<div id="faq">
  # FAQ
</div>

Réponses rapides et pistes de dépannage détaillées pour des configurations réelles (développement local, VPS, multi-agents, clés OAuth/API, basculement de modèle). Pour le diagnostic en cours d’exécution, voir [Dépannage](/fr/gateway/troubleshooting). Pour la référence complète de la configuration, voir [Configuration](/fr/gateway/configuration).

<div id="table-of-contents">
  ## Table des matières
</div>

* [Démarrage rapide et configuration du premier lancement](#quick-start-and-firstrun-setup)
  * [Je suis bloqué·e : quelle est la manière la plus rapide de me débloquer ?](#im-stuck-whats-the-fastest-way-to-get-unstuck)
  * [Quelle est la méthode recommandée pour installer et configurer OpenClaw ?](#whats-the-recommended-way-to-install-and-set-up-openclaw)
  * [Comment ouvrir le tableau de bord après l’onboarding ?](#how-do-i-open-the-dashboard-after-onboarding)
  * [Comment authentifier le tableau de bord (jeton) en localhost vs à distance ?](#how-do-i-authenticate-the-dashboard-token-on-localhost-vs-remote)
  * [De quel runtime / environnement d’exécution ai-je besoin ?](#what-runtime-do-i-need)
  * [Est-ce que ça fonctionne sur Raspberry Pi ?](#does-it-run-on-raspberry-pi)
  * [Des conseils pour les installations sur Raspberry Pi ?](#any-tips-for-raspberry-pi-installs)
  * [Bloqué sur &quot;wake up my friend&quot; / l’onboarding ne se termine pas. Que faire ?](#it-is-stuck-on-wake-up-my-friend-onboarding-will-not-hatch-what-now)
  * [Puis-je migrer ma configuration vers une nouvelle machine (Mac mini) sans refaire l’onboarding ?](#can-i-migrate-my-setup-to-a-new-machine-mac-mini-without-redoing-onboarding)
  * [Où voir les nouveautés de la dernière version ?](#where-do-i-see-whats-new-in-the-latest-version)
  * [Je ne peux pas accéder à docs.openclaw.ai (erreur SSL). Que faire ?](#i-cant-access-docsopenclawai-ssl-error-what-now)
  * [Quelle est la différence entre stable et beta ?](#whats-the-difference-between-stable-and-beta)
* [Comment installer la version bêta et quelle est la différence entre bêta et dev ?](#how-do-i-install-the-beta-version-and-whats-the-difference-between-beta-and-dev)
  * [Comment essayer les toutes dernières nouveautés ?](#how-do-i-try-the-latest-bits)
  * [Combien de temps prennent généralement l’installation et la prise en main ?](#how-long-does-install-and-onboarding-usually-take)
  * [Programme d’installation bloqué ? Comment obtenir plus d’informations ?](#installer-stuck-how-do-i-get-more-feedback)
  * [L’installation sous Windows signale que git est introuvable ou que openclaw n’est pas reconnu](#windows-install-says-git-not-found-or-openclaw-not-recognized)
  * [La doc n’a pas répondu à ma question – comment obtenir une meilleure réponse ?](#the-docs-didnt-answer-my-question-how-do-i-get-a-better-answer)
  * [Comment installer OpenClaw sur Linux ?](#how-do-i-install-openclaw-on-linux)
  * [Comment installer OpenClaw sur un VPS ?](#how-do-i-install-openclaw-on-a-vps)
  * [Où sont les guides d&#39;installation pour le cloud/VPS ?](#where-are-the-cloudvps-install-guides)
  * [Puis-je demander à OpenClaw de se mettre à jour automatiquement ?](#can-i-ask-openclaw-to-update-itself)
  * [Que fait concrètement l’assistant de configuration initiale ?](#what-does-the-onboarding-wizard-actually-do)
  * [Faut-il un abonnement Claude ou OpenAI pour l’exécuter ?](#do-i-need-a-claude-or-openai-subscription-to-run-this)
  * [Puis-je utiliser un abonnement Claude Max sans clé api ?](#can-i-use-claude-max-subscription-without-an-api-key)
  * [Comment fonctionne l’authentification « setup-token » d’Anthropic ?](#how-does-anthropic-setuptoken-auth-work)
  * [Où trouver un jeton de configuration Anthropic ?](#where-do-i-find-an-anthropic-setuptoken)
  * [Prenez-vous en charge l’authentification par abonnement Claude (Claude Code OAuth) ?](#do-you-support-claude-subscription-auth-claude-code-oauth)
  * [Pourquoi est-ce que je reçois `HTTP 429: rate_limit_error` de la part d&#39;Anthropic ?](#why-am-i-seeing-http-429-ratelimiterror-from-anthropic)
  * [AWS Bedrock est-il pris en charge ?](#is-aws-bedrock-supported)
  * [Comment fonctionne l’authentification de Codex ?](#how-does-codex-auth-work)
  * [Prenez-vous en charge l’authentification par abonnement OpenAI (Codex OAuth) ?](#do-you-support-openai-subscription-auth-codex-oauth)
  * [Comment configurer OAuth pour la CLI Gemini](#how-do-i-set-up-gemini-cli-oauth)
  * [Un modèle local convient-il pour des conversations informelles ?](#is-a-local-model-ok-for-casual-chats)
  * [Comment puis-je faire en sorte que le trafic des modèles hébergés reste dans une région spécifique ?](#how-do-i-keep-hosted-model-traffic-in-a-specific-region)
  * [Dois-je acheter un Mac Mini pour l’installer ?](#do-i-have-to-buy-a-mac-mini-to-install-this)
  * [Ai-je besoin d&#39;un Mac mini pour prendre en charge iMessage ?](#do-i-need-a-mac-mini-for-imessage-support)
  * [Si j’achète un Mac mini pour exécuter OpenClaw, puis-je le connecter à mon MacBook Pro ?](#if-i-buy-a-mac-mini-to-run-openclaw-can-i-connect-it-to-my-macbook-pro)
  * [Puis-je utiliser Bun ?](#can-i-use-bun)
  * [Telegram : que faut-il mettre dans `allowFrom` ?](#telegram-what-goes-in-allowfrom)
  * [Plusieurs personnes peuvent-elles utiliser un seul numéro WhatsApp avec différentes instances OpenClaw ?](#can-multiple-people-use-one-whatsapp-number-with-different-openclaw-instances)
  * [Puis-je exécuter un agent « fast chat » et un agent « Opus for coding » ?](#can-i-run-a-fast-chat-agent-and-an-opus-for-coding-agent)
  * [Homebrew fonctionne-t-il sur Linux ?](#does-homebrew-work-on-linux)
  * [Quelle est la différence entre l’installation « hackable » (git) et l’installation via npm ?](#whats-the-difference-between-the-hackable-git-install-and-npm-install)
  * [Puis-je alterner entre une installation npm et une installation git plus tard ?](#can-i-switch-between-npm-and-git-installs-later)
  * [Dois-je exécuter Gateway sur mon ordinateur portable ou sur un VPS ?](#should-i-run-the-gateway-on-my-laptop-or-a-vps)
  * [À quel point est-il important d’exécuter OpenClaw sur une machine dédiée ?](#how-important-is-it-to-run-openclaw-on-a-dedicated-machine)
  * [Quelles sont les exigences minimales pour le VPS et quel système d’exploitation est recommandé ?](#what-are-the-minimum-vps-requirements-and-recommended-os)
  * [Puis-je exécuter OpenClaw dans une VM et quels sont les prérequis ?](#can-i-run-openclaw-in-a-vm-and-what-are-the-requirements)
* [Qu&#39;est-ce qu&#39;OpenClaw ?](#what-is-openclaw)
  * [Qu’est-ce qu’OpenClaw, en un paragraphe ?](#what-is-openclaw-in-one-paragraph)
  * [Quelle est la proposition de valeur ?](#whats-the-value-proposition)
  * [Je viens de l’installer : que dois-je faire en premier ?](#i-just-set-it-up-what-should-i-do-first)
  * [Quels sont les cinq principaux cas d’usage au quotidien d’OpenClaw ?](#what-are-the-top-five-everyday-use-cases-for-openclaw)
  * [OpenClaw peut-il aider pour la génération de leads, la prospection, les publicités et les articles de blog pour un SaaS ?](#can-openclaw-help-with-lead-gen-outreach-ads-and-blogs-for-a-saas)
  * [Quels sont les avantages par rapport à Claude Code pour le développement web ?](#what-are-the-advantages-vs-claude-code-for-web-development)
* [Compétences et automatisation](#skills-and-automation)
  * [Comment personnaliser les compétences sans salir le dépôt ?](#how-do-i-customize-skills-without-keeping-the-repo-dirty)
  * [Puis-je charger des compétences depuis un dossier personnalisé ?](#can-i-load-skills-from-a-custom-folder)
  * [Comment puis-je utiliser différents modèles pour différentes tâches ?](#how-can-i-use-different-models-for-different-tasks)
  * [Le bot se fige lorsqu’il effectue des tâches lourdes. Comment puis-je déporter cette charge de travail ?](#the-bot-freezes-while-doing-heavy-work-how-do-i-offload-that)
  * [Cron ou les rappels ne se déclenchent pas. Que dois-je vérifier ?](#cron-or-reminders-do-not-fire-what-should-i-check)
  * [Comment installer des compétences sur Linux ?](#how-do-i-install-skills-on-linux)
  * [OpenClaw peut-il exécuter des tâches de manière planifiée ou en continu en arrière-plan ?](#can-openclaw-run-tasks-on-a-schedule-or-continuously-in-the-background)
  * [Puis-je exécuter des compétences réservées à Apple/macOS depuis Linux ?](#can-i-run-applemacosonly-skills-from-linux)
  * [Avez-vous une intégration avec Notion ou HeyGen ?](#do-you-have-a-notion-or-heygen-integration)
  * [Comment installer l’extension Chrome pour la prise de contrôle du navigateur ?](#how-do-i-install-the-chrome-extension-for-browser-takeover)
* [Sandbox et mémoire](#sandboxing-and-memory)
  * [Existe-t-il une documentation dédiée au sandboxing ?](#is-there-a-dedicated-sandboxing-doc)
  * [Comment monter un dossier hôte dans le sandbox ?](#how-do-i-bind-a-host-folder-into-the-sandbox)
  * [Comment fonctionne la mémoire ?](#how-does-memory-work)
  * [La mémoire continue d&#39;oublier des choses. Comment faire pour qu&#39;elle retienne mieux ?](#memory-keeps-forgetting-things-how-do-i-make-it-stick)
  * [La mémoire persiste-t-elle indéfiniment ? Quelles sont les limites ?](#does-memory-persist-forever-what-are-the-limits)
  * [La recherche dans la mémoire sémantique nécessite-t-elle une clé d’API OpenAI ?](#does-semantic-memory-search-require-an-openai-api-key)
* [Emplacement des fichiers sur le disque](#where-things-live-on-disk)
  * [Toutes les données utilisées avec OpenClaw sont-elles sauvegardées localement ?](#is-all-data-used-with-openclaw-saved-locally)
  * [Où OpenClaw stocke-t-il ses données ?](#where-does-openclaw-store-its-data)
  * [Où doivent se trouver AGENTS.md / SOUL.md / USER.md / MEMORY.md ?](#where-should-agentsmd-soulmd-usermd-memorymd-live)
  * [Quelle stratégie de sauvegarde est recommandée ?](#whats-the-recommended-backup-strategy)
  * [Comment désinstaller complètement OpenClaw ?](#how-do-i-completely-uninstall-openclaw)
  * [Les agents peuvent-ils fonctionner en dehors de l’espace de travail ?](#can-agents-work-outside-the-workspace)
  * [Je suis en mode distant : où se trouve le stockage des sessions ?](#im-in-remote-mode-where-is-the-session-store)
* [Bases de la configuration](#config-basics)
  * [Quel est le format de la configuration ? Où se trouve‑t‑elle ?](#what-format-is-the-config-where-is-it)
  * [J&#39;ai défini `gateway.bind: "lan"` (ou `"tailnet"`) et plus rien n&#39;écoute / l&#39;UI indique « unauthorized »](#i-set-gatewaybind-lan-or-tailnet-and-now-nothing-listens-the-ui-says-unauthorized)
  * [Pourquoi ai‑je besoin d&#39;un jeton sur localhost maintenant ?](#why-do-i-need-a-token-on-localhost-now)
  * [Dois‑je redémarrer après avoir modifié la configuration ?](#do-i-have-to-restart-after-changing-config)
  * [Comment activer la recherche Web (et le web fetch) ?](#how-do-i-enable-web-search-and-web-fetch)
  * [`config.apply` a effacé ma configuration. Comment la récupérer et éviter cela ?](#configapply-wiped-my-config-how-do-i-recover-and-avoid-this)
  * [Comment exécuter le Gateway central avec des workers spécialisés sur plusieurs appareils ?](#how-do-i-run-a-central-gateway-with-specialized-workers-across-devices)
  * [Le navigateur OpenClaw peut‑il fonctionner en mode headless ?](#can-the-openclaw-browser-run-headless)
  * [Comment utiliser Brave pour le contrôle du navigateur ?](#how-do-i-use-brave-for-browser-control)
* [Gateways distants + nœuds](#remote-gateways-nodes)
  * [Comment les commandes se propagent-elles entre Telegram, le Gateway et les nœuds ?](#how-do-commands-propagate-between-telegram-the-gateway-and-nodes)
  * [Comment mon agent peut-il accéder à mon ordinateur si le Gateway est hébergé à distance ?](#how-can-my-agent-access-my-computer-if-the-gateway-is-hosted-remotely)
  * [Tailscale est connecté mais je ne reçois aucune réponse. Que faire ?](#tailscale-is-connected-but-i-get-no-replies-what-now)
  * [Deux instances OpenClaw peuvent-elles communiquer entre elles (local + VPS) ?](#can-two-openclaw-instances-talk-to-each-other-local-vps)
  * [Ai-je besoin de VPS distincts pour plusieurs agents ?](#do-i-need-separate-vpses-for-multiple-agents)
  * [Y a-t-il un avantage à utiliser un nœud sur mon ordinateur portable personnel plutôt que d’utiliser SSH depuis un VPS ?](#is-there-a-benefit-to-using-a-node-on-my-personal-laptop-instead-of-ssh-from-a-vps)
  * [Les nœuds exécutent-ils un service Gateway ?](#do-nodes-run-a-gateway-service)
  * [Existe-t-il un moyen via API / RPC d’appliquer la configuration ?](#is-there-an-api-rpc-way-to-apply-config)
  * [Quelle est une configuration minimale « saine » pour une première installation ?](#whats-a-minimal-sane-config-for-a-first-install)
  * [Comment configurer Tailscale sur un VPS et me connecter depuis mon Mac ?](#how-do-i-set-up-tailscale-on-a-vps-and-connect-from-my-mac)
  * [Comment connecter un nœud Mac à un Gateway distant (Tailscale Serve) ?](#how-do-i-connect-a-mac-node-to-a-remote-gateway-tailscale-serve)
  * [Dois-je installer sur un deuxième ordinateur portable ou simplement ajouter un nœud ?](#should-i-install-on-a-second-laptop-or-just-add-a-node)
* [Variables d&#39;environnement et chargement de .env](#env-vars-and-env-loading)
  * [Comment OpenClaw charge-t-il les variables d’environnement ?](#how-does-openclaw-load-environment-variables)
  * [« J’ai démarré le Gateway via le service et mes variables d’environnement ont disparu. » Que faire ?](#i-started-the-gateway-via-the-service-and-my-env-vars-disappeared-what-now)
  * [J’ai défini `COPILOT_GITHUB_TOKEN`, mais le statut des modèles indique « Shell env : off. » Pourquoi ?](#i-set-copilotgithubtoken-but-models-status-shows-shell-env-off-why)
* [Sessions &amp; conversations multiples](#sessions-multiple-chats)
  * [Comment lancer une nouvelle conversation ?](#how-do-i-start-a-fresh-conversation)
  * [Les sessions se réinitialisent-elles automatiquement si je n’envoie jamais la commande `/new` ?](#do-sessions-reset-automatically-if-i-never-send-new)
  * [Existe-t-il un moyen de constituer une équipe d’instances OpenClaw, avec un CEO et plusieurs agents ?](#is-there-a-way-to-make-a-team-of-openclaw-instances-one-ceo-and-many-agents)
  * [Pourquoi le contexte a-t-il été tronqué en plein milieu d’une tâche ? Comment l’éviter ?](#why-did-context-get-truncated-midtask-how-do-i-prevent-it)
  * [Comment réinitialiser complètement OpenClaw tout en le gardant installé ?](#how-do-i-completely-reset-openclaw-but-keep-it-installed)
  * [Je reçois l’erreur « context too large » – comment réinitialiser ou compacter ?](#im-getting-context-too-large-errors-how-do-i-reset-or-compact)
  * [Pourquoi est-ce que je vois « LLM request rejected: messages.N.content.X.tool&#95;use.input: Field required » ?](#why-am-i-seeing-llm-request-rejected-messagesncontentxtooluseinput-field-required)
  * [Pourquoi est-ce que je reçois des messages de signal de vie toutes les 30 minutes ?](#why-am-i-getting-heartbeat-messages-every-30-minutes)
  * [Dois-je ajouter un « compte bot » à un groupe WhatsApp ?](#do-i-need-to-add-a-bot-account-to-a-whatsapp-group)
  * [Comment obtenir le JID d’un groupe WhatsApp ?](#how-do-i-get-the-jid-of-a-whatsapp-group)
  * [Pourquoi OpenClaw ne répond-il pas dans un groupe ?](#why-doesnt-openclaw-reply-in-a-group)
  * [Les groupes/fils partagent-ils le contexte avec les DM ?](#do-groupsthreads-share-context-with-dms)
  * [Combien d’espaces de travail et d’agents puis-je créer ?](#how-many-workspaces-and-agents-can-i-create)
  * [Puis-je exécuter plusieurs bots ou conversations en même temps (Slack), et comment dois-je configurer cela ?](#can-i-run-multiple-bots-or-chats-at-the-same-time-slack-and-how-should-i-set-that-up)
* [Modèles : valeurs par défaut, sélection, alias, changement de modèle](#models-defaults-selection-aliases-switching)
  * [Qu’est-ce que le « modèle par défaut » ?](#what-is-the-default-model)
  * [Quel modèle recommandez-vous ?](#what-model-do-you-recommend)
  * [Comment changer de modèle sans effacer ma configuration ?](#how-do-i-switch-models-without-wiping-my-config)
  * [Puis-je utiliser des modèles auto‑hébergés (llama.cpp, vLLM, Ollama) ?](#can-i-use-selfhosted-models-llamacpp-vllm-ollama)
  * [Quels modèles OpenClaw, Flawd et Krill utilisent‑ils ?](#what-do-openclaw-flawd-and-krill-use-for-models)
  * [Comment changer de modèle à la volée (sans redémarrer) ?](#how-do-i-switch-models-on-the-fly-without-restarting)
  * [Puis-je utiliser GPT 5.2 pour les tâches quotidiennes et Codex 5.2 pour le code ?](#can-i-use-gpt-52-for-daily-tasks-and-codex-52-for-coding)
  * [Pourquoi est‑ce que je vois « Model … is not allowed » puis aucune réponse ?](#why-do-i-see-model-is-not-allowed-and-then-no-reply)
  * [Pourquoi est‑ce que je vois « Unknown model: minimax/MiniMax-M2.1 » ?](#why-do-i-see-unknown-model-minimaxminimaxm21)
  * [Puis‑je utiliser MiniMax comme modèle par défaut et OpenAI pour les tâches complexes ?](#can-i-use-minimax-as-my-default-and-openai-for-complex-tasks)
  * [Opus / sonnet / gpt sont‑ils des raccourcis intégrés ?](#are-opus-sonnet-gpt-builtin-shortcuts)
  * [Comment définir ou redéfinir des raccourcis de modèles (alias) ?](#how-do-i-defineoverride-model-shortcuts-aliases)
  * [Comment ajouter des modèles à partir d’autres fournisseurs comme OpenRouter ou Z.AI ?](#how-do-i-add-models-from-other-providers-like-openrouter-or-zai)
* [Basculement de modèle et message « All models failed »](#model-failover-and-all-models-failed)
  * [Comment fonctionne le basculement ?](#how-does-failover-work)
  * [Que signifie cette erreur ?](#what-does-this-error-mean)
  * [Checklist de correction pour `No credentials found for profile "anthropic:default"`](#fix-checklist-for-no-credentials-found-for-profile-anthropicdefault)
  * [Pourquoi a-t-il aussi essayé Google Gemini et échoué ?](#why-did-it-also-try-google-gemini-and-fail)
* [Profils d’authentification : ce qu’ils sont et comment les gérer](#auth-profiles-what-they-are-and-how-to-manage-them)
  * [Qu’est-ce qu’un profil d’authentification ?](#what-is-an-auth-profile)
  * [Quels sont les ID de profil courants ?](#what-are-typical-profile-ids)
  * [Puis-je contrôler quel profil d’authentification est utilisé en premier ?](#can-i-control-which-auth-profile-is-tried-first)
  * [OAuth vs clé API : quelle est la différence ?](#oauth-vs-api-key-whats-the-difference)
* [Gateway : ports, « déjà en cours d’exécution » et mode distant](#gateway-ports-already-running-and-remote-mode)
  * [Quel port le Gateway utilise-t-il ?](#what-port-does-the-gateway-use)
  * [Pourquoi `openclaw gateway status` affiche `Runtime: running` mais `RPC probe: failed` ?](#why-does-openclaw-gateway-status-say-runtime-running-but-rpc-probe-failed)
  * [Pourquoi `openclaw gateway status` affiche-t-il `Config (CLI)` et `Config (service)` avec des valeurs différentes ?](#why-does-openclaw-gateway-status-show-config-cli-and-config-service-different)
  * [Que signifie « another gateway instance is already listening » ?](#what-does-another-gateway-instance-is-already-listening-mean)
  * [Comment exécuter OpenClaw en mode distant (le client se connecte à un Gateway distant) ?](#how-do-i-run-openclaw-in-remote-mode-client-connects-to-a-gateway-elsewhere)
  * [La Control UI affiche « unauthorized » (ou se reconnecte en boucle). Que faire ?](#the-control-ui-says-unauthorized-or-keeps-reconnecting-what-now)
  * [J’ai défini `gateway.bind: "tailnet"` mais la liaison échoue / rien n’écoute](#i-set-gatewaybind-tailnet-but-it-cant-bind-nothing-listens)
  * [Puis-je exécuter plusieurs Gateways sur le même hôte ?](#can-i-run-multiple-gateways-on-the-same-host)
  * [Que signifient « invalid handshake » / code 1008 ?](#what-does-invalid-handshake-code-1008-mean)
* [Journalisation et débogage](#logging-and-debugging)
  * [Où se trouvent les journaux ?](#where-are-logs)
  * [Comment démarrer/arrêter/redémarrer le service Gateway ?](#how-do-i-startstoprestart-the-gateway-service)
  * [J’ai fermé mon terminal sous Windows – comment redémarrer OpenClaw ?](#i-closed-my-terminal-on-windows-how-do-i-restart-openclaw)
  * [Le service Gateway est en ligne mais les réponses n’arrivent jamais. Que dois-je vérifier ?](#the-gateway-is-up-but-replies-never-arrive-what-should-i-check)
  * [&quot;Disconnected from gateway: no reason&quot; – et maintenant ?](#disconnected-from-gateway-no-reason-what-now)
  * [Telegram setMyCommands échoue avec des erreurs réseau. Que dois-je vérifier ?](#telegram-setmycommands-fails-with-network-errors-what-should-i-check)
  * [La TUI n’affiche aucune sortie. Que dois-je vérifier ?](#tui-shows-no-output-what-should-i-check)
  * [Comment arrêter complètement puis redémarrer le service Gateway ?](#how-do-i-completely-stop-then-start-the-gateway)
  * [ELI5 : `openclaw gateway restart` vs `openclaw gateway`](#eli5-openclaw-gateway-restart-vs-openclaw-gateway)
  * [Quel est le moyen le plus rapide d’obtenir plus de détails en cas d’échec ?](#whats-the-fastest-way-to-get-more-details-when-something-fails)
* [Médias et fichiers joints](#media-attachments)
  * [Mon skill a généré une image ou un PDF, mais rien n&#39;a été envoyé](#my-skill-generated-an-imagepdf-but-nothing-was-sent)
* [Sécurité et contrôle des accès](#security-and-access-control)
  * [Est-il sans danger d’exposer OpenClaw à des DMs entrants ?](#is-it-safe-to-expose-openclaw-to-inbound-dms)
  * [L’injection de prompt est-elle uniquement un problème pour les bots publics ?](#is-prompt-injection-only-a-concern-for-public-bots)
  * [Mon bot doit-il avoir son propre compte e-mail, compte GitHub ou numéro de téléphone ?](#should-my-bot-have-its-own-email-github-account-or-phone-number)
  * [Puis-je lui laisser le contrôle de mes SMS, et est-ce sans danger ?](#can-i-give-it-autonomy-over-my-text-messages-and-is-that-safe)
  * [Puis-je utiliser des modèles moins chers pour les tâches d’assistant personnel ?](#can-i-use-cheaper-models-for-personal-assistant-tasks)
  * [J’ai exécuté `/start` sur Telegram mais je n’ai pas reçu de code d’appairage](#i-ran-start-in-telegram-but-didnt-get-a-pairing-code)
  * [WhatsApp : va-t-il envoyer des messages à mes contacts ? Comment fonctionne l’appairage ?](#whatsapp-will-it-message-my-contacts-how-does-pairing-work)
* [Commandes de chat, interruption des tâches et « il ne s’arrête plus »](#chat-commands-aborting-tasks-and-it-wont-stop)
  * [Comment empêcher l’affichage des messages système internes dans le chat](#how-do-i-stop-internal-system-messages-from-showing-in-chat)
  * [Comment arrêter/annuler une tâche en cours d’exécution ?](#how-do-i-stopcancel-a-running-task)
  * [Comment envoyer un message Discord depuis Telegram ? (« Cross-context messaging denied »)](#how-do-i-send-a-discord-message-from-telegram-crosscontext-messaging-denied)
  * [Pourquoi a-t-on l’impression que le bot « ignore » les messages envoyés en rafale ?](#why-does-it-feel-like-the-bot-ignores-rapidfire-messages)

<div id="first-60-seconds-if-somethings-broken">
  ## Premières 60 secondes si quelque chose ne fonctionne plus
</div>

1. **État rapide (première vérification)**
   ```bash
   openclaw status
   ```
   Résumé local rapide : OS + mise à jour, accessibilité du Gateway/service, agents/sessions, problèmes de configuration de fournisseur + d’exécution (quand le Gateway est accessible).

2. **Rapport à coller (partageable en toute sécurité)**
   ```bash
   openclaw status --all
   ```
   Diagnostic en lecture seule avec fin des journaux (jetons masqués).

3. **État du démon + des ports**
   ```bash
   openclaw gateway status
   ```
   Affiche l’état d’exécution du superviseur vs l’accessibilité RPC, l’URL cible de la sonde et la configuration probablement utilisée par le service.

4. **Sondes approfondies**
   ```bash
   openclaw status --deep
   ```
   Exécute les vérifications d’état du Gateway + les sondes des fournisseurs (nécessite un Gateway accessible). Voir [Health](/fr/gateway/health).

5. **Suivre le dernier journal**
   ```bash
   openclaw logs --follow
   ```
   Si le RPC est en panne, revenir à :
   ```bash
   tail -f "$(ls -t /tmp/openclaw/openclaw-*.log | head -1)"
   ```
   Les journaux en fichier sont distincts des journaux du service : voir [Logging](/fr/logging) et [Troubleshooting](/fr/gateway/troubleshooting).

6. **Lancer le doctor (réparations)**
   ```bash
   openclaw doctor
   ```
   Répare/migre la configuration/l’état + exécute les vérifications d’état. Voir [Doctor](/fr/gateway/doctor).

7. **Instantané du Gateway**
   ```bash
   openclaw health --json
   openclaw health --verbose   # affiche l’URL cible + le chemin de config en cas d’erreurs
   ```
   Demande au Gateway en cours d’exécution un instantané complet (uniquement via WS). Voir [Health](/fr/gateway/health).

<div id="quick-start-and-first-run-setup">
  ## Démarrage rapide et configuration lors du premier lancement
</div>

<div id="im-stuck-whats-the-fastest-way-to-get-unstuck">
  ### Je suis bloqué : quelle est la méthode la plus rapide pour me débloquer ?
</div>

Utilise un agent d’IA local qui peut **voir ta machine**. C’est beaucoup plus efficace que de demander
sur Discord, car la plupart des cas « je suis bloqué » sont des **problèmes locaux de configuration ou d’environnement**
que les personnes à distance ne peuvent pas inspecter.

* **Claude Code**: https://www.anthropic.com/claude-code/
* **OpenAI Codex**: https://openai.com/codex/

Ces outils peuvent read le dépôt, exécuter des commandes, inspecter les logs et t’aider à corriger la configuration
de ta machine (PATH, services, permissions, fichiers d’authentification). Donne-leur le **checkout complet du code source**
via l’installation « hackable » (git) :

```bash
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --install-method git
```

Cela installe OpenClaw **à partir d’un checkout Git**, afin que l’agent puisse read le code et la doc
et raisonner sur la version exacte que vous exécutez. Vous pouvez toujours revenir plus tard à la version stable
en relançant le programme d’installation sans `--install-method git`.

Conseil : demandez à l’agent de **planifier et superviser** la résolution du problème (étape par étape), puis d’exécuter uniquement
les commandes nécessaires. Cela limite la portée des changements et les rend plus faciles à auditer.

Si vous découvrez un véritable bug ou une correction, veuillez ouvrir un ticket GitHub ou envoyer une PR :
https://github.com/openclaw/openclaw/issues
https://github.com/openclaw/openclaw/pulls

Commencez avec ces commandes (partagez les résultats lorsque vous demandez de l’aide) :

```bash
openclaw status
openclaw models status
openclaw doctor
```

Ce qu’elles font :

* `openclaw status` : vue d’ensemble rapide de l’état du Gateway/de l’agent + configuration de base.
* `openclaw models status` : vérifie l’authentification du fournisseur et la disponibilité des modèles.
* `openclaw doctor` : valide et répare les problèmes courants de configuration/état.

Autres vérifications CLI utiles : `openclaw status --all`, `openclaw logs --follow`,
`openclaw gateway status`, `openclaw health --verbose`.

Boucle de débogage rapide : [Premières 60 secondes si quelque chose ne va pas](#first-60-seconds-if-somethings-broken).
Documentation d’installation : [Installation](/fr/install), [Options de l’installateur](/fr/install/installer), [Mise à jour](/fr/install/updating).

<div id="whats-the-recommended-way-to-install-and-set-up-openclaw">
  ### Quelle est la méthode recommandée pour installer et configurer OpenClaw ?
</div>

Le dépôt recommande d’exécuter OpenClaw depuis les sources et d’utiliser l’assistant de configuration initiale :

```bash
curl -fsSL https://openclaw.bot/install.sh | bash
openclaw onboard --install-daemon
```

L&#39;assistant peut aussi générer automatiquement les ressources UI. Après l&#39;onboarding, vous lancez généralement le Gateway sur le port **18789**.

À partir des sources (contributeurs/dev) :

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm build
pnpm ui:build # installe automatiquement les dépendances de l'UI à la première exécution
openclaw onboard
```

Si vous n’avez pas encore effectué d’installation globale, exécutez la commande `pnpm openclaw onboard`.

<div id="how-do-i-open-the-dashboard-after-onboarding">
  ### Comment ouvrir le tableau de bord après l’onboarding
</div>

L’assistant ouvre désormais votre navigateur avec une URL de tableau de bord contenant un jeton juste après l’onboarding, et affiche également le lien complet (avec jeton) dans le récapitulatif. Gardez cet onglet ouvert ; si la page ne s’est pas ouverte, copiez/collez l’URL affichée sur la même machine. Les jetons restent locaux à votre machine hôte : rien n’est transmis depuis le navigateur.

<div id="how-do-i-authenticate-the-dashboard-token-on-localhost-vs-remote">
  ### Comment authentifier le jeton du tableau de bord en localhost vs à distance
</div>

**Localhost (même machine) :**

* Ouvrez `http://127.0.0.1:18789/`.
* S’il demande une authentification, exécutez `openclaw dashboard` et utilisez le lien avec jeton (`?token=...`).
* Le jeton a la même valeur que `gateway.auth.token` (ou `OPENCLAW_GATEWAY_TOKEN`) et est stocké par l’UI après le premier chargement.

**Pas en localhost :**

* **Tailscale Serve** (recommandé) : gardez le bind sur loopback, exécutez `openclaw gateway --tailscale serve`, ouvrez `https://<magicdns>/`. Si `gateway.auth.allowTailscale` est défini sur `true`, les en-têtes d’identité suffisent pour l’authentification (pas de jeton).
* **Bind Tailnet** : exécutez `openclaw gateway --bind tailnet --token "<token>"`, ouvrez `http://<tailscale-ip>:18789/`, collez le jeton dans les paramètres du tableau de bord.
* **Tunnel SSH** : `ssh -N -L 18789:127.0.0.1:18789 user@host` puis ouvrez `http://127.0.0.1:18789/?token=...` à partir de `openclaw dashboard`.

Voir [Dashboard](/fr/web/dashboard) et [Surfaces Web](/fr/web) pour les modes de bind et les détails d’authentification.

<div id="what-runtime-do-i-need">
  ### De quel environnement d’exécution ai-je besoin
</div>

Node.js **&gt;= 22** est requis. `pnpm` est recommandé. Bun n’est **pas recommandé** pour Gateway.

<div id="does-it-run-on-raspberry-pi">
  ### Est-ce qu’il fonctionne sur Raspberry Pi ?
</div>

Oui. Le Gateway est léger : la documentation indique **512 Mo-1 Go de RAM**, **1 cœur**, et environ **500 Mo**
d’espace disque comme suffisants pour une utilisation personnelle, et précise qu’un **Raspberry Pi 4 peut l’exécuter**.

Si vous voulez un peu plus de marge (journaux/logs, médias, autres services), **2 Go sont recommandés**, mais ce n’est
pas un minimum strict.

Conseil : un petit Pi/VPS peut héberger le Gateway, et vous pouvez appairer des **nœuds** sur votre ordinateur portable/téléphone pour
l’écran/appareil photo/canvas locaux ou l’exécution de commandes. Voir [Nodes](/fr/nodes).

<div id="any-tips-for-raspberry-pi-installs">
  ### Des conseils pour les installations sur Raspberry Pi ?
</div>

Version courte : ça fonctionne, mais attendez-vous à quelques aspérités.

* Utilisez un OS **64 bits** et gardez Node &gt;= 22.
* Privilégiez l’**installation bidouillable (git)** pour pouvoir voir les logs et mettre à jour rapidement.
* Commencez sans canaux/compétences, puis ajoutez-les une par une.
* Si vous rencontrez des problèmes bizarres liés aux binaires, il s’agit généralement d’un problème de **compatibilité ARM**.

Docs : [Linux](/fr/platforms/linux), [Install](/fr/install).

<div id="it-is-stuck-on-wake-up-my-friend-onboarding-will-not-hatch-what-now">
  ### L&#39;écran reste bloqué sur « Wake up, my friend » et l&#39;onboarding ne démarre jamais : que faire ?
</div>

Cet écran dépend du fait que le Gateway soit atteignable et authentifié. Le TUI envoie aussi
« Wake up, my friend! » automatiquement lors du premier lancement. Si vous voyez cette ligne **sans aucune réponse**
et que le nombre de jetons reste à 0, l&#39;agent n&#39;a jamais été lancé.

1. Redémarrez le Gateway :

```bash
openclaw gateway restart
```

2. Vérifiez l’état et l’authentification :

```bash
openclaw status
openclaw models status
openclaw logs --follow
```

3. S&#39;il se bloque encore, exécutez :

```bash
openclaw doctor
```

Si le Gateway est distant, assurez-vous que le tunnel est établi ou que la connexion Tailscale est active et que l&#39;UI
pointe vers le bon Gateway. Voir [Accès distant](/fr/gateway/remote).

<div id="can-i-migrate-my-setup-to-a-new-machine-mac-mini-without-redoing-onboarding">
  ### Puis-je migrer ma configuration vers un nouveau Mac mini sans refaire l’onboarding ?
</div>

Oui. Copiez le **répertoire d’état** et l’**espace de travail**, puis exécutez `openclaw doctor` une fois. Cela
garde votre bot « exactement tel quel » (mémoire, historique de session, auth et état
des canaux) tant que vous copiez ces **deux** emplacements :

1. Installez OpenClaw sur la nouvelle machine.
2. Copiez `$OPENCLAW_STATE_DIR` (valeur par défaut : `~/.openclaw`) depuis l’ancienne machine.
3. Copiez votre espace de travail (valeur par défaut : `~/.openclaw/workspace`).
4. Exécutez `openclaw doctor` et redémarrez le service Gateway.

Cela préserve la configuration, les profils d’authentification, les identifiants WhatsApp, les sessions et la mémoire. Si vous êtes en
mode distant, rappelez‑vous que l’hôte du Gateway possède le stockage des sessions et l’espace de travail.

**Important :** si vous ne faites qu’un commit/push de votre espace de travail sur GitHub, vous sauvegardez
la **mémoire + les fichiers de bootstrap**, mais **pas** l’historique des sessions ni l’auth. Ceux‑ci se trouvent
sous `~/.openclaw/` (par exemple `~/.openclaw/agents/<agentId>/sessions/`).

Voir aussi : [Migration](/fr/install/migrating), [Emplacement des données sur le disque](/fr/help/faq#where-does-openclaw-store-its-data),
[Espace de travail de l’Agent](/fr/concepts/agent-workspace), [Doctor](/fr/gateway/doctor),
[Mode distant](/fr/gateway/remote).

<div id="where-do-i-see-whats-new-in-the-latest-version">
  ### Où puis-je voir les nouveautés de la dernière version ?
</div>

Consultez le changelog GitHub :\
https://github.com/openclaw/openclaw/blob/main/CHANGELOG.md

Les entrées les plus récentes sont en haut. Si la section du haut est marquée **Unreleased**, la section datée suivante correspond à la dernière version publiée. Les entrées sont regroupées par **Highlights**, **Changes** et **Fixes** (ainsi que des sections docs/autres si nécessaire).

<div id="i-cant-access-docsopenclawai-ssl-error-what-now">
  ### Je ne peux pas accéder à docs.openclaw.ai, erreur SSL — que faire ?
</div>

Certaines connexions Comcast/Xfinity bloquent à tort `docs.openclaw.ai` via Xfinity Advanced Security. Désactivez cette fonctionnalité ou ajoutez `docs.openclaw.ai` à la liste d’autorisation, puis réessayez. Plus de détails : [Dépannage](/fr/help/troubleshooting#docsopenclawai-shows-an-ssl-error-comcastxfinity).\
Aidez-nous à le faire débloquer en le signalant ici : https://spa.xfinity.com/check&#95;url&#95;status.

Si vous ne pouvez toujours pas atteindre le site, la documentation est également disponible en miroir sur GitHub :\
https://github.com/openclaw/openclaw/tree/main/docs

<div id="whats-the-difference-between-stable-and-beta">
  ### Quelle est la différence entre stable et beta
</div>

**Stable** et **beta** sont des **dist‑tags npm**, pas des branches de code distinctes :

* `latest` = stable
* `beta` = build préliminaire pour les tests

Nous publions des builds sur **beta**, nous les testons, et une fois qu’un build est jugé stable, nous **promouvons
cette même version vers `latest`**. C’est pour cela que beta et stable peuvent pointer vers la
**même version**.

Voir ce qui a changé :\
https://github.com/openclaw/openclaw/blob/main/CHANGELOG.md

<div id="how-do-i-install-the-beta-version-and-whats-the-difference-between-beta-and-dev">
  ### Comment installer la version bêta et quelle est la différence entre bêta et dev
</div>

**Beta** correspond au dist‑tag npm `beta` (peut coïncider avec `latest`).
**Dev** suit la branche `main` (git) en évolution continue ; lorsqu’une version est publiée, elle utilise le dist‑tag npm `dev`.

Commandes en une ligne (macOS/Linux) :

```bash
curl -fsSL --proto '=https' --tlsv1.2 https://openclaw.bot/install.sh | bash -s -- --beta
```

```bash
curl -fsSL --proto '=https' --tlsv1.2 https://openclaw.bot/install.sh | bash -s -- --install-method git
```

Programme d’installation Windows (PowerShell) :
https://openclaw.ai/install.ps1

Pour plus de détails : [Canaux de développement](/fr/install/development-channels) et [Options de l’installateur](/fr/install/installer).

<div id="how-long-does-install-and-onboarding-usually-take">
  ### Combien de temps prennent généralement l’installation et la mise en route
</div>

Indication approximative :

* **Installation :** 2-5 minutes
* **Mise en route :** 5-15 minutes selon le nombre de canaux/modèles que vous configurez

Si ça se bloque, utilisez [Installer bloqué](/fr/help/faq#installer-stuck-how-do-i-get-more-feedback)
et la boucle de débogage rapide décrite dans [Je suis bloqué](/fr/help/faq#im-stuck--whats-the-fastest-way-to-get-unstuck).

<div id="how-do-i-try-the-latest-bits">
  ### Comment tester les toutes dernières nouveautés ?
</div>

Deux options :

1. **Canal de développement (git checkout) :**

```bash
openclaw update --channel dev
```

Cette commande bascule sur la branche `main` et met à jour depuis la source.

2. **Installation « hackable » (depuis le site d’installation) :**

```bash
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --install-method git
```

Cela vous donne un dépôt local que vous pouvez modifier, puis mettre à jour avec git.

Si vous préférez effectuer un clonage propre manuellement, utilisez :

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm build
```

Documentation : [Mise à jour](/fr/cli/update), [Canaux de développement](/fr/install/development-channels),
[Installation](/fr/install).

<div id="installer-stuck-how-do-i-get-more-feedback">
  ### Le programme d&#39;installation est bloqué : comment obtenir plus d&#39;informations ?
</div>

Relancez le programme d&#39;installation avec une **sortie détaillée** :

```bash
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --verbose
```

Installation de la version bêta en mode verbeux :

```bash
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --beta --verbose
```

Pour une installation à partir de Git :

```bash
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --install-method git --verbose
```

Plus d’options : [Options de l’installateur](/fr/install/installer).

<div id="windows-install-says-git-not-found-or-openclaw-not-recognized">
  ### L’installation Windows indique que git est introuvable ou que openclaw n’est pas reconnu
</div>

Deux problèmes fréquents sous Windows :

**1) erreur npm spawn git / git not found**

* Installez **Git for Windows** et assurez-vous que `git` est dans votre PATH.
* Fermez puis rouvrez PowerShell, puis relancez l’installateur.

**2) openclaw n’est pas reconnu après l’installation**

* Votre dossier `bin` global de npm n’est pas dans le PATH.
* Vérifiez le chemin :
  ```powershell
  npm config get prefix
  ```
* Assurez-vous que `<prefix>\\bin` est dans le PATH (sur la plupart des systèmes, c’est `%AppData%\\npm`).
* Fermez puis rouvrez PowerShell après la mise à jour du PATH.

Pour une configuration Windows plus fluide, utilisez plutôt **WSL2** que Windows natif.
Documentation : [Windows](/fr/platforms/windows).

<div id="the-docs-didnt-answer-my-question-how-do-i-get-a-better-answer">
  ### La doc n’a pas répondu à ma question : comment obtenir une meilleure réponse ?
</div>

Utilise l’**installation hackable (git)** pour avoir l’intégralité du code source et de la doc en local, puis pose ta question
à ton bot (ou à Claude/Codex) *depuis ce dossier* afin qu’il puisse lire le dépôt Git et répondre précisément.

```bash
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --install-method git
```

Pour plus de détails : [Installation](/fr/install) et [Options du programme d’installation](/fr/install/installer).

<div id="how-do-i-install-openclaw-on-linux">
  ### Comment installer OpenClaw sur Linux
</div>

Réponse courte : suivez le guide Linux, puis exécutez l’assistant de démarrage.

* Parcours rapide Linux + installation en tant que service système : [Linux](/fr/platforms/linux).
* Parcours complet : [Bien démarrer](/fr/start/getting-started).
* Programme d’installation + mises à jour : [Installation &amp; mises à jour](/fr/install/updating).

<div id="how-do-i-install-openclaw-on-a-vps">
  ### Comment installer OpenClaw sur un VPS
</div>

N&#39;importe quel VPS Linux convient. Installez OpenClaw sur le serveur, puis utilisez SSH ou Tailscale pour accéder au Gateway.

Guides : [exe.dev](/fr/platforms/exe-dev), [Hetzner](/fr/platforms/hetzner), [Fly.io](/fr/platforms/fly).\
Accès à distance : [Accès distant au Gateway](/fr/gateway/remote).

<div id="where-are-the-cloudvps-install-guides">
  ### Où se trouvent les guides d’installation cloud/VPS
</div>

Nous maintenons un **hub d’hébergement** avec les fournisseurs courants. Choisissez-en un et suivez le guide :

* [VPS hosting](/fr/vps) (tous les fournisseurs au même endroit)
* [Fly.io](/fr/platforms/fly)
* [Hetzner](/fr/platforms/hetzner)
* [exe.dev](/fr/platforms/exe-dev)

Fonctionnement dans le cloud : le **Gateway s’exécute sur le serveur**, et vous y accédez
depuis votre ordinateur portable/téléphone via le Control UI (ou Tailscale/SSH). Votre état + votre espace de travail
sont hébergés sur le serveur ; considérez donc cet hôte comme la source de vérité et sauvegardez-le.

Vous pouvez associer des **nœuds** (Mac/iOS/Android/headless) à ce Gateway dans le cloud pour accéder
à l’écran/appareil photo/canvas locaux ou exécuter des commandes sur votre ordinateur portable tout en gardant le
Gateway dans le cloud.

Hub : [Platforms](/fr/platforms). Accès distant : [Gateway remote](/fr/gateway/remote).
Nœuds : [Nodes](/fr/nodes), [Nodes CLI](/fr/cli/nodes).

<div id="can-i-ask-openclaw-to-update-itself">
  ### Puis-je demander à OpenClaw de se mettre à jour lui-même ?
</div>

Réponse courte : **possible, déconseillé**. La procédure de mise à jour peut
redémarrer le Gateway (ce qui met fin à la session active), peut nécessiter un
`git checkout` propre et peut demander une confirmation. Plus sûr : exécuter
les mises à jour depuis un shell en tant qu’opérateur.

Utilisez la CLI :

```bash
openclaw update
openclaw update status
openclaw update --channel stable|beta|dev
openclaw update --tag <dist-tag|version>
openclaw update --no-restart
```

Si vous devez automatiser à partir d’un agent :

```bash
openclaw update --yes --no-restart
openclaw gateway restart
```

Documentation : [update](/fr/cli/update), [Mise à jour](/fr/install/updating).

<div id="what-does-the-onboarding-wizard-actually-do">
  ### Que fait réellement l’assistant de configuration d’onboarding
</div>

`openclaw onboard` est le parcours de configuration recommandé. En **mode local**, il vous guide à travers :

* **Configuration du modèle/de l’authentification** (Anthropic **setup-token** recommandé pour les abonnements Claude, OAuth OpenAI Codex pris en charge, clés API facultatives, modèles locaux LM Studio pris en charge)
* Emplacement de l’**espace de travail** + fichiers de bootstrap
* **Paramètres du Gateway** (bind/port/auth/Tailscale)
* **Fournisseurs** (WhatsApp, Telegram, Discord, Mattermost (plugin), Signal, iMessage)
* **Installation du démon** (LaunchAgent sur macOS ; unité systemd utilisateur sur Linux/WSL2)
* **Contrôles de bon fonctionnement** et sélection des **compétences**

Il vous avertit également si le modèle que vous avez configuré est inconnu ou si l’authentification est manquante.

<div id="do-i-need-a-claude-or-openai-subscription-to-run-this">
  ### Ai-je besoin d’un abonnement Claude ou OpenAI pour faire fonctionner ceci ?
</div>

Non. Vous pouvez exécuter OpenClaw avec des **clés API** (Anthropic/OpenAI/autres) ou avec des **modèles entièrement locaux**, afin que vos données restent sur votre appareil. Les abonnements (Claude Pro/Max ou OpenAI Codex) sont des moyens optionnels d’authentifier ces fournisseurs.

Docs : [Anthropic](/fr/providers/anthropic), [OpenAI](/fr/providers/openai),
[Modèles locaux](/fr/gateway/local-models), [Modèles](/fr/concepts/models).

<div id="can-i-use-claude-max-subscription-without-an-api-key">
  ### Puis-je utiliser un abonnement Claude Max sans clé API ?
</div>

Oui. Vous pouvez vous authentifier avec un **setup-token**
au lieu d&#39;une clé API. C&#39;est le flux prévu pour les abonnements.

Les abonnements Claude Pro/Max **n&#39;incluent pas de clé API**, il s&#39;agit donc de
la bonne approche pour les comptes avec abonnement. Important : vous devez vérifier auprès
d&#39;Anthropic que cet usage est autorisé par leur politique d&#39;abonnement et leurs conditions générales.
Si vous voulez le chemin le plus explicite et officiellement pris en charge, utilisez une clé API Anthropic.

<div id="how-does-anthropic-setuptoken-auth-work">
  ### Comment fonctionne l’authentification Anthropic setuptoken
</div>

`claude setup-token` génère un **jeton** via la CLI Claude Code (non disponible dans la console web). Vous pouvez l’exécuter sur **n’importe quelle machine**. Choisissez **Anthropic token (paste setup-token)** dans l’assistant ou collez-le avec `openclaw models auth paste-token --provider anthropic`. Le jeton est stocké comme profil d’authentification pour le fournisseur **anthropic** et utilisé comme une clé API (sans renouvellement automatique). Plus de détails : [OAuth](/fr/concepts/oauth).

<div id="where-do-i-find-an-anthropic-setuptoken">
  ### Où trouver un setup-token Anthropic
</div>

Il **ne se trouve pas** dans la console Anthropic. Le setup-token est généré par la **CLI Claude Code** sur **n&#39;importe quelle machine** :

```bash
claude setup-token
```

Copiez le jeton qu’il affiche, puis choisissez **Anthropic token (paste setup-token)** dans l’assistant. Si vous voulez l’exécuter sur l’hôte Gateway, utilisez `openclaw models auth setup-token --provider anthropic`. Si vous avez exécuté `claude setup-token` ailleurs, collez-le sur l’hôte Gateway avec `openclaw models auth paste-token --provider anthropic`. Voir [Anthropic](/fr/providers/anthropic).

<div id="do-you-support-claude-subscription-auth-claude-promax">
  ### Prenez-vous en charge l’authentification par abonnement Claude (Claude Pro/Max) ?
</div>

Oui — via **setup-token**. OpenClaw ne réutilise plus les jetons OAuth de Claude Code CLI ; utilisez un setup-token ou une clé API Anthropic. Générez le jeton depuis n’importe où et collez-le sur l’hôte exécutant Gateway. Voir [Anthropic](/fr/providers/anthropic) et [OAuth](/fr/concepts/oauth).

Remarque : l’accès par abonnement Claude est régi par les conditions d’Anthropic. Pour des charges de travail en production ou multi-utilisateurs, les clés API sont généralement le choix le plus sûr.

<div id="why-am-i-seeing-http-429-ratelimiterror-from-anthropic">
  ### Pourquoi est‑ce que je vois une erreur HTTP 429 ratelimiterror d’Anthropic
</div>

Cela signifie que votre **quota/limite de débit Anthropic** est épuisé pour la fenêtre de temps actuelle. Si vous
utilisez un **abonnement Claude** (setup‑token ou OAuth Claude Code), attendez que la fenêtre se
réinitialise ou mettez votre offre à niveau. Si vous utilisez une **clé API Anthropic**, vérifiez la console Anthropic
pour l’utilisation/la facturation et augmentez les limites si nécessaire.

Astuce : définissez un **modèle de secours** pour qu’OpenClaw puisse continuer à répondre pendant qu’un fournisseur est soumis à une limite de débit.
Consultez [Models](/fr/cli/models) et [OAuth](/fr/concepts/oauth).

<div id="is-aws-bedrock-supported">
  ### AWS Bedrock est‑il pris en charge
</div>

Oui — via le fournisseur **Amazon Bedrock (Converse)** de pi‑ai avec une **configuration manuelle**. Vous devez fournir les identifiants AWS et la région sur la machine qui héberge le Gateway et ajouter une entrée de fournisseur Bedrock dans la configuration de vos modèles. Voir [Amazon Bedrock](/fr/bedrock) et [Fournisseurs de modèles](/fr/providers/models). Si vous préférez un flux de gestion de clés managé, un proxy compatible OpenAI devant Bedrock reste une option valable.

<div id="how-does-codex-auth-work">
  ### Comment fonctionne l’authentification Codex ?
</div>

OpenClaw prend en charge **OpenAI Code (Codex)** via OAuth (connexion via ChatGPT). L’assistant peut exécuter le flux OAuth et définira le modèle par défaut sur `openai-codex/gpt-5.2` lorsque cela est approprié. Consultez [Fournisseurs de modèles](/fr/concepts/model-providers) et [Assistant](/fr/start/wizard).

<div id="do-you-support-openai-subscription-auth-codex-oauth">
  ### Prenez-vous en charge l’OAuth d’abonnement OpenAI Code (Codex)
</div>

Oui. OpenClaw prend entièrement en charge **l’OAuth d’abonnement OpenAI Code (Codex)**. L’assistant de démarrage
peut exécuter le flux OAuth pour vous.

Consultez [OAuth](/fr/concepts/oauth), [Fournisseurs de modèles](/fr/concepts/model-providers) et [Assistant](/fr/start/wizard).

<div id="how-do-i-set-up-gemini-cli-oauth">
  ### Comment configurer OAuth pour Gemini CLI
</div>

Gemini CLI utilise un **flux d’authentification via plugin**, et non un ID client ou un secret dans `openclaw.json`.

Étapes :

1. Activer le plugin : `openclaw plugins enable google-gemini-cli-auth`
2. Se connecter : `openclaw models auth login --provider google-gemini-cli --set-default`

Cela stocke les jetons OAuth dans des profils d’authentification sur la machine hôte du Gateway. Détails : [Fournisseurs de modèles](/fr/concepts/model-providers).

<div id="is-a-local-model-ok-for-casual-chats">
  ### Un modèle local convient-il pour des conversations informelles ?
</div>

En général, non. OpenClaw a besoin d’un large contexte et de garanties de sécurité fortes ; les « small cards » tronquent le contexte et provoquent des fuites d’informations. Si vous n’avez pas le choix, exécutez localement la **plus grande** version MiniMax M2.1 possible (LM Studio) et consultez [/gateway/local-models](/fr/gateway/local-models). Les modèles plus petits/quantifiés augmentent le risque d’attaques d’injection de prompt – voir [Security](/fr/gateway/security).

<div id="how-do-i-keep-hosted-model-traffic-in-a-specific-region">
  ### Comment garder le trafic des modèles hébergés dans une région spécifique
</div>

Choisissez des endpoints spécifiques à une région. OpenRouter expose des options hébergées aux États-Unis pour MiniMax, Kimi et GLM ; sélectionnez la variante hébergée aux États-Unis pour conserver les données dans la région. Vous pouvez toujours inclure Anthropic/OpenAI en parallèle en utilisant `models.mode: "merge"` afin que les mécanismes de repli restent disponibles tout en respectant le fournisseur régional que vous sélectionnez.

<div id="do-i-have-to-buy-a-mac-mini-to-install-this">
  ### Dois‑je acheter un Mac mini pour installer ceci
</div>

Non. OpenClaw fonctionne sous macOS ou Linux (Windows via WSL2). Un Mac mini est facultatif : certaines personnes
en achètent un comme hôte toujours allumé, mais un petit VPS, un serveur domestique ou une machine de type Raspberry Pi convient aussi.

Vous n’avez besoin d’un Mac **que pour les outils réservés à macOS**. Pour iMessage, vous pouvez garder le Gateway sur Linux
et exécuter `imsg` sur n’importe quel Mac via SSH en faisant pointer `channels.imessage.cliPath` vers un wrapper SSH.
Si vous voulez d’autres outils réservés à macOS, exécutez le Gateway sur un Mac ou associez un nœud macOS.

Docs : [iMessage](/fr/channels/imessage), [Nodes](/fr/nodes), [Mac remote mode](/fr/platforms/mac/remote).

<div id="do-i-need-a-mac-mini-for-imessage-support">
  ### Ai-je besoin d’un Mac mini pour la prise en charge d’iMessage
</div>

Vous avez besoin **d’un appareil macOS** connecté à Messages. Ce n’a **pas** besoin d’être un Mac mini –
n’importe quel Mac convient. Les intégrations iMessage d’OpenClaw s’exécutent sur macOS (BlueBubbles ou `imsg`),
tandis que le Gateway peut tourner ailleurs.

Configurations courantes :

* Exécuter le Gateway sur Linux/VPS et faire pointer `channels.imessage.cliPath` vers un wrapper SSH qui
  lance `imsg` sur le Mac.
* Tout faire tourner sur le Mac si vous voulez la configuration mono‑machine la plus simple.

Docs : [iMessage](/fr/channels/imessage), [BlueBubbles](/fr/channels/bluebubbles),
[Mac remote mode](/fr/platforms/mac/remote).

<div id="if-i-buy-a-mac-mini-to-run-openclaw-can-i-connect-it-to-my-macbook-pro">
  ### Si j’achète un Mac mini pour exécuter OpenClaw, puis‑je le connecter à mon MacBook Pro ?
</div>

Oui. Le **Mac mini peut exécuter le Gateway**, et votre MacBook Pro peut se connecter en tant que
**nœud** (appareil compagnon). Les nœuds ne font pas tourner le Gateway – ils fournissent des
fonctionnalités supplémentaires comme l’écran/appareil photo/canevas et `system.run` sur cet appareil.

Schéma classique :

* Gateway sur le Mac mini (toujours allumé).
* Le MacBook Pro exécute l’app macOS ou un hôte de nœud et s’apparie au Gateway.
* Utilisez `openclaw nodes status` / `openclaw nodes list` pour les lister.

Docs : [Nœuds](/fr/nodes), [CLI des nœuds](/fr/cli/nodes).

<div id="can-i-use-bun">
  ### Puis-je utiliser Bun ?
</div>

Bun n’est **pas recommandé**. Nous observons des bugs d’exécution, en particulier avec WhatsApp et Telegram.
Utilisez **Node** pour des Gateways stables.

Si vous voulez tout de même expérimenter avec Bun, faites-le sur un Gateway hors production
sans WhatsApp/Telegram.

<div id="telegram-what-goes-in-allowfrom">
  ### Telegram : que mettre dans allowFrom
</div>

`channels.telegram.allowFrom` est **l’ID utilisateur Telegram de l’expéditeur humain** (numérique, recommandé) ou son `@username`. Ce n’est pas le nom d’utilisateur du bot.

Plus sûr (sans bot tiers) :

* Envoyez un DM à votre bot, puis exécutez `openclaw logs --follow` et lisez `from.id`.

Bot API officiel :

* Envoyez un DM à votre bot, puis appelez `https://api.telegram.org/bot<bot_token>/getUpdates` et lisez `message.from.id`.

Tiers (moins privé) :

* Envoyez un DM à `@userinfobot` ou `@getidsbot`.

Voir [/channels/telegram](/fr/channels/telegram#access-control-dms--groups).

<div id="can-multiple-people-use-one-whatsapp-number-with-different-openclaw-instances">
  ### Plusieurs personnes peuvent‑elles utiliser un même numéro WhatsApp avec différentes instances OpenClaw ?
</div>

Oui, via le **routage multi‑agent**. Associez le **DM** WhatsApp de chaque expéditeur (pair `kind: "dm"`, expéditeur E.164 comme `+15551234567`) à un `agentId` différent, afin que chaque personne dispose de son propre espace de travail et de son propre stockage de sessions. Les réponses proviennent toujours du **même compte WhatsApp**, et le contrôle d’accès aux DM (`channels.whatsapp.dmPolicy` / `channels.whatsapp.allowFrom`) est global par compte WhatsApp. Voir [Routage multi‑agent](/fr/concepts/multi-agent) et [WhatsApp](/fr/channels/whatsapp).

<div id="can-i-run-a-fast-chat-agent-and-an-opus-for-coding-agent">
  ### Puis-je exécuter un agent de chat rapide et un agent Opus pour le code ?
</div>

Oui. Utilisez le routage multi‑agents : donnez à chaque agent son propre modèle par défaut, puis liez les routes entrantes (compte de fournisseur ou pairs spécifiques) à chaque agent. Un exemple de configuration se trouve dans [Routage multi‑agents](/fr/concepts/multi-agent). Voir aussi [Modèles](/fr/concepts/models) et [Configuration](/fr/gateway/configuration).

<div id="does-homebrew-work-on-linux">
  ### Homebrew fonctionne-t-il sous Linux ?
</div>

Oui. Homebrew prend en charge Linux (Linuxbrew). Configuration rapide :

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
echo 'eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"' >> ~/.profile
eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"
brew install <formula>
```

Si vous exécutez OpenClaw via systemd, assurez-vous que le PATH du service inclut `/home/linuxbrew/.linuxbrew/bin` (ou votre préfixe brew) afin que les outils installés via `brew` soient trouvés dans les shells non-login.

Les versions récentes ajoutent également en tête du PATH les répertoires bin utilisateur courants dans les services systemd Linux (par exemple `~/.local/bin`, `~/.npm-global/bin`, `~/.local/share/pnpm`, `~/.bun/bin`) et prennent en compte `PNPM_HOME`, `NPM_CONFIG_PREFIX`, `BUN_INSTALL`, `VOLTA_HOME`, `ASDF_DATA_DIR`, `NVM_DIR` et `FNM_DIR` lorsqu&#39;ils sont définis.

<div id="whats-the-difference-between-the-hackable-git-install-and-npm-install">
  ### Quelle est la différence entre l&#39;installation hackable via git et l&#39;installation via npm ?
</div>

* **Installation hackable (git)** : checkout complet du code source, modifiable, idéale pour les contributeurs.
  Vous exécutez les builds en local et pouvez patcher le code/la documentation.
* **Installation npm** : installation CLI globale, sans dépôt, idéale pour « juste l&#39;exécuter ».
  Les mises à jour proviennent des dist‑tags npm.

Docs : [Bien démarrer](/fr/start/getting-started), [Mise à jour](/fr/install/updating).

<div id="can-i-switch-between-npm-and-git-installs-later">
  ### Puis-je basculer plus tard entre une installation npm et une installation git ?
</div>

Oui. Installez l’autre méthode d’installation, puis exécutez Doctor pour que le service Gateway pointe vers le nouveau point d’entrée.
Cela **ne supprime pas vos données** : cela ne change que l’installation du code OpenClaw. Votre état
(`~/.openclaw`) et votre espace de travail (`~/.openclaw/workspace`) restent intacts.

Depuis npm → git :

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm build
openclaw doctor
openclaw gateway restart
```

De Git → npm :

```bash
npm install -g openclaw@latest
openclaw doctor
openclaw gateway restart
```

Doctor détecte une incohérence du point d&#39;entrée du service Gateway et propose de réécrire la configuration du service pour l’aligner sur l’installation actuelle (utilisez `--repair` dans vos automatisations).

Conseils de sauvegarde : voir [Stratégie de sauvegarde](/fr/help/faq#whats-the-recommended-backup-strategy).

<div id="should-i-run-the-gateway-on-my-laptop-or-a-vps">
  ### Dois‑je exécuter le Gateway sur mon ordinateur portable ou sur un VPS
</div>

Réponse courte : **si vous voulez une fiabilité 24/7, utilisez un VPS**. Si vous voulez
le moins de friction possible et que la mise en veille/les redémarrages vous conviennent, exécutez‑le en local.

**Ordinateur portable (Gateway local)**

* **Avantages :** aucun coût de serveur, accès direct aux fichiers locaux, fenêtre de navigateur visible en direct.
* **Inconvénients :** mise en veille/coupures réseau = déconnexions, mises à jour/redémarrages de l’OS qui interrompent tout, la machine doit rester éveillée.

**VPS / cloud**

* **Avantages :** toujours allumé, réseau stable, pas de problèmes de mise en veille de l’ordinateur portable, plus facile à garder en fonctionnement.
* **Inconvénients :** s’exécute souvent sans écran (utilisez des captures d’écran), accès distant uniquement aux fichiers, vous devez passer par SSH pour les mises à jour.

**Note spécifique à OpenClaw :** WhatsApp/Telegram/Slack/Mattermost (plugin)/Discord fonctionnent tous très bien depuis un VPS. Le seul vrai compromis est **navigateur headless** vs fenêtre visible. Voir [Browser](/fr/tools/browser).

**Recommandation par défaut :** VPS si vous avez déjà eu des déconnexions du Gateway. Le local est idéal quand vous utilisez activement le Mac et que vous voulez accéder aux fichiers locaux ou faire de l’automatisation de l’UI avec un navigateur visible.

<div id="how-important-is-it-to-run-openclaw-on-a-dedicated-machine">
  ### À quel point est‑il important d’exécuter OpenClaw sur une machine dédiée ?
</div>

Ce n’est pas obligatoire, mais **recommandé pour la fiabilité et l’isolation**.

* **Hôte dédié (VPS/Mac mini/Pi) :** toujours allumé, moins d’interruptions liées à la mise en veille/redémarrage, permissions plus propres, plus simple à maintenir en fonctionnement.
* **Ordinateur portable/de bureau partagé :** tout à fait adapté pour les tests et l’utilisation active, mais attendez‑vous à des interruptions lorsque la machine se met en veille ou se met à jour.

Si vous voulez le meilleur des deux mondes, gardez le Gateway sur un hôte dédié et associez votre ordinateur portable comme **nœud** pour les outils locaux d’écran/caméra/exec. Consultez [Nodes](/fr/nodes).
Pour des recommandations en matière de sécurité, consultez [Security](/fr/gateway/security).

<div id="what-are-the-minimum-vps-requirements-and-recommended-os">
  ### Quelles sont les exigences minimales pour un VPS et quel système d’exploitation est recommandé
</div>

OpenClaw est léger. Pour un Gateway simple avec un seul canal de chat :

* **Minimum absolu :** 1 vCPU, 1 Go de RAM, ~500 Mo de disque.
* **Recommandé :** 1-2 vCPU, 2 Go de RAM ou plus pour garder de la marge (journaux/logs, médias, canaux multiples). Les outils de nœud et l’automatisation du navigateur peuvent être gourmands en ressources.

OS : utilisez **Ubuntu LTS** (ou toute distribution Debian/Ubuntu moderne). La procédure d’installation Linux a été principalement testée dans cet environnement.

Docs : [Linux](/fr/platforms/linux), [VPS hosting](/fr/vps).

<div id="can-i-run-openclaw-in-a-vm-and-what-are-the-requirements">
  ### Puis-je exécuter OpenClaw dans une VM et quelles sont les exigences
</div>

Oui. Traitez une VM comme un VPS : elle doit être en fonctionnement en permanence, accessible, et disposer
de suffisamment de RAM pour le Gateway et tous les canaux que vous activez.

Recommandations de base :

* **Strict minimum :** 1 vCPU, 1 Go de RAM.
* **Recommandé :** 2 Go de RAM ou plus si vous exécutez plusieurs canaux, de l’automatisation de navigateur ou des outils multimédia.
* **OS :** Ubuntu LTS ou une autre distribution Debian/Ubuntu moderne.

Si vous êtes sous Windows, **WSL2 est la configuration de VM la plus simple** et offre la meilleure compatibilité
avec les outils. Voir [Windows](/fr/platforms/windows), [VPS hosting](/fr/vps).
Si vous exécutez macOS dans une VM, voir [macOS VM](/fr/platforms/macos-vm).

<div id="what-is-openclaw">
  ## Qu&#39;est-ce qu&#39;OpenClaw ?
</div>

<div id="what-is-openclaw-in-one-paragraph">
  ### Qu’est-ce qu’OpenClaw en un paragraphe
</div>

OpenClaw est un assistant IA personnel que vous exécutez sur vos propres appareils. Il répond sur les plateformes de messagerie que vous utilisez déjà (WhatsApp, Telegram, Slack, Mattermost (plugin), Discord, Google Chat, Signal, iMessage, WebChat) et peut aussi gérer la voix ainsi qu’un Canvas en direct sur les plateformes prises en charge. Le **Gateway** est le plan de contrôle toujours actif ; l’assistant est le produit.

<div id="whats-the-value-proposition">
  ### Quelle est la proposition de valeur ?
</div>

OpenClaw n’est pas « juste un wrapper Claude ». C’est un **plan de contrôle « local‑first »** qui vous permet d’exécuter un assistant
puissant sur **votre propre matériel**, accessible depuis les applis de messagerie que vous utilisez déjà, avec des
sessions avec état, de la mémoire et des outils – sans confier le contrôle de vos workflows à un SaaS hébergé.

Points clés :

* **Vos appareils, vos données :** exécutez le Gateway où vous voulez (Mac, Linux, VPS) et gardez l’historique
  d’espace de travail et de session en local.
* **De vrais canaux, pas un simple bac à sable web :** WhatsApp/Telegram/Slack/Discord/Signal/iMessage/etc.,
  plus la voix sur mobile et Canvas sur les plateformes prises en charge.
* **Indépendant des modèles :** utilisez Anthropic, OpenAI, MiniMax, OpenRouter, etc., avec routage
  et basculement par agent.
* **Option 100 % locale :** exécutez des modèles locaux pour que **toutes les données puissent rester sur votre appareil**
  si vous le souhaitez.
* **Routage multi‑agents :** agents séparés par canal, compte ou tâche, chacun avec son propre
  espace de travail et ses valeurs par défaut.
* **Open source et hackable :** inspectez, étendez et auto‑hébergez sans verrouillage propriétaire.

Docs : [Gateway](/fr/gateway), [Canaux](/fr/channels), [Multi‑agents](/fr/concepts/multi-agent),
[Mémoire](/fr/concepts/memory).

<div id="i-just-set-it-up-what-should-i-do-first">
  ### Je viens de le configurer, que dois-je faire en premier ?
</div>

Bons premiers projets :

* Créer un site web (WordPress, Shopify ou un simple site statique).
* Prototyper une application mobile (structure, écrans, plan d’API).
* Organiser vos fichiers et dossiers (nettoyage, nommage, étiquetage).
* Connecter Gmail et automatiser des résumés ou des relances.

Il peut gérer des tâches volumineuses, mais il fonctionne mieux si vous les découpez en phases et que vous utilisez des sous-agents pour le travail en parallèle.

<div id="what-are-the-top-five-everyday-use-cases-for-openclaw">
  ### Quels sont les cinq principaux cas d’usage quotidiens d’OpenClaw ?
</div>

Les bénéfices au quotidien ressemblent généralement à ceci :

* **Briefings personnels :** résumés de votre boîte de réception, de votre calendrier et des actualités qui comptent pour vous.
* **Recherche et rédaction :** recherche rapide, résumés et premiers jets pour des e‑mails ou des documents.
* **Rappels et relances :** rappels automatiques et listes de contrôle déclenchés par cron ou par un signal de vie.
* **Automatisation du navigateur :** remplissage de formulaires, collecte de données et répétition de tâches web.
* **Coordination multi‑appareils :** envoyez une tâche depuis votre téléphone, laissez le Gateway l’exécuter sur un serveur, puis récupérez le résultat dans le chat.

<div id="can-openclaw-help-with-lead-gen-outreach-ads-and-blogs-for-a-saas">
  ### OpenClaw peut‑il aider avec les publicités de prospection, la génération de leads et les blogs pour un SaaS
</div>

Oui pour **la recherche, la qualification et la rédaction de brouillons**. Il peut analyser des sites, constituer des listes restreintes,
résumer les prospects et rédiger des brouillons de messages de prospection ou de textes publicitaires.

Pour les **campagnes de prospection ou les diffusions publicitaires**, gardez toujours un humain dans la boucle. Évitez le spam, respectez les lois locales et
les politiques des plateformes, et relisez tout avant envoi. Le modèle le plus sûr consiste à laisser
OpenClaw rédiger et vous validez.

Docs : [Security](/fr/gateway/security).

<div id="what-are-the-advantages-vs-claude-code-for-web-development">
  ### Quels sont les avantages par rapport à Claude Code pour le développement web ?
</div>

OpenClaw est un **assistant personnel** et une couche de coordination, pas un remplacement d’IDE. Utilisez
Claude Code ou Codex pour la boucle de codage directe la plus rapide à l’intérieur d’un dépôt. Utilisez OpenClaw lorsque vous
voulez une mémoire durable, un accès multi‑appareil et une orchestration d’outils.

Avantages :

* **Mémoire persistante + espace de travail** d’une session à l’autre
* **Accès multiplateforme** (WhatsApp, Telegram, TUI, WebChat)
* **Orchestration d’outils** (navigateur, fichiers, planification, hooks)
* **Gateway toujours en fonctionnement** (faites‑le tourner sur un VPS, interagissez depuis n’importe où)
* **Nodes** pour navigateur/écran/caméra/exécution locaux

Présentation : https://openclaw.ai/showcase

<div id="skills-and-automation">
  ## Compétences et automatisation
</div>

<div id="how-do-i-customize-skills-without-keeping-the-repo-dirty">
  ### Comment personnaliser les compétences sans salir le dépôt
</div>

Utilisez des remplacements gérés plutôt que de modifier la copie du dépôt. Placez vos modifications dans `~/.openclaw/skills/<name>/SKILL.md` (ou ajoutez un dossier via `skills.load.extraDirs` dans `~/.openclaw/openclaw.json`). L’ordre de priorité est `<workspace>/skills` &gt; `~/.openclaw/skills` &gt; compétences fournies, de sorte que les remplacements gérés l’emportent sans toucher à Git. Seules les modifications réellement destinées à être remontées en amont devraient vivre dans le dépôt et sortir sous forme de PRs.

<div id="can-i-load-skills-from-a-custom-folder">
  ### Puis-je charger des compétences à partir d’un dossier personnalisé ?
</div>

Oui. Ajoutez des répertoires supplémentaires via `skills.load.extraDirs` dans `~/.openclaw/openclaw.json` (priorité la plus basse). L’ordre de priorité par défaut reste : `<workspace>/skills` → `~/.openclaw/skills` → compétences intégrées → `skills.load.extraDirs`. `clawhub` installe par défaut dans `./skills`, qu’OpenClaw traite comme `<workspace>/skills`.

<div id="how-can-i-use-different-models-for-different-tasks">
  ### Comment puis-je utiliser différents modèles pour différentes tâches
</div>

Actuellement, les scénarios suivants sont pris en charge :

* **Jobs cron** : des tâches isolées peuvent définir une valeur `model` spécifique par tâche.
* **Sous-agents** : acheminez les tâches vers des agents distincts avec des modèles par défaut différents.
* **Changement à la demande** : utilisez `/model` pour changer le modèle de la session en cours à tout moment.

Voir [Jobs cron](/fr/automation/cron-jobs), [Routage multi-agents](/fr/concepts/multi-agent) et [Commandes slash](/fr/tools/slash-commands).

<div id="the-bot-freezes-while-doing-heavy-work-how-do-i-offload-that">
  ### Le bot se fige pendant des tâches lourdes Comment déléguer ce travail
</div>

Utilisez des **sous-agents** pour les tâches longues ou parallèles. Les sous-agents s’exécutent dans leur propre session, renvoient un résumé et permettent à votre discussion principale de rester réactive.

Demandez à votre bot de « créer un sous-agent pour cette tâche » ou utilisez `/subagents`.
Utilisez `/status` dans le chat pour voir ce que le Gateway fait en ce moment (et s’il est occupé).

Astuce concernant les jetons : les tâches longues et les sous-agents consomment tous les deux des jetons. Si le coût est un problème, définissez un modèle moins cher pour les sous-agents via `agents.defaults.subagents.model`.

Docs : [Sous-agents](/fr/tools/subagents).

<div id="cron-or-reminders-do-not-fire-what-should-i-check">
  ### Cron ou les rappels ne se déclenchent pas : que dois-je vérifier ?
</div>

Cron s’exécute au sein du processus Gateway. Si le Gateway ne fonctionne pas en continu,
les tâches planifiées ne s’exécuteront pas.

Liste de contrôle :

* Assure-toi que cron est activé (`cron.enabled`) et que `OPENCLAW_SKIP_CRON` n’est pas défini.
* Vérifie que le Gateway fonctionne 24/7 (pas de mise en veille ni de redémarrages).
* Vérifie les paramètres de fuseau horaire pour la tâche (`--tz` par rapport au fuseau horaire de l’hôte).

Diagnostic :

```bash
openclaw cron run <jobId> --force
openclaw cron runs --id <jobId> --limit 50
```

Documentation : [Tâches cron](/fr/automation/cron-jobs), [Cron vs signal de vie](/fr/automation/cron-vs-heartbeat).

<div id="how-do-i-install-skills-on-linux">
  ### Comment installer des compétences sous Linux
</div>

Utilisez **ClawHub** (CLI) ou ajoutez des compétences dans votre espace de travail. L’UI Skills de macOS n’est pas disponible sous Linux.
Parcourez les compétences sur https://clawhub.com.

Installez la CLI ClawHub (sélectionnez un gestionnaire de paquets) :

```bash
npm i -g clawhub
```

```bash
pnpm add -g clawhub
```

<div id="can-openclaw-run-tasks-on-a-schedule-or-continuously-in-the-background">
  ### OpenClaw peut-il exécuter des tâches selon un planning ou en continu en arrière-plan
</div>

Oui. Utilisez le planificateur du Gateway :

* **Cron jobs** pour des tâches planifiées ou récurrentes (persistantes entre les redémarrages).
* **Heartbeat** pour des vérifications périodiques de la « session principale ».
* **Isolated jobs** pour des agents autonomes qui publient des résumés ou envoient des messages dans des chats.

Docs : [Cron jobs](/fr/automation/cron-jobs), [Cron vs Heartbeat](/fr/automation/cron-vs-heartbeat),
[Heartbeat](/fr/gateway/heartbeat).

**Puis-je exécuter depuis Linux des compétences réservées à Apple macOS**

Pas directement. Les compétences macOS sont conditionnées par `metadata.openclaw.os` ainsi que par les binaires requis, et les compétences n’apparaissent dans le prompt système que lorsqu’elles sont éligibles sur l’**hôte Gateway**. Sous Linux, les compétences réservées à `darwin` (comme `imsg`, `apple-notes`, `apple-reminders`) ne se chargeront pas, sauf si vous contournez ce filtrage.

Vous avez trois modèles pris en charge :

**Option A - exécuter le Gateway sur un Mac (le plus simple).**\
Exécutez le Gateway là où les binaires macOS existent, puis connectez-vous depuis Linux en [mode distant](#how-do-i-run-openclaw-in-remote-mode-client-connects-to-a-gateway-elsewhere) ou via Tailscale. Les compétences se chargent normalement, car l’hôte Gateway est macOS.

**Option B - utiliser un nœud macOS (sans SSH).**\
Exécutez le Gateway sur Linux, associez un nœud macOS (app de barre de menus), et réglez **Node Run Commands** sur « Always Ask » ou « Always Allow » sur le Mac. OpenClaw peut considérer les compétences réservées à macOS comme éligibles lorsque les binaires requis existent sur le nœud. L’agent exécute ces compétences via l’outil `nodes`. Si vous choisissez « Always Ask », approuver « Always Allow » dans le prompt ajoute cette commande à la liste d’autorisation.

**Option C - proxifier des binaires macOS via SSH (avancé).**\
Laissez le Gateway sur Linux, mais faites en sorte que les binaires CLI requis pointent vers des wrappers SSH qui s’exécutent sur un Mac. Puis redéfinissez la compétence pour autoriser Linux afin qu’elle reste éligible.

1. Créez un wrapper SSH pour le binaire (exemple : `imsg`) :
   ```bash
   #!/usr/bin/env bash
   set -euo pipefail
   exec ssh -T user@mac-host /opt/homebrew/bin/imsg "$@"
   ```
2. Placez le wrapper dans le `PATH` sur l’hôte Linux (par exemple `~/bin/imsg`).
3. Redéfinissez les métadonnées de la compétence (dans l’espace de travail ou `~/.openclaw/skills`) pour autoriser Linux :
   ```markdown
   ---
   name: imsg
   description: CLI iMessage/SMS pour lister les chats, l’historique, surveiller et envoyer.
   metadata: {"openclaw":{"os":["darwin","linux"],"requires":{"bins":["imsg"]}}}
   ---
   ```
4. Démarrez une nouvelle session pour que l’instantané des compétences soit actualisé.

Pour iMessage en particulier, vous pouvez aussi faire pointer `channels.imessage.cliPath` vers un wrapper SSH (OpenClaw a seulement besoin de stdio). Voir [iMessage](/fr/channels/imessage).

<div id="do-you-have-a-notion-or-heygen-integration">
  ### Avez‑vous une intégration Notion ou HeyGen ?
</div>

Pas d’intégration native à ce jour.

Options :

* **Compétence ou plugin personnalisé :** idéal pour un accès API fiable (Notion et HeyGen ont tous les deux des API).
* **Automatisation via le navigateur :** fonctionne sans code mais est plus lente et plus fragile.

Si vous voulez conserver le contexte par client (workflows d’agence), un modèle simple est :

* Une page Notion par client (contexte + préférences + travail en cours).
* Demander à l’agent de récupérer cette page au début d’une session.

Si vous souhaitez une intégration native, ouvrez une demande de fonctionnalité ou créez une compétence
ciblant ces API.

Installez des compétences :

```bash
clawhub install <skill-slug>
clawhub update --all
```

ClawHub s’installe dans `./skills` sous votre répertoire courant (ou, à défaut, dans votre espace de travail OpenClaw configuré) ; OpenClaw le considère comme `<workspace>/skills` lors de la session suivante. Pour partager des compétences entre plusieurs agents, placez-les dans `~/.openclaw/skills/<name>/SKILL.md`. Certaines compétences nécessitent des binaires installés via Homebrew ; sous Linux, cela signifie Linuxbrew (voir l’entrée de la FAQ Homebrew pour Linux ci-dessus). Voir [Skills](/fr/tools/skills) et [ClawHub](/fr/tools/clawhub).

<div id="how-do-i-install-the-chrome-extension-for-browser-takeover">
  ### Comment installer l’extension Chrome pour la prise de contrôle du navigateur ?
</div>

Utilisez le programme d’installation intégré, puis chargez l’extension non empaquetée dans Chrome :

```bash
openclaw browser extension install
openclaw browser extension path
```

Ensuite Chrome → `chrome://extensions` → activer « Developer mode » → « Load unpacked » → choisir ce dossier.

Guide complet (y compris avec un Gateway distant + notes de sécurité) : [Chrome extension](/fr/tools/chrome-extension)

Si le Gateway s’exécute sur la même machine que Chrome (configuration par défaut), vous **n’avez généralement pas** besoin de configuration supplémentaire.
Si le Gateway s’exécute ailleurs, exécutez un hôte de nœud sur la machine du navigateur afin que le Gateway puisse relayer (proxy) les actions du navigateur.
Vous devez tout de même cliquer sur le bouton de l’extension dans l’onglet que vous voulez contrôler (elle ne s’attache pas automatiquement).

<div id="sandboxing-and-memory">
  ## Sandboxing et mémoire
</div>

<div id="is-there-a-dedicated-sandboxing-doc">
  ### Existe-t-il une documentation dédiée au sandboxing
</div>

Oui. Voir [Sandboxing](/fr/gateway/sandboxing). Pour la configuration spécifique à Docker (Gateway complet dans Docker ou images de sandbox), voir [Docker](/fr/install/docker).

**Puis-je garder les DMs personnels mais rendre les groupes publics, en sandbox, avec un seul agent**

Oui, si votre trafic privé est constitué de **DMs** et votre trafic public de **groupes**.

Utilisez `agents.defaults.sandbox.mode: "non-main"` pour que les sessions de groupe/canal (clés non-main) s&#39;exécutent dans Docker, tandis que la session DM principale reste sur l&#39;hôte. Restreignez ensuite les outils disponibles dans les sessions en sandbox via `tools.sandbox.tools`.

Guide de configuration + exemple de configuration : [Groups: personal DMs + public groups](/fr/concepts/groups#pattern-personal-dms-public-groups-single-agent)

Référence de configuration principale : [Gateway configuration](/fr/gateway/configuration#agentsdefaultssandbox)

<div id="how-do-i-bind-a-host-folder-into-the-sandbox">
  ### Comment puis-je monter un dossier de l’hôte dans le sandbox
</div>

Définissez `agents.defaults.sandbox.docker.binds` sur `["host:path:mode"]` (par exemple `"/home/user/src:/src:ro"`). Les montages globaux et par agent sont fusionnés ; les montages par agent sont ignorés lorsque `scope: "shared"`. Utilisez `:ro` pour tout ce qui est sensible et gardez à l’esprit que ces montages contournent les barrières d’isolement du système de fichiers du sandbox. Consultez [Sandboxing](/fr/gateway/sandboxing#custom-bind-mounts) et [Sandbox vs Tool Policy vs Elevated](/fr/gateway/sandbox-vs-tool-policy-vs-elevated#bind-mounts-security-quick-check) pour des exemples et des consignes de sécurité.

<div id="how-does-memory-work">
  ### Comment fonctionne la mémoire
</div>

La mémoire d’OpenClaw consiste simplement en des fichiers Markdown dans l’espace de travail de l’agent :

* Notes quotidiennes dans `memory/YYYY-MM-DD.md`
* Notes de long terme organisées dans `MEMORY.md` (sessions principales/privées uniquement)

OpenClaw exécute également un **vidage silencieux de la mémoire avant la pré-compaction** pour rappeler au modèle
d’écrire des notes persistantes avant l’auto-compaction. Cela ne s’exécute que lorsque l’espace de travail
est modifiable en écriture (les sandboxes en lecture seule l’ignorent). Voir [Mémoire](/fr/concepts/memory).

<div id="memory-keeps-forgetting-things-how-do-i-make-it-stick">
  ### La mémoire oublie sans cesse des choses : comment la rendre persistante ?
</div>

Demandez au bot d’**écrire ce fait en mémoire**. Les notes à long terme vont dans `MEMORY.md`,
le contexte à court terme va dans `memory/YYYY-MM-DD.md`.

C’est encore un aspect que nous sommes en train d’améliorer. Il est utile de rappeler au modèle de stocker les éléments en mémoire ; il saura quoi faire. S’il continue d’oublier, vérifiez que le Gateway utilise le même
espace de travail à chaque exécution.

Docs : [Mémoire](/fr/concepts/memory), [Espace de travail de l’Agent](/fr/concepts/agent-workspace).

<div id="does-semantic-memory-search-require-an-openai-api-key">
  ### La recherche en mémoire sémantique nécessite-t-elle une clé API OpenAI ?
</div>

Uniquement si vous utilisez les **embeddings OpenAI**. Codex OAuth couvre le chat/les complétions et ne donne **pas** accès aux embeddings, donc **vous connecter avec Codex (OAuth ou la commande de connexion Codex CLI)** n’aide pas pour la recherche en mémoire sémantique. Les embeddings OpenAI nécessitent toujours une véritable clé API (`OPENAI_API_KEY` ou `models.providers.openai.apiKey`).

Si vous ne définissez pas explicitement un fournisseur, OpenClaw sélectionne automatiquement un fournisseur lorsqu’il peut résoudre une clé API (profils d’authentification, `models.providers.*.apiKey` ou variables d’environnement). Il privilégie OpenAI si une clé OpenAI est trouvée, sinon Gemini si une clé Gemini est trouvée. Si aucune des deux clés n’est disponible, la recherche en mémoire reste désactivée jusqu’à ce que vous la configuriez. Si vous avez un chemin de modèle local configuré et présent, OpenClaw privilégie `local`.

Si vous préférez rester en local, définissez `memorySearch.provider = "local"` (et éventuellement
`memorySearch.fallback = "none"`). Si vous voulez des embeddings Gemini, définissez
`memorySearch.provider = "gemini"` et fournissez `GEMINI_API_KEY` (ou
`memorySearch.remote.apiKey`). Nous prenons en charge des modèles d’embeddings **OpenAI, Gemini ou locaux** – consultez **[Memory](/fr/concepts/memory)** pour les détails de configuration.

<div id="does-memory-persist-forever-what-are-the-limits">
  ### La mémoire persiste-t-elle indéfiniment ? Quelles sont les limites ?
</div>

Les fichiers de mémoire sont enregistrés sur le disque et persistent jusqu’à ce que vous les supprimiez. La limite, c’est votre
espace de stockage, pas le modèle. Le **contexte de session** reste limité par la fenêtre de contexte du modèle,
donc les longues conversations peuvent être compactées ou tronquées. C’est pour cela qu’il existe une recherche dans la mémoire : elle ne réinjecte dans le contexte que les parties pertinentes.

Docs : [Mémoire](/fr/concepts/memory), [Contexte](/fr/concepts/context).

<div id="where-things-live-on-disk">
  ## Emplacement des données sur le disque
</div>

<div id="is-all-data-used-with-openclaw-saved-locally">
  ### Toutes les données utilisées avec OpenClaw sont-elles enregistrées localement ?
</div>

Non – **l’état d’OpenClaw est local**, mais **les services externes voient toujours ce que vous leur envoyez**.

* **Local par défaut :** les sessions, fichiers de mémoire, configuration et espace de travail résident sur l’hôte Gateway
  (`~/.openclaw` + votre répertoire d’espace de travail).
* **À distance par nécessité :** les messages que vous envoyez aux fournisseurs de modèles (Anthropic/OpenAI/etc.) vont vers
  leurs API, et les plateformes de messagerie (WhatsApp/Telegram/Slack/etc.) stockent les données de messagerie sur
  leurs serveurs.
* **Vous contrôlez l’empreinte de vos données :** utiliser des modèles locaux garde les prompts sur votre machine, mais le trafic des canaux
  passe toujours par les serveurs de ces canaux.

Voir aussi : [Agent workspace](/fr/concepts/agent-workspace), [Memory](/fr/concepts/memory).

<div id="where-does-openclaw-store-its-data">
  ### Où OpenClaw stocke-t-il ses données ?
</div>

Tout se trouve dans `$OPENCLAW_STATE_DIR` (par défaut : `~/.openclaw`) :

| Path | Purpose |
|------|---------|
| `$OPENCLAW_STATE_DIR/openclaw.json` | Configuration principale (JSON5) |
| `$OPENCLAW_STATE_DIR/credentials/oauth.json` | Import OAuth hérité (copié dans les profils d’authentification lors de la première utilisation) |
| `$OPENCLAW_STATE_DIR/agents/<agentId>/agent/auth-profiles.json` | Profils d’authentification (OAuth + clés API) |
| `$OPENCLAW_STATE_DIR/agents/<agentId>/agent/auth.json` | Cache d’authentification d’exécution (géré automatiquement) |
| `$OPENCLAW_STATE_DIR/credentials/` | État du fournisseur (par ex. `whatsapp/<accountId>/creds.json`) |
| `$OPENCLAW_STATE_DIR/agents/` | État propre à chaque agent (agentDir + sessions) |
| `$OPENCLAW_STATE_DIR/agents/<agentId>/sessions/` | Historique des conversations et état (par agent) |
| `$OPENCLAW_STATE_DIR/agents/<agentId>/sessions/sessions.json` | Métadonnées de session (par agent) |

Ancien chemin pour un agent unique : `~/.openclaw/agent/*` (migré par `openclaw doctor`).

Votre **espace de travail** (AGENTS.md, fichiers de mémoire, compétences, etc.) est distinct et configuré via `agents.defaults.workspace` (par défaut : `~/.openclaw/workspace`).

<div id="where-should-agentsmd-soulmd-usermd-memorymd-live">
  ### Où doivent se trouver AGENTSmd SOULmd USERmd MEMORYmd
</div>

Ces fichiers se trouvent dans **l’espace de travail de l’agent**, pas dans `~/.openclaw`.

* **Espace de travail (par agent)** : `AGENTS.md`, `SOUL.md`, `IDENTITY.md`, `USER.md`,
  `MEMORY.md` (ou `memory.md`), `memory/YYYY-MM-DD.md`, `HEARTBEAT.md` facultatif.
* **Répertoire d’état (`~/.openclaw`)** : configuration, identifiants, profils d’authentification, sessions, journaux
  et compétences partagées (`~/.openclaw/skills`).

L’espace de travail par défaut est `~/.openclaw/workspace`, configurable via :

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } }
}
```

Si le bot « oublie » après un redémarrage, vérifiez que Gateway utilise le même
espace de travail à chaque lancement (et rappelez-vous : le mode distant utilise l’**espace de travail de l’hôte du Gateway**,
pas celui de votre ordinateur portable).

Astuce : si vous voulez un comportement ou une préférence durables, demandez au bot de **les écrire dans
AGENTS.md ou MEMORY.md** plutôt que de compter sur l’historique de conversation.

Voir [espace de travail de l’Agent](/fr/concepts/agent-workspace) et [Mémoire](/fr/concepts/memory).

<div id="whats-the-recommended-backup-strategy">
  ### Quelle est la stratégie de sauvegarde recommandée ?
</div>

Placez votre **espace de travail d’agent** dans un dépôt Git **privé** et sauvegardez-le
dans un emplacement privé (par exemple un dépôt GitHub privé). Cela capture la mémoire ainsi que les fichiers AGENTS/SOUL/USER,
et vous permet de restaurer « l’esprit » de l’assistant plus tard.

Ne validez **rien** qui se trouve sous `~/.openclaw` (identifiants, sessions, jetons).
Si vous avez besoin d’une restauration complète, sauvegardez à la fois l’espace de travail et le répertoire d’état
séparément (voir la question sur la migration ci-dessus).

Docs : [Espace de travail d’agent](/fr/concepts/agent-workspace).

<div id="how-do-i-completely-uninstall-openclaw">
  ### Comment désinstaller complètement OpenClaw ?
</div>

Voir le guide dédié : [Désinstallation](/fr/install/uninstall).

<div id="can-agents-work-outside-the-workspace">
  ### Les agents peuvent‑ils travailler en dehors de l’espace de travail
</div>

Oui. L’espace de travail est le **cwd par défaut** et l’ancrage de mémoire, pas un sandbox strict.
Les chemins relatifs se résolvent à l’intérieur de l’espace de travail, mais les chemins absolus
peuvent accéder à d’autres emplacements de l’hôte, sauf si le sandboxing est activé. Si vous avez
besoin d’isolation, utilisez [`agents.defaults.sandbox`](/fr/gateway/sandboxing) ou des paramètres
de sandbox par agent. Si vous voulez qu’un dépôt soit le répertoire de travail par défaut, pointez
le `workspace` de cet agent vers la racine du dépôt. Le dépôt OpenClaw n’est que du code source ;
gardez l’espace de travail séparé, sauf si vous voulez délibérément que l’agent travaille à
l’intérieur.

Exemple (dépôt comme cwd par défaut) :

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
  ### Je suis en mode distant : où se trouve le stockage des sessions ?
</div>

L’état des sessions est géré par l’**hôte du Gateway**. Si vous êtes en mode distant, le stockage des sessions qui vous intéresse se trouve sur la machine distante, pas sur votre ordinateur portable local. Voir [Gestion des sessions](/fr/concepts/session).

<div id="config-basics">
  ## Bases de la configuration
</div>

<div id="what-format-is-the-config-where-is-it">
  ### Quel est le format du fichier de configuration et où se trouve-t-il ?
</div>

OpenClaw lit un fichier de configuration **JSON5** optionnel à partir de `$OPENCLAW_CONFIG_PATH` (par défaut : `~/.openclaw/openclaw.json`) :

```
$OPENCLAW_CONFIG_PATH
```

Si le fichier est absent, il utilise des valeurs par défaut relativement sûres (dont un espace de travail par défaut `~/.openclaw/workspace`).

<div id="i-set-gatewaybind-lan-or-tailnet-and-now-nothing-listens-the-ui-says-unauthorized">
  ### J’ai défini gatewaybind sur lan ou tailnet et maintenant rien n’écoute, l’UI affiche « unauthorized »
</div>

Les liaisons non-loopback **requièrent une authentification**. Configurez `gateway.auth.mode` + `gateway.auth.token` (ou utilisez `OPENCLAW_GATEWAY_TOKEN`).

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

Notes :

* `gateway.remote.token` est uniquement destiné aux **appels CLI distants** ; il n’active pas l’authentification locale sur le Gateway.
* Le Control UI s’authentifie via `connect.params.auth.token` (stocké dans les paramètres de l’application/UI). Évitez de placer des jetons dans les URL.

<div id="why-do-i-need-a-token-on-localhost-now">
  ### Pourquoi ai-je désormais besoin d’un jeton sur localhost
</div>

L’assistant génère un jeton de Gateway par défaut (même sur l’interface loopback), donc **les clients WS locaux doivent s’authentifier**. Cela empêche les autres processus locaux d’appeler la Gateway. Collez le jeton dans les paramètres de la Control UI (ou dans la configuration de votre client) pour vous connecter.

Si vous voulez **vraiment** laisser le loopback ouvert, supprimez `gateway.auth` de votre configuration. La commande Doctor peut générer un jeton pour vous à tout moment : `openclaw doctor --generate-gateway-token`.

<div id="do-i-have-to-restart-after-changing-config">
  ### Dois‑je redémarrer après avoir modifié la configuration
</div>

Le Gateway surveille la configuration et prend en charge le rechargement à chaud :

* `gateway.reload.mode: "hybrid"` (par défaut) : applique à chaud les modifications sans risque, redémarre pour les modifications critiques
* `hot`, `restart`, `off` sont également pris en charge

<div id="how-do-i-enable-web-search-and-web-fetch">
  ### Comment activer la recherche web et la récupération web
</div>

`web_fetch` fonctionne sans clé d’API. `web_search` nécessite une clé d’API Brave Search. **Recommandé :** exécutez `openclaw configure --section web` pour l’enregistrer dans `tools.web.search.apiKey`. Alternative via variable d’environnement : définissez `BRAVE_API_KEY` pour le processus Gateway.

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

Remarques :

* Si vous utilisez des listes d’autorisation, ajoutez `web_search`/`web_fetch` ou `group:web`.
* `web_fetch` est activé par défaut (sauf si vous le désactivez explicitement).
* Les démons lisent les variables d’environnement depuis `~/.openclaw/.env` (ou à partir de l’environnement du service).

Docs : [Outils Web](/fr/tools/web).

<div id="how-do-i-run-a-central-gateway-with-specialized-workers-across-devices">
  ### Comment exécuter un Gateway central avec des workers spécialisés sur plusieurs appareils
</div>

Le schéma courant est **un Gateway** (par ex. Raspberry Pi) plus des **nœuds** et des **agents** :

* **Gateway (central)** : gère les canaux (Signal/WhatsApp), le routage et les sessions.
* **Nœuds (appareils)** : les Macs/iOS/Android se connectent comme périphériques et exposent des outils locaux (`system.run`, `canvas`, `camera`).
* **Agents (workers)** : cerveaux/espaces de travail séparés pour des rôles spécialisés (par ex. « Hetzner ops », « Données personnelles »).
* **Sous‑agents** : lancent du travail en arrière‑plan depuis un agent principal lorsque vous souhaitez du parallélisme.
* **TUI** : se connecte au Gateway et permet de changer d’agent/session.

Docs : [Nodes](/fr/nodes), [Remote access](/fr/gateway/remote), [Multi-Agent Routing](/fr/concepts/multi-agent), [Sub-agents](/fr/tools/subagents), [TUI](/fr/tui).

<div id="can-the-openclaw-browser-run-headless">
  ### Le navigateur OpenClaw peut-il s’exécuter en mode sans interface graphique (headless) ?
</div>

Oui. C’est une option de configuration :

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

La valeur par défaut est `false` (avec interface graphique). Le mode headless est plus susceptible de déclencher des contrôles anti‑bots sur certains sites. Voir [Browser](/fr/tools/browser).

Le mode headless utilise le **même moteur Chromium** et fonctionne pour la plupart des tâches d’automatisation (formulaires, clics, scraping, connexions). Les principales différences :

* Aucune fenêtre de navigateur affichée (utilisez des captures d’écran si vous avez besoin de visuels).
* Certains sites sont plus stricts concernant l’automatisation en mode headless (CAPTCHAs, mécanismes anti‑bot).
  Par exemple, X/Twitter bloque souvent les sessions en mode headless.

<div id="how-do-i-use-brave-for-browser-control">
  ### Comment utiliser Brave pour le contrôle du navigateur
</div>

Définissez `browser.executablePath` sur le binaire Brave (ou tout autre navigateur basé sur Chromium) et redémarrez Gateway.
Voir des exemples de configuration complets dans la section [Browser](/fr/tools/browser#use-brave-or-another-chromium-based-browser).

<div id="remote-gateways-nodes">
  ## Gateways et nœuds distants
</div>

<div id="how-do-commands-propagate-between-telegram-the-gateway-and-nodes">
  ### Comment les commandes sont-elles propagées entre Telegram, le Gateway et les nœuds
</div>

Les messages Telegram sont gérés par le **Gateway**. Le Gateway exécute l’agent, puis appelle les nœuds via le **Gateway WebSocket** uniquement lorsqu’un outil de nœud est nécessaire :

Telegram → Gateway → Agent → `node.*` → Nœud → Gateway → Telegram

Les nœuds ne voient pas le trafic entrant provenant des fournisseurs ; ils ne reçoivent que des appels RPC de nœud.

<div id="how-can-my-agent-access-my-computer-if-the-gateway-is-hosted-remotely">
  ### Comment votre agent peut-il accéder à votre ordinateur si le Gateway est hébergé à distance
</div>

Réponse courte : **appairez votre ordinateur en tant que nœud**. Le Gateway s’exécute ailleurs, mais il peut
appeler des outils `node.*` (écran, caméra, système) sur votre machine locale via le WebSocket du Gateway.

Configuration typique :

1. Exécutez le Gateway sur l’hôte toujours allumé (VPS/serveur à domicile).
2. Placez l’hôte du Gateway et votre ordinateur sur le même tailnet.
3. Assurez-vous que le WS du Gateway est joignable (bind tailnet ou tunnel SSH).
4. Ouvrez l’app macOS en local et connectez-vous en mode **Remote over SSH** (ou tailnet direct)
   pour qu’elle puisse s’enregistrer comme nœud.
5. Approuvez le nœud sur le Gateway :
   ```bash
   openclaw nodes pending
   openclaw nodes approve <requestId>
   ```

Aucun pont TCP séparé n’est requis ; les nœuds se connectent via le WebSocket du Gateway.

Rappel de sécurité : l’appairage d’un nœud macOS autorise `system.run` sur cette machine. N’appairez
que des appareils de confiance et consultez la page [Security](/fr/gateway/security).

Docs : [Nodes](/fr/nodes), [Gateway protocol](/fr/gateway/protocol), [macOS remote mode](/fr/platforms/mac/remote), [Security](/fr/gateway/security).

<div id="tailscale-is-connected-but-i-get-no-replies-what-now">
  ### Tailscale est connecté mais je ne reçois aucune réponse : que faire ?
</div>

Vérifiez les éléments de base :

* Gateway est en cours d’exécution : `openclaw gateway status`
* Statut du Gateway : `openclaw status`
* Statut des canaux : `openclaw channels status`

Ensuite, vérifiez l’authentification et le routage :

* Si vous utilisez Tailscale Serve, assurez-vous que `gateway.auth.allowTailscale` est correctement configuré.
* Si vous vous connectez via un tunnel SSH, confirmez que le tunnel local est actif et pointe vers le bon port.
* Vérifiez que vos listes d’autorisation (DM ou groupe) incluent bien votre compte.

Docs : [Tailscale](/fr/gateway/tailscale), [Accès distant](/fr/gateway/remote), [Canaux](/fr/channels).

<div id="can-two-openclaw-instances-talk-to-each-other-local-vps">
  ### Deux instances OpenClaw peuvent-elles communiquer entre elles sur un VPS local ?
</div>

Oui. Il n’y a pas de pont intégré « bot-to-bot », mais vous pouvez en configurer un de plusieurs façons fiables :

**La plus simple :** utilisez un canal de discussion normal auquel les deux bots ont accès (Telegram/Slack/WhatsApp).
Faites envoyer un message par le bot A au bot B, puis laissez le bot B répondre comme d’habitude.

**Pont CLI (générique) :** exécutez un script qui appelle l’autre Gateway avec
`openclaw agent --message ... --deliver`, en ciblant une conversation où l’autre bot
écoute. Si un bot est sur un VPS distant, faites pointer votre CLI vers cette Gateway distante
via SSH/Tailscale (voir [Accès distant](/fr/gateway/remote)).

Exemple type (à exécuter depuis une machine qui peut atteindre la Gateway cible) :

```bash
openclaw agent --message "Bonjour du bot local" --deliver --channel telegram --reply-to <chat-id>
```

Astuce : ajoutez un garde-fou pour empêcher les deux bots de boucler indéfiniment (mentions uniquement, listes d’autorisation de canaux ou règle « ne pas répondre aux messages des bots »).

Docs : [Accès distant](/fr/gateway/remote), [Agent CLI](/fr/cli/agent), [Agent send](/fr/tools/agent-send).

<div id="do-i-need-separate-vpses-for-multiple-agents">
  ### Ai-je besoin de VPS distincts pour plusieurs agents ?
</div>

Non. Un seul Gateway peut héberger plusieurs agents, chacun avec son propre espace de travail, ses paramètres de modèle par défaut
et son propre routage. C’est la configuration normale, et elle est bien moins coûteuse et plus simple que d’exécuter
un VPS par agent.

N’utilisez des VPS distincts que lorsque vous avez besoin d’une isolation forte (limites de sécurité strictes) ou de
configurations très différentes que vous ne voulez pas partager. Sinon, gardez un seul Gateway et
utilisez plusieurs agents ou sous-agents.

<div id="is-there-a-benefit-to-using-a-node-on-my-personal-laptop-instead-of-ssh-from-a-vps">
  ### Y a-t-il un avantage à utiliser un nœud sur mon ordinateur portable personnel plutôt que d’utiliser SSH depuis un VPS ?
</div>

Oui : les nœuds sont le mécanisme principal pour atteindre votre ordinateur portable depuis un Gateway distant, et ils
offrent bien plus que l’accès au shell. Le Gateway s’exécute sur macOS/Linux (Windows via WSL2) et est
léger (un petit VPS ou une machine de type Raspberry Pi suffit ; 4 Go de RAM sont largement suffisants), donc une
configuration courante consiste en un hôte toujours actif plus votre ordinateur portable en tant que nœud.

* **Aucun SSH entrant requis.** Les nœuds se connectent au Gateway via WS et utilisent l’appairage de l’appareil.
* **Contrôles d’exécution plus sûrs.** `system.run` est contrôlé par des listes d’autorisation et des approbations de nœud sur cet ordinateur portable.
* **Plus d’outils pour l’appareil.** Les nœuds exposent `canvas`, `camera` et `screen` en plus de `system.run`.
* **Automatisation locale du navigateur.** Gardez le Gateway sur un VPS, mais exécutez Chrome localement et relayer les commandes
  avec l’extension Chrome + un hôte de nœud sur l’ordinateur portable.

SSH convient pour un accès shell ponctuel, mais les nœuds sont plus simples pour des workflows d’agents continus et
l’automatisation de l’appareil.

Docs : [Nodes](/fr/nodes), [Nodes CLI](/fr/cli/nodes), [Chrome extension](/fr/tools/chrome-extension).

<div id="should-i-install-on-a-second-laptop-or-just-add-a-node">
  ### Dois-je installer sur un deuxième ordinateur portable ou simplement ajouter un nœud ?
</div>

Si vous avez seulement besoin d’**outils locaux** (screen/camera/exec) sur le deuxième ordinateur portable, ajoutez-le comme
**nœud**. Cela vous permet de conserver un seul Gateway et d’éviter la duplication de configuration. Les outils locaux côté nœud sont
actuellement disponibles uniquement sur macOS, mais nous prévoyons de les étendre à d’autres systèmes d’exploitation.

N’installez un deuxième Gateway que si vous avez besoin d’une **isolation stricte** ou de deux bots totalement distincts.

Docs : [Nodes](/fr/nodes), [Nodes CLI](/fr/cli/nodes), [Multiple gateways](/fr/gateway/multiple-gateways).

<div id="do-nodes-run-a-gateway-service">
  ### Les nœuds exécutent-ils un service de Gateway ?
</div>

Non. Une **seule Gateway** doit être exécutée par hôte, sauf si vous faites tourner intentionnellement des profils isolés (voir [Multiple gateways](/fr/gateway/multiple-gateways)). Les nœuds sont des périphériques qui se connectent
à la Gateway (nœuds iOS/Android, ou « mode nœud » macOS dans l’application de barre de menus). Pour les hôtes de nœud sans interface graphique et le contrôle via la CLI, voir [Node host CLI](/fr/cli/node).

Un redémarrage complet est nécessaire pour les changements de `gateway`, `discovery` et `canvasHost`.

<div id="is-there-an-api-rpc-way-to-apply-config">
  ### Existe-t-il une API RPC pour appliquer la configuration ?
</div>

Oui. `config.apply` valide et écrit la configuration complète, puis redémarre le Gateway dans le cadre de cette opération.

<div id="configapply-wiped-my-config-how-do-i-recover-and-avoid-this">
  ### configapply a effacé ma configuration Comment la restaurer et éviter que ça se reproduise ?
</div>

`config.apply` remplace **toute la configuration**. Si vous envoyez un objet partiel, tout
le reste est supprimé.

Pour la restaurer :

* Restaurez à partir d’une sauvegarde (git ou une copie de `~/.openclaw/openclaw.json`).
* Si vous n’avez aucune sauvegarde, relancez `openclaw doctor` et reconfigurez les canaux/modèles.
* Si cela était inattendu, signalez un bug et incluez votre dernière configuration connue ou toute sauvegarde.
* Un agent de développement local peut souvent reconstruire une configuration fonctionnelle à partir des journaux ou de l’historique.

Pour éviter cela :

* Utilisez `openclaw config set` pour les petits changements.
* Utilisez `openclaw configure` pour les modifications interactives.

Docs : [Config](/fr/cli/config), [Configure](/fr/cli/configure), [Doctor](/fr/gateway/doctor).

<div id="whats-a-minimal-sane-config-for-a-first-install">
  ### Quelle configuration minimale est recommandée pour une première installation
</div>

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
  channels: { whatsapp: { allowFrom: ["+15555550123"] } }
}
```

Cela définit votre espace de travail et limite qui peut déclencher le bot.

<div id="how-do-i-set-up-tailscale-on-a-vps-and-connect-from-my-mac">
  ### Comment configurer Tailscale sur un VPS et y connecter votre Mac
</div>

Étapes minimales :

1. **Installer + se connecter sur le VPS**
   ```bash
   curl -fsSL https://tailscale.com/install.sh | sh
   sudo tailscale up
   ```
2. **Installer + se connecter sur votre Mac**
   * Utilisez l’application Tailscale et connectez‑vous au même tailnet.
3. **Activer MagicDNS (recommandé)**
   * Dans la console d’administration Tailscale, activez MagicDNS pour que le VPS ait un nom d’hôte stable.
4. **Utiliser le nom d’hôte du tailnet**
   * SSH : `ssh user@your-vps.tailnet-xxxx.ts.net`
   * Gateway WS : `ws://your-vps.tailnet-xxxx.ts.net:18789`

Si vous voulez accéder à la Control UI sans SSH, utilisez Tailscale Serve sur le VPS :

```bash
openclaw gateway --tailscale serve
```

Cela maintient le Gateway sur la boucle locale et expose HTTPS via Tailscale. Voir [Tailscale](/fr/gateway/tailscale).

<div id="how-do-i-connect-a-mac-node-to-a-remote-gateway-tailscale-serve">
  ### Comment connecter un nœud Mac à un Gateway distant via Tailscale Serve
</div>

Serve expose la **Control UI du Gateway + WS**. Les nœuds se connectent via le même point de terminaison WS du Gateway.

Configuration recommandée :

1. **Assurez-vous que le VPS et le Mac sont sur le même tailnet**.
2. **Utilisez l’app macOS en mode Remote** (la cible SSH peut être le nom d’hôte du tailnet).
   L’app va établir un tunnel vers le port du Gateway et se connecter en tant que nœud.
3. **Approuvez le nœud** sur le Gateway :
   ```bash
   openclaw nodes pending
   openclaw nodes approve <requestId>
   ```

Docs : [Gateway protocol](/fr/gateway/protocol), [Discovery](/fr/gateway/discovery), [macOS remote mode](/fr/platforms/mac/remote).

<div id="env-vars-and-env-loading">
  ## Variables d&#39;environnement et chargement du fichier .env
</div>

<div id="how-does-openclaw-load-environment-variables">
  ### Comment OpenClaw charge-t-il les variables d&#39;environnement ?
</div>

OpenClaw lit les variables d&#39;environnement à partir du processus parent (shell, launchd/systemd, CI, etc.) et charge en plus :

* `.env` depuis le répertoire de travail courant
* un fichier `.env` global de repli depuis `~/.openclaw/.env` (alias `$OPENCLAW_STATE_DIR/.env`)

Aucun des fichiers `.env` ne remplace les variables d&#39;environnement existantes.

Vous pouvez également définir des variables d&#39;environnement en ligne dans la configuration (appliquées uniquement si elles sont absentes de l&#39;environnement du processus) :

```json5
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: { GROQ_API_KEY: "gsk-..." }
  }
}
```

Voir [/environment](/fr/environment) pour connaître l’ordre de priorité complet et les sources.

<div id="i-started-the-gateway-via-the-service-and-my-env-vars-disappeared-what-now">
  ### J’ai démarré le Gateway via le service et mes variables d’environnement ont disparu Que faire maintenant
</div>

Deux solutions courantes :

1. Place les clés manquantes dans `~/.openclaw/.env` pour qu’elles soient prises en compte même lorsque le service n’hérite pas de l’environnement de ton shell.
2. Active l’import du shell (option pratique à activer explicitement) :

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

Cela exécute votre shell de connexion et importe uniquement les clés attendues manquantes (sans jamais écraser les existantes). Équivalents pour les variables d&#39;environnement :
`OPENCLAW_LOAD_SHELL_ENV=1`, `OPENCLAW_SHELL_ENV_TIMEOUT_MS=15000`.

<div id="i-set-copilotgithubtoken-but-models-status-shows-shell-env-off-why">
  ### J’ai défini COPILOTGITHUBTOKEN mais `models status` indique « Shell env off » : pourquoi ?
</div>

`openclaw models status` indique si **l’import de l’environnement du shell** est activé. « Shell env : off »
ne signifie **pas** que vos variables d’environnement sont absentes : cela signifie simplement que OpenClaw ne chargera pas
automatiquement votre shell de connexion.

Si le Gateway s’exécute comme un service (launchd/systemd), il n’héritera pas de votre environnement
de shell. Corrigez cela en procédant de l’une des manières suivantes :

1. Placez le token dans `~/.openclaw/.env` :
   ```
   COPILOT_GITHUB_TOKEN=...
   ```
2. Ou activez l’import de l’environnement du shell (`env.shellEnv.enabled: true`).
3. Ou ajoutez-le dans le bloc `env` de votre configuration (ne s’applique que s’il est manquant).

Ensuite, redémarrez le Gateway et revérifiez :

```bash
openclaw models status
```

Les jetons Copilot sont lus depuis `COPILOT_GITHUB_TOKEN` (ainsi que `GH_TOKEN` / `GITHUB_TOKEN`).
Consultez [/concepts/model-providers](/fr/concepts/model-providers) et [/environment](/fr/environment).

<div id="sessions-multiple-chats">
  ## Sessions et conversations multiples
</div>

<div id="how-do-i-start-a-fresh-conversation">
  ### Comment démarrer une nouvelle conversation
</div>

Envoyez `/new` ou `/reset` dans un message autonome. Voir [Gestion des sessions](/fr/concepts/session).

<div id="do-sessions-reset-automatically-if-i-never-send-new">
  ### Les sessions se réinitialisent-elles automatiquement si je n&#39;envoie jamais de nouveau message ?
</div>

Oui. Les sessions expirent après `session.idleMinutes` (par défaut **60**). Le message
suivant démarre un nouvel ID de session pour cette clé de conversation. Cela ne supprime pas
les transcriptions — cela démarre simplement une nouvelle session.

```json5
{
  session: {
    idleMinutes: 240
  }
}
```

<div id="is-there-a-way-to-make-a-team-of-openclaw-instances-one-ceo-and-many-agents">
  ### Existe-t-il un moyen de créer une équipe d’instances OpenClaw, avec un PDG et de nombreux agents ?
</div>

Oui, via le **routage multi-agents** et les **sous-agents**. Vous pouvez créer un agent coordinateur
et plusieurs agents opérationnels avec leurs propres espaces de travail et modèles.

Cela dit, il faut surtout voir cela comme une **expérience amusante**. C’est très consommateur en jetons et souvent
moins efficace que d’utiliser un seul bot avec des sessions distinctes. Le modèle typique que nous envisageons est un seul bot avec lequel vous discutez, avec différentes sessions pour du travail en parallèle. Ce bot peut également créer des sous-agents si nécessaire.

Docs : [Routage multi-agents](/fr/concepts/multi-agent), [Sous-agents](/fr/tools/subagents), [CLI Agents](/fr/cli/agents).

<div id="why-did-context-get-truncated-midtask-how-do-i-prevent-it">
  ### Pourquoi le contexte a-t-il été tronqué en cours de tâche ? Comment l’éviter ?
</div>

Le contexte de session est limité par la fenêtre du modèle. Les conversations longues, les sorties d’outils volumineuses ou un grand nombre de fichiers peuvent déclencher une compression ou une troncature du contexte.

Que faire :

* Demandez à l’agent de résumer l’état actuel et de l’écrire dans un fichier.
* Utilisez `/compact` avant les tâches longues, et `/new` lorsque vous changez de sujet.
* Gardez le contexte important dans l’espace de travail et demandez à l’agent de le relire.
* Utilisez des sous-agents pour le travail long ou parallèle afin que la conversation principale reste plus compacte.
* Choisissez un modèle avec une fenêtre de contexte plus grande si cela arrive souvent.

<div id="how-do-i-completely-reset-openclaw-but-keep-it-installed">
  ### Comment réinitialiser complètement OpenClaw sans le désinstaller
</div>

Utilisez la commande de réinitialisation :

```bash
openclaw reset
```

Réinitialisation complète non interactive :

```bash
openclaw reset --scope full --yes --non-interactive
```

Relancez ensuite la procédure d’onboarding :

```bash
openclaw onboard --install-daemon
```

Notes :

* L’assistant d’onboarding propose également l’option **Reset** s’il détecte une configuration existante. Voir [Wizard](/fr/start/wizard).
* Si vous avez utilisé des profils (`--profile` / `OPENCLAW_PROFILE`), réinitialisez chaque répertoire d’état (par défaut : `~/.openclaw-<profile>`).
* Réinitialisation en dev : `openclaw gateway --dev --reset` (réservé au dev ; efface la configuration de dev + les identifiants + les sessions + l’espace de travail).

<div id="im-getting-context-too-large-errors-how-do-i-reset-or-compact">
  ### Je reçois des erreurs « context too large », comment puis-je réinitialiser ou compacter ?
</div>

Utilisez l’une des options suivantes :

* **Compacter** (conserve la conversation mais résume les anciens échanges) :
  ```
  /compact
  ```
  ou `/compact <instructions>` pour orienter le résumé.

* **Réinitialiser** (nouvel ID de session pour la même clé de conversation) :
  ```
  /new
  /reset
  ```

Si le problème persiste :

* Activez ou ajustez **la purge de session** (`agents.defaults.contextPruning`) pour supprimer les anciens résultats d’outils.
* Utilisez un modèle avec une fenêtre de contexte plus grande.

Docs : [Compaction](/fr/concepts/compaction), [Session pruning](/fr/concepts/session-pruning), [Session management](/fr/concepts/session).

<div id="why-am-i-seeing-llm-request-rejected-messagesncontentxtooluseinput-field-required">
  ### Pourquoi est-ce que je vois le message d’erreur « LLM request rejected: messagesNcontentXtooluseinput Field required » ?
</div>

Il s’agit d’une erreur de validation du fournisseur : le modèle a émis un bloc `tool_use` sans le
`input` requis. Cela signifie généralement que l’historique de la session est périmé ou corrompu (souvent après un long fil de discussion ou une modification d’outil/de schéma).

Solution : démarrez une nouvelle session avec `/new` (message autonome).

<div id="why-am-i-getting-heartbeat-messages-every-30-minutes">
  ### Pourquoi est-ce que je reçois des signaux de vie toutes les 30 minutes ?
</div>

Les signaux de vie s’exécutent toutes les **30 m** par défaut. Ajustez-les ou désactivez-les :

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "2h"   // ou « 0m » pour désactiver
      }
    }
  }
}
```

Si `HEARTBEAT.md` existe mais est effectivement vide (uniquement des lignes
blanches et des en-têtes Markdown comme `# Heading`), OpenClaw ignore l’exécution du signal de vie afin d’économiser des appels API.
Si le fichier est absent, le signal de vie s’exécute quand même et le modèle décide quoi faire.

Les remplacements par agent utilisent `agents.list[].heartbeat`. Docs : [Signal de vie](/fr/gateway/heartbeat).

<div id="do-i-need-to-add-a-bot-account-to-a-whatsapp-group">
  ### Dois-je ajouter un compte bot à un groupe WhatsApp
</div>

Non. OpenClaw s’exécute sur **votre propre compte**, donc si vous êtes dans le groupe, OpenClaw peut y accéder.
Par défaut, les réponses dans le groupe sont bloquées tant que vous n’avez pas autorisé les expéditeurs (`groupPolicy: "allowlist"`).

Si vous voulez que **vous seul** puissiez déclencher des réponses dans le groupe :

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
  ### Comment obtenir le JID d&#39;un groupe WhatsApp
</div>

Option 1 (la plus rapide) : consultez les journaux avec `tail` et envoyez un message de test dans le groupe :

```bash
openclaw logs --follow --json
```

Recherchez un `chatId` (ou `from`) se terminant par `@g.us`, par exemple :
`1234567890-1234567890@g.us`.

Option 2 (si déjà configuré ou présent dans la liste d’autorisation) : lister les groupes depuis la configuration :

```bash
openclaw directory groups list --channel whatsapp
```

Documentation : [WhatsApp](/fr/channels/whatsapp), [Directory](/fr/cli/directory), [Logs](/fr/cli/logs).

<div id="why-doesnt-openclaw-reply-in-a-group">
  ### Pourquoi OpenClaw ne répond-il pas dans un groupe
</div>

Deux causes fréquentes :

* La limitation par mention est activée (par défaut). Vous devez mentionner le bot avec @ (ou correspondre à `mentionPatterns`).
* Vous avez configuré `channels.whatsapp.groups` sans `"*"` et le groupe ne figure pas dans la liste d’autorisation.

Consultez [Groups](/fr/concepts/groups) et [Group messages](/fr/concepts/group-messages).

<div id="do-groupsthreads-share-context-with-dms">
  ### Les groupes / fils de discussion partagent-ils le contexte avec les messages privés ?
</div>

Par défaut, les discussions directes sont rattachées à la session principale. Les groupes / canaux ont leurs propres clés de session, et les sujets Telegram / fils Discord correspondent à des sessions distinctes. Voir [Groupes](/fr/concepts/groups) et [Messages de groupe](/fr/concepts/group-messages).

<div id="how-many-workspaces-and-agents-can-i-create">
  ### Combien d’espaces de travail et d’agents puis-je créer
</div>

Aucune limite stricte. Des dizaines (voire des centaines) sont possibles, mais surveillez :

* **La consommation d’espace disque :** les sessions et les journaux de conversation se trouvent sous `~/.openclaw/agents/<agentId>/sessions/`.
* **Le coût en jetons :** plus d’agents signifie plus d’utilisation concurrente des modèles.
* **La surcharge opérationnelle :** profils d’authentification par agent, espaces de travail et routage des canaux.

Conseils :

* Gardez un seul espace de travail **actif** par agent (`agents.defaults.workspace`).
* Supprimez les anciennes sessions (effacez les fichiers JSONL ou archivez les entrées ailleurs) si l’utilisation du disque augmente.
* Utilisez `openclaw doctor` pour repérer les espaces de travail orphelins et les incohérences de profils.

<div id="can-i-run-multiple-bots-or-chats-at-the-same-time-slack-and-how-should-i-set-that-up">
  ### Puis‑je exécuter plusieurs bots ou conversations Slack en même temps et comment dois‑je configurer cela
</div>

Oui. Utilisez le **routage multi‑agents** pour exécuter plusieurs agents isolés et router les messages entrants par
canal/compte/interlocuteur. Slack est pris en charge comme canal et peut être associé à des agents spécifiques.

L’accès au navigateur est puissant mais ne permet pas « de faire tout ce qu’un humain peut faire » : les protections anti‑bot, les CAPTCHA et l’authentification multifacteur (MFA) peuvent
toujours bloquer l’automatisation. Pour un contrôle du navigateur aussi fiable que possible, utilisez le relais via l’extension Chrome
sur la machine qui exécute le navigateur (et laissez le Gateway où vous voulez).

Configuration recommandée :

* Hôte Gateway toujours allumé (VPS/Mac mini).
* Un agent par rôle (liaisons).
* Canal(aux) Slack associé(s) à ces agents.
* Navigateur local via relais d’extension (ou un nœud) si nécessaire.

Docs : [Routage multi‑agents](/fr/concepts/multi-agent), [Slack](/fr/channels/slack),
[Navigateur](/fr/tools/browser), [Extension Chrome](/fr/tools/chrome-extension), [Nœuds](/fr/nodes).

<div id="models-defaults-selection-aliases-switching">
  ## Modèles : valeurs par défaut, sélection, alias, changement
</div>

<div id="what-is-the-default-model">
  ### Quel est le modèle par défaut ?
</div>

Le modèle par défaut d’OpenClaw est celui que vous définissez comme :

```
agents.defaults.model.primary
```

Les modèles sont identifiés sous la forme `provider/model` (exemple : `anthropic/claude-opus-4-5`). Si vous omettez le fournisseur, OpenClaw suppose actuellement `anthropic` comme valeur de repli temporaire liée à la dépréciation, mais vous devez quand même définir **explicitement** `provider/model`.

<div id="what-model-do-you-recommend">
  ### Quel modèle recommandez-vous
</div>

**Valeur par défaut recommandée :** `anthropic/claude-opus-4-5`.\
**Bonne alternative :** `anthropic/claude-sonnet-4-5`.\
**Fiable (moins de caractère) :** `openai/gpt-5.2` – presque aussi bon qu’Opus, avec simplement moins de personnalité.\
**Économique :** `zai/glm-4.7`.

MiniMax M2.1 a sa propre documentation : [MiniMax](/fr/providers/minimax) et
[Modèles locaux](/fr/gateway/local-models).

Règle générale : utilisez le **meilleur modèle que vous pouvez vous permettre** pour les tâches à forts enjeux, et un modèle moins coûteux pour le chat ou les résumés de routine. Vous pouvez router les modèles par agent et utiliser des sous-agents pour
paralléliser les tâches longues (chaque sous-agent consomme des jetons). Voir [Modèles](/fr/concepts/models) et
[Sous-agents](/fr/tools/subagents).

Avertissement important : les modèles plus faibles ou sur-quantifiés sont plus vulnérables aux attaques de type prompt
injection et aux comportements dangereux. Voir [Sécurité](/fr/gateway/security).

Plus de contexte : [Modèles](/fr/concepts/models).

<div id="can-i-use-selfhosted-models-llamacpp-vllm-ollama">
  ### Puis-je utiliser des modèles auto-hébergés llamacpp, vLLM, Ollama ?
</div>

Oui. Si votre serveur local expose une API compatible OpenAI, vous pouvez y
faire pointer un fournisseur personnalisé. Ollama est pris en charge
directement et constitue la solution la plus simple.

Note de sécurité : les modèles plus petits ou fortement quantifiés sont plus vulnérables à l’injection
de prompt. Nous recommandons fortement les **grands modèles** pour tout bot pouvant utiliser des outils.
Si vous souhaitez malgré tout utiliser de petits modèles, activez le sandbox et des listes d’autorisation strictes pour les outils.

Docs : [Ollama](/fr/providers/ollama), [Modèles locaux](/fr/gateway/local-models),
[Fournisseurs de modèles](/fr/concepts/model-providers), [Sécurité](/fr/gateway/security),
[Sandboxing](/fr/gateway/sandboxing).

<div id="how-do-i-switch-models-without-wiping-my-config">
  ### Comment changer de modèle sans effacer ma config
</div>

Utilisez des **commandes de modèle** ou modifiez uniquement les champs **model**. Évitez de remplacer toute la config.

Options sûres :

* `/model` dans le chat (rapide, par session)
* `openclaw models set ...` (met uniquement à jour la config de modèle)
* `openclaw configure --section models` (mode interactif)
* modifier `agents.defaults.model` dans `~/.openclaw/openclaw.json`

Évitez d’utiliser `config.apply` avec un objet partiel, sauf si vous voulez vraiment remplacer toute la config.
Si vous avez écrasé la config, restaurez à partir d’une sauvegarde ou relancez `openclaw doctor` pour réparer.

Docs : [Models](/fr/concepts/models), [Configure](/fr/cli/configure), [Config](/fr/cli/config), [Doctor](/fr/gateway/doctor).

<div id="what-do-openclaw-flawd-and-krill-use-for-models">
  ### Quels modèles sont utilisés par OpenClaw, Flawd et Krill ?
</div>

* **OpenClaw + Flawd :** Anthropic Opus (`anthropic/claude-opus-4-5`) - voir [Anthropic](/fr/providers/anthropic).
* **Krill :** MiniMax M2.1 (`minimax/MiniMax-M2.1`) - voir [MiniMax](/fr/providers/minimax).

<div id="how-do-i-switch-models-on-the-fly-without-restarting">
  ### Comment changer de modèle à la volée sans redémarrer ?
</div>

Utilisez la commande `/model` comme message autonome :

```
/model sonnet
/model haiku
/model opus
/model gpt
/model gpt-mini
/model gemini
/model gemini-flash
```

Vous pouvez afficher les modèles disponibles avec `/model`, `/model list` ou `/model status`.

`/model` (et `/model list`) affiche un sélecteur numéroté compact. Sélectionnez un modèle par son numéro :

```
/model 3
```

Vous pouvez également imposer un profil d’authentification spécifique pour le fournisseur (par session) :

```
/model opus@anthropic:default
/model opus@anthropic:work
```

Astuce : `/model status` affiche quel agent est actif, quel fichier `auth-profiles.json` est utilisé, et quel profil d&#39;authentification sera utilisé ensuite.
Il affiche aussi le point de terminaison (`baseUrl`) du fournisseur configuré et le mode d&#39;API (`api`) lorsqu&#39;ils sont disponibles.

**Comment désépingler un profil que j&#39;ai défini avec `profile`**

Relancez `/model` **sans** le suffixe `@profile` :

```
/model anthropic/claude-opus-4-5
```

Si vous souhaitez revenir à la valeur par défaut, sélectionnez-la dans `/model` (ou envoyez `/model &lt;fournisseur/modèle par défaut&gt;`).
Utilisez `/model status` pour vérifier quel profil d’authentification est actif.

<div id="can-i-use-gpt-52-for-daily-tasks-and-codex-52-for-coding">
  ### Puis-je utiliser GPT 5.2 pour les tâches quotidiennes et Codex 5.2 pour le code ?
</div>

Oui. Définissez-en un comme valeur par défaut et basculez selon les besoins :

* **Basculement rapide (par session)** : `/model gpt-5.2` pour les tâches quotidiennes, `/model gpt-5.2-codex` pour le code.
* **Valeur par défaut + bascule** : définissez `agents.defaults.model.primary` sur `openai-codex/gpt-5.2`, puis basculez vers `openai-codex/gpt-5.2-codex` pour coder (ou l’inverse).
* **Sous-agents :** orientez les tâches de programmation vers des sous-agents avec un modèle par défaut différent.

Voir [Modèles](/fr/concepts/models) et [Commandes slash](/fr/tools/slash-commands).

<div id="why-do-i-see-model-is-not-allowed-and-then-no-reply">
  ### Pourquoi vois-je « Model is not allowed » puis aucune réponse
</div>

Si `agents.defaults.models` est défini, il devient la **liste d’autorisation** pour `/model` et tout remplacement défini au niveau de la session. Choisir un modèle qui ne figure pas dans cette liste renvoie :

```
Model "provider/model" is not allowed. Use /model to list available models.
```

Cette erreur est renvoyée **au lieu** d’une réponse normale. Correctif : ajoutez le modèle à
`agents.defaults.models`, supprimez la liste d’autorisation ou choisissez un modèle à partir de `/model list`.

<div id="why-do-i-see-unknown-model-minimaxminimaxm21">
  ### Pourquoi est‑ce que je vois « Unknown model minimaxMiniMaxM21 »
</div>

Cela signifie que le **fournisseur n’est pas configuré** (aucune configuration ou
aucun profil d’authentification MiniMax n’a été trouvé), donc le modèle ne peut pas
être résolu. Une correction pour cette détection est incluse dans **2026.1.12**
(non publiée au moment de la rédaction).

Liste de vérifications :

1. Mettez à jour vers **2026.1.12** (ou exécutez depuis la branche `main`), puis redémarrez le Gateway.
2. Assurez‑vous que MiniMax est configuré (assistant de configuration ou JSON), ou qu’une clé API
   MiniMax existe dans les profils env/auth afin que le fournisseur puisse être injecté.
3. Utilisez l’identifiant de modèle exact (sensible à la casse) : `minimax/MiniMax-M2.1` ou
   `minimax/MiniMax-M2.1-lightning`.
4. Exécutez :
   ```bash
   openclaw models list
   ```
   et choisissez dans la liste (ou `/model list` dans le chat).

Voir [MiniMax](/fr/providers/minimax) et [Modèles](/fr/concepts/models).

<div id="can-i-use-minimax-as-my-default-and-openai-for-complex-tasks">
  ### Puis-je utiliser MiniMax comme valeur par défaut et OpenAI pour les tâches complexes ?
</div>

Oui. Utilisez **MiniMax par défaut** et changez de modèle **par session** si nécessaire.
Les bascules de secours sont prévues pour les **erreurs**, pas pour les « tâches complexes », donc utilisez `/model` ou un agent distinct.

**Option A : changer de modèle par session**

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

Ensuite :

```
/model gpt
```

**Option B : agents séparés**

* Agent A par défaut : MiniMax
* Agent B par défaut : OpenAI
* Router en fonction de l’agent ou utiliser `/agent` pour basculer

Docs : [Modèles](/fr/concepts/models), [Routage multi-agents](/fr/concepts/multi-agent), [MiniMax](/fr/providers/minimax), [OpenAI](/fr/providers/openai).

<div id="are-opus-sonnet-gpt-builtin-shortcuts">
  ### Les alias opus sonnet gpt sont-ils des raccourcis intégrés
</div>

Oui. OpenClaw est fourni avec quelques raccourcis par défaut (appliqués uniquement lorsque le modèle existe dans `agents.defaults.models`) :

* `opus` → `anthropic/claude-opus-4-5`
* `sonnet` → `anthropic/claude-sonnet-4-5`
* `gpt` → `openai/gpt-5.2`
* `gpt-mini` → `openai/gpt-5-mini`
* `gemini` → `google/gemini-3-pro-preview`
* `gemini-flash` → `google/gemini-3-flash-preview`

Si vous définissez votre propre alias avec le même nom, c’est votre valeur qui est utilisée.

<div id="how-do-i-defineoverride-model-shortcuts-aliases">
  ### Comment définir ou remplacer des alias de raccourcis de modèles
</div>

Les alias sont définis via `agents.defaults.models.<modelId>.alias`. Exemple :

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

Ensuite, `/model sonnet` (ou `/<alias>` lorsqu&#39;il est pris en charge) correspond à cet ID de modèle.

<div id="how-do-i-add-models-from-other-providers-like-openrouter-or-zai">
  ### Comment ajouter des modèles provenant d&#39;autres fournisseurs comme OpenRouter ou ZAI
</div>

OpenRouter (facturation à l&#39;usage, par token ; de nombreux modèles) :

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

Z.AI (modèles GLM) :

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

Si vous référencez un fournisseur/modèle mais que la clé du fournisseur requise est manquante, vous obtiendrez une erreur d’authentification à l’exécution (par exemple : `No API key found for provider "zai"`).

**Aucune clé API trouvée pour le fournisseur après l’ajout d’un nouvel agent**

Cela signifie généralement que le **nouvel agent** a un store d’authentification vide. L’authentification est gérée par agent et
stockée dans :

```
~/.openclaw/agents/<agentId>/agent/auth-profiles.json
```

Options de correction :

* Exécutez `openclaw agents add <id>` et configurez l’authentification pendant l’assistant de configuration.
* Ou copiez `auth-profiles.json` depuis le `agentDir` de l’agent principal vers le `agentDir` du nouvel agent.

**Ne réutilisez pas** `agentDir` entre plusieurs agents ; cela provoque des collisions d’authentification/de session.

<div id="model-failover-and-all-models-failed">
  ## Basculement entre modèles et « Tous les modèles ont échoué »
</div>

<div id="how-does-failover-work">
  ### Comment fonctionne le basculement
</div>

Le basculement se déroule en deux étapes :

1. **Rotation du profil d’authentification** au sein du même fournisseur.
2. **Basculement de modèle** vers le modèle suivant dans `agents.defaults.model.fallbacks`.

Des délais de temporisation (exponential backoff) sont appliqués aux profils en échec, ce qui permet à OpenClaw de continuer à répondre même lorsqu’un fournisseur est soumis à une limitation de débit ou subit des défaillances temporaires.

<div id="what-does-this-error-mean">
  ### Que signifie cette erreur ?
</div>

```
No credentials found for profile "anthropic:default"
```

Cela signifie que le système a tenté d’utiliser l’ID de profil d’authentification `anthropic:default`, mais n’a pas trouvé les identifiants correspondants dans le référentiel d’authentification prévu.

<div id="fix-checklist-for-no-credentials-found-for-profile-anthropicdefault">
  ### Liste de vérifications pour No credentials found for profile anthropicdefault
</div>

* **Confirmez où se trouvent les profils d’authentification** (nouveaux vs anciens chemins)
  * Actuel : `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`
  * Ancien : `~/.openclaw/agent/*` (migré par `openclaw doctor`)
* **Vérifiez que votre variable d’environnement est chargée par le Gateway**
  * Si vous définissez `ANTHROPIC_API_KEY` dans votre shell mais exécutez le Gateway via systemd/launchd, il se peut qu’il ne l’hérite pas. Placez‑la dans `~/.openclaw/.env` ou activez `env.shellEnv`.
* **Assurez‑vous que vous modifiez le bon agent**
  * Les configurations multi‑agents impliquent qu’il peut y avoir plusieurs fichiers `auth-profiles.json`.
* **Vérifiez rapidement l’état du modèle/de l’authentification**
  * Utilisez `openclaw models status` pour voir les modèles configurés et vérifier si les fournisseurs sont authentifiés.

**Liste de vérifications pour No credentials found for profile anthropic**

Cela signifie que l’exécution est liée à un profil d’authentification Anthropic, mais que le Gateway
ne le trouve pas dans son magasin d’authentification.

* **Utilisez un setup-token**
  * Exécutez `claude setup-token`, puis collez‑le avec `openclaw models auth setup-token --provider anthropic`.
  * Si le token a été créé sur une autre machine, utilisez `openclaw models auth paste-token --provider anthropic`.
* **Si vous préférez utiliser une clé API**
  * Placez `ANTHROPIC_API_KEY` dans `~/.openclaw/.env` sur l’**hôte du Gateway**.
  * Effacez tout ordre épinglé qui force un profil manquant :
    ```bash
    openclaw models auth order clear --provider anthropic
    ```
* **Vérifiez que vous exécutez les commandes sur l’hôte du Gateway**
  * En mode distant, les profils d’authentification résident sur la machine du Gateway, pas sur votre ordinateur portable.

<div id="why-did-it-also-try-google-gemini-and-fail">
  ### Pourquoi a‑t‑il aussi essayé Google Gemini, et échoué ?
</div>

Si ta configuration de modèle inclut Google Gemini comme solution de repli (ou si tu es passé à un raccourci Gemini), OpenClaw l’essaiera lors du repli de modèle. Si tu n’as pas configuré d’identifiants Google, tu verras `No API key found for provider "google"`.

Correction : soit tu fournis l’authentification Google, soit tu supprimes/évites les modèles Google dans `agents.defaults.model.fallbacks` / alias afin que le repli ne les utilise pas.

**Message « requête LLM rejetée car elle pense qu’une signature est requise pour google antigravity »**

Cause : l’historique de la session contient des **blocs de réflexion sans signature** (souvent issus
d’un flux interrompu/partiel). Google Antigravity exige des signatures pour les blocs de réflexion.

Correction : OpenClaw supprime désormais les blocs de réflexion non signés pour Google Antigravity Claude. Si le message apparaît encore, démarre une **nouvelle session** ou définis `/thinking off` pour cet agent.

<div id="auth-profiles-what-they-are-and-how-to-manage-them">
  ## Profils d’authentification : ce que c’est et comment les gérer
</div>

Voir aussi : [/concepts/oauth](/fr/concepts/oauth) (flux OAuth, stockage de jetons, modèles multi‑comptes)

<div id="what-is-an-auth-profile">
  ### Qu&#39;est-ce qu&#39;un profil d&#39;authentification ?
</div>

Un profil d&#39;authentification est un enregistrement nommé d&#39;identifiants (OAuth ou clé api) associé à un fournisseur. Les profils sont stockés dans :

```
~/.openclaw/agents/<agentId>/agent/auth-profiles.json
```

<div id="what-are-typical-profile-ids">
  ### Quels sont les ID de profil typiques
</div>

OpenClaw utilise des ID préfixés par le fournisseur, par exemple :

* `anthropic:default` (fréquent lorsqu’aucune identité e‑mail n’existe)
* `anthropic:<email>` pour les identités OAuth
* des ID personnalisés que vous définissez (par exemple `anthropic:work`)

<div id="can-i-control-which-auth-profile-is-tried-first">
  ### Puis-je contrôler quel profil d’authentification est essayé en premier ?
</div>

Oui. La configuration prend en charge des métadonnées optionnelles pour les profils et un ordre par fournisseur (`auth.order.<provider>`). Cela ne stocke **pas** de secrets ; cela associe des IDs à un fournisseur/mode et définit l’ordre de rotation.

OpenClaw peut ignorer temporairement un profil s’il est en court état de **cooldown** (limitations de débit, expirations, échecs d’authentification) ou dans un état **disabled** plus long (facturation/crédits insuffisants). Pour inspecter cela, exécutez `openclaw models status --json` et vérifiez `auth.unusableProfiles`. Paramétrage : `auth.cooldowns.billingBackoffHours*`.

Vous pouvez également définir une priorité **par agent** (enregistrée dans le fichier `auth-profiles.json` de cet agent) via la CLI :

```bash
# Defaults to the configured default agent (omit --agent)
openclaw models auth order get --provider anthropic

# Lock rotation to a single profile (only try this one)
openclaw models auth order set --provider anthropic anthropic:default

# Or set an explicit order (fallback within provider)
openclaw models auth order set --provider anthropic anthropic:work anthropic:default

# Effacer la substitution (revenir à config auth.order / round-robin)
openclaw models auth order clear --provider anthropic
```

Pour cibler un agent spécifique :

```bash
openclaw models auth order set --provider anthropic --agent main anthropic:default
```

<div id="oauth-vs-api-key-whats-the-difference">
  ### OAuth vs clé API : quelle est la différence ?
</div>

OpenClaw prend en charge les deux :

* **OAuth** s’appuie souvent sur un accès par abonnement (lorsque c’est possible).
* Les **clés API** utilisent une facturation à l’usage, au nombre de jetons.

L’assistant de configuration prend explicitement en charge le setup-token Anthropic et OAuth OpenAI Codex, et peut stocker des clés API pour vous.

<div id="gateway-ports-already-running-and-remote-mode">
  ## Gateway : ports, message « Already running » et mode distant
</div>

<div id="what-port-does-the-gateway-use">
  ### Quel port le Gateway utilise-t-il ?
</div>

`gateway.port` définit l’unique port multiplexé pour WebSocket et HTTP (Control UI, hooks, etc.).

Ordre de priorité :

```
--port > OPENCLAW_GATEWAY_PORT > gateway.port > default 18789
```

<div id="why-does-openclaw-gateway-status-say-runtime-running-but-rpc-probe-failed">
  ### Pourquoi `openclaw gateway status` affiche Runtime running mais RPC probe failed
</div>

Parce que « running » est le point de vue du **superviseur** (launchd/systemd/schtasks). Le RPC probe correspond à la CLI qui se connecte réellement au WebSocket du Gateway et exécute la commande `status`.

Utilisez `openclaw gateway status` et fiez-vous à ces lignes :

* `Probe target:` (l’URL réellement utilisée par la sonde)
* `Listening:` (ce qui est effectivement à l’écoute sur le port)
* `Last gateway error:` (cause principale fréquente lorsque le processus est vivant mais que le port n’écoute pas)

<div id="why-does-openclaw-gateway-status-show-config-cli-and-config-service-different">
  ### Pourquoi openclaw gateway status affiche-t-il des valeurs différentes pour Config cli et Config service
</div>

Vous modifiez un fichier de configuration tandis que le service en utilise un autre (souvent une incohérence entre `--profile` / `OPENCLAW_STATE_DIR`).

Solution :

```bash
openclaw gateway install --force
```

Exécutez cette commande dans le même `--profile`/environnement que celui que vous voulez que le service utilise.

<div id="what-does-another-gateway-instance-is-already-listening-mean">
  ### Que signifie « another gateway instance is already listening »
</div>

OpenClaw applique un verrou d’exécution en liant immédiatement l’écouteur WS (WebSocket) au démarrage (par défaut `ws://127.0.0.1:18789`). Si l’opération de bind échoue avec `EADDRINUSE`, il génère une erreur `GatewayLockError` indiquant qu’une autre instance est déjà à l’écoute.

Correctif : arrêtez l’autre instance, libérez le port ou exécutez `openclaw gateway --port <port>`.

<div id="how-do-i-run-openclaw-in-remote-mode-client-connects-to-a-gateway-elsewhere">
  ### Comment exécuter OpenClaw en mode distant, avec un client qui se connecte à un Gateway distant
</div>

Configurez `gateway.mode: "remote"` et pointez vers une URL WebSocket distante, éventuellement avec un jeton/mot de passe :

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

Remarques :

* `openclaw gateway` ne se lance que lorsque `gateway.mode` est `local` (ou si vous utilisez l’option d’override).
* L’app macOS surveille le fichier de configuration et bascule de mode à la volée lorsque ces valeurs changent.

<div id="the-control-ui-says-unauthorized-or-keeps-reconnecting-what-now">
  ### Le Control UI affiche « unauthorized » ou se reconnecte en boucle : que faire ?
</div>

Votre Gateway fonctionne avec l’authentification activée (`gateway.auth.*`), mais l’UI n’envoie pas le jeton/mot de passe correspondant.

Faits (d’après le code) :

* Le Control UI stocke le jeton dans la clé `openclaw.control.settings.v1` du localStorage du navigateur.
* L’UI peut importer `?token=...` (et/ou `?password=...`) une seule fois, puis le retirer de l’URL.

Solution :

* Le plus rapide : `openclaw dashboard` (affiche + copie un lien avec jeton, tente de l’ouvrir ; affiche une indication SSH si en mode headless).
* Si vous n’avez pas encore de jeton : `openclaw doctor --generate-gateway-token`.
* Si la Gateway est distante, créez d’abord un tunnel : `ssh -N -L 18789:127.0.0.1:18789 user@host` puis ouvrez `http://127.0.0.1:18789/?token=...`.
* Définissez `gateway.auth.token` (ou `OPENCLAW_GATEWAY_TOKEN`) sur l’hôte de la Gateway.
* Dans les paramètres du Control UI, collez le même jeton (ou actualisez avec un lien `?token=...` à usage unique).
* Toujours bloqué ? Lancez `openclaw status --all` et suivez [Troubleshooting](/fr/gateway/troubleshooting). Voir [Dashboard](/fr/web/dashboard) pour les détails d’authentification.

<div id="i-set-gatewaybind-tailnet-but-it-cant-bind-nothing-listens">
  ### J’ai défini gateway.bind sur `tailnet` mais il ne parvient pas à se lier et rien n’écoute
</div>

Le bind `tailnet` choisit une adresse IP Tailscale parmi vos interfaces réseau (100.64.0.0/10). Si la machine n’est pas sur Tailscale (ou que l’interface est down), il n’y a rien sur quoi se lier.

Correction :

* Démarrez Tailscale sur cet hôte (pour qu’il ait une adresse en 100.x), ou
* Passez à `gateway.bind: "loopback"` / `"lan"`.

Remarque : `tailnet` est explicite. `auto` privilégie loopback ; utilisez `gateway.bind: "tailnet"` lorsque vous voulez un bind uniquement sur le tailnet.

<div id="can-i-run-multiple-gateways-on-the-same-host">
  ### Puis-je exécuter plusieurs Gateways sur le même hôte
</div>

En général, non : un seul Gateway peut gérer plusieurs canaux de messagerie et agents. N’utilisez plusieurs Gateways que si vous avez besoin de redondance (ex. : bot de secours) ou d’isolation stricte.

Oui, mais vous devez isoler :

* `OPENCLAW_CONFIG_PATH` (configuration par instance)
* `OPENCLAW_STATE_DIR` (état par instance)
* `agents.defaults.workspace` (isolation de l’espace de travail)
* `gateway.port` (ports uniques)

Configuration rapide (recommandée) :

* Utilisez `openclaw --profile <name> …` par instance (crée automatiquement `~/.openclaw-<name>`).
* Définissez un `gateway.port` unique dans chaque configuration de profil (ou passez `--port` pour les exécutions manuelles).
* Installez un service par profil : `openclaw --profile <name> gateway install`.

Les profils ajoutent aussi un suffixe aux noms de service (`bot.molt.<profile>` ; anciennement `com.openclaw.*`, `openclaw-gateway-<profile>.service`, `OpenClaw Gateway (<profile>)`).
Guide complet : [Multiple gateways](/fr/gateway/multiple-gateways).

<div id="what-does-invalid-handshake-code-1008-mean">
  ### Que signifie le code de handshake invalide 1008
</div>

Le Gateway est un **serveur WebSocket** et il s’attend à ce que le tout premier message
soit une trame `connect`. S’il reçoit autre chose, il ferme la connexion
avec le **code 1008** (violation de politique).

Causes courantes :

* Vous avez ouvert l’URL **HTTP** dans un navigateur (`http://...`) au lieu d’un client WS.
* Vous avez utilisé le mauvais port ou le mauvais chemin d’accès.
* Un proxy ou un tunnel a supprimé les en‑têtes d’authentification ou a envoyé une requête qui ne cible pas le Gateway.

Correctifs rapides :

1. Utilisez l’URL WS : `ws://<host>:18789` (ou `wss://...` si HTTPS).
2. N’ouvrez pas le port WS dans un onglet de navigateur classique.
3. Si l’authentification est activée, incluez le jeton/mot de passe dans la trame `connect`.

Si vous utilisez la CLI ou la TUI, l’URL doit ressembler à :

```
openclaw tui --url ws://<host>:18789 --token <token>
```

Détails du protocole : [Protocole Gateway](/fr/gateway/protocol).

<div id="logging-and-debugging">
  ## Journaux et débogage
</div>

<div id="where-are-logs">
  ### Où se trouvent les journaux ?
</div>

Fichiers journaux (structurés) :

```
/tmp/openclaw/openclaw-YYYY-MM-DD.log
```

Vous pouvez définir un chemin fixe via `logging.file`. Le niveau de journalisation du fichier est contrôlé par `logging.level`. La verbosité de la console est contrôlée par `--verbose` et `logging.consoleLevel`.

Méthode la plus rapide pour suivre les journaux :

```bash
openclaw logs --follow
```

Journaux du service/superviseur (lorsque le Gateway s’exécute via launchd/systemd) :

* macOS : `$OPENCLAW_STATE_DIR/logs/gateway.log` et `gateway.err.log` (par défaut : `~/.openclaw/logs/...` ; les profils utilisent `~/.openclaw-<profile>/logs/...`)
* Linux : `journalctl --user -u openclaw-gateway[-<profile>].service -n 200 --no-pager`
* Windows : `schtasks /Query /TN "OpenClaw Gateway (<profile>)" /V /FO LIST`

Consultez la section [Dépannage](/fr/gateway/troubleshooting#log-locations) pour plus d’informations.

<div id="how-do-i-startstoprestart-the-gateway-service">
  ### Comment démarrer/arrêter/redémarrer le service Gateway
</div>

Utilisez les utilitaires du service Gateway :

```bash
openclaw gateway status
openclaw gateway restart
```

Si vous exécutez Gateway manuellement, `openclaw gateway --force` peut récupérer le port. Voir [Gateway](/fr/gateway).

<div id="i-closed-my-terminal-on-windows-how-do-i-restart-openclaw">
  ### J&#39;ai fermé mon terminal sous Windows, comment redémarrer OpenClaw ?
</div>

Il existe **deux modes d&#39;installation Windows** :

**1) WSL2 (recommandé) :** le Gateway s&#39;exécute sous Linux.

Ouvrez PowerShell, entrez dans WSL, puis redémarrez :

```powershell
wsl
openclaw gateway status
openclaw gateway restart
```

Si vous n&#39;avez encore jamais installé le service, lancez-le au premier plan :

```bash
openclaw gateway run
```

**2) Windows natif (non recommandé)\u00a0:** le Gateway s&#39;exécute directement sur Windows.

Ouvrez PowerShell et exécutez\u00a0:

```powershell
openclaw gateway status
openclaw gateway restart
```

Si vous l’exécutez manuellement (sans passer par un service), utilisez :

```powershell
openclaw gateway run
```

Documentation : [Windows (WSL2)](/fr/platforms/windows), [Runbook du service Gateway](/fr/gateway).

<div id="the-gateway-is-up-but-replies-never-arrive-what-should-i-check">
  ### Le Gateway est en ligne mais les réponses n’arrivent jamais — que dois-je vérifier ?
</div>

Commence par une vérification rapide de l’état de santé :

```bash
openclaw status
openclaw models status
openclaw channels status
openclaw logs --follow
```

Causes courantes :

* Authentification du modèle non chargée sur l’**hôte du Gateway** (vérifiez `models status`).
* Appairage du canal ou liste d’autorisation bloquant les réponses (vérifiez la configuration du canal et les journaux).
* WebChat ou le Dashboard est ouvert sans le bon jeton.

Si vous êtes à distance, confirmez que le tunnel ou la connexion Tailscale est actif et que le
WebSocket du Gateway est joignable.

Docs : [Canaux](/fr/channels), [Dépannage](/fr/gateway/troubleshooting), [Accès distant](/fr/gateway/remote).

<div id="disconnected-from-gateway-no-reason-what-now">
  ### Déconnecté du Gateway sans raison apparente : que faire maintenant ?
</div>

Cela signifie généralement que l’UI a perdu la connexion WebSocket. Vérifiez :

1. Le Gateway est-il en cours d’exécution ? `openclaw gateway status`
2. Le Gateway est-il en bon état de fonctionnement ? `openclaw status`
3. L’UI dispose-t-elle du bon jeton ? `openclaw dashboard`
4. En cas d’accès à distance, le tunnel ou le lien Tailscale est-il actif ?

Ensuite, affichez les journaux en continu :

```bash
openclaw logs --follow
```

Documentation : [Tableau de bord](/fr/web/dashboard), [Accès à distance](/fr/gateway/remote), [Dépannage](/fr/gateway/troubleshooting).

<div id="telegram-setmycommands-fails-with-network-errors-what-should-i-check">
  ### Telegram setMyCommands échoue avec des erreurs réseau : que dois-je vérifier ?
</div>

Commencez par consulter les logs et l’état du canal :

```bash
openclaw channels status
openclaw channels logs --channel telegram
```

Si vous êtes sur un VPS ou derrière un proxy, vérifiez que le trafic HTTPS sortant est autorisé et que la résolution DNS fonctionne.
Si le Gateway est distant, assurez-vous de consulter les journaux sur la machine hôte du Gateway.

Docs : [Telegram](/fr/channels/telegram), [Dépannage des canaux](/fr/channels/troubleshooting).

<div id="tui-shows-no-output-what-should-i-check">
  ### L&#39;interface TUI n&#39;affiche rien : que dois-je vérifier ?
</div>

Commencez par vérifier que le Gateway est joignable et que l&#39;agent peut s&#39;exécuter :

```bash
openclaw status
openclaw models status
openclaw logs --follow
```

Dans l’interface TUI, utilisez `/status` pour voir l’état actuel. Si vous attendez des réponses dans un canal de discussion,
assurez-vous que l’acheminement est activé (`/deliver on`).

Docs : [TUI](/fr/tui), [commandes slash](/fr/tools/slash-commands).

<div id="how-do-i-completely-stop-then-start-the-gateway">
  ### Comment arrêter complètement le Gateway, puis le démarrer à nouveau
</div>

Si vous avez installé le service :

```bash
openclaw gateway stop
openclaw gateway start
```

Cela arrête/démarre le **service supervisé** (launchd sur macOS, systemd sur Linux).
Utilisez ceci lorsque le Gateway s’exécute en arrière‑plan en tant que démon.

Si vous l’exécutez au premier plan, arrêtez‑le avec Ctrl‑C, puis :

```bash
openclaw gateway run
```

Docs : [Runbook du service Gateway](/fr/gateway).

<div id="eli5-openclaw-gateway-restart-vs-openclaw-gateway">
  ### ELI5 différence entre `openclaw gateway restart` et `openclaw gateway`
</div>

* `openclaw gateway restart` : redémarre le **service d’arrière-plan** (launchd/systemd).
* `openclaw gateway` : exécute le Gateway **au premier plan** pour cette session de terminal.

Si vous avez installé le service, utilisez les commandes gateway. Utilisez `openclaw gateway` quand
vous voulez une exécution ponctuelle au premier plan.

<div id="whats-the-fastest-way-to-get-more-details-when-something-fails">
  ### Quel est le moyen le plus rapide d’obtenir plus de détails lorsqu’une erreur survient ?
</div>

Démarrez le Gateway avec `--verbose` pour obtenir plus de détails dans la console. Ensuite, inspectez le fichier journal pour l’authentification des canaux, le routage des modèles et les erreurs RPC.

<div id="media-attachments">
  ## Médias et fichiers joints
</div>

<div id="my-skill-generated-an-imagepdf-but-nothing-was-sent">
  ### Mon skill a généré un PDF d’image mais rien n’a été envoyé
</div>

Les pièces jointes sortantes de l’agent doivent inclure une ligne `MEDIA:<path-or-url>` (sur sa propre ligne). Voir [Configuration de l’assistant OpenClaw](/fr/start/openclaw) et [Agent send](/fr/tools/agent-send).

Envoi via la CLI :

```bash
openclaw message send --target +15555550123 --message "Voici" --media /path/to/file.png
```

Vérifiez également :

* Le canal cible prend en charge l’envoi de médias sortants et n’est pas bloqué par une liste d’autorisation.
* Le fichier respecte les limites de taille imposées par le fournisseur (les images sont redimensionnées à un maximum de 2048 px).

Voir [Images](/fr/nodes/images).

<div id="security-and-access-control">
  ## Sécurité et contrôle des accès
</div>

<div id="is-it-safe-to-expose-openclaw-to-inbound-dms">
  ### Est-il sécurisé d’exposer OpenClaw à des DM entrants
</div>

Considérez les DM entrants comme une entrée non fiable. Les valeurs par défaut sont conçues pour réduire le risque :

* Le comportement par défaut sur les canaux compatibles DM est l’**appairage** :
  * Les expéditeurs inconnus reçoivent un code d’appairage ; le bot ne traite pas leur message.
  * Approuvez avec : `openclaw pairing approve <channel> <code>`
  * Les demandes en attente sont limitées à **3 par canal** ; vérifiez `openclaw pairing list <channel>` si un code n’est pas arrivé.
* L’ouverture publique des DM nécessite un opt‑in explicite (`dmPolicy: "open"` et `allowlist: "*"`).

Exécutez `openclaw doctor` pour identifier les stratégies DM à risque.

<div id="is-prompt-injection-only-a-concern-for-public-bots">
  ### L’injection de prompt est-elle uniquement un problème pour les bots publics ?
</div>

Non. L’injection de prompt concerne le **contenu non fiable**, pas seulement qui peut envoyer un message privé (DM) au bot.
Si votre assistant lit du contenu externe (recherche/fetch web, pages de navigateur, e‑mails,
documents, pièces jointes, journaux collés), ce contenu peut inclure des instructions qui tentent
de détourner le modèle. Cela peut arriver même si **vous êtes le seul expéditeur**.

Le plus grand risque survient lorsque les outils sont activés : le modèle peut être trompé pour
exfiltrer du contexte ou appeler des outils en votre nom. Réduisez le périmètre d’impact en :

* utilisant un agent « lecteur » en lecture seule ou avec outils désactivés pour résumer du contenu non fiable
* laissant `web_search` / `web_fetch` / `browser` désactivés pour les agents avec outils activés
* utilisant un sandbox et des listes d’autorisation d’outils strictes

Détails : [Sécurité](/fr/gateway/security).

<div id="should-my-bot-have-its-own-email-github-account-or-phone-number">
  ### Mon bot doit-il avoir sa propre adresse e-mail, son propre compte GitHub ou son propre numéro de téléphone ?
</div>

Oui, pour la plupart des configurations. Isoler le bot avec des comptes et des numéros
de téléphone séparés réduit la portée des dégâts si quelque chose se passe mal. Cela facilite
également le renouvellement des identifiants ou la révocation d&#39;accès sans affecter vos comptes
personnels.

Commencez de manière limitée. Donnez accès uniquement aux outils et comptes dont vous avez réellement
besoin, puis élargissez si nécessaire.

Docs : [Sécurité](/fr/gateway/security), [Appairage](/fr/start/pairing).

<div id="can-i-give-it-autonomy-over-my-text-messages-and-is-that-safe">
  ### Puis-je lui donner de l’autonomie sur mes messages texte, et est-ce sans danger ?
</div>

Nous déconseillons de lui donner une autonomie complète sur vos messages personnels. L’approche la plus sûre est :

* Conserver les messages privés (DM) en **mode d’appairage** ou avec une liste d’autorisation stricte.
* Utiliser un **numéro ou compte séparé** si vous voulez qu’il envoie des messages en votre nom.
* Le laisser rédiger, puis **approuver avant l’envoi**.

Si vous voulez expérimenter, faites-le avec un compte dédié et gardez-le isolé. Voir
[Sécurité](/fr/gateway/security).

<div id="can-i-use-cheaper-models-for-personal-assistant-tasks">
  ### Puis-je utiliser des modèles moins chers pour les tâches d’assistant personnel ?
</div>

Oui, **si** l’agent est uniquement conversationnel et que les données en entrée sont fiables. Les
modèles de gamme inférieure sont plus vulnérables au détournement d’instructions, donc évitez‑les
pour les agents avec outils activés ou lors de la lecture de contenu non fiable. Si
vous devez utiliser un modèle plus petit, verrouillez les outils et exécutez‑le dans
un sandbox. Voir [Sécurité](/fr/gateway/security).

<div id="i-ran-start-in-telegram-but-didnt-get-a-pairing-code">
  ### J’ai exécuté la commande /start dans Telegram mais je n’ai pas reçu de code d’appairage
</div>

Les codes d’appairage sont envoyés **uniquement** lorsqu’un expéditeur inconnu envoie un message au bot et que
`dmPolicy: "pairing"` est activé. `/start` seul ne génère pas de code.

Vérifiez les demandes en attente :

```bash
openclaw pairing list telegram
```

Si vous souhaitez un accès immédiat, ajoutez votre identifiant d’expéditeur à la liste d’autorisation ou définissez `dmPolicy: "open"` pour ce compte.

<div id="whatsapp-will-it-message-my-contacts-how-does-pairing-work">
  ### WhatsApp va-t-il envoyer des messages à mes contacts ? Comment fonctionne l’appairage ?
</div>

Non. La politique DM par défaut de WhatsApp est **l’appairage**. Les expéditeurs inconnus ne reçoivent qu’un code d’appairage et leur message **n’est pas traité**. OpenClaw ne répond qu’aux conversations qu’il reçoit ou aux envois explicites que vous déclenchez vous‑même.

Approuvez l’appairage avec :

```bash
openclaw pairing approve whatsapp <code>
```

Afficher les requêtes en attente :

```bash
openclaw pairing list whatsapp
```

Invite pour le numéro de téléphone de l’assistant : il est utilisé pour définir votre **liste d’autorisation/propriétaire** afin que vos propres messages privés (DM) soient autorisés. Ce numéro n’est pas utilisé pour des envois automatiques. Si vous utilisez votre numéro WhatsApp personnel, utilisez ce numéro et activez `channels.whatsapp.selfChatMode`.

<div id="chat-commands-aborting-tasks-and-it-wont-stop">
  ## Commandes de chat, annulation de tâches et « ça ne s’arrête plus »
</div>

<div id="how-do-i-stop-internal-system-messages-from-showing-in-chat">
  ### Comment empêcher l’affichage des messages système internes dans le chat
</div>

La plupart des messages internes ou des messages d’outils n’apparaissent que lorsque les modes **verbose** ou **reasoning** sont activés
pour cette session.

Corrigez cela dans le chat où vous les voyez :

```
/verbose off
/reasoning off
```

S&#39;il y a encore trop de bruit, vérifie les paramètres de la session dans le Control UI et règle `verbose`
sur **inherit**. Assure-toi également de ne pas utiliser un profil de bot avec `verboseDefault` défini
sur `on` dans la configuration.

Docs : [Raisonnement et mode verbeux](/fr/tools/thinking), [Sécurité](/fr/gateway/security#reasoning--verbose-output-in-groups).

<div id="how-do-i-stopcancel-a-running-task">
  ### Comment arrêter ou annuler une tâche en cours d&#39;exécution ?
</div>

Envoyez l&#39;un des messages suivants **comme message autonome** (sans slash) :

```
stop
abort
esc
wait
exit
interrupt
```

Ce sont des déclencheurs d’annulation (et non des commandes slash).

Pour les processus en arrière-plan (depuis l’outil `exec`), vous pouvez demander à l’agent d’exécuter :

```
process action:kill sessionId:XXX
```

Vue d’ensemble des commandes slash : voir [Commandes slash](/fr/tools/slash-commands).

La plupart des commandes doivent être envoyées dans un message **indépendant** qui commence par `/`, mais quelques raccourcis (comme `/status`) fonctionnent aussi en ligne pour les expéditeurs figurant sur la liste d’autorisation.

<div id="how-do-i-send-a-discord-message-from-telegram-crosscontext-messaging-denied">
  ### Comment envoyer un message Discord depuis Telegram — messagerie inter‑contexte refusée
</div>

OpenClaw bloque la messagerie **inter‑fournisseur** par défaut. Si un appel d’outil est lié
à Telegram, il n’enverra pas de message vers Discord, sauf si vous l’y autorisez explicitement.

Activez la messagerie inter‑fournisseur pour l’agent :

```json5
{
  agents: {
    defaults: {
      tools: {
        message: {
          crossContext: {
            allowAcrossProviders: true,
            marker: { enabled: true, prefix: "[de {channel}] " }
          }
        }
      }
    }
  }
}
```

Redémarrez Gateway après avoir modifié la configuration. Si vous ne voulez cela que pour un seul agent, définissez-le plutôt sous `agents.list[].tools.message`.

<div id="why-does-it-feel-like-the-bot-ignores-rapidfire-messages">
  ### Pourquoi a‑t‑on l&#39;impression que le bot ignore les messages envoyés en rafale ?
</div>

Le mode file d&#39;attente contrôle la façon dont les nouveaux messages interagissent avec une exécution en cours. Utilisez `/queue` pour changer de mode :

* `steer` - les nouveaux messages redirigent la tâche en cours
* `followup` - les messages sont traités un par un
* `collect` - les messages sont regroupés et une seule réponse est envoyée (par défaut)
* `steer-backlog` - réorienter immédiatement, puis traiter l&#39;arriéré
* `interrupt` - annuler l&#39;exécution en cours et repartir de zéro

Vous pouvez ajouter des options comme `debounce:2s cap:25 drop:summarize` pour les modes de suivi.

<div id="answer-the-exact-question-from-the-screenshotchat-log">
  ## Répondre exactement à la question tirée de la capture d’écran / du journal de conversation
</div>

**Q : « Quel est le modèle par défaut pour Anthropic avec une clé API ? »**

**R :** Dans OpenClaw, les identifiants et la sélection du modèle sont distincts. Définir `ANTHROPIC_API_KEY` (ou stocker une clé API Anthropic dans les profils d’authentification) active l’authentification, mais le véritable modèle par défaut est celui que vous configurez dans `agents.defaults.model.primary` (par exemple, `anthropic/claude-sonnet-4-5` ou `anthropic/claude-opus-4-5`). Si vous voyez `No credentials found for profile "anthropic:default"`, cela signifie que le Gateway n’a pas trouvé d’identifiants Anthropic dans le fichier `auth-profiles.json` attendu pour l’agent en cours d’exécution.

***

Toujours bloqué ? Posez votre question sur [Discord](https://discord.com/invite/clawd) ou ouvrez une [discussion GitHub](https://github.com/openclaw/openclaw/discussions).