---
title: Nostr
summary: "Nostr-DM-Kanal über NIP-04-verschlüsselte Nachrichten"
read_when:
  - Du möchtest, dass OpenClaw Direktnachrichten (DMs) über Nostr empfängt
  - Du richtest dezentralisiertes Messaging ein
---

<div id="nostr">
  # Nostr
</div>

**Status:** Optionales Plugin (standardmäßig deaktiviert).

Nostr ist ein dezentraler Protokollstandard für soziale Netzwerke. Dieser Kanal ermöglicht es OpenClaw, verschlüsselte Direktnachrichten (DMs) über NIP-04 zu empfangen und darauf zu antworten.

<div id="install-on-demand">
  ## Installieren (bei Bedarf)
</div>

<div id="onboarding-recommended">
  ### Onboarding (empfohlen)
</div>

- Der Onboarding-Assistent (`openclaw onboard`) und `openclaw channels add` führen optionale Channel-Plugins auf.
- Wenn du Nostr auswählst, wirst du aufgefordert, das Plugin bei Bedarf zu installieren.

Standardvorgaben für die Installation:

- **Dev-Channel + git-Checkout verfügbar:** verwendet den lokalen Plugin-Pfad.
- **Stable/Beta:** wird aus npm heruntergeladen.

Du kannst die Auswahl im Prompt jederzeit überschreiben.

<div id="manual-install">
  ### Manuelle Installation
</div>

```bash
openclaw plugins install @openclaw/nostr
```

Verwende ein lokales Git-Checkout (Dev-Workflows):

```bash
openclaw plugins install --link <path-to-openclaw>/extensions/nostr
```

Starte das Gateway nach der Installation oder Aktivierung von Plugins neu.


<div id="quick-setup">
  ## Schnellstart
</div>

1. Erzeuge ein Nostr-Schlüsselpaar (falls noch nicht vorhanden):

```bash
# Verwendung von nak
nak key generate
```

2. In die Konfiguration einfügen:

```json
{
  "channels": {
    "nostr": {
      "privateKey": "${NOSTR_PRIVATE_KEY}"
    }
  }
}
```

3. Exportieren Sie den Schlüssel:

```bash
export NOSTR_PRIVATE_KEY="nsec1..."
```

4. Starten Sie das Gateway neu.


<div id="configuration-reference">
  ## Konfigurationsreferenz
</div>

| Key | Typ | Standard | Beschreibung |
| --- | --- | --- | --- |
| `privateKey` | string | erforderlich | Privater Schlüssel im `nsec`- oder Hexformat |
| `relays` | string[] | `['wss://relay.damus.io', 'wss://nos.lol']` | Relay-URLs (WebSocket) |
| `dmPolicy` | string | `pairing` | DM-Zugriffsrichtlinie |
| `allowFrom` | string[] | `[]` | Zugelassene Absender-Pubkeys |
| `enabled` | boolean | `true` | Kanal aktivieren/deaktivieren |
| `name` | string | - | Anzeigename |
| `profile` | object | - | NIP-01-Profil-Metadaten |

<div id="profile-metadata">
  ## Profilmetadaten
</div>

Profildaten werden als NIP-01-Event mit `kind:0` veröffentlicht. Du kannst sie über die Control UI (Channels -&gt; Nostr -&gt; Profile) verwalten oder direkt in der Konfiguration festlegen.

Beispiel:

```json
{
  "channels": {
    "nostr": {
      "privateKey": "${NOSTR_PRIVATE_KEY}",
      "profile": {
        "name": "openclaw",
        "displayName": "OpenClaw",
        "about": "Persönlicher Assistent DM-Bot",
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

Hinweise:

* Profil-URLs müssen `https://` verwenden.
* Das Importieren von Relays führt Felder zusammen und behält lokale Überschreibungen bei.


<div id="access-control">
  ## Zugriffskontrolle
</div>

<div id="dm-policies">
  ### DM-Richtlinien
</div>

- **pairing** (Standard): Unbekannte Absender erhalten einen Pairing-Code.
- **allowlist**: Nur Pubkeys, die in `allowFrom` aufgeführt sind, können DMs senden.
- **open**: Öffentliche eingehende DMs (erfordert `allowFrom: ["*"]`).
- **disabled**: Eingehende DMs werden ignoriert.

<div id="allowlist-example">
  ### Beispiel für eine Allowlist
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
  ## Schlüsselformate
</div>

Akzeptierte Formate:

- **Privater Schlüssel:** `nsec...` oder 64-stelliger Hex-String
- **Pubkeys (`allowFrom`):** `npub...` oder Hex-String

<div id="relays">
  ## Relays
</div>

Standardmäßig sind `relay.damus.io` und `nos.lol`.

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

Tipps:

* Verwende 2–3 Relays für Redundanz.
* Vermeide zu viele Relays (Latenz, Duplikate).
* Kostenpflichtige Relays können die Zuverlässigkeit verbessern.
* Lokale Relays sind für Tests in Ordnung (`ws://localhost:7777`).


<div id="protocol-support">
  ## Protokollunterstützung
</div>

| NIP | Status | Beschreibung |
| --- | --- | --- |
| NIP-01 | Unterstützt | Grundlegendes Ereignisformat + Profilmetadaten |
| NIP-04 | Unterstützt | Verschlüsselte DMs (`kind:4`) |
| NIP-17 | Geplant | Gift-wrapped DMs |
| NIP-44 | Geplant | Versionierte Verschlüsselung |

<div id="testing">
  ## Tests
</div>

<div id="local-relay">
  ### Lokales Relay
</div>

```bash
# Starte strfry
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
  ### Manueller Test
</div>

1) Notiere dir den Bot-Pubkey (npub) aus den Logs.
2) Öffne einen Nostr-Client (Damus, Amethyst usw.).
3) Sende dem Bot-Pubkey eine Direktnachricht (DM).
4) Prüfe die Antwort.

<div id="troubleshooting">
  ## Fehlerbehebung
</div>

<div id="not-receiving-messages">
  ### Nachrichten werden nicht empfangen
</div>

- Überprüfe, ob der private Schlüssel gültig ist.
- Stelle sicher, dass Relay-URLs erreichbar sind und `wss://` verwenden (oder `ws://` für lokale Verbindungen).
- Stelle sicher, dass `enabled` nicht auf `false` gesetzt ist.
- Prüfe die Gateway-Logs auf Relay-Verbindungsfehler.

<div id="not-sending-responses">
  ### Antworten werden nicht gesendet
</div>

- Prüfe, ob das Relay Schreibzugriffe akzeptiert.
- Überprüfe die ausgehende Konnektivität.
- Achte auf Rate Limits des Relays.

<div id="duplicate-responses">
  ### Doppelte Antworten
</div>

- Bei Verwendung mehrerer Relays zu erwarten.
- Nachrichten werden anhand der Event-ID dedupliziert; nur die erste Zustellung löst eine Antwort aus.

<div id="security">
  ## Sicherheit
</div>

- Private Keys nie committen.
- Verwende Umgebungsvariablen für Schlüssel.
- Erwäge den Einsatz einer `allowlist` für Produktiv-Bots.

<div id="limitations-mvp">
  ## Einschränkungen (MVP)
</div>

- Nur Direktnachrichten (keine Gruppenchats).
- Keine Medienanhänge.
- Nur NIP-04 (NIP-17 Gift-Wrap geplant).