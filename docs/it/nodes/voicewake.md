---
title: Voicewake
summary: "Parole di attivazione vocale globali (gestite dal Gateway) e come vengono sincronizzate tra i nodi"
read_when:
  - Modificare il comportamento o i valori predefiniti delle parole di attivazione vocale
  - Aggiungere nuove piattaforme nodo che necessitano della sincronizzazione delle parole di attivazione vocale
---

<div id="voice-wake-global-wake-words">
  # Attivazione vocale (wake word globali)
</div>

OpenClaw tratta **le wake word come un singolo elenco globale** di proprietà del **Gateway**.

- **Non esistono wake word personalizzate per singolo nodo**.
- **Qualsiasi UI di nodo/app può modificare** l'elenco; le modifiche vengono salvate dal Gateway e distribuite a tutti.
- Ogni dispositivo mantiene comunque il proprio interruttore locale **Attivazione vocale abilitata/disabilitata** (l'UX locale e i permessi possono differire).

<div id="storage-gateway-host">
  ## Archiviazione (host del Gateway)
</div>

Le wake words sono memorizzate sulla macchina che ospita il Gateway in:

* `~/.openclaw/settings/voicewake.json`

Struttura:

```json
{ "triggers": ["openclaw", "claude", "computer"], "updatedAtMs": 1730000000000 }
```


<div id="protocol">
  ## Protocollo
</div>

<div id="methods">
  ### Metodi
</div>

- `voicewake.get` → `{ triggers: string[] }`
- `voicewake.set` con parametri `{ triggers: string[] }` → `{ triggers: string[] }`

Note:

- I trigger vengono normalizzati (spazi rimossi, elementi vuoti scartati). Le liste vuote vengono riportate ai valori predefiniti.
- I limiti vengono applicati per motivi di sicurezza (limiti su quantità e lunghezza).

<div id="events">
  ### Eventi
</div>

- `voicewake.changed` payload `{ triggers: string[] }`

Chi riceve questo evento:

- Tutti i client WebSocket (app macOS, WebChat, ecc.)
- Tutti i nodi connessi (iOS/Android), e anche al momento della connessione del nodo, come push iniziale dello stato corrente.

<div id="client-behavior">
  ## Comportamento del client
</div>

<div id="macos-app">
  ### app macOS
</div>

- Usa la lista globale per controllare i trigger di `VoiceWakeRuntime`.
- La modifica delle “Parole di attivazione” nelle impostazioni di Voice Wake richiama `voicewake.set` e poi si affida al broadcast per mantenere sincronizzati gli altri client.

<div id="ios-node">
  ### nodo iOS
</div>

- Usa l'elenco globale per il rilevamento dei trigger di `VoiceWakeManager`.
- La modifica delle wake word nelle Impostazioni invoca `voicewake.set` (tramite il WS del Gateway) e mantiene anche reattivo il rilevamento locale delle wake word.

<div id="android-node">
  ### Nodo Android
</div>

- Espone un editor di Wake Words nelle impostazioni.
- Invia `voicewake.set` tramite il WS del Gateway in modo che le modifiche si sincronizzino ovunque.