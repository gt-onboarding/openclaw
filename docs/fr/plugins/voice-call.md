---
title: Voice Call
summary: "Plugin Voice Call : appels vocaux sortants et entrants via Twilio/Telnyx/Plivo (installation du plugin, configuration et CLI)"
read_when:
  - Vous souhaitez passer un appel vocal sortant depuis OpenClaw
  - Vous configurez ou développez le plugin Voice Call
---

<div id="voice-call-plugin">
  # Voice Call (plugin)
</div>

Appels vocaux pour OpenClaw via un plugin. Prend en charge les notifications sortantes et
les conversations à plusieurs tours avec des stratégies pour les appels entrants.

Fournisseurs actuels :

- `twilio` (Programmable Voice + Media Streams)
- `telnyx` (Call Control v2)
- `plivo` (Voice API + transfert XML + GetInput speech)
- `mock` (dev/pas de réseau)

Modèle mental rapide :

- Installer le plugin
- Redémarrer le service Gateway
- Configurer sous `plugins.entries.voice-call.config`
- Utiliser `openclaw voicecall ...` ou l’outil `voice_call`

<div id="where-it-runs-local-vs-remote">
  ## Où il s'exécute (local vs distant)
</div>

Le plugin Voice Call s'exécute **au sein du processus Gateway**.

Si vous utilisez un Gateway distant, installez/configurez le plugin sur la **machine sur laquelle tourne le Gateway**, puis redémarrez le Gateway pour le charger.

<div id="install">
  ## Installation
</div>

<div id="option-a-install-from-npm-recommended">
  ### Option A : installation via npm (recommandée)
</div>

```bash
openclaw plugins install @openclaw/voice-call
```

Redémarrez ensuite le Gateway.


<div id="option-b-install-from-a-local-folder-dev-no-copying">
  ### Option B : installation à partir d’un dossier local (dev, sans copie)
</div>

```bash
openclaw plugins install ./extensions/voice-call
cd ./extensions/voice-call && pnpm install
```

Redémarrez ensuite le Gateway.


<div id="config">
  ## Configuration
</div>

Définissez la configuration dans `plugins.entries.voice-call.config` :

```json5
{
  plugins: {
    entries: {
      "voice-call": {
        enabled: true,
        config: {
          provider: "twilio", // ou "telnyx" | "plivo" | "mock"
          fromNumber: "+15550001234",
          toNumber: "+15550005678",

          twilio: {
            accountSid: "ACxxxxxxxx",
            authToken: "..."
          },

          plivo: {
            authId: "MAxxxxxxxxxxxxxxxxxxxx",
            authToken: "..."
          },

          // Webhook server
          serve: {
            port: 3334,
            path: "/voice/webhook"
          },

          // Public exposure (pick one)
          // publicUrl: "https://example.ngrok.app/voice/webhook",
          // tunnel: { provider: "ngrok" },
          // tailscale: { mode: "funnel", path: "/voice/webhook" }

          outbound: {
            defaultMode: "notify" // notify | conversation
          },

          streaming: {
            enabled: true,
            streamPath: "/voice/stream"
          }
        }
      }
    }
  }
}
```

Notes :

* Twilio/Telnyx requièrent une URL de webhook **publiquement accessible**.
* Plivo requiert une URL de webhook **publiquement accessible**.
* `mock` est un fournisseur de développement local (aucun appel réseau).
* `skipSignatureVerification` est réservé aux tests locaux.
* Si vous utilisez l’offre gratuite de ngrok, définissez `publicUrl` sur l’URL ngrok exacte ; la vérification de signature est toujours appliquée.
* `tunnel.allowNgrokFreeTierLoopbackBypass: true` autorise les webhooks Twilio avec des signatures invalides **uniquement** lorsque `tunnel.provider="ngrok"` et que `serve.bind` est en loopback (agent ngrok local). À n’utiliser que pour le développement local.
* Les URL ngrok de l’offre gratuite peuvent changer ou ajouter un écran intermédiaire (interstitiel) ; si `publicUrl` change, les signatures Twilio échoueront. En production, privilégiez un domaine stable ou Tailscale Funnel.


<div id="tts-for-calls">
  ## TTS pour les appels
</div>

Voice Call utilise la configuration principale `messages.tts` (OpenAI ou ElevenLabs) pour
la synthèse vocale en streaming lors des appels. Vous pouvez la redéfinir dans la configuration du plugin avec la
**même structure** — elle sera fusionnée en profondeur avec `messages.tts`.

```json5
{
  tts: {
    provider: "elevenlabs",
    elevenlabs: {
      voiceId: "pMsXgVXv3BLzUgSXRplE",
      modelId: "eleven_multilingual_v2"
    }
  }
}
```

Notes :

* **Edge TTS est ignoré pour les appels vocaux** (l’audio téléphonique doit être en PCM ; la sortie Edge est peu fiable).
* Core TTS est utilisé lorsque le streaming média Twilio est activé ; sinon, les appels utilisent les voix natives du fournisseur.


<div id="more-examples">
  ### Plus d’exemples
</div>

Utiliser uniquement le TTS de base (sans remplacement) :

```json5
{
  messages: {
    tts: {
      provider: "openai",
      openai: { voice: "alloy" }
    }
  }
}
```

Forcer ElevenLabs uniquement pour les appels (conserver la valeur par défaut du noyau pour le reste) :

```json5
{
  plugins: {
    entries: {
      "voice-call": {
        config: {
          tts: {
            provider: "elevenlabs",
            elevenlabs: {
              apiKey: "elevenlabs_key",
              voiceId: "pMsXgVXv3BLzUgSXRplE",
              modelId: "eleven_multilingual_v2"
            }
          }
        }
      }
    }
  }
}
```

Remplacer uniquement le modèle OpenAI pour les appels (exemple de fusion en profondeur) :

```json5
{
  plugins: {
    entries: {
      "voice-call": {
        config: {
          tts: {
            openai: {
              model: "gpt-4o-mini-tts",
              voice: "marin"
            }
          }
        }
      }
    }
  }
}
```


<div id="inbound-calls">
  ## Appels entrants
</div>

Par défaut, la stratégie pour les appels entrants est définie sur `disabled`. Pour activer les appels entrants, définissez :

```json5
{
  inboundPolicy: "allowlist",
  allowFrom: ["+15550001234"],
  inboundGreeting: "Bonjour ! Comment puis-je vous aider ?"
}
```

Les réponses automatiques utilisent le système d&#39;agents. Paramétrez via :

* `responseModel`
* `responseSystemPrompt`
* `responseTimeoutMs`


<div id="cli">
  ## CLI
</div>

```bash
openclaw voicecall call --to "+15555550123" --message "Hello from OpenClaw"
openclaw voicecall continue --call-id <id> --message "Any questions?"
openclaw voicecall speak --call-id <id> --message "One moment"
openclaw voicecall end --call-id <id>
openclaw voicecall status --call-id <id>
openclaw voicecall tail
openclaw voicecall expose --mode funnel
```


<div id="agent-tool">
  ## Outil d’agent
</div>

Nom de l’outil : `voice_call`

Actions :

- `initiate_call` (message, to?, mode?)
- `continue_call` (callId, message)
- `speak_to_user` (callId, message)
- `end_call` (callId)
- `get_status` (callId)

Ce dépôt inclut une documentation de compétence associée dans `skills/voice-call/SKILL.md`.

<div id="gateway-rpc">
  ## Appels RPC du Gateway
</div>

- `voicecall.initiate` (`to?`, `message`, `mode?`)
- `voicecall.continue` (`callId`, `message`)
- `voicecall.speak` (`callId`, `message`)
- `voicecall.end` (`callId`)
- `voicecall.status` (`callId`)