---
title: Nostr
summary: "Canale DM Nostr tramite messaggi NIP-04 crittografati"
read_when:
  - Vuoi che OpenClaw riceva DM tramite Nostr
  - Stai configurando la messaggistica decentralizzata
---

<div id="nostr">
  # Nostr
</div>

**Stato:** plugin opzionale (disabilitato per impostazione predefinita).

Nostr è un protocollo decentralizzato per social networking. Questo canale consente a OpenClaw di ricevere e rispondere a messaggi diretti crittografati (DM) tramite NIP-04.

<div id="install-on-demand">
  ## Installazione (su richiesta)
</div>

<div id="onboarding-recommended">
  ### Onboarding (consigliato)
</div>

- La procedura guidata di onboarding (`openclaw onboard`) e `openclaw channels add` elencano i plugin di canale opzionali.
- Selezionando Nostr ti viene richiesto di installare il plugin al bisogno.

Impostazioni predefinite per l'installazione:

- **Canale di sviluppo + git checkout disponibile:** usa il percorso locale del plugin.
- **Stable/Beta:** scarica da npm.

Puoi sempre modificare la scelta quando ti viene richiesto dal prompt.

<div id="manual-install">
  ### Installazione manuale
</div>

```bash
openclaw plugins install @openclaw/nostr
```

Usa un checkout locale del repository (workflow di sviluppo):

```bash
openclaw plugins install --link <path-to-openclaw>/extensions/nostr
```

Riavvia il Gateway dopo aver installato o abilitato i plugin.


<div id="quick-setup">
  ## Configurazione rapida
</div>

1. Genera una coppia di chiavi per Nostr (se necessario):

```bash
# Using nak
nak key generate
```

2. Aggiungi alla configurazione:

```json
{
  "channels": {
    "nostr": {
      "privateKey": "${NOSTR_PRIVATE_KEY}"
    }
  }
}
```

3. Esporta la chiave:

```bash
export NOSTR_PRIVATE_KEY="nsec1..."
```

4. Riavvia il Gateway.


<div id="configuration-reference">
  ## Riferimento di configurazione
</div>

| Key | Tipo | Predefinito | Descrizione |
| --- | --- | --- | --- |
| `privateKey` | string | obbligatorio | Chiave privata in formato `nsec` o esadecimale |
| `relays` | string[] | `['wss://relay.damus.io', 'wss://nos.lol']` | URL dei relay (WebSocket) |
| `dmPolicy` | string | `pairing` | Criterio di accesso ai DM (messaggi diretti) |
| `allowFrom` | string[] | `[]` | Chiavi pubbliche dei mittenti consentiti |
| `enabled` | boolean | `true` | Abilita/disabilita il canale |
| `name` | string | - | Nome visualizzato |
| `profile` | object | - | Metadati del profilo NIP-01 |

<div id="profile-metadata">
  ## Metadati del profilo
</div>

I dati del profilo vengono pubblicati come evento NIP-01 `kind:0`. Puoi gestirli dalla Control UI (Channels -&gt; Nostr -&gt; Profile) oppure configurarli direttamente nel file di configurazione.

Esempio:

```json
{
  "channels": {
    "nostr": {
      "privateKey": "${NOSTR_PRIVATE_KEY}",
      "profile": {
        "name": "openclaw",
        "displayName": "OpenClaw",
        "about": "Personal assistant DM bot",
        "picture": "https://example.com/avatar.png",
        "banner": "https://example.com/banner.png",
        "website": "https://example.com",
        "nip05": "openclaw@example.com",
        "lud16": "openclaw@example.com"
      }
    }
  }
}
```

Note:

* Gli URL dei profili devono utilizzare `https://`.
* L&#39;importazione dai relay unisce i campi e conserva le modifiche locali.


<div id="access-control">
  ## Controllo degli accessi
</div>

<div id="dm-policies">
  ### Criteri per i DM
</div>

- **abbinamento** (predefinito): i mittenti sconosciuti ricevono un codice di abbinamento.
- **lista di autorizzati**: solo le pubkey presenti in `allowFrom` possono inviare DM.
- **open**: DM in ingresso pubblici da qualsiasi utente (richiede `allowFrom: ["*"]`).
- **disabilitato**: ignora i DM in ingresso.

<div id="allowlist-example">
  ### Esempio di lista di autorizzati
</div>

```json
{
  "channels": {
    "nostr": {
      "privateKey": "${NOSTR_PRIVATE_KEY}",
      "dmPolicy": "allowlist",
      "allowFrom": ["npub1abc...", "npub1xyz..."]
    }
  }
}
```


<div id="key-formats">
  ## Formati principali
</div>

Formati accettati:

- **Chiave privata:** `nsec...` oppure esadecimale a 64 caratteri
- **Chiavi pubbliche (`allowFrom`):** `npub...` oppure in formato esadecimale

<div id="relays">
  ## Relay
</div>

Valori predefiniti: `relay.damus.io` e `nos.lol`.

```json
{
  "channels": {
    "nostr": {
      "privateKey": "${NOSTR_PRIVATE_KEY}",
      "relays": [
        "wss://relay.damus.io",
        "wss://relay.primal.net",
        "wss://nostr.wine"
      ]
    }
  }
}
```

Suggerimenti:

* Utilizza 2-3 relay per la ridondanza.
* Evita un numero eccessivo di relay (latenza, duplicazione).
* I relay a pagamento possono migliorare l&#39;affidabilità.
* I relay locali vanno bene per i test (`ws://localhost:7777`).


<div id="protocol-support">
  ## Supporto ai protocolli
</div>

| NIP | Stato | Descrizione |
| --- | --- | --- |
| NIP-01 | Supportato | Formato base degli eventi + metadati del profilo |
| NIP-04 | Supportato | DM crittografati (`kind:4`) |
| NIP-17 | Previsto | DM “gift-wrapped” |
| NIP-44 | Previsto | Crittografia con versioni |

<div id="testing">
  ## Test
</div>

<div id="local-relay">
  ### Relay locale
</div>

```bash
# Avvia strfry
docker run -p 7777:7777 ghcr.io/hoytech/strfry
```

```json
{
  "channels": {
    "nostr": {
      "privateKey": "${NOSTR_PRIVATE_KEY}",
      "relays": ["ws://localhost:7777"]
    }
  }
}
```


<div id="manual-test">
  ### Test manuale
</div>

1) Annota la pubkey (npub) del bot dai log.
2) Apri un client Nostr (Damus, Amethyst, ecc.).
3) Invia un DM alla pubkey del bot.
4) Verifica la risposta.

<div id="troubleshooting">
  ## Risoluzione dei problemi
</div>

<div id="not-receiving-messages">
  ### Non stai ricevendo messaggi
</div>

- Verifica che la chiave privata sia valida.
- Assicurati che gli URL di relay siano raggiungibili e usino `wss://` (o `ws://` in locale).
- Verifica che `enabled` non sia impostato su `false`.
- Controlla i log del Gateway per eventuali errori di connessione ai relay.

<div id="not-sending-responses">
  ### Le risposte non vengono inviate
</div>

- Verifica che il relay accetti operazioni di scrittura.
- Verifica la connettività in uscita.
- Controlla gli eventuali limiti di frequenza imposti dal relay.

<div id="duplicate-responses">
  ### Risposte duplicate
</div>

- Comportamento previsto quando utilizzi più relay.
- I messaggi vengono deduplicati in base all'ID dell'evento; solo la prima consegna attiva una risposta.

<div id="security">
  ## Sicurezza
</div>

- Non eseguire mai il commit delle chiavi private.
- Usa variabili d'ambiente per le chiavi.
- Valuta una lista di autorizzati (`allowlist`) per i bot in produzione.

<div id="limitations-mvp">
  ## Limitazioni (MVP)
</div>

- Solo messaggi diretti (niente chat di gruppo).
- Nessun allegato multimediale.
- Solo NIP-04 (supporto a NIP-17 gift-wrap previsto).